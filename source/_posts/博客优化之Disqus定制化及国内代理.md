---
title: 博客优化之Disqus定制化及代理
date: 2017-07-02 23:57:04
categories: coding
tags:
  - JavaScript
  - Node.js
  - Disqus
---

Disqus 是一个第三方整合的留言系统，除了可以让使用者通过第三方账户进行登录留言外，还支持匿名留言选项。在多说关闭以后，Disqus 成为了一个最好的选择。虽然在博客上使用 Disqus 很简单，但是在国内却会被墙，所以我最终使用 Disqus API 加国外 VPS 的模式为博客下留言的用户提供了代理。并且针对 Disqus 加载时会下载过多资源的现象，自己实现了一个针对博客定制版的评论系统。

<!--more-->

## Disqus 分析

### 使用 Disqus 自带的脚本

如果在页面中使用 Disqus 自带的脚本，其实是一个不复杂的事情，我们要做的就是在 Disqus 上注册，然后为自己的域名创建一个 site，在 Disqus 上下载在网页中插入的脚本代码。在本站中，我的脚本代码如下：

```
/**
*  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
*  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables*/
/*
var disqus_config = function () {
this.page.url = PAGE_URL;  // Replace PAGE_URL with your page's canonical URL variable
this.page.identifier = PAGE_IDENTIFIER; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
};
*/
var disqus_identifier = window.location.href.split('/post/')[1];
// var disqus_url = window.location.href;

(function() { // DON'T EDIT BELOW THIS LINE
var d = document, s = d.createElement('script');
s.src = 'https://www-zyuchen-com.disqus.com/embed.js';
s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();
```

其中，

```
var disqus_identifier = window.location.href.split('/post/')[1];
```

是我自己添加的，用来为当前页面设定一个 identifier url，以便于使用该 url 查询该页面上的所有评论信息。

以上的脚本内容可以添加到需要添加评论框的页面中，刷新页面就可以运行以上代码，在页面中嵌入评论框并且获取历史评论。

接下来，我们来分析一下，Disqus 在页面上的表现。

![](http://ojt6zsxg2.bkt.clouddn.com/c08cb067cf75703dc8aa99fa793b30fb.png)

这张图中展示的是我在 `http://zyuchen.com/post/TCP-flow-congestion-control` 中添加的测试评论，Disqus 在页面中显示的样式，我们可以看到，评论框的布局很合理，Disqus 还是很值得使用的~

接下来，我们来分析一下使用 Disqus 的缺点。

### 国内访问被墙

上面显示 Disqus 评论框，是在我本地的机器已经翻墙的条件下才能下载相关的 JS 资源。如果作为一个用户，其本地的浏览器并没有翻墙能力的时候，Disqus 的表现又如何呢？

当我关闭本地的翻墙软件后，Disqus 的加载效果如下：

![](http://ojt6zsxg2.bkt.clouddn.com/42dff48ede51ca0db9969e9d636d7331.png)

![](http://ojt6zsxg2.bkt.clouddn.com/fd42082a7f7bf88550edb504efde6cf9.png)

我们可以看到，Disqus 相关的脚本在用户浏览器没有翻墙的情况下，是无法下载的，此时，页面中的显示效果如下：

![](http://ojt6zsxg2.bkt.clouddn.com/2adca079cf49c21df8adbc4ba9dff6a7.png)

原来评论框的部分完全没有显示。作为一个博客，我们很难要求访问的用户都具备翻墙的能力，或者当前就处在翻墙的状态下。所以，针对不能翻墙的用户，我们还是需要提供代理，为他们获得“墙外”的留言。

### Disqus 资源加载

在使用 Disqus，还有一点让我最不能忍受的，可以见下图：

![](http://ojt6zsxg2.bkt.clouddn.com/cd94f1a59a5d4dc462c9f46db37affc9.png)

明明我博客的内容很快就加载完了，但是 Disqus 在加载自己的评论框的时候，居然加载了这么多的资源！从图中我们可以看到，在最开始的紫色竖线之前是加载我的博客的内容，紫色竖线之后全部都是加载 Disqus 的内容，整整运行了 10s，即使是这一部分作为懒加载来处理，需要 10s 才能加载完成所有的评论框内容，这是不能忍受的。因为这一点，我才下决心为博客中评论框的部分使用代理来获取数据，并且自己进行评论框的渲染。下面是我的做法

## 为 Disqus 搭建代理














## 参考

* [Disqus API Resources](https://disqus.com/api/docs/)
* [Disqus，我又回来了！| 屈屈老师](https://imququ.com/post/back-to-disqus.html)












