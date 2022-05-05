---
date created: 2021-12-09 22:58
---

#JavaScript

## 前端模块化一览

历史由来: 随着 web 技术的快速发展, 前端代码在规模和组织上日益扩大和复杂.  代码复杂度也日益增高, 越来越多的业务逻辑和交互都需要在 web 层实现, 代码一多, 各种命名冲突, 代码冗余, 文件间的依赖关系不清导致了一系列棘手的问题, 所以代码模块历程就出现了.

### JS 模块化

JS 语言早期并没有模块化编程的概念, 早期的模块化方案有:

- 普通函数
- 命名控件
- 立即执行函数(IIFE)
- 依赖注入(?)

### 模块化一览

CommonJS、AMD、CMD、UMD、ESM 方案相继推出
![[JS模块化一览.png]]

- **CommonJS**[[JS modules#common JS]]: 主要是 **Node.js** 使用，通过 `require` **同步**加载模块，`exports` 导出内容。
- **AMD**[[JS modules#AMD 规范]]: 主要是**浏览器端**使用，通过 `define` 定义模块和依赖，`require` **异步**加载模块，推崇**依赖前置**。
- **CMD**[[JS modules#CMD 规范]]: 和 AMD 比较类似，主要是**浏览器端**使用，通过 `require` **异步**加载模块，`exports` 导出内容，推崇**依赖就近**。
- **UMD**[[JS modules#UMD 规范 commonJS AMD]]: 通用模块规范，是 CommonJS、AMD 两个规范的大融合，是**跨平台**的解决方案。
- **ESM**[[JS modules#ES Modules]]: 官方模块化规范，**现代浏览器原生支持**，通过 `import` **异步**加载模块，`export` 导出内容。
