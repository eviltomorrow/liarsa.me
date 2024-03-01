---
title: 手动构建 React 项目
catalog: true
date: 2024-03-01 09:25:00
subtitle: ""
tags:
    - React
    - Web
    - 前端
categories:
    - 前端
index_img: https://liarsa-me.oss-cn-beijing.aliyuncs.com/img/logo/the_alchemy_of_finance.jpg
# banner_img: /img/bg/wallhaven-vqdmxm.png
---

# 手动构建 React 项目

## 一、初始化项目

```sh
$ mkdir -p project-react
$ cd project-react
$ npm init -y
$ echo "/node_modules\n/build" >> .gitignore
```

## 二、Webpack 配置

### 2.1 基础配置设置

- 创建文件 <span style="color:#ffbf00">/src/index.js</span> 作为 webpack 的入口文件

```sh
$ mkdir -p src
$ touch src/index.js
```

> index.js 内容
```js
import React, {StrictMode} from 'react';
import { createRoot } from "react-dom/client";

const App = () => (
  <StrictMode>
    <div>
      test page
    </div>
  </StrictMode>
);
const root = createRoot(document.getElementById("root"));
root.render(<App/>);

```

- 创建模板文件 <span style="color:#ffbf00">/public/index.html</span> webpack 打包后的文件将添加到该文件中

```sh
$ mkdir -p public
$ touch public/index.html
```

> index.html 内容
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <div id="root"></div>
</body>
</html>
```

- 创建 webpack 开发环境下配置文件 <span style="color:#ffbf00">/webpack/webpack.config.dev.js</span>

```sh
$ mkdir -p webpack
$ touch webpack/webpack.config.dev.js
```

> webpack.config.dev.js 内容
```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

const htmlWebpackPlugin = new HtmlWebpackPlugin({
  template: path.resolve(__dirname, '../public/index.html'),
});

module.exports = {
  mode: 'development',                              
  entry: path.resolve(__dirname, '../src/index.js'),
  output: {                                         
    path: path.resolve(__dirname, '../build'),      
    filename: 'js/[name].[chunkhash:8].bundle.js',         
  },
  module: {
    rules: [              
      {
        test: /\.(mjs|js|jsx)$/,
        exclude: /node_modules/,
        use: ['babel-loader'],
      }
    ],
  },

  plugins: [
    htmlWebpackPlugin,
  ],

  resolve: {
    extensions: ['.mjs', '.js', '.jsx'],
  },
};
```

- 创建 webpack 生产环境下配置文件 /webpack/webpack.config.js

```sh
$ mkdir -p webpack
$ touch webpack/webpack.config.js
```

> webpack.config.js 内容
```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

const htmlWebpackPlugin = new HtmlWebpackPlugin({
  template: path.resolve(__dirname, '../public/index.html'),
});

module.exports = {
  mode: 'production',  // 和开发环境下的配置只是修改了 mode                            
  entry: path.resolve(__dirname, '../src/index.js'),
  output: {                                         
    path: path.resolve(__dirname, '../build'),      
    filename: 'js/[name].[chunkhash:8].bundle.js',         
  },
  module: {
    rules: [              
      {
        test: /\.(mjs|js|jsx)$/,
        exclude: /node_modules/,
        use: ['babel-loader'],
      }
    ],
  },

  plugins: [
    htmlWebpackPlugin,
  ],

  resolve: {
    extensions: ['.mjs', '.js', '.jsx'],
  },
};
```

- 创建 babel 配置文件 .babelrc

```sh
$ touch .babelrc
```

> .babelrc 内容
```json
{
  "presets": [
    "@babel/preset-react",
    "@babel/preset-env"
  ]
}
```

- 修改 package.json: 添加 npm 脚本

```sh
$ vim package.json
```

```json
"scripts": {
   "start": "webpack-dev-server --config ./webpack/webpack.config.dev.js --open",
   "build": "rm -rf build/* && webpack --config ./webpack/webpack.config.js"
}
```

### 2.2 安装基础插件包

- webpack 相关依赖包、插件

    webpack:  webpack 基础包
    webpack-cli: webpack cli 工具包
    html-webpack-plugin: webpack 插件, 用于将打包后的文件添加到指定的 html 内
    webpack-dev-server: webpack 开发环境工具, 创建一个开发环境
    babel-loader: weboack loader, 用于编译打包 js 文件
    @babel/core: babel 依赖包, 将 js 代码分析成 ast
    @babel/preset-react: webpack react 相关预设
    @babel/preset-env: weboack react 相关预设, 这样就可以使用最新的 js 相关语法

```sh
$ npm i webpack webpack-cli html-webpack-plugin webpack-dev-server babel-loader @babel/core @babel/preset-react @babel/preset-env --save-dev
```

- react 相关依赖包、包

```sh
$ npm i react react-dom
```

### 2.3 测试

    1.执行 npm start 测试项目是否能够正常运行
    2.执行 npm run build 测试是否能够正常对项目进行打包、编译, 编译后目录结构如下


<hr/>
<b>参考：</b>
<ul>
    <li>[1] <a href="https://juejin.cn/post/6844904102573375495">纯手动搭建 React 项目 </a></li>
</ul>