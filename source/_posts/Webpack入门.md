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

我们可以直接使用 require(XXX) 的形式来引入各模块，即使它们可能需要经过编译（比如JSX和sass），但我们无须在上面花费太多心思，因为 webpack 有着各种健全的加载器（loader）在默默处理这些事情，这块我们后续会提到。

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

##### Webpack

优势：

1. webpack 是以 commonJS 的形式来书写脚本滴，但对 AMD/CMD 的支持也很全面，方便旧项目进行代码迁移。
2. 能被模块化的不仅仅是 JS 了。
3. 开发便捷，能替代部分 grunt/gulp 的工作，比如打包、压缩混淆、图片转base64等。
4. 扩展性强，插件机制完善，特别是支持 React 热插拔（见 react-hot-loader ）的功能让人眼前一亮。

我们谈谈第一点。以 AMD/CMD 模式来说，鉴于模块是异步加载的，所以我们常规需要使用 define 函数来帮我们搞回调：

```
define(['package/lib'], function(lib){
    function foo(){
        lib.log('hello world!');
    } 
    return {
        foo: foo
    };
});
```

另外为了可以兼容 commonJS 的写法，我们也可以将 define 这么写：

```
define(function (require, exports, module){
    var someModule = require("someModule");
    var anotherModule = require("anotherModule");    
 
    someModule.doTehAwesome();
    anotherModule.doMoarAwesome();
 
    exports.asplode = function (){
        someModule.doTehAwesome();
        anotherModule.doMoarAwesome();
    };
});
```

然而对 webpack 来说，我们可以直接在上面书写 commonJS 形式的语法，无须任何 define （毕竟最终模块都打包在一起，webpack 也会最终自动加上自己的加载器）：

```
var someModule = require("someModule");
var anotherModule = require("anotherModule");    

someModule.doTehAwesome();
anotherModule.doMoarAwesome();

exports.asplode = function (){
    someModule.doTehAwesome();
    anotherModule.doMoarAwesome();
};
```



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

当你为你的模块安装一个依赖模块时，正常情况下你得先安装他们（在模块根目录下`npm install module-name`），然后连同版本号手动将他们添加到模块配置文件package.json中的依赖里（`dependencies`）。

`--save`和`--save-dev`可以省掉你手动修改`package.json`文件的步骤。
`npm install module-name --save` 自动把模块和版本号添加到`dependencies`部分
`npm install module-name --save-dve` 自动把模块和版本号添加到`devdependencies`部分

通过这些命令，我们会得到一个新的package.json。然后再做一个试验就懂得了区别：删除node_modules目录，然后执行 `npm install --production`，可以看到，npm只帮我们自动安装package.json中`dependencies`部分的模块；如果执行`npm install` ，则package.json中指定的`dependencies`和`devDependencies`都会被自动安装进来。


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

首先创建一个静态页面 index.html 和一个 JS 入口文件 entry.js：

```
<!-- index.html -->
<html>
<head>
  <meta charset="utf-8">
</head>
<body>
  <script src="bundle.js"></script>
</body>
</html>
```

**可以看到我们连样式都不用引入，毕竟脚本执行时会动态生成`<style>`并标签打到head里。**

```
// entry.js
document.write('It works.')
```

然后编译 entry.js 并打包到 bundle.js：

```
webpack entry.js bundle.js
```

打包过程会显示日志：

```
Hash: 401027cb1e7b5eac6412
Version: webpack 2.2.1
Time: 49ms
    Asset     Size  Chunks             Chunk Names
bundle.js  2.54 kB       0  [emitted]  main
   [0] ./entry.js 27 bytes {0} [built]
```

用浏览器打开 `index.html` 将会看到 `It works.` 。

接下来添加一个模块 `module.js` 并修改入口 `entry.js`：

```
// module.js
module.exports = 'It works from module.js.'
```

```
// entry.js
document.write('It works.')
document.write(require('./module.js')) // 添加模块，在当前目录一定要写 ./ ，否则会找不到 module.js
```

重新打包 `webpack entry.js bundle.js` 后刷新页面看到变化 `It works.It works from module.js.`

```
Hash: b1d4f512f99135cb9a69
Version: webpack 2.2.1
Time: 76ms
    Asset     Size  Chunks             Chunk Names
bundle.js  3.42 kB       0  [emitted]  main
   [0] ./module.js 41 bytes {0} [built]
   [1] (webpack)/buildin/module.js 517 bytes {0} [built]
   [2] ./entry.js 74 bytes {0} [built]
```

Webpack 会分析入口文件，解析包含依赖关系的各个文件。这些文件（模块）都打包到 bundle.js 。Webpack 会给每个模块分配一个唯一的 id 并通过这个 id 索引和访问模块。在页面启动时，会先执行 entry.js 中的代码，**其它模块会在运行 `require` 的时候再执行**。

### Loader

Webpack 本身只能处理 JavaScript 模块，如果要处理其他类型的文件，就需要使用 loader 进行转换。

Loader 可以理解为是模块和资源的转换器，它本身是一个函数，接受源文件作为参数，返回转换的结果。这样，我们就可以通过 `require` 来加载任何类型的模块或文件，比如 CoffeeScript、 JSX、 LESS 或图片。

先来看看 loader 有哪些特性：

* Loader 可以通过管道方式链式调用，每个 loader 可以把资源转换成任意格式并传递给下一个 loader ，但是最后一个 loader 必须返回 JavaScript。
* Loader 可以同步或异步执行。
* Loader 运行在 node.js 环境中，所以可以做任何可能的事情。
* Loader 可以接受参数，以此来传递配置项给 loader。
* Loader 可以通过文件扩展名（或正则表达式）绑定给不同类型的文件。
* Loader 可以通过 `npm` 发布和安装。
* 除了通过 `package.json` 的 `main` 指定，通常的模块也可以导出一个 loader 来使用。
* Loader 可以访问配置。
* 插件可以让 loader 拥有更多特性。
* Loader 可以分发出附加的任意文件。

Loader 本身也是运行在 node.js 环境中的 JavaScript 模块，它通常会返回一个函数。大多数情况下，我们通过 npm 来管理 loader，但是你也可以在项目中自己写 loader 模块。

按照惯例，而非必须，loader 一般以 `xxx-loader` 的方式命名，`xxx` 代表了这个 loader 要做的转换功能，比如 `json-loader`。



**在引用 loader 的时候必须使用全名 `json-loader`**，使用短名 `json` 的方式已经被取消了！

```
Default: ["*-webpack-loader", "*-web-loader", "*-loader", "*"]
```

Loader 可以在 `require()` 引用模块的时候添加，也可以在 webpack 全局配置中进行绑定，还可以通过命令行的方式使用。

接上一节的例子，我们要在页面中引入一个 CSS 文件 style.css，首页将 style.css 也看成是一个模块，然后用 `css-loader` 来读取它，再用 `style-loader` 把它插入到页面中。

```
/* style.css */
body { background: yellow; }
```

修改 entry.js：

```
require("!style-loader!css-loader!./style.css") //  写成 require("!style!css!./style.css") 的简写形式已经无法使用了
document.write('It works.')
document.write(require('./module.js'))
```

安装 loader：

```
npm install css-loader style-loader
```

**要注意，css-loader 和 style-loader 不能使用 -g 安装，测试无法使用，只能在项目根目录 `npm init npm install css-loader style-loader --save-dev`**

重新编译打包，刷新页面，就可以看到黄色的页面背景了。

* css-loader 是处理css文件中的url()等
* style-loader 将css插入到页面的style标签顺便告诉你
* less-loader 是将less文件编译成css
* sass-loader 是将sass文件编译成css

**loader的加载顺序是从右往左，从下往上**

如果每次 `require` CSS 文件的时候都要写 loader 前缀，是一件很繁琐的事情。我们可以根据模块类型（扩展名）来自动绑定需要的 loader。

将 entry.js 中的 `require("!style-loader!css-loader!./style.css")` 修改为 `require("./style.css")` ，然后执行：

```
$ webpack entry.js bundle.js --module-bind 'css=style-loader!css-loader'

# 有些环境下可能需要使用双引号
$ webpack entry.js bundle.js --module-bind "css=style-loader!css-loader"
```

显然，这两种使用 loader 的方式，效果是一样的。

#### 独立打包样式文件

有时候可能希望项目的样式能不要被打包到脚本中，而是独立出来作为.css，然后在页面中以`<link>`标签引入。这时候我们需要 [extract-text-webpack-plugin](https://github.com/webpack-contrib/extract-text-webpack-plugin) 来帮忙：

最终 webpack 执行后会乖乖地把样式文件提取出来：

![](http://pic.w2bc.com/upload/201507/16/201507161700552446.png)

### 配置文件

更多配置文件写法在[这里](https://webpack.js.org/concepts/configuration/)

Webpack 在执行的时候，除了在命令行传入参数，还可以通过指定的配置文件来执行。默认情况下，会搜索当前目录的 `webpack.config.js` 文件，这个文件是一个 node.js 模块，返回一个 json 格式的配置信息对象，或者通过 `--config` 选项来指定配置文件。

继续我们的案例，在根目录创建 `package.json` 来添加 webpack 需要的依赖：

```
{
  "name": "webpack-example",
  "version": "1.0.0",
  "description": "A simple webpack example.",
  "main": "bundle.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "ShiningDan",
  "license": "ISC",
  "devDependencies": {
    "css-loader": "^0.26.1",
    "style-loader": "^0.13.1",
    "webpack": "^2.2.1"
  }
}
```

```
# 如果没有写入权限，请尝试如下代码更改权限
chflags -R nouchg .
sudo chmod  775 package.json
```

运行 `npm install`。

然后创建一个配置文件 `webpack.config.js`：

```
var webpack = require('webpack')

module.exports = {  // 是 module.exports 不是 module.export
  entry: './entry.js',
  output: {
    path: __dirname,
    filename: 'bundle.js'
  },
  module: {
    loaders: [
      {test: /\.css$/, loader: 'style!css'}
    ]
  }
}
```

同时简化 entry.js 中的 style.css 加载方式：

```
require('./style.css')
```

最后运行 `webpack`，可以看到 webpack 通过配置文件执行的结果和上一章节通过命令行 `webpack entry.js bundle.js --module-bind 'css=style!css'` 执行的结果是一样的。

#### 使用CDN/远程文件

有时候我们希望某些模块走CDN并以`<script>`的形式挂载到页面上来加载，但又希望能在 webpack 的模块中使用上。
这时候我们可以在配置文件里使用 externals 属性来帮忙：

```
{
    externals: {
        // require("jquery") 是引用自外部模块的
        // 对应全局变量 jQuery
        "jquery": "jQuery"
    }
}
```

需要留意的是，得确保 CDN 文件必须在 webpack 打包文件引入之前先引入。

下面有一另一个 `webpack.config.js` 的例子

```
var webpack = require('webpack');
var commonsPlugin = new webpack.optimize.CommonsChunkPlugin('common.js');
 
module.exports = {
    //插件项
    plugins: [commonsPlugin],
    //页面入口文件配置
    entry: {
        index : './src/js/page/index.js'
    },
    //入口文件输出配置
    output: {
        path: 'dist/js/page',
        filename: '[name].js'
    },
    module: {
        //加载器配置
        loaders: [
            { test: /\.css$/, loader: 'style-loader!css-loader' },
            { test: /\.js$/, loader: 'jsx-loader?harmony' },
            { test: /\.scss$/, loader: 'style!css!sass?sourceMap'},
            { test: /\.(png|jpg)$/, loader: 'url-loader?limit=8192'}
        ]
    },
    //其它解决方案配置
    resolve: {
        root: 'E:/github/flux-example/src', //绝对路径
        extensions: ['', '.js', '.json', '.scss'],
        alias: {
            AppStore : 'js/stores/AppStores.js',
            ActionType : 'js/actions/ActionType.js',
            AppAction : 'js/actions/AppAction.js'
        }
    }
};
```

`plugins` 是插件项，这里我们使用了一个 `CommonsChunkPlugin` 的插件，它用于提取多个入口文件的公共脚本部分，然后生成一个 common.js 来方便多页面之间进行复用。

`entry` 是页面入口文件配置，`output` 是对应输出项配置（即入口文件最终要生成什么名字的文件、存放到哪里），其语法大致为：

```
{
    entry: {
        page1: "./page1",
        //支持数组形式，将加载数组中的所有模块，但以最后一个模块作为输出
        page2: ["./entry1", "./entry2"]
    },
    output: {
        path: "dist/js/page",
        filename: "[name].bundle.js" // [name]就是 entry 的 key
    }
}
```

该段代码最终会生成一个 `page1.bundle.js` 和 `page2.bundle.js`，并存放到 ./dist/js/page 文件夹下。


module.loaders 是最关键的一块配置。它告知 webpack 每一种文件都需要使用什么加载器来处理：

```
module: {
        //加载器配置
        loaders: [
            //.css 文件使用 style-loader 和 css-loader 来处理
            { test: /\.css$/, loader: 'style-loader!css-loader' },
            //.js 文件使用 jsx-loader 来编译处理
            { test: /\.js$/, loader: 'jsx-loader?harmony' },
            //.scss 文件使用 style-loader、css-loader 和 sass-loader 来编译处理
            { test: /\.scss$/, loader: 'style!css!sass?sourceMap'},
            //图片文件使用 url-loader 来处理，小于8kb的直接转为base64
            { test: /\.(png|jpg)$/, loader: 'url-loader?limit=8192'}
        ]
    }
```

配置信息的参数 `?limit=8192` 表示将所有小于8kb的图片都转为base64形式（其实应该说超过8kb的才使用 url-loader 来映射到文件，否则转为data url形式）。

最后是 resolve 配置，这块很好理解，直接写注释了：

```
resolve: {
        //查找module的话从这里开始查找
        root: 'E:/github/flux-example/src', //绝对路径
        //自动扩展文件后缀名，意味着我们require模块可以省略不写后缀名
        extensions: ['', '.js', '.json', '.scss'],
        //模块别名定义，方便后续直接引用别名，无须多写长长的地址
        alias: {
            AppStore : 'js/stores/AppStores.js',//后续直接 require('AppStore') 即可
            ActionType : 'js/actions/ActionType.js',
            AppAction : 'js/actions/AppAction.js'
        }
    }
```

### 插件

插件可以完成更多 loader 不能完成的功能。

插件的使用一般是在 webpack 的配置信息 `plugins` 选项中指定。

Webpack 本身内置了一些常用的插件，还可以通过 `npm` 安装第三方插件。

接下来，我们利用一个最简单的 `BannerPlugin` 内置插件来实践插件的配置和运行，这个插件的作用是给输出的文件头部添加注释信息。

修改 `webpack.config.js`，添加 `plugins`：

```
var webpack = require("webpack");

module.exports = {
	entry: "./entry.js",
	output: {
		path: __dirname,
		filename: "bundle.js"
	},
	module: {
		loaders: [
			{test: /\.css/, loader: "style-loader!css-loader"}
		]
	},
	plugins: [
		new webpack.BannerPlugin("This file is created by ShiningDan")
	]
};
```

```
/*! This file is created by ShiningDan */
/******/ (function(modules) { // webpackBootstrap
/******/ 	// The module cache
/******/ 	var installedModules = {};
```

### 开发环境

当项目逐渐变大，webpack 的编译时间会变长，可以通过参数让编译的输出内容带有进度和颜色。

```
webpack --progress --colors
```
如果不想每次修改模块后都重新编译，那么可以启动监听模式。开启监听模式后，没有变化的模块会在编译后缓存到内存中，而不会每次都被重新编译，所以监听模式的整体速度是很快的。

```
webpack --progress --colors --watch
```

当然，使用 `webpack-dev-server` 开发服务是一个更好的选择。它将在 localhost:8080 启动一个 express 静态资源 web 服务器，并且会以监听模式自动运行 webpack，在浏览器打开 http://localhost:8080/ 或 http://localhost:8080/webpack-dev-server/ 可以浏览项目中的页面和编译后的资源输出，并且通过一个 socket.io 服务实时监听它们的变化并自动刷新页面。

**能够在上面的链接中输出编译后的 HTML 页面效果。如果修改了项目的代码并且保存，则会自动进行编译，并且在页面中输出新的效果。**

```
# 安装
$ npm install webpack-dev-server -g

# 运行
$ webpack-dev-server --progress --colors
```

其他主要的参数有：

```
webpack --config XXX.js   //使用另一份配置文件（比如webpack.config2.js）来打包
 
webpack --watch   //监听变动并自动打包
 
webpack -p    //压缩混淆脚本，这个非常非常重要！
 
webpack -d    //生成map映射文件，告知哪些模块被最终打包到哪里了
```


### 故障处理

Webpack 的配置比较复杂，很容出现错误，下面是一些通常的故障处理手段。

一般情况下，webpack 如果出问题，会打印一些简单的错误信息，比如模块没有找到。我们还可以通过参数 `--display-error-details` 来打印错误详情。

```
$ webpack --display-error-details

Hash: a40fbc6d852c51fceadb
Version: webpack 1.12.2
Time: 586ms
    Asset     Size  Chunks             Chunk Names
bundle.js  12.1 kB       0  [emitted]  main
   [0] ./entry.js 153 bytes {0} [built] [1 error]
   [5] ./module.js 43 bytes {0} [built]
    + 4 hidden modules

ERROR in ./entry.js
Module not found: Error: Cannot resolve 'file' or 'directory' ./badpathmodule in /Users/zhaoda/data/projects/webpack-handbook/examples
resolve file
  /Users/zhaoda/data/projects/webpack-handbook/examples/badpathmodule doesn't exist
  /Users/zhaoda/data/projects/webpack-handbook/examples/badpathmodule.webpack.js doesn't exist
  /Users/zhaoda/data/projects/webpack-handbook/examples/badpathmodule.js doesn't exist
  /Users/zhaoda/data/projects/webpack-handbook/examples/badpathmodule.web.js doesn't exist
  /Users/zhaoda/data/projects/webpack-handbook/examples/badpathmodule.json doesn't exist
resolve directory
  /Users/zhaoda/data/projects/webpack-handbook/examples/badpathmodule doesn't exist (directory default file)
  /Users/zhaoda/data/projects/webpack-handbook/examples/badpathmodule/package.json doesn't exist (directory description file)
[/Users/zhaoda/data/projects/webpack-handbook/examples/badpathmodule]
[/Users/zhaoda/data/projects/webpack-handbook/examples/badpathmodule.webpack.js]
[/Users/zhaoda/data/projects/webpack-handbook/examples/badpathmodule.js]
[/Users/zhaoda/data/projects/webpack-handbook/examples/badpathmodule.web.js]
[/Users/zhaoda/data/projects/webpack-handbook/examples/badpathmodule.json]
 @ ./entry.js 3:0-26
```

Webpack 的配置提供了 `resolve` 和 `resolveLoader` 参数来设置模块解析的处理细节，`resolve` 用来配置应用层的模块（要被打包的模块）解析，`resolveLoader` 用来配置 loader 模块的解析。

当引入通过 `npm` 安装的 node.js 模块时，可能出现找不到依赖的错误。Node.js 模块的依赖解析算法很简单，是通过查看模块的每一层父目录中的 `node_modules` 文件夹来查询依赖的。当出现 Node.js 模块依赖查找失败的时候，可以尝试设置 `resolve.fallback` 和 `resolveLoader.fallback` 来解决问题。

```
module.exports = {
  resolve: { fallback: path.join(__dirname, "node_modules") },
  resolveLoader: { fallback: path.join(__dirname, "node_modules") }
};
```

Webpack 中涉及路径配置最好使用绝对路径，建议通过 `path.resolve(__dirname, "app/folder")` 或 `path.join(__dirname, "app", "folder")` 的方式来配置，以兼容 Windows 环境。


