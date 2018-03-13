

在设计UI组件的过程中不可避免的需要考虑模态窗的需求，比如dialog，tooltip这些，但是在React的框架下，我们似乎遇到了一些问题

### React下的modal需求
通常在设计这些模态窗的时候，会把整个DOM结构尽量渲染在HTML位置比较顶层的地方，比如body。这样相对来说样式的自由度会比较高。
但是在React的整体框架下，它的数据流向是自上而下的，如果你的modal中的内容依赖父级的数据，那可能就要将对应的组建挂载在依赖组建里面。当然可以在顶层组件管理modal数据或者直接上Redux，但是这对于一个定位成UI组件的设计上来说，显示不够合理。这便促成了portal的想法出现---希望modal组件
能跟正常的组件一样不管哪里需要就在哪里挂载，但实际DOM的位置确是另外一个地方（比如React Bootstrap的Portal实现）

### React16之前的实现思路
首先组件不能渲染在它挂载的地方
```js
render() {
    return null;
}
```
DOM真正渲染的位置，通过renderLayer来实现
```js
renderLayer() {
    //这里我们假定render的执行是输出渲染内容
    const { render } = this.props;
    //构造DOM节点作为渲染内容的容器
    if (!this.layer) {
        this.layer = document.createElement('div');
        document.body.appendChild(this.layer);
    }
    const layerElement = render();
    this.layerElement = ReactDOM.unstable_renderSubtreeIntoContainer(this, layerElement, this.layer);
}

unrenderLayer() {
    if (this.layer) {
        React.unmountComponentAtNode(this.layer);
        document.body.removeChild(this.layer);
        this.layer = null;
    }
}
```
好了，我们在各个生命周期里面调用它们就行了

ReactDOM中提供了一个unstable_renderSubtreeIntoContainer，从名字上就可以发现，它并不推荐被使用，实际上它也的确表现得令人费解。

```js
class Test extends React.Component {
  componentDidMount() {
    console.log('test');
    setTimeout(() => this.forceUpdate(),5000)
  }
  
  componentDidUpdate() {
    console.log('did update test')
  }
  
  render() {
    return <p>test<A/></p>;
  }
}

class B extends React.Component {
  componentDidMount() {
    console.log('did mount B')
  }
  
  componentDidUpdate() {
    console.log('did update B')
  }
  
  render() {
    return <a>some thing</a>;
  }
}

class A extends React.Component {
  componentDidMount() {
    this.renderLayer();
    console.log('did mount A')
  }
  
  componentDidUpdate() {
    this.renderLayer();
    console.log('did update A')
  }
  
  renderLayer() {
    if (!this.layer) {
      this.layer = document.createElement('div');
      document.body.appendChild(this.layer);
    }
    ReactDOM.unstable_renderSubtreeIntoContainer(this, <B/>, this.layer);
  }
  render() {
    return null;
  }
}

ReactDOM.render(<Test />, document.getElementById('app'));
```
https://codepen.io/anon/pen/GQRaEo?editors=1112
按我们对React父子组件间生命周期的执行情况上理解应当输出
```js
"did mount B"
"did mount A"
"test"
"did update B"
"did update A"
"did update test"
```
而实际的结果却是
```js
"did mount B"
"did mount A"
"test"
"did update A"
"did update test"
"did update B"
```
显然在初始化的时候事情还是符合我们预期的 可是在执行更新组件的时候，生命周期的执行便显得很混乱，在React16的版本中这个问题得到了修复，但执行的结果显然也不是我们最终想要的
https://codepen.io/anon/pen/MQWdPq?editors=1111
```js
"did mount A"
"test"
"did mount B"
"did update A"
"did update test"
"did update B"
```
React Portal的出现彻底解决了这方面的问题
### React Portal
终于进入主题，先看看它是如何使用的
```js
const node = document.createElement('div');
document.body.appendChild(this.node);
...

render() {
    return createPortal(
      <div class="dialog">
        {this.props.children}
      </div>, //需要渲染的内容
      node //渲染内容的容器DOM
    );
}
```
除了node节点在一些场景下需要释放之外，你已经不需要在其他生命周期里面擦屁股了
让我们在回到之前生命周期执行上的问题
https://codepen.io/anon/pen/Jpjqwg?editors=1111
结果的执行跟我们正常组件保持了一致，再也不用担心一些依赖子组件完成更新后的监听或操作会出现异常情况了。
除此之外React Portal还新增了一个事件冒泡的实现
```js
<div onClick={handleClick}>
  <Dialog/>
</div>
```
如果在React16之前的实现方式，点击Dialog组件里面的内容handleClick是不会被触发，但通过React Portal实现的挂载方式将会发生冒泡。
这个特性见仁见智吧，一般情况下感觉也不会用到。
### 总结
总之React Portal的实现对于modal的实现是一个重大的更新，同时也避免了组件间生命周期的执行混乱。
