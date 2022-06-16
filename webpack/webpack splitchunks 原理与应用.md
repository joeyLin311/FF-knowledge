---
date created: 2022-06-14 00:26
---

之前 webpack 的 chunks 是通过内部 webpack 图谱中的父子关系关联的. `CommonChunksPlugin` 曾经被用来让 chunks 之间重复依赖, 但是无法做进一步的优化, webpack 4 开始新增加了 `optimization.splitChunks` 选项, 它开箱即用, 可以方便的**提取或分离公共代码, 防止代码被重复打包, 拆分过大的 js 文件, 合并零散的 js 文件**的目的.

webpack 根据一下条件自动拆分 chunks:

- 新的 chunks 可以被共享, 或者模块来自于 `node_modules` 文件
- 新的 chunk 体积大于 20kb (在 min+gz 之前的体积)
- 按需加载 chunks 时, 并行请求的最大数量小于或等于 30
- 当加载初始化页面时, 并发请求的最大数量小于或等于 30

以上是官方文档的描述, 但是实际上默认配置只会抽离出动态加载的模块, 通常情况下, 不是立即需要的包可以考虑动态加载
以下是 `SplitChunksPlugin` 的默认配置行为:

```jsx
// 如果使用的是 `initial` 或者 `all`, webpack 才会遵循这个配置进行打包.
module.exports = {
  //...
  optimization: {
    splitChunks: {
      chunks: 'async', // 指明要分割的插件类型, async:异步插件(动态导入),inital:同步插件,all：全部类型
      minSize: 20000,  // 文件最小大小,单位bite;即超过minSize有可能被分割；
      minRemainingSize: 0, // webpack5新属性，防止0尺寸的chunk
      minChunks: 1,
      maxAsyncRequests: 30, // webpack4,5区别较大
      maxInitialRequests: 30, // webpack4,5区别较大
      enforceSizeThreshold: 50000,
      cacheGroups: {
        defaultVendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: -10,
          reuseExistingChunk: true,
        },
        default: {
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true,
        },
      },
    },
  },
};
```

## 字段说明

### chunks: `async` | `initial` | `all`

默认值是 `async`, 表示对于动态加载的模块, 默认配置会将该模块单独打包, 用 `import('包')` 和 webpack 独有的 `require.ensure` 语法进行动态加载. 默认配置只会抽离出动态加载的模块, 通常情况下, 不是立即需要的包可以考虑动态加载. 比如: 不同入口引入了相同的包 `lodash` 但是 webpack 默认并不会抽出公共的 `lodash`.

当 chunks 值为 `all` 和 `initial` 时, 如果对于同一个包是区分成动态引入和正常引入, `initial` 会打包成两份, 而 `all` 只会打包成一份.

chunks 配置项还可以写成函数式, 函数返回值将决定是否包含每一个 chunk.

### minSize:  默认 20000

表示将要被分包的 chunks , 如果压缩前体积不足 20k,将不会被拆包

### minChunks: 1

某个 chunks 被多次引用, 如果这个应用次数小于某个值, 将不会被拆包

### enforceSizeThreshold: 50000

如果某个 chunk 的大小超过了 50k 以上的限制就不会生效.

### cacheGroup

`cachGroup` 有两个默认缓存策略, 也就是 chunks 值为 `all` 和 `initial` 时的默认配置: `defaultVendors` 和 `default`

1. `defaultVendors` 该策略会将源代码中所有引入 node_modules 的文件打包成一个大的 chunk.
2. `default` 则是对于多入口引入相同的模块超过两次后, 进行拆包操作, 需要注意的是, 在单页面应用中只有一个入口文件. 如果多入口文件需要单独打包, 需要考虑动态加载.

## css 分包

css 一般通过 style-loader 将 css 以 style 标签的形式插入文档, 这种方式正常引用是无法进行分包的. 可以通过动态引入的方式分包.

还可以用 MiniCssExtraPlugin 插件进行 css 分包.. 此插件默认为每个入口单独抽出 css, 也可以进行 cachGroup 的配置, 满足条件时, 会将多个入口的 css 打包在一起

```jsx
css: {
  name: "css",
  test: /\.css$/,
  minChunks: 1,
  enforce: true,
}

```

## 参考

- [[preload&prefetch#两者补充对比]]
