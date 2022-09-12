---
date created: 2021-12-09 22:53
date updated: 2021-12-17 16:20
---

#Vue

## [[Vue diff 算法]]

## [[Vue2 响应式原理]]

## [[Vue2 模板编译过程]]

## [[Vue2 生命周期]]

## [[Vue router 原理]]

## [[Vue keepAlive 组件原理]]

## [[Vue $nextTick]]

## [[Vue 3]]

## [[Vue computed & watch]]

## vue 组件通信的常用方式有那些?

### [[Vue props]]
### 自定义事件[[Vue event]]
### [[Vue eventBus]]
### [[Vuex]]
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

## 为什么 data 返回一个函数而不是一个对象?
- 根实例对象`data`可以是对象也可以是函数（根实例是单例），不会产生数据污染情况
- 组件实例对象`data`必须为函数，目的是为了防止多个组件实例对象之间共用一个`data`，产生数据污染。采用函数的形式，`initData`时会将其作为工厂函数都会返回全新`data`对象

[Vue各种问题](https://juejin.cn/post/6844903918753808398#heading-1)

## [[Vue SSR 实现原理]]

## [[Vue 30 面试题]]
