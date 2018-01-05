# React 应用实践

### A Simple Component

```js
class HelloWorld extends Component {
    render(){
        return <div>hello world</div>
    }
}
//react-dom render
render(<HelloWorld />, document.getElementById('app'))
```

### JSX

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

JS表达式

```js
//属性比阿达式
<div className={isDisplay ? "show" : "hide"}></div>

//子节点表达式
<div>
    {someFlag ? <span>something</span> : null}
</div>
```



