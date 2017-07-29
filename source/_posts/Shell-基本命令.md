---
title: Shell 基本命令
date: 2017-07-24 17:47:26
categories: coding
tags:
  - Shell
---

本文记录的是一些基本的 Shell 命令，仅供自己参考

<!--more-->

### find

find是一个基于查找的功能非常强大的命令，相对而言，它的使用也相对较为复杂，参数也比较多，所以在这里将给把它们分类列出，它的基本语法如下：

```
find [PATH] [option] [action]  
  
# 与时间有关的参数：  
-mtime n : n为数字，意思为在n天之前的“一天内”被更改过的文件；  
-mtime +n : 列出在n天之前（不含n天本身）被更改过的文件名；  
-mtime -n : 列出在n天之内（含n天本身）被更改过的文件名；  
-newer file : 列出比file还要新的文件名  
# 例如：  
find /root -mtime 0 # 在当前目录下查找今天之内有改动的文件  
  
# 与用户或用户组名有关的参数：  
-user name : 列出文件所有者为name的文件  
-group name : 列出文件所属用户组为name的文件  
-uid n : 列出文件所有者为用户ID为n的文件  
-gid n : 列出文件所属用户组为用户组ID为n的文件  
# 例如：  
find /home/ljianhui -user ljianhui # 在目录/home/ljianhui中找出所有者为ljianhui的文件  
  
# 与文件权限及名称有关的参数：  
-name filename ：找出文件名为filename的文件  
-size [+-]SIZE ：找出比SIZE还要大（+）或小（-）的文件  
-tpye TYPE ：查找文件的类型为TYPE的文件，TYPE的值主要有：一般文件（f)、设备文件（b、c）、  
             目录（d）、连接文件（l）、socket（s）、FIFO管道文件（p）；  
-perm mode ：查找文件权限刚好等于mode的文件，mode用数字表示，如0755；  
-perm -mode ：查找文件权限必须要全部包括mode权限的文件，mode用数字表示  
-perm +mode ：查找文件权限包含任一mode的权限的文件，mode用数字表示  
# 例如：  
find / -name passwd # 查找文件名为passwd的文件  
find . -perm 0755 # 查找当前目录中文件权限的0755的文件  
```

### ps命令

该命令用于将某个时间点的进程运行情况选取下来并输出，process之意，它的常用参数如下：


```
-A ：所有的进程均显示出来  
-a ：不与terminal有关的所有进程  
-u ：有效用户的相关进程  
-x ：一般与a参数一起使用，可列出较完整的信息  
-l ：较长，较详细地将PID的信息列出  
```

其实我们只要记住ps一般使用的命令参数搭配即可，它们并不多，如下：


```
ps aux # 查看系统所有的进程数据  
ps ax # 查看不与terminal有关的所有进程  
ps -lA # 查看系统所有的进程数据  
ps axjf # 查看连同一部分进程树状态  
```

![](http://ojt6zsxg2.bkt.clouddn.com/c505746b69951a6a7f7f55c1d223161d.png)

其中显示了：
 
USER 哪个用户启动了这个命令
PID 进程ID
CPU CPU占用率
MEM 内存使用量
VSZ 如果一个程序完全驻留在内存的话需要占用多少内存空间
RSS 当前实际占用了多少内存
TTY: 终端的次要装置号码 (minor device number of tty)
STAT 进程当前的状态("S":中断 sleeping,进程处在睡眠状态,表明这些进程在等待某些事件发生--可能是用户输入或者系统资源的可用性;"D":不可中断 uninterruptible sleep;"R":运行 runnable;"T":停止 traced or stopped;"Z":僵死 a defunct zombie process)
START 启动命令的时间点
TIME 进程执行起到现在总的CPU暂用时间
COMMAND 启动这个进程的命令

![](http://ojt6zsxg2.bkt.clouddn.com/35730fac98a30beafbb705618109bde2.png)

其中显示了：

UID 用户号
PID 进程ID
PPID 父进程号
C CPU占用率
TTY 终端的次要装置号码 (minor device number of tty)
TIME 进程执行起到现在总的CPU暂用时间
COMMAND 启动这个进程的命令

### jps 命令

jps位于jdk的bin目录下，其作用是显示当前系统的java进程情况，及其id号。 jps相当于Solaris进程工具ps。

### kill命令

该命令用于向某个工作（%jobnumber）或者是某个PID（数字）传送一个信号

```
kill -signal PID  

1：SIGHUP，启动被终止的进程  
2：SIGINT，相当于输入ctrl+c，中断一个程序的进行  
9：SIGKILL，强制中断一个进程的进行  
15：SIGTERM，以正常的结束进程方式来终止进程  
17：SIGSTOP，相当于输入ctrl+z，暂停一个进程的进行  
```

### tar命令

该命令用于对文件进行打包，默认情况并不会压缩，如果指定了相应的参数，它还会调用相应的压缩程序（如gzip和bzip等）进行压缩和解压。它的常用参数如下：

```
-c ：新建打包文件  
-t ：查看打包文件的内容含有哪些文件名  
-x ：解打包或解压缩的功能，可以搭配-C（大写）指定解压的目录，注意-c,-t,-x不能同时出现在同一条命令中  
-j ：通过bzip2的支持进行压缩/解压缩  
-z ：通过gzip的支持进行压缩/解压缩  
-v ：在压缩/解压缩过程中，将正在处理的文件名显示出来  
-f filename ：filename为要处理的文件  
-C dir ：指定压缩/解压缩的目录dir  
```

### cat、more、less 命令

cat、more、less均可用来查看文件内容，主要区别有：

cat是一次性显示整个文件的内容，还可以将多个文件连接起来显示，它常与重定向符号配合使用，适用于文件内容少的情况；

more和less一般用于显示文件内容超过一屏的内容，并且提供翻页的功能。more比cat强大，提供分页显示的功能

less比more更强大，提供翻页，跳转，查找等命令。而且more和less都支持：用空格显示下一页，按键b显示上一页。

### df

df -k 查看各文件系统剩余的空间，-k说明单位是千字节(kb)

### du

查看目录所占磁盘容量

du [-s] 目录
du dir1 显示目录 dir1 的总容量及其子目录的容量(以KB 为单位)。
du -s dir1 显示目录 dir1 的总容量。

### passwd

passwd
Old password: <输入旧密码>
New password: <输入新密码〉
Retype new password: <再输入一次密码>

### alias

查看所定义的命令的别名

 alias 查看自己目前定义的所有命令，及所对应的别名。
 
 alias name 查看指定的name 命令的别名。
 
 定义命令的别名： alias name‘command line’
 
 删除所定义的别名：unalias name
 
 ### history
 
 查看命令记录表的内容
 
 重复执行前第 n 个命令： !n，n：命令记录表的命令编号。
 
 重复执行前一个命令：!!
 
 ### `|`
 
 命令1 | 命令2 将命令1的执行结果送到命令2，做为命令2的输入。
 
 ls -Rl | more 以分页方式列出当前目录及其子目录下所有文件的名称。
cat file1 | more 以分页方式列出文件 file1 的内容。
 
### 输入/输出控制

#### 将文件做为命令的输入：命令 < 文件 

mail -s “mail test” 电子邮件地址 < file1     将文件file 当做信件的内容

#### 将命令的执行结果送至指定的文件中：命令 > 文件

ls -l > list 将执行 “ls -l” 命令的结果写入文件list 中。

#### 命令>! 文件 将命令的执行结果送至指定的文件中，若文件已经存在，则覆盖。

ls -lg >! list 将执行 “ls - lg” 命令的结果覆盖写入文件 list 中

#### 命令 >& 文件 将命令执行时屏幕上所产生的任何信息写入指定的文件中。

cc file1.c >& error 将编译 file1.c 文件时所产生的任何信息写入文件 error 中

#### 命令>> 文件 将命令执行的结果附加到指定的文件中。

ls - lag >> list 将执行 “ls - lag” 命令的结果附加到文件 list 中。

#### 命令 >>& 文件 将命令执行时屏幕上所产生的任何信息附加到指定的文件中。

cc file2.c >>& error 将编译 file2.c 文件时屏幕所产生的任何信息附加到文件error 中。

### netstat


常见参数
-a (all)显示所有选项，默认不显示LISTEN相关
-t (tcp)仅显示tcp相关选项
-u (udp)仅显示udp相关选项
-n 拒绝显示别名，能显示数字的全部转化成数字。
-l 仅列出有在 Listen (监听) 的服務状态


