---
title: Webpack 源码解析
date: 2017-10-01 12:06:16
categories: coding
tags:
  - JavaScript
  - Webpack
---

随着前端渲染这种开发模式变得越来越广泛，对于前端代码的自动打包以及资源的整合的需求变得日益强烈，Webpack 作为自动打包工具的必要之选，除了提供必备的打包功能，还提供了多种官方组件以及开发者自己定义组件的能力，已经成了前端工程化中不可缺少的一部分。在本文中，笔者会对 Webpack 源代码进行分析，让我们了解 Webpack 的执行流程，也对一些常用的插件代码进行解析，在大家阅读完本文 后，可以模仿这些代码，编写自己的插件。

<!--more-->

在我们使用 Webpack 的时候，一般有两种使用方式，第一种是定义好 `webpack.config.js` 然后在命令行调用 `webpack` 来进行打包；第二种方式是使用自动化构建工具，如 `Gulp`、`Grunt` 等，通过代码编写构建流程。在使用第二种方法的时候，需要使用 `const webpack = require('webpack');` 来引入 `Weboack` 模块，再引入本地 `webpack.config.js` 文件 `const config = require('./webpack.config');`，最后在初始化的时候，将 `config` 作为 `webpack` 的初始化参数传入：`const compiler = webpack(webpackConfig);`

无论是第一种方式还是第二种方式，最后在生成 `webpack` 对象的时候都是统一的入口。在本次源码解析的时候，我们选择分析通过命令行的形式调用 `Webpack` 的方法，同时也学习一下 `Webpack` 对命令行解析的处理逻辑。

**本文的 Webpack 版本为 2.2.1**

## Webpack 命令行解析

在看其他的代码的时候，比较好的流程是先看 `package.json`，可以了解本项目的入口文件，依赖的模块等，我们可以看看本项目的 `package.json`，寻找相关的线索：

在 `package.json` 中，我们可以找到 `bin` 中对应的代码位置：

```
"bin": {
    "webpack": "./bin/webpack.js"
},
```

表示的是，在命令行中使用 `webpack` 配合一些参数来执行打包任务的时候，就会自动调用该文件中的代码，我们可以看一看这部分代码的内容。

这部分代码很长，我们分拆开来看看：

```
var path = require("path");

// Local version replace global one
try {
	var localWebpack = require.resolve(path.join(process.cwd(), "node_modules", "webpack", "bin", "webpack.js"));
	if(__filename !== localWebpack) {
		return require(localWebpack);
	}
} catch(e) {}
```

首先，查找本地版本的 Webpack，然后使用本地版本来代替全局版本。

```
var yargs = require("yargs")
	.usage("webpack " + require("../package.json").version + "\n" +
		"Usage: https://webpack.js.org/api/cli/\n" +
		"Usage without config file: webpack <entry> [<entry>] <output>\n" +
		"Usage with config file: webpack");
		
// 设置 webpack 可以接收的参数，以及 --help 查询时返回的内容
require("./config-yargs")(yargs);

// 还有一些 webpack 可以接受的参数定义在了下面
yargs.options({
	"json": {
		type: "boolean",
		alias: "j",
		describe: "Prints the result as JSON."
	},
	...
}
```

如何实现 Webpack 和命令行进行交互呢？在 Webpack 中使用的是 `yargs` 这个模块来解决如何处理命令行参数。

其中，在 `./config-yargs` 中，会设置 `webpack --help` 后在命令行中返回的提示信息，其中包括了所有的 Webpack 可以接受的参数。

在设置完 `webpack` 可以接受的参数以后，然后调用 `yargs.parse` 来处理用户在命令行的输入：

```
yargs.parse(process.argv.slice(2), (err, argv, output) => {
    ...
})
```

其中，我们来看一看 `yargs.parse` 接受的参数。当我们输入 `webpack --help` 的时候，我们在 `process.argv.slice(2)` 就可以获得 `--help` 这个参数，至于 `process.argv` 中有什么，我们可以打印一下：

```
[ '/Users/yuchen/.nvm/versions/node/v7.10.0/bin/node',
  '/usr/local/bin/webpack',
  '--help' ]
```

我们可以看到，在 `argv` 中的参数，第一个是 node 的应用程序位置，第二个是 `webpack.js` 的位置，从第三个位置起，就是我们在输入 `webpack` 的时候附带的参数。

在 `yargs.parse` 中，接收的第二个参数是一个回调函数：

```
(err, argv, output) => {
    ...
}
```

同样，打印出回调函数的各个参数，我们可以得到：

`err`：表示执行该交互命令解析时报的错，如传入的参数不存在等

`argv`：是一个对象，对象中的键值对表示用户输入的参数。如，当用户输入：`webpack --help` 的时候，`argv` 中的内容为：

```
{ _: [],
  help: true,
  h: true,
  version: false,
  v: false,
  ...
}
```

对应 `help` 和 `h` 都为 `true`。同理，如果 `webpack` 命令附带了其他的参数，都可以在 `argv` 中查到。

`output` 作为最后的参数，表示命令行最终显示给用户的结果。

在知道了输入的参数分别代表了什么以后，我们来看一看 `yargs.parse` 对参数的处理流程：

首先，对 `argv` 中的参数进行处理，在原来 `argv` 中的参数，是命令行形式的参数，现在将用户的输入，以及 `webpack.config.js` 中的配置合成在一起，作为用户最终需要的参数：

```
var options = require("./convert-argv")(yargs, argv);
```

我找了一个之前的项目，输入 `webpack --watch`，打印了一下 `options` 中的内容：

```
{ entry:
   [ 'babel-polyfill',
     '/Users/yuchenzhang/Documents/web/react-animation-resume/app/app.js' ],
  output:
   { path: '/Users/yuchenzhang/Documents/web/react-animation-resume/build',
     filename: 'bundle.js' },
  module: { loaders: [ [Object], [Object] ] },
  bail: false,
  profile: false,
  context: '/Users/yuchenzhang/Documents/web/react-animation-resume',
  watch: true }
```

我们可以看到，这里的参数，混合了 `webpack.config.js` 中的内容，也包含了 `watch: true` 的内容，表示最终的配置。

获得了 `options` 之后，接下来运行 `processOptions (options) ` 对 `opetions` 进行处理，我们可以看一下 `processOptions` 中的内容，对 `processOptions` 整体流程的解析，参考了这篇文章 [[webpack]源码解读：命令行输入webpack的时候都发生了什么？](https://zhuanlan.zhihu.com/p/24717349)。

```
function processOptions (options) {
 // 支持 Promise 风格的异步回调
  if ((typeof options.then) === "function") {...}

 // 处理传入一个 webpack 编译对象是数组时的情况
  var firstOptions = (Array.isArray (options)) ? options[0]: options;

 // 设置输出 options
  var outputOptions = options.stats;
  if(typeof outputOptions === "boolean" || typeof   outputOptions === "string") {
  	outputOptions = statsPresetToOptions(outputOptions);
  } else if(!outputOptions) {
  	outputOptions = {};
  }

  // 处理各种显示相关的参数，从略
  ifArg ("json", 
    function (bool){...}
  );
  ...

  // 引入主入口模块 lib/webpack.js
  var webpack = require ("../lib/webpack.js") ;

  // 设置错误堆栈追踪上限
  Error.stackTraceLimit = 30 ;
  var lastHash = null ;

 // 执行编译
  var compiler = webpack (options) ;

 // 创建编译结束后的回调函数
 function compilerCallback (err, stats) {...}

 // 是否在编译完成后继续 watch 文件变更，如果是需要 watch，则调用 compiler.watch 来执行编译
  if (firstOptions.watch || options.watch) {
    compiler.watch(watchOptions, compilerCallback);
  }
  else 
 // 执行编译后的回调函数
  compiler.run (compilerCallback) ;
}
```

我们可以看到，主要的流程是使用 `webpack`，通过传入 `opetions` 来得到 `compiler`。如果设置了 `watch`，就调用 `compiler.watch`，如果没有设置 `watch`，则调用 `compiler.run`。

下面，我们可以查看 `"../lib/webpack.js` 中 `webpack` 的定义，以及其中的流程处理。

## lib/webpack.js 处理流程

```
// lib/webpack.js

// 引入 Compiler 模块
var Compiler = require ("./Compiler") ;

// 引入 MultiCompiler 模块，处理多个 webpack 配置文件的情况
var MultiCompiler = require ("./MultiCompiler") ;

// 引入 node 环境插件
var NodeEnvironmentPlugin = require ("./node/NodeEnvironmentPlugin") ;

// 引入 WebpackOptionsApply 模块，应用 webpack 配置文件
var WebpackOptionsApply = require ("./WebpackOptionsApply") ;

// 引入 WebpackOptionsDefaulter 模块，应用 webpack 默认配置
var WebpackOptionsDefaulter = require ("./WebpackOptionsDefaulter") ;

// 通过 ValidateSchema 来对传入的 options 进行检测，判断 options 中的参数是否符合规范
const validateSchema = require("./validateSchema");

const WebpackOptionsValidationError = require("./WebpackOptionsValidationError");

const webpackOptionsSchema = require("../schemas/webpackOptionsSchema.json");

// 核心函数，也是 ./bin/webpack.js 中引用的核心方法
function webpack (options, callback) {...}
exports = module.exports = webpack ;

// 在 webpack 对象上设置一些常用属性
webpack.WebpackOptionsDefaulter = WebpackOptionsDefaulter ;
webpack.WebpackOptionsApply = WebpackOptionsApply ;
webpack.Compiler = Compiler ;
webpack.MultiCompiler = MultiCompiler ;
webpack.NodeEnvironmentPlugin = NodeEnvironmentPlugin ;
webpack.validate = validateSchema.bind(this, webpackOptionsSchema);
webpack.validateSchema = validateSchema;
webpack.WebpackOptionsValidationError = WebpackOptionsValidationError;

// 暴露一些插件
function exportPlugins(obj, mappings) {...}
exportPlugins (exports, {...}) ;
exportPlugins (exports.optimize = {}, {...})
```

在 `lib/webpack.js` 中，做了以下事情：

1. 定义了对外提供的 `webpack` 函数
2. 在 `webpack` 对象上设置了一些常用属性和插件，这些插件也是我们在对打包过程进行个性化处理的时候经常使用的插件

首先，我们来分析一下对外提供的 `webpack` 函数中的内容。

### webpack 函数分析

我们来看一下 `webpack` 函数中的内容：

```
function webpack(options, callback) {

    // 对传入的 options 中的内容进行分析，判断传入的 options 内容是否符合规范
	const webpackOptionsValidationErrors = validateSchema(webpackOptionsSchema, options);
	if(webpackOptionsValidationErrors.length) {
		throw new WebpackOptionsValidationError(webpackOptionsValidationErrors);
	}
	
	let compiler;
	// 如果传入了数组类型的 webpack 编译对象，则实例化一个 MultiCompiler 来处理
	if(Array.isArray(options)) {
		compiler = new MultiCompiler(options.map(options => webpack(options)));  // 递归调用 webpack 函数
	} else if(typeof options === "object") {
	    // 如果传入了一个对象类型的 webpack 编译对象
  
        // 实例化一个 WebpackOptionsDefaulter 来处理默认配置项
		new WebpackOptionsDefaulter().process(options);

        // 实例化一个 Compiler
		compiler = new Compiler();
		compiler.context = options.context;
		compiler.options = options;
		// 配置 node 环境插件，如 inputFileSystem/outputFileSystem/watchFileSystem 等
		new NodeEnvironmentPlugin().apply(compiler);
		
		if(options.plugins && Array.isArray(options.plugins)) {
		    // 配置 options 中的 plugins
			compiler.apply.apply(compiler, options.plugins);
		}
		
		compiler.applyPlugins("environment");
		compiler.applyPlugins("after-environment");
		// 针对我们传进去的 webpack 编译对象进行逐一编译
		compiler.options = new WebpackOptionsApply().process(options, compiler);
	} else {
		throw new Error("Invalid argument: options");
	}
	
	// 如果给 webpack 对象传递了回调函数，则设置 webpack 回调函数的执行
	if(callback) {
		if(typeof callback !== "function") throw new Error("Invalid argument: callback");
		if(options.watch === true || (Array.isArray(options) && options.some(o => o.watch))) {
			const watchOptions = Array.isArray(options) ? options.map(o => o.watchOptions || {}) : (options.watchOptions || {});
			// 如果配置了 watch 选项，则调用 compilers.watch 来在每一次改动以及编译过后触发 callback
			return compiler.watch(watchOptions, callback);
		}
		// 如果没有设置 watch，则执行一次 callback
		compiler.run(callback);
	}
	return compiler;
}
exports = module.exports = webpack;
```

其中 `class Compiler extends Tapable` 说明 `Compiler` 继承自 `Tapable`，`Tapable` 是一个小型库,能够让我们为javascript模块添加并应用插件。可以参考 [Webpacl 中 Tapable 的介绍](https://doc.webpack-china.org/api/plugins/tapable/)

在这里的重点，是 `WebpackOptionsApply().process(options, compiler)` 这一句，通过 `WebpackOptionsApply` 来逐个编译 `webpack` 编译对象，下面，我们来看看 `WebpackOptionsApply` 的处理流程

### lib/WebpackOptionsApply.js 处理流程

本文中，处理流程分析参考 [[webpack]源码解读：命令行输入webpack的时候都发生了什么？](https://zhuanlan.zhihu.com/p/24717349)

在这里的调用方法是 `WebpackOptionsApply().process(options, compiler)`，所以，我们主要关心 `process` 方法的流程：

```
process(options, compiler) {
    // 缓存输入输出的目录地址等
    compiler.outputPath = options.output.path;
    compiler.recordsInputPath = options.recordsInputPath || options.recordsPath;
    compiler.recordsOutputPath = options.recordsOutputPath || options.recordsPath;
    compiler.name = options.name;
    compiler.dependencies = options.dependencies;
    
    // 处理 target 属性，该属性决定包 (bundle) 应该运行的环境，如 web/webworker/node/electron 等
  if ((typeof options.target) === "string") {...}
  
  // 处理 output.library 属性，该属性决定导出库 (exported library) 的名称
  if (options.output.library || (options.output.libraryTarget !== "var")) {...}
 
  // 处理 externals 属性，告诉 webpack 不要遵循/打包这些模块，而是在运行时从环境中请求他们
  if (options.externals) {...}
 
 // 处理 devtool 属性，它决定了 webpack 的 sourceMap 模式
  if (options.devtool && (((options.devtool.indexOf ("sourcemap")) >= 0) || ((options.devtool.indexOf ("source-map")) >= 0))) {...}
  else if (options.devtool && ((options.devtool.indexOf ("eval")) >= 0)) {...}


// 以下是安装并调用各种插件 plugin，由于功能众多个人阅历有限，不能面面俱到

  compiler.apply (new EntryOptionPlugin ()) ; // 调用处理入口 entry 的插件
  compiler.applyPluginsBailResult ("entry-option", options.context, options.entry) ;
 
  compiler.apply (new CompatibilityPlugin (),
                  new LoaderPlugin (), // 调用 loader 的插件
                  new NodeStuffPlugin (options.node), // 调用 nodejs 环境相关的插件
                  new RequireJsStuffPlugin (), // 调用 RequireJs 的插件
                  new APIPlugin (), // 调用变量名的替换，webpack 编译后的文件里随处可见的 __webpack_require__ 变量名就是在此处理
                  new ConstPlugin (), // 调用一些 if 条件语句、三元运算符等语法相关的插件
                  new RequireIncludePlugin (), // 调用 require.include 函数的插件
                  new RequireEnsurePlugin (), // 调用 require.ensure 函数的插件
                  new RequireContextPlugin(options.resolve.modulesDirectories, options.resolve.extensions),
                  new AMDPlugin (options.module, options.amd || ({})), // 调用处理符合 AMD 规范的插件
                  new CommonJsPlugin (options.module)) ; // 调用处理符合 CommonJs 规范的插件

  compiler.apply (new RemoveParentModulesPlugin (), // 调用移除父 Modules 的插件
                  new RemoveEmptyChunksPlugin (), // 调用移除空 chunk 的插件
                  new MergeDuplicateChunksPlugin (), // 调用合并重复多余 chunk 的插件
                  new FlagIncludedChunksPlugin ()) ; // 

  compiler.apply (new TemplatedPathPlugin ()) ;
  compiler.apply (new RecordIdsPlugin ()) ; // 调用记录 Modules 的 Id 的插件
  compiler.apply (new WarnCaseSensitiveModulesPlugin ()) ; // 调用警告大小写敏感的插件
    
}
```

我们可以在上面的代码中看到 webpack 文档中 [Configuration](http://webpack.github.io/docs/configuration.html) 中介绍的各个属性，同时看到了这些属性对应的处理插件都是谁。

## UglifyJsPlugin 处理流程

plugin是一个具有 `apply` 方法的 js对象。 `apply` 方法会被 Webpack的 `compiler`（编译器）对象调用，并且 `compiler` 对象可在整个 `compilation`（编译）生命周期内访问。

webpack插件的组成：

1. 一个JavaScript函数或者class（ES6语法）。
2. 在它的原型上定义一个apply方法。
3. 指定挂载的webpack事件钩子。
4. 处理webpack内部实例的特定数据。
5. 功能完成后调用webpack提供的回调。



我们可以通过 `UglifyJsPlugin.js` 来理解 Webpack 的处理流程，具体解释可以参考 [[webpack]源码解读：命令行输入webpack的时候都发生了什么？](https://zhuanlan.zhihu.com/p/24717349)


```
// 引入一些依赖，主要是与 sourceMap 相关
var SourceMapConsumer = require("source-map").SourceMapConsumer;
var SourceMapSource = require("webpack-sources").SourceMapSource;
var RawSource = require("webpack-sources").RawSource;
var ConcatSource = require("webpack-sources").ConcatSource;
var RequestShortener = require("webpack/lib/RequestShortener");
var ModuleFilenameHelpers = require("webpack/lib/ModuleFilenameHelpers");
var uglify = require("uglify-js");

// 通用的构造函数创建方法
var _createClass = function () { 
  function defineProperties(target, props) { 
    for (var i = 0; i < props.length; i++) { 
      var descriptor = props[i]; 
      descriptor.enumerable = descriptor.enumerable || false; 
      descriptor.configurable = true; 
      if ("value" in descriptor) 
        descriptor.writable = true; 
      Object.defineProperty(target, descriptor.key, descriptor); 
    } 
  } 
  return function (Constructor, protoProps, staticProps) { 
    if (protoProps) 
      defineProperties(Constructor.prototype, protoProps); 
    if (staticProps) 
      defineProperties(Constructor, staticProps); 
    return Constructor; 
  }; 
}();


//按照 Tapable 风格编写插件
_createClass(UglifyJsPlugin, [{
	key: "apply",
	value: function apply(compiler) {
		var options = this.options;
		...
		// 编译器开始编译
		compiler.plugin("compilation", function (compilation) {
			...
			 // 编译器开始调用 "optimize-chunk-assets" 插件编译
			compilation.plugin("optimize-chunk-assets", function (chunks, callback) {
				var files = [];
				...
				filteredFiles.forEach(function (file) {
				...
					try {
						var asset = compilation.assets[file];
						if (asset.__UglifyJsPlugin) {
							compilation.assets[file] = asset.__UglifyJsPlugin;
							return;
						}
						var input = void 0;
						var inputSourceMap = void 0;
						if (options.sourceMap) {
							// 需要 sourceMap 时要做的一些操作...
						} else {
                            // 获取读取到的源文件
							input = asset.source();
							...
							};
						}
						// base54 编码重置
						uglify.base54.reset();
						// 将源文件生成语法树
						var ast = uglify.parse(input, {
							filename: file
						});
						// 语法树转换为压缩后的代码
						if (options.compress !== false) {
							ast.figure_out_scope();
							var compress = uglify.Compressor(options.compress || {
								warnings: false
							}); // eslint-disable-line new-cap
							ast = compress.compress(ast);
						}
						// 处理混淆变量名
						if (options.mangle !== false) {
							ast.figure_out_scope(options.mangle || {});
							ast.compute_char_frequency(options.mangle || {});
							ast.mangle_names(options.mangle || {});
							if (options.mangle && options.mangle.props) {
								uglify.mangle_properties(ast, options.mangle.props);
							}
						}
						// 定义输出变量名
						var output = {};
						// 处理输出的注释
						output.comments = Object.prototype.hasOwnProperty.call(options, "comments") ? options.comments : /^\**!|@preserve|@license/;
						// 处理输出的美化
						output.beautify = options.beautify;
						for (var k in options.output) {
							output[k] = options.output[k];
						}
						...
						var map = void 0;
						// 处理输出的 sourceMap
						if (options.sourceMap) {
							map = uglify.SourceMap({ // eslint-disable-line new-cap
								file: file,
								root: ""
							});
							output.source_map = map; // eslint-disable-line camelcase
						}
						// 将压缩后的数据输出
						var stream = uglify.OutputStream(output); // eslint-disable-line new-cap
						ast.print(stream);
						if (map) map = map + "";
						var stringifiedStream = stream + "";
						var outputSource = map ? new SourceMapSource(stringifiedStream, file, JSON.parse(map), input, inputSourceMap) : new RawSource(stringifiedStream);
						
						asset.__UglifyJsPlugin = compilation.assets[file] = outputSource;
						if (warnings.length > 0) {
							compilation.warnings.push(new Error(file + " from UglifyJs\n" + warnings.join("\n")));
						}
					} catch (err) {
					    // 处理异常
					    ...
					} finally {
						...
					}
				});
				callback();
			});
		});
	}
}]);
```

从这个插件的源码分析，我们可以基本看到 webpack 编译时的读写过程大致是怎么样的：实例化插件（如 UglifyJsPlugin ）--> 读取源文件 --> 编译并输出


下面是非常简洁的代码逻辑：

```
class UglifyJsPlugin {
	apply(compiler) {
		const options = this.options;
		options.test = options.test || /\.js($|\?)/i;
		
		......
		
		//绑定compilation事件
		compiler.plugin("compilation", (compilation) => {
			if(options.sourceMap) {
				compilation.plugin("build-module", (module) => {
					// to get detailed location info about errors
					module.useSourceMap = true;
				});
			}
			//绑定optimize-chunk-assets事件
			compilation.plugin("optimize-chunk-assets", (chunks, callback) => {
				const files = [];
				chunks.forEach((chunk) => files.push.apply(files, chunk.files));
            
            ......
            
				callback();
			});
		});
	}
}
module.exports = UglifyJsPlugin;
```

在webpack插件开发中最重要的两个核心概念就是 compiler 和 compilation 。理解他们是扩展webpack功能的关键。

[webpack之plugin内部运行机制](https://fengmiaosen.github.io/2017/03/21/webpack-core-code/) 这篇文章中有 compiler 和 compilation 的详细介绍。

·`compiler` 对象代表的是配置完备的Webpack环境。 `compiler` 对象只在Webpack启动时构建一次，由Webpack组合所有的配置项构建生成。

`compilation` 对象代表了一次单一的版本构建和生成资源。当运行 webpack 开发环境中间件时，每当检测到一个文件变化，一次新的编译将被创建，从而生成一组新的编译资源。一个编译对象表现了当前的模块资源、编译生成资源、变化的文件、以及被跟踪依赖的状态信息。编译对象也提供了很多关键点事件回调供插件做自定义处理时选择使用。



## 总结

现在我们回过头来再看看整体流程，当我们在命令行输入 webpack 命令，按下回车时都发生了什么： 

1. 执行 bin 目录下的 webpack.js 脚本，解析命令行参数以及开始执行编译。 
2. 调用 lib 目录下的 webpack.js 文件的核心函数 webpack ，实例化一个 Compiler，继承 Tapable 插件框架，实现注册和调用一系列插件。 
3. 调用 lib 目录下的 /WebpackOptionsApply.js 模块的 process 方法，使用各种各样的插件来逐一编译 webpack 编译对象的各项。
4. 在3中调用的各种插件编译并输出新文件。

## 参考

* [[webpack]源码解读：命令行输入webpack的时候都发生了什么？](https://zhuanlan.zhihu.com/p/24717349)
* [webpack之plugin内部运行机制](https://fengmiaosen.github.io/2017/03/21/webpack-core-code/)