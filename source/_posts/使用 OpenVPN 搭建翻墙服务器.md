---
title: "使用 OpenVPN 搭建翻墙服务器"
date: 2016-10-24 21:40
categories: coding
tags:  
  - OpenVPN
---


## 已有教程

网上教程上介绍的均为使用国外的 vps (Ubuntu) 搭建的 OpenVPN Server，按照教程上一步步进行即可，故不再赘述。本教程所介绍的内容，**针对网上教程中的一些错误点。进行改成，并且提供了在 windows、Ubuntu和 MAC 上搭建 OpenVPN Client 的方法，亲测可用。**

在使用 OpenVPN 创建翻墙服务器的时候，可以使用 ipv4 和 ipv6（需要网络支持 ipv6） 模式。一般 [vultr](https://www.vultr.com/) 的 ipv4 地址没有被墙，[digital ocean](https://www.digitalocean.com/) 要看脸。

**下面两篇教程是使用 ipv6 网络来翻墙的教程，如果需要使用 ipv4 来翻墙，只需要添加 ipv4 地址，并且保证 ipv4 地址没有没墙即可。**

[校园网免流量之基于IPV6的OPENVPN搭建 _ Timothy's Blog.pdf](http://ofjld69e3.bkt.clouddn.com/doc/pdf/%E6%A0%A1%E5%9B%AD%E7%BD%91%E5%85%8D%E6%B5%81%E9%87%8F%E4%B9%8B%E5%9F%BA%E4%BA%8EIPV6%E7%9A%84OPENVPN%E6%90%AD%E5%BB%BA%20_%20Timothy%27s%20Blog.pdf)

[【教程】使用IPv6 Openvpn加速访问外国网站_pt吧_百度贴吧.pdf](http://ofjld69e3.bkt.clouddn.com/doc/pdf/%E3%80%90%E6%95%99%E7%A8%8B%E3%80%91%E4%BD%BF%E7%94%A8IPv6%20Openvpn%E5%8A%A0%E9%80%9F%E8%AE%BF%E9%97%AE%E5%A4%96%E5%9B%BD%E7%BD%91%E7%AB%99_pt%E5%90%A7_%E7%99%BE%E5%BA%A6%E8%B4%B4%E5%90%A7.pdf)

**上面提供的教程，在一些地方已经无法配置成功。所以在按照上面的教程配置的时候，需要参考下面的修改点，对其中的一些步骤进行修改。**

## 修改点与注意点

<!--more-->

### OpenVPN Server 版本

网上下载的 openvpn 版本无法使用，所以使用下面的 2.x.zip 中的 openvpn 版本，然后进行 unzip 2.x.zip 解压在/etc/下面，用来代替/etc/openvpn

[openvpn 2.x.zip](http://ofjm4ift4.bkt.clouddn.com/app/zip/openvpn%202.x.zip)

### 开机启动 OpenVPN Server

在 vps 中设置开机启动 openvpn，需要在 /etc/rc.local 中添加 

	/etc/init.d/openvpn start

来进行开机启动

### server.conf 修改

教程中使用的 openvpn 配置文件 server.conf 中有错误，要使用下面的server.conf，然后将 server.conf 放在 /etc/openvpn/下

[server.conf](http://ofjld69e3.bkt.clouddn.com/doc/conf/server.conf)

### 配置 server

在配置完 server.conf 后，需要将 /etc/openvpn/easy-rsa/2.0/key/下面的 server 的配置放在 server.conf 一样的位置。需要拷贝过来的配置有：ca.crt、server.crt、ta.key、dh2048.pem、server.key

### 添加 NAT 规则

在配置的过程中出现了以下问题：在 vps 开启的时候没有办法自动添加 iptables 中设置的 nat 规则，所以在 /etc/rc.local 中添加 iptables 规则：

	 iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o venet0 -j MASQUERADE

### 端口监听

在 openvpn 中设置的端口 1194 作为 ipv4 的端口，ipv6 端口为 1195。但是收到的 ipv6 信息都从 1195 端口使用 socat 转到 1194 端口进行处理，在 /etc/rc.local 中添加开机启动命令：
	
	socat UDP6-LISTEN:1195,reuseaddr,fork UDP4:127.0.0.1:1194 &

### 用户创建与配置

在为一个新的用户创建了权限以后，将 用户名.crt、用户名.key、ca.crt、ta.key（传输加密） 添加到 config 目录下面，修改所有的 .ovpn 中对应的部分

Windows OpenVPN Client 需要的 .opvn 文件可以从下面链接下载，根据你的 vps 和 创建的用户，将中文部分替换为你的信息。

[DigitalOcean_ipv4.ovpn](http://ofjld69e3.bkt.clouddn.com/doc/opvn/DigitalOcean_ipv4.ovpn)

[DigitalOcean_ipv6.ovpn](http://ofjld69e3.bkt.clouddn.com/doc/opvn/DigitalOcean_ipv6.ovpn)

### 删除用户

如果要删除一个用户，可以使用 /etc/openvpn/easy-rsa/2.0/ 下面的 revoke-full username 来删除一个用户的连接权限

## 配置 OpenVPN Client

### 安装与配置 Ubuntu 可用的 OpenVPN Client

安装 OpenVPN

	sudo apt-get install openvpn

然后把生成的秘钥等文件（.opvn 文件 ca.crt文件 .crt 文件 .key 文件 ta.key 文件）拷贝在 /etc/openvpn/config/ 目录下,在 ~/.zshrc 下添加alias命令，用来生成方便启动的命令。例如我的用户名为 Dane，则：

	alias openvpnconn="sudo /usr/sbin/openvpn --config /etc/openvpn/config/DigitalOcean_ipv6.ovpn --ca /etc/openvpn/config/ca.crt --cert /etc/openvpn/config/Dane.crt --key /etc/openvpn/config/Dane.key --tls-auth /etc/openvpn/config/ta.key"

修改 /etc/rc.local ，添加

	source ~/.zshrc

添加开机启动命令

修改 /etc/resolv.conf 添加 

	nameserver 8.8.8.8 nameserver 8.8.4.4
	nameserver 4.2.2.1 nameserver 4.2.2.2

并且将这个文件使用命令设置成只读 

	sudo chattr +i /etc/resolv.conf

**如果报错 chattr: Operation not supported while reading flags...**

需要卸除resolvconf模块。卸除方法是执行：

	apt-get remove resolvconf

### 配置 MAC 可用的 OpenVPN Client

MAC OpenVPN 的安装与密钥的拷贝与 Ubuntu 相同。

**在MAC下使用openvpn的时候，会显示找不到 /etc/tun，这时候需要安装 tun。**

由于不确定是使用哪种 tun 软件，所以全安装了。。。

	brew install openvpn
	brew install utun
	brew install tun
	brew install tuntap

### 配置 Windows 可用的 OpenVPN Client

Windows 平台可用的 OpenVPN Client 可以在[国科学技术大学网络OpenVPN系统](http://openvpn.ustc.edu.cn/)，找到对应的版本进行下载安装。

在下载 OpenVPN Client 并且完成安装之后，需要在 OpenVPN 安装的根目录创建一个叫做 config 文件夹，并且在文件夹中拷贝 Server 创建的密钥等文件，同时将 .opvn 文件也拷贝在 config 目录内，运行 OpenVPN 即可。






