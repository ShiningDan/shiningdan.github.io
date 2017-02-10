---
title: ES6入门
date: 2017-02-09 16:38:32
categories: coding
tags:
  - ES6
---

本文是我在学习 ECMAScript 6 时的记录，用于个人查询总结。在学习的过程中参考过很多别人的文章，如有需要，可以根据链接详细学习：

* [ECMAScript 6 入门](http://es6.ruanyifeng.com)

## ECMAScript 6简介

### ECMAScript和JavaScript的关系

要讲清楚这个问题，需要回顾历史。1996年11月，JavaScript的创造者Netscape公司，决定将JavaScript提交给国际标准化组织ECMA，希望这种语言能够成为国际标准。次年，ECMA发布262号标准文件（ECMA-262）的第一版，规定了浏览器脚本语言的标准，并将这种语言称为ECMAScript，这个版本就是1.0版。

该标准从一开始就是针对JavaScript语言制定的，但是之所以不叫JavaScript，有两个原因。一是商标，Java是Sun公司的商标，根据授权协议，只有Netscape公司可以合法地使用JavaScript这个名字，且JavaScript本身也已经被Netscape公司注册为商标。二是想体现这门语言的制定者是ECMA，不是Netscape，这样有利于保证这门语言的开放性和中立性。

因此，ECMAScript和JavaScript的关系是，前者是后者的规格，后者是前者的一种实现（另外的ECMAScript方言还有Jscript和ActionScript）。日常场合，这两个词是可以互换的。

### 部署进度

各大浏览器的最新版本，对ES6的支持可以查看 kangax.github.io/es5-compat-table/es6/。随着时间的推移，支持度已经越来越高了，ES6的大部分特性都实现了

<!--more-->

## ES6 和 ES5 的对比

### let 和 const

#### let 命令

`let` 命令相比于 `var` 命令，有一下区别：

1. 不存在变量提升
2. 暂时性死区
3. 不允许重复声明


##### 块级作用域

ES5只有全局作用域和函数作用域，没有块级作用域。`let`实际上为JavaScript新增了块级作用域。

**块级作用域与函数声明**：

ES5 规定，函数只能在顶层作用域和函数作用域之中声明，不能在块级作用域声明。

ES6 引入了块级作用域，明确允许在块级作用域之中声明函数。

ES6 改变了块级作用域内声明的函数的处理规则，显然会对老代码产生很大影响。为了减轻因此产生的不兼容问题，ES6在附录B里面规定，浏览器的实现可以不遵守上面的规定，有自己的行为方式。

* 允许在块级作用域内声明函数。
* 函数声明类似于`var`，即会提升到全局作用域或函数作用域的头部。
* 同时，函数声明还会提升到所在的块级作用域的头部。

注意，上面三条规则只对 ES6 的浏览器实现有效，其他环境的实现不用遵守，还是将块级作用域的函数声明当作let处理。

考虑到环境导致的行为差异太大，应该避免在块级作用域内声明函数。如果确实需要，也应该写成函数表达式，而不是函数声明语句。

#### const命令

`const`声明一个只读的常量。一旦声明，常量的值就不能改变。

`const`声明的变量不得改变值，这意味着，const一旦声明变量，就必须立即初始化，不能留到以后赋值。

```
const foo;
// SyntaxError: Missing initializer in const declaration
```

`const`的作用域与`let`命令相同：只在声明所在的块级作用域内有效。

`const`命令声明的常量也是不提升，同样存在暂时性死区，只能在声明的位置后面使用。

`const`声明的常量，也与`let`一样不可重复声明。

**对于复合类型的变量，变量名不指向数据，而是指向数据所在的地址。`const`命令只是保证变量名指向的地址不变，并不保证该地址的数据不变，所以将一个对象声明为常量必须非常小心。**

如果真的想将对象冻结，应该使用`Object.freeze`方法。除了将对象本身冻结，对象的属性也应该冻结

```
var constantize = (obj) => {
  Object.freeze(obj);
  Object.keys(obj).forEach( (key, value) => {
    if ( typeof obj[key] === 'object' ) {
      constantize( obj[key] );
    }
  });
};
```

ES5只有两种声明变量的方法：`var`命令和`function`命令。ES6除了添加`let`和`const`命令，后面章节还会提到，另外两种声明变量的方法：`import`命令和`class`命令。所以，ES6一共有6种声明变量的方法。

#### 顶层对象的属性

顶层对象，在浏览器环境指的是`window`对象，在Node指的是`global`对象。ES5之中，顶层对象的属性与全局变量是等价的。

ES6为了改变这一点，一方面规定，为了保持兼容性，`var`命令和`function`命令声明的全局变量，依旧是顶层对象的属性；另一方面规定，`let`命令、`const`命令、`class`命令声明的全局变量，不属于顶层对象的属性。也就是说，从ES6开始，全局变量将逐步与顶层对象的属性脱钩。

#### global 对象

ES5的顶层对象，本身也是一个问题，因为它在各种实现里面是不统一的。

* 浏览器里面，顶层对象是`window`，但 Node 和 Web Worker 没有`window`。
* 浏览器和 Web Worker 里面，`self`也指向顶层对象，但是Node没有`self`。
* Node 里面，顶层对象是`global`，但其他环境都不支持。

* 全局环境中，this会返回顶层对象。但是，Node模块和ES6模块中，this返回的是当前模块。
* 函数里面的this，如果函数不是作为对象的方法运行，而是单纯作为函数运行，this会指向顶层对象。但是，严格模式下，这时this会返回undefined。
* 不管是严格模式，还是普通模式，new Function('return this')()，总是会返回全局对象。但是，如果浏览器用了CSP（Content Security Policy，内容安全政策），那么eval、new Function这些方法都可能无法使用。

现在有一个提案，在语言标准的层面，引入`global`作为顶层对象。也就是说，在所有环境下，`global`都是存在的，都可以从它拿到顶层对象。

垫片库`system.global`模拟了这个提案，可以在所有环境拿到`global`。

```
// CommonJS的写法
var global = require('system.global')();

// ES6模块的写法
import getGlobal from 'system.global';
const global = getGlobal();
```

上面代码将顶层对象放入变量`global`。

### 变量的解构赋值

#### 数组的解构赋值

ES6 允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这被称为解构（Destructuring）。

```
let [a, b, c] = [1, 2, 3];
```

##### 默认值

解构赋值允许指定默认值。

```
let [foo = true] = [];
foo // true

let [x, y = 'b'] = ['a']; // x='a', y='b'
let [x, y = 'b'] = ['a', undefined]; // x='a', y='b'
```

注意，ES6 内部使用严格相等运算符（`===`），判断一个位置是否有值。所以，如果一个数组成员不严格等于`undefined`，默认值是不会生效的。

#### 对象的解构赋值

解构不仅可以用于数组，还可以用于对象。

```
let { foo, bar } = { foo: "aaa", bar: "bbb" };
foo // "aaa"
bar // "bbb"
let { baz } = { foo: "aaa", bar: "bbb" };
baz // undefined
```

对象的解构与数组有一个重要的不同。数组的元素是按次序排列的，变量的取值由它的位置决定；而对象的属性没有次序，变量必须与属性同名，才能取到正确的值。

如果变量名与属性名不一致，必须写成下面这样。

```
var { foo: baz } = { foo: 'aaa', bar: 'bbb' };
baz // "aaa"

let obj = { first: 'hello', last: 'world' };
let { first: f, last: l } = obj;
f // 'hello'
l // 'world'
```

#### 字符串的解构赋值

字符串也可以解构赋值。这是因为此时，字符串被转换成了一个类似数组的对象。

```
const [a, b, c, d, e] = 'hello';
a // "h"
b // "e"
c // "l"
d // "l"
e // "o"
```

### 数值和布尔值的解构赋值

解构赋值时，如果等号右边是数值和布尔值，则会先转为对象。

```
let {toString: s} = 123;
s === Number.prototype.toString // true

let {toString: s} = true;
s === Boolean.prototype.toString // true
```

上面代码中，数值和布尔值的包装对象都有`toString`属性，因此变量s都能取到值。

解构赋值的规则是，只要等号右边的值不是对象，就先将其转为对象。由于`undefined`和`null`无法转为对象，所以对它们进行解构赋值，都会报错。


#### 函数参数的解构赋值

函数的参数也可以使用解构赋值。

```
function add([x, y]){
  return x + y;
}

add([1, 2]); // 3
```

```
[[1, 2], [3, 4]].map(([a, b]) => a + b);
// [ 3, 7 ]
```

#### 用途 

##### 交换变量的值

```
let x = 1;
let y = 2;

[x, y] = [y, x];
```

##### 从函数返回多个值

```
// 返回一个数组

function example() {
  return [1, 2, 3];
}
let [a, b, c] = example();

// 返回一个对象

function example() {
  return {
    foo: 1,
    bar: 2
  };
}
let { foo, bar } = example();
```

##### 函数参数的定义

```
// 参数是一组有次序的值
function f([x, y, z]) { ... }
f([1, 2, 3]);

// 参数是一组无次序的值
function f({x, y, z}) { ... }
f({z: 3, y: 2, x: 1});
```

##### 提取JSON数据

解构赋值对提取JSON对象中的数据，尤其有用。

```
let jsonData = {
  id: 42,
  status: "OK",
  data: [867, 5309]
};

let { id, status, data: number } = jsonData;

console.log(id, status, number);
// 42, "OK", [867, 5309]
```

##### 函数参数的默认值

```
jQuery.ajax = function (url, {
  async = true,
  beforeSend = function () {},
  cache = true,
  complete = function () {},
  crossDomain = false,
  global = true,
  // ... more config
}) {
  // ... do stuff
};
```

指定参数的默认值，就避免了在函数体内部再写`var foo = config.foo || 'default foo';`这样的语句。

##### 遍历Map结构

任何部署了`Iterator`接口的对象，都可以用`for...of`循环遍历。`Map`结构原生支持`Iterator`接口，配合变量的解构赋值，获取键名和键值就非常方便。

```
var map = new Map();
map.set('first', 'hello');
map.set('second', 'world');

for (let [key, value] of map) {
  console.log(key + " is " + value);
}
// first is hello
// second is world
```

如果只想获取键名，或者只想获取键值，可以写成下面这样。

```
// 获取键名
for (let [key] of map) {
  // ...
}

// 获取键值
for (let [,value] of map) {
  // ...
}
```

##### 输入模块的指定方法

加载模块时，往往需要指定输入那些方法。解构赋值使得输入语句非常清晰。

```
const { SourceMapConsumer, SourceNode } = require("source-map");
```

### 字符串的扩展

ES6加强了对Unicode的支持，并且扩展了字符串对象。


#### 字符的Unicode表示法

JavaScript允许采用\uxxxx形式表示一个字符，其中“xxxx”表示字符的码点。但是，这种表示法只限于\u0000——\uFFFF之间的字符。超出这个范围的字符，必须用两个双字节的形式表达。

```
"\uD842\uDFB7"
// "𠮷"

"\u20BB7"
// " 7"
```

上面代码表示，如果直接在\u后面跟上超过0xFFFF的数值（比如\u20BB7），JavaScript会理解成\u20BB+7。由于\u20BB是一个不可打印字符，所以只会显示一个空格，后面跟着一个7。

ES6 对这一点做出了改进，只要将码点放入大括号，就能正确解读该字符。

```
"\u{20BB7}"
// "𠮷"

"\u{41}\u{42}\u{43}"
// "ABC"

let hello = 123;
hell\u{6F} // 123

'\u{1F680}' === '\uD83D\uDE80'
// true
```

上面代码中，最后一个例子表明，大括号表示法与四字节的UTF-16编码是等价的。

有了这种表示法之后，JavaScript共有6种方法可以表示一个字符。

```
'\z' === 'z'  // true
'\172' === 'z' // true
'\x7A' === 'z' // true
'\u007A' === 'z' // true
'\u{7A}' === 'z' // true
```

#### codePointAt()

JavaScript内部，字符以UTF-16的格式储存，每个字符固定为`2`个字节。对于那些需要4个字节储存的字符（Unicode码点大于`0xFFFF`的字符），JavaScript会认为它们是两个字符。

```
var s = "𠮷";

s.length // 2
s.charAt(0) // ''
s.charAt(1) // ''
s.charCodeAt(0) // 55362
s.charCodeAt(1) // 57271
```

ES6提供了`codePointAt`方法，能够正确处理4个字节储存的字符，返回一个字符的码点。

```
var s = '𠮷a';

s.codePointAt(0).toString(16) // "20bb7"
s.codePointAt(2).toString(16) // "61"
```

`codePointAt`方法会正确返回32位的UTF-16字符的码点。对于那些两个字节储存的常规字符，它的返回结果与`charCodeAt`方法相同。


#### String.fromCodePoint()

ES5提供`String.fromCharCode`方法，用于从码点返回对应字符，但是这个方法不能识别32位的UTF-16字符（Unicode编号大于0xFFFF）。

ES6提供了String.fromCodePoint方法，可以识别0xFFFF的字符，弥补了String.fromCharCode方法的不足。在作用上，正好与codePointAt方法相反。

```
String.fromCodePoint(0x20BB7)
// "𠮷"
String.fromCodePoint(0x78, 0x1f680, 0x79) === 'x\uD83D\uDE80y'
// true
```

上面代码中，如果String.fromCodePoint方法有多个参数，则它们会被合并成一个字符串返回。

注意，fromCodePoint方法定义在String对象上，而codePointAt方法定义在字符串的实例对象上。

#### 字符串的遍历器接口

ES6为字符串添加了遍历器接口（详见《Iterator》一章），使得字符串可以被for...of循环遍历。

```
for (let codePoint of 'foo') {
  console.log(codePoint)
}
// "f"
// "o"
// "o"
```

除了遍历字符串，这个遍历器最大的优点是可以识别大于0xFFFF的码点，传统的for循环无法识别这样的码点。

#### includes(), startsWith(), endsWith()

传统上，JavaScript只有indexOf方法，可以用来确定一个字符串是否包含在另一个字符串中。ES6又提供了三种新方法。

* includes()：返回布尔值，表示是否找到了参数字符串。
* startsWith()：返回布尔值，表示参数字符串是否在源字符串的头部。
* endsWith()：返回布尔值，表示参数字符串是否在源字符串的尾部。

#### repeat() 

`repeat`方法返回一个新字符串，表示将原字符串重复`n`次。

```
'x'.repeat(3) // "xxx"
'hello'.repeat(2) // "hellohello"
'na'.repeat(0) // ""
```

#### padStart()，padEnd() 

ES2017 引入了字符串补全长度的功能。如果某个字符串不够指定长度，会在头部或尾部补全。`padStart()`用于头部补全，`padEnd()`用于尾部补全。

#### 模板字符串

传统的JavaScript语言，输出模板通常是这样写的。

```
$('#result').append(
  'There are <b>' + basket.count + '</b> ' +
  'items in your basket, ' +
  '<em>' + basket.onSale +
  '</em> are on sale!'
);
```

上面这种写法相当繁琐不方便，ES6引入了模板字符串解决这个问题。

```
$('#result').append(`
  There are <b>${basket.count}</b> items
   in your basket, <em>${basket.onSale}</em>
  are on sale!
`);
```

### 正则的扩展

#### RegExp构造函数

ES6改变了这种行为。如果RegExp构造函数第一个参数是一个正则对象，那么可以使用第二个参数指定修饰符。而且，返回的正则表达式会忽略原有的正则表达式的修饰符，只使用新指定的修饰符。

```
new RegExp(/abc/ig, 'i').flags
// "i"
```

上面代码中，原有正则对象的修饰符是ig，它会被第二个参数i覆盖。

#### 字符串的正则方法

字符串对象共有4个方法，可以使用正则表达式：`match()`、`replace()`、`search()`和`split()`。

ES6将这4个方法，在语言内部全部调用RegExp的实例方法，从而做到所有与正则相关的方法，全都定义在RegExp对象上。

* String.prototype.match 调用 RegExp.prototype[Symbol.match]
* String.prototype.replace 调用 RegExp.prototype[Symbol.replace]
* String.prototype.search 调用 RegExp.prototype[Symbol.search]
* String.prototype.split 调用 RegExp.prototype[Symbol.split]

#### u修饰符

S6对正则表达式添加了u修饰符，含义为“Unicode模式”，用来正确处理大于`\uFFFF`的Unicode字符。也就是说，会正确处理四个字节的UTF-16编码。

```
/^\uD83D/u.test('\uD83D\uDC2A')
// false
/^\uD83D/.test('\uD83D\uDC2A')
// true
```

上面代码中，`\uD83D\uDC2A`是一个四个字节的UTF-16编码，代表一个字符。但是，ES5不支持四个字节的UTF-16编码，会将其识别为两个字符，导致第二行代码结果为`true`。加了`u`修饰符以后，ES6就会识别其为一个字符，所以第一行代码结果为`false`。

一旦加上`u`修饰符号，就会修改一些正则表达式的行为。

#### y 修饰符

除了`u`修饰符，ES6还为正则表达式添加了`y`修饰符，叫做“粘连”（sticky）修饰符。

`y`修饰符的作用与`g`修饰符类似，也是全局匹配，后一次匹配都从上一次匹配成功的下一个位置开始。不同之处在于，`g`修饰符只要剩余位置中存在匹配就可，而y修饰符确保匹配必须从剩余的第一个位置开始，这也就是“粘连”的涵义。

进一步说，`y`修饰符号隐含了头部匹配的标志`^`。

```
var s = 'aaa_aa_a';
var r1 = /a+/g;
var r2 = /a+/y;

r1.exec(s) // ["aaa"]
r2.exec(s) // ["aaa"]

r1.exec(s) // ["aa"]
r2.exec(s) // null
```

与`y`修饰符相匹配，ES6的正则对象多了`sticky`属性，表示是否设置了`y`修饰符。

```
var r = /hello\d/y;
r.sticky // true
```

#### flags属性

ES6为正则表达式新增了flags属性，会返回正则表达式的修饰符。

```
// ES5的source属性
// 返回正则表达式的正文
/abc/ig.source
// "abc"

// ES6的flags属性
// 返回正则表达式的修饰符
/abc/ig.flags
// 'gi'
```

#### RegExp.escape()

字符串必须转义，才能作为正则模式

```
function escapeRegExp(str) {
  return str.replace(/[\-\[\]\/\{\}\(\)\*\+\?\.\\\^\$\|]/g, '\\$&');
}

let str = '/path/to/resource.html?search=query';
escapeRegExp(str)
// "\/path\/to\/resource\.html\?search=query"
```

用字符串生成正则匹配模式：

```
RegExp.escape('The Quick Brown Fox');
// "The Quick Brown Fox"

RegExp.escape('Buy it. use it. break it. fix it.');
// "Buy it\. use it\. break it\. fix it\."

RegExp.escape('(*.*)');
// "\(\*\.\*\)"
```

#### s 修饰符：dotAll 模式

正则表达式中，点（`.`）是一个特殊字符，代表任意的单个字符，但是行终止符（line terminator character）除外。

以下四个字符属于”行终止符“。

* U+000A 换行符（\n）
* U+000D 回车符（\r）
* U+2028 行分隔符（line separator）
* U+2029 段分隔符（paragraph separator）

```
/foo.bar/.test('foo\nbar')
// false
```

现在有一个提案，引入`/s`修饰符，使得`.`可以匹配任意单个字符。

```
/foo.bar/s.test('foo\nbar') // true
```

### 数值的扩展

#### 二进制和八进制表示法 

ES6 提供了二进制和八进制数值的新的写法，分别用前缀`0b`（或`0B`）和`0o`（或`0O`）表示。

```
0b111110111 === 503 // true
0o767 === 503 // true
```

如果要将`0b`和`0o`前缀的字符串数值转为十进制，要使用`Number`方法。

```
Number('0b111')  // 7
Number('0o10')  // 8
```

#### Number.isFinite(), Number.isNaN()

ES6在`Number`对象上，新提供了`Number.isFinite()`和`Number.isNaN()`两个方法。

`Number.isFinite()`用来检查一个数值是否为有限的（`finite`）。

`Number.isNaN()`用来检查一个值是否为`NaN`。

它们与传统的全局方法`isFinite()`和`isNaN()`的区别在于，传统方法先调用`Number()`将非数值的值转为数值，再进行判断，而这两个新方法只对数值有效，非数值一律返回`false`。

#### Number.parseInt(), Number.parseFloat()

ES6将全局方法`parseInt()`和`parseFloat()`，移植到Number对象上面，行为完全保持不变。

```
// ES5的写法
parseInt('12.34') // 12
parseFloat('123.45#') // 123.45

// ES6的写法
Number.parseInt('12.34') // 12
Number.parseFloat('123.45#') // 123.45
```

#### Number.isInteger()

`Number.isInteger()`用来判断一个值是否为整数。需要注意的是，在JavaScript内部，整数和浮点数是同样的储存方法，所以3和3.0被视为同一个值。

```
Number.isInteger(25) // true
Number.isInteger(25.0) // true
Number.isInteger(25.1) // false
Number.isInteger("15") // false
Number.isInteger(true) // false
```

#### Number.EPSILON

ES6在Number对象上面，新增一个极小的常量`Number.EPSILON`。

引入一个这么小的量的目的，在于为浮点数计算，设置一个误差范围。

但是如果这个误差能够小于`Number.EPSILON`，我们就可以认为得到了正确结果。

因此，`Number.EPSILON`的实质是一个可以接受的误差范围。

```
Number.EPSILON
// 2.220446049250313e-16
Number.EPSILON.toFixed(20)
// '0.00000000000000022204'
5.551115123125783e-17 < Number.EPSILON
// true
```

#### 安全整数和Number.isSafeInteger()

JavaScript能够准确表示的整数范围在`-2^53`到`2^53`之间（不含两个端点），超过这个范围，无法精确表示这个值。

```
Math.pow(2, 53) // 9007199254740992

9007199254740992  // 9007199254740992
9007199254740993  // 9007199254740992

Math.pow(2, 53) === Math.pow(2, 53) + 1
// true
```

ES6引入了`Number.MAX_SAFE_INTEGER`和`Number.MIN_SAFE_INTEGER`这两个常量，用来表示这个范围的上下限。`Number.isSafeInteger()`则是用来判断一个整数是否落在这个范围之内。

```
Number.MAX_SAFE_INTEGER === Math.pow(2, 53) - 1
// true
Number.MAX_SAFE_INTEGER === 9007199254740991
// true

Number.MIN_SAFE_INTEGER === -Number.MAX_SAFE_INTEGER
// true
Number.MIN_SAFE_INTEGER === -9007199254740991
// true
```

### Math对象的扩展

#### Math.trunc()

`Math.trunc`方法用于去除一个数的小数部分，返回整数部分。

```
Math.trunc(4.1) // 4
Math.trunc(4.9) // 4
Math.trunc(-4.1) // -4
Math.trunc(-4.9) // -4
Math.trunc(-0.1234) // -0
```

#### Math.sign()

`Math.sign`方法用来判断一个数到底是正数、负数、还是零。

它会返回五种值。

* 参数为正数，返回+1；
* 参数为负数，返回-1；
* 参数为0，返回0；
* 参数为-0，返回-0;
* 其他值，返回NaN。

#### Math.cbrt()

`Math.cbrt`方法用于计算一个数的立方根。

#### Math.clz32()

JavaScript的整数使用32位二进制形式表示，`Math.clz32`方法返回一个数的32位无符号整数形式有多少个前导0。












## ES6 转码

### Babel转码器

[Babel](https://babeljs.io) 是一个广泛使用的ES6转码器，可以将ES6代码转为ES5代码，从而在现有环境执行。这意味着，你可以用ES6的方式编写程序，又不用担心现有环境是否支持。下面是一个例子。

```
// 转码前
input.map(item => item + 1);

// 转码后
input.map(function (item) {
  return item + 1;
});
```

上面的原始代码用了箭头函数，这个特性还没有得到广泛支持，Babel将其转为普通函数，就能在现有的JavaScript环境执行了。

### 配置文件.babelrc

Babel的配置文件是`.babelrc`，存放在项目的根目录下。使用Babel的第一步，就是配置这个文件。

该文件用来设置转码规则和插件，基本格式如下

```
{
  "presets": [],
  "plugins": []
}
```

`presets`字段设定转码规则，官方提供以下的规则集，你可以根据需要安装。

```
# ES2015转码规则
$ npm install --save-dev babel-preset-es2015

# react转码规则
$ npm install --save-dev babel-preset-react

# ES7不同阶段语法提案的转码规则（共有4个阶段），选装一个
$ npm install --save-dev babel-preset-stage-0
$ npm install --save-dev babel-preset-stage-1
$ npm install --save-dev babel-preset-stage-2
$ npm install --save-dev babel-preset-stage-3
```
然后，将这些规则加入`.babelrc`。

```
{
    "presets": [
      "es2015",
      "react",
      "stage-2"
    ],
    "plugins": []
  }
```

注意，以下所有Babel工具和模块的使用，都必须先写好`.babelrc`。

### 命令行转码babel-cli

Babel提供`babel-cli`工具，用于命令行转码。

它的安装命令如下。

```
$ npm install --global babel-cli
```

```
# 转码结果输出到标准输出
$ babel example.js

# 转码结果写入一个文件
# --out-file 或 -o 参数指定输出文件
$ babel example.js --out-file compiled.js
# 或者
$ babel example.js -o compiled.js

# 整个目录转码
# --out-dir 或 -d 参数指定输出目录
$ babel src --out-dir lib
# 或者
$ babel src -d lib

# -s 参数生成source map文件
$ babel src -d lib -s
```

上面代码是在全局环境下，进行Babel转码。这意味着，如果项目要运行，全局环境必须有Babel，也就是说项目产生了对环境的依赖。另一方面，这样做也无法支持不同项目使用不同版本的Babel。

一个解决办法是将`babel-cli`安装在项目之中。

```
# 安装
$ npm install --save-dev babel-cli
```

然后，改写`package.json`。

```
{
  // ...
  "devDependencies": {
    "babel-cli": "^6.0.0"
  },
  "scripts": {
    "build": "babel src -d lib"
  },
}
```

转码的时候，就执行下面的命令。

```
$ npm run build
```

### babel-node

`babel-cli`工具自带一个`babel-node`命令，提供一个支持ES6的REPL环境。它支持Node的REPL环境的所有功能，而且可以直接运行ES6代码。

它不用单独安装，而是随`babel-cli`一起安装。然后，执行babel-node`就进入REPL环境。

```
$ babel-node
> (x => x * 2)(1)
2
```

`babel-node`命令可以直接运行ES6脚本。将上面的代码放入脚本文件es6.js，然后直接运行。

```
$ babel-node es6.js
2
```

`babel-node`也可以安装在项目中。

```
$ npm install --save-dev babel-cli
```

然后，改写`package.json`

```
{
  "scripts": {
    "script-name": "babel-node script.js"
  }
}
```

上面代码中，使用`babel-node`替代`node`，这样`script.js`本身就不用做任何转码处理。

### babel-core

如果某些代码(Webpack 代码)需要调用Babel的API进行转码，就要使用`babel-core`模块。

安装命令如下。

```
npm install babel-core --save
```

然后，在项目中就可以调用babel-core。

```
var babel = require('babel-core');

// 字符串转码
babel.transform('code();', options);
// => { code, map, ast }

// 文件转码（异步）
babel.transformFile('filename.js', options, function(err, result) {
  result; // => { code, map, ast }
});

// 文件转码（同步）
babel.transformFileSync('filename.js', options);
// => { code, map, ast }

// Babel AST转码
babel.transformFromAst(ast, code, options);
// => { code, map, ast }
```

配置对象`options`，可以参看官方文档http://babeljs.io/docs/usage/options/。

下面是一个例子。

```
var es6Code = 'let x = n => n + 1';
var es5Code = require('babel-core')
  .transform(es6Code, {
    presets: ['es2015']
  })
  .code;
// '"use strict";\n\nvar x = function x(n) {\n  return n + 1;\n};'
```

上面代码中，`transform`方法的第一个参数是一个字符串，表示需要被转换的ES6代码，第二个参数是转换的配置对象。

### babel-polyfill

Babel默认只转换新的JavaScript句法（syntax），而不转换新的API，比如Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise等全局对象，以及一些定义在全局对象上的方法（比如`Object.assign`）都不会转码。

举例来说，ES6在`Array`对象上新增了`Array.from`方法。Babel就不会转码这个方法。如果想让这个方法运行，必须使用`babel-polyfill`，为当前环境提供一个垫片。

安装命令如下。

```
$ npm install --save babel-polyfill
```

然后，在脚本头部，加入如下一行代码。

```
import 'babel-polyfill';
// 或者
require('babel-polyfill');
```

Babel默认不转码的API非常多，详细清单可以查看babel-plugin-transform-runtime模块的[definitions.js](https://github.com/babel/babel/blob/master/packages/babel-plugin-transform-runtime/src/definitions.js)文件。

### 在线转换

Babel提供一个[REPL在线编译器](https://babeljs.io/repl/)，可以在线将ES6代码转为ES5代码。转换后的代码，可以直接作为ES5代码插入网页运行。


