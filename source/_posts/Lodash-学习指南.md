---
title: Lodash 学习指南
date: 2018-01-29 18:25:41
categories: coding
tags:
  - Lodash
  - JavaScript
  - 函数式编程
---

本文是学习 Lodash 的学习记录

<!--more-->

Lodash是一个著名的javascript原生库，不需要引入其他第三方依赖。是一个意在提高开发者效率,提高JS原生方法性能的JS库。

Lodash 还在以下几个方面有提升：

1. 提供了很多函数式编程的函数，并且提供了链式调用的能力
2. 提供了能力更加全面的函数
3. 引入的惰性计算，提升了性能

所以，Lodash 本身来说其实是一个库，当然，我们在开发中，对于库能力的掌握，主要取决于使用的次数，单纯在学习总结中罗列库的 API 没有什么意义。

* [Lodash 官方文档](https://lodash.com/)
* [Lodash 中文文档](http://lodash.think2011.net/chunk)

## JS 中函数式编程的注意

函数式编程可以理解为，**以函数作为主要载体的编程方式**，用函数去拆解、抽象一般的表达式

也就是**允许函数作为参数，允许函数作为返回值，将函数作为代码复用的单元**

好处有：

* 语义更加清晰
* 可复用性更高
* 可维护性更好
* **作用域局限，副作用少**

一些常见的函数式编程的应用：

* 闭包
* 高阶函数
* 函数柯里化
* 组合 compose

## immutable 和 pure function

immutable 和 pure function 帮助更好的管理状态，使得应用的可预测性更强，降低代码管理难度。

immutable 和 pure function 帮助更好的管理状态，使得应用的可预测性更强，降低代码管理难度。

在前端基本满地都是共享状态，一个状态被多个组件依赖或者影响基本是不可避免的问题。那如何规避上述的不可预测的难题？我们来约法三章：谁都不可以修改状态，问题解决。怎么做到的？你要修改数据，你得新建数据。

## 函数式编程和面向对象编程的区别

这一点在月影老师的博客 [漫谈 JS 函数式编程（一）](https://www.h5jun.com/post/js-functional-1.html) 中很清晰地解释了：

面向对象对数据进行抽象，将行为以对象方法的方式封装到数据实体内部，从而降低系统的耦合度。而函数式编程，选择对过程进行抽象，将数据以输入输出流的方式封装进过程内部，从而也降低系统的耦合度。

一个庞大的系统，可能既要对数据进行抽象，又要对过程进行抽象，或者一个局部适合进行数据抽象，另一个局部适合进行过程抽象，这都是可能的。数据抽象不一定以对象实体为形式，同样过程抽象也不是说形式上必然是 functional 的，比如流式对象（InputStream、OutputStream）、Express 的 middleware，就带有明显的过程抽象的特征。但是在通常情况下，OOP更适合用来做数据抽象，FP更适合用来做过程抽象。

## 纯函数在测试上的应用

对于纯函数，因为是无状态的，测试的时候不需要构建运行时环境，也不需要用特定的顺序进行测试。

函数式编程能够减少系统中的非纯函数

```
//two impure functions

function setColor(el, color){
  el.style.color = color;
}

function setColors(els, color){
  els.forEach(el => setColor(el, color));
}

let items1 = document.querySelectorAll('ul > li:nth-child(2n + 1)');
let items2 = document.querySelectorAll('ul > li:nth-child(3n + 1)');

setColors(items2, 'green');
setColors(items1, 'red');
```

在测试的时候，我们需要构建环境来测试两个函数。

现在，我们用函数式编程思想来改造这个系统：

```
//only one impure function

function batch(fn){
  return function(target, ...args){
    if(target.length >= 0){
      return Array.from(target).map(item => fn.apply(this, [item, ...args]));
    }else{
      return fn.apply(this, [target, ...args]);
    }
  }
}

function setColor(el, color){
  el.style.color = color;
}

let setColors = batch(setColor);

let items1 = document.querySelectorAll('ul > li:nth-child(2n + 1)');
let items2 = document.querySelectorAll('ul > li:nth-child(3n + 1)');

setColors(items2, 'green');
setColors(items1, 'red');
```

batch(fn) 本身虽然看似复杂，但是有意思的事，这个函数无疑是纯函数，所以 batch(fn) 自身的测试是非常简单的：

```
test(t => {
  let add = (x, y) => x + y;
  let listAdd = batch(add);

  t.deepEqual(listAdd([1,2,3], 1), [2,3,4]);
});
```

设想，如果你定义了一个变量A，A在其他地方被其他人修改了，这样是不方便定位A的当前值的。关于定义多个变量引发的内存等问题，可以通过重用结构或部分引用的方式来减轻，可参考 immutable.js

## 容器、Functor

### 容器

```
var Container = function(x) {
  this.__value = x;
}
Container.of = x => new Container(x);

//试试看
Container.of(1);
//=> Container(1)

Container.of('abcd');
//=> Container('abcd')
```

我们调用 `Container.of` 把东西装进容器里之后，由于这一层外壳的阻挡，普通的函数就对他们不再起作用了

### Functor

需要加一个接口来让外部的函数也能作用到容器里面的值：

```
Container.prototype.map = function(f){
  return Container.of(f(this.__value))
}
```

我们可以这样使用它：

```
Container.of(3)
    .map(x => x + 1)                //=> Container(4)
    .map(x => 'Result is ' + x);    //=> Container('Result is 4')
```

Functor（函子）是实现了 map 并遵守一些特定规则的容器类型。

具体可以参考这篇文章 [JavaScript函数式编程（二）](https://zhuanlan.zhihu.com/p/21926955)

### Monad

Monad 让我们避开了嵌套地狱，可以轻松地进行深度嵌套的函数式编程，比如IO和其它异步任务。

Promise 就是一种 Monad

具体可以参考这篇文章 [JavaScript函数式编程（三）](https://zhuanlan.zhihu.com/p/22094473)


## 参考

* [我眼中的 JavaScript 函数式编程](http://taobaofed.org/blog/2017/03/16/javascript-functional-programing/)
* [漫谈 JS 函数式编程（一）](https://www.h5jun.com/post/js-functional-1.html)
* [JavaScript函数式编程（二）](https://zhuanlan.zhihu.com/p/21926955)
* [JavaScript函数式编程（三）](https://zhuanlan.zhihu.com/p/22094473)



