---
date created: 2021-12-09 22:53
---

#webpack

## 哈希也有讲究

webpack 给我们提供了三种哈希值计算方式，分别是 hash、chunkhash 和 contenthash。那么这三者有什么区别呢？

- hash：跟整个项目的构建相关，构建生成的文件 hash 值都是一样的，只要项目里有文件更改，整个项目构建的 hash 值都会更改。
- chunkhash：根据不同的入口文件(Entry)进行依赖文件解析、构建对应的 chunk，生成对应的 hash 值。
- contenthash：由文件内容产生的 hash 值，内容不同产生的 contenthash 值也不一样。

显然，我们是不会使用第一种的。改了一个文件，打包之后，其他文件的 hash 都变了，缓存自然都失效了。这不是我们想要的。

那 chunkhash 和 contenthash 的主要应用场景是什么呢？在实际在项目中，我们一般会把项目中的 css 都抽离出对应的 css 文件来加以引用。如果我们使用 chunkhash，当我们改了 css 代码之后，会发现 css 文件 hash 值改变的同时，js 文件的 hash 值也会改变。这时候，contenthash 就派上用场了。
[[webpack 组成部分]]
[[webpack 运行原理]]
[[HMR 热更新]]

## webpack 核心概念

**Compiler** : webpack 的运行入口, compiler 对象代表了完整的 webpack 环境配置. 这个对象在启动 wbepack 时被一次性建立, 并配置好所有可操作的设置, 包括: options, loader 和 plugin. 当在 wbepack 环境中应用一个插件时, 插件将接收到该 compiler 对象的引用, 可是使用它来访问 webpack 的主环境
**Compilation** : 代表一次资源的构建, 当运行 webpack 开发环境时, 每当检测到一个文件的变化, 就会创建一个新的 Compilation , 从而生成一组新的编译资源. 一个 compilation 对象表示了当前的模块资源, 编译生成资源, 交换的文件, 以及被跟踪依赖的状态信息. compilation 也提供了很多关键步骤的回调, 以供插件在自定义处理时选择使用
**Module** : 用于表示代码模块的基础类, 关于代码模块的所有信息都会存储在 modules 实例中, 例如: dependencies 这种记录代码模块的依赖, 等. 创建一个 module 对象, 主要做以下操作: 1. 搜集所有依赖的 module. 2. 执行对应的 loader
**Chunk** : 一个 Chunk 是由一个或多个 Module 生成. 一般根据入口文件生成 Chunk , 然后把入口文件所依赖文件的 Module 集合加入到 Chunk 中. 简单来说的就是: Chunk 是打包后的代码块, 如果没有使用代码拆分, 那么打包后的 bundle 和 chunk 就是一样的. 生成 chunk 有两种方式: 1. 入口文件模块. 2. 动态引入的模块
**Parser** : 基于 acorn 来分析 ast 语法树, 解析出代码模块的依赖
**Dependency** : 保存代码模块对应的依赖使用的对象, module 实例的 build 方法在执行完对应的 loader 后, 会继续调用 parser 实例来解析自身依赖的模块, 解析后的结果存放在 module.denpendencies 中, 具体步骤如下: 首先保存的是依赖的路径, 后续会经由 compilation.processModuleDependencies 方法处理模块的依赖.
**Template** : 生成最终代码要使用到的代码模块, 相当于是根据 modules 创建一个自执行函数来执行所有 modules.

## Loader, Plugin

[[Loader 和 Plugin]]
[[自实现 loader 和 plugin]]

## source map

[[source map]]

## webpack 构建性能

[[打包优化]]
[官网](https://www.webpackjs.com/guides/build-performance/)
