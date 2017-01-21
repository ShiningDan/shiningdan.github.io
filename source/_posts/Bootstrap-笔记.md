---
title: Bootstrap 笔记
date: 2017-01-14 14:52:11
categories: coding
tags:
  - Bootstrap
---
#  Bootstrap 笔记

本系列的是参考[慕课网 大漠](http://www.imooc.com/u/104044/courses?sort=publish)老师的视频，仅供个人查阅使用，具体讲解推荐参考视频。

在 Bootstrap 的发展中，有一些样式发生了改变，在本文中分析的是较新的 Bootstrap 样式。

## 使用 Bootstrap

使用 Bootstrap，可以将文件下载到本地使用，或者使用 Bootstrap 提供的 CDN。

下面是 Bootstrap 的 CDN，最新的 CDN 可以在 [Bootstrap 官网](http://getbootstrap.com) 上下载。：

```
<!-- Latest compiled and minified CSS -->
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u" crossorigin="anonymous">

<!-- Optional theme -->
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap-theme.min.css" integrity="sha384-rHyoN1iRsVXV4nD0JutlnGaslCJuC7uwjduW9SVrLvRYooPp2bWYgmgJQIXwl/Sp" crossorigin="anonymous">

<!-- Latest compiled and minified JavaScript -->
<script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js" integrity="sha384-Tc5IQib027qvyjSMfHjOMaLkfuWVxZxUPnCJA7l2mCWNIpG9mGCD8wGNIcPD7Txa" crossorigin="anonymous"></script>
```

下载的 Bootstrap 的内容格式如下，包括一些 CSS 样式和 js 文件，和一些字体文件。：

```
bootstrap/
├── css/
│   ├── bootstrap.css
│   ├── bootstrap.css.map
│   ├── bootstrap.min.css
│   ├── bootstrap.min.css.map
│   ├── bootstrap-theme.css
│   ├── bootstrap-theme.css.map
│   ├── bootstrap-theme.min.css
│   └── bootstrap-theme.min.css.map
├── js/
│   ├── bootstrap.js
│   └── bootstrap.min.js
└── fonts/
    ├── glyphicons-halflings-regular.eot
    ├── glyphicons-halflings-regular.svg
    ├── glyphicons-halflings-regular.ttf
    ├── glyphicons-halflings-regular.woff
    └── glyphicons-halflings-regular.woff2
```

** Bootstrap所有的 JS 插件都依赖于 jQuery，因此 jQuery 要在 Bootstrap 之前使用**，下面为使用 Bootstrap 的一个模板：

<!--more-->

```HTML
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">//在 IE 浏览器中运行最新的渲染模式
    <meta name="viewport" content="width=device-width, initial-scale=1">//初始化移动浏览器的显示 移动浏览器把文件放在一个虚拟的视口 viewport 中，width 用来设置 viewport 的大小为移动浏览器的大小，initial-scale 为设置页面初始缩放比例为 1 （不缩放）。一般针对移动浏览器浏览都会预先设置这一段代码。
    <!-- The above 3 meta tags *must* come first in the head; any other head content must come *after* these tags -->
    <title>Bootstrap 101 Template</title>

    <!-- Bootstrap -->
    <link href="css/bootstrap.min.css" rel="stylesheet">

    <!-- HTML5 shim and Respond.js for IE8 support of HTML5 elements and media queries -->
    <!-- WARNING: Respond.js doesn't work if you view the page via file:// -->
    <!--[if lt IE 9]>
    
    // 浏览器的版本低于 IE9 的浏览器兼容 HTML5 需要添加的标签
      <script src="https://oss.maxcdn.com/html5shiv/3.7.3/html5shiv.min.js"></script>
      
      // 浏览器版本低于 IE9 的浏览器兼容 CSS3 需要添加的标签
      <script src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
    <![endif]-->
  </head>
  <body>
    <h1>Hello, world!</h1>

    <!-- jQuery (necessary for Bootstrap's JavaScript plugins) -->
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script> // jquery 在 Bootstrap 之前加载
    <!-- Include all compiled plugins (below), or include individual files as needed -->
    <script src="js/bootstrap.min.js"></script>
  </body>
</html>
```

下面是使用 Bootstrap 和不使用 Bootstrap 显示的差异：

使用 Bootstrap：

![](http://ofjjubwp5.bkt.clouddn.com/image/png/img84.png)

不使用 Bootstrap：

![](http://ofjjubwp5.bkt.clouddn.com/image/png/img85.png)

## Bootstrap 全局样式

Bootstrap 的全局样式是基于 [normalize.css](http://necolas.github.io/normalize.css/)的，并且在此基础上实现了一些改良，具体的内容如下：

* 移除body的margin声明
* 设置body的背景色为白色
* 为排版设置了基本的字体、字号和行高
* 设置全局链接颜色，且当链接处于悬浮“:hover”状态时才会显示下划线样式

在 Bootstrap 的 CSS 样式中关于全局样式的部分有：

```CSS
html {
  font-family: sans-serif;
  -webkit-text-size-adjust: 100%;
      -ms-text-size-adjust: 100%;
}
body {
  margin: 0;
}
article,
aside,
details,
figcaption,
figure,
footer,
header,
hgroup,
main,
nav,
section,
summary {
  display: block;
}
audio,
canvas,
progress,
video {
  display: inline-block;
  vertical-align: baseline;
}
audio:not([controls]) {
  display: none;
  height: 0;
}
[hidden],
template {
  display: none;
}
a {
  background: transparent;
}
a:active,
a:hover {
  outline: 0;
}
abbr[title] {
  border-bottom: 1px dotted;
}
b,
strong {
  font-weight: bold;
}
dfn {
  font-style: italic;
}
h1 {
  margin: .67em 0;
  font-size: 2em;
}
mark {
  color: #000;
  background: #ff0;
}
small {
  font-size: 80%;
}
sub,
sup {
  position: relative;
  font-size: 75%;
  line-height: 0;
  vertical-align: baseline;
}
sup {
  top: -.5em;
}
sub {
  bottom: -.25em;
}
img {
  border: 0;
}
svg:not(:root) {
  overflow: hidden;
}
figure {
  margin: 1em 40px;
}
hr {
  height: 0;
  -moz-box-sizing: content-box;
       box-sizing: content-box;
}
pre {
  overflow: auto;
}
code,
kbd,
pre,
samp {
  font-family: monospace, monospace;
  font-size: 1em;
}
button,
input,
optgroup,
select,
textarea {
  margin: 0;
  font: inherit;
  color: inherit;
}
button {
  overflow: visible;
}
button,
select {
  text-transform: none;
}
button,
html input[type="button"],
input[type="reset"],
input[type="submit"] {
  -webkit-appearance: button;
  cursor: pointer;
}
button[disabled],
html input[disabled] {
  cursor: default;
}
button::-moz-focus-inner,
input::-moz-focus-inner {
  padding: 0;
  border: 0;
}
input {
  line-height: normal;
}
input[type="checkbox"],
input[type="radio"] {
  box-sizing: border-box;
  padding: 0;
}
input[type="number"]::-webkit-inner-spin-button,
input[type="number"]::-webkit-outer-spin-button {
  height: auto;
}
input[type="search"] {
  -webkit-box-sizing: content-box;
     -moz-box-sizing: content-box;
          box-sizing: content-box;
  -webkit-appearance: textfield;
}
input[type="search"]::-webkit-search-cancel-button,
input[type="search"]::-webkit-search-decoration {
  -webkit-appearance: none;
}
fieldset {
  padding: .35em .625em .75em;
  margin: 0 2px;
  border: 1px solid #c0c0c0;
}
legend {
  padding: 0;
  border: 0;
}
textarea {
  overflow: auto;
}
optgroup {
  font-weight: bold;
}
table {
  border-spacing: 0;
  border-collapse: collapse;
}
td,
th {
  padding: 0;
}
@media print {
  * {
    color: #000 !important;
    text-shadow: none !important;
    background: transparent !important;
    box-shadow: none !important;
  }
  a,
  a:visited {
    text-decoration: underline;
  }
  a[href]:after {
    content: " (" attr(href) ")";
  }
  abbr[title]:after {
    content: " (" attr(title) ")";
  }
  a[href^="javascript:"]:after,
  a[href^="#"]:after {
    content: "";
  }
  pre,
  blockquote {
    border: 1px solid #999;

    page-break-inside: avoid;
  }
  thead {
    display: table-header-group;
  }
  tr,
  img {
    page-break-inside: avoid;
  }
  img {
    max-width: 100% !important;
  }
  p,
  h2,
  h3 {
    orphans: 3;
    widows: 3;
  }
  h2,
  h3 {
    page-break-after: avoid;
  }
  select {
    background: #fff !important;
  }
  .navbar {
    display: none;
  }
  .table td,
  .table th {
    background-color: #fff !important;
  }
  .btn > .caret,
  .dropup > .btn > .caret {
    border-top-color: #000 !important;
  }
  .label {
    border: 1px solid #000;
  }
  .table {
    border-collapse: collapse !important;
  }
  .table-bordered th,
  .table-bordered td {
    border: 1px solid #ddd !important;
  }
}
```

## Bootstrap 排版

### 标题

#### 使用方法

定义标题都是使用标签`<h1>`到`<h6>`，也可以使用 `class="h1"`来让非标题元素和标题使用相同的样式。

```HTML
<h1>Bootstrap标题一</h1>
<div class="h1">Bootstrap标题一</div>
```

#### CSS分析

Bootstrap 中标题的配置统计如下表：

![](http://img.mukewang.com/53acce330001429807730337.jpg)

Bootstrap标题样式进行了以下显著的优化重置：

1. 重新设置了margin-top和margin-bottom的值，  h1~h3重置后的值都是20px；h4~h6重置后的值都是10px。
2. 所有标题的行高都是1.1（也就是font-size的1.1倍）。所有的标题文本颜色和字体都继承父元素的颜色和字体。
3. 固定不同级别标题字体大小，h1=36px，h2=30px，h3=24px，h4=18px，h5=14px和h6=12px。

**小标题**

我们在Web的制作中，常常会碰到在一个标题后面紧跟着一行小的副标题。在Bootstrap中他也考虑了这种排版效果，使用了`<small>`标签来制作副标题。其 CSS 效果是：

1. 行高都是1，而且font-weight设置了normal变成了常规效果（不加粗），同时颜色被设置为灰色（#777）。
2. 由于`<small>`内的文本字体在h1~h3内，其大小都设置为当前字号的65%；而在h4~h6内的字号都设置为当前字号的75%；

```CSS
h1 small,
h2 small,
h3 small,
h4 small,
h5 small,
h6 small,
.h1 small,
.h2 small,
.h3 small,
.h4 small,
.h5 small,
.h6 small,
h1 .small,
h2 .small,
h3 .small,
h4 .small,
h5 .small,
h6 .small,
.h1 .small,
.h2 .small,
.h3 .small,
.h4 .small,
.h5 .small,
.h6 .small {
  font-weight: normal;
  line-height: 1;
  color: #777;
}
h1 small,
.h1 small,
h2 small,
.h2 small,
h3 small,
.h3 small,
h1 .small,
.h1 .small,
h2 .small,
.h2 .small,
h3 .small,
.h3 .small {
  font-size: 65%;
}
h4 small,
.h4 small,
h5 small,
.h5 small,
h6 small,
.h6 small,
h4 .small,
.h4 .small,
h5 .small,
.h5 .small,
h6 .small,
.h6 .small {
  font-size: 75%;
}
```

### 段落

#### 使用方法

使用`<p>` 标签将段落包括起来

#### CSS 分析

Bootstrap 对正文文本设置了一个全局样式：
```CSS
body {
  font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
  font-size: 14px;
  line-height: 1.42857143; // 20/14
  color: #333;
  background-color: #fff;
}
```

1. 全局文本字号为14px(font-size)。
2. 行高为1.42857143（line-height），大约是20px(大家看到一串的小数或许会有疑惑，其实他是通过LESS编译器计算出来的，当然Sass也有这样的功能)。
3. 颜色为深灰色（#333）；
4. 字体为"Helvetica Neue", Helvetica, Arial, sans-serif;（font-family），或许这样的字体对我们中文并不太合适，但在实际项目中，大家可以根据自己的需求进行重置，在此我们不做过多阐述，我们回到这里。该设置都定义在<body>元素上，由于这几个属性都是继承属性，所以Web页面中文本（包括段落p元素）如无重置都会具有这些样式效果。

```CSS
p {
  margin: 0 0 10px;
}
```

在Bootstrap中，为了让段落p元素之间具有一定的间距，便于用户阅读文本，特意**设置了p元素的margin值**（默认情况之下，p元素具有一个上下外边距，并且保持一个行高的高度）：

#### 段落强调

##### 使用方法

使用 `<p class="lead">` 来让一个段落突出显示。

##### CSS 分析

```CSS
.lead {
  margin-bottom: 20px;
  font-size: 16px;
  font-weight: 300;
  line-height: 1.4;
}
@media (min-width: 768px) {/*大中型浏览器字体稍大*/
  .lead {
    font-size: 21px;
  }
}
```
针对大中型浏览器，将字体的大小放大到 21px。

Bootstrap还通过元素标签:`<small>`和`<strong>`给文本做突出样式处理。

```CSS
small,
.small {
  font-size: 85%; /*标准字体的85%,也就是14px * 0.85px，差不多12px*/
}
```

其中还有 small 的 font-size 为 80% 的样式被覆盖掉了。

```CSS
b,
strong {
  font-weight: bold;/*文本加粗*/
}
```

这个样式实现的是文本**粗体**。

**粗体可以使用`<strong>`样式来实现，斜体可以使用`<em>`或者`<i>`样式来实现。不过斜体的实现使用的是浏览器自带的实现方法，Bootstrp没有进行 CSS 的修改。**

#### 通过颜色进行强调

Bootstrap还定义了一套类名，这里称其为强调类名（类似前面说的“.lead”）,这些强调类都是通过颜色来表示强调，具本说明如下：

* .text-muted：提示，使用浅灰色（#999）
* .text-primary：主要，使用蓝色（#428bca）
* .text-success：成功，使用浅绿色(#3c763d)
* .text-info：通知信息，使用浅蓝色（#31708f）
* .text-warning：警告，使用黄色（#8a6d3b）
* .text-danger：危险，使用褐色（#a94442）

显示效果如下：

![](http://ofjjubwp5.bkt.clouddn.com/image/png/img86.png)

##### 使用方法

可以对所有的标签添加对应的类实现文字颜色的改变。当对 `<a>` 标签添加该类的时候，还有自带的 :hover 颜色改变。

##### CSS 分析

CSS 代码如下：

```CSS
.text-muted {
  color: #777;
}
.text-primary {
  color: #337ab7;
}
.text-success {
  color: #3c763d;
}
.text-info {
  color: #31708f;
}
.text-warning {
  color: #8a6d3b;
}
.text-danger {
  color: #a94442;
}
```
#### 段落对齐方法

在CSS中常常使用text-align来实现文本的对齐风格的设置。其中主要有四种风格，在CSS中常常使用text-align来实现文本的对齐风格的设置。其中主要有四种风格：

* .text-left：左对齐
* .text-center：居中对齐
* .text-right：右对齐
* .text-justify：两端对齐

**目前两端对齐在各浏览器下解析各有不同，特别是应用于中文文本的时候。所以项目中慎用。**

##### 使用方法

在段落中使用相关的类即可。

##### CSS 分析

```CSS
.text-left {
  text-align: left;
}
.text-right {
  text-align: right;
}
.text-center {
  text-align: center;
}
.text-justify {
  text-align: justify;
}
```

通过 text-align 实现。

#### 其他文本样式

```CSS
.text-nowrap {
  white-space: nowrap; /*段落中的文本不想允许换行*/
}
.text-lowercase {
  text-transform: lowercase; /*段落中的文字全部小写*/
}
.text-uppercase {
  text-transform: uppercase; /*段落中的文字全部大写*/
}
.text-capitalize {
  text-transform: capitalize; /*段落中所有的单词进行首字母大写*/
}
```

### 列表

在HTML文档中，列表结构主要有三种：有序列表、无序列表和定义列表。

Bootstrap根据平时的使用情形提供了六种形式的列表：
* 普通列表
* 有序列表
* 去点列表
* 内联列表
* 描述列表
* 水平描述列表

#### 有序列表和无序列表

##### 使用方法

使用`ul`标签表示无序列表，使用`ol`标签表示有序列表，和普通表示方法一样。

##### CSS 分析

```CSS
ul,
ol {
  margin-top: 0;
  margin-bottom: 10px;
}
ul ul,
ol ul,
ul ol,
ol ol {
  margin-bottom: 0;
}
```

Bootstrap对于列表，只是在margin上做了一些调整。

#### 去点列表

##### 使用方法

通过对列表添加一个类名“.list-unstyled”,这样就可以去除默认的列表样式的风格。

##### CSS 分析

```CSS
.list-unstyled {
  padding-left: 0;
  list-style: none;
}
```

该效果除了清除项目编号之外，还将列表默认的左边内距也清０了。

**注意：**

如果是对`<ul>`标签添加该类，则`<ul>`标签下所有的`<li>`都没有点，同时所有的`<li>`显示都会提前，因为`<ul>`标签会给其子元素默认添加 padding-left:40px，如下图：

![](http://ojt6zsxg2.bkt.clouddn.com/59414748c29df3f72c47077c27cee106.png)

如果是对单个`<li>`标签使用取点，则该标签前面没有点，但是不会提前，因为此时 padding-left 本来就等于0。如下图：

![](http://ojt6zsxg2.bkt.clouddn.com/413216e4860b7455b45b9dcf529291ad.png)

**所以要分清楚是要给 `<li>` 还是给 `<ul>` 添加 .list-unstyled**

#### 内联列表

通过添加类名“.list-inline”来实现内联列表，简单点说就是把垂直列表换成水平列表，而且去掉项目符号（编号），保持水平显示。

显示效果如下：

![](http://ojt6zsxg2.bkt.clouddn.com/7231e37af375e269dd0d39f11852c1d3.png)

#### 使用方法

在`<ul>` 或 `<ol>` 标签添加类 .list-inline

#### CSS 分析

```CSS
.list-inline {
  padding-left: 0;
  margin-left: -5px;
  list-style: none;
}
.list-inline > li {
  display: inline-block;
  padding-right: 5px;
  padding-left: 5px;
}
```

#### 定义列表

##### 使用方法

定义列表 dl（声明定义列表）、dt（定义列表中的项目）、dd（描述列表中的项目） 使用方法和平常无异。

##### CSS 分析

```CSS
dl {
  margin-top: 0;
  margin-bottom: 20px;
}
dt,
dd {
  line-height: 1.42857143;
}
dt {
  font-weight: bold;
}
dd {
  margin-left: 0;
}
```
只是调整了行间距，外边距和字体加粗效果。

#### 水平定义列表

是定义列表的水平显示，显示效果如下：

![](http://ojt6zsxg2.bkt.clouddn.com/fbdf0471607acb7979a414f7267b4859.png)

##### 使用方法

水平定义列表就像内联列表一样，Bootstrap可以给<dl>添加类名“.dl-horizontal”给定义列表实现水平显示效果

##### CSS 分析

```css
@media (min-width: 768px) {
  .dl-horizontal dt {
    float: left;
    width: 160px;
    overflow: hidden;
    clear: left;
    text-align: right;
    text-overflow: ellipsis; //当发生文本溢出的时候，显示省略符号来代表被修剪的文本。
    white-space: nowrap; // 文本不会换行，文本会在在同一行上继续
  }
  .dl-horizontal dd {
    margin-left: 180px;
  }
}
```

只有屏幕大于768px的时候，添加类名“.dl-horizontal”才具有水平定义列表效果，当屏幕小于768px的时候，定义列表又变为原本的形式。水平列表的修改点有：

1. 将dt设置了一个左浮动，并且设置了一个宽度为160px
2. 将dd设置一个margin-left的值为180px，达到水平的效果
3. 当标题宽度超过160px时，将会显示三个省略号

### 代码

Bootstrap主要提供了三种代码风格：
1. 使用`<code></code>`来显示单行内联代码，一般是针对于单个单词或单个句子的代码
2. 使用`<pre></pre>`来显示多行块代码，一般是针对于多行代码（也就是成块的代码
3. 使用`<kbd></kbd>`来显示用户输入代码，一般是表示用户要通过键盘输入的内容

显示效果如下：

![](http://ojt6zsxg2.bkt.clouddn.com/f9924efb7cc82d6228ad33c43117d0ac.png)

**不管使用哪种代码风格，在代码中碰到小于号（<）要使用硬编码“`&lt;`”来替代，大于号(>)使用“`&gt;`”来替代。**

`<pre>`元素一般用于显示大块的代码，并保证原有格式不变。但有时候代码太多，而且不想让其占有太大的页面篇幅，只需要在pre标签上添加类名“.pre-scrollable”，就可以控制代码块区域最大高度为340px，一旦超出这个高度，就会在Y轴出现滚动条。

#### CSS 分析

```css
.pre-scrollable {
  max-height: 340px;
  overflow-y: scroll;
}
```

### 表格

####  表格的初始化

Boostrap 对表格进行了一些初始化，方便在通过 class 引入自定义的类有一致的显示效果：

```css
table {
  background-color: transparent;
}
table {
  border-spacing: 0; //设置相邻单元格的边框间的距离（仅用于“边框分离”模式）。
  border-collapse: collapse;  //设置表格的边框是否被合并为一个单一的边框，会忽略 border-spacing 和 empty-cells 属性。
}
td,
th {
  padding: 0;
}
```

Bootstrap为表格不同的样式风格提供了不同的类名，主要包括：
1. .table：基础表格
2. .table-striped：斑马线表格
3. .table-bordered：带边框的表格
4. .table-hover：鼠标悬停高亮的表格
5. .table-condensed：紧凑型表格
6. .table-responsive：响应式表格

使用方法是 `<table>` 标签添加上面的类来实现不同样式的标签。**也可以将多个类添加在一起组成显示效果更好的表格**

比较推荐的表格样式显示效果如下：

![](http://ojt6zsxg2.bkt.clouddn.com/626daf1428fbca5e1437b1787c2f1720.png)

![](http://ojt6zsxg2.bkt.clouddn.com/c13aba0c87d66f619a8d220183aaf045.png)

其对应的代码为：

```css
<h1>基础表格</h1>
<table class="table">
   <thead>
     <tr>
       <th>表格标题</th>
       <th>表格标题</th>
       <th>表格标题</th>
     </tr>
   </thead>
   <tbody>
     <tr>
       <td>表格单元格</td>
       <td>表格单元格</td>
       <td>表格单元格</td>
     </tr>
     <tr>
       <td>表格单元格</td>
       <td>表格单元格</td>
       <td>表格单元格</td>
     </tr>
   </tbody>
 </table>
<h1>斑马线表格</h1>
<table class="table table-striped">
   <thead>
     <tr>
       <th>表格标题</th>
       <th>表格标题</th>
       <th>表格标题</th>
     </tr>
   </thead>
   <tbody>
     <tr>
       <td>表格单元格</td>
       <td>表格单元格</td>
       <td>表格单元格</td>
     </tr>
     <tr>
       <td>表格单元格</td>
       <td>表格单元格</td>
       <td>表格单元格</td>
     </tr>
     <tr>
       <td>表格单元格</td>
       <td>表格单元格</td>
       <td>表格单元格</td>
     </tr>
     <tr>
       <td>表格单元格</td>
       <td>表格单元格</td>
       <td>表格单元格</td>
     </tr>
   </tbody>
 </table>
<h1>带边框的表格</h1>
 <table class="table table-bordered">
   <thead>
     <tr>
       <th>表格标题</th>
       <th>表格标题</th>
       <th>表格标题</th>
     </tr>
   </thead>
   <tbody>
     <tr>
       <td>表格单元格</td>
       <td>表格单元格</td>
       <td>表格单元格</td>
     </tr>
     <tr>
       <td>表格单元格</td>
       <td>表格单元格</td>
       <td>表格单元格</td>
     </tr>
     <tr>
       <td>表格单元格</td>
       <td>表格单元格</td>
       <td>表格单元格</td>
     </tr>
     <tr>
       <td>表格单元格</td>
       <td>表格单元格</td>
       <td>表格单元格</td>
     </tr>
   </tbody>
 </table>
<h1>鼠标悬浮高亮的表格</h1>
<table class="table table-striped table-bordered table-hover">
   <thead>
     <tr>
       <th>表格标题</th>
       <th>表格标题</th>
       <th>表格标题</th>
     </tr>
   </thead>
   <tbody>
     <tr>
       <td>表格单元格</td>
       <td>表格单元格</td>
       <td>表格单元格</td>
     </tr>
     <tr>
       <td>表格单元格</td>
       <td>表格单元格</td>
       <td>表格单元格</td>
     </tr>
     <tr>
       <td>表格单元格</td>
       <td>表格单元格</td>
       <td>表格单元格</td>
     </tr>
     <tr>
       <td>表格单元格</td>
       <td>表格单元格</td>
       <td>表格单元格</td>
     </tr>
   </tbody>
 </table>
 <h1>紧凑型表格</h1>
  <table class="table table-condensed">
   <thead>
     <tr>
       <th>表格标题</th>
       <th>表格标题</th>
       <th>表格标题</th>
     </tr>
   </thead>
   <tbody>
     <tr>
       <td>表格单元格</td>
       <td>表格单元格</td>
       <td>表格单元格</td>
     </tr>
     <tr>
       <td>表格单元格</td>
       <td>表格单元格</td>
       <td>表格单元格</td>
     </tr>
     <tr>
       <td>表格单元格</td>
       <td>表格单元格</td>
       <td>表格单元格</td>
     </tr>
     <tr>
       <td>表格单元格</td>
       <td>表格单元格</td>
       <td>表格单元格</td>
     </tr>
   </tbody>
 </table>
 <h1>响应式表格</h1>
 <div class="table-responsive">
   <table class="table table-bordered">
   <thead>
     <tr>
       <th>表格标题</th>
       <th>表格标题</th>
       <th>表格标题</th>
     </tr>
   </thead>
   <tbody>
     <tr>
       <td>表格单元格</td>
       <td>表格单元格</td>
       <td>表格单元格</td>
     </tr>
     <tr>
       <td>表格单元格</td>
       <td>表格单元格</td>
       <td>表格单元格</td>
     </tr>
     <tr>
       <td>表格单元格</td>
       <td>表格单元格</td>
       <td>表格单元格</td>
     </tr>
     <tr>
       <td>表格单元格</td>
       <td>表格单元格</td>
       <td>表格单元格</td>
     </tr>
   </tbody>
 </table>
```

#### 表格的类

只需要在`<tr>`元素中添加上表对应的类名，可以使用不同的颜色表示不同的意义：

![](http://ojt6zsxg2.bkt.clouddn.com/69f7b0ee848d406f6fdce773c4971673.png)

除了”.active”之外，其他四个类名和”.table-hover”配合使用时，Bootstrap针对这几种样式也做了相应的**悬浮状态的样式**设置，所以如果需要给tr元素添加其他颜色样式时，在”.table-hover”表格中也要做相应的调整。

注意要实现悬浮状态，需要在`<table>`标签上加入table-hover类。

#### 基础表格

**不管制作哪种表格都离不开类名“table”。所以大家在使用Bootstrap表格时，千万注意，你的`<table>`元素中一定不能缺少类名“table”。**

##### 使用方法

通过对`<table>`标签添加 .table 类来实现。

##### CSS 分析

“.table”主要有三个作用：
1. 给表格设置了margin-bottom:20px以及设置单元内距
2. 在thead底部设置了一个2px的浅灰实线
3. 每个单元格顶部设置了一个1px的浅灰实线

#### 斑马线表格

其效果与基础表格相比，仅是在tbody隔行有一个浅灰色的背景色。

##### 使用方法

在`<table class="table">`的基础上增加类名“.table-striped”即可：

```css
<table class="table table-striped">
…
</table>
```

##### CSS分析

```css
.table-striped > tbody > tr:nth-of-type(odd) {
  background-color: #f9f9f9;
}
```

#### 带边框的表格

##### 使用方法

在基础表格`<table class="table">`基础上添加一个“.table-bordered”类名即可：

```css
<table  class="table table-bordered">
  …
</table>
```

##### CSS分析

```css
.table-bordered {
  border: 1px solid #ddd;/*整个表格设置边框*/
}
.table-bordered > thead > tr > th,
.table-bordered > tbody > tr > th,
.table-bordered > tfoot > tr > th,
.table-bordered > thead > tr > td,
.table-bordered > tbody > tr > td,
.table-bordered > tfoot > tr > td {
  border: 1px solid #ddd;/*每个单元格设置边框*/
}
.table-bordered > thead > tr > th,
.table-bordered > thead > tr > td {
  border-bottom-width: 2px;/*表头底部边框*
}
```

#### 鼠标悬浮高亮的表格

##### 使用方法

仅需要`<table class="table">`元素上添加类名“table-hover”即可：

```css
<table class="table table-hover">
…
</table>
```

当你鼠标悬浮在某一单元格上时，单元格所在行的背景色都会变成浅灰色。

##### CSS 分析

```css
.table-hover > tbody > tr:hover {
  background-color: #f5f5f5;
}
```

同时也针对不同类型的表格（.active .success .info .warning .error）的 hover 动作设计了不同的颜色

#### 紧凑型表格

##### 使用方法

只是需要在`<table class="table">`基础上添加类名“table-condensed”：

```css
<table class="table table-condensed">
…
</table>
```

##### CSS 分析

```css
.table-condensed > thead > tr > th,
.table-condensed > tbody > tr > th,
.table-condensed > tfoot > tr > th,
.table-condensed > thead > tr > td,
.table-condensed > tbody > tr > td,
.table-condensed > tfoot > tr > td {
  padding: 5px;
}
```

Bootstrap中紧凑型的表格与基础表格差别不大，因为只是将单元格的内距由8px调至5px。

#### 应式表格

##### 使用方法

Bootstrap提供了一个容器，并且此容器设置类名“.table-responsive”,此容器就具有响应式效果，然后将`<table class="table">`置于这个容器当中，这样表格也就具有响应式效果。

````css
<div class="table-responsive">
<table class="table table-bordered">
   …
</table>
</div>
```

##### CSS分析

```css
.table-responsive {
  min-height: .01%;
  overflow-x: auto;
}
@media screen and (max-width: 767px) {
  .table-responsive {
    width: 100%;
    margin-bottom: 15px;
    overflow-y: hidden;
    -ms-overflow-style: -ms-autohiding-scrollbar;
    border: 1px solid #ddd;
  }
  ...
}
```
当你的浏览器可视区域小于768px时，表格底部会出现水平滚动条。当你的浏览器可视区域大于768px时，表格底部水平滚动条就会消失。

## 表单

表单中常见的元素主要包括：**文本输入框、下拉选择框、单选按钮、复选按钮、文本域和按钮**等

Bootstrap并未对其做太多的定制性效果设计，仅仅对表单内的fieldset、legend、label标签进行了定制。如：

```css
fieldset {
  min-width: 0;
  padding: 0;
  margin: 0;
  border: 0;
}
legend {
  display: block;
  width: 100%;
  padding: 0;
  margin-bottom: 20px;
  font-size: 21px;
  line-height: inherit;
  color: #333;
  border: 0;
  border-bottom: 1px solid #e5e5e5;
}
label {
  display: inline-block;
  max-width: 100%;
  margin-bottom: 5px;
  font-weight: bold;
}
```
当然表单除了这几个元素之外，还有input、select、textarea等元素。在Bootstrap框架中，通过定制了一个类名`form-control`，如果这几个元素使用了类名“form-control”，将会实现一些设计上的定制效果：

1. 宽度变成了100%
2. 设置了一个浅灰色（#ccc）的边框
3. 具有4px的圆角
4. 设置阴影效果，并且元素得到焦点之时，阴影和边框效果会有所变化
5. 设置了placeholder的颜色为#999

代码如下：

```css
.form-control {
  display: block;
  width: 100%;
  height: 34px;
  padding: 6px 12px;
  font-size: 14px;
  line-height: 1.42857143;
  color: #555;
  background-color: #fff;
  background-image: none;
  border: 1px solid #ccc;
  border-radius: 4px;
  -webkit-box-shadow: inset 0 1px 1px rgba(0, 0, 0, .075);
          box-shadow: inset 0 1px 1px rgba(0, 0, 0, .075);
  -webkit-transition: border-color ease-in-out .15s, -webkit-box-shadow ease-in-out .15s;
       -o-transition: border-color ease-in-out .15s, box-shadow ease-in-out .15s;
          transition: border-color ease-in-out .15s, box-shadow ease-in-out .15s;
}
.form-control:focus {
  border-color: #66afe9;
  outline: 0;
  -webkit-box-shadow: inset 0 1px 1px rgba(0,0,0,.075), 0 0 8px rgba(102, 175, 233, .6);
          box-shadow: inset 0 1px 1px rgba(0,0,0,.075), 0 0 8px rgba(102, 175, 233, .6);
}
.form-control::-moz-placeholder {
  color: #999;
  opacity: 1;
}
.form-control:-ms-input-placeholder {
  color: #999;
}
.form-control::-webkit-input-placeholder {
  color: #999;
}
```

### 基础表单

下面给的是基础表单的样式与代码：

```css
<form role="form">
  <div class="form-group">
    <label for="exampleInputEmail1">邮箱：</label>
    <input type="email" class="form-control" id="exampleInputEmail1" placeholder="请输入您的邮箱地址">
  </div>
  <div class="form-group">
    <label for="exampleInputPassword1">密码</label>
    <input type="password" class="form-control" id="exampleInputPassword1" placeholder="请输入您的邮箱密码">
  </div>
  <div class="checkbox">
    <label>
      <input type="checkbox"> 记住密码
    </label>
  </div>
  <button type="submit" class="btn btn-default">进入邮箱</button>
</form>
```

![](http://ojt6zsxg2.bkt.clouddn.com/47472e45e9aef5bfd8b0cd933ce40f25.png)

### 水平表单

#### 使用方法

在Bootstrap框架中要实现水平表单效果，必须满足以下两个条件：
1. 在`<form>`元素是使用类名“form-horizontal”。
2. 配合Bootstrap框架的网格系统。因为网格系统带有 float:left 属性，可以自动浮动

下面是一个水平表单的显示示例：

```css
<form class="form-horizontal" role="form">
  <div class="form-group">
    <label for="inputEmail3" class="col-sm-2 control-label">邮箱</label> //添加 col-sm-x 类的对象都是默认 float:left;
    <div class="col-sm-10">
      <input type="email" class="form-control" id="inputEmail3" placeholder="请输入您的邮箱地址">
    </div>
  </div>
  <div class="form-group">
    <label for="inputPassword3" class="col-sm-2 control-label">密码</label>
    <div class="col-sm-10">
      <input type="password" class="form-control" id="inputPassword3" placeholder="请输入您的邮箱密码">
    </div>
  </div>
  <div class="form-group">
    <div class="col-sm-offset-2 col-sm-10">
      <div class="checkbox">
        <label>
          <input type="checkbox"> 记住密码
        </label>
      </div>
    </div>
  </div>
  <div class="form-group">
    <div class="col-sm-offset-2 col-sm-10">
      <button type="submit" class="btn btn-default">进入邮箱</button>
    </div>
  </div>
</form>
```
![](http://ojt6zsxg2.bkt.clouddn.com/a8271bf5e0859bfa7aea5f9d2fc365dd.png)

#### CSS 分析

在`<form>`元素上使用类名“form-horizontal”主要有以下几个作用：

1. 设置表单控件padding和margin值。
2. 改变“form-group”的表现形式，类似于网格系统的“row”。


```css
.form-horizontal .radio,
.form-horizontal .checkbox,
.form-horizontal .radio-inline,
.form-horizontal .checkbox-inline {
  padding-top: 7px;
  margin-top: 0;
  margin-bottom: 0;
}
.form-horizontal .radio,
.form-horizontal .checkbox {
  min-height: 27px;
}
.form-horizontal .form-group {
  margin-right: -15px;
  margin-left: -15px;
}
@media (min-width: 768px) {
  .form-horizontal .control-label {
    padding-top: 7px;
    margin-bottom: 0;
    text-align: right; /*水平的表单，文字右对齐*/
  }
}
.form-horizontal .has-feedback .form-control-feedback {
  right: 15px;
}
@media (min-width: 768px) {
  .form-horizontal .form-group-lg .control-label {
    padding-top: 11px;
    font-size: 18px;
  }
}
@media (min-width: 768px) {
  .form-horizontal .form-group-sm .control-label {
    padding-top: 6px;
    font-size: 12px;
  }
}
```

### 内联表单

#### 使用方法

如果你要在input前面添加一个label标签时，会导致input换行显示。如果你必须添加这样的一个label标签，并且不想让input换行，你需要将label标签也放在容器“form-group”中

只需要在`<form>`元素中添加类名“form-inline”即可。

下面是一个内联表单的示例：

```css
<form class="form-inline" role="form">
  <div class="form-group">
    <label class="sr-only" for="exampleInputEmail2">邮箱</label>
    <input type="email" class="form-control" id="exampleInputEmail2" placeholder="请输入你的邮箱地址">
  </div>
  <div class="form-group">
    <label class="sr-only" for="exampleInputPassword2">密码</label> // 使用 sr-only 将 label 隐藏。
    <input type="password" class="form-control" id="exampleInputPassword2" placeholder="请输入你的邮箱密码">
  </div>
  <div class="checkbox">
    <label>
      <input type="checkbox"> 记住密码
    </label>
  </div>
  <button type="submit" class="btn btn-default">进入邮箱</button>
</form>
```

![](http://ojt6zsxg2.bkt.clouddn.com/44cbe75503c5284b5662b111293cbb3f.png)


#### CSS 分析

```css
@media (min-width: 768px) {
  .form-inline .form-group {
    display: inline-block;
    margin-bottom: 0;
    vertical-align: middle;
  }
  .form-inline .form-control {
    display: inline-block;
    width: auto;
    vertical-align: middle;
  }
  .form-inline .form-control-static {
    display: inline-block;
  }
  .form-inline .input-group {
    display: inline-table;
    vertical-align: middle;
  }
  .form-inline .input-group .input-group-addon,
  .form-inline .input-group .input-group-btn,
  .form-inline .input-group .form-control {
    width: auto;
  }
  .form-inline .input-group > .form-control {
    width: 100%;
  }
  .form-inline .control-label {
    margin-bottom: 0;
    vertical-align: middle;
  }
  .form-inline .radio,
  .form-inline .checkbox {
    display: inline-block;
    margin-top: 0;
    margin-bottom: 0;
    vertical-align: middle;
  }
  .form-inline .radio label,
  .form-inline .checkbox label {
    padding-left: 0;
  }
  .form-inline .radio input[type="radio"],
  .form-inline .checkbox input[type="checkbox"] {
    position: relative;
    margin-left: 0;
  }
  .form-inline .has-feedback .form-control-feedback {
    top: 0;
  }
}
```

内联表单实现原理非常简单，欲将表单控件在一行显示，就需要将表单控件设置成内联块元素（display:inline-block）。

### 输入框 input

#### 使用方法

单行输入框，常见的文本输入框，也就是input的type属性值为text。在Bootstrap中使用input时也必须添加type类型，如果没有指定type类型，将无法得到正确的样式。

为了让控件在各种表单风格中样式不出错，需要添加类名“form-control”

```css
<form role="form">
<div class="form-group">
<input type="email" class="form-control" placeholder="Enter email">
</div>
</form>
```

#### CSS 分析

对 form-control 的分析可以参考表单开始部分

有input、select、textarea等元素。在Bootstrap框架中，通过定制了一个类名`form-control`，如果这几个元素使用了类名“form-control”，将会实现一些设计上的定制效果：

1. 宽度变成了100%
2. 设置了一个浅灰色（#ccc）的边框
3. 具有4px的圆角
4. 设置阴影效果，并且元素得到焦点之时，阴影和边框效果会有所变化
5. 设置了placeholder的颜色为#999

### 下拉选择列表 select

#### 使用方法

Bootstrap框架中的下拉选择框使用和原始的一致，多行选择设置multiple属性的值为multiple。

```css
<form role="form">
<div class="form-group">
  <select class="form-control">
    <option>1</option>
    <option>2</option>
    <option>3</option>
    <option>4</option>
    <option>5</option>
  </select>
  </div>
  <div class="form-group">
  <select multiple class="form-control">
    <option>1</option>
    <option>2</option>
    <option>3</option>
    <option>4</option>
    <option>5</option>
  </select>
</div>
</form>
```

#### CSS 分析

```css
select[multiple],
select[size] {
  height: auto;
}
```

对 select 有一些自定义的初始化设置，如果是 multiple，则height:auto。

### 文本域 textarea

#### 使用方法

文本域和原始使用方法一样，设置rows可定义其高度，设置cols可以设置其宽度。但如果textarea元素中添加了类名“form-control”类名，则无需设置cols属性。因为Bootstrap框架中的“form-control”样式的表单控件宽度为100%或auto。

```css
<form role="form">
  <div class="form-group">
    <textarea class="form-control" rows="3"></textarea>
  </div>
</form>
```

### 单选按钮和复选按钮(checkbox、radio)

#### 使用方法

![](http://ojt6zsxg2.bkt.clouddn.com/8482332185719b98381b2eee952c97e5.png)

显示效果是 checkbox 和 radio 都是竖直排列。

Bootstrap框架中checkbox和radio有点特殊，Bootstrap针对他们做了一些特殊化处理，主要是checkbox和radio与label标签配合使用会出现一些小问题（最头痛的是对齐问题）。使用Bootstrap框架，开发人员无需考虑太多，只需要按照下面的方法使用即可。

1. 不管是checkbox还是radio都使用label包起来了
2. checkbox连同label标签放置在一个名为“.checkbox”的容器内
3. radio连同label标签放置在一个名为“.radio”的容器内
在Bootstrap框架中，主要借助“.checkbox”和“.radio”样式，来处理复选框、单选按钮与标签的对齐方式。

```css
<form role="form">
  <h3>案例1</h3>
  <div class="checkbox">
    <label>
      <input type="checkbox" value="">
      记住密码
    </label>
  </div>
  <div class="radio">
    <label>
      <input type="radio" name="optionsRadios" id="optionsRadios1" value="love" checked>
      喜欢
    </label>
  </div>
    <div class="radio">
    <label>
      <input type="radio" name="optionsRadios" id="optionsRadios2" value="hate">
      不喜欢
    </label>
  </div>
</form>
```

#### CSS 分析

```css
.radio,
.checkbox {
  position: relative;
  display: block;
  margin-top: 10px;
  margin-bottom: 10px;
}
.radio label,
.checkbox label {
  min-height: 20px;
  padding-left: 20px;
  margin-bottom: 0;
  font-weight: normal;
  cursor: pointer;
}
.radio input[type="radio"],
.radio-inline input[type="radio"],
.checkbox input[type="checkbox"],
.checkbox-inline input[type="checkbox"] {
  position: absolute;
  margin-top: 4px \9;
  margin-left: -20px;
}
.radio + .radio,
.checkbox + .checkbox {
  margin-top: -5px;
}
```

#### checkbox 和 radio 水平排列

##### 使用方法

![](http://ojt6zsxg2.bkt.clouddn.com/059be45d1dc0a08186bccefc74d470e4.png)

将复选框和单选按钮需要水平排列。Bootstrap框架也做了这方面的考虑：
1. 如果checkbox需要水平排列，只需要在label标签上添加类名“checkbox-inline”
2. 如果radio需要水平排列，只需要在label标签上添加类名“radio-inline”

**注意，如果是水平排列 checkbox 和 radio，则外层不用嵌套 div.checkbox 或 div.radio，而是使用 div.form-group**

```css
<form role="form">
  <div class="form-group">
    <label class="checkbox-inline">
      <input type="checkbox"  value="option1">游戏
    </label>
    <label class="checkbox-inline">
      <input type="checkbox"  value="option2">摄影
    </label>
    <label class="checkbox-inline">
    <input type="checkbox"  value="option3">旅游
    </label>
  </div>
  <div class="form-group">
    <label class="radio-inline">
      <input type="radio"  value="option1" name="sex">男性
    </label>
    <label class="radio-inline">
      <input type="radio"  value="option2" name="sex">女性
    </label>
    <label class="radio-inline">
      <input type="radio"  value="option3" name="sex">中性
    </label>
  </div>
</form>
```

##### CSS 分析

```css
.radio-inline,
.checkbox-inline {
  position: relative;
  display: inline-block;
  padding-left: 20px;
  margin-bottom: 0;
  font-weight: normal;
  vertical-align: middle;
  cursor: pointer;
}
.radio-inline + .radio-inline,
.checkbox-inline + .checkbox-inline {
  margin-top: 0;
  margin-left: 10px;
}
```

### 表单控件（按钮）

按钮也是表单重要控件之一,制作按钮通常使用下面代码来实现：
1. `input[type=“submit”]`
2. `input[type=“button”]`
3. `input[type=“reset”]`
4. `<button>`

**在Bootstrap框架中的按钮都是采用`<button>`来实现。**

所有的按钮显示效果如下：

![](http://ojt6zsxg2.bkt.clouddn.com/3468353145788a309a05c857a49d4940.png)

#### 使用方法
可以使用 `<button>`标签或者 `<a>` 标签作为按钮，其他的方法，比如 input.btn、div.btn、span.btn 不推荐。

```css
<button class="btn btn-default" href="#">Default</button>
a href="##" class="btn btn-default">a标签按钮</a>
```

#### CSS 分析

```
.btn {
  display: inline-block;
  padding: 6px 12px;
  margin-bottom: 0;
  font-size: 14px;
  font-weight: normal;
  line-height: 1.42857143;
  text-align: center;
  white-space: nowrap;
  vertical-align: middle;
  -ms-touch-action: manipulation;
      touch-action: manipulation;
  cursor: pointer;
  -webkit-user-select: none;
     -moz-user-select: none;
      -ms-user-select: none;
          user-select: none;
  background-image: none;
  border: 1px solid transparent;
  border-radius: 4px;
}
.btn:focus,
.btn:active:focus,
.btn.active:focus,
.btn.focus,
.btn:active.focus,
.btn.active.focus {
  outline: 5px auto -webkit-focus-ring-color;
  outline-offset: -2px;
}
.btn:hover,
.btn:focus,
.btn.focus {
  color: #333;
  text-decoration: none;
}
.btn:active,
.btn.active {
  background-image: none;
  outline: 0;
  -webkit-box-shadow: inset 0 3px 5px rgba(0, 0, 0, .125);
          box-shadow: inset 0 3px 5px rgba(0, 0, 0, .125);
}
.btn.disabled,
.btn[disabled],
fieldset[disabled] .btn {
  cursor: not-allowed;
  filter: alpha(opacity=65);
  -webkit-box-shadow: none;
          box-shadow: none;
  opacity: .65;
}

```

```css
.btn-default {
  color: #333;
  background-color: #fff;
  border-color: #ccc;
}
```

显示效果都是通过设置字体颜色，背景及边框颜色来实现的。

#### 按钮大小

##### 使用方法

使用 .btn-lg、btn-sm、btn-xs来控制按键的大小

```
<button class="btn btn-primary btn-lg" type="button">大型按钮.btn-lg</button>
    <button class="btn btn-primary" type="button">正常按钮</button>
    <button class="btn btn-primary btn-sm" type="button">小型按钮.btn-sm</button>
    <button class="btn btn-primary btn-xs" type="button">超小型按钮.btn-xs</button>
```

![](http://ojt6zsxg2.bkt.clouddn.com/67aac03db1ed3c15b03d43941d28a8c9.png)

##### CSS 分析

```
.btn-lg,
.btn-group-lg > .btn {
  padding: 10px 16px;
  font-size: 18px;
  line-height: 1.3333333;
  border-radius: 6px;
}
.btn-sm,
.btn-group-sm > .btn {
  padding: 5px 10px;
  font-size: 12px;
  line-height: 1.5;
  border-radius: 3px;
}
.btn-xs,
.btn-group-xs > .btn {
  padding: 1px 5px;
  font-size: 12px;
  line-height: 1.5;
  border-radius: 3px;
}
```
在Bootstrap框架中控制按钮的大小都是通过修改按钮的padding、line-height、font-size和border-radius几个属性

#### 块状按钮

每个示例中的按钮宽度都是依靠按钮文本和padding的值来决定。但有时候在制作按钮的时候需要按钮宽度充满整个父容器（width:100%），特别是在移动端的制作中。

##### 使用方法

添加 .btn-block

```
<button class="btn btn-primary btn-lg btn-block" type="button">大型按钮.btn-lg</button>
<button class="btn btn-primary btn-block" type="button">正常按钮</button>
<button class="btn btn-primary btn-sm btn-block" type="button">小型按钮.btn-sm</button>
<button class="btn btn-primary btn-xs btn-block" type="button">超小型按钮.btn-xs</button>
```
![](http://ojt6zsxg2.bkt.clouddn.com/0d46e8f4bd4669155c6d957817c111a2.png)

##### CSS 分析

```
.btn-block {
  display: block;
  width: 100%;
}
.btn-block + .btn-block {
  margin-top: 5px;
}
input[type="submit"].btn-block,
input[type="reset"].btn-block,
input[type="button"].btn-block {
  width: 100%;
}
```

#### 按钮状态

Bootstrap按钮的活动状态主要包括按钮的悬浮状态(:hover)，点击状态(:active)、焦点状态（:focus）和 disabled

##### disabled 使用方法

在Bootstrap框架中，要禁用按钮有两种实现方式：
1. 在标签中添加disabled属性
2. 在元素标签中添加类名“disabled”

```
<button class="btn btn-primary btn-lg btn-block" type="button" disabled="disabled">通过disabled属性禁用按钮</button> 
	<button class="btn btn-primary btn-block disabled" type="button">通过添加类名disabled禁用按钮</button>
	<button class="btn btn-primary btn-sm btn-block" type="button">未禁用的按钮</button>
```

同样的，其他风格按钮也具有这样的效果，只是颜色做了一定的调整

##### CSS 分析

```
.btn:focus,
.btn:active:focus,
.btn.active:focus,
.btn.focus,
.btn:active.focus,
.btn.active.focus {
  outline: 5px auto -webkit-focus-ring-color;
  outline-offset: -2px;
}
.btn:hover,
.btn:focus,
.btn.focus {
  color: #333;
  text-decoration: none;
}
.btn:active,
.btn.active {
  background-image: none;
  outline: 0;
  -webkit-box-shadow: inset 0 3px 5px rgba(0, 0, 0, .125);
          box-shadow: inset 0 3px 5px rgba(0, 0, 0, .125);
}
```

```
.btn.disabled,
.btn[disabled],
fieldset[disabled] .btn {
  cursor: not-allowed;
  filter: alpha(opacity=65);
  -webkit-box-shadow: none;
          box-shadow: none;
  opacity: .65;
}
```

### 表单控件大小

#### 控制表单控件的高度

Bootstrap框架还提供了两个不同的类名，用来控制表单控件的高度。这两个类名是：
1. input-sm:让控件比正常大小更小
2. input-lg:让控件比正常大小更大

两个类适用于表单中的**input，textarea和select**控件

##### 使用方法

```css
<input class="form-control input-lg" type="text" placeholder="添加.input-lg，控件变大">
<input class="form-control" type="text" placeholder="正常大小">
<input class="form-control input-sm" type="text" placeholder="添加.input-sm，控件变小">
```
![](http://ojt6zsxg2.bkt.clouddn.com/9acbe051a7628e9b59a13d9e1fa74b4c.png)

##### css 分析

```css
.input-sm {
  height: 30px;
  padding: 5px 10px;
  font-size: 12px;
  line-height: 1.5;
  border-radius: 3px;
}
select.input-sm {
  height: 30px;
  line-height: 30px;
}
textarea.input-sm,
select[multiple].input-sm {
  height: auto;
}
.input-lg {
  height: 46px;
  padding: 10px 16px;
  font-size: 18px;
  line-height: 1.3333333;
  border-radius: 6px;
}
select.input-lg {
  height: 46px;
  line-height: 46px;
}
textarea.input-lg,
select[multiple].input-lg {
  height: auto;
}
```
修改了 height、line-height、padding、font-size。
不管是“input-sm”还是“input-lg”仅对控件高度做了处理。

#### 控制表单空间的高度

需要控件宽度也要做一定的变化处理。这个时候就要借住Bootstrap框架的网格系统

##### 使用方法

```css
<form role="form" class="form-horizontal">
  <div class="form-group">
  <div class="col-xs-4">
    <input class="form-control input-lg" type="text" placeholder=".col-xs-4">
  </div>
  <div class="col-xs-4">
    <input class="form-control input-lg" type="text" placeholder=".col-xs-4">
  </div>
  <div class="col-xs-4">
    <input class="form-control input-lg" type="text" placeholder=".col-xs-4">
  </div>
  </div>
    …
</form>
```

### 表单控件状态

**表单有焦点的状态可以告诉用户可以输入或选择东西，禁用状态可以告诉用户不可以输入或选择东西，还有就是表单控件验证状态，可以告诉用户的操作是否正确等。**

#### 焦点状态

焦点状态是通过伪类“:focus”来实现。Bootstrap框架中表单控件的焦点状态删除了outline的默认样式，重新添加阴影效果。

#### 使用方法

要让控件在焦点状态下有上面样式效果，需要给控件添加类名“form-control”，这种方法适合 input、select、textarea 使用：

```css
<form role="form" class="form-horizontal">
  <div class="form-group">
    <div class="col-xs-6">
      <input class="form-control input-lg" type="text" placeholder="不是焦点状态下效果">
    </div>
    <div class="col-xs-6">
      <input class="form-control input-lg" type="text" placeholder="焦点点状态下效果">
    </div>
  </div>
</form>
```

![](http://ojt6zsxg2.bkt.clouddn.com/2124de1bab751578cff1fd353867aaf5.png)

file、radio和checkbox控件在焦点状态下的效果也与普通的input控件不太一样，不过不用添加类来添加 focus 效果。



##### css 分析

```css
.form-control:focus {
  border-color: #66afe9;
  outline: 0;
  -webkit-box-shadow: inset 0 1px 1px rgba(0,0,0,.075), 0 0 8px rgba(102, 175, 233, .6);
          box-shadow: inset 0 1px 1px rgba(0,0,0,.075), 0 0 8px rgba(102, 175, 233, .6);
}
```
file、radio、checkbox 的 focus 效果：
```css
input[type="file"]:focus,
input[type="radio"]:focus,
input[type="checkbox"]:focus {
  outline: 5px auto -webkit-focus-ring-color;
  outline-offset: -2px;
}
```
#### 禁用状态

Bootstrap框架的表单控件的禁用状态和普通的表单禁用状态实现方法是一样的，在相应的表单控件上添加属性“disabled”。

##### 使用方法

需要在需要禁用的表单控件上加上“disabled”即可：

```css
<input class="form-control" type="text" placeholder="表单已禁用，不能输入" disabled>
```

##### css 分析

```css
.form-control[disabled],
.form-control[readonly],
fieldset[disabled] .form-control {
  background-color: #eee;
  opacity: 1;
}
.form-control[disabled],
fieldset[disabled] .form-control {
  cursor: not-allowed;
}
```

```css
input[type="radio"][disabled],
input[type="checkbox"][disabled],
input[type="radio"].disabled,
input[type="checkbox"].disabled,
fieldset[disabled] input[type="radio"],
fieldset[disabled] input[type="checkbox"] {
  cursor: not-allowed;
}
```

#### 验证状态

在Bootstrap框架中同样提供这几种效果。
1. .has-warning:警告状态（黄色）
2. .has-error：错误状态（红色）
3. .has-success：成功状态（绿色）
使用的时候只需要在form-group容器上对应添加状态类名。

##### 使用方法

```css
<form role="form">
<div class="form-group has-success">
  <label class="control-label" for="inputSuccess1">成功状态</label>
  <input type="text" class="form-control" id="inputSuccess1" placeholder="成功状态" >
</div>
<div class="form-group has-warning">
  <label class="control-label" for="inputWarning1">警告状态</label>
  <input type="text" class="form-control" id="inputWarning1" placeholder="警告状态">
</div>
<div class="form-group has-error">
  <label class="control-label" for="inputError1">错误状态</label>
  <input type="text" class="form-control" id="inputError1" placeholder="错误状态">
</div>
</form>
```
![](http://ojt6zsxg2.bkt.clouddn.com/5d880e8eb7c8914024a73926f434447f.png)

在表单验证的时候，不同的状态会提供不同的 icon，比如成功是一个对号（√），错误是一个叉号（×）等。在Bootstrap框中也提供了这样的效果。如果你想让表单在对应的状态下显示 icon 出来，只需要在对应的状态下添加类名“has-feedback”。请注意，此类名要与“has-error”、“has-warning”和“has-success”在一起

使用方法如下：

```css
<form role="form">
  <div class="form-group has-success has-feedback">
    <label class="control-label" for="inputSuccess1">成功状态</label>
    <input type="text" class="form-control" id="inputSuccess1" placeholder="成功状态" >
    <span class="glyphicon glyphicon-ok form-control-feedback"></span>
  </div>
  <div class="form-group has-warning has-feedback">
    <label class="control-label" for="inputWarning1">警告状态</label>
    <input type="text" class="form-control" id="inputWarning1" placeholder="警告状态">
    <span class="glyphicon glyphicon-warning-sign form-control-feedback"></span>
  </div>
  <div class="form-group has-error has-feedback">
    <label class="control-label" for="inputError1">错误状态</label>
    <input type="text" class="form-control" id="inputError1" placeholder="错误状态">
    <span class="glyphicon glyphicon-remove form-control-feedback"></span>  
  </div>
</form>
```

**在 Bootstrap 的小图标都是使用@font-face来制作（后面的内容中将会着重用一节内容来介绍）。而且必须在表单中添加了一个 span 元素：**

![](http://ojt6zsxg2.bkt.clouddn.com/7431975a289976c22c691d15ad5274b6.png)

##### css 分析

```css
.has-success .help-block,
.has-success .control-label,
.has-success .radio,
.has-success .checkbox,
.has-success .radio-inline,
.has-success .checkbox-inline,
.has-success.radio label,
.has-success.checkbox label,
.has-success.radio-inline label,
.has-success.checkbox-inline label {
  color: #3c763d;
}
.has-success .form-control {
  border-color: #3c763d;
  -webkit-box-shadow: inset 0 1px 1px rgba(0, 0, 0, .075);
          box-shadow: inset 0 1px 1px rgba(0, 0, 0, .075);
}
.has-success .form-control:focus {
  border-color: #2b542c;
  -webkit-box-shadow: inset 0 1px 1px rgba(0, 0, 0, .075), 0 0 6px #67b168;
          box-shadow: inset 0 1px 1px rgba(0, 0, 0, .075), 0 0 6px #67b168;
}
.has-success .input-group-addon {
  color: #3c763d;
  background-color: #dff0d8;
  border-color: #3c763d;
}
.has-success .form-control-feedback {
  color: #3c763d;
}
```
主要是修改了 color、background-color和 border-color。

### 表单提示信息

#### 使用方法

使用了一个"help-block"样式，将提示信息以块状显示，并且显示在控件底部。

```html
<form role="form">
<div class="form-group has-success has-feedback">
  <label class="control-label" for="inputSuccess1">成功状态</label>
  <input type="text" class="form-control" id="inputSuccess1" placeholder="成功状态" >
  <span class="help-block">你输入的信息是正确的</span>
  <span class="glyphiconglyphicon-ok form-control-feedback"></span>
</div>
</form>
```

![](http://ojt6zsxg2.bkt.clouddn.com/e333b203d6a7d1357365ad9f65a0ae71.png)

或者使用了类名“help-inline”。一般让提示信息显示在控件的后面，也就是同一水平显示。但是这个类是在Bootstrap V2.x版本中才有。如果你想在BootstrapV3.x版本也有这样的效果，你可以添加这段代码：

```
.help-inline{
  display:inline-block;
  padding-left:5px;
  color: #737373;
}
```
 也可以使用 Bootstrap 的网格系统
 
```
<form role="form">
    <div class="form-group has-success">
        <label class="control-label" for="inputSuccess1">成功状态</label>
        <div class="row">
            <div class="col-xs-6">
                <input type="text" class="form-control" id="inputSuccess1" placeholder="成功状态">
            </div>
            <span class="col-xs-6 help-block">你输入的信息是正确的</span>
        </div>
    </div>
</form>
```
 
 ![](http://ojt6zsxg2.bkt.clouddn.com/cba154406f73590c611038c174f7097b.png)

### 图像

在Bootstrap框架中对于图像的样式风格提供以下几种风格：

1.  img-responsive：响应式图片，主要针对于响应式设计
2.  img-rounded：圆角图片
3.  img-circle：圆形图片
4.  img-thumbnail：缩略图片

#### 使用方法

```
<div class="container">
  <div class="row">
    <div class="col-sm-4">
      <img   alt="140x140" src="http://placehold.it/140x140">
        <div>默认图片</div>
    </div>
    <div class="col-sm-4">
      <img  class="img-rounded" alt="140x140" src="http://placehold.it/140x140"> 
        <div>圆角图片</div>
    </div>
    <div class="col-sm-4">
      <img  class="img-circle" alt="140x140" src="http://placehold.it/140x140">
        <div>圆形图片</div>
    </div>
      <div class="row">
        <div class="col-sm-6">
          <img  class="img-thumbnail" alt="140x140" src="http://placehold.it/140x140"> 
            <div>缩略图</div>
        </div>
        <div class="col-sm-6">
          <img  class="img-responsive" alt="140x140" src="http://placehold.it/140x140" /> 
          <div>响应式图片</div>
        </div>
      </div>
  </div>
</div>
```
![](http://ojt6zsxg2.bkt.clouddn.com/c5419c2a91dc19d384e86148d228a7b8.png)

#### css 分析

```
img {
  vertical-align: middle;
}
.img-responsive,
.thumbnail > img,
.thumbnail a > img,
.carousel-inner > .item > img,
.carousel-inner > .item > a > img {
  display: block;
  max-width: 100%;
  height: auto;
}
.img-rounded {
  border-radius: 6px;
}
.img-thumbnail {
  display: inline-block;
  max-width: 100%;
  height: auto;
  padding: 4px;
  line-height: 1.42857143;
  background-color: #fff;
  border: 1px solid #ddd;
  border-radius: 4px;
  -webkit-transition: all .2s ease-in-out;
       -o-transition: all .2s ease-in-out;
          transition: all .2s ease-in-out;
}
.img-circle {
  border-radius: 50%;
}
```

对于圆角图片和圆形图片效果，因为是使用了CSS3的圆角样式来实现的，所以注意对于IE8以及其以下版本不支持，是没有圆角效果的。

### 字体图标(font-face)

#### 使用方法

自定义完字体之后，需要对icon设置一个默认样式，在Bootstrap框架中是通过给元素添加“glyphicon”类名来实现，然后通过伪元素“:before”的“content”属性调取对应的icon编码。所有icon都是以”glyphicon-”前缀的类名开始，然后后缀表示图标的名称。

**所有图标可以在[这个链接](http://getbootstrap.com/components/#glyphicons)**

```
<span class="glyphicon glyphicon-search"></span>
<span class="glyphicon glyphicon-asterisk"></span>
<span class="glyphicon glyphicon-plus"></span>
<span class="glyphicon glyphicon-cloud"></span>
```
![](http://ojt6zsxg2.bkt.clouddn.com/5c45b6615fab0b56cca12a15ba3ec3ef.png)

#### css 分析

```
@font-face {
  font-family: 'Glyphicons Halflings';

  src: url('../fonts/glyphicons-halflings-regular.eot');
  src: url('../fonts/glyphicons-halflings-regular.eot?#iefix') format('embedded-opentype'), url('../fonts/glyphicons-halflings-regular.woff2') format('woff2'), url('../fonts/glyphicons-halflings-regular.woff') format('woff'), url('../fonts/glyphicons-halflings-regular.ttf') format('truetype'), url('../fonts/glyphicons-halflings-regular.svg#glyphicons_halflingsregular') format('svg');
}
.glyphicon {
  position: relative;
  top: 1px;
  display: inline-block;
  font-family: 'Glyphicons Halflings';
  font-style: normal;
  font-weight: normal;
  line-height: 1;

  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
```

## 网格系统

### 工作原理

首先：数据行(.row)必须包含在容器（.container）中，以便为其赋予合适的对齐方式和内距(padding)。如：

```
<div class="container">
<div class="row"></div>
</div>
```

其次：在行(.row)中可以添加列(.column)，但列数之和不能超过平分的总列数，比如12

```
<div class="container">
<div class="row">
  <div class="col-md-4"></div>
  <div class="col-md-8"></div>
```

然后：具体内容应当放置在列容器（column）之内，而且只有列（column）才可以作为行容器(.row)的直接子元素

最后：通过设置内距（padding）从而创建列与列之间的间距。然后通过为第一列和最后一列设置负值的外距（margin）来抵消内距(padding)的影响


![](http://img.mukewang.com/53b0f9c000018b9305540282.jpg)

上面这张图显示的是 Bootstrap 网格系统的工作原理，一共分为5步。

#### 第一步

最外边框，带有一大片白色区域，就是相当于浏览器的可视区域。在Bootstrap框架的网格系统中带有响应式效果，其带有四种类型的浏览器（超小屏(xs)，小屏(sm)，中屏(md)和大屏(lg)），其断点（像素的分界点）是768px、992px和1220px。

#### 第二步

第二个边框(1)相当于容器(.container)。针对不同的浏览器分辨率，其宽度也不一样：自动、750px、970px和1170px

```
.container {
  padding-right: 15px;
  padding-left: 15px;
  margin-right: auto;
  margin-left: auto;
}
@media (min-width: 768px) {
  .container {
    width: 750px;
  }
}
@media (min-width: 992px) {
  .container {
    width: 970px;
  }
}
@media (min-width: 1200px) {
  .container {
    width: 1170px;
  }
}
```

#### 第三步

２号横条阐述的是，将容器的行（.row）平分了12等份，也就是列。每个列都有一个“padding-left:15px”(图中粉红色部分)和一个“padding-right:15px”(图中紫色部分)。这样也导致了第一个列的padding-left和最后一列的padding-right占据了总宽度的30px，从而致使页面不美观，当然，如果你需要留有一定的间距，这个做法是不错的

```
.col-xs-1, .col-sm-1, .col-md-1, .col-lg-1, .col-xs-2, .col-sm-2, .col-md-2, .col-lg-2, .col-xs-3, .col-sm-3, .col-md-3, .col-lg-3, .col-xs-4, .col-sm-4, .col-md-4, .col-lg-4, .col-xs-5, .col-sm-5, .col-md-5, .col-lg-5, .col-xs-6, .col-sm-6, .col-md-6, .col-lg-6, .col-xs-7, .col-sm-7, .col-md-7, .col-lg-7, .col-xs-8, .col-sm-8, .col-md-8, .col-lg-8, .col-xs-9, .col-sm-9, .col-md-9, .col-lg-9, .col-xs-10, .col-sm-10, .col-md-10, .col-lg-10, .col-xs-11, .col-sm-11, .col-md-11, .col-lg-11, .col-xs-12, .col-sm-12, .col-md-12, .col-lg-12 {
  position: relative;
  min-height: 1px;
  padding-right: 15px;
  padding-left: 15px;
}
```

#### 第四步

３号横条就是行容器(.row),其定义了“margin-left”和”margin-right”值为”-15px”，用来抵消第一个列的左内距和最后一列的右内距。

```
.row {
  margin-right: -15px;
  margin-left: -15px;
}
```

#### 第五步

将行与列给合在一起就能看到横条4的效果。也就是我们期望看到的效果，第一列和最后一列与容器（.container）之间没有间距。

### 基本用法

![](http://img.mukewang.com/53e483500001c7f408770494.jpg)

上图是不同的网格大小之间的区别。

#### 使用方法

列组合简单理解就是更改数字来合并列（原则：列总和数不能超12）

```
<div class="container">
  <div class="row">
    <div class="col-md-4">.col-md-4</div>
    <div class="col-md-8">.col-md-8</div>
  </div>
  <div class="row">
    <div class="col-md-4">.col-md-4</div>
    <div class="col-md-4">.col-md-4</div>
    <div class="col-md-4">.col-md-4</div>
  </div>
  <div class="row">
    <div class="col-md-3">.col-md-3</div>
    <div class="col-md-6">.col-md-6</div>
    <div class="col-md-3">.col-md-3</div>
 </div>
</div>
```

#### css 分析

实现列组合方式非常简单，只涉及两个CSS两个特性：浮动与宽度百分比。

```
.col-md-1, .col-md-2, .col-md-3, .col-md-4, .col-md-5, .col-md-6, .col-md-7, .col-md-8, .col-md-9, .col-md-10, .col-md-11, .col-md-12 {
    float: left;
 }
 .col-md-12 {
    width: 100%;
  }
  .col-md-11 {
    width: 91.66666667%;
  }
  .col-md-10 {
    width: 83.33333333%;
  }
  .col-md-9 {
    width: 75%;
  }
  .col-md-8 {
    width: 66.66666667%;
  }
  .col-md-7 {
    width: 58.33333333%;
  }
  .col-md-6 {
    width: 50%;
  }
  .col-md-5 {
    width: 41.66666667%;
  }
  .col-md-4 {
    width: 33.33333333%;
  }
  .col-md-3 {
    width: 25%;
  }
  .col-md-2 {
    width: 16.66666667%;
  }
  .col-md-1 {
    width: 8.33333333%;
  }
```

### 列偏移

我们不希望相邻的两个列紧靠在一起，但又不想使用margin或者其他的技术手段来。这个时候就可以使用列偏移（offset）功能来实现。

#### 使用方法

需要在列元素上添加类名“col-md-offset-*”(其中星号代表要偏移的列组合数)，那么具有这个类名的列就会向右偏移。例如，你在列元素上添加“col-md-offset-4”，表示该列向右移动4个列的宽度。

```
<h4>列向右移动四列的间距</h4>
<div class="container">
  <div class="row">
    <div class="col-md-4">.col-md-4</div>
    <div class="col-md-2 col-md-offset-4">列向右移动四列</div>
    <div class="col-md-2">.col-md-3</div>
  </div>
  <div class="row">
    <div class="col-md-4">.col-md-4</div>
    <div class="col-md-4 col-md-offset-4">列向右移动四列</div>
  </div>
</div>
<br />
<h4>发生行断裂</h4>
<div class="container">
  <div class="row">
    <div class="col-md-4">.col-md-4</div>
    <div class="col-md-2 col-md-offset-4">列向右移动四列</div>
    <div class="col-md-2">.col-md-3</div>
  </div>
  <div class="row">
    <div class="col-md-4">.col-md-4</div>
    <div class="col-md-4 col-md-offset-4">列向右移动四列</div>
  </div>
  <div class="row">
    <div class="col-md-3">.col-md-3</div>
    <div class="col-md-3 col-md-offset-3">col-md-offset-3</div>
    <div class="col-md-4">col-md-4</div>
  </div>
</div>
```
![](http://ojt6zsxg2.bkt.clouddn.com/76565b67a3141c8042fbb2f62510c024.png)

**使用”col-md-offset-*”对列进行向右偏移时，要保证列与偏移列的总数不超过12，不然会致列断行显示**

#### css 分析

```
.col-md-offset-12 {
   margin-left: 100%;
}
  .col-md-offset-11 {
    margin-left: 91.66666667%;
  }
  .col-md-offset-10 {
    margin-left: 83.33333333%;
...
  .col-md-offset-2 {
    margin-left: 16.66666667%;
  }
  .col-md-offset-1 {
    margin-left: 8.33333333%;
  }
  .col-md-offset-0 {
    margin-left: 0;
  }
```










