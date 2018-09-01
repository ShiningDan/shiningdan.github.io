---
title: Docker学习笔记
date: 2018-04-21 17:23:29
categories: coding
tags:
  - Docker
---

本文是我学习 Docker 的资料

<!--more-->

在国内下载 Docker，可以去 DaoCloud 提供的镜像站点下载 [Docker 极速下载 | DaoCloud](https://get.daocloud.io/)。

## Docker 常用命令

```
docker images  //查看所有镜像列表

docker pull 镜像名称 // 下载一个镜像

docker rmi 镜像名称 // 删除一个镜像

docker ps    //查看运行的容器

docker ps -a  //查看所有容器

docker ps -l  //查看上一个执行的容器

docker start ID  //运行一个容器

docker stop ID  // 停止运行一个容器

docker rm ID // 删除一个容器

docker exec -t -i 容器名称 /bin/bash    //进入到容器内部命令

dock

docker logs 容器名称     //查看容器日志

docker inspect container/image  // 有关容器和镜像的底层信息

docker run -d -p 80:80 --name webserver -v /fromVolume:/toVolume nginx   //第一次开启容器
```

## docker 相关的工具

### Docker Machine

Docker Machine是一个工具，用来在虚拟主机上安装Docker Engine，并使用 `docker-machine` 命令来管理这些虚拟主机。

### Docker Compose

Docker Compose是Docker编排服务的最后一块，前面提到的Machine可以让用户在其它平台快速安装Docker，Swarm可以让Docker容器在集群中高效运转，而Compose可以让用户在集群中部署分布式应用。简单的说，Docker Compose属于一个“应用层”的服务，用户可以定义哪个容器组运行哪个应用，它支持动态改变应用，并在需要时扩展。

### Docker Daemon

Docker Daemon — docker 的守护进程，属于C/S中的server

### Docker REST API 

Docker REST API — docker daemon向外暴露的REST 接口

### Docker CLI

Docker CLI — docker向外暴露的命令行接口（Command Line API）

## docker 命令

**搜索镜像**：

```
docker search nginx
```

**镜像相关操作**：

```
docker image xxx
// 查看镜像
docker image ls
// 删除镜像
docler rmi nginx
// 下载镜像
docker pull nginx
```

**导出镜像**：

```
[root@tiejiang ~]# docker save -o nginx.tar daocloud.io/library/nginx

[root@tiejiang ~]# docker save -o cnetos.tar centos
```

导入镜像

```
[root@tiejiang ~]# docker load --input cnetos.tar 或者 [root@docker ~]# docker load < cnetos.tar
```

**查看所有的容器，包括未运行的**：

```
docker ps -a
```

**容器相关操作**

```
[root@]# docker stop mydocker    停止容器
[root@]# docker start 1a67f4c92b6e   启动容器
[root@]# docker run -d -p 80:80 --name webserver nginx  //docker run 只在第一次运行时使用，将镜像放到容器中，以后再次启动这个容器时，只需要使用命令docker start 即可。之后使用 docker start，会保留 docker run 的配置
[root@]# docker rm bfd094233f96   #删除一个容器
```

```
[root@]# docker inspect container/image  // 有关容器和镜像的底层信息
```

**为容器命名：**

```
docker rename  old_name new_name
```

```
# docker build [options] PATH | URL  // 使用Dockerfile来构建镜像
```

**进入一个容器**

```
docker exec -it CONTAINER_NAME /bin/bash
```

## 网络配置

可以使用 `–net=` 这个选项来执行 docker run 启动一个容器，这个选项有一下可选参数。

```
–net=bridge   //默认选项，用网桥的方式来连接docker容器。
–net=host   //docker跳过配置容器的独立网络栈，使用本地的网络。
–net=container:NAME_or_ID   //告诉docker让这个新建的容器使用已有容器的网络配置。
–net=none   //告诉docker为新建的容器建立一个网络栈，但不对这个网络栈进行任何配置，所以只能访问本地网络，没有外网。
```

手动给一个 container 配置网络桥接

```
[root@]# docker port webserver 443/tcp -> 0.0.0.0:32774 80/tcp -> 0.0.0.0:32775
```

使用 `-p`（小写的）则可以指定要映射的端口，并且，在一个指定端口上只可以绑定一个容器，-p 标记可以多次使用来绑定多个端口。支持的格式有 ip:hostPort:containerPort

```
# docker run -d -p 127.0.0.1:5000:5000 --name mydocker nginx
```

### host 模式

此模式使用主机的网络

```
# docker run -it --name feiyu-host --net=host busybox sh
```

### other container 模式

这种模式下与其他容器共享一个网络

```
# docker run -it --name feiyu-con --net=container:feiyu busybox sh
```

### none 模式

这种模式只能访问本地网络，没有外网。

## 数据卷的使用

数据卷是一个可供一个或多个容器使用的特殊目录，它绕过 UFS，可以提供很多有用的特性：

* 数据卷可以在容器之间共享和重用
* 对数据卷的修改会立马生效
* 对数据卷的更新，不会影响镜像
* 数据卷默认会一直存在，即使容器被删除

### 为容器挂载磁盘 

```
docker run -d -p 80:80 --name webserver -v /fromVolume:/toVolume nginx
```

数据卷是被设计用来持久化数据的，它的生命周期独立于容器，Docker不会在容器被删除后自动删除数据卷，并且也不存在垃圾回收这样的机制来处理没有任何容器引用的数据卷。如果需要在删除容器的同时移除数据卷。可以在删除容器的时候使用 `docker rm -v` 这个命令。

### 数据卷容器

如果你有一些持续更新的数据需要在容器之间共享，最好创建数据卷容器。

```
[root@]# docker run -d -v /data/ --name dbdata busybox //首先，创建一个名为 dbdata 的数据卷容器
[root@]# docker run -d --volumes-from dbdata --name db1 nginx //在其他容器中使用 –volumes-from 来挂载 dbdata 容器中的数据卷。
```

备份数据卷容器中的数据：

首先使用 `–volumes-from` 标记来创建一个加载 dbdata 容器卷的容器，并从主机挂载当前目录到容器的 `/backup` 目录。命令如下：

```
# docker run --volumes-from dbdata -v /data:/backup busybox tar cvf /backup/backup.tar.gz /data
```

恢复数据到数据卷容器：

如果要恢复数据到一个容器，首先创建一个带有空数据卷的容器 dbdata2。

```
# docker run -v /dbdata --name dbdata2 ubuntu /bin/bash
```

然后创建另一个容器，挂载 dbdata2 容器卷中的数据卷，并使用 untar 解压备份文件到挂载的容器卷中。

```
# docker run --volumes-from dbdata2 -v /data:/backup busybox tar xvf /backup/backup.tar.gz
```

为了查看/验证恢复的数据，可以再启动一个容器挂载同样的容器卷来查看

```
# docker run --volumes-from dbdata2 busybox /bin/ls /dbdata
```

### 删除数据

如果删除了挂载的容器（db1 和 db2），数据卷并不会被自动删除。如果要删除一个数据卷，必须在删除最后一个还挂载着它的容器时使用 `docker rm -v` 命令来指定同时删除关联的容器。

## Dockerfile

制作Docker image 有两种方式：一是使用 Docker container，直接构建容器，再导出成 image 使用；二是使用 Dockerfile，将所有动作写在文件中，再 build 成 image。

### Dockerfile 基本结构

一般的，Dockerfile 分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令。

```
# This my first nginx Dockerfile
# Version 1.0

# Base images 基础镜像
FROM centos

#MAINTAINER 维护者信息
MAINTAINER tianfeiyu 

#ENV 设置环境变量
ENV PATH /usr/local/nginx/sbin:$PATH

#ADD  文件放在当前目录下，拷过去会自动解压
ADD nginx-1.8.0.tar.gz /usr/local/  
ADD epel-release-latest-7.noarch.rpm /usr/local/  

#RUN 执行以下命令 
RUN rpm -ivh /usr/local/epel-release-latest-7.noarch.rpm
RUN yum install -y wget lftp gcc gcc-c++ make openssl-devel pcre-devel pcre && yum clean all
RUN useradd -s /sbin/nologin -M www

#WORKDIR 相当于cd
WORKDIR /usr/local/nginx-1.8.0 

RUN ./configure --prefix=/usr/local/nginx --user=www --group=www --with-http_ssl_module --with-pcre && make && make install

RUN echo "daemon off;" >> /etc/nginx.conf

#EXPOSE 映射端口
EXPOSE 80

#CMD 运行以下命令
CMD ["nginx"]
```

具体解释可以参考这篇文章 [Docker 使用指南 (五)—— Dockerfile 详解](https://cloud.tencent.com/developer/article/1004349)

### RUN vs CMD vs ENTRYPOINT

* RUN 执行命令并创建新的镜像层，RUN 经常用于安装软件包。
* CMD 设置容器启动后默认执行的命令及其参数，但 CMD 能够被 docker run 后面跟的命令行参数替换。
* ENTRYPOINT 配置容器启动时运行的命令。

RUN 在 Dockerfile 构建镜像的过程(Build)中运行，最终被commit的到镜像。

ENTRYPOINT 和 CMD 在容器运行(run、start)时运行。

CMD 后面可以跟参数，但是在 `docker run -it nginx /bin/bash` 中添加了参数 `/bin/bash`，则 CMD 中的内容就会被覆盖掉，不会执行。

而使用 ENTRYPOINT 的会后，如果 `docker run ` 后面添加参数，参数不会覆盖 ENTRYPOINT 命令，而是作为参数添加到 ENTRYPOINT 的执行命令中。

可以参考 [论docker中 CMD 与 ENTRYPOINT 的区别](http://cloud.51cto.com/art/201411/457338.htm)

### 创建镜像

使用以下命令来构建一个镜像：

```
# docker build -t feiyu/nginx:1.8 .
```

## 参考

* [Docker 极速下载 | DaoCloud](https://get.daocloud.io/)
* [Docker 使用指南 (一)—— 基本操作](https://cloud.tencent.com/developer/article/1004345)


