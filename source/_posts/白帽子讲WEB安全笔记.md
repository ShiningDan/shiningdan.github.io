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

在浏览器中，`<script>、<img>、<iframe>、<link>` 等标签都可以通过 src 属性加载资源，但是**通过 src 加载的资源，浏览器限制了 JavaScript 的权限，使其不能读写返回的内容**。并且，在通过 src 访问资源的时候，会附带被请求域的 cookie。

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

#### HttpOnly

标有 HttpOnly 的 Cookie，页面的 JavaScript 都不允许访问

服务器可能设置多个 Cookie，而 HttpOnly 可以选择性地加在任何一个 Cookie 上面。然后在输入 `document.cookie` 的时候，返回的值是没有设置 HttpOnly 的 Cookie 值。

#### 输入检查

常见的 Web 漏洞如 XSS，SQL 注入，都要求攻击者构造一些特殊字符串，这些字符串可能是用户不会用到的，所以输入检查就很有必要了。

**前端的输入检查很容易被攻击者绕过，目前的做法是在客户端 JavaScript 和服务器端代码中实现相同的输入检查，客户端的输入检查可以节约服务器资源。**

比如：

1. 用户的用户名只能限为字母，数字和某些符号（不包括：`<`、`>`、`、` 等危险字符）的组合
2. 比如电话、邮件、生日等都有自己特殊的规范格式
3. 比较智能的 XSS 输入检查，还可以匹配 XSS 的特征，比如用户输入的数据中是否包含 `<script>`，`javascript` 等敏感信息

#### 输出检查

除了富文本输入外，在变量输出到 HTML 页面时，可以使用编码或转义的方式来防御 XSS 攻击

##### 针对 HTML 的编码

为了对抗 XSS，在 HTMLEncode 中要求至少转义以下的字符：`&, <, >, ", ', /`，将其转义为 `&amp;, &lt;, &gt;, &quot;, &#x27;, &#x2F`

##### 针对 JavaScript 的转义

JavaScript 中对于特殊字符的转义方法和 HTMLEncode 的方法不同，它需要使用 `\` 符号进行转义。

在对抗 XSS 的时候，**还需要输出的变量在引号内部，避免造成安全问题**。

如：

```
var x = escapeJavaScript($evil);
var y = '"' + escapeJavaScript($evil) + '"';
```

转义后输出就变成：

```
var x = 1;alert(2);
var y = "1;alert(2);"
```

还有更加严格的 JavaScript 编码方式：除了数字，字母以外的所有字符，都使用十六进制的 `\xHH` 的方式进行编码。

##### 结合多种编码方式

XSS 攻击主要发生在 MVC 中的 View 层，也就是将变量输出到模板的时候。

使用单一的编码方式也不能完全防御 XSS 攻击，还需要针对不同的情况使用多种编码方式结合的方法来防御。

如何针对不同的情景防御 XSS，请参考《白帽子讲 WEB 安全》3.3.4 节，正确地防御 XSS。

#### 处理富文本

一些网站允许用户提交自定义的 HTML 代码，被称为富文本，这些富文本也需要进行 XSS 防御。

针对富文本的防御机制有：

1. “事件”应该被明确禁止
2. 一些危险的标签，比如`<iframe>、<script>、<base>、<form>`等标签也需要被禁止。
3. 在标签的选择上，应该使用白名单，而不是黑名单。
4. 允许用户自定义 CSS，style 也可能导致 XSS 攻击

#### 防御 DOM Based XSS

DOM Based XSS 是**从 JavaScript 中直接输出数据到 HTML 页面中导致的，前面的方法都是针对服务器直接输出变量到模板 HTML 导致的**

针对 DOM Based XSS，防御方法也是要进行输入检查和输出检查：

1. 变量输入的时候，要进行一次 JavaScriptEncode
2. 变量输出的时候，如果输出到事件或者脚本中，则需要再做一次 JavaScriptEncode，如果是输出到 HTML 内容或者属性中，则要做一次 HTMLEncode。

DOM Based XSS **需要关注的输入**的地方

1. 页面中所有的 input 框
2. window.location(href, hash 等)
3. window.name
4. document.cookie
5. localstorage
6. XMLHttpRequest 返回的数据

DOM Based XSS **需要关注的输出**的地方：

1. document.write() / docuemnt.writeln()
2. xxx.innetHTML / xxx.outerHTML
3. innerHTML.replace
4. document.attachEvent / window.attachEvent
5. document.location.replace()
6. document.location.saaign()

## 跨站点请求伪造（CSRF）

诱导用户访问了攻击的页面，然后通过 img 或者其他标签的 src 来发送 GET 请求，并且该请求可以附带用户的 Cookie，就以用户的身份执行了一次操作。

### 浏览器的 Cookie 策略

浏览器的 Cookie 分为两种：

1. session cookie：没有指定 Expire 时间，所以在浏览器关闭之后，cookie 会消失
2. 本地 cookie（第三方 Cookie）：指定了 Expire 时间，只有到 Expire 时间后 Cookie 才会消失

**Session Cookie 的生存时间为浏览器进程的生存之间，即浏览器打开了新的 Tab 页面，Session Cookie 也是有效的。Session Cookie 保存在浏览器进程的内存空间中。**

如果浏览器从一个域的页面中，要加载另一个域的资源，某些浏览器会阻止本地 Cookie 的发送（IE、Safari），也有的浏览器不会阻止（FireFox、Opera、Chrome、Android 等）

如果浏览器不会阻止用于验证的本地 Cookie 的发送，则很容易导致 CSRF 攻击成功。

而对于 IE 浏览器，可以先诱导用户在当前浏览器中访问目标站点，使得 Session Cookie 有效，然后实施 CSRF。

### P3P 头的副作用

P3P Header 全称是 The Platform for Privacy Preferences。用于允许跨域设置 Cookie，并且允许浏览器的 `<iframe>、<script>` 等标签发送第三方 Cookie。这样可能导致可以发送第三方 Cookie 使得 CSRF 攻击成功。

### GET？POST?

CSRF 攻击大多使用的 HTML 标签 `<img>、<iframe>、<script>` 等带 src 属性发送 GET 请求。

如果服务器针对 GET 请求进行 CSRF 防御，攻击者也可以通过构造 POST 请求来发起攻击，比如攻击页面使用 `<form>` 标签构造 POST 请求，提交到目标网站。通过 `<form>` 的 action 中设置的目标网站的地址，发送的 Cookie 是目标网站的 Cookie，则完成 POST CSRF 攻击。

### CSRF 的防御

#### 验证码

CSRF 攻击在于用户不知情的情况下构造了网络请求，而验证码强制用户与应用进行交互，才能完成最终请求。

#### Referer Check

Referer Check 目前最常见的应用就是“防止图片盗链”。

Refer 的值指向请求来自的页面，不过缺点是**服务器并非什么时候都能取得 Referer 的值**

#### Anti CSRF Token

CSRF 攻击的本质是，攻击者发送的请求中包含用户的信息，以及用户的身份认证 Cookie。如果浏览器允许发送本地 Cookie 的值，只能在验证用户的信息中进行设置，如：

一个删除操作：

```
http://host/path/delete?username=abc&item=123
```

可以把 username 改成哈希值

```
http://host/path/delete?username=md5(salt+abc)&item=123
```

使用 salt 或者随机数来验证请求参数。

目前最常用的解决 CSRF 的方法是 URL 中新增一个参数 Token，这个 Token 为用户与服务器共同所有，不能被第三者知晓。这个 Token 可以放在用户的 Session 或者浏览器 Cookie 中。并且这个 Token 会在发送一次请求后消耗，然后再重新生成一次 Token。

并且，Token 的发送最好放在表单中，作为一个隐藏的 Input 进行发送，而不是添加到链接中，否则可能会被发送给攻击者的 Referer 捕获。

**Token 只是用来对抗 CSRF，但如果网站还存在 XSS 或其他漏洞的情况下，Token 就可能被攻击者捕获，这种方案就不可行，这个过程被称为 XSRF**

## 点击劫持

点击劫持，是攻击者使用一个透明的，不可见的 iframe，或者图片，或者定义拖拽行为，覆盖在网页上，当用户点击或进行任何操作的时候，就正好将事件传输到 iframe 上。

### 点击劫持的防御

#### frame busting

可以写一段 JavaScript 代码，用来防止 iframe 的嵌套。但是这种方法很容易被绕过。frame busting 的大意为：

```
if (top != self)
if (top.location != self.location)
if (top.location != location)
if (parent.frame.length > 0)
if (window != top)
if (window.top != window,self)
......
```

#### X-Frame-Options

使用 X-Frame-Options 头，里面有三个可选值：`DENY、SAMEORIGIN、ALLOW_FROM origin`，可以选择是否拒绝当前页面下加载任何 frame 页面。

## HTML5 安全

### Cross-Origin Resource Sharing

可以在 HTTP Headers 中设置多种跨域安全访问权限，进行精细的控制：

```
Access-Control-Allow-Origin:指定了允许访问该资源的外域 URI
Access-Control-Max-Age:preflight请求的结果能够被缓存多久
Access-Control-Allow-Credentials:当浏览器的credentials设置为 true 时是否允许浏览器读取response的内容。
Access-Control-Allow-Methods:首部字段用于预检请求的响应。其指明了实际请求所允许使用的 HTTP 方法。
Access-Control-Allow-Headers:首部字段用于预检请求的响应。其指明了实际请求中允许携带的首部字段。
Origin:首部字段表明预检请求或实际请求的源站。
Access-Control-Request-Method:首部字段用于预检请求。其作用是，将实际请求所使用的 HTTP 方法告诉服务器
Access-Control-Request-Headers:首部字段用于预检请求。其作用是，将实际请求所携带的首部字段告诉服务器。
```

#### 预检请求（preflight request）

当请求的 Method 不是 GET、HEAD、POST 之一，请求的首部不在 [对 CORS 安全的首部字段集合](https://fetch.spec.whatwg.org/#cors-safelisted-request-header) 之一，或者 Content-Type 不在 `application/x-www-form-urlencoded、multipart/form-data、text/plain` 中，Client 首先会给 Server 发送一个 Option（类似于 GET、POST）请求，查看 Server 是否允许这些字段被发送到服务器。进行确认了以后，再次发送请求内容。首先发送 Option 请求的过程被称为**预检请求**

#### 附带身份凭证的请求

Fetch 与 CORS 的一个有趣的特性是，可以基于  HTTP cookies 和 HTTP 认证信息发送身份凭证。一般而言，对于跨域 XMLHttpRequest 或 Fetch 请求，浏览器不会发送身份凭证信息。如果要发送凭证信息，需要设置 XMLHttpRequest 的某个特殊标志位。

将 XMLHttpRequest 的 withCredentials 标志设置为 true，从而向服务器发送 Cookies。

如果服务器端的响应中未携带 Access-Control-Allow-Credentials: true ，浏览器将不会把响应内容返回给请求的发送者。

### postMessage

`window.postMessage` 对象不受同源策略的限制，允许每一个 window（包括当前窗口，弹出窗口，iframe 等）对象向其他窗口发送文本消息。

在使用 `postMessage` 时，有两个安全问题需要注意：

1. 在必要的时候，可以在接收窗口中验证 Domain，甚至验证 URL
2. 接收的消息写入 `innerHTML`，甚至写入 `script` 中的时候，可能会导致 DOM based XSS 产生。

## 参考

* [HTTP访问控制（CORS）](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)
