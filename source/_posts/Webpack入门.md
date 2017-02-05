---
title: Webpack入门
date: 2017-02-05 16:56:42
categories: coding
tags:
  - Webpack
---

本文是我在学习 Webpack 时的记录，用于个人查询总结。在学习的过程中参考过很多别人的文章，如有需要，可以根据链接详细学习：

* [Webpack 中文指南](http://webpackdoc.com)
* [一小时包教会 —— webpack 入门指南](http://www.w2bc.com/Article/50764)
* [Webpack 官网](http://webpack.github.io/docs/)
* [阮一峰 Webpack demos](https://github.com/ruanyf/webpack-demos)

## 介绍

Webpack 是当下最热门的前端资源模块化管理和打包工具。它可以将许多松散的模块按照依赖和规则打包成符合生产环境部署的前端资源。还可以将按需加载的模块进行代码分隔，等到实际需要的时候再异步加载。通过 `loader` 的转换，任何形式的资源都可以视作模块，比如 CommonJs 模块、 AMD 模块、 ES6 模块、CSS、图片、 JSON、Coffeescript、 LESS 等。

## 前言

### 模块系统

#### 现状

伴随着移动互联的大潮，当今越来越多的网站已经从网页模式进化到了 Webapp 模式。它们运行在现代的高级浏览器里，使用 HTML5、 CSS3、 ES6 等更新的技术来开发丰富的功能，网页已经不仅仅是完成浏览的基本需求，并且webapp通常是一个单页面应用，每一个视图通过异步的方式加载，这导致页面初始化和使用过程中会加载越来越多的 JavaScript 代码，这给前端开发的流程和资源组织带来了巨大的挑战。

前端开发和其他开发工作的主要区别，首先是前端是基于多语言、多层次的编码和组织工作，其次前端产品的交付是基于浏览器，这些资源是通过增量加载的方式运行到浏览器端，如何在开发环境组织好这些碎片化的代码和资源，并且保证他们在浏览器端快速、优雅的加载和更新，就需要一个模块化系统，这个理想中的模块化系统是前端工程师多年来一直探索的难题。

#### 模块系统的演进

模块系统主要解决模块的定义、依赖和导出，先来看看已经存在的模块系统。

##### `<script>`标签

```
<script src="module1.js"></script>
<script src="module2.js"></script>
<script src="libraryA.js"></script>
<script src="module3.js"></script>
```

这是最原始的 JavaScript 文件加载方式，如果把每一个文件看做是一个模块，那么他们的接口通常是暴露在全局作用域下，也就是定义在 `window` 对象中，不同模块的接口调用都是一个作用域中，一些复杂的框架，会使用命名空间的概念来组织这些模块的接口，典型的例子如 YUI 库。

这种原始的加载方式暴露了一些显而易见的弊端：

* 全局作用域下容易造成变量冲突
* 文件只能按照 `<script>` 的书写顺序进行加载
* 开发人员必须主观解决模块和代码库的依赖关系
* 在大型项目中各种资源难以管理，长期积累的问题导致代码库混乱不堪

##### CommonJS

服务器端的 [Node.js](http://wiki.commonjs.org/wiki/CommonJS) 遵循 CommonJS规范，该规范的核心思想是允许模块通过 require 方法来同步加载所要依赖的其他模块，然后通过 exports 或 module.exports 来导出需要暴露的接口

```
require("module");
require("../file.js");
exports.doStuff = function() {};
module.exports = someValue;
```

优点：

* 服务器端模块便于重用
* NPM 中已经有将近20万个可以使用模块包
* 简单并容易使用

缺点：

* 同步的模块加载方式不适合在浏览器环境中，同步意味着阻塞加载，浏览器资源是异步加载的
* 不能非阻塞的并行加载多个模块

实现：

* 服务器端的 Node.js
* Browserify，浏览器端的 CommonJS 实现，可以使用 NPM 的模块，但是编译打包后的文件体积可能很大
* modules-webmake，类似Browserify，还不如 Browserify 灵活
wreq，Browserify 的前身

##### AMD

Asynchronous Module Definition 规范其实只有一个主要接口 `define(id?, dependencies?, factory)`，它要在声明模块的时候指定所有的依赖 `dependencies`，并且还要当做形参传到 `factory` 中，对于依赖的模块提前执行，依赖前置。

```
define("module", ["dep1", "dep2"], function(d1, d2) {
  return someExportedValue;
});
require(["module", "../file"], function(module, file) { /* ... */ });
```

优点：

* 适合在浏览器环境中异步加载模块
* 可以并行加载多个模块

缺点：

* 提高了开发成本，代码的阅读和书写比较困难，模块定义方式的语义不顺畅
* 不符合通用的模块化思维方式，是一种妥协的实现

实现：

* RequireJS
* curl

##### CMD

Common Module Definition 规范和 AMD 很相似，尽量保持简单，并与 CommonJS 和 Node.js 的 Modules 规范保持了很大的兼容性。

```
define(function(require, exports, module) {
  var $ = require('jquery');
  var Spinning = require('./spinning');
  exports.doSomething = ...
  module.exports = ...
})
```

优点：

* 依赖就近，延迟执行
* 可以很容易在 Node.js 中运行

缺点：

* 依赖 SPM 打包，模块的加载逻辑偏重

实现：

* Sea.js
coolie

##### ES6 模块

EcmaScript6 标准增加了 JavaScript 语言层面的模块体系定义。ES6 模块的设计思想，是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。CommonJS 和 AMD 模块，都只能在运行时确定这些东西。

```
import "jquery";
export function doStuff() {}
module "localModule" {}
```

优点：

* 容易进行静态分析
* 面向未来的 EcmaScript 标准

缺点：

* 原生浏览器端还没有实现该标准
* 全新的命令字，新版的 Node.js才支持

实现：

* Babel

##### 期望的模块系统

可以兼容多种模块风格，尽量可以利用已有的代码，不仅仅只是 JavaScript 模块化，还有 CSS、图片、字体等资源也需要模块化。

#### 前端模块加载


前端模块要在客户端中执行，所以他们需要增量加载到浏览器中。

模块的加载和传输，我们首先能想到两种极端的方式，一种是每个模块文件都单独请求，另一种是把所有模块打包成一个文件然后只请求一次。显而易见，每个模块都发起单独的请求造成了请求次数过多，导致应用启动速度慢；一次请求加载所有模块导致流量浪费、初始化过程慢。这两种方式都不是好的解决方案，它们过于简单粗暴。

**分块传输**，按需进行懒加载，在实际用到某些模块的时候再增量更新，才是较为合理的模块加载方案。

要实现模块的按需加载，就需要一个对整个代码库中的模块进行静态分析、编译打包的过程。

#### 所有资源都是模块

在上面的分析过程中，我们提到的模块仅仅是指JavaScript模块文件。然而，在前端开发过程中还涉及到样式、图片、字体、HTML 模板等等众多的资源。这些资源还会以各种方言的形式存在，比如 coffeescript、 less、 sass、众多的模板库、多语言系统（i18n）等等。

如果他们都可以视作模块，并且都可以通过 `require` 的方式来加载，将带来优雅的开发体验，比如：

```
require("./style.css");
require("./style.less");
require("./template.jade");
require("./image.png");
```

那么如何做到让 `require` 能加载各种资源呢？

#### 静态分析

在编译的时候，要对整个代码进行静态分析，分析出各个模块的类型和它们依赖关系，然后将不同类型的模块提交给适配的加载器来处理。比如一个用 LESS 写的样式模块，可以先用 LESS 加载器将它转成一个CSS 模块，在通过 CSS 模块把他插入到页面的 `<style>` 标签中执行。Webpack 就是在这样的需求中应运而生。

同时，为了能利用已经存在的各种框架、库和已经写好的文件，我们还需要一个模块加载的兼容策略，来避免重写所有的模块。

### 重复的轮子

#### 什么是 Webpack

Webpack 是一个模块打包器。它将根据模块的依赖关系进行静态分析，然后将这些模块按照指定的规则生成对应的静态资源。

![](http://webpackdoc.com/images/what-is-webpack.png)

#### 为什么重复造轮子

市面上已经存在的模块管理和打包工具并不适合大型的项目，尤其单页面 Web 应用程序。最紧迫的原因是如何在一个大规模的代码库中，维护各种模块资源的分割和存放，维护它们之间的依赖关系，并且无缝的将它们整合到一起生成适合浏览器端请求加载的静态资源。

这些已有的模块化工具并不能很好的完成如下的目标：

* 将依赖树拆分成按需加载的块
* 初始化加载的耗时尽量少
* 各种静态资源都可以视作模块
* 将第三方库整合成模块的能力
* 可以自定义打包逻辑的能力
* 适合大项目，无论是单页还是多页的 Web 应用

#### Webpack 的特点

Webpack 和其他模块化工具有什么区别呢？

##### 代码拆分

Webpack 有两种组织模块依赖的方式，同步和异步。异步依赖作为分割点，形成一个新的块。在优化了依赖树后，每一个异步区块都作为一个文件被打包。

##### Loader

Webpack 本身只能处理原生的 JavaScript 模块，但是 loader 转换器可以将各种类型的资源转换成 JavaScript 模块。这样，任何资源都可以成为 Webpack 可以处理的模块。

##### 智能解析

Webpack 有一个智能解析器，几乎可以处理任何第三方库，无论它们的模块形式是 CommonJS、 AMD 还是普通的 JS 文件。甚至在加载依赖的时候，允许使用动态表达式 `require("./templates/" + name + ".jade")`。

##### 插件系统

Webpack 还有一个功能丰富的插件系统。大多数内容功能都是基于这个插件系统运行的，还可以开发和使用开源的 Webpack 插件，来满足各式各样的需求。

##### 快速运行

Webpack 使用异步 I/O 和多级缓存提高运行效率，这使得 Webpack 能够以令人难以置信的速度快速增量编译。

## 准备开始

### 安装

首先要安装 Node.js， Node.js 自带了软件包管理器 npm，Webpack 需要 Node.js v0.6 以上支持，建议使用最新版 Node.js。

用 npm 安装 Webpack：

```
npm install webpack -g
```

此时 Webpack 已经安装到了全局环境下，可以通过命令行 `webpack -h` 试试。

通常我们会将 Webpack 安装到项目的依赖中，这样就可以使用项目本地版本的 Webpack。

```
# 进入项目目录
# 确定已经有 package.json，没有就通过 npm init 创建
# 安装 webpack 依赖
npm install webpack --save-dev
```

Webpack 目前有两个主版本，一个是在 master 主干的稳定版，一个是在 webpack-2 分支的测试版，测试版拥有一些实验性功能并且和稳定版不兼容，在正式项目中应该使用稳定版。

```
# 查看 webpack 版本信息
$ npm info webpack

# 安装指定版本的 webpack
$ npm install webpack@1.12.x --save-dev
```

如果需要使用 Webpack 开发工具，要单独安装：

```
npm install webpack-dev-server --save-dev
```

### 使用




































