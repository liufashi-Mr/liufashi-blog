---
title: 发布-订阅设计模式
date: 2022-08-20 15:36:51
categories:
  - 软件架构
tags:
  - 软件架构
  - FE
  - react
  - vue
---

## 前言

先说一下 react/vue 的组件通信方式

1. 父组件向子组件通信
2. 子组件向父组件通信
3. 跨级组件通信
4. 非嵌套关系的组件通信

其中最后一种非嵌套的组件通信方式要怎么实现呢

- 如果是兄弟组件通信，可以找到这两个兄弟节点共同的父节点, 结合父子间通信方式进行通信。
- 可以通过 redux 等进行全局状态管理
- 可以使用自定义事件通信（发布订阅模式），使用 events.js

然后今天的主角就是他基于发布订阅模式实现的组建通信

## 发布订阅模式

什么是发布订阅模式呢？

> 在软件架构中，发布订阅是一种消息范式，消息的发送者（称为发布者）不会将消息直接发送给特定的接收者（称为订阅者）。而是将发布的消息分为不同的类别，无需了解哪些订阅者（如果有的话）可能存在。同样的，订阅者可以表达对一个或多个类别的兴趣，只接收感兴趣的消息，无需了解哪些发布者（如果有的话）存在。

{% gallery %}![发布订阅模式](https://blog.liufashi.top/img/eventEmitter/event-bus.png){% endgallery %}

结合 react 和 event.js 的使用来具体了解下。
首先有两个非嵌套关系的组件
全局新建 eventBus.js 文件,安装 event.js 后导出实例
这个时候实例相当于消息主体，负责存储消息与订阅者的对应关系，有消息触发时，负责通知订阅者。

```js
import { EventEmitter } from "events";
export default new EventEmitter();
```

```jsx
import React from "react";
import ClassTest from "./ClassTest";
import FuncTest from "./FuncTest";
function App() {
  return (
    <>
      <ClassTest />
      <FuncTest />
    </>
  );
}

export default App;
```

组件 ClassTest，这里的`emitter.on('test')`就是相当于订阅者，去消息中心订阅自己感兴趣的消息。`emitter.off('test')`取消订阅

```jsx
import React, { Component } from "react";
import emitter from "./eventbus";

class ClassTest extends Component {
  handleChange = (param1, param2) => {
    console.log(param1);
    console.log(param2);
  };
  componentDidMount() {
    emitter.on("test", this.handleChange);
  }
  componentWillUnmount() {
    emitter.off("test", this.handleChange);
  }
  render() {
    return <div>class test</div>;
  }
}
export default ClassTest;
```

组件 FuncTest`emitter.emit('test')`在这里是订阅者的身份，满足条件时，通过消息中心发布消息。此处的条件是点击这个 div

```jsx
import React, { useState, useContext } from "react";
import emitter from "./eventbus";
const FuncTest = () => {
  const handleChange = () => {
    emitter.emit("test", "参数1", "参数2");
  };
  return <div onClick={handleChange}>B组件</div>;
};

export default FuncTest;
```

在点击这个 div 之后，通过事件总线将 emit 中携带的 "参数 1", "参数 2"，传递到 on 的回调函数中。实现了组建的通信
在 vue 中也是类似，可以 new 一个 Vue 的实例，然后 vue 中提供了$on和$emit 等方法能达到同样的效果.

```js
export const EventBus = new Vue();
```

## 源码简单实现

简单实现下我们只需在 emit 的时候触发 on 订阅的事件，并且将参数带过去。详细实现可以看注释

```js
// 构造函数
function EventEmitter() {
  // 将事件名作为key，事件触发后所执行的回调作为value，
  // 然后可能会有多个地方订阅同一个事件但是他们需要执行不同的回调，所以这里value是个数组
  this.eventObj = {};
}
// 添加订阅方法
EventEmitter.prototype.on = function (event, callback) {
  //没有就添加
  this.eventObj[event] = this.eventObj[event] || [];
  this.eventObj[event].push(callback);
};
// 添加发布方法
EventEmitter.prototype.emit = function (event, ...args) {
  // 如果存在事件
  if (this.eventObj[event]) {
    this.eventObj[event].forEach((callback) => {
      // 遍历执行所有的回调，并将emit的时候的参数传给他
      callback(...args);
    });
  }
};
// 取消订阅
EventEmitter.prototype.off = function (event) {
  delete this.eventObj[event];
};
```

可以将上面的 eventbus.js 中引用的包替换为这个，看看是否正常运行。看到这里是不是豁然开朗呢，相信对 vue 中的 bus 总线或者是 event.js 一定有了更深的理解！
