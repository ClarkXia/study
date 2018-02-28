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