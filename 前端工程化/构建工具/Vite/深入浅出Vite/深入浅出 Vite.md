**无论工具层面如何更新，它们解决的核心问题，即前端工程的痛点是不变的**。
# 痛点

1. **模块化需求**，前端模块标准特别多，AMD，CMD，ESM，Commonjs等，前端工程一方面需要落实这些模块规范，保证模块正常加载。另一方面需要兼容不同的模块规范，以适应不同的执行环境。
2. **兼容浏览器，编译高级语法**,由于浏览器的实现规范所限，只要高级语言/语法（TypeScript、 JSX 等）想要在浏览器中正常运行，就必须被转化为浏览器可以理解的形式。
3. **代码质量问题**，和开发阶段的考虑侧重点不同，生产环境中，我们不仅要考虑代码的`安全性`、`兼容性`问题，保证线上代码的正常运行，也需要考虑代码运行时的性能问题。由于浏览器的版本众多，代码兼容性和安全策略各不相同，线上代码的质量问题也将是前端工程中长期存在的一个痛点。
4. **开发效率**，**项目的冷启动/二次启动时间**、**热更新时间**都可能严重影响开发效率，尤其是当项目越来越庞大的时候。因此，提高项目的启动速度和热更新速度也是前端工程的重要需求。

# 痛点解决方案
- 模块化方面，提供模块加载方案，并兼容不同的模块规范。
    
- 语法转译方面，配合 `Sass`、`TSC`、`Babel` 等前端工具链，完成高级语法的转译功能，同时对于静态资源也能进行处理，使之能作为一个模块正常加载。
    
- 产物质量方面，在生产环境中，配合 `Terser`等压缩工具进行代码压缩和混淆，通过 `Tree Shaking` 删除未使用的代码，提供对于低版本浏览器的语法降级处理等等。
    
- 开发效率方面，构建工具本身通过各种方式来进行性能优化，包括`使用原生语言 Go/Rust`、`no-bundle`等等思路，提高项目的启动性能和热更新的速度。

# 模块标准
## CommonJS 规范
CommonJS 是业界最早正式提出的 JavaScript 模块规范，主要用于服务端，随着 Node.js 越来越普及，这个规范也被业界广泛应用。对于模块规范而言，一般会包含 2 方面内容:

- 统一的模块化代码规范
    
- 实现自动加载模块的加载器(也称`loader`)
    

对于 CommonJS 模块规范本身，相信有 Node.js 使用经验的同学都不陌生了，为了方便你理解，我举一个使用 CommonJS 的简单例子:
```ts
// module-a.js
var data = "hello world";
function getData() {
  return data;
}
module.exports = {
  getData,
};

// index.js
const { getData } = require("./module-a.js");
console.log(getData());

```

代码中使用 `require` 来导入一个模块，用`module.exports`来导出一个模块。实际上 Node.js 内部会有相应的 loader 转译模块代码，最后模块代码会被处理成下面这样:
```js
(function (exports, require, module, __filename, __dirname) {
  // 执行模块代码
  // 返回 exports 对象
});

```

对 CommonJS 而言，一方面它定义了一套完整的模块化代码规范，另一方面 Node.js 为之实现了自动加载模块的`loader`，看上去是一个很不错的模块规范，但也存在一些问题:

1. 模块加载器由 Node.js 提供，依赖了 Node.js 本身的功能实现，比如文件系统，如果 CommonJS 模块直接放到浏览器中是无法执行的。当然, 业界也产生了 [browserify](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fbrowserify%2Fbrowserify "https://github.com/browserify/browserify") 这种打包工具来支持打包 CommonJS 模块，从而顺利在浏览器中执行，相当于社区实现了一个第三方的 loader。
2. CommonJS 本身约定以同步的方式进行模块加载，这种加载机制放在服务端是没问题的，一来模块都在本地，不需要进行网络 IO，二来只有服务启动时才会加载模块，而服务通常启动后会一直运行，所以对服务的性能并没有太大的影响。但如果这种加载机制放到浏览器端，会带来明显的性能问题。它会产生大量同步的模块请求，浏览器要等待响应返回后才能继续解析模块。也就是说，**模块请求会造成浏览器 JS 解析过程的阻塞**，导致页面加载速度缓慢。

总之，CommonJS 是一个不太适合在浏览器中运行的模块规范。因此，业界也设计出了全新的规范来作为浏览器端的模块标准，最知名的要数 `AMD` 了。

## AMD规范
`AMD`全称为`Asynchronous Module Definition`，即异步模块定义规范。模块根据这个规范，在浏览器环境中会被异步加载，而不会像 CommonJS 规范进行同步加载，也就不会产生同步请求导致的浏览器解析过程阻塞的问题了。我们先来看看这个模块规范是如何来使用的:
```js
// main.js
define(["./print"], function (printModule) {
  printModule.print("main");
});

// print.js
define(function () {
  return {
    print: function (msg) {
      console.log("print " + msg);
    },
  };
});

```

在 AMD 规范当中，我们可以通过 define 去定义或加载一个模块，比如上面的 `main` 模块和`print`模块，如果模块需要导出一些成员需要通过在定义模块的函数中 return 出去(参考 `print` 模块)，如果当前模块依赖了一些其它的模块则可以通过 define 的第一个参数来声明依赖(参考`main`模块)，这样模块的代码执行之前浏览器会先**加载依赖模块**。

当然，你也可以使用 require 关键字来加载一个模块，如:
```js
// module-a.js
require(["./print.js"], function (printModule) {
  printModule.print("module-a");
});

```
不过 require 与 define 的区别在于前者只能加载模块，而`不能定义一个模块`。
## ES6 Module
`ES6 Module` 也被称作 `ES Module`(或 `ESM`)， 是由 ECMAScript 官方提出的模块化规范，作为一个官方提出的规范，`ES Module` 已经得到了现代浏览器的内置支持。在现代浏览器中，如果在 HTML 中加入含有`type="module"`属性的 script 标签，那么浏览器会按照 ES Module 规范来进行依赖加载和模块解析，这也是 Vite 在开发阶段实现 no-bundle 的原因，由于模块加载的任务交给了浏览器，即使不打包也可以顺利运行模块代码。

下面是一个使用 ES Module 的简单例子:
```js
// main.js
import { methodA } from "./module-a.js";
methodA();

//module-a.js
const methodA = () => {
  console.log("a");
};

export { methodA };

```
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/src/favicon.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vite App</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="./main.js"></script>
  </body>
</html>

```
如果在 Node.js 环境中，你可以在`package.json`中声明`type: "module"`
```json
// package.json
{
  "type": "module"
}

```
然后 Node.js 便会默认以 ES Module 规范去解析模块:
```js
node main.js // 打印 a
```
顺便说一句，在 Node.js 中，即使是在 CommonJS 模块里面，也可以通过 `import` 方法顺利加载 ES 模块，如下所示:
```ts
async function func() {
  // 加载一个 ES 模块
  // 文件名后缀需要是 mjs
  const { a } = await import("./module-a.mjs");
  console.log(a);
}

func();

module.exports = {
  func,
};

```