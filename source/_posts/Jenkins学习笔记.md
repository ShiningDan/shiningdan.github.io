---
title: Jenkins学习笔记
date: 2018-04-21 17:38:02
categories: coding
tags:
  - Jenkins
  - Docker
---

本文是我学习 Jenkins 的笔记

<!--more-->

## Jenkins 基本介绍

Jenkins 是一个可扩展的持续集成引擎。

主要用于：

* 持续、自动地构建/测试软件项目。
* 监控一些定时执行的任务。

Jenkins拥有的特性包括：

* 易于安装-只要把jenkins.war部署到servlet容器，不需要数据库支持。
* 易于配置-所有配置都是通过其提供的web界面实现。
* 集成RSS/E-mail通过RSS发布构建结果或当构建完成时通过e-mail通知。
* 生成JUnit/TestNG测试报告。
* 分布式构建支持Jenkins能够让多台计算机一起构建/测试。
* 文件识别:Jenkins能够跟踪哪次构建生成哪些jar，哪次构建使用哪个版本的jar等。
* 插件支持:支持扩展插件，你可以开发适合自己团队使用的工具。

### Jenkins Pipeline

Pipeline，简单来说，就是一套运行于Jenkins上的工作流框架，将原本独立运行于单个或者多个节点的任务连接起来，实现单个任务难以完成的复杂发布流程。

Jenkins Pipeline 的描述是用 Groovy DSL 写的 Jenkinsfile 文件。

### 定义持续构建的运行环境

Jenkins 推荐使用包含运行环境的 Docker 镜像。

### 自定义 Jenkinsfile

Jenkinsfile 可以指定构建的环境，构建的步骤，构建中需要的环境变量

### 记录测试的结果

Jenkins 支持将测试结果作为文件导出。也支持选择性保存构建后生成的文件。

### 持续构建后的通知

Jenkins 支持在一次构建完成后，对不同的构建结果，如 `success`，`failure` 等配置不同的通知。

### 部署

一个完整的持续交付的步骤，至少需要包含：构建，测试，以及部署。Jenkins 也支持使用 stage 创建部署的步骤。

### 支持用户输入

在部署的过程中，可能会产生需要用户验证等与人交互的行为，Jenkins 支持在 stage 中通过 `input` 命令创建等待用户输入的步骤。

### Jenkins Blue Ocean

新的 UI 插件

## 使用 Docker 安装单节点 Jenkins


Jenkins 的下载 [Jenkins | Docker Hub](https://hub.docker.com/_/jenkins/)

Jenkins 单节点的配置，需要指定对应的端口映射和数据卷：

```
docker run --name myjenkins -p 8080:8080 -p 50000:50000 -v /Users/yuchenzhang/Project:/var/jenkins_home jenkins
```

之后，在运行 Jenkins 运行的命令行，会提示 Jenkins 第一次登录的管理员密码。

在打开 Jenkins 后，会提示是否安装插件，在建议中提供的插件有：

![](http://ojt6zsxg2.bkt.clouddn.com/17a359c193ddb11b34e3baad4d50e89a.png)

在选择需要的插件之后，Jenkins 会提示创建第一个 Admin 用户，一定要将创建第一个 Admin 用户的所有信息都填写完成，否则在点击创建用户的时候，页面会卡住。。。坑。

### 创建一个 Node.js 持续构建的项目

在官网上，有一个 [Node.js 和 React 的持续集成项目示例](https://jenkins.io/doc/tutorials/build-a-node-js-and-react-app-with-npm/)

启动 Jenkins 镜像

```
docker run --name myjenkins -p 8080:8080 -p 50000:50000 -v /Users/yuchenzhang/Docker/Jenkins:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -v "$HOME":/home -v /usr/local/bin:/user/bin jenkins
```



