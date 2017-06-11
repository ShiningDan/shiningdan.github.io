---
title: Redux入门
date: 2017-06-09 18:15:14
categories: coding
tags:
  - React
  - Redux
---

本文记录的是我在学习 React Redux 中参考的文章以及个人心得与总结

<!--more-->

当我们要使用一个架构的时候，我们首先要知道，为什么要使用这个架构，以及使用这个架构可以为我们带来什么样的好处。下面，针对 Redux 框架，我们什么时候需要使用呢？

简单说，如果你的UI层非常简单，没有很多互动，Redux 就是不必要的，用了反而增加复杂性。

* 用户的使用方式非常简单
* 用户之间没有协作
* 不需要与服务器大量交互，也没有使用 WebSocket
* 视图层（View）只从单一来源获取数据

上面这些情况，都不需要使用 Redux。

* 用户的使用方式复杂
* 不同身份的用户有不同的使用方式（比如普通用户和管理员）
* 多个用户之间可以协作
* 与服务器大量交互，或者使用了WebSocket
* View要从多个来源获取数据

上面这些情况才是 Redux 的适用场景：多交互、多数据源。

从组件角度看，如果你的应用有以下场景，可以考虑使用 Redux。

* 某个组件的状态，需要共享
* 某个状态需要在任何地方都可以拿到
* 一个组件需要改变全局状态
* 一个组件需要改变另一个组件的状态

从上面介绍的场景，我们可以看到，Redux 主要是用在复杂的数据流应用中，用来解决组件之间沟通的机制的管理能力。

那么，当不使用 Redux 的时候，组件之间的沟通是怎么样的呢？

## React 组件之间的沟通方式

### 组件之间的关系

#### 父子组件

ReactJS中数据的流动是单向的，父组件的数据可以通过设置子组件的props传递数据给子组件。如果想让子组件改变父组件的数据，可以在父组件中传一个callback(回调函数)给子组件，子组件内调用这个callback即可改变父组件的数据。

**也就是父组件设置子组件的 props 来把数据传递给子组件。子组件通过父组件传的回调函数来讲数据传给父组件。**

#### 兄弟组件

兄弟组件不能直接相互传送数据，此时可以将数据挂载在父组件中，由两个组件共享：如果组件需要数据渲染，则由父组件通过props传递给该组件；如果组件需要改变数据，则父组件传递一个改变数据的回调函数给该组件，并在对应事件中调用。

**也就是公共数据作为父组件的 state 中的数据进行管理，如果兄弟组件需要修改父组件中的公共 state，则通过父组件中提供的回调函数来进行修改。**

兄弟组件的沟通的解决方案就是找到两个组件共同的父组件，一层一层的调用上一层的回调，再一层一层地传递props。如果组件树嵌套太深，就会非常难以管理。

为了解决过深的组件之间的沟通方案，可以使用全局事件和 Context。

### 全局事件

可以使用事件来实现组件间的沟通：改变数据的组件发起一个事件，使用数据的组件监听这个事件，在事件处理函数中触发setState来改变视图或者做其他的操作。使用事件实现组件间沟通脱离了单向数据流机制，不用将数据或者回调函数一层一层地传给子组件。

```
var EventEmitter = {
    _events: {},
    dispatch: function (event, data) {
        if (!this._events[event]) return; // no one is listening to this event
        for (var i = 0; i < this._events[event].length; i++)
            this._events[event][i](data);
    },
    subscribe: function (event, callback) {
      if (!this._events[event]) this._events[event] = []; // new event
      this._events[event].push(callback);
    },
    unSubscribe: function(event){
    	if(this._events && this._events[event]) {
    		delete this._events[event];
    	}
    }
}
```

**这种实现方法就是数据的双向绑定。**

###  context（上下文）

使用上下文可以让子组件直接访问祖先的数据或函数，无需从祖先组件一层层地传递数据到子组件中。

Context 是 React 中的一种传递数据的方式，在React中，数据可以以流的形式自上而下的传递，每当你使用一个组件的时候，你可以看到组件的props属性会自上而下的传递。但是，在某些情况下，我们不想通过父组件的props属性一级一级的往下传递，我们希望在某一级子组件中，直接得到上N级父组件中props中的值，就可以使用 Context。

## Redux

Redux 的设计思想很简单，就两句话。

```
（1）Web 应用是一个状态机，视图与状态是一一对应的。
（2）所有的状态，保存在一个对象里面。
```

**由于只有一个对象保存所有的状态，所以在对状态进行管理的时候（特别是全局的状态），可以很容易找到状态修改的源头，这样在 Debug 的时候会带来很多的方便。当然，局部的状态修改，使用双向数据流进行处理会更方便。这些都是个人的选择。**

### 基本概念和 API

#### store

`Store` 就是保存数据的地方，你可以把它看成一个容器。整个应用只能有一个 `Store`。


Redux 提供 `createStore` 这个函数，用来生成 `Store`。

```
import { createStore } from 'redux';
const store = createStore(fn);
```

上面代码中，`createStore`函数接受另一个函数 `fn` 作为参数，返回新生成的 `Store` 对象。`fn` 这个函数是 reducer 函数，表示的是当前 state 如何变成新的 newState。

#### State

`Store` 对象包含所有数据。如果想得到某个时点的数据，就要对 `Store` 生成快照。这种时点的数据集合，就叫做 `State`。
当前时刻的 `State`，可以通过 `store.getState()` 拿到。

```
import { createStore } from 'redux';
const store = createStore(fn);

const state = store.getState();
```

#### Action 

Action 描述当前发生的事情。改变 State 的唯一办法，就是使用 Action。它会运送数据到 Store。

Action 是一个对象。其中的type属性是必须的，表示 Action 的名称。其他属性可以自由设置，社区有一个[规范](https://github.com/acdlite/flux-standard-action)可以参考。

```
const action = {
  type: 'ADD_TODO',
  payload: 'Learn Redux'
};
```

#### Action Creator

View 要发送多少种消息，就会有多少种 Action。如果都手写，会很麻烦。可以定义一个函数来生成 Action，这个函数就叫 Action Creator。

```
const ADD_TODO = '添加 TODO';

function addTodo(text) {
  return {
    type: ADD_TODO,
    text
  }
}

const action = addTodo('Learn Redux');
```

#### store.dispatch()

`store.dispatch()` 是 **View 发出 Action 的唯一方法**。

```
import { createStore } from 'redux';
const store = createStore(fn);

store.dispatch({
  type: 'ADD_TODO',
  payload: 'Learn Redux'
});
```

结合 Action Creator，这段代码可以改写如下。

```
store.dispatch(addTodo('Learn Redux'));
```

#### Reducer

Store 收到 Action 以后，必须给出一个新的 State，这样 View 才会发生变化。这种 State 的计算过程就叫做 Reducer。

```
整个应用的初始状态，可以作为 State 的默认值。下面是一个实际的例子。

const defaultState = 0;
const reducer = (state = defaultState, action) => {
  switch (action.type) {
    case 'ADD':
      return state + action.payload;
    default: 
      return state;
  }
};

const state = reducer(1, {
  type: 'ADD',
  payload: 2
});
```

实际应用中，Reducer 函数不用像上面这样手动调用，`store.dispatch` 方法会触发 Reducer 的自动执行。为此，Store 需要知道 Reducer 函数，做法就是在生成 Store 的时候，将 Reducer 传入 `createStore` 方法。

```
import { createStore } from 'redux';
const store = createStore(reducer);
```

为什么这个函数叫做 Reducer 呢？因为它可以作为数组的reduce方法的参数。请看下面的例子，一系列 Action 对象按照顺序作为一个数组。

```
const actions = [
  { type: 'ADD', payload: 0 },
  { type: 'ADD', payload: 1 },
  { type: 'ADD', payload: 2 }
];

const total = actions.reduce(reducer, 0); // 3
```

 ### 纯函数
 
Reducer 函数最重要的特征是，它是一个纯函数。也就是说，只要是同样的输入，必定得到同样的输出。
纯函数是函数式编程的概念，必须遵守以下一些约束。

```
不得改写参数
不能调用系统 I/O 的API
不能调用Date.now()或者Math.random()等不纯的方法，因为每次会得到不一样的结果
```

#### store.subscribe()

Store 允许使用 `store.subscribe` 方法设置监听函数，一旦 State 发生变化，就自动执行这个函数。

```
import { createStore } from 'redux';
const store = createStore(reducer);

store.subscribe(listener);
```

`store.subscribe` 方法返回一个函数，调用这个函数就可以解除监听。

```
let unsubscribe = store.subscribe(() =>
  console.log(store.getState())
);

unsubscribe();
```

### Store 的实现

 Store 提供了三个方法。
 
```
store.getState()
store.dispatch()
store.subscribe()
```

`createStore` 方法还可以接受第二个参数，表示 State 的最初状态。这通常是服务器给出的。

```
let store = createStore(todoApp, window.STATE_FROM_SERVER)
```

上面代码中，`window.STATE_FROM_SERVER` 就是整个应用的状态初始值。注意，如果提供了这个参数，它会覆盖 Reducer 函数的默认初始值。

```
const createStore = (reducer) => {
  let state;
  let listeners = [];

  const getState = () => state;

  const dispatch = (action) => {
    state = reducer(state, action);
    listeners.forEach(listener => listener());
  };

  const subscribe = (listener) => {
    listeners.push(listener);
    return () => {
      listeners = listeners.filter(l => l !== listener);
    }
  };

  dispatch({});

  return { getState, dispatch, subscribe };
};
```

### Reducer 的拆分

```
const chatReducer = (state = defaultState, action = {}) => {
  const { type, payload } = action;
  switch (type) {
    case ADD_CHAT:
      return Object.assign({}, state, {
        chatLog: state.chatLog.concat(payload)
      });
    case CHANGE_STATUS:
      return Object.assign({}, state, {
        statusMessage: payload
      });
    case CHANGE_USERNAME:
      return Object.assign({}, state, {
        userName: payload
      });
    default: return state;
  }
};
```

这三个属性之间没有联系，这提示我们可以把 Reducer 函数拆分。不同的函数负责处理不同属性，最终把它们合并成一个大的 Reducer 即可。

```
const chatReducer = (state = defaultState, action = {}) => {
  return {
    chatLog: chatLog(state.chatLog, action),
    statusMessage: statusMessage(state.statusMessage, action),
    userName: userName(state.userName, action)
  }
};
```

Redux 提供了一个 `combineReducers` 方法，用于 Reducer 的拆分。你只要定义各个子 Reducer 函数，然后用这个方法，将它们合成一个大的 Reducer。

```
import { combineReducers } from 'redux';

const chatReducer = combineReducers({
  chatLog,
  statusMessage,
  userName
})

export default todoApp;
```

下面是combineReducer的简单实现。

```
const combineReducers = reducers => {
  return (state = {}, action) => {
    return Object.keys(reducers).reduce(
      (nextState, key) => {
        nextState[key] = reducers[key](state[key], action);
        return nextState;
      },
      {} 
    );
  };
};
```

你可以把所有子 Reducer 放在一个文件里面，然后统一引入。

```
import { combineReducers } from 'redux'
import * as reducers from './reducers'

const reducer = combineReducers(reducers)
```

### 实例

```
const Counter = ({ value }) => (
  <h1>{value}</h1>
  <button onClick={onIncrement}>+</button>
  <button onClick={onDecrement}>-</button>
);

const reducer = (state = 0, action) => {
  switch (action.type) {
    case 'INCREMENT': return state + 1;
    case 'DECREMENT': return state - 1;
    default: return state;
  }
};

const store = createStore(reducer);

const render = () => {
  ReactDOM.render(
    <Counter
      value={store.getState()}
      onIncrement={() => store.dispatch({type: 'INCREMENT'})}
      onDecrement={() => store.dispatch({type: 'DECREMENT'})}
    />,
    document.getElementById('root')
  );
};

render();
store.subscribe(render);
```

### 中间件

如果要添加功能，你会在哪个环节添加？

```
（1）Reducer：纯函数，只承担计算 State 的功能，不合适承担其他功能，也承担不了，因为理论上，纯函数不能进行读写操作。
（2）View：与 State 一一对应，可以看作 State 的视觉层，也不合适承担其他功能。
（3）Action：存放数据的对象，即消息的载体，只能被别人操作，自己不能进行任何操作。
```

只有发送 Action 的这个步骤，即 `store.dispatch()` 方法，可以添加功能。举例来说，要添加日志功能，把 Action 和 State 打印出来，可以对 `store.dispatch` 进行如下改造。

```
let next = store.dispatch;
store.dispatch = function dispatchAndLog(action) {
  console.log('dispatching', action);
  next(action);
  console.log('next state', store.getState());
}
```

#### 中间件的用法

（1）`createStore` 方法可以接受整个应用的初始状态作为参数，那样的话`，applyMiddleware` 就是第三个参数了。

```
const store = createStore(
  reducer,
  initial_state,
  applyMiddleware(logger)
);
```

（2）中间件的次序有讲究。

```
const store = createStore(
  reducer,
  applyMiddleware(thunk, promise, logger)
);
```

上面代码中，`applyMiddleware` 方法的三个参数，就是三个中间件。有的中间件有次序要求，使用前要查一下文档。比如，`logger` 就一定要放在最后，否则输出结果会不正确。

#### applyMiddlewares()

它是 Redux 的原生方法，作用是将所有中间件组成一个数组，依次执行。下面是它的源码。

```
export default function applyMiddleware(...middlewares) {
  return (createStore) => (reducer, preloadedState, enhancer) => {
    var store = createStore(reducer, preloadedState, enhancer);
    var dispatch = store.dispatch;
    var chain = [];

    var middlewareAPI = {
      getState: store.getState,
      dispatch: (action) => dispatch(action)
    };
    chain = middlewares.map(middleware => middleware(middlewareAPI));
    dispatch = compose(...chain)(store.dispatch);

    return {...store, dispatch}
  }
}
```

常用的中间件有：

* redux-thunk 中间件：如果需要 `store.dispatch` 方法接收的参数可以是一个函数，比如异步执行的函数，就可以使用 redux-thunk
* redux-promise 中间件：如果需要 `store.dispatch` 方法接收的参数可以是一个 Promise 对象，可以使用 redux-promise

## React-Redux 的用法

React-Redux 将所有组件分成两大类：UI 组件（presentational component）和容器组件（container component）。

UI 组件负责 UI 的呈现，容器组件负责管理数据和逻辑。

### UI 组件

UI 组件有以下几个特征。

```
只负责 UI 的呈现，不带有任何业务逻辑
没有状态（即不使用this.state这个变量）
所有数据都由参数（this.props）提供
不使用任何 Redux 的 API
```

下面就是一个 UI 组件的例子。

```
const Title =
  value => <h1>{value}</h1>;
```

UI 组件又称为"纯组件"，即它纯函数一样，纯粹由参数决定它的值。

### 容器组件

容器组件的特征恰恰相反。

```
负责管理数据和业务逻辑，不负责 UI 的呈现
带有内部状态
使用 Redux 的 API
```











## 参考

* [ReactJS组件间沟通的一些方法](http://www.alloyteam.com/2016/01/some-methods-of-reactjs-communication-between-components/)
* [Redux 入门教程](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_one_basic_usages.html)
* [Redux 中文文档](http://cn.redux.js.org/index.html)
* [react+redux渲染性能优化原理](http://foio.github.io/react-redux-performance-boost/)
* [数据流管理架构之 Redux 介绍](http://www.alloyteam.com/2015/09/react-redux/)
* [听说你需要这样了解 Redux](https://github.com/rccoder/blog/issues/18)


