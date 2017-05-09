---
title: Elasticsearch 入门
date: 2017-05-09 16:22:24
categories: coding
tags:
  - Elasticsearch
---

本文介绍了一些 Elasticsearch 的入门知识，由于笔者的本意是使用 Elasticsearch 搭建一个博客站内搜索工具，所以很多没有使用到的内容就没有进行深入。如果有需要的读者，可以点击下面的《Elasticsearch: 权威指南》链接，或者购买该书籍进行深入的学习。


本文参考的资料有：

* [Elasticsearch: 权威指南](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/)
* [使用 Elasticsearch 实现博客站内搜索](https://imququ.com/post/elasticsearch.html)

<!--more-->

## 前言

Elasticsearch 是一个分布式、可扩展、实时的搜索与数据分析引擎。

Elasticsearch 不仅仅可以提供全文搜索，还可以提供结构化搜索、数据分析、复杂的语言处理、地理位置和对象间关联关系等。 我们还将探讨如何给数据建模来充分利用 Elasticsearch 的水平伸缩性，以及在生产环境中如何配置和监视你的集群。

## 基础入门

Elasticsearch 是一个开源的搜索引擎，建立在一个全文搜索引擎库 [Apache Lucene™](https://lucene.apache.org/core/) 基础之上。 Lucene 可能是目前存在的，不论开源还是私有的，拥有最先进，高性能和全功能搜索引擎功能的库。

但是 Lucene 仅仅只是一个库。为了利用它，你需要编写 java 程序，并在你的 java 程序里面直接集成 Lucene 包。 Elasticsearch 也是使用 Java 编写的，它的内部使用 Lucene 做索引与搜索，但是它的目标是使全文检索变得简单， 通过隐藏 Lucene 的复杂性，取而代之的提供一套简单一致的 RESTful API。

### 安装并运行 Elasticsearch

安装 Elasticsearch 之前，你需要先安装一个较新的版本的 Java

之后，你可以从 [elastic 的官网](https://www.elastic.co/downloads/elasticsearch) 获取最新版本的 Elasticsearch。在[ Docs Installation](https://www.elastic.co/guide/en/elasticsearch/reference/master/_installation.html)部分可以查看官方的安装介绍。

在安装并运行完 Elasticsearch 之后，测试 Elasticsearch 是否启动成功，可以打开另一个终端，执行以下操作：

```
curl 'http://localhost:9200/?pretty'
```

你应该得到和下面类似的响应(response)：

```
{
  "name" : "Tom Foster",
  "cluster_name" : "elasticsearch",
  "version" : {
    "number" : "2.1.0",
    "build_hash" : "72cd1f1a3eee09505e036106146dc1949dc5dc87",
    "build_timestamp" : "2015-11-18T22:40:03Z",
    "build_snapshot" : false,
    "lucene_version" : "5.3.1"
  },
  "tagline" : "You Know, for Search"
}
```

单个**节点**可以作为一个运行中的 Elasticsearch 的实例。 而一个**集群**是一组拥有相同 cluster.name 的节点， 他们能一起工作并共享数据，还提供容错与可伸缩性。

#### 安装 Sense

Sense 是一个 Kibana（一个开源的分析与可视化平台） 应用 它提供交互式的控制台，通过你的浏览器直接向 Elasticsearch 提交请求。

安装与运行 Sense：

在 Kibana 目录下运行下面的命令，下载并安装 Sense app：

```
./bin/kibana plugin --install elastic/sense
```

启动 Kibana.

```
./bin/kibana
```

### 和 Elasticsearch 交互

### Java API

#### 节点客户端（Node client）

节点客户端作为一个非数据节点加入到本地集群中。换句话说，它本身不保存任何数据，但是它知道数据在集群中的哪个节点中，并且可以把请求转发到正确的节点。

#### 传输客户端（Transport client）

轻量级的传输客户端可以可以将请求发送到远程集群。它本身不加入集群，但是它可以将请求转发到集群中的一个节点上。

### RESTful API with JSON over HTTP

所有其他语言可以使用 RESTful API 通过端口 9200 和 Elasticsearch 进行通信

一个 Elasticsearch 请求和任何 HTTP 请求一样由若干相同的部件组成：

```
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
```

被 `< >` 标记的部件：

```
VERB

适当的 HTTP 方法 或 谓词 : GET`、 `POST`、 `PUT`、 `HEAD 或者 `DELETE`。

PROTOCOL

http 或者 https`（如果你在 Elasticsearch 前面有一个 `https 代理）

HOST

Elasticsearch 集群中任意节点的主机名，或者用 localhost 代表本地机器上的节点。

PORT

运行 Elasticsearch HTTP 服务的端口号，默认是 9200 。

PATH

API 的终端路径（例如 _count 将返回集群中文档数量）。Path 可能包含多个组件，例如：_cluster/stats 和 _nodes/stats/jvm 。

QUERY_STRING

任意可选的查询字符串参数 (例如 ?pretty 将格式化地输出 JSON 返回值，使其更容易阅读)

BODY

一个 JSON 格式的请求体 (如果请求需要的话)
```

例如，计算集群中文档的数量，我们可以用这个:

```
curl -XGET 'http://localhost:9200/_count?pretty' -d '
{
    "query": {
        "match_all": {}
    }
}
```

Elasticsearch 返回一个 HTTP 状态码（例如：200 OK`）和（除`HEAD`请求）一个 JSON 格式的返回值。前面的 `curl 请求将返回一个像下面一样的 JSON 体：

```
{
    "count" : 0,
    "_shards" : {
        "total" : 5,
        "successful" : 5,
        "failed" : 0
    }
}
```

### 面向文档

























