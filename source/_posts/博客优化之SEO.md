---
title: 博客优化之SEO优化
date: 2017-05-26 11:28:56
categories: coding
tags:
  - SEO
  - sitemap.xml
  - robots.txt
---

如何推广自己的博客，让自己的文章更快被搜索引擎收录，属于搜索引擎优化的范畴。如何设计出对于搜索引擎更加友好的页面，除了保证语义化的页面结构，使用友好的 URL 规则，在页面添加 Description 以外，还有很多可以提升的地方。本文中，我将介绍一下自己的博客中使用到的 SEO 相关的优化，例如 Sitemap 和 Robots

<!--more-->

## Sitemap

XML Sitemap 的主要作用是让搜索引擎特别是 Google 更好地收录网站所有 URL。XML Sitemap 是为了让你的站点更好地被收录。特别是当你的站点内容层次比较深，或者包含许多通过js或提交表单才能获得 URL 时，XML Sitemap 可以帮助搜索引擎机器人抓取原本不好获得的 URL。

我的博客中有一个持续更新的 [sitemap](/sitemap.xml) 文件。

### Sitemap 格式

`sitemap.xml` 作为一个 xml 文件，有其特殊的格式，下面介绍的是一个 Sitemap 文件具有的格式：

![](http://ojt6zsxg2.bkt.clouddn.com/f51e1c4b6996b84110992f77bf8b6cdf.png)

在处理 Sitemap 的时候，我在创建新文章和更新文章的地方都调用了一个函数 `updateSitemap`，这个函数处理的内容是，读取现有的所有文章，然后把文章中相关的数据按照格式写入 `sitemap.xml`，此时，每当我们添加一篇新的文章，或者修改一篇文章的时候，`sitemap.xml` 就会自动更新。

下面展示的是 `updateSitemap` 的部分内容：

```
let xmlbuilder = require('xmlbuilder');
let moment = require('moment');

var root = xmlbuilder.create('urlset', {version: '1.0', encoding: 'UTF-8'}).att('xmlns', 'http://www.sitemaps.org/schemas/sitemap/0.9');

root.ele('url').ele('loc', 'http://zyuchen.com/archives/').insertAfter('lastmod', moment().format('YYYY-MM-DD')).insertAfter('priority', 0.6);

root.ele('url').ele('loc', 'http://zyuchen.com/series/').insertAfter('lastmod', moment().format('YYYY-MM-DD')).insertAfter('priority', 0.6);

return Article.find({}).then((articles) => {
  articles.forEach((article) => {
    root.ele('url').ele('loc', 'http://zyuchen.com' + article.link).insertAfter('lastmod', moment(article.meta.updateAt).format('YYYY-MM-DD')).insertAfter('priority', 0.6);
  })
  fs.writeFile(sitemapPath, root.end({pretty: true}), { flag: 'wx' }, function(err) {
    if (err) {
      console.log(err);
    }
  })
});
```

通过 `updateSitemap` 创建的 `sitemap.xml` 内容如下：

```

<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>http://zyuchen.com/archives/</loc>
    <lastmod>2017-05-26</lastmod>
    <priority>0.6</priority>
  </url>
  <url>
    <loc>http://zyuchen.com/series/</loc>
    <lastmod>2017-05-26</lastmod>
    <priority>0.6</priority>
  </url>
  <url>
    <loc>http://zyuchen.com/post/alfred-workflow-qiniu</loc>
    <lastmod>2017-05-18</lastmod>
    <priority>0.6</priority>
  </url>
  <url>
    <loc>http://zyuchen.com/post/optimize-blog-statistic-resource</loc>
    <lastmod>2017-05-22</lastmod>
    <priority>0.6</priority>
  </url>
  <url>
    <loc>http://zyuchen.com/post/blog-cache-and-update</loc>
    <lastmod>2017-05-24</lastmod>
    <priority>0.6</priority>
  </url>
</urlset>
```

当我们创建完成 `sitemap.xml` 之后，还会主动调用各大搜索引擎的 ping 服务即时推送，主动告诉搜索引擎来爬取我的文章，

```
async function pingSpider(article, url) {
  let data = xmlbuilder.create('methodCall', {version: '1.0', encoding: 'UTF-8'});
  data.ele('methodName', 'weblogUpdates.extendedPing');
  let params = data.ele('params');
  params.ele('param').ele('value').ele('string', 'Yuchen 的主页');
  params.ele('param').ele('value').ele('string', 'http://zyuchen.com/');
  params.ele('param').ele('value').ele('string', 'http://zyuchen.com' + article.link);
  params.ele('param').ele('value').ele('string', ' '); // dont have rss
  return agent.post(url, data.end({pretty: true})).end().then((value) => console.log(value.body));
}
```

在参数 `url` 的部分，我们可以传入多个搜索引擎提供的 ping 服务地址，如

* 百度：http://ping.baidu.com/ping/RPC2
* 谷歌：http://blogsearch.google.com/ping/RPC2  // 测试的时候链接不上。
* Ping-o-Matic!：http://rpc.pingomatic.com/

## robots.txt

`robots.txt` 是一个纯文本文件，在这个文件中网站管理者可以声明该网站中不想被搜索引擎访问的部分，或者指定搜索引擎只收录指定的内容。

　　当一个搜索引擎（又称搜索机器人或蜘蛛程序）访问一个站点时，它会首先检查该站点根目录下是否存在 `robots.txt` ，如果存在，搜索机器人就会按照该文件中的内容来确定访问的范围；如果该文件不存在，那么搜索机器人就沿着链接抓取。

下面将介绍 `robots.txt` 中三种语法：`User-agent`、`Disavow` 和 `Allow`

### User-agent

定义哪些搜索引擎可以爬取。一般情况下是所有的搜索引擎都可以爬取。`User-agent` 有以下几种写法：

```
User-agent: *（定义所有搜索引擎）
User-agent: Googlebot （定义谷歌，只允许谷歌蜘蛛爬取）
User-agent: Baiduspider  （定义百度，只允许百度蜘蛛爬取）
```

### Disallow

定义禁止爬取的页面或目录：

```
Disallow: /（禁止蜘蛛爬取网站的所有目录 "/" 表示根目录下）
Disallow: /admin （禁止蜘蛛爬取admin目录）
Disallow: /abc.html （禁止蜘蛛爬去abc.html页面）
Disallow: /help.html （禁止蜘蛛爬去help.html页面）
```

### Allow

定义允许爬取的页面和目录

```
Allow: /admin/test/（允许蜘蛛爬取admin下的test目录）
Allow: /admin/abc.html（允许蜘蛛爬去admin目录中的abc.html页面）
```

### Sitemap

可以在 `robots.txt` 中添加 `Sitemap` 来引导爬虫根据 `sitemap.xml` 获取需要的数据：

```
Sitemap: http://zyuchen.com/sitemap.xml
```

### 使用方法

`robots.txt` 文件必须放在网站的根目录，也就是通过 `<Domain>.robots.txt` 就可以访问到你的网站的爬取策略。

`robots.txt` 文件名命名必须小写，记得在 `robot` 面加 `s`。

[我的 robots.txt](http://zyuchen.com/robots.txt)

我的 `robots.txt` 内容如下：

```
User-agent: *
Disallow: /search
Disallow: /disqus
Disallow: /data
Disallow: *.md
Allow: /
Sitemap: http://zyuchen.com/sitemap.xml
```

## 参考

* [本博客零散优化点汇总 | QuQu](https://imququ.com/post/summary-of-my-blog-optimization.html)
* [SEO入门指南](https://sdk.cn/news/4807)
* [hexo高阶教程：教你怎么让你的hexo博客在搜索引擎中排第一](https://juejin.im/post/590b451a0ce46300588c43a0)
* [Prerender + vuejs SEO最佳实践 百度爬虫 + 谷歌收录](http://www.deboy.cn/prerender-vuejs1-X-SEO-best-practice.html)
* [robots.txt作用和写法](http://www.admin10000.com/document/189.html)
* [为 Laravel 博客添加 SiteMap 和 RSS 订阅](https://lufficc.com/blog/add-site-map-and-rss-for-your-laravel-blog)









