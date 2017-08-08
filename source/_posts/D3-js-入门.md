---
title: D3.js 入门
date: 2017-07-16 14:29:44
categories: coding
tags:
  - JavaScript
  - D3.js
---

本文是我在学习 D3.js 的总结

<!--more-->

## 元素操作

D3.js 使用**链式语法**，和 JQuery 的语法很像，如：

### 选择元素

```
d3.select('body').selectAll('p').text('Hello D3 world!')
```

在 D3 中，用于选择元素的函数有两个：

* `d3.select()`：是选择所有指定元素的第一个
* `d3.selectAll()`：是选择指定元素的全部

### 绑定数据

D3 有一个很独特的功能：能将数据绑定到 DOM 上，绑定之后，当需要依靠这个数据才操作元素的时候，会很方便。

D3 中是通过以下两个函数来绑定数据的：

* `datum()`：绑定一个数据到选择集上
* `data()`：绑定一个数组到选择集上，数组的各项值分别与选择集的各元素绑定

设现在有三个段落元素如下。

```
<p>Apple</p>
<p>Pear</p>
<p>Banana</p>
```

#### datum()

假设有一字符串 China，要将此字符串分别与三个段落元素绑定，代码如下：

```
var str = "China";

var body = d3.select("body");
var p = body.selectAll("p");

p.datum(str);

p.text(function(d, i){
    return "第 "+ i + " 个元素绑定的数据是 " + d;
});
```

绑定数据后，使用此数据来修改三个段落元素的内容，其结果如下：

```
第 0 个元素绑定的数据是 China

第 1 个元素绑定的数据是 China

第 2 个元素绑定的数据是 China
```

在上面的代码中，用到了一个无名函数 `function(d, i)`。当选择集需要使用被绑定的数据时，常需要这么使用。其包含两个参数，其中：

* `d` 代表数据，也就是与某元素绑定的数据。
* `i` 代表索引，代表数据的索引号，从 0 开始。

#### data()

有一个数组，接下来要分别将数组的各元素绑定到三个段落元素上。

```
var dataset = ["I like dog","I like cat","I like snake"];
```

调用 data() 绑定数据，并替换三个段落元素的字符串为被绑定的字符串，代码如下：

```
var body = d3.select("body");
var p = body.selectAll("p");

p.data(dataset)
  .text(function(d, i){
      return d;
  });
```

这段代码也用到了一个无名函数 function(d, i)，其对应的情况如下：

* 当 i == 0 时， d 为 I like dog。
* 当 i == 1 时， d 为 I like cat。
* 当 i == 2 时， d 为 I like snake。

```
I like dog

I like cat

I like snake
```

### 选择、插入、删除元素

`select` 还可以通过 id，class 来选择元素，和 jQuery 差不多

```
var p2 = body.select("#myid");
var p = body.selectAll(".myclass");
```

插入元素涉及的函数有两个：

* `append()`：在选择集末尾插入元素
* `insert()`：在选择集前面插入元素

```
body.append("p").text("append p element");
body.insert("p","#myid").text("insert p element");
```

删除一个元素时，对于选择的元素，使用 remove 即可，例如：

```
var p = body.select("#myid");
p.remove();
```

## SVG 介绍

SVG是XML语言的一种形式，有点类似XHTML，它可以用来绘制矢量图形，例如右面展示的图形。SVG可以通过定义必要的线和形状来创建一个图形，也可以修改已有的位图，或者将这两种方式结合起来创建图形。图形和其组成部分可以变形，可以合成，还可以通过滤镜完全改变外观。

正式开始之前，你需要基本掌握XML和其它标记语言比如说HTML，如果你不是很熟悉XML，这里有几个重点一定要记住：

* SVG的元素和属性必须按标准格式书写，因为XML是区分大小写的（这一点和html不同）
* SVG里的属性值必须用引号引起来，就算是数值也必须这样做。

让我们直接从一个简单的例子开始，看一下下面代码：

```
<svg version="1.1"
     baseProfile="full"
     width="300" height="200"
     xmlns="http://www.w3.org/2000/svg">

  <rect width="100%" height="100%" fill="red" />

  <circle cx="150" cy="100" r="80" fill="green" />

  <text x="150" y="125" font-size="60" text-anchor="middle" fill="white">SVG</text>

</svg>
```

从svg根元素开始：

* 应舍弃来自 (X)HTML的doctype声明，因为基于SVG的DTD验证导致的问题比它能解决的问题更多。
* 属性version和属性baseProfile属性是必不可少的，供其它类型的验证方式确定SVG版本。
* 作为XML的一种方言，SVG必须正确的绑定命名空间 （在xmlns属性中绑定）。

**最值得注意的一点是元素的渲染顺序。SVG文件全局有效的规则是“后来居上”，越后面的元素越可见。**

web上的svg文件可以直接在浏览器上展示，或者通过以下几种方法嵌入到HTML文件中：

* 如果HTML是XHTML并且声明类型为application/xhtml+xml，可以直接把SVG嵌入到XML源码中。
* 如果HTML是HTML5并且浏览器支持HTML5，同样可以直接嵌入SVG。然而为了符合HTML5标准，可能需要做一些语法调整。
* 可以通过 object 元素引用SVG文件：

```
<object data="image.svg" type="image/svg+xml" />
```

类似的也可以使用 iframe 元素引用SVG文件：

```
<iframe src="image.svg"></iframe>
```

理论上同样可以使用 img 元素，但是在低于4.0版本的Firefox 中不起作用。

最后SVG可以通过JavaScript动态创建并注入到HTML DOM中。 这样具有一个优点，可以对浏览器使用替代技术，在不能解析SVG的情况下，可以替换创建的内容。

#### 坐标定位

对于所有元素，SVG使用的坐标系统或者说网格系统，和Canvas用的差不多（所有计算机绘图都差不多）。这种坐标系统是：**以页面的左上角为(0,0)坐标点，坐标以像素为单位，x轴正方向是向右，y轴正方向是向下。**


#### 基本形状

```
<?xml version="1.0" standalone="no"?>
<svg width="200" height="250" version="1.1" xmlns="http://www.w3.org/2000/svg">

  <rect x="10" y="10" width="30" height="30" stroke="black" fill="transparent" stroke-width="5"/>
  <rect x="60" y="10" rx="10" ry="10" width="30" height="30" stroke="black" fill="transparent" stroke-width="5"/>

  <circle cx="25" cy="75" r="20" stroke="red" fill="transparent" stroke-width="5"/>
  <ellipse cx="75" cy="75" rx="20" ry="5" stroke="red" fill="transparent" stroke-width="5"/>

  <line x1="10" x2="50" y1="110" y2="150" stroke="orange" fill="transparent" stroke-width="5"/>
  <polyline points="60 110 65 120 70 115 75 130 80 125 85 140 90 135 95 150 100 145"
      stroke="orange" fill="transparent" stroke-width="5"/>

  <polygon points="50 160 55 180 70 180 60 190 65 205 50 195 35 205 40 190 30 180 45 180"
      stroke="green" fill="transparent" stroke-width="5"/>

  <path d="M20,230 Q40,205 50,230 T90,230" fill="none" stroke="blue" stroke-width="5"/>
</svg>
```
显示效果如下：

![](http://ojt6zsxg2.bkt.clouddn.com/5412e3fe01558aee6924ed08aa7f27f8.png)

#### 路径

`<path>`元素是SVG基本形状中最强大的一个，它不仅能创建其他基本形状，还能创建更多其他形状。

每一个命令都用一个关键字母来表示，每一个命令都有两种表示方式，一种是用大写字母，表示采用绝对定位。另一种是用小写字母，表示采用相对定位，比如移动到(10,10)这个点的命令，应该写成“M 10 10”

路径分为：

* 直线命令：M(Move to)、L(Line to)、H(绘制平行线)、V(绘制垂直线)、Z(闭合路径命令)
* 曲线命令：绘制平滑曲线的命令有三个，其中两个用来绘制贝塞尔曲线，另外一个用来绘制弧形或者说是圆的一部分。
    * C(三次贝塞尔曲线)
    * S(简写的三次贝塞尔曲线命令)
    * Q(二次贝塞尔曲线)
    * T(简写的二次贝塞尔曲线命令)
    * A(弧形命令)

#### 填充和边框

fill属性设置对象内部的颜色，stroke属性设置绘制对象的线条的颜色。

除了颜色属性，还有其他一些属性用来控制绘制描边的方式

#### 渐变

可以创建和并在填充和描边上应用渐变色。

有两种类型的渐变：**线性渐变和径向渐变**。你必须给渐变内容指定一个id属性，否则文档内的其他元素就不能引用它。为了让渐变能被重复使用，渐变内容需要定义在`<defs>`标签内部，而不是定义在形状上面。

#### 图案

在pattern元素内部你可以包含任何之前包含过的其它基本形状，并且每个形状都可以使用之前学习过的任何样式样式化，包括渐变和半透明。

#### 文字

在SVG中有两种截然不同的文本模式. 一种是写在图像中的文本，另一种是SVG字体。

在一个SVG文档中，`<text>`元素内部可以放任何的文字。

#### 在一个SVG文档中，<text>元素内部可以放任何的文字。

`<g>`元素，可以把属性赋给一整个元素集合。实际上，这是它唯一的目的。

```
<g fill="red">
  <rect x="0" y="0" width="10" height="10" />
  <rect x="20" y="0" width="10" height="10" />
</g>
```

输出两个红色矩形。

基本的变换有：平移、旋转、斜切、缩放、用matrix()实现复杂变形、SVG嵌在SVG内部创建一个新的坐标系统。

#### 剪切和遮罩

Clipping用来移除在别处定义的元素的部分内容。在这里，任何半透明效果都是不行的。它只能要么显示要么不显示。

Masking允许使用透明度和灰度值遮罩计算得的软边缘。

用opacity定义透明度

#### 嵌入光栅图像

像在HTML中的img元素，SVG有一个image元素，用于同样的目的。你可以利用它嵌入任意光栅（以及矢量）图像。它的规格要求应用至少支持PNG、JPG和SVG格式文件。

#### 嵌入任意XML

因为SVG是一个XML应用，所以你总是可以在SVG文档的任何位置嵌入任意XML。

#### SVG字体

定义一个SVG字体的基础是`<font>`元素。

如果你已经把你的字体声明放在一起，你可以使用一个单一的font-family属性以实现在SVG文本上应用字体

```
<text font-family="Super Sans">My text uses Super Sans</text>
```

## 使用 SVG 制作图表

讲解如何使用 D3 在 SVG 画布中绘图。

HTML 5 提供两种强有力的“画布”：SVG 和 Canvas。

SVG 有如下特点：

* SVG 绘制的是矢量图，因此对图像进行放大不会失真。
* 基于 XML，可以为每个元素添加 JavaScript 事件处理器。
* 每个图形均视为对象，更改对象的属性，图形也会改变。
* 不适合游戏应用。

Canvas 有如下特点：

* 绘制的是位图，图像放大后会失真。
* 不支持事件处理器。
* 能够以 .png 或 .jpg 格式保存图像
* 适合游戏应用

D3 虽然没有明文规定一定要在 SVG 中绘图，但是 D3 提供了众多的 SVG 图形的生成器，它们都是只支持 SVG 的。因此，建议使用 SVG 画布。

在 SVG 中，矩形的元素标签是 rect。例如：

```
<svg>
<rect></rect>
<rect></rect>
</svg>
```

上面的 rect 里没有矩形的属性。矩形的属性，常用的有四个：

* x：矩形左上角的 x 坐标
* y：矩形左上角的 y 坐标
* width：矩形的宽度
* height：矩形的高度

现在给出一组数据，要对此进行可视化。数据如下：

```
var dataset = [ 250 , 210 , 170 , 130 , 90 ];  //数据（表示矩形的宽度）
```

```
var rectHeight = 25;   //每个矩形所占的像素高度(包括空白)

svg.selectAll("rect")
    .data(dataset)
    .enter()
    .append("rect")
    .attr("x",20)
    .attr("y",function(d,i){
         return i * rectHeight;
    })
    .attr("width",function(d){
         return d;
    })
    .attr("height",rectHeight-2)
    .attr("fill","steelblue");
```

这段代码添加了与 dataset 数组的长度相同数量的矩形，所使用的语句是：

```
svg.selectAll("rect")   //选择svg内所有的矩形
    .data(dataset)  //绑定数组
    .enter()        //指定选择集的enter部分
    .append("rect") //添加足够数量的矩形元素
```

### 比例尺的使用

将某一区域的值映射到另一区域，其大小关系不变。

这就是比例尺（Scale）。

将 dataset 中最小的值，映射成 0；将最大的值，映射成 300。

代码如下：

```
var min = d3.min(dataset);
var max = d3.max(dataset);

var linear = d3.scale.linear()
        .domain([min, max])
        .range([0, 300]);

linear(0.9);    //返回 0
linear(2.3);    //返回 175
linear(3.3);    //返回 300
```

D3 中的比例尺，也有定义域和值域，分别被称为 domain 和 range。开发者需要指定 domain 和 range 的范围，如此即可得到一个计算关系。

#### 序数比例尺

有时候，定义域和值域不一定是连续的。例如，有两个数组：

```
var index = [0, 1, 2, 3, 4];
var color = ["red", "blue", "green", "yellow", "black"];
```

我们希望 0 对应颜色 red，1 对应 blue，依次类推。

```
var ordinal = d3.scale.ordinal()
        .domain(index)
        .range(color);

ordinal(0); //返回 red
ordinal(2); //返回 green
ordinal(4); //返回 black
```

#### 给柱形图添加比例尺

在上一章的基础上，修改一下数组，再定义一个线性比例尺。

```
var dataset = [ 2.5 , 2.1 , 1.7 , 1.3 , 0.9 ];

var linear = d3.scale.linear()
        .domain([0, d3.max(dataset)])
        .range([0, 250]);
```

```
var rectHeight = 25;   //每个矩形所占的像素高度(包括空白)

svg.selectAll("rect")
    .data(dataset)
    .enter()
    .append("rect")
    .attr("x",20)
    .attr("y",function(d,i){
         return i * rectHeight;
    })
    .attr("width",function(d){
         return linear(d);   //在这里用比例尺
    })
    .attr("height",rectHeight-2)
    .attr("fill","steelblue");
```

### 坐标轴

在 SVG 画布的预定义元素里，有六种基本图形：

* 矩形
* 圆形
* 椭圆
* 线段
* 折线
* 多边形

另外，还有一种比较特殊，也是功能最强的元素：

* 路径

D3 提供了一个组件：`d3.svg.axis()`。

要生成坐标轴，需要用到比例尺，它们二者经常是一起使用的。下面，在上一章的数据和比例尺的基础上，添加一个坐标轴的组件。

```
//数据
var dataset = [ 2.5 , 2.1 , 1.7 , 1.3 , 0.9 ];
//定义比例尺
var linear = d3.scale.linear()
      .domain([0, d3.max(dataset)])
      .range([0, 250]);

var axis = d3.svg.axis()
     .scale(linear)      //指定比例尺
     .orient("bottom")   //指定刻度的方向
     .ticks(7);          //指定刻度的数量
```

默认的坐标轴样式不太美观，下面提供一个常见的样式：

```
<style>
.axis path,
.axis line{
    fill: none;
    stroke: black;
    shape-rendering: crispEdges;
}

.axis text {
    font-family: sans-serif;
    font-size: 11px;
}
</style>
```

坐标轴的位置，可以通过 transform 属性来设定。

通常在添加元素的时候就一并设定，写成如下形式：

```
svg.append("g")
  .attr("class","axis")
  .attr("transform","translate(20,130)")
  .call(axis);
```

定义了坐标轴之后，只需要在 SVG 中添加一个分组元素 ，再将坐标轴的其他元素添加到这个 里即可。代码如下：

```
svg.append("g")
   .call(axis);
```

等价于 

```
axis(svg.append(g));
```

总体效果如下：

![](http://ojt6zsxg2.bkt.clouddn.com/d317e80a2a56de2208a5238f08c1639a.png)

其中 `<svg>` 下面的标签为：

![](http://ojt6zsxg2.bkt.clouddn.com/7f2d514fd7c5f9f8f04ac4af2591451d.png)

## 让图表动起来

动态的图表，是指图表在某一时间段会发生某种变化，可能是形状、颜色、位置等，而且用户是可以看到变化的过程的。

D3 提供了 4 个方法用于实现图形的过渡：从状态 A 变为状态 B。

### transition()

启动过渡效果。

其前后是图形变化前后的状态（形状、位置、颜色等等），例如：

```
.attr("fill","red")         //初始颜色为红色
.transition()               //启动过渡
.attr("fill","steelblue")   //终止颜色为铁蓝色
```

### duration()

指定过渡的持续时间，单位为毫秒。

如 `duration(2000)` ，指持续 2000 毫秒，即 2 秒。

### ease()

指定过渡的方式，常用的有：

* linear：普通的线性变化
* circle：慢慢地到达变换的最终状态
* elastic：带有弹跳的到达最终状态
* bounce：在最终状态处弹跳几次

调用时，格式形如： `ease(“bounce”)`。

### delay()

指定延迟的时间，表示一定时间后才开始转变，单位同样为毫秒。此函数可以对整体指定延迟，也可以对个别指定延迟。

例如，对整体指定时：

```
.transition()
.duration(1000)
.delay(500)
```

又如，对一个一个的图形（图形上绑定了数据）进行指定时：

```
.transition()
.duration(1000)
.delay(funtion(d,i){
    return 200*i;
})
```

## 理解 Update、Enter、Exit

Update、Enter、Exit 是 D3 中三个非常重要的概念，它处理的是当选择集和数据的数量关系不确定的情况。

前几章里，反复出现了形如以下的代码。

```
svg.selectAll("rect")   //选择svg内所有的矩形
    .data(dataset)      //绑定数组
    .enter()            //指定选择集的enter部分
    .append("rect")     //添加足够数量的矩形元素
```

有数据，而没有足够图形元素的时候，使用此方法可以添加足够的元素。

假设，在 body 中有三个 p 元素，有一数组 [3, 6, 9]，则可以将数组中的每一项分别与一个 p 元素绑定在一起。但是，有一个问题：**当数组的长度与元素数量不一致（数组长度 > 元素数量 or 数组长度 < 元素数量）时**呢？这时候就需要理解 Update、Enter、Exit 的概念。

如果数组为 `[3, 6, 9, 12, 15]`，将此数组绑定到三个 p 元素的选择集上。可以想象，会有两个数据没有元素与之对应，这时候 D3 会建立两个空的元素与数据对应，这一部分就称为 **Enter**。而有元素与数据对应的部分称为 **Update**。如果数组为 [3]，则会有两个元素没有数据绑定，那么没有数据绑定的部分被称为 **Exit**。示意图如下所示。

![](http://ojt6zsxg2.bkt.clouddn.com/992dbae9105ea6dcabf0a410f9522a07.png)

将数组与元素数量为 0 的选择集绑定后，选择其 Enter 部分（请仔细看上图），然后添加（append）元素，也就是添加足够的元素，使得每一个数据都有元素与之对应。

### 当对应的元素不足时 

当对应的元素不足时 （ 绑定数据数量 > 对应元素 ），需要添加元素（append）。

```
var dataset = [ 3 , 6 , 9 , 12 , 15 ];

//选择body中的p元素
var p = d3.select("body").selectAll("p");

//获取update部分
var update = p.data(dataset);

//获取enter部分
var enter = update.enter();

//update部分的处理：更新属性值
update.text(function(d){
    return "update " + d;
});

//enter部分的处理：添加元素后赋予属性值
enter.append("p")
    .text(function(d){
        return "enter " + d;
    });
```

* update 部分的处理办法一般是：更新属性值
* enter 部分的处理办法一般是：添加元素后，赋予属性值

### 当对应的元素过多时

当对应的元素过多时 （ 绑定数据数量 < 对应元素 ），需要删掉多余的元素。

```
var dataset = [ 3 ];

//选择body中的p元素
var p = d3.select("body").selectAll("p");

//获取update部分
var update = p.data(dataset);

//获取exit部分
var exit = update.exit();

//update部分的处理：更新属性值
update.text(function(d){
    return "update " + d;
});

//exit部分的处理：修改p元素的属性
exit.text(function(d){
        return "exit";
    });

//exit部分的处理通常是删除元素
// exit.remove();
```

* exit 部分的处理办法一般是：删除元素（remove）

## 交互式操作

与图表的交互，指在图形元素上设置一个或多个监听器，当事件发生时，做出相应的反应。

## 布局

* 选择 D3：如果希望开发脑海中任意想象到的图表。
* 选择 Highcharts、Echarts 等：如果希望开发几种固定种类的、十分大众化的图表。

布局的作用是：将**不适合用于绘图的数据**转换成了**适合用于绘图的数据。**

因此，为了便于初学者理解，本站的教程叫布局的作用解释成：**数据转换**。

**布局不是直接绘图，而是为了得到绘图所需的数据。**

D3 总共提供了 12 个布局：饼状图（Pie）、力导向图（Force）、弦图（Chord）、树状图（Tree）、集群图（Cluster）、捆图（Bundle）、打包图（Pack）、直方图（Histogram）、分区图（Partition）、堆栈图（Stack）、矩阵树图（Treemap）、层级图（Hierarchy）。


## 参考

* [D3.js 入门教程](http://wiki.jikexueyuan.com/project/d3wiki/)
* [十二月咖啡馆工作室](http://www.decembercafe.org/)
* [SVG教程- SVG | MDN](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Tutorial)
* [D3.js | API 中文手册](https://github.com/d3/d3/wiki/API--%E4%B8%AD%E6%96%87%E6%89%8B%E5%86%8C)
* [D3.js | Gallery](https://github.com/d3/d3/wiki/Gallery)










