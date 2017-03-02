---
title: React Router入门
date: 2017-03-01 22:49:11
categories: coding
tags:
  - HTML
  - CSS
  - JavaScript
  - react-router
---

本文是根据[React学习资源汇总](https://github.com/tsrot/study-notes/blob/master/React学习资源汇总.md)这个文章中提及的文章阅读后的总结，仅供个人学习使用，更清楚详细的介绍可以根据链接阅读原贴。


React Router 保持 UI 与 URL 同步。它拥有简单的 API 与强大的功能例如代码缓冲加载、动态路由匹配、以及建立正确的位置过渡处理。你第一个念头想到的应该是 URL，而不是事后再想起。

<!--more-->

## 简介

### 路由配置

#### 不使用 React Router

```
import React from 'react'
import { render } from 'react-dom'

const About = React.createClass({/*...*/})
const Inbox = React.createClass({/*...*/})
const Home = React.createClass({/*...*/})

const App = React.createClass({
  getInitialState() {
    return {
      route: window.location.hash.substr(1)
    }
  },

  componentDidMount() {
    window.addEventListener('hashchange', () => {
      this.setState({
        route: window.location.hash.substr(1)
      })
    })
  },

  render() {
    let Child
    switch (this.state.route) {
      case '/about': Child = About; break;
      case '/inbox': Child = Inbox; break;
      default:       Child = Home;
    }

    return (
      <div>
        <h1>App</h1>
        <ul>
          <li><a href="#/about">About</a></li>
          <li><a href="#/inbox">Inbox</a></li>
        </ul>
        <Child />
      </div>
    )
  }
})

ReactDOM.render(<App />, document.body)
```

当 URL 的 hash 部分（指的是 `#` 后的部分）变化后，`<App>` 会根据 `this.state.route` 来渲染不同的 `<Child>`。看起来很直接，但它很快就会变得复杂起来。

现在设想一下 `Inbox` 下面嵌套一些分别对应于不同 URL 的 UI 组件，就像下面这样的**列表-详情**视图：

```
path: /inbox/messages/1234

+---------+------------+------------------------+
| About   |    Inbox   |                        |
+---------+            +------------------------+
| Compose    Reply    Reply All    Archive      |
+-----------------------------------------------+
|Movie tomorrow|                                |
+--------------+   Subject: TPS Report          |
|TPS Report        From:    boss@big.co         |
+--------------+                                |
|New Pull Reque|   So ...                       |
+--------------+                                |
|...           |                                |
+--------------+--------------------------------+
```

还可能有一个状态页，用于在没有选择 message 时展示：

```
path: /inbox

+---------+------------+------------------------+
| About   |    Inbox   |                        |
+---------+            +------------------------+
| Compose    Reply    Reply All    Archive      |
+-----------------------------------------------+
|Movie tomorrow|                                |
+--------------+   10 Unread Messages           |
|TPS Report    |   22 drafts                    |
+--------------+                                |
|New Pull Reque|                                |
+--------------+                                |
|...           |                                |
+--------------+--------------------------------+
```

为了让我们的 URL 解析变得更智能，我们需要编写很多代码来实现指定 URL 应该渲染哪一个嵌套的 UI 组件分支：`App -> About, App -> Inbox -> Messages -> Message, App -> Inbox -> Messages -> Stats`，等等。

### 使用 React Router 后

```
import React from 'react'
import { render } from 'react-dom'

// 首先我们需要导入一些组件...
import { Router, Route, Link, browserHistory } from 'react-router'

// 然后我们从应用中删除一堆代码和
// 增加一些 <Link> 元素...
const App = React.createClass({
  render() {
    return (
      <div>
        <h1>App</h1>
        {/* 把 <a> 变成 <Link> */}
        <ul>
          <li><Link to="/about">About</Link></li>
          <li><Link to="/inbox">Inbox</Link></li>
        </ul>

        {/*
          接着用 `this.props.children` 替换 `<Child>`
          router 会帮我们找到这个 children
        */}
        {this.props.children}
      </div>
    )
  }
})

// 最后，我们用一些 <Route> 来渲染 <Router>。
// 这些就是路由提供的我们想要的东西。
ReactDOM.render((
  <Router history={browserHistory}>
    <Route path="/" component={App}>
      <Route path="about" component={About} />
      <Route path="inbox" component={Inbox} />
    </Route>
  </Router>
```

React Router 知道如何为我们搭建嵌套的 UI，因此我们不用手动找出需要渲染哪些 `<Child>` 组件。举个例子，对于一个完整的 `/about` 路径，React Router 会搭建出 `<App><About /></App>`。

`Router`组件本身只是一个容器，真正的路由要通过`Route`组件定义。


关于 History 的介绍可以看：[Histories](https://github.com/ReactTraining/react-router/blob/v3.0.0/docs/guides/Histories.md)


A brief explanation of the different options:

`browserHistory`: bases your path of the full url starting with the route '/'. So if your SPA is at www.myapp.com, the root path will coorespond to `www.myapp.com/` by default. So if you wanted a route to `www.myapp.com/blog`, you would need to define a router path of '`/blog`'.

`hasHistory`: does the same thing but the root starts from the the first `#` character found in the url: `example.com/#/some/path` would correspond to: `/some/path`

`createMemoryHistory`: used for testing and for server rendering. This option does not read from or manipulate the address bar.

`Route`组件定义了URL路径与组件的对应关系。你可以同时使用多个`Route`组件。

```
<Router history={hashHistory}>
  <Route path="/" component={App}/>
  <Route path="/repos" component={Repos}/>
  <Route path="/about" component={About}/>
</Router>
```

上面代码中，用户访问`/repos`（比如`http://localhost:8080/#/repos`）时，加载`Repos`组件；访问`/about`（`http://localhost:8080/#/about`）时，加载`About`组件。

`App` 组件要写成下面的样子。

```
render() {
    return <div>
      {this.props.children}
    </div>
  }
```

上面代码中，`App` 组件的`this.props.children`属性就是子组件。

子路由也可以不写在`Router`组件里面，单独传入`Router`组件的`routes`属性。

```
let routes = <Route path="/" component={App}>
  <Route path="/repos" component={Repos}/>
  <Route path="/about" component={About}/>
</Route>;

<Router routes={routes} history={browserHistory}/>
```

#### path 属性

`Route`组件的`path`属性指定路由的匹配规则。这个属性是可以省略的，这样的话，不管路径是否匹配，总是会加载指定组件。





#### 添加更多的 UI

现在我们准备在 inbox UI 内嵌套 inbox messages。

```
// 新建一个组件让其在 Inbox 内部渲染
const Message = React.createClass({
  render() {
    return <h3>Message</h3>
  }
})

const Inbox = React.createClass({
  render() {
    return (
      <div>
        <h2>Inbox</h2>
        {/* 渲染这个 child 路由组件 或者默认的样式 */}
        {this.props.children || "Welcome to your Inbox"}
      </div>
    )
  }
})

ReactDOM.render((
  <Router history={browserHistory}>
    <Route path="/" component={App}>
      <Route path="about" component={About} />
      <Route path="inbox" component={Inbox}>
        {/* 添加一个路由，嵌套进我们想要嵌套的 UI 里 */}
        <Route path="messages/:id" component={Message} />
      </Route>
    </Route>
  </Router>
), document.body)
```

现在访问 URL `inbox/messages/Jkei3c32` 将会匹配到一个新的路由，并且它成功指向了 `App -> Inbox -> Message` 这个 UI 的分支。

```
<App>
  <Inbox>
    <Message params={ {id: 'Jkei3c32'} } />
  </Inbox>
</App>
```

当用户访问 `Inbox` 的时候，会先加载 `App` 组件，再加载 `Inbox` 组件。

#### 获取 URL 参数

为了从服务器获取 message 数据，我们首先需要知道它的信息。当渲染组件时，React Router 会自动向 Route 组件中注入一些有用的信息，尤其是路径中动态部分的参数。我们的例子中，它指的是 `:id`。

```
const Message = React.createClass({

  componentDidMount() {
    // 来自于路径 `/inbox/messages/:id`
    const id = this.props.params.id

    fetchMessage(id, function (err, message) {
      this.setState({ message: message })
    })
  },

  // ...

})
```

你也可以通过 query 字符串来访问参数。比如你访问 `/foo?bar=baz`，你可以通过访问 `this.props.location.query.bar` 从 Route 组件中获得 `"baz"` 的值。

## 基础

### 路由配置

```
import React from 'react'
import ReactDOM from 'react-dom'
import { Router, Route, Link, browserHistory } from 'react-router'

const App = React.createClass({
  render() {
    return (
      <div>
        <h1>App</h1>
        <ul>
          <li><Link to="/about">About</Link></li>
          <li><Link to="/inbox">Inbox</Link></li>
        </ul>
        {this.props.children}
      </div>
    )
  }
})

const About = React.createClass({
  render() {
    return <h3>About</h3>
  }
})

const Inbox = React.createClass({
  render() {
    return (
      <div>
        <h2>Inbox</h2>
        {this.props.children || "Welcome to your Inbox"}
      </div>
    )
  }
})

const Message = React.createClass({
  render() {
    return <h3>Message {this.props.params.id}</h3>
  }
})

ReactDOM.render((
  <Router history={browserHistory}>
    <Route path="/" component={App}>
      <Route path="about" component={About} />
      <Route path="inbox" component={Inbox}>
        <Route path="messages/:id" component={Message} />
      </Route>
    </Route>
  </Router>
), document.body)
```

URL 与页面的对应关系为
```
URL	                            组件
/	                            App
/about	                     App -> About
/inbox	                     App -> Inbox
/inbox/messages/:id     App -> Inbox -> Message
```

#### 添加首页

想象一下当 URL 为 `/` 时，我们想渲染一个在 `App` 中的组件。不过在此时，`App` 的 `render` 中的 `this.props.children` 还是 `null`。这种情况我们可以使用 `IndexRoute` 来设置一个默认页面。

```
import { IndexRoute } from 'react-router'

const Dashboard = React.createClass({
  render() {
    return <div>Welcome to the app!</div>
  }
})

ReactDOM.render((
  <Router history={browserHistory}>
    <Route path="/" component={App}>
      {/* 当 url 为/时渲染 Dashboard */}
      <IndexRoute component={Dashboard} />
      <Route path="about" component={About} />
      <Route path="inbox" component={Inbox}>
        <Route path="messages/:id" component={Message} />
      </Route>
    </Route>
  </Router>
), document.body)
```

现在，`App` 的 `render` 中的 `this.props.children` 将会是 `<Dashboard>`这个元素。这个功能类似 Apache 的 `DirectoryIndex` 以及 nginx的 `index` 指令，上述功能都是在当请求的 URL 匹配某个目录时，允许你制定一个类似 `index.html` 的入口文件。

**也就是通过 `<Link>` component 指向的组件在页面刚开始的时候是不存在的，只有当页面的 url 转到了对应的位置，该组件才会被 append 到当前页面中。而 `<IndexRoute>` 的行为是，在当前 url 中会默认显示，如果跳转到其他的 url 中则会消失。**

URL 与页面的对应关系为
```
URL	                            组件
/	                         App -> Dashboard
/about	                     App -> About
/inbox	                     App -> Inbox
/inbox/messages/:id     App -> Inbox -> Message
```

#### 让 UI 从 URL 中解耦出来

如果我们可以将 `/inbox` 从 `/inbox/messages/:id` 中去除，并且还能够让 Message 嵌套在 `App -> Inbox` 中渲染，那会非常赞。绝对路径可以让我们做到这一点。

```
ReactDOM.render((
  <Router history={browserHistory}>
    <Route path="/" component={App}>
      {/* 当 url 为/时渲染 Dashboard */}
      <IndexRoute component={Dashboard} />
      <Route path="about" component={About} />
      <Route path="inbox" component={Inbox}>
        {/* 使用 /messages/:id 替换 messages/:id */}
        <Route path="/messages/:id" component={Message} />
      </Route>
    </Route>
  </Router>
), document.body)
```

在多层嵌套路由中使用绝对路径的能力让我们对 URL 拥有绝对的掌控。我们无需在 URL 中添加更多的层级，从而可以使用更简洁的 URL。

URL 与页面的对应关系为
```
URL	                            组件
/	                         App -> Dashboard
/about	                     App -> About
/inbox	                     App -> Inbox
/messages/:id     App -> Inbox -> Message
```

#### 兼容旧的 URL

 现在任何人访问 /inbox/messages/5 都会看到一个错误页面。:(
 
 不要担心。我们可以使用 `<Redirect>` 使这个 URL 重新正常工作。
 
```
 ReactDOM.render((
  <Router history={browserHistory}>
    <Route path="/" component={App}>
      {/* 当 url 为/时渲染 Dashboard */}
      <IndexRoute component={Dashboard} />
      <Route path="about" component={About} />
      <Route path="inbox" component={Inbox}>
        {/* 使用 /messages/:id 替换 messages/:id */}
        <Route path="/messages/:id" component={Message} />
        {/* 跳转 /inbox/messages/:id 到 /messages/:id */}
        <Redirect from="messages/:id" to="/messages/:id" />
      </Route>
    </Route>
  </Router>
), document.body)
```

现在当有人点击 `/inbox/messages/5` 这个链接，他们会被自动跳转到 `/messages/5`

#### 进入和离开的Hook

Route 可以定义 onEnter 和 onLeave 两个 hook ，这些hook会在页面跳转确认时触发一次。这些 hook 对于一些情况非常的有用，例如权限验证或者在路由跳转前将一些数据持久化保存起来。

在路由跳转过程中，onLeave hook 会在所有将离开的路由中触发，从最下层的子路由开始直到最外层父路由结束。然后onEnter hook会从最外层的父路由开始直到最下层子路由结束。

继续我们上面的例子，如果一个用户点击链接，从 `/messages/5` 跳转到 `/about`，下面是这些 hook 的执行顺序：

`/messages/:id` 的 `onLeave`
`/inbox` 的 `onLeave`
`/about` 的 `onEnter`

### 路由匹配原理

路由拥有三个属性来决定是否“匹配“一个 URL：

#### 嵌套关系

React Router 使用路由嵌套的概念来让你定义 view 的嵌套集合，当一个给定的 URL 被调用时，整个集合中（命中的部分）都会被渲染。嵌套路由被描述成一种树形结构。React Router 会深度优先遍历整个路由配置来寻找一个与给定的 URL 相匹配的路由。

#### 路径语法

路由路径是匹配一个（或一部分）URL 的 一个字符串模式。大部分的路由路径都可以直接按照字面量理解，除了以下几个特殊的符号：

* `:paramName` – 匹配一段位于 /、? 或 # 之后的 URL。 命中的部分将被作为一个参数
* `()` – 在它内部的内容被认为是可选的
* `*` – 匹配任意字符（非贪婪的）直到命中下一个字符或者整个 URL 的末尾，并创建一个 splat 参数

```
<Route path="/hello/:name">         // 匹配 /hello/michael 和 /hello/ryan
<Route path="/hello(/:name)">       // 匹配 /hello, /hello/michael 和 /hello/ryan
<Route path="/files/*.*">           // 匹配 /files/hello.jpg 和 /files/path/to/hello.jpg
```

**参数是自动生成在组件 props 中的对象，如 name 可以在组件的 `this.props.params.name` 中找到。**

如果一个路由使用了相对路径，那么完整的路径将由它的所有祖先节点的路径和自身指定的相对路径拼接而成。使用绝对路径可以使路由匹配行为忽略嵌套关系。

#### 优先级

路由算法会根据定义的顺序自顶向下匹配路由。因此，当你拥有两个兄弟路由节点配置时，你必须确认前一个路由不会匹配后一个路由中的路径。例如，千万**不要**这么做：

```
<Route path="/comments" ... />
<Redirect from="/comments" ... />
```

### Histories

React Router 是建立在 history 之上的。 简而言之，一个 history 知道如何去监听浏览器地址栏的变化， 并解析这个 URL 转化为 `location` 对象， 然后 router 使用它匹配到路由，最后正确地渲染对应的组件。

常用的 history 有三种形式， 但是你也可以使用 React Router 实现自定义的 history。

* browserHistory
* hashHistory
* createMemoryHistory

你可以从 React Router 中引入它们，然后将它们传递给`<Router>`：

```
import { browserHistory } from 'react-router'

render(
  <Router history={browserHistory} routes={routes} />,
  document.getElementById('app')
)
```

#### browserHistory

Browser history 是使用 React Router 的应用推荐的 history。它使用浏览器中的 History API 用于处理 URL，创建一个像 `example.com/some/path` 这样真实的 URL 。

##### 服务器配置

服务器需要做好处理 URL 的准备。处理应用启动最初的 `/` 这样的请求应该没问题，但当用户来回跳转并在 `/accounts/123` 刷新时，服务器就会收到来自 `/accounts/123` 的请求，这时你需要处理这个 URL 并在响应中包含 JavaScript 应用代码。

#### hashHistory

Hash history 使用 URL 中的 hash（`#`）部分去创建形如 `example.com/#/some/path` 的路由。

#### createMemoryHistory

Memory history 不会在地址栏被操作或读取。这就解释了我们是如何实现服务器渲染的。同时它也非常适合测试和其他的渲染环境（像 React Native ）。

和另外两种history的一点不同是你必须创建它，这种方式便于测试

```
const history = createMemoryHistory(location)
```

### 默认路由（IndexRoute）与 IndexLink

#### 默认路由（IndexRoute）

在解释 默认路由(IndexRoute) 的用例之前，我们来设想一下，一个不使用默认路由的路由配置是什么样的：

```
<Router>
  <Route path="/" component={App}>
    <Route path="accounts" component={Accounts}/>
    <Route path="statements" component={Statements}/>
  </Route>
</Router>
```

当用户访问 / 时, App 组件被渲染，但组件内的子元素却没有， App 内部的 this.props.children 为 undefined 。 你可以简单地使用 `{this.props.children || XXX }` 来渲染一些默认的 UI 组件。

router 允许你使用 IndexRoute ，以使 `Home` 作为最高层级的路由出现.

```
<Router>
  <Route path="/" component={App}>
    <IndexRoute component={Home}/>
    <Route path="accounts" component={Accounts}/>
    <Route path="statements" component={Statements}/>
  </Route>
</Router>
```

现在 `App` 能够渲染 `{this.props.children}` 了， 我们也有了一个最高层级的路由，使 `Home` 可以参与进来。

### Index Links

如果你在这个 app 中使用 `<Link to="/">Home</Link>` , 它会一直处于激活状态，因为所有的 URL 的开头都是 `/` 。 这确实是个问题，因为我们仅仅希望在 `Home` 被渲染后，激活并链接到它。

如果需要在 `Home` 路由被渲染后才激活的指向 `/` 的链接，请使用 `<IndexLink to="/">Home</IndexLink>`

## 高级用法

### 动态路由

对于大型应用来说，一个首当其冲的问题就是所需加载的 JavaScript 的大小。程序应当只加载当前渲染页所需的 JavaScript。有些开发者将这种方式称之为“代码分拆” —— 将所有的代码分拆成多个小包，在用户浏览过程中按需加载。

React Router 里的路径匹配以及组件加载都是异步完成的，不仅允许你延迟加载组件，并且可以延迟加载路由配置。在首次加载包中你只需要有一个路径定义，路由会自动解析剩下的路径。

Route 可以定义 `getChildRoutes`，`getIndexRoute` 和 `getComponents` 这几个函数。它们都是异步执行，并且只有在需要时才被调用。我们将这种方式称之为 “逐渐匹配”。 React Router 会逐渐的匹配 URL 并只加载该 URL 对应页面所需的路径配置和组件。

```
const CourseRoute = {
  path: 'course/:courseId',

  getChildRoutes(location, callback) {
    require.ensure([], function (require) {
      callback(null, [
        require('./routes/Announcements'),
        require('./routes/Assignments'),
        require('./routes/Grades'),
      ])
    })
  },

  getIndexRoute(location, callback) {
    require.ensure([], function (require) {
      callback(null, require('./components/Index'))
    })
  },

  getComponents(location, callback) {
    require.ensure([], function (require) {
      callback(null, require('./components/Course'))
    })
  }
}
```

### 跳转前确认

React Router 提供一个 routerWillLeave 生命周期钩子，这使得 React 组件可以拦截正在发生的跳转，或在离开 route 前提示用户。routerWillLeave 返回值有以下两种：

* return false 取消此次跳转
* return 返回提示信息，在离开 route 前提示用户进行确认。

你可以在 route 组件 中引入 Lifecycle mixin 来安装这个钩子。

```
import { Lifecycle } from 'react-router'

const Home = React.createClass({

  // 假设 Home 是一个 route 组件，它可能会使用
  // Lifecycle mixin 去获得一个 routerWillLeave 方法。
  mixins: [ Lifecycle ],

  routerWillLeave(nextLocation) {
    if (!this.state.isSaved)
      return 'Your work is not saved! Are you sure you want to leave?'
  },

  // ...

})
```

如果你在组件中使用了 ES6 类，你可以借助 react-mixin 包将 Lifecycle mixin 添加到组件中，不过我们推荐使用 React.createClass 来创建组件，初始化路由的生命周期钩子函数。

### 服务端渲染

服务端渲染与客户端渲染有些许不同，因为你需要：

* 发生错误时发送一个 500 的响应
* 需要重定向时发送一个 30x 的响应
* 在渲染之前获得数据 (用 router 帮你完成这点)

为了迎合这一需求，你要在 `<Router> API` 下一层使用：

* 使用 `match` 在渲染之前根据 location 匹配 route
* 使用 RoutingContext 同步渲染 route 组件

### 组件生命周期

路由组件的生命周期和 React 组件相比并没有什么不同。

```
<Route path="/" component={App}>
  <IndexRoute component={Home}/>
  <Route path="invoices/:invoiceId" component={Invoice}/>
  <Route path="accounts/:accountId" component={Account}/>
</Route>
```

#### 路由切换时，组件生命周期的变化情况

##### 1. 当用户打开应用的 '/' 页面

```
组件	  生命周期
App	    componentDidMount
Home	componentDidMount
Invoice	N/A
Account	N/A
```

##### 2. 当用户从 '/' 跳转到 '/invoice/123'

```
组件	    生命周期
App	    componentWillReceiveProps, componentDidUpdate
Home	componentWillUnmount
Invoice	componentDidMount
Account	N/A
```

##### 3. 当用户从 /invoice/123 跳转到 /invoice/789

```
组件	    生命周期
App	    componentWillReceiveProps, componentDidUpdate
Home	N/A
Invoice	componentWillReceiveProps, componentDidUpdate
Account	N/A
```

##### 4. 当从 /invoice/789 跳转到 /accounts/123

```
组件	    生命周期
App	    componentWillReceiveProps, componentDidUpdate
Home	N/A
Invoice	componentWillUnmount
Account	componentDidMount
```

### 在组件外部使用导航

虽然在组件内部可以使用 `this.context.router` 来实现导航，但许多应用想要在组件外部使用导航。使用Router组件上被赋予的history可以在组件外部实现导航。























