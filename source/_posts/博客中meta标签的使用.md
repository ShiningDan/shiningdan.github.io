---
title: 博客中meta标签的使用
date: 2017-05-25 21:45:17
categories: coding
tags:
  - meta
---

在我的博客中，通过 `<meta>` 标签添加了一些附加的小功能，或者禁用了一些选项，这篇文章中介绍了我遇到的一些 `<meta>` 标签及其作用。本文长期更新

<!--more-->

## meta 的介绍

在 HTML 标签中，`<meta>` 标签表示那些不能由其它HTML元相关元素 (`<base>, <link>, <script>, <style> 或 <title>`) 之一表示的任何元数据信息.

## meta 标签的使用

### charset

此特性声明当前文档所使用的字符编码，但该声明可以被任何一个元素的 `lang` 特性的值覆盖。

鼓励使用 `UTF-8`。

### X-UA-Compatible

```
<meta http-equiv="X-UA-Compatible" content="IE=edge">
```

用于告知浏览器以何种版本来渲染页面，比如上面这句话指定 IE 使用最新版本来渲染页面。

### format-detection

```
<meta name="format-detection" content="telephone=no">
```

上面这句话表面，禁止把数字转化为拨号链接，默认是 `yes`

同样类型的还有以下几种：

```
meta name="format-detection" content="email=no"
meta name="format-detection" content="adress=no" 
也可以连写：meta name="format-detection" content="telephone=no,email=no,adress=no"
```



`email=no` 禁止作为邮箱地址，告诉设备不识别邮箱，点击之后不自动发送

`email=yes` 就开启了把文字默认为邮箱地址，这个meta就不用写了,在
默认是情况下就是开启


`adress=no` 禁止跳转至地图

`adress=yes` 就开启了点击地址直接跳转至地图的功能,在默认是情况下就是开启

### theme-color

```
<meta name="theme-color" content="#db5945">
```

![](http://ojt6zsxg2.bkt.clouddn.com/d3bc402fc2437feab32018d7d478eacf.png)

Android Lollipop 中的 Chrome 39 增加 `theme-color meta` 标签，用来控制选项卡颜色。

### description

```
<meta name="description" content="不超过150个字符"/>
```
页面描述

### apple-mobile-web-app-title

```
<meta name="apple-mobile-web-app-title" content="标题">
```

添加到主屏后的标题（iOS 6 新增）

### apple-mobile-web-app-capable

```
<meta name="apple-mobile-web-app-capable" content="yes"/>
```
添加到主屏后是否启用 WebApp 全屏模式

### apple-mobile-web-app-status-bar-style

只有在 `"apple-mobile-web-app-capable" content="yes"` 时生效

如果设置为 `default` 或 `black` ，网页内容从状态栏底部开始。

如果设置为 `black-translucent` ，网页内容充满整个屏幕，顶部会被状态栏遮挡。

```
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent"/>
```

添加到主屏后设置状态栏的背景颜色

### apple-touch-icon

```
<link rel="apple-touch-icon" href="/img/icon.png">
```

iOS 图标，图片自动处理成圆角和高光等效果，默认 57x57 像素



## 参考

* [meta | MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/meta)
* [Meta标签中的format-detection属性及含义](http://blog.sina.com.cn/s/blog_51048da70101cgea.html)
* [移动端的头部标签和 meta](https://github.com/gafish/gafish.github.com/issues/2)
* [移动前端不得不了解的 HTML5 head 头标签](https://juejin.im/entry/58085b3267f3560057a0053a)

