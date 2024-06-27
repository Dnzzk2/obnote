---
tags:
  - 前端工程化
  - webpack
---

# 什么是webpack?
本质上，**webpack** 是一个用于现代 JavaScript 应用程序的 _静态模块打包工具_。当 webpack 处理应用程序时，它会在内部从一个或多个入口点构建一个 [依赖图(dependency graph)](https://www.webpackjs.com/concepts/dependency-graph/)，然后将你项目中所需的每一个模块组合成一个或多个 _bundles_，它们均为静态资源，用于展示你的内容。
>_bundles_： 是指将多个独立的模块（可能是JavaScript文件、CSS文件、图片等）按照一定的规则和依赖关系打包在一起后的最终输出文件。
# 核心概念
## 入口(Entry)
入口指示 **webpack** 使用哪个模块来作为构建依赖图的开始。
默认值：`./src/index.js` 可以在webpack config 中配置一个或多个入口
```js
module.exports = {
  entry: './path/to/my/entry/file.js',
};
```
### 单个入口语法
用法：`entry: string | [string]`
扩展性差
```javascript
module.exports = {
  entry: './path/to/my/entry/file.js',
};
```
`entry` 属性的单个入口语法，是以下形式的简写：
```javascript
module.exports = {
  entry: {
    main: './path/to/my/entry/file.js',
  },
};
```
我们也可以将一个文件路径数组传递给 `entry` 属性，这将创建一个所谓的 **"multi-main entry"**。在你想要一次注入多个依赖文件，并且将它们的依赖关系绘制在一个 "chunk" 中时，这种方式就很有用。
```javascript
module.exports = {
  entry: ['./src/file_1.js', './src/file_2.js'],
  output: {
    filename: 'bundle.js',
  },
};
```
### 对象语法
用法：`entry: { <entryChunkName> string | [string] } | {}`
扩展性高
```javascript
module.exports = {
  entry: {
    app: './src/app.js',
    adminApp: './src/adminApp.js',
  },
};
```


## 输出(output)
output指定 **webpack** 输出的 _bundles_ 的输出地址，以及如何命名这些文件
主要输出文件的默认值：`./dist/main.js `  ，其他生成文件默认放置在 `./dist` 文件夹中。
```javascript
const path = require('path');

module.exports = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js',
  },
};
```
## loader
webpack 只能理解 JavaScript 和 JSON 文件，这是 webpack 开箱可用的自带能力。**loader** 让 webpack 能够去处理其他类型的文件，并将它们转换为有效 [模块](https://www.webpackjs.com/concepts/modules)，以供应用程序使用，以及被添加到依赖图中。
