## 前言

**首先简单的说明一下redux中的几个名词的作用。**

​	**一个灯有 亮 或 不亮 两个状态( state)，提供了一个开关用于控制灯亮或不亮 (reducer)， 往下按是亮、往上按是不亮（由action.type定义，也可以上下反过来），接下来人就可以选择往上按还是往下按（dispatch） 控制灯的亮或不亮**

- state:  可能会发生改变的值  (状态，也就是变量， 不变的那叫常量)

- reducer ：用于修改state的状态

- action： 约定是一个包含type与paylaod的一个对象，

- dispatch:  传入action 触发reducer 并根据action.type 决定执行reducer中的哪个操作

**执行流程：**  如：`dispatch(action) `  =    `dispatch({type:'', payload：{}  }) `   ->  执行 reducer()  ->  修改state。

## 本文案例

```js
import {  createStore, combineReducers } from 'redux';

function countReducer(state = 0, action) {
    switch (action.type) {
        case 'INCREMENT':
          return state + 1;
        case 'DECREMENT':
          return state - 1;
        default:
          return state;
    }
}

function sumReducer(state = 1, action) {
    switch (action.type) {
        case 'INCREMENT':
          return state + 1;
        case 'DECREMENT':
          return state - 1;
        default:
          return state;
    }
}

const combinefn = combineReducers({ countReducer, sumReducer})
const  store =  createStore(combinefn);

export default store
```

**描述：**

- 有两个`reducer`, `countReducer` 的默认`state`为`0`， `sumReducer`的默认`state`为`1`
- 并且`countReducer` 的`action.type`与`sumReducer`的`action.type`都有`‘INCREMENT’`或`'DECREMENT'`
- 使用`combineReducers` 组合两个`reducer`并将返回的函数`combinefn`传入`createStore`中执行返回需要的`store`

**`reducer` 的 `state` 是在 参数上给的默认值**

## combineReducers

**源码如下：**

```js
export default function combineReducers(reducers) {
  const reducerKeys = Object.keys(reducers)
  const finalReducers = {}
  for (let i = 0; i < reducerKeys.length; i++) {
    const key = reducerKeys[i]
    if (typeof reducers[key] === 'function') {
      finalReducers[key] = reducers[key]
    }
  }
  const finalReducerKeys = Object.keys(finalReducers)
    return function combination(
        state= {},
        action
        ) {
        let hasChanged = false
        const nextState = {}
        for (let i = 0; i < finalReducerKeys.length; i++) {
            const key = finalReducerKeys[i]
            const reducer = finalReducers[key]
            const previousStateForKey = state[key]
            const nextStateForKey = reducer(previousStateForKey, action)
            if (typeof nextStateForKey === 'undefined') {
                const actionType = action && action.type
                throw new Error('more error')
            }
            nextState[key] = nextStateForKey
            hasChanged = hasChanged || nextStateForKey !== previousStateForKey
        }
        hasChanged =
            hasChanged || finalReducerKeys.length !== Object.keys(state).length
        return hasChanged ? nextState : state
    }
}
```

`reducers`参数: 在这个例子中此时为 `{ countReducer, sumReducer}`。

1. 首先排除`reducers`对象中值不是函数的项 , 并赋值给 `finalReducers `

2. 然后取出 `reducers` 所有的键保存至`finalReducerKeys` 

3. 最后返回`combination`函数，此函数作为`createStore `的参数

   

   **`combination`就是一个`reducer`，因为他就是一个用于修改`state`状态的方法,**  

   **为了避免跟初始时传入的`reducer`混淆，以下都使用`combination` 代表这个 `reducer`**


   **在该例中:** 此时

   ```
   finalReducers = {
       countReducer: countReducer,
       sumReducer: sumReducer
   }
   finalReducerKeys = ['countReducer', 'sumReducer'] 
   ```

**`finalReducers`与 `finalReducerKeys`  会在combination中用到 （下面会再次提及）**

`combination` 当`dispatch({type: ''})` 时触发，遍历所有的`reducer`, 执行所有`reducer`中逻辑满足 `type` 的 修改`state`的语句。比如：

`type`为` INCREMENT` 就执行`return state + 1;` 修改状态

```js
dispatch({type: 'INCREMENT'}) 
function sumReducer(state = 1, action) {
    switch (action.type) {
        case 'INCREMENT':
          return state + 1;
    }
}
```

## createStore

1. `reducer` *(Function)*:  使用了 `combineReducers`时，对应就是`combination` 函数，没使用对应的就是传入的单个`reducer` 函数

2. [`preloadedState`] *(any)*: 初始时的 `state`。

   ​	如果使用 [`combineReducers`](https://www.redux.org.cn/docs/api/combineReducers.html) 创建 `reducer`，`preloadedState`最终作为第一次执行`combination`时的参数传入。 默认不传时也就是`combination`的`state ={}`，所以它必须是一个普通对象，并且与combineReducers传入的对象 keys 保持同样的结构， 如果不保持，会抛出错误，`（期望参数与combineReducers传入的对象，keys保持一致）`

   ​	在同构应用中，你可以决定是否把服务端传来的 state 水合）（hydrate）后传给它，或者从之前保存的用户会话中恢复一个传给它。。

3. `enhancer` *(Function)*:  `Store enhancer` 是一个组合 `store creator` 的高阶函数，返回一个新的强化过的 `store creator`。一个简单的作用就是支持的中间件。

   具体作用参照  [createStore 的第三个参数还有这个作用](https://jishuin.proginn.com/p/763bfbd4f433) , [Redux源码解读拾遗，createStore的第三个参数](https://github.com/forthealllight/blog/issues/11)

**这里将 reducer  赋值给了 currentReducer， 也就是将combination 赋值给了currentReducer** 

```js
// 省略了若干代码
export default function createStore(reducer, preloadedState, enhancer) {
    // 省略参数校验和替换
    // 当前的 reducer 函数， 在这个案例 初次也就是`combination` 函数
    let currentReducer = reducer
    // 当前state
    let currentState = preloadedState
    // 当前的监听数组函数
    let currentListeners = []
    // 下一个监听数组函数
    let nextListeners = currentListeners
    // 是否正在dispatch中
    let isDispatching = false
    function ensureCanMutateNextListeners() {
        if (nextListeners === currentListeners) {
        nextListeners = currentListeners.slice()
        }
    }
    function getState() {
        return currentState
    }
    function subscribe(listener) {}
    function dispatch(action) {}
    function replaceReducer(nextReducer) {}
    function observable() {}
    dispatch({ type: ActionTypes.INIT })
    return {
        dispatch,
        subscribe,
        getState,
        replaceReducer,
        [$$observable]: observable
    }
}
```

也就是说最终 `store` 为一个对象并包括如下几个属性：

```js
{
    dispatch,
    subscribe,
    getState,
    replaceReducer,
    [$$observable]: observable
}
```

接下来看这几个属性对应的函数的作用

## createStore 中的一些函数

### dispatch 

**作用：** 通过`dispatch`更新数据

执行`createStore`时内部第一次执行  `dispatch({ type: ActionTypes.INIT })` 执行 `currentReducer` 初始化`state`, （`currentReducer` 对应的就是上面提到的`combineReducers`返回的函数`combination`）

**注意：** 除了执行 `currentReducer` 计算state, 还遍历执行了`listeners`也就是`currentListeners` ，`listeners`是`subscribe`订阅的函数，也就是当状态改变，通知订阅者更新。  （下面会继续提及`listeners`, 先记住）

**源码如下：**

**再次提醒`currentReducer` 就是 上面提及的`combination`函数** 

```js
//  currentState = createStore的第二个参数，而且不要是函数类型的，不然会被认为是`enhancer` 
function dispatch(action: A) {
    try {
      isDispatching = true
        			// currentReducer 就是 combination
      currentState = currentReducer(currentState, action)
    } finally {
      isDispatching = false
    }
	// 执行所有状态更新后需要执行的函数
    const listeners = (currentListeners = nextListeners)
    for (let i = 0; i < listeners.length; i++) {
      const listener = listeners[i]
      listener()
    }

    return action
  }
```

**再次贴上`combination`代码：**

```js
// 前面在combineReducers中提及了，此时这两个对象的值，
// finalReducers = {
//    countReducer: countReducer,
//    sumReducer: sumReducer
// }
// finalReducerKeys = ['countReducer', 'sumReducer'] 

return function combination(
    state= {},
    action
    ) {
    let hasChanged = false
    const nextState = {}
    for (let i = 0; i < finalReducerKeys.length; i++) {
        const key = finalReducerKeys[i]
        const reducer = finalReducers[key]
        const previousStateForKey = state[key]
        const nextStateForKey = reducer(previousStateForKey, action)
        if (typeof nextStateForKey === 'undefined') {
            const actionType = action && action.type
            throw new Error('more error')
        }
        nextState[key] = nextStateForKey
        hasChanged = hasChanged || nextStateForKey !== previousStateForKey
    }
    hasChanged =
        hasChanged || finalReducerKeys.length !== Object.keys(state).length
    return hasChanged ? nextState : state
}
```

每次执行`dispatch`时，都会执行该函数，并且传入`上一次state`与 `本次dispatch的action`，遍历`finalReducerKeys`， 循环执行所有的`reducer`。

**也就是说 不同的reducer中如果action.type一样，那么都会执行，可能会发生预期之外的事情** 

如：`countReducer` 与 `sumReducer`， 中`action` 都有 `INCREMENT`,那么他们的状态都会变，细思极恐，所以避免使用同样的`action.type`， 如下例子：

```js
dispatch({type: 'INCREMENT'})
function countReducer(state = 0, action) {
    switch (action.type) {
        case 'INCREMENT':
          return state + 1;
    }
}

function sumReducer(state = 1, action) {
    switch (action.type) {
        case 'INCREMENT':
          return state + 1;
    }
}
```

也就是说第一次内部执行`dispatch({ type: ActionTypes.INIT })` 后，整个状态树的值就是如下所示内容。

```js
currentState = {
	countReducer: 0,
	sumReducer: 1
}
```

`dispatch({type:'INCREMENT'})` 执行后

```js
 currentState = {
     countReducer: 1,
     sumReducer: 2
 }
```

### **getState**

作用：通过`store.getState()` 获取值

源码如下：返回`currentState`

```js
  function getState(){
    return currentState
  }
```

## 阶段总结

目前为止，就已经定义好状态树且可以修改状态树了，并且`dispatch`函数中会遍历`listeners` 执行。前面也提及到了`listeners`  中的函数 是由 `subscribe` 订阅而来，接下来就继续看下 `subscribe` 的实现

### subscribe

**react中使用方式如下：**

`store.subscribe(render)` 将`render`放入`subscribe`中, 也就是说，每当`dispatch` 更改状态，都会使整个`App`进行重新渲染，这也符合`react`的执行机制。

```js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import store from './redux/index';
const render = () => ReactDOM.render(
    <App  store={ store } />,
  document.getElementById('root')
);

render();
const unsubscribe = store.subscribe(render)
// unsubscribe()  执行这句就会取消订阅
```

**部分源码：**
就是将函数加入到数组中，并且返回一个`unsubscribe`，作用是可从数组中删除刚加入的函数， 在`dispatch`时就遍历执行`currentListeners`数组中的函数。

```js
  let currentListeners = []
  let nextListeners = currentListeners;
  
  function ensureCanMutateNextListeners() {
    if (nextListeners === currentListeners) {
      nextListeners = currentListeners.slice()
    }
  }

function subscribe(listener: () => void) {
    
    let isSubscribed = true
    ensureCanMutateNextListeners()
    nextListeners.push(listener)
    return function unsubscribe() {
      isSubscribed = false
      ensureCanMutateNextListeners()
      const index = nextListeners.indexOf(listener)
      nextListeners.splice(index, 1)
      currentListeners = null
    }
  }

```

### replaceReducer

由一般情况下会在初始化的时候使用`combineReducers `组合所有的`reducer` 初始化整个 `state` 树，

但使用路由时我们希望当页面加载后，再组合这个页面需要的 `reducer` 与`state`, 

这时候就需要用到`combineReducers`  将新的`reducer`与已存在的`reducer`组合到一起，

前面提及`combineReducers`  返回的是 `combination` 函数，

并且在`createStore ` 中将`combination`  赋值给了 `currentReducer` (忘记了可以在看下前面分析`createStore`的解释)， 并在`dispatch`中使用的就是 `currentReducer` 重新计算`state`,

`combineReducers`  合并新的`reducer`之后，依旧返回的是`combination` 函数

 所以现在我们就需要使用`replaceReduce` 把 旧的 `currentReducer` 替换掉，并执行一次`dispatch`, 重新计算整个`state`树。

到此`state`树中就加入了新页面的`reducer`和`state`了

```js
  function replaceReduce(nextReducer) {
    currentReducer = nextReducer
    dispatch({ type: ActionTypes.REPLACE } as A)
    return store
  }
```

**这里不提供示例， 详细示例使用请参考：** [Redux store 的动态注入reducer](https://www.pianshen.com/article/65891328050/)


## redux其他的一些函数

### bindactioncreators

此函数就是一个工具函数。具体功能 **推荐看这篇内容:**  [React+Redux之bindactioncreators的用法](https://www.jianshu.com/p/26356a9c5365)

### compose 与 applyMiddleware

这两个函数用于中间件，单独放redux中间件中描述

## 参考链接

[Redux store 的动态注入reducer](https://www.pianshen.com/article/65891328050/)

[redux官网文档](https://www.redux.org.cn/docs/api/createStore.html)

[川哥 - 学习 redux 源码整体架构，深入理解 redux 及其中间件原理](https://www.lxchuan12.cn/redux/#_1-%E5%89%8D%E8%A8%80)

[React+Redux之bindactioncreators的用法](https://www.jianshu.com/p/26356a9c5365)