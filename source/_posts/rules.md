---
title: 配置react+typescript项目的lint，commit，prettier规范
date: 2022-04-27 15:26:12
categories:
  - FE
tags:
  - eslint
  - prettier
  - FE
  - react
  - typescript
---

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

在根目录创建.prettierrc.js 文件（同理也可以创建.prettierrcignore）.prettierrc.js 中添加如下内容，具体的配置可以看 prettierrc 官网配置项。以下为我比较习惯的 react 项目的配置

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
