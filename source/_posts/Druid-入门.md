---
title: Druid 入门
date: 2017-07-28 11:56:05
categories: coding
tags:
  - 数据库
  - Druid
---

本文记录的是我在学习实时 OLAP 数据分析存储系统 Druid 的笔记

<!--more-->

## Druid是否适合我？

许多组织部署Druid用于分析广告技术、 运维、 网络流量、 云安全、 网站流量、 金融， 以及传感器数据。 如果你存在以下需求，那么Druid会是不错的选择：

* 你正在构建一个需要快速聚合和探索分析的应用
* 你需要基于正在发生的数据进行分析（实时）
* 你拥有大量的数据（万亿级事件，PB级数据）
* 你需要一个无单点问题，始终可用的数据存储


## 什么是 OLAP

数据处理大致可以分成两大类：联机事务处理OLTP（on-line transaction processing）、联机分析处理OLAP（On-Line Analytical Processing）。OLTP是传统的关系型数据库的主要应用，主要是基本的、日常的事务处理，例如银行交易。OLAP是数据仓库系统的主要应用，支持复杂的分析操作，侧重决策支持，并且提供直观易懂的查询结果。

OLTP，也叫联机事务处理（Online Transaction Processing），表示事务性非常高的系统，一般都是高可用的在线系统，以小的事务以及小的查询为主，评估其系统的时候，一般看其每秒执行的Transaction以及Execute SQL的数量。在这样的系统中，单个数据库每秒处理的Transaction往往超过几百个，或者是几千个，Select 语句的执行量每秒几千甚至几万个。典型的OLTP系统有电子商务系统、银行、证券等，如美国eBay的业务数据库，就是很典型的OLTP数据库。

对于OLAP系统，在内存上可优化的余地很小，增加CPU 处理速度和磁盘I/O 速度是最直接的提高数据库性能的方法，当然这也意味着系统成本的增加。比如我们要对几亿条或者几十亿条数据进行聚合处理，这种海量的数据，全部放在内存中操作是很难的，同时也没有必要，因为这些数据快很少重用，缓存起来也没有实际意义，而且还会造成物理I/O相当大。 所以这种系统的瓶颈往往是磁盘I/O上面的。

[OLAP、OLTP的介绍和比较](http://blog.csdn.net/jiangshouzhuang/article/details/46884707)

## Druid (大数据实时统计分析数据存储)

Druid是一个为在大数据集之上做实时统计分析而设计的开源数据存储。

### Hadoop 的缺点

Hadoop擅长的是存储和获取大规模数据，但是它并不提供任何性能上的保证它能多快获取到数据。

此外，虽然Hadoop是一个高可用的系统，但是在高并发负载下性能会下降。

最后，Hadoop对于存储数据可以工作得很好，但是并没有对数据导入进行优化，使导入的数据立即可读。

### Druid 优点

[Druid (大数据实时统计分析数据存储)](https://github.com/QLeelulu/research/blob/master/druid/White_Paper/druid_cn.md)

Druid 的最初目的是设计来解决导入和分析大规模交易事件（日志数据）。这种时间序列形式的数据通常在OALP类工作流中比较常见，且数据的本质是非常重的**追加写**（实时写入）。

之所以需要Druid，是因为现实情况是现有的开源关系型数据库(RDBMS)和NoSQL key/value 数据库没办法为一些交互式应用提供低延迟的数据导入和查询。

从一个事件数据被创建，到这个事件数据可以被查询的时间，决定了利益相关方能够在他们的系统出现潜在灾难性情况时多快做出反应。流行的开源数据仓库系统，例如Hadoop，并不能达到我们所需要的秒级的数据导入和查询的要求。

### 架构

Druid集群的构成和数据流向

![](http://ojt6zsxg2.bkt.clouddn.com/a4721de03eeffd09e75180d93e414b77.png)

#### 实时节点

实时节点封装了导入和查询事件数据的功能，经由这些节点导入的事件数据可以立刻被查询。实时节点只关心一小段时间内的事件数据，并定期把这段时间内收集的这批不可变事件数据导入到Druid集群里面另外一个专门负责处理不可变的批量数据的节点中去。实时节点通过Zookeeper的协调和Druid集群的其他节点协调工作。实时节点通过Zookeeper来宣布他们的在线状态和他们提供的数据。

实时节点为所有传入的事件数据维持一个内存中的索引缓存。为了避免堆溢出问题，实时节点会定期地、或者在达到设定的最大行限制的时候，把内存中的索引持久化到磁盘去。这个持久化进程会把保存于内存缓存中的数据转换为基于列存储的格式

所有的实时节点都会周期性的启动后台的计划任务搜索本地的持久化索引，后台计划任务将这些持久化的索引合并到一起并生成一块不可变的数据

#### 可用性与可扩展性

实时节点是一个数据的消费者，需要有相应的生产商为其提供数据流。通常，为了数据耐久性的目的，会在生产商与实时节点间放置一个类似于Kafka这样的消息总线来进行连接

消息总线作为传入数据的缓冲区。类似于Kafka这样的消息总线会维持一个指示当前消费者（实时节点）从事件数据流中已经读取数据的位置偏移量，实时节点每次持久化内存中的缓存到磁盘的时候，都会更新这个偏移量。

多个实时节点可以从数据总线导入同一组数据，为数据创建一个副本。这样当一个节点完全挂掉并且磁盘上的数据也丢失了，副本可以确保不会丢失任何数据。

#### 历史节点

历史节点封装了加载和处理由实时节点创建的不可变数据块（segment）的功能。

类似于实时节点，历史节点在Zookeeper中通告它们的在线状态和为哪些数据提供服务。

#### Broker节点

Broker节点扮演着历史节点和实时节点的查询路由的角色。Broker节点知道发布于Zookeeper中的关于哪些segment是可查询的和这些segment是保存在哪里的，Broker节点就可以将到来的查询请求路由到正确的历史节点或者是实时节点，Broker节点也会将历史节点和实时节点的局部结果进行合并，然后返回最终的合并后的结果给调用者。

#### 协调节点

Druid的协调节点主要负责数据的管理和在历史节点上的分布。协调节点告诉历史节点加载新数据、卸载过期数据、复制数据、和为了负载均衡移动数据。

#### 存储格式

Druid中的数据表（称为数据源）是一个时间序列事件数据的集合，并分割到一组segment中，而每一个segment通常是0.5-1千万行。在形式上，我们定义一个segment为跨越一段时间的数据行的集合。Segment是Druid里面的基本存储单元，复制和分布都是在segment基础之上进行的。

### 其他依赖

Druid 包含3个外部依赖 ：Mysql、Deep storage、Zookeeper

#### Mysql

Mysql：存储关于Druid中的metadata而不是存储实际数据，包含3张表：”druid_config”（通常是空的）, “druid_rules”（协作节点使用的一些规则信息，比如哪个segment从哪个node去load）和“druid_segments”（存储 每个segment的metadata信息）；

#### Deep storage

Deep storage: 存储segments，Druid目前已经支持本地磁盘，NFS挂载磁盘，HDFS，S3等。Deep Storage的数据有2个来源，一个是批数据摄入, 另一个来自实时节点；

#### ZooKeeper

ZooKeeper: 被Druid用于管理当前cluster的状态，比如记录哪些segments从实时节点移到了历史节点；

## Druid vs 其他系统

### Druid vs Impala/Shark

Druid和Impala、Shark 的比较基本上可以归结为需要设计什么样的系统

Druid被设计用于：

1. 一直在线的服务
2. 获取实时数据
3. 处理slice-n-dice式的即时查询

#### 查询速度不同：

Druid是列存储方式，数据经过压缩加入到索引结构中，压缩增加了RAM中的数据存储能力，能够使RAM适应更多的数据快速存取。索引结构意味着，当添加过滤器来查询，Druid少做一些处理，将会查询的更快。
Impala/Shark可以认为是HDFS之上的后台程序缓存层。 但是他们没有超越缓存功能，真正的提高查询速度。

#### 数据的获取不同：

Druid可以获取实时数据。
Impala/Shark是基于HDFS或者其他后备存储，限制了数据获取的速度。

##### 查询的形式不同：

Druid支持时间序列和groupby样式的查询，但不支持join。
Impala/Shark支持SQL样式的查询。

### Druid vs Elasticsearch

Elasticsearch(ES) 是基于Apache Lucene的搜索服务器。它提供了全文搜索的模式，并提供了访问原始事件级数据。 Elasticsearch还提供了分析和汇总支持。根据研究，ES在数据获取和聚集用的资源比在Druid高。

Druid侧重于OLAP工作流程。Druid是高性能（快速聚集和获取）以较低的成本进行了优化，并支持广泛的分析操作。Druid提供了结构化的事件数据的一些基本的搜索支持。

### Druid vs Spark

Spark 是围绕弹性分布式数据集（ RDD ）的概念，建立了一个集群计算框架，可以被看作是一个后台分析平台。 RDD启用数据复用保持中间结果存在内存中，给Spark提供快速计算的迭代算法。这对于某些工作流程，如机器学习，相同的操作可应用一遍又一遍，直到有 结果后收敛尤其有益。Spark提供分析师与不同算法各种各样运行查询和分析大量数据的能力。

Druid重点是数据获取和提供查询数据的服务，如果建立一个web界面，用户可以随意查看数据。

## 查询 API

[Druid | 大规模交互式查询](http://druidio.cn/)

[说明文档](http://druidio.cn/docs/0.9.0/design/index.html)

我们看看一个数据集的例子 (来源于线上广告):

```
timestamp             publisher          advertiser  gender  country  click  price
2011-01-01T01:01:35Z  bieberfever.com    google.com  Male    USA      0      0.65
2011-01-01T01:03:63Z  bieberfever.com    google.com  Male    USA      0      0.62
2011-01-01T01:04:51Z  bieberfever.com    google.com  Male    USA      1      0.45
2011-01-01T01:00:00Z  ultratrimfast.com  google.com  Female  UK       0      0.87
2011-01-01T02:00:00Z  ultratrimfast.com  google.com  Female  UK       0      0.99
2011-01-01T02:00:00Z  ultratrimfast.com  google.com  Female  UK       1      1.53
```

这个数据集有三部分组成。如果你比较熟悉OLAP的术语的话,应该会很熟悉下面的概念。

* **Timestamp** 列: 我们将timestamp区别开是因为我们所有的查询都以时间为中心。
* **Dimension** 列: Dimensions对应事件的维度,通常用于筛选过滤数据。 在我们例子中的数据有四个dimensions: publisher, advertiser, gender, and country。 它们每一个都可以看作是我们已选都数据的主体。
* **Metric** 列: Metrics是用于聚合和计算的列。在我们的例子中,click和price就是metrics。 Metrics通常是数字,并且包含支持count、sum、mean等计算操作。 在OLAP的术语中也被叫做measures。




## 参考

* [OLAP、OLTP的介绍和比较](http://blog.csdn.net/jiangshouzhuang/article/details/46884707)
* [Druid (大数据实时统计分析数据存储)](https://github.com/QLeelulu/research/blob/master/druid/White_Paper/druid_cn.md)
* [Druid 实时数据分析存储系统](https://my.oschina.net/betaoo/blog/530088)
* [Druid | 大规模交互式查询](http://druidio.cn/)
* [说明文档](http://druidio.cn/docs/0.9.0/design/index.html)







