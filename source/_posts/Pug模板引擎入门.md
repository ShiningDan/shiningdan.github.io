---
title: Pug模板引擎入门
date: 2017-04-20 18:44:57
categories: coding
tags:
  - pug
  - Node.js
---

本系列的是参考慕课网 [scott老师](http://www.imooc.com/learn/259)的视频，仅供个人查阅使用，具体讲解推荐参考视频。

同时也参考了[Jade —— 源于 Node.js 的 HTML 模板引擎](https://segmentfault.com/a/1190000000357534#articleHeader0)这篇文章的内容

**Jade 已经更名为 Pug**

安装 Pug：`npm install -g pug`

## 使用 Pug

如果要通过命令行使用 Pug，需要安装 pug-cli

```
npm install -g pug-cli
```

通过命令行使用 pug

```
使用: pug [options] [dir|file ...]

选项:

  -h, --help         输出帮助信息
  -v, --version      输出版本号
  -o, --out <dir>    输出编译后的 HTML 到 <dir>
  -O, --obj <str>    JavaScript 选项(为模板变量提供定义)
  -p, --path <path>  在处理 stdio 时，查找包含文件时的查找路径
  -P, --pretty       格式化 HTML 输出
  -c, --client       编译浏览器端可用的 runtime.js
  -D, --no-debug     关闭编译的调试选项(函数会更小)
  -w, --watch        监视文件改变自动刷新编译结果

Examples:

  # 编译整个目录
  $ pug templates

  # 生成 {foo,bar}.html
  $ pug {foo,bar}.jade

  # 在标准IO下使用jade 
  $ pug < my.jade > my.html
  
  # 在标准IO下使用jade, 同时指定用于查找包含的文件
  $ pug < my.jade -p my.jade > my.html
```

<!--more-->

## API

```
var pug = require('pug');
 
// compile 返回一个可以执行的函数，可以传递参数用来多次调用
var fn = pug.compile('string of pug', options);
var html = fn(locals);
 
// render 会直接返回 HTML 文件，而不是可执行函数
var html = pug.render('string of pug', merge(options, locals)); 
 
// renderFile 
var html = pug.renderFile('filename.pug', merge(options, locals));
```

![](http://ojt6zsxg2.bkt.clouddn.com/f9bd7429859a265237ce4a10d5f38040.png)

For full API, see [jade-lang.com/api](https://pugjs.org/api/getting-started.html)

## Pug 基础语法

### Pug 文档声明和头尾标签

使用 Pug 创建文档声明，可以使用 `!!!`(不推荐)，也可以使用 `doctype`：

或

```
!!! html
// 或者
doctype html
```

当然也是可以直接传递一段文档类型的文本：

```
doctype html PUBLIC "-//W3C//DTD XHTML Basic 1.1//EN"
```

### 标签

标签就是一个简单的单词:

```
html
```

它会被转换为 `<html></html>`

标签也是可以有 id 的:

```
div#container
```

它会被转换为 `<div id="container"></div>`

怎么加 class 呢？

```
div.user-details
```

转换为 `<div class="user-details"></div>`

```
div#foo.bar.baz
```

转换为 `<div id="foo" class="bar baz"></div>`

**如果有时候想在 .jade 文件中自行添加完整的 HTML 标签，Pug 也是可以顺利解析的。**

```
section
  <a> this is a self close a</a>
// 解析结果为 

<section>
    <a> this is a self close a</a>
</section>
```

### 标签文本

只需要简单的把内容放在标签之后：

```
p wahoo!
```

它会被渲染为 `<p>wahoo!</p>`

但是大段的文本怎么办呢：

```
p
  | foo bar baz
  | rawr rawr
  | super cool
  | go jade go
// or
p.
  foo bar baz
  rawr rawr
  super cool
  go jade go
```
  
渲染为 `<p>foo bar baz rawr.....</p>`

怎么和数据结合起来？ 所有类型的文本展示都可以和数据结合起来，如果我们把 { name: 'tj', email: 'tj@vision-media.ca' } 传给编译函数，下面是模板上的写法:

```
#user #{name} &lt;#{email}&gt;
```

它会被渲染为 `<div id="user">tj &lt;tj@vision-media.ca&gt;</div>`

要输出 `#{}` 的时候怎么办? 转义一下!

```
p \#{something}
```

同样可以使用非转义的变量 `!{html}`, 下面的模板将直接输出一个 `<script>` 标签:

```
- var html = "<script></script>"
| !{html}
```

### 属性

Pug 现在支持使用 ( 和 ) 作为属性分隔符

```
a(href='/login', title='View login page') Login
// 编译结果
<a href="/login" title="View login page">Login</a>
```

当一个值是 undefined 或者 null 属性 不 会被加上,
所以呢，它不会编译出 'something="null"'.

```
div(something=null)
```

Boolean 属性也是支持的:

```
input(type="checkbox", checked)
```

假如我有一个 user 对象 `{ id: 12, name: 'tobi' }`

我们希望创建一个指向 `/user/12` 的链接 `href`, 我们可以使用普通的 JavaScript 字符串连接，如下:

```
a(href='/user/' + user.id)= user.name
// or
a(href='/user/#{user.id}')= user.name
```

### 注释

单行注释和 JavaScript 里是一样的，通过 `//` 来开始，并且必须单独一行：

```
// just some paragraphs
p foo
p bar
```

Pug 同样支持不输出的注释，加一个短横线就行了：

```
//- will not output within markup
p foo
p bar
```

渲染为：

```
<p>foo</p>
<p>bar</p>
```

#### 块注释

块注释也是支持的：

```
body
  //
    #content
      h1 Example
```
      
渲染为：

```
<body>
  <!--
  <div id="content">
    <h1>Example</h1>
  </div>
  -->
</body>
```

Pug 同样很好的支持了条件注释：

```
body
  //if IE
    a(href='http://www.mozilla.com/en-US/firefox/') Get Firefox
```
    
渲染为：

```
<body>
  <!--[if IE]>
    <a href="http://www.mozilla.com/en-US/firefox/">Get Firefox</a>
  <![endif]-->
</body>
```

### 代码

#### 变量声明和传递

可以在 `.jade` 文件中使用 `- var course = jade` 来定义变量，也可以在编译 `.jade` 文件的时候再指定变量。编译的时候指定变量的方法为：

```
jade index.jade --obj {"course": "jade"}
```

也可以先将变量写在一个 Json 文件里：

```
// course.json
{
    "course": "jade"
}
jade index.jade -O course.json
```

#### `-`

Jade 目前支持三种类型的可执行代码。第一种是前缀 `-`， 这是不会被输出的：

```
- var foo = 'bar';
```

这可以用在条件语句或者循环中：

```
- for (var key in obj)
  p= obj[key]
```

由于 Jade 的缓存技术，下面的代码也是可以的：

```
- if (foo)
  ul
    li yay
    li foo
    li worked
- else
  p oh no! didnt work
```

甚至是很长的循环也是可以的：

```
- if (items.length)
  ul
    - items.forEach(function(item){
      li= item
    - })
```

#### `=`

下一步我们要 **转义** 输出的代码，比如我们返回一个值，只要前缀一个 `=`：

```
- var foo = 'bar'
= foo
h1= foo
```

它会渲染为 `bar<h1>bar</h1>`. 为了安全起见，使用 `=` 输出的代码默认是转义的,如果想直接输出不转义的值可以使用 `!=`：

```
p!= aVarContainingMoreHTML
```

Jade 同样是设计师友好的，它可以使 JavaScript 更直接更富表现力。比如下面的赋值语句是相等的，同时表达式还是通常的 JavaScript：

```
- var foo = 'foo ' + 'bar'
foo = 'foo ' + 'bar'
```

Jade 会把 if, else if, else, until, while, unless 同别的优先对待, 但是你得记住它们还是普通的 JavaScript：

```
- var foo = 'bar'
if foo == 'bar'
  ul
    li yay
    li foo
    li= foo
else
  p oh no! didnt work 
```

输出是：

```
<ul>
  <li>yay</li>
  <li>foo</li>
  <li>bar</li>
</ul>
```

### 循环

**遍历数组或者对象的时候，使用 `for` 或者 `each`，不可以在之前加上 `-`。因为 `for` 和 `each` 是 pug 中的关键字，不是代表着 JavaScript 的代码**

一个遍历数组的例子 ：

```
- var items = ["one", "two", "three"]
each item in items
  li= item
```

遍历一个数组同时带上索引：

```
items = ["one", "two", "three"]
each item, i in items
  li #{item}: #{i}
```

遍历一个数组的键值：

```
obj = { foo: 'bar' }
each val, key in obj
  li #{key}: #{val}
```

如果你喜欢，也可以使用 for ：

```
for user in users
  for role in user.roles
    li= role
```

还可以使用 `while`

```
- var n = 0
    ul
      while n < 4
        li= n++
```

### 条件语句

```
for user in users
  if user.role == 'admin'
    p #{user.name} is an admin
  else
    p= user.name
```

Jade 同时支持 unless, 这和 if (!(expr)) 是等价的：

```
for user in users
  unless user.isAnonymous
    p
      | Click to view
      a(href='/users/' + user.id)= user.name 
```

### Mixins

**一般是定义一个列表里面重复的区块**

Mixins 在编译的模板里会被 Jade 转换为普通的 JavaScript 函数。 Mixins 可以带参数

```
mixin list
  ul
    li foo
    li bar
    li baz
```

Mixins 也可以带一个或者多个参数，参数就是普通的 JavaScript 表达式，比如下面的例子：

```
mixin pets(pets)
  ul.pets
    - each pet in pets
      li= pet
```

或者传入不定个数的参数

```
mixin pets(pets, ...item)
```

使用 `+` 调用 mixin

```
+pets(pets)
```

会输出像下面的 HTML:

```
<ul class="pets">
  <li>tobi</li>
  <li>loki</li>
  <li>jane</li>
  <li>manny</li>
</ul>
```

mixin 中也可以嵌套调用 mixin，使用 `+` 就可以调用。

## Pug 进级

### 模板继承

Jade 支持通过 `block` 和 `extends` 关键字来实现模板继承。 一个块就是一个 Jade 的 block ，它将在子模板中实现，同时是支持递归的。

Jade 块如果没有内容，Jade 会添加默认内容，下面的代码默认会输出 `block scripts`, `block content`, 和 `block foot`.

```
html
  head
    h1 My Site - #{title}
    block scripts
      script(src='/jquery.js')
  body
    block content
    block foot
      #footer
        p some footer content
```

```
extends extend-layout

block scripts
  script(src='/jquery.js')
  script(src='/pets.js')

block content
  h1= title
  each pet in pets
    include pet
```

**注意，子模板的 block 会覆盖父模板中同名的 block**

### 前置、追加代码块

**使用 `append` 来为 block 添加额外的部分**

Jade允许你 替换 （默认）、 前置 和 追加 blocks. 比如，假设你希望在 所有 页面的头部都加上默认的脚本，你可以这么做：

```
html
  head
    block head
      script(src='/vendor/jquery.js')
      script(src='/vendor/caustic.js')
  body
    block content
```

现在假设你有一个Javascript游戏的页面，你希望在默认的脚本之外添加一些游戏相关的脚本，你可以直接append上代码块：

```
extends layout

block append head
  script(src='/vendor/three.js')
  script(src='/game.js')
```

### 包含

**一般使用继承 `extend` 的时候，还需要为其中的 `block` 定义。但是使用 `include` 的时候，就人为 `include` 进来的文件是一个完整的文件，不需要为其中的 `block` 定义**

`include` 允许你静态包含一段 Jade, 或者别的存放在单个文件中的东西比如 CSS, HTML 非常常见的例子是包含头部和页脚。 假设我们有一个下面目录结构的文件夹：

```
./layout.jade
./includes/
  ./head.jade
  ./tail.jade
```

下面是 `layout.jade` 的内容:

```
html
  include includes/head  
  body
    h1 My Site
    p Welcome to my super amazing site.
    include includes/foot
```

这两个包含 `includes/head` 和 `includes/foot` 都会读取相对于给 `layout.jade` 参数 `filename` 的路径的文件, 这是一个绝对路径

前面已经提到，include 可以包含比如 HTML 或者 CSS 这样的内容。给定一个扩展名后，Jade 不会把这个文件当作一个 Jade 源代码，并且会把它当作一个普通文本包含进来：

```
include head.html
include stylesheet.css
include script.js
```

### 过滤器

**在过滤器里面不能传递动态的数据！**

可以通过插件添加生成 HTML 的功能，常用的有 `:babel, :uglify-js, :scss, :markdown-it`

### 反编译

可以使用 `html2jade` 模块将 HTML 文件转换为 Jade 文件























































