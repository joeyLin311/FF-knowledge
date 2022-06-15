---
date created: 2021-12-09 22:53
date updated: 2021-12-17 16:27
---

#webpack

# webpack 配置文件组成部分

## webpack4 的一级配置

参考资料:
[Webpack4 配置详解](https://lq782655835.github.io/blogs/project/webpack4-2.config-setting.html)

```javascript
module.exports = {
  mode: 'development', // 模式配置,webpack4.0新增
  entry: '', // 入口文件
  output: {}, // 出口文件
  module: {
    rules: [/*loader setting*/]
  }, // 配置modules，包括loader
  plugins: [], // 对应的插件
  devServer: {}, // 开发服务器配置
  optimization: {}, // 最佳实践
  devtool: '',
  resolve: { alias: {} },
}

```

## entry 入口:

entry 是 webpack 构建开始的地方, 通过入口文件, webpack 可以找打入口文件所依赖的文件, 并逐步递归, 找出所有依赖的文件

```javascript
module.export = {
  entry: "./path/my/entry/file.js"
}
```

## output 输出:

output 是 webpack 配置在哪个路径输出 webpack 构建好的 bundle 文件, 以及如何命名这些文件

```javascript
const path = require("path");
module.export = {
   entry: "./path/my/entry/file.js",
   output: {
     path: path.resolve(__dirname, "dist"),
     filename: "my-wenpack.bundle.js"
   }
 }
```

## Loader:

webpack 只支持 JavaScript, loader 能够让 webpack 以什么方式处理那些非 JavaScript 文件, 并预先将它们转换为有效的模块添加到依赖图中, 以供应用程序使用

```javascript
module.export = {
    entry: "./path/my/entry/file.js",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "my-wenpack.bundle.js"
  },
  module: {
    rules: [
      {
        // 根据后缀名匹配需要处理的非 JS 文件
        test: /\.txt$/,
        // 使用对应的loader处理该类文件
        use: "raw-loader"
      }
    ]
  }
}
```
## plugins 插件:

在 webpack 打包过程中, 我们可以使用 webpack 执行流程中的钩子函数, 更加精确地控制 webpack 的文件输出, 例如: 打包优化, 资源管理, 注入环境变量等

- 使用插件, 一般插件文件带 `-plugin` 后缀
- 编写自有插件, 利用 webpack 提供的钩子函数, 编写自定义插件, 相当于监听 webpack 的事件, 做相应的动作. 这里就涉及到 webpack 是通过 Tapable 进行事件流管理

## loader 和 plugin
[[webpack Loader 和 Plugin]]
