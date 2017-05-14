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
cd elasticsearch-<version>
./bin/elasticsearch

// 等待运行成功以后
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

Elasticsearch 是**面向文档**的，意味着它**存储整个对象或 文档**。Elasticsearch 不仅存储文档，而且每个文档的内容使之可以被检索。在 Elasticsearch 中，你 **对文档进行索引、检索、排序和过滤**--而不是对行列数据。这是一种完全不同的思考数据的方式，也是 Elasticsearch 能支持复杂全文检索的原因。

**Elasticsearch 使用 JavaScript Object Notation 或者 JSON 作为文档的序列化格式** 

### 索引

**索引**这个词在 Elasticsearch 语境中包含多重意思

#### 索引（名词）：

如前所述，一个**索引**类似于传统关系数据库中的一个**数据库** ，是一个存储关系型文档的地方。 索引 (index) 的复数词为 indices 或 indexes 。

#### 索引（动词）：

索引一个文档 就是存储一个文档到一个 索引 （名词）中以便它可以被检索和查询到。这非常类似于 SQL 语句中的 INSERT 关键词，除了文档已存在时新文档会替换就文档情况之外。

#### 倒排索引：

关系型数据库通过增加一个 索引 比如一个 B树（B-tree）索引 到指定的列上，以便提升数据检索速度。Elasticsearch 和 Lucene 使用了一个叫做 倒排索引 的结构来达到相同的目的。

#### 索引雇员文档

对于雇员目录，我们将做如下操作：

* 每个雇员索引一个文档，包含该雇员的所有信息。
* 每个文档都将是 employee 类型 。
* 该类型位于 索引 megacorp 内。
* 该索引保存在我们的 Elasticsearch 集群中。

```
curl -XPUT 'http://localhost:9200/megacorp/employee/1' -d '
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}'
curl -XPUT 'http://localhost:9200/megacorp/employee/2' -d '
{
    "first_name" :  "Jane",
    "last_name" :   "Smith",
    "age" :         32,
    "about" :       "I like to collect rock albums",
    "interests":  [ "music" ]
}'
curl -XPUT 'http://localhost:9200/megacorp/employee/3' -d '
{
    "first_name" :  "Douglas",
    "last_name" :   "Fir",
    "age" :         35,
    "about":        "I like to build cabinets",
    "interests":  [ "forestry" ]
}'
```

注意，路径 `/megacorp/employee/1` 包含了三部分的信息：

`megacorp`：索引名称
`employee`：类型名称
`1`：特定雇员的ID

#### 检索文档

```
curl -XGET 'http://localhost:9200/megacorp/employee/1'
```

返回结果包含了文档的一些元数据，以及 `_source` 属性，`_source` 属性中包含了原有存入的数据，内容是 John Smith 雇员的原始 JSON 文档：

```
{
  "_index" :   "megacorp",
  "_type" :    "employee",
  "_id" :      "1",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "first_name" :  "John",
      "last_name" :   "Smith",
      "age" :         25,
      "about" :       "I love to go rock climbing",
      "interests":  [ "sports", "music" ]
  }
}
```

**将 HTTP 命令由 PUT 改为 GET 可以用来检索文档，同样的，可以使用 DELETE 命令来删除文档，以及使用 HEAD 指令来检查文档是否存在。如果想更新已存在的文档，只需再次 PUT 。**

#### 轻量搜索

```
curl -XGET 'http://localhost:9200/megacorp/employee/_search'
```

这个返回值是所有的数据。其检索返回的数据都存在 `_source` 中，其他的数据都是一些元数据。
：

```
{
   "took":      6,
   "timed_out": false,
   "_shards": { ... },
   "hits": {
      "total":      3,
      "max_score":  1,
      "hits": [
         {
            "_index":         "megacorp",
            "_type":          "employee",
            "_id":            "3",
            "_score":         1,
            "_source": {
               "first_name":  "Douglas",
               "last_name":   "Fir",
               "age":         35,
               "about":       "I like to build cabinets",
               "interests": [ "forestry" ]
            }
         },
         {
            "_index":         "megacorp",
            "_type":          "employee",
            "_id":            "1",
            "_score":         1,
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         },
         {
            "_index":         "megacorp",
            "_type":          "employee",
            "_id":            "2",
            "_score":         1,
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}
```

还可以通过指定属性来进行搜索：

```
curl -XGET 'http://localhost:9200/megacorp/employee/_search?q=last_name:Smith'
```

#### 使用查询表达式搜索

Query-string 搜索通过命令非常方便地进行临时性的即席搜索 ，但它有自身的局限性）。Elasticsearch 提供一个丰富灵活的查询语言叫做**查询表达式**， 它支持构建更加复杂和健壮的查询。

领域特定语言 （DSL）， 指定了使用一个 JSON 请求。我们可以像这样重写之前的查询所有 Smith 的搜索 ：

```
curl -XGET 'http://localhost:9200/megacorp/employee/_search' -d '
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}'
```

返回结果与之前的查询一样，但还是可以看到有一些变化。其中之一是，不再使用 `query-string` 参数，而是一个请求体替代。这个请求使用 JSON 构造，并使用了一个 `match` 查询（属于查询类型之一，后续将会了解）。

#### 更复杂的搜索

现在尝试下更复杂的搜索。 同样搜索姓氏为 Smith 的雇员，但这次我们只需要年龄大于 `30` 的。查询需要稍作调整，使用过滤器 `filter` ，它支持高效地执行一个结构化查询。

```
curl -XGET 'http://localhost:9200/megacorp/employee/_search' -d '
{
    "query" : {
        "bool": {
            "must": {
                "match" : {
                    "last_name" : "smith" 
                }
            },
            "filter": {
                "range" : {
                    "age" : { "gt" : 30 } 
                }
            }
        }
    }
}'
```

#### 全文搜索

搜索下所有喜欢攀岩（`rock climbing`）的雇员：

```
curl -XGET 'http://localhost:9200/megacorp/employee/_search' -d '
{
    "query" : {
        "match" : {
            "about" : "rock climbing"
        }
    }
}'
```

Elasticsearch 默认按照相关性得分排序，即每个文档跟查询的匹配程度。

```
{
   ...
   "hits": {
      "total":      2,
      "max_score":  0.16273327,
      "hits": [
         {
            ...
            "_score":         0.16273327, 
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         },
         {
            ...
            "_score":         0.016878016, 
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}
```

我们可以看到结果中，只有第一个结果出现了 `rock climbing`，第二个结果中只出现了 `rock`。

#### 短语搜索

找出一个属性中的独立单词是没有问题的，但有时候想要精确匹配一系列单词或者短语 。 比如， 我们想执行这样一个查询，仅匹配同时包含 “rock” 和 “climbing” ，并且 二者以短语 “rock climbing” 的形式紧挨着的雇员记录。

为此对 match 查询稍作调整，使用一个叫做 `match_phrase` 的查询：

```
curl -XGET 'http://localhost:9200/megacorp/employee/_search' -d '
{
    "query" : {
        "match_phrase" : {
            "about" : "rock limbing" // 其中空格的数量不会影响查询的结果
        }
    }
}'
```

#### 高亮搜索

许多应用都倾向于在每个搜索结果中 高亮 部分文本片段，以便让用户知道为何该文档符合查询条件。在 Elasticsearch 中检索出高亮片段也很容易。

再次执行前面的查询，并增加一个新的 `highlight` 参数：

```
curl -XGET 'http://localhost:9200/megacorp/employee/_search' -d '
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    },
    "highlight": {
        "fields" : {
            "about" : {}
        }
    }
}'
```

当执行该查询时，返回结果与之前一样，与此同时结果中还多了一个叫做 `highlight` 的部分。这个部分包含了 `about` 属性匹配的文本片段，并以 HTML 标签 `<em></em>` 封装：

```
{
   ...
   "hits": {
      "total":      1,
      "max_score":  0.23013961,
      "hits": [
         {
            ...
            "_score":         0.23013961,
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            },
            "highlight": {
               "about": [
                  "I love to go <em>rock</em> <em>climbing</em>" 
               ]
            }
         }
      ]
   }
}
```

#### 分析

终于到了最后一个业务需求：支持管理者对雇员目录做分析。 Elasticsearch 有一个功能叫聚合（aggregations），允许我们基于数据生成一些精细的分析结果。聚合与 SQL 中的 `GROUP BY` 类似但更强大。也就是对一个分组中的数据进行分析，比如统计次数，求平均值等。

举个例子，统计雇员中兴趣爱好的次数排名：

```
curl -XGET 'http://localhost:9200/megacorp/employee/_search' -d '
{
  "aggs": {
    "all_interests": {
      "terms": { "field": "interests" }
    }
  }
}'
```

还可以求平均值：

```
curl -XGET 'http://localhost:9200/megacorp/employee/_search' -d '
{
    "aggs" : {
        "all_interests" : {
            "terms" : { "field" : "interests" },
            "aggs" : {
                "avg_age" : {
                    "avg" : { "field" : "age" }
                }
            }
        }
    }
}
```

## 深入搜索

### 结构化搜索

**结构化搜索（Structured search）**是指有关探询那些具有内在结构数据的过程。比如日期、时间和数字都是结构化的：它们有精确的格式，我们可以对这些格式进行逻辑操作。比较常见的操作包括比较数字或时间的范围，或判定两个值的大小。

在结构化查询中，我们得到的结果 总是 非是即否，要么存于集合之中，要么存在集合之外。结构化查询不关心文件的相关度或评分；它简单的对文档包括或排除处理。

#### 精确值查找

当进行精确值查找时， 我们会使用过滤器（filters）。过滤器很重要，因为它们执行速度非常快，不会计算相关度（直接跳过了整个评分阶段）而且很容易被缓存。现在只要记住：请尽可能多的使用过滤式查询。

##### term 查询

类似于 `SQL` 语句中的

```
SELECT document
FROM   products
WHERE  price = 20
```

最为常用的 `term` 查询， 可以用它处理数字（numbers）、布尔值（Booleans）、日期（dates）以及文本（text）。

```
curl -XPUT 'http://localhost:9200/my_store/products/_bulk' -d '
{ "index": { "_id": 1 }}
{ "price" : 10, "productID" : "XHDK-A-1293-#fJ3" }
{ "index": { "_id": 2 }}
{ "price" : 20, "productID" : "KDKE-B-9947-#kL5" }
{ "index": { "_id": 3 }}
{ "price" : 30, "productID" : "JODL-X-1937-#pV7" }
{ "index": { "_id": 4 }}
{ "price" : 30, "productID" : "QQPX-R-3956-#aD8" }'
```

这是通过 `_bulk` 添加数据的另一种方法

在 Elasticsearch 的查询表达式（query DSL）中，我们可以使用 term 查询达到相同的目的。 term 查询会查找我们指定的精确值。作为其本身， term 查询是简单的。它接受一个字段名以及我们希望查找的数值：

```
curl -XGET 'http://localhost:9200/my_store/products/_search' -d '
{
    "query" : {
        "term" : { 
            "price" : 20
        }
    }
}'
```

通常当查找一个精确值的时候，我们不希望对查询进行评分计算。只希望对文档进行包括或排除的计算，所以我们会使用 `constant_score` 将 term 查询转化成为过滤器。查询以非评分模式来执行 `term` 查询并以一作为统一评分。

最终组合的结果是一个 `constant_score` 查询，它包含一个 `term` 查询：

```
curl -XGET 'http://localhost:9200/my_store/products/_search' -d '
{
    "query" : {
        "constant_score": {
            "filter": {
                "term" : { 
                    "price" : 20
                }
            }
        }
    }
}'
```

**term 查询文本**

使用如下的查询表达式（query DSL）不会获得查询结果。

```
curl -XGET 'http://localhost:9200/my_store/products/_search' -d '
{
    "query" : {
        "constant_score": {
            "filter": {
                "term" : { 
                    "productID" : "XHDK-A-1293-#fJ3"
                }
            }
        }
    }
}'
```
问题不在 `term` 查询，而在于索引数据的方式。 如果我们使用 `analyze` API ([分析 API](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/analysis-intro.html#analyze-api))，我们可以看到这里的 UPC（Universal Product Code） 码被拆分成多个更小的 token ：

```
curl -XGET 'http://localhost:9200/my_store/_analyze' -d '{
  "field": "productID",
  "text": "XHDK-A-1293-#fJ3"
}'
```

得到如下的返回，可以看到 `XHDK-A-1293-#fJ3` 被分解成了 `xhdk`、`a`、`1293`、`fj3`

```
{
  "tokens" : [ {
    "token" :        "xhdk",
    "start_offset" : 0,
    "end_offset" :   4,
    "type" :         "<ALPHANUM>",
    "position" :     1
  }, {
    "token" :        "a",
    "start_offset" : 5,
    "end_offset" :   6,
    "type" :         "<ALPHANUM>",
    "position" :     2
  }, {
    "token" :        "1293",
    "start_offset" : 7,
    "end_offset" :   11,
    "type" :         "<NUM>",
    "position" :     3
  }, {
    "token" :        "fj3",
    "start_offset" : 13,
    "end_offset" :   16,
    "type" :         "<ALPHANUM>",
    "position" :     4
  } ]
}
```

* Elasticsearch 用 4 个不同的 token 而不是单个 token 来表示这个 UPC 。
* 所有字母都是小写的。
* 丢失了连字符（`-`）和哈希符（`#`）。

所以当我们用 term 查询查找精确值 `XHDK-A-1293-#fJ3` 的时候，找不到任何文档，因为它并不在我们的倒排索引中

为了避免这种问题，我们需要告诉 Elasticsearch 该字段具有精确值，要将其设置成 `not_analyzed` 无需分析的。为了修正搜索结果，我们需要首先删除旧索引（因为它的映射不再正确）然后创建一个能正确映射的新索引:

```
curl -XDELETE 'http://localhost:9200/my_store'

curl -XPUT 'http://localhost:9200/my_store' -d '
{
    "mappings" : {
        "products" : {
            "properties" : {
                "productID" : {
                    "type" : "string",
                    "index" : "not_analyzed" 
                }
            }
        }
    }
}'
```

然后为文档重建索引：

```
curl -XPUT 'http://localhost:9200/my_store/products/_bulk' -d '
{ "index": { "_id": 1 }}
{ "price" : 10, "productID" : "XHDK-A-1293-#fJ3" }
{ "index": { "_id": 2 }}
{ "price" : 20, "productID" : "KDKE-B-9947-#kL5" }
{ "index": { "_id": 3 }}
{ "price" : 30, "productID" : "JODL-X-1937-#pV7" }
{ "index": { "_id": 4 }}
{ "price" : 30, "productID" : "QQPX-R-3956-#aD8" }'
```

此时就可以精确查询到我们需要的结果。

#### 组合过滤器

类似于 SQL 语句中的

```
SELECT product
FROM   products
WHERE  (price = 20 OR productID = "XHDK-A-1293-#fJ3")
  AND  (price != 30)
```

这种情况下，我们需要 `bool` （布尔）过滤器。

##### 布尔过滤器

一个 `bool` 过滤器由三部分组成：

```
{
   "bool" : {
      "must" :     [],
      "should" :   [],
      "must_not" : [],
   }
}
```

`must`：所有的语句都 必须（must） 匹配，与 `AND` 等价。
`must_not`：所有的语句都 不能（must not） 匹配，与 `NOT` 等价。
`should`：至少有一个语句要匹配，与 `OR` 等价。

上面的 SQL 语句解释出来如下：

```
curl -XGET 'http://localhost:9200/my_store/products/_search' -d '
{
   "query" : {
        "bool" : {
          "should" : [
             { "term" : {"price" : 20}}, 
             { "term" : {"productID" : "XHDK-A-1293-#fJ3"}} 
          ],
          "must_not" : {
             "term" : {"price" : 30} 
          }
       }
    }
}'
```

在 `should` 语句块里面的两个 `term` 过滤器与 `bool` 过滤器是父子关系，两个 `term` 条件需要匹配其一。

##### 嵌套布尔过滤器

尽管 `bool` 是一个复合的过滤器，可以接受多个子过滤器，需要注意的是`bool` 过滤器本身仍然还只是一个过滤器。 这意味着我们可以将一个 `bool` 过滤器置于其他 `bool` 过滤器内部，这为我们提供了对任意复杂布尔逻辑进行处理的能力。

对于以下这个 SQL 语句：

```
SELECT document
FROM   products
WHERE  productID = "KDKE-B-9947-#kL5"
  OR ( productID = "JODL-X-1937-#pV7" AND price = 30 )
```

我们将其转换成一组嵌套的 `bool` 过滤器：

```
curl -XGET 'http://localhost:9200/my_store/products/_search' -d '
{
   "query" : {
        "bool" : {
              "should" : [
                { "term" : {"productID" : "KDKE-B-9947-#kL5"}}, 
                { "bool" : { 
                  "must" : [
                    { "term" : {"productID" : "JODL-X-1937-#pV7"}}, 
                    { "term" : {"price" : 30}} 
                  ]
                }}
              ]
           }
    }
}'
```

#### 查找多个精确值

不需要使用多个 `term` 查询，我们只要用单个 `terms` 查询（注意末尾的 s ）， `terms` 查询好比是 `term` 查询的复数形式（以英语名词的单复数做比）

与 `term` 查询一样，也需要将其置入 `filter` 语句的常量评分查询中使用：

```
curl -XGET 'http://localhost:9200/my_store/products/_search' -d '
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "terms" : { 
                    "price" : [20, 30]
                }
            }
        }
    }
}'
```

#### 范围

本章到目前为止，对于数字，只介绍如何处理精确值查询。 实际上，对数字范围进行过滤有时会更有用。

```
SELECT document
FROM   products
WHERE  price BETWEEN 20 AND 40
```

range 查询可同时提供包含（inclusive）和不包含（exclusive）这两种范围表达式，可供组合的选项如下：

`gt`: > 大于（greater than）
`lt`: < 小于（less than）
`gte`: >= 大于或等于（greater than or equal to）
`lte`: <= 小于或等于（less than or equal to）

```
curl -XGET 'http://localhost:9200/my_store/products/_search' -d '
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "range" : {
                    "price" : {
                        "gte" : 20,
                        "lte"  : 40
                    }
                }
            }
        }
    }
}'
```

##### 日期范围

range 查询同样可以应用在日期字段上：

```
"range" : {
    "timestamp" : {
        "gt" : "2014-01-01 00:00:00",
        "lt" : "2014-01-07 00:00:00"
    }
}
```

 range 查询支持对 **日期计算（date math）** 进行操作，比方说，如果我们想查找时间戳在过去一小时内的所有文档：

```
"range" : {
    "timestamp" : {
        "gt" : "now-1h"
    }
}
```

日期计算还可以被应用到某个具体的时间，并非只能是一个像 `now` 这样的占位符。只要在某个日期后加上一个双管符号 (`||`) 并紧跟一个日期数学表达式就能做到：

```
"range" : {
    "timestamp" : {
        "gt" : "2014-01-01 00:00:00",
        "lt" : "2014-01-01 00:00:00||+1M" 
    }
}
```

#### 处理 Null 值

回想在之前例子中，有的文档有名为 tags （标签）的字段，它是个多值字段， 一个文档可能有一个或多个标签，也可能根本就没有标签。**如果一个字段没有值，那么如何将它存入倒排索引**中的呢？

答案是：什么都不存。

一个倒排索引只是一个 token 列表和与之相关的文档信息，如果字段不存在，那么它也不会持有任何 token，也就无法在倒排索引结构中表现。

最终，这也就意味着 ，**`null`, `[]` （空数组）和 `[null]` 所有这些都是等价的，它们无法存于倒排索引中。**

Elasticsearch 提供了一些工具来处理空或缺失值

##### 存在查询

`exists` 可以实现存在查询。

在 SQL 语句中，如下的查询：

```
SELECT tags
FROM   posts
WHERE  tags IS NOT NULL
```

可以用 `exists` 实现：

```
GET /my_index/posts/_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "exists" : { "field" : "tags" }
            }
        }
    }
}
```

`missing` 查询本质上与 `exists` 恰好相反： 它返回某个特定 **无**值字段的文档

```
SELECT tags
FROM   posts
WHERE  tags IS NULL
```

的实现方法为：

```
GET /my_index/posts/_search
{
    "query" : {
        "constant_score" : {
            "filter": {
                "missing" : { "field" : "tags" }
            }
        }
    }
}
```

##### 对象上的存在于缺失

我们不仅可以检查 `name.first` 和 `name.last` 的存在性，也可以检查 `name` ，不过在 映射 中，如上对象的内部是个扁平的字段与值（field-value）的简单键值结构，类似下面这样：

```
{
   "name.first" : "John",
   "name.last"  : "Smith"
}
```

`name` 字段并不真实存在于倒排索引中。

原因是当我们执行下面这个过滤的时候：

```
{
    "exists" : { "field" : "name" }
}
```

实际执行的是：

```
{
    "bool": {
        "should": [
            { "exists": { "field": "name.first" }},
            { "exists": { "field": "name.last" }}
        ]
    }
}
```

这也就意味着，如果 `first` 和 `last` 都是空，那么 `name` 这个命名空间才会被认为不存在。

#### 关于缓存

过滤器是如何计算的： 其核心实际是采用一个 bitset 记录与过滤器匹配的文档。Elasticsearch 积极地把这些 bitset 缓存起来以备随后使用。一旦缓存成功，bitset 可以复用**任何 已使用过的相同过滤器，而无需再次计算整个过滤器**。

开发者难以区分有良好表现的缓存以及无用缓存。

为了解决问题，Elasticsearch 会基于使用频次自动缓存查询。如果一个非评分查询在最近的 256 词查询中被使用过（次数取决于查询类型），那么这个查询就会作为缓存的候选。

### 全文搜索

怎样在全文字段中搜索到最相关的文档。

全文搜索两个最重要的方面是：

**相关性（Relevance）**：它是评价查询与其结果间的相关程度，并根据这种相关程度对结果排名的一种能力，这种计算方式可以是 TF/IDF 方法、地理位置邻近、模糊相似，或其他的某些算法。

**分析（Analysis）**：它是将文本块转换为有区别的、规范化的 token 的一个过程，目的是为了（a）创建倒排索引以及（b）查询倒排索引。

一旦谈论相关性或分析这两个方面的问题时，我们所处的语境是关于**查询（`match`）**的而不是**过滤（`filter`）**。

#### 基于词项与基于全文

文本查询可以划分成两大家族：

##### 基于词项的查询

如 `term` 或 `fuzzy` 这样的底层查询不需要分析阶段，它们对单个词项进行操作。用 `term` 查询词项 `Foo` 只要在倒排索引中查找 准确词项 ，并且用 TF/IDF 算法为每个包含该词项的文档计算相关度评分 `_score` 。

记住 `term` 查询只对倒排索引的词项精确匹配，这点很重要，它不会对词的多样性进行处理（如， `foo` 或 `FOO` ）。

##### 基于全文的查询

像 `match` 或 `query_string` 这样的查询是高层查询，它们了解字段映射的信息：

* 如果查询 日期（date） 或 整数（integer） 字段，它们会将查询字符串分别作为日期或整数对待。
* 如果查询一个（ not_analyzed ）未分析的精确值字符串字段， 它们会将整个查询字符串作为单个词项对待。
* 但如果要查询一个（ analyzed ）已分析的全文字段， 它们会先将查询字符串传递到一个合适的分析器，然后生成一个供查询的词项列表。

**一旦组成了词项列表，这个查询会对每个词项逐一执行底层的查询，再将结果合并，然后为每个文档生成一个最终的相关度评分。**

我们很少直接使用基于词项的搜索，通常情况下都是对全文进行查询

#### 匹配查询

匹配查询 `match` 是个**核心**查询。无论需要查询什么字段， `match` 查询都应该会是首选的查询方式。 它是一个高级**全文查询**，这表示它**既能处理全文字段，又能处理精确字段**。

```
//删除已有的索引。
curl -XDELETE 'http://localhost:9200/my_index'

//只为这个索引分配一个主分片
curl -XPUT 'http://localhost:9200/my_index' -d '{ "settings": { "number_of_shards": 1 }}'

curl -XPOST 'http://localhost:9200/my_index/my_type/_bulk' -d '
{ "index": { "_id": 1 }}
{ "title": "The quick brown fox" }
{ "index": { "_id": 2 }}
{ "title": "The quick brown fox jumps over the lazy dog" }
{ "index": { "_id": 3 }}
{ "title": "The quick brown fox jumps over the quick dog" }
{ "index": { "_id": 4 }}
{ "title": "Brown fox brown dog" }
'
```

##### 单个词查询

我们用第一个示例来解释使用 `match` 查询搜索全文字段中的单个词：

```
curl -XGET 'http://localhost:9200/my_index/my_type/_search?pretty' -d '
{
    "query": {
        "match": {
            "title": "QUICK!"
        }
    }
}
'
```

Elasticsearch 执行上面这个 match 查询的步骤是：

* 检查字段类型 ：标题 `title` 字段是一个 `string` 类型（ analyzed ）已分析的全文字段，这意味着查询字符串本身也应该被分析。
* 分析查询字符串 ：将查询的字符串 `QUICK!` 传入标准分析器中，输出的结果是单个项 `quick` 。因为只有一个单词项，所以 `match` 查询执行的是单个底层 `term` 查询。
* 查找匹配文档 ：用 `term` 查询在倒排索引中查找 `quick` 然后获取一组包含该项的文档，本例的结果是文档：`1`、`2` 和 `3` 。
* 为每个文档评分 ：用 `term` 查询计算每个文档相关度评分 `_score` ，这是种将**词频**（term frequency，即词 `quick` 在相关文档的 `title` 字段中出现的频率）和反向文档频率（inverse document frequency，即词 `quick` 在所有文档的 title 字段中出现的频率），以及字段的长度（即字段越短相关度越高）相结合的计算方式。

#### 多词查询

```
curl -XGET 'http://localhost:9200/my_index/my_type/_search?pretty' -d '
{
    "query": {
        "match": {
            "title": "BROWN DOG!"
        }
    }
}
'
```

`match` 查询必须查找两个词（ `["brown","dog"]` ），它在内部实际上先执行两次 `term` 查询，然后将两次查询的结果合并作为最终结果输出。为了做到这点，它将两个 `term` 查询包入一个 `bool` 查询中

##### 提高精度

用 **任意** 查询词项匹配文档可能会导致结果中出现不相关的长尾。 这是种散弹式搜索。可能我们只想搜索包含 **所有** 词项的文档，也就是说，不去匹配 `brown OR dog` ，而通过匹配 `brown AND dog` 找到所有文档。

```
curl -XGET 'http://localhost:9200/my_index/my_type/_search?pretty' -d '
{
    "query": {
        "match": {
            "title": {      
                "query":    "BROWN DOG!",
                "operator": "and"
            }
        }
    }
}
'
```

##### 控制精度

在 **所有** 与 **任意** 间二选一有点过于非黑即白。 如果用户给定 5 个查询词项，想查找只包含其中 4 个的文档

`match` 查询支持 `minimum_should_match` 最小匹配参数， 这让我们可以指定必须匹配的词项数用来表示一个文档是否相关。我们可以将其设置为某个具体数字，更常用的做法是将其设置为一个百分数，因为我们无法控制用户搜索时输入的单词数量：

```
curl -XGET 'http://localhost:9200/my_index/my_type/_search?pretty' -d '
{
  "query": {
    "match": {
      "title": {
        "query": "quick brown dog",
        "minimum_should_match": "75%"
      }
    }
  }
}
'
```

除此以外 `minimum_should_match` 还有很多种写法，具体可以产考[Query DSL » Minimum Should Match](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-minimum-should-match.html#query-dsl-minimum-should-match)

#### 组合查询

在 组合过滤器 中，我们讨论过如何使用 `bool` 过滤器通过 `and` 、 `or` 和 `not` 逻辑组合将多个过滤器进行组合。在查询中， `bool` 查询有类似的功能，只有一个重要的区别。

```
curl -XGET 'http://localhost:9200/my_index/my_type/_search?pretty' -d '
{
  "query": {
    "bool": {
      "must":     { "match": { "title": "quick" }},
      "must_not": { "match": { "title": "lazy"  }},
      "should": [
                  { "match": { "title": "brown" }},
                  { "match": { "title": "dog"   }}
      ]
    }
  }
}
'
```

以上的查询结果返回 `title` 字段包含词项 `quick` 但不包含 `lazy` 的任意文档。目前为止，这与 `bool` 过滤器的工作方式非常相似。

区别就在于两个 `should` 语句，也就是说：一个文档不必包含 `brown` 或 `dog` 这两个词项，但如果一旦包含，我们就认为它们 更相关 

##### 评分计算

`bool` 查询会为每个文档计算相关度评分 `_score` ， 再将所有匹配的 `must` 和 `should` 语句的分数 `_score` 求和，最后除以 `must` 和 `should` 语句的总数。

`must_not` 语句不会影响评分；它的作用只是将不相关的文档排除。

##### 控制精度

所有 `must` 语句必须匹配，所有 `must_not` 语句都必须不匹配，但有多少 `should` 语句应该匹配呢？ 默认情况下，没有 `should` 语句是必须匹配的，只有一个例外：那就是当没有 `must` 语句的时候，至少有一个 `should` 语句必须匹配。

就像我们能控制 `match` 查询的精度 一样，我们可以通过 `minimum_should_match` 参数控制需要匹配的 `should` 语句的数量， 它既可以是一个绝对的数字，又可以是个百分比：

```
GET /my_index/my_type/_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title": "brown" }},
        { "match": { "title": "fox"   }},
        { "match": { "title": "dog"   }}
      ],
      "minimum_should_match": 2 
    }
  }
}
```

#### 查询语句提升权重

`should` 语句匹配得越多表示文档的相关度越高。目前为止还挺好。

但是如果我们想让包含 `Lucene` 的有更高的权重，并且包含 `Elasticsearch` 的语句比 `Lucene` 的权重更高，该如何处理?

我们可以通过指定 `boost` 来控制任何查询语句的相对的权重， `boost` 的默认值为 1 ，大于 1 会提升一个语句的相对权重。

```
GET /_search
{
    "query": {
        "bool": {
            "must": {
                "match": {  
                    "content": {
                        "query":    "full text search",
                        "operator": "and"
                    }
                }
            },
            "should": [
                { "match": {
                    "content": {
                        "query": "Elasticsearch",
                        "boost": 3 
                    }
                }},
                { "match": {
                    "content": {
                        "query": "Lucene",
                        "boost": 2 
                    }
                }}
            ]
        }
    }
}
```

### 多字段搜索

查询很少是简单一句话的 `match` 匹配查询。通常我们需要用相同或不同的字符串查询一个或多个字段，也就是说，需要对多个查询语句以及它们相关度评分进行合理的合并。

#### 多字符串查询

最简单的多字段查询可以将搜索项映射到具体的字段。 如果我们知道 War and Peace 是标题，Leo Tolstoy 是作者，很容易就能把两个条件用 match 语句表示， 并将它们用 `bool` 查询 组合起来，当然，并不是只能使用 match 语句：可以用 bool 查询来包裹组合任意其他类型的查询， 甚至包括其他的 bool 查询。我们可以添加一条语句来指定译者版本的偏好：

```
ET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title":  "War and Peace" }},
        { "match": { "author": "Leo Tolstoy"   }},
        { "bool":  {
          "should": [
            { "match": { "translator": "Constance Garnett" }},
            { "match": { "translator": "Louise Maude"      }}
          ]
        }}
      ]
    }
  }
}
```

为什么将译者条件语句放入另一个独立的 bool 查询中呢？所有的四个 match 查询都是 should 语句，所以为什么不将 translator 语句与其他如 title 、 author 这样的语句放在同一层呢？

答案在于评分的计算方式。含 `translator` 语句的 `bool` 查询，只占总评分的三分之一。如果将 `translator` 语句与 `title` 和 `author` 两条语句放入同一层，那么 `title` 和 `author` 语句只贡献四分之一评分。

##### 语句的优先级

为了提升 `title` 和 `author` 字段的权重， 为它们分配的 `boost` 值大于 1 ：

```
{
  "query": {
    "bool": {
      "should": [
        { "match": { 
            "title":  {
              "query": "War and Peace",
              "boost": 2
        }}},
        { "match": { 
            "author":  {
              "query": "Leo Tolstoy",
              "boost": 2
        }}},
        { "bool":  { 
            "should": [
              { "match": { "translator": "Constance Garnett" }},
              { "match": { "translator": "Louise Maude"      }}
            ]
        }}
      ]
    }
  }
}
```

#### 单字符串查询

有些用户期望将所有的搜索项堆积到单个字段中，并期望应用程序能为他们提供正确的结果。

对于**多词（multiword）、多字段（multifield）查询来说，不存在简单的 万能 方案。为了获得最好结果，需要 了解我们的数据 ，并了解如何使用合适的工具。**

#### 最佳字段

下面两篇博客内容文档为例：

```
PUT /my_index/my_type/1
{
    "title": "Quick brown rabbits",
    "body":  "Brown rabbits are commonly seen."
}

PUT /my_index/my_type/2
{
    "title": "Keeping pets healthy",
    "body":  "My quick brown fox eats rabbits on a regular basis."
}
```

现在运行以下 `bool` 查询：

```
{
    "query": {
        "bool": {
            "should": [
                { "match": { "title": "Brown fox" }},
                { "match": { "body":  "Brown fox" }}
            ]
        }
    }
}
```

用肉眼判断，文档 2 的匹配度更高

我们发现查询的结果是文档 1 的评分更高：

```
  "hits": [
     {
        "_id":      "1",
        "_score":   0.14809652,
        "_source": {
           "title": "Quick brown rabbits",
           "body":  "Brown rabbits are commonly seen."
        }
     },
     {
        "_id":      "2",
        "_score":   0.09256032,
        "_source": {
           "title": "Keeping pets healthy",
           "body":  "My quick brown fox eats rabbits on a regular basis."
        }
     }
  ]
}
```

回想一下 bool 是如何计算评分的：

1. 它会执行 should 语句中的两个查询。
2. 加和两个查询的评分。
3. 乘以匹配语句的总数。
4. 除以所有语句总数（这里为：2）。

在本例中， `title` 和 `body` 字段是相互竞争的关系，所以就需要找到单个 **最佳匹配** 的字段

##### dis_max 查询

将任何与任一查询匹配的文档作为结果返回，但只将**最佳匹配的评分作为查询的评分结果**返回 ：

```
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Brown fox" }},
                { "match": { "body":  "Brown fox" }}
            ]
        }
    }
}
```

##### 最佳字段查询调优

因为使用 `dis_max` 的时候，只会将其中最佳匹配的份数作为总体的分数，但是在如下的情况下：两个文档中都不具有同时包含 **两个词** 的 **相同字段** 。则分数一致。

```
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Quick pets" }},
                { "match": { "body":  "Quick pets" }}
            ]
        }
    }
}
```

```
{
  "hits": [
     {
        "_id": "1",
        "_score": 0.12713557, 
        "_source": {
           "title": "Quick brown rabbits",
           "body": "Brown rabbits are commonly seen."
        }
     },
     {
        "_id": "2",
        "_score": 0.12713557, 
        "_source": {
           "title": "Keeping pets healthy",
           "body": "My quick brown fox eats rabbits on a regular basis."
        }
     }
   ]
}
```

但是肉眼观察，第二个的分数应该比第一个高。

##### tie_breaker 参数

可以通过指定 `tie_breaker` 这个参数将其他匹配语句的评分也考虑其中：

```
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Quick pets" }},
                { "match": { "body":  "Quick pets" }}
            ],
            "tie_breaker": 0.3
        }
    }
}
```

`tie_breaker`` 参数提供了一种 `dis_max` 和 `bool` 之间的折中选择，它的评分方式如下：

1. 获得最佳匹配语句的评分 _score 。
2. 将其他匹配语句的评分结果与 tie_breaker 相乘。
3. 对以上评分求和并规范化。

有了 `tie_breaker` ，会考虑所有匹配语句，但**最佳匹配语句依然占最终结果里的很大**一部分。

```
tie_breaker 可以是 0 到 1 之间的浮点数，其中 0 代表使用 dis_max 最佳匹配语句的普通逻辑， 1 表示所有匹配语句同等重要。最佳的精确值需要根据数据与查询调试得出，但是合理值应该与零接近（处于 0.1 - 0.4 之间），这样就不会颠覆 dis_max 最佳匹配性质的根本。
```

#### multi_match 查询

`multi_match` 查询为能在多个字段上反复执行相同查询提供了一种便捷方式。

`multi_match` 多匹配查询的类型有多种，其中的三种恰巧与 了解我们的数据 中介绍的三个场景对应，即： `best_fields` 、 `most_fields` 和 `cross_fields` （最佳字段、多数字段、跨字段）。

默认情况下，查询的类型是 `best_fields` ，它会为每个字段生成一个 `match` 查询，然后将它们组合到 `dis_max` 查询的内部

```
{
    "multi_match": {
        "query":                "Quick brown fox",
        "type":                 "best_fields", 
        "fields":               [ "title", "body" ],
        "tie_breaker":          0.3,
        "minimum_should_match": "30%" 
    }
}
```

等价于：

```
  "dis_max": {
    "queries":  [
      {
        "match": {
          "title": {
            "query": "Quick brown fox",
            "minimum_should_match": "30%"
          }
        }
      },
      {
        "match": {
          "body": {
            "query": "Quick brown fox",
            "minimum_should_match": "30%"
          }
        }
      },
    ],
    "tie_breaker": 0.3
  }
}
```

##### 查询字段名称的模糊匹配

匹配结尾为 `_title` 的字段

```
{
    "multi_match": {
        "query":  "Quick brown fox",
        "fields": "*_title"
    }
}
```

##### 提升单个字段的权重

可以使用`^` 字符语法为单个字段提升权重，在字段名称的末尾添加 `^boost` ， 其中 `boost` 是一个浮点数：

```
{
    "multi_match": {
        "query":  "Quick brown fox",
        "fields": [ "*_title", "chapter_title^2" ] 
    }
}
```

#### 多数字段

召回率（返回结果中的所有文档都是相关的）：扩大搜索范围 ——不仅返回与用户搜索词精确匹配的文档，还会返回我们认为与查询相关的所有文档。如果一个用户搜索 `quick brown fox` ，一个包含词语 `fast foxes` 的文档被认为是非常合理的返回结果。

##### 多字段映射

首先要做的事情就是对我们的字段索引两次： 一次使用词干模式以及一次非词干模式。

`title` 字段使用 `english` 英语分析器来提取词干。用来扩大匹配的范围。

`title.std` 字段使用 `standard` 标准分析器，所以没有词干提取。用来进行精确匹配

```
DELETE /my_index

PUT /my_index
{
    "settings": { "number_of_shards": 1 }, 
    "mappings": {
        "my_type": {
            "properties": {
                "title": { 
                    "type":     "string",
                    "analyzer": "english",
                    "fields": {
                        "std":   { 
                            "type":     "string",
                            "analyzer": "standard"
                        }
                    }
                }
            }
        }
    }
}
```

```
PUT /my_index/my_type/1
{ "title": "My rabbit jumps" }

PUT /my_index/my_type/2
{ "title": "Jumping jack rabbits" }
```
我们希望将所有匹配字段的评分合并起来，所以使用 `most_fields` 类型。这让 `multi_match` 查询用 `bool` 查询将两个字段语句包在里面，而不是使用 `dis_max` 查询。

每个字段对于最终评分的贡献可以通过自定义值 `boost` 来控制。比如，使 `title` 字段更为重要，这样同时也降低了其他信号字段的作用：

用广度匹配字段 `title` 包括尽可能多的文档——以提升召回率——同时又使用字段 `title.std` 作为**信号**将相关度更高的文档置于结果顶部。

```
GET /my_index/_search
{
   "query": {
        "multi_match": {
            "query":       "jumping rabbits",
            "type":        "most_fields",
            "fields":      [ "title^10", "title.std" ] 
        }
    }
}
```

#### 跨字段实体搜索

采用 `multi_match` 查询， 将 `type`设置成` most_fields` 然后告诉 Elasticsearch 合并所有匹配字段的评分：

```
{
  "query": {
    "multi_match": {
      "query":       "Poland Street W1V",
      "type":        "most_fields",
      "fields":      [ "street", "city", "country", "postcode" ]
    }
  }
}
```

##### most_fields 方式的问题

用` most_fields` 这种方式搜索也存在某些问题，这些问题并不会马上显现：

1. 它是为多数字段匹配 任意 词设计的，而不是在 所有字段 中找到最匹配的。
2. 它不能使用 `operator` 或 `minimum_should_match` 参数来降低次相关结果造成的长尾效应。
3. 词频对于每个字段是不一样的，而且它们之间的相互影响会导致不好的排序结果。

##### 字段中心式查询

`most_fields` 和 `best_fields` 两个被查询字段都与一个 field 匹配的文档要比一个字段同时匹配两个field 文档的评分高。

#### cross-fields 跨字段查询

使用 `cross_fields` 类型进行 `multi_match `查询。 `cross_fields` 使用词中心式（`term-centric`）的查询方式，这与 `best_fields` 和 `most_fields` 使用字段中心式（`field-centric`）的查询方式非常不同，它将所有字段当成一个大字段，并在 **每个字段** 中查找 **每个词** 。

**字段中心式 **会使用以下逻辑：

```
(+first_name:peter +first_name:smith)
(+last_name:peter  +last_name:smith)
```

**词中心式 **会使用以下逻辑：

```
+(first_name:peter last_name:peter)
+(first_name:smith last_name:smith)
```

换句话说，词 peter 和 smith 都必须出现，但是可以出现在任意字段中。

### 近似匹配

`match` 查询可以告知我们这大袋子中是否包含查询的词条，但却无法告知词语之间的关系。不能确定这词是否只来自于一种语境，甚至都不能确定是否来自于同一个段落。

我们可能会希望得到尽可能包含这词的文档，但我们也同样需要这些**文档与分词有很高的相关度**。

这就是短语匹配或者近似匹配的所属领域。

#### 短语匹配

```
GET /my_index/my_type/_search
{
    "query": {
        "match_phrase": {
            "title": "quick brown fox"
        }
    }
}
```

一个被认定为和短语 quick brown fox 匹配的文档，必须满足以下这些要求：

1. quick 、 brown 和 fox 需要全部出现在域中。
2. brown 的位置应该比 quick 的位置大 1 。
3. fox 的位置应该比 quick 的位置大 2 。

如果以上任何一个选项不成立，则该文档不能认定为匹配

#### 混合起来

精确短语匹配 或许是过于严格了。也许我们想要包含 “quick brown fox” 的文档也能够匹配 “quick fox,” ， 尽管情形不完全相同。

我们能够通过使用 `slop` 参数将灵活度引入短语匹配中：

```
GET /my_index/my_type/_search
{
    "query": {
        "match_phrase": {
            "title": {
                "query": "quick fox",
                "slop":  1
            }
        }
    }
}
```

`slop` 参数告诉 `match_phrase` 查询词条相隔多远时仍然能将文档视为匹配 。**如果找到了多条匹配结构，则匹配结果的评分中，如果两个词的距离越近，则评分越高。**

#### 多值字段

对多值字段使用短语匹配时会发生奇怪的事。 想象一下你索引这个文档:

```
PUT /my_index/groups/1
{
    "names": [ "John Abraham", "Lincoln Smith"]
}
```

然后运行一个对 Abraham Lincoln 的短语查询:

```
GET /my_index/groups/_search
{
    "query": {
        "match_phrase": {
            "names": "Abraham Lincoln"
        }
    }
}
```

令人惊讶的是， 即使 Abraham 和 Lincoln 在 names 数组里属于两个不同的人名， 我们的文档也匹配了查询。

幸运的是， 在这样的情况下有一种叫做 `position_increment_gap` 的简单的解决方案， 它在字段映射中配置 。

```
//首先删除映射 groups 以及这个类型内的所有文档。
DELETE /my_index/groups/ 
//然后创建一个有正确值的新的映射 groups 。
PUT /my_index/_mapping/groups 
{
    "properties": {
        "names": {
            "type":                "string",
            "position_increment_gap": 100
        }
    }
}
```

会产生如下的结果：

1. Position 1: john
2. Position 2: abraham
3. Position 103: lincoln
4. Position 104: smith

现在我们的短语查询可能无法匹配该文档因为 abraham 和 lincoln 之间的距离为 100 。 为了匹配这个文档你必须添加值为 100 的 `slop` 。

#### 性能优化

短语查询和邻近查询都比简单的 `query` 查询代价更高 。 一个 `match` 查询仅仅是看词条是否存在于倒排索引中，而一个 `match_phrase` 查询是必须计算并比较多个可能重复词项的位置。

##### 结果集重新评分

```
GET /my_index/my_type/_search
{
    "query": {
        "match": {  
            "title": {
                "query":                "quick brown fox",
                "minimum_should_match": "30%"
            }
        }
    },
    // 先进行一次搜索排序，把所有包含词的文档都提取出来，然后对前 50 个结果进行评分排序。来减少 match_phrase 的操作时间。
    "rescore": {
        "window_size": 50, 
        "query": {         
            "rescore_query": {
                "match_phrase": {
                    "title": {
                        "query": "quick brown fox",
                        "slop":  50
                    }
                }
            }
        }
    }
}
```

#### 寻找相关词

两个子句 `I’m not happy I’m working` 和 `I’m happy I’m not working` 包含相同 的单词，也拥有相同的邻近度，但含义截然不同。

如果索引单词对而不是索引独立的单词，就能对这些单词的上下文尽可能多的保留。

对句子 `Sue ate the alligator` ，不仅要将每一个单词（或者 unigram ）作为词项索引

```
["sue", "ate", "the", "alligator"]
```

也要将每个单词 以及它的邻近词 作为单个词项索引：

```
["sue ate", "ate the", "the alligator"]
```

这些单词对（或者 bigrams ）被称为 `shingles` 。

查看[寻找相关词](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/shingles.html)来学习如何创建 `shingles` 以及如何使用。

`shingles `不仅比短语查询更灵活， 而且性能也更好。 `shingles` 查询跟一个简单的 `match` 查询一样高效，而不用每次搜索花费短语查询的代价。只是在索引期间因为更多词项需要被索引会付出一些小的代价， 这也意味着有 `shingles `的字段会占用更多的磁盘空间。 然而，大多数应用写入一次而读取多次，所以在索引期间优化我们的查询速度是有意义的。

### 部分匹配

但如果想匹配部分而不是全部的词该怎么办？ **部分匹配** 允许用户指定查找词的一部分并找出所有包含这部分片段的词。

#### prefix 前缀查询

为了找到所有以 `W1` 开始的邮编，可以使用简单的 `prefix` 查询：

```
GET /my_index/address/_search
{
    "query": {
        "prefix": {
            "postcode": "W1"
        }
    }
}
```

`prefix` 查询是一个词级别的底层的查询，它不会在搜索之前分析查询字符串，它假定传入前缀就正是要查找的前缀

```
默认状态下， prefix 查询不做相关度评分计算，它只是将所有匹配的文档返回，并为每条结果赋予评分值 1 。它的行为更像是过滤器而不是查询。 prefix 查询和 prefix 过滤器这两者实际的区别就是过滤器是可以被缓存的，而查询不行。
```

为了支持前缀匹配，查询会做以下事情：

1. 扫描词列表并查找到第一个以 `W1` 开始的词。
2. 搜集关联的文档 ID 。
3. 移动到下一个词。
4. 如果这个词也是以 `W1` 开头，查询跳回到第二步再重复执行，直到下一个词不以`W1` 为止。

也就是顺序匹配，非常耗时。

#### 通配符与正则表达式查询

 `wildcard` 通配符查询也是一种底层基于词的查询， 与前缀查询不同的是它允许指定匹配的正则式。它使用标准的 shell 通配符查询： `?` 匹配任意字符， `*` 匹配 0 或多个字符。

```
GET /my_index/address/_search
{
    "query": {
        "wildcard": {
            "postcode": "W?F*HW" 
        }
    }
}
```

regexp 正则式查询允许写出更复杂的模式：

```
GET /my_index/address/_search
{
    "query": {
        "regexp": {
            "postcode": "W[0-9].+" 
        }
    }
}
```

`wildcard` 和 `regexp` 查询的工作方式与 `prefix` 查询完全一样，它们也需要扫描倒排索引中的词列表才能找到所有匹配的词，然后依次获取每个词相关的文档 ID ，与 `prefix` 查询的唯一不同是：它们能支持更为复杂的匹配模式。

```
prefix 、 wildcard 和 regexp 查询是基于词操作的，如果用它们来查询 analyzed 字段，它们会检查字段里面的每个词，而不是将字段作为整体来处理。
```

#### 查询时输入即搜索

用户已经渐渐习惯在输完查询内容之前，就能为他们展现搜索结果，这就是所谓的 即时搜索（instant search） 或 输入即搜索（search-as-you-type） 。不仅用户能在更短的时间内得到搜索结果，我们也能引导用户搜索索引中真实存在的结果。

例如，如果用户输入 `johnnie walker bl` ，我们希望在它们完成输入搜索条件前就能得到：Johnnie Walker Black Label 和 Johnnie Walker Blue Label 。

```
{
    "match_phrase_prefix" : {
        "brand" : "johnnie walker bl",
        "slop":  10
    }
}
```

这种查询的行为与 `match_phrase` 查询一致，不同的是它将查询字符串的最后一个词作为前缀使用，换句话说，可以将之前的例子看成如下这样：

1. johnnie
2. 跟着 walker
3. 跟着以 bl 开始的词

可以通过设置 max_expansions 参数来限制前缀扩展的影响， 一个合理的值是可能是 50 ：

```
{
    "match_phrase_prefix" : {
        "brand" : {
            "query":          "johnnie walker bl",
            "max_expansions": 50
        }
    }
}
```

参数 `max_expansions` 控制着可以与前缀匹配的词的数量，它会先查找第一个与前缀 `bl` 匹配的词，然后依次查找搜集与之匹配的词（按字母顺序），直到没有更多可匹配的词或当数量超过 `max_expansions` 时结束。

不要忘记，当用户每多输入一个字符时，这个查询又会执行一遍，所以查询需要快，如果第一个结果集不是用户想要的，他们会继续输入直到能搜出满意的结果为止。

#### Ngrams 在部分匹配的应用

但单个词的查找 确实 要比在词列表中盲目挨个查找的效率要高得多。 在搜索之前准备好供部分匹配的数据可以提高搜索的性能。但是会消耗存储的空间。

在索引时准备数据意味着要选择合适的分析链，这里部分匹配使用的工具是 `n-gram` 。可以将 `n-gram` 看成一个在词语上 滑动窗口 ， n 代表这个 “窗口” 的长度。如果我们要 `n-gram` quick 这个词 —— 它的结果取决于 n 的选择长度：

```
长度 1（unigram）： [ q, u, i, c, k ]
长度 2（bigram）： [ qu, ui, ic, ck ]
长度 3（trigram）： [ qui, uic, ick ]
长度 4（four-gram）： [ quic, uick ]
长度 5（five-gram）： [ quick ]
```

朴素的 `n-gram` 对 词语内部的匹配 非常有用，即在 Ngram 匹配复合词 介绍的那样。但对于输入即搜索（search-as-you-type）这种应用场景，我们会使用一种特殊的 `n-gram` 称为 `边界 n-grams （edge n-grams）`。所谓的`边界 n-gram` 是说它会固定词语开始的一边，以单词 `quick `为例，它的边界 n-gram 的结果为：

```
q
qu
qui
quic
quick
```

##### 索引时输入即搜索

如何使用 `-gram` 可以参考[
索引时输入即搜索](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/_index_time_search_as_you_type.html)

### 处理人类语言

#### 使用语言分析器

可以在字段映射中将语言分析器直接指定在某字段上

```
PUT /my_index
{
  "mappings": {
    "blog": {
      "properties": {
        "title": {
          "type":     "string",
          "analyzer": "english" 
        }
      }
    }
  }
}
```

还可以每一份文档，每一个域一种语言，或者使用混合语言。

#### 词汇识别

需要把合并词拆成词组。

#### 归一化词元

我们还需要去掉有意义的差别, 让 `esta`、`ésta` 和 `está` 都能用同一个词元(token)来搜索。

#### 同义词

同义词扩大了一个匹配文件的范围。正如 词干提取 或者 部分匹配 ，同义词的字段不应该被单独使用，而应该与一个针对主字段的查询操作一起使用，这个主字段应该包含纯净格式的原始文本。 在使用同义词时，参阅 多数字段 的解释来维护相关性。

#### 拼写错误

`Fuzzy matching` 允许查询时匹配错误拼写的单词，而语音语汇单元过滤器可以在索引时用来进行 近似读音 匹配
