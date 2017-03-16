---
title: Node.js 包教不会笔记
date: 2017-03-12 21:16:46
categories: coding
tags:
  - Node.js
---

本文是笔者在学习[《Node.js 包教不包会》 -- by alsotang](https://github.com/alsotang/node-lessons) 中的例子的笔记。

* [superagent](http://visionmedia.github.io/superagent/) 是个 http 方面的库，可以**发起** get 或 post 请求。
* [cheerio](https://github.com/cheeriojs/cheerio ) 大家可以理解成一个 Node.js 版的 jquery，用来从网页中以 css selector 取数据，使用方式跟 jquery 一样一样的。
* [eventproxy](https://www.npmjs.com/package/eventproxy) 管理到底异步操作是否完成，完成之后，它会自动调用你提供的处理函数，并将抓取到的数据当参数传过来。
* [async](https://github.com/caolan/async/blob/v1.5.2/README.md) 当你需要去多个源(一般是小于 10 个)汇总数据的时候，用 eventproxy 方便；当你需要用到队列，需要控制并发数，或者你喜欢函数式编程思维时，使用 async。
* [mocha](http://mochajs.org/) 测试框架 
* [should](http://shouldjs.github.io) 断言库 
* [istanbul](https://github.com/gotwarlost/istanbul) 测试率覆盖工具
* [chai](http://chaijs.com/) 全栈的断言库
* [phantomjs](http://phantomjs.org/) headless 浏览器 
* [supertest](https://www.npmjs.com/package/supertest)专门用来配合 express （准确来说是所有兼容 connect 的 web 框架）进行集成测试的。
* [benchmark](https://github.com/bestiejs/benchmark.js) 可以用来测试 JavaScript 语句执行所需要的时间
* [《线上部署：heroku》](https://github.com/alsotang/node-lessons/tree/master/lesson12)：如何使用 Paas 部署个人展示项目
* [《持续集成平台：travis》](https://github.com/alsotang/node-lessons/tree/master/lesson13)：使用 travis 可以创建一个空白的环境，针对不同的 node 版本测试整个项目的运行情况。
* [cookie-parser](https://github.com/expressjs/cookie-parser) express 中操作cookie
* [express-session](https://github.com/expressjs/session ) express 中操作 session
* [q](https://github.com/kriskowal/q) Node.js 中 Promise 的实现
* [connect](https://github.com/senchalabs/connect) Node.js 中用来实现服务器处理中间件
* [express](https://github.com/expressjs/express) Node.js 中实现服务器处理的中间件，是 Connect 的升级版，还封装了很多对象来方便业务逻辑处理。


<!--more-->

## 《学习使用外部模块》

```
npm install express utility --save
```

多了个 `--save` 参数，这个参数的作用，就是会在你安装依赖的同时，自动把这些依赖写入 package.json。命令执行完成之后，查看 package.json，会发现多了一个 `dependencies` 字段

我们开始写应用层的代码，建立一个 `app.js` 文件，复制以下代码进去：

```
// 引入依赖
var express = require('express');
var utility = require('utility');

// 建立 express 实例
var app = express();

app.get('/', function (req, res) {
  // 从 req.query 中取出我们的 q 参数。
  // 如果是 post 传来的 body 数据，则是在 req.body 里面，不过 express 默认不处理 body 中的信息，需要引入 https://github.com/expressjs/body-parser 这个中间件才会处理，这个后面会讲到。
  var q = req.query.q;

  // 调用 utility.md5 方法，得到 md5 之后的值
  // 调用 utility.sha1
  // utility 的 github 地址：https://github.com/node-modules/utility
  // 里面定义了很多常用且比较杂的辅助方法，可以去看看
  var md5Value = utility.md5(q);

  res.send(md5Value);
});

app.listen(3000, function (req, res) {
  console.log('app is running at port 3000');
});
```

## 《使用 superagent 与 cheerio 完成简单爬虫》

### 知识点

1. 学习使用 superagent 抓取网页
2. 学习使用 cheerio 分析网页

### 课程内容

[superagent](http://visionmedia.github.io/superagent/) 是个 http 方面的库，可以**发起** get 或 post 请求。

[cheerio](https://github.com/cheeriojs/cheerio ) 大家可以理解成一个 Node.js 版的 jquery，用来从网页中以 css selector 取数据，使用方式跟 jquery 一样一样的。

**Node.js 里面没有 XMLHttpRequest 对象，所以需要用 superagent 替代。**

```
app.get('/', function (req, res, next) {
  // 用 superagent 去抓取 https://cnodejs.org/ 的内容
  superagent.get('https://cnodejs.org/')
    .end(function (err, sres) {
      // 常规的错误处理
      if (err) {
        return next(err);
      }
      // sres.text 里面存储着网页的 html 内容，将它传给 cheerio.load 之后
      // 就可以得到一个实现了 jquery 接口的变量，我们习惯性地将它命名为 `$`
      // 剩下就都是 jquery 的内容了
      var $ = cheerio.load(sres.text);
      var items = [];
      $('#topic_list .topic_title').each(function (idx, element) {
        var $element = $(element);
        items.push({
          title: $element.attr('title'),
          href: $element.attr('href')
        });
      });

      res.send(items);
    });
});
```

## 《使用 eventproxy 控制并发》

注意，cnodejs.org 网站有并发连接数的限制，所以当请求发送太快的时候会导致返回值为空或报错。建议一次抓取3个主题即可。文中的40只是为了方便讲解

上一课我们介绍了如何使用 superagent 和 cheerio 来取主页内容，那只需要发起一次 http get 请求就能办到。但这次，我们需要取出每个主题的第一条评论，这就要求我们对每个主题的链接发起请求，并用 cheerio 去取出其中的第一条评论。

首先 app.js 应该长这样

```
var eventproxy = require('eventproxy');
var superagent = require('superagent');
var cheerio = require('cheerio');
// url 模块是 Node.js 标准库里面的
// http://nodejs.org/api/url.html
var url = require('url');

var cnodeUrl = 'https://cnodejs.org/';

superagent.get(cnodeUrl)
  .end(function (err, res) {
    if (err) {
      return console.error(err);
    }
    var topicUrls = [];
    var $ = cheerio.load(res.text);
    // 获取首页所有的链接
    $('#topic_list .topic_title').each(function (idx, element) {
      var $element = $(element);
      // $element.attr('href') 本来的样子是 /topic/542acd7d5d28233425538b04
      // 我们用 url.resolve 来自动推断出完整 url，变成
      // https://cnodejs.org/topic/542acd7d5d28233425538b04 的形式
      // 具体请看 http://nodejs.org/api/url.html#url_url_resolve_from_to 的示例
      var href = url.resolve(cnodeUrl, $element.attr('href'));
      topicUrls.push(href);
    });

    console.log(topicUrls);
  });
```

运行 node app.js

![](https://raw.githubusercontent.com/alsotang/node-lessons/master/lesson4/1.png)

抓取之前，还是得介绍一下 `eventproxy` 这个库。

用 js 写过异步的同学应该都知道，如果你要并发异步获取两三个地址的数据，并且要在获取到数据之后，对这些数据一起进行利用的话，常规的写法是自己维护一个计数器。

先定义一个 `var count = 0`，然后每次抓取成功以后，就 `count++`。如果你是要抓取三个源的数据，由于你根本不知道这些异步操作到底谁先完成，那么每次当抓取成功的时候，就判断一下 `count === 3`。当值为真时，使用另一个函数继续完成操作。

而 `eventproxy` 就起到了这个计数器的作用，它来帮你管理到底这些异步操作是否完成，完成之后，它会自动调用你提供的处理函数，并将抓取到的数据当参数传过来。

于是我们用计数器来写，会写成这样：

```
(function () {
  var count = 0;
  var result = {};

  $.get('http://data1_source', function (data) {
    result.data1 = data;
    count++;
    handle();
    });
  $.get('http://data2_source', function (data) {
    result.data2 = data;
    count++;
    handle();
    });
  $.get('http://data3_source', function (data) {
    result.data3 = data;
    count++;
    handle();
    });

  function handle() {
    if (count === 3) {
      var html = fuck(result.data1, result.data2, result.data3);
      render(html);
    }
  }
})();
```

如果我们用 eventproxy，写出来是这样的：

```
var ep = new eventproxy();
ep.all('data1_event', 'data2_event', 'data3_event', function (data1, data2, data3) {
  var html = fuck(data1, data2, data3);
  render(html);
});

$.get('http://data1_source', function (data) {
  ep.emit('data1_event', data);
  });

$.get('http://data2_source', function (data) {
  ep.emit('data2_event', data);
  });

$.get('http://data3_source', function (data) {
  ep.emit('data3_event', data);
  });
```

`ep.all('data1_event', 'data2_event', 'data3_event', function (data1, data2, data3) {});`

这一句，监听了三个事件，分别是 `data1_event, data2_event, data3_event`，每次当一个源的数据抓取完成时，就通过 `ep.emit()` 来告诉 `ep` 自己，某某事件已经完成了。

当三个事件未同时完成时，`ep.emit()` 调用之后不会做任何事；当三个事件都完成的时候，就会调用末尾的那个回调函数，来对它们进行统一处理。

回到正题，之前我们已经得到了一个长度为 40 的 `topicUrls` 数组，里面包含了每条主题的链接。那么意味着，我们接下来要发出 40 个并发请求。我们需要用到 `eventproxy` 的 `#after` API。

```
// 得到 topicUrls 之后

// 得到一个 eventproxy 的实例
var ep = new eventproxy();

// 命令 ep 重复监听 topicUrls.length 次（在这里也就是 40 次） `topic_html` 事件再行动
ep.after('topic_html', topicUrls.length, function (topics) {
  // topics 是个数组，包含了 40 次 ep.emit('topic_html', pair) 中的那 40 个 pair

  // 开始行动
  topics = topics.map(function (topicPair) {
    // 接下来都是 jquery 的用法了
    var topicUrl = topicPair[0];
    var topicHtml = topicPair[1];
    var $ = cheerio.load(topicHtml);
    return ({
      title: $('.topic_full_title').text().trim(),
      href: topicUrl,
      comment1: $('.reply_content').eq(0).text().trim(),
    });
  });

  console.log('final:');
  console.log(topics);
});

topicUrls.forEach(function (topicUrl) {
  superagent.get(topicUrl)
    .end(function (err, res) {
      console.log('fetch ' + topicUrl + ' successful');
      ep.emit('topic_html', [topicUrl, res.text]);
    });
});
```

## 《使用 async 控制并发》

代码的入口是 app.js，当调用 node app.js 时，它会输出 CNode(https://cnodejs.org/ ) 社区首页的所有主题的标题，链接和第一条评论，以 json 的格式。

注意：与上节课不同，并发连接数需要控制在 5 个。

lesson4 的代码其实是不完美的。为什么这么说，是因为在 lesson4 中，我们一次性发了 40 个并发请求出去，要知道，除去 CNode 的话，别的网站有可能会因为你发出的并发连接数太多而当你是在恶意请求，把你的 IP 封掉。

这次我们要介绍的是 `async` 的 `mapLimit(arr, limit, iterator, callback)` 接口。另外，还有个常用的控制并发连接数的接口是 `queue(worker, concurrency)`，大家可以去 https://github.com/caolan/async#queueworker-concurrency 看看说明。

对了，还有个问题是，什么时候用 `eventproxy`，什么时候使用 `async` 呢？它们不都是用来做异步流程控制的吗？

我的答案是：

当你需要去多个源(一般是小于 10 个)汇总数据的时候，用 `eventproxy` 方便；当你需要用到队列，需要控制并发数，或者你喜欢函数式编程思维时，使用 `async`。

```
var fetchUrl = function (url, callback) {
  // delay 的值在 2000 以内，是个随机的整数
  var delay = parseInt((Math.random() * 10000000) % 2000, 10);
  concurrencyCount++;
  console.log('现在的并发数是', concurrencyCount, '，正在抓取的是', url, '，耗时' + delay + '毫秒');
  setTimeout(function () {
    concurrencyCount--;
    callback(null, url + ' html content');
  }, delay);
};
```

接着，我们使用 `async.mapLimit` 来并发抓取，并获取结果。

```
async.mapLimit(urls, 5, function (url, callback) {
  fetchUrl(url, callback);
}, function (err, result) {
  console.log('final:');
  console.log(result);
});
```

并发链接数是从 1 开始增长的，增长到 5 时，就不再增加。当其中有任务完成时，再继续抓取。并发连接数始终控制在 5 个。

最终代码：

```
let eventproxy = require('eventproxy');
let superagent = require('superagent');
let cheerio = require('cheerio');
let url = require('url');
let async = require('async');

let codeURL = 'https://cnodejs.org/';


superagent.get(codeURL)
.end(function(err, response) {
    if (err) {
        return console.log(err);
    }
    
    let topicURL = [];
    let $ = cheerio.load(response.text);
    $('#topic_list .topic_title').each(function(index, element) {
        topicURL.push(url.resolve(codeURL, element.attribs.href));
    });

    

    let concurrencyCount = 0;
    async.mapLimit(topicURL, 5, function(url, callback) {
        concurrencyCount++;
        console.log('current url is: ' + url + ' and concurrencyCount is: ' + concurrencyCount);
        superagent.get(url).end(function(i){ return function(err, data) {
            concurrencyCount--;
            if (err) {
                console.log('error while GET ' + err);
            }
            callback(err, [url, data])
        }}(url))
    }, function( err, result) {
        console.log('final:');
        result.forEach(function(value, key) {
            let topicHref = value[0];
            let $ = cheerio.load(value[1].text);
            console.log({
                title: $('.topic_full_title').text().trim(),
                href: topicHref,
                comment: $('.reply_content').eq(0).text().trim(),
            })

        })
    })
})
```

## 《测试用例：mocha，should，istanbul》

学习使用测试框架 mocha : http://mochajs.org/
学习使用断言库 should : https://github.com/tj/should.js
学习使用测试率覆盖工具 istanbul : https://github.com/gotwarlost/istanbul
简单 Makefile 的编写 : http://blog.csdn.net/haoel/article/details/2886

```
var fibonacci = function (n) {
  if (n === 0) {
    return 0;
  }
  if (n === 1) {
    return 1;
  }
  return fibonacci(n-1) + fibonacci(n-2);
};

if (require.main === module) {
  // 如果是直接执行 main.js，则进入此处
  // 如果 main.js 被其他文件 require，则此处不会执行。
  var n = Number(process.argv[2]);
  console.log('fibonacci(' + n + ') is', fibonacci(n));
}
```

我们先得把 main.js 里面的 `fibonacci` 暴露出来，这个简单。加一句

```
exports.fibonacci = fibonacci;
```

然后我们在 `test/main.test.js` 中引用我们的 main.js，并开始一个简单的测试。

```
// file: test/main.test.js
var main = require('../main');
var should = require('should');

describe('test/main.test.js', function () {
  it('should equal 55 when n === 10', function () {
    main.fibonacci(10).should.equal(55);
  });
});
```

装个全局的 mocha: `$ npm install mocha -g`。

在 lesson6 目录下，直接执行

```
$ mocha
```

那么，代码中的 describe 和 it 是什么意思呢？其实就是 BDD 中的那些意思，把它们当做语法来记就好了。

`describe` 中的字符串，用来描述你要测的主体是什么；it 当中，描述具体的 case 内容。

`should` 在 js 的 Object “基类”上注入了一个 `#should` 属性，这个属性中，又有着许许多多的属性可以被访问。

比如测试一个数是不是大于3，则是 `(5).should.above(3)`；测试一个字符串是否有着特定前缀：`'foobar'.should.startWith('foo')`;。should.js API 在：https://github.com/tj/should.js

回到正题，还记得我们 fibonacci 函数的几个要求吗？

```
* 当 n === 0 时，返回 0；n === 1时，返回 1;
* n > 1 时，返回 `fibonacci(n) === fibonacci(n-1) + fibonacci(n-2)`，如 `fibonacci(10) === 55`;
* n 不可大于10，否则抛错，因为 Node.js 的计算性能没那么强。
* n 也不可小于 0，否则抛错，因为没意义。
* n 不为数字时，抛错。
```

我们用测试用例来描述一下这几个要求，更新后的 main.test.js 如下：

```
var main = require('../main');
var should = require('should');

describe('test/main.test.js', function () {
  it('should equal 0 when n === 0', function () {
    main.fibonacci(0).should.equal(0);
  });

  it('should equal 1 when n === 1', function () {
    main.fibonacci(1).should.equal(1);
  });

  it('should equal 55 when n === 10', function () {
    main.fibonacci(10).should.equal(55);
  });

  it('should throw when n > 10', function () {
    (function () {
      main.fibonacci(11);
    }).should.throw('n should <= 10');
  });

  it('should throw when n < 0', function () {
    (function () {
      main.fibonacci(-1);
    }).should.throw('n should >= 0');
  });

  it('should throw when n isnt Number', function () {
    (function () {
      main.fibonacci('呵呵');
    }).should.throw('n should be a Number');
  });
});
```

我们这时候跑一下 `$ mocha`，会发现后三个 case 都没过。

于是我们更新 `fibonacci` 的实现：

```
var fibonacci = function (n) {
  if (typeof n !== 'number') {
    throw new Error('n should be a Number');
  }
  if (n < 0) {
    throw new Error('n should >= 0')
  }
  if (n > 10) {
    throw new Error('n should <= 10');
  }
  if (n === 0) {
    return 0;
  }
  if (n === 1) {
    return 1;
  }

  return fibonacci(n-1) + fibonacci(n-2);
};
```

再跑一次 `$ mocha`，就过了。这就是传说中的测试驱动开发：**先把要达到的目的都描述清楚，然后让现有的程序跑不过 case，再修补程序，让 case 通过。**

安装一个 `istanbul : $ npm i istanbul -g`

执行 `$ istanbul cover _mocha`

这会比直接使用 `mocha` 多一行覆盖率的输出，

可以看到，我们其中的分支覆盖率是 91.67%，行覆盖率是 87.5%。

运行这条命令以后，会在命令运行的目录生成 `coverage` 文件夹。

打开 `open coverage/lcov-report/index.html` 看看，可以得到测试的覆盖率。

## 《浏览器端测试：mocha，chai，phantomjs》

### 知识点

1. 学习使用测试框架 mocha 进行前端测试 : http://mochajs.org/
2. 了解全栈的断言库 chai: http://chaijs.com/
3. 了解 headless 浏览器 phantomjs: http://phantomjs.org/

lesson6 的内容都是针对后端环境中 node 的一些单元测试方案，出于应用健壮性的考量，针对前端 js 脚本的单元测试也非常重要。而**前后端通吃，也是 mocha 的一大特点**。

首先，前端脚本的单元测试主要有两个困难需要解决。

1. 运行环境应当在浏览器中，可以操纵浏览器的DOM对象，且可以随意定义执行时的 html 上下文。
2. 测试结果应当可以直接反馈给 mocha，判断测试是否通过。

### 浏览器环境执行

我们首先搭建一个测试原型，用 mocha 自带的脚手架可以自动生成。

```
cd vendor            # 进入我们的项目文件夹
npm i -g mocha       # 安装全局的 mocha 命令行工具
mocha init .         # 生成脚手架
```

mocha就会自动帮我们生成一个简单的测试原型, 目录结构如下

```
.
├── index.html       # 这是前端单元测试的入口
├── mocha.css
├── mocha.js
└── tests.js         # 我们的单元测试代码将在这里编写
```

其中 index.html 是单元测试的入口，tests.js 是我们的测试用例文件。

我们直接在 index.html 插入上述示例的 fibonacci 函数以及断言库 chaijs。

```
<div id="mocha"></div>
<script src='https://cdn.rawgit.com/chaijs/chai/master/chai.js'></script>
<script>
  var fibonacci = function (n) {
    if (n === 0) {
      return 0;
    }
    if (n === 1) {
      return 1;
    }
    return fibonacci(n-1) + fibonacci(n-2);
  };
</script>
```

然后在tests.js中写入对应测试用例

```
var should = chai.should();
describe('simple test', function () {
  it('should equal 0 when n === 0', function () {
    window.fibonacci(0).should.equal(0);
  });
});
```

这时打开index.html，可以发现测试结果，我们完成了浏览器端的脚本测试

mocha没有提供一个命令行的前端脚本测试环境(因为我们的脚本文件需要运行在浏览器环境中)，因此我们使用phanatomjs帮助我们搭建一个模拟环境。不重复制造轮子，这里直接使用mocha-phantomjs帮助我们在命令行运行测试。

首先安装mocha-phanatomjs

```
npm i -g mocha-phantomjs
```

然后在 index.html 的页面下加上这段兼容代码

```
<script>mocha.run()</script>
```

改为

```
<script>
  if (window.initMochaPhantomJS && window.location.search.indexOf('skip') === -1) {
    initMochaPhantomJS()
  }
  mocha.ui('bdd');
  expect = chai.expect;

  mocha.run();
</script>
```

这时候, 我们在命令行中运行

```
mocha-phantomjs index.html --ssl-protocol=any --ignore-ssl-errors=true
```

结果展现是不是和后端代码测试很类似

更进一步，我们可以直接在 package.json 的 scripts 中添加 (package.json 通过 npm init 生成，这里不再赘述)

```
"scripts": {
  "test": "mocha-phantomjs index.html --ssl-protocol=any --ignore-ssl-errors=true"
},
```

将mocha-phantomjs作为依赖

```
npm i mocha-phantomjs --save-dev
```

直接运行

```
npm test
```

至此,我们实现了前端脚本的单元测试，基于 phanatomjs 你几乎可以调用所有的浏览器方法，而 mocha-phanatomjs 也可以很便捷地将测试结果反馈到 mocha，便于后续的持续集成。

## 《测试用例：supertest》

superagent 是用来抓取页面用的，而 supertest，是专门用来配合 express （准确来说是所有兼容 connect 的 web 框架）进行集成测试的。

对了，大家去装个 nodemon https://github.com/remy/nodemon 。

```
$ npm i -g nodemon
```

这个库是专门调试时候使用的，它会自动检测 node.js 代码的改动，然后帮你自动重启应用。在调试时可以完全用 nodemon 命令代替 node 命令。

```
$ nodemon app.js
```

启动我们的应用试试，然后随便改两行代码，就可以看到 nodemon 帮我们重启应用了。

## 《正则表达式》

```
var web_development = "python php ruby javascript jsonp perhapsphpisoutdated";
```

找出其中 包含 p 但不包含 ph 的所有单词，即

```
[ 'python', 'javascript', 'jsonp' ]
```

开始这门课之前，大家先去看两篇文章。

《正则表达式30分钟入门教程》：http://deerchao.net/tutorials/regex/regex.htm

上面这篇介绍了正则表达式的基础知识，但是对于零宽断言没有展开来讲，零宽断言看下面这篇：

《正则表达式之：零宽断言不『消费』》：http://fxck.it/post/50558232873

接下来我们主要讲讲 js 中需要注意的地方，至于正则表达式的内容，上面那两篇文章足够学习了。

第一，

js 中，对于四种零宽断言，只支持 零宽度正预测先行断言 和 零宽度负预测先行断言 这两种。

第二，

js 中，正则表达式后面可以跟三个 flag，比如 /something/igm。

他们的意义分别是，

* `i` 的意义是不区分大小写
* `g` 的意义是，匹配多个
* `m` 的意义是，是 `^` 和 `$` 可以匹配每一行的开头。

## 《benchmark 怎么写》

[benchmark](https://github.com/bestiejs/benchmark.js) 可以用来测试 JavaScript 语句执行所需要的时间

Using npm:

```
$ npm i --save benchmark
```

In Node.js:

```
var Benchmark = require('benchmark');
```

Usage example:

```
var suite = new Benchmark.Suite;

// add tests
suite.add('RegExp#test', function() {
  /o/.test('Hello World!');
})
.add('String#indexOf', function() {
  'Hello World!'.indexOf('o') > -1;
})
// add listeners
.on('cycle', function(event) {
  console.log(String(event.target));
})
.on('complete', function() {
  console.log('Fastest is ' + this.filter('fastest').map('name'));
})
// 这里的 async 不是 mocha 测试那个 async 的意思，这个选项与它的时间计算有关，默认勾上就好了。
.run({ 'async': true });

// logs:
// => RegExp#test x 4,161,532 +-0.99% (59 cycles)
// => String#indexOf x 6,139,623 +-1.00% (131 cycles)
// => Fastest is String#indexOf
```

## 《Mongodb 与 Mongoose 的使用》

### mongodb

mongodb 这个名词相信大家不会陌生吧。有段时间 nosql 的概念炒得特别火，其中 hbase redis mongodb couchdb 之类的名词都相继进入了大众的视野。

**hbase 和 redis 和 mongodb 和 couchdb 虽然都属于 nosql 的大范畴。但它们关注的领域是不一样的。hbase 是存海量数据的，redis 用来做缓存，而 mongodb 和 couchdb 则试图取代一些使用 mysql 的场景。**

在 sql 中，我们的数据层级是：数据库（db） -> 表（table） -> 记录（record）-> 字段；在 mongodb 中，数据的层级是：数据库 -> collection -> document -> 字段。这四个概念可以对应得上。

文档型数据这个名字中，“文档”两个字很容易误解。其实这个文档就是 bson 的意思。bson 是 json 的超集，比如 json 中没法储存二进制类型，而 bson 拓展了类型，提供了二进制支持。mongodb 中存储的一条条记录都可以用 bson 来表示。所以你也可以认为，**mongodb 是个存 bson 数据的数据库，或是存哈希数据的数据库**。

在 mongodb 中，**表与表之间是没有联系的，不像 sql 中一样，可以设定外键，可以进行表连接**。mongodb 中，也无法支持事务。所以这样的表，无债一身轻。可以很轻易地 scale 至多个实例（假设实例都有不同的物理位置）上。

mongodb 中，collection 是 schema-less 的。**在 sql 中，我们需要用建表语句来表明数据应该具有的形式，而 mongodb 中，可以在同一张里存各种各样不同的形式的数据**。

mongodb 和 mysql 要我选的话，无关紧要的应用我会选择 mongodb，就当个简单的存 json 数据的数据库来用；如果是线上应用，肯定还是会选择 mysql。毕竟 sql 比较成熟，而且各种常用场景的最佳实践都有先例了。

顺便说说 mongodb 与 redis 的不同。mongodb 是用来存非临时数据的，可以认为是存在硬盘上，而 redis 的数据可以认为都在内存中，存储临时数据，丢了也无所谓。对于稍微复杂的查询，redis 支持的查询方式太少太少了，几乎可以认为是 key-value 的。

## cookie 和 session

首先产生了 cookie 这门技术来解决这个问题，cookie 是 http 协议的一部分，它的处理分为如下几步：

* 服务器向客户端发送 cookie。
    * 通常使用 HTTP 协议规定的 set-cookie 头操作。
    * 规范规定 cookie 的格式为 name = value 格式，且必须包含这部分。
* 浏览器将 cookie 保存。
* 每次请求浏览器都会将 cookie 发向服务器。

其他可选的 cookie 参数会影响将 cookie 发送给服务器端的过程，主要有以下几种：

* path：表示 cookie 影响到的路径，匹配该路径才发送这个 cookie。
* expires 和 maxAge：告诉浏览器这个 cookie 什么时候过期，expires 是 UTC 格式时间，maxAge 是 cookie 多久后过期的相对时间。当不设置这两个选项时，会产生 session cookie，session cookie 是 transient 的，当用户关闭浏览器时，就被清除。一般用来保存 session 的 session_id。
* secure：当 secure 值为 true 时，cookie 在 HTTP 中是无效，在 HTTPS 中才有效。
* httpOnly：浏览器不允许脚本操作 document.cookie 去更改 cookie。一般情况下都应该设置这个为 true，这样可以避免被 xss 攻击拿到 cookie。

### express 中的 cookie

express 在 4.x 版本之后，session管理和cookies等许多模块都不再直接包含在express中，而是需要单独添加相应模块。

express4 中操作 cookie 使用 `cookie-parser` 模块(https://github.com/expressjs/cookie-parser )。

### session

cookie 虽然很方便，但是使用 cookie 有一个很大的弊端，cookie 中的所有数据在客户端就可以被修改，数据非常容易被伪造，那么一些重要的数据就不能存放在 cookie 中了，而且如果 cookie 中数据字段太多会影响传输效率。为了解决这些问题，就产生了 session，session 中的数据是保留在服务器端的。

session 的运作通过一个 `session_id` 来进行。`session_id` 通常是存放在客户端的 cookie 中，比如在 express 中，默认是 `connect.sid` 这个字段，当请求到来时，服务端检查 cookie 中保存的 `session_id` 并通过这个 `session_id` 与服务器端的 session data 关联起来，进行数据的保存和修改。

session 可以存放在 1）内存、2）cookie本身、3）redis 或 memcached 等缓存中，或者4）数据库中。线上来说，缓存的方案比较常见，存数据库的话，查询效率相比前三者都太低，不推荐；

express 中操作 session 要用到 `express-session` (https://github.com/expressjs/session ) 这个模块，主要的方法就是 `session(options)`，其中 options 中包含可选参数，主要有：

* name: 设置 cookie 中，保存 session 的字段名称，默认为 `connect.sid` 。
* store: session 的存储方式，默认存放在内存中，也可以使用 redis，mongodb 等。express 生态中都有相应模块的支持。
* secret: 通过设置的 secret 字符串，来计算 hash 值并放在 cookie 中，使产生的 signedCookie 防篡改。
* cookie: 设置存放 session id 的 cookie 的相关选项，默认为
    * (default: { path: '/', httpOnly: true, secure: false, maxAge: null })
* genid: 产生一个新的 session_id 时，所使用的函数， 默认使用 `uid2` 这个 npm 包。
* rolling: 每个请求都重新设置一个 cookie，默认为 false。
resave: 即使 session 没有被修改，也保存 session 值，默认为 true。

###  在内存中存储 session

`express-session` 默认使用内存来存 session，对于开发调试来说很方便。

```
var express = require('express');
// 首先引入 express-session 这个模块
var session = require('express-session');

var app = express();
app.listen(5000);

// 按照上面的解释，设置 session 的可选参数
app.use(session({
  secret: 'recommand 128 bytes random string', // 建议使用 128 个字符的随机字符串
  cookie: { maxAge: 60 * 1000 }
}));

app.get('/', function (req, res) {

  // 检查 session 中的 isVisit 字段
  // 如果存在则增加一次，否则为 session 设置 isVisit 字段，并初始化为 1。
  if(req.session.isVisit) {
    req.session.isVisit++;
    res.send('<p>第 ' + req.session.isVisit + '次来此页面</p>');
  } else {
    req.session.isVisit = 1;
    res.send("欢迎第一次来这里");
    console.log(req.session);
  }
});
```

###  在 redis 中存储 session

session 存放在内存中不方便进程间共享，因此可以使用 redis 等缓存来存储 session。

假设你的机器是 4 核的，你使用了 4 个进程在跑同一个 node web 服务，当用户访问进程1时，他被设置了一些数据当做 session 存在内存中。而下一次访问时，他被负载均衡到了进程2，则此时进程2的内存中没有他的信息，认为他是个新用户。这就会导致用户在我们服务中的状态不一致。

使用 redis 作为缓存，可以使用 `connect-redis` 模块(https://github.com/tj/connect-redis )来得到 redis 连接实例，然后在 session 中设置存储方式为该实例。

```
var express = require('express');
var session = require('express-session');
var redisStore = require('connect-redis')(session);

var app = express();
app.listen(5000);

app.use(session({
  // 假如你不想使用 redis 而想要使用 memcached 的话，代码改动也不会超过 5 行。
  // 这些 store 都遵循着统一的接口，凡是实现了那些接口的库，都可以作为 session 的 store 使用，比如都需要实现 .get(keyString) 和 .set(keyString, value) 方法。
  // 编写自己的 store 也很简单
  store: new redisStore(),
  secret: 'somesecrettoken'
}));

app.get('/', function (req, res) {
  if(req.session.isVisit) {
    req.session.isVisit++;
    res.send('<p>第 ' + req.session.isVisit + '次来到此页面</p>');
  } else {
    req.session.isVisit = 1;
    res.send('欢迎第一次来这里');
  }
});
```

### 各种存储的利弊

上面我们说到，session 的 store 有四个常用选项：1）内存 2）cookie 3）缓存 4）数据库

其中，开发环境存内存就好了。一般的小程序为了省事，如果不涉及状态共享的问题，用内存 session 也没问题。但内存 session 除了省事之外，没有别的好处。

cookie session 我们下面会提到，现在说说利弊。用 cookie session 的话，是不用担心状态共享问题的，因为 session 的 data 不是由服务器来保存，而是保存在用户浏览器端，每次用户访问时，都会主动带上他自己的信息。当然在这里，安全性之类的，只要遵照最佳实践来，也是有保证的。它的弊端是增大了数据量传输，利端是方便。

缓存方式是最常用的方式了，即快，又能共享状态。相比 cookie session 来说，当 session data 比较大的时候，可以节省网络传输。推荐使用。

数据库 session。除非你很熟悉这一块，知道自己要什么，否则还是老老实实用缓存吧。

### signedCookie

cookie 虽然很方便，但是使用 cookie 有一个很大的弊端，cookie 中的所有数据在客户端就可以被修改，数据非常容易被伪造

其实不是这样的，那只是为了方便理解才那么写。要知道，计算机领域有个名词叫 **签名**，专业点说，叫 **信息摘要算法**。

而如果我们签个名，比如把 `dotcom_user` 的值跟我的 `secret_string` 做个 sha1

```
sha1('this_is_my_secret_and_fuck_you_all' + 'alsotang') === '4850a42e3bc0d39c978770392cbd8dc2923e3d1d'
```

然后把 cookie 变成这样

```
{
  dotcom_user: 'alsotang',
  'dotcom_user.sig': '4850a42e3bc0d39c978770392cbd8dc2923e3d1d',
}
```

这样一来，用户就没法伪造信息了。一旦它更改了 cookie 中的信息，则服务器会发现 hash 校验的不一致。

### session cookie

初学者容易犯的一个错误是，忘记了 session_id 在 cookie 中的存储方式是 session cookie。即，当用户一关闭浏览器，浏览器 cookie 中的 session_id 字段就会消失。

常见的场景就是在开发用户登陆状态保持时。

假如用户在之前登陆了你的网站，你在他对应的 session 中存了信息，当他关闭浏览器再次访问时，你还是不懂他是谁。所以我们要在 cookie 中，也保存一份关于用户身份的信息。

```
{username: 'alsotang', age: 22, company: 'alibaba', location: 'hangzhou'}
```

我们可以考虑把这四个字段的信息都存在 session 中，而在 cookie，我们用 signedCookies 来存个 username。

## 《使用 promise 替代回调函数》

### promise基本概念

先学习promise的基本概念。

* promise只有三种状态，未完成，完成(fulfilled)和失败(rejected)。
* promise的状态可以由未完成转换成完成，或者未完成转换成失败。
* promise的状态转换只发生一次

promise有一个`then`方法，`then`方法可以接受3个函数作为参数。前两个函数对应promise的两种状态`fulfilled`, `rejected`的回调函数。第三个函数用于处理进度信息。

```
promiseSomething().then(function(fulfilled){
        //当promise状态变成fulfilled时，调用此函数
    },function(rejected){
        //当promise状态变成rejected时，调用此函数
    },function(progress){
        //当返回进度信息时，调用此函数
    });
```

学习一个简单的例子：

```
var Q = require('q');
var defer = Q.defer();
/**
 * 获取初始promise
 * @private
 */
function getInitialPromise() {
  return defer.promise;
}
/**
 * 为promise设置三种状态的回调函数
 */
getInitialPromise().then(function(success){
    console.log(success);
},function(error){
    console.log(error);
},function(progress){
    console.log(progress);
});
defer.notify('in progress');//控制台打印in progress
defer.resolve('resolve');   //控制台打印resolve
defer.reject('reject');     //没有输出。promise的状态只能改变一次
```

以上的代码，通过 `defer.resolve('resolve)` 来触发 `then` 中的 `function(success)` 方法。

### promise的传递

`then`方法会返回一个promise，在下面这个例子中，我们用outputPromise指向then返回的promise。

```
var outputPromise = getInputPromise().then(function (fulfilled) {
    }, function (rejected) {
    });
```

现在outputPromise就变成了受 `function(fulfilled)` 或者 `function(rejected)`控制状态的promise了。怎么理解这句话呢？

当`function(fulfilled)`或者`function(rejected)`返回一个值，比如一个字符串，数组，对象等等，那么`outputPromise`的状态就会变成`fulfilled`。

```
var Q = require('q');
var defer = Q.defer();
/**
 * 通过defer获得promise
 * @private
 */
function getInputPromise() {
    return defer.promise;
}

/**
 * 当inputPromise状态由未完成变成fulfil时，调用function(fulfilled)
 * 当inputPromise状态由未完成变成rejected时，调用function(rejected)
 * 将then返回的promise赋给outputPromise
 * function(fulfilled) 和 function(rejected) 通过返回字符串将outputPromise的状态由
 * 未完成改变为fulfilled
 * @private
 */
var outputPromise = getInputPromise().then(function(fulfilled){
    return 'fulfilled';
},function(rejected){
    return 'rejected';
});

/**
 * 当outputPromise状态由未完成变成fulfil时，调用function(fulfilled)，控制台打印'fulfilled: fulfilled'。
 * 当outputPromise状态由未完成变成rejected, 调用function(rejected), 控制台打印'rejected: rejected'。
 */
outputPromise.then(function(fulfilled){
    console.log('fulfilled: ' + fulfilled);
},function(rejected){
    console.log('rejected: ' + rejected);
});

/**
 * 将inputPromise的状态由未完成变成rejected
 */
defer.reject(); //输出 fulfilled: rejected

/**
 * 将inputPromise的状态由未完成变成fulfilled
 */
//defer.resolve(); //输出 fulfilled: fulfilled
```

当`function(fulfilled)`或者`function(rejected)`抛出异常时，那么`outputPromise`的状态就会变成`rejected`

当`function(fulfilled)`或者`function(rejected)`返回一个promise时，`outputPromise`就会成为这个新的promise.
这样做有什么意义呢? 主要在于聚合结果(`Q.all`)，管理延时，异常恢复等等

比如说我们想要读取一个文件的内容，然后把这些内容打印出来。可能会写出这样的代码：

```
//错误的写法
var outputPromise = getInputPromise().then(function(fulfilled){
    fs.readFile('test.txt','utf8',function(err,data){
        return data;
    });
});
```

然而这样写是错误的，因为function(fulfilled)并**没有返回任何值(指的是没有任何同步的返回值)**。需要下面的方式:

```
var Q = require('q');
var fs = require('fs');
var defer = Q.defer();

/**
 * 通过defer获得promise
 * @private
 */
function getInputPromise() {
    return defer.promise;
}

/**
 * 当inputPromise状态由未完成变成fulfil时，调用function(fulfilled)
 * 当inputPromise状态由未完成变成rejected时，调用function(rejected)
 * 将then返回的promise赋给outputPromise
 * function(fulfilled)将新的promise赋给outputPromise
 * 未完成改变为reject
 * @private
 */
var outputPromise = getInputPromise().then(function(fulfilled){
    var myDefer = Q.defer();
    fs.readFile('test.txt','utf8',function(err,data){
        if(!err && data) {
            myDefer.resolve(data);
        }
    });
    return myDefer.promise;
},function(rejected){
    throw new Error('rejected');
});

/**
 * 当outputPromise状态由未完成变成fulfil时，调用function(fulfilled)，控制台打印test.txt文件内容。
 *
 */
outputPromise.then(function(fulfilled){
    console.log(fulfilled);
},function(rejected){
    console.log(rejected);
});

/**
 * 将inputPromise的状态由未完成变成rejected
 */
//defer.reject();

/**
 * 将inputPromise的状态由未完成变成fulfilled
 */
defer.resolve(); //控制台打印出 test.txt 的内容
```

### 方法传递

方法传递的含义是当一个状态没有响应的回调函数，就会沿着then往下找。

* 没有提供`function(rejected)`

```
var outputPromise = getInputPromise().then(function(fulfilled){})
```

* 没有提供`function(fulfilled)`

```
var outputPromise = getInputPromise().then(null,function(rejected){})
```

* 可以使用`fail(function(error))`来专门针对错误处理，而不是使用`then(null,function(error))`

```
 var outputPromise = getInputPromise().fail(function(error){})
```

* 可以使用`progress(function(progress))`来专门针对进度信息进行处理，而不是使用 `then(function(success){},function(error){},function(progress){})`

### promise链

promise链提供了一种让函数顺序执行的方法。

```
var Q = require('q');
var defer = Q.defer();

//一个模拟数据库
var users = [{'name':'andrew','passwd':'password'}];

function getUsername() {
return defer.promise;
}

function getUser(username){
    var user;
    users.forEach(function(element){
        if(element.name === username) {
            user = element;
        }
    });
    return user;
}

//promise链
getUsername().then(function(username){
 return getUser(username);
}).then(function(user){
 console.log(user);
});

defer.resolve('andrew');
```

我们通过两个then达到让函数顺序执行的目的。

then的数量其实是没有限制的。当然，then的数量过多，要手动把他们链接起来是很麻烦的。

这时我们需要用代码来动态制造promise链

```
var funcs = [foo,bar,baz,qux]
var result = Q(initialVal)
funcs.forEach(function(func){
    result = result.then(func)
})
return result
```

### promise组合

我们可以通过`Q.all([promise1,promise2...])`将多个promise组合成一个promise返回。 注意：

* 当all里面所有的promise都`fulfill`时，`Q.all`返回的promise状态变成`fulfill`
* 当任意一个promise被`reject`时，`Q.all`返回的promise状态立即变成`reject`

通常，对于一个promise链，有两种结束的方式。第一种方式是返回最后一个promise

如 `return foo().then(bar);`

第二种方式就是通过done来结束promise链

如 `foo().then(bar).done()`

为什么需要通过done来结束一个promise链呢? 如果在我们的链中有错误没有被处理，那么在一个正确结束的promise链中，这个**没被处理的错误会通过异常抛出**。

```
var Q = require('q');
/**
 *@private
 */
function getPromise(msg,timeout,opt) {
    var defer = Q.defer();
    setTimeout(function(){
    console.log(msg);
        if(opt)
            defer.reject(msg);
        else
            defer.resolve(msg);
    },timeout);
    return defer.promise;
}
/**
 *没有用done()结束的promise链
 *由于getPromse('2',2000,'opt')返回rejected, getPromise('3',1000)就没有执行
 *然后这个异常并没有任何提醒，是一个潜在的bug
 */
getPromise('1',3000)
    .then(function(){return getPromise('2',2000,'opt')})
    .then(function(){return getPromise('3',1000)});
/**
 *用done()结束的promise链
 *有异常抛出
 */
getPromise('1',3000)
    .then(function(){return getPromise('2',2000,'opt')})
    .then(function(){return getPromise('3',1000)})
    .done();
```

## 《何为 connect 中间件》

### HTTP

Nodejs 的经典 httpServer 代码

```
var http = require('http');

var server = http.createServer(requestHandler);
function requestHandler(req, res) {
  res.end('hello visitor!');
}
server.listen(3000);
```

然而，业务逻辑越来越复杂，会出发展成30个回调逻辑，那么就出现了30个 }); 及30个 err异常。更严重的是，到时候写代码根本看不清自己写的逻辑在30层中的哪一层，极其容易出现 多次返回 或返回地方不对等问题，这就是 回调金字塔 问题了。

大多数同学应该能想到解决回调金字塔的办法，朴灵的《深入浅出Node.js》里讲到的三种方法。下面列举了这三种方法加上ES6新增的Generator，共四种解决办法。

* EventProxy —— 事件发布订阅模式(第四课讲到)
* BlueBird —— Promise方案(第十七课讲到)
* Async —— 异步流程控制库(第五课讲到)
* Generator —— ES6原生Generator

而Connect和Express用的是 类似异步流程控制的思想 。

我们动手实现一个类似的链式调用，其中 `funlist` 更名为 `middlewares`、`callback` 更名为 `next`，码如下：

```
var middlewares = [
  function fun1(req, res, next) {
    parseBody(req, function(err, body) {
      if (err) return next(err);
      req.body = body;
      next();
    });
  },
  function fun2(req, res, next) {
    checkIdInDatabase(req.body.id, function(err, rows) {
      if (err) return next(err);
      res.dbResult = rows;
      next();
    });
  },
  function fun3(req, res, next) {
    if (res.dbResult && res.dbResult.length > 0) {
      res.end('true');
    }
    else {
      res.end('false');
    }
    next();
  }
]

function requestHandler(req, res) {
  var i=0;

  //由middlewares链式调用
  function next(err) {

    if (err) {
      return res.end('error:', err.toString());
    }

    if (i<middlewares.length) {
      middlewares[i++](req, res, next);
    } else {
      return ;
    }
  }

  //触发第一个middleware
  next();
}
```

上面用`middlewares+next`完成了业务逻辑的 链式调用，而middlewares里的每个函数，都是一个 **中间件**。

整体思路是：

1. 将所有 处理逻辑函数(中间件) 存储在一个list中；
2. 请求到达时 循环调用 list中的 处理逻辑函数(中间件)；

### Connect的实现

Connect的思想跟上面阐述的思想基本一样，先将处理逻辑存起来，然后循环调用。

Connect中主要有五个函数 PS: Connect的核心代码是200+行，建议对照源码看下面的函数介绍。

```
函数名	                  作用
createServer	包装httpServer形成app
listen	             监听端口函数
use	          向middlewares里面放入业务逻辑
handle	    上一章的requestHandler函数增强版
call	           业务逻辑的真正执行者
```

### Express

大家都知道Express是Connect的升级版。

Express不只是Connect的升级版，它还封装了很多对象来方便业务逻辑处理。Express里的Router是Connect的升级版。

Express大概可以分为几个模块

```
模块	             描述
router	    路由模块是Connect升级版
request	    经过Express封装的req对象
response	经过Express封装的res对象
application	app上面的各种默认设置
```

































