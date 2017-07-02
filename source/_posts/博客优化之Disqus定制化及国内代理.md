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













