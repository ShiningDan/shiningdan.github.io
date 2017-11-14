---
title: Vue 入门
date: 2017-10-22 12:01:02
categories: coding
tags:
  - JavaScript
  - Vue
---

本文是学习 Vue.js 的笔记

<!--more-->

## Vue 实例

每个 Vue 应用都是通过 Vue 函数创建一个新的 Vue 实例开始的：

```
var vm = new Vue({
  // 选项
})
```


当创建一个 Vue 实例时，你可以传入一个选项对象。可以在 [API 文档](https://cn.vuejs.org/v2/api/#选项-数据) 中浏览完整的选项列表。

### 选项对象参数

#### data

类型：`Object | Function`

限制：组件的定义只接受 `function`

详细：Vue 实例的数据对象。Vue 将会递归**将 data 的属性转换为 `getter/setter`**，从而让 data 的属性能够响应数据变化。大概来说，data 应该只能是数据 - **不推荐观察拥有状态行为的对象**，返回一个**初始数据**对象

实例创建之后，可以通过 `vm.$data` 访问原始数据对象。Vue 实例也代理了 data 对象上所有的属性，因此访问 `vm.a` 等价于访问 `vm.$data.a`。以 `_` 或 `$` 开头的属性 **不会** 被 Vue 实例代理，可以使用例如 `vm.$data._property` 的方式访问这些属性。

如果 `data` 仍然是一个纯粹的对象，则所有的实例将**共享引用同一个数据对象**！通过提供 `data` 函数，每次创建一个新实例后，我们能够调用 `data` 函数，从而**返回初始数据的一个全新副本数据对象**。

```
var data = { a: 1 }
// 直接创建一个实例
var vm = new Vue({
  data: data
})
vm.a // => 1
vm.$data === data // => true
// Vue.extend() 中 data 必须是函数
var Component = Vue.extend({
  data: function () {
    return { a: 1 }
  }
})
```

```
注意，不应该对 data 属性使用箭头函数 (例如data: () => { return { a: this.myProp }})。理由是箭头函数绑定了父级作用域的上下文，所以 this 将不会按照期望指向 Vue 实例，this.myProp 将是 undefined。
```

#### props

类型：`Array<string> | Object`

props 可以是数组或对象，用于接收来自父组件的数据。props 可以是简单的数组，或者使用对象作为替代，对象允许配置高级选项，如类型检测、自定义校验和设置默认值。


#### propsData

类型：`{ [key: string]: any }`

限制：只用于 new 创建的实例中。

创建实例时传递 props。主要作用是方便测试。

```
var Comp = Vue.extend({
  props: ['msg'],
  template: '<div>{{ msg }}</div>'
})
var vm = new Comp({
  propsData: {
    msg: 'hello'
  }
})
```

#### methods

类型：`{ [key: string]: Function }`

methods 将被混入到 Vue 实例中。**可以直接通过 VM 实例访问这些方法，或者在指令表达式中使用**。方法中的 this 自动绑定为 Vue 实例。

```
var vm = new Vue({
  data: { a: 1 },
  methods: {
    plus: function () {
      this.a++
    }
  }
})
vm.plus()
vm.a // 2
```

```
注意，不应该使用箭头函数来定义 method 函数 (例如 plus: () => this.a++)。理由是箭头函数绑定了父级作用域的上下文，所以 this 将不会按照期望指向 Vue 实例，this.a 将是 undefined。
```

#### computed

计算属性将被混入到 Vue 实例中。所有 getter 和 setter 的 this 上下文自动地绑定为 Vue 实例。

```
var vm = new Vue({
  data: {
    firstName : 'Foo',
    lastName : 'Bar',
    a : 1
  },
  computed: {
    // 仅读取
    aDouble: function () {
      return this.a * 2
    },
    // 读取和设置
    aPlus: {
      get: function () {
        return this.a + 1
      },
      set: function (v) {
        this.a = v - 1
      }
    },
    fullName : function() {
      return this.firstName + this.lastName;
    }
  }
})
vm.aPlus   // => 2
vm.aPlus = 3
vm.a       // => 2
vm.aDouble // => 4
```

```
注意，不应该使用箭头函数来定义计算属性函数 (例如 aDouble: () => this.a * 2)。理由是箭头函数绑定了父级作用域的上下文，所以 this 将不会按照期望指向 Vue 实例，this.a 将是 undefined。
```

**计算属性具有缓存**。计算属性是基于它们的依赖进行缓存的。计算属性只有在它的相关依赖发生改变时才会重新求值。这就意味着只要 lastName和firstName都没有发生改变，多次访问 fullName计算属性会立即返回之前的计算结果，而不必再次执行函数。

#### watch

类型：`{ [key: string]: string | Function | Object }`

一个对象，键是需要观察的表达式，值是对应回调函数。值也可以是方法名，或者包含选项的对象。Vue 实例将会在实例化时调用 $watch()，遍历 watch 对象的每一个属性。

```
var vm = new Vue({
  data: {
    a: 1,
    b: 2,
    c: 3,
    d: 4
  },
  watch: {
    a: function (val, oldVal) {
      console.log('new: %s, old: %s', val, oldVal)
    },
    // 方法名
    b: 'someMethod',
    // 深度 watcher
    c: {
      handler: function (val, oldVal) { /* ... */ },
      deep: true
    },
    // 该回调将会在侦听开始之后被立即调用
    d: {
      handler: function (val, oldVal) { /* ... */ },
      immediate: true
    }
  }
})
vm.a = 2 // => new: 2, old: 1
```

`watch`是观察一个特定的值，当该值变化时执行特定的函数。

`watch` 和 `computed` 的区别可以参考 [计算属性和观察者](https://cn.vuejs.org/v2/guide/computed.html)

### 声明周期

具体声明周期的触发方式可以查看 [Vue 实例#实例生命周期](https://cn.vuejs.org/v2/guide/instance.html#实例生命周期)

![](https://cn.vuejs.org/images/lifecycle.png)

```
所有的生命周期钩子自动绑定 this 上下文到实例中，因此你可以访问数据，对属性和方法进行运算。这意味着 你不能使用箭头函数来定义一个生命周期方法 (例如 created: () => this.fetchTodos())。这是因为箭头函数绑定了父上下文，因此 this 与你期待的 Vue 实例不同
```

### beforeCreate

在实例初始化之后，数据观测 (data observer) 和 event/watcher 事件配置之前被调用。

### created

在实例创建完成后被立即调用。在这一步，实例已完成以下的配置：数据观测 (data observer)，属性和方法的运算，watch/event 事件回调。然而，挂载阶段还没开始，`$el` 属性目前不可见。

### beforeMount

在挂载开始之前被调用：相关的 `render` 函数首次被调用。

**该钩子在服务器端渲染期间不被调用。**

### mounted

`el` 被新创建的 `vm.$el` 替换，并挂载到实例上去之后调用该钩子。如果 root 实例挂载了一个文档内元素，当 `mounted` 被调用时 `vm.$el` 也在文档内。

**重要！**注意 `mounted` 不会承诺所有的子组件也都一起被挂载。如果你希望等到整个视图都渲染完毕，可以用 `vm.$nextTick` 替换掉 `mounted`：

```
mounted: function () {
  this.$nextTick(function () {
    // Code that will run only after the
    // entire view has been rendered
  })
}
```

**该钩子在服务器端渲染期间不被调用。**

### beforeUpdate

数据更新时调用，发生在虚拟 DOM 重新渲染和打补丁之前。

**你可以在这个钩子中进一步地更改状态，这不会触发附加的重渲染过程。**

**该钩子在服务器端渲染期间不被调用。**

### updated

由于数据更改导致的虚拟 DOM 重新渲染和打补丁，在这之后会调用该钩子。

当这个钩子被调用时，组件 DOM 已经更新，所以你现在可以执行依赖于 DOM 的操作。然而在大多数情况下，你应该避免在此期间更改状态。如果要相应状态改变，通常最好使用**计算属性**或 `watcher` 取而代之。

**重要！**注意 `updated` 不会承诺所有的子组件也都一起被重绘。如果你希望等到整个视图都重绘完毕，可以用 `vm.$nextTick` 替换掉 `updated`

**该钩子在服务器端渲染期间不被调用。**

### beforeDestroy

实例销毁之前调用。在这一步，实例仍然完全可用。

**该钩子在服务器端渲染期间不被调用。**

### destroyed

Vue 实例销毁后调用。调用后，Vue 实例指示的所有东西都会解绑定，所有的事件监听器会被移除，所有的子实例也会被销毁。

**该钩子在服务器端渲染期间不被调用。**

## 模板语法

Vue 将模板编译成虚拟 DOM 渲染函数。结合响应系统，在应用状态改变时，Vue 能够智能地计算出重新渲染组件的最小代价并应用到 DOM 操作上。

也可以不用模板，[直接写渲染 (render) 函数](https://cn.vuejs.org/v2/guide/render-function.html)，使用可选的 JSX 语法。

### 插值

数据绑定最常见的形式就是使用“Mustache”语法 (双大括号) 的文本插值：

```
<span>Message: {{ msg }}</span>
```

Mustache 标签将会被替代为对应数据对象上 msg 属性的值。

双大括号会将数据解释为普通文本，而非 HTML 代码。为了输出真正的 HTML ，你需要使用 v-html 指令：

```
<div v-html="rawHtml"></div>
```

Mustache 语法不能作用在 HTML 特性上，遇到这种情况应该使用 v-bind 指令：

```
<div v-bind:id="dynamicId"></div>
```

### 常用的指令

#### v-html

预期：`string`

更新元素的 `innerHTML` 。注意：内容按普通 HTML 插入 - 不会作为 Vue 模板进行编译

```
在网站上动态渲染任意 HTML 是非常危险的，因为容易导致 XSS 攻击。只在可信内容上使用 v-html，永不用在用户提交的内容上。
```

#### v-show

预期：`any`

根据表达式之真假值，切换元素的 `display` CSS 属性。

```
当和 v-if 一起使用时，v-for 的优先级比 v-if 更高。
```

#### v-if

预期：`any`

根据表达式的值的真假条件渲染元素。在切换时元素及它的数据绑定 / 组件被销毁并重建。如果元素是` <template>` ，将提出它的内容作为条件块。

#### v-else

限制：前一兄弟元素必须有 `v-if` 或 `v-else-if`。

为 `v-if` 或者 `v-else-if` 添加“else 块”。


#### v-else-if

限制：前一兄弟元素必须有 `v-if` 或 `v-else-if`。

```
<div v-if="type === 'A'">
  A
</div>
<div v-else-if="type === 'B'">
  B
</div>
<div v-else>
  Not A/B/C
</div>
```

#### v-for

预期：`Array | Object | number | string`

基于源数据多次渲染元素或模板块。此指令之值，必须使用特定语法 `alias in expression` ，为当前遍历的元素提供别名

```
<div v-for="(item, index) in items"></div>
<div v-for="(val, key) in object"></div>
<div v-for="(val, key, index) in object"></div>
```

v-for **默认行为试着不改变整体，而是替换元素**。迫使其重新排序的元素，你需要提供一个 key 的特殊属性

```
<div v-for="item in items" :key="item.id">
  {{ item.text }}
</div>
```

当 Vue.js 用 `v-for` 正在更新已渲染过的元素列表时，它默认用“就地复用”策略。如果数据项的顺序被改变，**Vue 将不会移动 DOM 元素来匹配数据项的顺序， 而是简单复用此处每个元素**。

**也就是默认的行为是使用本来的元素，然后替换其中的内容，而不是重排序元素**


为了给 Vue 一个提示，以便它能跟踪每个节点的身份，从而重用和重新排序现有元素，你需要为每项提供一个唯一 `key` 属性

注意：建议尽可能在使用 `v-for` 时提供 `key`，除非遍历输出的 DOM 内容非常简单，或者是刻意依赖默认行为以获取性能上的提升。

#### v-on

缩写：`@`

预期：`Function | Inline Statement | Object`

参数：`event`

修饰符：

* .stop - 调用 event.stopPropagation()。
* .prevent - 调用 event.preventDefault()。
* .capture - 添加事件侦听器时使用 事件捕获 模式。
* .self - 只当事件是从侦听器绑定的元素本身触发时才触发回调，比如不是子元素。
* .{keyCode | keyAlias} - 只当事件是从特定键触发时才触发回调。
* .native - 监听组件根元素的原生事件。
* .once - 只触发一次回调。
* .left - (2.2.0) 只当点击鼠标左键时触发。
* .right - (2.2.0) 只当点击鼠标右键时触发。
* .middle - (2.2.0) 只当点击鼠标中键时触发。
* .passive - (2.3.0) 以 { passive: true } 模式添加侦听器

**绑定事件监听器**。事件类型由参数指定。表达式可以是一个方法的名字或一个内联语句，如果没有修饰符也可以省略。

```
<!-- 方法处理器 -->
<button v-on:click="doThis"></button>
<!-- 对象语法 (2.4.0+) -->
<button v-on="{ mousedown: doThis, mouseup: doThat }"></button>
<!-- 内联语句 -->
<button v-on:click="doThat('hello', $event)"></button>
<!-- 缩写 -->
<button @click="doThis"></button>
<!-- 停止冒泡 -->
<button @click.stop="doThis"></button>
<!-- 阻止默认行为 -->
<button @click.prevent="doThis"></button>
<!-- 阻止默认行为，没有表达式 -->
<form @submit.prevent></form>
<!--  串联修饰符 -->
<button @click.stop.prevent="doThis"></button>
<!-- 键修饰符，键别名 -->
<input @keyup.enter="onEnter">
<!-- 键修饰符，键代码 -->
<input @keyup.13="onEnter">
<!-- 点击回调只会触发一次 -->
<button v-on:click.once="doThis"></button>
```

在子组件上监听自定义事件 (当子组件触发“my-event”时将调用事件处理器)：

```
<my-component @my-event="handleThis"></my-component>
<!-- 内联语句 -->
<my-component @my-event="handleThis(123, $event)"></my-component>
<!-- 组件中的原生事件 -->
<my-component @click.native="onClick"></my-component>
```

##### 使用 v-on 绑定自定义事件

每个 Vue 实例都实现了事件接口，即：

* 使用 `$on(eventName)` 监听事件
* 使用 `$emit(eventName)` 触发事件

#### v-bind

缩写：`:`

预期：`any (with argument) | Object (without argument)`

参数：`attrOrProp (optional)`

修饰符：

* .prop - 被用于绑定 DOM 属性 (property)。(差别在哪里？)
* .camel - (2.1.0+) 将 kebab-case 特性名转换为 camelCase. (从 2.1.0 开始支持)
* .sync (2.3.0+) 语法糖，会扩展成一个更新父组件绑定值的 v-on 侦听器。

动态地绑定一个或多个特性，或一个组件 prop 到表达式

在绑定 `class` 或 `style` 特性时，支持其它类型的值，如数组或对象。可以通过下面的教程链接查看详情。

```
<!-- 绑定一个属性 -->
<img v-bind:src="imageSrc">
<!-- 缩写 -->
<img :src="imageSrc">
<!-- 内联字符串拼接 -->
<img :src="'/path/to/images/' + fileName">
<!-- class 绑定 -->
<div :class="{ red: isRed }"></div>
<div :class="[classA, classB]"></div>
<div :class="[classA, { classB: isB, classC: isC }]">
<!-- style 绑定 -->
<div :style="{ fontSize: size + 'px' }"></div>
<div :style="[styleObjectA, styleObjectB]"></div>
<!-- 绑定一个有属性的对象 -->
<div v-bind="{ id: someProp, 'other-attr': otherProp }"></div>
<!-- 通过 prop 修饰符绑定 DOM 属性 -->
<div v-bind:text-content.prop="text"></div>
<!-- prop 绑定。“prop”必须在 my-component 中声明。-->
<my-component :prop="someThing"></my-component>
<!-- 通过 $props 将父组件的 props 一起传给子组件 -->
<child-component v-bind="$props"></child-component>
<!-- XLink -->
<svg><a :xlink:special="foo"></a></svg>
```

#### v-model

预期：随表单控件类型不同而不同。

限制：

* `<input>`
* `<select>`
* `<textarea>`
* components

修饰符：

* .lazy - 取代 input 监听 change 事件
* .number - 输入字符串转为数字
* .trim - 输入首尾空格过滤

#### v-pre

跳过这个元素和它的子元素的编译过程。可以用来显示原始 Mustache 标签。跳过大量没有指令的节点会加快编译。

```
<span v-pre>{{ this will not be compiled }}</span>
```

#### v-cloak

这个指令保持在元素上直到关联实例结束编译。和 CSS 规则如 `[v-cloak] { display: none }` 一起用时，这个指令可以隐藏未编译的 Mustache 标签直到实例准备完毕。

#### v-once

只渲染元素和组件一次。随后的重新渲染，元素/组件及其所有的子节点将被视为静态内容并跳过。这可以用于优化更新性能。

### 使用 JavaScript 表达式

```
{{ number + 1 }}
{{ ok ? 'YES' : 'NO' }}
{{ message.split('').reverse().join('') }}
<div v-bind:id="'list-' + id"></div>
```

## 计算属性和观察者

对于任何复杂逻辑，你都应当使用**计算属性**

```
<div id="example">
  {{ message.split('').reverse().join('') }}
</div>
```

应该修改为：

```
<div id="example">
  <p>Original message: "{{ message }}"</p>
  <p>Computed reversed message: "{{ reversedMessage }}"</p>
</div>
```

```
var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // 计算属性的 getter
    reversedMessage: function () {
      // `this` 指向 vm 实例
      return this.message.split('').reverse().join('')
    }
  }
})
```

### 计算属性缓存 VS 方法

我们可以通过在表达式中调用方法来达到同样的效果：

```
<p>Reversed message: "{{ reversedMessage() }}"</p>
```

```
// 在组件中
methods: {
  reversedMessage: function () {
    return this.message.split('').reverse().join('')
  }
}
```

两种方式的最终结果确实是完全相同的。然而，**不同的是计算属性是基于它们的依赖进行缓存的。**计算属性只有在它的相关依赖发生改变时才会重新求值。

### 计算属性 VS 侦听属性

`watch`是观察一个特定的值，当该值变化时执行特定的函数。

### 侦听器

Vue 通过 `watch` 选项提供了一个更通用的方法，来响应数据的变化。当需要在数据变化时**执行异步或开销较大的操作**时，这个方式是最有用的。

## Class 与 Style 绑定

在将 v-bind 用于 `class` 和 `style` 时，Vue.js 做了专门的增强。表达式结果的类型除了字符串之外，还可以是对象或数组。

### 对象语法

```
<div v-bind:class="classObject"></div>
```

```
data: {
  isActive: true,
  error: null
},
computed: {
  classObject: function () {
    return {
      active: this.isActive && !this.error,
      'text-danger': this.error && this.error.type === 'fatal'
    }
  }
}
```

### 数组语法

```
<div v-bind:class="[{ active: isActive }, errorClass]"></div>
```

这样写将始终添加 `errorClass`，但是只有在 `isActive` 是 truthy 时才添加 `activeClass`。

### 用在组件上

当在一个自定义组件上使用 `class` 属性时，这些类将被添加到根元素上面。这个元素上已经存在的类不会被覆盖。

```
Vue.component('my-component', {
  template: '<p class="foo bar">Hi</p>'
})
```

然后在使用它的时候添加一些 class：

```
<my-component class="baz boo"></my-component>
```

HTML 将被渲染为:

```
<p class="foo bar baz boo">Hi</p>
```

## 条件渲染

```
<h1 v-if="ok">Yes</h1>
<h1 v-else>No</h1>
```

###用 key 管理可复用的元素

Vue 会尽可能高效地渲染元素，通常会复用已有元素而不是从头开始渲染。

这样也不总是符合实际需求，所以 Vue 为你提供了一种方式来表达“这两个元素是完全独立的，不要复用它们”。只需添加一个具有唯一值的 key 属性即可。

### v-show

另一个用于根据条件展示元素的选项是 v-show 指令。用法大致一样：

```
<h1 v-show="ok">Hello!</h1>
```

不同的是带有 `v-show` 的元素始终会被渲染并保留在 DOM 中。`v-show` 只是简单地切换元素的 CSS 属性 `display`。

v-if 是“真正”的条件渲染，因为它会确保在切换过程中条件块内的事件监听器和子组件适当地被销毁和重建。v-show 就简单得多——不管初始条件是什么，元素总是会被渲染，并且只是简单地基于 CSS 进行切换。**v-if 有更高的切换开销，而 v-show 有更高的初始渲染开销。因此，如果需要非常频繁地切换，则使用 v-show 较好；如果在运行时条件很少改变，则使用 v-if 较好。**

## 列表渲染

### 用 v-for 把一个数组对应为一组元素

在 v-for 块中，我们拥有对**父作用域属性**的完全访问权限。

```
<ul id="example-1">
  <li v-for="item in items">
    {{ item.message }}
  </li>
</ul>
```

```
var example1 = new Vue({
  el: '#example-1',
  data: {
    items: [
      { message: 'Foo' },
      { message: 'Bar' }
    ]
  }
})
```

也可以用 v-for 通过一个对象的属性来迭代。

```
在遍历对象时，是按 Object.keys() 的结果遍历，但是不能保证它的结果在不同的 JavaScript 引擎下是一致的。
```

### 数组更新检测

Vue 包含一组观察数组的变异方法，所以它们也将会触发视图更新。这些方法如下：

* push()
* pop()
* shift()
* unshift()
* splice()
* sort()
* reverse()

### 替换数组

也会触发视图更新：

```
example1.items = example1.items.filter(function (item) {
  return item.message.match(/Foo/)
})
```

### 注意事项

由于 JavaScript 的限制，Vue 不能检测以下变动的数组：

1. 当你利用索引直接设置一个项时，例如：`vm.items[indexOfItem] = newValue`
2. 当你修改数组的长度时，例如：`vm.items.length = newLength`


为了解决第一类问题，以下两种方式都可以实现和 `vm.items[indexOfItem] = newValue` 相同的效果，同时也将触发状态更新：

```
// Vue.set
Vue.set(example1.items, indexOfItem, newValue)

// Array.prototype.splice
example1.items.splice(indexOfItem, 1, newValue)
```

为了解决第二类问题，你可以使用 splice：

```
example1.items.splice(newLength)
```

有时你可能需要为已有对象赋予多个新属性，比如使用 `Object.assign()` 或 `_.extend()`。在这种情况下，你应该用两个对象的属性创建一个新的对象。所以，如果你想添加新的响应式属性，**不要**像这样：

```
Object.assign(this.userProfile, {
  age: 27,
  favoriteColor: 'Vue Green'
})
```

你应该这样做：

```
this.userProfile = Object.assign({}, this.userProfile, {
  age: 27,
  favoriteColor: 'Vue Green'
})
```

### 显示过滤/排序效果

们想要显示一个数组的过滤或排序副本，而不实际改变或重置原始数据。在这种情况下，可以创建返回过滤或排序数组的计算属性。

```
<li v-for="n in evenNumbers">{{ n }}</li>
```

```
data: {
  numbers: [ 1, 2, 3, 4, 5 ]
},
computed: {
  evenNumbers: function () {
    return this.numbers.filter(function (number) {
      return number % 2 === 0
    })
  }
}
```

或者

```
data: {
  numbers: [ 1, 2, 3, 4, 5 ]
},
methods: {
  even: function (numbers) {
    return numbers.filter(function (number) {
      return number % 2 === 0
    })
  }
}
```

### 一段取值的 v-for

```
<div>
  <span v-for="n in 10">{{ n }} </span>
</div>
```

### v-for 和 v-if

当它们处于同一节点，`v-for` 的优先级比 `v-if` 更高，这意味着 `v-if` 将分别重复运行于每个 `v-for` 循环中。

```
<li v-for="todo in todos" v-if="!todo.isComplete">
  {{ todo }}
</li>
```

### 一个组件的 v-for

```
<my-component v-for="item in items" :key="item.id"></my-component>
```

## 事件处理

### 事件监听

可以用 v-on 指令监听 DOM 事件来触发一些 JavaScript 代码。

除了直接绑定到一个方法，也可以用内联 JavaScript 语句：

```
<div id="example-3">
  <button v-on:click="say('hi')">Say hi</button>
  <button v-on:click="say('what')">Say what</button>
</div>
```

有时也需要在内联语句处理器中访问原生 DOM 事件。可以用特殊变量 `$event` 把它传入方法：

```
<button v-on:click="warn('Form cannot be submitted yet.', $event)">
  Submit
</button>
```

```
使用修饰符时，顺序很重要；相应的代码会以同样的顺序产生。因此，用 @click.prevent.self 会阻止所有的点击，而 @click.self.prevent 只会阻止元素上的点击。
```

在监听键盘事件时，我们经常需要监测常见的键值。Vue 允许为 v-on 在监听键盘事件时添加关键修饰符：

```
<!-- 只有在 keyCode 是 13 时调用 vm.submit() -->
<input v-on:keyup.13="submit">
```

## 表单输入绑定

可以用 `v-model` 指令在表单控件元素上创建双向数据绑定。它会根据控件类型自动选取正确的方法来更新元素。

```
v-model 会忽略所有表单元素的 value、checked、selected 特性的初始值。因为它会选择 Vue 实例数据来作为具体的值。你应该通过 JavaScript 在组件的 data 选项中声明初始值。
```

### 修饰符

#### .lazy

在默认情况下，v-model 在 input 事件中同步输入框的值与数据 (除了 上述 IME 部分)，但你可以添加一个修饰符 lazy ，从而转变为在 change 事件中同步：

```
<!-- 在 "change" 而不是 "input" 事件中更新 -->
<input v-model.lazy="msg" >
```

#### .number

如果想自动将用户的输入值转为 Number 类型 (如果原值的转换结果为 NaN 则返回原值)，可以添加一个修饰符 number 给 v-model 来处理输入值：

```
<input v-model.number="age" type="number">
```

这通常很有用，因为在 type="number" 时 HTML 中输入的值也总是会返回字符串类型。

#### .trim

如果要自动过滤用户输入的首尾空格，可以添加 trim 修饰符到 v-model 上过滤输入：

```
<input v-model.trim="msg">
```

## 组件

### 全局注册

要注册一个全局组件，可以使用 `Vue.component(tagName, options)`。

```
<div id="example">
  <my-component></my-component>
</div>
```

```
// 注册
Vue.component('my-component', {
  template: '<div>A custom component!</div>'
})
// 创建根实例
new Vue({
  el: '#example'
})
```

渲染为：

```
<div id="example">
  <div>A custom component!</div>
</div>
```

### 局部注册

你不必把每个组件都注册到全局。你可以通过某个 Vue 实例/组件的实例选项 `components` 注册仅在其作用域中可用的组件

```
var Child = {
  template: '<div>A custom component!</div>'
}
new Vue({
  // ...
  components: {
    // <my-component> 将只在父组件模板中可用
    'my-component': Child
  }
})
```

### DOM 模板解析注意事项

当使用 DOM 作为模板时 (例如，使用 `el` 选项来把 Vue 实例挂载到一个已有内容的元素上)，你会受到 HTML 本身的一些限制

在自定义组件中使用这些受限制的元素时会导致一些问题，例如：

```
<table>
  <my-row>...</my-row>
</table>
```

自定义组件 `<my-row>` 会被当作无效的内容，因此会导致错误的渲染结果。变通的方案是使用特殊的 `is` 特性：

```
<table>
  <tr is="my-row"></tr>
</table>
```

**需要注意的标签有：**`<ul>、<ol>、<table>、<select>、<option>`


应当注意，如果使用来自以下来源之一的字符串模板，则没有这些限制：

* `<script type="text/x-template">`
* JavaScript 内联模板字符串
* .vue 组件

因此，请尽可能使用字符串模板。

#### is 的介绍

用于动态组件且基于 DOM 内模板的限制来工作。

[官网上关于 is 的介绍](https://cn.vuejs.org/v2/api/#is)

 ### data 必须是函数
 
 ### 组件组合
 
 在 Vue 中，父子组件的关系可以总结为 **prop 向下传递，事件向上传递**。父组件通过 prop 给子组件下发数据，子组件通过事件给父组件发送消息。

### prop

子组件要显式地用 props 选项声明它预期的数据：

```
Vue.component('child', {
  // 声明 props
  props: ['message'],
  // 就像 data 一样，prop 也可以在模板中使用
  // 同样也可以在 vm 实例中通过 this.message 来使用
  template: '<span>{{ message }}</span>'
})
```

然后我们可以这样向它传入一个普通字符串：

```
<child message="hello!"></child>
```

#### 动态 prop

我们可以用 `v-bind` 来动态地将 prop 绑定到父组件的数据。

```
<div>
  <input v-model="parentMsg">
  <br>
  <child v-bind:my-message="parentMsg"></child>
</div>
```

你也可以使用 v-bind 的缩写语法：

```
<child :my-message="parentMsg"></child>
```

如果你想把一个对象的所有属性作为 `prop` 进行传递，可以使用不带任何参数的 `v-bind`

```
todo: {
  text: 'Learn Vue',
  isComplete: false
}

<todo-item v-bind="todo"></todo-item>
```

将等价于：

```
<todo-item
  v-bind:text="todo.text"
  v-bind:is-complete="todo.isComplete"
></todo-item>
```

#### 字面量语法 VS 动态语法

如果想传递一个真正的 JavaScript 数值，则需要使用 v-bind，从而让它的值被当作 JavaScript 表达式计算：

```
<!-- 传递了一个字符串 "1" -->
<comp some-prop="1"></comp>
<!-- 传递真正的数值 -->
<comp v-bind:some-prop="1"></comp>
```

#### 单项数据流

**不应该**在子组件内部改变 prop。如果你这么做了，Vue 会在控制台给出警告。

修改 prop 中数据，正确的应对方式是：

定义一个局部变量，并用 prop 的值初始化它：

```
props: ['initialCounter'],
data: function () {
  return { counter: this.initialCounter }
}
```

定义一个计算属性，处理 prop 的值并返回：

```
props: ['size'],
computed: {
  normalizedSize: function () {
    return this.size.trim().toLowerCase()
  }
}
```

**如果 prop 是一个对象或数组，在子组件内部改变它会影响父组件的状态。** 


#### prop 验证

要指定验证规则，需要用对象的形式来定义 prop，而不能用字符串数组：

```
Vue.component('example', {
  props: {
    // 基础类型检测 (`null` 指允许任何类型)
    propA: Number,
    // 可能是多种类型
    propB: [String, Number],
    // 必传且是字符串
    propC: {
      type: String,
      required: true
    },
    // 数值且有默认值
    propD: {
      type: Number,
      default: 100
    },
    // 数组/对象的默认值应当由一个工厂函数返回
    propE: {
      type: Object,
      default: function () {
        return { message: 'hello' }
      }
    },
    // 自定义验证函数
    propF: {
      validator: function (value) {
        return value > 10
      }
    }
  }
})
```

**注意 prop 会在组件实例创建之前进行校验，所以在 default 或 validator 函数里，诸如 data、computed 或 methods 等实例属性还无法使用。**

### 非 Prop 特性

组件的作者却并不总能预见到组件被使用的场景。所以，组件可以接收任意传入的特性，这些特性都会被添加到**组件的根元素**上。

#### 替换/合并现有的特性

我们对待 `class` 和 `style` 特性会更聪明一些，这两个特性的值都会做合并 (merge) 操作


```
<input type="date" class="form-control">

<bs-date-input
  data-3d-date-picker="true"
  class="date-picker-theme-dark"
></bs-date-input>
```

最终生成的值为：`form-control date-picker-theme-dark`

### 自定义事件

自定义事件：子组件怎么跟父组件通信

每个 Vue 实例都实现了事件接口，即：

* 使用 `$on(eventName)` 监听事件
* 使用 `$emit(eventName)` 触发事件

另外，父组件可以在使用子组件的地方直接用 `v-on` 来监听子组件触发的事件。

```
<div id="counter-event-example">
  <p>{{ total }}</p>
  <button-counter v-on:increment="incrementTotal"></button-counter>
  <button-counter v-on:increment="incrementTotal"></button-counter>
</div>

-------------

Vue.component('button-counter', {
  template: '<button v-on:click="incrementCounter">{{ counter }}</button>',
  data: function () {
    return {
      counter: 0
    }
  },
  methods: {
    incrementCounter: function () {
      this.counter += 1
      this.$emit('increment')
    }
  },
})
new Vue({
  el: '#counter-event-example',
  data: {
    total: 0
  },
  methods: {
    incrementTotal: function () {
      this.total += 1
    }
  }
})
```

也有其他的使用方法：

#### vm.$on(event, callback)

参数：

* `{string | Array<string>}` event (数组只在 2.2.0+ 中支持)
* `{Function} callback`

```
vm.$on('test', function (msg) {
  console.log(msg)
})
vm.$emit('test', 'hi')
// => "hi"
```

#### vm.$emit(event, [...args])

参数：

* `{string} event`
* `[...args]`

触发当前实例上的事件。附加参数都会传给监听器回调。

#### vm.$once(event, callback)

* `{string}` event 
* `{Function} callback`

监听一个自定义事件，但是只触发一次，在第一次触发之后移除监听器。

#### vm.$off([event, callback])

* `{string | Array<string>}` event (数组只在 2.2.0+ 中支持)
* `{Function} callback`

移除自定义事件监听器。

* 如果没有提供参数，则移除所有的事件监听器；
* 如果只提供了事件，则移除该事件所有的监听器；
* 如果同时提供了事件与回调，则只移除这个回调的监听器。

#### 给组件绑定原生事件

在某个组件的根元素上监听一个原生事件。可以使用 v-on 的修饰符 `.native`。例如：

```
<my-component v-on:click.native="doTheThing"></my-component>
```

#### .sync 修饰符

对一个 prop 进行“双向绑定”。

```
<comp :foo.sync="bar"></comp>
```

会被扩展为：

```
<comp :foo="bar" @update:foo="val => bar = val"></comp>
```

当子组件需要更新 foo 的值时，它需要显式地触发一个更新事件：

```
this.$emit('update:foo', newValue)
```

#### 使用自定义事件的表单输入组件

自定义事件可以用来创建自定义的表单输入组件，使用 v-model 来进行数据双向绑定。要牢记：

```
<input v-model="something">
```

这不过是以下示例的语法糖：

```
<input
  v-bind:value="something"
  v-on:input="something = $event.target.value">
```

所以要让组件的 v-model 生效，它应该：

1. 接受一个 `value` prop
2. 在有新的值时触发 `input` 事件并将新值作为参数

```
<currency-input v-model="price"></currency-input>

Vue.component('currency-input', {
  template: '\
    <span>\
      $\
      <input\
        ref="input"\
        v-bind:value="value"\
        v-on:input="updateValue($event.target.value)"\
      >\
    </span>\
  ',
  props: ['value'],
  methods: {
    // 不是直接更新值，而是使用此方法来对输入值进行格式化和位数限制
    updateValue: function (value) {
      var formattedValue = value
        // 删除两侧的空格符
        .trim()
        // 保留 2 位小数
        .slice(
          0,
          value.indexOf('.') === -1
            ? value.length
            : value.indexOf('.') + 3
        )
      // 如果值尚不合规，则手动覆盖为合规的值
      if (formattedValue !== value) {
        this.$refs.input.value = formattedValue
      }
      // 通过 input 事件带出数值
      this.$emit('input', Number(formattedValue))
    }
  }
})
```

#### 自定义组件的 `v-model`

默认情况下，一个组件的 v-model 会使用 value prop 和 input 事件。但是诸如单选框、复选框之类的输入类型可能把 value 用作了别的目的。`model` 选项可以避免这样的冲突

#### 非父子组件的通信

在简单的场景下，可以使用一个空的 Vue 实例作为事件总线：

```
var bus = new Vue()

// 触发组件 A 中的事件
bus.$emit('id-selected', 1)

// 在组件 B 创建的钩子中监听事件
bus.$on('id-selected', function (id) {
  // ...
})
```

在复杂的情况下，我们应该考虑使用专门的状态管理模式。

### 使用插槽分发内容

```
父组件模板的内容在父组件作用域内编译；子组件模板的内容在子组件作用域内编译。
```

#### 单个插槽

除非子组件模板包含至少一个 `<slot>` 插口，否则父组件的内容将会被丢弃。

最初在` <slot>` 标签中的任何内容都被视为**备用内容**。备用内容在子组件的作用域内编译，并且只有在宿主元素为空，且没有要插入的内容时才显示备用内容。

#### 具名插槽

`<slot>` 元素可以用一个特殊的特性 `name` 来进一步配置如何分发内容。多个插槽可以有不同的名字。具名插槽将匹配内容片段中有对应 slot 特性的元素。

### 动态组件

通过使用保留的 `<component>` 元素，动态地绑定到它的 `is` 特性，我们让多个组件可以使用同一个挂载点

```
var vm = new Vue({
  el: '#example',
  data: {
    currentView: 'home'
  },
  components: {
    home: { /* ... */ },
    posts: { /* ... */ },
    archive: { /* ... */ }
  }
})
<component v-bind:is="currentView">
  <!-- 组件在 vm.currentview 变化时改变！ -->
</component>
```

也可以直接绑定到组件对象上：

```
var Home = {
  template: '<p>Welcome home!</p>'
}
var vm = new Vue({
  el: '#example',
  data: {
    currentView: Home
  }
})
```

如果把切换出去的组件保留在内存中，可以保留它的状态或避免重新渲染。为此可以添加一个 `keep-alive` 指令参数

`<keep-alive>` 包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们。和 `<transition>` 相似，`<keep-alive>` 是一个抽象组件：它自身不会渲染一个 DOM 元素，也不会出现在父组件链中。

当组件在 `<keep-alive>` 内被切换，它的 `activated` 和 `deactivated` 这两个生命周期钩子函数将会被对应执行。

### 编写可复用的组件

Vue 组件的 API 来自三部分——prop、事件和插槽：

1. **Prop**：允许外部环境传递数据给组件；
2. **事件**：允许从组件内触发外部环境的副作用；
3. **插槽**：允许外部环境将额外的内容组合在组件中。

### 子组件引用

尽管有 prop 和事件，但是有时仍然需要在 JavaScript 中直接访问子组件。为此可以使用 `ref` 为子组件指定一个引用 ID。

### 异步组件

Vue.js 允许将组件定义为一个工厂函数，异步地解析组件的定义。

### 组件命名约定

 PascalCase 是最通用的声明约定而 kebab-case 是最通用的使用约定。

## 过渡 & 动画

Vue 在插入、更新或者移除 DOM 时，提供多种不同方式的应用过渡效果。
包括以下工具：

1. 在 CSS 过渡和动画中自动应用 class
2. 可以配合使用第三方 CSS 动画库，如 Animate.css
3. 在过渡钩子函数中使用 JavaScript 直接操作 DOM
4. 可以配合使用第三方 JavaScript 动画库，如 Velocity.js




## 参考

* [Vue.js](https://cn.vuejs.org/index.html)