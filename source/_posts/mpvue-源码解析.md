---
title: mpvue 源码解析
date: 2018-11-17 14:44:47
categories: coding
tags:
  - 小程序
  - mpvue
  - vue
---

本文是学习 mpvue 源码的总结

<!--more-->

## 使用 mpvue 模板

mpvue 有自己的  vue-cli 模板，基于 vue-cli@2.* 版本，具体初始化过程见 [mpvue-docs | quickstart](http://mpvue.com/mpvue/quickstart/)

```
vue init mpvue/mpvue-quickstart my-mpvue-project
```

下面，从 yarn run dev 开始进行分析

## yarn run dev

`yarn run dev` 运行的脚本是：`node build/dev-server.js wx`

### dev-server.js 整体流程

`dev-server.js` 中的 `config` 是从 `../config/index.js` 中引入的：

```
{ build:
   { env: { NODE_ENV: '"production"' },
     index: '/Users/zhangyuchen/Documents/project/my-mpvue-project/dist/wx/index.html',
     assetsRoot: '/Users/zhangyuchen/Documents/project/my-mpvue-project/dist/wx',
     assetsSubDirectory: '',
     assetsPublicPath: '/',
     productionSourceMap: false,
     productionGzip: false,
     productionGzipExtensions: [ 'js', 'css' ],
     bundleAnalyzerReport: undefined,
     fileExt: { template: 'wxml', script: 'js', style: 'wxss', platform: 'wx' } },
  dev:
   { env: { NODE_ENV: '"development"' },
     port: 8080,
     autoOpenBrowser: false,
     assetsSubDirectory: '',
     assetsPublicPath: '/',
     proxyTable: {},
     cssSourceMap: false,
     fileExt: { template: 'wxml', script: 'js', style: 'wxss', platform: 'wx' } } }
```

Server 使用的是 express，使用 webpack 来进行构建，使用的 webpack 版本为：3.12.0，webpackConfig 来自于 `webpack.dev.conf.js`

首先，在 express 中使用 `http-proxy-middleware` 实现一些用户自定义的代理配置。

在 express 中使用 `connect-history-api-fallback` 来支持。有什么作用？？？

在对于路径进行处理的时候，使用的是 `path.posix.join` 这个方法，是为了在 Windows 操作系统上也支持类 Unix 类型的路径（POSIX 格式）

最后开启一个 Server，Server 首先找到可用的端口，然后启动 `webpack-dev-middleware-hard-disk`，这个 `webpack-dev-middleware-hard-disk`，是在 `webpack-dev-middleware` 基础上进行开发的。`webpack-dev-middleware` 这个库，使用在 development 环境下，可以监听文件的变化，然后自动执行编译函数，但是 `webpack-dev-middleware` 不会将编译后的输出写到硬盘上，而 `webpack-dev-middleware-hard-disk` 就是在其监听以及构建能力的基础上，将构建的结果输出到 `webpackConfig.output.path 下面`

### dev-server.js webpackConfig 生成

```
{ entry:
   { app: '/Users/zhangyuchen/Documents/project/my-mpvue-project/src/main.js',
     'pages/counter/main': '/Users/zhangyuchen/Documents/project/my-mpvue-project/src/pages/counter/main.js',
     'pages/index/main': '/Users/zhangyuchen/Documents/project/my-mpvue-project/src/pages/index/main.js',
     'pages/logs/main': '/Users/zhangyuchen/Documents/project/my-mpvue-project/src/pages/logs/main.js' },
  target: [Function],
  output:
   { path: '/Users/zhangyuchen/Documents/project/my-mpvue-project/dist/wx',
     filename: '[name].js',
     publicPath: '/',
     chunkFilename: '[id].js' },
  resolve:
   { extensions: [ '.js', '.vue', '.json' ],
     alias:
      { vue: 'mpvue',
        '@': '/Users/zhangyuchen/Documents/project/my-mpvue-project/src' },
     symlinks: false,
     aliasFields: [ 'mpvue', 'weapp', 'browser' ],
     mainFields: [ 'browser', 'module', 'main' ] },
  module:
   { rules:
      [ [Object],
        [Object],
        [Object],
        [Object],
        [Object],
        [Object],
        [Object],
        [Object],
        [Object],
        [Object],
        [Object], 
        [Object],
        [Object],
        [Object] ] },
  plugins:
   [ MpvuePlugin {},
     { apply: [Function: apply] },
     { apply: [Function: apply] },
     DefinePlugin { definitions: [Object] },
     ExtractTextPlugin { filename: '[name].wxss', id: 1, options: {} },
     OptimizeCssAssetsPlugin { options: [Object], lastCallInstance: [Object] },
     CommonsChunkPlugin {
       chunkNames: [Array],
       filenameTemplate: undefined,
       minChunks: [Function: minChunks],
       selectedChunks: undefined,
       children: undefined,
       deepChildren: undefined,
       async: undefined,
       minSize: undefined,
       ident: '/Users/zhangyuchen/Documents/project/my-mpvue-project/node_modules/webpack/lib/optimize/CommonsChunkPlugin.js0' },
     CommonsChunkPlugin {
       chunkNames: [Array],
       filenameTemplate: undefined,
       minChunks: undefined,
       selectedChunks: [Array],
       children: undefined,
       deepChildren: undefined,
       async: undefined,
       minSize: undefined,
       ident: '/Users/zhangyuchen/Documents/project/my-mpvue-project/node_modules/webpack/lib/optimize/CommonsChunkPlugin.js1' },
     NoEmitOnErrorsPlugin {},
     FriendlyErrorsWebpackPlugin {
       compilationSuccessInfo: {},
       onErrors: undefined,
       shouldClearConsole: true,
       formatters: [Array],
       transformers: [Array] } ],
  devtool: '#source-map' }
```

下面是几个需要关注的点：

### Entry 

entry 里面分了好几块：

```
{
	app: '/Users/zhangyuchen/Documents/project/my-mpvue-project/src/main.js',
	'pages/counter/main': '/Users/zhangyuchen/Documents/project/my-mpvue-project/src/pages/counter/main.js',
	'pages/index/main': '/Users/zhangyuchen/Documents/project/my-mpvue-project/src/pages/index/main.js',
	'pages/logs/main': '/Users/zhangyuchen/Documents/project/my-mpvue-project/src/pages/logs/main.js' 
}
```

app 是 `src/main.js`，然后 pages 是 `src/pages/**/main.js`。

每一个 entry 里面的内容都是一样的：

```
import Vue from 'vue'
import App from './index'

const app = new Vue(App)
app.$mount()
```

说明每一个页面都被构建成单独的页面。也就侧面说明，小程序中的 pages 基本上都是独立的页面，而不是单页应用的形式。

### target

`target: require('mpvue-webpack-target')`

```
// mpvue webpack-target
const JsonpTemplatePlugin = require("./JsonpTemplatePlugin");
const NodeSourcePlugin = require("webpack/lib/node/NodeSourcePlugin");
const FunctionModulePlugin = require("webpack/lib/FunctionModulePlugin");
const LoaderTargetPlugin = require("webpack/lib/LoaderTargetPlugin");

module.exports = function (compiler) {
  const { options } = compiler
  compiler.apply(
    new JsonpTemplatePlugin(options.output),
    new FunctionModulePlugin(options.output),
    new NodeSourcePlugin(options.node),
    new LoaderTargetPlugin(options.target)
  );
};
```

具体内容是什么呢，暂时不仔细分析，

### output

```
{ path: '/Users/zhangyuchen/Documents/project/my-mpvue-project/dist/wx',
 filename: '[name].js',
 // 所有资源指定一个基础路径
 publicPath: '/',
 // 决定了非入口(non-entry) chunk 文件的名称。从 output.filename 中推断出的值（[name] 会被预先替换为 [id] 或 [id].）
 chunkFilename: '[id].js' },
```

### resolve

```
{ extensions: [ '.js', '.vue', '.json' ],
     alias:
     // 将 vue 替换为 mpvue
      { vue: 'mpvue',
        '@': '/Users/zhangyuchen/Documents/project/my-mpvue-project/src' },
     symlinks: false,
     aliasFields: [ 'mpvue', 'weapp', 'browser' ],
     // 指定在 package.json 中使用哪个字段导入模块
     mainFields: [ 'browser', 'module', 'main' ] },
```

### module.rules.mpvue-loader

主要是在下面的这个 rule 中指定解析 loader：

```
{
	test: /\.vue$/,
	loader: 'mpvue-loader',
	options: vueLoaderConfig
},
{
	test: /\.js$/,
	include: [resolve('src'), resolve('test')],
	use: [
	  'babel-loader',
	  {
	    loader: 'mpvue-loader',
	    options: Object.assign({checkMPEntry: true}, vueLoaderConfig)
	  },
	]
},
```

其中 vueLoaderConfig 的内容为：

```
{ loaders:
   { css: [ [Object], [Object], [Object], [Object], [Object] ],
     wxss: [ [Object], [Object], [Object], [Object], [Object] ],
     postcss: [ [Object], [Object], [Object], [Object], [Object] ],
     less: [ [Object], [Object], [Object], [Object], [Object], [Object] ],
     // 举例
     sass:
	   [ { loader: '/Users/zhangyuchen/Documents/project/my-mpvue-project/node_modules/extract-text-webpack-plugin/dist/loader.js',
	       options: [Object] },
	     { loader: 'vue-style-loader' },
	     { loader: 'css-loader', options: [Object] },
	     { loader: 'px2rpx-loader', options: [Object] },
	     { loader: 'postcss-loader', options: [Object] },
	     { loader: 'sass-loader', options: [Object] } ],
     scss: [ [Object], [Object], [Object], [Object], [Object], [Object] ],
     stylus: [ [Object], [Object], [Object], [Object], [Object], [Object] ],
     styl: [ [Object], [Object], [Object], [Object], [Object], [Object] ] },
  transformToRequire: { video: 'src', source: 'src', img: 'src', image: 'xlink:href' },
  fileExt: { template: 'wxml', script: 'js', style: 'wxss', platform: 'wx' } 
}
```

在介绍完 plugins 之后，我们再对 mpvue-loader 中的源码进行介绍

### plugins

```
plugins: [
	 // webpack-mpvue-asset-plugin：mpvue 资源路径解析插件，主要是将 js 文件和 wxss 文件通过 require 和 @import 整合成同一个文件
    new MpvuePlugin(),
    new CopyWebpackPlugin([{
      from: '**/*.json',
      to: ''
    }], {
      context: 'src/'
    }),
    new CopyWebpackPlugin([
      {
        from: path.resolve(__dirname, '../static'),
        to: path.resolve(config.build.assetsRoot, './static'),
        ignore: ['.*']
      }
    ])
    ,
    new webpack.DefinePlugin({
      'process.env': config.dev.env
    }),

    // copy from ./webpack.prod.conf.js
    // extract css into its own file
    new ExtractTextPlugin({
      // filename: utils.assetsPath('[name].[contenthash].css')
      filename: utils.assetsPath(`[name].${config.dev.fileExt.style}`)
    }),
    // Compress extracted CSS. We are using this plugin so that possible
    // duplicated CSS from different components can be deduped.
    new OptimizeCSSPlugin({
      cssProcessorOptions: {
        safe: true
      }
    }),
    new webpack.optimize.CommonsChunkPlugin({
      name: 'common/vendor',
      minChunks: function (module, count) {
        // any required modules inside node_modules are extracted to vendor
        return (
          module.resource &&
          /\.js$/.test(module.resource) &&
          module.resource.indexOf('node_modules') >= 0
        ) || count > 1
      }
    }),
    new webpack.optimize.CommonsChunkPlugin({
      name: 'common/manifest',
      chunks: ['common/vendor']
    }),

    // https://github.com/glenjamin/webpack-hot-middleware#installation--usage
    // new webpack.HotModuleReplacementPlugin(),
    new webpack.NoEmitOnErrorsPlugin(),
    // https://github.com/ampedandwired/html-webpack-plugin
    // new HtmlWebpackPlugin({
    //   filename: 'index.html',
    //   template: 'index.html',
    //   inject: true
    // }),
    new FriendlyErrorsPlugin()
  ]
```

## mpvue runtime

`mpvue` 这个库里面是 mpvue runtime 部分的代码，代码仓库在：[Meituan-Dianping/mpvue | Github](https://github.com/Meituan-Dianping/mpvue)

mpvue 的源码入口文件在 `/src/platforms/mpvue/entry-runtime.js` 下面，主要做的是运行时将小程序的声明周期以及数据绑定映射到 Vue 本身的机制中。除了 runtime 以外，在 `/src/platforms/mpvue/entry-compiler.js` 中是编译的入口文件，主要做的是把 Vue 的代码在构建阶段编译成小程序使可以识别的代码。

### entry-runtime.js

在 `entry-runtime.js` 中，首先添加一些平台相关的方法

```
Vue.config.mustUseProp = mustUseProp   // null，指定一个标签必须配套什么属性
Vue.config.isReservedTag = isReservedTag   // 是不是平台已有的组件，但是现在 isReservedTag 的标签比较奇怪，需要再看看？？？
Vue.config.isReservedAttr = isReservedAttr   // style 和 class 是内建属性，但是实际使用的是 isReservedAttribute，内容是 'key,ref,slot,is'
Vue.config.getTagNamespace = getTagNamespace   // null
Vue.config.isUnknownElement = isUnknownElement   // null
```

```
// 更新数据，里面的方法，将新旧vnode使用 diff算法进行比对，找出要替换的地方，这样更新dom的性能会有较大优化。最后会返回一个dom节点。
Vue.prototype.__patch__ = patch
```

```
// 挂载节点，将数据以及 vnode 对象挂载到真实的 DOM 上
Vue.prototype.$mount = function() {
	return this._initMP(mpType, () => {
      return mountComponent(this, undefined, undefined)
    })
}

import { initMP } from './lifecycle'
Vue.prototype._initMP = initMP
```

```
// render 时使用的更新数据的方法，一般为 render => (vm._render to get/set data)=> vnode => (vm._update) => vm.$el
import { updateDataToMP, initDataToMP } from './render'
Vue.prototype.$updateDataToMP = updateDataToMP
Vue.prototype._initDataToMP = initDataToMP
```

```
// 将小程序的事件用 vue 的事件来进行处理
import { handleProxyWithVue } from './events'
Vue.prototype.$handleProxyWithVue = handleProxyWithVue
```

### this._initMP

在挂载节点的时候，会使用 `this._initMP` 这个方法进行一层包装，我们来看一下这个方法中的内容。

![](http://image.zyuchen.com/592dff1d1b62e1ce20d2f5827729970c.png)

目前看起来，是在小程序 APP 启动的时候，会进行一次 init 方法。小程序启动后，会对所有的页面都进行加载，现在所有的页面都会运行一次 init 方法。

`_initMP` 方法的内容是什么呢？很多，分步来看：

```
// 在 Vue 的实例基础上，添加 $mp 属性，用来存放小程序相关的方法
const rootVueVM = this.$root
const mp = this.$root.$mp
```

**`this.$mp` 是在哪里初始化的？？？？**

直接编译默认是 page，识别到启始页就是 app，component 是自定义的。

首先，当 `mpType === 'app'` 的时候，也就是注册 `app` 的时候：

```
global.App({
  // 页面的初始数据
  globalData: {
    //
    appOptions: {}
  },
  // 将小程序的事件交给 Vue 进行处理
  handleProxy (e) {
    return rootVueVM.$handleProxyWithVue(e)
  },

  // Do something initial when launch.
  onLaunch (options = {}) {
    mp.app = this
    mp.status = 'launch'
    // 接收 onLaunch 传入的参数，放在 app.globalData.appOptions 中
    this.globalData.appOptions = mp.appOptions = options
    // 如果用户在 Vue 文件中定义了 onLaunch 方法，也就是需要使用小程序的生命周期做一些处理，在这里会调用 onLaunch 方法
    callHook(rootVueVM, 'onLaunch', options)
    next()
  },

  // Do something when app show.
  onShow (options = {}) {
    mp.status = 'show'
    this.globalData.appOptions = mp.appOptions = options
    callHook(rootVueVM, 'onShow', options)
  },

  // Do something when app hide.
  onHide () {
    mp.status = 'hide'
    callHook(rootVueVM, 'onHide')
  },

  onError (err) {
    callHook(rootVueVM, 'onError', err)
  }
})
```

小程序的声明周期中，`global` 中包含了五个方法，也就是小程序的五个初始化函数：

```
global: {
	App,
	Component,
	Page,
	getApp,
	process: {
		env: {...}
	}
}
```

```
// 小程序的生命周期中调用 Vue 模板中写的事件
export function callHook (vm, hook, params) {
  // 如果是需要使用小程序的生命周期事件，则在 Vue 代码中写对应的小程序生命周期函数，该函数会被放到 vm.$option 中，所以通过 vm.$option 可以拿到小程序的生命周期事件代码
  let handlers = vm.$options[hook]
  if (hook === 'onError' && handlers) {
    handlers = [handlers]
  }

  let ret
  if (handlers) {
    for (let i = 0, j = handlers.length; i < j; i++) {
      try {
        ret = handlers[i].call(vm, params)
      } catch (e) {
        handleError(e, vm, `${hook} hook`)
      }
    }
  }
  if (vm._hasHookEvent) {
    vm.$emit('hook:' + hook)
  }

  // for child
  if (vm.$children.length) {
    vm.$children.forEach(v => callHook(v, hook, params))
  }

  return ret
}
```

当 `mpType === 'page'` 的时候

```
const app = global.getApp()
global.Page({
  // 页面的初始数据
  data: {
    $root: {}
  },

  handleProxy (e) {
    return rootVueVM.$handleProxyWithVue(e)
  },

  // mp lifecycle for vue
  // 生命周期函数--监听页面加载
  onLoad (query) {
    mp.page = this
    mp.query = query
    mp.status = 'load'
    // 将 app.globalData.appOptions 的值挂载到 this.$root.$mp.appOptions 中。appOptions 中的值是初始化 App 的时候的 onShow() 的函数参数
    getGlobalData(app, rootVueVM)
    // 如果用户在 vue 文件中自定义了 onLoad，则调用用户的 onLoad 方法
    callHook(rootVueVM, 'onLoad', query)
  },

  // 生命周期函数--监听页面显示
  onShow () {
    mp.page = this
    mp.status = 'show'
    callHook(rootVueVM, 'onShow')

    // 只有页面需要 setData
    rootVueVM.$nextTick(() => {
      rootVueVM._initDataToMP()
    })
  },

  // 生命周期函数--监听页面初次渲染完成，onReady 的时候，运行 Vue 的 mounted，说明 onReady 和 mounted 是等价的生命周期
  onReady () {
    mp.status = 'ready'

    callHook(rootVueVM, 'onReady')
    // () => {
    //   return mountComponent(this, undefined, undefined)
    // })
    // next 就是 mountComponent，也就是在 onReady 的时候，将 Vue 的实例挂载到 Dom 元素上，虽然这里是小程序的运行环境，没有 DOM 元素，mountComponent 的第二个参数是 undifined。但是也可以通过这个方法去触发 Vue mounted 以及之后的声明周期
    next()
  },

  // 生命周期函数--监听页面隐藏
  onHide () {
    mp.status = 'hide'
    callHook(rootVueVM, 'onHide')
    mp.page = null
  },

  // 生命周期函数--监听页面卸载
  onUnload () {
    mp.status = 'unload'
    callHook(rootVueVM, 'onUnload')
    mp.page = null
  },

  // 页面相关事件处理函数--监听用户下拉动作
  onPullDownRefresh () {
    callHook(rootVueVM, 'onPullDownRefresh')
  },

  // 页面上拉触底事件的处理函数
  onReachBottom () {
    callHook(rootVueVM, 'onReachBottom')
  },

  // 用户点击右上角分享
  onShareAppMessage: rootVueVM.$options.onShareAppMessage
    ? options => callHook(rootVueVM, 'onShareAppMessage', options) : null,

  // Do something when page scroll
  onPageScroll (options) {
    callHook(rootVueVM, 'onPageScroll', options)
  },

  // 当前是 tab 页时，点击 tab 时触发
  onTabItemTap (options) {
    callHook(rootVueVM, 'onTabItemTap', options)
  }
})
```

每一次页面初始化的时候，都会调用 `rootVueVM._initDataToMP()` 进行数据初始化，下面我们来看一下其中的代码：

```
// rootVueVM._initDataToMP()
// runtime/render.js
export function initDataToMP () {
  // this is vm.$root
  // 获得 this.$mp.page 对象，这个对象只有在 Page 的实例中才有
  const page = getPage(this)
  if (!page) {
    return
  }

  // 使用字符串拼接等方法，将 Vue 中的数据传递给小程序的 data
  // 传递的数据有 vm._data，vm._props，vm._mpProps，vm._computedWatchers
  const data = collectVmData(this.$root)
  page.setData(data)
}
```

每一次的 data 是会发生改变的，所以每一次在 onShow 的时候都需要调用 `initDataToMP`

```
// data 的例子
{
  $root.0: {motto: "Hello World", userInfo: {…}, $k: "0", $kk: "0,", $p: ""},
  $root.0,0: {text: "单人舞", $k: "0,0", $kk: "0,0,", $p: "0"},
  $root.0,1: {text: "Hello World", $k: "0,1", $kk: "0,1,", $p: "0"},
}
```

其中，`data` 中数据的编号是如下定义的：

1. `$root` 后面的标签是从 0 开始表示最外层的根组件或者页面的数据。如果是该页面的子组件，则 `0,0` 中后面的这个 `0` 表示它的第一个子组件
2. `$k` 指的是 `$root` 本身的标识符
3. `$kk` 指的是该组件或页面的子组件的标识符前缀
4. `$p` 指的是该组件的父组件或页面的标识符

### updateDataToMP 更新 Vue 对象的 Data

每一次页面事件，网络事件等对数据进行更新，都是通过 `handleProxy` 对 Vue 的数据进行更新，然后 Vue 的数据进行更新后，再对小程序页面的数据进行更新。更新小程序页面数据的方法，是通过修改 Vue 本身的 patch 方法，在对自身数据更新后，调用 `updateDataToMP` 来更新小程序的数据：

```
// runtime/patch.js
export function patch () {
  corePatch.apply(this, arguments)
  this.$updateDataToMP()
}

// runtime/index.js
Vue.prototype.__patch__ = patch
```

下面来看一下 updateDataToMP 做了什么：

```
// 优化每次 setData 都传递大量新数据
export function updateDataToMP () {
  const page = getPage(this)
  if (!page) {
    return
  }

  const data = formatVmData(this)
  // 过快 setData 会对小程序的性能造成影响
  throttleSetData(page.setData.bind(page), data)
}


const throttleSetData = throttle((handle, data) => {
  handle(data)
}, 50)

function getPage (vm) {
  const rootVueVM = vm.$root
  const { mpType = '', page } = rootVueVM.$mp || {}

  // 优化后台态页面进行 setData: https://mp.weixin.qq.com/debug/wxadoc/dev/framework/performance/tips.html
  if (mpType === 'app' || !page || typeof page.setData !== 'function') {
    return
  }
  return page
}
```

### 声明周期

**Vue 的声明周期**：

* beforeCreate
* created
* beforeMount
* mounted
* beforeUpdate
* updated
* activated
* deactivated
* beforeDestroy
* destroyed

除了 Vue 本身的生命周期外，mpvue 还兼容了小程序生命周期，这部分生命周期钩子的来源于微信小程序的 Page， 除特殊情况外，不建议使用小程序的生命周期钩子。

**app 部分**：

* onLaunch，初始化
* onShow，当小程序启动，或从后台进入前台显示
* onHide，当小程序从前台进入后台

**page 部分**：

* onLoad，监听页面加载
* onShow，监听页面显示
* onReady，监听页面初次渲染完成
* onHide，监听页面隐藏
* onUnload，监听页面卸载
* onPullDownRefresh，监听用户下拉动作
* onReachBottom，页面上拉触底事件的处理函数
* onShareAppMessage，用户点击右上角分享
* onPageScroll，页面滚动
* onTabItemTap, 当前是 tab 页时，点击 tab 时触发 （mpvue 0.0.16 支持）
* onResize，页面尺寸改变时触发

**Component 部分**，目前 mpvue 不支持生成小程序的 Component，而是使用 template：

* created，在组件实例进入页面节点树时执行，注意此时不能调用 setData
* attached，在组件实例进入页面节点树时执行，不能获得节点信息
* ready，在组件布局完成后执行，此时可以获取节点信息
* moved，在组件实例被移动到节点树另一个位置时执行
* detached，在组件实例被从页面节点树移除时执行


## mpvue compiler

在 runtime 的时候，主要处理的一些函数的绑定，以及生命周期的触发。但是小程序组件上定义的函数如何处理，什么时候对小程序的 data 进行更新，小程序的事件又如何处理呢，这些可以在 compiler 里面查看。

compiler 是被 wepback 在构建的时候调用的，入口文件在 `platforms/mp/compiler/index.js`

```
import { baseOptions } from './options'
import { createCompiler } from './create-compiler'

const { compile, compileToFunctions } = createCompiler(baseOptions)

import { compileToWxml } from './codegen/index'

export { compile, compileToFunctions, compileToWxml }
```

### compileToWxml

首先，先从 `compileToWxml` 开始看起。 `compileToWxml` 这个方法会接收两个参数，第一个参数是 compiled，这个 compiled 指的是从 Vue 的模板代码，被转化成 createElement 这种 JS 包裹的代码

```
// compileToWxml
export function compileToWxml (compiled, options = {}) {
  // TODO, compiled is undefined
  const { components = {}} = options
  const log = utils.log(compiled)

  const { wxast, deps = {}, slots = {}} = wxmlAst(compiled, options, log)
  let code = generate(wxast, options)

  // 引用子模版
  const importCode = Object.keys(deps).map(k => components[k] ? `<import src="${components[k].src}" />` : '').join('')
  code = `${importCode}<template name="${options.name}">${code}</template>`

  // 生成 slots code
  Object.keys(slots).forEach(k => {
    const slot = slots[k]
    slot.code = generate(slot.node, options)
  })

  // TODO: 后期优化掉这种暴力全部 import，虽然对性能没啥大影响
  return { code, compiled, slots, importCode }
}
```

我们先从最简单的例子看起：

```
// logs.vue
<template>
  <div>
    <ul class="container log-list">
      <li v-for="(log, index) in logs" :class="{ red: aa }" :key="index" class="log-item">
      </li>
    </ul>
  </div>
</template>

<script>
import { formatTime } from '@/utils/index'

export default {
  data () {
    return {
      logs: []
    }
  },

  created () {
    const logs = (wx.getStorageSync('logs') || [])
    this.logs = logs.map(log => formatTime(new Date(log)))
  }
}
</script>

<style>
.log-list {
  display: flex;
  flex-direction: column;
  padding: 40rpx;
}

.log-item {
  margin: 10rpx;
}
</style>
```

这个例子中，使用的最简单的在 Vue 的模板中绑定 data 中的数据。此时 `compiled` 参数就是经过 vue-template-compiler 处理后的代码：

```
{ ast:
   { type: 1,
     tag: 'div',
     attrsList: [],
     attrsMap: {},
     parent: undefined,
     children: [ [Object] ],
     plain: true,
     static: false,
     staticRoot: false },
  render: 'with(this){return _c('div',[_c('ul',{staticClass:"container log-list"},_l((logs),function(log,index){return _c('li',{key:index,staticClass:"log-item",class:{ red: aa }})}))],1)}',
  staticRenderFns: [],
  errors: [],
  tips: [] }
```

其中 `render` 部分被转换之后的代码，使用 `_c` 是 `createElement`，`_l` 用 `renderList` 进行代替，可以获得 `render` 为 

```
// render
// createElement 接收三个参数，第一个参数是标签名：div 等，第二个参数是 props，第三个参数是子节点，最后通过 createElement 生成一个 vnode 对象
with (this) {
  return createElement(
    'div',
    [createElement(
      'ul',
      { staticClass: "container log-list" },
      renderList(
        (logs),
        function (log, index) {
          return createElement(
            'li',
            {
              key: index,
              staticClass: "log-item",
              class: {
                red: aa
              }
            }
          )
        }
      ))
    ],
    1)
}
```

下面，我们一点一点来看代码，首先：

```
// log 方法，为 compiled 对象添加 mpErrors = [] 以及 mpTips = []，然后返回一个函数，向这两个数组中填充值
const log = utils.log(compiled)

// log 的内容，在 mpvue-loader 中调用了 mpTips 以及 mpErrors，将构建时的错误丢给 webpack Errors
return (str, type) => {
  if (type === 'waring') {
    compiled.mpTips.push(str)
  } else {
    compiled.mpErrors.push(str)
  }
}
```

#### wxmlAst，将 Vue 解析结果转换成小程序语法树

```
// wxmlAst 做的事情，就是将 Vue 编译出来的生成 vnode 的代码，转换成小程序的的语法树
const { wxast, deps = {}, slots = {}} = wxmlAst(compiled, options, log)

// wxast
{ type: 1,
  tag: 'view',
  attrsList: [],
  attrsMap: { class: '_div data-v-319c166a' },
  parent: undefined,
  children:
    [
      { type: 1,
        tag: 'view',
        attrsList: [],
        attrsMap: { class: '_ul data-v-319c166a container log-list' },
        parent:
         { type: 1,
           tag: 'div',
           attrsList: [],
           attrsMap: {},
           parent: undefined,
           children: [ [Object] ],
           plain: true,
           static: false,
           staticRoot: false },
        children:
         [ { type: 1,
             tag: 'view',
             attrsList: [],
             attrsMap: [Object],
             parent: [Object],
             children: [],
             for: 'logs',
             alias: 'log',
             iterator1: 'index',
             key: 'index',
             plain: false,
             staticClass: '_li data-v-319c166a log-item',
             classBinding: '{ red: aa }',
             staticRoot: false,
             forProcessed: true,
             slots: {} } ],
        plain: false,
        staticClass: '_ul data-v-319c166a container log-list',
        static: false,
        staticRoot: false,
        slots: {} }
    ],
  plain: true,
  static: false,
  staticRoot: false,
  staticClass: '_div data-v-319c166a',
  slots: {}
}
// deps 
{}
// slots
{}
```

我们来看一下将 `createElement` 解析成小程序的语法树的 `wxmlAst` 函数的内容：

```
// wxmlAst
export default function wxmlAst (compiled, options = {}, log) {
  const { ast } = compiled
  const deps = {
    // slots: 'slots'
  }
  const slots = {
    // slotId: nodeAst
  }
  const slotTemplates = {
  }

  const wxast = convertAst(ast, options, { log, deps, slots, slotTemplates })
  const children = Object.keys(slotTemplates).map(k => convertAst(slotTemplates[k], options, { log, deps, slots, slotTemplates }))
  wxast.children = children.concat(wxast.children)
  return {
    wxast,
    deps,
    slots
  }
}

// convertAst
function convertAst (node, options = {}, util) {
  const { children, ifConditions, staticClass = '', mpcomid } = node
  let { tag: tagName } = node
  const { log, deps, slots, slotTemplates } = util
  
  // wxmlAst 中首先包含 compiled 的全部内容
  let wxmlAst = Object.assign({}, node)
  const { moduleId, components } = options
  // 将驼峰形式的组件组件或者标签名称转换成连字符形式
  wxmlAst.tag = tagName = tagName ? hyphenate(tagName) : tagName
  // 引入 import, isSlot 是使用 slot 的编译地方，意即 <slot></slot> 的地方
  const isSlot = tagName === 'slot'
  // 对 slot 进行处理
  if (isSlot) {
    deps.slots = 'slots'
    // 把当前 slot 节点包裹 template
    const defSlot = Object.assign({}, wxmlAst)
    defSlot.tag = 'template'
    const templateName = `${defSlot.attrsMap.name || 'default'}`
    defSlot.attrsMap.name = templateName
    wxmlAst.children = []
    defSlot.parent = node.parent.parent
    slotTemplates[templateName] = defSlot
  }

  // 判断当前要处理的 tag 是否是一个组件，如何判断是组件呢？在 components 里面有写明当前 tag 的父元素有哪些组件。下面是一个 components 的例子，components 是 mpvue-loader 传入的 options 中的数据，是 mpvue-loader 的解析结果
  
  // options 中的数据示例
  //  { components:
  //   { card: { src: '/components/card.vue.wxml', name: '64c5b64a' },
  //     paper: { src: '/components/paper.vue.wxml', name: '09070551' },
  //     isCompleted: true,
  //     slots: { src: '/components/slots', name: 'slots' } },
  //  pageType: 'component',
  //  name: 'cccea208',
  //  moduleId: 'data-v-65f59a68' }
  
  const currentIsComponent = component.isComponent(tagName, components)
  // 如果当前元素是父元素的一个组件，在 deps 中记录
  if (currentIsComponent) {
    deps[tagName] = tagName
  }

  if (moduleId && !currentIsComponent && tagConfig.virtualTag.indexOf(tagName) < 0) {
    // 如果是一个普通的小程序元素，并且不是【虚标签】，这里定义 'slot', 'template', 'block' 这三个在小程序页面中没有显式影响 DOM 的标签为虚标签，则将 moduleId 拼接到当前元素的 class 中
    wxmlAst.staticClass = staticClass ? `${moduleId} ${staticClass}`.replace(/\"/g, '') : moduleId
  } else {
    // 如果是一个组件，则在这个组件的 class 里面拼接 moduleId
    wxmlAst.staticClass = staticClass.replace(/\"/g, '')
  }

  wxmlAst.slots = {}
  // 如果一个元素是组件，则该元素和该元素的子元素都被丢到 slot.wxml 中
  if (currentIsComponent && children && children.length) {
    // 只检查组件下的子节点（不检查孙子节点）是不是具名 slot，不然就是 default slot
    children
      .reduce((res, n) => {
        const { slot } = n.attrsMap || {}
        // 不是具名的，全部放在第一个数组元素中
        // 关于具名 slot，可以参考 https://cn.vuejs.org/v2/guide/components-slots.html#%E5%85%B7%E5%90%8D%E6%8F%92%E6%A7%BD
        // 也就是使用 slot 属性把节点插到不同的 slot 上，具体可以参考小程序的文档：https://developers.weixin.qq.com/miniprogram/dev/framework/custom-component/wxml-wxss.html?search-key=slot
        const arr = slot ? res : res[0]
        arr.push(n)
        return res
      }, [[]])
      .forEach(n => {
        // 将组件的子元素都放到 slots 里面，然后从这个组件的 template 中删掉所有的子元素。slots 里面的子元素都设置一个 name，以供组件进行调用
        const isDefault = Array.isArray(n)
        const slotName = isDefault ? 'default' : n.attrsMap.slot
        const slotId = `${moduleId}-${slotName}-${mpcomid.replace(/\'/g, '')}`
        const node = isDefault ? { tag: 'slot', attrsMap: {}, children: n } : n

        node.tag = 'template'
        node.attrsMap.name = slotId
        delete node.attrsMap.slot
        // 缓存，会集中生成一个 slots 文件
        slots[slotId] = { node: convertAst(node, options, util), name: slotName, slotId }
        // 将生成的 name 提供给父组件，父组件就可以通过 name 对 slots 中的子元素进行引用
        wxmlAst.slots[slotName] = slotId
      })
    // 清理当前组件下的节点信息，因为 slot 都被转移了
    children.length = 0
    wxmlAst.children.length = 0
  }

  // 为了方便之后的处理，将 @ 转换成 v-on，将 : 转换成 v-bind
  wxmlAst.attrsMap = attrs.format(wxmlAst.attrsMap)
  // 进行 tag 转换
  wxmlAst = tag(wxmlAst, options)
  // 对于 v-for 进行转换，修改为 wx:for，wx:for-item，wx:for-index，wx:key
  wxmlAst = convertFor(wxmlAst, options)
  // 对于属性进行转换，比如 Vue 中定义的事件 v-on，v-bind 等，以及其他的自定义属性
  wxmlAst = attrs.convertAttr(wxmlAst, log)
  if (children && !isSlot) {
    // 对于 children 也进行语法树转换，由于 slot 直接被提取成 template，所以 slot 就不用转换
    wxmlAst.children = children.map((k) => convertAst(k, options, util))
  }
  // 处理 if，但是 没搞懂
  if (ifConditions) {
    const length = ifConditions.length
    for (let i = 1; i < length; i++) {
      wxmlAst.ifConditions[i].block = convertAst(ifConditions[i].block, options, util)
    }
  }

  return wxmlAst
}
```

##### tag 转换部分

```
// tag 方法，进行 tag 的转换
export default function (ast, options) {
  // 从语法树中获得转换该节点的必备属性
  const { tag, elseif, else: elseText, for: forText, staticClass = '', attrsMap = {}} = ast
  // 获得该节点的父节点里面的所有子组件，在 components 中
  const { components } = options
  // 获得属性
  const { 'v-if': ifText, href, 'v-bind:href': bindHref, name } = attrsMap

  if (!tag) {
    return ast
  }
  
  // 如果是父节点的直接子组件
  const isComponent = component.isComponent(tag, components)
  // 如果不是一个子组件，也不是小程序中内置的一些虚拟标签
  if (tag !== 'template' && tag !== 'block' && tag !== 'slot' && !isComponent) {
    // 在这个节点的 class 上面，添加 `_${tag}` 这个类名
    ast.staticClass = staticClass ? `_${tag} ${staticClass}` : `_${tag}`
  }
  // tagMap 中记录了前端的标签对应小程序标签的转换规则
  ast.tag = tagMap[tag] || tag

  const isSlot = tag === 'slot'

  // 处理 tag 为 template 以及节点上使用了 if/esle/for 属性的情况
  if ((ifText || elseif || elseText || forText) && tag === 'template') {
    ast.tag = 'block'
  } else if (isComponent || isSlot) {
    // 将组件和 slot 都作为转换为小程序的 template
    const originSlotName = name || 'default'
    const slotName = isSlot ? `$slot${originSlotName} || '${originSlotName}'` : undefined

    // 用完必须删除，不然会被编译成 <template name="xxx"> 在小程序中就会表示这是一个模版申明而不是使用，小程序中不能同时申明和使用模版
    delete ast.attrsMap.name
    // 组件或 slot 转换成小程序的逻辑，主要是处理 data 和 is 属性
    ast = component.convertComponent(ast, components, slotName)
    ast.tag = 'template'
  } else if (tag === 'a' && !(href || bindHref)) {
    // 剩下都是一些其他的标签转换逻辑
    ast.tag = 'view'
  } else if (ast.events && ast.events.scroll) {
    ast.tag = 'scroll-view'
  } else if (tag === 'input') {
    const type = attrsMap.type
    if (type && ['button', 'checkbox', 'radio'].indexOf(type) > -1) {
      delete ast.attrsMap.type
      ast.tag = type
    }
    if (type === 'button') {
      ast.children.push({
        text: attrsMap.value || '',
        type: 3
      })
      delete ast.attrsMap.value
    }
  }
  return ast
}
```

##### attr 转换部分

```
convertAttr (ast, log) {
  const { attrsMap = {}, tag, staticClass } = ast
  let attrs = {}
  // 处理 class name，这里的 class name 可能是静态的 class，也可能是 v-bind:class 这种动态绑定的 class
  const wxClass = this.classObj(attrsMap['v-bind:class'], staticClass)
  // 处理后的 class 再赋值给 attrsMap['class']
  wxClass.length ? attrsMap['class'] = wxClass : ''
  // 处理 style，这里的 style 同理可能是静态的 Style 以及动态的 style
  const wxStyle = this.styleObj(attrsMap['v-bind:style'], attrsMap['style'])
  // 处理后的 style 再赋值给 attrsMap['style']
  wxStyle.length ? attrsMap['style'] = wxStyle : ''

  Object.keys(attrsMap).map(key => {
    const val = attrsMap[key]
    if (key === 'v-bind:class' || key === 'v-bind:style') {
      return
    }
    // 支持 v-text 属性：https://cn.vuejs.org/v2/api/#v-text
    if (key === 'v-text') {
      ast.children.unshift({
        text: `{{${val}}}`,
        type: 3
      })
    // 使用 rich-text 支持 v-html
    } else if (key === 'v-html') {
      ast.tag = 'rich-text'
      attrs['nodes'] = `{{${val}}}`
    // 支持 v-show
    } else if (key === 'v-show') {
      attrs['hidden'] = `{{!(${val})}}`
    // 支持 v-on，对事件进行处理，事件处理分析见下面
    } else if (/^v\-on\:/i.test(key)) {
      attrs = this.event(key, val, attrs, tag)
    // 支持 v-bind，处理对于属性的传值
    } else if (/^v\-bind\:/i.test(key)) {
      attrs = this.bind(key, val, attrs, tag, attrsMap['wx:key'])
    // 支持 v-model。v-model 其实是由两个基础的事件组成的，所以这里需要将 v-model 转换成 Vue 的基础事件，然后再转换成小程序的基础事件，最后使用 handleProxy 进行处理
    } else if (/^v\-model/.test(key)) {
      attrs = this.model(key, val, attrs, tag, log)
    } else if (wxmlDirectiveMap[key]) {
      // 在这里处理一些之前没有处理到的标签，比如 v-if，v-else-if，v-else，v-html 等
      const { name = '', type, map = {}, check } = wxmlDirectiveMap[key] || {}
      if (!(check && !check(key, val, log)) && !(!name || typeof type !== 'number')) {
        // 见 ./wxmlDirectiveMap.js 注释
        if (type === 0) {
          attrs[name] = `{{${val}}}`
        }

        if (type === 1) {
          attrs[name] = undefined
        }

        if (type === 2) {
          attrs[name] = val
        }

        if (type === 3) {
          attrs[map[name] || name] = `{{${val}}}`
          return
        }
      }
    } else if (/^v\-/.test(key)) {
      log(`不支持此属性-> ${key}="${val}"`, 'waring')
    } else {
      // ['slot', 'template', 'block'] 不支持 class、style 和 data-mpcomid 属性
      if ((tagConfig.virtualTag.indexOf(tag) > -1) && (key === 'class' || key === 'style' || key === 'data-mpcomid')) {
        if (key !== 'data-mpcomid') {
          log(`template 不支持此属性-> ${key}="${val}"`, 'waring')
        }
      } else {
        attrs[key] = val
      }
    }
  })
  ast.attrsMap = attrs
  return ast
}
```

##### event，处理 vue 的事件转换到小程序的事件


```
// key 是事件的名称，如 v-on:click，val 是赋给该事件的处理函数，如：bindViewTap，attr 是该节点的属性，tag 是该节点的标签名，这里的 tag 已经转换为小程序的标签名
event (key, val, attrs, tag) {
  // 小程序能力所致，bind 和 catch 事件同时绑定时候，只会触发 bind ,catch 不会被触发。
  // .stop 的使用会阻止冒泡，但是同时绑定了一个非冒泡事件，会导致该元素上的 catchEventName 失效！
  // .prevent 可以直接干掉，因为小程序里没有什么默认事件，比如submit并不会跳转页面
  // .capture 不能做，因为小程序没有捕获类型的事件
  // .self 没有可以判断的标识
  // .once 也不能做，因为小程序没有 removeEventListener, 虽然可以直接在 handleProxy 中处理，但非常的不优雅，违背了原意，暂不考虑
  const name = key.replace(/^v\-on\:/i, '').replace(/\.prevent/i, '')
  // 将事件名称赋值给 eventName
  const [eventName, ...eventNameMap] = name.split('.')
  // 获得 Vue 的事件在小程序事件上的映射关系，在 evnetMap 里
  const { 'v-on': eventMap, check } = wxmlDirectiveMap

  if (check) {
    check(key, val)
  }
  // 获得小程序的事件名称
  let wxmlEventName = ''
  if (eventName === 'change' && (tag === 'input' || tag === 'textarea')) {
    wxmlEventName = 'blur'
  } else {
    wxmlEventName = eventMap.map[eventName]
  }

  // 小程序有四种处理事件方案：bind事件绑定不会阻止冒泡事件向上冒泡，catch事件绑定可以阻止冒泡事件向上冒泡。
  // 在捕获阶段处理：capture-bind、capture-catch关键字，后者将中断捕获阶段和取消冒泡阶段。
  let eventType = 'bind'
  const isStop = eventNameMap.includes('stop')
  if (eventNameMap.includes('capture')) {
    eventType = isStop ? 'capture-catch:' : 'capture-bind:'
  } else if (isStop) {
    eventType = 'catch'
  }

  // 所有的事件都交给 handleProxy 进行处理
  wxmlEventName = eventType + (wxmlEventName || eventName)
  attrs[wxmlEventName] = 'handleProxy'

  return attrs
},
```

handleProxy 的部分在 runtime 里面，在 `runtime/lifecycle.js` 有定义 `handleProxy` 这个方法，调用的是 `handleProxyWithVue`：

```
// runtime/lifecycle.js
handleProxy (e) {
  return rootVueVM.$handleProxyWithVue(e)
},

// runtime/event.js
export function handleProxyWithVue (e) {
  const rootVueVM = this.$root
  // target 是当前事件发生的节点
  // currentTarget 是 event 对象绑定的节点
  const { type, target = {}, currentTarget } = e
  const { dataset = {}} = currentTarget || target
  const { comkey = '', eventid } = dataset
  // 通过 comkey，从 root 节点遍历 children，找到事件处理函数的节点
  const vm = getVM(rootVueVM, comkey.split(','))

  if (!vm) {
    return
  }

  // 在小程序中，使用的是小程序是事件，但是处理事件的处理函数是 Vue 写的，所以需要转换成 Vue 的事件
  const webEventTypes = eventTypeMap[type] || [type]
  // 通过 vm._vnode 来找到该事件的处理函数，以数组的形式进行返回
  const handles = getHandle(vm._vnode, eventid, webEventTypes)

  // TODO, enevt 还需要处理更多
  // https://developer.mozilla.org/zh-CN/docs/Web/API/Event
  if (handles.length) {
    // 事件的内容是小程序的事件内容，还需要转换成 Web 事件内容的格式
    const event = getWebEventByMP(e)
    if (handles.length === 1) {
      const result = handles[0](event)
      return result
    }
    handles.forEach(h => h(event))
  }
}
```


#### generate，将转换成的小程序语法树转换成小程序的代码

```
//compiler/index.js
let code = generate(wxast, options)

// compiler/generate.js
export default function generate (obj, options = {}) {
  const { tag, attrsMap = {}, children, text, ifConditions } = obj
  if (!tag) return text
  let child = ''
  if (children && children.length) {
    // 递归子节点
    child = children.map(v => generate(v, options)).join('')
  }

  // v-if 指令
  const ifConditionsArr = []
  if (ifConditions) {
    const length = ifConditions.length
    for (let i = 1; i < length; i++) {
      ifConditionsArr.push(generate(ifConditions[i].block, options))
    }
  }

  const attrs = Object.keys(attrsMap).map(k => convertAttr(k, attrsMap[k])).join(' ')

  // 处理小程序代码的拼接，有一些比较特殊的标签代码需要特殊处理
  const tags = ['progress', 'checkbox', 'switch', 'input', 'radio', 'slider', 'textarea']
  if (tags.indexOf(tag) > -1 && !(children && children.length)) {
    return `<${tag}${attrs ? ' ' + attrs : ''} />${ifConditionsArr.join('')}`
  }
  return `<${tag}${attrs ? ' ' + attrs : ''}>${child || ''}</${tag}>${ifConditionsArr.join('')}`
}

// 将属性转换成 key=value 的形式 或者 key 的形式
function convertAttr (key, val) {
  return (val === '' || typeof val === 'undefined') ? key : `${key}="${val}"`
}
```

#### compileToWxml 剩下的操作

```
export function compileToWxml (compiled, options = {}) {
  // TODO, compiled is undefined
  const { components = {}} = options
  const log = utils.log(compiled)

  // 解析 vue 的语法树
  const { wxast, deps = {}, slots = {}} = wxmlAst(compiled, options, log)
  // 生成小程序的语法树
  let code = generate(wxast, options)

  // 引用子模版，处理 Vue 中的组件在小程序中的引用
  const importCode = Object.keys(deps).map(k => components[k] ? `<import src="${components[k].src}" />` : '').join('')
  code = `${importCode}<template name="${options.name}">${code}</template>`

  // 生成 slots code
  Object.keys(slots).forEach(k => {
    const slot = slots[k]
    slot.code = generate(slot.node, options)
  })

  // TODO: 后期优化掉这种暴力全部 import，虽然对性能没啥大影响
  return { code, compiled, slots, importCode }
}
```


这里会导出三个函数，但是这三个函数是怎么被调用的？又是在什么时候被调用的？从 mpvue compiler 中找不到，因为 mpvue-template-compiler 是通过 mpvue-loader 被引入到 webpack 中，所以这些问题都需要在 mpvue-loader 中找答案。

## mpvue-loader 

mpvue loader 的源码在 [mpvue/mpvue-loader | Github](https://github.com/mpvue/mpvue-loader) 上









## 参考

* [mpvue-docs | quickstart](http://mpvue.com/mpvue/quickstart/)