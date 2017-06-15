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

## Object 函数方法之存在性

### in

```
var myObject = {
	a: 2
};

("a" in myObject);				// true
("b" in myObject);				// false
```

`in` 操作符会检查属性是否存在于对象 中，**无论是否可枚举**，或者是否存在于[[Prototype]]链对象遍历的更高层中。相比之下，`hasOwnProperty(..)` **仅仅** 检查 `myObject` 是否拥有属性，但 **不会** 查询[[Prototype]]链。

```
注意：

4 in [2, 4, 6] 的结果是 false，因为这个数组中包含的属性名是 0、1、2，没有 4
```

`for...in...` 会返回所有的存在于对象，或者否存在于[[Prototype]]链对象遍历的更高层中，**所有的可枚举** 属性值。**这一点和 `in` 操作符有区别。**最好将 `for..in` 循环 **仅** 用于对象。

```
注意：

（当下）没有与in操作符的查询方式（在整个[[Prototype]]链上遍历所有的属性）等价的，内建的方法可以得到一个 所有属性 的列表。你可以近似地模拟一个这样的工具：递归地遍历一个对象的[[Prototype]]链，在每一层都从Object.keys(..)中取得一个列表——仅包含可枚举属性。
```


### propertyIsEnumerable(..)

`propertyIsEnumerable(..)` 测试一个给定的属性名是否直接存在于对象上（而不是在原型链上），并且是 `enumerable:true`。

```
var myObject = { };

Object.defineProperty(
	myObject,
	"a",
	// 使`a`可枚举，如一般情况
	{ enumerable: true, value: 2 }
);

Object.defineProperty(
	myObject,
	"b",
	// 使`b`不可枚举
	{ enumerable: false, value: 3 }
);

myObject.propertyIsEnumerable( "a" ); // true
myObject.propertyIsEnumerable( "b" ); // false
```

### Object.keys(..)

`Object.keys(..) `只会查找对象直接包含的属性，而不查找[[prototype]] 链。

`Object.keys(..)` 返回一个所有 **可枚举** 属性的数组

### Object.getOwnPropertyNames(..)

`Object.getOwnPropertyNames(..) `只会查找对象直接包含的属性，而不查找[[prototype]] 链。

`Object.getOwnPropertyNames(..)` 返回一个 **所有** 属性的数组，不论能不能枚举。

```
var myObject = { };

Object.defineProperty(
	myObject,
	"a",
	// 使`a`可枚举，如一般情况
	{ enumerable: true, value: 2 }
);

Object.defineProperty(
	myObject,
	"b",
	// 使`b`不可枚举
	{ enumerable: false, value: 3 }
);

Object.keys( myObject ); // ["a"]
Object.getOwnPropertyNames( myObject ); // ["a", "b"]
```

## Object 函数方法之 Prototype 相关

### getPrototypeOf(...)

所有的函数默认都会得到一个公有的，不可枚举的属性，称为prototype，它可以指向一个对象。

这个函数会返回某个对象的构造函数的 prototype。

```
function Foo() {
	// ...
}

var a = new Foo();

Object.getPrototypeOf( a ) === Foo.prototype; // true
```

### create(...)

`Object.create(..)` 凭空 创建 了一个“新”对象，并将这个新对象内部的[[Prototype]]链接到你指定的对象上

```
function Foo(name) {
	this.name = name;
}

Foo.prototype.myName = function() {
	return this.name;
};

function Bar(name,label) {
	Foo.call( this, name );
	this.label = label;
}

// 这里，我们创建一个新的`Bar.prototype`链接链到`Foo.prototype`
Bar.prototype = Object.create( Foo.prototype );

// 注意！现在`Bar.prototype.constructor`不存在了，
// 如果你有依赖这个属性的习惯的话，可以被手动“修复”。

Bar.prototype.myLabel = function() {
	return this.label;
};

var a = new Bar( "a", "obj a" );

a.myName(); // "a"
a.myLabel(); // "obj a"
```


## Object.prototype 的方法

### hasOwnProperty(..)

通过委托到 `Object.prototype`，所有的普通对象都可以访问 `hasOwnProperty(..)`。但是创建一个不链接到 `Object.prototype` 的对象也是可能的（通过 `Object.create(null)`）。这种情况下，像 `myObject.hasOwnProperty(..)` 这样的方法调用将会失败。

在这种场景下，一个进行这种检查的更健壮的方式是 `Object.prototype.hasOwnProperty.call(myObject,"a")` ，它借用基本的 `hasOwnProperty(..)` 方法而且使用 明确的 `this` 绑定

`hasOwnProperty(..)` **仅仅** 检查 `myObject` 是否拥有属性，**无论能不能枚举**，但 **不会** 查询[[Prototype]]链。

### isPrototypeOf(...)

某个对象是否出现在另一个对象的 **整条** [[prototype]] 链中。

```
Foo.prototype.isPrototypeOf( a ); // true
```

表示：在a的整个[[Prototype]]链中，`Foo.prototype` 出现过吗。


### toString()

返回的是代表该对象的 String 类型的值。

### valueOf()

返回的是该 object 对应的基本数据类型的值（string, boolean, number, null, undifined, symbol）。

## 参考

[You-Dont-Know-JS](https://github.com/getify/You-Dont-Know-JS)
