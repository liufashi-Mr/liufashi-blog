---
title: 关于我的组件库 rain-ui
date: 2023-02-02 09:51:40
categories:
  - FE
  - react
tags:
  - react
  - FE
  - rain-ui
  - 组件库
---

## 前言

最近两个月来博主在做一件事情，彻底重构了之前搭建的组件库，目前已经初具雏形，[前往](https://rain-ui.liufashi.top/)，在这个过程中，遇到了很多问题，这里分享下过程和解决方式，同时也会将我至今尚未想通的问题提出来，还望指点一二。

## 项目准备

之前的组件库是源于公司的需求，基于[tuya expo](https://expo.tuya.com)的主题和所需要的组件进行定制开发的组件库，需求完成之后让我萌生了写个组件库的想法。

首先，我想写的组件库是一个实用性较大的组件库不是某个项目定制的，因此需要将项目的一些变量暴露出来，便于修改。其次需要支持深色模式以及紧凑模式。可以在项目运行时使用js修改主题色。后续将在主题站点推出主题选项

## 方案选择

- 首先想要在运行时修改主题色，能做到这一点的方案最合适的就是采用 `css variables` 或者是 `css in js`, `css in js`本质上就是通过js写法写的css，存在于js的代码中，优势在于我们可以代码的任何时候修改其中的变量，并且不会影响其他组件，但是过多的js会增加运行时的消耗，而 `css variables` 则是需要通过js操作dom 修改 `css variables` 挂载节点的css变量，当然对象能影响可以忽略不计，而且仅仅只在切换的时候执行不影响项目正常代码的性能。 由于项目的主题没有支持某一个组件单独进行主题修改（antd 5.x 支持），因此这里采用`css variables + less`的方式，更小的使用及更小的性能消耗。
- 色彩适配，例如一个按钮颜色可能是蓝色，hover之后颜色浅一点的蓝色，这个时候用户将`primary-color`修改为橙色，那么按钮hover 的时候也需要是浅一点的橙色，这个时候我们不可能把所有情况的变量都暴露出来供用户消费，因此需要根据主色计算其余颜色。这里采用了 `@ant-design/colors` 的方式，根据颜色计算出对应渐变的是个色阶。
- 深色模式，切换为深色模式之后为了更加的舒适视觉效果，组件的颜色需要调整。这里同样是`@ant-design/colors`，配置深色的背景颜色，根据背景色和原有颜色计算出深色模式下的颜色。

## 组件打包

令我头大的地方来了，首先是工具选择，显然rollup 支持esm cjs umd 等产物的构建，更适合库的打包，并且配置简单。但是了解到webpack5.x 支持了构建esm的能力，出于对老朋友的偏爱最终还是选择了webpack。

### webpack配置

首先常见的 less css 的配置就不多赘述了，可以参考这一篇 {% post_link react-cli %}

```js
const path = require('path');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const TerserWebpackPlugin = require('terser-webpack-plugin');
const CopyWebpackPlugin = require('copy-webpack-plugin');
const CompressionPlugin = require('compression-webpack-plugin');
const getStyleLoaders = (preProcessor) => {
  return [
    MiniCssExtractPlugin.loader,
    'css-loader',
    {
      loader: 'postcss-loader',
      options: {
        postcssOptions: {
          plugins: ['postcss-preset-env'],
        },
      },
    },
    preProcessor === 'less-loader'
      ? {
          loader: 'less-loader',
          options: {
            lessOptions: {
              javascriptEnabled: true,
            },
          },
        }
      : preProcessor,
  ].filter(Boolean);
};
const baseConfig = {
  entry: './src/index.ts',
  module: {
    rules: [
      {
        oneOf: [
          {
            test: /\.css$/,
            use: getStyleLoaders(),
          },
          {
            test: /\.less$/,
            use: getStyleLoaders('less-loader'),
          },
          {
            test: /\.(js|jsx)$/,
            use: [
              {
                loader: 'babel-loader',
                options: {
                  cacheDirectory: true,
                  cacheCompression: false,
                  plugins: [['@babel/plugin-transform-runtime']],
                  presets: [['@babel/preset-env']],
                  ignore: ['node_modules/**'],
                },
              },
            ],
          },
          {
            test: /\.(ts|tsx)$/,
            use: 'ts-loader',
          },
        ],
      },
    ],
  },
  plugins: [
    // 同事生成gzip的产物，在大部分情况下提供更小的体积
    new CompressionPlugin({
      algorithm: 'gzip',
      test: /\.(js|css|json|txt|html|ico|svg)(\?.*)?$/i,
      threshold: 10240,
      minRatio: 0.8,
      deleteOriginalAssets: false,
    }),
  ],
  optimization: {
    minimize: true,
    // 压缩的操作
    minimizer: [
      // 压缩css
      new MiniCssExtractPlugin({
        // 这里的目录在dist/[mode] 所以用 ../ 提出来
        filename: '../style/rain-ui.min.css',
      }),
      // 压缩js
      new TerserWebpackPlugin(),
    ],
  },
  resolve: {
    extensions: ['.tsx', '.ts', '.json', '.js', '.jsx'],
  },
  mode: 'production',
  devtool: 'source-map',
};
```

### umd

首先是打包生成 `umd` 的产物，`umd`的产物支持多种模块以及`script` 标签引入，体积稍大，通常作为备选的方案。在以上的baseConfig中添加 `output`，设置`libraryTarget:umd`

```js
module.exports ={
    output: {
      path: path.resolve(__dirname, './dist/umd'),
      filename: 'index.js',
      libraryTarget: 'umd',
    },
    ...baseConfig
}
```

### cjs

然后是 `commonjs` ，相比之下拥有更小的体积，在webpack不支持ES Module的日子里常常作为首选。

```js
module.exports ={
    output: {
      path: path.resolve(__dirname, './dist/cjs'),
      filename: 'index.js',
      libraryTarget: 'commonjs',
    },
    ...baseConfig
}
```

### esm

最后是新宠`ES Module`,天然支持依赖分离，支持`Tree-shaking`,基于 ESM 的能力，模块化交给浏览器端，不存在资源重复加载问题，这也是为什么基于es 的构建工具会比webpack快得多。
webpack5 构建esm产物的时候需要设置`experiments.outputModule=true` 。

```js
module.exports ={
    output: {
      path: path.resolve(__dirname, './dist/esm'),
      filename: 'index.js',
      libraryTarget: 'module',
      chunkFormat: 'module',
    },
    experiments: {
      outputModule: true,
    },
}
```

### 配置合并

webpack5 支持 导出一个数组，然后给数组每一项添加 `name` 最后采用 `webpack --config-name esm` 的方式执行。修改代码如下

``` js
module.exports = [
  {
    name: 'esm',
    output: {
      path: path.resolve(__dirname, './dist/esm'),
      filename: 'index.js',
      libraryTarget: 'module',
      chunkFormat: 'module',
    },
    experiments: {
      outputModule: true,
    },
    ...baseConfig,
  },
  {
    name: 'cjs',
    output: {
      path: path.resolve(__dirname, './dist/cjs'),
      filename: 'index.js',
      libraryTarget: 'commonjs',
    },
  },
  {
    name: 'umd',
    output: {
      path: path.resolve(__dirname, './dist/umd'),
      filename: 'index.js',
      libraryTarget: 'umd',
    },

    ...baseConfig,
  },
];
```

`package.json` 添加命令

```json
{
    "build": "rm -rf ./dist &&  npm run build:esm & npm run build:umd & npm run build:cjs",
    "build:esm": "webpack --config-name esm",
    "build:cjs": "webpack --config-name cjs",
    "build:umd": "webpack --config-name umd",
}
```

### 问题

在开发一切顺利的时候将包打包好发布到npm之后，在项目中引用报错 `Minified React error #321;`
![报错](https://blog.liufashi.top/img/rain-ui/1.png)

>Invalid hook call. Hooks can only be called inside of the body of a function component. This could happen for one of the following reasons: 1. You might have mismatching versions of React and the renderer (such as React DOM) 2. You might be breaking the Rules of Hooks 3. You might have more than one copy of React in the same app See <https://reactjs.org/link/invalid-hook-call> for tips about how to debug and fix this problem.

根据官网的提示 ：

1. 不匹配的react和react-dom版本
2. 违反了hooks规则
3. 应用程序中拥有多个react副本

根据提示就想到需要使用externals将react和react-dom抽离出来，添加如下代码:

```js
{ 
    "externals": {
        "react": {
        commonjs: 'react',
        commonjs2: 'react',
        module: 'react',
        amd: 'react',
        root: 'React',
        },
        'react-dom': {
        commonjs: 'react-dom',
        commonjs2: 'react-dom',
        module: 'react-dom',
        amd: 'react-dom',
        root: 'ReactDOM',
        },
  },
}
```

问题到这里就解决了。

### 聊一聊组件库中 package.json 需要的修改

`main`: 引用文件的入口，通常是作为备选，一般设置为umd的构建产物。
`types/typing`: 类型文件，在ts中使用库时会在`types/typing`指定的目录找类型文件。
`module`: 在ES Module 引用项目的时候指定的路径，优先级高于main 设置为库构建的esm产物的目录。
`unpkg`: CDN方式下，引入当前npm包的链接。

另外作为一个react的组件库，其宿主环境肯定是有`react` 和 `react-dom`的，因此我们的组件库不需要安装这两个依赖，
package.json 设置

```json
  "peerDependencies": {
    "react": ">=16.14.0",
    "react-dom": ">=16.14.0"
  },
```

## 问题

问题1: 如果组件库的代码只会在es module项目中被引用，通常项目中都会有`@babel/preset-env`,`core-js` 对js版本和一些方法做兼容，所以这些处理在组件库打包时是必要的吗？

问题2：项目打包之后在项目中可以正常使用，在`stackblitz`中也是正常使用，但是在`codesandbox`中会报错`Cannot read properties of undefined (reading 'forwardRef')`, 但是手动引用 umd 或者cjs 模块的时候是没有问题的，只有esm会有问题，这个是什么原因呢，困扰好多天了。 [地址](https://codesandbox.io/s/bzotei)。

问题3：组件库打包生成cjs的意义在哪里 我看`antd`等等都构建了该产物。仅仅是因为体积小于 umd吗

## 总结

还望有大佬解答疑问，如果上述方案有缺陷或者是有优化的地方也希望不吝指教。

最后最后，欢迎小伙伴[加入](https://rain-ui.liufashi.top/)👏👏👏
