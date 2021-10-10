#webpack 
## 有哪些loader和plugin
[https://juejin.cn/post/6844904094281236487#heading-3](https://juejin.cn/post/6844904094281236487#heading-3)
### plugin
-   `define-plugin`：定义环境变量 (Webpack4 之后指定 mode 会自动配置)
-   `ignore-plugin`：忽略部分文件
-   `html-webpack-plugin`：简化 HTML 文件创建 (依赖于 html-loader)
-   `web-webpack-plugin`：可方便地为单页应用输出 HTML，比 html-webpack-plugin 好用
-   `uglifyjs-webpack-plugin`：不支持 ES6 压缩 (Webpack4 以前)
-   `terser-webpack-plugin`: 支持压缩 ES6 (Webpack4)
-   `webpack-parallel-uglify-plugin`: 多进程执行代码压缩，提升构建速度
-   `mini-css-extract-plugin`: 分离样式文件，CSS 提取为独立文件，支持按需加载 (替代extract-text-webpack-plugin)
-   `serviceworker-webpack-plugin`：为网页应用增加离线缓存功能
-   `clean-webpack-plugin`: 目录清理
-   `ModuleConcatenationPlugin`: 开启 Scope Hoisting
-   `speed-measure-webpack-plugin`: 可以看到每个 Loader 和 Plugin 执行耗时 (整个打包耗时、每个 Plugin 和 Loader 耗时)
-   `webpack-bundle-analyzer`: 可视化 Webpack 输出文件的体积 (业务组件、依赖第三方模块)
当然我们可以在官网的插件市场上看到所有的插件
[官网插件](https://webpack.docschina.org/plugins/)
### loader
-   `raw-loader`：加载文件原始内容。
-   `file-loader`：将引用文件输出到目标文件夹中，在代码中通过相对路径引用输出的文件。
-   `url-loader`：和 `file-loader` 类似，但是能在文件很小的情况下以 base64 的方式将文件内容注入到代码中。
-   `babel-loader`：将 ES 较新的语法转换为浏览器可以兼容的语法。
-   `style-loader`：将 CSS 代码注入到 JavaScript 中，通过 DOM 操作加载 CSS。
-   `css-loader`：加载 CSS，支持模块化、压缩、文件导入等特性。
[官网loader](https://webpack.js.org/plugins/)

## loader 和 plugin 的区别
### loader 机制
- **Loader** 概念
	- loader 与对模块的源代码进行转换
	- 是个转换器, 针对某种类型进行转换的模块功能, 比如: 统一语法, 对sass,less 文件做处理, 对非JS文件转换成commonJS规范的文件 output ony JS
	- [[编写 webpack loader]]
### plugin 机制
- **plugin** 概念
	- webpack 插件是一个具有 apply 方法的 JavaScript 对象。apply 方法会被 webpack compiler 调用，并且在 整个 编译生命周期都可以访问 compiler 对象
	- 是作用在webpack的工作流程中, 使用plugin API 监听 compilation 广播出的时间对流程节点进行操作, 执行某种通用操作. 比如, 打包优化和压缩, 定义环境变量, 资源管理等等, 是作用于过程的.
### 区别
`Loader` 本质就是一个函数，在该函数中对接收到的内容进行转换，返回转换后的结果。 因为 Webpack 只认识 JavaScript，所以 Loader 就成了翻译官，对其他类型的资源进行转译的预处理工作。
`Plugin` 就是插件，基于事件流框架 `Tapable`，插件可以扩展 Webpack 的功能，在 Webpack 运行的生命周期中会广播出许多事件，Plugin 可以监听这些事件，在合适的时机通过 Webpack 提供的 API 改变输出结果。
`Loader` 在 module.rules 中配置，作为模块的解析规则，类型为数组。每一项都是一个 Object，内部包含了 test(类型文件)、loader、options (参数)等属性。
`Plugin` 在 plugins 中单独配置，类型为数组，每一项是一个 Plugin 的实例，参数都通过构造函数传入。

## [[自实现loader和plugin]]

