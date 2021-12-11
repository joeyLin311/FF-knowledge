---
date created: 2021-12-09 22:53
---

#Vue

## vue2 模板编译过程

[模板编译原理](https://ustbhuangyi.github.io/vue-analysis/v2/compile/)

- 入口函数 `ccompilerToFunctions()`
  - 内部首先从缓存加载编译好的 `render` 函数
  - 如果没有，调用 `baseCompile()` 开始编译
- `baseCompile()` **函数**
  - 合并 `options`
  - 调用 `baseCompile` 编译模板
- `baseCompile()` 的核心是三件事情, **解析, 优化, 生成(此处核心)**
  - `parse()` 解析: 把模板转换成 AST tree, 也就是抽象语法树
  - `optimize()` 优化:
    - 标记 AST tree 中的静态节点
    - 检测到静态节点时, 设置为静态属性, 不需要在每次重新渲染的时候生成新的节点
    - `patch` 阶段跳过静态节点
    - `generator()` 生成: 把优化过的 AST 对象，转化为字符串形式的代码
- 执行完成之后，回到入口函数 `complieToFunctions()`
  - 继续把字符串代码转化为函数
  - 调用 `createFunctions()`
  - 初始化 `render` 和 `staticRenderFns` , 挂载到 Vue 实例的 `options` 对应的属性中
