---
title: js异步编程、Promise的应用以及在循环中使用Promise。
date: 2022-07-01 16:45:00
categories:
  - FE
tags:
  - javascript
  - FE
  - Promise
---

## 异步编程

Javascript 是一个单线程的语言，在前端编程中，我们在处理一些简短、快速的操作时，往往在主线程中就可以完成。主线程作为一个线程，不能够同时接受多方面的请求。坏处是只要有一个任务耗时很长，后面的任务都必须排队等着，会拖延整个程序的执行。常见的浏览器无响应（假死），往往就是因为某一段 Javascript 代码长时间运行（比如死循环），导致整个页面卡在这个地方，其他任务无法执行。
为了解决这个问题，Javascript 语言将任务的执行模式分成两种：同步（Synchronous）和异步（Asynchronous）。

> "同步模式"就是上一段的模式，后一个任务等待前一个任务结束，然后再执行，程序的执行顺序与任务的排列顺序是一致的、同步的；"异步模式"则完全不同，每一个任务有一个或多个回调函数（callback），前一个任务结束后，不是执行后一个任务，而是执行回调函数，后一个任务则是不等前一个任务结束就执行，所以程序的执行顺序与任务的排列顺序是不一致的、异步的。
>
> "异步模式"非常重要。在浏览器端，耗时很长的操作都应该异步执行，避免浏览器失去响应，最好的例子就是 Ajax 操作。在服务器端，"异步模式"甚至是唯一的模式，因为执行环境是单线程的，如果允许同步执行所有 http 请求，服务器性能会急剧下降，很快就会失去响应。

## 异步变成的方法

我们在实际编程的时候往往需要使用异步代码执行完之后的结果，那用什么方式可以拿到异步代码的结果呢？

### 回调函数

我们可以用一个简单的例子来理解

```js
function addCount(count, success) {
  setTimeout(() => {
    count += 1;
    if (typeof success === "function") success(count);
  }, 100);
}

let num = 0;
addCount(num, (res) => {
  console.log("这里是addCount过后的count", res); // 1
});
```

以上例子就是一个会调函数解决异步问题的例子，在异步代码执行完成后执行传入的回调函数，将异步代码的结果作为回调函数的实参，在回调函数中拿到异步结果，进行异步代码执行完后的操作。
使用回调封装的 ajax

```js
// 格式化请求参数
function formatParams(data) {
  let arr = [];
  for (let name in data) {
    arr.push(encodeURIComponent(name) + "=" + encodeURIComponent(data[name]));
  }
  arr.push(("v=" + Math.random()).replace(".", ""));
  return arr.join("&");
}
function ajax(options) {
  options = options || {};
  options.method = (options.method || "GET").toUpperCase(); // 请求格式GET、POST，默认为GET
  options.dataType = options.dataType || "json"; // 响应数据格式，默认json
  options.timeout = options.timeout || 30000;
  let params = formatParams(options.data); // options.data请求的数据
  let xhr;
  if (window.XMLHttpRequest) {
    xhr = new XMLHttpRequest();
  } else if (window.ActiveObject) {
    xhr = new ActiveXobject("Microsoft.XMLHTTP");
  }

  // 启动并发送一个请求
  if (options.method == "GET") {
    xhr.open("get", options.url + "?" + params, true);
    xhr.send(null);
  } else if (options.method == "POST") {
    xhr.open("post", options.url, true);
    xhr.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
    xhr.send(params);
  }

  // 设置有效时间
  setTimeout(function () {
    if (xhr.readySate != 4) {
      xhr.abort();
    }
  }, options.timeout);

  xhr.onreadystatechange = function () {
    if (xhr.readyState == 4) {
      let status = xhr.status;
      if ((status >= 200 && status < 300) || status == 304) {
        options.success && options.success(xhr.responseText, xhr.responseXML);
      } else {
        options.error && options.error(status);
      }
    }
  };
}
```

使用的时候传 success 回调

```js
ajax({
  url: "http://localhost:3000/test_get",
  method: "get",
  data: {
    name: "name",
    age: 24,
  },
  success: function (data) {
    console.log(data, "asdasdsa");
  },
});
```

> 回调函数的优点是简单、容易理解和部署，缺点是不利于代码的阅读和维护，各个部分之间高度耦合（Coupling），流程会很混乱，而且每个任务只能指定一个回调函数。

### 使用 Promise

还是上面的 addCount 的例子,使用 Promise 改写

```js
function addCount(count) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      count += 1;
      resolve(count);
    }, 100);
  });
}

let num = 0;
addCount(num).then((res) => console.log(res)); // 1
```

上面的 ajax 请求封装也可以使用 promise 的形式，这里简单改写一下

```js
//封装ajax请求
function ajax(url, method, data) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.open(method, url);
    xhr.send(data);
    xhr.onreadystatechange = function () {
      if (xhr.readyState === 4) {
        if (xhr.status === 200) {
          resolve(xhr.responseText);
        } else {
          reject(xhr.statusText);
        }
      }
    };
  });
}
```

async/await 是 Promise().then()的一个语法糖，目的是让你的异步代码看起来像同步的一样。但是需要我们在一个 async function 中使用它，并且使用 async/await 的时候如果需要拿捕获错误就需要使用 try catch，但是使用 Promise 则可以使用.catch 来捕获异常，如下：

```js
function addCount(count) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      count += 1;
      reject(count);
    }, 100);
  });
}

let num = 0;

// 使用async、await + try catch
(async () => {
  try {
    const res = await addCount(num);
    console.log(res);
  } catch (error) {
    console.log(error);
  }
})();

addCount(num)
  .then((res) => {
    console.log(res);
  })
  .catch((err) => console.log(err));
```

另外还有其他的解决异步请求的方式就不多赘述了 可以看看这篇文章[Javascript 异步编程的 4 种方法](https://www.ruanyifeng.com/blog/2012/12/asynchronous%EF%BC%BFjavascript.html)
这里主要讲下 Promise 的运用

### 在循环中使用 Promise

例如下面一个数据

```js
// 初始数据
const data = [
  { id: 1, name: "罗翔" },
  { id: 2, name: "谭乔" },
  { id: 3, name: "张三" },
];
// 通过接口拿到数据
const res = [
  { id: 1, name: "罗翔", list: [] },
  { id: 2, name: "谭乔", list: [] },
  { id: 3, name: "张三", list: [] },
];
```

如果有需求需求我们依次拿 id 请求数据，然后再将所有的对应结果放进 res，最后再将 res 渲染到页面上。虽然这样的需求不多，一般都是传个 list<id>后端直接返回给你 res 了，但是如果真的遇到了我们可以用更优雅的方式解决。
先看下不优雅的方式：

```js
// 下面的axios随便写的，理解为异步请求就完了

//1. 使用setTimeout，这也是最不靠谱的方式了，不仅不够优雅，而且接口慢数据多的时候还不能拿到正确的值
const result = [];
data.forEach((item, index) => {
  axios.get("url", { ...item }).then((res) => result.push(res.data));
});
setTimeout(() => {
  console.log(result);
}, 1000);

// 2. 加个计数器
let count = 0;
const result = [];
data.forEach((item, index) => {
  axios.get("url", { ...item }).then((res) => {
    result.push(res.data);
    count += 1;
    // 在最后一个执行完的.then中拿到想要的数据
    // 注意，这里是最后一个执行完，并不是数组的最后一个数据，因为请求时间有长短
    if (count === data?.length) {
      console.log(result);
    }
  });
});
```

最后说一下靠谱且优雅的方法吧
我们都知道 Promise.all()是传入一个 Promise 数组`Array<Promise<*>>`在.then 中就可以拿到所有请求完成的结果

```js
Promise.all(data.map((x) => axios.get("url", { ...x }))).then((res) => {
  const result = res.map((x) => x.data);
  console.log(result);
});
```

### 在循环中使用 Promise + 递归

我们将这个离谱的需求再升级,假设该接口需要传一个 time 的参数，表示在今天之后几天的数据，现在需求是拿到这三个人同一天都有数据的结果。那么在这个时候可能就需要递归了？

```ts
const getList = (data, time = 0) => {
  return new Promise(() => {
    Promise.all(data.map((x) => axios.get("url", { ...x, time }))).then(
      (res) => {
        let hasNullData = false;
        // 存在空数据则请求后一天的数据
        res?.forEach((x) => {
          if (x.data.length <= 0) {
            hasNullData = true;
          }
        });
        time += 1;
        !hasNullData ? resolve(res) : resolve(queryRecordList(data, time));
      }
    );
  });
};
```
