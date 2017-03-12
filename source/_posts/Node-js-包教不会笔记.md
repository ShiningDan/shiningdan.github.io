---
title: Node.js 包教不会笔记
date: 2017-03-12 21:16:46
categories: coding
tags:
  - Node.js
---

本文是笔者在学习[《Node.js 包教不包会》 -- by alsotang](https://github.com/alsotang/node-lessons) 中的例子的笔记。

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







