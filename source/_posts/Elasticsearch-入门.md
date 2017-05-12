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

