---
title: Preact 源码解析之元素创建流程
date: 2017-09-28 17:09:09
categories: coding
tags:
  - JavaScript
  - Preact
---


Preact 作为实现大部分 React 的接口，并且专注于轻量的框架，在前一阵 React 由于专利事件受到质疑的时候，进入了大家的视野，并且成为了在不得已需要放弃 React 之后的首选。虽然在今天，React 在 Twiter 上宣布其转向了 MIT 许可证，但也不影响我们对本框架设计的学习。本文作为 Preact 源码解读系列的第二篇，比较简短，将介绍 Preact 如何将 JSX 代码转义成 DOM 输出的流程

<!--more-->

我们知道，在 React 中，对 JSX 代码的处理，是使用 `React.React.createElement` 来转换的，例如：

```
import ReactDOM from 'react-dom'

const App = (props)=>(<div>Hello World</div>)
ReactDOM.render(<APP />, document.body);
```

经过 Bable 转换后，得到的结果如下：

```
var App = function App(props) {
  return React.createElement(
    'div',
    null,
    'Hello World'
  );
};
```

同样，在 Preact 中，对于 JSX 语法结构的处理，是使用 `h` 方法来处理的，比如：

```
import { h, render, Component } from 'preact;
const Index = () => {
  return (
    <div id="test">Test</div>
  );
}
render(<Index />, document.getElementById('container'))
```

使用 Babel 转换的结果为：

```
var Index = function Index() {
  return (0, _preact.h)(
    'div',
    { id: 'test' },
    'Test'
  );
};
(0, _preact.render)((0, _preact.h)(Index, null), document.getElementById('container'));
```

也就是使用 `h` 方法来处理。其中，`h` 方法的第一个参数是 `nodeName`，代表元素的类型，第二个参数是元素的属性，第三个参数是元素中包裹的内容。同样，在调用 `render` 的时候，也是对生成的 `Index` 组件，调用 `h` 方法，我们可以看一下，`h` 函数内部到底做了什么：

其实 `h` 函数的代码并不多，所以可以先贴出来，然后慢慢分析流程

```
const stack = [];

const EMPTY_CHILDREN = [];

/** JSX/hyperscript reviver
*	Benchmarks: https://esbench.com/bench/57ee8f8e330ab09900a1a1a0
 *	@see http://jasonformat.com/wtf-is-jsx
 *	@public
 */
export function h(nodeName, attributes) {
	let children=EMPTY_CHILDREN, lastSimple, child, simple, i;
	// 首先，存下来除了 node 和 attributes 以外的其他参数，由于第三个参数之后都是该节点的子元素，所以 stack 中存储的都是子元素
	for (i=arguments.length; i-- > 2; ) {
		stack.push(arguments[i]);
	}
	if (attributes && attributes.children!=null) {
	    // 如果 attributes 属性中也有 children，并且该节点子元素为空，则将 attributes 也添加到 stack 中
		if (!stack.length) stack.push(attributes.children);
		delete attributes.children;
	}
	// 遍历所有的子节点
	while (stack.length) {
	    // 如果 stack 中的对象是一个数组，取出数组中的所有元素，添加到 stack 中
		if ((child = stack.pop()) && child.pop!==undefined) {
			for (i=child.length; i--; ) stack.push(child[i]);
		}
		else {
			if (typeof child==='boolean') child = null;
            
            // 针对子元素的类型进行判断，如果 typeof nodeName === 'function'，则代表子元素为一个子组件
			if ((simple = typeof nodeName!=='function')) {
				if (child==null) child = '';
				else if (typeof child==='number') child = String(child);
				else if (typeof child!=='string') simple = false;
			}
			
            // 如果是字符串类型，则做一个字符串拼接
			if (simple && lastSimple) {
				children[children.length-1] += child;
			}
			else if (children===EMPTY_CHILDREN) {
				children = [child];
			}
			// 如果是复杂类型，则添加为 children 的新元素
			else {
				children.push(child);
			}

			lastSimple = simple;
		}
	}

    // 最后，生成一个 VNode 的对象，将之前对本组件的解析结果都存在这个对象中，VNode 本身只是一个空白对象： export function VNode() {}
	let p = new VNode();
	p.nodeName = nodeName;
	p.children = children;
	p.attributes = attributes==null ? undefined : attributes;
	p.key = attributes==null ? undefined : attributes.key;

	// if a "vnode hook" is defined, pass every created VNode to it
	if (options.vnode!==undefined) options.vnode(p);

	return p;
}
```

其中，有几个注意点：

```
if (simple && lastSimple) {
 children[children.length - 1] += child;
}
```

做一个字符串拼接，是因为某些编译器会将下面代码

```
let foo = <div id="foo">Hello World!</div>;
```

转化为:

```
var foo = h(
"div",
{ id: "foo" },
"Hello",
"World!"
);
```

最后将处理子节点的传入数组children中，现在传入children中的节点有三种类型: 纯字符串、代表dom节点的字符串以及代表组件的函数(或者是类)

还有：

我们可以看最后一个转化的例子：

```
/* jsx
class App extends Component{
//....
}

class Child extends Component{
//....
}
*/

let Element = <App><Child>Hello World!</Child></App>

//js
var Element = h(
  App,
  null,
  h(
    Child,
    null,
    "Hello World!"
  )
);

//转化为的元素节点
{
    nodeName: ƒ App(argument), 
    children: [
        {
            nodeName: ƒ Child(argument),
            children: ["Hello World!"],
            attributes: undefined,
            key: undefined
        }
    ], 
    attributes: undefined,
    key: undefined
}
```

最后，使用 `render` 方法将生成的 VNode 对象添加到 DOM 树中：

```
render(<Index />, document.getElementById('container'))
```

其中 `render` 的源代码为：

```
export function render(vnode, parent, merge) {
	return diff(merge, vnode, {}, false, parent, false);
}
```

其实也是用的 `diff` 算法，至于 `diff` 算法的解析，可以参考我以前的文章 [Preact 源码解析之 setState 相关流程](http://zyuchen.com/post/preact-setState)

## 参考

* [从Preact了解一个类React的框架是怎么实现的(一): 元素创建](https://juejin.im/post/59b69b6e5188257e6b6d7bfc)