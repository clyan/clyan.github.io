## 前言

**redux触发基本流程：** `执行dispatch(action)  =    dispatch({type:'', payload：{}  })   ->  执行 reducer()  ->  修改state`。

也就是说我只要使用`dispatch`就可以正常执行`reducer`修改`state`了,  那为什么需要中间件？

**中间件的作用：** 就是在 源数据 到   目标数据  中间做各种处理，有利于程序的可拓展性，通常情况下，一个中间件就是一个函数，且一个中间件最好只做一件事情。

![](https://pic4.zhimg.com/v2-ad83775df4f3d6b516f3b6b2c87a640b_b.png)

在`redux`中，中间件的作用在于， 调用 `dispatch` 触发 `reducer`之前做一些其他操作，也就是说，它改变的是执行`dispatch`到 触发`reducer`的流程。

**原来流程是这样的：**

![](https://pic2.zhimg.com/v2-8c7c035a1ea3604c31f7e0a8adceb621_b.png)

**加入中间件就变成这样了：**

![](https://pic3.zhimg.com/v2-7ec7c52e43441190c10c41d1a354c5ba_b.png)

理解了`redux`中间件的作用与基本流程，就可以开始分析`redux`中间件源码啦

## 复习函数的执行顺序

以下输出顺序是什么？ 

```js
 function fn1() {
   console.log(1)
 }
 ​
 function fn2() {
   console.log(2)
 }
 fn1(fn2())
```

正确答案 的是`2 1` ，因为 `fn2()` 的返回结果作为`fn1`的参数。所以`fn2`需要先执行完毕

时刻记住此执行顺序，对以下分析很重要！！！

## 书写redux中间件

先看下redux的中间件怎么写的,下面分析会说到为什么这么写。

```js
 function ({ dispatch, getState }) {
   return next => action => {
     next(action)
   };
 }
 // 这里转换箭头函数并给它加上名字，方便观察，阅读
 function mid1({ dispatch, getState }) {
   return function fn1(next) {
       return function fn1Rf(action){
         next(action)
     };
   }
 }
```

**例子：** 
自定义`md1, md2, md3`三个中间件。

```js
 import {  createStore,  applyMiddleware } from 'redux';
 ​
 // 中间件1
 function mid1({ dispatch, getState }) {
   return function fn1(next) {
     return function fn1Rf(action) {
       return next(action)
     }
   }
 }
 // 中间件2
 function mid2({ dispatch, getState }) {
   return function fn2(next) {
     return function fn2Rf(action) {
        return next(action)
     }
   }
 }
 // 中间件3
 function mid3({ dispatch, getState }) {
   return function fn3(next) {
     return function fn3Rf(action) {
       return next(action)
     }
   }
 }
 ​
 function countReducer(state = 0, action) {
     switch (action.type) {
         case 'INCREMENT':
           return state + 1;
     }
 }
                                                             
 const  store =  createStore(countReducer, applyMiddleware(mid1， mid2, mid3));
 ​
 export default store
```

**createStore:**
使用createStore创建store ，并使用
 只关注走中间件的流程，也就是假如`preloadedState`与`enhancer`必定传了一个。

 如果第二个参数`preloadedState`为函数，则 `enhancer = preloadedState`，不然则需传递第三个参数，

 第二个参数与第三个参数不能同时为函数。

```js
 export default function createStore(
   reducer,
   preloadedState,
   enhancer
 ){
   if (
     (typeof preloadedState === 'function' && typeof enhancer === 'function') ||
     (typeof enhancer === 'function' && typeof arguments[3] === 'function')
   ) {
     throw new Error('........')
     )
   }
 ​
   if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
     enhancer = preloadedState
     preloadedState = undefined
   }
   if (typeof enhancer !== 'undefined') {
     if (typeof enhancer !== 'function') {
       throw new Error('.......')
     }
     return enhancer(createStore)(
       reducer,
       preloadedState
     ) 
       // ...enhancer 为undefined 或者不为function 时的后续处理逻辑, 不是本文关键
   }
```

**返回：** `enehancer(createStore)(educer,preloadedState) `

也就是执行 `applyMiddleware()`的返回值就是`enehancer`, 接着看下`applyMiddleware` 的实现，看看`enehancer`是什么

## applyMiddleware

```js
 import compose from './compose'
 export default function applyMiddleware(...middlewares){
   return (createStore) => (reducer, preloadedState) => {
     const store = createStore(reducer, preloadedState)
     let dispatch = () => {
       throw new Error(
         'Dispatching while constructing your middleware is not allowed. ' +
           'Other middleware would not be applied to this dispatch.'
       )
     }
     const middlewareAPI = {
       getState: store.getState,
       dispatch: (action, ...args) => dispatch(action, ...args)
     }
 ​
     const chain = middlewares.map(middleware => middleware(middlewareAPI))
     dispatch = compose(...chain)(store.dispatch)
 ​
     return {
       ...store,
       dispatch
     }
   }
 }
```
可以看到applyMiddleware返回值，确实符合enehancer执行方式 

 `return (createStore) => (reducer, preloadedState) => {}`
 
 //与

 `enehancer(createStore)(educer,preloadedState) `

**主要做了以下几件事情：**

1. 先使用`createStore`创建 `store`

2. 再执行 `middlewares.map`为 每个中间件提供`getState`和 `dispatch` 方法，该`dispathch`指的是 `throw new Error`的`dispathch`, 用作中间件中调用`dispathch`的错误处理（不要与`store.dispatch`混淆）。

3. 此时 `middlewares.map` 执行每个中间的第一层函数，例：`mid1`，每个`fn1`和`fn1Rf`函数中就可以使用了`getState`和`dispatch`方法了。

```js
 function mid1({ dispatch, getState }) {
   return function fn1(next) {
       return function fn1Rf(action){
          next(action)
       };
   }
 }
```

第一层函数执行完后, 将`middlewares`传入`compose` 中时，其实每个中间件就只有两层了。

```js
function fn1(next) {
   return function fn1Rf(action){
     next(action)
   };
 }
```

1. 接着执行`compose`组合所有中间件。

**compose源码如下：**
```js
 export default function compose(...funcs) {
   // 没传任何参数，则默认返回一个空的函数
   if (funcs.length === 0) {
     return (arg) => arg
   }
     // 只传一个那就直接返回这一个咯
   if (funcs.length === 1) {
     return funcs[0]
   }
     // 将所有函数组合，返回一个函数
   return funcs.reduce((a, b) => (...args) => a(b(...args)))
 }
```

**精简一下，并将箭头函数改掉，方便观察：**
```js
 export default function compose(...funcs) {
   return funcs.reduce(function(a, b) {
     return function (...args) {
       return a(b(...args))
     }
   })
 }
```

例子中`mid1, mid2, mid3` 被 `applyMiddleware`中执行过后只剩下两层，并这两层都可访问`dispatch`，与`getState`方法，如下：

```js
 function fn1(next) {
   return function fn1Rf(action) {
     return next(action)
   }
 }
 ​
 function fn2(next) {
   return function fn2Rf(action) {
     return next(action)
   }
 }
 ​
 function fn3(next) {
   return function fn3Rf(action) {
     return next(action)
   }
 }
 // 对应的就是applyMiddleware 中的
 const composeFn = compose(fn1, fn2, fn3);
```

步骤如下：

1. `a`为 `fn1`, `b`为`fn2`， 返回 `(...args) => fn1(fn2(...args))`，在这将其命名为A
2. `a`为 `(...args) => fn1(fn2(...args))`, b为 fn3, 返回`(...args) => ((...args) => fn1(fn2(...args)))(fn3(...args))` ， 在这将其命名为`B`

第1步返回结果 等价于

```js
 function A(...args) {
   return fn1(fn2(...args))
 }
```

第二步返回结果等价于

第二步的`a`为第一步的返回结果`A`，在第二步中会自执行`A`

```js
 function B(...args) {
   return (function A(...args) {
             return fn1(fn2(...args))
          })(fn3(...args))
 }
```

将`A`自执行，`fn3(...args)` 作为`A的参数`，传入`fn2()`中
**化简得到：**

```js
 function B(...args) {
    return fn1(fn2(fn3(...args)))
 }
 const composeFn = compose(fn1, fn2, fn3);
 // composeFn === B
 composeFn = function(...args) {
    return fn1(fn2(fn3(...args)))
 }
```

接下来继续分析执行`composeFn` 就得到了最终的我们需要的`dispatch`

// 模拟 `store.disptch`，避免与最终的到的`dispatch`混淆，命名为`storeDispatch`

 ```js
let storeDispatch = (action)=> {
  console.log('修改成功')
}
// 执行composeFn                                  
dispatch = composeFn(storeDispatch);
// 得到
dispatch = fn1(fn2(fn3(storeDispatch)))
 ```

将`fn3,fn2, fn1`执行后 整理得到:

```js
 dispatch = function fn1Rf(action) {
   return function fn2Rf(action) {
     return function fn3Rf(action) {
         // 这个 storeDispatch 是传入的 store.disptch
       return storeDispatch(action)
     }
   }
 }
```

中间件在`redux`内部执行顺序是跟传入的顺序反过来的。

但最终组合完毕，使用`dispatch`时执行顺序跟中间件传入的顺序是保持一致的。 

因为一个完整的中间件是由三层函数嵌套的！！！, `dispatch` 只执行最里面一层

到此为止，中间件核心原理就解释完毕了

## 编写中间件的注意点

正常情况下一个中间价的基本结构, 但要注意以下三点

```js
 function mid1({ dispatch, getState }) {
     //1.这里使用 dispatch() 会触发错误
   return function fn1(next) {
       //2. 这里使用 dispatch() 会触发错误
     return function fn1Rf(action) {
         //3. 这里使用 dispatch() 会导致死循环
       next(action)
     }
   }
 }
```

**原因：**

**第一个和第二个原因如下：**

`applyMiddleware` 中 `middlewareAPI` 中的`dispatch` 指向的是 带有 `throw new Error()` 的这个 `dispatch` 函数，所以在组合中间件时使用`dispatch` 就会抛出异常。

```js
 export default function applyMiddleware(...middlewares){
   return (createStore) => (reducer, preloadedState) => {
     let dispatch = () => {
       throw new Error(
         'Dispatching while constructing your middleware is not allowed. ' +
           'Other middleware would not be applied to this dispatch.'
       )
     }
     const middlewareAPI = {
       getState: store.getState,
       dispatch: (action, ...args) => dispatch(action, ...args)
     }
 ​
     const chain = middlewares.map(middleware => middleware(middlewareAPI))
     dispatch = compose(...chain)(store.dispatch)
   }
 }
```

**第三个原因：**
`dispatch = compose(...chain)(store.dispatch)`

这段代码最终组合中间件的结果 (上面已经分析过了)

**结果如下:**
```js
 // 暂时命名为 mark
 dispatch = function fn1Rf(action) {

     // 此时按照第三个错误原因在这执行 dispatch(), 这个dispatch指向的是mark, 所以会导致，无限重新执行fn1Rf
    // dispatch() , 死循环
   return function fn2Rf(action) {
     return function fn3Rf(action) {
         // 这个storeDispatch是 compose(...chain)(store.dispatch) 是传入的 store.dispatch
         // 避免混淆，使用storeDispatch命名， 源码中就是到dispatch(action)
       return storeDispatch(action)
     }
   }
 }
```

## 参考链接

[川哥--学习 redux 源码整体架构，深入理解 redux 及其中间件原理](https://www.lxchuan12.cn/redux/#_1-%E5%89%8D%E8%A8%80)