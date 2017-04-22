---
title: Gulp 笔记
date: 2017-04-22 19:55:44
categories: coding
tags:
  - Gulp
---

如果需要参考的时候，可以去官网[gulp 技巧集](http://www.gulpjs.com.cn/docs/recipes/) 查看 Gulp 配合插件可以实现什么功能。

[Gulp API 文档](http://www.gulpjs.com.cn/docs/api/)

[Gulp API 初探](http://aikin.me/2014/10/24/gulp-nodemon-livereload/)

<!--more-->

## Gulp 中常见的命令

例子：

```
var gulp = require('gulp');
var sass = require('gulp-sass')；
gulp.task('sass', function() {
    gulp.src('./scss/*.scss')
        .pipe(sass())
        .pipe(gulp.dest('./css'));
});
gulp.watch('./scss/*.scss', ['sass']);

```

### gulp.task()

`task()`方法是 gulp 用于定义一个具体任务的方法。如果需要执行任务，在终端执行 `gulp task-name`。

`task()` 方法的语法如:

```
语法：
gulp.task(name[, deps], fn)

参数：

name
　　类型：String， 指任务名，就像上述的eg-1例子的sass。

deps
　　类型：Array，指在跑当前任务时，对其它任务的依赖。也就是要执行当前任务，会先执行这些依赖的任务。如: gulp.task('demo', ['demo1', 'demo2'], function(){ });会先同时执行任务'demo1', 'demo2'，最后执行任务'demo'。
fn
　　类型：function，指运行任务时，要执行的具体操作的内容。
```

### gulp.src()

`src` 方法是指定源文件的输入路径，pipe有点像是封闭的“流水线”，某个产品经过上一个工序处理后，就转入下一个工序去处理，直到完成。也就是将上一步的输出转化下一步的输入的中间者。

`src()` 方法的语法如:

```
语法：
gulp.src(globs[, options])

参数：

globs
　　类型：String 或 Array，指定源文件的路径，可以是单个路径，也可以是个路径数组。
　　路径匹配支持通配符：
　　　　1. app.js　　　　指定具体文件
　　　　2. js/*　　　　　匹配 js 目录下所有的文件，不包括子文件夹
　　　　3. js/*.js　　　 匹配 js 目录下所有的扩展名为 .js 的文件，不包括子文件夹
　　　　4. js/*/*.js　　匹配 js 目录下第一层子文件夹里的扩展名为 .js 的文件
　　　　5. js/**/*.js　 匹配 js 目录下所有文件夹层次下扩展名为 .js 的文件
　　　　6. !js/try.js　 不包括 try.js 文件，在前五条文件匹配模式前加!，就忽略掉相应的文件

options
　　类型：Object，有3个属性buffer，read，base。
　　　options.butter
　　　　类型：Boolean，默认：true
　　　　gulp-api 上描述到，如设置为false，返回的文件内容将会以数据流的形式体现，而不是数据
　　　　块的形式。还提示到有可能一些插件没有实现支持数据流的形式。
　　　　(表示不太明白，有待研究。-_-|||)
　　　options.read
　　　　类型：Boolean，默认：true
　　　　返回的文件内容为null，不执行读取文件操作。
　　　options.base
　　　　类型：String
　　　　设置输出路径以某个路径的某个组成部分为基础向后拼接。具体例子可以参考 gulp-api。
```

### gulp.dest()

`dest()` 方法是指定被处理完的文件的输出路径，就像例子里的 `gulp.dest('./css')`意思就是将编译完成的 css 文件保存到 `/css`目录中。

### gulp.watch()

`watch()` 方法是用于监听文件变化，文件一修改就会执行指定的任务。在例子中，通过监听`./scss/*.scss`文件，一旦文件发生修改就会执行任务sass。

```
语法：
gulp.watch(glob [, opts], tasks) or gulp.watch(glob [, opts, cb])

参数：

glob
　　类型：String 或 Array，指定源文件的路径，可以是单个路径，也可以是个路径数组。路径匹配和上述gulp.src()方法路径匹配的模式一样。

opts
　　类型：Object，有4个属性interval，debounceDelay，mode，cwd。
　　具体可以参考gulp-api这里就不一一介绍了。

tasks
　　类型：Array，监听到文件变化后，要被执行的任务的名字组成的数组。

cb(event)
　　类型：Function，监听到变化后，回调的函数。会传递出一个对象类型的event参数。
　　　event.type
　　　　类型：String，表示操作的类型：added, changed or deleted
　　　event.path
　　　　类型：String，被修改文件的路径。
```

## 自启动服务器

npm包 `gulp-nodemon` 和npm包 `nodemon` 的功能差不多一样，只是支持了与gulp的结合。根据npm包`nodemon` 官网的介绍，`nodemon` 的作用类似于 `node`。我们通常在命令行中输入 `node app.js` 就能启动某个本地的服务器（app.js 是某个文件名，名字由程序员决定）。如果我们输入 `nodemon app.js`同样也能启动。不同的是，`nodemon`能够检测 `app.js` 所在目录下的文件，如果有改变，就会自动重新启动 `app.js`。而`gulp-nodemon` 包的使用也类似的

安装 `gulp-nodemon`(https://github.com/JacksonGariety/gulp-nodemon)：

```
npm install --save-dev gulp-nodemon
```

在 `gulpfile.js` 中添加启动 `gulp-nodemon` 的代码：

```
let gulp = require('gulp');
let nodemon = require('gulp-nodemon');

gulp.task('server', function() {
  nodemon({
    script: './app/app.js',
    ignore: ['README.md', 'node_modules/**', '.DS_Store'], 
    env: { 'NODE_ENV': 'development' },
  })
})
```

使用 `gulp server` 就可以监控文件的修改，然后重启 `./app/app.js`










