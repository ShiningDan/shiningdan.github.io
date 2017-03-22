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

本地环境使用的是 less + cssmin + jsHint + UglifyJS + mocha + grunt

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

在 app.js 中需要获取静态资源，新版express4中，要独立安装`static`，`npm install serve-static --save`

在app.js中，

```
var serveStatic = require('serve-static')

app.use(serveStatic('bower_components')
```

bodyParser 已经不再与Express捆绑，需要独立安装。

命令行执行：`npm install body-parser --save`

程序中修改：

```
var bodyParser = require('body-parser')

app.use(bodyParser.urlencoded())
```

最终的项目目录如下：

![](http://ojt6zsxg2.bkt.clouddn.com/c865bfd5e8457711deba5c25763eff33.png)

其中，每一部分的内容为：

app.js

```
let express = require('express');
let path = require('path');
let serveStatic = require('serve-static');
let bodyPaerser = require('body-parser');

let port = process.env.PORT || 3000;
let app = express();

app.set('views', './views/pages/');  // 应用的视图目录
app.set('view engine', 'jade'); // 应用的视图引擎
app.use(serveStatic('bower_components'));
app.use(bodyPaerser.urlencoded({extended: true}));
console.log('listen to port', port);
app.listen(port);


app.get('/', function(req, res) {
    res.render('index', {
        title: 'shiningdan 首页',
        movies: [
            {
                title: '美女与野兽',
                _id: 1,
                poster: 'https://img3.doubanio.com/view/movie_poster_cover/lpst/public/p2417948644.webp'
            },
            {
                title: '美女与野兽',
                _id: 2,
                poster: 'https://img3.doubanio.com/view/movie_poster_cover/lpst/public/p2417948644.webp'
            },
            {
                title: '美女与野兽',
                _id: 3,
                poster: 'https://img3.doubanio.com/view/movie_poster_cover/lpst/public/p2417948644.webp'
            },
            {
                title: '美女与野兽',
                _id: 4,
                poster: 'https://img3.doubanio.com/view/movie_poster_cover/lpst/public/p2417948644.webp'
            },
            {
                title: '美女与野兽',
                _id: 5,
                poster: 'https://img3.doubanio.com/view/movie_poster_cover/lpst/public/p2417948644.webp'
            },
            {
                title: '美女与野兽',
                _id: 6,
                poster: 'https://img3.doubanio.com/view/movie_poster_cover/lpst/public/p2417948644.webp'
            },
        ],
    });
});

app.get('/movie/:id', function(req, res) {   // 访问 /admin/3 返回 detail.jade 渲染后的效果
    res.render('detail', {
        title: 'shiningdan 详情页',
        movie: {
            title: '美女与野兽',
            doctor: '比尔·康顿',
            country: ' 美国',
            language: '英语',
            year: '2017',
            flash: 'https://movie.douban.com/trailer/211279/#content/swf/movie_player_1.4.4.swf',
            poster: 'https://img3.doubanio.com/view/movie_poster_cover/lpst/public/p2417948644.jpg',
            summary: '《美女与野兽》根据迪士尼1991年经典动画片及闻名全球的经典童话改编，讲述了少女贝儿的奇幻旅程——为了解救触怒野兽的父亲，勇敢善良的她只身一人来到古堡，代替父亲被囚禁其中。贝儿克服了恐惧，和城堡里的魔法家具们成为了朋友，也渐渐发现野兽其实是受了诅咒的王子，他可怖的外表下藏着一颗善良温柔的内心；这个故事也带领观众明白——美不仅仅是外表，更重要的是内心。',
        }
    })
})


app.get('/admin/movie', function(req, res) {
    res.render('admin', {
        title: 'shiningdan 后台录入页',
        movie: {
            title: '',
            doctor: '',
            country: '',
            language: '',
            year: '',
            flash: '',
            poster: '',
            summary: '',
        }
    })
})

app.get('/admin/list', function(req, res) {  //访问 /admin/list 返回 list.jade 渲染后的效果
    res.render('list', {        // list 是从 views 里面找到的 list.jade
        title: 'shiningdan 列表页',
        movies: [{
            title: '美女与野兽',
            _id: 1,
            doctor: '比尔·康顿',
            country: ' 美国',
            lanuage: '英语',
            year: '2017',
            flash: 'https://movie.douban.com/trailer/211279/#content/swf/movie_player_1.4.4.swf',
            poster: 'https://img3.doubanio.com/view/movie_poster_cover/lpst/public/p2417948644.jpg',
            summary: '《美女与野兽》根据迪士尼1991年经典动画片及闻名全球的经典童话改编，讲述了少女贝儿的奇幻旅程——为了解救触怒野兽的父亲，勇敢善良的她只身一人来到古堡，代替父亲被囚禁其中。贝儿克服了恐惧，和城堡里的魔法家具们成为了朋友，也渐渐发现野兽其实是受了诅咒的王子，他可怖的外表下藏着一颗善良温柔的内心；这个故事也带领观众明白——美不仅仅是外表，更重要的是内心。',
        }]
    })
})
```

head.jade

```
link(href='/bootstrap/dist/css/bootstrap.min.css', rel='stylesheet', type='text/css')
script(src='/jquery/dist/jquery.min.js')
script(src='/bootstrap/dist/js/bootstrap.min.js')
```

header.jade

```
.container
    .row
        .page-header
            h1 #{title}
            small 重度科幻迷
```

layout.jade

```
<!DOCTYPE html>
html(lang="en")
head
    meta(charset="UTF-8")
    meta(name="viewport", content="width=device-width, initial-scale=1.0")
    meta(http-equiv="X-UA-Compatible", content="ie=edge")
    title #{title}
    include ./includes/head
body
    include ./includes/header
    block content
```

admin.jade

```
extends ../layout

block content
    .container
        .row
            form.form-horizontal(method='post', action='/admin/movie')
                .from-group
                    label.col-sm-2.control-label(for='inputTitle') 电影名字
                    .col-sm-10
                        input#inputTitle.form-control(type='text', name='movie[title]', value='#{movie.title}')
                .from-group
                    label.col-sm-2.control-label(for='inputDoctor') 电影导演
                    .col-sm-10
                        input#inputDoctor.form-control(type='text', name='movie[doctor]', value='#{movie.doctor}')
                .from-group
                    label.col-sm-2.control-label(for='inputCountry') 国家
                    .col-sm-10
                        input#inputCountry.form-control(type='text', name='movie[country]', value='#{movie.country}')
                .from-group
                    label.col-sm-2.control-label(for='inputLanguage') 语种
                    .col-sm-10
                        input#inputLanguage.form-control(type='text', name='movie[language]', value='#{movie.language}')
                .from-group
                    label.col-sm-2.control-label(for='inputPoster') 海报地址
                    .col-sm-10
                        input#inputPoster.form-control(type='text', name='movie[poster]', value='#{movie.poster}')
                .from-group
                    label.col-sm-2.control-label(for='inputYear') 年份
                    .col-sm-10
                        input#inputYear.form-control(type='text', name='movie[year]', value='#{movie.year}')
                .from-group
                    label.col-sm-2.control-label(for='inputFlash') Flash
                    .col-sm-10
                        input#inputFlash.form-control(type='text', name='movie[flash]', value='#{movie.flash}')
                .from-group
                    label.col-sm-2.control-label(for='inputSummary') 简介
                    .col-sm-10
                        input#inputSummary.form-control(type='text', name='movie[summary]', value='#{movie.summary}')
                .form-group
                    .col-sm-offset-2.col-sm-10
                    button.btn.btn-default(type='submit') 录入
```

detail.jade

```
extends ../layout

block content
    .container
        .row
            .col-md-7
                embed(src='#{movie.flash}', allowFullScreen='true', quality='high', width='720', height='600', align='middle', type='application/x-shockwave-flash')
            .col-md-5
                dl.dl-horizontal
                    dt 电影名字
                    dd #{movie.title}
                    dt 导演
                    dd #{movie.doctor}
                    dt 国家
                    dd #{movie.country}
                    dt 语言
                    dd #{movie.language}
                    dt 上映年份
                    dd #{movie.year}
                    dt 简介
                    dd #{movie.summary}
```

index.jade

```
extends ../layout

block content
    .container
        .row
            each item in movies
                .col-md-2
                    .thumbnail
                        a(href='/movie/#{item._id}')
                            img(src='#{item.poster}', alt='#{item.title}')
                        .caption
                            h3 #{item.title}
                            p: a.btn.btn-primary(href='/movie/#{item._id}', role='button')
                                观看预告片
```

list.jade

```
extends ../layout

block content
    .container
        .row
             table.table.table-hover.table-bordered
                thead
                    tr
                        th 电影名字
                        th 导演
                        th 国家
                        th 上映年份
                        //- th 录入时间
                        th 查看
                        th 更新
                        th 删除
                    tbody
                        each item in movies
                            tr(class='item-id-#{item._id}')
                                td #{item.title}
                                td #{item.doctor}
                                td #{item.country}
                                td #{item.year}
                                //- td #{moment(item.meta.createAt).format('MM/DD/YYYY')}
                                td: a(target='_blank', href='../movie/#{item._id}') 查看
                                td: a(target='_blank', href='../admin/update/#{item._id}') 修改
                                td
                                    button.btn.btn-danger.del(type='button', data-id='#{item._id}') 删除
```

## 项目数据库的实现

### mongodb 模式模型设计及编码

使用 mongoose 对 mongodb 进行操作

* Schema 模式定义：进行数据库每一个字段的定义

![](http://ojt6zsxg2.bkt.clouddn.com/0a6ed1fa1a328be7d0d20b580681401c.png)

* Model 编译模型，生成构造函数

![](http://ojt6zsxg2.bkt.clouddn.com/8c46e8bd40ac760f4a1e88a54f568337.png)

* Document：文档实例化

![](http://ojt6zsxg2.bkt.clouddn.com/d0b2004c48d1114beab8314ad3025353.png)

### 编写数据库 Schema 定义

model.js

```
let mongoose = require('mongoose');

let MovieSchema = new mongoose.Schema({
    doctor: String,
    title: String,
    language: String,
    summary: String,
    flash: String,
    poster: String,
    year: String,
    meta: {
        createAt: {
            type: DAte,
            default: Data.now(),
        },
        updateAt: {
            type: Data,
            default: Date.now(),
        },
    },
});

MovieSchema.pre('save', function(next) {
    if (this.isNew) {
        this.meta.createAt = this.meta.updateAt = Data.now();
    } else {
        this.meta.updateAt = Data.now();
    }
});

MovieSchema.statics = {
    fetch: function(cb) {
        return this.find({})
        .sort('meta.updataAt')
        .exec(cb);
    },
    findById: function(id, cb) {
        return this.findOne({_id, id})
        .exec(cb);
    },
};

module.exports = MovieSchema;
```

### 编写数据库编译模型，生成构造函数

```
let mongoose = require('mongoose');
let MovieSchema = require('../schemas/model');
let Movie = mongoose.model('Moive', MovieSchema);

module.exports = Movie;
```

### 编写数据库交互代码

