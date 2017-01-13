---
title: 用Sass实现新浪微博首页
date: 2017-01-13 17:07:52
categories: coding
tags:
  - HTML
  - Sass
---

**在该笔记中将学到列表展示中新浪微博主页的制作方法与展示效果**

本系列的是参考[慕课网 Busy](http://www.imooc.com/u/114418/courses?sort=publish)老师的视频，仅供个人查阅使用，具体讲解推荐参考视频。

显示效果为：

![](http://ofjjubwp5.bkt.clouddn.com/image/png/img83.png)

其中需要注意的要点有：

**如果有 `logo` 和文字出现在同一行的情况下，如何设定对齐方式来使得文字和图片对齐：**

解决方案为：将图片和文字分别放在两个相同高度的 div 中，同时设定两个 Div 为 `display:inline-block`。此时因为默认的纵轴对齐方式是`vertical-align:baseline`，导致图片的最下部和文字的底部对齐，可能会造成文字的下沉，所以要对包裹文字的 Div 设置 `vertical-align:top`，使得文字和图片的容器以 top 形式进行纵轴对齐。`vertical-align`必须在`display:inline/inline-block`的时候才生效。

<!--more-->

代码如下：

```HTML
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Sass实现新浪微博</title>
    <link rel="stylesheet" href="test.css" type="text/css">
</head>
<body>
    <div class="header">
        <div class="header-content">
            <div class="logo"><a href="#">新浪微博</a></div>
            <div class="search">
                <input type="text" class="search-content">
            </div>
            <ul class="headerNav">
                <li><a href="#"><i class="nav-logo home"></i><i class="text">首页</i></a></li>
                <li><a href="#"><i class="nav-logo video"></i><i class="text">视频</i></a></li>
                <li><a href="#"><i class="nav-logo find"></i><i class="text">发现</i></a></li>
                <li><a href="#"><i class="nav-logo game"></i><i class="text">游戏</i></a></li>
            </ul>
        </div>
    </div>
    <div class="main">
        <div class="content-list">
            <div class="list-nav"><i class="nav-icon"></i><i class="nav-text">热门</i><i class="nav-tag"></i></div>
            <div class="list-nav"><i class="nav-icon"></i><i class="nav-text">明星</i><i class="nav-tag"></i></div>
            <div class="list-nav"><i class="nav-icon"></i><i class="nav-text">搞笑</i><i class="nav-tag"></i></div>
            <div class="list-nav"><i class="nav-icon"></i><i class="nav-text">社会</i><i class="nav-tag"></i></div>
            <div class="list-nav"><i class="nav-icon"></i><i class="nav-text">视频</i><i class="nav-tag"></i></div>
            <div class="list-nav"><i class="nav-icon"></i><i class="nav-text">头条</i><i class="nav-tag hot-tag"></i></div>
            <div class="list-nav"><i class="nav-icon"></i><i class="nav-text">情感</i><i class="nav-tag"></i></div>
            <div class="list-nav"><i class="nav-icon"></i><i class="nav-text">时尚</i><i class="nav-tag"></i></div>
            <div class="list-nav"><i class="nav-icon"></i><i class="nav-text">军事</i><i class="nav-tag"></i></div>
            <div class="list-nav"><i class="nav-icon"></i><i class="nav-text">榜单</i><i class="nav-tag"></i></div>
        </div>
        <div class="content-l">
            <div class="videos">
               <div class="video"><a href="#"><img src="http://ofjjubwp5.bkt.clouddn.com/image/jpg/12.jpg" alt=""></a><a href="#" class="video-text">这是第一张视频的描述。。。。blablalblablabla</a></div>
               <div class="video"><a href="#"><img src="http://ofjjubwp5.bkt.clouddn.com/image/jpg/11.jpg" alt=""></a><a href="#" class="video-text">这是第一张视频的描述。。。。blablalblablabla</a></div>
               <div class="video"><a href="#"><img src="http://ofjjubwp5.bkt.clouddn.com/image/jpg/10.jpg" alt=""></a><a href="#" class="video-text">这是第一张视频的描述。。。。blablalblablabla</a></div>
            </div>
        </div>
        <div class="content-r">
            <div class="registion">
                <div class="login">
                    <a href="#" class="account-login">账号登录</a>
                    <a href="#" class="secure-login">安全登录</a>
                </div>
                <div class="input-wrap">
                    <input type="text" class="account" value="邮箱/会员账号/手机号">
                    <input type="password" class="password" placeholder="请输入密码">
                </div>
                <div class="login-tip">
                    <div class="rem">
                        <i class="rem-btn"></i>
                        <i class="rem-text">记住我</i>
                    </div>
                    <div class="forg">
                        <i class="forg-text">忘记密码</i>
                    </div>
                </div>
                <input type="button" value="登录">
            </div>
        </div>
    </div>
</body>
</html>
```

Sass:

```css
$header-topc: #EB834C;
$header-bgc: #fff;
$body-bgc: #BDCBE3;

body{
	margin: 0;
	padding: 0;
	background-color: $body-bgc;
	i{
		font-style: normal;
	}
}
.header{
	.header-content{
		height: 28px;
		width: 1200px;
		margin-top: 10px;
		margin-left: auto;
		margin-right: auto;
	}
	border-top: 2px solid $header-topc;
	background-color: $header-bgc;
	height: 48px;
	overflow: hidden;
	.logo{
		display: inline-block;
		width: 80px;
		height: 28px;
		margin-left: 140px;
		float: left;
		background: url("http://ofjjubwp5.bkt.clouddn.com/weibo-logo.png") left top no-repeat;
		a{
			display: none;
		}
	}
	.search{
		display: inline-block;
		margin-left: 70px;
		float: left;
		width: 218px;
		height: 28px;
		line-height: 28px;
		.search-content{
			background-color: #f2f2f5;
			width: 218px;
			height: 28px;
			padding: 0px;
			border: 1px solid #ccc;
		}
	}
	.search-btn{
		display: inline-block;
		width: 16px;
		height: 16px;
		background-image: url("images/magnifier.png");
	}
	.headerNav{
		display: inline-block;
		height: 28px;
		line-height: 28px;
		float: left;
		list-style: none;
		margin: 0 0 0 200px;
		padding: 0; 
		li{
			font-size: 16px;
			display: inline-block;
			float: left;
			width: 80px;
			a{
				text-decoration: none;
				display: inline-block;
				margin-left: 50px;
				width: 80px;
				height: 28px;
				color: #696e78;
				i.nav-logo{
					display: inline-block;
					width: 20px;
					height: 28px;
					line-height: 28px;
					vertical-align: top;
					margin-top: 4px;
				}
				i.text{
					font-style: normal;
					width: 50px;
					background: none;
					text-align: center;
					display: inline-block;
				}
				i.text:hover{
					color: #EB834C;
				}
				i.home{
					background: url("ihttp://ofjjubwp5.bkt.clouddn.com/home.png") no-repeat;
				}
				i.video{
					width: 26px;
					background: url("http://ofjjubwp5.bkt.clouddn.com/video.png") no-repeat;
				}
				i.find{
					width: 22px;
					margin-top: 2px;
					background: url("http://ofjjubwp5.bkt.clouddn.com/find.png") no-repeat;
				}
				i.game{
					width: 24px;
					background: url("http://ofjjubwp5.bkt.clouddn.com/game.png") no-repeat;
				}
			}
		}
	}
}
.main{
	width: 1000px;
	background-color: #87a6cf;
	height: 500px;
	padding: 16px 10px 16px 0;
	margin: 0 auto;
}
.content-list{
	width: 150px;
	height: 100%;
	background-color: #87a6cf;
	float: left;
	.list-nav{
		width: 100%;
		height: 42px;
		vertical-align: top;
		.nav-icon{
			display: inline-block;
			width: 24px;
			height: 42px;
			margin-left: 36px;
		}
		.nav-text{
			display: inline-block;
			width: 44px;
			height: 42px;
			line-height: 42px;
			text-align: center;
			vertical-align: top;
			color: #fff;
		}
		.nav-tag{
			display: inline-block;
			width: 46px;
			height: 42px;
		}
		.hot-tag{
			background: url("http://ofjjubwp5.bkt.clouddn.com/hot-tag.png") no-repeat 8px 13px;
		}
	}
	.list-nav:hover{
		background-color: #9fb8d8;
	}
}
.content-l{
	width: 600px;
	height: 100%;
	margin-right: 10px;
	border: none;
	float: left;
	.videos{
		padding: 20px;
		width: 560px;
		height: 165px;
		overflow:hidden;
		background-color: #fff;
		border-radius: 4px;
		.video{
			margin-right: 10px;
			float: left;
			width: 180px;
			height: 165px;
		}
		.video:last-child{
			margin-right: 0px;
		}
		img{
			width: 180px;
			height: 130px;
		}
		.video-text{
			display: inline-block;
			height: 35px;
			line-height: 18px;
			font-size: 12px;
			text-align: center;
			color: #000;
			text-decoration: none;
		}
	}
}
.content-r{
	height: 100%;
	width: 230px;
	background-color: #fff;
	float: right;
	border: none;
	border-radius: 4px;
	.registion{
		width: 200px;
		height: 286px;
		background-color: #fff;
		padding: 11px 15px;
		.login{
			width: 200px;
			height: 33px;
			a{
				display: inline-block;
				color: #828282;
				width: 100px;
				height: 33px;
				line-height: 33px;
				font-size: 14px;
				text-align: center;
				float: left;
				text-decoration: none;
			}
			a.account-login{
				color:#828282;
				background: url("http://ofjjubwp5.bkt.clouddn.com/login-bg-1.png") no-repeat 0px 30px;
			}
			a.account-login:hover{
				color:#000;
				background: url("http://ofjjubwp5.bkt.clouddn.com/login-bg-2.png") no-repeat 0px 30px;
			}
			a.secure-login{
				color:#828282;
				background: url("http://ofjjubwp5.bkt.clouddn.com/login-bg-1.png") no-repeat 0px 30px;
			}
			a.secure-login:hover{
				color:#000;
				background: url("http://ofjjubwp5.bkt.clouddn.com/login-bg-2.png") no-repeat 0px 30px;
			}
		}
		.input-wrap{
			padding: 16px 0;
			width: 200px;
			input{
				width: 200px;
				height: 34px;
				margin-bottom: 12px;
				color: #828282;
			}
		}
		.login-tip{
			color: #828282;
			font-size: 12px;
			width: 200px;
			height: 30px;
			div.rem{
				float: left;
				cursor: pointer;
				.rem-btn{
					display: inline-block;
					border: 1px solid #828282;
					width: 12px;
					height: 12px;
				}
				.rem-text{
					display: inline-block;
					vertical-align: top;
				}
			}
			div.forg{
				cursor: pointer;
				float: right;
			}
		}
		input[type="button"] {
			width: 200px;
			height: 34px;
			border-radius: 4px;
			border: none;
			background-color: #ff8140;
			color: #fff;
			font-size: 16px;
			text-align: center;
		}
	}
}
```

CSS：

```css
body {
  margin: 0;
  padding: 0;
  background-color: #BDCBE3; }
  body i {
    font-style: normal; }

.header {
  border-top: 2px solid #EB834C;
  background-color: #fff;
  height: 48px;
  overflow: hidden; }
  .header .header-content {
    height: 28px;
    width: 1200px;
    margin-top: 10px;
    margin-left: auto;
    margin-right: auto; }
  .header .logo {
    display: inline-block;
    width: 80px;
    height: 28px;
    margin-left: 140px;
    float: left;
    background: url("http://ofjjubwp5.bkt.clouddn.com/weibo-logo.png") left top no-repeat; }
    .header .logo a {
      display: none; }
  .header .search {
    display: inline-block;
    margin-left: 70px;
    float: left;
    width: 218px;
    height: 28px;
    line-height: 28px; }
    .header .search .search-content {
      background-color: #f2f2f5;
      width: 218px;
      height: 28px;
      padding: 0px;
      border: 1px solid #ccc; }
  .header .search-btn {
    display: inline-block;
    width: 16px;
    height: 16px;
    background-image: url("images/magnifier.png"); }
  .header .headerNav {
    display: inline-block;
    height: 28px;
    line-height: 28px;
    float: left;
    list-style: none;
    margin: 0 0 0 200px;
    padding: 0; }
    .header .headerNav li {
      font-size: 16px;
      display: inline-block;
      float: left;
      width: 80px; }
      .header .headerNav li a {
        text-decoration: none;
        display: inline-block;
        margin-left: 50px;
        width: 80px;
        height: 28px;
        color: #696e78; }
        .header .headerNav li a i.nav-logo {
          display: inline-block;
          width: 20px;
          height: 28px;
          line-height: 28px;
          vertical-align: top;
          margin-top: 4px; }
        .header .headerNav li a i.text {
          font-style: normal;
          width: 50px;
          background: none;
          text-align: center;
          display: inline-block; }
        .header .headerNav li a i.text:hover {
          color: #EB834C; }
        .header .headerNav li a i.home {
          background: url("ihttp://ofjjubwp5.bkt.clouddn.com/home.png") no-repeat; }
        .header .headerNav li a i.video {
          width: 26px;
          background: url("http://ofjjubwp5.bkt.clouddn.com/video.png") no-repeat; }
        .header .headerNav li a i.find {
          width: 22px;
          margin-top: 2px;
          background: url("http://ofjjubwp5.bkt.clouddn.com/find.png") no-repeat; }
        .header .headerNav li a i.game {
          width: 24px;
          background: url("http://ofjjubwp5.bkt.clouddn.com/game.png") no-repeat; }

.main {
  width: 1000px;
  background-color: #87a6cf;
  height: 500px;
  padding: 16px 10px 16px 0;
  margin: 0 auto; }

.content-list {
  width: 150px;
  height: 100%;
  background-color: #87a6cf;
  float: left; }
  .content-list .list-nav {
    width: 100%;
    height: 42px;
    vertical-align: top; }
    .content-list .list-nav .nav-icon {
      display: inline-block;
      width: 24px;
      height: 42px;
      margin-left: 36px; }
    .content-list .list-nav .nav-text {
      display: inline-block;
      width: 44px;
      height: 42px;
      line-height: 42px;
      text-align: center;
      vertical-align: top;
      color: #fff; }
    .content-list .list-nav .nav-tag {
      display: inline-block;
      width: 46px;
      height: 42px; }
    .content-list .list-nav .hot-tag {
      background: url("http://ofjjubwp5.bkt.clouddn.com/hot-tag.png") no-repeat 8px 13px; }
  .content-list .list-nav:hover {
    background-color: #9fb8d8; }

.content-l {
  width: 600px;
  height: 100%;
  margin-right: 10px;
  border: none;
  float: left; }
  .content-l .videos {
    padding: 20px;
    width: 560px;
    height: 165px;
    overflow: hidden;
    background-color: #fff;
    border-radius: 4px; }
    .content-l .videos .video {
      margin-right: 10px;
      float: left;
      width: 180px;
      height: 165px; }
    .content-l .videos .video:last-child {
      margin-right: 0px; }
    .content-l .videos img {
      width: 180px;
      height: 130px; }
    .content-l .videos .video-text {
      display: inline-block;
      height: 35px;
      line-height: 18px;
      font-size: 12px;
      text-align: center;
      color: #000;
      text-decoration: none; }

.content-r {
  height: 100%;
  width: 230px;
  background-color: #fff;
  float: right;
  border: none;
  border-radius: 4px; }
  .content-r .registion {
    width: 200px;
    height: 286px;
    background-color: #fff;
    padding: 11px 15px; }
    .content-r .registion .login {
      width: 200px;
      height: 33px; }
      .content-r .registion .login a {
        display: inline-block;
        color: #828282;
        width: 100px;
        height: 33px;
        line-height: 33px;
        font-size: 14px;
        text-align: center;
        float: left;
        text-decoration: none; }
      .content-r .registion .login a.account-login {
        color: #828282;
        background: url("http://ofjjubwp5.bkt.clouddn.com/login-bg-1.png") no-repeat 0px 30px; }
      .content-r .registion .login a.account-login:hover {
        color: #000;
        background: url("http://ofjjubwp5.bkt.clouddn.com/login-bg-2.png") no-repeat 0px 30px; }
      .content-r .registion .login a.secure-login {
        color: #828282;
        background: url("http://ofjjubwp5.bkt.clouddn.com/login-bg-1.png") no-repeat 0px 30px; }
      .content-r .registion .login a.secure-login:hover {
        color: #000;
        background: url("http://ofjjubwp5.bkt.clouddn.com/login-bg-2.png") no-repeat 0px 30px; }
    .content-r .registion .input-wrap {
      padding: 16px 0;
      width: 200px; }
      .content-r .registion .input-wrap input {
        width: 200px;
        height: 34px;
        margin-bottom: 12px;
        color: #828282; }
    .content-r .registion .login-tip {
      color: #828282;
      font-size: 12px;
      width: 200px;
      height: 30px; }
      .content-r .registion .login-tip div.rem {
        float: left;
        cursor: pointer; }
        .content-r .registion .login-tip div.rem .rem-btn {
          display: inline-block;
          border: 1px solid #828282;
          width: 12px;
          height: 12px; }
        .content-r .registion .login-tip div.rem .rem-text {
          display: inline-block;
          vertical-align: top; }
      .content-r .registion .login-tip div.forg {
        cursor: pointer;
        float: right; }
    .content-r .registion input[type="button"] {
      width: 200px;
      height: 34px;
      border-radius: 4px;
      border: none;
      background-color: #ff8140;
      color: #fff;
      font-size: 16px;
      text-align: center; }
```










