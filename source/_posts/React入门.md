---
title: React入门
date: 2017-02-02 15:20:56
categories: coding
tags:
  - HTMl
  - CSS
  - JavaScript
  - React
---

本文是根据[React学习资源汇总](https://github.com/tsrot/study-notes/blob/master/React学习资源汇总.md)这个文章中提及的文章阅读后的总结，仅供个人学习使用，更清楚详细的介绍可以根据链接阅读原贴。

## React 产生的背景

[原文参考链接](http://www.cocoachina.com/webapp/20150721/12692.html)

在Web开发中，我们总需要将变化的数据实时反应到UI上，这时就需要对DOM进行操作。而复杂或频繁的DOM操作通常是性能瓶颈产生的原因（如何进行高性能的复杂DOM操作通常是衡量一个前端开发人员技能的重要指标）。React为此引入了虚拟DOM（Virtual DOM）的机制：在浏览器端用Javascript实现了一套DOM API。基于React进行开发时所有的DOM构造都是通过虚拟DOM进行，每当数据变化时，React都会重新构建整个DOM树，然后React将当前整个DOM树和上一次的DOM树进行对比，得到DOM结构的区别，然后仅仅将需要变化的部分进行实际的浏览器DOM更新。而且React能够批处理虚拟DOM的刷新，在一个事件循环（Event Loop）内的两次数据变化会被合并，例如你连续的先将节点内容从A变成B，然后又从B变成A，React会认为UI不发生任何变化，而如果通过手动控制，这种逻辑通常是极其复杂的。尽管每一次都需要构造完整的虚拟DOM树，但是因为虚拟DOM是内存数据，性能是极高的，而对实际DOM进行操作的仅仅是Diff部分，因而能达到提高性能的目的。这样，在保证性能的同时，开发者将不再需要关注某个数据的变化如何更新到一个或多个具体的DOM元素，而只需要关心在任意一个数据状态下，整个界面是如何Render的。

<!--more-->

如果你像在90年代那样写过服务器端Render的纯Web页面那么应该知道，服务器端所要做的就是根据数据Render出HTML送到浏览器端。如果这时因为用户的一个点击需要改变某个状态文字，那么也是通过刷新整个页面来完成的。服务器端并不需要知道是哪一小段HTML发生了变化，而只需要根据数据刷新整个页面。换句话说，任何UI的变化都是通过整体刷新来完成的。而React将这种开发模式以高性能的方式带到了前端，每做一点界面的更新，你都可以认为刷新了整个页面。至于如何进行局部更新以保证性能，则是React框架要完成的事情。

借用Facebook介绍React的视频中聊天应用的例子，当一条新的消息过来时，传统开发的思路如上图，你的开发过程需要知道哪条数据过来了，如何将新的DOM结点添加到当前DOM树上；而基于React的开发思路如下图，你永远只需要关心数据整体，两次数据之间的UI如何变化，则完全交给框架去做。可以看到，使用React大大降低了逻辑复杂性，意味着开发难度降低，可能产生Bug的机会也更少。

## React 是什么

和 Angular，Ember，Backbone 等等比起来 React 的表现如何？要如何处理数据？要如何连接服务器？JSX 到底是什么？“组件”又是如何定义的？

停。

立刻停下来。

**React 仅仅是 VIEW 层。**

React 通常和其他的 JavaScript 框架同时被提及，但是说“React 对比 Angular”却讲不通，因为它们之间是不可比较的。Angular 是一个完整的框架（包括一个 view 层），React 却并不是。这也是 React 很难于理解的原因，它虽然抽离自一个具备完整框架的生态系统中，但仅仅是一个 view 层。

React 提供了模板语法以及一些函数钩子用于基本的 HTML 渲染。这就是 React 全部的输出——HTML。你把 HTML / JavaScript 合到一起，被称为“组件”，允许把它们自己内部的状态存到内存中（比如在一个选项卡中哪个被选中），不过最后你只是吐出 HTML。

很明显，**你没办法单单使用 React 来创建一个功能完善的动态应用**。


## React 好处

### 通过查看一个源文件就可以知道你的组件将会如何渲染。

这是最大的好处，尽管这和 Angular 模板没什么不同。下面看一个真实的应用实例。

假设你需要在用户登录后更新站点头部的用户名。如果没有使用 JavaScript MVC 框架，可能要这么做：

```
<header>  
   <div class="name"></div>
</header> 
```

```
$.post('/login', credentials, function( user ) {
    // Modify the DOM here
    $('header .name').show().text( user.name );
});
```

按照我的经验，这些代码要毁掉你的生活甚至你同事的生活。如何对输出调试？谁来更新头部？谁还可以访问头部的 HTML？谁来维护名字的显示隐藏状态？这个 DOM 操作会让你的项目**像 GOTO 语句那样糟糕。**

在 React 中你可以像下面这样做：

```
render: function() {  
    return <header>
        { this.state.name ? <div>this.state.name</div> : null }
    </header>;
}
```

我们会清楚的分辨出这个组件可能会如何渲染。如果你知道这个语句，就会知道渲染后的输出。你没必要去记录程序的流程。在复杂应用中，特别是团队开发中，显得尤为重要。

### 将 JavaScript 和 HTML 绑定到 JSX 使组件更易懂

上面的那种把 HTML 和 JavaScript 混合在一起的写法可能让你很不适应。我们会很自然地拒绝将 JavaScript 放在 DOM 当中（比如 onClick 事件处理函数）即便我们是小小的开发者。但是，请一定要相信我；JSX 组件真的会让你的工作变得很“nice”。

按照传统，你会把视图（HTML）从功能（JavaScript）中分离开出来。这会产生一个巨大的 JavaScript 文件，包含了一个“页面”的所有功能需求，并且你必须记录复杂的流程，从 JS 到 HTML 到 JS 到悲痛欲绝。

捆绑功能直接标记和打包成一个可移植的，自主控制的“组件”，让你更开心，且减少脏乱的代码。因为 JavaScript 与 HTML 关系密切，揉到一起也正常。

## React 坏处

不要忘了 React 仅仅是个 **view 层**。

**下面这些都没有：**

* 事件系统（除了原生的 DOM 事件）
* AJAX 功能
* 数据层
* Promises
* 应用程序架构
* 实现以上功能的规范

## React 使用

### React 官方

* 官网地址：http://facebook.github.io/react/
* Github地址：https://github.com/facebook/react

### React 安装

最新的安装包可以在[React.cn](http://reactjs.cn/react/index.html)上下载。

可以使用[React 官方网址安装教程](https://facebook.github.io/react/docs/installation.html) 中的介绍来安装 React。

也可以使用[阮一峰的网络日志 - React 入门实例教程](http://www.ruanyifeng.com/blog/2015/03/react.html) 中下载 demo，已经自带 React 源码

#### 从 React.cn 下载

下载下来的文件夹中/build 目录下有在编译的时候需要导入的 js 文件，如下图所示：

![](http://ojt6zsxg2.bkt.clouddn.com/7a09dbb3821a2e5d122704e06ac91c31.png)

但是我们在本地使用的时候，最好添加上 **brower.min.js**，这个文件可以在别的地方下载得到。我找的是[菜鸟教程-React安装](http://www.runoob.com/react/react-install.html)中给出的菜鸟教程 React CDN 链接，打开并保存 brower.min.js 文件到 /build 目录下。

### HTML 模板

**该模板文件位置要能够根据路径找到 React 的 js 文件。**

使用 Reac 的网页模板大致如下：

```
<!DOCTYPE html>
<html>
<head>
	<meta charset="UTF-8">
	<title>Hello React!</title>
	<!--uncomment bellow when you need jQuery-->
	<!--<script src="http://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>-->
	<script src="build/react.js"></script>
	<script src="build/react-dom.js"></script>
	<script src="build/browser.min.js"></script>
</head>
<body>
	<div id="example"></div>
	<script type="text/babel">
		<!--Your React code here-->
	</script>
</body>
</html>
```

上面代码有两个地方需要注意。首先，最后一个 `<script>` 标签的 `type` 属性为 `text/babel` 。这是因为 React 独有的 JSX 语法，跟 JavaScript 不兼容。凡是使用 JSX 的地方，都要加上 `type="text/babel"` 。


其次，上面代码一共用了三个库： react.js 、react-dom.js 和 Browser.js ，它们必须首先加载。其中，react.js 是 React 的核心库，react-dom.js 是提供与 DOM 相关的功能，Browser.js 的作用是将 JSX 语法转为 JavaScript 语法，这一步很消耗时间，实际上线的时候，应该将它放到服务器完成。

### JSX 

React 使用 JSX 来替代常规的 JavaScript。
JSX 是一个看起来很像 XML 的 JavaScript 语法扩展。
我们不需要一定使用 JSX，但它有以下优点：

1. JSX 执行更快，因为它在编译为 JavaScript 代码后进行了优化。
2. 它是类型安全的，在编译过程中就能发现错误。
3. 使用 JSX 编写模板更加简单快速。

#### 使用 JSX

##### 在 HTML 文件中插入 JSX

我们可以使用上面的模板，将我们的代码写在模板中，如下：

```
<!DOCTYPE html>
<html>
<head>
	<meta charset="UTF-8">
	<title>Hello React!</title>
	<!--uncomment bellow when you need jQuery-->
	<!--<script src="http://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>-->
	<script src="build/react.js"></script>
	<script src="build/react-dom.js"></script>
	<script src="build/browser.min.js"></script>
</head>
<body>
	<div id="example"></div>
	<script type="text/babel">
		ReactDOM.render(
			<h1>Hello React!</h1>,
			document.getElementById("example")
		);
	</script>
</body>
</html>
```

显示效果如下：

![](http://ojt6zsxg2.bkt.clouddn.com/1357ca55f1ceb1b5b8e380f33a2ca82f.png)

页面中，只有`<div id='example'>`这里的内容是使用react渲染出来的，代码中这里是空的，依赖下面的js进行渲染页面中，只有`<div id='example'>`这里的内容是使用React渲染出来的，代码中这里是空的，依赖下面的js进行渲染

##### 使用独立文件

```
<body>
  <div id="example"></div>
  <script type="text/babel" src="helloworld_react.js"></script>
</body>
```

#### JSX 语法

```
var names = ['Alice', 'Emily', 'Kate'];

ReactDOM.render(
  <div>
  {
    names.map(function (name) {
      return <div>Hello, {name}!</div>
    })
  }
  </div>,
  document.getElementById('example')
);
```

上面代码体现了 JSX 的基本语法规则：遇到 HTML 标签（以 `<` 开头），就用 HTML 规则解析；遇到代码块（以 `{` 开头），就用 JavaScript 规则解析。

##### JSX CSS 样式

```
var myStyle = {
    fontSize: 100, // fontSize: '100px',
    color: '#FF0000'
};
ReactDOM.render(
    <h1 style = {myStyle}>菜鸟教程</h1>,
    document.getElementById('example')
);
```

React 会在指定元素数字后自动添加 `px`，或者使用 ` fontSize: '100px'` 

##### JSX 数组

JSX 允许在模板中插入数组，数组会自动展开所有成员：

```
var arr = [
  <h1>Hello world!</h1>,
  <h2>React is awesome</h2>,
];
ReactDOM.render(
  <div>{arr}</div>,
  document.getElementById('example')
);
```

数组中所有的元素都会被装在 div 中。

显示效果为：

![](http://ojt6zsxg2.bkt.clouddn.com/93505c6cd573be633cac36f704d44479.png)

##### JSX 注释

注释需要写在花括号中，实例如下：

```
ReactDOM.render(
    <div>
    <h1>菜鸟教程</h1>
    {/*注释...*/}
     </div>,
    document.getElementById('example')
);
```

**由于 JSX 就是 JavaScript，一些标识符像 class 和 for 不建议作为 XML 属性名。作为替代，React DOM 使用 className 和 htmlFor 来做对应的属性。**

### 组件

React 允许将代码封装成组件（component），然后像插入普通 HTML 标签一样，在网页中插入这个组件。`React.createClass` 方法就用于生成一个组件类

创建组件的时候，其中参数有很多，但都可以省略，唯有render不可以省略，因为这是用来表述这个插件被加载后，显示的是什么样子，它的返回结果，就是加载在页面上最终的样式。

```
var HelloMessage = React.createClass({
  render: function() {
    return <h1>Hello {this.props.name}</h1>;
  }
});

ReactDOM.render(
  <HelloMessage name="John" age={24} />,
  document.getElementById('example')
);
```

变量 HelloMessage 就是一个组件类。模板插入 `<HelloMessage />` 时，会自动生成 HelloMessage 的一个实例（下文的"组件"都指组件类的实例）。所有组件类都必须有自己的 render 方法，用于输出组件。

当我们传递参数时，写了两种方式，一种是 `name="John"` 另一种是 `age={24}`，这两种写法是有区别的，并不仅仅因为一个是str，一个是int。如果是str这种类型，写成 `name="xxx"` 或者 `name={"xxx"}` 都是可以的，加了 `{}` 的意思就是js中的变量，更加精确了。而后者 `age={24}` 是不可以去掉 `{}` 的，这样会引起异常，所以这里要注意下，并且建议任何类型都加上 `{}` 来确保统一

**组件类的第一个字母必须大写，否则会报错，比如HelloMessage不能写成helloMessage。另外，组件类只能包含一个顶层标签，否则也会报错。**

#### this.props

组件的用法与原生的 HTML 标签完全一致，可以任意加入属性，比如 `<HelloMessage name="John">` ，就是 HelloMessage 组件加入一个 name 属性，值为 John。组件的属性可以在组件类的 `this.props` 对象上获取，比如 name 属性就可以通过 `this.props.name` 读取。

添加组件属性，有一个地方需要注意，就是 class 属性需要写成 `className` ，for 属性需要写成 `htmlFor` ，这是因为 class 和 for 是 JavaScript 的保留字。

#### this.props.children

this.props 对象的属性与组件的属性一一对应，但是有一个例外，就是 `this.props.children` 属性。它表示组件的所有子节点

```
var NotesList = React.createClass({
  render: function() {
    return (
      <ol>
      {
        React.Children.map(this.props.children, function (child) {
          return <li>{child}</li>;
        })
      }
      </ol>
    );
  }
});

ReactDOM.render(
  <NotesList>
    <span>hello</span>
    <span>world</span>
  </NotesList>,
  document.body
);
```

上面代码的 NoteList 组件有两个 span 子节点，它们都可以通过 `this.props.children` 读取

这里需要注意， `this.props.children` 的值有三种可能：如果当前组件没有子节点，它就是 `undefined` ;如果有一个子节点，数据类型是 `object` ；如果有多个子节点，数据类型就是 `array` 。所以，处理 `this.props.children` 的时候要小心。
React 提供一个工具方法 `React.Children` 来处理 `this.props.children` 。我们可以用 `React.Children.map` 来遍历子节点，而不用担心 `this.props.children` 的数据类型是 `undefined` 还是 `object`。

#### PropTypes

组件的属性可以接受任意值，字符串、对象、函数等等都可以。有时，我们需要一种机制，验证别人使用组件时，提供的参数是否符合要求。

组件类的PropTypes属性，就是用来验证组件实例的属性是否符合要求

```
var MyTitle = React.createClass({
  propTypes: {
    title: React.PropTypes.string.isRequired,
  },

  render: function() {
     return <h1> {this.props.title} </h1>;
   }
});
```

上面的`Mytitle`组件有一个`title`属性。`PropTypes` 告诉 React，这个 `title` 属性是必须的，而且它的值必须是字符串。现在，我们设置 `title` 属性的值是一个数值

```
var data = 123;

ReactDOM.render(
  <MyTitle title={data} />,
  document.body
);
```

这样一来，title属性就通不过验证了。控制台会显示一行错误信息。

```
Warning: Failed prop type: Invalid prop `title` of type `number` supplied to `MyTitle`, expected `string`.
    in MyTitle
```

#### getDefaultProps

`getDefaultProps` 方法可以用来设置组件属性的默认值。

```
var MyTitle = React.createClass({
  getDefaultProps : function () {
    return {
      title : 'Hello World'
    };
  },

  render: function() {
     return <h1> {this.props.title} </h1>;
   }
});

ReactDOM.render(
  <MyTitle />,
  document.body
);
```

上面代码会输出"Hello World"。

#### 获取真实的DOM节点

组件并不是真实的 DOM 节点，而是存在于内存之中的一种数据结构，叫做虚拟 DOM （virtual DOM）。只有当它插入文档以后，才会变成真实的 DOM 。根据 React 的设计，所有的 DOM 变动，都先在虚拟 DOM 上发生，然后再将实际发生变动的部分，反映在真实 DOM上，这种算法叫做 DOM diff ，它可以极大提高网页的性能表现。

但是，有时需要从组件获取真实 DOM 的节点，这时就要用到 `ref` 属性

```
var MyComponent = React.createClass({
  handleClick: function() {
    this.refs.myTextInput.focus();
  },
  render: function() {
    return (
      <div>
        <input type="text" ref="myTextInput" />
        <input type="button" value="Focus the text input" onClick={this.handleClick} />
      </div>
    );
  }
});

ReactDOM.render(
  <MyComponent />,
  document.getElementById('example')
);
```

上面代码中，组件 `MyComponent` 的子节点有一个文本输入框，用于获取用户的输入。这时就必须获取真实的 DOM 节点，虚拟 DOM 是拿不到用户输入的。为了做到这一点，文本输入框必须有一个 `ref` 属性，然后 `this.refs.[refName]` 就会返回这个真实的 DOM 节点。

需要注意的是，由于 `this.refs.[refName]` 属性获取的是真实 DOM ，所以必须等到虚拟 DOM 插入文档以后，才能使用这个属性，否则会报错。上面代码中，通过为组件指定 `Click` 事件的回调函数，确保了只有等到真实 DOM 发生 `Click` 事件之后，才会读取 `this.refs.[refName]` 属性。

React 组件支持很多事件，除了 Click 事件以外，还有 KeyDown 、Copy、Scroll 等，完整的事件清单请查看[官方文档](https://facebook.github.io/react/docs/events.html#supported-events)。

#### 状态(this.state)

React 把组件看成是一个状态机（State Machines）。通过与用户的交互，实现不同状态，然后渲染 UI，让用户界面和数据保持一致。
React 里，只需更新组件的 `state`，然后根据新的 `state` 重新渲染用户界面（不要操作 DOM）。


```
var LikeButton = React.createClass({
  getInitialState: function() {
    return {liked: false};
  },
  handleClick: function(event) {
    this.setState({liked: !this.state.liked});
  },
  render: function() {
    var text = this.state.liked ? '喜欢' : '不喜欢';
    return (
      <p onClick={this.handleClick}>
        你<b>{text}</b>我。点我切换状态。
      </p>
    );
  }
});

ReactDOM.render(
  <LikeButton />,
  document.getElementById('example')
);
```

上面的实例中创建了 `LikeButton` 组件，`getInitialState` 方法用于定义初始状态，也就是一个对象，这个对象可以通过 `this.state` 属性读取。当用户点击组件，导致状态变化，`this.setState` 方法就修改状态值，每次修改以后，**自动调用** `this.render` 方法，再次渲染组件。

`getInitialState`这个函数在组件初始化的时候执行，必需返回NULL或者一个对象

##### state 和 props

**由于 `this.props` 和 `this.state` 都用于描述组件的特性，可能会产生混淆。一个简单的区分方法是，`this.props` 表示那些一旦定义，就不再改变的特性，而 `this.state` 是会随着用户互动而产生变化的特性。**

以下实例演示了如何在应用中组合使用 `state` 和 `props` 。我们可以在父组件中设置 ``， 并通过在子组件上使用 `props` 将其传递到子组件上。在 `render` 函数中, 我们设置 `name` 和 `site` 来获取父组件传递过来的数据。

```
var WebSite = React.createClass({
  getInitialState: function() {
    return {
      name: "菜鸟教程",
      site: "http://www.runoob.com"
    };
  },
 
  render: function() {
    return (
      <div>
        <Name name={this.state.name} />
        <Link site={this.state.site} />
      </div>
    );
  }
});

var Name = React.createClass({
  render: function() {
    return (
      <h1>{this.props.name}</h1>
    );
  }
});

var Link = React.createClass({
  render: function() {
    return (
      <a href={this.props.site}>
        {this.props.site}
      </a>
    );
  }
});

ReactDOM.render(
  <WebSite />,
  document.getElementById('example')
);
```


#### 表单

用户在表单填入的内容，属于用户跟组件的互动，所以不能用 `this.props` 读取

```
var Input = React.createClass({
  getInitialState: function() {
    return {value: 'Hello!'};
  },
  handleChange: function(event) {
    this.setState({value: event.target.value});
    console.log(this.state.value);
  },
  render: function () {
    var value = this.state.value;
    return (
      <div>
        <input type="text" value={value} onChange={this.handleChange} />
        <p>{value}</p>
      </div>
    );
  }
});

ReactDOM.render(<Input/>, document.body);
```

上面代码中，文本输入框的值，不能用 `this.props.value` 读取，而要定义一个 onChange 事件的回调函数，通过 `event.target.value` 读取用户输入的值。textarea 元素、select元素、radio元素都属于这种情况，更多介绍请参考[官方文档](https://facebook.github.io/react/docs/forms.html)。

我们在 `setState` 下面加了一个 `console`，通过控制台可以发现，每次打印的值并不是当前输入的值，而是上一次输入的值，这是怎么回事呢？在 `setState` 中，这是一个异步处理的函数，并不是同步的，`console` 在 `setState` 后立刻执行了，所以这时候状态还没有真正变更完，所以这里取到的状态仍旧是更新前的。这里要特殊注意下。如果需要在更新状态后，再执行操作怎么办呢，`setState` 还有第二个参数，接受一个 `callback`，我们尝试将 `handleChange` 中代码改成这样

```
handleChange: function(event) {
    this.setState({value: event.target.value}, function() {
    	console.log(this.state.value);
    });
  },
```

当你需要从子组件中更新父组件的 state 时，你需要在父组件通过创建事件句柄 (handleChange) ，并作为 prop (updateStateProp) 传递到你的子组件上。

```
var Content = React.createClass({
  render: function() {
    return  <div>
              <button onClick = {this.props.updateStateProp}>点我</button>
              <h4>{this.props.myDataProp}</h4>
           </div>
  }
});
var HelloMessage = React.createClass({
  getInitialState: function() {
    return {value: 'Hello Runoob!'};
  },
  handleChange: function(event) {
    this.setState({value: '菜鸟教程'})
  },
  render: function() {
    var value = this.state.value;
    return <div>
            <Content myDataProp = {value} 
              updateStateProp = {this.handleChange}></Content>
           </div>;
  }
});
ReactDOM.render(
  <HelloMessage />,
  document.getElementById('example')
);

```

在封装React时，我们往往按照最小单位封装，例如封装一个通用的div，一个通用的span，或者一个通用的table等，所以各自组件对应的方法都会随着组件封装起来，例如div有自己的方法可以更改背景色，span可以有自己的方法更改字体大小，或者table有自己的方法来更新table的内容等

这里我们用一个div相互嵌套的例子来查看父子组件如何相互嵌套及调用各自的方法。在下面的例子中，父组件与子组件都有一个方法，来改变自身的背景色，我们实现父子组件相互调用对方的方法，来改变对方的背景色。

```
<!DOCTYPE html>
<html>
<head>
	<meta charset="UTF-8">
	<title>Hello React!</title>
	<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u" crossorigin="anonymous">

	<!--uncomment bellow when you need jQuery-->
	<script src="http://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>
	<script src="build/react.js"></script>
	<script src="build/react-dom.js"></script>
	<script src="build/browser.min.js"></script>
</head>
<body>
	
	<h1><span class="label label-info">DEMO 5</span></h1>
    <br><br><br>
    <div id="well">
    </div>
	
	<script type="text/babel" >
		var Child = React.createClass({
            getInitialState: function() {
                return {color: ""};
            },
            changeColor: function(e) {
                this.setState({color: e.target.getAttribute("data-color")});
            },
            render: function() {
                return (
                    <div style={{backgroundColor: this.state.color}} className="col-xs-5 col-xs-offset-1 child">
                        <br/>
                        <ul className="list-inline">
                            <li><a href="#" data-color="#286090" className="btn btn-primary" onClick={this.props.parentChangeColor}>&nbsp;</a></li>
                            <li><a href="#" data-color="#31b0d5" className="btn btn-info" onClick={this.props.parentChangeColor}>&nbsp;</a></li>
                            <li><a href="#" data-color="#c9302c" className="btn btn-danger" onClick={this.props.parentChangeColor}>&nbsp;</a></li>
                            <li><a href="#" data-color="#ec971f" className="btn btn-warning" onClick={this.props.parentChangeColor}>&nbsp;</a></li>
                            <li><a href="#" data-color="#e6e6e6" className="btn btn-default" onClick={this.props.parentChangeColor}>&nbsp;</a></li>
                        </ul>
                    </div>
                );
            }
        });

        var Parent = React.createClass({
            getInitialState: function() {
                return {color: ""};
            },
            changeColor: function(e) {
                this.setState({color: e.target.getAttribute("data-color")});
            },
            child1ChangeColor: function(e) {
                this.refs["child1"].changeColor(e);
            },
            child2ChangeColor: function(e) {
                this.refs["child2"].changeColor(e);
            },
            render: function() {
                return (
                    <div style={{backgroundColor: this.state.color}} className="col-xs-10 col-xs-offset-1 parent">
                        <br/>
                        <ul className="list-inline">
                            <li>对应第一个child</li>
                            <li><a href="#" data-color="#286090" className="btn btn-primary" onClick={this.child1ChangeColor}>&nbsp;</a></li>
                            <li><a href="#" data-color="#31b0d5" className="btn btn-info" onClick={this.child1ChangeColor}>&nbsp;</a></li>
                            <li><a href="#" data-color="#c9302c" className="btn btn-danger" onClick={this.child1ChangeColor}>&nbsp;</a></li>
                            <li><a href="#" data-color="#ec971f" className="btn btn-warning" onClick={this.child1ChangeColor}>&nbsp;</a></li>
                            <li><a href="#" data-color="#e6e6e6" className="btn btn-default" onClick={this.child1ChangeColor}>&nbsp;</a></li>
                        </ul>
                        <ul className="list-inline">
                            <li>对应第二个child</li>
                            <li><a href="#" data-color="#286090" className="btn btn-primary" onClick={this.child2ChangeColor}>&nbsp;</a></li>
                            <li><a href="#" data-color="#31b0d5" className="btn btn-info" onClick={this.child2ChangeColor}>&nbsp;</a></li>
                            <li><a href="#" data-color="#c9302c" className="btn btn-danger" onClick={this.child2ChangeColor}>&nbsp;</a></li>
                            <li><a href="#" data-color="#ec971f" className="btn btn-warning" onClick={this.child2ChangeColor}>&nbsp;</a></li>
                            <li><a href="#" data-color="#e6e6e6" className="btn btn-default" onClick={this.child2ChangeColor}>&nbsp;</a></li>
                        </ul>
                        <hr/>

                        <Child ref="child1" parentChangeColor={this.changeColor} />  // 在父组件中调用子组件
                        <Child ref="child2" parentChangeColor={this.changeColor} />
                    </div>
                );
            }
        });


        ReactDOM.render(
            <Parent/>,
            document.getElementById('well')
        );
	</script>
</body>
</html>
```

* 父组件如何调用子组件：代码中，子组件里面定义了changeColor函数，用来接收onClick事件，并将点击的按钮的data-color属性值作为色值，更改到state中的color属性中，然后触发render来更新背景色。在父组件调用子组件时，我们写了，里面的ref=”child1”就是react中提供的一个属性标签，它与普通的props不同，这里写上ref=”xxx”后，在父组件中，使用this.refs[“child1”]就可以引用对应的子组件，当然这里的ref的值是可以随意定义，只要不重复就好。这样就可以实现组组件引用子组件，然后直接调用里面的方法就好，例如child1ChangeColor中就有this.refs["child1"].changeColor(e);的使用。连起来说下逻辑，在点击父组件中第一列中的按钮后，触发onClick事件，然后onClick事件后，传递到child1ChangeColor后，将事件传递进入，然后再次传递给子组件的changeColor中，因为子组件的changeColor是更改子组件自身的state，所以这时候子组件再次渲染，于是改变了颜色。这就是父组件调用子组件的逻辑。
* 再说下子组件何如调用父组件的方法，父组件自身也有一个changeColor函数，用来改变自身的背景色。当父组件调用子组件时，通过props，也就是第二个例子中讲的那样，通过参数的方式传递给子组件，这样子组件中就可以使用this.props.parentChangeColor，来把子组件的onClick事件传递给父组件的changeColor方法中，来改变父组件的背景色。这就是子组件调用父组件函数的方法。
* 还有一种情况，就是一个父组件下有多个子组件，但是子组件中并没有直接的关系，这时候如果一个子组件调用另一个子组件的方法，就得通过他们共同的父组件来作为中转，在父组件中增加函数来作为中转的函数，来实现子组件间的调用。


#### 组件 API

我们将讲解以下7个方法:

* 设置状态：setState
* 替换状态：replaceState
* 设置属性：setProps
* 替换属性：replaceProps
* 强制更新：forceUpdate
* 获取DOM节点：findDOMNode
* 判断组件挂载状态：isMounted

##### 设置状态 setState

```
setState(object nextState[, function callback])
```

参数说明:

* `nextState`，将要设置的新状态，该状态会和当前的state合并
* `callback`，可选参数，回调函数。该函数会在setState设置成功，且组件重新渲染后调用。

合并`nextState`和当前`state`，并重新渲染组件。**`setState`是React事件处理函数中和请求回调函数中触发UI更新的主要方法**。

不能在组件内部通过`this.state`修改状态，因为该状态会在调用`setState()`后被替换。

`setState()`并不会立即改变`this.state`，而是创建一个即将处理的state。`setState()`并不一定是同步的，为了提升性能React会批量执行state和DOM渲染。

`setState()`总是会触发一次组件重绘，除非在`shouldComponentUpdate()`中实现了一些条件渲染逻辑。

```
var Counter = React.createClass({
  getInitialState: function () {
    return { clickCount: 0 };
  },
  handleClick: function () {
    this.setState(function(state) {
      return {clickCount: state.clickCount + 1};
    });
  },
  render: function () {
    return (<h2 onClick={this.handleClick}>点我！点击次数为: {this.state.clickCount}</h2>);
  }
});
ReactDOM.render(
  <Counter />,
  document.getElementById('message')
);
```

##### 替换状态 replaceState

```
replaceState(object nextState[, function callback])
```

* `nextState`，将要设置的新状态，该状态会替换当前的state。
* `callback`，可选参数，回调函数。该函数会在`replaceState`设置成功，且组件重新渲染后调用。

`replaceState()`方法与`setState()`类似，但是方法只会保留`nextState`中状态，**原`state`对象中不在`nextState`对象中的属性值(状态)都会被删除**。

##### 设置属性 setProps

```
setProps(object nextProps[, function callback])
```

* nextProps，将要设置的新属性，该状态会和当前的props合并
* callback，可选参数，回调函数。该函数会在setProps设置成功，且组件重新渲染后调用。

设置组件属性，并重新渲染组件。

`props`相当于组件的数据流，它总是会从父组件向下传递至所有的子组件中。当和一个外部的JavaScript应用集成时，我们可能会需要向组件传递数据或通知`React.render()`组件需要重新渲染，可以使用`setProps()`。

更新组件，我可以在节点上再次调用`React.render()`，也可以通过`setProps()`方法改变组件属性，触发组件重新渲染。

##### 替换属性 replaceProps

```
replaceProps(object nextProps[, function callback])
```

* nextProps，将要设置的新属性，该属性会替换当前的props。
* callback，可选参数，回调函数。该函数会在replaceProps设置成功，且组件重新渲染后调用。

`replaceProps()`方法与`setProps`类似，但它会删除原有`props`

##### 强制更新 forceUpdate

```
强制更新：forceUpdate
```

`callback`，可选参数，回调函数。该函数会在组件render()方法调用后调用。

`forceUpdate()`方法会使组件调用自身的`render()`方法重新渲染组件，组件的子组件也会调用自己的`render()`。但是，组件重新渲染时，依然会读取`this.props`和`this.state`，如果状态没有改变，那么React只会更新DOM。

`forceUpdate()`方法适用于`this.props`和`this.state`之外的组件重绘（如：修改了`this.state`后），通过该方法通知React需要调用`render()`

**一般来说，应该尽量避免使用`forceUpdate()`，而仅从this.props和this.state中读取状态并由React触发`render()`调用。**

##### 获取DOM节点 findDOMNode

```
DOMElement findDOMNode()
```

返回值：DOM元素DOMElement

如果组件已经挂载到DOM中，该方法返回对应的本地浏览器 DOM 元素。当render返回null 或 false时，`this.findDOMNode()`也会返回null。从DOM 中读取值的时候，该方法很有用，如：获取表单字段的值和做一些 DOM 操作。

##### 判断组件挂载状态 isMounted

```
bool isMounted()
```

返回值：true或false，表示组件是否已挂载到DOM中

`isMounted()`方法用于判断组件是否已挂载到DOM中。可以使用该方法保证了`setState()`和`forceUpdate()`在异步场景下的调用不会出错。

#### 复合组件

React是基于组件化的开发，那么组件化开发最大的优点是什么？毫无疑问，当然是复用，下面我们来看看React中到底是如何实现组件的复用的

我们可以通过创建多个组件来合成一个组件，即把组件的不同功能点进行分离。

```
var WebSite = React.createClass({
  render: function() {
    return (
      <div>
        <Name name={this.props.name} />
        <Link site={this.props.site} />
      </div>
    );
  }
});

var Name = React.createClass({
  render: function() {
    return (
      <h1>{this.props.name}</h1>
    );
  }
});

var Link = React.createClass({
  render: function() {
    return (
      <a href={this.props.site}>
        {this.props.site}
      </a>
    );
  }
});

ReactDOM.render(
  <WebSite name="菜鸟教程" site=" http://www.runoob.com" />,
  document.getElementById('example')
);
```

#### 组件的生命周期

组件的[生命周期](https://facebook.github.io/react/docs/state-and-lifecycle.html)可分成三个状态：

```
Mounting：已插入真实 DOM
Updating：正在被重新渲染
Unmounting：已移出真实 DOM
```

生命周期的方法有：

![](http://ojt6zsxg2.bkt.clouddn.com/b71eac106855edc950d5c0427ed01c1c.png)


* `componentWillMount` : 在渲染前调用,在客户端也在服务端。
* `componentDidMount` : 在第一次渲染后调用，只在客户端。之后组件已经生成了对应的DOM 结构，可以通过this.getDOMNode()来进行访问。 如果你想和其他JavaScript框架一起使用，可以在这个方法中调用setTimeout, setInterval或者发送AJAX请求等操作(防止异部操作阻塞UI)。
* `componentWillUpdate` : 在组件接收到新的props或者state但还没有render时被调用。在初始化时不会被调用。
* `componentDidUpdate` :  在组件完成更新后立即调用。在初始化时不会被调用。
* `componentWillUnmount` : 在组件从 DOM 中移除的时候立刻被调用。

此外，React 还提供两种特殊状态的处理函数。

* `componentWillReceiveProps` :  在组件接收到一个新的prop时被调用。这个方法在初始化render时不会被调用。
* `shouldComponentUpdate` :  返回一个布尔值。在组件接收到新的props或者state时被调用。在初始化时或者使用forceUpdate时不被调用。可以在你确认不需要更新组件时使用。

```
var Hello = React.createClass({
  getInitialState: function () {
    return {
      opacity: 1.0
    };
  },

  componentDidMount: function () {
    this.timer = setInterval(function () {
      var opacity = this.state.opacity;
      opacity -= .05;
      if (opacity < 0.1) {
        opacity = 1.0;
      }
      this.setState({
        opacity: opacity
      });
    }.bind(this), 100); // 如果不 bind，这里的 this 就是 window
  },

  render: function () {
    return (
      <div style={{opacity: this.state.opacity}}>
        Hello {this.props.name}
      </div>
    );
  }
});

ReactDOM.render(
  <Hello name="world"/>,
  document.body
);
```


上面代码在hello组件加载以后，通过 `componentDidMount` 方法设置一个定时器，每隔100毫秒，就重新设置组件的透明度，从而引发重新渲染。


另外，组件的style属性的设置方式也值得注意，不能写成

```
style="opacity:{this.state.opacity};"
```

而要写成

```
style={{opacity: this.state.opacity}}
```

这是因为 React 组件样式是一个对象，所以第一重大括号表示这是 JavaScript 语法，第二重大括号表示样式对象。

#### AJAX

组件的数据来源，通常是通过 Ajax 请求从服务器获取，可以使用 `componentDidMount` 方法设置 Ajax 请求，等到请求成功，再用 `this.setState` 方法重新渲染 UI 

当使用异步加载数据时，在组件卸载前使用 componentWillUnmount 来取消未完成的请求。

```
var UserGist = React.createClass({
  getInitialState: function() {
    return {
      username: '',
      lastGistUrl: ''
    };
  },

  componentDidMount: function() {
    $.get(this.props.source, function(result) {
      var lastGist = result[0];
      if (this.isMounted()) {
        this.setState({
          username: lastGist.owner.login,
          lastGistUrl: lastGist.html_url
        });
      }
    }.bind(this));
  },

  render: function() {
    return (
      <div>
        {this.state.username}'s last gist is
        <a href={this.state.lastGistUrl}>here</a>.
      </div>
    );
  }
});

ReactDOM.render(
  <UserGist source="https://api.github.com/users/octocat/gists" />,
  document.body
);
```

上面代码使用 jQuery 完成 Ajax 请求，这是为了便于说明。React 本身没有任何依赖，完全可以不用jQuery，而使用其他库。

**第二个例子**


```
<!DOCTYPE html>
<html>
<head>
	<meta charset="UTF-8">
	<title>Hello React!</title>
	<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u" crossorigin="anonymous">

	<!--uncomment bellow when you need jQuery-->
	<script src="http://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>
	<script src="build/react.js"></script>
	<script src="build/react-dom.js"></script>
	<script src="build/browser.min.js"></script>
</head>
<body>
	<h1><span class="label label-info">DEMO 4</span></h1>
        <br><br><br>
        <div class="well div" id="well">
        </div>
	<script type="text/babel" >
		var Text = React.createClass({
            getInitialState: function() {
                return {cur_time: 0};
            },
            request: function() {
                $.ajax({
                    type: "GET",
                    url: "http://xxx",
                    success: function(data){
                        this.setState({cur_time: data.timestamp});
                    }.bind(this),
                    complete: function(){
                        this.setState({cur_time: Date.parse(new Date()) / 1000});
                    }.bind(this)
                });
            },
            componentDidMount: function(){
                setInterval(this.request, 1000);
            },
            render: function() {
                return (
                    <div className="col-xs-12">
                        当前时间{this.state.cur_time}
                        <div className={ this.state.cur_time % 2 == 0?"hidden": "col-xs-6 alert alert-success"}>
                            <span>最后一位奇数</span>
                        </div>
                        <div className={ this.state.cur_time % 2 != 0?"hidden": "col-xs-6 alert alert-danger"}>
                            <span>最后一位偶数</span>
                        </div>
                    </div>
                );
            }
        });

        ReactDOM.render(
            <Text></Text>,
            document.getElementById('well')
        );
	</script>
</body>
</html>
```

* 这个例子中，页面每秒会请求一次网络，将请求到的数据中时间戳更新到状态中。但是这个例子中实际上并没有使用服务返回的时间戳，因为我在公司使用测试网的接口拿数据，但是开放给大家要换一个公网能拿到时间戳的api，简单的找了下没找到，也不想用公司的接口试，所以算是mock了下~，在 `complete` 中每次都取浏览器时间来更新。
* 仍旧是先看代码，相比于上一个例子，这里多了两个函数 `request` 和 `componentDidMount` ，其中 `request` 是请求网络的函数，`componentDidMount` 是react内部的函数，也是react生命周期的一部分，它会在render第一次渲染后执行，而且只会执行一次。
* 先看 `request`，一个普通的ajax请求，在 `success` 回调中，假设服务器返回的json为{“timestamp”: 1467425645}，那 `data.timestamp` 就取得是1467425645，然后将值赋给 `state` 中的 `cur_time` 属性。这时状态发生了变化，render函数会重新渲染。当然例子中 `success` 不会被访问到，因为那个url根本不存在，所以我在complete回调中来写了一个状态变化，模拟success成功。
* **为什么success回调函数最后会加一个 `bind(this)`**？因为这个函数已经不是react内部的函数了，它是一个外部函数，它里面的this并不是react组件中的this，所以要将外部函数绑定到react中，并能使用react内部的方法，例如 `setState` ，就要在函数最后 `bind(this)` ，这样就完成了绑定。
* 再看下 `componentDidMount` 函数，这个函数在render渲染前会执行，里面的代码也很简单，增加了一个定时器，1秒钟执行一次request。
这里应该在加一个回调，就是定时器在初始化时创建，却没有对应的销毁，所以在组件销毁的时候，应该在这个生命周期中销毁定时器。








