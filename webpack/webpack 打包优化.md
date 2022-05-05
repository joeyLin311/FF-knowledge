---
date created: 2021-12-09 23:00
---

#webpack

## 如何提高 webpack 的构建速度

多入口情况下，使用 `CommonsChunkPlugin` 来提取公共代码

通过 `externals` 配置来提取常用库

利用 `DllPlugin` 和 `DllReferencePlugin` 预编译资源模块 通过 `DllPlugin` 来对那些我们引用但是绝对不会修改的 npm 包来进行预编译，再通过 `DllReferencePlugin` 将预编译的模块加载进来。

使用 `Happypack` 实现多线程加速编译

使用 `webpack-uglify-parallel` 来提升 `uglifyPlugin` 的压缩速度。 原理上 `webpack-uglify-parallel` 采用了多核并行压缩来提升压缩速度

使用 `Tree-shaking` 和 `Scope Hoisting` 来剔除多余代码

## 如何利用 webpack 来优化前端性能？（提高性能和体验）

用 webpack 优化前端性能是指优化 webpack 的输出结果，让打包的最终结果在浏览器运行快速高效。

### thread-loader 构建时间优化

使用 `thread-loader` 开启多线程打包, 提高项目的打包速度. 该 loader 作用是在在某个资源使用中, 放在其它 loader 之前, 之后使用的 loader 就会在一个单独的 worker 池(worker pool) 中运行. 但是开启之后, 这些 loader 是受到限制的:
- 不能产生新的文件.
- 不能使用定制的 loader API 也就是通过插件
- 无法获取 webpack 的选项配置
使用 `thead-loader` 需要注意, 每个 worker 是单独的有 600ms 限制的 nodeJS 进程. 同事跨进程的数据交换也会被限制.
**建议在耗时较大的loader上使用该loader**

### cache-loader 缓存资源

使用缓存loader 对性能开销较大的loader使用, 目的是提高二次构建的速度. 比如 `babel-loader` 这种耗时较长的 loader , 其结果可以通过缓存loader放进磁盘里.
**但是该loader保存读取缓存也会有时间开销, 所以只适合对性能开销较大的loader 使用**

### 启用HMR 热更新

启用热更新可以让我们在修改小部分代码时, 通过热更新的原理, 只刷新修改的部分. 这样能有效提高项目的重新构建时间. [[webpack HMR 热更新]]

### exclude & include

- `exclude`: 不需要处理的文件
- `include`: 需要处理的文件
合理使用上述两个属性, 可以提高项目构建速度.

### 区分开发和生产环境的webpack配置

- 开发环境: 去除代码压缩, gzip, 体积分析等优化的配置
- 生产环境: 目标是为了构建体积更小的代码

### 其它

- 压缩代码。删除多余的代码、注释、简化代码的写法等等方式。可以利用 webpack 的 `UglifyJsPlugin` 和 `ParallelUglifyPlugin` 来压缩 JS 文件， 利用 `cssnano`（**css-loader?minimize）**来压缩 css
- 利用 CDN 加速。在构建过程中，将引用的静态资源路径修改为 CDN 上对应的路径。可以利用 webpack 对于 `output` 参数和各 loader 的 `publicPath` 参数来修改资源路径
- 删除死代码（Tree Shaking）。将代码中永远不会走到的片段删除掉。可以通过在启动 webpack 时追加参数 `--optimize-minimize` 来实现
- 提取公共代码。

## 如何排查 wabpack 打包产物过大

首先我们要了解为什么 webpack 输出的 bundle 会很大, 在项目开发过程中随着业务需求的叠加 bundle 产物增加是必然的.

### css 部分

CSS 文件里面, 重复的 style 代码会导致在 build 过程中急速增大 bundle 体积(**这里涉及到 webpack 打包 css 的原理**), 所以才用 `css in js` 或者原子化css 可以减少部分打包体积

## 静态资源部分

我们需要或多或少在项目中引入静态资源文件, 比如图标, 图片, 文件(之前的代码中有协议类的 doc 文件), 这些文件并不需要全部都打包成 `base64` , 我们可以根据需要对这些文件进行处理(**写一个 plugin 专门处理静态文件**)

## 代码部分

业务需求增加就会增加 `bundle` 体积, 对于这些 `bundle` 工具而言, 有 `tree shaking` 和 `DEC` , 但是我们无法确保我们使用的包是否只是 `tree shaking` , 所以在面对一些无法 `tree shaking` 的包, 我们可以使用 `antd` 的 `babel-import` 插件去做死代码消除. 在编写一些代码的时候, 我们应该明确指明相关代码是否有副作用, 合理地给代码加上 `@pure` 标记. 

拥抱现代标准, 采用 `esm` 开发, 让 webpack 更好地耕作, 同时在工程上设置 `sideEffect` 保证项目打包时充分地 `tree shaking`

以上是编码过程中的考虑, 以下是 build 之后还是 `bundle` 太大的分析方法:

- 合理地分割代码, 部分模块设置为 `external` 防止被打包进 `bundle` 中. 这些包在发布过程中采用 `CDN` 方式引入
- 使用 `webpack-analyze` 或者 `rollup-analyze` 分析工具对项目分析 chunk 和 module. 根据业务需求进行优化改造 **(具体是如何做的)**
- 合理选择兼容性, 是否需要兼容到低版本浏览器, 控制好 `babel` 或者 `swc` 需要 `transform` 的版本
- 关于 `@babel/core` 和 `@babel/runtime-helper` 等包的依赖, 应该明确它们如何使用, 使用不当也会导致 `bundle` 体积增大
- 使用 `treser` 或者 `exbuild` 等压缩工具进行代码压缩
- 生产环境考虑关闭 `source-map` 也能减少打包体积

#### dev 环境下
- 在 dev 环境下使用 `vite` , webpack 的架构设计跟 `vite` 没法比, 我们可以在开发环境下使用 `vite` 提升开发效能.
- 如果项目强依赖于 webpack 那么需要增量编译的时候合理利用缓存, 对于 `require.context` 这样的东西应该避免使用
- 然后 `ts-loader` 完全可以使用 `thread-lodaer` 替代, 多线程构建并不是良药, 进程的开销是需要代价, 我们在本地环境开发, 应该测试得出到底是否需要启动多线程和编译. 
- `source-map` 选择合理的配置模式
- 开发环境下我们或许不需要让 `babel` 兼容更低的版本