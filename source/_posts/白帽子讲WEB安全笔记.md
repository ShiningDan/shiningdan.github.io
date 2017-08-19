---
title: 白帽子讲WEB安全笔记
date: 2017-08-19 15:02:14
categories: coding
tags:
  - WEB
  - 安全
---

本文是我读《白帽子讲 WEB 安全的笔记》

<!--more-->


## 浏览器安全

### 同源策略

**浏览器的同源策略，限制了来自不同源的 document 或者脚本，对当前 Document 的读取或设置某些属性**

源：host（域名或者 IP，如果是 IP 则看成一个根域名），子域名，端口，协议

**即使是相同根域名，不同子域名，也会受到同源策略的限制**

注意：存放文件的域名不重要，重要的是加载文件的域是什么样的。

如：

```
<script src='http://b.com/b.js'></script>
```

这段代码是加载在 `a.com` 这个页面下的，则 `b.js` 的 Origin 应该是 `a.com` 而不是 `b.com`

在浏览器中，`<script>、<img>、<iframe>、<link>` 等标签都可以通过 src 属性加载资源，但是**通过 src 加载的资源，浏览器限制了 JavaScript 的权限，使其不能读写返回的内容**

**浏览器的策略是，不阻止向另一个域名发送请求（表单，img，iframe，link 等 src 都可以跨域），但是不得读取另一个域名的内容**

W3C 委员会制定了 XMLHttpRequest 跨域访问的标准，它需要通过目标域返回的 HTTP 头 Origin 来授权是否允许跨域访问。并且 **JavaScript 无法控制该 HTTP 头**


## 跨站脚本攻击（XSS）

XSS 的定义：XSS 攻击，通常指黑客通过 “HTML 注入”篡改了网页，插入了恶意的脚本，从而在用户浏览网页的时候，控制用户浏览器的一种攻击

XSS 分为几种不同的类型：

1. 反射型 XSS：简单地把用户输入的数据“反射”给浏览器，诱使用户“点击”一个恶意链接，才能攻击成功。（**修改一个正常的 URL 并且发送给用户**）
2. 存储型 XSS：把用户输入的数据存储在服务器端，比如黑客写下一篇包含恶意的 JavaScript 代码的博客文章，所有阅读该文章的用户都会在他们的浏览器中执行这段恶意的 JavaScript 代码，黑客把恶意的脚本存储在服务器端。（**用户在访问正常 URL 的时候就会自动触发攻击**）
3. DOM Based XSS：通过修改页面的 DOM 节点来形成 XSS

### XSS 攻击的技巧

1. 获取用户传入的参数，没有将其包裹在双引号中作为字符串展现，而是直接插入 HTML 页面中，使得变量可以包含脚本等可执行程序，并且在用户页面中运行
2. 如果对用户输入做了严格的长度限制，还可以通过 `location.hash` 方法，绕过长度限制。
3. 使用 `<base>` 标签修改页面上所有标签的相对路径到远程服务器上。
4. 通过 window.name 在统一浏览器的标签的前后页面中传递用户信息。
5. Flash XSS 攻击

### XSS 攻击进阶

#### XSS 攻击能够实现的能力

##### Cookie 劫持

可以通过 `document.cookie` 来读取用户的 cookie，并且根据 cookie 来模拟其他用户进行登录。可以使用 Cookie 中的 `HttpOnly` 使得 document对象中看不到 cookie，来防止 cookie 劫持

##### 构造 GET、POST 请求

可以使用 XMLHttpRequest 来构造网络请求（理论上 JS 能做的事情 XSS 都可以做）

##### XSS 钓鱼

利用 JS 在页面上画出一个伪造的登录框等。

##### 识别用户浏览器

可以根据 `window.userAgent` 加上正则匹配来识别用户浏览器。但是 `userAgent` 是可以伪造的，比如 Firefox 很多扩展可以屏蔽或自定义浏览器发送的 UserAgent，所以信息不一定准确

所以可以根据浏览器的能力或属性来识别浏览器类型，比如 `window.opera` 来识别 Opera，`window.netscape` 来识别 Mozilla 等

##### 识别用户安装的软件

比如安装了迅雷以后，可以在浏览器的属性中检测到用户是否安装了迅雷软件

Flash 有一个 `system.capabilities` 对象，中统计了客户端电脑中的部分硬件信息。

`window.navigator.plugins` 可以检测用户安装了哪些插件。

可以根据检测扩展栏的图标来检测用户安装的扩展（Extension）

##### CSS History Hack

用户在访问过某些链接后，链接的颜色会变成其他的颜色，通过 `window.defaultView.getComputedStyle` 可以获得链接的颜色，然后判断用户是否访问过哪些链接。

现在这种通过 CSS 泄露用户信息的问题已经被浏览器修复了

##### 获取用户真实 IP

JavaScript 本身没有获得本地 IP 的能力，但是可以通过第三方软件来完成，比如客户端安装 Java 环境（JRE），使用 Attack API 等框架可以实现访问本地 JRE 的能力，然后获得 IP 等信息。

### XSS 的防御









