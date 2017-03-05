---
title: Node.js入门
date: 2017-03-05 19:27:42
categories: coding
tags:
  - Node.js
---

本笔记记录着学习 [Node 入门](http://www.nodebeginner.org/index-zh-cn.html) 这本书的练习步骤以及一些总结

## 实现功能

* 用户可以通过浏览器使用我们的应用。
* 当用户请求`http://domain/start`时，可以看到一个欢迎页面，页面上有一个文件上传的表单。
* 用户可以选择一个图片并提交表单，随后文件将被上传到`http://domain/upload`，该页面完成上传后会把图片显示在页面上。

## 模块分析

为了实现上文的用例，我们需要实现哪些部分

* 我们需要提供Web页面，因此需要一个HTTP服务器
* 对于不同的请求，根据请求的URL，我们的服务器需要给予不同的响应，因此我们需要一个`路由`，用于把请求对应到请求处理程序（request handler）
* 当请求被服务器接收并通过路由传递之后，需要可以对其进行处理，因此我们需要最终的请求处理程序
* 路由还应该能处理POST数据，并且把数据封装成更友好的格式传递给请求处理入程序，因此需要请求数据处理功能
* 我们不仅仅要处理URL对应的请求，还要把内容显示出来，这意味着我们需要一些视图逻辑供请求处理程序使用，以便将内容发送给用户的浏览器
* 最后，用户需要上传图片，所以我们需要上传处理功能来处理这方面的细节

## 构建应用的模块

### 一个基础的HTTP服务器

```
var http = require("http");

let server = http.createServer(function(request, response) {
	response.writeHead(200, {"Content-type": "text/plain"});
	response.write("Hello World!");
	response.end();
});

server.listen(8888);
```

第一行请求（`require`）Node.js自带的 `http` 模块，并且把它赋值给 `http` 变量。

接下来我们调用`http`模块提供的函数： `createServer` 。这个函数会返回一个对象，这个对象有一个叫做 `listen` 的方法，这个方法有一个数值参数，指定这个HTTP服务器监听的端口号。

`function(request, response)`这个函数定义是 `createServer()` 的第一个也是唯一一个参数。因为在JavaScript中，函数和其他变量一样都是可以被传递的。

当我们使用 `http.createServer` 方法的时候，我们当然不只是想要一个侦听某个端口的服务器，我们还想要它在服务器收到一个HTTP请求的时候做点什么。

问题是，这是异步的：请求任何时候都可能到达，但是我们的服务器却跑在一个单进程中。

写PHP应用的时候，我们一点也不为此担心：任何时候当有请求进入的时候，网页服务器（通常是Apache）就为这一请求新建一个进程，并且开始从头到尾执行相应的PHP脚本。

那么在我们的Node.js程序中，当一个新的请求到达8888端口的时候，我们怎么控制流程呢？

我们创建了服务器，并且向创建它的方法传递了一个函数。无论何时我们的服务器收到一个请求，这个函数就会被调用。

这个就是传说中的 回调 。我们给某个方法传递了一个函数，这个方法在有相应事件发生时调用这个函数来进行 回调 。

### 服务器是如何处理请求的

当回调启动，我们的回调函数被触发的时候，有两个参数被传入： `request` 和 `response` 。

它们是对象，你可以使用它们的方法来处理HTTP请求的细节，并且响应请求（比如向发出请求的浏览器发回一些东西）。

所以我们的代码就是：当收到请求时，使用 `response.writeHead() `函数发送一个HTTP状态`200`和HTTP头的内容类型（`content-type`），使用 `response.write()` 函数在HTTP相应主体中发送文本“Hello World"。

最后，我们调用 `response.end()` 完成响应。

### 服务端的模块放在哪里

我们现在在 server.js 文件中有一个非常基础的HTTP服务器代码，而且我提到通常我们会有一个叫 index.js 的文件去调用应用的其他模块（比如 server.js 中的HTTP服务器模块）来引导和启动应用。

事实上，我们不用做太多的修改。把某段代码变成模块意味着我们需要把我们希望提供其功能的部分 导出 到请求这个模块的脚本。

我们把我们的服务器脚本放到一个叫做 `start` 的函数里，然后我们会导出这个函数。

```
var http = require("http");

function start() {
	function onRequest(request, response) {
		console.log('request received.');
		response.writeHead(200, {"Content-type": "text/plain"});
		response.write("Hello World!");
		response.end();
	}
	http.createServer(onRequest).listen(8888);
	console.log('server start listen.');
}

exports.start = start;
```

我们现在就可以创建我们的主文件 index.js 并在其中启动我们的HTTP了，虽然服务器的代码还在 server.js 中。

创建 index.js 文件并写入以下内容：

```
let server = require("./server.js");

server.start();
```

我们现在就可以从我们的主要脚本启动我们的的应用了，而它还是老样子：

```
node index.js
```

## 如何来进行请求的“路由”

我们要为路由提供请求的URL和其他需要的GET及POST参数，随后路由需要根据这些数据来执行相应的代码（这里“代码”对应整个应用的第三部分：一系列在接收到请求时真正工作的处理程序）。

我们需要的所有数据都会包含在`request`对象中，该对象作为`onRequest()`回调函数的第一个参数传递。但是为了解析这些数据，我们需要额外的Node.JS模块，它们分别是`url`和`querystring`模块。

```
                               url.parse(string).query
                                           |
           url.parse(string).pathname      |
                       |                   |
                       |                   |
                     ------ -------------------
http://localhost:8888/start?foo=bar&hello=world
                                ---       -----
                                 |          |
                                 |          |
              querystring(string)["foo"]    |
                                            |
                         querystring(string)["hello"]
```

现在我们来给`onRequest()`函数加上一些逻辑，用来找出浏览器请求的URL路径：











































































































