---
date created: 2021-12-09 22:58
date updated: 2021-12-17 14:41
---

#JavaScript
[比较](https://juejin.cn/post/6844903663404580878)
<https://zhuanlan.zhihu.com/p/108217164>
<https://www.cnblogs.com/chenwenhao/p/12153332.html>
<https://juejin.cn/post/7007946894605287432?utm_source=gold_browser_extension#comment>
<https://juejin.cn/post/6994814324548091940> (取图)

## common JS

common JS 规范是 nodeJS 诞生的基础, 规范中定义:

- 每个文件就是一个独立的模块, 有自己的作用域, 模块的变量, 函数, 类都是文件的私有成员, 对其它文件不可见
- 使用 `module.exports` 主动暴露, 而在另一个文件中引用则直接使用 `require(path)` 即可.

**使用场景:**

- 服务端: 模块的加载是运行时同步加载
- 浏览器端: 模块需要提前编译打包处理, 不然浏览器不能识别 require 语法

**common JS 特点**

- 所有代码都运行在模块作用域, 不会污染全局作用域
- 模块可以多次加载, 但只会在第一次加载时运行一次, 然后运行结果就被缓存再来, 往后再加载就直接读取缓存结果. 如果想让模块再次运行, 比如先清楚缓存
- 模块加载是同步加载, 属于阻塞操作. 模块记载的顺序, 按照其在代码中出现的顺序

**common JS 加载机制** : common JS 模块的加载机制是输出一个值得拷贝, 一旦输出一个值, 模块内部的变化不会影响这个值
[[CommonJs 是如何导致打包体积增大_摘要]]
## AMD 规范

asynchrounos Module Definition, 称为: 异步模块定义. 它完整描述了模块的定义, 依赖关系, 引用关系以及加载机制. 主要用于浏览器, 采用异步方式加载模块, 模块的加载不影响它后面语句的运行. 所有依赖这个模块的语句, 都定义在一个回调函数中, 等到加载完成后, 这个回调函数才会运行. 因为 AMD 规范不是 ECMAScript 支持的规范, 所以需要引入第三方的库函数, 大名鼎鼎的 `RequireJS`.  它是 AMD 规范的标准化实现
模块功能主要的几个命令：`define`、`require`、`return` 和 `define.amd`。define 是全局函数，用来定义模块,`define(id?, dependencies?, factory)`。`require` 命令用于输入其他模块提供的功能，`return` 命令用于规范模块的对外接口，`define.amd` 属性是一个对象，此属性的存在来表明函数遵循 AMD 规范。

### AMD 使用:

- 使用 `define` 和 `require` 用来定义模块和调用模块方法, 合称为 AMD 模式. 模块定义的方法清晰, 不会污染全局环境, 能够清楚地显示依赖关系
- 可用于浏览器环境, 允许异步加载模块, 也可以根据需要动态加载模块

### define 方法: 定义模块

define 方法用于定义模块, RequireJS 要求每个模块放在一个单独的文件里. 按照是否依赖其它模块, 可分为: 一,定义独立模块, 不依赖其它模块; 二, 定义非独立模块, 有依赖其它模块

### require 方法: 调用模块

require 方法用于调用模块, 参数与 define 方法类似

### 小实现

```javascript
(function () {
  // 缓存
  const cache = {}
  let moudle = null
  const tasks = []
  
  // 创建script标签，用来加载文件模块
  const createNode = function (depend) {
    let script = document.createElement("script");
    script.src = `./${depend}.js`;
    // 嵌入自定义 data-moduleName 属性，后可由dataset获取
    script.setAttribute("data-moduleName", depend);
    let fs = document.getElementsByTagName('script')[0];
    fs.parentNode.insertBefore(script, fs);
    return script;
  }

  // 校验所有依赖是否都已经解析完成
  const hasAlldependencies = function (dependencies) {
    let hasValue = true
    dependencies.forEach(depd => {
      if (!cache.hasOwnProperty(depd)) {
        hasValue = false
      }
    })
    return hasValue
  }

  // 递归执行callback
  const implementCallback = function (callbacks) {
    if (callbacks.length) {
      callbacks.forEach((callback, index) => {
        // 所有依赖解析都已完成
        if (hasAlldependencies(callback.dependencies)) {
          const returnValue = callback.callback(...callback.dependencies.map(it => cache[it]))
          if (callback.name) {
            cache[callback.name] = returnValue
          }
          tasks.splice(index, 1)
          implementCallback(tasks)
        }
      })
    }
  }
   
  // 根据依赖项加载js文件
  const require = function (dependencies, callback) {
    if (!dependencies.length) { // 此文件没有依赖项
      moudle = {
        value: callback()  
      }
    } else { //此文件有依赖项
      moudle = {
        dependencies,
        callback
      }
      tasks.push(moudle)
      dependencies.forEach(function (item) {
        if (!cache[item]) {
          // script表亲加载文件结束
          createNode(item).onload = function () {
            // 获取嵌入属性值，即module名
            let modulename = this.dataset.modulename
            console.log(moudle)
            // 校验module中是否存在value属性
            if (moudle.hasOwnProperty('value')) {
              // 存在，将其module value（模块返回值｜导出值）存入缓存
              cache[modulename] = moudle.value
            } else {
              // 不存在
              moudle.name = modulename
              if (hasAlldependencies(moudle.dependencies)) {
                // 所有依赖解析都已完成，执行回调，抛出依赖返回（导出）值
                cache[modulename] = callback(...moudle.dependencies.map(v => cache[v]))
              }
            }
            // 递归执行callback
            implementCallback(tasks)
          }
        }
      })
    }
  }
  window.require = require
  window.define = require
})(window)
```

调用 `require` 或者 `define` 方法，首先是根据依赖数组加载 js 文件，不同于 commonJS，AMD 基于浏览器，要读文件，我们只能动态创建 script 标签，所以 `createNode` 即创建 `script` 标签，用来加载文件模块。
`script` 引入文件加载完成后会触发 `onload` 事件，我们以此控制依赖的加载顺序。只有在 JS 模块加载完成后，才能执行其 `callback` 回调，但是我们引入的 JS 依赖项中都是使用 `define` 方法定义的，而 `define` 方法还可能会依赖某些 js 文件模块，但总有一个源头是不存在依赖的，如此，递归便派上了用场。
我们的目的是模块加载完成后执行 callbck 回调，但如果是 A 依赖 B，B 又依赖 C 等等的关系，我们想要执行 A 回调，那必须等 B 和 C 都加载完，所以我们使用一个栈（数组） tasks 来存储 callback 回调，等所有依赖都加载完了，再依次执行，就和 Node 框架 koa 的洋葱模型一样。这是为了让 callback 回调函数的执行顺序正确。

## CMD 规范:

Common Module Definition, 通用模块定义, 主要是在 Sea.js 推广中形成的, 汲取了 AMD 和 commonJS 的优点, 只不过在模块定义方式和模块加载时机上不同.
不同点在于: AMD 推崇依赖前置, 提前执行, CMD 推崇依赖就近, 延迟执行. CMD 规范也需要引入第三方的库文件, SeaJS.

- AMD/RequireJS 的 JS 模块实现有很多不优雅的地方，主要原因不能以一种更好的管理模块的依赖加载和执行；
- 那么就出现了 SeaJS，SeaJs 遵循的是 CMD 规范，CMD 规范在 AMD 的基础上改进的一种规范，解决了 AMD 对依赖模块的执行时机的问题；
- SeaJS 模块化的顺序是：模块化预加载=》主逻辑调用模块时才执行模块中的代码；
- SeaJS 的用法和 AMD 基本相同，并且融合了 CommonJS 的写法；

## UMD 规范: commonJS + AMD

UMD(Universal Module Definition - 通用模块定义)模式，该模式主要用来解决 CommonJS 模式和 AMD 模式代码不能通用的问题，并同时还支持老式的全局变量规范。
CommonJS 适用于服务端，AMD、CMD 适用于 web 端，那么需要同时运行在这两端的模块就可以采用 UMD 的方法，使用该模块化方案，可以很好地兼容 AMD、CommonJS 语法。UMD 先判断是否支持 Node.js 的模块（exports）是否存在，存在则使用 node.js 模块模式。再判断是否支持 AMD（define 是否存在），存在则使用 AMD 方式加载模块。由于这种通用模块解决方案的适用性强，很多 JS 框架和类库都会打包成这种形式的代码。

## ES Modules

ESM 是 ECMAScript 官方的标准化模块系统.

### 仅支持静态导入导出

ESM 规范仅支持静态的导入和导出, ES Modules 需要在编译时期进行模块化静态优化, 也就是必须要在编译时就确定模块定义引用和依赖关系. 有两点原因:

1. 性能, 在编译阶段即完成所有的模块导入, 如果在运行时进行会降低速度
2. 编译阶段就能更好地检测错误, 对变量类型进行筛错

### 特点:

1. 官方标准, 未来浏览器都会支持, 方便在浏览器中使用. 同时又兼容在 node 环境下运行
2. 模块的导入导出, 通过 `import` 和 `export` 来确定, 可以和 CommonJs 模块混合使用
3. ESM 输出的是对值的引用, 输出接口动态绑定, CommonJs 输出的是值得拷贝
4. 上面说了 ESM 在编译时执行, CommonJs 总是在运行时加载
5. ESM  模块编译时执行时, import 会被 JS 引擎今天分析, 优先于模块内的其它内容执行. export 命令会有变量声明提前的效果. 这种设计有利于编译器提高效率, 但也导致无法在运行时加载模块(动态加载)
6. TC39 有提案为 ESM 增加动态加载的特性 `import(模块位置)`

### 动态加载和静态编译的区别?

动态加载: 只有当模块运行后, 才能知道导出的模块是什么
静态编译: 在编译阶段就能知道导出什么模块

## JS 模块化发展产物：

`CommonJS(服务端)` =>`AMD(浏览器)` => `CMD`=>`ES Modules`

## Tree shaking

这个概念是 Rollup 提出来的。Rollup 推荐使用 ES2015 Modules 来编写模块代码，这样就可以使用 Tree-shaking 来对代码做静态分析消除无用的代码，可以查看 Rollup 网站上的 REPL 示例，代码打包前后之前的差异，就会清晰的明白 Tree-shaking 的作用。

1. 没有使用额外的模块系统，直接定位 import 来替换 export 的模块
2. 去掉了未被使用的代码

## ES Modules 之所以能 Tree-shaking 主要为以下四个原因（摘自尤雨溪在知乎的回答）:

1. import 只能作为模块顶层的语句出现，不能出现在 function 里面或是 if 里面。
2. import 的模块名只能是字符串常量。
3. 不管 import 的语句出现的位置在哪里，在模块初始化的时候所有的 import 都必须已经导入完成。
4. import binding 是 immutable 的，类似 const。比如说你不能 import { a } from ‘./a’ 然后给 a 赋值个其他什么东西。
