在设计UI组件的过程中不可避免的需要考虑模态窗的需求，比如dialog，tooltip这些，但是在React的框架下，我们似乎遇到了一些问题

###React下的modal需求
通常在设计这些模态窗的时候，会把整个DOM结构尽量渲染在HTML位置比较顶层的地方，比如body。这样相对来说样式的自由度会比较高。
但是在React的整体框架下，它的数据流向是自上而下的，如果你的modal中的内容依赖父级的数据，那可能就要将对应的组建挂载在依赖组建里面。当然可以在顶层组件管理modal数据或者直接上Redux，但是这对于一个定位成UI组件的设计上来说，显示不够合理。这便促成了portal的想法出现---希望modal组件
能跟正常的组件一样不管哪里需要就在哪里挂载，但实际DOM的位置确是另外一个地方（比如React Bootstrap的Portal实现）

###React16之前的实现思路
首先组件不能渲染在它挂载的地方
```js
render() {
    return null;
}
```
DOM真正渲染的位置，通过renderLayer来实现
```js
renderLayer() {
    //这里我们假定render的执行是输出渲染内容，container是DOM需要挂载的节点
    const { render, container } = this.props;
    const layerElement = render();
    this.layerElement = ReactDOM.unstable_renderSubtreeIntoContainer(this, layerElement, container);
}
```
React还是

