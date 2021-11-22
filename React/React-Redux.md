
## 前言
学习`react-redux`之前需要有`redux`和`react`的基础。

接下来使用redux案例和react-redux案例对比两个案例 与 Provide与connect的核心实现看一下react-redux的作用

## 使用redux的案例

**以下代码：** [代码地址： redux-test-subscript-parent](https://codesandbox.io/s/redux-test-subscript-parent-e7f2b?file=/src/App.js:0-229) :  建议请打开并点击 页面的`+` 观察输出

**store.js：** 使用`countReducer`和`sumReducer`创建一个`store`.

```js
import { createStore, combineReducers } from "redux";
function countReducer(state = { count: 0 }, action) {
  switch (action.type) {
    case "countAdd":
      return { count: state.count + 1 };
    default:
      return state;
  }
}
function sumReducer(state = { sum: 1 }, action) {
  switch (action.type) {
    case "sumAdd":
      return { sum: state.sum + 1 };
    default:
      return state;
  }
}
const combinefn = combineReducers({ countReducer, sumReducer });
const store = createStore(combinefn);
export default store;

```

**index.js：** 将`store`传入`App组件`中，并订阅`store.subscribe(render)`， 使得`dispatch`更新数据后，能重新渲染页面。

```js
import React from "react";
import ReactDOM from "react-dom";
import App from "./App";
import store from "./store";
const render = () =>
  ReactDOM.render(<App store={store} />, document.getElementById("root"));

render();
store.subscribe(render);
```

**App.js：** 使用`A`与`B`两个组件，并给`A`和`B`传入`store`,因为`A`与`B`中需要获取`store`的数据和使用`store.dispatch`更新数据

```js
import A from "./A";
import B from "./B";
function App(props) {
  console.log("App执行");	// 查看App的执行情况
  const { store } = props;
  return (
    <div className="App">
      <A store={store} />
      <B store={store} />
    </div>
  );
}

export default App;

```

**A:** 使用`store`中 `countReducer.count`的数据，并点击`A+`更新 `countReducer.count`

```js
function A(props) {
  console.log("A组件执行");  // 查看A的执行情况
  const { store } = props;
  const { dispatch, getState } = store;
  const count = getState().countReducer.count;
  return (
    <>
      <p> A（count）: {count} </p>
      <button onClick={() => { dispatch({ type: "countAdd" }); }}  >
        +
      </button>
    </>
  );
}
export default A;
```

**B：** 使用`store`中`sumReducer.sum`的数据，并点击`B+`更新 `sumReducer.sum`

```js
function B(props) {
  console.log("B组件执行");  // 查看B的执行情况
  const { store } = props;
  const { dispatch, getState } = store;
  const sum = getState().sumReducer.sum;
  return (
    <>
      <p> B（sum）: {sum} </p>
      <button
        onClick={() => {
          dispatch({ type: "sumAdd" });
        }}
      >
        +
      </button>
    </>
  );
}
export default B;
```

**最后展现效果：**

![image-20210629001704952.png](https://i.loli.net/2021/11/21/PxXBhm5KcDl8vYq.png)



当我们点击`A+` 与`B+`按钮时，会输出以下内容。

![image-20210628225757861.png](https://i.loli.net/2021/11/21/jAROvcJiKYnSXVx.png)

**以上案例，存在两个问题**

1. 使用`store.subscribe(render)`订阅整个页面，导致不管更新哪个组件都重新渲染整个页面，正确的渲染应该是，`A`组件依赖了`count`数据，那么只有当`count`改变时再重新渲染`A`组件，B组件没有依赖`count`就不重新渲染`B`
2. 需要给每个子组件传递`store`, 组件嵌套过深时，会需要一直传递下去，导致非常复杂



**先来解决第一个问题，将修改案例中的`index.js` , 不使用`store.subscribe(render)`订阅整个页面。**

```js
import React from "react";
import ReactDOM from "react-dom";
import App from "./App";
import store from "./store";
const render = () =>
  ReactDOM.render(<App store={store} />, document.getElementById("root"));

render();
```

修改完成后，执行`dispatch`更新数据后，就无法自动重新渲染组件了，所以需要我们在子组件中使用`store.subscribe()`使得组件可以重新渲染，如下:

**A.js**  改成以下形式，也就是当组件依赖的数据发生变化时，使用`forceUpdate`使得该组件重新，更新该组件需要的数据时，其他组件就不会更新，也就解决了第一个问题！！！

```js
import { useReducer, useEffect } from "react";
function A(props) {
  console.log("A组件执行");
  const { store } = props;
  const { dispatch, getState } = store;
    // subscribe会利用到
  const count = getState().countReducer.count;
  // 模拟class 组件中的 forceUpdate
  // 实现方式参考react官网
  //https://zh-hans.reactjs.org/docs/hooks-faq.html#is-there-something-like-forceupdate
  const [ignored, forceUpdate] = useReducer((x) => x + 1, 0);
    // 每次重新渲染重新订阅
  useEffect(() => {
      // subscribe订阅的函数会在dispatch数据更新后执行
    let unsubscribe = store.subscribe(() => {
      // 如果本次count与上次count不一致，才重新更新组件， count为上一轮的count,这里利用了闭包
      if (getState().countReducer.count !== count) {
        forceUpdate();
      }
    });
      // 每次都卸载订阅
    return () => {
      unsubscribe();
    };
  }, [store, count, getState]);
  return (
    <>
      <p> {count} </p>
      <button
        onClick={() => {
          dispatch({ type: "countAdd" });
        }}
      >
        +
      </button>
    </>
  );
}
export default A;
```

**B.js 同理**： [上面改动后的源码地址](https://codesandbox.io/s/redux-test-subscript-child-bpm2d?file=/src/index.js)  建议打开源码查看并点击 页面的 `+` 观察输出

此处点击9次`A`组件进行测试，B组件并没有再次执行：

![image-20210629001058057.png](https://i.loli.net/2021/11/21/SGXtgwR4VLCIYBl.png)

第一个问题虽然解决了，但是这又导致，我需要把`forceUpdate` 和`store.subscribe`，在所有用到`store`中的数据的组件，都需要写一遍，造成了大量的冗余（`react-redux`中使用`connect`高阶组件自动解决了这个问题）



然后剩下第二个向子组件中传递数据的问题，在分析`react-redux`中的`Provider`时说明。我们继续往下看。

## 使用react-redux的案例

**以下代码：**[源码地址--建议打开直接查看](https://codesandbox.io/s/react-redux-test-km9d6?file=/src/react-redux-test.js)

**store.js**

**使用`countReduce` 和 `sumReducer`创建一个`store`与`redux案例`中保持一致**

```js
import { createStore, combineReducers } from "redux";

function countReducer(state = { count: 0 }, action) {
  switch (action.type) {
    case "countAdd":
      return { count: state.count + 1 };
    default:
      return state;
  }
}
function sumReducer(state = { sum: 1 }, action) {
  switch (action.type) {
    case "sumAdd":
      return { sum: state.sum + 1 };
    default:
      return state;
  }
}
const combinefn = combineReducers({ countReducer, sumReducer });
const store = createStore(combinefn);
export default store;

```

**index.js** ： 使用 传入store参数至Provider组件并包裹App组件

```js
import { StrictMode } from "react";
import ReactDOM from "react-dom";
import { Provider } from "react-redux";
import store from "./store";
import App from "./App";

const rootElement = document.getElementById("root");
ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  rootElement
);
```

**App.js**: 不再需要向子组件中传递`store`

```js
import "./styles.css";
import ReactReduxTest from "./react-redux-test";
function App(props) {
  console.log("App执行");
  return (
    <div className="App">
      <A />
      <B />
    </div>
  );
}
export default App;
```

**A.js:**   使用`connect`将`store`中的`countReducer.count`注入到组件的`props`中

```js
import { connect } from "react-redux";
function A(props) {
  console.log("A组件执行");
  const { count, countAdd } = props;
  return (
    <>
      <p> {count} </p>
      <button onClick={() => { countAdd();}}>
        A+
      </button>
    </>
  );
}
const mapStateToProps = (state, ownProps) => {
  const { count } = state.countReducer;
  return {
    count: count
  };
};

const mapDispatchToProps = (dispatch, ownProps) => {
  return {
    countAdd: () =>
      dispatch({
        type: "countAdd"
      })
  };
};
export default connect(mapStateToProps, mapDispatchToProps)(ReactReduxTest);

```

**B.js:**  使用`connect`将`store`中`sumReducer.sum`的注入到组件的`props`中, 代码与A基本保持一致。

[react-redux案例源码地址](https://codesandbox.io/s/react-redux-test-km9d6?file=/src/react-redux-test.js) -- 建议打开直接查看

点击页面`A+` 或 `B+` 按钮，比如点击7次`A+`按钮， 输出如下，

![image-20210628232911473.png](https://i.loli.net/2021/11/21/4y8qLYtbk79OWaf.png)

![image-20210628232926850.png](https://i.loli.net/2021/11/21/glpE1QIqOaVbNvF.png)

可以看到B也没有再次执行，与我上面修改过的`redux案例`一样的效果，但是不需要我手动在每个组件内部写`forceUpdate`和订阅组件更新的逻辑了。

相比`redux案例`中，有如下改变：通过 传入`store`参数至`Provider`组件并包裹`App`组件，子组件使用`connect`就能通过`props`获取数据了。接下来看看`Provide`和`connect`的核心逻辑。

## Provider

对于`Provider`和`connect`并没有贴真正的源码，我们只需要关注核心逻辑原理即可，感兴趣的可以自己再去查看完整的源码。只看带有注释的地方

```js
import React, { useMemo } from 'react'

import Subscription from '../utils/Subscription'
import { useIsomorphicLayoutEffect } from '../utils/useIsomorphicLayoutEffect'

// 这句来自于源码中Contextjs， 使用 React.createContext(null),创建一个创建一个上下文的容器(组件)，就是通过此Api,解决在子组件中获取 store
const ReactReduxContext = /*#__PURE__*/ React.createContext(null)

function Provider({ store, context, children }) {
    
  const contextValue = useMemo(() => {
      // new一个subscription实例，用于订阅组件
    const subscription = new Subscription(store)
    subscription.onStateChange = subscription.notifyNestedSubs
    return {
      store,
      subscription,
    }
  }, [store])
  const previousState = useMemo(() => store.getState(), [store])

  //发现没，不就是我在上面写的改版redux案例吗，建议参考我上面的案例
  useIsomorphicLayoutEffect(() => {
    const { subscription } = contextValue
    subscription.trySubscribe()
	// 如果上一次数据与本次数据不同，则重新更新依赖的次数据的组件。。
    if (previousState !== store.getState()) {
      subscription.notifyNestedSubs()
    }
      // 卸载所有的订阅
    return () => {
      subscription.tryUnsubscribe()
      subscription.onStateChange = null
    }
  }, [contextValue, previousState])
   
  const Context = context || ReactReduxContext
  return <Context.Provider value={contextValue}>{children}</Context.Provider>
}

export default Provider
```

可以看到： 解决`store`向下传递的问题是通过`react`的`Api`,  `React.createContext`，将`store`注入到`react`的上下文中，并且自动更新组件也是通过`redux`的`subscribe`原理实现的。

## connect

[react-redux 作者写的connect核心英文源码解析](https://gist.github.com/gaearon/1d19088790e70ac32ea636c025ba424e)

下面的`connect`实现，不是真正的实现！！！！，而是一种心智模型。
它跳过了从哪里得到“store”的问题  (答案: 通过 `<Provider>` 把它放在`React`上下文中，`Provider`中已经提及过了)
它会跳过任何性能优化(真正的`connect()`会确保我们不会发生不必要的渲染)，也就是`Provide`r中`useIsomorphicLayoutEffect`的实现。建议参考上面的`redux案例`。

```js
// connect() 是一个高阶函数 ，返回一个高阶组件，用于将redux中数据通过props注入到你的组件.
// 你可以传入store中的state数据和用于修改state的dispatch(action) 函数。
function connect(mapStateToProps, mapDispatchToProps) {
  // connect(mapStateToProps, mapDispatchToProps)(你的组件)
  // 返回一个高阶组件，此组件用于增强你原本的组件
  return function (WrappedComponent) {
    // It returns a component
    return class extends React.Component {
      render() {
        return (
          // connect(mapStateToProps, mapDispatchToProps)(你的组件) 传入的组件
          <WrappedComponent
            {/* 组件原本的props  */}
            {...this.props}
            {/* 添加store中的数据到原本组件的props中 */}
            {...mapStateToProps(store.getState(), this.props)}
      		{/* 添加用于修改state的dispatch(action) 函数到原本组件的props中  */}
            {...mapDispatchToProps(store.dispatch, this.props)}
          />
        )
      }
      componentDidMount() {
        // 利用redux的原生能力，订阅handleChange
        this.unsubscribe = store.subscribe(this.handleChange.bind(this))
      }
      
      componentWillUnmount() {
        // 组件卸载的时候取消订阅
        this.unsubscribe()
      }
    
      handleChange() {
        // 强制重新更新此组件
        this.forceUpdate()
      }
    }
  }
}

// connect()的目的是您不必考虑
// 订阅store或自己进行性能优化, and
// 相反，你可以指定如何根据Redux存储状态获取道具:
const mapStateToProps =  (state) => ({
    value: state.counter,
  })
const mapDispatchToProps = (dispatch) => ({
    onIncrement() {
      dispatch({ type: 'INCREMENT' })
    }
  })    
// mapStateToProps 与 mapDispatchToProps 这两个函数，
// 在connect内部执行，如{...mapStateToProps(store.getState(), this.props)}，
// mapStateToProps 执行后的返回值就是 { value: state.counter}，然后通过...解构传入组件中
// 在组件中就能直接获取了，如：class组件中 props.value。
// mapDispatchToProps 同理，不再阐述。
const ConnectedCounter = cnnect(
  mapStateToProps,
  mapDispatchToProps
)(Counter)
```

## 总结

**react-redux相比redux**

1. 使用`React.createContext`,解决`store`传递的问题，
2. 对更新前后的值进行对比，避免重复更新的问题，重新渲染组件依旧使用的是`redux`的`subscribe`订阅组件，`dispatch`时遍历所有订阅的组件，重新渲染。
3. 使用`connect`高阶组件，将组件需要`store`中的值，注入到组件的`props`中。

`react-redux`还提供了`hooks`的用法，相比`connect`高阶组件，使用时，对于代码更易于理解.

## 参考链接

[为什么要使用 React Redux？](https://react-redux.js.org/introduction/why-use-react-redux)

[ react-redux 作者写的connect核心英文源码解析](https://gist.github.com/gaearon/1d19088790e70ac32ea636c025ba424e)

