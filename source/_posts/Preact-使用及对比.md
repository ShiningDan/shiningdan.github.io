---
title: Preact 使用及对比
date: 2017-09-22 13:30:00
categories: coding
tags:
  - JavaScript
  - Preact
---

其实本文也没什么值得记录的东西，就是最近 React 的专利被炒的沸沸扬扬，百度已经宣布放弃 React，阿里也有打算抛弃 React，所以作为一个 React 的替代物，Preact 很快就吸引的大家的目光。当然，Preact 作为 React 的 3kb 轻量化方案，几乎封装了 React 原有的 API，所以本文只是对 Preact 的 API 测试，以后的文章中会有对 Preact 源码的解读。

<!--more-->

[Getting Started | Preact](https://preactjs.com/guide/getting-started)

并且，Preact 官方也提供了从 React 迁移到 Preact 的方法：[用 Preact 替换 React](https://preactjs.com/guide/switching-to-preact)

首先，我们先来测试一下 Preact 的生命周期：

关于 Preact 的生命周期测试都在我的测试项目 [preact-test](https://github.com/ShiningDan/preact-test.git) 里面了，从测试的结果上来看，React 里面的生命周期在 Preact 里面都存在，并且保持了一致。

![](http://ojt6zsxg2.bkt.clouddn.com/89a081eeb64176523ee4a64914fd07bc.png)

除此以外，Preact 和 React 之间有什么不同呢？我们可以从官网上得到 [与 React 的不同之处](https://preactjs.com/guide/differences-to-react)

比如说，有几点我们可能需要注意：

1. 从 Preact 4.0 起支持 函数 refs 引用。字符串 `refs` 引用在 `preact-compact` 中支持，在本身中并不支持字符串 `refs`
2. 你可以只用 class 作为 CSS 的类。 className 也仍然被支持， 但推荐使用 class。关于为什么推荐使用 class 而不是 className，可以参考这个回答 [What's the reason behind using class instead of className?](https://github.com/developit/preact/issues/103)
3. PropType 验证：并非所有人使用 PropTypes，所以它们并非 Preact 的核心
4. Children: 在 Preact 中并非必要的 Preact, 因为 props.children 总是一个数组
5. Synthetic Events: Preact 的浏览器支持并不需要这个开销，具体的 React 事件机制可以查看一些现有的文章，React 支持用 `addEventListener` 绑定原生事件，在 `componentWillUnmount` 中解绑，也支持使用 `JAX` 的合成事件，事件监听会被自动销毁，使用事件代理实现。

## 从 React 切换到 Preact

在 [用 Preact 替换 React](https://preactjs.com/guide/switching-to-preact) 这篇文章中介绍了如何将现有的代码从 React 切换到 Preact。当然，切换并不是一件很难的事情，我们主要要观察其中的不同点。在这里，需要注意的是这一段话：

```
背景： JSX 扩展语法是从 React 独立出来的语法，Babel 和 Bublé 这些编译器默认会把调用 React.createElement() 来编译 JSX。这其中有一些历史原因，但是值得探究的是，这种方式来源于一个已有的技术 Hyperscript。Preact 向它致敬，并试图用使用 h() 来更好理解并简化 jsx。

长话短说： 我们需要把 React.createElement() 转换成 preact 的 h()
```

在 JSX 中，`h()` 用于生成每一个元素：

```
<div /> 编译成 h('div')

<Foo /> 编译成 h(Foo)

<a href="/">Hello</a> 编译成 h('a', { href:'/' }, 'Hello')
```

## Preact 提供的一些常用方法

### Preact.Component.render(props, state)

render() 方法是所有组件必需的一个方法。它可以接受组件的 porps 和 state作为参数，并且这个方法应该返回一个Preact元素或者返回null。

```
import { Component } from 'preact';

class MyComponent extends Component {
    render(props, state) {
        // props === this.props
        // state === this.state

        return <h1>Hello, {props.name}!</h1>;
    }
}
```

### Preact.render()

```
render(component, containerNode, [replaceNode])
```

将一个Preact组件渲染到 containerNode 这个DOM节点上。 返回一个对渲染的DOM节点的引用。

如果提供了可选的DOM节点参数 replaceNode 并且是 containerNode 的子节点，Preact将使用它的diffing算法来更新或者替换该元素节点。否则，Preact将把渲染的元素添加到 containerNode 上。

```
import { render } from 'preact';

// 下面这些例子展示了如何在具有以下HTML标记的页面中执行render()
// <div id="container">
//   <h1>My App</h1>
// </div>

const container = document.getElementById('container');

render(MyComponent, container);
// 将 MyComponent 添加到 container 中
//
// <div id="container">
//   <h1>My App</h1>
//   <MyComponent />
// </div>

const existingNode = container.querySelector('h1');

render(MyComponent, container, existingNode);
// 对比 MyComponent 和 <h1>My App</h1> 的不同
//
// <div id="container">
//   <MyComponent />
// </div>
```

### Preact.h() / Preact.createElement()

```
h(nodeName, attributes, [...children])
```

返回具有给定 `attributes` 属性的Preact虚拟DOM元素。

所有的剩余参数都会被放置在一个 children 数组中， 并且是以下所列的任意一种：

* 纯粹的值（字符串、数字、布尔值、null、undefined等）
* 虚拟DOM元素
* 上面两种的嵌套数组

```
import { h } from 'preact';

h('div', { id: 'foo' }, 'Hello!');
// <div id="foo">Hello!</div>

h('div', { id: 'foo' }, 'Hello', null, ['Preact!']);
// <div id="foo">Hello Preact!</div>

h(
    'div',
    { id: 'foo' },
    h('span', null, 'Hello!')
);
// <div id="foo"><span>Hello!</span></div>
```


