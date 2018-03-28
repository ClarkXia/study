####前言
MVVM在这些年的前端领域算是掀起了一股热潮，比较有代表性的几个框架便是Vue和Angular。介绍MVVM之前，我们还是简单来了解下经常被提及的MV*框架。

####MVC
MVC是一种架构设计的经典模式，随着Web的发展这种设计模式也慢慢融入到前端开发中。大家都懂的前端喜欢造轮子，经典的MVC开始不断变成诸多MV*模式
MVC包括三个部分：
* M（Model）用于封装业务逻辑相关的数据及数据的处理方法，一旦数据发生变化通知相关视图
* V（view）视图界面
* C（controller）定义用户界面对输入的响应方式，用于控制应用的流程。处理用户行为和数据model的改变

上一张经典的图
![](/assets/javascriptMVC.png)

其中涉及到两种设计模式：
1. V和M之间的观察者模式，V观察M，在对应的M上注册，以便了解数据M上发生的变化
2. V和C之间的策略模式，V使用C子类的实例来实现一个特定的响应策略
实际操作的时候C和V间会耦合在一起，因为获取数据逻辑、业务逻辑、渲染视图逻辑在一块，C还可能包含绑定DOM节点事件。
当然前端的实现可以很灵活，比如backbone，它V层上的DOM可以直接改变Model状态，也可以直接向C发送指令（改变前端路由）。相当于业务逻辑全部署在V，C层只保留Router（路由）

####MVP
MVP可以说是MVC模式的改良，将C替换成P究竟发生了什么变化
在MVC中V是可以直接访问M的，但MVP中则不行，需要通过P（Presenter）提供的接口，让P去更细M，M通过观察者模式再去更新V。
![](/assets/mvp.png)
MVP的重大变化就是V与M解耦，可以将V抽离出来。
P将充当V和M之间的交互，除了实现业务逻辑外，还要控制V和M之间的数据“手动同步”，所以P会成为最难维护的部分
####MVVM
MVVM实际上就是把V和M的同步逻辑自动化，去除了MVP中P手动同步的部分。
![](/assets/mvvm.png)
下面我们具体看看MVVM的简单实现
先定个目标，以Vue为例，采用MVVM模式后是这样的
Model
```js
const data = {name: 'name', age: 12};
```
View
```html
<div id="app">
    <div>
        <span>{{name}}</span>
        <span>{{age}}</span>
    </div>
    <button v-on:click="changeName('newName')">change</button>
</div>

```
Controller
```js
new MVVM({
    el: '#app',
    data: data,
    methonds: {
        changeName(name) {
            this.name = name;
        }
    }
});
```
实现上基本可以归结为三步：
1. Observer：实现数据的监听（数据劫持），可以利用Object.defineProperty()来实现，通过getter和setter的设置，监听数据是否发生变化
2. Compiler：主要解决解析模版指令的事情，包括模版中变量的数据替换，初始化渲染，对应节点的事件绑定，添加监听数据的订阅者
3. Watcher：订阅者，关联Observer和Compile，能够订阅并收到属性变化的通知，执行绑定的相应操作，更新视图

####总结
单单看上面的描述还是很难理解的，下面列举一些源码和参考资料供参考学习
[MVVM的实现](https://github.com/DMQ/mvvm)
[前端MVC变形记](http://efe.baidu.com/blog/mvc-deformation/)



