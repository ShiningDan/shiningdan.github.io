---
title: Redis入门
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

关于所有的参数说明，可以参考[Redis 配置
](http://www.runoob.com/redis/redis-conf.html)

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

string是redis最基本的类型，你可以理解成与Memcached一模一样的类型，一个key对应一个value。

string类型是二进制安全的。意思是redis的string可以包含任何数据。比如jpg图片或者序列化的对象 。

string类型是Redis最基本的数据类型，一个值最大能存储512MB。其他语言的数据类型都会被转换为 string 类型，并且在 Redis 中，键也是 string 类型的。

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
Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。

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

Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。

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

不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。

**zset的成员是唯一的,但分数(score)却可以重复**。

![](http://ojt6zsxg2.bkt.clouddn.com/54c2df197aa058fcfb9dc2bde5337151.png)

![](http://ojt6zsxg2.bkt.clouddn.com/d8b5a73e9d54075b7afa5142c6064356.png)


* `ZADD key score member`：添加元素到集合
* `ZRANGEBYSCORE runoob start stop`：显示分数 `start` 到 `stop` 的元素







