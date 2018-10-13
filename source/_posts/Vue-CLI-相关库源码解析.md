---
title: Vue CLI 相关库源码解析
date: 2018-09-01 15:20:13
categories: coding
tags:
  - JavaScript
  - Vue
  - Vue CLI
  - Webpack
---

本文是学习 Vue CLI v3 源码学习的笔记

## 总结 Vue CLI 的相关库

关于 Vue CLI 的相关库的总结，可以参考上一篇博客 [Vue CLI 学习](https://shiningdan.github.io/2018/08/18/Vue-CLI-%E5%AD%A6%E4%B9%A0/)

在本文中，会对相关的库的源码以及实现原理进行分析。

在本文中会涉及到的库有：

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

## 概述

首先，所有的项目代码在 Github [vuejs/vue-cli](https://github.com/vuejs/vue-cli) 这个项目下。

这个项目使用的是 monorepo 风格的项目管理模式，除了 `@vue/cli` 以外，所有 @vue 的官方插件都可以在 `package/` 目录下找到。关于 monorepo 的介绍，以及 monorepo 与 multirepos 的区别，可以参考 [monorepo 新浪潮 | introduce lerna](https://github.com/pigcan/blog/issues/3) 这篇文章。

Monorepo 它是一种管理 organisation 代码的方式，在这种方式下会摒弃原先一个 module 一个 repo 的方式，取而代之的是把所有的 modules 都放在一个 repo 内来管理。目前诸如 Babel, React, Angular, Ember, Meteor, Jest 等等都采用了 Monorepo 这种方式来进行源码的管理。

配合 Monorepo 理念的工具（例如 lerna），可以进行多个 repo 的统一发版，互相依赖引用，互相测试，自动生成 changelog 之类的能力。

项目运行的方法可以参考 [vue-cli/.github/CONTRIBUTING.md](https://github.com/vuejs/vue-cli/blob/dev/.github/CONTRIBUTING.md)

在 vue-cli 这个项目中，所有的 repo 的代码在 `package/` 目录下，我们先从项目生成的入口 `@vue/cli` 来开始分析

## @vue/cli

### package.json

```
"bin": {
	"vue": "bin/vue.js"
},
```

提供了 vue 命令

### bin/vue.js

#### 验证版本号

```
const semver = require('semver')
//"engines": {
//  "node": ">=8.9"
//}
const requiredVersion = require('../package.json').engines.node

function checkNodeVersion (wanted, id) {
  if (!semver.satisfies(process.version, wanted)) {
    console.log(chalk.red(
      'You are using Node ' + process.version + ', but this version of ' + id +
      ' requires Node ' + wanted + '.\nPlease upgrade your Node version.'
    ))
    process.exit(1)
  }
}

checkNodeVersion(requiredVersion, 'vue-cli')
```

通过 `semver` 来验证用户本地的 node 版本是否大于等于 8.9

```
版本号格式：X.Y.Z：
（主版本号.次版本号.修订号）修复问题但不影响API 时，递增修订号；API 保持向下兼容的新增及修改时，递增次版本号；进行不向下兼容的修改时，递增主版本号。
```

#### 判断是否进入 vue-cli debug 模式

通过判断项目是否存在某些文件夹来判断用户是开发 vue-cli，还是在使用 vue-cli。因为 `@vue/cli` 是全局安装的，所以使用 `@vue/cli` 没有上门的文件夹。

```
// enter debug mode when creating test repo
if (
  slash(process.cwd()).indexOf('/packages/test') > 0 && (
    fs.existsSync(path.resolve(process.cwd(), '../@vue')) ||
    fs.existsSync(path.resolve(process.cwd(), '../../@vue'))
  )
) {
  process.env.VUE_CLI_DEBUG = true
}
```

```
slash：
Convert Windows backslash paths to slash paths: foo\\bar ➔ foo/bar
```

如果用户是开发 vue-cli，在使用 `vue create app` 创建你项目的时候，设置的 VUE_CLI_DEBUG 全局变量会在调试的时候打印调试信息。这个环境变量在一下的地方被用到：

![](http://ojt6zsxg2.bkt.clouddn.com/d573d8a8e38a3e9ed34986ddc6794408.png) 

#### 定义 shell 命令

使用的是 [commander](https://www.npmjs.com/package/commander) 这个库来定义 shell 命令

```
const program = require('commander')
const loadCommand = require('../lib/util/loadCommand')

program
  .version(require('../package').version)
  .usage('<command> [options]')
```

使用 `command`，并且添加 `option`

我们先来看一下 `create` 这个 Command 背后的命令

### `vue create`

下面是 `vue create` 的命令的定义

```
program
  .command('create <app-name>')
  .description('create a new project powered by vue-cli-service')
  .option('-p, --preset <presetName>', 'Skip prompts and use saved or remote preset')
  .option('-d, --default', 'Skip prompts and use default preset')
  .option('-i, --inlinePreset <json>', 'Skip prompts and use inline JSON string as preset')
  .option('-m, --packageManager <command>', 'Use specified npm client when installing dependencies')
  .option('-r, --registry <url>', 'Use specified npm registry when installing dependencies (only for npm)')
  .option('-g, --git [message]', 'Force git initialization with initial commit message')
  .option('-n, --no-git', 'Skip git initialization')
  .option('-f, --force', 'Overwrite target directory if it exists')
  .option('-c, --clone', 'Use git clone when fetching remote preset')
  .option('-x, --proxy', 'Use specified proxy when creating project')
  .option('-b, --bare', 'Scaffold project without beginner instructions')
  .action((name, cmd) => {
    const options = cleanArgs(cmd)
    // --no-git makes commander to default git to true
    if (process.argv.includes('-g') || process.argv.includes('--git')) {
      options.forceGit = true
    }
    require('../lib/create')(name, options)
  })
```

在 `action` 方法中，传入两个参数：
1. 第一个参数是 `name`，也就是 `command` 里面定义，用户输入的 `app-name`
2. 第二个参数是 `cmd`，是 `command` 这个库提供的一个对象，用来存储用户 option 的输入

如用户运行 `vue create app-test --default --force`，则被解析出来的 `name` 和 `cmd` 为：

```
name: app-test
cmd: {
  // option defines
  optisons: {
    ...
  },
  
  ...
  
  default: true,
  force: true
}
```

在知道 `cmd` 变量的格式之后，我们就知道 `action` 方法中如何通过 `cleanArgs` 将用户传入的 `options` 解析出来：

```
// commander passes the Command object itself as options,
// extract only actual options into a fresh object.
function cleanArgs (cmd) {
  const args = {}
  cmd.options.forEach(o => {
    const key = o.long.replace(/^--/, '')
    // if an option is not present and Command has a method with the same name
    // it should not be copied
    if (typeof cmd[key] !== 'function' && typeof cmd[key] !== 'undefined') {
      args[key] = cmd[key]
    }
  })
  return args
}
```

首先，通过 `cmd.options` 拿到所有用户可以提供的 `options`，然后对每一个 `option` 进行遍历，判断用户是否提供该参数，最后将所有用户提供的参数返回。如 `vue create app-test --default --force`，最后被 `cleanArgs` 解析出来的结果就是：

```
options: { 
  default: true, 
  git: true, 
  force: true 
}
```

在命令定义的最后，拿到了 `name` 和 `options`，作为参数传递给 `lib/create.js`

```
require('../lib/create')(name, options)
```

#### `lib/create.js`

##### 初始化项目名和路径
 
在 `create.js` 中，首先通过传入的 `name`，以及当前的目录，得到创建项目的一些信息：

```
const cwd = options.cwd || process.cwd()
const inCurrent = projectName === '.'
const name = inCurrent ? path.relative('../', cwd) : projectName
const targetDir = path.resolve(cwd, projectName || '.')
```

在使用 `vue create app-test --default --force` 后，这些变量分别被赋值为：

```
/Users/shiningdan/Documents/project
false
app-test
/Users/shiningdan/Documents/project/app-test
```

从上面的代码中，我们还可以看到，`vue create` 还可以在一个已经创建好的目录下创建项目，不过目录名需要提供 `.`。如，我们已经创建好 `app-test` 目录，然后运行 `vue create . --default --force`，这些变量被初始化的值为：

```
/Users/shiningdan/Documents/project/app-test
true
app-test
/Users/shiningdan/Documents/project/app-test
```

##### 判断项目名是否符合规范

```
const validateProjectName = require('validate-npm-package-name')

const result = validateProjectName(name)
if (!result.validForNewPackages) {
  console.error(chalk.red(`Invalid project name: "${projectName}"`))
  result.errors && result.errors.forEach(err => {
    console.error(chalk.red(err))
  })
  exit(1)
}
```

直接调用 `validate-npm-package-name` 这个库

##### 项目目录已经存在的交互处理

```
const { clearConsole } = require('./util/clearConsole')
const inquirer = require('inquirer')

if (fs.existsSync(targetDir)) {
  if (options.force) {
    await fs.remove(targetDir)
  } else {
    await clearConsole()
    if (inCurrent) {
      const { ok } = await inquirer.prompt([
        {
          name: 'ok',
          type: 'confirm',
          message: `Generate project in current directory?`
        }
      ])
      if (!ok) {
        return
      }
    } else {
      const { action } = await inquirer.prompt([
        {
          name: 'action',
          type: 'list',
          message: `Target directory ${chalk.cyan(targetDir)} already exists. Pick an action:`,
          choices: [
            { name: 'Overwrite', value: 'overwrite' },
            { name: 'Merge', value: 'merge' },
            { name: 'Cancel', value: false }
          ]
        }
      ])
      if (!action) {
        return
      } else if (action === 'overwrite') {
        console.log(`\nRemoving ${chalk.cyan(targetDir)}...`)
        await fs.remove(targetDir)
      }
    }
  }
}
```

如果项目已经存在了，首先使用 `clearConsole` 清空 console 里面的内容，然后用 `inquirer` 这个库提供和用户的交互，是否覆盖当前路径或使用其他的处理方法。

其中 `clearConsole` 是通过 `@vue/cli-shared-utils` 引入的库，这个库在当前项目的 `packages/` 下面也可以找到，最终对应的代码为： 

```
// node 内置的模块
const readline = require('readline')

exports.clearConsole = title => {
  if (process.stdout.isTTY) {
    const blank = '\n'.repeat(process.stdout.rows)
    console.log(blank)
    readline.cursorTo(process.stdout, 0, 0)
    readline.clearScreenDown(process.stdout)
    if (title) {
      console.log(title)
    }
  }
}
```

其原理是，通过输入当前 Terminal 行数的回车，将用户之前的输入顶出当前页面，然后将光标移动到 (0, 0) 的位置，最后删除输出流余下所有的内容，实现清空输入流的能力。

#### 调用 Creator 构造函数

在确定了项目名和项目的绝对路径后，`create.js` 会调用 `Creator` 构造函数来进行项目初始化：

```
const Creator = require('./Creator')
const { getPromptModules } = require('./util/createTools')

const creator = new Creator(name, targetDir, getPromptModules())
await creator.create(options)
```

在这里面，`name` 和 `targetDir` 都是我们知道的参数，那 `getPromptModules()` 是什么呢？下面来看看 `getPromptModules()` 的内容：

```
exports.getPromptModules = () => {
  return [
    'babel',
    'typescript',
    'pwa',
    'router',
    'vuex',
    'cssPreprocessors',
    'linter',
    'unit',
    'e2e'
  ].map(file => require(`../promptModules/${file}`))
}
```

我们可以看到，`getPromptModules ` 返回的是 `../promptModules/` 下面对应的配置文件，每一个配置文件都是我们在生成项目的时候的选项，以及一些自定义的交互操作。之后如果我们需要自定义一些原子化的模块，可以在这个文件下面添加。

至于每一个原子模块的内容，我们在 `Creator.js` 的流程中再分析。

我们去看一下 `Creator.js` 这个构造函数

```
module.exports = class Creator extends EventEmitter {
  constructor (name, context, promptModules) {
    super()

    this.name = name
    // context is targetDir
    this.context = process.env.VUE_CLI_CONTEXT = context
    const { presetPrompt, featurePrompt } = this.resolveIntroPrompts()
    this.presetPrompt = presetPrompt
    this.featurePrompt = featurePrompt
    this.outroPrompts = this.resolveOutroPrompts()
    this.injectedPrompts = []
    this.promptCompleteCbs = []
    this.createCompleteCbs = []

    this.run = this.run.bind(this)

    const promptAPI = new PromptModuleAPI(this)
    promptModules.forEach(m => m(promptAPI))
  }
  
  ...
  
}
```

##### resolveIntroPrompts

下面分析 `resolveIntroPrompts` 的内容

```
resolveIntroPrompts () {
  const presets = this.getPresets()
  const presetChoices = Object.keys(presets).map(name => {
    return {
      name: `${name} (${formatFeatures(presets[name])})`,
      value: name
    }
  })
  const presetPrompt = {
    name: 'preset',
    type: 'list',
    message: `Please pick a preset:`,
    choices: [
      ...presetChoices,
      {
        name: 'Manually select features',
        value: '__manual__'
      }
    ]
  }
  const featurePrompt = {
    name: 'features',
    when: isManualMode,
    type: 'checkbox',
    message: 'Check the features needed for your project:',
    choices: [],
    pageSize: 10
  }
  return {
    presetPrompt,
    featurePrompt
  }
}
```

首先是 `getPresets`

```
getPresets () {
  const savedOptions = loadOptions()
  return Object.assign({}, savedOptions.presets, defaults.presets)
}
```

在 `getPresets` 中，首先会去拿用户在之前保存过的选项。

用户之前保存过的生成项目的选项，在 `~/.vuerc` 文件中可以找到，具体的文档可以参考 [vue create | vue-cli](https://cli.vuejs.org/zh/guide/creating-a-project.html#vue-create) 

所以在 `loadOptions` 中，会去该文件里拿对应的配置，如果用户之前没有保存过该文件，则找不到  `~/.vuerc` 这个文件，此时会返回 `{}`：

```
exports.loadOptions = () => {
  if (cachedOptions) {
    return cachedOptions
  }
  if (fs.existsSync(rcPath)) {
    try {
      cachedOptions = JSON.parse(fs.readFileSync(rcPath, 'utf-8'))
    } catch (e) {
      ...
      exit(1)
    }
    validate(cachedOptions, schema, () => {
      error(
        `~/.vuerc may be outdated. ` +
        `Please delete it and re-run vue-cli in manual mode.`
      )
    })
    return cachedOptions
  } else {
    return {}
  }
}
```

在这里 `schema` 的生成和 `validate` 的验证，使用的是 `joi` 库。

回到 `getPresets` 这里，由于我本地没有定义 Presets，所以 `loadOptions` 返回值是 `{}`，之后再 merge `defaults.presets`，就是 `getPresets` 的返回值。

```
exports.defaults = {
  packageManager: undefined,
  useTaobaoRegistry: undefined,
  presets: {
    'default': {
	  router: false,
	  vuex: false,
	  useConfigFiles: false,
	  cssPreprocessor: undefined,
	  plugins: {
	    '@vue/cli-plugin-babel': {},
	    '@vue/cli-plugin-eslint': {
	      config: 'base',
	      lintOn: ['save']
	    }
	  }
	}
  }
}
```

即，在默认配置中，添加的 Plugins 有 babel 和 eslint。 

最终，

```
this.presetPrompt = presetPrompt
this.featurePrompt = featurePrompt
```

的值分别是：

```

```


下面来看一下

```
this.outroPrompts = this.resolveOutroPrompts()
```



