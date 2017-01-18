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

### 单选按钮和复选按钮

#### 使用方法

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

### 表单控件











