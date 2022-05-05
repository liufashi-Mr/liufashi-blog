---
title: react+typescript构建h5项目
date: 2022-04-27 10:47:54
categories:
  - FE
  - react
  - typescript
tags:
  - react
  - typescript
  - FE
pin: true
headimg: /img/typescript-h5-template/headimg.png
---

## 介绍

&#8195;&#8195;项目为基于 create-react-app 后面称为 cra 实现的移动端的一个项目模板。从零开始的项目构建，包括路由配置（react-router-dom v6），lint 规则，commit 规则，移动端适配等等。大家可以基于这个项目进行修改，[github 地址](https://github.com/liufashi-Mr/h5-react-typescript)，[预览地址](http://h5.template.liufashi.top)，也可以按照我的思路自己去搭建一遍。

## 起步

&#8195;&#8195;使用 cra 创建项目`npx create-react-app [project name] --template typescript`，现在新版本 cra 只能使用 npx 来创建项目了。由于我们需要使用修改项目中 webpack 的配置文件，使用`npm run eject`将 webpack 配置暴露出来进行修改，也可以使用`npm i react-app-rewired -D`安装插件，在根目录创建 config-overrides.js 文件来覆盖配置。这里我推荐第一种方式，一是可以试着熟悉 cra 的 webpack 是怎样配置的，其次能避免一些奇奇怪怪的问题。

## 配置规范

&#8195;&#8195;项目没有一个统一的规范和代码风格看起来既难受又不利于维护，随着项目接手的人越多项目就会越来越乱，久而久之就成了所谓的屎山。

> 不以规矩,不成方圆

### lint 规则

安装 eslint 和相关的包`npm i eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin -D`，根目录下创建.eslintrc.js，和.eslintignore 文件。前者为 eslint 的检测规则，后者为不参与检测的文件。
在.eslintrc.js 中添加如下内容，具体的配置可以看 eslint 官网配置项。以下为我的配置

```js
module.exports = {
  //此项是用来告诉eslint找当前配置文件不能往父级查找
  root: true,
  parser: "@typescript-eslint/parser",
  //此项指定环境的全局变量
  env: {
    browser: true,
    es6: true,
    node: true,
  },
  //此项是用来指定javaScript语言类型和风格，sourceType用来指定js导入的方式
  parserOptions: {
    ecmaFeatures: {
      jsx: true,
    },
    ecmaVersion: 2018,
    sourceType: "module",
  },
  // 此项是用来配置标准的js风格，就是说写代码的时候要规范的写
  extends: [
    "eslint:recommended",
    "eslint:recommended",
    "plugin:@typescript-eslint/eslint-recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:react/recommended",
  ],
  plugins: ["@typescript-eslint", "prettier", "react", "react-hooks"],
  // "off" -> 0 关闭规则
  // "warn" -> 1 开启警告规则 可以提交
  // "error" -> 2 开启错误规则 无法提交
  rules: {
    // 检查 Hooks 的使用规则
    "react-hooks/rules-of-hooks": "error",
    // 检查依赖项的声明
    "react-hooks/exhaustive-deps": "warn",
    //console
    "no-console": "warn",
    //debugger
    "no-debugger": "off",
    //定义未使用警告
    "no-unused-vars": "warn",
  },
};
```

然后再 package.json 中添加 script

```json
{
  "scripts": {
    "lint": "eslint . --ext .ts && eslint . --ext .tsx",
    "lint:fix": "eslint --fix"
  }
}
```

### prettier 规范

在根目录创建.prettierrc.js 文件（同理也可以创建.prettierignore）.prettierrc.js 中添加如下内容，具体的配置可以看 prettierrc 官网配置项。以下为我比较习惯的 react 项目的配置

```js
module.exports = {
  //每行宽度
  printWidth: 100,
  //制表符宽度
  tabWidth: 2,
  //使用分号
  semi: true,
  //单引号
  singleQuote: false,
  //行末尾标识
  endOfLine: "auto",
  //对象中的空格
  bracketSpacing: true,
  //箭头函数中的括号, avoid->需要时使用  always->永远使用
  arrowParens: "avoid",
  //es5有效的地方保留逗号
  trailingComma: "es5",
  //对象属性有一个存在引号,全部加上引号
  quoteProps: "consistent",
};
```

可以在 vscode 的 setting.json 中添加如下代码在保存和粘贴的时候格式化代码

```json
  "editor.formatOnPaste": true,
  "editor.formatOnSave": false
```

### commit 规则

新建.commitlintrc.js 文件，添加如下规范。这个也是目前主流的 commit 规则，我看好多 git 上的项目使用的都是这一套规范

```js
module.exports = {
  extends: ["@commitlint/config-conventional"],
  rules: {
    //0->off 1->warn 2->error
    "type-enum": [
      2,
      "always",
      [
        "feat", // 新功能（feature）
        "fix", // 修补bug
        "bugfix", // 修补bug
        "docs", // 文档（documentation）
        "style", // 格式（不影响代码运行的变动）
        "refactor", // 重构（即不是新增功能，也不是修改bug的代码变动）
        "test", // 增加测试
        "revert", // 回滚
        "config", // 构建过程或辅助工具的变动
        "chore", // 其他改动
        "message", //注释&文案更改
      ],
    ],
    "type-empty": [2, "never"],
    "subject-empty": [2, "never"],
    "subject-full-stop": [0, "never"],
    "subject-case": [0, "never"],
  },
};
```

&#8195;&#8195;规则配置就绪后，我们可以添加 git 的钩子，在 pre-commit 的时候执行规则。安装`npm i husky@4.3.8 lint-staged -D`。**踩坑：尽量安装该版本的 husky，我在使用 7.0.1 版本的时候 commit 无法触发改检验。**
然后`package.json`的最外层添加如下代码。

```json
{
  ...
  "lint-staged": {
    "*.{jsx,ts,tsx,js}": [
      "node -v",
      "npm run lint:fix",
      "prettier --write "
    ]
  },
  "husky": {
    "hooks": {
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS",
      "pre-commit": "lint-staged"
    }
  }
  ...
}
```

这样你在 commit 之前就会先检验 commit 内容是狗符合规范，然后就是当前 node 版本，然后检测 lint 规则，最后就是自动格式化代码。

## 项目配置

### 项目别名

在`/config/webpack.config.js`中搜索 alias 约 300 行左右,添加如下配置，可以根据自己的命名和编码习惯来配置修改。

```js
alias: {
        // Support React Native Web
        // https://www.smashingmagazine.com/2016/08/a-glimpse-into-the-future-with-react-native-for-web/
        "react-native": "react-native-web",
        // Allows for better profiling with ReactDevTools
        ...(isEnvProductionProfile && {
          "react-dom$": "react-dom/profiling",
          "scheduler/tracing": "scheduler/tracing-profiling",
        }),
        ...(modules.webpackAliases || {}),
        +"@": path.resolve(__dirname, "../src"),
        +"@apis": path.resolve(__dirname, "../src/api"),
        +"@icons": path.resolve(__dirname, "../src/assets/icons"),
        +"@images": path.resolve(__dirname, "../src/assets/images"),
        +"@components": path.resolve(__dirname, "../src/components"),
        +"@pages": path.resolve(__dirname, "../src/pages"),
        +"@utils": path.resolve(__dirname, "../src/utils"),
        +"@router/*": path.resolve(__dirname, "../src/router"),
      },
```

此时就可以采用路径别名的方式在项目中引入模块/文件。此时我们使用 command/control+click 还不能跳转至对应的文件。为了方便代码的查阅，在 cra 生成的`tsconfig.json`中添加如下配置，路径要和 alias 配置的一至。不添加`baseUrl: "./"`配置的话，路径需要为相对路径

```json
{
  "compilerOptions": {
    ...
    "baseUrl": "./",
    "paths": {
      "@/*": ["src/*"],
      "@api/*": ["src/apis/*"],
      "@icons/*": ["src/assets/icons/*"],
      "@images/*": ["src/assets/images/*"],
      "@components/*": ["src/components/*"],
      "@pages/*": ["src/pages/*"],
      "@utils/*": ["src/utils/*"],
      "@router/*": ["src/router/*"]
    }
  },
  ...
}

```

### 配置全局的 types

在 tsconfig 中添加配置项

```json
{
  "compilerOptions": {
    ...
    "baseUrl": "./",
  + "typeRoots": ["src/@types/"],//全局types的路径
    "paths": {
      "@/*": ["src/*"],
      "@api/*": ["src/apis/*"],
      "@icons/*": ["src/assets/icons/*"],
      "@images/*": ["src/assets/images/*"],
      "@components/*": ["src/components/*"],
      "@pages/*": ["src/pages/*"],
      "@utils/*": ["src/utils/*"],
      "@router/*": ["src/router/*"]
    }
  },
  ...
}
```

### 配置项目结构

按照我个人习惯的命名方式，项目结构如下：

```bash

 medical-guidanceH5-web
    ├── config
    │     ├── webpack.config.js
    │     └── ...
    ├── script
    │     ├── start.js
    │     ├── build.js
    │     └── test.ks
    ├── .commitlintrc.js
    ├── .eslintrc.js
    ├── .prettierrc.js
    ├── tsconfig.json
    ├── package.json
    ├── .gitignore
    ├── .eslintignore
    ├── public
    │   ├── favicon.ico
    │   └── index.html
    ├── src
    │    ├── apis
    │    │     ├── config.ts
    │    │     ├── index.ts
    │    │     ├── request.ts
    │    ├── assets #资源文件
    │    ├── components #项目组件
    │    ├── pages #页面
    │    │      ├──<Name> #页面名称 驼峰命名
    │    │          ├── index.tsx
    │    │          ├── index[.module].(scss|sass)
    │    ├── router #路由配置
    │    └── utils #工具函数&&自定义hooks
    ├── README.md
    ├── package-lock.json
    └── yarn.lock


```

### 移动端适配

> 一般情况下设计稿的设计师按照 375 的尺寸设计，然而，在现在移动终端（就是手机）快速更新的时代，每个品牌的手机都有着不同的物理分辨率，这样就会导致，每台设备的逻辑分辨率也不尽相同，此时 357 的设计稿，如果想要还原那基本是不可能了，因为如果一个左右布局，左边如果写死，右边自适应的话，每个设备的右边所展示的内容大小就不尽相同，这是移动端适配就显得尤其重要

更多有关移动端适配的信息[移动端适配](https://juejin.cn/post/6844903631993454600)。
本项目采用 rem 实现移动端的适配：

#### 基本原理

通常的情况下给的 UI 给的设计稿一般为 375 或者 750。以 375 为例设计稿中给出的一个宽为 75 的 div，此时这个 div 占页面宽度的 1/5。我们需要实现的就是在一定的尺寸范围内不管页面尺寸为多少，这个 div 始终占 1/5。

- 在代码中使用 rem 作为单位，为了便于理解此处将根节点字体大小设置为 10px，此时只需将设计稿上的尺码/10 即可。然后在屏幕尺寸改变的时候修改根节点字体大小。
  根节点字体大小 = 屏幕宽度/37.5 （px）如果是 750 的设计稿调整为 屏幕宽度/75 （px）。在修改完 html 的字体大小后最后是在 body 上加上一个默认字体大小，防止后面没有设置 font-size 的元素集成根节点的字体大小。

```css
div {
  width: 7.5rem;
}
```

- 如果不想在项目中使用 rem 可以使用 pxtorem 插件，达到同样的效果，在 cra 项目的 postcss-loader options 中`postcss-normalize`下方大约 144 行加入

```js
  [
    "postcss-pxtorem",
    {
      rootValue: 10,
      propList: ["*"],
      exclude: /node_modules/i,
    },
  ],
```

- 此时再让根节点字体大小随着页面变化而变化就实现了移动端的适配。可以用最简单的方式：

```css
html {
  font-size: 2.6667vw !important; //750的设计稿则为1.3333vw
```

#### 项目实现

以上为移动端适配的基本原理，在项目中我们可以在使用更成熟全面的方案，`npm i lib-flexible -S`安装插件，该插件可以根据屏幕大小修改根节点的字体大小，在 index.tsx 中引入。然后根绝设计稿修改 pxtorem 插件的 rootvalue。如下：

```js
  [
    "postcss-pxtorem",
    {
      rootValue:37.5,//37.5对应375的设计稿，75对应750的设计稿
      propList: ["*"],
      exclude: /node_modules/i,
    },
  ],
```

### 配置路由

路由采用的是最新的 react-router-dom v6 版本，也是抱着使用同时学习的心态。首先呢
