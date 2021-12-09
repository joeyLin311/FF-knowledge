---
date created: 2021-12-09 22:57
---

#webpack

## 自实现 loader

[Writing a Loader](https://webpack.docschina.org/contribute/writing-a-loader/)
官方文档说实现一个 loader 有以下几点编写原则:

[Writing a Loader](https://webpack.docschina.org/contribute/writing-a-loader/)
官方文档说实现一个 loader 有以下几点编写原则:

- Keep them **simple**.
  - 单一原则, 一个 loader 只做一件事
- Utilize **chaining**.
  - 使用链式调用
- Emit **modular** output.
  - 模块化输出
- Make sure they're **stateless**.
  - 无状态化 loader, 函数式编程
- Employ **loader utilities**.
  - 使用 `loader-utils` 包
- Mark **loader dependencies**.
  - 显式声明 loader 的依赖, `addDependency()`
- Resolve **module dependencies**. 处理模块依赖
  - 使用 require 声明
  - 使用 `this.resolve()` 处理路径
- Extract **common code**.
  - 避免在每个模块中生成通用代码加载程序进程。 相反，在加载器中创建一个运行时文件，并为该共享模块生成一个 require
- Avoid **absolute paths**.
  - 避免使用绝对路径
- Use **peer dependencies**. 使用

简短的原则:

- 单一原则: 一个 loader 只做一件事
- 链式调用: webpack 会按顺序链式调用每个 loader
- 统一原则: 遵循 webpack 设计原则, 输入与输出均为字符串, 各个 loader 完全独立, 即插即用

### 实现 babel-loader

[babel-loader](https://blog.csdn.net/qq_38935512/article/details/112918516)
[jianshu](https://www.jianshu.com/p/297e838b104e)

## 自实现 plugin

webpack 插件由以下部分组成

1. 一个 JavaScript 命名函数。
2. 在插件函数的 prototype 上定义一个 apply 方法。
3. 指定一个绑定到 webpack 自身的事件钩子。
4. 处理 webpack 内部实例的特定数据。
5. 功能完成后调用 webpack 提供的回调。

实现将经过 loader 处理后的打包文件 bundle.js 引入到 index.html 中

```javscript
function Plugin(options) { }

Plugin.prototype.apply = function (compiler) {
  // 所有文件资源经过不同的 loader 处理后触发这个事件
  compiler.plugin('emit', function (compilation, callback) {
    // 获取打包后的 js 文件名
    const filename = compiler.options.output.filename
    // 生成一个 index.html 并引入打包后的 js 文件
    const html = `<!DOCTYPE html>
                    <html lang="en">
                    <head>
                        <meta charset="UTF-8">
                        <meta name="viewport" content="width=device-width, initial-scale=1.0">
                        <title>Document</title>
                        <script src="${filename}"></script>
                    </head>
                    <body>
                        
                    </body>
                    </html>`
    // 所有处理后的资源都放在 compilation.assets 中
    // 添加一个 index.html 文件
    compilation.assets['index.html'] = {
      source: function () {
        return html
      },
      size: function () {
        return html.length
      }
    }

    // 功能完成后调用 webpack 提供的回调
    callback()
  })
}

module.exports = Plugin
```

### compiler 钩子函数

<https://webpack.docschina.org/api/compiler-hooks/>
参考链接:
[手写 loader 和 plugin](https://github.com/woai3c/webpack-demo/tree/master/src)
