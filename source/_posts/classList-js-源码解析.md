---
title: classList.js 源码解析
date: 2017-09-16 13:50:58
categories: coding
tags:
  - JavaScript
---

在低版本的浏览器中，对于标签元素的类型的处理大多都是通过 `className` 来实现的，`element.className` 返回的是一个字符串，当我们要添加某个类名，删除某个类名的时候，都需要在该字符串上进行很多的处理，这无异是一个很麻烦的问题。所以，在 HTML5 新规范中，添加了 `classList` 属性，该属性封装了很多已有的方法来操作标签的类，但是在我们的项目中，至少目前来看，还有很多用户的浏览器停留在低版本的状态下，并不支持该属性，所以我还是需要使用一些现有的脚本来实现该功能。在本文中，分析的是 GitHub 上 classList.js 项目，通过学习源码，来体会其中代码的精妙之处。

<!--more-->

首先放上 [classList.js | Github](https://github.com/eligrey/classList.js) 链接地址，有兴趣的同学可以直接去学习源代码。

下面，我们开始进行源代码的学习：

首先，创建一个元素，判断该元素是否存在 `classList` 这个属性。在 IE < Edge 版本的浏览器下，SVG 标签也没有 `classList` 属性。

```
// Full polyfill for browsers with no classlist support
// Including IE < Edge missing SVGElement.classlist
if (!("classList" in document.createElement("_")) 
    || document.createElementNS && !("classList" in document.createElementNS("http://www.w3.org/2000/svg","g")))
```

在判断了当前标签元素不存在 `classList` 属性后，创建一个立即执行函数，并且将 `self`，在浏览器中，也就是 `window` 作为参数传递给 `view`，下面，我们来看看在这个立即执行函数中做了什么事情。

在这个立即执行函数中，定义了实现 `classList` 属性所需要的函数以及一些参数，因为浏览器实现的原因，有的必须的函数需要自己实现，如：

`trim` 函数：

```
strTrim = String[protoProp].trim || function () {
    return this.replace(/^\s+|\s+$/g, "");
}
```

实现的是将开头和结尾的多数空格（`\s+`）替换成 `""`


`indexOf` 函数：

```
arrIndexOf = Array[protoProp].indexOf || function (item) {
    var
        i = 0
      , len = this.length
    ;
    for (; i < len; i++) {
        if (i in this && this[i] === item) {
            return i;
        }
    }
    return -1;
}
```

`checkTokenAndGetIndex` 函数，这个函数在 `arrIndexOf` 函数的基础上，判断了输入的要被寻找的字符串是否为 `""`，或者是否含有空格：

```
checkTokenAndGetIndex = function (classList, token) {
  if (token === "") {
    throw new DOMEx(
        "SYNTAX_ERR"
      , "An invalid or illegal string was specified"
    );
  }
  if (/\s/.test(token)) {
    throw new DOMEx(
        "INVALID_CHARACTER_ERR"
      , "String contains an invalid character"
    );
  }
  return arrIndexOf.call(classList, token);
}
```

设置了 `classListProto`，代表 `ClassList` 属性继承于数组，关于所有 className 处理得到的每个类名都放在这个数组中。

```
classListProto = ClassList[protoProp] = []
```

`ClassList` 函数，这个函数获得，并处理 `class`，使用正则表达式 `/\s+/` 将其分成多个单独的类名，并且因为 ClassList 继承于数组，所以可以将获得的单个类名保存在 ClassList 生成的对象自身：

```
ClassList = function (elem) {
  var
      trimmedClasses = strTrim.call(elem.getAttribute("class") || "")
    , classes = trimmedClasses ? trimmedClasses.split(/\s+/) : []
    , i = 0
    , len = classes.length
  ;
  for (; i < len; i++) {
    this.push(classes[i]);
  }
  this._updateClassName = function () {
    elem.setAttribute("class", this.toString());
  };
}
```

然后在 `classListProto` 中添加了对象共有的方法，如 `add、contains、remove、toggle、toString、item`。我们可以看看它的实现方法：

`item` 方法，返回第 i 个 class：

```
classListProto.item = function (i) {
    return this[i] || null;
};
```

`contains` 方法，通过遍历 ClassList 数组，来返回是否包含某个类名。

```
classListProto.contains = function (token) {
    token += "";
    return checkTokenAndGetIndex(this, token) !== -1;
  };
```

`add` 方法，给元素添加一个类：

```
classListProto.add = function () {
var
    tokens = arguments
  , i = 0
  , l = tokens.length
  , token
  , updated = false
;
  do {
    token = tokens[i] + "";  // 可以在形参中传入多个参数，都添加到类中
    if (checkTokenAndGetIndex(this, token) === -1) {
      this.push(token);
      updated = true;
    }
  }while (++i < l);

  if (updated) {
    this._updateClassName();
  }
};
```

注意，在上面的 add 函数中，有一个 bug，就是在调用 `elem.add()` 的时候，如果没有传递参数，则会在当前标签中生成 `"undefined"` 类名并添加到标签的 className 中，关于这个问题我已经向作者提了一个 Issue，期待他的回答。

`remove` 方法，删除一个类名：

```
classListProto.remove = function () {
    var
          tokens = arguments
        , i = 0
        , l = tokens.length
        , token
        , updated = false
        , index
    ;
    do {
        token = tokens[i] + "";
        index = checkTokenAndGetIndex(this, token);
        // 这一点还是蛮有意思的，因为之前的 className 可能不是通过 add 添加，而是已经存在于标签上，则有可能会在标签上有多个相同的类名，所以需要注意
        while (index !== -1) {
            this.splice(index, 1);
            updated = true;
            index = checkTokenAndGetIndex(this, token);
        }
    } while (++i < l);

    if (updated) {
        this._updateClassName();
    }
};
```

`toggle` 方法，如果有该类名，就删除该类名，没有则添加该类名：

```
classListProto.toggle = function (token, force) {
    // 同样，不判断输入，则可能会在类名上添加 "undifined"
    token += "";

    var
          result = this.contains(token)
        , method = result ?
            force !== true && "remove"
        :
            force !== false && "add"
    ;

    if (method) {
        this[method](token);
    }

    if (force === true || force === false) {
        return force;
    } else {
        return !result;
    }
};
```

`toString` 方法，返回类名的形式：

```
classListProto.toString = function () {
    return this.join(" ");
};
```

在定义完了 `ClassList` 之后，我们如何将 `ClassList` 属性添加到所有的标签对象的属性中呢？我们使用的方法是在 `window.Element` 构造方法的 `prototype` 中添加 `ClassList` 属性，由于所有的标签元素都是 `Element` 的对象，则他们都具有了 `ClassList` 属性：

```
if (objCtr.defineProperty) {
    var classListPropDesc = {
        get: classListGetter
      , enumerable: true
      , configurable: true
    };
    try {
      // 使用 Object.defineProperty 来为 Element.propotype 添加属性，让所有 Element 对象都具有该属性
      objCtr.defineProperty(elemCtrProto, classListProp, classListPropDesc);
    } catch (ex) { // IE 8 doesn't support enumerable:true
      // adding undefined to fight this issue https://github.com/eligrey/classList.js/issues/36
      // modernie IE8-MSW7 machine has IE8 8.0.6001.18702 and is affected
      if (ex.number === undefined || ex.number === -0x7FF5EC54) {
        classListPropDesc.enumerable = false;
        objCtr.defineProperty(elemCtrProto, classListProp, classListPropDesc);
      }
    }
  } else if (objCtr[protoProp].__defineGetter__) {
    elemCtrProto.__defineGetter__(classListProp, classListGetter);
  }
```

上面一段代码实现的就是将 `ClassList` 添加到 `Elememnt.prototype` 对象中。其中，`classListGetter` 函数返回了一个 `ClassList` 对象，在调用 `element.classList` 时候就会自动生成该对象：

```
classListGetter = function () {
  return new ClassList(this);
}
```

通过以上的代码，我们就实现了 `classList` 属性添加的功能，最后，我们再总结一下为所有元素添加 `classList` 并且提供相关函数的实现方案：

1. 创建 `ClassList` 类，并且在该类中添加对类名进行处理的 `add、remove、contains` 等方法
2. 使用 `Object.defineProperty` 来为 `window.Element.propotype` 添加 `classList` 属性，则所有 `Element` 的对象都会继承该属性。在访问属性的时候，会调用 `classListGetter`，然后返回一个 `ClassList` 对象

在 classList.js 结尾的地方，针对 IE 10/11 and Firefox < 26 版本的浏览器，本身提供了`add、remove` 方法，但是提供的方法只能处理一个参数的情况，进行了修复，修复的大致逻辑是，首先测试在一次函数调用中添加两个类名，如果添加的第二个类名没有成功，则使用一个新函数来代替原有的 `add、remove` 函数，新函数的作用是判断参数的个数，然后反复调用原有 `add、remove` 函数：

```
testElement.classList.add("c1", "c2");
  
// Polyfill for IE 10/11 and Firefox <26, where classList.add and
// classList.remove exist but support only one argument at a time.
if (!testElement.classList.contains("c2")) {
  var createMethod = function(method) {
    var original = DOMTokenList.prototype[method];

    DOMTokenList.prototype[method] = function(token) {
      var i, len = arguments.length;

      for (i = 0; i < len; i++) {
        token = arguments[i];
        original.call(this, token);
      }
    };
  };
  createMethod('add');
  createMethod('remove');
}
```

