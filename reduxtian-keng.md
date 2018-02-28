###前言
React应用中，组件间状态管理和维护的难度往往随着应用的复杂而指数上升，redux的核心设计思路“将所有状态保存在一个对象里面”大大简化状态的管理。当然是不是真的实践中间所有的react状态都放到顶层这就是另外一个话题了，本文将简单分析redux源码，方便我们更好的理解其思路。

###代码结构
![](/assets/WX20180228-163926.png)
redux的代码相对来说是比较简单的，我们从实际的应用代码开始进行一步步解析
```js
const store = createStore(
    rootReducer,
    initialState,
    compose(applyMiddleware(...middlewares))
);

```

####createStore
createStore主要接收三个参数
* reducer 函数类型，用来返回下一个state
```js
//redux在调用reducer时会注入当前state和dispatch函数中的action参数
reducer(state, action) {
    switch(action.type) {
      case A:
        reutrn newState
      default:
        return state
    }
}
```
* preloadedState 初始的state
* enhancer store的增强器，后面还会同applyMiddleware一起详细介绍

提供的返回值
* dispatch

```js
//用来改变state的唯一入口
//传入action，每个action必须包含type属性
function dispatch(action) {
    if (!isPlainObject(action)) {
      //throw error
    }

    if (typeof action.type === 'undefined') {
      //throw error
    }

    if (isDispatching) {
      //throw error
    }

    try {
      isDispatching = true
      //通过reducer函数获取最新的state
      currentState = currentReducer(currentState, action)
    } finally {
      isDispatching = false
    }
    //触发通过subscibe添加的监听
    const listeners = (currentListeners = nextListeners)
    for (let i = 0; i < listeners.length; i++) {
      const listener = listeners[i]
      listener()
    }

    return action
  }

```
* subscibe
```js
//接收一个listener，给store添加监听函数
function subscribe(listener) {
    if (typeof listener !== 'function') {
      //throw error
    }

    if (isDispatching) {
      //throw error
    }

    let isSubscribed = true

    ensureCanMutateNextListeners()
    nextListeners.push(listener)
    //返回解绑函数，用来删除已添加的监听
    return function unsubscribe() {
      if (!isSubscribed) {
        return
      }

      if (isDispatching) {
        //throw error
      }

      isSubscribed = false

      ensureCanMutateNextListeners()
      const index = nextListeners.indexOf(listener)
      nextListeners.splice(index, 1)
    }
  }
```
* getState 
很简单获取当前的state对象
* repalceReducer
```js
//替换当前的reducer函数，并通过dispatch初始化store的状态
function replaceReducer(nextReducer) {
    if (typeof nextReducer !== 'function') {
      //throw error
    }

    currentReducer = nextReducer
    dispatch({ type: ActionTypes.REPLACE })
  }
```
这个方法一般很少使用，官方给出的使用场景主要有三种：
1.程序需要进行代码分割
2.需要动态加载不同的reducer
3.需要实现一个实时的reloading机制

* [$$observable]: observable
并不是供开发的时候使用的，感兴趣的可以去源码test目录下查看用法

####combineReducers
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

  return function combination(state = {}, action) {
    let hasChanged = false
    const nextState = {}
    //遍历所有的reducer，将每个reducer的返回再组合成新的state树
    for (let i = 0; i < finalReducerKeys.length; i++) {
      const key = finalReducerKeys[i]
      const reducer = finalReducers[key]
      const previousStateForKey = state[key]
      const nextStateForKey = reducer(previousStateForKey, action)
      
      nextState[key] = nextStateForKey
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey
    }
    return hasChanged ? nextState : state
  }
}
```
combineReducers实现很简单，可以避免我们在大型应用里面写一个超级到的switch...case，如果你设计的state树层次比较深，可以嵌套combineReducers最终合成一个reducer

####compose
```js
export default function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}
```
如果清楚reduce的原理 这段代码理解起来应该并不困难，compose([a, b, c])(...args)相当于a(b(c(...args)))

####applyMiddleware
直接看applyMiddleware可能并不清楚这个是什么用，我们先来看下redux中间件的格式
```js
({dispatch, getState}) => (next) => (action) => {}
```
再结合applyMiddleware的实现
```js
export default function applyMiddleware(...middlewares) {
  return createStore => (...args) => {
    const store = createStore(...args)
    let dispatch = () => {
      //throw error
    }
    let chain = []

    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    }
    //循环调用中间件注入state和dispatch，每个中间价返回 (next)=> (action) => {}
    chain = middlewares.map(middleware => middleware(middlewareAPI))
    //通过compose组合成dispatch
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}
```
好了 到这里如果还清楚发生了什么，我们就通过具体的例子来说明， 先写两个简单的中间件
```js
let m1 = ({dispatch, getState}) => (next) => (action) => {
  console.log('m1');
  //next(action)
}
let m2 = ({dispatch, getState}) => (next) => (action) => {
  console.log('m2');
  //next(action)
}
let store = createStore(reducer,applyMiddleware(m1,m2))
store.dispatch({type: "type"})
```
如果不执行next(action), 这个结果输出应该只有m1，结合compose代码，m2相当于m1的next参数，最终通过applyMiddleware得到的函数执行实际效果如下
```js
function m() {
  console.log('m1');
  //next(action)等价于执行m2
  console.log('m2');
  //next(action)等价于执行最终的dispatch
}
```
中间价也正式Redux生态活跃的关键因素，你可以通过中间价加各种黑魔法

###END
redux还包括bindActionCreator，实现一个简单的actionCreator和dispatch联结，这里不再赘述
redux非常优秀，希望大家能从redux设计中获得一些心得体会