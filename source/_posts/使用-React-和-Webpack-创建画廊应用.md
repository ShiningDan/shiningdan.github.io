---
title: 使用 React 和 Webpack 创建画廊应用
date: 2017-02-16 14:12:33
categories: coding
tags:
  - HTML
  - CSS
  - JavaScript
  - React
  - Webpack
---

本文是在学习 React 的时候，根据 [React实战--打造画廊应用](http://www.imooc.com/learn/507) 这篇教程来进行练习的Demo，详细文章请阅读原文。

<!--more-->

## 项目生成

这一次我们使用[Yeoman](http://yeoman.io) 来创建项目的框架。

### 安装 Yeoman

在此之前你应该已经安装了 node.js.

```
npm install -g yo
```

### 安装 react-webpack generator

针对不同的项目，使用不同的工具，可以下载并安装不同的 [Yeoman Generator](http://yeoman.io/generators/) 来创建项目的架构。

我们在项目中要使用的是 React + Webpack，所以在 Yeoman Generator 上，搜索 React，并且下载 [react-webpack generator](https://github.com/react-webpack-generators/generator-react-webpack#readme)。

```
npm install -g generator-react-webpack
```

### 创建项目

在 Github 上创建一个 Repository，叫 [react-gallery](https://github.com/ShiningDan/react-gallery)，并且使用 Git 在本地创建一个 react-gallery 文件夹。

```
git clone https://github.com/ShiningDan/react-gallery.git
```

上面 git clone 后面的链接为你自己创建好的项目。

进入创建好的项目文件夹，使用 Yeoman Generator 创建项目的框架。

```
cd react-gallery
yo react-webpack
```

然后 Yeoman 会提示你选择项目名称，项目要使用的 css 语言等。我就选用了最基本的 css。

![](http://ojt6zsxg2.bkt.clouddn.com/e064ad627d943fed6ed5af9fb43473b6.png)

然后回车，根据你的选项，react-webpack generator 安装需要的工具。

## 准备工作

### 创建项目

在 Github 上创建一个 Repository，叫 [react-gallery](https://github.com/ShiningDan/react-gallery)，并且使用 Git 在本地创建一个 react-gallery 文件夹。

```
git clone https://github.com/ShiningDan/react-gallery.git
```

上面 git clone 后面的链接为你自己创建好的项目。

### 安装 webpack

进入项目目录：

```
cd react-gallery
npm init
```

在此之前你应该已经安装了 node.js.

```
npm install webpack --save-dev
```

参数 `-g` 表示我们将全局(global)安装 webpack, 这样你就能使用 webpack 命令了.

webpack 也有一个 web 服务器 webpack-dev-server, 我们也安装上

```
npm install webpack-dev-server --save-dev
```

### webpack 配置文件

webpack 使用一个名为 webpack.config.js 的配置文件, 现在在你的项目根目录下创建该文件. 我们假设我们的工程有一个入口文件 `app.js`, 该文件位于 app/ 目录下, 并且希望 webpack 将它打包输出为 build/ 目录下的 `bundle.js` 文件. `webpack.config.js` 配置如下:

```
var path = require("path");

module.exports = {
	entry: path.resolve(__dirname, "app/app.js"),
	output: {
		path: path.resolve(__dirname, "build"),
		filename: "bundle.js"
	}
}
```

**注意，如果文件的路径不是在当前目录下，则需要使用 `path.resolve` 来创建文件路径**

现在让我们测试一下, 创建 `app/app.js`文件, 填入一下内容:

```
document.write("Hello React");
```

创建 index.html, 填入以下内容:

```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>React Webpack Gallery</title>
</head>
<body>
	<script src="build/bundle.js"></script>
</body>
</html>
```

其中 script 引入了 `build/bundle.js`, 这是 webpack 打包后的输出文件.

运行 `webpack` 打包, 运行 `webpack-dev-server` 启动服务器. 访问 `http://localhost:8080/index.html`, 如果一切顺利, 你会看到打印出了 Hello React.

### 配置 package.json

在 `package.json`, 修改 `scripts` 的键值如下:

```
"scripts": {
  "start": "webpack-dev-server --progress --colors  --watch",
  "build": "webpack"
}
```

注：`package.json`中的脚本部分已经默认在命令前添加了`node_modules/.bin`路径，所以无论是全局还是局部安装的Webpack，你都不需要写前面那指明详细的 script 中运行命令所对应的路径了

现在执行 `npm run build` 相当于 `webpack`, `npm run start` 相当于 `webpack-dev-server`. 当项目变得相当复杂时, 你可以使用这种技巧隐藏其中的细节.

### 安装依赖

安装 React:

```
npm install react react-dom --save-dev
```

安装 Babel 的 loader 以支持 ES6 语法:

```
npm install babel-core babel-loader babel-preset-es2015 babel-preset-react --save-dev
```

然后配置 `webpack.config.js` 来使用安装的 loader.

```
var path = require("path");

module.exports = {
	entry: path.resolve(__dirname, "app/app.js"),
	output: {
		path: path.resolve(__dirname, "build"),
		filename: "bundle.js"
	},
	module: {
		loaders: [
			{
				test: /\.jsx?$/,
				exclude: /node_modules/,
				loader: "babel-loader",
				query: {
					presets: ["es2015", "react"]
				}
			}
		],
	}
}
```

接下来测试一下开发环境是否搭建完成.

打开 app.js, 修改内容为:

```
import React from "react";
import ReactDOM from "react-dom";

class HelloReact extends React.Component {
	render() {
		return (
			<div>Hello World, Hello React!</div>
		);
	}
}

ReactDOM.render(<HelloReact />,
	document.getElementById("content")
);
```

这里组件 `HelloReact` 被装载到 `id` 为 `content` 的 DOM 元素, 所以相应的需要修改 `index.html` .

```
<body>
	<div id="content"></div>
	<script src="build/bundle.js"></script>
</body>
```

再次打包运行, 访问 `http://localhost:8080/index.html` 如果可以看到打印出了 Hello World 那么开发环境就算搭建完成了.

## 在写项目之前

关于散列画廊的制作方法，可以参考[慕课网 Lyn](http://www.imooc.com/learn/366) 的视频，这里不对实现的方法做过多的讲解。

在开始写第一个组件之前, 让我们分析一下到底我们需要哪些组件.

![](http://ofjjubwp5.bkt.clouddn.com/image/png/img70.PNG)

上图是的最终效果的部分截图, 我们将其分为几个组件, 用不同颜色的线框标出

![](http://ojt6zsxg2.bkt.clouddn.com/image/pngimg701.png)

我们可以看出其中包含以下几个组件.

* GallerryContainer(蓝色)：所有组件的容器
* GalleryPhoto(红色)：照片代表的组件
* GalleryNavbar(最下面)：对应每张照片的导航条外框
* GalleryNavIcon(每一个圆点)：每一个原点对应一张图片

把这些组成为一棵组件树.

* GalleryContainer
    * GalleryPhoto * n
    * GalleryNavbar
        * GalleryIcon * n

### 编写大致的模板

上一节我们知道了整个页面的组件结构, 在这一节我们根据这些结果编写出一个大致的模板. 先在 app 目录下为每个组件建立文件，`GalleryContainer.js GalleryPhoto.js GalleryNavbar.js GalleryIcon.js`

```
cd app
touch GalleryContainer.js GalleryPhoto.js GalleryNavbar.js GalleryIcon.js
```

编辑 `GalleryIcon.js`

```
import React from "react";

export default class GalleryIcon extends React.Component{
	render() {
		return (
			<div>I am GalleryIcon</div>
		);
	}
}
```

就先这样, 具体的实现现在先不去考虑, 记得我们现在只是编写一个大致的结构.

编辑`GalleryNavbar.js`

```
import React from "react";
import GalleryIcon from "./GalleryIcon.js"

export default class GalleryNavbar extends React.Component {
	render() {
		return (
			<div>
				I am GalleryNavbar
				<GalleryIcon />
				<GalleryIcon />
			</div>
		);
	}
}
```

因为 `GalleryNavbar` 是 `GalleryIcon` 的容器，所以需要引入它

编辑`GalleryPhoto.js`

```
import React from "react";

export default class GalleryPhoto extends React.Component {
	render() {
		return (
			<div>I am GalleryPhoto</div>
		);
	}
}
```

编辑`GalleryContainer.js`

因为`GalleryContainer` 是 `GalleryNavbar` 和 `GalleryPhoto` 的容器，所以也需要引入他们。

```
import React from "react";
import GalleryPhoto from "./GalleryPhoto.js";
import GalleryNavbar from "./GalleryNavbar.js"

export default class GalleryContainer extends React.Component {
	render() {
		return (
			<div>
				I am GalleryContainer
				<GalleryPhoto />
				<GalleryPhoto />
				<GalleryNavbar />
			</div>
		);
	}
}
```

最后修改入口文件 `app.js`:

```
import React from "react";
import ReactDOM from "react-dom";
import GalleryContainer from "./GalleryContainer.js";

ReactDOM.render(
	<GalleryContainer />,
	document.getElementById("content")
);
```

这时访问 http://localhost:8080/index.html 可以看到下图的效果

![](http://ojt6zsxg2.bkt.clouddn.com/3e5faac3197dd50dfe70ff314c31e4d6.png)

说明几个组件的创建都成功了，下面我们开始分别写各个组件。

### 编写 GalleryPhoto

整个 GalleryPhoto 包含了正面和背面，我们先从正面来写：

#### 照片正面效果

正面的效果如下：

![](http://ojt6zsxg2.bkt.clouddn.com/acc3a77537e172ed14e5ea47eeec402c.png)

重新编辑 `GalleryPhoto.js` 的代码:

首先在 `GalleryPhoto.js`里添加 `getFront()` 函数，返回正面的 div。

```
getFront() {
		return (
			<div className="side-front">
				<img src={photoImage} alt=""/>
				<p>第一张图片的描述</p>
			</div>
		);
	}
```


在前面几节的开发中, 还记得你是怎么引入其他的 js 文件的吗? import. 实际上这是 ES6 的模块系统, 这里的 js 文件作为模块被其他模块引入. 但除了 js 文件, 在开发时我们还会涉及其他的资源文件, 如图像, 字体, 样式等, 它们也需要被模块化. 在这里, 如果 Logo 图片也能被模块化然后引入该多好. 我们需要再次配置 Webpack.

安装对应的 loader:

```
npm install url-loader file-loader --save-dev
```

配置 `webpack.config.js`

```
//...
 loaders: [
		{
			test: /\.jsx?$/,
			exclude: /node_modules/,
			loader: "babel-loader",
			query: {
				presets: ["es2015", "react"]
			}
		},
		{
			test: /\.(png|jpg|gif)$/,
			loader: "url-loader?limit=8192" // 这里的 limit=8192 表示用 base64 编码 <= ８K 的图像
		},
		]
//...
```

这时使用 webpack 就可以将图像作为正确的模块导入了。

```
import React from "react";
import photoImage from "./image/1.jpg";

export default class GalleryPhoto extends React.Component {
	getFront() {
		return (
			<div className="side-front">
				<img src={photoImage} alt=""/>
				<p>第一张图片的描述</p>
			</div>
		);
	}

	render() {
		return (
			<div className="photo">
				{this.getFront()}
			</div>
		);
	}
}
```

最后我们需要添加样式, 还是回到刚刚的问题, 怎么引入样式?

我们也需要将样式模块化.

安装相应的 loader:

```
npm install css-loader style-loader --save-dev
```
`css-loader` 处理 css 文件中的 url() 表达式.

`style-loader` 将 css 代码插入页面中的 style 标签中.

在 `webpack.config.js` 中配置新的 loader.

```
{
	 test: /\.css$/,
	 loader: 'style-loader!css-loader'
},
```

新建一个 css 文件`app/style/GalleryPhoto.css`

```
.photo{width: 160px;height: 190px;position: absolute;box-shadow: 0 0 1px rgba(0, 0, 0, 0.01);transition: all 0.5s;}
.photo .side-front{padding: 10px;position: absolute;padding-bottom: 0px;background-color: #ddd;}
.photo .side-front img{width: 138px;height: 148px;display: block;margin: 0 auto;border: 1px solid #aaa;}
.photo .side-front p{display: block;height: 30px;line-height: 30px;text-align: center;color: #333;font-size: 12px;}

```

然后在 `GalleryPhoto.js` 中引用它：

```
import "./style/GalleryPhoto.css";
```

再建立一个全局的 css 文件 `app.css`

```
*{padding: 0; margin: 0;font-size: 14px;}
```

然后在 `app.js` 中引入

```
import "./style/app.css";
```

打包运行看看吧.

#### 照片背面效果

重新编辑 `GalleryPhoto.js` 的代码:

在 `GalleryPhoto.js`里添加 `getBack()` 函数，返回背面的 div。

```
getBack() {
		return (
			<div className="side-back">
				<p>这是背面的描述，这是背面的描述，这是背面的描述，这是背面的描述，这是背面的描述，这是背面的描述，这是背面的描述，
				</p>
			</div>
		);
	}
```

然后调用 `getBack()` 函数：

```
<div className="photo">
				{this.getFront()}
				{this.getBack()}
			</div>

```

为 `.side-back` 添加样式:

```
.photo .side-back{padding: 10px;position: absolute;width: 140px;height: 180px;overflow: hidden;background-color: #ddd;padding-bottom: 0px;}
```

可以打包运行看一看 `.side-back` 的结果。

#### 实现点击翻面效果

点击翻面效果的实现，其实就是 `.side-front` 和 `.side-back` 显示的交换.

为了方便实现照片的翻转，使用`.photo-wrap`包裹照片的正反两面。

```
getPhotoWrap() {
		return (
			<div className="photo-wrap">
				{this.getFront()}
				{this.getBack()}
			</div>
		);
	}
```

然后对 `.photo-wrap` 添加 CSS 样式，并且将 `.side-front` 放在正面，`.side-back` 放在背面：

```
.photo-wrap{-webkit-transform-style: preserve-3d;-webkit-backface-visibility: hidden;width: 100%;height: 100%;transition: all 0.6s;border: 1px solid #aaa;cursor: pointer;}
.photo-wrap .side-front{-webkit-transform: rotateY(0deg);}
.photo-wrap .side-back{-webkit-transform: rotateY(180deg);}
```

下面我们要做的是，如果点击 `.photo-wrap`，就将图片旋转 180 度.

我们将使用两个 class 来决定图片是正面朝上还是反面朝上。

`photo-f` 代表当前图片正面朝上，`.photo-b`代表当前图片反面朝上，并且在点击 `.photo-wrap` 的时候触发这两个 Class 之间的修改。

首先添加 state 代表这两个 class

```
constructor(props) {
		super(props);
		this.state = {
			photoFB: "photo-f",
		};
	}
```

注意：ES6 中不适用 getInitialState 来设置 state，而是在 constructor 中设置 state。

然后添加 `{photoFB}` 到 `className` 里面，并且添加 `onClick` 事件。

```
handleClick(event) {
		if (this.state.photoFB === "photo-f") {
			this.setState({
				photoFB: "photo-b",
			})
		}
		else
			this.setState({
				photoFB: "photo-f",
			});
	}
	getPhotoWrap() {
		return (
			<div className={"photo-wrap " + this.state.photoFB} onClick={this.handleClick.bind(this)}>
				{this.getFront()}
				{this.getBack()}
			</div>
		);
	}
```

最后添加对应`.photo-f`和 `.photo-b` 的 css 样式。

```
.photo-f.photo-wrap{transform: rotateY(0deg);}
.photo-b.photo-wrap{transform: rotateY(180deg);}
```

**注意，使用 `transform-style: preserve-3d;backface-visibility:hidden;` 等属性有可能会在不同的浏览器中导致图像闪烁无法旋转等情况出现。**

所以最后迫于无奈，只能用 `display:none` 来控制背面的显示，并且将`transform-style: preserve-3d;backface-visibility:hidden;`注释掉，以后如果没有闪烁的 bug，就可以将其反注释，并删除 `display:none`。

此时的CSS 样式为：

```
.photo{width: 160px;height: 190px;position: absolute;box-shadow: 0 0 1px rgba(0, 0, 0, 0.01);transition: all 0.5s;}
.photo .side-front{padding: 10px;position: absolute;padding-bottom: 0px;background-color: #ddd;}
.photo .side-front img{width: 138px;height: 148px;display: block;margin: 0 auto;border: 1px solid #aaa;}
.photo .side-front p{display: block;height: 30px;line-height: 30px;text-align: center;color: #333;font-size: 12px;}
.photo .side-back{padding: 10px;position: absolute;width: 140px;height: 180px;overflow: hidden;background-color: #ddd;padding-bottom: 0px;}
.photo-wrap{/*transform-style: preserve-3d; backface-visibility:hidden;*/width: 100%;height: 100%;transition: all 0.6s;border: 1px solid #aaa;cursor: pointer;}
.photo-wrap .side-front{transform: rotateY(0deg);}
.photo-wrap .side-back{transform: rotateY(180deg);}
.photo-f.photo-wrap{transform: rotateY(0deg);}
.photo-b.photo-wrap{transform: rotateY(180deg);}
.photo-f.photo-wrap .side-back{display: none;}
.photo-f.photo-wrap .side-front{display: block;}
.photo-b.photo-wrap .side-front{display: none;}
.photo-b.photo-wrap .side-back{display: block;}
```

### 编写 GalleryContainer

修改 `GalleryContainer.js`，返回 `div.container`

```
render() {
		return (
            <div className="container">
				<GalleryPhoto />
			</div>
		);
	}
```

并且创建 `GalleryContianer.css`

```
.container{width: 800px;height: 400px;margin: 100px auto;border: 1px solid rebeccapurple;overflow: hidden; perspective: 800px;-webkit-perspective:800px;background-color: #666}
```

在 `GalleryContainer.js` 导入 css 文件

```
import "./style/GalleryContainer.css";
```

#### 批量加载 GalleryPhoto



























