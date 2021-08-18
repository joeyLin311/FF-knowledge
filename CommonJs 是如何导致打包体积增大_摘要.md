#日常记录 #阅读笔记
_注: 原文为英文, _[_链接在此:How CommonJS is making your bundles larger_](https://web.dev/commonjs-larger-bundles/)

1. 什么是 CommonJs 
   1. 历史由来
   1. node端的模块化规范
   1. 因为2010年前浏览器缺乏标准化的模块系统所以 CommonJS 也成为了 JavaScript 客户端主流的模块化标准.
2. CommonJs 是如何影响最终的打包体积的
   1. 因为 CommonJs 主要用于服务端(不怎么考虑程序体积大小), 并没有在设计之初考虑 CommonJs 在浏览器端需要减少包体积, 分析也表明JS包大小依然是影响浏览器体验的主要原因.
   1. JavaScript 打包和压缩程序(例如 `webpack` 和 `terser` ) 通过执行不同的优化来减小应用程序的大小。他们在构建时分析你的程序, 尝试尽可能多地删除那些没有用到的代码.
   1. 比较了一波用 CommonJs 和 ECMAscript 模式的打包体积, 发现后者bundle体积小了很多.
   1. 结论是, **一般 CommonJS 模块很难优化, 因为它们比 ES 模块要动态得多. 为确保打包和压缩程序能够成功优化应用程序, 应该避免依赖 CommonJS 模块, 并在整个程序中使用 ES2015 模块语法.**
3. 为什么 CommonJs 使你的打包体积更大
   1. 提及 `webpack`中 `ModuleConcatenationPlugin` 的行为, 也就是大名鼎鼎的 `tree-shaking`, 旨在打包过程中删除未被使用的imports. 一般ES模块会自动启用此行为, 与 CommonJs 模块相比, ES可以对其静态分析.
   1. 所以webpack在打包构建过程中, 如果无法了解依赖中的确切用途的话(非静态), 就不会启用`tree-shaking`
4. 其实是可以用 CommonJs 的 tree-shaking , 但这是第三方 webpack plugin, 并未涵盖可以使用的的 CommonJs 的所有方式, 无法保证获得与ES模块相同的保证. 而且会在构建过程中增加额外的成本.

**总结, 如无必要, 避免项目依赖CommonJs模块**
