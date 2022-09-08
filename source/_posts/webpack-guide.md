---
title: webpack优化指南
date: 2022-09-07 19:55:24
categories:
  - FE
tags:
  - webpack
  - FE
---

## 开发体验优化

### sourceMap

是一个用来生成源代码与构建后代码一一映射的文件的方案。简单的来说 sourceMap 会生成一个 xxx.map 文件，就是在我们写错代码或者代码发生错误的时候能够准确提示我们代码是在那个文件哪一行，哪一列。sourceMap 的值有很多种情况，但是我们通常只需要关注`cheap-module-source-map`和`source-map`，前者打包编译速度快，但是只包含行的映射，一般用在开发的时候使用，因为我们开发需要较快的编译速度，而且我们编写的代码都是有格式规范的一般也不会出现很难找的情况。后者则是会包含行和列的映射，在生产环境使用，因为生产环境的代码通常会压缩。
webpack5 已经内置,使用时只需要在最外层添加上 devtool 即可，

```js
module.exports = {
  // 其他省略
  mode: "development",
  devtool: "cheap-module-source-map",
  // mode: "production",
  // devtool: "source-map",
};
```

## 提升打包构建速度

### HMR 热模替换

在开发的时候，我们的项目在开发的时候通常是将代码打包然后在 devServer 上运行，当我们修改其中某个模块代码之后 webpack 会将所有的模块全部重新打包编译，项目较大时打包速度很慢。
这个时候我们使用 HotModuleReplacement,是的项目在某个模块发生改动之后不比全部的模块都重新编译打包。
我们的 css 会经过 style-loader 处理，已经是具有 HMR 的效果了
js 的添加 HMR 功能

```js
//入口文件 main.js
// 假设引入的两个js文件
import count from "./js/count";
import sum from "./js/sum";

const result1 = count(2, 1);
console.log(result1);
const result2 = sum(1, 2, 3, 4);
console.log(result2);

// 判断是否支持HMR功能
if (module.hot) {
  // 当./js/count.js文件发生变化后只加载这个文件
  module.hot.accept("./js/count.js", function (count) {
    const result1 = count(2, 1);
    console.log(result1);
  });
  // 同上
  module.hot.accept("./js/sum.js", function (sum) {
    const result2 = sum(1, 2, 3, 4);
    console.log(result2);
  });
}
```

这样其实还是非常麻烦，我们通常在项目开发中会借助对应的比如create-react-app中使用的babel-plugin`react-refresh/babel`。在后面[搭建react脚手架](https://blog.liufashi.top/2022/09/08/react-cli//#%E8%84%9A%E6%89%8B%E6%9E%B6%E6%90%AD%E5%BB%BA)的时候会用到

### 使用 oneOf

当我们配置了`css-loader`,`less-loader`,`sass-loader`等等

```js
module.exports = {
  module: {
    rules: [
      {
        // 用来匹配 .css 结尾的文件
        test: /\.css$/,
        // use 数组里面 Loader 执行顺序是从右到左
        use: ["style-loader", "css-loader"],
      },
      {
        test: /\.less$/,
        use: ["style-loader", "css-loader", "less-loader"],
      },
      {
        test: /\.s[ac]ss$/,
        use: ["style-loader", "css-loader", "sass-loader"],
      },
      {
        test: /\.styl$/,
        use: ["style-loader", "css-loader", "stylus-loader"],
      },
    ],
  },
};
```

此时我们的文件都会经过 loader 的处理，即使不满足正则的条件也会去过一遍，判断一下。这个时候我们使用`oneOf`可以在满足一个 loader 之后直接跳出，像下面这样使用即可

```js
module.exports = {
  module: {
    rules: [
      {
        oneOf: [
          {
            // 用来匹配 .css 结尾的文件
            test: /\.css$/,
            // use 数组里面 Loader 执行顺序是从右到左
            use: ["style-loader", "css-loader"],
          },
          {
            test: /\.less$/,
            use: ["style-loader", "css-loader", "less-loader"],
          },
          {
            test: /\.s[ac]ss$/,
            use: ["style-loader", "css-loader", "sass-loader"],
          },
          {
            test: /\.styl$/,
            use: ["style-loader", "css-loader", "stylus-loader"],
          },
        ],
      },
    ],
  },
};
```

### include 和 exclude

字面意思包含和不包含，也很容易理解因为他们不会存在交集的情况,因此不能同时使用。
在某个 loader 或者 plugins 中使用，例如：

```js
module.exports = {
  module: {
    rules: [
      {
        oneOf: [
          {
            // 用来匹配 .css 结尾的文件
            test: /\.css$/,
            // use 数组里面 Loader 执行顺序是从右到左
            use: ["style-loader", "css-loader"],
            include: path.resolve(__dirname, "../src"), // 只处理src下的
          },
        ],
      },
    ],
  },
  plugins: [
    new ESLintWebpackPlugin({
      // 指定检查文件的根目录
      context: path.resolve(__dirname, "../src"),
      exclude: "node_modules", // 不处理node_modules中的，也是默认值
    }),
  ],
};
```

### cache

还是字面意思，缓存。我们的项目一般会使用 eslint 检查和 babel 编译,我们可以对上一次的 eslint 检查结果和 babel 编译结果进行缓存,加快第二次打包的速度。
我们只需要在`babel-loader`添加配置，开启缓存即可,eslint 也是类似

```js
module.exports = {
  module: {
    rules: [
      {
        oneOf: [
          {
            test: /\.js$/,
            loader: "babel-loader",
            options: {
              cacheDirectory: true, // 开启babel编译缓存
              cacheCompression: false, // 缓存文件不要压缩,因为这个缓存文件不需要打包，压缩反而会减慢构建速度
            },
          },
        ],
      },
    ],
  },
  plugins: [
    new ESLintWebpackPlugin({
      // 指定检查文件的根目录
      context: path.resolve(__dirname, "../src"),
      exclude: "node_modules", // 默认值
      cache: true, // 开启缓存
      // 缓存目录
      cacheLocation: path.resolve(
        __dirname,
        "../node_modules/.cache/.eslintcache"
      ),
    }),
  ],
};
```

### thead

当项目越来越大的时候我们可以开启多进程同时打包，下载包`npm i thread-loader -D`，我们项目中一般只需要对 js 进行处理，因为其他文件都是比较少的, 我们可以在`babel-loader`处理完之后采用`thread-loader`开启多线程打包.

```js
// nodejs核心模块，直接使用
const os = require("os");
// cpu核数
const threads = os.cpus().length;
const TerserPlugin = require("terser-webpack-plugin");

module.exports = {
  module: {
    rules: [
      {
        oneOf: [
          //   ...省略很多配置
          {
            test: /\.js$/,
            // exclude: /node_modules/, // 排除node_modules代码不编译
            include: path.resolve(__dirname, "../src"), // 也可以用包含
            use: [
              {
                loader: "thread-loader", // 开启多进程
                options: {
                  workers: threads, // 数量
                },
              },
              {
                loader: "babel-loader",
                options: {
                  cacheDirectory: true,
                  cacheCompression: false,
                },
              },
            ],
          },
        ],
      },
    ],
  },
  // webpack5 推荐将所有压缩的操作放在optimization里面
  optimization: {
    minimize: true,
    minimizer: [
      new CssMinimizerPlugin(),
      // 当生产模式会默认开启TerserPlugin，但是我们需要进行其他配置，就要重新写
      new TerserPlugin({
        parallel: threads, // 开启多进程
      }),
    ],
  },
};
```

## 优化打包体积

### Tree Shaking

`Tree Shaking` 是一个术语，通常用于描述移除 JavaScript 中的没有使用上的代码。依赖于`ES Module`
我们在项目开发的时候必定会对第三方的库进行使用，如果没有特殊处理的话我们打包时会引入整个库，但是实际上可能我们可能只用上极小部分的功能。
这样打包体积就大了。
其实 webpack 已经默认开启了这个功能。
webpack 的 tree-shaking 和其他的打包工具相比有一个特殊的地方，那就是 sideEffects，在 package.json 中添加这个配置可以指定你当前的模块是否存在副作用。在一个纯粹的 ESM 模块世界中，很容易识别出哪些文件有副作用，然而，我们的项目无法达到这种纯度，所以需要我们去手动指定哪些模块是是否存在副作用。

假设我们有以下配置和文件，webpack config 就是一个最简单的生产环境配置，index.js 引用了 module 里的 random 方法。

```js
// webpack.config.js
const path = require("path");

module.exports = {
  entry: "./src/index.js",
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname, "dist"),
  },
  mode: "production",
};

// src/index.js
import { random } from "./module";
const res = random();
console.log("res: ", res);

// src/module.js
export function random() {
  return Math.random();
}
export function add(a, b) {
  return a + b;
}
```

按照期望我们的打包产物里应该只有 random 方法的调用，让我们看下打包的结果。

```js
(() => {
  "use strict";
  const o = Math.random();
  console.log("res: ", o);
})();
```

这个结果完全符合我们的期望，但是如果我们的模块中存在有副作用的代码会有什么影响呢？我们在 module.js 中添加一个全局变量。

```js
// src/module.js
export function random() {
  return Math.random();
}
export function add(a, b) {
  return a + b;
}
// 有副作用的代码
window.test = true;
```

我们再看一下打包后的代码，可以发现 window.test 被打包到了最后的产物中，这是完全合理的，因为通过 ESM 的静态分析是无法判断你是否使用了 test 这个变量，为了保证程序最终的运行不会出现问题，webpack 只能将这段有副作用的代码打包进最终的产物。

```js
(() => {
  "use strict";
  window.test = !0;
  const o = Math.random();
  console.log("res: ", o);
})();
```

当我们在 package.json 中添加 sideEffects 为 false,在最外层加上`"sideEffects": false`

我们发现最终的产物里还是存在 window.test。
是文档里说错了吗？其实是我们理解的有偏差，sideEffects 是可以指定我们项目中所有的模块都是纯净的，但是 webpack 删除有副作用模块的前提是我们没有使用这个模块里的任何方法或者变量。我们在 index.js 里使用到了 module.js 里的 random 方法，所以 webpack 就不能删除 module 这个模块里的副作用代码。
让我们再添加一个 module2.js。

```js
// src/module2.js
export const value = 1;
// 有副作用
window.value = 1;
```

然后在 module.js 中引入 value 并且导出，index.js 保持不变，仍然只引用 random 方法。

```js
// src/module.js
import { value } from "./module2";

export function random() {
  return Math.random();
}
export function add(a, b) {
  return a + b;
}
export { value };
window.test = true;
```

再让我们看下现在的打包结果，我们发现在 module2.js 中声明的 window.value 没有出现在最终的打包产物里，因为自始至终 module2 除了被 module 导出以外，没有任何地方使用到 module2 里的任何变量或者方法，这时候我们通过 sideEffects 指定所有的模块都是纯净的，webpack 就会直接把 module2 整个模块过滤掉，而不再去解析内部的代码。

```js
(() => {
  "use strict";
  window.test = !0;
  const o = Math.random();
  console.log("res: ", o);
})();
```

如果我们删除 package.json 中的 `"sideEffects": false`我们会发现 window.value 出现在了最终的产物里。

```js
(() => {
  "use strict";
  (window.value = 1), (window.test = !0);
  const o = Math.random();
  console.log("res: ", o);
})();
```

到这里我们应该可以理解 sideEffects 的作用了，他并不是无脑帮 webpack 删除有副作用的代码，而是在确定了某个模块里没有任何内容被使用的时将这个模块在打包流程中过滤掉。

### babel runtime

Babel 为编译的每个文件都插入了辅助代码，使代码体积过大！可以将这些辅助代码作为一个独立模块，来避免重复引入。
`@babel/plugin-transform-runtime` 禁用了 Babel 自动对每个文件的 runtime 注入，而是引入 `@babel/plugin-transform-runtime` 并且使所有辅助代码从这里引用。~~~~

下载包`npm i @babel/plugin-transform-runtime -D`,在`babel-loader` ~~options 中添加插件

```js
module.exports = {
  module: {
    rules: [
      {
        oneOf: [
          //   ...省略很多配置
          {
            test: /\.js$/,
            // exclude: /node_modules/, // 排除node_modules代码不编译
            include: path.resolve(__dirname, "../src"), // 也可以用包含
            use: [
              {
                loader: "babel-loader",
                options: {
                  cacheDirectory: true,
                  cacheCompression: false,
                  plugins: ["@babel/plugin-transform-runtime"], // 减少代码体积
                },
              },
            ],
          },
        ],
      },
    ],
  },
};
```

### image Minimizer

图片压缩
下载包

```shell
npm i image-minimizer-webpack-plugin imagemin -D
# 无损压缩下这些
npm install imagemin-gifsicle imagemin-jpegtran imagemin-optipng imagemin-svgo -D

#有损下这些
npm install imagemin-gifsicle imagemin-mozjpeg imagemin-pngquant imagemin-svgo -D

```

无损压缩程度相对有损低，但是不会对图片质量有很大影响

```js
const ImageMinimizerPlugin = require("image-minimizer-webpack-plugin");
module.export = {
  optimization: {
    minimizer: [
      // css压缩也可以写到optimization.minimizer里面，效果一样的
      new CssMinimizerPlugin(),
      // 当生产模式会默认开启TerserPlugin，但是我们需要进行其他配置，就要重新写了
      new TerserPlugin({
        parallel: threads, // 开启多进程
      }),
      // 压缩图片
      new ImageMinimizerPlugin({
        minimizer: {
          implementation: ImageMinimizerPlugin.imageminGenerate,
          options: {
            plugins: [
              ["gifsicle", { interlaced: true }],
              ["jpegtran", { progressive: true }],
              ["optipng", { optimizationLevel: 5 }],
              [
                "svgo",
                {
                  plugins: [
                    "preset-default",
                    "prefixIds",
                    {
                      name: "sortAttrs",
                      params: {
                        xmlnsOrder: "alphabetical",
                      },
                    },
                  ],
                },
              ],
            ],
          },
        },
      }),
    ],
  },
};
```

## 优化代码运行性能

### Code Split

打包代码时会将所有 js 文件打包到一个文件中，体积太大了。我们如果只要渲染首页，就应该只加载首页的 js 文件，其他文件不应该加载。

所以我们需要将打包生成的文件进行代码分割，生成多个 js 文件，渲染哪个页面就只加载某个 js 文件，这样加载的资源就少，速度就更快。

一般会采用多入口或者动态导入

#### 多入口

在这之前，我们假设有一个简单的 webpack 配置和两个 js 文件

```js
// webpack.config.js
const path = require("path");

module.exports = {
  entry: "./src/index.js",
  mode: "development",

  output: {
    filename: "main.js",
    path: path.resolve(__dirname, "dist"),
  },
};

// src/index.js
import _ from "lodash";
import "./module";

console.log(_.join(["Another", "module", "loaded!"], " "));

// src/module.js
import _ from "lodash";

console.log(_.join(["Another", "module", "loaded!"], " "));
```

上述的配置会通过 /src/index.js 作为入口，收集所有的依赖并且打包在一个 main.js 的文件中。

使用多入口

```js
const path = require("path");

module.exports = {
  mode: "development",
  entry: {
    index: "./src/index.js",
    another: "./src/module.js",
  },
  output: {
    filename: "[name].bundle.js",
    path: path.resolve(__dirname, "dist"),
  },
};
```

现在就会生成两个 js 文件了，但是存在很严重的问题，就是生成的 js 中都包含了 lodash，lodash 被打包了两次。所以我们需要指定公共依赖

```js
const path = require("path");

module.exports = {
  mode: "development",
  entry: {
    index: {
      import: "./src/index.js",
      // 关键代码
      dependOn: "shared",
    },
    another: {
      import: "./src/module.js",
      // 关键代码
      dependOn: "shared",
    },
    // 关键代码
    shared: "lodash",
  },
  output: {
    filename: "[name].bundle.js",
    path: path.resolve(__dirname, "dist"),
  },
};
```

还有一个问题就是手动指定过于繁琐，我们可以使用插件`SplitChunksPlugin`

```js
const path = require("path");

module.exports = {
  mode: "development",
  entry: {
    index: "./src/index.js",
    another: "./src/module.js",
  },
  output: {
    filename: "[name].bundle.js",
    path: path.resolve(__dirname, "dist"),
  },
  // 关键代码
  optimization: {
    splitChunks: {
      chunks: "all",
      // 以下是 all 的默认值,可以在下面进行覆盖
      // minSize: 20000, // 分割代码最小的大小
      // minRemainingSize: 0, // 类似于minSize，最后确保提取的文件大小不能为0
      // minChunks: 1, // 至少被引用的次数，满足条件才会代码分割
      // maxAsyncRequests: 30, // 按需加载时并行加载的文件的最大数量
      // maxInitialRequests: 30, // 入口js文件最大并行请求数量
      // enforceSizeThreshold: 50000, // 超过50kb一定会单独打包（此时会忽略minRemainingSize、maxAsyncRequests、maxInitialRequests）
      // cacheGroups: { // 组，哪些模块要打包到一个组
      //   defaultVendors: { // 组名
      //     test: /[\\/]node_modules[\\/]/, // 需要打包到一起的模块
      //     priority: -10, // 权重（越大越高）
      //     reuseExistingChunk: true, // 如果当前 chunk 包含已从主 bundle 中拆分出的模块，则它将被重用，而不是生成新的模块
      //   },
      //   default: { // 其他没有写的配置会使用上面的默认值
      //     minChunks: 2, // 这里的minChunks权重更大
      //     priority: -20,
      //     reuseExistingChunk: true,
      //   },
      // },
    },
  },
};
```

#### 动态导入

当涉及到动态代码拆分时，webpack 提供了两个类似的技术。第一种，也是推荐选择的方式是，使用符合 ECMAScript 提案 的 import() 语法 来实现动态导入。第二种，则是 webpack 的遗留功能，使用 webpack 特定的 require.ensure，这里我们只关心第一种。

让我们将配置恢复原来的模样。

```js
// webpack.config.js
const path = require("path");

module.exports = {
  entry: "./src/index.js",
  mode: "development",

  output: {
    filename: "main.js",
    path: path.resolve(__dirname, "dist"),
  },
};
```

修改一下 src/index.js 引入其他模块的方式。

```js
import("lodash").then(({ default: _ }) => {
  console.log(_.join(["Another", "module", "loaded!"], " "));
});

import("./module");
```

而且与多入口的方式不同，多入口的打包方式需要将拆分后的 js 都引入的 html 中，但是动态引入我们只需要引入 main.js 即可，原因是 main.js 会动态创建两个 script 标签去引入拆分的 bundle。

我们在 vue/react 使用路由懒加载的时候会使用动态导入，此时代码就会被拆分为多个文件[路由懒加载](https://blog.liufashi.top/2022/04/12/qian-duan-xiang-mu-da-bao-ti-ji-you-hua/#%E8%B7%AF%E7%94%B1%E6%87%92%E5%8A%A0%E8%BD%BD)

### preload prefetch

我们想在浏览器空闲时间，加载后续需要使用的资源。我们就需要用上 Preload 或 Prefetch 技术

- Preload：告诉浏览器立即加载资源。
- Prefetch：告诉浏览器在空闲时才开始加载资源。

他们只会加载并不会执行，Preload 加载优先级高，Prefetch 加载优先级低。Preload 只能加载当前页面需要使用的资源，Prefetch 可以加载当前页面资源，也可以加载下一个页面需要使用的资源。
但是这个兼容性比较差，大概使用：使用这个插件`@vue/preload-webpack-plugin`，在 plugins 中使用

```js
const PreloadWebpackPlugin = require("@vue/preload-webpack-plugin");

module.exports = {
  plugins: [
    new PreloadWebpackPlugin({
      rel: "preload", // preload兼容性更好
      as: "script",
      // rel: 'prefetch' // prefetch兼容性更差
    }),
  ],
};
```

### Network Cache

这个也算不上优化，将来开发时我们对静态资源会使用缓存来优化，这样浏览器第二次请求资源就能读取缓存了，速度很快。但是这样的话就会有一个问题, 因为前后输出的文件名是一样的，都叫 main.js，一旦将来发布新版本，因为文件名没有变化导致浏览器会直接读取缓存，不会加载新资源，项目也就没法更新了。
所以我们从文件名入手，确保更新前后文件名不一样，这样就可以做缓存了。

```js
module.exports = {
  entry: "./src/main.js",
  output: {
    path: path.resolve(__dirname, "../dist"), // 生产模式需要输出
    // [contenthash:8]使用contenthash，取8位长度
    filename: "static/js/[name].[contenthash:8].js", // 入口文件打包输出资源命名方式
    chunkFilename: "static/js/[name].[contenthash:8].chunk.js", // 动态导入输出资源命名方式
    assetModuleFilename: "static/media/[name].[hash][ext]", // 图片、字体等资源命名方式（注意用hash）
    clean: true,
  },
  plugins: [
    // css也别忘了哦
    new MiniCssExtractPlugin({
      // 定义输出文件名和目录
      filename: "static/css/[name].[contenthash:8].css",
      chunkFilename: "static/css/[name].[contenthash:8].chunk.css",
    }),
  ],
};
```

- 问题：
  当我们修改 xxx.js 文件再重新打包的时候，因为 contenthash 原因，xxx.js 文件 hash 值发生了变化（这是正常的）。

但是 main.js 文件的 hash 值也发生了变化，这会导致 main.js 的缓存失效。明明我们只修改 xxx.js, 为什么 main.js 也会变身变化呢？

- 原因：

更新前：xxx.[contenthash:8].js, main.js 引用的 xxx.[contenthash:8].js
更新后：main.引用的 xxx.[contenthash:8].js 的[contenthash:8]发生改变，导致 main.js 缓存失效

- 解决：

将 hash 值单独保管在一个 runtime 文件中。

我们最终输出三个文件：main、math、runtime。当 math 文件发送变化，变化的是 math 和 runtime 文件，main 不变。

runtime 文件只保存文件的 hash 值和它们与文件关系，整个文件体积就比较小，所以变化重新请求的代价也小。

```js
module.exports = {
  entry: "./src/main.js",
  output: {
    path: path.resolve(__dirname, "../dist"), // 生产模式需要输出
    // [contenthash:8]使用contenthash，取8位长度
    filename: "static/js/[name].[contenthash:8].js", // 入口文件打包输出资源命名方式
    chunkFilename: "static/js/[name].[contenthash:8].chunk.js", // 动态导入输出资源命名方式
    assetModuleFilename: "static/media/[name].[hash][ext]", // 图片、字体等资源命名方式（注意用hash）
    clean: true,
  },
  plugins: [
    // css也别忘了哦
    new MiniCssExtractPlugin({
      // 定义输出文件名和目录
      filename: "static/css/[name].[contenthash:8].css",
      chunkFilename: "static/css/[name].[contenthash:8].chunk.css",
    }),
  ],

  optimization: {
    minimizer: [
      // css压缩也可以写到optimization.minimizer里面，效果一样的
      new CssMinimizerPlugin(),
      // 当生产模式会默认开启TerserPlugin，但是我们需要进行其他配置，就要重新写了
      new TerserPlugin({
        parallel: threads, // 开启多进程
      }),
      // 压缩图片
      new ImageMinimizerPlugin({
        minimizer: {
          implementation: ImageMinimizerPlugin.imageminGenerate,
          options: {
            plugins: [
              ["gifsicle", { interlaced: true }],
              ["jpegtran", { progressive: true }],
              ["optipng", { optimizationLevel: 5 }],
              [
                "svgo",
                {
                  plugins: [
                    "preset-default",
                    "prefixIds",
                    {
                      name: "sortAttrs",
                      params: {
                        xmlnsOrder: "alphabetical",
                      },
                    },
                  ],
                },
              ],
            ],
          },
        },
      }),
    ],
    // 代码分割配置
    splitChunks: {
      chunks: "all", // 对所有模块都进行分割
      // 其他内容用默认配置即可
    },
    // 提取runtime文件
    runtimeChunk: {
      name: (entrypoint) => `runtime~${entrypoint.name}`, // runtime文件命名规则
    },
  },
};
```

### core-js

过去我们使用 babel 对 js 代码进行了兼容性处理，其中使用@babel/preset-env 智能预设来处理兼容性问题。

它能将 ES6 的一些语法进行编译转换，比如箭头函数、点点点运算符等。但是如果是 async 函数、promise 对象、数组的一些方法（includes）等，它没办法处理。

所以此时我们 js 代码仍然存在兼容性问题，一旦遇到低版本浏览器会直接报错。所以我们想要将 js 兼容性问题彻底解决
core-js 是专门用来做 ES6 以及以上 API 的 polyfill。

polyfill 翻译过来叫做垫片/补丁。就是用社区上提供的一段代码，让我们在不兼容某些新特性的浏览器上，使用该新特性。

下载包`npm i core-js`,
babel.config.js

```js
module.exports = {
  // 智能预设：能够编译ES6语法
  presets: [
    [
      "@babel/preset-env",
      // 按需加载core-js的polyfill
      { useBuiltIns: "usage", corejs: { version: "3", proposals: true } },
    ],
  ],
};
```
