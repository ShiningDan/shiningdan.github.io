---
title: 开始使用Docker
date: 2017-05-31 16:12:36
categories: coding
tags:
  - Docker
---

Docker 是一个开源的应用容器引擎，由于我经常折腾自己的博客系统，所以最后使用 Docker 来部署和运行我的线上服务，这样在出问题的时候可以很快捷的重新快速部署业务，方便我的管理。

<!--more-->

## 环境配置

为了运行需要的镜像，我在虚拟机环境中预先装好了 Java 和 Node 和 pm2。

当然，不能忘记安装 [Docker](https://docs.docker.com/engine/installation/linux/ubuntu/#install-using-the-repository) 。由于我们的服务中涉及到多个容器的组合，所以，我们要使用 [Docker compose](https://docs.docker.com/compose/overview/) 来管理多个容器。

在 Docker 安装的时候需要使用 add-apt-repository，但是在 Ubuntu 16.04 中没有这个 package，所以需要使用其他的 package 来替代：

```
apt-get install software-properties-common
```

首先 `git clone` 我的项目的代码，然后使用 `npm install` 来安装需要的 npm 依赖

`docker-compose.yml` 文件内容如下，它定义了每个容器基于什么镜像运行，映射哪些目录，开放哪些端口：

### 添加 Elasticsearch

Elasticsearch 官方提供了一个使用 Docker 安装 Elasticsearch 的说明：[
Install Elasticsearch with Docker](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)

我们在 `docker-compose.yml` 中添加启动 Elasticsearch 的代码：

```
version: '2'
services:
  es:
    image: docker.elastic.co/elasticsearch/elasticsearch:5.4.0
    container_name: yuchen_es
    environment:
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "xpack.monitoring.enabled=false"
      - "xpack.security.enabled=false"
      - "Dlog4j2.disable.jmx=false"
      - "cluster.name=yuchen_es"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    mem_limit: 1g
    volumes:
      - esdata:/usr/share/elasticsearch/data
      - esconfig:/usr/share/elasticsearch/config
      - esplugins:/usr/share/elasticsearch/plugins
    ports:
      - 9200:9200
    restart: always
volumes:
  esdata:
    driver: local
  esconfig:
    driver: local
  esplugins:
    driver: local
```

由于是生产环境，我们按照官方文档要求设置：

```
sysctl -w vm.max_map_count=262144   // 设置进程中内存映射区域的最大数量。
```



## 参考

* [开始使用 Docker](https://imququ.com/post/use-docker.html)
* [Docker 官方网站](https://www.docker.com/community-edition)


