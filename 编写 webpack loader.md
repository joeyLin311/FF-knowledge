---
date created: 2021-12-09 23:00
---

#webpack

### 1. 编写 webpack loader

#### 1.1 同步 loader

同步转换内容后，可以通过 return 或调用 this.callback 返回结果。

```js
export default function loader(content, map, meta) {  
    return someSyncOperation(content);
}
```

通过 this.callback 可以返回除内容以外的其他信息（如 sourcemap）。

```js
export default function loader(content, map, meta) { 

 this.callback(null, someSyncOperation(content), map, meta); 

 return; // 当调用 callback() 时，始终返回 undefined

}
```

#### 1.2 异步 loader

通过 this.async 可以获取异步操作的回调函数，并在回调函数中返回结果。

```js
export default function (content, map, meta) {
 const callback = this.async();
 someAsyncOperation(content, (err, result, sourceMaps, meta) => {
   if (err) return callback(err);
   callback(null, result, sourceMaps, meta);
 });
}
```

除非计算很小，否则对于 Node.js 这种单线程环境，尽可能使用异步 loader。

#### 1.3 loader 开发辅助工具及 loaderContext

`loader-utils` 与 `schema-utils`，可以使获取及验证传递给 loader 的参数的工作简单化。

```js
import { getOptions } from "loader-utils";

import { validate } from "schema-utils";

const schema = {

 type: "object",

 properties: {

 test: { type: "string", },

 },

};

export default function (source) {

 const options = getOptions(this);

 validate(schema, options, { name: "Example Loader", baseDataPath: "options", });

 // Apply some transformations to the source...

 return `export default ${JSON.stringify(source)}`;

}
```

loader-utils 主要有以下工具方法：

- `parseQuery`：解析 loader 的 query 参数，返回一个对象。
- `stringifyRequest`：将请求的资源转换为可以在 loader 生成的代码中 require 或 import 使用的相对路径字符串，同时避免绝对路径导致重新计算 hash 值。

```js
loaderUtils.stringifyRequest(this, "./test.js");  
// "\"./test.js\""
```

- `urlToRequest`：将请求的资源路径转换成 webpack 可以处理的形式。

```js
const url = "~path/to/module.js";  
const request = loaderUtils.urlToRequest(url); // "path/to/module.js"
```

- `interpolateName`：对文件名模板进行插值。

```js
// loaderContext.resourcePath = "/absolute/path/to/app/js/hzfe.js"  
loaderUtils.interpolateName(loaderContext, "js/[hash].script.[ext]", { content: ... });  
// => js/9473fdd0d880a43c21b7778d34872157.script.js

```

- `getHashDigest`：获取文件内容的 hash 值。

在编写 loader 的过程中，还可以利用 loaderContext 对象来获取 loader 的相关信息和进行一些高级的操作，常见的属性和方法有：

- `this.addDependency`：加入一个文件，作为 loader 产生的结果的依赖，使其在有任何变化时可以被监听到，从而触发重新编译。
- `this.async`：告诉 loader-runner 这个 loader 将会异步的执行回调。
- `this.cacheable`：默认情况下，将 loader 的处理结果标记为可缓存。传入 false 可以关闭 loader 处理结果的缓存能力。
- `this.fs`：用于访问 compilation 的 inputFileSystem 属性。
- `this.getOptions`：提取 loader 的配置选项。从 webpack 5 开始，可以获取到 loader 上下文对象，用于替代 `loader-utils` 中的 getOptions 方法。
- `this.mode`： webpack 的运行模式，可以是 "development" 或 "production"。
- `this.query`：如果 loader 配置了 options 对象，则指向这个对象。如果 loader 没有 options，而是以 query 字符串作为参数，query 则是一个以 `?` 开头的字符串。

### 2. webpack loader 工作机制

#### 2.1 根据 module.rules 解析 loader 加载规则

当 webpack 处理一个模块（module）时，会根据配置文件中 `module.rules` 的规则，使用 loader 处理对应资源，得到可供 webpack 使用的 JavaScript 模块。

根据具体的配置情况，loader 会有不同的类型，可以影响 loader 的执行顺序。具体类型如下所示：

```js
rules: [  // pre 前置 loader  { enforce: "pre", test: /\.js$/, loader: "eslint-loader" },  // normal loader  { test: /\.js$/, loader: "babel-loader" },  // post 后置 loader  { enforce: "post", test: /\.js$/, loader: "eslint-loader" },];
```

Copy

以及内联使用的 inline loader：

```js
import "style-loader!css-loader!sass-loader!./hzfe.scss";
```

Copy

在正常的执行流程中，这些不同类型的 loader 的执行顺序是：`pre -> normal -> inline -> post`。在下一节将会提到的 pitch 流程中，这些 loader 的执行顺序是反过来的：`post -> inline -> normal -> pre`。

对于内联 loader，可以通知修饰前缀改变 loader 的执行顺序：

```js
// ! 前缀会禁用 normal loaderimport { HZFE } from "!./hzfe.js";// -! 前缀会禁用 pre loader 和 normal loaderimport { HZFE } from "-!./hzfe.js";// !! 前缀会禁用 pre、normal 和 post loaderimport { HZFE } from "!!./hzfe.js";
```

Copy

一般情况下，`!` 前缀和 inline loader 一起使用仅出现在 loader（如 style-loader）生成的代码中，webpack 官方不建议用户同时使用 inline loader 和 `!` 前缀。

webpack rules 中配置的 loader 可以是多个链式串联的。在正常流程中，链式 loader 会按照从后往前的顺序执行。

- 最后的 loader 最先执行，它接收的是资源文件（resource file）的内容。
- 第一个 loader 最后执行，它将返回 JavaScript 模块和可选的 source map。
- 位于中间的 loader，对接收和返回没有特定要求，只要能处理之前 loader 返回的内容，产出下一个 loader 能够理解的内容就可以。

#### 2.2 loader-runner 的执行流程

webpack 调用 loader 的时机在触发 compilation 的 buildModule 钩子之后。webpack 会在 `NormalModule.js` 中，调用 runLoaders 运行 loader：

```js
runLoaders({    
    resource: this.resource, // 资源文件的路径，可以有查询字符串。如：'./test.txt?query'    
    loaders: this.loaders, // loader 的路径。    
    context: loaderContext, // 传递给 loader 的上下文    
    processResource: (loaderContext, resourcePath, callback) => {      
        // 获取资源的方式，有 scheme 的文件通过 readResourceForScheme 读取，否则通过 fs.readFile 读取。      
        const resource = loaderContext.resource;      
        const scheme = getScheme(resource);      
        if (scheme) {        
            hooks.readResourceForScheme          
                .for(scheme)          
                .callAsync(resource, this, (err, result) => {            
                // ...            
                return callback(null, result);          
            });      
        } else {        
            loaderContext.addDependency(resourcePath);        
            fs.readFile(resourcePath, callback);      
        }    
    },  
},  (err, result) => {    
    // 当 loader 转换完成后，会将结果返回到 webpack 中继续处理。    
    processResult(err, result.result);  
});
```

runLoaders 函数来自 `loader-runner` 包。在介绍 runLoaders 的具体流程之前，先介绍一下 pitch 阶段，上一节中所讲的这种从后往前执行 loader 的流程，一般叫做 normal 阶段。与之相对的，还有一种叫做 pitch 阶段的流程。

一个 loader 如果在导出的函数的 pitch 属性上挂在了方法，那这个方法将在 pitch 阶段执行。pitch 阶段不同于 normal 阶段，pitch 阶段的执行顺序是从前往后的，整个流程类似浏览器事件模型或洋葱模型，pitch 阶段先从前往后执行 loader，然后再进入 normal 阶段从后往前执行 loader。注意，pitch 阶段一般不返回值，一旦 pitch 阶段有 loader 返回值，则从这里开始进入从后往前执行的 normal 阶段。

loader-runner 的具体流程如下：

1. 处理从 webpack 接收的 context，继续添加必要的属性和辅助方法。

2. iteratePitchingLoaders 处理 pitch loader。

   如果我们给一个 module 配置了三个 loader，每个 loader 都配置了 pitch 函数：

   ```js
   module.exports = {  //...  module: {    rules: [      {        //...        use: ["a-loader", "b-loader", "c-loader"],      },    ],  },};
   ```

   那么处理这个 module 的流程如下：

   ```js
   |- a-loader `pitch`  |- b-loader `pitch`    |- c-loader `pitch`      |- requested module is picked up as a dependency    |- c-loader normal execution  |- b-loader normal execution|- a-loader normal execution
   ```

   如果 b-loader 在 pitch 中提前返回了值，那么流程如下：

   ```js
   |- a-loader `pitch`  |- b-loader `pitch` returns a module|- a-loader normal execution
   ```

3. iterateNormalLoaders 处理 normal loader。

   当 pitch loader 的流程处理完后，就来到了处理 normal loader 的流程。处理 normal loader 的流程和 pitch loader 相似，只是从后往前迭代。

   iterateNormalLoaders 和 iteratePitchingLoaders 都会调用 runSyncOrAsync 来执行 loader。runSyncOrAsync 会提供 context.async，这是一个返回 callback 的 async 函数，用于异步处理。

### 3. 常见 webpack loader 原理解析

loader 本身的操作并不复杂，就是一个负责转换其他资源到 JavaScript 模块的函数。

#### 3.1 raw-loader 分析

该 loader 是功能非常简单的同步 loader，它的核心步骤是从文件原始内容中取得序列化的字符串，修复 JSON 序列化特殊字符时的 bug，添加导出语句，使其成为 JavaScript 模块。

该 loader 在 webpack 5 中已废弃，直接使用 asset modules 的功能代替即可。该 loader 源码如下：

```js
import { getOptions } from "loader-utils";import { validate } from "schema-utils";
import schema from "./options.json";
export default function rawLoader(source) {  const options = getOptions(this);
  validate(schema, options, {    name: "Raw Loader",    baseDataPath: "options",  });
  const json = JSON.stringify(source)    .replace(/\u2028/g, "\\u2028")    .replace(/\u2029/g, "\\u2029");
  const esModule =    typeof options.esModule !== "undefined" ? options.esModule : true;
  return `${esModule ? "export default" : "module.exports ="} ${json};`;}
```

#### 3.2 babel-loader 分析

babel loader 是一个综合了同步和异步的 loader，在使用缓存配置时以异步模式运行，否则以同步方式运行。该 loader 的主要源码如下：

```
// imports ...// ...
const transpile = function (source, options) {  // ...
  let result;  try {    result = babel.transform(source, options);  } catch (error) {    // ...  }  // ...
  return {    code: code,    map: map,    metadata: metadata,  };};
// ...
module.exports = function (source, inputSourceMap) {  // ...
  if (cacheDirectory) {    const callback = this.async();    return cache(      {        directory: cacheDirectory,        identifier: cacheIdentifier,        source: source,        options: options,        transform: transpile,      },      (err, { code, map, metadata } = {}) => {        if (err) return callback(err);
        metadataSubscribers.forEach((s) => passMetadata(s, this, metadata));
        return callback(null, code, map);      }    );  }
  const { code, map, metadata } = transpile(source, options);
  this.callback(null, code, map);};
```

babel-loader 通过 callback 传递了经过 babel.transform 转换后的代码及 source map。

#### 3.3 style-loader 与 css-loader 分析

style-loader 负责将样式插入到 DOM 中，使样式对页面生效。css-loader 主要负责处理 import、url 路径等外部引用。

style-loader 只有 pitch 函数。css-loader 是 normal module。整个执行流程是先执行 style-loader 阶段，style-loader 会创建形如 `require(!!./hzfe.css)` 的代码返回给 webpack。webpack 会再次调用 css-loader 处理样式，css-loader 会返回包含 runtime 的 js 模块给 webpack 去解析。style-loader 在上一步注入 `require(!!./hzfe.css)` 的同时，也注入了添加 style 标签的代码。这样，在运行时（浏览器中），style-loader 就可以把 css-loader 的样式插入到页面中。

常见的疑问就是为什么不按照 normal 模式组织 style-loader 和 css-loader。

首先 css-loader 返回的是形如这样的代码：

```
import ___CSS_LOADER_API_IMPORT___ from "../node_modules/_css-loader@5.1.3@css-loader/dist/runtime/api.js";var ___CSS_LOADER_EXPORT___ = ___CSS_LOADER_API_IMPORT___(function (i) {  return i[1];});// Module___CSS_LOADER_EXPORT___.push([  module.id,  ".hzfe{\r\n    height: 100px;\r\n}",  "",]);// Exportsexport default ___CSS_LOADER_EXPORT___;
```

style-loader 无法在编译时获取 CSS 相关的内容，因为 style-loader 无法处理 css-loader 生成结果的 runtime 依赖。style-loader 也无法在运行时获取 CSS 相关的内容，因为无论怎样拼接运行时代码，都无法获取到 CSS 的内容。

作为替代，style-loader 采用了 pitch 方案，style-loader 的核心功能如下所示：

style-loader

```
module.exports.pitch = function (request) {  var result = [    // 生成 require CSS 文件的语句，交给 css-loader 解析 得到包含 CSS 内容的 JS 模块    // 其中 !! 是为了避免 webpack 解析时递归调用 style-loader    `var content=require("${loaderUtils.stringifyRequest(this, `!!${request}`)}")`,    // 在运行时调用 addStyle 把 CSS 内容插入到 DOM 中    `require("${loaderUtils.stringifyRequest(this, `!${path.join(__dirname, "add-style.js")}`)}")(content)`    // 如果发现启用了 CSS modules，则默认导出它    "if(content.locals) module.exports = content.locals",  ];  return result.join(";");};
```

add-style.js

```js
module.exports = function (content) {  var style = document.createElement("style");  style.innerHTML = content;  document.head.appendChild(style);};
```

在 pitch 阶段，style-loader 生成 require CSS 以及注入 runtime 的代码。该结果会返回给 webpack 进一步解析，css-loader 返回的结果会作为模块在运行时导入，在运行时能够获得 CSS 的内容，然后调用 add-style.js 把 CSS 内容插入到 DOM 中。
