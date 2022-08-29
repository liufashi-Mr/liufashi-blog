---
title: 柯里化和组合函数
date: 2022-08-29 16:35:50
categories:
  - FE
  - javascript
tags:
  - javascript
---

## 柯里化

> 柯里化是一种函数的转换，它是指将一个函数从可调用的 f(a, b, c) 转换为可调用的 f(a)(b)(c)。柯里化不会调用函数。它只是对函数进行转换。

柯里化的作用是函数执行产生一个闭包，把一些信息预先存储起来供下级上下文使用。柯里化就是闭包一个很典型的应用。
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

// 如果传入了部分的参数，此时它会返回当前函数，并且等待接收剩余参数
curried(1)(2)(3);
// => [1, 2, 3]
curried(1, 2)(3);
// => [1, 2, 3]
// 如果输入了全部的参数，则立即返回结果
curried(1, 2, 3);
// => [1, 2, 3]
```

那我们可以试试是实现这个效果

```javascript
function curry(fn) {
  return function curried(...args) {
    // fn的length属性指明函数的形参个数。args的长度少于形参个数对应上面的传入部分参数
    // 如果传入参数 args.length 小于 fn 函数的形参个数 fn.length，需要重新递归
    if (args.length < fn.length) {
      return function (...args2) {
        // 之前传入的参数都储存在 args2 中
        // 递归执行，重复之前的过程
        return curried([...args, ...args2]);
      };
    } else {
      // 输入了全部的参数直接将参数给fn
      return fn(...args);
    }
  };
}
```

## 组合函数
