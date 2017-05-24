---
title: Web缓存策略总结
date: 2017-05-23 17:19:15
categories: coding
tags:
  - 资源缓存
---

本文是我在学习的过程中自己总结的一些关于 Web 缓存策略的一些要点，有前端相关的，也有后端相关。

<!--more-->

在介绍缓存与更新之前，有几篇文章大家可以参考：

* [前端工程与性能优化](https://github.com/fouber/blog/issues/3)
* [HTTP1.1与前端性能](http://imweb.io/topic/554c5879718ba1240cc1dd8a)

## 总体介绍

在下图中，展示了一些常用的缓存策略，原图以及文章可以在 [缓存策略](http://imweb.io/topic/55c6f9bac222e3af6ce235b9) 这篇博客中找到。

![](http://ojt6zsxg2.bkt.clouddn.com/89d71570371948c678ba1139c9538641.png)

下面，我们会针对这张图中的一些要点，以及其他的一些要点，介绍一下 Web 缓存相关的知识。

## 用户的刷新和缓存

![](http://ojt6zsxg2.bkt.clouddn.com/dbae71ff2ef6174158bbc842dd1f2c11.png)

浏览器中的操作对缓存的影响:

* 强制刷新 – 当按下ctrl+F5来刷新页面（Mac 下是 Command + Shift + R）的时候, 浏览器将绕过各种缓存(本地缓存和协商缓存), 直接让服务器返回最新的资源;
* 普通刷新 – 当按下F5来刷新页面（Mac 下是 Command + R）的时候,浏览器将绕过本地缓蹲来发送请求到服务器, 此时, 协商缓存是有效的
* 回车或转向 – 当在地址栏上输入回车或者按下跳转按钮的时候, 所有缓存都生效

## 浏览器的缓存策略

当一个用户发起一个静态资源请求的时候，浏览器会通过以下几步来获取资源：

1. **本地缓存阶段**：先在本地查找该资源，如果有发现该资源，而且该资源还没有过期，就使用这一个资源，完全不会发送http请求到服务器；
2. **协商缓存阶段**：如果在本地缓存找到对应的资源，但是不知道该资源是否过期或者已经过期，则发一个http请求到服务器,然后服务器判断这个请求，如果请求的资源在服务器上没有改动过，则返回304，让浏览器使用本地找到的那个资源；
3. **缓存失败阶段**：当服务器发现请求的资源已经修改过，或者这是一个新的请求(在本来没有找到资源)，服务器则返回该资源的数据，并且返回200， 当然这个是指找到资源的情况下，如果服务器上没有这个资源，则返回404。

## 本地缓存阶段

![](http://ojt6zsxg2.bkt.clouddn.com/34346a1019704bd9db8ba8db1e495c67.png)

我们可以看到，本地缓存相关的设置，大多属于响应（`response`） 的，也就是服务器设置客户端什么时候才需要再次请求资源。**但是有些值也可以在请求（`request`）中进行设置**。上面图片中的值不一定还继续能用，在使用之前建议去查找最新的支持情况。

### Expires

指定缓存到期GMT的绝对时间，如果设了 `Cache-Control: max-age`，`max-age` 就会覆盖 `Expires`。如果 `Expires` 到期需要重新请求。

### Cache-Control

每个属性值的区别可以去 [Cache-Control | MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Cache-Control) 查看。

`Cache-Control`：是http 1.1中为了弥补 `Expires` 缺陷新加入的。

#### 缓存请求指令

**客户端**可以在HTTP请求中使用的标准 `Cache-Control` 指令。

```
Cache-Control: max-age=<seconds>
Cache-Control: max-stale[=<seconds>]
Cache-Control: min-fresh=<seconds>
Cache-control: no-cache 
Cache-control: no-store
Cache-control: no-transform
Cache-control: only-if-cached
```

`Cache-Control/Expires` 通常和 `Last-Modified/ETag` 一起使用。

`Cache-Control/Expires` 的优先级要高于 `Last-Modified/ETag` 。即当本地副本根据 `Cache-Control/Expires` 发现还在有效期内时，则不会再次发送请求去服务器询问修改时间（ `Last-Modified` ）或实体标识（ `Etag` ）了。

#### 缓存响应指令

**服务器**可以在响应中使用的标准 `Cache-Control` 指令。

```
Cache-control: must-revalidate
Cache-control: no-cache
Cache-control: no-store
Cache-control: no-transform
Cache-control: public
Cache-control: private
Cache-control: proxy-revalidate
Cache-Control: max-age=<seconds>
Cache-control: s-maxage=<seconds>
```

## 协商缓存阶段

### Last-Modified & If-Modified-Since

![](http://ojt6zsxg2.bkt.clouddn.com/25ab8adac8f687e28d091dee5805d229.png)

`Last-Modified` 与 `If-Modified-Since` 是一对报文头，属于http 1.0。
`Last-Modified` 是WEB服务器认为对象的最后修改时间，比如文件的最后修改时间，动态页面的最后产生时间。

使用的方法是，第一次服务器**响应**时，会设置资源的 `Last-Modified` 时间。当浏览器第二次请求同一资源的时候，会自动把前一次 `Last-Modified` 的时间发送给服务器。

### ETag & If-None-Match

![](http://ojt6zsxg2.bkt.clouddn.com/bd6d74a330043fe64ffa9aa4b1fa24b9.png)

`ETag` 与 `If-None-Match` 是一对报文，属于http 1.1。

`ETag` 可以用来解决这种问题。`ETag` 是一个文件的唯一标志符。就像一个哈希或者指纹，每个文件都有一个单独的标志，只要这个文件发生了改变，这个标志就会发生变化。

`ETag` 机制类似于乐观锁机制，如果请求报文的 `ETag` 与服务器的不一致，则表示该资源已经被修改过来，需要发最新的内容给浏览器。

同时使用这两个报文头，在完全匹配 `If-Modified-Since` 和 `If-None-Match` 即检查完修改时间和 `Etag` 之后，如都与服务器的相符，服务器返回304，否则，发送最新内容给浏览器。

`Last-Modified` 与 `ETag` 是可以一起使用的，服务器会优先验证 `ETag` ，一致的情况下，才会继续比对 `Last-Modified` ，最后才决定是否返回 `304`。

### Etag/Last-Modified过程如下：

1. 客户端请求一个页面（A）。
2. 服务器返回页面A，并在给A加上一个 `Last-Modified/ETag`。
3. 客户端展现该页面，并将页面连同 `Last-Modified/ETag` 一起缓存。
4. 客户再次请求页面A，并将上次请求时服务器返回的 `Last-Modified/ETag` 一起传递给服务器。
5. 服务器检查该 `Last-Modified` 或 `ETag`，并判断出该页面自上次客户端请求之后还未被修改，直接返回响应 `304` 和一个空的响应体。

#### Etag 主要为了解决 Last-Modified 无法解决的一些问题

1. 一些文件也许会周期性的更改，但是他的内容并不改变(仅仅改变的修改时间)，这个时候我们并不希望客户端认为这个文件被修改了，而重新GET
2. 某些服务器不能精确的得到文件的最后修改时间。

一般情况下，使用 `Cache-Control/Expires` 会配合 `Last-Modified/ETag` 一起使用，因为即使服务器设置缓存时间, 当用户点击“刷新”按钮时，浏览器会忽略缓存继续向服务器发送请求，这时 `Last-Modified/ETag` 将能够很好利用 `304` ，从而减少响应开销。

![](http://ojt6zsxg2.bkt.clouddn.com/3b34b77c5c86a397dbe6cf4ec65ab837.png)

## Vary: Accept-Encoding

这个头对于一些人来说可能比较陌生。

当一个资源启用了 gzip 压缩，并且被代理服务器缓存，客户端如果不支持 gzip 压缩，那么在这样的情况下将会得到不正确的数据（也就是，压缩过的数据）。这将会使代理服务器缓存两个版本的资源：一个是压缩过的，一个是没压缩过的。正确版本的资源将在请求头发送之后进行传输。

还有一个现实的原因：IE 浏览器不缓存任何带有 `Vary` 头但值不为 `Accept-Encoding` 和 `User-Agent` 的资源。所以通过这种方式添加这个头，才能确保这些资源在 IE 下被缓存。

## 服务器端缓存

### 代理服务器缓存

代理服务器是浏览器和源服务器之间的中间服务器，浏览器先向这个中间服务器发起Web请求，经过处理后（比如权限验证，缓存匹配等），再将请求转发到源服务器。代理服务器缓存的运作原理跟浏览器的运作原理差不多，只是规模更大。可以把它理解为一个共享缓存，不只为一个用户服务，一般为大量用户提供服务，因此在减少相应时间和带宽使用方面很有效，同一个副本会被重用多次。

### CDN缓存

CDN全称是Content Delivery Network，即内容分发网络。其目的是通过在现有的Internet中增加一层新的网络架构，将网站的内容发布到最接近用户的网络“边缘”，使用户可以就近去的所需的内容

![](http://ojt6zsxg2.bkt.clouddn.com/69c670aadad2beadaa0f20b295717008.png)

我们可以了解到使用CDN缓存后的网站访问过程为：

1. 用户向浏览器提供要访问的域名。
2. 浏览器调用域名解析库对域名进行解析，由于CDN对域名解析过程进行了调整，所以解析函数库一般得到的是该域名对应的CNAME记录，为了得到实际的IP地址，浏览器需要再次获得对CNAME域名进行解析以得到实际的IP地址，在此过程中，使用的全局负载均衡DNS解析，如根据地理位置信息解析对应的IP地址，使得用户能就近访问。
3. 此次解析得到CDN缓存服务器的IP地址，浏览器在得到实际的IP地址以后，向缓存服务器发出访问请求。
4. 若请求文件未修改，返回304（充当服务器的角色）。若档期文件已过去，则缓存服务器根据浏览器提供的要访问的域名，通过Cache内部专用的DNS解析得到此域名的实际IP地址，再由缓存服务器向此实际IP地址提交访问请求。
5. 缓存服务器从实际IP地址得到内容以后，一方面在本地进行保存，以备以后使用，二方面把获取的数据返回给客户端，完成数据服务过程。
6. 客户端得到由缓存服务器返回的数据以后显示出来并完成整个浏览器的数据请求过程。

#### CDN的优势

* CDN节点解决了跨运营商和跨地域访问的问题，访问延时大大降低；
* 大部分请求在CDN边缘节点完成，CDN起到了分流作用，减轻了源站的负载。

### Combo服务

Combo服务，也就是我们在最终拼接生成页面资源引用的时候，并不是生成多个独立的link标签，而是将资源地址拼接成一个url路径，请求一种线上的动态资源合并服务，从而实现减少HTTP请求的需求。

#### Combo 缺点

* 浏览器有url长度限制，因此不能无限制的合并资源。
* 如果用户在网站内有公共资源的两个页面间跳转访问，由于两个页面的combo的url不一样导致用户不能利用浏览器缓存来加快对公共资源的访问速度。
* 如果combo的url中任何一个文件发生改变，都会导致整个url缓存失效，从而导致浏览器缓存利用率降低。

## 数据库数据缓存

Web应用，特别是SNS类型的应用，往往关系比较复杂，数据库表繁多，如果频繁进行数据库查询，很容易导致数据库不堪重荷。为了提供查询的性能，会将查询后的数据放到内存中进行缓存，下次查询时，直接从内存缓存直接返回，提供响应效率。比如常用的缓存方案有 [memcached](https://memcached.org/)、[redis](https://redis.io/) 等。

## HTML5缓存思路

### manifest

#### 离线应用访问及更新流程

1. 第一次访问离线应用的入口页HTML（引用了manifest文件），正常发送请求，获取manifest文件并在本地缓存，陆续拉取manifest中的需要缓存的文件
2. 再次访问时，无法在线离线与否，都会直接从缓存中获取入口页HTML和其他缓存的文件进行展示。如果此时在线，浏览器会发送请求到服务器请求manifest文件，并与第一次访问的副本进行比对，如果发现版本不一致，会陆续发送请求重新拉取入口文件HTML和需要缓存的文件并更新本地缓存副本
3. 之后的访问重复第2步的行为

#### manifest 文件

manifest文件罗列了需要被缓存的文件清单。

```
CACHE MANIFEST
# wanz app v1

# 指明缓存入口
CACHE:
index.html
style.css
images/logo.png
scripts/main.js

# 以下资源必须在线访问
NETWORK:
login.php

# 如果index.php无法访问则用404.html代替
FALLBACK:
/index.php /404.html
```

这个过程中有几个问题需要注意：

* 如果服务器对离线的资源进行了更新，那么必须更新manifest文件之后这些资源才能被浏览器重新下载，如果只是更新了资源而没有更新manifest文件的话，浏览器并不会重新下载资源，也就是说还是使用原来离线存储的资源。
* 对于manifest文件进行缓存的时候需要十分小心，因为可能出现一种情况就是你对manifest文件进行了更新，但是http的缓存规则告诉浏览器本地缓存的manifest文件还没过期，这个情况下浏览器还是使用原来的manifest文件，所以对于manifest文件最好不要设置缓存。
* 浏览器在下载manifest文件中的资源的时候，它会一次性下载所有资源，如果某个资源由于某种原因下载失败，那么这次的所有更新就算是失败的，浏览器还是会使用原来的资源。
* 在更新了资源之后，新的资源需要到下次再打开app才会生效，如果需要资源马上就能生效，那么可以使用 `window.applicationCache.swapCache()` 方法来使之生效，出现这种现象的原因是浏览器会先使用离线资源加载页面，然后再去检查manifest是否有更新，所以需要到下次打开页面才能生效。

### localStorage

```
localStorage.fresh = "shiningdan";   //设置一个键值 
var a = localStorage.fresh;   //获取键值

//API

//清空storage 
localStorage.clear();

//设置一个键值 
localStorage.setItem(“fresh”,“vfresh.org”);

//获取一个键值 
localStorage.getItem(“fresh”); 

//return “vfresh.org” //获取指定下标的键的名称（如同Array） 
localStorage.key(0); 

//return “fresh” //删除一个键值 
localStorage.removeItem(“fresh”);
```

#### 不总是有用

如果已经缓存的资源超过浏览器可允许的 LocalStorage 缓存的最大值，则浏览器会抛出一个错误 `QUOTA_EXCEEDED_ERR`

```
if (window.localStorage) {
    try {
        localStorage.setItem('bla', 'bla');
    } catch (e) {
        if (e.name === 'QUOTA_EXCEEDED_ERR' || e.name === 'NS_ERROR_DOM_QUOTA_REACHED') {
            // todo
        } else {
            // todo
        }
    }
}
```

#### 使用 LocalStorage 的规划

1. 只保存重要页面的重要数据
    * 典型的，首页首屏
    * 对业务庞大的站点，这点尤其重要

2. 极大提高用户体验的数据
    * 比如表单的状态，可以提交之前保存，当用户刷新页面时可以还原
    * 静态资源，比如 js 和 css

## 无法被缓存的请求

无法被浏览器缓存的请求：

1. HTTP信息头中包含Cache-Control:no-cache，pragma:no-cache，或Cache-Control:max-age=0等告诉浏览器不用缓存的请求
2. 需要根据Cookie，认证信息等决定输入内容的动态请求是不能被缓存的
3. 经过HTTPS安全加密的请求（有人也经过测试发现，ie其实在头部加入 `Cache-Control：max-age` 信息，firefox在头部加入 `Cache-Control:Public` 之后，能够对HTTPS的资源进行缓存，参考[《HTTPS的七个误解》](http://www.ruanyifeng.com/blog/2011/02/seven_myths_about_https.html)）
4. POST请求无法被缓存
5. HTTP响应头中不包含 `Last-Modified/Etag`，也不包含 `Cache-Control/Expires` 的请求无法被缓存

## 缓存的建议

### 给Css、js、图片等资源增加HTTP缓存头，并强制入口Html不被缓存

对于不经常修改的静态资源，比如Css，js，图片等，可以设置一个较长的过期的时间，或者至少加上 `Last-Modified/Etag` 来表示版本号 ，而对于html页面这种入口文件，不建议设置缓存。这样既能保证在静态资源不变了情况下，可以不重发请求或直接通过 `304` 避免重复下载，又能保证在资源有更新的，只要通过给资源增加时间戳或者更换路径，就能让用户访问最新的资源

使用这些 HTTP 头：

```
Cache-Control: public, max-age=31536000
Expires: (一年后的今天)
ETag: (基于内容生成)
Last-Modified: (过去某个时间)
Vary: Accept-Encoding
```

### 动态资源的缓存

针对应用程序私密性和新鲜度方面需求的不同，我们应该使用不同的缓存控制设置。


对于非私密性和经常性变动的资源（想像一下股票信息），我们应该使用下面这些：

```
Cache-Control: public, max-age=0
Expires: (当前时间)
ETag: (基于内容生成)
Last-Modified: (过去某个时间)
Vary: Accept-Encoding
```

这些设置的效果是：这些资源可以被公开地（通过浏览器和代理服务器）缓存起来。每一次在浏览器使用这些资源之前，浏览器或者代理服务器会检查这些资源是否有更新的版本，如果有，就把它们下载下来。

这样的设置需要注意，浏览器在重新检查资源时效性方面有一定的灵活性。典型的是，当用户点击了「返回／前进」按钮时，浏览器不会重新检查这些资源文件，而是直接使用缓存的版本。你如果需要更严格的控制，需要告知浏览器即使当用户点击了「返回／前进」按钮，也需要重新检查这些资源文件，那么可以使用：


```
Cache-Control: public, no-cache, no-store
```


不是所有的动态资源都会马上变成过时的资源。如果它们可以保持至少5分钟的时效，可以使用：


```
Cache-Control: public, max-age=300
```


经过这样的设置，浏览器只会在5分钟之后才重新检查。在这之前，缓存的内容会被直接使用。如果在5分钟后，这些过时的内容需要严格控制，你可以添加 `must-revalidate` 字段：


```
Cache-Control: public, max-age=300, must-revalidate
```

对于私密或者针对用户的内容，需要把 public 替换为 private 以避免内容被代理缓存。

```
Cache-Control: private, …
```

### 减少对Cookie的依赖

过多的使用Cookie会大大增加HTTP请求的负担，每次GET或POST请求，都会把Cookie都带上，增加网络传输流量，导致增长交互时间；同时Cache是很难被缓存的，应该尽量少使用，或者这在动态页面上使用。

### 多用Get方式请求动态CGI

虽然POST的请求方式比Get更安全，可以避免类似密码这种敏感信息在网络传输，被代理或其他人截获，但是Get请求方式更快，效率更高，而且能被缓存，建议对于那些不涉及敏感信息提交的请求尽量使用Get方式请求


## 参考

* [缓存策略](http://imweb.io/topic/55c6f9bac222e3af6ce235b9)
* [Cache-Control | MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Cache-Control)
* [Web缓存机制系列](http://www.alloyteam.com/2012/03/web-cache-1-web-cache-overview/)
* [浅谈Web缓存](http://www.alloyteam.com/2016/03/discussion-on-web-caching/)
* [说说 web 缓存](https://juejin.im/entry/583e933861ff4b006ce3e71c)
* [HTTP 缓存](https://aotu.io/notes/2016/09/22/http-caching/)
* [你所知道的3xx状态码](https://aotu.io/notes/2016/01/28/3xx-of-http-status/)






