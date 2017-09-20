---
title: Babel帮我们做了啥？
date: 2017-09-19 17:43:11
categories: coding
tags:
  - JavaScript
  - Babel
---

Babel 作为一个能够让我们愉快地用上 ES6 的工具，大大地解放了我们的“生产力”。但是，在我使用 Babel 的时候，总是很好奇，Babel 到底把我们的代码转换成什么样子了呢？会不会在其中会有什么坑呢？所以在本文中，我们会针对一些测试用例，来查看一下 Babel 转换的结果到底是什么。

<!--more-->

为了测试 Babel 为我们做了什么事情，我们需要在本地安装 Babel，下面简要介绍一下 Babel 的相关库以及其作用，以及我们如何安装并使用 Babel。

关于不同版本的 Babel presets 能处理什么样的事情，在 [如何写好.babelrc？Babel的presets和plugins配置解析](https://excaliburhan.com/post/babel-preset-and-plugins.html)，以及 [如何区分Babel中的stage-0,stage-1,stage-2以及stage-3](http://www.cnblogs.com/chris-oil/p/5717544.html)  这两篇文章中有介绍。

在我们的测试中，我要测试的主要是 ES6 被转换到 ES5 的版本，所以需要添加的转码规则（presets）有：`babel-preset-es2015`

并且为了添加一些 ES7 的提案，我们也会在转码规则中添加 `babel-preset-stage-0`，`stage-0` 是包含`stage-1, stage-2以及stage-3`的所有功能

最后，我使用两个文件，一个叫做 `example.js`，用来装转换前的代码，`compiled.js` 用来装转换后的代码。

然后在 `package.json` 里面添加 `script`：

```
"scripts": {
    "build": "babel example.js --out-file compiled.js"
},
```

然后尽情开始我的测试吧：

测试的顺序按照 [ ECMAScript 6 入门 | 阮一峰](http://es6.ruanyifeng.com) 以及 [babel到底将代码转换成什么鸟样？](https://github.com/lcxfs1991/blog/issues/9) 这两篇文章，并且加入一些我自己的额外测试用例。

## var, let, const

const和let现在一律转换成var。那const到底如何保证不变呢？如果你在源码中第二次修改const常量的值，babel编译会直接报错。

转换前

```
var a = 1;
let b = 2;
const c = 3;
```

转换后：

```
var a = 1;
var b = 2;
var c = 3;
```

如果对 `c` 进行赋值 `c = 1`，则编译的结果为 Babel 报错：

```
SyntaxError: example.js: "c" is read-only
```

但是有一些特性是 Babel 没有实现的，如**不存在变量提升：**

转换前

```
console.log(b)
let b = 2;
```

转换后：

```
console.log(b);
var b = 2;
```

我们可以看到，在编译后的代码中，并**没有实现限制变量提升的部分**。

## 变量的解构赋值

转换前

```
let a = {
  first: 1,
  second: 2,
}

let {first, second} = a;
```

转换后：

```
"use strict";

var a = {
  first: 1,
  second: 2
};

var first = a.first,
    second = a.second;
```

也就是分别创建了并且赋值该对象。

如果是对数组进行赋值，则会创建预先定义一个变量存储数组的值：

转换前

```
let [a1, a2, a3] = [1, 2]
```

转换后：

```
"use strict";

var _ref = [1, 2],
    a1 = _ref[0],
    a2 = _ref[1],
    a3 = _ref[2];
```

如果使用匿名对象直接进行赋值解构会怎样呢？如下。babel为了使接收的变量唯一，直接就将匿名对象里的属性拼在一起，组成接收这个匿名对象的变量

```
let {first, second} = {first: 1, second: 2, third: 3}
```

转换后：

```
"use strict";

var _first$second$third = { first: 1, second: 2, third: 3 },
    first = _first$second$third.first,
    second = _first$second$third.second;
```

还有一种对象深层次的解构赋值：
转换前：

```
var obj = {
    p1: [
        "Hello",
        { p2: "World" }
    ]
};

var { p1: [s1, { p2 }] } = obj;
```

转换后：

```
// 为解释本人将代码美化了
var _slicedToArray = (function() {
    function sliceIterator(arr, i) {
        var _arr = [];
        var _n = true;
        var _d = false;
        var _e = undefined;
        try {
           // 用Symbol.iterator造了一个可遍历对象，然后进去遍历。
            for (var _i = arr[Symbol.iterator](), _s; !(_n = (_s = _i.next()).done); _n = true) {
                _arr.push(_s.value);
                if (i && _arr.length === i) break;
            }
        } catch (err) {
            _d = true;
            _e = err;
        } finally {
            try {
                if (!_n && _i["return"]) _i["return"]();
            } finally {
                if (_d) throw _e;
            }
        }
        return _arr;
    }
    return function(arr, i) {
        if (Array.isArray(arr)) {
            return arr;
        } else if (Symbol.iterator in Object(arr)) {
            return sliceIterator(arr, i);
        } else {
            throw new TypeError("Invalid attempt to destructure non-iterable instance");
        }
    };
})();

var obj = {
   p1: ["Hello", { p2: "World" }]
};

var _obj$p = _slicedToArray(obj.p1, 2);

var s1 = _obj$p[0];
var p2 = _obj$p[1].p2;
```

babel在代码顶部生产了一个公共的代码_slicedToArray。大概就是将对象里面的一些属性转换成数组，方便解构赋值的进行。但Symbol.iterator的兼容性并不好

解构带有默认值的：

转换前

```
let [x, y = 'b'] = ['a']; // x='a', y='b'
```

转换后：

```
'use strict';

var _ref = ['a'],
    x = _ref[0],
    _ref$ = _ref[1],
    y = _ref$ === undefined ? 'b' : _ref$; // x='a', y='b'
```

### 函数参数的默认值

转换前

```
function f(x = 1, y = 2) {
  return [x, y]
}
```

转换后：

```
"use strict";

function f() {
  var x = (arguments.length > 0 && arguments[0] !== undefined) ? arguments[0] : 1;
  var y = (arguments.length > 1 && arguments[1] !== undefined) ? arguments[1] : 2;

  return [x, y];
}
```

为了方便理解，我将 `arguments.length > 0 && arguments[0] !== undefined` 这一段添加了括号，来查看运算符的优先级。其中，对 `arguments[0] !== undefined` 做了一次判断，因为存在以下的情况：

```
function f(x = 1, y = 2) {
  return [x, y]
}

console.log(f(undefined))
```

在 `x` 赋值为 `undefined` 的情况下，还是使用 `x` 的默认值 `1`。

针对多个输入参数的写法：`...rest`，Babel 会做如下的转换：

转换前

```
function func(x, ...y) {
  console.log(x);
  console.log(y);
  return x * y.length;
}
```

转换后：

```
"use strict";

function func(x) {
  console.log(x);

  for (var _len = arguments.length, y = Array(_len > 1 ? _len - 1 : 0), _key = 1; _key < _len; _key++) {
    y[_key - 1] = arguments[_key];
  }

  console.log(y);
  return x * y.length;
}
```

也就是创建一变量 `y`，然后遍历 `arguments`，将剩下的参数都拷贝到 `y` 之中。

## 箭头函数

在 Babel 中对箭头函数的处理，和我们的处理方法一样，都是讲 `this` 对象先存下来

转换前

```
function foo() {
  setTimeout(() => {
    console.log('id:', this.id);
  }, 100);
}
```

转换后：

```
'use strict';

function foo() {
  var _this = this;

  setTimeout(function () {
    console.log('id:', _this.id);
  }, 100);
}
```

如果要在箭头函数中使用 `arguments` 等不可以使用的对象，或者对 `this` 进行赋值，在 Babel 编译的时候就会报错：

## 对象的扩展

es2015开始新增了在对象中用中括号解释属性的功能，这对变量、常量等当对象属性尤其有用。
转换前：

```
const prop2 = "PROP2";
var obj = {
    ['prop']: 1,
    ['func']: function() {
        console.log('func');
    },
        [prop2]: 3
};
```

转换后：

```
var _obj;
// 已美化
function _defineProperty(obj, key, value) {
    if (key in obj) {
        Object.defineProperty(obj, key, {
            value: value,
            enumerable: true,
            configurable: true,
            writable: true
        });
    } else {
        obj[key] = value;
    }
    return obj;
}

var prop2 = "PROP2";
var obj = (_obj = {}, _defineProperty(_obj, 'prop', 1), _defineProperty(_obj, 'func', function func() {
    console.log('func');
}), _defineProperty(_obj, prop2, 3), _obj);
```

其实主要是用 `Object.defineProperty` 为已有的对象添加属性。

以前我们一般都用obj.prototype或者尝试用this去往上寻找prototype上面的方法。而babel则自己写了一套在prototype链上寻找方法/属性的算法。
转换前

```
var obj = {
    toString() {
     // Super calls
     return "d " + super.toString();
    },
};
```

转换后：

```
var _obj;
// 已美化
var _get = function get(object, property, receiver) {
   // 如果prototype为空，则往Function的prototype上寻找
    if (object === null) object = Function.prototype;
    var desc = Object.getOwnPropertyDescriptor(object, property);
    if (desc === undefined) {
        var parent = Object.getPrototypeOf(object);
        // 如果在本层prototype找不到，再往更深层的prototype上找
        if (parent === null) {
            return undefined;
        } else {
            return get(parent, property, receiver);
        }
    }
    // 如果是属性，则直接返回
    else if ("value" in desc) {
        return desc.value;
    }
    // 如果是方法，则用call来调用，receiver是调用的对象 
    else {
        var getter = desc.get;  // getOwnPropertyDescriptor返回的getter方法
        if (getter === undefined) {
            return undefined;
        }
        return getter.call(receiver);
    }
};

var obj = _obj = {
  toString: function toString() {
    // Super calls
    return "d " + _get(_obj.__proto__ || Object.getPrototypeOf(_obj), "toString", this).call(this);
  }
};
```

## 类

javascript实现oo一直是非常热门的话题。从最原始时代需要手动维护在构造函数里调用父类构造函数,到后来封装好函数进行extend继承，再到babel出现之后可以像其它面向对象的语言一样直接写class。es2015的类方案仍然算是过渡方案，它所支持的特性仍然没有涵盖类的所有特性。目前主要支持的有：

* constructor
* static方法
* get 方法
* set 方法
* 类继承
* super调用父类方法。

我们来一个一个看：

转换前：

```
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }
}
```

转换后：

首先，定义 `_createClass` 函数，用来在一个构造函数上添加非静态方法和静态方法：

```
var _createClass = (function() {
    function defineProperties(target, props) {
        for (var i = 0; i < props.length; i++) {
            var descriptor = props[i];
            // es6规范要求类方法为non-enumerable
            descriptor.enumerable = descriptor.enumerable || false;
            descriptor.configurable = true;
            // 对于setter和getter方法，writable为false
            if ("value" in descriptor) descriptor.writable = true;
            Object.defineProperty(target, descriptor.key, descriptor);
        }
    }
    return function(Constructor, protoProps, staticProps) {
        // 非静态方法定义在原型链上
        if (protoProps) defineProperties(Constructor.prototype, protoProps);
        // 静态方法直接定义在constructor函数上，这样静态方法就不会被对象所继承，但是会被子类构造函数继承
        if (staticProps) defineProperties(Constructor, staticProps);
        return Constructor;
    };
})();
```

然后再生成 `_classCallCheck` 函数，用来在构造函数中判断实例的类型：

```
// 检测constructor正确与否
function _classCallCheck(instance, Constructor) {
    if (!(instance instanceof Constructor)) {
        throw new TypeError("Cannot call a class as a function");
    }
}
```

最后，调用以上两个函数来实现类的创建：

```
var Point = function () {
  function Point(x, y) {
    _classCallCheck(this, Point);

    this._x = x;
    this._y = y;
  }

  _createClass(Point, [{
    key: 'toString',
    value: function toString() {
      return '(' + this._x + ', ' + this._y + ')';
    }
  }]);

  return Point;
}();
```

与 ES5 一样，在“类”的内部可以使用get和set关键字，对某个属性设置存值函数和取值函数，拦截该属性的存取行为。

我们再来看看 get 和 set 关键字如何在 Babel 中进行处理的：

转换前：

```
class Point {
  constructor(x, y) {
    this._x = x;
    this._y = y;
  }

  toString() {
    return '(' + this._x + ', ' + this._y + ')';
  }

  get x() {
    return "point x: " + this._x;
  }
  set x(value) {
    console.log('set x: ' + value);
    this._x = value;
  }
}
```

其中，`_createClass` 和 `_classCallCheck` 并没有什么变化，而是在构造函数创建的时候发生了一些变化：

```
var Point = function () {
  function Point(x, y) {
    _classCallCheck(this, Point);

    this._x = x;
    this._y = y;
  }

  _createClass(Point, [{
    key: 'toString',
    value: function toString() {
      return '(' + this._x + ', ' + this._y + ')';
    }
  }, {
    key: 'x',
    get: function get() {
      return "point x: " + this._x;
    },
    set: function set(value) {
      console.log('set x: ' + value);
      this._x = value;
    }
  }]);

  return Point;
}();
```

因为 ES6 明确规定，Class 内部只有静态方法，没有静态属性，所以我们可以看一看静态方法怎么写：

转换前：

```
class Point {
  constructor(x, y) {
    this._x = x;
    this._y = y;
  }

  static classMethod() {
    return "Point class method";
  }
}
```

转换后：

```
var Point = function () {
  function Point(x, y) {
    _classCallCheck(this, Point);

    this._x = x;
    this._y = y;
  }

  _createClass(Point, null, [{
    key: "classMethod",
    value: function classMethod() {
      return "Point class method";
    }
  }]);

  return Point;
}();
```

其实也就是将 `classMethod` 传递给了 `_createClass` 的第三个参数。

我们再来看看类继承是如何实现的：

转化前：

```
class Point {
  constructor(x, y) {
    this._x = x;
    this._y = y;
  }

  static classMethod() {
    return "Point class method";
  }
}

class subPoint extends Point {
  constructor(x, y, z) {
    super(x, y);
    this._z = z;
  }

  toString() {
    return "this is subPoing instance";
  }
}
```

转换后：

为了实现类的继承，Babel 还构造了几个函数来处理继承的逻辑：

```
// 继承类
function _inherits(subClass, superClass) {
   // 父类一定要是function类型
    if (typeof superClass !== "function" && superClass !== null) {
        throw new TypeError("Super expression must either be null or a function, not " + typeof superClass);
    }
    // 使原型链subClass.prototype.__proto__指向父类superClass，同时保证constructor是subClass自己
    subClass.prototype = Object.create(superClass && superClass.prototype, {
        constructor: {
            value: subClass,
            enumerable: false,
            writable: true,
            configurable: true
        }
    });
    // 保证subClass.__proto__指向父类superClass
    if (superClass) 
        Object.setPrototypeOf ? 
        Object.setPrototypeOf(subClass, superClass) :    subClass.__proto__ = superClass;
}
```

该函数主要的功能是，判断父类是否是合理的构造函数，如果是合理构造函数，则生成 `subClass.prototype.constructor`，并且将 `subClass.__proto__ = superClass`，用来保证构造函数的继承和静态函数的继承。

除此以外，还有 `_possibleConstructorReturn` 这个函数：

```
// 子类实现constructor
function _possibleConstructorReturn(self, call) {
    if (!self) {
        throw new ReferenceError("this hasn't been initialised - super() hasn't been called");
    }
    // 若call是函数/对象则返回
    return call && (typeof call === "object" || typeof call === "function") ? call : self;
}
```

最后，使用上面定义的函数，实现继承部分的逻辑：

```
var subPoint = function (_Point) {
  _inherits(subPoint, _Point);

  function subPoint(x, y, z) {
    _classCallCheck(this, subPoint);

    var _this = _possibleConstructorReturn(this, (subPoint.__proto__ || Object.getPrototypeOf(subPoint)).call(this, x, y));

    _this._z = z;
    return _this;
  }
  
  _createClass(subPoint, [{
    key: "toString",
    value: function toString() {
      return "this is subPoing instance";
    }
  }]);

  return subPoint;
}(Point);
```

所以，继承的实现逻辑为：首先创建一个子类对象 `this`，然后调用 `superClass.call(this)` 来将父类中的属性赋值到 `this` 上。

## 模块化

这一部分直接使用 [babel到底将代码转换成什么鸟样？](https://github.com/lcxfs1991/blog/issues/9) 中的结论了


