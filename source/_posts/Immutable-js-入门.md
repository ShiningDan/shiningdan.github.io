---
title: Immutable.js 入门
date: 2018-01-30 17:49:28
categories: coding
tags:
  - JavaScript
  - Immutable.js
---

本文是学习 Immutable.js 的笔记

<!--more-->

Immutable Data是函数式编程中的一个概念，在前端组件化框架中能起到一些很独特的作用。

它的大致理念是，任何一种赋值，都应当被转化成复制，不存在指向同一个地方的引用。

JavaScript 中的对象一般是可变的（Mutable），因为使用了引用赋值，新的对象简单的引用了原始对象，改变新的对象将影响到原始对象。虽然这样做可以节约内存，但当应用复杂后，这就造成了非常大的隐患，Mutable 带来的优点变得得不偿失。为了解决这个问题，一般的做法是使用 shallowCopy（浅拷贝）或 deepCopy（深拷贝）来避免被修改，但这样做造成了 CPU 和内存的浪费。

Immutable 可以很好地解决这些问题。

immutable赋值应当仅存在于组件边界上，在组件内部不是特别有必要使用。例如接收 Props 的时候和 shouldComponentUodate 的时候

## 优点

### 节省内存 

Immutable.js 使用了 Structure Sharing 会尽量复用内存。没有被引用的对象会被垃圾回收。

### 并发安全

传统的并发非常难做，因为要处理各种数据不一致问题，因此『聪明人』发明了各种锁来解决。但使用了 Immutable 之后，数据天生是不可变的，并发锁就不需要了。

### 拥抱函数式编程

Immutable 本身就是函数式编程中的概念，纯函数式编程比面向对象更适用于前端开发。因为只要输入一致，输出必然一致，这样开发的组件更易于调试和组装。

## 缺点

### 容易与原生对象混淆

虽然 Immutable.js 尽量尝试把 API 设计的原生对象类似，有的时候还是很难区别到底是 Immutable 对象还是原生对象，容易混淆操作。

当使用外部库的时候，一般需要使用原生对象，也很容易忘记转换。

下面给出一些办法来避免类似问题发生：

* 使用 Flow 或 TypeScript 这类有静态类型检查的工具
* 约定变量命名规则：如所有 Immutable 类型对象以 `$$` 开头。
* 使用 `Immutable.fromJS` 而不是 `Immutable.Map` 或 `Immutable.List` 来创建对象，这样可以避免 Immutable 和原生对象间的混用


## 与 Object.freeze、const 区别

`Object.freeze` 和 ES6 中新加入的 `const` 都可以达到防止对象被篡改的功能，但它们是 shallowCopy 的。对象层级一深就要特殊处理了。

## 使用 

### 数据类型

* List：有序索引集，类似于 JavaScript 中的 Array。\
* Map：类似于 JavaScript 中的 Object。
* OrderedMap：有序 Map，排序依据是数据的 set() 操作。
* Set：和 ES6 中的 Set 类似，都是没有重复值的集合。
* OrderedSet：Set 的变体，可以保证遍历的顺序性。排序依据是数据的 add 操作。
* Stack：有序集合，且使用 unshift(v) 和 shift() 进行添加和删除操作的复杂度为 O(1)
* Range()：返回一个 Seq.Indexed 类型的数据集合，该方法接收三个参数 (start = 1, end = infinity, step = 1)，分别表示起始点、终止点和步长，如果 start 等于 end，则返回空的数据结合。
* Repeat()：返回一个 Seq.indexed 类型的数据结合，该方法接收两个参数 (value，times)，value 表示重复生成的值，times 表示重复生成的次数，如果没有指定 times，则表示生成的 Seq 包含无限个 value。
* Record：在表现上类似于 ES6 中的 Class，但在某些细节上还有所不同。
* Seq：序列（may not be backed by a concrete data structure）
* Iterable：可以被迭代的 (Key, Value) 键值对集合，是 Immutable.js 中其他所有集合的基类，为其他所有集合提供了 基础的 Iterable 操作函数（比如 map() 和 filter）。
* Collection：创建 Immutable 数据结构的最基础的抽象类，不能直接构造该类型。
* Iterable：可以被迭代的 (Key, Value) 键值对集合，是 Immutable.js 中其他所有集合的基类，为其他所有集合提供了 基础的 Iterable 操作函数（比如 map() 和 filter）。

好吧，上面那么多行常用的也就是 List和Map，顶多再加个Seq。

### 常见 API

immutable数据应该被当作**值而不是对象，值是表示该事件在特定时刻的状态**。这个原则对理解不可变数据的适当使用是最重要的。为了将Immutable.js数据视为值，就必须使用Immutable.is()函数或.equals()方法来确定值相等，而不是确定对象引用标识的 === 操作符。

见 [Immutable 官网](https://facebook.github.io/immutable-js/)

或者 [immutable.js 在React、Redux中的实践以及常用API简介](https://yq.aliyun.com/articles/69516) 介绍

## Immutable 源码

在 Immutatable 中，数据的复制是经常发生的操作，为了加速 Immuatable.js 的草最速度它在数据操作的时候进行了一些优化，主要是分为以下两点：

1. 在进行 Map 和 Vector 复制的时候，对于可以重用的数据，尽量选择重用而不是创建新的数据
2. 对于 Map 中数据的查询，不是进行顺序遍历，而是使用了 5 bits 的 Hash Map Trie 来进行数据查询

更多的介绍，可以在 [精读 Immutable 结构共享](https://zhuanlan.zhihu.com/p/27133830) 中查看

## 参考

* [2015前端组件化框架之路](https://github.com/xufei/blog/issues/19)
* [Immutable 详解及 React 中实践](https://zhuanlan.zhihu.com/p/20295971)
* [immutable.js 在React、Redux中的实践以及常用API简介](https://yq.aliyun.com/articles/69516)
* [精读 Immutable 结构共享](https://zhuanlan.zhihu.com/p/27133830)


