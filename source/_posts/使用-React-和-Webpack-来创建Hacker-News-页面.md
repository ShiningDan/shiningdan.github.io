---
title: 使用 React 和 Webpack 来创建Hacker News 页面
date: 2017-02-05 16:50:27
categories: coding
tags:
  - HTML
  - CSS
  - JavaScript
  - React
  - Webpack
---

本文是在学习 React 的时候，根据 [Learn React & Webpack by building the Hacker News front page](https://github.com/theJian/build-a-hn-front-page#learn-react--webpack-by-building-the-hacker-news-front-page) 这篇教程来进行联系的记录，详细文章请阅读原文。

## 准备工作

### 安装 Webpack

在此之前你应该已经安装了 node.js.

```
npm install webpack --save-dev
```

参数 `-g` 表示我们将全局(global)安装 webpack, 这样你就能使用 webpack 命令了.

webpack 也有一个 web 服务器 webpack-dev-server, 我们也安装上

```
npm install webpack-dev-server --save-dev
```

<!--more-->

### webpack 配置文件

webpack 使用一个名为 webpack.config.js 的配置文件, 现在在你的项目根目录下创建该文件. 我们假设我们的工程有一个入口文件 app.js, 该文件位于 `app/` 目录下, 并且希望 webpack 将它打包输出为 `build/` 目录下的 bundle.js 文件. webpack.config.js 配置如下:

```
var path = require('path');

module.exports = {
	entry: path.resolve(__dirname, "app/app.js"),
	output: {
		path: path.resolve(__dirname, "build/"),
		filename: "bundle.js"
	}
}
```

**注意，如果文件的路径不是在当前目录下，则需要使用 `path.resolve` 来创建文件路径**

现在让我们测试一下, 创建 `app/app.js` 文件, 填入一下内容:

```
document.write("It works");
```

创建 `build/index.html`, 填入以下内容:

```
<!DOCTYPE html>
<head>
  <meta charset="UTF-8">
  <title>Hacker News Front Page</title>
</head>
<body>
  <script src="./bundle.js"></script>  
</body>
</html>
```

其中 script 引入了 `bundle.js`, 这是 webpack 打包后的输出文件.

运行 webpack 打包, 运行 `webpack-dev-server` 启动服务器. 访问 `http://localhost:8080/build/index.html`, 如果一切顺利, 你会看到打印出了 It works.

### 配置 `package.json`

在项目根目录下运行 `npm init -y` 会按照默认设置生成 package.json, 修改 scripts 的键值如下:

```
"scripts": {
  "start": "webpack-dev-server",
  "build": "webpack"
}
```

现在执行 `npm run build` 相当于 `webpack`, `npm run start` 相当于 `webpack-dev-server`. 当项目变得相当复杂时, 你可以使用这种技巧隐藏其中的细节.

### 安装依赖

安装 React:

```
npm install react react-dom --save-dev
```

为了简化 AJAX 请求代码, 这里引入 jQuery:

```
npm install jquery --save-dev
```

安装 Babel 的 loader 以支持 ES6 语法:

```
npm install babel-core babel-loader babel-preset-es2015 babel-preset-react --save-dev
```

然后配置 `webpack.config.js` 来使用安装的 loader.

```
var path = require('path');

module.exports = {
	entry: path.resolve(__dirname, "app/app.js"),
	output: {
		path: path.resolve(__dirname, "build/"),
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
		},
		]
	}
}
```

接下来测试一下开发环境是否搭建完成.

打开 `app.js`, 修改内容为:

```
import $ from "jquery";
import React from "react";
import ReactDOM from "react-dom";

var Helloworld = React.createClass({
	render : function() {
		return (
			<div>Hello World! Hello React!</div>
		);
	}
});

ReactDOM.render(
	<Helloworld />,
	document.getElementById("content")
);
```

这里组件 `HelloWorld` 被装载到 `id` 为 `content` 的 DOM 元素, 所以相应的需要修改 `index.html` .

```
...
<body>
  <div id="content"></div>
  <script src="./bundle.js"></script>  
</body>
...
```

再次打包运行, 访问 `http://localhost:8080/build/index.html` 如果可以看到打印出了 Hello World 那么开发环境就算搭建完成了.

## 在开始写第一个组件之前 

在开始写第一个组件之前, 让我们分析一下到底我们需要哪些组件.

![](https://raw.githubusercontent.com/theJian/build-a-hn-front-page/master/assets/overview.png)

上图是的最终效果的部分截图, 我们将其分为几个组件, 用不同颜色的线框标出

![](https://raw.githubusercontent.com/theJian/build-a-hn-front-page/master/assets/mock.png)


我们可以看出其中包含以下几个组件.

* `NewsList` (蓝色): 所有组件的容器.
* `NewsHeader` (绿色): Logo, 标题, 导航栏等.
* `NewsItem` (红色): 对应每条资讯.

把这些组成为一棵组件树.

* NewsList
    * NewsHeader
    * NewsItem * n

## 编写大致的模板

上一节我们知道了整个页面的组件结构, 在这一节我们根据这些结果编写出一个大致的模板. 先在 app 目录下为每个组件建立文件, `NewsList.js`, `NewsHeader.js` 和 `NewsItem.js`.

```
cd app
touch NewsList.js NewsHeader.js NewsItem.js
```

编辑 `NewsHeader.js`

```
// NewsHeader.js

import React from 'react';

export default class NewsHeader extends React.Component {
  render() {
    return (
        <div className="newsHeader">
          I am NewsHeader.
        </div>
        );
  }
}
```

`NewsHeader` 组件就先这样, 具体的实现现在先不去考虑, 记得我们现在只是编写一个大致的结构.

同样的, 编辑 `NewsItem.js`

```
// NewsItem.js

import React from 'react';

export default class NewsItem extends React.Component {
  render() {
    return (
        <div className="newsItem">
          I am NewsItem.
        </div>
        );
  }
}
```

接着是 `NewsList.js`, 因为 NewsList 是前两个组件的容器, 所以我们需要引入它们

```
// NewsList.js

import React from 'react';
import NewsHeader from './NewsHeader.js';
import NewsItem from './NewsItem.js';

export default class NewsList extends React.Component {
  render() {
    return (
        <div className="newsList">
          <NewsHeader />
          <NewsItem />
        </div>
        );
  }
}
```

最后修改入口文件 `app.js`:

```
import $ from "jquery";
import React from "react";
import ReactDOM from "react-dom";
import NewsList from "./NewsList.js"

ReactDOM.render(
	<NewsList />,
	document.getElementById("content")
);
```

这时访问 `http://localhost:8080/build/index.html` 可以看到下图的效果

![](http://ojt6zsxg2.bkt.clouddn.com/cee65616ba6110b9d7431f40a765da7b.png)

## NewsHeader

![](https://raw.githubusercontent.com/theJian/build-a-hn-front-page/master/assets/newsheader.png)

整个 `NewsHeader` 包含了三个部分, 左边的Logo和标题, 中间的导航栏和右边的登录入口.

让我们先从左边的 Logo 和标题开始.

### Logo 和标题

先下载 Logo

在 `NewsHeader` 组件里新增一个方法 `getLogo`, 就像下面这样.

```
// NewsHeader.js

// ...

export default class NewsHeader extends React.Component {
 //...
 getLogo() {
   /* Do Something */
 }
 //...
}
```

这个方法返回一个包含 Logo 的子组件.

```
getLogo() {
 return (
    <div className="newsHeader-logo">
      <a href="https://news.ycombinator.com/"><img src="PATH_TO_IMAGE" /></a>
    </div>
    );
}
```

这里遇到一个问题, img 的 src 填什么?

在前面几节的开发中, 还记得你是怎么引入其他的 js 文件的吗? `import`. 实际上这是 ES6 的模块系统, 这里的 js 文件作为模块被其他模块引入. 但除了 js 文件, 在开发时我们还会涉及其他的资源文件, 如图像, 字体, 样式等, 它们也需要被模块化. 在这里, 如果 Logo 图片也能被模块化然后引入该多好. 我们需要再次配置 Webpack.

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

然后回到 `NewsHeader.js`

这时候你就可以使用 `import` 引入图片了.

```
import imageLogo from './y18.gif';
```

然后像这样使用.

```
<img src={imageLogo} />
```

注意这里用 `{}` 包起来, 这样其中的内容会作为表达式.

`getLogo` 方法完成了, 再写一个 `getTitle` 方法.

```
getTitle() {
 return (
     <div className="newsHeader-title">
       <a className="newsHeader-textLink" href="https://news.ycombinator.com/">Hacker News</a>
     </div>
     );
}
```

修改 `render` 方法, 引用我们刚刚写好的那两个方法.

```
render() {
 return (
     <div className="newsHeader">
       {this.getLogo()}
       {this.getTitle()}
     </div>
     );
}
```

最后我们需要添加样式, 还是回到刚刚的问题, 怎么引入样式?

我们也需要将样式模块化.

安装相应的 loader:

```
npm install css-loader style-loader --save-dev
```

`css-loader` 处理 css 文件中的 `url()` 表达式.

`style-loader` 将 css 代码插入页面中的 style 标签中.

在 `webpack.config.js` 中配置新的 loader.

```
{
 test: /\.css$/,
 loader: 'style!css'
}
```

新建一个 css 文件 `NewsHeader.css`

```
.newsHeader{background-color: #ff6600; font-size: 10pt; padding: 2px;color: #000;}
.newsHeader-Logo{float: left;width: 20px; height: 20px; padding-right: 4px;}
.newsHeader-Logo img{width: 18px; height: 18px; border: 1px solid #fff;}
.newsHeader-title{height: 20px;line-height: 20px; vertical-align: middle;}
.newsHeader-titleLink{text-decoration: none;font-weight: bold;color: #000;}
```

然后在 `NewsHeader.js` 中引入它

```
import './NewsHeader.css';
```

再建立一个全局的 css 文件 `app.css`

```
body {
 font-family: Verdana, sans-serif;
}
```

然后在 `app.js` 中引入

```
import './app.css'
```

打包运行看看吧.

### 导航栏

接下来是导航栏, 也就是中间那部分.

回到 `NewsHeader.js`, 增加一个 `getNav` 方法.

```
getNav() {
  	let navLinks = [
		{
			name: "new",
			url: "newest"
		},
		{
			name: "comments",
			url: "newComments"
		},
		{
			name: "show",
			url: "show"
		},
		{
			name: "ask",
			url: "ask"
		},
		{
			name: "jobs",
			url: "jobs"
		},
		{
			name: "submit",
			url: "submit"
		},
  	];
  	return (
		<div className="newsHeader-nav">
			{
				navLinks.map(function(navLink) {
					return (
						<a key={navLink.name} className="newsHeader-navLink" href={"https://news.ycombinator.com/" + navLink.url}>{navLink.name}</a>
					);
				})
			}
		</div>
  	);
  }
```

同样, 记得在 `render` 方法中引用

```
render() {
 return (
     <div className="newsHeader">
       {this.getLogo()}
       {this.getTitle()}
       {this.getNav()}
     </div>
     );
}
```
添加样式.

```
.newsHeader-nav{height: 20px; line-height: 20px;}
.newsHeader-navLink{text-decoration: none; color: #000;}
.newsHeader-navLink:not(:first-child)::before{content: " | "}
```

### 登录入口

增加一个 `getLogin` 方法.

```
getLogin() {
  	return (
		<div className="newsHeader-login">
			<a href="https://news.ycombinator.com/login?goto=news" className="newsHeader-loginLink">login</a>
		</div>
  	);
  }

```

在 `render` 中引用

```
render() {
 return (
     <div className="newsHeader">
       {this.getLogo()}
       {this.getTitle()}
       {this.getNav()}
       {this.getLogin()}
     </div>
     );
}
```

更新样式

```
.newsHeader{background-color: #ff6600; font-size: 10pt; padding: 2px;color: #000;height: 20px;clear: both;}
.newsHeader-Logo{float: left;width: 20px; height: 20px; padding-right: 4px;}
.newsHeader-Logo img{width: 18px; height: 18px; border: 1px solid #fff;}
.newsHeader-title{height: 20px;line-height: 20px; vertical-align: middle; float: left;padding-right: 8px;}
.newsHeader-titleLink{text-decoration: none;font-weight: bold;color: #000;}
.newsHeader-nav{height: 20px; line-height: 20px;margin-right: 50px;float: left;}
.newsHeader-navLink{text-decoration: none; color: #000;}
.newsHeader-navLink:not(:first-child)::before{content: " | "}
.newsHeader-login{height: 20px; line-height: 20px; padding-right: 8px; float: right;}
.newsHeader-loginLink{text-decoration: none;color: #000;}
```

至此整个 `NewsHeader` 就完成了, 你应该能看到如本节初所展示的效果.

![](http://ojt6zsxg2.bkt.clouddn.com/1ba763525944d6e03c425bd122590949.png)

## NewsItem

![](http://ojt6zsxg2.bkt.clouddn.com/bfd213bf206697ae9e6ccc07ba89deaf.png)

如图, 每条资讯对应着这样一个 `NewsItem`, 本节我们将编写 `NewsItem` 组件.

可以看到, 一个 `NewsItem` 包含了资讯的标题, 来源地址, 什么时候发布的以及评论数等等. 它依赖于传入的数据, 那么怎么传入数据呢?

对于父子组件间的通信, 可以使用属性传递. 子组件可以使用 `this.props` 访问到父组件传入的属性数据.

回到 `NewsList` 组件, 它作为 `NewsItem` 的父组件可以使用如下方式传入数据.

回到 `NewsList` 组件, 它作为 `NewsItem` 的父组件可以使用如下方式传入数据.

```
<NewsItem item={data} />
```

`NewsItem` 中可以使用 `this.props.item` 访问 `item` 属性.

### NewsItem 标题

先来简单点的, 第一步我们只获取并显示标题.

修改 `NewsList.js`

```
export default class NewsList extends React.Component {
	var testData = {
		"by" : "bane",
	   "descendants" : 49,
	   "id" : 11600137,
	   "kids" : [ 11600476, 11600473, 11600501, 11600463, 11600452, 11600528, 11600421, 11600577, 11600483 ],
	   "score" : 56,
	   "time" : 1461985332,
	   "title" : "Yahoo's Marissa Mayer could get $55M in severance pay",
	   "type" : "story",
	   "url" : "http://www.latimes.com/business/technology/la-fi-0429-tn-marissa-mayer-severance-20160429-story.html"
	};
    render() {
    	return (
	        <div className="newsList">
	          <NewsHeader />
	          <NewsItem item={testData} rank={1}/>
	        </div>
        );
    }
}
```

这里我们声明一个 testData 作为测试数据传入 NewsItem.

修改 `NewsItem.js

```
render() {
    return (
        <div className="newsItem">
        	<a className="newItem-title" href={this.props.item.url}>{this.props.item.title}</a>
        </div>
    );
```

在这里使用 this.props.item 访问 item 属性.

建立 NewsItem.css

```
.newsItem{background-color: #f6f6ef;margin-top: 5px;}
.newItem-title{text-decoration: none;color: #000;display: block;font-size: 10pt}
```

在 `NewsItem.js` 中引入

```
import './NewsItem.css';
```

运行看看效果, 显示的标题应该和传入的测试数据中的一样.

### NewsItem 来源地址

我们现在添加来源地址到标题的末尾. 先在 `NewsItem.js` 中引入 `url` 模块

```
import URL from 'url';
```

然后增加一个 `getDomain` 方法.

```
getDomain() {
 return URL.parse(this.props.item.url).hostname;
}
```

然后再增加一个 `getTitle` 方法, 这个方法会返回一个包含了标题(我们上一节做的事)和地址的组件.

```
getTitle() {
		return (
			<div className="newItem-title">
				<a className="newItem-titleLink" href={this.props.item.url}>{this.props.item.title}</a>
        		<span className="newItem-titleDomain">
				<a href={'https://news.ycombinator.com/from?site=' + this.getDomain()}> ({this.getDomain()})</a>
        		</span>
			</div>
		);
	}
```

修改 `render`

```
render() {
	    return (
	        <div className="newsItem">
	        	{this.getTitle()}
	        </div>
	    );
	}
```

增加样式

```
.newsItem{background-color: #f6f6ef;margin-top: 5px;}
.newItem-title{}
.newItem-titleLink{text-decoration: none; color: #000;font-size: 10pt;}
.newItem-titleDomain a{text-decoration: none;color: 828282; font-size: 8pt}
.newItem-titleDomain a:hover{text-decoration: underline;}
```

好了, 看起来不错, 但是有个问题, 这个项目最终需要从 Hacker News 的 API 取得资讯数据, 而其中有些是没有 `url` 属性的, 看看我们的 `getTitle()` 方法, 我们似乎忽略了这个特例, 让我们做些修改.

```
getTitle() {
		return (
			<div className="newItem-title">
				<a className="newItem-titleLink" href={this.props.item.url}>{this.props.item.title}</a>
        		{
        			this.props.item.url && <span className="newItem-titleDomain">
				<a href={'https://news.ycombinator.com/from?site=' + this.getDomain()}> ({this.getDomain()})</a>
        		</span>
        		}
			</div>
		);
	}
```

试着去掉 `testData` 的 `url` 属性, 看看是不是一切正常.

### NewsItem 其余部分

我们现在加上其余部分, 你已经看过了前两节, 这节应该是没有什么难度的, 我们快速带过.

下载 `grayarrow.gif`, 在 `NewsItem.js` 中引入

```
import ImageGrayArrow from '../img/grayarrow.gif';
```

这里计算时间间距我们使用了 [Moment](http://momentjs.com), 如果你要使用你需要安装并引入它, 或者使用你喜欢的实现方法.

```
npm install moment --save-dev
```

修改 `NewsItem.js`

```
getDomain() {
		return URL.parse(this.props.item.url).hostname;
	}
	getTitle() {
		return (
			<div className="newItem-title">
				<a className="newItem-titleLink" href={this.props.item.url}>{this.props.item.title}</a>
        		{
        			this.props.item.url && <span className="newItem-titleDomain">
				<a href={'https://news.ycombinator.com/from?site=' + this.getDomain()}> ({this.getDomain()})</a>
        		</span>
        		}
			</div>
		);
	}

	getCommentLink() {
		var commentText = "discuss";
		if(this.props.item.kids && this.props.item.kids.length)
			commentText = this.props.item.kids.length + " comments";
		return (
			<a href={"https://news.ycombinator.com/item?id=" + this.props.item.id}>{commentText}</a>
		);
	}

	getSubtext() {
		return (
			<div className="newsItem-subtext">
				{this.props.item.score} points by <a href={"https://news.ycombinator.com/user?id=" + this.props.item.by}>{this.props.item.by} {Moment.utc(this.props.item.time * 1000).fromNow()} </a> | {this.getCommentLink()}
			</div>
		);
	}

	getVote() {
		return (
			<div className="newsItem-vote">
				<a href={"https://news.ycombinator.com/vote?id=" + this.props.item.id + "&how=up&goto=news"}>
					<img src={ImageGrayArray} width="10" />
				</a>
			</div>
		);
	}

	getRank() {
		return (
			<div className="newsItem-rank">
				{this.props.rank}.
			</div>
		);
	}

  	render() {
	    return (
	        <div className="newsItem">
	        	{this.getRank()}
	        	{this.getVote()}
	        	<div className="newsItem-itemText">
	        		{this.getTitle()}
	        		{this.getSubtext()}
	        	</div>
	        </div>
	    );
	}
```

`NewItem.css`

```
.newsItem{background-color: #f6f6ef;padding-top: 6px;}
.newItem-title{}
.newItem-titleLink{text-decoration: none; color: #000;font-size: 10pt;}
.newItem-titleDomain a{text-decoration: none;color: 828282; font-size: 8pt}
.newItem-titleDomain a:hover{text-decoration: underline;}
.newsItem-rank{font-size: 10pt;width: 20px;float: left;text-align: center;}
.newsItem-vote{float: left;margin: 3px 2px 6px 2px;}
.newsItem-subtext{font-size: 7pt; color: #828282;}
.newsItem-subtext a{text-decoration: none; color: #828282;}
.newsItem-subtext a:hover{text-decoration: underline;}
```

## NewsList

上一节中为了测试 `NewsItem`, 我们定义了一个测试数据 `testData`, `NewsList` 中也只有一个 `NewsItem`, 而真实的情况不会只有一条资讯, 而应该是一组资讯, 每一条对应有一个 `NewsItem`, 本节中我们来实现这个功能.

首先我们确定传入的数据是一个数组, 其中每一个元素都是一条资讯, 至于这个数据由哪里传入, 怎么生成我们先不关心, 但我们可以用 `this.props.items` 获取到. `NewsList` 对于其中的每一个元素都生成一个 `NewsItem`.

下面是修改完的 `render`

```
render() {
    return (
        <div className="newsList">
          <NewsHeader />
          <div className="newsList-newsItem">
            {
              (this.props.items).map(function(item, index) {
                return (
                    <NewsItem key={item.id} item={item} rank={index+1} />
                    );
              })
            }
          </div>
        </div>
        );
  }
```

新建样式 NewsList.css

```
.newsList{width: 85%; margin: 0 auto;}
```

目前的代码是没法运行的, 我们还没有取得数据传入给 `NewsList`, 这将在下一节完善.

### Hacker News API

本节中我们使用 [Hacker News API](https://github.com/HackerNews/API) 来获取数据, 具体请参考 API 文档.

`app.js`

```
function get(url) {
  return Promise.resolve($.ajax(url));
}

get('https://hacker-news.firebaseio.com/v0/topstories.json').then( function(stories) {
  return Promise.all(stories.slice(0, 30).map(itemId => get('https://hacker-news.firebaseio.com/v0/item/' + itemId + '.json')));
}).then(function(items) {
	console.log(items);
  ReactDOM.render(
  	<NewsList items={items}/>,
  	document.getElementById("content")
  );
}).catch(function(err) {
  console.log('error occur', err);
});
```

`items` 就是处理完后的数据, 一个由资讯数据组成的数组, 我们将它作为属性传入 `NewsList`.

最后的效果如下：

![](http://ojt6zsxg2.bkt.clouddn.com/25fe7e54707be92b2884feb8c82af4a3.png)

## Demo

这个小项目的 Demo 我已经传到网上去了，可以在 github 上进行下载。具体步骤如下：

安装方法是：

```
git clone https://github.com/ShiningDan/React-hacker-news.git
cd React-hacker-news/
npm install
npm run build
npm run start
```

就可以在 http://localhost:8080/build/index.html 上看到项目的展示效果。因为是从 Hacker News 上获取最新的新闻，页面的显示速度和网速以及 Hacker News 的访问速度有关，请耐心等待~


