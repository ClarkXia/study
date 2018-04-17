#### What's new in React16
* Error Boundaries
* Fragments ／ Strings
* Portals
* ReactDOMServer
* DOM Attributes
* Fiber

#### What's new in React16.2.0
* Better support of Fragments

#### What's new in React16.3.0
* New Context API
* New life-cycle methods: getDerivedStateFromProps, UNSAFE(componentWillMount, componentWillReceiveProps, componentWillUpdate)
* StrictMode
* unstable_AsyncMode

#### 升级
React16.3内容的变化相对较大，特别是Context即将面临调整和新的组件生命周期函数将对整体架构和周边生态产生较大影响，因此还是选用相比React16版本变化较小的React16.2作为升级版本

##### React插件
React16将停止对React Addons的维护和支持，具体可以参照[discontinuing support for react addons](https://reactjs.org/blog/2017/04/07/react-v15.5.0.html#discontinuing-support-for-react-addons)

##### 注意的问题
* React16依赖于es6的Map和Set，引入bable-polyfill或抽离单独的pollyfill（注意需要先于react进行加载）
* React.PropTypes废弃
```js
import React, {PropTypes} from 'react';
//React16之后
import React from 'react';
import PropTypes from 'prop-types';
```
* ReactDOM.render() 和 React.unstable_renderSubtreeIntoContainer()
这两个方法在生命周期方法中执行是将会返回null，要解决这个问题，可以借助[protals](https://reactjs.org/docs/portals.html)或[refs](https://reactjs.org/docs/refs-and-the-dom.html)
* componentDidUpdate声明周期方法不再接受prevContext参数




