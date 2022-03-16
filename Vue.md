---
date created: 2021-12-09 22:53
date updated: 2021-12-17 16:20
---

#Vue

## [[Vue diff 算法]]

## [[Vue2 响应式原理]]

## [[vue2 模板编译过程]]

## [[Vue2 生命周期]]

## [[vue router 原理]]

## [[keepAlive 原理]]

## [[nextTick]]

## [[vue2 vs vue3]]

## [[computedVSwatch]]

## vue 组件通信的常用方式有那些?

### [[props]]
### 自定义事件[[vue event]]
### [[eventBus]]
### [[vuex]]
### 边界情况的 `$parent` `$root` `$refs`, 依赖注入 provide/inject
### `$attrs` `$listeners`

## 什么是 mvvm，mvc？区别？

### MVC（Model-View-Controller）

MVC 是比较直观的架构模式，用户操作
->View（负责接收用户的输入操作） ->Controller（业务逻辑处理）-> Model（数据持久化）->View（将结果反馈给 View）。MVC 使用非常广泛，比如 JavaEE 中的 SSH 框架。

### MVVM（Model-View-ViewModel）

如果说 MVP 是对 MVC 的进一步改进，那么 MVVM 则是思想的完全变革。
它是将“数据模型数据双向绑定”的思想作为核心，因此在 View 和 Model 之间没有联系，
通过 ViewModel 进行交互，而且 Model 和 ViewModel 之间的交互是双向的，
因此视图的数据的变化会同时修改数据源，而数据源数据的变化也会立即反应 view。
微信小程序前端使用 mvvm。
