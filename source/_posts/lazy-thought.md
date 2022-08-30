---
title: 谈一谈前端开发中的惰性思想
date: 2022-08-30 10:59:29
categories:
  - FE
  - javascript
tags:
  - javascript
  - 高阶函数
---

## 惰性思想

惰性函数：避免重复地去做某一样东西形成冗余。
惰性函数优点：就是能避免多次重复的步骤判断，冗余等，只需一次判定，即可直接去使用，不用做无用的重复步骤。
惰性函数的应用场景：常用于函数库的编写，单例模式之中。在固定的应用环境不会发生改变，频繁要使用同一判断逻辑的。

## 示例

在日常的项目中，其实我们很多地方都可以运用到惰性思想。例如要封装一个获取元素属性的方法，因为低版本的 ie 浏览器不支持 getComputedStyle 方法，做了一个容错处理：

```javascript
function getCss(element, attr) {
  if ("getComputedStyle" in window) {
    return window.getComputedStyle(element)[attr];
  }
  return element.currentStyle[attr];
}
```

但是每次进这个方法都要做一下判断，为了提高代码的可维护性，我们可以存一个变量，然后每次进去判断变量就好了

```js
var flag = "getComputedStyle" in window;
function getCss(element, attr) {
  if (flag) {
    return window.getComputedStyle(element)[attr];
  }
  return element.currentStyle[attr];
}
```

但是每次执行 getCss 函数的时候，都需要判断执行，那有没有一种方式可以优化了，这时候惰性思想就可以用上了。

```js
function getCss(element, attr) {
  if ("getComputedStyle" in window) {
    getCss = function (element, attr) {
      return window.getComputedStyle(element)[attr];
    };
  } else {
    getCss = function (element, attr) {
      return element.currentStyle[attr];
    };
  }
  // 为了第一次也能拿到值
  return getCss(element, attr);
}

getCss(document.body, "margin");
getCss(document.body, "padding");
getCss(document.body, "width");
```

第一次执行，如果有 getComputedStyle 这个方法，getCss 就被赋值成

```js
function (element, attr) {
    return window.getComputedStyle(element)[attr];
};
```

类似于这种判断之后不同条件返回不通的内容的还有很多，这个时候就可以通过这种方式进行优化

## 总结

示例参考[高阶函数](https://juejin.cn/post/7086393986780233736#heading-3)
