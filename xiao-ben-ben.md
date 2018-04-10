#### 捕获脚本错误

window.onerror = function\(a,b,c,d,e\) {}
bugsnag-js

#### 设计模式六大原则

##### 单一职责原则
降低类／方法的复杂度
提高可读性，提高系统的可维护性
变更引起的风险降低

##### 里氏替换原则
子类可以拓展父类的功能，但不能改变父类原有的功能
> 子类可以实现父类的抽象方法，但不能覆盖父类的非抽象方法
> 子类中可以重载父类的方法时，方法的前置条件（即方法的形参）要比父类方法的输入参数更宽松
>当子类的方法实现父类的抽象方法时，方法的后置条件（即方法的返回值）要比父类更严格

##### 依赖倒置原则
高层模块不应该依赖低层模块，二者都应该依赖其抽象；抽象不应该依赖细节；细节应该依赖抽象
核心面向接口编程
> 低层模块尽量都要有抽象类或接口，或者两者都有
> 变量的声明类型尽量是抽象类或接口
> 使用继承时遵循里氏替换原则

##### 接口隔离原则
不应该依赖它不需要的接口；一个类对另一个类的依赖应该建立在最小的接口上

##### 迪米特法则
一个对象应该对其他对象保持最少的了解

##### 开闭原则
一个软件实体如类、模块和函数应该对扩展开放，对修改关闭


#### POST和GET
HTTP协议中两种发送请求的方法
HTTP是基于TCP/IP的关于数据如何在www总通信的协议、
GET和POST本质上就是TCP链接，并无差别。但是由于HTTP的规定和浏览器/服务器的限制，导致他们在应用过程中体现出一些不同
GET产生一个TCP数据包；POST产生两个TCP数据包。

#### macrotask和microtask
* 一个事件循环(event loop)会有一个或多个任务队列(task queue) 
task queue 就是 macrotask queue
* 每一个 event loop 都有一个 microtask queue
* task queue == macrotask queue != microtask queue
* 一个任务 task 可以放入 macrotask queue 也可以放入 microtask queue 中
* 当一个 task 被放入队列 queue(macro或micro) 那这个 task 就可以被立即执行了

event-loop 执行模型
在 macrotask 队列中执行最早的那个 task ，然后移出
执行 microtask 队列中所有可用的任务，然后移出
下一个循环，执行下一个 macrotask 中的任务 (再跳到第2步)

macrotasks: setTimeout ，setInterval， setImmediate，requestAnimationFrame,I/O ，UI渲染
microtasks: Promise， process.nextTick， Object.observe， MutationObserver

#### 函数式编程
* 函数是一等共鸣
* 数据是不可变的（immutable）
* 纯函数
* 支持函数递归调用
* 函数只接收一个参数

#### new的实现
```js
function newOperator(Constr, args) {
    var thisValue = Object.create(Constr.prototype); // (1)
    var result = Constr.apply(thisValue, args);
    //如果构造函数有返回值且返回值是对象，则直接返回该对象
    if (typeof result === 'object' && result !== null) {
        return result; // (2)
    }
    return thisValue;
}
```