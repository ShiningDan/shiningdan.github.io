---
title: 博客性能优化之React版静态资源优化
date: 2017-06-16 14:19:56
tags:
  - React
  - 性能优化
  - CSS
  - JavaScript
---

本博客使用 React 重新构建了一遍，最基础的部分是导出了一个 JS 文件，然后再前端页面上引用。但是本博客，作为一个很小的前端项目，打包后的 JS 文件都达到了 1.4M，相比于后端渲染的架构，传输的数据过于庞大，这样很不利于首屏渲染。在本文中，介绍了我对于使用 React 来进行按需加载等静态资源的一些探索。

<!--more-->

在优化之前，我们可以看一下，作为一个博客的项目，目前使用 Webpack 打包以后的数据量大小：

我们在 Chrome Network 里面设置网络的情况为：Good3G(40mx, 1.5Mb/s, 750kb/s)，为什么不设置其他的的网络环境呢？因为实在是太慢了！

![](http://ojt6zsxg2.bkt.clouddn.com/d615ce7ebd7b8ec1271b9b496bf7fbb2.png)

在这种网络环境下，我们可以看到，加载 `bundle.js` 使用了 7.66s 的时间，而触发 `DOMContentLoaded` 时间一共使用了 7.90s，触发 `Load` 事件使用了 8.40s，如果我是用户，估计已经切换页面了吧 :(

下面，我们来着手进行项目的按需打包的实现

## 打包公共库

我统计了一下，在 React 博客项目中，涉及到的公共库有：`react`、`react-router-dom`、`moment`、`axios`、`whatwg-fetch`，所以我第一步，要先配置 Webpack，将这些公共的库分别打包出来：

我们要做的，是在 `webpack.config.js` 文件中，设置 `entry`，将公共的库打包到 `vendor.js` 下面，并且使用 `CommonsChunkPlugin` 将相关的部分都从 `bundle.js` 中提取出来。

把公共库提取出来了以后，我们就可以把其中的内容存到 LocalStorage 里面啦，下一次用户就不需要再次获得该部分的内容了。

```
entry: {
    bundle: path.resolve(__dirname, 'app/web/entry.jsx'),
    vendor: ['react', 'react-router-dom', 'moment', 'axios', 'whatwg-fetch'],
},
plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      names: ['vendor'],
      filename: 'vendor.js',
      minChucks: Infinity,
    })
],
```

最后在 HTML 页面上导入 `vendor.js` 文件：

```
<body>
  <script src="/js/vendor.js"></script>
  <script src="/js/bundle.js"></script>  
</body>
```

我们在再次加载页面的时候，可以看到一个大的文件变成了两个小的文件了。虽然文件的大小还有加载的时间没有改变，但是我们可以对这部分文件进行缓存等方法，这部分我们在后面的章节中进行处理。

![](http://ojt6zsxg2.bkt.clouddn.com/dd59fdb0822ae625101bf72e3524329d.png)

## 页面按需加载

目前我们的 `bundle.js` 中打包了除了首页以外的不同页面，比如归档、专题、查询等，既然这些页面不会在首屏中出现，我们就不应该在首屏渲染之前加载该页面的代码，这样会延迟首屏渲染的时间。虽然在本博客项目中，涉及到的其他页面很简单，不会带来特别多的代码加载，但是，作为一个合格的网站，我们还是需要具有考虑这些优化方面的思维。所以，在下面的步骤中，我们将针对 React-Router v4 版本，进行页面按需加载的实现。

在 v4 版本中，官方使用了一种新的方式来实现页面按需加载。我们首先要安装 [bundle-loader](https://github.com/webpack-contrib/bundle-loader) 来实现按需加载的能力：

```
npm install --save-dev bundle-loader
```

然后，再创建一个 `Bundle` 组件，专门用来封装需要按需加载的组件：

```
import React from 'react';

export default class Bundle extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      mod: null,
    }
  }

  componentWillMount() {
    this.load(this.props);
  }

  componentWillReceoveProps(nextProps) {
    if (nextProps.load !== this.props.load) {
      this.load(nextProps);
    }
  }

  load(props) {
    // 重置状态
    this.setState({
      mod: null
    });
    props.load((mod) => {
      this.setState({
        mod: mod.default ? mod.default : mod,
      })
    })
  }

  render() {
    return this.state.mod ? this.props.children(this.state.mod) : null;
  }
}
```

在封装完该组件后，我们就可以对原来的首屏加载的 React-Router 相关的代码进行重写了。

在原来的代码中，我们使用以下的方法来实现 React-Router 的跳转：

```
<div id="content">
    <Switch>
      <Route exact path='/' component={Home} />
      <Route path='/archives' component={Archives}/>
      <Route path='/series' component={Series}/>
      <Route path='/search' component={Search}/> 
      <Route path='/post/:link' component={Article}/> 
      <Route component={Error} />              
    </Switch>
</div>
```

现在，我们将使用 `Bundle` 来重新封装需要按需加载的组件，如 `Archives、Series、Search、Article`：

首先，`import` 组件的时候，我们要使用 `bundle-loader` 来处理：

```
import Archives from 'bundle-loader?lazy&name=[name]!../archives/archives.jsx';
```

然后，再定义一个新的组件 `ArchivesLazy` 来包裹原来的 `Archives` 组件：

```
const ArchivesLazy = (props) => {
  return (
    <Bundle load={Archives}>
      {/*//这里只是给this.props.child传一个方法，最后在Bundle的render里面调用*/}
      {(Container) => <Container />}
    </Bundle>
  );
} 
```

最后，我们把包裹好的组件来替换原来组件的位置：

```
<div id="content">
  <Switch>
    <Route exact path='/' component={Home} />
    <Route path='/archives' component={ArchivesLazy}/>
    <Route path='/series' component={SeriesLazy}/>
    <Route path='/search' component={SearchLazy}/> 
    <Route path='/post/:link' component={ArticleLazy}/> 
    <Route component={ErrorLazy} />              
  </Switch>
</div>
```

并且，修改 `webpack.config.js`，来将这几部分的组件代码提取出来：

```
output: {
    path: path.resolve(__dirname, 'www/static/js'),
    filename: '[name].js',
    // 新添加的
    publicPath: 'js/',
    chunkFilename: '[name].[chunkhash:5].chunk.js',
  },
```

这样，在我们使用 Webpack 编译的时候，就可以得到只包含该组件的文件，如 `archives.00739.chunk.js`。这个文件在跳转到该目录下的时候会被自动下载，下面我们来看一下优化的效果：

在我们使用异步加载之前，可以看到，我们需要加载 `vendor.js` 中代码的大小为 `671KB`：

![](http://ojt6zsxg2.bkt.clouddn.com/b94d2722a0254480c77211d5525cb698.png)

在我们使用异步加载之后，可以看到，首屏下载的 `vendor.js` 中代码的大小为 `628KB` ：

![](http://ojt6zsxg2.bkt.clouddn.com/ff6c49b2c887a9620ea979beec19be46.png)

然后，当我们跳转到其他的页面上时，才会去获取并加载其他页面：

![](http://ojt6zsxg2.bkt.clouddn.com/249a99c03d8b5de21076755a8f63cbd1.png)

`bundle-loader` 是如何实现页面异步加载的呢？我们可以查看一下它的源代码。

它输出的源码就如下所示：

```
/*
Output format:

	var cbs = [],
		data;
	module.exports = function(cb) {
		if(cbs) cbs.push(cb);
		else cb(data);
	}
	require.ensure([], function(require) {
		data = require("xxx");
		var callbacks = cbs;
		cbs = null;
		for(var i = 0, l = callbacks.length; i < l; i++) {
			callbacks[i](data);
		}
	});

*/
```

再配合 `Bundle` 组件，我们来分析它的处理流程：

```
import React from 'react';

export default class Bundle extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      mod: null,
    }
  }

  componentWillMount() {
    this.load(this.props);
  }

  componentWillReceoveProps(nextProps) {
    if (nextProps.load !== this.props.load) {
      this.load(nextProps);
    }
  }

  load(props) {
    // 重置状态
    this.setState({
      mod: null
    });
    props.load((mod) => {
      this.setState({
        mod: mod.default ? mod.default : mod,
      })
    })
  }

  render() {
    return this.state.mod ? this.props.children(this.state.mod) : null;
  }
}
```

当我们在页面加载 `Bundle` 组件的时候，如下所示：

```
const ArticleLazy = (props) => {
  return (
    <Bundle load={Article}>
      {(Container) => <Container {...props}/>}
    </Bundle>
  );
}
```

我们首先会调用 `Bundle` 组件的 `componentWillMount` 方法，这个方法会调用 `load` 方法。当传入的参数为 `Article` 的时候，`load` 函数会调用

```
this.setState({
    mod: mod.default ? mod.default : mod,
  })
```

那 `mod` 又是什么呢？我们打印 `mod`，可以看到它其实是 `Article` 的构造函数：

![](http://ojt6zsxg2.bkt.clouddn.com/275990c495ea26e93d5ed6db251d6248.png)

所以，我们在 `componentWillMount` 中做的事情，就是把对应组件的构造函数传给 `state.mod`。

接下来，React 的组件声明周期运行到 `render` 方法，我们可以看到 `render` 方法中：

```
render() {
    return this.state.mod ? this.props.children(this.state.mod) : null;
  }
```

做的事情，就是使用 `this.props.children(this.state.mod)`，也就是使用 `this.props.children` 来处理 `Article` 构造函数。那 `this.props.children` 是什么呢？

返回 `ArticleLazy` 的定义，我们可以在 `Bundle` 组件中找到 `this.props.children` 的内容，就是：

```
(Container) => <Container {...props}/>
```

其实也就是把 `Article` 构造函数作为组件添加到页面中，就完成了对组件的按需加载了。

## LocalStorage 缓存

为了节省用户重复访问的时候，多次加载 JS 文件，我们可以将网页涉及到的 JS 文件，如 `vendor.js` 和 `bundle.js` 预先缓存到浏览器的 LocalStorage 中，并且通过 Cookie 标识 LocalStorage 中存储的文件的版本。如果文件缓存版本和服务器的版本相同，则脚本文件就从本地进行加载。

首先，我们定义将文件保存在 LocalStorage 的函数 `ls` 和从 LocalStorage 读取文件的函数 `ll`。

```
function ls(name, tag) {
  let target = document.getElementById(name);
  if (!target) {
    throw new Error('ls save fn not find ' + name);
  } else {
    if (window.localStorage) {
      try {
        window.localStorage.setItem(name, target.innerHTML);
        document.cookie = name + '=' +tag;
      } catch(e) {
        console.log(e);
      }
    }
  }
}
function ll(name, isScript) {
  let storage = window.localStorage.getItem(name);
  if (!storage) {
    // 如果 cookie 中存在，但是在 localstorage 中找不到需要如何处理？先删除 cookie，然后刷新页面。
    document.cookie = 'v=; expires=Thu, 01 Jan 1970 00:00:01 GMT;';
    // window.location.reload();
    throw new Error('ls load fn not find ' + name);
  } else {
    let type = isScript ? 'script' : 'style';  // 设置 0 来添加 style，设置 1 来添加 script
    let elem = document.createElement(type);
    elem.innerHTML = storage;
    document.head.appendChild(elem);
  }
}
```

并且在 HTML 加载的时候将该函数先执行。

然后在后台设置逻辑，判断用户请求附带的 Cookie 是否表示该用户已经缓存的资源，并且要验证该资源是否为最新的资源。这一部分的工作可以作为 middleware 来执行：

```
exports.checkll = function(req, res, next) {
  if (req.cookies.vendor && req.cookies.vendor === vendorTag) {
    req.vendor = true;
  } else {
    req.vendor = false;
    req.vendorTag = vendorTag;
  }
  if (req.cookies.bundle && req.cookies.bundle === bundleTag) {
    req.bundle = true;
  } else {
    req.bundle = false;
    req.bundleTag = bundleTag;
  }
  next();
}
```

最后，将 req 参数传给 Pug 模板，通过在模板中判断中间件验证的结果，来决定传输附带 JS 文件还是不附带 JS 文件的 HTML：

```
  body
    #entry
    - if (vendor)
      script ll('vendor', 1)
    - else
      script#vendor
        include ../../www/static/js/vendor.js
      script ls('vendor', '#{vendorTag}')
    - if (bundle)
      script ll('bundle', 1)
    - else
      script#bundle
        include ../../www/static/js/bundle.js
      script ls('bundle', '#{bundleTag}')
```

当我们首次请求的时候：

![](http://ojt6zsxg2.bkt.clouddn.com/f75f2fd5afaac1f556cabc74ee56ec33.png)

可以看到请求的文件大小是 1.4MB，然后返回的 DOM 内容如下：

![](http://ojt6zsxg2.bkt.clouddn.com/bad36806c739feb7f67385fdffb51678.png)

返回的是附带脚本以及执行 `ls` 函数的 DOM，表示下载执行脚本，并且把该脚本存储在 LocalStorage 里面。

当再次访问的时候；

![](http://ojt6zsxg2.bkt.clouddn.com/f5c761bf9456a8df5da25f5e49da2c97.png)

我们可以看到，请求文件的大小变得非常小，因为此次请求没有下载脚本文件：

![](http://ojt6zsxg2.bkt.clouddn.com/2d90ead132bca7d4f0d60277c4ab9c18.png)

取而代之的是执行 `ll` 函数，从 LocalStorage 中获取代码，并且添加到 DOM 树中执行。

## Webpack 代码压缩

虽然我们已经把代码放在了 LocalStorage 里面，但其实代码量还是很大的，所以，我们要对 Webpack 打包的代码进行压缩。

webpack 自带了一个压缩插件 UglifyJsPlugin，只需要在配置文件中引入即可。

```
{
  plugins: [
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      }
    })
  ]
}
```

加入了这个插件之后，编译的速度会明显变慢，所以一般只在生产环境启用。

在使用了这个插件以后，我们可以看到，首次加载网页的时候，代码的大小从原来的 1.4MB 减小到了现在的 540KB ，效果非常好。

![](http://ojt6zsxg2.bkt.clouddn.com/d03a8b4fc714cd8b05a381439f163256.png)

