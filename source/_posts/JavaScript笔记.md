---
title: JavaScript笔记
date: 2016-11-29 22:38:50
categories: coding
tags:
  - JavaScript
---

JavaScript 笔记

## DOM 

### DOM 级别

**DOM0**是不存在的，DOM0 只是 DOM 历史中一个参照点而已，一般是指最初支持的 DHTML。

**DOM1**是 1998 年被推荐称为 W3C 的推荐标准，DOM1 被分为两个模块组成的：DOM Core 和 DOM HTML。

1. DOM Core：如何映射基于 XML 的文档结构，以便简化为文档中任意部分的访问和操作。DOM Core 有几个相关的方法，如：getElementById、getElementsByTagName、getAttribute、setAttribute，它们并不属于 JavaScript。
2. DOM HTML：在 DOM Core 的基础上加以扩展，增加了针对 HTML 的对象和方法。比如 onclick 属性，比如 src 属性等。


**DOM2**在 DOM1 的基础上扩展了鼠标和用户界面事件、范围、遍历迭代 DOM 文档的方法，也增加了对 CSS 的支持。DOM2 引入下列新模块：

1. DOM 视图(DOM View)：定义了跟踪不同文档(例如：应用了 CSS 之前和之后的文档)视图的接口
2. DOM 事件(DOM Event)：定义了事件和事件处理的接口
3. DOM 样式(DOM Style)：定义了基于 CSS 为元素应用样式的接口
4. DOM 遍历和范围(DOM Traversal and Range)：定义了遍历和操作文档树的接口

**DOM3**进一步扩展了 DOM，引入了以统一方式加载和保存文档的方法，也对 DOM Core 进行了扩展，开始支持 XML 1.0，设计 XPath 和 XML Base：

1. DOM 加载和保存(DOM Load and Save)
2. DOM 验证(DOM Validation)：DOM 验证

<!--more-->

### Node 类型

**cloneNode()**函数可以复制节点，后面的参数如果为 true 就执行深复制，深复制是复制节点和整个子节点树。

#### NodeList 对象

NodeList 对象不是某一个时间保存下来的快照，DOM 的变化能自动反应到 NodeList 对象上。


### Document 类型

**nodeName**是 "#document"

**nodeValue**是 null

**document.documentElement**指向的是`<html>`元素

**document.body**指向的是`<body>`元素

**document.compatMode**告诉开发人员浏览器使用的是什么渲染模式，如果是 `CSS1Compat`，就是标准模式，如果是 `BackCompat`，就是混杂模式

DOM3 中添加了遍历元素 DOM 结构的函数，分别是 `document.createNoeIterator` 和 `document.createTreeWalker`

**查找元素的方法：**

1. getElementById
2. getElementByTaagName：返回一个 HTMLCollection(与 NodeList 相似)类型的元素，如果传入的参数是 `"*"`，可以获得所有的元素
3. getElementsByName：获得给定 name 特性的元素，一般用在获得单选按钮
4. getElementsByClassName

### Element 类型

Node类型的元素，即node.nodeType == 1 为 true 的元素，**nodeName**的值是 tagName，**nodeValue**的值是 null。

**tagName**的返回值都是大写的，如"DIV"

**innerHTML、outerHTML或者 insertAdjacentHTML()** 使用的注意点：

1. 在删除带有事件处理或者引用了其他 JavaScript 对象子树时，会出现元素与处理程序之间的绑定关系依旧存在，页面占用的内存就会显著增加。所以为了提高性能，需要手工删除被替换的元素所有的事件处理程序和 JavaScript 对象属性
2. 在插入大量的 HTML 元素时，使用 innerHTML 属性与多次创建 DOM 节点然后再指定关系，**效率高得多**，因为使用 innerHTML 和 outerHTML 会创建一个解析器，这个解析器是浏览器级别的代码(通常是 C++)，所以执行速度快得多。

**childNodes 与 children** 之间的差异：`children` 属性返回的是元素中还是元素的节点， `childNodes`除了元素节点以外，还会返回文本、注释等节点。


与特性相关的函数是：`getAttribute()、setAttribute()、removeAttribute()`，自定义的特性应该加上 `data-` 前缀以便验证。

**attributes**属性经常用于遍历元素的属性，返回值是 NamedNodeMap，与 NodeList 类似。

**classList**属性可以比 `className`属性更加方便与操作元素的类，classList 包含很有用的方法，比如：`add、remove、item(按照集合中的索引返回类值)、contains、toggle("className")(如果类名存在，则删除类名；如果类名不存在，则添加类名)`
**使用 document.createElement() 可以创建一个新元素，其他的示例没有这个函数**

**查找元素的方法：**

1. getElementsByTagName
2. getElementsByClassName

#### 元素的大小

`offsetTop、offsetLeft`表示的是元素左(上)外框到包含元素的左(上)内框之间的距离

`offsetHeight、offsetWidth`表示的是元素在垂直(水平)方向上所占用的空间的大小，包括**边框+内边距+内容**

`clientWidth、clientHeight`表示的是元素内容在垂直(水平)方向上所占用的空间的大小，包括**内边距+内容**

`scrollWidth、scrollHeight`表示的是包含滚动隐藏的内容元素在垂直(水平)方向上所占用的空间的大小

`scrollLeft、scrollTop`表示的是被隐藏区域的左(上)部的像素数，通过设置这个值也可以实现滚动位置的重定位。

#### 样式(style)

**如果没有给元素设置 style 的特性， elem.style 对象会包含一些默认的值。style 对象只能返回 HTML 标签中设置的 style 的值(HTML style 特性或者 `<style>` 标签中设置的值)，或者 JavaScript 中为元素的 style 添加的值，使用 `<link>` 标签外链得到的其他层叠样式信息无法得到。**

`document.defaultView.getComputedStyle()` 方法，通过设置参数(第一个是要取得计算的元素，第二个是伪元素的字符串，例如 :after，没有就写 null)，可以过的当前元素所有计算样式。所有的计算样式都是只读的。

`document.styleSheet` 可以获得所有的 `<link>`标签和`<style>`标签中的内容。

### DocumentFragment

如果我们要更新文档中的结构，如果要逐条修改结构，会导致浏览器反复渲染。为了避免这个问题，可以先创建一个文档片段(document fragment)，**将结构预先修改到文档片段中，最后一并添加到文档内**

	let fragment = document.createDocumentFragment();

### 动态脚本，样式

使用 document.createElement("script")，添加脚本中的内容，最后把标签添加到当前页面中，才能开始下载外部文件。

动态样式也是同样道理。

### querySelector、querySelectorAll

**这两个函数的作用都是使用 CSS 选择符来查找并返回元素，**querySelector 返回匹配的第一个元素，querySelectorAll 返回的是一个 NodeList 实例，得到的是匹配到的所有元素。

### 表单(form)

#### HTMLFormElement

常用的属性有：

* **acceptCharset**：服务器可以接受的字符集
* **action**：接受请求的 URL
* **elements**：控件集合
* **length**：控件数量
* **method**：http 请求类型，是 post 还是 get
* reset()：所有表单域重置为默认值(可以触发 reset 事件)
* submit()：提交表单(这个函数不会触发 submit 事件，所以在调用这个函数之前要先验证表单数据)

#### 提交表单

使用 input[type="submit"] 或者 button[type="submit"] 都可以提交表单。**如果一个提交表单长时间没有反应，用户可能会多次点击提交按钮，结果可能很麻烦，所以在第一次提交表单以后，可以禁用提交按钮(`btn.disable = true`)，或者利用 onsubmit 函数取消后续的表单提交动作。**

#### 表单事件

表单常用的事件有：

* `onclick`：点击的时候
* `onmouseout`：当鼠标移出的时候
* `onmouseover`：当鼠标移入的时候
* `onfocus`：控件获得焦点
* `onblur`：控件失去焦点

#### 表单约束

可以在每个文本框填写完成以后，对表单填写的内容进行验证，除此以外，HTML5 还对表单的自动验证提供了一些支持，比如有：

* **required**：必填
* **type="url"、"email"**：多种输入类型
* **type="number"、"range"、"datetime"、"datetime-local"、"date"、"month"、"week"、"time"等 **：多种输入类型，不过浏览器对这几个类型的支持情况不好
* **pattern**：在 HTML 中添加正则表达式进行验证

#### 表单序列化

**表单序列化**解决的是如何向服务器传输表单获得的数据，一般有以下要求：

* 对表单字段的名称和值进行 URL 编码，使用(`&`)符号进行分割
* 不发送禁用的表单字段
* 只发送勾选的单选框和复选框按钮
* 不发送 type 为 reset 或 button 的按钮
* 多选框中每一个选中的值单独一个条目
* `<select>`元素的值，就是选中的`<option>`的 value 值，如果没有 value，就是 `<option>`的文本值

#### 富文本编辑

**富文本编辑**允许在网页中编辑富文本内容。

### 事件

一般的事件处理使用的事件冒泡(事件从发生元素向 document 元素冒泡)，在特殊情况下使用事件捕获(document 先捕获事件，最后是发生元素捕获事件)

**事件处理函数有一个局部变量 event，通过 event 对象，可以直接访问事件对象，不用自己定义它，也不用从函数的参数列表中读取。在事件处理函数的内部， this 值等于事件的目标元素。**

使用 `event.preventDefault()` 函数可以阻止事件的默认行为，如果事件的处理函数返回值是 false，也可以阻止事件的默认行为。

**JavaScript 支持自定义事件，事件是观察者的设计模式。**

**有的事件可能会在事件期间被重复触发，所以要注意，类似的事件有：scroll、mousemove、keydown、keypress**

有的键盘事件可能触发鼠标事件，也可能影响鼠标事件的结果。

DOM 结构的改变也会出发相关的事件，详细参考变动事件。

**beforeunload**：事件可以让程序员在页面卸载之前做一些提示用户的动作，但是不能通过这个事件阻止页面的关闭行为。

**如果页面中绑定了过多的事件处理程序**，会直接影响到页面的运行效率，这个时候，可以利用事件的冒泡在高层元素上添加事件处理程序，以减少事件处理程序的数量。

**当使用 removeChild() 方法，或者使用 innerHTML 等方法替换掉带有事件处理程序的元素时，**原来添加到元素中的事件处理程序极有可能无法被当作垃圾回收，所以最好提前移除事件处理程序，如 `btn.onclick = null;`

`HTML5` 支持了一些和拖放相关的事件，比如`dragstart、drag、dragend`等

**模拟事件发生**：可以使用 `createEvent`方法创建一个事件，使用`dsipatchEvent`方法来触发这个事件，来模拟事件的发生。

#### DOM0级事件处理程序

	btn.onclick = function(){
		alter("press!");
	}

好处是具有跨浏览器的优势。**缺点是使用这种方法，只能给 onclick 赋值一个处理函数，如果再次调用，会覆盖前一个处理函数。**

#### DOM2级事件处理程序

`addEventListener` 和 `removeEventListener` 函数，在 **IE** 下面是 `attachEvent()` 和 `detachEvent()`

## BOM

BOM 提供的相关功能如下：

1. 弹出浏览器窗口的功能：window.open(**关于弹出窗口有安全限制**)
2. 移动，缩放和关闭浏览器窗口的功能。**但是有的方法可能被浏览器禁用**
3. `window`对象：window 既是 JavaScript 访问浏览器窗口的一个接口，又是 ECMAScript 规定的 Global 对象
3. `navigator`对象：提供浏览器的详细信息
4. `location`对象：提供浏览器所加载的文档的详细信息，比如 URI、端口相关信息。还提供导航功能
5. `screen`对象：提供用户显示器分辨率详细信息
6. `history`对象：保存用户的上网记录，进行跳转等
6. 对`cookie`的支持
7. 对`XMLHttpRequest`和`ActiveXObject`等自定义对象的支持

### 窗口位置和大小

1. **窗口位置**：窗口相对于屏幕左边和上边的位置。(screenLeft、screenTop、screenX、screenY)、
2. **窗口大小**：(outerWidth、outerHeight、innerWidth、innerHeight)

### 系统对话框

1. 确定按键：alter()
2. 确定、取消按键：confirm()
3. 提示用户输入文本：prompt()

### 离线应用与客户端存储

#### 离线检测

navigator.onLine 可以判断是否离线，并且还有 online 和 offline 两个事件

#### 应用缓存

appcache 是用来开发离线 WEB 应用而设计的，本质上是从浏览器缓存中分出来一块缓存区，如果要使用这个缓存区，可以使用一个描述文件(manifest file)，写出要下载和缓存的资源。

#### Cookie

用来在客户端存储会话信息和数据，一般每个域名的 cookie 属性有限制，浏览器的 cookie 总大小也有所限制。

**cookie 中任何数据都可以被他人访问，所以不要在 cookie 中存储隐私数据**

#### Web Storage(Web 存储)

**在 chrome 调试中，对应的是本地存储这一项。**

存储大量可跨越会话存在的数据的机制

#### indexDB

**在浏览器中保存结构化数据的一种数据库**


## 元素标签

### script标签

在使用`<script>`标签添加 JavaScript 代码的时候，不要在代码的任何地方出现 "`</script>`"字符串，否则当浏览器遇到 "`</script>`"字符串的时候，会被会被解析成 `</script>`标签。不过可以使用转义字符 "\" 来解决这个问题。例如：

	<script type="text/javascript">
		function say(){
			alert("<\/script>");
			}
	</script>

#### script 标签属性

1. defer：表示脚本被延迟到整个文档完全被解析和显示之后再执行，只对外部脚本有效。**相当于告诉浏览器，立即下载，但是延迟执行**
2. async：表示痢疾下载脚本，但是不妨碍页面中的其他操作。**告诉浏览器，不让页面等待脚本的下载和执行，而是异步加载页面的其他内容**

## JavaScript 语法

### 操作符

#### 逻辑非

逻辑非操作符遵循下列规则：

1. 非空对象，则返回 false
2. 空字符串，则返回 true，否则为 false
3. 如果是 0 或者 NaN，则返回 true，否则返回 false
4. 如果是 null 或者 undefined，则返回 ture，否则返回 false

**使用场景**：

	var a;
	var b = !!a;

使用 `!!`在变量的前面，如果 a 是 undefined，`!a`是true，`!!a`是 false，这样方便后续判断。

#### 逻辑与

逻辑与`&&`是短路操作，如果第一个操作数能够决定结果，就不会对第二个操作数进行求值。

**非常重要！：逻辑与操作的返回值不一定是布尔类型！**有一个操作数不是布尔值，则返回值也不一定是布尔值


遵循的规则：

1. 如果第一个操作数是对象，则返回第二个操作数
2. 如果第二个操作数是对象，则只有在第一个操作数的求值结果是 true 的情况下，才返回第二个对象，否则返回第一个对象(**第一个操作数是 undefined、null、NaN、0、""**)
3. 如果两个操作数都是对象，则返回第二个操作数

#### 逻辑或

逻辑与`||`是短路操作，如果第一个操作数能够决定结果，就不会对第二个操作数进行求值。

**非常重要！：逻辑或操作的返回值不一定是布尔类型！**有一个操作数不是布尔值，则返回值也不一定是布尔值

遵循的规则：

1. 如果第一个操作数是对象，则返回第一个操作数
2. 如果第一个操作数为 false(**第一个操作数是 undefined、null、NaN、0、""**)，则返回第二个操作数
3. 如果两个操作数都是对象，则返回第一个操作数
4. 如果两个操作数都是 null、undefined、NaN，则返回 null、undefined、NaN

#### 相等操作符(`==`和`!=`)

**相等与不相等操作符，要先进行数据类型转换，再进行比较**

转换规则如下：

1. Boolean 类型的都转换为 Number 类型
2. 如果一个操作数是 String，另一个是 Number，则使用 Number() 将字符串转换为数值
3. 如果一个是 object，另一个不是，则调用对象的 valueOf() 方法，用得到的基本数据类型进行比较
4. null 和 undefined 是相等的，因为 undefined 派生自 null，他们也可以等于自身
5. null 和 undefined 不能转换为其他值
6. 如果一个操作数是 NaN，则它永远不和任何操作数相等，即使 `NaN == NaN` 的返回值也是false
7. 如果两个操作数是对象，则比较他们是不是指向同一个对象

#### 全等操作符(`===`和`!==`)

**全等与不全等操作符，不进行数据类型转换，直接进行比较**，所以不仅要值相等，数据的类型也要相等

null == undefined 的返回值是 true，但是**null === undefined 的值是 false，因为它们属于不同的数据类型** 

#### 乘法和除法操作符

如果参与计算的操作数不是数值，将先使用 Number() 转换类型函数将其转换为数值类型进行计算。

#### 加法操作符

如果操作数不是数值，而是字符串、对象、数值、布尔值、null、undefined，则会调用 String() 方法将其先转换为字符串，然后进行拼接。

#### 减法操作符

如果一个操作数是对象，则调用 valueOf() 方法来获取值，如果没有 valueOf 方法，则调用 String() 方法获取值。

如果有一个操作数是字符串、布尔值、null、undefined，则在后台调用 Number() 方法进行类型强制转换，如果转换结果是 NaN，则计算结果是 NaN。

#### 大小比较符(`<`，`>`)

1. 如果都是数值，则进行数值比较
2. 如果都是字符串，则分别比较字符串对应的字符编码值(**大写字母的字符编码全部小于小写字母的字符编码**)
3. 如果有一个操作数是数值，则将另一个操作数转换为数值进行比较
4. 如果有一个操作数是对象，则调用这个对象的 valueOf() 方法，如果没有，则调用 toString() 方法
5. 如果有一个操作数是布尔值，则转换为数值进行比较

### 变量

如果在创建变量的时候**省略 var 操作符，则会创建一个全局变量**，不过这样做会在局部作用域中定义全局变量，会变得很难维护

### 数据类型

ECMAScript 中有 **5 种简单的数据类型(基本数据类型)：Undefined、Null、Boolean、Number、String，和一种复杂数据模型：Object。**

使用 typeof 操作符来检测特定变量的数据类型，而不是使用 "==" 等操作符来检测数据类型，例如**undefined 派生自 null 值，null == undefined 返回值是 true**。

**基本数据类型在内存中占据固定大小的空间，所以被保存在栈内存中。引用类型的值是对象，所以被保存在堆内存中**

#### String 类型

**转义字符**是 String 类型包含的一些特殊的字符变量，只有在字符串里才有转义字符。

数值，布尔值，对象和字符串这四个数据类型有一个 toString() 方法，但是**null 和 undefined 值没有这个方法。**

#### Object 类型

**constractor**保存的是用于创建当前对象的函数，在一定程度上可以用来判断数据类型，但是不推荐。

**如果想要检测对象的类型，使用 instanceof 操作符**

##### 安全的类型检测

	var isArray = value instanceof Array;

一般对 Object 类型使用 instanceof 操作符进行类型检测，**但是如果 value 和 Array 不在同一个全局作用域中的时候，比如 value 是在另一个 frame 中定义的数组，此时返回值就是 false**，所以我们推荐使用 Object.prototype.toString() 方法来检测**原生的**对象。尤其是 JSON 对象，很多人使用第三方的 JSON 库，所以很难确定页面中的 JSON 是否是原生的。

	function isArray(value){
		return Object.prototype.toString.call(value) == "[object Array]";
		}

##### Array

数组的 length 不是只读的，通过设置 length 的值，可以对数组的末尾删除项目或者添加新的项目，**如果数组的某一些项没有设置值，默认为 undefined。**

1. concat 方法可以创建一个当前数组的副本，并且将参数添加到这个副本的末尾，得到一个新的数组
2. 迭代方法：数组有 5 中迭代方法，**对数组的每一项都运行给定的函数**，它们是 every()、filter()、forEach()、map()、some()
3. 并归方法：数组有 2 中并归方法，**都会迭代数组的所有项，然后构建一个最终的返回值。**

##### RegExp

在正则表达式之中，需要对某些特殊的字符进行转义，如果要对正则表达式的内容转换为字符串的正则表达式，则需要进行双重转义

#### Function

函数名实际上是指向函数对象的一个指针，**注意JavaScript 的函数没有重载**

函数的定义有**两种方式，一种是函数声明，一种是函数表达式**

函数声明：

	function sum(arg1, arg2){
		return arg1 + arg2;
	}

函数表达式：

	var sum = function(arg1, arg2) {
		return arg1 + arg2;
	}

**函数声明和函数表达式的区别：**

函数声明的好处是，如果在函数声明之前调用了函数，代码会成功执行，因为 JavaScript 的解析器有一个函数声明提升的过程。但是函数表达式如果在使用之前没有定义，则会出现问题。

**因为函数也是一个对象，所以可以将函数作为参数进行传递，返回值也可以 return 一个函数。**

	function createSum(arg1, arg2){
		return function(arg1, arg2){
			return arg1 + arg2;
		}
	}

##### 函数的内部属性

`arguments`：保存函数的参数，arguments 还有一个 callee 的属性，该属性是一个指针，指向拥有这个 arguments 对象的函数

**this**：引用的是函数执行的环境对象，即调用了这个函数的对象。

**length**：函数想要接收的参数个数

**apply() 和 call()**：在固定的作用域中调用函数，实际上等于设置函数体内 this 的值。这两个函数的区别只不过是接收参数的方式不同。

##### 函数的闭包

闭包是指有权访问另一个函数作用域中的变量的函数，通常是在一个函数内部创建另一个函数。

**因为作用域链的原因。闭包只能获得包含函数中任何变量的最后一个值，闭包所保存的是整个变量对象，而不是某个特殊变量。**

	function test(){
		var result = new Array();
		for(var i=0;i<10;i++){
			result[i] = function(){
				return i;
			};
		}
		return result;
	}

这个时候所有输出的值都是 10，此时有多种方法来解决这个问题：

可以使用一个**立即执行的匿名函数**，这样可以获得每一个 i 的值。

	function test(){
		var result = new Array();
		for(var i=0;i<10;i++){
			result[i] = function(num){
				return function(){
					return num;
				}
			}(i);
		}
		return result;
	}

也可以使用 ES6 的 let 操作符创建变量 i ，使用 let 操作符创建的变量 i 具有块级作用域，它的值不会发生变化

	function test(){
		var result = new Array();
		for(let i=0;i<10;i++){
			result[i] = function(){
				return i;
			};
		}
		return result;
	}

##### 模仿块级作用域

`(function(){//这里是块级作用域})();`可以用来模仿块级作用域。

可以利用块级作用域定义静态私有变量。

#### 数值转换

**Number()**函数将任何数据类型转化为数值

1. 如果是 Boolean 值，true 会被转化成 1，false 会被转化称为0
2. 如果是数字值，只是简单的传入和返回
3. 如果是 null 值，返回 0
4. 如果是 undefined 值，返回 NaN
5. 如果是字符串，则分为多种情况：要是只包含数字，则会被转换成十进制的数值，如"123"；要是字符串是浮点的格式，则会转换为对应的浮点数，如"1.1"；如果字符串包含有效的十六进制的格式，则转换为相同大小的十进制数。如"0xf"；如果字符串为空，则转换为0，如果不为上述所有的格式，则转换成 NaN

isNaN()函数，**先将传进去的参数转换为数值，然后判断转换的结果是不是 NaN**，所有不能被转换为数值的参数使用 isNaN() 函数的返回值都是 true，而并非 NaN 才返回 true。

**parseInt() 和 parseFloat()**函数将字符串转化为数值

**parseInt()妙用：**在使用 `parseInt()`处理值的时候，如果单位是无法被转换的字符串，则会被自动去掉。例如：`aprseInt("16px")`的返回值是 **16**


**String()**方法将任何数据类型的值转换为字符串：

1. 如果有 toString() 方法，则调用该方法并返回结果
2. 如果值是 null，则返回 "null"
3. 如果值是 undefined，则返回 "undefined"

**Boolean()**方法将任何数据类型转换为布尔类型

1. String：如果是空字符串，则为 false，否则为 true
2. Number：如果是 0 或者 NaN，则为 false，否则为 true(包括无穷大)
3. Object：如果是 null，则为 false，否则为 true
4. Undefined：被转换为 false

### 语句

#### for-in 语句

`for (property in expression) statement`

可以用来枚举对象的属性。

如果要迭代的对象是 null，则不执行循环体，如果是 undefined，则会报错。**推荐在执行 for-in 之前先确认对象是不是 null 或者 undefiend**

#### break & continue

break 用来跳出循环，不再执行循环。continue 跳出本次循环，但是会继续执行接下来的循环。

**如果在多层循环中 break 和 continue 只能跳出一层循环，如果想跳出多层循环，需要使用 label 和 continue 或者 break 合作**

#### switch

switch 语句中可以使用任何数据类型进行判断，**switch 语句在比较的时候使用的是全等操作符，所以需要考虑数据类型。**

### 函数

**ECMAScript 没有重载，如果定义了两个相同名字的函数，则后定义的函数生效。**

#### arguments

函数体内可以通过 arguments 获得所有参数

arguments 和函数参数的内存空间不同，但是他们的值会同步，对 arguments 的值进行修改，会导致实参的值也被修改。

#### 执行环境及作用域

全局的执行环境是 window 对象。每一个函数都会有自己的执行环境

**在 JavaScript 中，花括号所包裹的范围没有自己的作用域(执行环境)**

#### 惰性载入函数

**一般用在多浏览器兼容处理上，会对浏览器所支持的能力进行检查，然后在决定如何对函数的行为进行赋值。《JavaScript 高级程序设计》P600**

#### 函数绑定

ECMAScript 5 为函数定义了一个 bind() 方法，可以在函数上调用，用来指定某个对象作为函数运行时的 this 对象，类似于 call() 和 apply() 。不过 bind() 函数的返回值有一些不同，它返回设置好 this 的函数变量，当需要的时候，再设置参数。

#### 函数柯里化

**用于创建已经设置好一个或多个参数的函数，有指定作用域**

**函数绑定和函数柯里化都会带来额外的开销，不应该滥用。**

#### 内存管理

JavaScript 具有自动的内存管理机制，但是为了确保占用最少的内存，就需要对数据进行处理。**当数据不再使用的时候，最好将其设置为 null，这样内存处理机制就会将其释放**


### ECMAscript 内置对象

#### URI 编码方法

浏览器对于 URI 的理解中，有一些字符无法解释，所以，需要使用 encodeURI() 方法可以对 URI 进行编码，用特殊的 UTF-8 编码替换所有无效的字符串，让浏览器能够理解。与其对应的方法是 decodeURI()。**对于整个 URI ，使用 encodeURI() 进行编码，它不会对属于 URI 的特殊字符进行编码，比如冒号，正斜线，问号，井字号，这样会导致有一些需要进行编码的特殊字符被漏掉，所以使用 encodeURI() 的时候，最好先确定 URI 中其他的部分没有属于 URI 的特殊字符**

encodeURI() 主要用于整个 URI，而 encodeURIComponent() 主要用于 URI 中的某一段进行编码，与之对应的是 decodeURIComponent()。**因为 encodeURIComponent() 会对所有的非字母数字字符都进行编码，如果对整个 URI 使用这个方法，会将属于 URI 的特殊字符进行编码，导致浏览器解析发生错误。**

###  面向对象的程序设计

#### 数据属性

1. [[configurable]]：能否通过 delete 来删除属性从而重新定义，默认为 true
2. [[enumerable]]：能否通过 for-in 循环返回属性，默认为 true
3. [[writable]]：能否修改属性值，默认为 true
4. [[value]]：包含这个属性的数据值，读取值的时候，从这个位置读；写入属性的时候，把值写到这个位置。默认为 undefined

#### 访问器属性

访问器属性包含一对 getter 和 setter，当读取该属性时，会自动调用 getter 函数，这个函数负责返回有效的值。在写入数据属性时，会调用 setter 函数并传入新的值，这个函数决定如何处理数据：

1. [[configurable]]：能否通过 delete 来删除属性从而重新定义，默认为 true
2. [[enumerable]]：能否通过 for-in 循环返回属性，默认为 true
3. [[get]]：读取属性时调用的函数，默认是 undefined
4. [[set]]：写入属性时调用的函数，默认是 undefined

#### 防篡改对象

**一旦把对象设定为防篡改，就无法撤销了。**

##### 不可扩展对象

	Object.preventExtension(arg);

设置后，无法给 arg 对象添加属性和方法

##### 密封的对象

	Object.seal(arg);

设置后，arg 对象无法扩展，也无法删除属性和方法。

##### 冻结的对象

	Object.freeze(arg);

设置后，arg 独享无法扩展，又是密封的，并且对象属性不能被修改。

#### 原型链

**在构造函数中， prototype指向的是构造函数的原型，当通过构造函数生成一个对象以后，对象拥有的属性是 `__proto__` ，prototype 在原型链中起到的是一个辅助作用，用来将原型赋值给 `__proto__`，而`__proto__`代表了原型链的本质。**

**使用原型链的时候，如果原型中的属性使用了引用类型，而不是基本数据类型，则不同的实例会共享属性，一个实例修改了引用类型的属性，可能会影响到其他实例。**

为了向超类的构造函数中传递数据，要使用 call() 或者 apply() 方法来传递参数，设置运行环境。

##### 作用域安全的构造函数

如果构造函数如下定义：

	function Person(name, age){
		this.name = name;
		this age = age;
	}

**当没有使用 new 操作符调用该构造函数的情况下，没有为这个对象创建新的对象，此时 this 对象会绑定到 window 对象上，导致错误。**

所以要判断 this 对象的类型：

	function Person(name, age){
		if(this instanceof Person){
			this.name = name;
			this.age = age;
		} else{
			return new Person(name, age);
		}
	}

**但是这种方法进行继承的时候，如果没有使用原型链，可能会破坏继承，见《JavaScript 高级程序设计》P599**

### 数据交互

#### 跨文档数据交互

**跨文档信息传送(cross-document messaging)**：简称 `XDM`，指的是来自不同域的页面相互之间传递消息。核心方法是 `postMessage`，会触发 `message` 事件。一般发送的对象是当前页面中的 `<iframe>`，或者当前页面弹出的窗口。

#### AJAX

使用 `XMLHttpRequest` 对象，能够以异步的方式从服务器获得更多的信息，然后通过 DOM 将数据插入到页面中。

**只能向同一个域中使用相同端口和协议的 URL 发送请求，否则会引发安全错误(无法跨域)。同时，请求的对方必须运行在一个服务器上，因为 XHR 只支持 http、https、data 等协议，不支持本地以文件形式打开**

##### `GET`方法

有关 GET 请求的其他一些注释：
* GET 请求可被缓存
* GET 请求保留在浏览器历史记录中
* GET 请求可被收藏为书签
* GET 请求不应在处理敏感数据时使用
* GET 请求有长度限制
* GET 请求只应当用于取回数据

##### `POST`方法

有关 POST 请求的其他一些注释：
* POST 请求不会被缓存
* POST 请求不会保留在浏览器历史记录中
* POST 不能被收藏为书签
* POST 请求对数据长度没有要求

##### POST&GET 比较

[w3school 上有 POST 和 GET 的对比](http://www.w3school.com.cn/tags/html_ref_httpmethods.asp)

与 POST 相比，GET 更简单也更快，并且在大部分情况下都能用。

然而，在以下情况中，请使用 POST 请求：

* 无法使用缓存文件（更新服务器上的文件或数据库）
* 向服务器发送大量数据（POST 没有数据量限制）
* 发送包含未知字符的用户输入时，POST 比 GET 更稳定也更可靠

虽然 `XMLHttpRequest` 也支持使用同步的方法进行数据的请求，但是使用同步的方法进行数据的请求**已经要被标准废除了**，因为会影响用户的体验。

##### FormData

`FormData` 对象用来进行表单的序列化

##### 超时设定

XHR 对象有一个 timeout 属性，表示请求在等待相应多少毫秒后终止。

##### XHR 事件

目前有 6 个事件绑定在 XHR 上用来处理请求发生的相关状况：`loadstart、progress、error、abort、load、loadned`

#### 图像 ping

**图像 ping 可以跨域**，原理是使用 `<img>`标签，在 src 属性中添加请求的链接。因为网页可以从任何网页中加载图像，所以可以跨域。

**缺点是**：

1. 只能发送 GET 请求
2. 无法访问服务器的响应文本，只能用于浏览器与服务器之间的单向通讯

####  JSONP

JSONP(JSON with padding)，填充式 JSON 或者 参数式 JSON，是 JSON 的新用法。**JSONP 可以跨域，支持浏览器与服务器的双向通讯**

方法是：创建处理响应到来时的回调函数，然后通过动态`<script>`标签，为 src 属性执行跨域的 URL。

#### Comet

**Comet 是一种服务器向页面推送数据的技术，通常用于数据经常更新的场景，比如股票报价，体育文字直播等。**实现方法有长轮询和 HTTP 流。

**SSE(Server-Sent Event)**：服务器发送事件，是围绕只读 Comet 交互推出的 API。使用 XHR 和 SSE 配合能实现双向通信。

#### Web Sockets

**在单独，持久连接上提供全双工，双向通信。**当创建了 Web Socket 之后，会发出一个 HTTP 请求，当获得的服务器的响应了以后，会从 HTTP 协议变成 Web Socket 协议(ws:// 或者加密的 wss://)

Web Socket 不支持 DOM 2 级事件监听，只能使用 DOM0 语法定义事件处理程序。

**想要使用 Web Socket，需要有 Web Socket 服务器**

### 异常处理

**一般在页面中的 JavaScript 发生错误会导致下面的脚本无法继续执行，影响用户的体验，所以要在合适的地方加上异常处理程序。**

	function test(){
		try{
			return 0;
		} catch(error) {
			return 1;
		} finally{
			return 2;
		}
	}

如果执行这条函数，我们可以看到，无论 try 和 catch 中包含什么样的语句，甚至是 return 语句，都不会阻止 finally 语句的执行，最后函数的**返回结果是 2**。**只要代码中包含 finally 子句，则无论 try 还是 catch 中的语句块的 return 都会被忽略，这一点需要注意。**

可以使用 `throw` 操作符主动抛出一个错误，然后终止代码，`throw`的值没有类型的限制。

	throw 1;
	throw "11";
	throw true;
	throw {name:"JavaScript"};

**一般需要关注的错误有：**

1. 类型转换错误
2. 数据类型错误
3. 通信错误

### JavaScript 与 XML

**将字符串解析为 DOM**：

	var parser = new DOMParser();
	var xmldom = parser.parseFromString(str);

**将 DOM 文档序列化为字符串：**

	var serializer = new XMLSerializer();
	var xml = serializer.serializerToString(xmldom);


**Xpath**是用来在 DOM 文档中查找节点的一种手段。

####  E4X

`E4X` 指的是 ECMAScript 对于 XML 原生的支持，在 ECMAScript 中创建 XML 对象用来处理 XML，但是很多浏览器对这个扩展标准的支持并不好。

### JSON

`JSON`是 JavaScript 中读写结构化数组的更好的方法，因为可以把 JSON 直接传给 eval()。而且不必创建 DOM 对象。

JSON 支持以下三种类型的值

* **简单值**：可以表示字符串、数值、布尔值、null，**但是没有 undefined**
* **对象**：表示一组无序的键值对
* **数组**：表示一组有序的列表

**注意点**：JSON 中的对象需要给所有的属性值加引号，并且同一个对象中不可以出现两个同名的属性。

#### JSON 的序列化和解析

`JSON.stringify()`：将 JavaScript 对象序列化为 JSON 字符串，在进行序列化的时候，可以控制参数，添加一个过滤器，也可以选择是否在 JSON 中保留缩进。还可以定义 toJSON() 函数，来自定义自身返回的 JSON 数据格式。

`JSON.parse()`：把 JSON 字符串解析为 JavaScript 对象，也可以在参数上提供一个函数，来确定如何进行 JavaScript 对象的构造

### 性能优化

#### 避免全局查找

如果作用域嵌套很深或者作用域的数量很多，访问当前作用域以外的变量的时间就会增加，导致**访问全局变量比访问局部变量慢，因为需要遍历作用域，特别是 document 对象。**所以，如果函数中使用 document 对象的次数过多，可以创建一个指向 document 对象的局部对象，通过限制全局查找改变函数的性能：`var doc = document;`

#### 避免 with 语句

`with` 语句会创建自己的作用域，导致执行代码的作用域链增加，在 `with`语句内的代码执行要慢。

#### 避免不必要的属性查找

使用变量和数组比访问对象上的属性更有效率，因为对象上的属性查找，都是对原型链的属性进行一次搜索。

多次用到对象属性，应该将其存储到局部变量中。

#### 优化循环

* **减值迭代**：大多数循环从 0 开始，很多情况下，从最大值开始，在循环中不断减值的迭代器更高效
* **简化终止条件**：由于每次循环都要计算终止条件，所以要保证终止条件尽可能快
* **简化循环体**：如果有多余的计算，从循环里删除
* **使用后测试循环**：do-while 这种后测试循环，可以避免终止条件的计算，因此运行更快
* **展开循环**：如果循环次数是确定的，消除循环并多次使用函数往往更快
* **避免双重解释**：避免参数为 JavaScript 语句的字符串，因为 JavaScript 解析 JavaScript 会存在双重解释惩罚

#### 最小化语句数

**JavaScript 中语句的数量会影响到执行的速度**。

	var a = 1;
	var b = 2;
	var c = 3;

的速度比

	var a = 1,b = 2, c = 3;

要慢得多。

	var name = value[i];
	i++;

最好也修改为：

	var name = value[i++];

#### 优化 DOM 操作

对于小的 DOM 更改，使用 `createElement()` 和 `appnedChild()` 与使用 `innerHTML` 的效率都差不多。

**如果要多次进行 DOM 的操作，最好是使用一个 documentFragment 先保存所有的更新操作，然后一次添加到 DOM 中，或者构造成一个字符串，然后一次使用 innerHTML 添加到 DOM 中。**

#### 使用事件代理

页面上有过多的事件处理程序，会很大程度上影响页面的响应速度，所以**使用事件冒泡的原理，在祖先节点上使用一个统一的事件处理函数来处理子节点相同的事件**，用以提升效率。

#### 注意类 Array 的 DOM 数据结构

**类 Array 的 DOM 数据结构，如 `HTMLCollection、NodeList、NamedNodeMap`等，无论是访问它的属性还是方法，都是在文档上进行一个查询，开销很昂贵，所以要最小化访问这些数据结构的次数**，如在循环的终止条件中：

	var imgs = document.getElementsByTagName("img"), i,len;
	len = imgs.length;
	for(i=0;i<len;i++){...}

使用本地变量保存 length，而不是每次都去访问 imgs 的属性。

### Web Worker

**因为页面加载的过程是一个单线程的过程，所以如果 JavaScript 的计算过于复杂，会导致用户的界面被冻结，所以 Web Worker 让 JavaScript 在后台运行来解决这个问题。**

一般用 Web Worker 运行一个 .js 程序，并且**它所执行的 JavaScript 代码完全在另一个作用域中(worker对象)**，所以最好把长时间、大型、独立的计算放在一个 worker 中执行。

建议使用 Web Worker 的时候，始终使用 onerror 事件处理程序，否则 Worker 会在发生错误时静默失败。

###  JavaScript里的循环方法forEach，for-in，for-of

JavaScript诞生已经有20多年了，我们一直使用的用来循环一个数组的方法是这样的：

```
for (var index = 0; index < myArray.length; index++) {
console.log(myArray[index]);
}
```
自从JavaScript5起，我们开始可以使用内置的 forEach 方法：

```
myArray.forEach(function (value) {
console.log(value);
});
```

写法简单了许多，但也有短处：你不能中断循环(使用语句或使用语句。

for-in循环实际是为循环”enumerable“对象而设计的：

for – in 是用来循环带有字符串key的对象的方法。

```
var obj = {a:1, b:2, c:3};
for (var prop in obj) {
console.log("obj." + prop + " = " + obj[prop]);
}
// 输出:
// "obj.a = 1"
// "obj.b = 2"
// "obj.c = 3"
```

JavaScript6里引入了一种新的循环方法，它就是for-of循环，它既比传统的for循环简洁，同时弥补了forEach和for-in循环的短板。

for-of 可以用来循环数组：

```
let iterable = [10, 20, 30];
for (let value of iterable) {
console.log(value);
}
// 10
// 20
// 30
```
循环一个字符串：

```
let iterable = "boo";
for (let value of iterable) {
console.log(value);
}
// "b"
// "o"
// "o"
```

循环一个 DOM collection

```
// Note: This will only work in platforms that have
// implemented NodeList.prototype[Symbol.iterator]
let articleParagraphs = document.querySelectorAll("article > p");
for (let paragraph of articleParagraphs) {
paragraph.classList.add("read");
}
```

循环一个生成器(generators)

```
function* fibonacci() { // a generator function
    let [prev, curr] = [0, 1];
    while (true) {
        [prev, curr] = [curr, prev + curr];
        yield curr;
    }
}
for (let n of fibonacci()) {
console.log(n);
    // truncate the sequence at 1000
    if (n >= 1000) {
        break;
    }
}
```

for–of循环并不能直接使用在普通的对象上，但如果我们按对象所拥有的属性进行循环，可使用内置的Object.keys()方法：

```
for (var key of Object.keys(someObject)) {
console.log(key + ": " + someObject[key]);
}
```



















