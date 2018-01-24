### React基础
#### A Simple Component

```js
class HelloWorld extends Component {
    render(){
        return <div>hello world</div>
    }
}
//react-dom render
render(<HelloWorld />, document.getElementById('app'))
```

#### JSX

约定使用首字母大小写来区分本地组件和HTML标签

装换createElement

```js
//JSX
var root = <ul className="my-list">
             <li>Text Content</li>
           </ul>;
React.render(root, document.body);

//Javascript
var child = React.createElement('li', null, 'Text Content');
var root = React.createElement('ul', { className: 'my-list' }, child);
React.render(root, document.body);
```

#### JS表达式

```js
//属性比阿达式
<div className={isDisplay ? "show" : "hide"}></div>

//子节点表达式
<div>
    {someFlag ? <span>something</span> : null}
</div>
```

如果三元操作表达式不够用，可以通过if语句来决定渲染哪个组件
```js
class Sample extends React.Component {
    _decideWitchToRender(){
        if (this.props.index > 0 && this.props.someCondition){
            return <div>condition1</div>
        } else if (this.props.otherCondition) {
            return <div>condition2</div>
        }
        return <div>condition3</div>
    }
    render() {
        return <div>{ this.() }</div>
    }
}
```

#### 属性扩散
spread operator(...)
```js
var props = {}
props.a = x
props.b = y
var component = <Component {...props} />
```

#### HTML实体处理
```js
//Unicode字符
<div>{'First · Second'}</div>
//Unicode编号
<div>{'First \u00b7 Second'}</div>
<div>{'First ' + String.fromCharCode(183) + ' Second'}</div>
//数组里混合使用字符串
<div>{['First ', <span>&middot;</span>, ' Second']}</div>
//dangerouslySetInnerHTML
<div dangerouslySetInnerHTML={{__html: 'First &middot; Second'}} />
```

#### 自定义HTML属性
如果往原生 HTML 元素里传入 HTML 规范里不存在的属性，React 不会显示它们，data、aria（视听无障碍）前缀除外
特殊情况：自定义元素支持任意属性
```js
<x-my-component custom-attribute="foo" />
```

#### JSX与HTML差异
class -> className, for -> htmlFor, style由css属性构成的JS对象
```js
...
render(){
    const imgStyle = {
        transform: "rotate(0)",
        WebkitTransform: "rotate(0)"
    }
    return <div className="rotate" style={imgStyle} ></div>
}
...
```

#### React Component

事件系统（合成事件和原生事件）
合成事件会以事件委托（event delegation）的方式绑定到组件最上层，并且在组件卸载（unmount）的时候自动销毁绑定的事件
React合成事件最终是通过委托到document这个DOM节点进行实现，其他节点没有绑定事件*
React合成事件有自己的队列方式，可以从触发事件的组建向父组件回溯，可以通过e.stopPropagation来停止合成事件的传播，但无法阻止原生事件，原生事件可以阻止合成事件
React会管理合成事件的对象创建和销毁

#### DOM操作
findDOMNode()
```js
import { findDOMNode } from 'react-dom'
...
componentDidMound() {
  const el = findDOMNode(this)
}
...
```

Refs HTML元素，获取到DOM元素  自定义组件，获取到组件实例
无状态组件无法添加ref
```js
...
componentDidMound() {
    const el = this.refs.el
}
render() {
    return <div ref="el"></div>
}
...
```

#### 组合组件
```js
const ProfilePic = (props) => {
    return (
        <img src={'http://graph.facebook.com/' + props.username + '/picture'} />
    )
}
const ProfileLink = (props) => {
    return (
        <a href={'http://www.facebook.com/' + props.username}>
            {props.username}
        </a>
    )
}
const Avatar = (props) => {
    return (
        <div>
            <ProfilePic username={props.username} />
            <ProfileLink username={props.username} />
        </div>
    )
}
```

父级能通过专门的this.props.children来读取子级
```js
const Parent = (props) => {
    return <div>
        <p>something</p>
        {props.children}
    </div>
}
React.render(<Parent><Avatar username="clark" /></Parent>, document.body)
```
>props.children
>通常是一个组件对象的数组，当只有一个子元素的时候prop.children将是这个唯一子元素

##### 组件生命周期

实例化

 - getDefaultProps 只调用一次，所有组件共享
 - getInitialState 每个实例调用有且只有一次
 - componentWillMount
 - render 通过this.props/this.state访问数据、可以返回null/false/React组件、只能出现一个顶级组件、不能改变组件的状态、 不要去修改DOM的输出
 - componentDidMount 真实DOM已被渲染，这时候可以通过ref属性（this.refs.[refName]）获取真实DOM节点进行操作，如一些事件绑定

存在期

 - componentWillReceiveProps 组件的props通过父组件来更改，可以在这里更新state以触发render来重新渲染组件
 - shouldComponentUpdate 判断在props/state发生改变后需不需要重新渲染，优化性能的重要途径
 - componentWillUpdate 注意不要在此方法中更新props/state
 - render
 - componentDidUpdate 类似于componentDidMount

销毁期

 - componentWillUnmount 如在componentDidMount中添加相应的方法与监听，需在此时销毁

 #### Mixins

ES6写法中已经不支持，因为使用mixin场景可以用组合组件方式实现
#### 高阶组件
```js
export const ShouldUpdate = (ComposedComponent) => class extends React.Component{
    constructor(props) {
        super(props)
    }
    shouldComponentUpdate(nextProps) {
        ...
    }
    render() {
        return <ComposedComponent {...this.props}/>
    }
}

//usage
class MyComponent = class extends Component {
    ...
}

export default ShouldUpdate(MyComponent)
```
高阶组件的思路是函数式的 每一个扩展就是一个函数
```js
const newComponent = A(B(C(MyComponent)))
```
实现方式包括：
属性代理（Props Proxy）和反向继承（Inheritance Inversion）
```js
//属性代理
const ppHOC = ComponsedComponent => class extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            newState: ''
        };
    }

    handleSomething = (e) => {

    };

    render() {
        const newProps = Object.assign({}, this.props, this.state);
        return <ComponsedComponent ref="compnent" {...this.props} onSomething={this.handleSomething}/>;
    }
}

//反向继承
function iiHOC(WrappedComponent) {
    return class Enhancer extends WrappedComponent {
        render() {
            return supper.render();
        }
    }
}

```
注：反向继承可以做到渲染劫持，通常不建议操作state特别是新增

#### 虚拟DOM Diff
这里有两个假设用来降低Diff算法的复杂度O(n^3) -> O(n)
1.两个相同的组件产生类似的DOM结构，不同组件产生不同的DOM结构
2.对于同一层次的组件子节点，它们可以通过唯一的id进行区分
React的比较仅仅是按照树的层级分解
React仅仅对同一层的节点尝试匹配，因为实际上，Web中不太可能把一个Component在不同层中移动
不同节点类型比较
```js
//节点类型不同
renderA:<div />
renderB:<span />
=>[removeNode <div />], [insertNode <span />]
//对于React组件的比较同样适用,React认为不需要话太多时间去匹配两个不大可能有相似之处的component
renderA:<Header />
renderB:<Content />
=>[removeNode <Header />], [insertNode <Content />]
```
相同类型节点比较
```js
renderA: <div id="before" />
renderB: <div id="after" />
=> [replaceAttribute id "after"]
renderA: <div style={{color: 'red'}} />
renderB: <div style={{fontWeight: 'bold'}} />
=> [removeStyle color], [addStyle font-weight 'bold']
```
tips：实现自己的组件的时候，保持稳定的DOM结构会有助于性能的提升，例如我们可以通过css隐藏来控制节点显示，而不是添加／删除

列表节点的比较
当React校正带有key的子级时，它会被确保它们被重新排序
```js
import React from 'react'
import { render } from 'react-dom'
function createComponent(name) {
    class TestNode extends React.Component{
        constructor(props) {
            super(props)
            console.log(name + ' is init')
        }
        componentDidMount() {
            console.log(name + ' did mount')
        }

        componentWillUnmount() {
            console.log(name + ' will unmount')
        }

        componentDidUpdate() {
            console.log(name + ' is updated')
        }

        render() {
          return (
                <div className={'node ' + name} data-name={name}>
                    {this.props.children}
                </div>
            )
        }
    }

    return TestNode
}

const Root = createComponent('R')
const A = createComponent('A')
const B = createComponent('B')
const C = createComponent('C')
const D = createComponent('D')

class Wrapper extends React.Component{
    test1() {
        return (<Root>
                    <A>
                        <B />
                        <C />
                    </A>
                </Root>)
    }
    test2() {
        return (<Root>
                    <A>
                        <C />
                        <B />
                    </A>
                </Root>)
    }
    test3() {
        return (<Root>
                    <A>
                        <B key="B"/>
                        <C key="C"/>
                    </A>
                </Root>)
    }
    test4() {
        return (<Root>
                    <A>
                        <C key="C"/>
                        <B key="B"/>
                    </A>
                </Root>)
    }

    render() {
        if (this[this.props.testType]) {
            return this[this.props.testType]()
        } else {
            return <Root />
        }
    }
}
window.renderTest = function(testType){
    render(<Wrapper testType={testType} />, document.getElementById('app'))
}

```

避免使用state
 - componentDidMount、componentDidUpdate、render
 - computed data, react components, duplicated data from props

#### React与AJAX

componentDidMount中发起ajax请求，拿到数据后通过setState方法更新UI
如果异步请求请注意在componentWillUnmount中的abort请求

组件之间相互调用

- 父子组件 React数据流是单向的，父组件数据可以通过设置子组件props进行传递，如果想让子组件改变父组件数据，父组件传递一个回调函数给子组件即可（函数传递注意事项this.fn.bind(this)和箭头函数均会返回新的函数）
- 兄弟组件 将数据挂在在父组件中，由多个子组件共享数据，组件层次太深的问题 -> 全局事件/Context(getChildContext&childContextTypes)

```js
class Parent extends React.Component {
    getChildContext() {
        return { value: 'parent' };
    }

    render() {
        return <div>
                {this.props.children}
            </div>
    }
}
Parent.childContextTypes = {
    value: React.PropTypes.string
}

class Children extends React.Component {
  // 如果不需要在构造函数中使用可以不写，没有影响
    constructor(props, context) {
        super(props, context)
        console.log(context)
    }
    render() {
        return <div>{'context is: ' + this.context.value}</div>
    }
}
//如果要context的内容必须校验contextTypes
Children.contextTypes = {
    value: React.PropTypes.string
}
```

#### 组件化开发的思考

- 组件尽可能无状态化（stateless）
- 细粒度的把握，提高复用性
- 配合高阶组件实现复杂逻辑

React API
unstable_renderSubtreeIntoContainer

### Redux
- 整个应用只有一个store（单一数据源）
- State只能通过触发Action来更改
- State每次更改总是返回一个新的State，即Reducer

#### Actions

一个包含{type, payload}的对象,
type是一个常量标示的动作类型,
payload是动作携带的数据,
一般我们通过创建函数的方式来生产action，即Action Creator
```js
{
    type: 'ADD_ITEM',
    name: 'item1'
}
function addItem(id, name) {
    return {
        type: 'ADD_ITEM',
        name,
        id
    }
}
```
#### Reducer

Reducer用来处理Action触发的对状态树的更改
(oldState, action) => newState
Redux中只有一个Store，对应一个State状态，所以如果把处理都放到一个reducer中，显示会让这个函数显得臃肿和难以维护
将reducer拆分很小的reducer，每个reducer中关注state树上的特定字段，最后将reducer合并(combineReducers)成root reducer
```js
function items(state = [], action) {
    switch (action.type) {
        case 'ADD_ITEM'
            return [...state, {
                action.name,
                action.id
            }]
        default:
            return state
    }
}
function selectItem(state = '', action) {
    switch (action.type) {
        case 'SELECT_ITEM'
            return action.id
        default:
            return state
    }
}
var rootReducer = combineReducers({
    items,
    selectItem
})
```
#### Store

- 提供State状态树
- getState()方法获取State
- dispatch()方法发送action更改State
- subscribe()方法注册回调函数监听State的更改

根据已有的reducer创建store非常容易
```js
import { createStore } from 'redux'
let store = createStore(rootReducer)
```
#### 数据流
store.dispatch(action) -> reducer(state, action) -> store.getState()

- 1.调用store.dispatch(action)
- 2.redux store调用传入的reducer函数，根reducer应该把多个reducer输出合并成一个单一的state树
- 3.redux store保存了根reducer返回的完整的state树

### react-redux

- Provider：
    容器组件，用来接收Store，让Store对子组件可用
- connect：
    提供参数如下：
    * mapStateToProps 返回state中挑出部分值合并到props
    * mapDispatchToProps 返回actionCreators合并到props
    * mergeProps 自定义需要合并到props的值
    * options pure、withRef

Connect本身是一个组件，通过监听Provider提供的store变化来调用this.setState操作，这里特别需要注意传入的mapStateToProps必需只是你需要的数据，不然作为全局公用的store每次都进行对比显然不高效也不合理
通过connect()方法包装好的组件可以得到dispath方法作为组件的props，以及全局state中所需的内容

#### 四个要点

- Redux提供唯一store
- Provider组件包含住最顶层组件，将store作为props传入
- connect方法将store中数据以及actions通过props传递到业务子组件
- 子组件调用action，dispatch到reducer返回新的state，store同步更新，并通知组件进行更新

#### 展示组件

- 关注UI
- 不依赖action来更新组件
- 通过props接收数据和回调函数改变数据
- 无状态组件，一般情况下没有state

#### 容器组件

- 关注运作方式
- 调用action
- 为展示组件提供数据和方法
- 作为数据源，展示组件UI变化控制器

UI与逻辑的分离，利于复用，易于重构

#### redux中间件
```js
//redux中间件格式
({dispatch, store}) => (next) => (action) => {}
import {createStore, applyMiddleware, combineReducers} from "redux"

let reducer = (store={},action)=>{
    return store
}

let logger1 = ({dispatch, getState}) => (next) => (action) => {
    console.log("第一个logger开始");
    next(action);
}

let logger2 = ({dispatch, getState}) => (next) => (action) => {
    console.log("第二个logger开始");
    next(action);
}

let store = createStore(reducer,applyMiddleware(logger1,logger2));

store.dispatch({
  type: "type1",
})
//输出
第一个logger开始
第二个logger开始

//通过applyMiddleware包装后的dispatch
let new_dispatch = (...args) = >  logger1(logger2(dispatch(...args)))
```

#### immutableJS
替代方案 seamless-immutable

immutable对象的任何修改或者添加删除都会返回一个新的immutable对象
其实现原理是持久化数据结构（Persistent Data Structure），即旧数据创建新数据时，保证旧数据可用且不变，避免deepcopy把所有节点都复制一遍

两个 immutable 对象可以使用 === 来比较，这样是直接比较内存地址，性能最好。但即使两个对象的值是一样的，也会返回 false,为了直接比较对象的值，immutable.js 提供了 Immutable.is 来做『值比较』
```js
let map1 = Immutable.Map({a:1, b:1, c:1});
let map2 = Immutable.Map({a:1, b:1, c:1});
map1 === map2;             // false
Immutable.is(map1, map2);  // true
```
Immutable.is通过hashCode/valueOf提升比较性能，在react中使用shouldComponentUpdate来进行性能优化的时候能避免deepCopy和deepCompare造成的性能损耗

### normalizr
尽可能把state范式化，不存在嵌套
```js
[{
    id: 1,
    title: 'Some Article',
    author: {
        id: 1,
        name: 'Dan'
    }
}, {
    id: 2,
    title: 'Other Article',
    author: {
        id: 1,
        name: 'Dan'
    }
}]
//希望的结果
{
    result: [1, 2],
    entities: {
        articles: {
            1: {
                id: 1,
                title: 'Some Article',
                author: 1
            },
            2: {
                id: 2,
                title: 'Other Article',
                author: 1
            }
        },
        users: {
            1: {
                id: 1,
                name: 'Dan'
            }
        }
    }
}
```