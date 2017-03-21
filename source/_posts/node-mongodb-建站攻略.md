---
title: node+mongodb 建站攻略
date: 2017-03-20 21:35:20
categories: coding
tags:
  - Node.js
  - express
  - mongodb
  - jade
---

本笔记是根据 [node+mongodb 建站攻略](http://www.imooc.com/learn/75) 教程所记录的笔记。如有需要，可以参考原视频。

## 项目前期准备

后端使用的是 node.js + express。模板引擎是 jade，对时间和日期的格式化使用的是 moment.js

前端使用的是 jQuery 和 Bootstrap。

本地环境使用的是 less + cssmn + jsHint + UglifyJS + mocha + grunt

[express](http://www.expressjs.com.cn/)


开发的流程为：

![](http://ojt6zsxg2.bkt.clouddn.com/ee3c9abb0323bdd59b1474c306fa2da2.png)

<!--more-->

## 项目前后端流程的打通

### 项目结构的初始化

```
npm install express --save
npm install jade --save
npm install mongoose --save
npm install bower -g --save
bower install boostrap
```

### 入口文件编码

app.js:

```
let express = require('express');

let app = express();

app.set('view engine', 'jade');
app.set('port', 3000);

app.get('/', function(req, res) {
    res.render('index', {title: 'imooc'});
})
```

index.jade:

```
<!DOCTYPE html>
html
    head
    body
        title #{title}
```

项目的结构为：

![](http://ojt6zsxg2.bkt.clouddn.com/4da689f7513ed364d721dbf91d5518e7.png)

### 创建四个 jade 视图以及入口文件处理

入口文件如下：

app.js
```
let express = require('express');

let port = process.env.PORT || 3000;
let app = express();

app.set('views', './views');  // 应用的视图目录
app.set('view engine', 'jade'); // 应用的视图引擎
console.log('listen to port', port);
app.listen(port);


app.get('/', function(req, res) {
    res.render('index', {
        title: 'shiningdan 首页'
    });
});

app.get('/admin/list', function(req, res) {  //访问 /admin/list 返回 list.jade 渲染后的效果
    res.render('list', {        // list 是从 views 里面找到的 list.jade
        title: 'shiningdan 列表页'
    })
})

app.get('/admin/movie', function(req, res) {
    res.render('admin', {
        title: 'shiningdan 后台录入页'
    })
})

app.get('/admin/:id', function(req, res) {   // 访问 /admin/3 返回 detail.jade 渲染后的效果
    res.render('detail', {
        title: 'shiningdan 详情页'
    })
})
```

模板文件如下：

```
<!DOCTYPE html>
html(lang="en")
head
    meta(charset="UTF-8")
    meta(name="viewport", content="width=device-width, initial-scale=1.0")
    meta(http-equiv="X-UA-Compatible", content="ie=edge")
    title #{title}
body
    h1 #{title}
```

当访问 http://127.0.0.1:3000/admin/list 等 url 的时候，会找到 list.jade，渲染模板并且返回 html 文件。

### 伪造模板数据跑通前后端交互流程









