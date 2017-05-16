---
title: PostCSS 入门
date: 2017-05-16 16:53:01
categories: coding
tags:
  - PostCSS
---

本文主要记录的是入门学习 PostCSS 的笔记。

[PostCSS | GitHub](https://github.com/postcss/postcss)

<!--more-->

## 前言

我认为的 PostCSS，是一个平台，其本身不对 CSS 进行处理，但是通过在该平台上集成插件，可以实现对 CSS 的操作。

无论你是想使用PostCSS什么功能，你都只需要根据你自己特定目定安装所需的插件。意思就是说，你不需要安装任何依赖的插件。著作权归作者所有。

几个不同使用PostCSS的方法：

* 跨浏览器
* 压缩和优化
* PreCSS预处理器
* 定义自己的预处理器
* 结合Stylus、Sass或LESS
* CSS的BEM或SUI方法
* 快捷键与缩写
* PostCSS的语法糖

## 在 Gulp 中使用 PostCSS

首先安装 `gulp-postcss`。

```
npm install --save-dev gulp-postcss
```

现在通过编辑器打开 `gulpfile.js` 文件，并且创建 `gulp` 和 `gulp-postcss` 变量，如下面代码所示：

```
var gulp = require('gulp');
var postcss = require('gulp-postcss');

gulp.task('css', function () { 
  var processors = [ ]; 
  return gulp.src('./src/*.css')
    .pipe(postcss(processors))
    .pipe(gulp.dest('./dest')); 
});
```

在第一行，设置了一个任务名叫 `css` 。这个任务将会执行一个函数，同时在这个函数中创建了一个名为 `processors` 的数组。现在这个数组为空，这里将插入我们想使用的 PostCSS 插件。

在 `src` 目录中创建一个测试文件 `style.css` ，并在这个文件中添加一些CSS的测试代码

现在在命令终端的项目目录下执行下面的命令：

```
gulp css
```

执行完上面的任务之后，在 `dest` 目录中可以找到一个新的 `style.css` 文件。

### 添加PostCSS插件

现在我们添加需要的 PostCSS 插件：Autoprefixer (处理浏览器私有前缀)

运行下面的命令，将插件安装到你的项目：

```
npm install autoprefixer --save-dev
```

在我们的项目中定义变量，将这些插件加载到我们的项目中。和前面的方式一样，在 `gulpfile.js` 文件中添加下面的代码：

```
var autoprefixer = require('autoprefixer');

var processors = [autoprefixer]; 
```

在命令终端执行gulp css命令。会自动调用 Autoprefixer 来处理 CSS 代码。

## 管理插件

### 查找插件

[PostCSS | GitHub](https://github.com/postcss/postcss#plugins) 上维护了一个仓库

[PostCSS.parts](http://postcss.parts/) 这个网站也提供了非常多的 PostCSS 插件

### 插件类型

#### Pack

Pack 可以将多个模块插件打包在一起，允许他们安装和部署。著作权归作者所有。

#### Future

未来的CSS语法(Future CSS Syntax)是让写的CSS能更符合W3C规范，但可能不是完全的支持浏览器。

#### Fallbacks

fallbacks本质上是将今天的代码在以前的浏览器上能工作。

#### 语言扩展

语言扩展插件就是添加CSS功能。

#### Color

PostCSS提供了许多颜色转换的插件，可以将一种颜色格式转换成另一种，比如 `#hex.a` 转换成 `rgba()` , `hcl(H,C,L)` 转换成 `#rgb` 或者普通色转换成 `#rgb` 。还有一些插件提供颜色功能上的操作，比如说两个颜色混合在一起，修改颜色的亮度或深度得到新颜色等。

#### 图片和字体

如何生成base64数据,如何生成CSS Sprites和SVG的优化。你还会发现其他一些类型的图像和字体的工具，比如说在IE8下将SVG转换成PNG，自动生成Webp图像格式和 `@font-face` 以及Retina下图片

#### 网格

PostCSS也可以处理网格系统

#### 优化

PostCSS优化插件主要分为两大类：压缩和代码修改。通过这些插件可以将代码进行压缩，删除掉代码中的空格和注释，更复杂一点的可以修改类似于匹配的媒体查询、@import内联样式，font-weight优化，删除空格和重复的样式等。

#### Shortcuts

使用快捷键或缩写的插件提高代码的编写效率。

## 跨浏览器的兼容性

PostCSS 插件对跨浏览器的兼容性提供以下的功能：

* 有前缀的自动添加前缀
* 添加一系列的IE版本，回退到IE8、IE9和IE10
* 为没有支持的属性添加 `will-change` 属性

### 自动添加浏览器前缀

[AutoPrxfixer](https://github.com/postcss/autoprefixer)

#### 指定支持的浏览器版本

Autoprefixer使用Browserlist来确定哪些浏览器版本将得到支持。

如：

```
autoprefixer({browsers:'safari >= 9, ie >= 11'})
```

### 为will-change属性添加回退

will-change属性用于提前让浏览器知道某些元素设计的动画。这允许浏览器优化呈现动画渲染过程，防止延误和闪烁。

### 给IE老版本添加降级属性

#### 给rgba()颜色创建降级方案

IE8不支持rgba()颜色，所以@Guillaume Demesy的 [postcss-color-rgba-fallback](https://github.com/postcss/postcss-color-rgba-fallback) 插件添加了一个十六进制颜色作为降级处理。

#### 给opacity提供降级方案

IE8也不支持opacity属性，所以@Vincent De Oliveira提供了 [postcss-opacity](https://github.com/iamvdo/postcss-opacity) 插件，给IE浏览器添加滤镜属性，作为降级处理。

#### 将伪元素的`::`转换为`:`

例如 `.element::before` 使用两个冒号 `::` 表示为伪元素，而 `.element:hover` 使用一个冒号 `:` 表示为伪类。实践中表明使用 `::` 主要是用来区分伪类。

然而，在IE8中仅支持一个冒号 `:` ，并不支持 `::` 的伪元素。通过@Sven Tschui的 [postcss-pseudoelements](https://github.com/axa-ch/postcss-pseudoelements) 插件，可以得到最好的实践，插件能自将的将 `::` 转换为 `:`。

#### 给rem添加px作为降级处理

IE8一直都不支持 `rem` 单位，而且在IE9和IE10中他们都不支持伪元素和 `font` 的缩写。使用@Vincent De Oliveira和@Rob Wierzbowski的[node-pixrem](https://github.com/robwierzbowski/node-pixrem)插件，可以自动为 `rem` 添加 `px` 单位作为降级处理。

## PostCSS-modules

CSS-Module 允许你将所有CSS class自动打碎，这是CSS模块（CSS Modules）的默认设置。然后生成一个JSON文件（sources map）和原本的class关联：

```
/* post.css */
.article {
    font-size: 16px;
}

.title {
    font-weight: 24px;
}
```

上面的post.css将会被转换成类似下面这样：

```
.xkpka {
    font-size: 16px;
}

.xkpkb {
    font-size: 24px;
}
```

被打碎替换的classes将被保存在一个JSON对象中：

```
{  "article":  "xkpka",  "title":  "xkpkb"  }
```

在转换完成后，你可以直接引用这个JSON对象到项目中，这样就可以用之前写过的class名来直接使用它了。

```
import styles from './post.json';

class Post extends React.Component {
  render() {
    return (
      <div className={ styles.article }>
        <div className={ styles.title }>…</div>
        …
      </div>
    );
  }
}
```

## 压缩和优化CSS

### 使用 `@import` 压缩文件

通过@import将多个样式组合为一个，即使你的一些样式来自Bower组件或npm模块,都可以确保你你只需要请求一个单独的http，加载网站的CSS

使用@Maxime Thirouin的 [postcss-import]()https://github.com/postcss/postcss-import 插件，遵循 `@import` 规则，你可以将分样式合并到你的主样式表中。

这个插件还有一个非常有用特性，它能够自动发现bower_components或node_modules文件夹内的CSS文件位置。

### 结合匹配的媒体查询

@Kyo Nagashima的 [css-mqpacker](https://github.com/hail2u/node-css-mqpacker) 插件可以找到样式表中相同的媒体查询样式合并到一个媒体查询中。这样允许你在写CSS的时候，媒体查询可以重复编写，你也不用担心这样会对你的样式产生冗余代码，而影响你的效率。

### cssnano插件包

@Ben Briggs提供了一个非常强大的CSS优化的插件包 [cssnano](http://cssnano.co/)，这个插件包是一个可以即插即用的。它汇集了大约25个插件，只需要执行一个命令，就可以做多方面不同类型的优化。

cssnano 优化包括下面一些类型：

* 删除空格和最后一个分号
* 删除注释
* 优化字体权重
* 丢弃重复的样式规则
* 优化calc()
* 压缩选择器
* 减少手写属性
* 合并规则

## 参考

[PostCSS 深入学习 | 大漠老师](https://www.w3cplus.com/blog/tags/516.html?page=1)







