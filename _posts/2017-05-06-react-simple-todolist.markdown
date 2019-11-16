---
layout:     post
title:      "React制作简易Todolist"
date:       2017-05-06 09:51:00
author:     "Echo"
header-img: "img/post-bg-js-module.jpg"
catalog: true
tags:
    - React
    - javascript
---

> “A simple Todolist with React”

## Walk with difficulty

自去年第一次听说 React 的时候，就一直想尝试使用这种新颖的方式写前端。但第一次通过教程学习的时候效果却并不好，
甚至可以说是一团乱。突然跳出来这么多没听说过的术语，npm、webpack、redux、routing 等等，根本不知道如何处理，
也就耽搁了。最近再次捡起，积累了几次项目经验，写起来虽然依然有些颠簸，但却比之前的毫无头绪进步了不少。

一个简单的 Todolist 制作过程。

---
## Learn & apply immediately 

#### 简易的 Todolist

<p id = "TodoList"></p>

![A simple todolist with React](/my-blog/img/in-post/post-3-todolist.gif "TodoList")

#### 使用 create-react-app 快速构建 React 开发环境

create-react-app 是来自于 Facebook，通过该命令我们无需配置就能**快速构建 React 开发环境**。
create-react-app 自动创建的项目是基于 Webpack + ES6 。
执行以下命令创建项目（[npm教程](http://www.runoob.com/nodejs/nodejs-npm.html)）：

```
$ cnpm install -g create-react-app
$ create-react-app react-todolist
$ cd react-todolist
$ npm start
```

在浏览器中打开 **http://localhost:3000/** ，即可看到效果。

#### 组件模块

首先，从[上面的效果图](#TodoList)我们可以分析出 TodoList 大致可分为三个部分，如下图：

![Todolist structure](/my-blog/img/in-post/post-3-structure.png "Structure")

**红色部分为父模块**，包裹了整个 app ；**绿色部分为添加模块**，用于添加条目；**黄色部分
为显示模块**，用于显示条目，并实现点击删除功能。

#### 父组件

在 `src/app.js` 文件中，先创建一个父模块，在该组件中调用添加模块和显示模块，同时通过构造函数初始化状态
，在状态中添加 todolist 数组，用于存储条目列表，接着添加 `handleChange`方法，当条目发生增
删改查时调用该方法改变 state 重新渲染。代码如下：

```javascript
class TodoList extends Component{ // 父组件
  constructor(props) {
    super(props);
    //初始化state，存储todolist条目数组
    this.state = {todolist:[]};
  }
  handleChange(rows){
    //当发生增删改查时改变state重新渲染
    this.setState({
        todolist:rows
    });
  }
  render(){
    return (
        <div className="App-content">
          <h1>TodoList</h1>,
          <AddItem todo={this.state.todolist} add={this.handleChange.bind(this)} />
          <ItemList todo={this.state.todolist}  change={this.handleChange.bind(this)} />
        </div>
      )
  }
}
```

#### 添加模块

添加模块由一个输入框和一个按钮组成，输入框用于输入条目，我们为其添加 `ref` 属性。接着为按钮添加点击事件，触
发 `addItem` 方法，用于获取输入框中的值，并将其添加至 todolist 数组，然后回调 `add` 方法，改变模块状态重新渲染，
代码如下：

```javascript
class AddItem extends Component{ // 添加任务
  addItem() {
    //获取真实DOM 虚拟DOM无法获取表单元素的数据
    var inputDom = ReactDOM.findDOMNode(this.refs.inputnew);
    //获取数据
    var newthing = inputDom.value.trim();
    //如果输入的数据为空值则返回提示无法添加
    if(newthing === ""){
        alert("The input cannot be empty. ");
        return;
    }
    //获取todolist列表
    var rows = this.props.todo;
    //在列表内添加新数据
    rows.push(newthing);
    //回调改变state
    this.props.add(rows);
    //清空输入框
    inputDom.value = "";
  }
  render(){
    return (
        <div>
          <input type="text" ref="inputnew" placeholder="Typing a newthing todo , click the item to delete." className="Item-input" />,
          <button onClick={this.addItem.bind(this)} className="Item-button"> Add </button>
        </div>
      )
  }
}
```

#### 显示模块

显示模块直接通过 `map` 方法遍历出 todolist 数组的所有元素，即可起到显示的作用。同时为每个条目添加点击事件，
获取当前条目的标识符，根据标识符删除 todolist 对应条目，并回调改变状态重新渲染，代码如下：

```javascript
class ItemList extends Component{ //任务列表
  deleteItem(e) {
    //获取todolist列表
    var rows = this.props.todo;
    //获取当前条目的data-index
    var index = e.target.getAttribute("data-index");
    //根据获取的data-index在rows里删除对应条目
    rows.splice(index,1);
    //回调改变state
    this.props.change(rows);
  }
  render() {
    return (
        <ul id="todolist" className="Item-ul" >
          {
            // 遍历数据
            this.props.todo.map(
              (item,i)=>{
                return(
                <li key={i} data-index={i} className="Item-li" onClick={this.deleteItem.bind(this)} > 
                  <span>{item}</span>
                </li>
              )
              })
          }
        </ul>
      )
    }
}
```
## Key point

#### 关于 bind(this)

es6 绑定事件需要 `onClick={this.exampleFunction.bind(this)}` ，这样 `exampleFunction` 和 `bind` 里面的参数 `this`  的作用域
才绑定到了一起（注意es5是不需要这个 `bind(this)`的），`exampleFunction` 中如果有this.name这类语句，相当于是使用参数 `this` 里面的
变量 `name` 的值，或者使用箭头函数 `exampleFunction= (e)=> {函数体}` 。

关于 `bind` 的官方 API 文档是这样描述的：

<cite>bind方法会创建一个新函数,称为绑定函数.当调用这个绑定函数时,绑定函数会以创建它时传入
bind方法的第一个参数作为this,传入bind方法的第二个以及以后的参数加上绑定函数运行时本身的参数按照顺序作为原函数的参数来调用原函数.</cite>

## Thanks

React 写起来确实挺有成就感的，有层次有逻辑。 Demo 已上传至github，地址：
[https://github.com/chenwenhang/react-todolist](https://github.com/chenwenhang/react-todolist)

能闲下来花点时间学一学自己想学的东西还是挺愉快的，后面期末又要忙起来了。

致谢 React ！

