---
title: Egg入门学习
date: 2018-04-21 17:27:32
categories: coding
tags:
  - Egg.js
  - JavaScript
  - Node.js
---

本文是我学习 Egg.js 的笔记

<!--more-->

## 目录结构

```
egg-project
├── package.json
├── app.js (初始化工作)
├── agent.js (多进程)
├── app
|   ├── router.js
│   ├── controller
│   |   └── home.js
│   ├── service (业务逻辑)
│   |   └── user.js
│   ├── middleware (中间件)
│   |   └── response_time.js
│   ├── schedule (定时任务)
│   |   └── my_task.js
│   ├── public (静态资源)
│   |   └── reset.css
│   ├── view (模板文件)
│   |   └── home.tpl
│   └── extend (框架的扩展)
│       ├── helper.js (可选)
│       ├── request.js (可选)
│       ├── response.js (可选)
│       ├── context.js (可选)
│       ├── application.js (可选)
│       └── agent.js (可选)
├── config（配置文件）
|   ├── plugin.js
|   ├── config.default.js
│   ├── config.prod.js
|   ├── config.test.js (可选)
|   ├── config.local.js (可选)
|   └── config.unittest.js (可选)
└── test
    ├── middleware
    |   └── response_time.test.js
    └── controller
        └── home.test.js
```

## 框架内置基础对象

包括从 Koa 继承而来的 4 个对象（Application, Context, Request, Response) 以及框架扩展的一些对象（Controller, Service, Helper, Config, Logger）

### Application

挂载一些全局的方法和对象。

#### 事件

```
// app.js

module.exports = app => {
  app.once('server', server => {
    // websocket
    // 该事件一个 worker 进程只会触发一次，在 HTTP 服务完成启动后，会将 HTTP server 通过这个事件暴露出来给开发者。
  });
  app.on('error', (err, ctx) => {
    // report error
    // 运行时有任何的异常被 onerror 插件捕获后，都会触发 error 事件
  });
  app.on('request', ctx => {
    // log receive request
    // 应用收到请求和响应请求时
  });
  app.on('response', ctx => {
    // ctx.starttime is set by framework
    const used = Date.now() - ctx.starttime;
    // log total cost
    // 应用收到请求和响应请求时
  });
};
```

#### 获取方式

在继承于 Controller, Service 基类的实例中，可以通过 this.app 访问到 Application 对象。

启动自定义脚本

```
// app.js
module.exports = app => {
  app.cache = new Cache();
};
```

### Context

Context 是一个请求级别的对象

框架会将所有的 Service 挂载到 Context 实例上，一些插件也会将一些其他的方法和对象挂载到它上面

最常见的 Context 实例获取方式是在 Middleware, Controller 以及 Service 中。

### Request & Response

可以在 Context 的实例上获取到当前请求的 Request(ctx.request) 和 Response(ctx.response) 实例。

```
// app/controller/user.js
class UserController extends Controller {
  async fetch() {
    const { app, ctx } = this;
    const id = ctx.request.query.id;
    ctx.response.body = app.cache.get(id);
  }
}
```

### Controller

框架提供了一个 Controller 基类，并推荐所有的 Controller 都继承于该基类实现。这个 Controller 基类有下列属性：

* `ctx` - 当前请求的 Context 实例。
* `app` - 应用的 Application 实例。
* `config` - 应用的配置。
* `service` - 应用所有的 service。
* `logger` - 为当前 controller 封装的 logger 对象。

### Helper

Helper 用来提供一些实用的 utility 函数。它的作用在于我们可以将一些常用的动作抽离在 helper.js 里面成为一个独立的函数

可以在 Context 的实例上获取到当前请求的 Helper(ctx.helper) 实例。

#### 自定义 helper 方法

我们可以通过框架扩展的形式来自定义 helper 方法。

```
// app/extend/helper.js
module.exports = {
  formatUser(user) {
    return only(user, [ 'name', 'phone' ]);
  }
};
```

### Logger

5 个级别的方法：

```
logger.debug()
logger.info()
logger.warn()
logger.error()
```

#### 获取方式和使用场景。

##### App Logger

我们可以通过 app.logger 来获取到它，如果我们想做一些应用级别的日志记录，如记录启动阶段的一些数据信息，记录一些业务上与请求无关的信息

##### Context Logger

我们可以通过 ctx.logger 从 Context 实例上获取到它，从访问方式上我们可以看出来，Context Logger 一定是与请求相关的，它打印的日志都会在前面带上一些当前请求相关的信息（如 [`$userId/$ip/$traceId/${cost}ms $method $url`]）

### Subscription

订阅模型是一种比较常见的开发模式，譬如消息中间件的消费者或调度任务。因此我们提供了 Subscription 基类来规范化这个模式。

可以通过以下方式来引用 Subscription 基类：

```
const Subscription = require('egg').Subscription;

class Schedule extends Subscription {
  // 需要实现此方法
  // subscribe 可以为 async function 或 generator function
  async subscribe() {}
}
```

## 运行环境

### 指定运行环境

框架有两种方式指定运行环境：

1. 通过 `config/env` 文件指定，该文件的内容就是运行环境，如 `prod`。一般通过构建工具来生成这个文件。
2. 通过 `EGG_SERVER_ENV` 环境变量指定。

### 应用内获取运行环境

框架提供了变量 `app.config.env` 来表示应用当前的运行环境。

### 运行环境相关配置

不同的运行环境会对应不同的配置

服务器环境的 `NODE_ENV` 应该为 `production`。而且 npm 也会使用这个变量，在应用部署的时候一般不会安装 `devDependencies`，所以这个值也应该为 `production`。

框架默认支持的运行环境及映射关系（如果未指定 EGG_SERVER_ENV 会根据 NODE_ENV 来匹配）

```
NODE_ENV	EGG_SERVER_ENV	说明
test	    unittest	单元测试
production	prod	生产环境
```

## Config 配置

### 多环境配置

```
config
|- config.default.js
|- config.test.js
|- config.prod.js
|- config.unittest.js
|- config.local.js
```

`config.default.js` 为默认的配置文件，**所有环境都会加载这个配置文件**

### 配置加载顺序

```
比如在 prod 环境加载一个配置的加载顺序如下，后加载的会覆盖前面的同名配置。

-> 插件 config.default.js
-> 框架 config.default.js
-> 应用 config.default.js
-> 插件 config.prod.js
-> 框架 config.prod.js
-> 应用 config.prod.js
```

### Middleware 中间件


