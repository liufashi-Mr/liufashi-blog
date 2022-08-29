---
title: 高阶函数之柯里化和组合函数
date: 2022-08-29 16:35:50
categories:
  - FE
  - javascript
tags:
  - javascript
---

## 高阶函数

简单的说高阶函数就是一个函数就可以接收另一个函数作为参数，这种函数就称之为高阶函数

## 柯里化

> 柯里化是一种函数的转换，它是指将一个函数从可调用的 f(a, b, c) 转换为可调用的 f(a)(b)(c)。柯里化不会调用函数。它只是对函数进行转换。

柯里化的作用是函数执行产生一个闭包，把一些信息预先存储起来供下级上下文使用。
代码实现

```javascript
const curry = (fn) => (a) => (b) => fn(a, b);
// 用法
const sum = (a, b) => a + b;

let curriedSum = curry(sum);
console.log(curriedSum(1)(2)); // 3
```

箭头函数确实简洁但是可读性也确实会低一点，建议适当的使用！
换个写法

```javascript
function curry(fn) {
  return function (a) {
    return function (b) {
      return fn(a, b);
    };
  };
}
function sum(a, b) {
  return a + b;
}
let curriedSum = curry(sum);
console.log(curriedSum(1)(2)); // 3
```

`curriedSum`是函数 curry 返回一个函数，形参为 a，此时`curriedSum(1)`返回的是另一个函数，形参为 b，`curriedSum(1)(2)`返回的是`sum(1,2)`执行的结果，这样就实现了柯里化之后的函数从 sum(a,b)想 curriedSum(a)(b)的转化。在这个过程中，return 的函数形成了闭包，将参数存储起来了。

我们看看 lodash 中的柯里化是怎么用的

```js
var abc = function (a, b, c) {
  return [a, b, c];
};

var curried = _.curry(abc);

curried(1)(2)(3);
// => [1, 2, 3]

curried(1, 2)(3);
// => [1, 2, 3]

curried(1, 2, 3);
// => [1, 2, 3]

// Curried with placeholders.
curried(1)(_, 3)(2);
// => [1, 2, 3]
```
