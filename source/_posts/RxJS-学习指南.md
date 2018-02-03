---
title: RxJS 学习指南
date: 2018-02-01 16:25:55
categories: coding
tags:
  - JavaScript
  - 函数式编程
  - RxJS
---

本文是 RxJS 学习的笔记

<!--more-->

## RxJS 简介

在前端，我们通常有这么一些方式来处理异步的东西：

* 回调
* 事件
* Promise
* Generator

其中，存在两种处理问题的方式，因为需求也是两种：

* 分发
* 流程

在处理分发的需求的时候，回调、事件或者类似订阅发布这种模式是比较合适的；而在处理流程性质的需求时，Promise和Generator比较合适。

RxJS的优势在于结合了两种模式，它的每个Observable上都能够订阅，而Observable之间的关系，则能够体现流程

例如

```
getDataO().subscribe(data => {
  // render
})
```

这么一句好像就搞定了我们要求的所有事情。我们可以这么去理解这件事：

* getDataO是一个业务过程
* 业务过程的结果数据可以被订阅

这样，我们就可以把获取和订阅这两件事合并到一起，视图层的关注点就简单很多了。


RxJS还提供了BehaviourSubject和ReplaySubject这样的东西，用于记录数据流上一些比较重要的信息，让那些“我们来晚了”的订阅者们回放之前错过的一切。

RxJS 比较适合：整体性、实时性的要求高的项目。

什么是整体性？这是一种系统设计的理念，系统中的很多业务模块不是孤立的，比如说，从展示上，GUI与命令行的差异在于什么？在于数据的冗余展示。我们可以把同一份业务数据以不同形态展示在不同视图上，甚至在PC端，由于屏幕大，可以允许同一份数据以不同形态同时展现，这时候，为了整体协调，对此数据的更新就会要产生很多分发和联动关系。

什么是实时性？这个其实有多个含义，一个比较重要的因素是服务端是否会主动向推送一些业务更新信息，如果用得比较多，也会产生不少的分发关系。

在分发和联动关系多的时候，RxJS才能更加体现出它比Generator、Promise的优势。

视图层如 ng/vue/react，做的是『视图状态』到『渲染结果』的声明式映射，而 Rx 做的则是『数据源』到『视图状态』（或者其他副作用）的声明式映射。两边接起来就可以完全声明式地去描述从『数据源』到『渲染结果』的映射关系

通常，对RxJS的解释会是这么一些东西，我们来分别看看它们的含义是什么。

* Reactive
* Lodash for events
* Observable
* Stream-based

### Reactive

什么是Reactive呢，一个比较直观的对比是这样的：

比如说，abc三个变量之间存在加法关系：

```
a = b + c
```

在传统方式下，这是一种一次性的赋值过程，调用一次就结束了，后面b和c再改变，a也不会变了。

而在Reactive的理念中，我们定义的不是一次性赋值过程，而是可重复的赋值过程，或者说是变量之间的关系：

```
a: = b + c
```

定义出这种关系之后，每次b或者c产生改变，这个表达式都会被重新计算。

### Stream-based

```
data1      data2      data3
  |          |          |
  ------------          |
        |               |
        -----------------
                |
              state
```

一个视图所需要的数据可能是这样的：

* data1跟data2通过某种组合，得到一个结果
* 这个结果再去跟data3组合，得到最终结果

RxJS给我们提供了一堆操作符用于处理这些Observable之间的关系，比如说，我们可以这样：

```
const A$ = Observable.interval(1000)
const B$ = Observable.of(3)
const C$ = Observable.from([5, 6, 7])

const D$ = C$.toArray()
  .map(arr => arr.reduce((a, b) => a + b), 0)
const E$ = Observable.combineLatest(A$, B$, D$)
   .map(arr => arr.reduce((a, b) => a + b), 0)
```

上述的D就是通过C进行一次转换所得到的数据管道，而E是把A，B，D进行拼装之后得到的数据管道，

```
A ------> |
B ------> | -> E
C -> D -> |
```


## 视图如何使用数据流

现在主流的MV*框架都基于一个共同的理念：MDV（模型驱动视图），在这个理念下，一切对于视图的变更，首先都应当是模型的变更，然后通过模型和视图的映射关系，自动同步过去。

在这些体系中，如果要使用RxJS的Observable，都非常简单：

```
data$.subscribe(data => {
  // 这里根据所使用的视图库，用不同的方式响应数据
  // 如果是 React 或者 Vue，手动把这个往 state 或者 data 设置
  // 如果是 Angular 2，可以不用这步，直接把 Observable 用 async pipe 绑定到视图
  // 如果是 CycleJS ……
})
```

## RxJS 详解

关于 RxJS 的详解以及 API 的使用，可以参考：[RxJS | 入门](http://cn.rx.js.org/manual/overview.html)

## 参考

* [RxJS 入门指引和初步应用](https://zhuanlan.zhihu.com/p/25383159)



