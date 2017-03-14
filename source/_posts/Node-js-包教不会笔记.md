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

## 











































