---
title: JavaScript 的类与对象
date: 2017-06-06 00:22:42
categories: coding
tags:
  - JavaScript
---

JavaScript 的类与对象，是一个非常大的命题，即使是已经使用了很久 JavaScript 的人，都有可能对其中的知识点掌握出错，JavaScript 的类的实现和继承机制在性质上与 Java 的有着明显的区别，这也让很多从别的语言转过来的开发者不适应。在本文中，我们将从多点对 JavaScript 的类与对象进行讨论，并且从 Object 类提供的 API 和开发中的使用方式来加强对类以及对象的理解。

<!--more-->


## 对象

### 内置对象

在 JS 中，函数就是一种对象，是**对象的一个子类型**。

JS 中有一些对象子类型，通常被称为内置对象，他们有：

* String
* Number
* Boolean
* Object
* Function
* Array
* Date
* RegExp
* Error

在 JavaScript 中，他们实际上是一些内置的函数。

### 对象属性的访问

如果要访问一个对象中在位置 `a` 的值，我们需要使用 `.` 或 `[ ]` 操作符。`.a` 语法通常称为“属性（property）”访问，而 `["a"]` 语法通常称为“键（key）”访问。在现实中，它们俩都访问相同的位置，而且会拿出相同的值

**由于["a"]语法使用字符串的值 来指定位置，这意味着程序可以动态地组建字符串的值。**

```
var myObject = {
	a: 2
};

var idx = "a";

console.log( myObject[idx] ); // 2
```


在对象中，属性名 **总是 字符串**。如果你使用字符串以外（基本）类型的值，它会首先被转换为字符串。

```
var myObject = { };

myObject[true] = "foo";
myObject[3] = "bar";
myObject[myObject] = "baz";

myObject["true"];				// "foo"
myObject["3"];					// "bar"
myObject["[object Object]"];	// "baz"
```

ES6加入了 计算型属性名，在一个字面对象声明的键名称位置，你可以指定一个表达式，用[ ]括起来：

```
var prefix = "foo";

var myObject = {
	[prefix + "bar"]: "hello",
	[prefix + "baz"]: "world"
};

myObject["foobar"]; // hello
myObject["foobaz"]; // world
```

## Object 函数的方法之描述符

### Object.defineProperty

在ES5中，所有的属性都用 **属性描述符（Property Descriptors）** 来描述。

```
var myObject = {};

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: true,
	configurable: true,
	enumerable: true
} );
```

#### 写性（writable）

writable控制着你改变属性值的能力。

考虑这段代码：

```
var myObject = {};

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: false, // 不可写！
	configurable: true,
	enumerable: true
} );

myObject.a = 3;

myObject.a; // 2

"use strict";

myObject.a = 3; // TypeError
```

我们对value的修改悄无声息地失败了。如果我们在strict mode下进行尝试，会得到一个错误

```
注意：

简单地说，你可以观察到writable:false意味着值不可改变，和你定义一个空的setter是有些等价的。实际上，你的空setter在被调用时需要扔出一个TypeError，来和writable:false保持一致。
```

#### 可配置性（configurable）

只要属性当前是可配置的，我们就可以使用同样的 `defineProperty(..)` 工具，修改它的描述符定义。

当 `configurable: false` 的时候，使用 `defineProperty(..)` 就会报错。

```
var myObject = {
	a: 2
};

myObject.a = 3;
myObject.a;					// 3

Object.defineProperty( myObject, "a", {
	value: 4,
	writable: true,
	configurable: false,	// 不可配置！
	enumerable: true
} );

myObject.a;					// 4
myObject.a = 5;
myObject.a;					// 5

Object.defineProperty( myObject, "a", {
	value: 6,
	writable: true,
	configurable: true,
	enumerable: true
} ); // TypeError
```

将 `configurable` 设置为 `false` 是 一个**单向操作，不可撤销**！

`configurable:false` 阻止的另外一个事情是使用 `delete` 操作符移除既存属性的能力。

#### 可枚举性（Enumerable）

这个性质控制着一个属性是否能在特定的对象属性枚举操作中出现，比如 `for..in` 循环。设置为 `false` 将会阻止它出现在这样的枚举中，即使它依然完全是可以访问的。设置为 `true` 会使它出现。

#### [[Get]]

```
var myObject = {
	a: 2
};

myObject.a; // 2
```

`myObject.a` 是一个属性访问，上面的代码实际上在 `myObject` 上执行了一个 `[[Get]]` 操作（有些像 `[[Get]]()` 函数调用）。对一个对象进行默认的内建 `[[Get]]` 操作，会 **首先** 检查对象，寻找一个拥有被请求的名称的属性，如果找到，就返回相应的值。如果按照被请求的名称 **没能** 找到属性，就会遍历可能存在的 `[[Prototype]]` 链。如果无论如何都没有找到名称相同的属性，`[[Get]]` 操作会返回 `undefined` 值。

#### [[Put]]

调用 `[[Put]]` 时，它根据几个因素表现不同的行为，包括属性是否已经在对象中存在了。

如果属性存在，`[[Put]]` 算法将会大致检查：

1. 这个属性是访问器描述符吗）？如果是并存在 `setter`，就调用 `setter`。
2. 这个属性是 `writable:false` 数据描述符吗？如果是，在非strict mode下静默失败，或者在strict mode下抛出 `TypeError`。
3. 设置属性的值。

#### Getter 和 Setter

ES5引入了一个方法来覆盖这些默认操作的一部分，但不是在对象级别而是针对每个属性，就是通过getters和setters。Getter是实际上调用一个隐藏函数来取得值的属性。Setter是实际上调用一个隐藏函数来设置值的属性。

当你将一个属性定义为拥有 `getter` 或 `setter` 或两者兼备，那么属性就会被定义成为“访问器描述符”（与“数据描述符”相对）。对于访问器描述符，它的 `value` 和 `writable` 性质没有意义而被忽略，取而代之的是JS将会考虑属性的 `set` 和 `get` 性质（还有 `configurable` 和 `enumerable`）。

```
var myObject = {
	// 为`a`定义一个getter
	get a() {
		return 2;
	}
};

Object.defineProperty(
	myObject,	// 目标对象
	"b",		// 属性名
	{			// 描述符
		// 为`b`定义getter
		get: function(){ return this.a * 2 },

		// 确保`b`作为对象属性出现
		enumerable: true
	}
);

myObject.a; // 2

myObject.b; // 4
```

不管是通过在字面对象语法中使用 `get a() { .. }`，还是通过使用 `defineProperty(..)` 明确定义，我们都在对象上**创建了一个没有实际持有值的属性**。

```
myObject.a = 3;

myObject.a; // 2
```

因为我们仅为 `a` 定义了一个 `getter`，如果之后我们试着设置 `a` 的值时 `setter` 操作会忽略赋值操作。

为了让属性合理，还应当定义 `setter` 操作。

```
var myObject = {
	// 为`a`定义getter
	get a() {
		return this._a_;
	},

	// 为`a`定义setter
	set a(val) {
		this._a_ = val * 2;
	}
};

myObject.a = 2;

myObject.a; // 4
```



### Object.getOwnPropertyDescriptor

用来获得对象属性的描述符

```
var myObject = {
	a: 2
};

Object.getOwnPropertyDescriptor( myObject, "a" );
// {
//    value: 2,
//    writable: true,
//    enumerable: true,
//    configurable: true
// }
```


### Object.assign

```
语法：

Object.assign(target, ...sources)

参数：

target: The target object.
sources: The source object(s).
```

**描述**

```
The Object.assign() method only copies enumerable and own properties from a source object to a target object. It uses [[Get]] on the source and [[Set]] on the target, so it will invoke getters and setters. 
```

。它会在源对象上迭代所有的可枚举（ `enumerable` ），`owned keys`（直接拥有的键），并把它们拷贝到目标对象上（仅通过 `=` 赋值）。不过在赋值的过程中会涉及到 `[[Get]]` 和 `[[Set]]` 属性描述符。由于复制是单纯的 `=` 式赋值，任何在源对象属性的特殊性质（比如 `writable` ）在目标对象上 **都不会保留**。

在 `Object.assign(..)` 中发生的复制是单纯的 `=` 式赋值，是**浅拷贝**

## Object 函数方法之不变性

**所有** 这些方法都创建的是浅不可变性。也就是，它们仅影响对象和它的直属属性的性质。如果对象拥有对其他对象（数组，对象，函数等）的引用，那个对象的 内容 不会受影响，**任然保持可变**。

```
myImmutableObject.foo; // [1,2,3]
myImmutableObject.foo.push( 4 );
myImmutableObject.foo; // [1,2,3,4]
```

### 对象常量

通过将 `writable:false` 与 `configurable:false` 组合，你可以实质上创建了一个作为对象属性的 常量（不能被改变，重定义或删除）

```
var myObject = {};

Object.defineProperty( myObject, "FAVORITE_NUMBER", {
	value: 42,
	writable: false,
	configurable: false
} );
```

### Object.preventExtensions 防止扩展

如果你想防止一个对象被添加新的属性，但另一方面保留其他既存的对象属性，调用 `Object.preventExtensions(..)`：

```
var myObject = {
	a: 2
};

Object.preventExtensions( myObject );

myObject.b = 3;
myObject.b; // undefined
```

在 non-strict mode模式下，b的创建会无声地失败。在strict mode下，它会抛出 `TypeError`。

### Object.seal 密封

实质上在当前的对象上调用 `Object.preventExtensions(..)`，同时也将它所有的既存属性标记为 `configurable:false`。

你既不能添加更多的属性，也不能重新配置或删除既存属性（虽然你依然 可以 修改它们的值）。

### Object.freeze 冻结

它实质上在当前的对象上调用 `Object.seal(..)`，同时也将它所有的“数据访问”属性设置为 `writable:false`，所以他们的值不可改变。

你可以“深度冻结”一个对象：在这个对象上调用 `Object.freeze(..)`，然后递归地迭代所有它引用的对象（目前还没有受过影响的），然后在它们上也调用 `Object.freeze(..)`。但是要小心，这可能会影响其他（共享的）对象。


## 参考

[You-Dont-Know-JS](https://github.com/getify/You-Dont-Know-JS)
