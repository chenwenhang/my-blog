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

> “A simple todolist with React”

## Walk with difficulty

自去年第一次听说 React 的时候，就一直想尝试使用这种新颖的方式写前端。但第一次通过教程学习的时候效果却并不好，
甚至可以说是一团乱。突然跳出来这么多没听说过的术语，npm、webpack、redux、routing 等等，根本不知道如何处理，
也就耽搁了。最近再次捡起，积累了几次项目经验，写起来虽然依然有些颠簸，但却比之前的毫无头绪进步了不少。

一个简单的 todolist 制作过程。

---
## Learn & apply immediately 

#### 简易的 todolist

<p id = "TodoList"></p>
![A simple todolist with React](/img/in-post/post-3-todolist.gif "TodoList")

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

![Todolist structure](/img/in-post/post-4-structure.png "Structure")

**红色部分为父模块**，包裹了整个 app ；**绿色部分为添加模块**，用于添加条目；**黄色部分
为显示模块**，用于显示条目，并实现点击删除功能。

#### 父组件

先通过 `createReactClass` 创建一个父模块，在该组件中调用添加模块和显示模块，同时添加初始化
状态方法`getInitialState`，返回一个 todolist 数组，用于存储条目列表，接着添加 `handleChange`
方法，当条目发生增删改查时调用该方法改变 state 重新渲染。代码如下：

```javascript
var TodoList = createReactClass({ // 父组件
  getInitialState:function(){
    return {
        todolist:[]  //返回列表
      };
  },
  handleChange:function(rows){
    //当发生增删改查时改变state重新渲染
    this.setState({
        todolist:rows
    });
  },
  render: function(){
    return (
        <div className="App-content">
          <h1>TodoList</h1>,
          <AddItem todo={this.state.todolist} add={this.handleChange} />
          <ItemList todo={this.state.todolist}  change={this.handleChange} />
        </div>
      );
  }
});
```






