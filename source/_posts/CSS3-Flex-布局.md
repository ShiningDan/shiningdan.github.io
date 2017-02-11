---
title: CSS3 Flex 布局
date: 2017-02-11 14:24:38
categories: coding
tags:
  - HTML
  - CSS
  - Flex
---

本笔记是根据[CSS 3 Flex 全解【视频】](https://zhuanlan.zhihu.com/p/25173221)介绍总结，如有需要，可以去原视频查看。

## Flex 之前

布局主要使用：

* noraml flow(正常文档流)
* float + clear
* position relative + absolute
* display inline + block
* 负 margin

## Flex 布局的特点

1. 块状布局侧重垂直方向，行内布局侧重水平方向，flex 布局与方向无关
2. flex 布局可以实现**空间自动分配、自动对齐**
3. flex 布局适用于简单的线性布局，复杂的布局交给 Grid 布局

## Flex 基本概念

![](http://ojt6zsxg2.bkt.clouddn.com/d7714d26c651961da0e4d506020c2650.png)

Flex 布局的时候，有两个轴，一个是主轴，一个是侧轴，注意主轴不一定是横轴，它的方向也可以改变成纵向。主轴的方向是 item 的排列方向。

使用 Flex 布局的时候，给父容器添加 `display: flex;` 就可以。

Flex 布局的父容器叫 **flex container**，子容器叫 **flex item**。

### flex container

flex container 有六个主要的属性：

1. flex-direction：方向
    * row（行）
    * row-reverse
    * column（列）
    * column-reverse
2. flex-wrap：换行，默认是胡换行，所有的 flex-item 都挤在一行，
    * nowrap(不换行)
    * wrap
    * wrap-reverse(换行，换行以后上下(row)/左右(column)方向交换)
3. flex-flow：上面两个的简写
4. justify-content：**主轴**方向对齐方式
    * auto
    * baseline
    * center(所有 item 都在中间)
    * end
    * flex-end(向终点靠拢)
    * flex-start(向�起点靠拢)
    * last-baseline
    * space-around(空间分布在所有 item 周围)
    * sapce-between(空间分布在所有 item 之间)
5. align-item：**侧轴**对齐方式
    * stretch(默认值，将所有的元素都伸展开到最高的元素一样的高度，但是元素的 height 不能设置。**只有这个值会延伸 item 的高度**)
    * flex-start/flex-end：所有的元素向侧轴开始/终点的地方对齐，元素的高度不会改变
    * center：都放在侧轴中点
    * baseline
6. align-content：多行/多列内容对齐方式（使用较少）

### flex item

flex item 主要有六个属性：

1. flex-grow：增长比例（空间过多时）
    * 参数是数值。如果有多余的空间，每个 flex item 的 flwx-grow(1, 2, 3...) 值代表将多余的空间按照份数分给不同的 item。条件是 item 没有设置 width(或者 height)
2. flex-shrink：收缩比例（空间不够时）
    * 参数是数值。当空间被填满时，如果加入这个参数，则会根据数值按比例收缩。
3. flex-basis：默认大小（一般不用）
4. flex：上面三个的缩写
5. order：顺序（代替双飞翼）
    * 参数是数值，并且如果要使 order 生效，需要对所有的 item 都设置 order 值。参数越小，则该 item 放置越前面。
6. align-self：自身的对齐方式
    * 参数和在 flex container 上设置的侧轴对齐方式(align-item)一样，表示该 item 和侧轴对齐方式和其他的 item 对齐方式不一样。



















