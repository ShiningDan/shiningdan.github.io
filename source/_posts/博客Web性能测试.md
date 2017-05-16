---
title: 博客Web性能测试
date: 2017-05-15 18:01:39
categories: coding
tags:
  - Chrome Timeline
  - Chrome Network
---

## 前言

我的个人博客已经上线了！很开心这个博客可以成为我个人的试验田，本次盯上的是前端性能优化这一点，所以我准备在接下来的一段时间，用上几篇文章来介绍一下针对我的博客的优化步骤。为了更好的展示性能优化的效果，我在写这个博客系统的时候估计留下来了一些不好的编程风格 XD，其目的一是更加全面地介绍前端性能优化的方方面面，目的二是通过对这些方面更全面地学习，再重新更新一下自己的技能树，温故而知新咯。

```
时间与用户感觉

本页数据来自于《WEB性能权威指南》10.2.1节

时间	                        感觉
0 ~ 100 ms	                  很快
100 ~ 300 ms	           有一点点慢
300 ~ 1000 ms	          机器在工作呢
> 1000 ms	              先干点别的吧
> 10000 ms	                不能用了
```

<!--more-->

[PageSpeed Insights规则](https://developers.google.com/speed/docs/insights/rules?hl=zh-cn) 提供了一些针对网页进行优化的出发点。我们在对网页速度进行分析了以后，可以从这些出发点进行考虑。

## 测试

现在我的博客有很多需要改进的地方，那如何来评价什么是快，什么是慢呢？当然是用数字说话，现在，我们先使用几种常用的前端工具来测试一下博客在各个方面的速度吧~

首先，在进行测试的时候，要注意以下几点：

1. 禁用缓存，来获得首次浏览性能
2. 打开隐身模式来禁用所有的插件
3. 通过 Chrome Network 来模拟不同条件下的网络速度

使用的测试工具有：

* Chrome Network
* Chrome Timeline

## Chrome Network

我们可以在[Chrome 开发者工具](https://developers.google.com/web/tools/chrome-devtools/?hl=zh-cn) 查看 Chrome 提供的浏览器工具的使用方法

Chrome Network 是我们常用的获取浏览器网络操作的相关信息的地方，包括详细的耗时数据、HTTP 请求与响应标头和 Cookie，等等。

关于 Chrome Network 的各个部分中不同按钮的含义，可以从 [测量资源加载时间](https://developers.google.com/web/tools/chrome-devtools/network-performance/resource-loading?hl=zh-cn) 这篇文章来获得，在这里就不再赘述各个按钮的含义了。

目前我的博客是运行在本地环境上的，用来保证博客的响应速度不受到网络环境不稳定性的影响。

在使用 Network 进行测试的时候，需要关心两个事件的时间，第一个事件是 `DOMContentLoaded`，第二个是 `load`事件。关于 `DOMContentLoaded` 事件的触发时间，可以参考 [JS、CSS以及img对DOMContentLoaded事件的影响](http://www.alloyteam.com/2014/03/effect-js-css-and-img-event-of-domcontentloaded/) 这篇文章，其中大致的意思是：

`DOMContentLoaded` 事件本身不会等待CSS文件、图片、iframe加载完成。它的触发时机是：加载完页面，解析完所有标签（不包括执行CSS和JS），并如规范中所说的设置 `interactive` 和**执行每个静态的script标签中的JS**，然后触发。而JS的执行，需要等待位于它前面的CSS加载（如果是外联的话）、执行完成，因为JS可能会依赖位于它前面的CSS计算出来的样式。

1. `DOMContentLoaded` 事件不直接等待CSS文件、图片的加载完成
2. `DOMContentLoaded` 事件需要等待JS执行完才触发，但是其中包含一种情况，`script` 标签中的JS需要等待位于其前面的CSS的加载完成。

下面我们来看一下如何使用 Chrome Network 来测试一下我的博客的网络速度。


### 首页

#### Regular 2G

在测试的时候，我选择的是 `disable cache`，并且在 Network 上设置网络的状态为 `Regular 2G, 300ms, 250kb/s, 50kb/s` 来测试网站的速度，结果如下：

```
DOMContentLoaded: 754 ms
Load: 1.27s
// 全网站加载完时间
Total: 21s
```

![](http://ojt6zsxg2.bkt.clouddn.com/d6afd4dbd686cda3790fbfd27997ca6c.png)

在博客的 DOM 结构和文本内容以及 CSS 都加载完的情况下，一共花费了 1.27s，剩下将近 20s 的时间都用来加载图片，由此可见，过大的图片对网页的浏览效果有多么大的影响。所以，在网站部署之前，对网站的静态资源都进行压缩等处理是非常有必要的。并且针对不经常改动的静态资源，可以预先存放在客户端，就不需要再重复下载了。

#### 线上测试

```
DOMContentLoaded: 160 ms
Load: 300ms - 600ms
// 全网站加载完时间
Total: 4s - 6s
```

我的博客是预先部署在阿里云上，所以针对线上的环境，我们也可以使用 Network 来测试一下效果。

![](http://ojt6zsxg2.bkt.clouddn.com/36e90b7b12fe4b3f28c30a60cda4ea7d.png)

其中小的文件都需要一个 `waiting(TTFB)` 时间以后才获得第一个返回的比特，再经过蓝色的 `Content Download` 时间再获取到资源。虽然浏览器支持并发的下载外部资源，但是整体还是需要花费 `100ms` 的时间。针对这些时间，我们其实可以将这些 css 资源都以内联模式的形式写在 HTML 中，这样就不需要再经过 `waiting(TTFB)` 时间。

除此以外，图片的下载还是需要 4s 多的时间，说明对图片的压缩很必要。

### 文章页

#### Regular 2G

由于涉及的图片太多，Regular 2G 的测试太慢。。。所以不测试了。

#### 线上测试

![](http://ojt6zsxg2.bkt.clouddn.com/b8fe203e2b1d0e7db787727160f2adf4.png)

当博客的 Disqus 评论能够加载的时候，我们可以看到 `Disqus` 请求了很多额外的数据，导致网页的加载变得非常慢，所以必须要对评论框进行处理，将其改变成懒加载的形式。

除此以外，Disqus 因为被“墙”的缘故，经常加载不了，导致评论框不能用，所以针对评论框的修改，还是需要判断其 JS 文件是否能加载，如果不能加载，就调用 Disqus API 获得评论的数据。

![](http://ojt6zsxg2.bkt.clouddn.com/7f60c25e12bd4a879f32bc485de8bc01.png)

从这张图可以分析得到，CSS 文件的加载推迟了图片文件的加载，并且网页中图片文件没有设置懒加载，而是一起加载，超过了浏览器同源资源请求的并发数，导致 `header-bg.jpg` 文件和 `avatar.jpg` 文件的请求和下载被推迟，影响了侧边栏的显示效果。

## Chrome Timeline

同样，我们可以在[Chrome 开发者工具](https://developers.google.com/web/tools/chrome-devtools/?hl=zh-cn) 查看 Chrome 提供的浏览器 Timeline 的使用方法

使用 Chrome DevTools 的 Timeline 面板可以记录和分析您的应用在运行时的所有活动。 这里是开始调查应用中可觉察性能问题的最佳位置。

火焰图上看到一到三条垂直的虚线。蓝线代表 `DOMContentLoaded` 事件。 绿线代表首次绘制的时间。 红线代表 `load` 事件。

在我的博客中，由于涉及到的 JS 脚本和 CSS 重绘比较少，所以使用 Timeline 查看资源的使用率暂时获得不了太多信息。

## PageSpeed Insights

使用 [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/?hl=zh-CN) 可以对网页进行分析，并且提出针对网页性能的优化方案。使用的时候直接把链接输入即可。下面我使用这个网站对我的首页以及文章页进行分析：

### 首页

![](http://ojt6zsxg2.bkt.clouddn.com/638c79bd4bef272386fc054d38ff66db.png)

给出的优化策略如下：

1. 清除阻止呈现的 JS 和 CSS，如将 `<link>` 标签请求的资源使用 `<style>` 标签添加到 HTML 页面中
2. 优化图片，对图片的大小进行压缩
3. 对网站的传输格式使用 GZip 进行压缩

### 文章页

然后我再对现有的博客进行分析，得到以下分析结论：

![](http://ojt6zsxg2.bkt.clouddn.com/beaf84d6955fc12d14a40cc18c9d5520.png)

一共提出了五点优化策略：

1. 对文章的图片进行优化，包括图片压缩和懒加载
2. 对阻止呈现的 JS 和 CSS 进行优化，将 CSS 以内联的形式写在网页中
3. 使用浏览器缓存，对不经常变动的文件写入缓存中
4. 启用压缩，通过 GZip 对传输文件进行压缩
5. 缩减 JS 文件的大小，对 JS 文件进行压缩

除此之外，可以提供网页性能检测的网站还有：[GTmetrix](https://gtmetrix.com/) 和 [WebPagetest](https://www.webpagetest.org/)，大家不妨多去尝试一下。

## 参考

* [Measure Network Performance](https://developers.google.com/web/tools/chrome-devtools/network-performance/?hl=zh-cn)
* [了解 Resource Timing](https://developers.google.com/web/tools/chrome-devtools/network-performance/understanding-resource-timing?hl=zh-cn)


