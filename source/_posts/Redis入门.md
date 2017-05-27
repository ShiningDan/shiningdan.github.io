---
title: redis入门
date: 2017-05-26 19:27:25
categories: coding
tags:
  - Redis
---

Redis 是一个key-value存储系统，值（value）可以是 **字符串(String), 表(Hash), 列表(List), 集合(Sets) 和 有序集合(Sorted Sets)等**类型。

<!--more-->

## Redis 简介

Redis支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用。

Redis的所有操作都是原子性的，同时Redis还支持对几个操作全并后的原子性执行。
 
## Redis 应用场景

因为 Redis 是基于内存来存储数据的，所以 Redis 对数据读取的速度远远大于使用硬盘来读取数据的速度，因此经常被用作缓存。

因为 Redis 的列表提供 `POP` 和 `PUSH` 操作，所以可以使用 Redis 作为队列。

虽然 Redis 是基于内存的，但是 Redis 提供了持久化机制，定期存储数据到硬盘中。

除此之外，Redis 还经常使用在应用排行榜、网站访问统计、数据过期处理、商品秒杀（队列机制）等地方。

## Redis 配置

Redis 的配置文件位于 Redis 安装目录下，文件名为 redis.conf。
你可以通过 `CONFIG` 命令查看或设置配置项。

```
redis 127.0.0.1:6379> CONFIG GET CONFIG_SETTING_NAME
```

比如：

```
redis 127.0.0.1:6379> CONFIG GET loglevel
1) "loglevel"
2) "notice"
```

使用 `*` 可以获得所有的配置

```
redis 127.0.0.1:6379> CONFIG GET *
```

关于所有的参数说明，可以参考 f[Redis 配置
](http://www.runoob.com/redis/redis-conf.html)

常用要修改的配置有：

```
daemonize yes      // 设置 Redis 后台启动
port 7200          // 设置 Redis 的启动端口
```

在 Mac 下面，可以使用 

```
brew install redis 
```

来安装 Redis，在 Mac 下面，安装完 Redis 之后，直接输入 `redis-server` 就可以使用 Redis 了，如果需要配置文件启动参数，可以使用和修改默认的参数：

```
redis-server /usr/local/etc/redis.conf
```

首先运行 `redis-server` 来启动一个 Redis，然后使用命令 `redis-cli` 来通过客户端连接 Redis。

```
redis-cli
127.0.0.1:6379> 
/* 由于是本机连接，所以默认ip为127.0.0.1，默认端口为6379*/
```

如果设置了密码，或者配置了新的端口，则需要在使用 `redis-cli` 的时候声明端口和密码：

```
redis-cli -h host -p port -a password
// 举例
redis-cli -h 127.0.0.1 -p 6379 -a "mypass"
```
### 连接命令

![](http://ojt6zsxg2.bkt.clouddn.com/ac49e0d920ceb9ba1cc087efc8951d27.png)

### 查看 Redis 信息

获得 Redis 服务器的统计信息，可以输入：

```
127.0.0.1:6379> info
```

### 退出 Redis

在 `redis-cli` 里面输入 `shutdown`，就可以关闭 Redis 服务器，然后输入 `exit` 就可以退出 Redis 客户端。

## Redis 键(key)

查看所有的 key

```
redis 127.0.0.1:6379> keys *
```

![](http://ojt6zsxg2.bkt.clouddn.com/f90e341f0fb9cd6270367b41c7a2a36d.png)

![](http://ojt6zsxg2.bkt.clouddn.com/37e366197085ec1aa0fd2aeec10e51db.png)

具体的解释可以查看[Redis 键(key)](http://www.runoob.com/redis/redis-keys.html)


## Redis 数据类型

Redis支持五种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)。

[查询所有 Redis 命令](https://redis.io/commands)

### String（字符串）

string是Redis最基本的类型，你可以理解成与Memcached一模一样的类型，一个key对应一个value。

string类型是二进制安全的。意思是Redis的string可以包含任何数据。比如jpg图片或者序列化的对象 。

string类型是Redis最基本的数据类型，一个值最大能存储512MB。其他语言的数据类型都会被转换为 string 类型，并且在 Redis 中，键也是 string 类型的。String 类型的值**可以包裹的是字符串，整数和浮点数，并且可以对整数和浮点数进行增减，对字符串进行操作**

```
redis 127.0.0.1:6379> SET name 1
OK
redis 127.0.0.1:6379> GET name
"1"
```

![](http://ojt6zsxg2.bkt.clouddn.com/6c5c0bf36a37ee7f8afb1315e3baf136.png)

![](http://ojt6zsxg2.bkt.clouddn.com/9af6118777adbf5980b235db4788e83e.png)


### Hash（哈希）

Redis hash 是一个键名对集合。
Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。**值可以是 String 类型，也可以是 整数，或者 浮点数 类型**

![](http://ojt6zsxg2.bkt.clouddn.com/ddbb807f152bd4bcce56e24fd92ce845.png)

![](http://ojt6zsxg2.bkt.clouddn.com/fb8e9b067b7b1a8817f96d9920aa6b0b.png)

使用 `HMSET` 和 `HGETALL` 来设置和获取 Hash 值。

```
127.0.0.1:6379> HMSET user:1 username runoob password runoob points 200
OK
127.0.0.1:6379> HGETALL user:1
1) "username"
2) "runoob"
3) "password"
4) "runoob"
5) "points"
6) "200"
```

每个 hash 可以存储 2^32 -1 键值对（40多亿）。

### List（列表）

Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。**值可以是 String 类型，也可以是 整数，或者 浮点数 类型**

其中 `LINDEX` 表示了在 Redis 中数据的下表是如何计算的，具体可以参考：[LINDEX key index](https://redis.io/commands/lindex)

![](http://ojt6zsxg2.bkt.clouddn.com/00f9abc43b090d868426e089c21b271f.png)

![](http://ojt6zsxg2.bkt.clouddn.com/2f4dcf56f421e2e0e025614fffa09b63.png)

```
redis 127.0.0.1:6379> lpush runoob redis
(integer) 1
redis 127.0.0.1:6379> lpush runoob mongodb
(integer) 2
redis 127.0.0.1:6379> rpush runoob rabitmq
(integer) 3
redis 127.0.0.1:6379> lrange runoob 0 10
1) "rabitmq"
2) "mongodb"
3) "redis"
redis 127.0.0.1:6379>
```

列表最多可存储 232 - 1 元素 (4294967295, 每个列表可存储40多亿)。

### Set（集合）

Redis的Set是string类型的无序不重复集合。

![](http://ojt6zsxg2.bkt.clouddn.com/610cbc7988fc3796d8ef4d23142021e7.png)

![](http://ojt6zsxg2.bkt.clouddn.com/4b277047e08d554cdd7ed654db3067ac.png)

* `SADD` ：添加数据
* `SMEMBERS`：展示所有的数据

```
redis 127.0.0.1:6379> sadd runoob redis
(integer) 1
redis 127.0.0.1:6379> sadd runoob mongodb
(integer) 1
redis 127.0.0.1:6379> sadd runoob rabitmq
(integer) 1
redis 127.0.0.1:6379> sadd runoob rabitmq
(integer) 0
redis 127.0.0.1:6379> smembers runoob
1) "rabitmq"
2) "mongodb"
3) "redis"
```


### zset(sorted set：有序集合)

Redis zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。

**值可以是 String 类型，也可以是 整数，或者 浮点数 类型**

不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。

**zset的 value 是唯一的,但分数(score)却可以重复**。 如果两个元素的 score 是一样的，则排名顺序按照这两个元素的 value 的字典排列顺序。

![](http://ojt6zsxg2.bkt.clouddn.com/54c2df197aa058fcfb9dc2bde5337151.png)

![](http://ojt6zsxg2.bkt.clouddn.com/d8b5a73e9d54075b7afa5142c6064356.png)


* `ZADD key score member`：添加元素到集合
* `ZRANGEBYSCORE runoob start stop`：显示分数 `start` 到 `stop` 的元素

## HyperLogLog

Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

### 什么是基数?

比如数据集 `{1, 3, 5, 7, 5, 7, 8}`， 那么这个数据集的基数集为 `{1, 3, 5 ,7, 8}`, 基数(不重复元素)为 `5`。 基数估计就是在误差可接受的范围内，快速计算基数。

![](http://ojt6zsxg2.bkt.clouddn.com/29837f8a1504ecdddee4a7b0db864312.png)

```
redis 127.0.0.1:6379> PFADD runoobkey "redis"
1) (integer) 1
redis 127.0.0.1:6379> PFADD runoobkey "mongodb"
1) (integer) 1
redis 127.0.0.1:6379> PFADD runoobkey "mysql"
1) (integer) 1
redis 127.0.0.1:6379> PFCOUNT runoobkey
(integer) 3
```

## 发布订阅

Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。

Redis 客户端可以订阅任意数量的频道。

![](http://ojt6zsxg2.bkt.clouddn.com/caa84e357a42a748a8aa139fa9179b17.png)

![](http://ojt6zsxg2.bkt.clouddn.com/bdbdd164ff4c8d93bf58e3ba74cd5a5c.png)

## 事务

Redis 事务可以一次执行多个命令， 并且带有以下两个重要的保证：

* 事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。
* 事务是一个原子操作：事务中的命令要么全部被执行，要么全部都不执行。

以 `MULTI` 开始一个事务， 然后将多个命令入队到事务中， 最后由 `EXEC` 命令触发事务， 一并执行事务中的所有命令：

```
redis 127.0.0.1:6379> MULTI
OK
redis 127.0.0.1:6379> SET book-name "Mastering C++ in 21 days"
QUEUED
redis 127.0.0.1:6379> GET book-name
QUEUED
redis 127.0.0.1:6379> SADD tag "C++" "Programming" "Mastering Series"
QUEUED
redis 127.0.0.1:6379> SMEMBERS tag
QUEUED
redis 127.0.0.1:6379> EXEC
1) OK
2) "Mastering C++ in 21 days"
3) (integer) 3
4) 1) "Mastering Series"
   2) "C++"
   3) "Programming"
```

![](http://ojt6zsxg2.bkt.clouddn.com/371ca1a0523726d41e96976c48c2590a.png)

## 脚本

Redis 脚本使用 Lua 解释器来执行脚本。 Reids 2.6 版本通过内嵌支持 Lua 环境。执行脚本的常用命令为 `EVAL`。

## 数据备份与恢复

Redis `SAVE` 命令用于创建当前数据库的备份。

```
redis 127.0.0.1:6379> SAVE 
OK
```

该命令将在 redis 安装目录中创建 `dump.rdb` 文件。

如果需要恢复数据，只需将备份文件 (`dump.rdb`) 移动到 Redis 安装目录并启动服务即可。获取 Redis 目录可以使用 `CONFIG` 命令，如下所示：
 
```
redis 127.0.0.1:6379> CONFIG GET dir
1) "dir"
2) "/usr/local/redis/bin"
```

**默认路径就是在哪个文件夹下运行的 `redis-server`**

## 安全

我们可以通过以下命令查看是否设置了密码验证：

```
127.0.0.1:6379> CONFIG get requirepass
1) "requirepass"
2) ""
```

默认情况下 `requirepass` 参数是空的，这就意味着你无需通过密码验证就可以连接到 Redis 服务。
你可以通过以下命令来修改该参数：

```
127.0.0.1:6379> CONFIG set requirepass "runoob"
OK
127.0.0.1:6379> CONFIG get requirepass
1) "requirepass"
2) "runoob"
```

设置密码后，客户端连接 Redis 服务就需要密码验证，否则无法执行命令。

AUTH 命令基本语法格式如下：

```
127.0.0.1:6379> AUTH runoob
```

## 性能测试

redis 性能测试的基本命令如下：

```
redis-benchmark [option] [option value]
```

## 管道技术

Redis是一种基于客户端-服务端模型以及请求/响应协议的TCP服务。这意味着通常情况下一个请求会遵循以下步骤：

* 客户端向服务端发送一个查询请求，并监听Socket返回，通常是以阻塞模式，等待服务端响应。
* 服务端处理命令，并将结果返回给客户端。

Redis 管道技术可以在服务端未响应时，客户端可以继续向服务端发送请求，并最终一次性读取所有服务端的响应。

```
$(echo -en "PING\r\n SET runoobkey redis\r\nGET runoobkey\r\nINCR visitor\r\nINCR visitor\r\nINCR visitor\r\n"; sleep 10) | nc localhost 6379

+PONG
+OK
redis
:1
:2
:3
```

以上实例中我们通过使用 PING 命令查看redis服务是否可用， 之后我们们设置了 runoobkey 的值为 redis，然后我们获取 runoobkey 的值并使得 visitor 自增 3 次。
在返回的结果中我们可以看到这些命令一次性向 redis 服务提交，并最终一次性读取所有服务端的响应

