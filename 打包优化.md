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

- 压缩代码。删除多余的代码、注释、简化代码的写法等等方式。可以利用 webpack 的 `UglifyJsPlugin` 和 `ParallelUglifyPlugin` 来压缩 JS 文件， 利用 `cssnano`（css-loader?minimize）来压缩 css
- 利用 CDN 加速。在构建过程中，将引用的静态资源路径修改为 CDN 上对应的路径。可以利用 webpack 对于 `output` 参数和各 loader 的 `publicPath` 参数来修改资源路径
- 删除死代码（Tree Shaking）。将代码中永远不会走到的片段删除掉。可以通过在启动 webpack 时追加参数 `--optimize-minimize` 来实现
- 提取公共代码。
