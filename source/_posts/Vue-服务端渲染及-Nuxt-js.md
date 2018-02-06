---
title: Vue 服务端渲染及 Nuxt.js
date: 2018-02-04 15:31:53
categories: coding
tags:
  - JavaScript
  - Vue
  - Nuxt.js
  - SSR
---

本文是 Vue.js 服务端渲染，以及服务端渲染脚手架 Nuxt.js 的学习笔记

<!--more-->

## 服务端渲染

Vue.js 是构建客户端应用程序的框架。默认情况下，可以在浏览器中输出 Vue 组件，进行生成 DOM 和操作 DOM。然而，也可以将同一个组件渲染为服务器端的 HTML 字符串，将它们直接发送到浏览器，最后将静态标记"混合"为客户端上完全交互的应用程序。

### 服务端渲染的好处

* 更好的 SEO，由于搜索引擎爬虫抓取工具可以直接查看完全渲染的页面。
* 更快的内容到达时间(time-to-content)，特别是对于缓慢的网络情况或运行缓慢的设备。

### 服务端渲染的不方便之处

* 开发条件所限。浏览器特定的代码，只能在某些生命周期钩子函数(lifecycle hook)中使用；一些外部扩展库(external library)可能需要特殊处理，才能在服务器渲染应用程序中运行。
* 涉及构建设置和部署的更多要求。与可以部署在任何静态文件服务器上的完全静态单页面应用程序(SPA)不同，服务器渲染应用程序，需要处于 Node.js server 运行环境。
* 更多的服务器端负载。

### 服务器端渲染 vs 预渲染(SSR vs Prerendering)

如果你调研服务器端渲染(SSR)只是用来改善少数营销页面（例如 /, /about, /contact 等）的 SEO，那么你可能需要预渲染。

在构建时(build time)简单地生成针对特定路由的静态 HTML 文件。

### 组件生命周期钩子函数

由于没有动态更新，所有的生命周期钩子函数中，只有 beforeCreate 和 created 会在服务器端渲染(SSR)过程中被调用。这就是说任何其他生命周期钩子函数中的代码（例如 beforeMount 或 mounted），只会在客户端执行。

请将副作用代码移动到 beforeMount 或 mounted 生命周期中。

### 自定义指令

大多数自定义指令直接操作 DOM，因此会在服务器端渲染(SSR)过程中导致错误。有两种方法可以解决这个问题：

* 推荐使用组件作为抽象机制，并运行在「虚拟 DOM 层级(Virtual-DOM level)」（例如，使用渲染函数(render function)）。
* 如果你有一个自定义指令，但是不是很容易替换为组件，则可以在创建服务器 renderer 时，使用 directives 选项所提供"服务器端版本(server-side version)"。

### 如何构建

[Vue.js 服务器端渲染指南](https://ssr.vuejs.org/zh/) 这篇文章介绍了，如何构建服务端渲染 + 客户端渲染的混合应用

## Nuxt.js

Nuxt.js 预设了利用Vue.js开发服务端渲染的应用所需要的各种配置。

Nuxt.js 集成了以下组件/框架，用于开发完整而强大的 Web 应用：

* Vue 2
* Vue-Router
* Vuex (当配置了 Vuex 状态树配置项 时才会引入)
* Vue-Meta

当运行 nuxt 命令时，会启动一个支持 热加载 和 服务端渲染（基于Vue.js的 vue-server-renderer 模块）的开发服务器。

### 静态化

支持 Vue.js 应用的静态化算是 Nuxt.js 的一个创新点，通过 nuxt generate 命令实现。

该命令依据应用的路由配置将每一个路由静态化成为对应的HTML文件。

例子：

```
-| pages/
----| about.vue
----| index.vue
```

静态化后变成：

```
-| dist/
----| about/
------| index.html
----| index.html
```

静态化可以让你在任何一个静态站点服务商托管你的Web应用。

### 安装 

注意：Nuxt.js 会监听 pages 目录中的文件变更并自动重启， 当添加新页面时没有必要手工重启应用。

### 目录结构

#### 资源目录

资源目录 assets 用于组织未编译的静态资源如 LESS、SASS 或 JavaScript。

默认情况下 Nuxt 使用 vue-loader、file-loader 以及 url-loader 这几个 Webpack 加载器来处理文件的加载和引用。对于不需要通过 Webpack 处理的静态资源文件，可以放置在 static 目录中。

Nuxt 服务器启动的时候，该目录下的文件会映射至应用的根路径 / 下，像 robots.txt 或 sitemap.xml 这种类型的文件就很适合放到 static 目录中。

你可以在代码中使用根路径 / 结合资源相对路径来引用静态资源：

```
<!-- 引用 static 目录下的图片 -->
<img src="/my-image.png"/>

<!-- 引用 assets 目录下经过 webpack 构建处理后的图片 -->
<img src="/assets/my-image-2.png"/>
```

### 组件目录

组件目录 components 用于组织应用的 Vue.js 组件。Nuxt.js 不会扩展增强该目录下 Vue.js 组件，即这些组件不会像页面组件那样有 `asyncData` 方法的特性。

### 布局目录

布局目录 layouts 用于组织应用的布局组件。

该目录名为Nuxt.js保留的，不可更改。

#### 默认布局

可通过添加 layouts/default.vue 文件来扩展应用的默认布局。

别忘了在布局文件中添加 `<nuxt/>` 组件用于显示页面的主体内容。

#### 错误页面

你可以通过编辑 layouts/error.vue 文件来定制化错误页面.

#### 个性化布局

layouts 根目录下的所有文件都属于个性化布局文件，可以在页面组件中利用 layout 属性来引用。

[layout 属性](https://zh.nuxtjs.org/api/pages-layout)

### 中间件目录

middleware 目录用于存放应用的中间件。中间件允许您定义一个自定义函数运行在一个页面或一组页面渲染之前。

一个使用中间件的真实例子，请参阅在 GitHub 上的[example-auth0](https://github.com/nuxt/example-auth0)。

### 页面目录

页面目录 pages 用于组织应用的路由及视图。Nuxt.js 框架读取该目录下所有的 .vue 文件并自动生成对应的路由配置。

该目录名为Nuxt.js保留的，不可更改。

#### 模板

定制化默认的 html 模板，只需要在应用根目录下创建一个 app.html 的文件。

默认模板为：

```
<!DOCTYPE html>
<html {{ HTML_ATTRS }}>
  <head>
    {{ HEAD }}
  </head>
  <body {{ BODY_ATTRS }}>
    {{ APP }}
  </body>
</html>
```

#### 页面

页面组件实际上是 Vue 组件，只不过 Nuxt.js 为这些组件添加了一些特殊的配置项（对应 Nuxt.js 提供的功能特性）以便你能快速开发通用应用。

具体页面相对于普通的组件，提供了哪些新的属性，我们可以在 [视图 - 页面](https://zh.nuxtjs.org/guide/views#%E9%A1%B5%E9%9D%A2) 这个表格里找到

### 插件目录

插件目录 plugins 用于组织那些需要在 根vue.js应用 实例化之前需要运行的 Javascript 插件。

需要注意的是，在任何 Vue 组件的生命周期内， 只有 beforeCreate 和 created 这两个钩子方法会在 客户端和服务端均被调用。其他钩子方法仅在客户端被调用。

#### 使用第三方模块

通过在 nuxt.config.js 里面配置 build.vendor 来解决：

```
module.exports = {
  build: {
    vendor: ['axios']
  }
}
```

经过上面的配置后，我们可以在任何页面里面引入 axios 而不用担心它会被重复打包

### 静态文件目录

静态文件目录 static 用于存放应用的静态文件，此类文件不会被 Nuxt.js 调用 Webpack 进行构建编译处理。 服务器启动的时候，该目录下的文件会映射至应用的根路径 `/` 下。

举个例子: `/static/robots.txt` 映射至 `/robots.txt`

你可以在代码中使用根路径 / 结合资源相对路径来引用静态资源：

```
<!-- 引用 static 目录下的图片 -->
<img src="/my-image.png"/>

<!-- 引用 assets 目录下经过 webpack 构建处理后的图片 -->
<img src="/assets/my-image-2.png"/>
```

### Store 目录

store 目录用于组织应用的 Vuex 状态树 文件。 Nuxt.js 框架集成了 Vuex 状态树 的相关功能配置，在 store 目录下创建一个 index.js 文件可激活这些配置。

该目录名为Nuxt.js保留的，不可更改。

Nuxt.js 会尝试找到应用根目录下的 store 目录，如果该目录存在，它将做以下的事情：

1. 引用 vuex 模块
2. 将 vuex 模块 加到 vendors 构建配置中去
3. 设置 Vue 根实例的 store 配置项

#### fetch 方法

fetch 方法会在渲染页面前被调用，作用是填充状态树 (store) 数据，与 asyncData 方法类似，不同的是它不会设置组件的数据。

如果页面组件设置了 fetch 方法，它会在组件每次加载前被调用（在服务端或切换至目标路由之前）。

### nuxt.config.js 文件

nuxt.config.js 文件用于组织Nuxt.js 应用的个性化配置，以便覆盖默认配置。

#### build
Nuxt.js 允许你在自动生成的 vendor.bundle.js 文件中添加一些模块，以减少应用 bundle 的体积。如果你的应用依赖第三方模块，这个配置项是十分实用的。

关于 build 的配置，可以查看 [构建配置](https://zh.nuxtjs.org/api/configuration-build)




## 参考

* [Vue.js 服务器端渲染指南](https://ssr.vuejs.org/zh/)
* [关于 Nuxt.js](https://zh.nuxtjs.org/guide)



