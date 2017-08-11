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

### Redis的Java客户端Jedis的八种调用方式(事务、管道、分布式…)介绍

* [Redis的Java客户端Jedis的八种调用方式(事务、管道、分布式…)介绍](http://www.blogways.net/blog/2013/06/02/jedis-demo.html)

## 几种不同的数据库之间的比较

### 数据库的方向 - 行vs列

在行式数据库中，每一行中的每一块数据都是紧挨着另一块数据存放在硬盘中。一般情况下，你可以认为每一行存贮的内容就是硬盘中的一组连续的字节。

为了方便我们的讨论，我们假设每一行都包含一个用户的信息，每个用户的所有属性都整块儿存储在硬盘上。

![](http://ojt6zsxg2.bkt.clouddn.com/9905096c020f11360a788a0f983456e6.png)

在上边的例子中，Alice的所有信息都存储在一个页面中。如果需要获取或更新Alice的信息，那么某一时刻在内存中仅需存储关于Alice的单一页面。

如果是基于列的数据库，所有的数据都是以列的形式存储的。回到之前的例子，假设每一列的存储对应一个页面。如下图所示，所有的ZIP code将会存储到一个页面中，而所有的“2013 Total Order”则会存储在另一个页面中。

所以，如果你使用的是行式数据库，那么你对一行数据进行操作时，数据库的性能会是最好的。在上面的例子中，仅一个页面被放到了内存中。（这只是一个示例，事实上，操作系统会带来不止一页的数据，稍后详细说明）

另一方面，如果你的数据库是基于行的，但是你要想得到所有数据中，某一列上的数据来做一些操作，这就意味着你将花费时间去访问每一行，可你用到的数据仅是一行中的小部分数据。

可关键在于你使用列式数据库时，当你想要得到Alice的所有信息时，你又必须要读取大量的列（页面）来获取所有的数据。

OLTP工作负载是数据库现有业务的关键业务。一般而言，这些应用程序在使用行数据库时会有更好的表现，因为其工作负载趋向于单一实体的多个属性（存储在很多的列中）。由于这些应用程序都是基于行工作的，所以在使用时，从硬盘中获取的页面数量是最小的。

在线分析处理（OLAP）工作负载常常需要收集列中的数据。当你使用基于列的数据库时，你可以将这一列放到内存中并统计所有值。但当使用的是基于行的数据库时，就必须去访问每一行而获取对应的数据。

#### 对比

数据修改：行存储是在指定位置写入一次，列存储是将磁盘定位到多个列上分别写入

数据读取：行存储通常将一行数据完全读出，如果只需要其中几列数据的情况，就会存在冗余列；列存储每次读取的数据是集合的一段或者全部，如果读取多列时，就需要移动磁头，再次定位到下一列的位置继续读取。 

行存储的写入是一次性完成，消耗的时间比列存储少，并且能够保证数据的完整性，缺点是数据读取过程中会产生冗余数据，如果只有少量数据，此影响可以忽略；数量大可能会影响到数据的处理效率。列存储在写入效率、保证数据完整性上都不如行存储，它的优势是在读取过程，不会产生冗余数据，这对数据完整性要求不高的大数据处理领域，比如互联网，犹为重要。

如果以保存数据为主，行存储的写入性能比列存储高很多。在需要频繁读取单列集合数据的应用中，列存储是最合适的。

若采用列存储方案，为保证读写入效率，每列数据尽可能分别保存到不同的磁盘上，多个线程并行读写各自的数据，这样避免了磁盘竞用的同时也提高了处理效率。

#### 改进

行存储的改进：减少冗余数据首先是用户在定义数据时避免冗余列的产生；其次是优化数据存储记录结构，保证从磁盘读出的数据进入内存后，能够被快速分解，消除冗余列。

列存储的两点改进：1.在计算机上安装多块硬盘，以多线程并行的方式读写它们。多块硬盘并行工作可以减少磁盘读写竞用，这种方式对提高处理效率优势十分明显。缺点是需要更多的硬盘，这会增加投入成本，在大规模数据处理应用中是不小的数目，运营商需要认真考虑这个问题。2.对写过程中的数据完整性问题，可考虑在写入过程中加入类似关系数据库的“回滚”机制，当某一列发生写入失败时，此前写入的数据全部失效，同时加入散列码校验，进一步保证数据完整性。

频繁的小量的数据写入对磁盘影响很大，更好的解决办法是将数据在内存中暂时保存并整理，达到一定数量后，一次性写入磁盘，这样消耗时间更少一些。

* [数据库的方向 - 行vs列](https://www.ibm.com/developerworks/community/blogs/IBMi/entry/database?lang=en)
* [为什么列存储数据库读取速度会比传统的行数据库快？](https://www.zhihu.com/question/29380943)

### hbase

hbase 是 noSql数据库的一种，最常见的应用场景就是采集的网页数据的存储，由于是key-value型数据库，可以再扩展到各种key-value应用场景，如日志信息的存储，对于内容信息不需要完全结构化出来的类CMS应用等。注意hbase针对的仍然是OLTP应用为主。

而HBase表是物理表，适合存放非结构化的数据。

### hive

对于hbase当前noSql数据库的一种，最常见的应用场景就是采集的网页数据的存储，由于是key-value型数据库，可以再扩展到各种key-value应用场景，如日志信息的存储，对于内容信息不需要完全结构化出来的类CMS应用等。注意hbase针对的仍然是OLTP应用为主。

 Hive中的表是纯逻辑表，就只是表的定义等，即表的元数据。Hive本身不存储数据，它完全依赖HDFS和MapReduce。这样就可以将结构化的数据文件映射为为一张数据库表，并提供完整的SQL查询功能，并将SQL语句最终转换为MapReduce任务进行运行。

**Hive是基于MapReduce来处理数据,而MapReduce处理数据是基于行的模式；HBase处理数据是基于列的而不是基于行的模式，适合海量数据的随机访问。**

