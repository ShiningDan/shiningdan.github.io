---
title: 博客优化之 Google-Analytics 代理
date: 2017-06-24 20:46:24
categories: coding
tags:
  - Google Analytics
---

在博客中，我使用了 Google 的静态统计脚本来统计博客的访问量，页面访问信息，用户的地理位置等数据，当然这里面并没有暴露用户个人隐私的数据，而是我希望通过统计数据，得到热门的文章，从而实现对热门文章的缓存策略。默认的 Google 统计是下载了一个脚本文件并且在浏览器上运行，然后将统计数据发回到 Google 的统计服务器，但是由于网络的问题，可能用户所在的网络无法连接到服务器，所以我用自己的服务器作为一个代理，获得用户的统计请求，然后再转发给 Google 服务器。这篇文章主要讲的是我如何实现这一功能。

<!--more-->

## 开通 Google-Analytics 并下载代码

开通 Google-Analytics 很简单，这一部分我就不再详述了，大家可以点击 [Google Analytics（分析）](https://www.google.com/analytics/) 这一链接跳转到注册和登录首页。不过该网站在国内访问的速度比较慢，大家可以挂 VPN 进行访问或者。。。耐心等待（手动微笑）


注册完成以后，在 `管理 -> <你的网站名称> -> JS 跟踪信息 -> 跟踪代码` 中，可以找到为你生成的 Google 统计代码，类似于下面的样子，普通情况下，我们用一个 `<script>` 标签包裹这段代码，添加在 `<body>` 的最后面即可（像这种独立，并且首屏不依赖的脚本，建议添加在代码的最后面）。

```
<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','https://www.google-analytics.com/analytics.js','ga');

  ga('create', '<Your ID>', 'auto');
  ga('send', 'pageview');

</script>
```

代码的内容大致如上，做的事情是，从 `https://www.google-analytics.com/analytics.js` 中下载并执行其中的 `ga` 函数。


当我们在页面上添加该脚本之后，重启页面，打开 Chrome 调试工具，查看该脚本发送给 Google 服务器的信息。

从 Network 中我们可以看到，打开该页面以后，请求的文件变多了两个，一个是 `https://www.google-analytics.com/analytics.js`，当然这是 Google 的统计代码，另一个是一个以 `collec` 开头的 GET 请求。

![](http://ojt6zsxg2.bkt.clouddn.com/fdd66f8611d5b47727d559559afbc899.png)

这个 GET 请求中 query 中附带的参数以及 Cookie 附带的参数，就是网站当前用户浏览的数据统计值，也就是我需要用服务器转发的请求。

## 测试服务器的连通性

为了避免因为用户网络的问题导致统计请求无法发送到 Google 服务器，我们的做法是接收该 GET 请求，然后在我的服务器上再 **模拟用户数据发送一个 GET 请求给 Google 服务器**，当然，这首先要求我们的服务器可以连接到 Google 服务器。

在这里，我写了一个简易的脚本，周期性地请求 GET 请求的目的地址，也就是 `http://www.google-analytics.com`，然后在服务器上运行，获得我的服务器（阿里云上买的 ECS）到 Google 统计服务器的联通时间。脚本内容如下：

```
let schedule = require('node-schedule');
let request = require('superagent');
let fs = require('fs');

function testGaTime() {
  let startTime = new Date();
  let endTime;
  request.get("http://www.google-analytics.com/collect")
    .query("v=1&_v=j56&a=54916858&t=pageview&_s=1&...<你的 GET 中 query 的值>..._gid=1961109138.1497964145&z=1668004735")
    .end((err, data) => {
      if (err) {
        console.log(err)
      }
      endTime = new Date();
      console.log(endTime.getTime() - startTime.getTime());
      fs.appendFileSync('./galog.txt', endTime.getTime() - startTime.getTime() + " ", (err) => {
        console.log(err);
      })
    })
}

try {
  var rule = new schedule.RecurrenceRule();
  rule.minute = [0, 5, 10, 15, 20, 25, 30, 35, 40, 45, 50, 55];
  schedule.scheduleJob(rule, function(){
    testGaTime();
  }); 
} catch(e) {
  console.log(e);
}
```

其中的 query 参数是我打开页面后发送的 query 的值，你也可以换成自己的 query 值，然后将该代码做成一个脚本，运行在服务器上，最后，可以在 `./galog.txt` 中获得每次请求所需要的时间。

在运行了一天的脚本以后，我大致看了一下，从我的 ECS 发送请求，到获取 Google 统计服务器数据的时间，大致在 120ms 左右，可以接受，所以我决定使用 ECS 来发送用户的统计请求。

## 修改 Google-Analytics 脚本

原来脚本中，当获得完 query 参数以后，默认情况下是发送到 `http://www.google-analytics.com`，我们需要求改，发送到本地服务器，这段代码在 `oc` 函数中：

```
var oc = function () {
    return (Ba || Ud() ? "https:" : "http:") + "//www.google-analytics.com"
}
```

我们只需要修改返回值为 

```
return '';
```

即可。这样，发送的 GET 请求不会去寻找 `http://www.google-analytics.com` 域名，而是寻找网站的域名

接着，在 Express 中设置处理该路由的函数，我们在这里需要处理的路由有 `/collect` 和 `/r/collect`。前者是新用户的请求路径，后者是老用户的请求路径。

我在 `/collect` 中处理该请求的逻辑如下：

```
let request = require('superagent');

exports.ga = function(req, res) {
  let headers = req.headers;
  delete headers.host;
  let query = req.query;
  let queryZ = query.z;
  delete query.z;
  query['uip'] = req.connection.remoteAddress.split(":").reverse()[0];
  query['z'] = queryZ;
  request.get('http://www.google-analytics.com/collect')
    .query(query)
    .set(headers)
    .end((err, data) => {
      if (err) {
        console.log(err);
      } else {
        // console.log(data);        
      }
    })
  res.end();
}
```

这段代码的逻辑很简单，获取了用户请求中的 `headers` 和 `query` 中的参数，然后在服务器构造 HTTP GET 请求，将该参数赋值给新请求，由服务器来发送请求。

其中有几点需要注意：

### 删除 host

因为这里的 `host` 指的是本博客的域名 `http://zyuchen.com`，而不是 Google 统计的域名，所以讲该 `host` 放在新请求中发出，会导致该请求无法被接收。

### 重新设置 IP 

Google 统计中，判断用户的地理位置是根据请求的 IP 地址来寻址的。因为新请求是由我的 ECS 发出，IP 地址是我的虚拟服务器的地址，所以每一次 Google 统计所获得的地址都是 ECS 所在的云数据中心的地址。但是，根据 [Google Analytics Measurement Protocol 参考](https://developers.google.com/analytics/devguides/collection/protocol/v1/reference?hl=zh-cn)，我们可以在 query 中设置 `uip` 字段，来主动设置请求相关的 IP 地址，所以，在处理逻辑上，会获得用户的 IP 地址，然后设置在 `uip` 中

重设 z 字段

由于我们设置了 `uip` 字段，在 `superagent` 解析 query 的时候，会将 `uip` 作为 query 中最后一个值，但是根据  [Measurement Protocol 缓存无效化](https://developers.google.com/analytics/devguides/collection/protocol/v1/reference?hl=zh-cn#header) 要求，需要将 `z` 作为 query 中最后一个参数，所以我们需要删掉已有的 `z`，然后在最后重新赋值

这样，我们就完成了服务器对用户 Google 统计请求的转发功能了，当然，Google 统计中有很多我们不需要的统计的项目，我们可以阅读 [设置 Google Analytics（分析）](https://developers.google.com/analytics/devguides/collection/?hl=zh-cn) 这篇文档，然后选择我们需要的项目，来自定义我们的统计数据。

## 设置 LocalStorage 缓存

完整的 `analytics.js` 文件的大小有 68KB 左右，没有必要每一次打开页面都进行下载一遍，所以，通过设置 Cookie，来判断用户是否已经下载过 `analytics.js` 文件。如果用户是首次访问网站，则将 `analytics.js` 作为内联脚本添加到 HTML 中，并且将该脚本的内容下载到 LocalStorage 里面。如果用户不是是首次访问，并且在本地拥有 `analytics.js` 的副本，则调用本地的脚本内容，在浏览器端添加到页面中即可。

首先定义将脚本内容存储到 LocalStorage 中的函数 `ls` 和从 LocalStorage 中读取内容的函数 `ll`：

```
function ls(name, tag) {
  let target = document.getElementById(name);
  if (!target) {
    throw new Error('ls save fn not find ' + name);
  } else {
    if (window.localStorage) {
      try {
        window.localStorage.setItem(name, target.innerHTML);
        document.cookie = name + '=' +tag;
      } catch(e) {
        console.log(e);
      }
    }
  }
}
function ll(name, isScript) {
  let storage = window.localStorage.getItem(name);
  if (!storage) {
    // 如果 cookie 中存在，但是在 localstorage 中找不到需要如何处理？先删除 cookie，然后刷新页面。
    document.cookie = 'v=; expires=Thu, 01 Jan 1970 00:00:01 GMT;';
    // window.location.reload();
    throw new Error('ls load fn not find ' + name);
  } else {
    let type = isScript ? 'script' : 'style';  // 设置 0 来添加 style，设置 1 来添加 script
    let elem = document.createElement(type);
    elem.innerHTML = storage;
    document.head.appendChild(elem);
  }
}
```

然后，对用户的 Cookie 进行验证，判断用户是否在本地拥有 `analytics.js` 的代码，这里设置的 `analytics.js` 的版本和 Google 为 `analytics.js` 设置的脚本保持一致，都为 `1`：

```
let gavTag = "1";

if (req.cookies.analytics && req.cookies.analytics === gavTag) {
    req.gav = true;
} else {
    req.gav = false;
    req.gavTag = gavTag;
}
next();
```

最后，设置模板引擎逻辑，判断根据 `gav` 来判断是否要内联 `analytics.js` 中的代码，我这里用的模板库为 Pug：

```
block googleAnalystic
  - if (!gav)
    script#analytics
      include ../../www/static/js/analytics.js
    script ls('analytics', '#{gavTag}');
  - else
    script ll('analytics', true)
```

在这里，我对 `analytics.js` 中的内容做了一点小加工，第一是，修改 `oc` 函数中返回值，设置为 `return ""`，第二是，将原来 Google 统计推荐的 JS 追踪代码逻辑放在了 `analytics.js` 的最底部，在运行 `analytics.js` 中配置函数的逻辑之后，就会调用配置好的函数，添加在最底部的代码如下：

```
window['GoogleAnalyticsObject'] = 'ga';
window['ga'] = window['ga'] || function() {
  (window['ga'].q = window['ga'].q || []).push(window, document, 'script', '/js/analytics.js', 'ga');
}
window['ga'].l = 1 * new Date();

ga('create', 'UA-101336456-1', 'auto');
ga('send', 'pageview');
```

这样，在用户加载了 `analytics.js` 文件中的内容以后，就会自动调用并执行 `ga` 函数了。

## 参考

* [设置 Google Analytics（分析）](https://developers.google.com/analytics/devguides/collection/?hl=zh-cn) 









