---
title: Vue CLI 学习
date: 2018-08-18 23:24:44
categories: coding
tags:
  - JavaScript
  - Vue
  - Vue CLI
  - Webpack
---

本文是学习 Vue CLI v3 的学习笔记

## 总结 Vue CLI 的相关库

1. @vue/cli：是一个全局安装的 npm 包，提供了终端里的 `vue` 命令，如 `vue create、vue serve、vue ui、vue build、vue add`等
2. @vue/cli-service-global：使用 vue serve 和 vue build 命令对单个 *.vue 文件进行快速原型开发，不过这需要先额外安装一个全局的扩展
3. @vue/cli-init：旧版本的 `vue init` 功能，你可以全局安装一个桥接工具：`npm install -g @vue/cli-init`
4. @vue/cli-service：开发环境依赖。它是一个 npm 包，局部安装在每个 `@vue/cli` 创建的项目中，加载其它 CLI 插件的核心服务；针对绝大部分应用优化过的内部的 webpack 配置；`vue-cli-service` 命令，提供 `serve、build` 和 `inspect` 命令。
5. `@vue/cli-plugin-` (内建插件) 或 `vue-cli-plugin-` (社区插件)
6. @vue/babel-preset-app：babel 相关的默认配置
7. @vue/eslint-config-standard：eslint standard 相关配置
8. @vue/eslint-config-typescript：eslint typescript 相关配置
9. @vue/preload-webpack-plugin：Preload 和 Prefetch 支持
10. thread-loader：在多核 CPU 的机器上为 Babel/TypeScript 转译开启
11. webpack-chain：Vue CLI 内部的 webpack 配置是通过 `webpack-chain` 维护的。




<!--more-->

本文内容参考 [Vue CLI 官方文档](https://cli.vuejs.org/zh/)

需要测试的命令：

1. `vue serve`
2. `vue build`
3. `vue create`

## 前言

Vue CLI 分为多个部分，这几个部分分为独立的包进行发布，用户在选择能力之后会自动下载最新的包。在 [Vue-CLI 项目](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue) 的 `package/` 目录下可以找到分包。

### 常用的包

#### CLI


CLI (`@vue/cli`) 是一个全局安装的 npm 包，提供了终端里的 `vue` 命令。

* `vue create` 快速创建一个新项目的脚手架
* `vue serve` 运行调试项目。缺点是它需要安装全局依赖，这使得它在不同机器上的一致性不能得到保证。因此这只适用于快速原型开发。
* `vue ui` 通过一套图形化界面管理你的所有项目，包含 `vue create` 流程，以及 `npm run serve/build/inspect/lint` 等结果。
* `vue --version` 用这个命令来检查其版本是否正确 (3.x)

#### CLI 服务

CLI 服务 (`@vue/cli-service`) 是一个开发环境依赖。它是一个 npm 包，局部安装在每个 `@vue/cli` 创建的项目中。

CLI 服务是构建于 `webpack` 和 `webpack-dev-server` 之上的。它包含了：

* 加载其它 CLI 插件的核心服务；
* 一个针对绝大部分应用优化过的内部的 webpack 配置；
* 项目内部的 `vue-cli-service` 命令，提供 `serve、build` 和 `inspect` 命令。**这个命令是配置在 package.json script 中的**

### CLI 插件


CLI 插件是向你的 Vue 项目提供可选功能的 npm 包，例如 Babel/TypeScript 转译、ESLint 集成、单元测试和 end-to-end 测试等。

`@vue/cli-plugin-` (内建插件) 或 `vue-cli-plugin-` (社区插件) 开头

当你在项目内部运行 `vue-cli-service` 命令时，它会自动解析并加载 `package.json` 中列出的所有 CLI 插件。


## 基础

### 快速原型开发

**注意，这种方案只适合于快速开发，如果是用在生产环境中，可能因为 serve/build 没有配置好指定的环境变量，导致开发，或者构建的包出现问题，所以，在没有足够了解的情况下，使用这种方案需要谨慎。**

使用 vue serve 和 vue build 命令对单个 *.vue 文件进行快速原型开发，不过这需要先额外安装一个全局的扩展：

```
npm install -g @vue/cli-service-global
```

`vue serve` 的缺点就是它需要安装全局依赖，这使得它在不同机器上的一致性不能得到保证。因此这只适用于快速原型开发。

#### vue serve

`vue serve` 使用了和 `vue create` 创建的项目相同的默认设置 (webpack、Babel、PostCSS 和 ESLint)。

入口可以是 `main.js、index.js、App.vue` 或 `app.vue` 中的一个。你也可以显式地指定入口文件：

```
vue serve MyComponent.vue
```

```
Usage: serve [options] [entry]

在开发环境模式下零配置为 .js 或 .vue 文件启动一个服务器


Options:

  -o, --open  打开浏览器
  -c, --copy  将本地 URL 复制到剪切板
  -h, --help  输出用法信息
```

#### vue build

使用 `vue build` 将目标文件构建成一个生产环境的包并用来部署，

`vue build` 也提供了将组件构建成为一个库或一个 Web Components 组件的能力，可以通过设置构建目标来实现。

```
vue build MyComponent.vue
```

```
Usage: build [options] [entry]

在生产环境模式下零配置构建一个 .js 或 .vue 文件


Options:

  -t, --target <target>  构建目标 (app | lib | wc | wc-async, 默认值：app)
  -n, --name <name>      库的名字或 Web Components 组件的名字 (默认值：入口文件名)
  -d, --dest <dir>       输出目录 (默认值：dist)
  -h, --help             输出用法信息
```

### 创建一个项目

```
vue create hello-world
```

![](http://ojt6zsxg2.bkt.clouddn.com/d86a046fe759c4ceb1bc93cc206880a4.png)

E2E 测试要从 `https://chromedriver.storage.googleapis.com/2.41/chromedriver_mac64.zip` 下载包，我的网比较差，所以暂时不测试 E2E 了。。。

```
用法：create [options] <app-name>

创建一个由 `vue-cli-service` 提供支持的新项目


选项：

  -p, --preset <presetName>       忽略提示符并使用已保存的或远程的预设选项
  -d, --default                   忽略提示符并使用默认预设选项
  -i, --inlinePreset <json>       忽略提示符并使用内联的 JSON 字符串预设选项
  -m, --packageManager <command>  在安装依赖时使用指定的 npm 客户端
  -r, --registry <url>            在安装依赖时使用指定的 npm registry (仅用于 npm 客户端)
  -g, --git [message]             强制 / 跳过 git 初始化，并可选的指定初始化提交信息
  -n, --no-git                    跳过 git 初始化
  -f, --force                     覆写目标目录可能存在的配置
  -c, --clone                     使用 git clone 获取远程预设选项
  -x, --proxy                     使用指定的代理创建项目
  -b, --bare                      创建项目时省略默认组件中的新手指导信息
  -h, --help                      输出使用帮助信息
```


#### 拉取 2.x 模板 (旧版本)


Vue CLI 3 和旧版使用了相同的 vue 命令，所以 Vue CLI 2 (`vue-cli`) 被覆盖了。如果你仍然需要使用旧版本的 `vue init` 功能，你可以全局安装一个桥接工具：


```
npm install -g @vue/cli-init
# `vue init` 的运行效果将会跟 `vue-cli@2.x` 相同
vue init webpack my-project
```

Vue CLI 使用了一套基于插件的架构。依赖都是以 `@vue/cli-plugin-` 开头的。插件可以修改 webpack 的内部配置，也可以向 `vue-cli-service` 注入命令。


### 插件和 Preset

#### 在现有的项目中安装插件


每个 CLI 插件都会包含一个 (用来创建文件的) 生成器和一个 (用来调整 webpack 核心配置和注入命令的) 运行时插件。

```
vue add @vue/eslint

# 这个和之前的用法等价
vue add @vue/cli-plugin-eslint
```

如果不带 @vue 前缀，该命令会换作解析一个 unscoped 的包。例如以下命令会安装第三方插件 `vue-cli-plugin-apollo`

```
# 安装并调用 vue-cli-plugin-apollo
vue add apollo
```

你也可以基于一个指定的 scope 使用第三方插件，`@foo/vue-cli-plugin-bar`

```
vue add @foo/bar
```

```
推荐在运行 vue add 之前将项目的最新状态提交，因为该命令可能调用插件的文件生成器并很有可能更改你现有的文件。
```

如果一个插件已经被安装，你可以使用 `vue invoke` 命令跳过安装过程，只调用它的生成器。

#### 项目本地的插件

[插件开发指南和 UI 开发指南](https://cli.vuejs.org/zh/dev-guide/plugin-dev.html)

#### Preset

一个 Vue CLI preset 是一个包含创建新项目所需预定义选项和插件的 JSON 对象

`vue create` 过程中保存的 preset 会被放在你的 home 目录下的一个配置文件中 (`~/.vuerc`)。你可以通过直接编辑这个文件来调整、添加、删除保存好的 preset。

```
"no-e2e-all-preset": {
  "useConfigFiles": false,
  "plugins": {
    "@vue/cli-plugin-babel": {},
    "@vue/cli-plugin-typescript": {
      "classComponent": true,
      "useTsWithBabel": true
    },
    "@vue/cli-plugin-pwa": {},
    "@vue/cli-plugin-eslint": {
      "config": "standard",
      "lintOn": [
        "save",
        "commit"
      ]
    },
    "@vue/cli-plugin-unit-mocha": {}
  },
  "router": true,
  "routerHistoryMode": true,
  "vuex": true,
  "cssPreprocessor": "sass"
}
```

额外的配置将会根据 `useConfigFiles` 的值被合并到 `package.json` 或相应的配置文件中（比如 `vue.config.js`）。

**推荐为 preset 列出的所有第三方插件提供显式的版本范围。**

### CLI

在一个 Vue CLI 项目中，`@vue/cli-service` 安装了一个名为 `vue-cli-service` 的命令。你可以在 npm scripts 中以 `vue-cli-service`、或者从终端中以 `./node_modules/.bin/vue-cli-service` 访问这个命令。

[vue.config.js 配置命令](https://cli.vuejs.org/zh/config/)


查看所有支持的命令：

```
npx vue-cli-service help
```

#### 缓存和并行处理

`cache-loader` 会默认为 Vue/Babel/TypeScript 编译开启。文件会缓存在 `node_modules/.cache` 中


`thread-loader` 会在多核 CPU 的机器上为 Babel/TypeScript 转译开启。

#### Git Hook

`@vue/cli-service` 也会安装 `yorkie`，它会让你在 `package.json` 的 `gitHooks` 字段中方便地指定 Git hook

## 开发

### 浏览器兼容性

package.json 文件里的 `browserslist` 字段 (或一个单独的 `.browserslistrc` 文件)，指定了项目的目标浏览器的范围。这个值会被 `@babel/preset-env` 和 `Autoprefixer` 用来确定需要转译的 JavaScript 特性和需要添加的 CSS 浏览器前缀。


### Polyfill

默认的 Vue CLI 项目会使用 `@vue/babel-preset-app`，它通过 `@babel/preset-env` 和 `browserslist` 配置来决定项目需要的 polyfill。

**如果其中一个依赖需要特殊的 polyfill，默认情况下 Babel 无法将其检测出来。需要使用其他几种方法添加**：[Polyfill](https://cli.vuejs.org/zh/guide/browser-compatibility.html#usebuiltins-usage)

* **依赖**基于一个目标环境不支持的 ES 版本撰写: 配置 transpileDependencies 选项。默认情况下 babel-loader 会忽略所有 node_modules 中的文件。如果你想要通过 Babel 显式转译一个依赖，可以在这个选项中列出来。
* 如果该依赖交付了 ES5 代码并显式地列出了需要的 polyfill，使用 `@vue/babel-preset-app` 的 `polyfills` 选项预包含所需要的 polyfill。

#### 现代模式

```
vue-cli-service build --modern
```

Vue CLI 会产生两个应用的版本：一个现代版的包，面向支持 ES modules 的现代浏览器，另一个旧版的包，面向不支持的旧浏览器。

### HTML 和静态资源

#### HTML


`public/index.html` 文件是一个会被 `html-webpack-plugin` 处理的模板。在构建过程中，资源链接会被自动注入。另外，Vue CLI 也会自动注入 `resource hint (preload/prefetch、manifest` 和图标链接 (当用到 PWA 插件时) 以及构建过程中处理的 JavaScript 和 CSS 文件的资源链接。

#### Preload 和 Prefetch 支持

使用  `@vue/preload-webpack-plugin` 注入

```
提示

Prefetch 链接将会消耗带宽。如果你的应用很大且有很多 async chunk，而用户主要使用的是对带宽较敏感的移动端，那么你可能需要关掉 prefetch 链接并手动选择要提前获取的代码区块。
```

#### 不生成 index

当基于已有的后端使用 Vue CLI 时，你可能不需要生成 index.html，这样生成的资源可以**用于一个服务端渲染的页面。**

#### 构建一个多页应用

Vue CLI 支持使用 vue.config.js 中的 pages 选项构建一个多页面的应用。构建好的应用将会在不同的入口之间高效共享通用的 chunk 以获得最佳的加载性能。

```
提示

当在 multi-page 模式下构建时，webpack 配置会包含不一样的插件 (这时会存在多个 html-webpack-plugin 和 preload-webpack-plugin 的实例)。如果你试图修改这些插件的选项，请确认运行 vue inspect。
```

### 处理静态资源


静态资源可以通过两种方式进行处理：

1. 在 JavaScript 被导入或在 template/CSS 中通过相对路径被引用。这类引用会被 webpack 处理。
2. 放置在 public 目录下或通过绝对路径被引用。这类资源将会直接被拷贝，而不会经过 webpack 的处理。

#### 从相对路径导入

在 JavaScript、CSS 或 *.vue 文件中使用相对路径 (必须以 `.` 开头) 引用一个静态资源

所有诸如 `<img src="...">、background: url(...) 和 CSS @import` 的资源 URL 都会被解析为一个模块依赖。

#### public 文件夹

任何放置在 public 文件夹的静态资源都会被简单的复制，而不经过 webpack。需要通过绝对路径来引用它们。

`public` 目录提供的是一个**应急手段**，当你通过绝对路径引用它时，留意应用将会部署到哪里。如果你的应用没有部署在域名的根部，那么你需要为你的 URL 配置 `baseUrl` 前缀

### CSS 相关

Vue CLI 项目天生支持 PostCSS、CSS Modules 和包含 Sass、Less、Stylus 在内的预处理器。


#### 引用静态资源

所有编译后的 CSS 都会通过 `css-loader` 来解析其中的 `url()` 引用，并将这些引用作为模块请求来处理。这意味着你可以根据本地的文件结构用相对路径来引用静态资源。 

#### PostCSS

你可以通过 `.postcssrc` 或任何 `postcss-load-config` 支持的配置源来配置 PostCSS。

#### CSS Modules

通过 `<style module>` 以开箱即用的方式在 `*.vue` 文件中使用 CSS Modules

```
import styles from './foo.module.css'
// 所有支持的预处理器都一样工作
import sassStyles from './foo.module.scss'
```

#### 向预处理器 Loader 传递选项

使用 `vue.config.js` 中的 `css.loaderOptions` 选项。

### webpack 相关

调整 webpack 配置最简单的方式就是在 `vue.config.js` 中的 `configureWebpack` 选项提供一个对象，该对象将会被 `webpack-merge` 合并入最终的 webpack 配置。


### 链式操作 (高级)


Vue CLI 内部的 webpack 配置是通过 `webpack-chain` 维护的。

这个库提供了一个 webpack 原始配置的**上层抽象**，使其可以定义具名的 loader 规则和具名插件


### 审查项目的 webpack 配置

`vue-cli-service` 暴露了 `inspect` 命令用于审查解析好的 webpack 配置。

```
vue inspect > output.js
```

也可以通过指定一个路径来审查配置的一小部分：

```
只审查第一条规则
vue inspect module.rules.0
```

或者指向一个规则或插件的名字：

```
vue inspect --rule vue
vue inspect --plugin html
```

最后，你可以列出所有规则和插件的名字：

```
vue inspect --rules
vue inspect --plugins
```

### 环境变量和模式

可以替换你的项目根目录中的下列文件来指定环境变量：

```
.env                # 在所有的环境中被载入
.env.local          # 在所有的环境中被载入，但会被 git 忽略
.env.[mode]         # 只在指定的模式中被载入
.env.[mode].local   # 只在指定的模式中被载入，但会被 git 忽略
```

```
环境加载属性

为一个特定模式准备的环境文件的 (例如 .env.production) 将会比一般的环境文件 (例如 .env) 拥有更高的优先级。
```

#### 模式



### 构建目标

运行 `vue-cli-service build` 时，你可以通过 `--target` 选项指定不同的构建目标。

构建目标根据项目本身的属性而有不同的配置。项目的属性有：**应用，库 和 组件**



## 参考

* [Vue CLI 官方文档](https://cli.vuejs.org/zh/)