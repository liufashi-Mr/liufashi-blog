---
title: 使用node将你的学习笔记一键转化为markdown
date: 2022-07-28 17:19:57
categories:
  - node.js
tags:
  - node.js
---

## 背景

最近在学习前端算法与数据结构的时候突然有一个想法，正常做练习算法题肯定是新建 js 文件，然后在文件中将算法题放在注释中像下图这样，{% gallery %}![题目+代码](https://blog.liufashi.top/img/js-to-md/code.png){% endgallery %}
然后我们就可以在终端中运行 js 代码。
然后我本来是打算将这些题目刷完之后整理成 markdown 文件的，然后又觉得到时候题目太多肯定懒得整理。于是乎灵机一动，为什么不写一个自动转化的工具呢？
先看下我的笔记的结构
{% gallery %}![目录](https://blog.liufashi.top/img/js-to-md/dirs.png){% endgallery %}
我们可以使用 nodejs 的文件系统读取文件目录作为 markdown 的 h2~h6 标题，然后再将注释和代码替换成 markdown 的引用和代码块。

先看效果 {% post_link algorithms %}，这个文件就是转化得到的。
然后这里是[源码](https://github.com/liufashi-Mr/algorithms)

## 开始动手

### 处理文件(正则)

先定好不同的注释对应 md 文档的格式，后面写注释的收稍微注意一下就行。
例如 //- ... 和 /_text ... _/ 直接转化为文本 /_quote ... _/ 转化为引用//\```js ... //\```转化为代码

这个时候就需要我们多多了解正则的使用啦

````js
const getContent = (data) => {
  // 文本注释
  const regText = /(\/\*text)([\s\S]*?)(\*\/)/g;
  // 引用注释
  const regQuote = /(\/\*quote)([\s\S]*?)(\*\/)/g;
  // 代码注释
  const regCode = /(\/\/```)([\s\S]*?)(\/\/```)/g;
  // 短文本注释
  const regText_ = /\/\/-.*/g;
  return data
    .replace(regText, (replacement) => {
      return replacement.replace(/\/\*text/g, "").replace(/\*\//g, "");
    })
    .replace(regQuote, (replacement) => {
      return ">" + replacement.replace(/\/\*quote/g, "").replace(/\*\//g, "");
    })
    .replace(regCode, (replacement) => {
      return replacement.replace(/\/\/```/g, "```");
    })
    .replace(regText_, (replacement) => {
      return replacement.replace(/\/\/-/g, "");
    });
};
````

我们可以先试试有没有转化成功，随便读一个文件试试

```js
const path = require("path");
const fs = require("fs");
const file = fs.readFileSync(
  path.join(__dirname, "./1.链表/链表初识.js"),
  "utf8"
);
fs.writeFileSync(path.join(__dirname, "index.md"), getContent(file), "utf8");
```

以下是 js 和 md
{% gallery %}![代码](https://blog.liufashi.top/img/js-to-md/file.png){% endgallery %}
{% gallery %}![生成的md](https://blog.liufashi.top/img/js-to-md/file-res.png){% endgallery %}

可以发现基本没有什么问题啦

### 处理目录

我们可以递归整个文件目录，然后如果是文件夹就继续向下遍历

```js
const getDir = (directory) => {
  if (/\.\w+\/$/.test(directory)) {
    const file = fs.readFileSync(
      path.join(__dirname, directory.substr(0, directory.length - 1)),
      "utf8"
    );
    return [{ file: getContent(file) }];
  }
  return (
    fs
      .readdirSync(path.join(__dirname, directory))
      // 去掉隐藏文件和最外层的index.js,和生成的index.md
      .filter(
        (item) =>
          !/(^|\/)\.[^\/\.]/g.test(item) &&
          item !== "index.js" &&
          item !== "index.md"
      )
      .map((x) => {
        return {
          title: x,
          children: getDir(`${directory}${x}/`),
        };
      })
  );
};
```

然后会生成一个这样的树状结构，避免太长把 file 中的内容去掉了，上面可以看到将 file 中的内容用上面的`getContent`方法处理过.

```js
[
  {
    title: "1.链表",
    children: [
      {
        title: "链表初识.js",
        children: [
          {
            file: "",
          },
        ],
      },
    ],
  },
  {
    title: "2.二叉树",
    children: [
      {
        title: "二叉树.js",
        children: [
          {
            file: "",
          },
        ],
      },
    ],
  },
  {
    title: "3.数组的应用",
    children: [
      {
        title: "1.Map的妙用.js",
        children: [
          {
            file: "",
          },
        ],
      },
      {
        title: "2.双指针法合并有序数组.js",
        children: [
          {
            file: "",
          },
        ],
      },
      {
        title: "3.双指针法三数求和.js",
        children: [
          {
            file: "",
          },
        ],
      },
    ],
  },
];
```

### 渲染这个树结构

我采用深度优先的遍历方式，然后 level 控制标题层级 也就是#的数量

```js
// 深度优先遍历
// markdownTree就是上面getDir的结果
const renderMarkDown = (markdownTree, level) => {
  level++;
  return markdownTree.reduce((pre, item) => {
    if (!item.title) {
      return pre + item.file;
    }
    return (
      pre +
      `\n${new Array(level).fill("#").join("")}\t${item.title
        .replace(/^\d+./, "")
        .replace(/.\w+$/, "")}\n${renderMarkDown(item.children, level)}\n`
    );
  }, "");
};
```

### 最后写入结果

```js
fs.writeFileSync(
  path.join(__dirname, "index.md"),
  // 初始层级为1
  renderMarkDown(getDir("./"), 1),
  "utf8"
);
```

大功告成！
然后在贴一下效果和源码
先看效果 {% post_link algorithms %}，这个文件就是转化得到的。
然后这里是[源码](https://github.com/liufashi-Mr/algorithms)
