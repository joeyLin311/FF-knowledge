#webpack 
## webpack 运行原理
![[webpack打包流程.jpg]]
运行原理大致归纳为如下几个阶段:
1.  读取 webpack.config.js 配置文件, 生成 compiler 实例, 并把 compiler 实例注入 plugin 中的 apply 方法中
2.  读取配置中的 `Entries` , 递归遍历所有的入口文件
3.  对入口文件进行编译, 开始 `compilation` 过程, 使用 `loader` 对文件内容编译, 再将编译好的文件内容解析成 `AST`静态语法树
4.  递归依赖的模块, 重复上一步, 生成`AST`语法树, 在 `AST`语法树中可以分析模块之间的依赖关系, 对应做出优化
5.  将所有模块中的 require 语法替换成 `__webpack_require__` , 用来模拟模块化操作
6.  完成上述动作后, 把所有的模块打包进一个自执行函数 (IIFE) 中