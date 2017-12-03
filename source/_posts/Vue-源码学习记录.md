---
title: Vue 源码学习记录
date: 2017-12-03 19:08:25
categories: coding
tags:
  - Vue.js
---

本文是我在学习 Vue.js 做的笔记，Vue.js 的版本为 2.5.9

<!--more-->

首先，为了能够在阅读源码的时候，更方便地知道这部分代码在项目的结构、功能中属于那部分，我们要对整个项目的文件结构进行一个展示。即使是文件结构，也能够体现出作者的设计思路

Vue.js 的源代码设计结构如下：

```
├── build --------------------------------- 构建相关的文件，里面有构建的配置，脚本等信息
├── dist ---------------------------------- 构建后文件的输出目录
├── examples ------------------------------ 存放一些使用Vue开发的应用案例
├── flow ---------------------------------- 使用开源项目 Flow 声明的一些类型
├── package.json -------------------------- 不解释
├── test ---------------------------------- 包含所有测试文件，测试的类型有：端到端测试、继承测试，服务端渲染测试相关、Weex 测试相关
├── src ----------------------------------- 这个是我们最应该关注的目录，包含了源码
│   ├── compiler -------------------------- 编译器代码的存放目录，将 template 编译为 render 函数
│   │   ├── parser ------------------------ 存放将模板字符串转换成元素抽象语法树的代码
│   │   ├── codegen ----------------------- 存放从抽象语法树(AST)生成render函数的代码
│   │   ├── optimizer.js ------------------ 分析静态树，优化vdom渲染
│   ├── core ------------------------------ 存放通用的，平台无关的代码
│   │   ├── observer ---------------------- 反应系统，包含数据观测的核心代码
│   │   ├── vdom -------------------------- 包含虚拟DOM创建(creation)和打补丁(patching)的代码
│   │   ├── instance ---------------------- 包含Vue构造函数设计相关的代码
│   │   ├── global-api -------------------- 包含给Vue构造函数挂载全局方法(静态方法)或属性的代码
│   │   ├── components -------------------- 包含抽象出来的通用组件
│   ├── server ---------------------------- 包含服务端渲染(server-side rendering)的相关代码
│   ├── platforms ------------------------- 包含平台特有的相关代码
│   ├── sfc ------------------------------- 包含单文件组件(.vue文件)的解析逻辑，用于vue-template-compiler包
│   ├── shared ---------------------------- 包含整个代码库通用的代码
```

我们在 `package.json` 中，可以看到，整个项目所使用的打包工具是 rollup.js，然后运行 `npm run dev`进行调试，可以监听文件的变化进行自动打包输出到 `dist/vue.js`。所以，我们在自己学习 Vue.js 的时候，可以引入 `dist/vue.js` 并且开启调试模式，此时就可以一遍修改源码，一遍查看输出效果

## Vue 代码的运行流程

当我们看代码的时候，第一遍最好不要过于深入细节，而是先从整体流程上对项目有一个总体的把控，所以，我在第一节中，要记录的是项目的整体运行流程设计。

当我们引入 `dist/vue.js` 之后，要使用 `new Vue({...})` 来创建一个 Vue 的实例，用来将模板中的 HTML 代码嵌入到页面中，如下面的代码：

```
new Vue({
  el: '#app',
  template: '<div>Hello World</div>'
})
```

通过这一段代码，我们要实现用 `<div>Hello World</div>` 来替换 `id` 为 `app` 的元素。所以，我们可以看到，整个 Vue.js 的入口在 `Vue` 这个构造函数中，下面我们来看一看 `Vue` 这个构造函数的内容。

为了找到 `Vue` 构造函数的内容，我们从 `npm run dev` 这条调试命令入手。打开 `package.json`，在 `scripts` 中，我们可以找到 `npm run dev` 这条语句背后的命令：

```
"dev": "rollup -w -c build/config.js --environment TARGET:web-full-dev"
```

我们可以看到，当输入 `npm run dev` 之后，系统将使用 rollup.js，根据 `build/config.js` 下面的配置，将整个框架进行打包，并且还附带设置了环境变量 `process.env.TARGET = web-full-dev`。`web-full-dev` 这个环境变量有什么用，我们先记下来之后再用，让我们去 `build/config.js` 中看看其中的配置。

在 `build/config.js` 中，整体代码结构如下

```

// 记录的不同环境变量对应的获取配置目录
const builds = {
    ...
}

// 输入参数为 process.env.TARGET，输出为打包配置
function genConfig (name) {
    ...    
}

// 通过 process.env.TARGET 环境变量设置要输出的配置
if (process.env.TARGET) {
  module.exports = genConfig(process.env.TARGET)
} else {
  exports.getBuild = genConfig
  exports.getAllBuilds = () => Object.keys(builds).map(genConfig)
}
```

当我设置环境变量 `process.env.TARGET = web-full-dev`，则运行的代码为 `module.exports = genConfig('web-full-dev')`，然后在 `genConfig` 函数中，其对应的代码如下：

```
function genConfig (name) {
  // 根据 build['web-full-dev'] 获取 opt
  const opts = builds[name]
  // 根据 opt 生成最终的 config
  const config = {
    ...
  }

  if (opts.env) {
    config.plugins.push(replace({
      'process.env.NODE_ENV': JSON.stringify(opts.env)
    }))
  }

  Object.defineProperty(config, '_name', {
    enumerable: false,
    value: name
  })

  return config
}
```

其中的逻辑并不复杂：通过 `'web-full-dev'` 找到 `build` 对象中对应的配置，然后根据配置信息生成 `config` 并添加一些参数后，输出最终 `config` 对象，用于寻找打包文件。

我们通过 `build` 对象，找到所有的配置信息如下：

```
'web-full-dev': {
    entry: resolve('web/entry-runtime-with-compiler.js'),
    dest: resolve('dist/vue.js'),
    format: 'umd',
    env: 'development',
    alias: { he: './entity-decoder' },
    banner
  },
```

由此，我们可以找到，入口文件是 `src/platforms/web/entry-runtime-with-compiler.js`，然后在 `entry-runtime-with-compiler.js` 中，我们一层一层深入，通过 `import Vue from xxx` 指引的防线，找到 `Vue` 这个构造函数最终的位置在 `src/core/instance/index.js` 中。现在，我们开始，从 `instance/index.js` 里面向外看，打包给用户使用之前，`Vue` 中到底被添加了什么东西。






## 参考

* [Vue2.1.7源码学习](http://hcysun.me/2017/03/03/Vue%E6%BA%90%E7%A0%81%E5%AD%A6%E4%B9%A0/)





