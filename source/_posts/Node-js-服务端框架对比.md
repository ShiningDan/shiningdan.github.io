---
title: Node.js 服务端框架对比
date: 2019-11-03 16:03:20
categories:
  - coding
tags:
 - Express
 - Koa
 - Eggjs
 - Nuxt.js
 - Next.js
---

本文是常见的 Nodejs 服务端框架能力对比

<!--more-->


## Express

[Express JS](https://expressjs.com/zh-cn/)

Express 是一个路由和中间件 Web 框架，其自身只具有最低程度的功能

Express 可以通过 `require('express');`，无需初始化脚手架，直接生成一个最简 Server，提供逻辑调用。

```
var express = require('express');
var app = express();

app.get('/', function (req, res) {
  res.send('Hello World!');
});

app.listen(3000, function () {
  console.log('Example app listening on port 3000!');
});
```

也可以使用 Express 的脚手架来生成 Express 应用。

```
.
├── app.js
├── bin
│   └── www
├── package.json
├── public
│   ├── images
│   ├── javascripts
│   └── stylesheets
│       └── style.css
├── routes
│   ├── index.js
│   └── users.js
└── views
    ├── error.pug
    ├── index.pug
    └── layout.pug
```

整体比较灵活，并没有直接约定，在哪个目录下面写什么样的内容。比如静态资源文件夹的引入，要在代码里声明引入路径：

[静态资源中间件的实现方案](https://github.com/expressjs/serve-static)

```
app.use(express.static('public'));
app.use(express.static('files'));
```

### 配置文件

并没有像 Egg 一样默认的配置管理方案

### 路由

没有特定给的路由文件，支持中间件，也支持 RESTFul 形式的路由

### 中间件

connect 曾经是 express 3.x 之前的核心，而 express 4.x 已经把 connect 移除，在 express 中自己实现了 connect 的接口，来实现中间件能力。

Express 应用程序可以使用以下类型的中间件：

* 应用层中间件
* 路由器层中间件
* 错误处理中间件
* 内置中间件
* 第三方中间件

Express 的中间件，除了 static（提供静态资源路径）以外，其他的都被从框架中拆出来了，现有的第三方中间件列表可以在 [第三方中间件](https://expressjs.com/zh-cn/resources/middleware.html) 看到

Express 的中间件执行逻辑是线性的。

```
var express = require('express');

var app = express();
app.use(function (req, res, next) {
    console.log('第一个中间件start');
    setTimeout(() => {
        next();
    }, 1000)
    console.log('第一个中间件end');
});
app.use(function (req, res, next) {
    console.log('第二个中间件start');
    setTimeout(() => {
        next();
    }, 1000)
    console.log('第二个中间件end');
});
app.use('/foo', function (req, res, next) {
    console.log('接口逻辑start');
    next();
    console.log('接口逻辑end');
});
app.listen(4000);
```

输出结果为：

```
第一个中间件start
第一个中间件end
第二个中间件start
第二个中间件end
接口逻辑start
接口逻辑end
```

也就是中间件中如果包含了异步的逻辑，则第一个中间件执行完毕后，才会执行第二个中间件。

而如果中间件中都是同步的逻辑，则执行结果比较类似于 Koa，像剥洋葱一样：

```
var express = require('express');

var app = express();
app.use(function (req, res, next) {
    console.log('第一个中间件start');
    next()
    console.log('第一个中间件end');
});
app.use(function (req, res, next) {
    console.log('第二个中间件start');
    next()
    console.log('第二个中间件end');
});
app.use('/foo', function (req, res, next) {
    console.log('接口逻辑start');
    next();
    console.log('接口逻辑end');
});
app.listen(4000);
```

```
第一个中间件start
第二个中间件start
接口逻辑start
接口逻辑end
第二个中间件end
第一个中间件end
```

也就是，Express 本身对于中间件中异步逻辑的执行，我认为是不如 Koa 支持的好的。

### 错误处理

请在其他 app.use() 和路由调用之后，最后定义错误处理中间件，例如：

```
var bodyParser = require('body-parser');
var methodOverride = require('method-override');

app.use(bodyParser());
app.use(methodOverride());
app.use(function(err, req, res, next) {
  // logic
});
```

### 前置代理模式

[代理背后的 Express](https://expressjs.com/zh-cn/guide/behind-proxies.html)

### 进程管理器

* 在应用程序崩溃后将其重新启动。
* 获得对运行时性能和资源消耗的洞察。
* 动态修改设置以改善性能。
* 控制集群。

用于 Express 和其他 Node.js 应用程序的最流行的进程管理器包括：

* StrongLoop Process Manager
* PM2
* Forever

### 生产环境最佳实践：安全

包括:

1. 使用 TLS
2. 使用 Helmet 适当地设置 HTTP 头，帮助您保护应用程序避免一些众所周知的 Web 漏洞。
3. 安全地使用 cookie：如果有任何理由将其保持安全或隐藏状态，那么 express-session 可能是更好的选择
	1. express-session：中间件将会话数据存储在服务器上；它仅将会话标识（而非会话数据）保存在 cookie 中
	2. cookie-session 中间件实现 cookie 支持的存储：它将整个会话序列化到 cookie 中，而不只是会话键。

[安全](https://expressjs.com/zh-cn/advanced/best-practice-security.html)

### 性能和可靠性

1. 使用 gzip 压缩
2. 不使用同步函数
3. 使用中间件提供静态文件
4. 正确进行日志记录
5. 正确处理异常

### Express 在 js http 上做了什么

源码地址：

## Koa

特点：

1. 由 Express 幕后的原班人马打造，共用的同一套 HTTP 基础库 [https://github.com/jshttp](https://github.com/jshttp)
2. 洋葱模型
3. Context。和 Express 只有 Request 和 Response 两个对象不同，Koa 增加了一个 Context 的对象。类似 traceId 这种需要贯穿整个请求的属性就可以挂载上去。

### 洋葱模型

![](https://camo.githubusercontent.com/d80cf3b511ef4898bcde9a464de491fa15a50d06/68747470733a2f2f7261772e6769746875622e636f6d2f66656e676d6b322f6b6f612d67756964652f6d61737465722f6f6e696f6e2e706e67)

所有的请求经过一个中间件的时候都会执行两次，对比 Express 形式的中间件，Koa 的模型可以非常方便的实现后置处理逻辑。

参考 Compress 中间件在 Expree 和 Koa 的代码实现上：

1. [koa-compress](https://github.com/koajs/compress/blob/master/index.js) for Koa.
2. [compression](https://github.com/expressjs/compression/blob/master/index.js) for Express.

### 能力

### Koa 在 js http 上做了什么

源码地址：

## EggJS

[Eggjs](https://eggjs.org/zh-cn/)

特点：

1. 约定优于配置，避免 Express 标准的 MVC 模型带来的千奇百怪的写法。比如静态资源放在 app/public 下面、不同环境的插件中间件配置在 config/ 目录下面等约定
2. 基于 Koa
3. 服务器环境的 NODE_ENV 应该为 production，而且 npm 也会使用这个变量。Egg 中还另外使用 EGG_SERVER_ENV 精细区分环境

### 能力

从 Koa 继承而来的 4 个对象（Application, Context, Request, Response) 以及框架扩展的一些对象（Controller, Service, Helper, Config, Logger）。扩展的对象都是通过加载器（Loader 进行自定义的）

1. Application：是全局应用对象，在一个应用中，只会实例化一个，挂载一些全局的方法和对象
	1. 事件：框架运行时，会在 Application 实例上触发一些事件，比如： worker 进程启动、异常被 onerror 插件捕获、应用收到请求和响应请求
2. Context：是一个请求级别的对象
3. Request：是一个请求级别的对象
4. Response：是一个请求级别的对象
5. Controller：框架提供了一个 Controller 基类，并推荐所有的 Controller 都继承于该基类实现
6. Service：框架提供了一个 Service 基类，并推荐所有的 Service 都继承于该基类实现
7. Helper：用来提供一些实用的 utility 函数
8. Config：应用开发遵循配置和代码分离的原则
9. Config：框架中提供了多个 Logger 对象
	1. app.logger：应用级别的日志记录，如记录启动阶段的一些数据信息，记录一些业务上与请求无关的信息
	2. app.coreLogger：在开发应用时都不应该通过 CoreLogger 打印日志，而框架和插件则需要通过它来打印应用级别的日志
	3. ctx.logger：打印的日志都会在前面带上一些当前请求相关的信息
	4. ctx.coreLogger：只有插件和框架会通过它来记录日志
10. Subscription：订阅模型是一种比较常见的开发模式，譬如消息中间件的消费者或调度任务

### 中间件

基本上和 Koa 一样，约定一个中间件是一个放置在 app/middleware 目录下的单独文件。

除了应用中间件以外，还支持框架和插件的中间件，和 Router 中使用中间件

框架自带的中间件有好几个，比如 bodyParser。

** 通用配置**：

* enable：控制中间件是否开启。
* match：设置只有符合某些规则的请求才会经过这个中间件。
* ignore：设置符合某些规则的请求不经过这个中间件。

### 路由（Router）

RESTful 风格的 URL 定义：

```
module.exports = app => {
  const { router, controller } = app;
  router.resources('posts', '/api/posts', controller.posts);
  router.resources('users', '/api/v1/users', controller.v1.users); // app/controller/v1/users.js
};
```

### 控制器（Controller）

Controller 负责**解析用户的输入，处理后返回相应的结果**。主要对用户的请求参数进行处理（校验、转换），然后调用对应的 service 方法处理业务，得到业务结果后封装并返回：

1. 获取用户通过 HTTP 传递过来的请求参数。
2. 校验、组装参数。
3. 调用 Service 进行业务处理，必要时处理转换 Service 的返回结果，让它适应用户的需求。
4. 通过 HTTP 将结果响应给用户。

### 服务（Service）

逻辑和展现分离，更容易编写测试用例

### 插件

插件是在中间件上面的补充

许多插件的目的都是将一些已有的服务引入到框架中，如 egg-mysql, egg-oss。他们都需要在 app 上创建对应的实例。

1. 间件加载其实是有先后顺序的，但是中间件自身却无法管理这种顺序，只能交给使用者。这样其实非常不友好，一旦顺序不对，结果可能有天壤之别。
2. 中间件的定位是拦截用户请求，并在它前后做一些事情，例如：鉴权、安全检查、访问日志等等。但实际情况是，有些功能是和请求无关的，例如：定时任务、消息订阅、后台逻辑等等。
3. 有些功能包含非常复杂的初始化逻辑，需要在应用启动的时候完成。这显然也不适合放到中间件中去实现。

插件其实就是一个『迷你的应用』，和应用（app）几乎一样：

1. 它包含了 Service、中间件、配置、框架扩展等等。
2. 它没有独立的 Router 和 Controller。
3. 它没有 plugin.js，只能声明跟其他插件的依赖，而不能决定其他插件的开启与否。

常见的有：模板渲染， onerror 插件，安全插件 egg-security，国际化插件 egg-i18n 插件


### 启动自定义

框架提供了统一的入口文件（app.js）进行启动过程自定义。框架提供了这些 生命周期函数供开发人员处理

### 部署

框架内置了 egg-cluster 来启动 Master 进程，Master 有足够的稳定性，不再需要使用 pm2 等进程守护模块。

### 多进程模型和进程间通讯

单个 Node.js 实例在单线程环境下运行。为了更好地利用多核环境，用户有时希望启动一批 Node.js 进程用于加载。

集群化模块使得你很方便地创建子进程，以便于在服务端口之间共享。

Worker 进程的数量一般根据服务器的 CPU 核数来定，这样就可以完美利用多核资源。

[Cluster 实现原理](https://cnodejs.org/topic/56e84480833b7c8a0492e20c)

分为 Master，Worker，Agent：

* Master：负责启动其他进程的叫做 Master 进程
* Worker：其他被启动的叫 Worker 进程
* Agent：后台运行的逻辑，我们希望将它们放到一个单独的进程上去执行，比如：每天凌晨 0 点，将当前日志文件按照日期进行重命名，或者销毁以前的文件句柄，并创建新的日志文件继续写入

可以在应用或插件根目录下的 agent.js 中实现你自己的逻辑

在 Egg 中，多进程的通信还可以直接由 Worker 连接 Agent，而不需要 Master 的中转，可以参考 [多进程研发模式增强](https://eggjs.org/zh-cn/advanced/cluster-client.html)

### 异常处理

框架通过 onerror 插件提供了统一的错误处理机制。对一个请求的所有处理方法（Middleware、Controller、Service）中抛出的任何异常都会被它捕获

### RESTful

RESTful 风格的设计中，我们会通过响应状态码来标识响应的状态，保持响应的 body 简洁，只返回接口数据。

### 前置代理模式

应用就默认自己处于反向代理之后，会支持通过解析约定的请求头来获取用户真实的 IP，协议和域名。

### 加载器（Loader）

[应用、插件和框架三者之间的关系](https://eggjs.org/zh-cn/advanced/loader.html)

1. 我们在应用中完成业务，需要指定一个框架才能运行起来，当需要某个特性场景的功能时可以配置插件（比如 MySQL）。
2. 插件只完成特定功能，当两个独立的功能有互相依赖时，还是分开两个插件，但需要配置依赖。
3. 框架是一个启动器（默认就是 Egg），必须有它才能运行起来。框架还是一个封装器，将插件的功能聚合起来统一提供，框架也可以配置插件。
4. 在框架的基础上还可以扩展出新的框架，也就是说框架是可以无限级继承的，有点像类的继承。

### 框架开发

框架是一层抽象，可以基于 Egg 去封装上层框架，并且 Egg 支持多层继承。

比如有三层框架：部门框架（department）> 企业框架（enterprise）> Egg


### Egg 在 Koa 上做了什么

源码地址：

## Nuxt.js & Next.js

Next.js 是一个轻量级的React 服务端渲染应用框架

Nuxt.js 是 Vue.js 的服务端渲染框架，预设了利用 Vue.js 开发服务端渲染的应用所需要的各种**配置**。

我们可以理解为，Nuxt.js 提供的是一个开发框架，基于服务器框架 Koa/Express 等，前端模板基于 Vue.js，并且预制了很多功能，成为了一键生成的开发套件。

当然，Nuxt.js 也支持静态托管的能力。

### 特点

1. 依据 pages/ 目录结构自动生成 vue-router 模块的路由配置


## 其他

**一个完整的线上工程，应该包含哪些能力：**