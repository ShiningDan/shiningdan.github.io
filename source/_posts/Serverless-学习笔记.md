---
title: Serverless 学习笔记
date: 2018-01-23 16:43:32
categories: coding
tags:
  - serverless
  - 云计算
---

本文是我记录 Serverless 学习的笔记

<!--more-->

本文所记录的，是使用阿里云函数计算相关的服务的总结笔记，源文档可以在 [函数计算 | 阿里云](https://help.aliyun.com/product/50980.html) 上进行阅读。

## 介绍

函数计算是一个事件驱动的全托管计算服务。通过函数计算，您无需管理服务器等基础设施，只需编写代码并上传。函数计算会为您准备好计算资源，以弹性、可靠的方式运行您的代码，并提供日志查询，性能监控，报警等功能。更棒的是，您只需要为代码实际运行消耗的资源付费 - 代码未运行则不产生费用。

函数计算以事件驱动的方式连接不同的服务。当事件源服务触发事件时，系统会自动调用关联的函数处理事件。例如阿里云对象存储服务（OSS）中有新数据上传时，自动调用函数进行处理；以及使用 API 网关触发函数以响应 HTTP 请求。当然，您也可以使用函数计算 SDK/API 调用您的代码。

## 计费模型

函数计算的另一个好处是按需付费，只有在函数触发的时候才需要进行付费，阿里云的[费用的结算方法](https://help.aliyun.com/document_detail/54301.html)，

总费用 = 调用次数费用 + 执行时间费用 + 公网流量费用（可选）

## 基本概念

在使用函数计算的时候，有以下的几个概念需要知道：

* 服务（Service）：服务是资源管理的基本单位。您可以在服务上执行授权、配置日志、创建函数等操作。服务下的所有函数都共享这些设置。一个服务下能创建的函数是有限制的，请参阅函数计算限制项
* 函数（Function）：函数是用户编写的，由事件触发，执行特定功能的一段代码。函数是调度和运行的基本单位。
* 事件（Event）：任何能够触发函数执行的事情称之为事件。例如，一个调用函数的http请求，可以看做一个事件。上传对象到特定的OSS bucket并触发函数调用，也是一个事件
* 触发器（Trigger）：用户通过触发器定义和管理事件的生成方式。例如，当您创建一个OSS PutObject触发器后，当put object到指定的OSS位置时，就会产生一个事件，触发对应的函数

还有一些我们需要知道的：

如果只是函数计算本身，其能做的事情还是有限，我们在搭建服务的时候，需要将函数计算和一些其他常用的服务结合在一起，组成一个完整的应用进行交付。这里有一些常见的概念：

* 角色：角色的策略主要是授权给Service能去访问（read、delete、write等）其他云产品的权限。角色配置在服务层，该服务下的所有函数都将具有这个角色的权限
* API 网关：API 网关（API Gateway）提供高性能、高可用的 API 托管服务，帮助用户对外开放其部署在 ECS、容器服务等云产品上的应用，提供完整的 API 发布、管理、维护生命周期管理。
* 日志服务：在函数计算中，函数执行的日志可以存储在日志服务中，用户可以根据日志服务中存储对的函数执行的日志进行一些故障分析，数据分析等相关。
* OSS（对象存储）:对象存储服务（Object Storage Service，简称 OSS），可以通过调用 API，在任何应用、任何时间、任何地点上传和下载数据，也可以通过 Web 控制台对数据进行简单的管理。

## 编程指南

由于我目前使用的是阿里云的函数计算服务，Node.js 环境，所以这里记录的主要是 Node.js 中，我认为比较重要或者特殊的要点。

其他环境下的编程指南可以在[阿里云官网上查看](https://help.aliyun.com/document_detail/52699.html)

### 基本概念

#### Context 对象

在执行函数时，您可以通过 context 对象与函数计算系统交互，获取有用的**运行时信息**。

#### 错误类型

需要注意的，当用户捕获错误并通过 callback(err) 返回时，如果参数 err 是 Error 类型的对象，调用栈信息（stack trace）也会被返回。

```
exports.handler = function(event, context, callback) {
    var error = new Error("something is wrong");
    callback(error);
};
// Function response.
{
   "errorMessage": "something is wrong",
   "errorType": "Error",
   "stackTrace": [
       "export.handler (/var/task/index.js.3:16)"
   ]
}
```

对于Nodejs运行环境的函数，用户可能收到两种错误，错误类型记录在返回的HTTP Header字段中(X-Fc-Error-Type)：

* HandledInvocationError: 通过callback的第一个参数返回的错误
* UnhandledInvocationError: 其他错误，包括未接住的异常/超时/OOM等


#### 日志记录

日志服务也是有一定的免费额度的，使用日志记录可以更加方便我们的线上调试。具体如何使用日志，可以参考 [函数访问日志服务](https://help.aliyun.com/document_detail/61023.html)

#### 事件处理函数

我们对外提供给函数计算的函数，需要有三个参数，满足以下的形式：

```
exports.myHandler = function(event, context, callback) {
   ...
   // Use callback() to return information to the caller.
}
```

* event：事件数据，类型为 Buffer 对象。
* context：当前函数的运行时信息，类型为 Context 对象。
* callback：通知函数计算系统函数执行完毕，并将信息返回给调用方。

注意：

```
您必须主动调用callback函数，通知函数计算系统函数执行完毕；否则函数的执行不会退出，最后会导致超时错误。

当您使用callback error参数返回错误信息时，函数计算系统会自动将 error 信息记录到与该函数关联的 SLS 日志库中。如果error信息大于256KB，则系统只会记录前256KB的数据，并在错误消息后显示 “Truncated by FunctionCompute”。关于如何设置函数日志，请参阅相关文档。

如果函数的调用类型是同步调用，错误消息或者结果数据会填充到http body中，并返回给客户端。如果是异步调用，则错误信息或者结果数据不会被返回。
```

### Node.js 环境

函数计算目前支持以下Nodejs运行环境：

* Nodejs6.10 (runtime = nodejs6)

除了Nodejs的标准模块，函数计算的Nodejs运行环境中还包含了一些常用模块，用户可以直接引用，目前包含的模块有：

* co: 控制流，https://github.com/tj/co
* gm: 图片处理库，https://github.com/aheckmann/gm
* ali-oss: OSS SDK, https://github.com/ali-sdk/ali-oss
* aliyun-sdk: Aliyun SDK, https://github.com/aliyun-UED/aliyun-sdk-js
* @alicloud/fc2: 函数计算SDK，https://github.com/aliyun/fc-nodejs-sdk
* opencv: 视觉算法库，https://github.com/peterbraden/node-opencv

如果用户需要使用自定义的模块，则需要将它们与代码一起打包。安装第三方库以及打包的例子可以参考 [应用示例3 - 图像处理](https://help.aliyun.com/document_detail/52586.html) 

#### 使用代码目录

如果用户将一些配置文件或者数据文件与代码一起打包上传，并且需要在代码中访问这些文件的话，就需要使用 `FC_FUNC_CODE_PATH` 这个环境变量来获取文件的绝对路径

```
exports.handler = function(event, context, callback) {
  cfgFile = process.env['FC_FUNC_CODE_PATH'] + '/config.json';
  console.log(cfgFile);
  callback(null, 'done');
}
```

#### 使用内网域名

在函数中访问其他云服务，建议使用内网域名，一方面使用内网域名能够有更好的性能；另一方面可以避免公网流量收费。

```
var OSSClient = require('ali-oss').Wrapper;
exports.handler = function(event, context, callback) {
  console.log(event.toString());
  var ossClient = new OSSClient({
    accessKeyId: context.credentials.accessKeyId,
    accessKeySecret: context.credentials.accessKeySecret,
    stsToken: context.credentials.securityToken,
    region: 'oss-cn-shanghai',
    internal: true,
    bucket: 'my-bucket',
  });
  ossClient.put('my-object', new Buffer('hello world'))
    .then(function(res) {
      callback(null, res);
    }).catch(function(err) {
      callback(err);
    });
};
```

### 开发工具

#### fcli

fcli 是一个强大的命令行工具，能帮助您便捷的管理函数计算中的资源。

使用方法可以参考：[fcli | 阿里云](https://help.aliyun.com/document_detail/52995.html)

#### fun

fun 是一个用于支持函数计算和 API 网关的工具，能帮助您便捷地管理函数计算和 API 网关的资源。它通过一个资源配置文件（faas.yaml），协助您进行开发、构建、部署操作。

针对 Node.js 用户，它能自动安装打包依赖模块，使您可以继续使用 package.json 的方式进行依赖管理。

使用方法可以参考：[fun | 阿里云](https://help.aliyun.com/document_detail/64204.html)