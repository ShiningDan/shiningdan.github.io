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

<!--more-->

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

其中，一些相关的对象为：

`request.url`：类型为 string，其中的值为：`/start?foo=bar&hello=world`

经过 `url.parse(request.url)` 后生成一个 Object 类型的对象。其中的内容为：

```
Url {
  protocol: null,
  slashes: null,
  auth: null,
  host: null,
  port: null,
  hostname: null,
  hash: null,
  search: '?foo=bar&hello=world',
  query: 'foo=bar&hello=world',
  pathname: '/start',
  path: '/start?foo=bar&hello=world',
  href: '/start?foo=bar&hello=world' }
```

解析的代码为：

```
let http = require("http");
let url = require("url");

function start() {
	function onRequest(request, response) {
		console.log('request received.');
		let pathname = url.parse(request.url).pathname;
		console.log('request for ' + pathname + ' received.');
		response.writeHead(200, {"Content-type": "text/plain"});
		response.write("Hello World!");
		response.end();
	}
	http.createServer(onRequest).listen(8888);
	console.log('server start listen.');
}

exports.start = start;
```

好了，我们的应用现在可以通过请求的URL路径来区别不同请求了--这使我们得以使用路由（还未完成）来将请求以URL路径为基准映射到处理程序上。

在我们所要构建的应用中，这意味着来自`/start`和`/upload`的请求可以使用不同的代码来处理。稍后我们将看到这些内容是如何整合到一起的。

现在我们可以来编写路由了，建立一个名为`router.js`的文件，添加以下内容：

```
function route(pathname) {
	console.log('About to route a request for ' + pathname);
}

exports.route = route;
```

我们来扩展一下服务器的start()函数，以便将路由函数作为参数传递过去：

```
let http = require("http");
let url = require("url");

function start(route) {
	function onRequest(request, response) {
		console.log('request received.');
		let pathname = url.parse(request.url).pathname;
		console.log('request for ' + pathname + ' received.');

		route(pathname);

		response.writeHead(200, {"Content-type": "text/plain"});
		response.write("Hello World!");
		response.end();
	}
	http.createServer(onRequest).listen(8888);
	console.log('server start listen.');
}

exports.start = start;
```

同时，我们会相应扩展index.js，使得路由函数可以被注入到服务器中：

```
let server = require("./server.js");
let router = require("./router.js")

server.start(router.route);
```

### 路由给真正的请求处理程序

当然这还远远不够，路由，顾名思义，是指我们要针对不同的URL有不同的处理方式。例如处理`/start`的“业务逻辑”就应该和处理`/upload`的不同。

应用程序需要新的部件，因此加入新的模块 -- 已经无需为此感到新奇了。我们来创建一个叫做requestHandlers的模块，并对于每一个请求处理程序，添加一个占位用函数，随后将这些函数作为模块的方法导出：

```
function start() {
	console.log('Reques handler "start" was called');
}

function upload() {
	console.log('Request handler "upload" was called');
}

exports.start = start;
exports.upload = upload;
```

在这里我们得做个决定：是将`requestHandlers`模块硬编码到路由里来使用，还是再添加一点依赖注入？虽然和其他模式一样，依赖注入不应该仅仅为使用而使用，但在现在这个情况下，使用依赖注入可以让路由和请求处理程序之间的耦合更加松散，也因此能让路由的重用性更高。

那么我们要怎么传递这些请求处理程序呢？别看现在我们只有2个处理程序，在一个真实的应用中，请求处理程序的数量会不断增加，我们当然不想每次有一个新的URL或请求处理程序时，都要为了在路由里完成请求到处理程序的映射而反复折腾。除此之外，在路由里有一大堆`if request == x then call handler y`也使得系统丑陋不堪。

我们已经确定将一系列请求处理程序通过一个对象来传递，并且需要使用松耦合的方式将这个对象注入到`route()`函数中。

我们先将这个对象引入到主文件`index.js`中：

```
let server = require("./server.js");
let router = require("./router.js");
let requestHandlers = require("./requestHandlers.js");

let handle = {};
handle["/"] = requestHandlers.start;
handle["/start"] = requestHandlers.start;
handle["/upload"] = requestHandlers.upload;
server.start(router.route, handle);
```

在完成了对象的定义后，我们把它作为额外的参数传递给服务器，为此将server.js修改如下：

```
let http = require("http");
let url = require("url");

function start(route, handle) {
	function onRequest(request, response) {
		console.log('request received.');
		let pathname = url.parse(request.url).pathname;
		console.log('request for ' + pathname + ' received.');

		route(handle, pathname);

		response.writeHead(200, {"Content-type": "text/plain"});
		response.write("Hello World!");
		response.end();
	}
	http.createServer(onRequest).listen(8888);
	console.log('server start listen.');
}

exports.start = start;
```

然后我们相应地在route.js文件中修改`route()`函数：

```
function route(handle, pathname) {
	console.log('About to route a request for ' + pathname);
	if (typeof handle[pathname] === 'function') {
		handle[pathname]();
	} else {
		console.log('No request handler found for ' + pathname);
	}
}

exports.route = route;
```

## 让请求处理程序作出响应

浏览器发出请求后获得并显示的“Hello World”信息仍是来自于我们server.js文件中的`onRequest`函数。

其实“处理请求”说白了就是“对请求作出响应”，因此，我们需要让请求处理程序能够像`onRequest`函数那样可以和浏览器进行“对话”。

### 不好的实现方式

让请求处理程序通过`onRequest`函数直接返回`（return()）`他们要展示给用户的信息。

让我们从让请求处理程序返回需要在浏览器中显示的信息开始。我们需要将requestHandler.js修改为如下形式：

```
function start() {
  console.log("Request handler 'start' was called.");
  return "Hello Start";
}

function upload() {
  console.log("Request handler 'upload' was called.");
  return "Hello Upload";
}

exports.start = start;
exports.upload = upload;
```

好的。同样的，请求路由需要将请求处理程序返回给它的信息返回给服务器。因此，我们需要将router.js修改为如下形式：

```
function route(handle, pathname) {
  console.log("About to route a request for " + pathname);
  if (typeof handle[pathname] === 'function') {
    return handle[pathname]();
  } else {
    console.log("No request handler found for " + pathname);
    return "404 Not found";
  }
}

exports.route = route;
```

最后，我们需要对我们的server.js进行重构以使得它能够将请求处理程序通过请求路由返回的内容响应给浏览器，如下所示：

```
var http = require("http");
var url = require("url");

function start(route, handle) {
  function onRequest(request, response) {
    var pathname = url.parse(request.url).pathname;
    console.log("Request for " + pathname + " received.");

    response.writeHead(200, {"Content-Type": "text/plain"});
    var content = route(handle, pathname)
    response.write(content);
    response.end();
  }

  http.createServer(onRequest).listen(8888);
  console.log("Server has started.");
}

exports.start = start;
```

如果我们运行重构后的应用，一切都会工作的很好：

请求http://localhost:8888/start,浏览器会输出“Hello Start”，
请求http://localhost:8888/upload会输出“Hello Upload”,
请求http://localhost:8888/foo 会输出“404 Not found”。

好，那么问题在哪里呢？简单的说就是： 当未来有请求处理程序需要进行非阻塞的操作的时候，我们的应用就“挂”了。

### 阻塞与非阻塞

我们来修改下start请求处理程序，我们让它等待10秒以后再返回“Hello Start”。

```
function start() {
  console.log("Request handler 'start' was called.");

  function sleep(milliSeconds) {
    var startTime = new Date().getTime();
    while (new Date().getTime() < startTime + milliSeconds);
  }

  sleep(10000);
  return "Hello Start";
}

function upload() {
  console.log("Request handler 'upload' was called.");
  return "Hello Upload";
}

exports.start = start;
exports.upload = upload;
```

在第一个窗口中（“/start”）按下回车，然后快速切换到第二个窗口中（“/upload”）按下回车。

发生了什么： /start URL加载花了10秒，这和我们预期的一样。但是，/upload URL居然也花了10秒，

这到底是为什么呢？原因就是`start()`包含了阻塞操作。形象的说就是“它阻塞了所有其他的处理工作”。

Node.js可以在不新增额外线程的情况下，依然可以对任务进行并行处理 —— Node.js是单线程的。它通过事件轮询（event loop）来实现并行操作，对此，我们应该要充分利用这一点 —— 尽可能的避免阻塞操作，取而代之，多使用非阻塞操作。

接下来，我们会介绍一种错误的使用非阻塞操作的方式。

```
var exec = require("child_process").exec;

function start() {
  console.log("Request handler 'start' was called.");
  var content = "empty";

  exec("ls -lah", function (error, stdout, stderr) {
    content = stdout;
  });

  return content;
}

function upload() {
  console.log("Request handler 'upload' was called.");
  return "Hello Upload";
}

exports.start = start;
exports.upload = upload;
```

上述代码中，我们引入了一个新的Node.js模块，child_process。之所以用它，是为了实现一个既简单又实用的非阻塞操作：`exec()`。

`exec()`做了什么呢？它从Node.js来执行一个shell命令。在上述例子中，我们用它来获取当前目录下所有的文件（“`ls -lah`”）,然后，当/startURL请求的时候将文件信息输出到浏览器中。

### 以非阻塞操作进行请求响应

到目前为止，我们的应用已经可以通过应用各层之间传递值的方式（请求处理程序 -> 请求路由 -> 服务器）将请求处理程序返回的内容（请求处理程序最终要显示给用户的内容）传递给HTTP服务器。

现在我们采用如下这种新的实现方式：相对采用将内容传递给服务器的方式，我们这次采用将服务器“传递”给内容的方式。 从实践角度来说，就是将`response`对象（从服务器的回调函数`onRequest()`获取）通过请求路由传递给请求处理程序。 随后，处理程序就可以采用该对象上的函数来对请求作出响应。

先从server.js开始：

```
let http = require("http");
let url = require("url");

function start(route, handle) {
	function onRequest(request, response) {
		console.log('request received.');
		let pathname = url.parse(request.url).pathname;
		console.log('request for ' + pathname + ' received.');

		route(handle, pathname, response);
	}
	http.createServer(onRequest).listen(8888);
	console.log('server start listen.');
}

exports.start = start;
```

我们将`onRequest()`处理程序中所有有关`response`的函数调都移除，因为我们希望这部分工作让`route()`函数来完成。

```
function route(handle, pathname, response) {
	console.log('About to route a request for ' + pathname);
	if (typeof handle[pathname] === 'function') {
		handle[pathname](response);
	} else {
		console.log('No request handler found for ' + pathname);
		response.writeHead(404, {'Content-Type': 'text/plain'});
		response.write('404 Not found');
		response.end();
	}
}

exports.route = route;
```

最后，我们将requestHandler.js修改为如下形式：

```
let exec = require("child_process").exec;

function start(response) {
	console.log('Reques handler "start" was called');
	exec("find /", {timeout: 10000, maxBuffer: 20000*1024}, 
		function(error, stdout, stderr) {
		response.writeHead(200, {"Content-Type": "text/plain"});
		response.write(stdout);
		response.end();
	});
}

function upload(response) {
	console.log('Request handler "upload" was called');
	response.writeHead(200, {"Content-Type": "text/plain"});
	response.write("Hello Upload");
	response.end();
}

exports.start = start;
exports.upload = upload;
```

这时再次我们启动应用（node index.js），一切都会工作的很好。


当请求http://localhost:8888/start的时候，会花10秒钟的时间才载入，而当请求http://localhost:8888/upload的时候，会立即响应，纵然这个时候/start响应还在处理中。

## 更有用的场景

服务器，请求路由以及请求处理程序都已经完成了，下面让我们按照此前的用例给网站添加交互：用户选择一个文件，上传该文件，然后在浏览器中看到上传的文件。 为了保持简单，我们假设用户只会上传图片，然后我们应用将该图片显示到浏览器中。

要实现该功能，分为如下两步： 首先，让我们来看看如何处理POST请求（非文件上传），之后，我们使用Node.js的一个用于文件上传的外部模块。之所以采用这种实现方式有两个理由。

第一，尽管在Node.js中处理基础的POST请求相对比较简单，但在这过程中还是能学到很多。 
第二，用Node.js来处理文件上传（multipart POST请求）是比较复杂的，它不在本书的范畴，但，如何使用外部模块却是在本书涉猎内容之内。

### 处理POST请求

考虑这样一个简单的例子：我们显示一个文本区（textarea）供用户输入内容，然后通过POST请求提交给服务器。最后，服务器接受到请求，通过处理程序将输入的内容展示到浏览器中。

`/start`请求处理程序用于生成带文本区的表单，因此，我们将requestHandlers.js修改为如下形式：

```
let exec = require("child_process").exec;

function start(response) {
	console.log('Reques handler "start" was called');
	
	let body = `<html>
	<head>
		<meta http-qnuiv="Content-Type" content="text/html" charset=UTF-8/>
	</head>
	<body>
		<form action="/upload" method="post">
			<textarea name="text" cols="60" rows="20"></textarea>
			<input type="submit" value="Submit Text"/>
		</form>
	</body>
	</html>`
	response.writeHead(200, {"Content-Type": "text/html"});
	response.write(body);
	response.end();
}

function upload(response) {
	console.log('Request handler "upload" was called');
	response.writeHead(200, {"Content-Type": "text/plain"});
	response.write("Hello Upload");
	response.end();
}

exports.start = start;
exports.upload = upload;
```

当请求 `/start` 的时候，会收到一个页面，页面中有 `textarea` 元素和 `input[type="submit"]` 元素。当点击 `submit` 的时候，通过 `input` 的 `action` 导向到 `/upload` 链接，从而触发 upload 的处理函数。

采用异步回调来实现非阻塞地处理POST请求的数据。

这里采用非阻塞方式处理是明智的，因为POST请求一般都比较“重” —— 用户可能会输入大量的内容。用阻塞的方式处理大数据量的请求必然会导致用户操作的阻塞。

为了使整个过程非阻塞，Node.js会将POST数据拆分成很多小的数据块，然后通过触发特定的事件，将这些小数据块传递给回调函数。这里的特定的事件有`data`事件（表示新的小数据块到达了）以及`end`事件（表示所有的数据都已经接收完毕）。

我们需要告诉Node.js当这些事件触发的时候，回调哪些函数。怎么告诉呢？ 我们通过在`request`对象上注册监听器（listener） 来实现。这里的`request`对象是每次接收到HTTP请求时候，都会把该对象传递给`onRequest`回调函数。

**获取所有来自请求的数据，然后将这些数据给应用层处理，应该是HTTP服务器要做的事情。**

因此，我建议，我们直接在服务器中处理POST数据，然后将最终的数据传递给请求路由和请求处理器，让他们来进行进一步的处理。

实现思路就是： 将`data`和`end`事件的回调函数直接放在服务器中，在`data`事件回调中收集所有的POST数据，当接收到所有数据，触发`end`事件后，其回调函数调用请求路由，并将数据传递给它，然后，请求路由再将该数据传递给请求处理程序。

先从server.js开始：

```
var http = require("http");
var url = require("url");

function start(route, handle) {
  function onRequest(request, response) {
    var postData = "";
    var pathname = url.parse(request.url).pathname;
    console.log("Request for " + pathname + " received.");

    request.setEncoding("utf8");

    request.addListener("data", function(postDataChunk) {
      postData += postDataChunk;
      console.log("Received POST data chunk '"+
      postDataChunk + "'.");
    });

    request.addListener("end", function() {
      route(handle, pathname, response, postData);
    });

  }

  http.createServer(onRequest).listen(8888);
  console.log("Server has started.");
}

exports.start = start;
```

首先，我们设置了接收数据的编码格式为UTF-8，然后注册了“data”事件的监听器，用于收集每次接收到的新数据块，并将其赋值给`postData` 变量，最后，我们将请求路由的调用移到`end`事件处理程序中，以确保它只会当所有数据接收完毕后才触发，并且只触发一次。我们同时还把POST数据传递给请求路由，因为这些数据，请求处理程序会用到。

我建议可以尝试下，尝试着去输入一小段文本，以及大段内容，当大段内容的时候，就会发现data事件会触发多次。

我们接下来在`/upload`页面，展示用户输入的内容。要实现该功能，我们需要将`postData`传递给请求处理程序，修改router.js为如下形式：

```
function route(handle, pathname, response, postData) {
	console.log('About to route a request for ' + pathname);
	if (typeof handle[pathname] === 'function') {
		handle[pathname](response, postData);
	} else {
		console.log('No request handler found for ' + pathname);
		response.writeHead(404, {'Content-Type': 'text/plain'});
		response.write('404 Not found');
		response.end();
	}
}

exports.route = route;
```

然后，在requestHandlers.js中，我们将数据包含在对`upload`请求的响应中：

```
function start(response, postData) {
  console.log("Request handler 'start' was called.");

  var body = '<html>'+
    '<head>'+
    '<meta http-equiv="Content-Type" content="text/html; '+
    'charset=UTF-8" />'+
    '</head>'+
    '<body>'+
    '<form action="/upload" method="post">'+
    '<textarea name="text" rows="20" cols="60"></textarea>'+
    '<input type="submit" value="Submit text" />'+
    '</form>'+
    '</body>'+
    '</html>';

    response.writeHead(200, {"Content-Type": "text/html"});
    response.write(body);
    response.end();
}

function upload(response, postData) {
  console.log("Request handler 'upload' was called.");
  response.writeHead(200, {"Content-Type": "text/plain"});
  response.write("You've sent: " + postData);
  response.end();
}

exports.start = start;
exports.upload = upload;
```

好了，我们现在可以接收POST数据并在请求处理程序中处理该数据了。

我们最后要做的是： 当前我们是把请求的整个消息体传递给了请求路由和请求处理程序。我们应该只把POST数据中，我们感兴趣的部分传递给请求路由和请求处理程序。在我们这个例子中，我们感兴趣的其实只是`text`字段。

例如我们在输入一段“Hello Node.js” 的时候，`postData` 的返回值是 `text=Hello+Node.js`，我们需要把 `text` 其中的内容保留显示。所以使用 querystring 来进行处理。

```
var querystring = require("querystring");

function upload(response, postData) {
  console.log("Request handler 'upload' was called.");
  response.writeHead(200, {"Content-Type": "text/plain"});
  response.write("You've sent the text: "+
  querystring.parse(postData).text);
  response.end();
}
```

### 处理文件上传

最后，我们来实现我们最终的用例：允许用户上传图片，并将该图片在浏览器中显示出来。

这里我们要用到的外部模块是Felix Geisendörfer开发的`node-formidable`模块。它对解析上传的文件数据做了很好的抽象。 其实说白了，处理文件上传“就是”处理POST数据 —— 但是，麻烦的是在具体的处理细节，所以，这里采用现成的方案更合适点。

```
npm install formidable
```

这里该模块做的就是将通过HTTP POST请求提交的表单，在Node.js中可以被解析。我们要做的就是创建一个新的`IncomingForm`，它是对提交表单的抽象表示，之后，就可以用它解析`request`对象，获取表单中需要的数据字段。

`node-formidable`官方的例子展示了这两部分是如何融合在一起工作的：

```
var formidable = require('formidable'),
    http = require('http'),
    util = require('util');

http.createServer(function(req, res) {
  if (req.url == '/upload' && req.method.toLowerCase() == 'post') {
    // parse a file upload
    var form = new formidable.IncomingForm();
    form.parse(req, function(err, fields, files) {
      res.writeHead(200, {'content-type': 'text/plain'});
      res.write('received upload:\n\n');
      res.end(util.inspect({fields: fields, files: files}));
    });
    return;
  }

  // show a file upload form
  res.writeHead(200, {'content-type': 'text/html'});
  res.end(
    '<form action="/upload" enctype="multipart/form-data" '+
    'method="post">'+
    '<input type="text" name="title"><br>'+
    '<input type="file" name="upload" multiple="multiple"><br>'+
    '<input type="submit" value="Upload">'+
    '</form>'
  );
}).listen(8888);
```

可以看到通过调用`form.parse`传递给回调函数的`files`对象的内容，如下所示：

```
received upload:

{ fields: { title: 'Hello World' },
  files:
   { upload:
      { size: 1558,
        path: '/tmp/1c747974a27a6292743669e91f29350b',
        name: 'us-flag.png',
        type: 'image/png',
        lastModifiedDate: Tue, 21 Jun 2011 07:02:41 GMT,
        _writeStream: [Object],
        length: [Getter],
        filename: [Getter],
        mime: [Getter] } } }
```

为了实现我们的功能，我们需要将上述代码应用到我们的应用中，另外，我们还要考虑如何将上传文件的内容（保存在`/tmp`目录中）显示到浏览器中。

显然，我们需要将该文件读取到我们的服务器中，使用一个叫`fs`的模块。

我们来添加`/showURL`的请求处理程序，该处理程序直接硬编码将文件`/tmp/test.png`内容展示到浏览器中。当然了，首先需要将该图片保存到这个位置才行。

将requestHandlers.js修改为如下形式：

```
var querystring = require("querystring"),
    fs = require("fs");

function start(response, postData) {
  console.log("Request handler 'start' was called.");

  var body = '<html>'+
    '<head>'+
    '<meta http-equiv="Content-Type" '+
    'content="text/html; charset=UTF-8" />'+
    '</head>'+
    '<body>'+
    '<form action="/upload" method="post">'+
    '<textarea name="text" rows="20" cols="60"></textarea>'+
    '<input type="submit" value="Submit text" />'+
    '</form>'+
    '</body>'+
    '</html>';

    response.writeHead(200, {"Content-Type": "text/html"});
    response.write(body);
    response.end();
}

function upload(response, postData) {
  console.log("Request handler 'upload' was called.");
  response.writeHead(200, {"Content-Type": "text/plain"});
  response.write("You've sent the text: "+
  querystring.parse(postData).text);
  response.end();
}

function show(response, postData) {
  console.log("Request handler 'show' was called.");
  fs.readFile("/tmp/test.png", "binary", function(error, file) {
    if(error) {
      response.writeHead(500, {"Content-Type": "text/plain"});
      response.write(error + "\n");
      response.end();
    } else {
      response.writeHead(200, {"Content-Type": "image/png"});
      response.write(file, "binary");
      response.end();
    }
  });
}

exports.start = start;
exports.upload = upload;
exports.show = show;
```

我们还需要将这新的请求处理程序，添加到index.js中的路由映射表中：

```
var server = require("./server");
var router = require("./router");
var requestHandlers = require("./requestHandlers");

var handle = {}
handle["/"] = requestHandlers.start;
handle["/start"] = requestHandlers.start;
handle["/upload"] = requestHandlers.upload;
handle["/show"] = requestHandlers.show;

server.start(router.route, handle);
```

重启服务器之后，通过访问http://localhost:8888/show，就可以看到保存在`/tmp/test.png`的图片了。

好，最后我们要的就是：

在`/start`表单中添加一个文件上传元素
将`node-formidable`整合到我们的`upload`请求处理程序中，用于将上传的图片保存到`/tmp/test.png`
将上传的图片内嵌到`/uploadURL`输出的HTML中

第一项很简单。只需要在HTML表单中，添加一个`multipart/form-data`的编码类型，移除此前的文本区，添加一个文件上传组件，并将提交按钮的文案改为“Upload file”即可。 如下requestHandler.js所示：

```
function start(response, postData) {
	console.log('Reques handler "start" was called');
	
	let body = `<html>
	<head>
		<meta http-qnuiv="Content-Type" content="text/html" charset=UTF-8/>
	</head>
	<body>
		<form action="/upload" method="post" enctype="multipart/from-data">
			<input type="file" name="upload" />
			<input type="submit" value="Upload file" />
		</form>
	</body>
	</html>`
	response.writeHead(200, {"Content-Type": "text/html"});
	response.write(body);
	response.end();
}
```

很好。下一步相对比较复杂。这里有这样一个问题： 我们需要在`upload`处理程序中对上传的文件进行处理，这样的话，我们就需要将`request`对象传递给`node-formidable`的`form.parse`函数。

但是，我们有的只是`response`对象和`postData`数组。看样子，我们只能不得不将`request`对象从服务器开始一路通过请求路由，再传递给请求处理程序。 或许还有更好的方案，但是，不管怎么说，目前这样做可以满足我们的需求。

到这里，我们可以将`postData`从服务器以及请求处理程序中移除了 —— 一方面，对于我们处理文件上传来说已经不需要了，另外一方面，它甚至可能会引发这样一个问题： 我们已经“消耗”了`request`对象中的数据，这意味着，对于`form.parse`来说，当它想要获取数据的时候就什么也获取不到了。（因为Node.js不会对数据做缓存）

我们从server.js开始 —— 移除对`postData`的处理以及`request.setEncoding` （这部分node-formidable自身会处理），转而采用将request对象传递给请求路由的方式：

```
let http = require("http");
let url = require("url");

function start(route, handle) {
	function onRequest(request, response) {
		console.log('request received.');
		let pathname = url.parse(request.url).pathname;
		console.log('request for ' + pathname + ' received.');
		route(handle, pathname, response, request);
	}
	http.createServer(onRequest).listen(8888);
	console.log('server start listen.');
}

exports.start = start;
```

接下来是 router.js —— 我们不再需要传递`postData`了，这次要传递`request`对象：

```
function route(handle, pathname, response, request) {
	console.log('About to route a request for ' + pathname);
	if (typeof handle[pathname] === 'function') {
		handle[pathname](response, request);
	} else {
		console.log('No request handler found for ' + pathname);
		response.writeHead(404, {'Content-Type': 'text/plain'});
		response.write('404 Not found');
		response.end();
	}
}

exports.route = route;
```

现在，`request`对象就可以在我们的`upload`请求处理程序中使用了。node-formidable会处理将上传的文件保存到本地`/tmp`目录中，而我们需要做的是确保该文件保存成`/tmp/test.png`。 没错，我们保持简单，并假设只允许上传PNG图片。

这里采用`fs.renameSync(path1,path2)`来实现。要注意的是，正如其名，该方法是同步执行的， 也就是说，如果该重命名的操作很耗时的话会阻塞。 这块我们先不考虑。

接下来，我们把处理文件上传以及重命名的操作放到一起，如下requestHandlers.js所示：

```
let querystring = require("querystring");
let fs = require("fs");
let formidable = require("formidable");

function start(response) {
	console.log('Reques handler "start" was called');
	
	let body = `<html>
	<head>
		<meta http-qnuiv="Content-Type" content="text/html" charset=UTF-8/>
	</head>
	<body>
		<form action="/upload" method="post" enctype="multipart/form-data">
			<input type="file" name="upload" multiple="multiple"/>
			<input type="submit" value="Upload file" />
		</form>
	</body>
	</html>`
	response.writeHead(200, {"Content-Type": "text/html"});
	response.write(body);
	response.end();
}

function upload(response, request) {
	console.log('Request handler "upload" was called');

	let form = new formidable.IncomingForm();
	console.log('About to parse');
	form.parse(request, function(error, fields, files) {
	    console.log("parsing done");
	    fs.renameSync(files.upload.path, "./tmp/test.png");
	    response.writeHead(200, {"Content-Type": "text/html"});
	    response.write("received image:<br/>");
	    response.write("<img src='/show' />");
	    response.end();
	});
}

function show(response) {
	console.log('Request handler "show" was called.');
	fs.readFile("./tmp/test.png", "binary", function(error, file) {
		if (error) {
			response.writeHead(500, {"Content-Type": "text/plain"});
			response.write(error + "\n");
			response.end();
		} else {
			response.writeHead(200, {"Content-Type": "image/png"});
			response.write(file, "binary");
			response.end();
		}
	})
}

exports.start = start;
exports.upload = upload;
exports.show = show;
```

好了，重启服务器，我们应用所有的功能就可以用了。选择一张本地图片，将其上传到服务器，然后浏览器就会显示该图片。





































































































































































































