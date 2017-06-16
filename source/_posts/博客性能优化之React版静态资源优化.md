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

## 分别打包不同的页面




