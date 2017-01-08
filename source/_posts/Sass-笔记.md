---
title: ' Sass 笔记'
date: 2016-12-29 22:30:32
categories: coding
tags:
  - Sass
---
#  Sass 笔记

本系列的是参考[慕课网 大漠](http://www.imooc.com/u/104044/courses?sort=publish)老师的视频，仅供个人查阅使用，具体讲解推荐参考视频。

## Sass 简介

**CSS 预处理器**定义了一种新的语言，其基本思想是，用一种专门的编程语言，为 CSS 增加了一些编程的特性

### Sass 和 SCSS 的区别

Sass 和 SCSS 其实是同一种东西，我们平时都称之为 Sass，两者之间不同之处有以下两点：

1. 文件扩展名不同，Sass 是以“.sass”后缀为扩展名，而 SCSS 是以“.scss”后缀为扩展名
2. 语法书写方式不同，**Sass 是以严格的缩进式语法规则来书写，不带大括号({})和分号(;)，而 SCSS 的语法书写和我们的 CSS 语法书写方式非常类似**。

**Sass 是基于 Ruby 环境运行的，如要要运行 Sass，首先要安装 Ruby。**

<!--more-->

## Sass 语法以及编译调试

**Sass 的语法格式和 SCSS 的语法格式，他们的功能都是一样的，不同的是其书写格式和文件扩展名不同**

### 命令编译

#### 单文件编译

	sass <要编译的Sass文件路径>/style.scss:<要输出CSS文件路径>/style.css

例如：
	
	sass test.scss:test.css

#### 多文件编译

针对文件夹的编译：

	sass sass/:css/

#### 监控自动编译

**重新保存了修改的文件，命令终端就能监测，并重新编译出文件**

	sass --watch <要编译的Sass文件路径>/style.scss:<要输出CSS文件路径>/style.css

**但是这种方法，如果文件夹或文件的名称里面有中文，会出现问题。**

### 输出样式风格

#### 嵌套输出方式 nested

在编译的时候带上参数“ --style nested”

	sass --watch test.scss:test.css --style nested

风格为：

	nav ul {
	  margin: 0;
	  padding: 0;
	  list-style: none; }
	nav li {
	  display: inline-block; }
	nav a {
	  display: block;
	  padding: 6px 12px;
	  text-decoration: none; }

#### 展开输出方式 expanded  

在编译的时候带上参数“ --style expanded”

	sass --watch test.scss:test.css --style expanded

风格为：

	nav ul {
	  margin: 0;
	  padding: 0;
	  list-style: none;
	}
	nav li {
	  display: inline-block;
	}
	nav a {
	  display: block;
	  padding: 6px 12px;
	  text-decoration: none;
	}

#### 紧凑输出方式 compact 

在编译的时候带上参数“ --style compact”

	sass --watch test.scss:test.css --style compact

风格为：

	nav ul { margin: 0; padding: 0; list-style: none; }
	nav li { display: inline-block; }
	nav a { display: block; padding: 6px 12px; text-decoration: none; }

#### 压缩输出方式 compressed

在编译的时候带上参数“ --style compressed”

	sass --watch test.scss:test.css --style compressed

风格为：

	nav ul{margin:0;padding:0;list-style:none}nav li{display:inline-block}nav a{display:block;padding:6px 12px;text-decoration:none}

### Sass 的调试

**可以使用流量器的 sourcemap 功能**

## Sass 的@规则

### @import

Sass 扩展了 CSS 的 @import 规则，让它能够引入 SCSS 和 Sass 文件。 

被导入的文件中所定义的变量或 mixins 都可以在主文件中使用。

Sass 会在当前目录下寻找其他 Sass 文件， 如果是 Rack、Rails 或 Merb 环境中则是 Sass 文件目录。 也可以通过 :load_paths 选项或者在命令行中使用 --load-path 选项来指定额外的搜索目录。

### @media

Sass 中的 @media 指令和 CSS 的使用规则一样

### @extend

Sass 中的 @extend 是用来扩展选择器（不止是类选择器）或占位符。

### @at-root
@at-root 从字面上解释就是跳出根元素。当你选择器嵌套多层之后，想让某个选择器跳出，此时就可以使用 @at-root。

```css
.a {
  color: red;

  .b {
    color: orange;

    .c {
      color: yellow;

      @at-root .d {
        color: green;
      }
    }
  }  
}
```
编译的结果为：

```css
.a {
  color: red;
}

.a .b {
  color: orange;
}

.a .b .c {
  color: yellow;
}

.d {
  color: green;
}
```

### @debug(@warn、@error)

@debug 在 Sass 中是用来调试的，当你的在 Sass 的源码中使用了 @debug 指令之后，Sass 代码在编译出错时，在命令终端会输出你设置的提示 Bug:



## Sass 基础

**普通变量：**普通变量定义以后可以在全局范围内使用

**默认变量：**默认变量一般是用来设置默认值，然后根据需求来覆盖的，**默认变量在组件化开发的时候非常有用。**

	$baseLineHeight: 2;
	$baseLineHeight: 1.5 !default;
	body{
	    line-height: $baseLineHeight; 
	}

编译后为：

	body{
	    line-height:2;
	}

### 局部变量与全局变量

**在选择器、函数、混合宏...的外面定义的变量为全局变量**

**定义在元素内部的变量，比如 $color:red; 是一个局部变量。**

例如：

	//SCSS
	$color: orange !default;//定义全局变量(在选择器、函数、混合宏...的外面定义的变量为全局变量)
	.block {
	  color: $color;//调用全局变量
	}
	em {
	  $color: red;//定义局部变量
	  a {
	    color: $color;//调用局部变量
	  }
	}
	span {
	  color: $color;//调用全局变量
	}

编译的结果为：

	//CSS
	.block {
	  color: orange;
	}
	em a {
	  color: red;
	}
	span {
	  color: orange;
	}

### 什么时候声明变量

创建变量只适用于感觉确有必要的情况下。不要为了某些骇客行为而声明新变量，这丝毫没有作用。只有满足所有下述标准时方可创建新变量：

1. 该值至少重复出现了两次；
2. 该值至少可能会被更新一次；
3. 该值所有的表现都与变量有关（非巧合）。

### 选择器嵌套

Sass 的嵌套分为三种：

1. 选择器嵌套
2. 属性嵌套
3. 伪类嵌套

#### 选择器嵌套的代词(`&`)

`&`：在选择器嵌套中代表当前所有祖先嵌套关系，如：

	nav {
	    a{
	        color:red;
	        header & {
	            color: green;
	        }
	    }
	}

编译结果是：

	nav a {
	   color: red;
	}
	
	header nav a {
	   color: green;
	}

#### 属性嵌套

CSS 有一些**属性前缀相同，只是后缀不一样**，比如：border-top/border-right，与这个类似的还有 margin、padding、font 等属性
```sass
    .box {
	  border: {
	   top: 1px solid red;
	   bottom: 1px solid green;
	  }
	}
```
编译出来是：

	.box {
	    border-top: 1px solid red;
	    border-bottom: 1px solid green;
	}

#### 伪类嵌套

其实伪类嵌套和属性嵌套非常类似，只不过他**需要借助`&`符号一起配合使用**

	.clearfix{
    	&:before, &:after {
    	    content:"";
    	    display: table;
    	}
    	&:after {
    	    clear:both;
    	    overflow: hidden;
    	}
	}

编译结果是：

	clearfix:before, .clearfix:after {
	  content: "";
	  display: table;
	}
	.clearfix:after {
	  clear: both;
	  overflow: hidden;
	}

选择器嵌套最大的问题是将使最终的代码难以阅读，我们应该尽可能**避免选择器嵌套**

### 混合宏

需要重复使用大段的样式时，使用变量就无法达到我们目的，此时需要使用宏

#### 声明混合宏

##### 不带参数的混合宏

	@mixin border-radius{
	    -webkit-border-radius: 5px;
	    border-radius: 5px;
	}

##### 带参数的混合宏

参数中的 `:5px`是该参数的默认值

	@mixin border-radius($radius:5px){
	    -webkit-border-radius: $radius;
	    border-radius: $radius;
	}

可以是几个参数或者是不知道数量的参数：

	@mixin center($width,$height){
	    width: $width;
	    height: $height;
	    position: absolute;
	    top: 50%;
	    left: 50%;
	    margin-top: -($height) / 2;
	    margin-left: -($width) / 2;
	}

有一个特别的参数“…”。当混合宏传的参数过多之时，可以使用参数来替代

##### 带有逻辑的混合宏
	
`...`：代表有多个参数

`@include`：代表调用声明好的混合宏

	@mixin box-shadow($shadows...){
	  @if length($shadows) >= 1 {
	    -webkit-box-shadow: $shadows;
	    box-shadow: $shadows;
	  } @else {
	    $shadows: 0 0 2px rgba(#000,.25);
	    -webkit-box-shadow: $shadow;
	    box-shadow: $shadow;
	  }
	}

调用方法为：
```sass
	.box {
	  @include box-shadow(0 0 1px rgba(#000,.5),0 0 2px rgba(#000,.2));
	}
```

编译结果为：

	.box {
	  -webkit-box-shadow: 0 0 1px rgba(0, 0, 0, 0.5), 0 0 2px rgba(0, 0, 0, 0.2);
	  box-shadow: 0 0 1px rgba(0, 0, 0, 0.5), 0 0 2px rgba(0, 0, 0, 0.2);
	}

#### 调用混合宏

	@mixin border-radius($radius:6px) {
	    -webkit-border-radius: $radius;
	    border-radius: $radius;
	}
	
	body{
	    @include border-radius(20px);
	}

编译的结果是：

	body {
	  -webkit-border-radius: 20px;
	  border-radius: 20px; 
	}

#### 混合宏的不足

混合宏在实际编码中给我们带来很多方便之处，特别是对于**复用重复代码块**。但其最大的**不足之处**是会**生成冗余的代码块**。

	@mixin border-radius{
	  -webkit-border-radius: 3px;
	  border-radius: 3px;
	}
	
	.box {
	  @include border-radius;
	  margin-bottom: 5px;
	}
	
	.btn {
	  @include border-radius;
	}

编译结果为：

	.box {
	  -webkit-border-radius: 3px;
	  border-radius: 3px;
	  margin-bottom: 5px;
	}
	
	.btn {
	  -webkit-border-radius: 3px;
	  border-radius: 3px;
	}

Sass 在调用相同的混合宏时，**并不能智能的将相同的样式代码块合并在一起**

### 继承

在 Sass 中也具有继承一说，也是继承类中的样式代码块。在 Sass 中是通过关键词 `@extend` 来继承已存在的类样式块，从而实现代码的继承。

**使用继承可以实现将相同样式的代码块组合在一起**

	.btn {
	  border: 1px solid #ccc;
	  padding: 6px 10px;
	  font-size: 14px;
	}
	
	.btn-primary {
	  background-color: #f36;
	  color: #fff;
	  @extend .btn;
	}
	
	.btn-second {
	  background-color: orange;
	  color: #fff;
	  @extend .btn;
	}

编译后显示为：

	.btn, .btn-primary, .btn-second {
	  border: 1px solid #ccc;
	  padding: 6px 10px;
	  font-size: 14px;
	}
	
	.btn-primary {
	  background-color: #f36;
	  color: #fff;
	}
	
	.btn-second {
	  background-clor: orange;
	  color: #fff;
	}

#### 继承与占位符

 %placeholder 声明的代码，如果不被 @extend 调用的话，不会产生任何代码。

**@extend 调用的占位符，编译出来的代码会将相同的代码合并在一起**

使用占位符的方法，比直接继承其他选择符中所有的样式更加灵活。

	%mt5 {
	  margin-top: 5px;
	}
	%pt5{
	  padding-top: 5px;
	}
	
	.btn {
	  @extend %mt5;
	  @extend %pt5;
	}
	
	.block {
	  @extend %mt5;
	
	  span {
	    @extend %pt5;
	  }
	}

编译结果为：

	.btn, .block {
	  margin-top: 5px;
	}
	
	.btn, .block span {
	  padding-top: 5px;
	}

### 混合宏 VS 继承 VS 占位符

**混合宏的优点**：可以用来传递参数，如果你的代码块中涉及到变量，建议使用混合宏来创建相同的代码块。

**混合宏的缺点**：如果在样式文件中调用同一个混合宏，会产生多个对应的样式代码，造成代码的冗余

**继承的优点**： CSS 会将使用继承的代码块合并到一起，通过组合选择器的方式向大家展现，如果你的代码块不需要专任何变量参数，而且有一个基类已在文件中存在，那么建议使用 Sass 的继承。

**继承的缺点**：不能传变量参数，**继承是首先有一个基类存在，不管调用与不调用，基类的样式都将会出现在编译出来的 CSS 代码中。**

**占位符的优点**：编译出来的 CSS 代码和使用继承基本上是相同，只是不会在代码中生成占位符 mt 的选择器。

**占位符的缺点**：不能传变量参数。

**在占位符中也可以使用混合宏，用来解决不能传递参数的缺点。**

### 插值#{}

**差值是很好用的一种传递字符串的方法，这种方法除了传递参数，还可以传递`CSS`属性名**

**一般使用插值的时候，是要将变量的值插入到某些属性名或属性值中，与其他的字符一起组成完整的属性名或变量名**

下面的例子中，除了使用差值`#{}`传递值以外，还传递了属性名`top`：

```sass
$properties: (margin, padding);
@mixin set-value($side, $value) {
	@each $prop in $properties {
		#{$prop}-#{$side}: $value;
	}
}

.log-box {
	@include set-value(top, 14px);
}
```

编译出来的结果为：

```css
.log-box {
  margin-top: 14px;
  padding-top: 14px; }
```

**但是在一些地方无法使用差值，比如在宏（`@mixin`）中**

在宏 `@mixin` 的定义中有使用到插值 `#{size}` 来构成**其他变量**的名字，编译无法通过。

例如下面的例子：

```sass
$margin-big: 40px;
$margin-medium: 20px;
$margin-small: 12px;
@mixin set-value($size) {
    margin-top: $margin-#{$size};
}
.login-box {
    @include set-value(big);
}
```

但是如果是直接使用插值作为选择器，则**编译可以通过**：

```Sass
@mixin firefox-message($selector) {
  body.firefox #{$selector}:before {
    content: "Hi, Firefox users!";
  }
}
@include firefox-message(".header");
```

下面的例子中显示，插值也不能在 `@mixin` 名字的构成中调用：

```sass
@mixin updated-status {
    margin-top: 20px;
    background: #F00;
}
$flag: "status";
.navigation {
    @include updated-#{$flag};
}
```

但是插值可以在 `@include`中调用：

```sass
%updated-status {
    margin-top: 20px;
    background: #F00;
}
.selected-status {
    font-weight: bold;
}
$flag: "status";
.navigation {
    @extend %updated-#{$flag};
    @extend .selected-#{$flag};
}
```

### 注释

在 `Sass` 中，可以使用类似 CSS 的 `/* 这是注释 */` 的注释，也可以使用 JavaScript 中的 `//` 注释

**不过，使用`/* 这是注释 */` 包裹的注释在编译完成后可以在 CSS 文件中显示，但是 `//` 包裹的注释不会显示**

### Sass 数据类型

在 Sass 中包含以下几种数据类型：

1. 数字: 如，1、 2、 13、 10px；
2. 字符串：有引号字符串或无引号字符串，如，"foo"、 'bar'、 baz，**在使用字符串类型的时候，一定要注意什么时候是字符串类型，什么时候是其他的类型，不能混用**；
3. 颜色：如，blue、 #04a3f9、 rgba(255,0,0,0.5)；
4. 布尔型：如，true、 false；
5. 空值：如，null；
6. 值列表：用空格或者逗号分开，如，1.5em 1em 0 2em 、 Helvetica, Arial, sans-serif。

#### Sass 字符串

SassScript 支持 CSS 的两种字符串类型：

1. 有引号字符串 (quoted strings)，如 "Lucida Grande" 、'http://sass-lang.com'；
2. 无引号字符串 (unquoted strings)，如 sans-serifbold。

#### Sass 值列表

列表 (lists) 是指 Sass 如何处理 CSS 中： 
```
margin: 10px 15px 0 0
```
或者： 
```
font-face: Helvetica, Arial, sans-serif
```
像上面这样通过空格或者逗号分隔的一系列的值。

事实上，独立的值也被视为值列表——只包含一个值的值列表。

Sass列表函数（Sass list functions）赋予了值列表更多功能（Sass进级会有讲解）：

* nth函数（nth function） 可以直接访问值列表中的某一项；
* join函数（join function） 可以将多个值列表连结在一起；
* append函数（append function） 可以在值列表中添加值； 
* @each规则（@each rule） 则能够给值列表中的每个项目添加样式。

**1px 2px, 5px 6px 是包含 1px 2px 与 5px 6px 两个值列表的值列表，也可以写成(1px 2px) (5px 6px)。但是当值列表被编译为 CSS 时，Sass 不会添加任何圆括号，因为 CSS 不允许这样做。(1px 2px) (5px 6px)与 1px 2px 5px 6px 在编译后的 CSS 文件中是一样的，但是它们在 Sass 文件中却有不同的意义，前者是包含两个值列表的值列表，而后者是包含四个值的值列表。**

## Sass 运算

### 加法与减法

CSS 中能做运算的，到目前为止仅有 calc() 函数可行

加法运算是 Sass 中运算中的一种，在变量或属性中都可以做加法运算

```Sass
$sidebar-width: 220px;
$content-width: 720px;
$gap-width: 20px;

.container {
    width: $sidebar-width + $content-width + $gap-width;
    margin: 0 auto;
}
```

### 乘法

如果进行乘法运算时，两个值单位相同时，只需要为一个数值提供单位即可。

```
.box {
  width:10px * 2px;  
}
```

需要写成
 
```css
.box {
  width:10px * 2;  
}
```

### 除法

`/`符号在 CSS 中已做为一种符号使用。因此在 Sass 中做除法运算时，直接使用`/`符号做为除号时，将不会生效，编译时既得不到我们需要的效果，也不会报错。

```css
body{
    font-size:8px/10px;
}
```

需要给运算的外面添加一个小括号( )

```css
box {
  width: (100px / 2);  
}
```

**如果是对变量使用除法 `/`，则不需要使用 ( )，除法会被自动识别。**

在除法运算时，如果两个值带有相同的单位值时，除法运算之后会得到一个不带单位的数值：

```
.box {
  width: (1000px / 100px);
}
```

编译的结果是：

```css
.box {
  width: 10;
}
```

### 颜色运算

所有算数运算都支持颜色值，并且是分段运算的。也就是说，红、绿和蓝各颜色分段单独进行运算。

```css
p {
  color: #010203 + #040506;
}
```

### 字符串运算

在 Sass 中可以通过加法符号“+”来对字符串进行连接

**如果是有引号的字符串和没有引号的文本进行 "+" 运算，则自动转换为字符串运算，如果是和变量进行运算，则变量会被转换为其表示的内容，然后进行字符串拼接运算。**

```css
$font-size: 12px;
body{
	font-size: "8px" + font-size;
}
```
编译结果为：
```css
body {
  font-size: "8pxfont-size"; }
```

如果是和变量进行运算：

```css
$font-size: 12px;
body{
	font-size: "8px" + $font-size;
}
```
编译的结果为：
```css
body {
  font-size: "8px12px"; }
```

## Sass 逻辑命令

### @if命令

在 Sass 中除了 @if 之，还可以配合 @else if 和 @else 一起使用。

```css
@mixin backgroundRed($boolean:true) {
	@if $boolean {
		background-color:red;
	} @else {
		background-color:green;
	}
}

.red {
	@include backgroundRed(true);
}

.notRed {
	@include backgroundRed(false);
}
```

### @for 命令

在 Sass 的 @for 循环中有两种方式：

```css
@for $i from <start> through <end>
@for $i from <start> to <end>
```

这两个的区别是**关键字 through 表示包括 end 这个数，而 to 则不包括 end 这个数**。

```css
@mixin backgroundRed($boolean, $start, $end) {
	@if $boolean {
		@for $i from $start to $end {
			.item-#{$i} {background-color: red;}
		}
	} @else {
		@for $i from $start to $end {
			.item-#{$i} {background-color: green;}
		}
	}
}

.br {
	@include backgroundRed(true, 1, 3);
}

.bnr{
	@include backgroundRed(false, 2, 3);
}
```

编译的结果为：

```css
.br .item-1 {
  background-color: red; }
.br .item-2 {
  background-color: red; }

.bnr .item-2 {
  background-color: green; }
```

在上面的例子中，有很多冗余的代码，可以使用继承将冗余的代码提出，这样编译生成的 CSS 中就不会包含冗余代码。如下：

```css
%bcr {background-color: red;}
%bcg {background-color: green;}
@mixin backgroundRed($boolean, $start, $end) {
	@if $boolean {
		@for $i from $start to $end {
			.item-#{$i} {@extend %bcr;}
		}
	} @else {
		@for $i from $start to $end {
			.item-#{$i} {@extend %bcg;}
		}
	}
}

.br {
	@include backgroundRed(true, 1, 3);
}

.bnr{
	@include backgroundRed(false, 2, 3);
}
```
编译的结果为：
```css
.br .item-1, .br .item-2 {
  background-color: red; }

.bnr .item-2 {
  background-color: green; }
```


下面的例子是`@for`应用在网格系统生成各个格子 class 的代码
```css
$grid-prefix: span !default;
$grid-width: 60px !default;
$grid-gutter: 20px !default;

%grid {
  float: left;
  margin-left: $grid-gutter / 2;
  margin-right: $grid-gutter / 2;
}
@for $i from 1 through 12 {
  .#{$grid-prefix}#{$i}{
    width: $grid-width * $i + $grid-gutter * ($i - 1);
    @extend %grid;
  }  
}
```

### @while 循环

只要 @while 后面的条件为 true 就会执行。

```css
//SCSS
$types: 4;
$type-width: 20px;

@while $types > 0 {
    .while-#{$types} {
        width: $type-width + $types;
    }
    $types: $types - 1; /*修改 $type 的值*/
}
```

### @each 循环

@each 循环就是去遍历一个列表，然后从列表中取出对应的值。

@each 循环指令的形式：

```css
@each $var in <list>
```

```css
$list: adam john wynn mason kuroir;//$list 就是一个列表

@mixin author-images {
    @each $author in $list {
        .photo-#{$author} {
            background: url("/images/avatars/#{$author}.png") no-repeat;
        }
    }
}

.author-bio {
    @include author-images;
}
```

## Sass 函数

```
sass -i
```

在命令终端开启这个命令，相当于开启 Sass 的函数计算。

在 Sass 中除了可以定义变量，具有 @extend、%placeholder 和 mixins 等特性之外，还自备了一系列的函数功能。其主要包括：

* 字符串函数
* 数字函数
* 列表函数
* 颜色函数
* Introspection 函数
* 三元函数等

除此之外还可以自定义函数。

### 字符串函数

* `unquote($string)`：删除字符串中的引号，unquote( ) 函数只能删除字符串最前和最后的引号（双引号或单引号），而无法删除字符串中间的引号。如果字符没有带引号，返回的将是字符串本身。
* `quote($string)`：给字符串添加引号使用 quote() 函数只能给字符串增加双引号，而且**字符串中间有单引号或者空格时，需要用单引号或双引号括起，否则编译的时候将会报错**。同时 quote() 碰到特殊符号，比如： !、?、> 等，除中折号 - 和 下划线_ 都需要使用双引号括起，否则编译器在进行编译的时候同样会报错。
* `To-upper-case()` ：函数将字符串小写字母转换成大写字母。
* `To-lower-case()`：函数将字符串转换成小写字母。


```css
.test1 {
    content:  quote(Hello Sass);
}
```
编译会报错，需要改成 `quote("Hello Sass")`

### 数字函数

**这些数字函数的参数中如果带有单位，则计算之后单位也会被保留。**

* percentage($value)：将一个不带单位的数转换成百分比值；
* round($value)：将数值四舍五入，转换成一个最接近的整数；
* ceil($value)：将大于自己的小数转换成下一位整数；
* floor($value)：将一个数去除他的小数部分；
* abs($value)：返回一个数的绝对值；
* min($numbers…)：找出几个数值之间的最小值，**同时出现两种不同类型的单位，将会报错误信息**；
* max($numbers…)：找出几个数值之间的最大值，**同时出现两种不同类型的单位，将会报错误信息**；
* random(): 获取随机数，0到1之间

### 列表函数

* `length($list)`：返回一个列表的长度值, **`length()` 函数中的列表参数之间使用空格隔开，不能使用逗号，否则函数将会出错**。如果传入的是一个列表变量，变量中用逗号隔开是可以的。；
* `nth($list, $n)`：返回一个列表中指定的某个标签值。1 是指列表中的第一个标签值，2 是指列给中的第二个标签值。
* `join($list1, $list2, [$separator])`：将**两个列表**给连接在一起，变成一个列表。如果列表中有表示颜色的值，则所有的表示颜色的值都会变成十六进制表示。`[$separator]`的值可以是 `space`或`comma`；
* `append($list1, $val, [$separator])`：将某个值放在列表的最后；
* `zip($lists…)`：将几个列表结合成一个多维的列表，**在使用zip()函数时，每个单一的列表个数值必须是相同的**；
* `index($list, $value)`：返回一个值在列表中的位置值，第一个值就是1，第二个值就是 2。

### Introspection函数（判断函数）

几个函数主要用来对值做一个判断的作用

* `type-of($value)`：返回一个值的类型：number 为数值型。string 为字符串型。bool 为布尔型。color 为颜色型。
* `unit($number)`：返回一个值的单位
* `unitless($number)`：判断一个值是否带有单位
* `comparable($number-1, $number-2)`：判断两个值是否可以做加、减和合并

### Miscellaneous函数（三元函数）

```css
if($condition,$if-true,$if-false)
```

上面表达式的意思是当 `$condition` 条件成立时，返回的值为 `$if-true`，否则返回的是 `$if-false `值。

### Map

```css
$map: (
    $key1: value1,
    $key2: value2,
    $key3: value3
)
```

map 的嵌套实用性也非常的强，大家可能有碰到过**换皮肤**的项目，可能每一套皮肤对应的颜色蛮多的，那么使用此功能来管理颜色的变量就非常的有条理性，便于维护与管理。

#### Map 函数

* `map-get($map,$key)`：根据给定的 key 值，返回 map 中相关的值,如果 $key 不存在 `$map`中，将返回 `null` 值。
* `map-merge($map1,$map2)`：将两个 map 合并成一个新的 map。
* `map-remove($map,$key)`：从 map 中删除一个 key，返回一个新 map。**如果 `$map1` 和 `$map2` 中有相同的 `$key` 名，那么将 `$map2` 中的 `$key` 会取代 `$map1` 中**。
* `map-keys($map)`：返回 map 中所有的 key。
* `map-values($map)`：返回 map 中所有的 value。
* `map-has-key($map,$key)`：根据给定的 key 值判断 map 是否有对应的 value 值，如果有返回 true，否则返回 false。
* `keywords($args)`：返回一个函数的参数，这个参数可以动态的设置 key 和 value。

**`keywords($args)`**

keywords($args) 函数可以说是一个动态创建 map 的函数。可以通过混合宏或函数的参数变创建 map。参数也是成对出现，其中 $args 变成 key(会自动去掉$符号)，而 $args 对应的值就是value。

```css
@mixin map($args...){
    @debug keywords($args);
}

@include map(
  $dribble: #ea4c89,
  $facebook: #3b5998,
  $github: #171515,
  $google: #db4437,
  $twitter: #55acee
);
```

debug 的结果是：

```
DEBUG: (dribble: #ea4c89, facebook: #3b5998, github: #171515, google: #db4437, twitter: #55acee)
```

**遍历获得 Map 中参数**
在下面的例子中，自定义了一个函数，如果没有在map 中找到对应的值，在编译 Sass 的时候会返回一个警告。
```css
$social-colors: (
    dribble: #ea4c89,
    facebook: #3b5998,
    github: #171515,
    google: #db4437,
    twitter: #55acee
);
@function colors($color){
    @if not map-has-key($social-colors,$color){
        @warn "No color found for `#{$color}` in $social-colors map. Property omitted.";
    }
    @return map-get($social-colors,$color);
}
.btn-dribble {
    color: colors(dribble);
}
.btn-facebook {
    color: colors(facebook);
}
.btn-github {
    color: colors(github);
}
.btn-google {
    color: colors(google);
}
.btn-twitter {
    color: colors(twitter);
}
.btn-weibo {
    color: colors(weibo);
}
```

也可以使用 @each 遍历 map：

```css
$social-colors: (
    dribble: #ea4c89,
    facebook: #3b5998,
    github: #171515,
    google: #db4437,
    twitter: #55acee
);
@function colors($color){
    @if not map-has-key($social-colors,$color){
        @warn "No color found for `#{$color}` in $social-colors map. Property omitted.";
    }
    @return map-get($social-colors,$color);
}
@each $social-network,$social-color in $social-colors {
    .btn-#{$social-network} {
        color: colors($social-network);
    }
}
```
### 颜色函数

#### RGB颜色函数

* `rgb($red,$green,$blue)`：根据红、绿、蓝三个值创建一个颜色；
* `rgba($red,$green,$blue,$alpha)`：根据红、绿、蓝和透明度值创建一个颜色；
* `red($color)`：从一个颜色中获取其中红色值；
* `green($color)`：从一个颜色中获取其中绿色值；
* `blue($color)`：从一个颜色中获取其中蓝色值；
* `mix($color-1,$color-2,[$weight])`：把两种颜色混合在一起。`$weight` 为 合并的比例（选择权重），默认值为 50%，其取值范围是 0~1 之间。。


**`rgba()`函数的使用场景：**假设在实际中知道的颜色值是 #f36 或者 red，但在使用中，需要给他们配上一个透明度

```css
$color: #f36;
.rbga{
	color: rgba($color,0.5)
}
```
#### HSL颜色函数

* `hsl($hue,$saturation,$lightness)`：通过色相（hue）、饱和度(* saturation)和亮度（lightness）的值创建一个颜色；
* `hsla($hue,$saturation,$lightness,$alpha)`：通过色相（hue）、饱和度(saturation)、亮度（lightness）和透明（alpha）的值创建一个颜色；
* `hue($color)`：从一个颜色中获取色相（hue）值；
* `saturation($color)`：从一个颜色中获取饱和度（saturation）值；
* `lightness($color)`：从一个颜色中获取亮度（lightness）值；
* `adjust-hue($color,$degrees)`：通过改变一个颜色的色相值，创建一个新的颜色；
* `lighten($color,$amount)`：通过改变颜色的亮度值，让颜色变亮，创建一个新的颜色；
* `darken($color,$amount)`：通过改变颜色的亮度值，让颜色变暗，创建一个新的颜色；
* `saturate($color,$amount)`：通过改变颜色的饱和度值，让颜色更饱和，从而创建一个新的颜色
* `desaturate($color,$amount)`：通过改变颜色的饱和度值，让颜色更少的饱和，从而创建出一个新的颜色；
* `grayscale($color)`：将一个颜色变成灰色，相当于`desaturate($color,100%)`;
* `complement($color)`：返回一个补充色，相当于`adjust-hue($color,180deg)`;
* `invert($color)`：反回一个反相色，红、绿、蓝色值倒过来，而透明度不变。

#### Opacity函数

* `alpha($color) /opacity($color)`：获取颜色透明度值；
* `rgba($color, $alpha)`：改变颜色的透明度值；
* `opacify($color, $amount) / fade-in($color, $amount)`：使颜色更不透明.其取值范围主要是在 0~1 之间。当透明度值增加到大于 1 时，会以 1 计算，表示颜色不具有任何透明度；
* `transparentize($color, $amount) / fade-out($color, $amount)`：使颜色更加透明。






















