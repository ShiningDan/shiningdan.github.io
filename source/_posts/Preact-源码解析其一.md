---
title: Preact 源码解析之 setState 相关流程
date: 2017-09-23 20:35:58
categories: coding
tags:
  - JavaScript
  - Preact
---

Preact 作为实现大部分 React 的接口，并且专注于轻量的框架，在前一阵 React 由于专利事件受到质疑的时候，进入了大家的视野，并且成为了在不得已需要放弃 React 之后的首选。虽然在今天，React 在 Twiter 上宣布其转向了 MIT 许可证，但也不影响我们对本框架设计的学习。本文作为 Preact 源码解读系列的第一篇，将介绍一些关于 Preact 的基础源码。

<!--more-->

注：笔者作为一个应届生，水平有限，在有的地方无法理解开发者代码的精髓，大家可以来 [developit/preact | GitHub](https://github.com/developit/preact/) 获得最新源代码。

其中，关于 `package.json` 中的组成，可以查看 [Preact 源码剖析（一）解读 package.json](https://lewis617.github.io/2017/03/29/preact-package-json/) 这篇文章，讲的很好。

## src/preact.js

首先，我们可以在 `src` 目录下找到项目的主入口代码 `preact.js`，我们在 `import preact from 'preact'` 的时候，引入的函数，对象都是从这个入口中提供的。`preact.js` 中的内容也很简单，介绍了各部分的函数实现分别在哪个位置：

```
import { h, h as createElement } from './h';
import { cloneElement } from './clone-element';
import { Component } from './component';
import { render } from './render';
import { rerender } from './render-queue';
import options from './options';
```

## src/component.js

当我们需要创建一个组建的时候，就需要引入 `Component`，下面我们来看一看 `Component` 中代码是怎样组成的：

首先，对外提供了 `Component` 类，用来继承，其代码为：

```
export function Component(props, context) {
    // 
	this._dirty = true;

	/** @public
	 *	@type {object}
	 */
	this.context = context;

	/** @public
	 *	@type {object}
	 */
	this.props = props;

	/** @public
	 *	@type {object}
	 */
	this.state = this.state || {};
}
```

当然，这一段代码只是提供了最基础的框架，下面使用了 `extend` 方法为 `Component.prototype` 提供了新的方法：

```
extend(Component.prototype, {

	setState(state, callback) {
		...
	},

	forceUpdate(callback) {
	    ...
	},
	
	render() {}
});
```

我们先暂时不看 `setState` 等方法的实现，先看看 `extend` 方法实现了什么功能，在 `util.js` 中，我们找到了 `extend` 方法的源代码，功能很简单，就是遍历 `props` 对象的属性，然后赋值给 `obj`：

```
export function extend(obj, props) {
	for (let i in props) obj[i] = props[i];
	return obj;
}
```

则上面的代码含义是，给 `Component.prototype` 添加 `setState, forceUpdate, render` 这三个方法，使得所有 `Component` 的对象都具有这三个方法。下面，我们来分别看看这三个方法的实现原理：

### setState

```
setState(state, callback) {
	let s = this.state;
	// 在 prevState 中保留原来的状态
	if (!this.prevState) this.prevState = extend({}, s);
	// 将新的状态通过 extend 添加到现在的 state 中
	extend(s, typeof state==='function' ? state(s, this.props) : state);
	if (callback) (this._renderCallbacks = (this._renderCallbacks || [])).push(callback);
	enqueueRender(this);
},
```

我们在第三行代码中，可以看到，最终使用 `extend` 方法，将传入的新的 `state` 添加到现有的状态中，这里出现了一个平常我们不注意的小点，就是传给 `setState` 的参数，除了对象以外，还可以是一个函数：

```
(state, this.props) => {...}
```

传递的第一个参数是现在的 `state`，第二个参数是现在的 `props`，返回的对象是新的 `nextState`。

关于传递函数给 `setState` 这一点，有一些文章在介绍这个技巧，比如：[[译] 在 setState 中使用函数替代对象](https://juejin.im/entry/5873b04f61ff4b006d4d45f7) 以及 [為何要在 React 的 setState() 內傳入 function](https://neighborhood999.github.io/2017/03/06/learning-functional-setState-in-react/) 和 [Functional setState](http://padma0.farbox.com/post/ji-zhu/2017-03-15)，也就是说，在 `setState` 中传入函数，可以保证每一次使用的都是最新的 state，而不是老的 state 的叠加。

如果在 setState 中传递了回调函数 `callback`，则本组件会创建一个 `_renderCallbacks` 数组，用来存放回调函数。

最后将本组件添加到 Render 队列中。

我们来看一看 `enqueueRender` 中做了什么：

```
export function enqueueRender(component) {
	if (!component._dirty && (component._dirty = true) && items.push(component)==1) {
		(options.debounceRendering || defer)(rerender);
	}
}
```

在这段函数中，其实做了一件事，就是判断传入的 `component` 中 `_dirty` 是否为 `true`。如果 `_dirty` 为 `false`，并且当前队列中没有需要渲染的组件，则添加该组件，然后将队列中的所有组件都做一次渲染。

```
items.push(component)==1
```

这句话的意思是，原来 `item` 中没有元素，在推入了 `component` 中，返回值为 `length = 1`，则说明有元素需要进行渲染，开始渲染。如果返回值不是 `1`，则说明已经在一个 `rerender` 的进行中，则不会再次进行 `rerender`，而是等待在本次 `rerender` 中被 `pop` 出来。

在默认情况下，所有组件初始化的时候，赋值 `_dirty` 都是 `true`，也就是不立刻触发 `rerender`，那什么时候 `_dirty` 被赋值为 `false` 呢？我们下面再分析。

假设，我们遇到了一个 `_dirty` 为 `false` 的组件，然后会发生什么呢，会激活以下的代码：

```
(options.debounceRendering || defer)(rerender);
```

让我们来看看 `options.debounceRendering` 是什么：

当我进 `options` 中查看，额。。。什么都没有，返回了一个空对象：

```
export default {};
```

那我们只能在 `defer` 中查看它是什么了：

```
export const defer = typeof Promise=='function' ? Promise.resolve().then.bind(Promise.resolve()) : setTimeout;
```

它做的功能，就是判断现有环境中是否有 `Promise`，如果有 `Pormise`，则在当前 `macrotask` 后面的 `microtask` 中执行回调函数，也就是 `rerender`，如果没有 `Promise`，则在下一个 `macrotask` 中执行回调函数。

OK，我们也知道了这一句代码是做什么事情，那我们来看 `rerender` 中做了什么事情吧：

```
export function rerender() {
	let p, list = items;
	items = [];
	while ( (p = list.pop()) ) {
		if (p._dirty) renderComponent(p);
	}
}
```

我们可以看到，`rerender` 中做的事情，就是遍历所有需要重新渲染的组件，然后对其调用 `renderComponent` 方法，那 `renderComponent` 做了什么呢？由于 `renderComponent` 方法太长，所以我又开了一个小结来进行分析

### renderComponent

```
isUpdate = component.base

// if updating
if (isUpdate) {
	component.props = previousProps;
	component.state = previousState;
	component.context = previousContext;
	if (opts!==FORCE_RENDER
		&& component.shouldComponentUpdate
		&& component.shouldComponentUpdate(props, state, context) === false) {
		skip = true;
	}
	else if (component.componentWillUpdate) {
		component.componentWillUpdate(props, state, context);
	}
	component.props = props;
	component.state = state;
	component.context = context;
}
```

首先判断该组件是不是已经在一个更新中（可能是因为调用 `forceUpdate` 方法等原因造成），如果在一个更新中，则判断本次进入 `rerender` 的原因：

1. 如果不是因为 `forceUpdate` 进入更新，则判断 `shouldComponentUpdate` 的返回值是否为 `false`，如果返回值要求不更新，则设置 `skip = true`
2. 如果是因为 `forceUpdate` 进入更新，则运行该组件的 `componentWillUpdate`

首先，在进行了判断是否在更新中以后，设置 `component._dirty = false;`，表明该组件又可以进行 `rerender`，允许在 `enqueueRender` 中被添加到队列里面。越早设置 `_dirty = false;`，则对下一次 `rerender` 的响应越快。

接下来，首先判断 `skip`，来决定要不要对组件进行更新。当 `skip === false` 的时候，说明要进行组件重新渲染的操作，操作如下：

首先，运行 `rendered = component.render(props, state, context);`，获得当前组件的返回值。

然后，判断该组件中是否有子组件，如果有子组件，则更新 `content`：

```
// context to pass to the child, can be updated via (grand-)parent component
if (component.getChildContext) {
	context = extend(extend({}, context), component.getChildContext());
}
```

获得子组件：

```
let childComponent = rendered && rendered.nodeName
```

下面进行判断子组件是否存在，如果子组件存在，将会进行一下的操作：

```
if (typeof childComponent==='function') {
	// set up high order component link

    //得到子组件的 props
	let childProps = getNodeProps(rendered);
	inst = initialChildComponent;

	if (inst && inst.constructor===childComponent && childProps.key==inst.__key) {
		setComponentProps(inst, childProps, SYNC_RENDER, context, false);
	}
	else {
		toUnmount = inst;

		component._component = inst = createComponent(childComponent, childProps, context);
		inst.nextBase = inst.nextBase || nextBase;
		inst._parentComponent = component;
		setComponentProps(inst, childProps, NO_RENDER, context, false);
		renderComponent(inst, SYNC_RENDER, mountAll, true);
	}

	base = inst.base;
}
```

如果子组件存在，首先得到子组件的 `props`，我们来看看 `getNodeProps` 的实现：

```
/**
 * Reconstruct Component-style `props` from a VNode.
 * Ensures default/fallback values from `defaultProps`:
 * Own-properties of `defaultProps` not present in `vnode.attributes` are added.
 * @param {VNode} vnode
 * @returns {Object} props
 */
export function getNodeProps(vnode) {
	let props = extend({}, vnode.attributes);
	props.children = vnode.children;

	let defaultProps = vnode.nodeName.defaultProps;
	if (defaultProps!==undefined) {
		for (let i in defaultProps) {
			if (props[i]===undefined) {
				props[i] = defaultProps[i];
			}
		}
	}

	return props;
}
```

获得的是一个 VNode 对象的 `attributes`，`children`，以及所有的 `defaultProps`。至于这三个属性分别代表什么，我会在接下来的文章中分析，现在我们就暂且认为拿到了子组件的必要属性吧~

下面，首先进行了一次 `if` 判断：

```
if (inst && inst.constructor===childComponent && childProps.key==inst.__key)
```

我所理解的判断内容是：

1. `inst` 的初始化为：`component._component`，也就指的是现在还没有 `rerender` 的时候，子组件是什么
2. `childComponent` 的初始化为：`component.render(props, state, context).nodeName`，也就指的是新的 `render` 函数子组件是什么

所以，这个 `if` 语句的判断，也就是想知道该组件的子组件类型是否发生了变化？如果没有发生变化，则在原来 `inst` 上更新新的 `props` 等。如果变成了新的子组件，则将原来的 `inst` 删掉，然后创建新的子组件。

所以，在 `if` 语句中，执行的代码如下：

```
if (inst && inst.constructor===childComponent && childProps.key==inst.__key) {
	setComponentProps(inst, childProps, SYNC_RENDER, context, false);
}
```

其中，设置了同步方法为 `SYNC_RENDER`，也就是将子组件添加到渲染队列中，在本次 macrotask 中进行渲染： `enqueueRender(component);`

`setComponentProps` 的代码如下，大致做的工作为，在某个已有的组件中替换新的属性：

```
export function setComponentProps(component, props, opts, context, mountAll) {
	if (component._disable) return;
	component._disable = true;

	if ((component.__ref = props.ref)) delete props.ref;
	if ((component.__key = props.key)) delete props.key;

	if (!component.base || mountAll) {
		if (component.componentWillMount) component.componentWillMount();
	}
	else if (component.componentWillReceiveProps) {
		component.componentWillReceiveProps(props, context);
	}

	if (context && context!==component.context) {
		if (!component.prevContext) component.prevContext = component.context;
		component.context = context;
	}

	if (!component.prevProps) component.prevProps = component.props;
	component.props = props;

	component._disable = false;

	if (opts!==NO_RENDER) {
		if (opts===SYNC_RENDER || options.syncComponentUpdates!==false || !component.base) {
			renderComponent(component, SYNC_RENDER, mountAll);
		}
		else {
			enqueueRender(component);
		}
	}

	if (component.__ref) component.__ref(component);
}
```

如果子组件的类型发生了变化，或者变成了另一个同类型的对象，则删除原来的子组件，然后创建新的子组件：

```
else {
    toUnmount = inst;
    
    component._component = inst = createComponent(childComponent, childProps, context);
    inst.nextBase = inst.nextBase || nextBase;
    inst._parentComponent = component;
    setComponentProps(inst, childProps, NO_RENDER, context, false);
    renderComponent(inst, SYNC_RENDER, mountAll, true);
}
```

其中，创建新的组件使用的是 `createComponent` 方法：

```
/** Create a component. Normalizes differences between PFC's and classful Components. */
export function createComponent(Ctor, props, context) {
    // 在这里，components 对象是一个包含所有组件的一个组件池
	let list = components[Ctor.name],
		inst;

	if (Ctor.prototype && Ctor.prototype.render) {
		inst = new Ctor(props, context);
		Component.call(inst, props, context);
	}
	else {
		inst = new Component(props, context);
		inst.constructor = Ctor;
		inst.render = doRender;
	}


	if (list) {
		for (let i=list.length; i--; ) {
		    // 如果该组件被重新渲染了，则在组件池中删除原有组件
			if (list[i].constructor===Ctor) {
				inst.nextBase = list[i].nextBase;
				list.splice(i, 1);
				break;
			}
		}
	}
	return inst;
}
```

最后，在创建完组件以后，调用 `renderComponent` 方法，对该新生成的子组件进行重新渲染。

如果新生成的子组件不存在，则使用使用 `diff` 算法来对进行处理：

```
if (typeof childComponent==='function') {
    ...
} else {
    cbase = initialBase;
    
	// destroy high order component link
	toUnmount = initialChildComponent;
	if (toUnmount) {
		cbase = component._component = null;
	}

	if (initialBase || opts===SYNC_RENDER) {
		if (cbase) cbase._component = null;
		base = diff(cbase, rendered, context, mountAll || !isUpdate, initialBase && initialBase.parentNode, true);
	}
}
```

现在，我们要看看 Preact 的 `diff` 算法是怎么样的流程了。首先，我们来看看 `diff` 算法的主流程：

```
/** Apply differences in a given vnode (and it's deep children) to a real DOM Node.
 *	@param {Element} [dom=null]		A DOM node to mutate into the shape of the `vnode`
 *	@param {VNode} vnode			A VNode (with descendants forming a tree) representing the desired DOM structure
 *	@returns {Element} dom			The created/mutated element
 *	@private
 */
export function diff(dom, vnode, context, mountAll, parent, componentRoot) {
	// diffLevel having been 0 here indicates initial entry into the diff (not a subdiff)
	if (!diffLevel++) {
		// when first starting the diff, check if we're diffing an SVG or within an SVG
		isSvgMode = parent!=null && parent.ownerSVGElement!==undefined;

		// hydration is indicated by the existing element to be diffed not having a prop cache
		hydrating = dom!=null && !(ATTR_KEY in dom);
	}

	let ret = idiff(dom, vnode, context, mountAll, componentRoot);

	// append the element if its a new parent
	if (parent && ret.parentNode!==parent) parent.appendChild(ret);

	// diffLevel being reduced to 0 means we're exiting the diff
	if (!--diffLevel) {
		hydrating = false;
		// invoke queued componentDidMount lifecycle methods
		if (!componentRoot) flushMounts();
	}

	return ret;
}
```

首先，`diff` 算法接受的参数中，第一个是当前的 DOM 结构： `dom`，第二个是将要成为的 DOM 结构。在 `diff` 算法中，使用 `diffLevel` 来控制算法是否要退出。在 `diff` 算法要退出的时候，会通过：

```
if (!componentRoot) flushMounts();
```

来判断是否要调用所有组件的 `componentDidMount` 方法。

在 `diff` 算法中，计算得到不同组件的功能是通过 `idiff` 得到的：

```
let ret = idiff(dom, vnode, context, mountAll, componentRoot);
```

由于 `idiff` 很长。。恩，我们只能分布拆开来看其功能。

首先，对传入的新节点的类型进行判断：

```
// empty values (null, undefined, booleans) render as empty Text nodes
	if (vnode==null || typeof vnode==='boolean') vnode = '';
```

如果传入的是 `null, undefined, booleans`，则渲染的结果是一个空的 Text Node。

如果传入的是 `string, number` 类型，则创建，或者更新节点为一个新的 Text Node。

```
// Fast case: Strings & Numbers create/update Text nodes.
if (typeof vnode==='string' || typeof vnode==='number') {

	// update if it's already a Text node:
	if (dom && dom.splitText!==undefined && dom.parentNode && (!dom._component || componentRoot)) {
		/* istanbul ignore if */ /* Browser quirk that can't be covered: https://github.com/developit/preact/commit/fd4f21f5c45dfd75151bd27b4c217d8003aa5eb9 */
		if (dom.nodeValue!=vnode) {
			dom.nodeValue = vnode;
		}
	}
	else {
		// it wasn't a Text node: replace it with one and recycle the old Element
		out = document.createTextNode(vnode);
		if (dom) {
			if (dom.parentNode) dom.parentNode.replaceChild(out, dom);
			recollectNodeTree(dom, true);
		}
	}

	out[ATTR_KEY] = true;

	return out;
}
```

这里会判断，需要被更新的节点，如果本身就是一个 Text Node，则更新其中的字符串即可。如果不是一个 Text Node，则创建一个新的 Text Node，然后替换该节点。

最后要进行的一步，如果有替换 DOM 元素的行为，则将原来的 DOM 元素删除 `recollectNodeTree(dom, true)`：

```
/** Recursively recycle (or just unmount) a node and its descendants.
 *	@param {Node} node						DOM node to start unmount/removal from
 *	@param {Boolean} [unmountOnly=false]	If `true`, only triggers unmount lifecycle, skips removal
 */
export function recollectNodeTree(node, unmountOnly) {
	let component = node._component;
	if (component) {
		// if node is owned by a Component, unmount that component (ends up recursing back here)
		unmountComponent(component);
	}
	else {
		// If the node's VNode had a ref function, invoke it with null here.
		// (this is part of the React spec, and smart for unsetting references)
		if (node[ATTR_KEY]!=null && node[ATTR_KEY].ref) node[ATTR_KEY].ref(null);

		if (unmountOnly===false || node[ATTR_KEY]==null) {
			removeNode(node);
		}

		removeChildren(node);
	}
}
```

`recollectNodeTree` 的逻辑如上，如果这个 node 本身是一个 Component，将该组件删除，并且调用该组件的 `componentDidMount()`，否则，调用 `removeChildren` 来删除该组件。`removeChildren` 方法的内容很简单，就是找到 `node` 的父元素，然后调用 `removeChild`：

```
export function removeNode(node) {
	let parentNode = node.parentNode;
	if (parentNode) parentNode.removeChild(node);
}
```

如果传入的 `vnode` 既不是 `null, undefined, booleans`，也不是 `string, number`，而是一个 `function` 类型，说明 `vnode` 代表一个组件，则调用 `buildComponentFromVNode` 直接生成新的组件：

```
// If the VNode represents a Component, perform a component diff:
let vnodeName = vnode.nodeName;
if (typeof vnodeName==='function') {
	return buildComponentFromVNode(dom, vnode, context, mountAll);
}
```

我们可以看一看 `buildComponentFromVNode`：

```
/** Apply the Component referenced by a VNode to the DOM.
 *	@param {Element} dom	The DOM node to mutate
 *	@param {VNode} vnode	A Component-referencing VNode
 *	@returns {Element} dom	The created/mutated element
 *	@private
 */
export function buildComponentFromVNode(dom, vnode, context, mountAll) {
	let c = dom && dom._component,
		originalComponent = c,
		oldDom = dom,
		isDirectOwner = c && dom._componentConstructor===vnode.nodeName,
		isOwner = isDirectOwner,
		props = getNodeProps(vnode);
	while (c && !isOwner && (c=c._parentComponent)) {
		isOwner = c.constructor===vnode.nodeName;
	}

	if (c && isOwner && (!mountAll || c._component)) {
		setComponentProps(c, props, ASYNC_RENDER, context, mountAll);
		dom = c.base;
	}
	else {
		if (originalComponent && !isDirectOwner) {
			unmountComponent(originalComponent);
			dom = oldDom = null;
		}

		c = createComponent(vnode.nodeName, props, context);
		if (dom && !c.nextBase) {
			c.nextBase = dom;
			// passing dom/oldDom as nextBase will recycle it if unused, so bypass recycling on L229:
			oldDom = null;
		}
		setComponentProps(c, props, SYNC_RENDER, context, mountAll);
		dom = c.base;

		if (oldDom && dom!==oldDom) {
			oldDom._component = null;
			recollectNodeTree(oldDom, false);
		}
	}

	return dom;
}
```

在这段代码中，我们会比较，如果 `dom` 是否和 `vnode` 是同样类型的组件，如果是，则将 `vnode` 的属性赋值给 `dom`。如果不是，则将 `dom` 元素替换为新的 `vnode` 元素，然后将原来的组件回收。

下一步，判断 `vnode` 的类型是不是 SVG，然后针对 SVG 进行处理并返回新的节点。

如果 `dom` 不存在或者 `dom` 的类型与 `vnode.nodeName` 不同，则创建一个新的 DOM 元素，并且把之前 `dom` 元素中的子元素挂载到新的 DOM 元素中，然后回收 `dom` 元素，代码部分如下：

```
// If there's no existing element or it's the wrong type, create a new one:
vnodeName = String(vnodeName);
if (!dom || !isNamedNode(dom, vnodeName)) {
	out = createNode(vnodeName, isSvgMode);

	if (dom) {
		// move children into the replacement node
		while (dom.firstChild) out.appendChild(dom.firstChild);

		// if the previous Element was mounted into the DOM, replace it inline
		if (dom.parentNode) dom.parentNode.replaceChild(out, dom);

		// recycle the old element (skips non-Element node types)
		recollectNodeTree(dom, true);
	}
}
```

然后针对以上种情况，对 `vnode.children` 调用 `innerDiffNode`，将 `vnode.chilren` 中的元素添加到新创建的 DOM 元素中。然后调用 `diffAttributes` 将 `vnode` 的属性添加到新创建的 DOM 元素中。




关于 `diff` 算法，我自己也感觉了解的不是很透彻，如果有疑问的同学，可以参考这一篇文章 [从Preact了解一个类React的框架是怎么实现的(二): 元素diff](https://juejin.im/post/59c76e515188254f584132af)



## 参考

* [Preact 源码剖析（一）解读 package.json](https://lewis617.github.io/2017/03/29/preact-package-json/)
* [[译] 在 setState 中使用函数替代对象](https://juejin.im/entry/5873b04f61ff4b006d4d45f7)








