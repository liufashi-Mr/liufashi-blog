---
title: 搭建react脚手架，以及脚手架工具
date: 2022-08-12 19:40:33
categories:
  - react
  - 开发工具
tags:
  - react
  - FE
  - react-cli
---

## 使用指南

[npm地址](https://www.npmjs.com/package/react-client-create)

```shell
# global install
npm i react-client-create -g
# and then run
create-cli create [name]

# or
npm i react-client-create
#then run
npx create-cli create [name]
```

like this

{% gallery %}![选择](https://blog.liufashi.top/img/typescript-h5-template/cli.png){% endgallery %}

选择第一个是常见空的脚手架，可以选择js或者ts（ts还没写完）
选择第二个是react+antd 的中后台管理模板 [详情](https://blog.liufashi.top/2022/06/13/react-antd-admin/)
选择第二个是react+typescript+ant-mobile h5模板（仅提供常见配置） [详情](https://blog.liufashi.top/2022/04/27/typescript-h5-template/)

## 脚手架搭建

项目[源码](https://github.com/liufashi-Mr/react-cli)

webpack配置贴这里，里面有注释解释了各个loader或者插件的作用,也可以参考[webpack优化指南](https://blog.liufashi.top/2022/09/07/webpack-guide/)来详细了解其中的配置

```js
const path = require("path");
const ESLintWebpackPlugin = require("eslint-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const CssMinimizerPlugin = require("css-minimizer-webpack-plugin");
const TerserWebpackPlugin = require("terser-webpack-plugin");
const ImageMinimizerPlugin = require("image-minimizer-webpack-plugin");
const ReactRefreshWebpackPlugin = require("@pmmmwh/react-refresh-webpack-plugin");
const CopyPlugin = require("copy-webpack-plugin");
// 需要通过 cross-env 定义环境变量
const isProduction = process.env.NODE_ENV === "production";

const getStyleLoaders = (preProcessor) => {
  return [
    isProduction ? MiniCssExtractPlugin.loader : "style-loader",
    "css-loader",
    {
      loader: "postcss-loader",
      options: {
        postcssOptions: {
          plugins: [
            "postcss-preset-env", // 能解决大多数样式兼容性问题
          ],
        },
      },
    },
    preProcessor,
  ].filter(Boolean);
};

module.exports = {
  entry: "./src/index.js",
  output: {
    path: isProduction ? path.resolve(__dirname, "../dist") : undefined,
    filename: isProduction
      ? "static/js/[name].[contenthash:10].js"
      : "static/js/[name].js",
    chunkFilename: isProduction
      ? "static/js/[name].[contenthash:10].chunk.js"
      : "static/js/[name].chunk.js",
    assetModuleFilename: "static/js/[hash:10][ext][query]",
    clean: true,
  },
  module: {
    rules: [
      {
        oneOf: [
          {
            // 用来匹配 .css 结尾的文件
            test: /\.css$/,
            // use 数组里面 Loader 执行顺序是从右到左
            use: getStyleLoaders(),
          },
          {
            test: /\.less$/,
            use: getStyleLoaders("less-loader"),
          },
          {
            test: /\.s[ac]ss$/,
            use: getStyleLoaders("sass-loader"),
          },
          {
            test: /\.styl$/,
            use: getStyleLoaders("stylus-loader"),
          },
          {
            test: /\.(png|jpe?g|gif|svg)$/,
            type: "asset",
            parser: {
              dataUrlCondition: {
                maxSize: 10 * 1024, // 小于10kb的图片会被base64处理
              },
            },
          },
          {
            test: /\.(ttf|woff2?)$/,
            type: "asset/resource",
          },
          {
            test: /\.(jsx|js)$/,
            include: path.resolve(__dirname, "../src"),
            loader: "babel-loader",
            options: {
              presets: ["react-app"],
              cacheDirectory: true, // 开启babel编译缓存
              cacheCompression: false, // 缓存文件不要压缩
              plugins: [
                // "@babel/plugin-transform-runtime",  // presets中包含了
                !isProduction && "react-refresh/babel",
              ].filter(Boolean),
            },
          },
        ],
      },
    ],
  },
  plugins: [
    new ESLintWebpackPlugin({
      extensions: [".js", ".jsx"],
      context: path.resolve(__dirname, "../src"),
      exclude: "node_modules",
      cache: true,
      cacheLocation: path.resolve(
        __dirname,
        "../node_modules/.cache/.eslintcache"
      ),
    }),
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, "../public/index.html"),
    }),
    isProduction &&
      new MiniCssExtractPlugin({
        filename: "static/css/[name].[contenthash:10].css",
        chunkFilename: "static/css/[name].[contenthash:10].chunk.css",
      }),
    !isProduction && new ReactRefreshWebpackPlugin(),
    new CopyPlugin({
      patterns: [
        {
          from: path.resolve(__dirname, "../public"),
          to: path.resolve(__dirname, "../dist"),
          toType: "dir",
          noErrorOnMissing: true, // 不生成错误
          globOptions: {
            // 忽略文件
            ignore: ["**/index.html"],
          },
          info: {
            // 跳过terser压缩js
            minimized: true,
          },
        },
      ],
    }),
  ].filter(Boolean),
  optimization: {
    minimize: isProduction,
    // 压缩的操作
    minimizer: [
      // 压缩css
      new CssMinimizerPlugin(),
      // 压缩js
      new TerserWebpackPlugin(),
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
      chunks: "all",
      cacheGroups: {
        react: {
          test: /[\\/]node_modules[\\/]react(.*)?[\\/]/,
          name: "chunk-react",
          priority: 40,
        },
        antd: {
          test: /[\\/]node_modules[\\/]antd[\\/]/,
          name: "chunk-antd",
          priority: 30,
        },
        libs: {
          test: /[\\/]node_modules[\\/]/,
          name: "chunk-libs",
          priority: 20,
        },
      },
    },
    runtimeChunk: {
      name: (entrypoint) => `runtime~${entrypoint.name}`,
    },
  },
  resolve: {
    extensions: [".jsx", ".js", ".json"],
  },
  devServer: {
    host: "localhost",
    port: 3000,
    hot: true,
    compress: true,
    historyApiFallback: true,
    client: {
      overlay: {
        errors: true,
        warnings: false,
      },
    },
  },
  mode: isProduction ? "production" : "development",
  devtool: isProduction ? "source-map" : "cheap-module-source-map",
};

```

## 脚手架工具

项目[源码](https://github.com/liufashi-Mr/react-client-create)

大概步骤就是

1. 创建执行文件，`npm link`链接到本地 方便调试
2. 下载commander，通过一下api实现
    - command：自定义执行的命令
    - option：可选参数
    - alias：用于 执行命令的别名
    - description：命令描述
    - action：执行命令后所执行的方法
    - usage：用户使用提示
    - parse：解析命令行参数，注意这个方法一定要放到最后调用
3. 下载child_process，子进程，在process.exec()中可执行git命令和shell命令
4. 通过inquirer提供可视化选项
5. 使用ora，figlet等等优化细节，美化界面

贴个代码

```js
#!/usr/bin/env node
const program = require("commander");
const { promisify } = require("util");
const figlet = promisify(require("figlet"));
const chalk = require("chalk");
const inquirer = require("inquirer");
const process = require("child_process");
const ora = require("ora");
const spinner = ora("try to get the registry ...");

const getClone = (type, name) => {
  let url;
  switch (type) {
    case "javascript": {
      url = "https://github.com/liufashi-Mr/react-cli.git ";
      break;
    }
    case "typescript": {
      url = "https://github.com/liufashi-Mr/react-cli.git ";
      break;
    }
    case "admin template (with react + ant-design)": {
      url = "https://github.com/liufashi-Mr/react-antd-admin.git ";
      break;
    }
    case "h5 template (with react + ant-mobile)": {
      url = "https://github.com/liufashi-Mr/h5-react-typescript.git ";
      break;
    }
  }
  spinner.start();
  process.exec("git clone " + url + name, (error, stdout, stderr) => {
    if (error !== null) {
      spinner.fail("exec error: " + error);
      console.log(stdout);
      return;
    }
    console.log(stdout);
    process.exec(
     `cd ${name} && rm -rf .git && git init && git add . && git commit -m "init with create-cli"`,
      (error, stdout, stderr) => {
        if (error !== null) {
          spinner.fail("exec error: " + error);
          console.log(stdout);
          return;
        }
        console.log(stdout);
        spinner.succeed("download successfully!!!");
      }
    );
  });
};
program
  .version(require("../package.json").version)
  .command("create <name>")
  .alias("c")
  .action((name) => {
    figlet(`react - client`).then((data) => {
      console.log(chalk.yellow(data));
      inquirer
        .prompt([
          {
            type: "list",
            name: "type",
            message: "which do you want",
            choices: [
              "only react-cli ( you can user-defined )",
              "admin template ( with react + ant-design )",
              "h5 template (with react + ant-mobile )",
            ],
            filter: function (val) {
              return val.toLowerCase();
            },
          },
        ])
        .then(({ type }) => {
          if (type === "only react-cli ( you can user-defined )") {
            inquirer
              .prompt([
                {
                  type: "list",
                  name: "lang",
                  message: "work with javascript or typescript",
                  choices: ["javascript", "typescript"],
                  filter: function (val) {
                    return val.toLowerCase();
                  },
                },
              ])
              .then(({ lang }) => {
                if (lang === "javascript") {
                  getClone(lang, name);
                } else {
                  spinner.fail("还没有写！！！");
                }
              })
              .catch((err) => console.log(err));
          } else {
            getClone(type, name);
          }
        })
        .catch((error) => {
          console.log(error);
        });
    });
  });
program.parse(process.argv);

```

最后发布到npm中[npm地址](https://www.npmjs.com/package/react-client-create)
