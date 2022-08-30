---
title: 柯里化和组合函数
date: 2022-08-29 16:35:50
categories:
  - FE
  - javascript
tags:
  - javascript
  - 高阶函数
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
        return curried(...args.concat(args2));
      };
    } else {
      // 输入了全部的参数直接将参数给fn
      return fn(...args);
    }
  };
}
```

## 组合函数

先看个例子

```javascript
const a = (x) => x + 1;
const b = (x) => x - 2;
const c = (x) => x * 3;
console.log(a(b(c(2)))); //5
```

上面是把 `c(2)`的结果作为参数传给 `b` 再将 `b(c(2))`的结果作为参数传给 `c`，但是这样写多少有点不够优雅

```javascript
const a = (x) => x + 1;
const b = (x) => x - 2;
const c = (x) => x * 3;
console.log(a(b(c(2))));
const compose = (...funcs) => {
  // 空不作处理直接把入参返回
  if (funcs.length === 0) return (args) => args;
  // 只有一个参数直接返回这个函数，因为下面使用的reduce必须要至少两个参数
  if (funcs.length === 1) return funcs[0];

  if (funcs.length > 1)
    return funcs.reduce(
      (a, b) =>
        (...args) =>
          a(b(...args))
    );
};
const composed = compose(c);
console.log(composed(2));
```

以上这个实现参考的是 redux 中 compose 的实现
{% gallery %}![redux 中 compose 的实现](https://blog.liufashi.top/img/curing_compose/compose.png){% endgallery %}

## 总结

> 柯里化思想，利用闭包，把一些信息预先存储起来，目的是供下级上下文使用。
> 组合函数，把处理的函数数据像管道一样连接起来，然后让数据穿过管道连接起来，得到最终的结果。
