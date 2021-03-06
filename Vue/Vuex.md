---
date created: 2021-12-09 22:53
date updated: 2021-12-17 16:25
---

#Vue

## vuex 原理

![[Vuex原理图.png]]
Vuex 应用的核心就是 store（仓库）。“store”基本上就是一个容器，它包含着你的应用中大部分的状态 (state)。有些同学可能会问，那我定义一个全局对象，再去上层封装了一些数据存取的接口不也可以么？

Vuex 和单纯的全局对象有以下两点不同：

- Vuex 的状态存储是响应式的。当 Vue 组件从 store 中读取状态的时候，若 store 中的状态发生变化，那么相应的组件也会相应地得到高效更新。
- 你不能直接改变 store 中的状态。改变 store 中的状态的唯一途径就是显式地提交 (commit) mutation。这样使得我们可以方便地跟踪每一个状态的变化，从而让我们能够实现一些工具帮助我们更好地了解我们的应用。

另外，通过定义和隔离状态管理中的各种概念并强制遵守一定的规则，我们的代码将会变得更结构化且易维护。

## 相关问题

- 思路分析：3w1h（what why where how）
  - 首先给 vuex 下一个定义
  - vuex 解决了哪些问题，解读理念
  - 什么时候我们需要 veux
  - 你的具体用法
  - 简述原理，提升层级
- 回答范例：
  - vuex 是 vue 专用的状态管理库，它以全局方式集中管理应用的状态，并且可以保证状态变更的可预测性
  - vuex 主要解决的问题是多组件之间状态共享的问题，利用各种组件通信方式，我们虽然能够做到状态共享，但是往往需要在多个组件之间保持状态的一致性，这种模式很容易出现问题，也会是程序逻辑变得复杂。vuex 通过吧组件的共享状态抽取出来，以全局单例模式管理，这样任何组件都能用一致的方式获取和修改状态，响应式的数据也能够保证简介的单向数据流动，我们的代码将变得更结构化且易维护
  - vuex 并非必须的，它帮助我们管理共享状态，但却带来更多的概念和框架。如果我们不打算开发大型单页应用或者我们的应用并没有大量全局的状态需要维护，完全没有使用 vuex 的必要。一个简单的 store 模式就足够了。反之，vuex 将会成为自然而然的选择。引用 Redux 的作者 Dan Abramov 的话说就是：Flux 构架就像眼镜：您会知道什么时候需要它
  - 我在使用 vuex 过程中有如下理解：首先是对核心概念的理解和运用，将全局状态放入 state 对象中，它本身是一棵状态树，组件中使用 store 实例的 state 访问这些状态；然后有配套的 mutation 方法修改这些状态，并且只能用 mutation 修改状态，在组件中调用 commit 方法提交 mutation；如果应用中有异步操作或者复杂逻辑组合，我们需要编写 action，执行结束如果有状态修改仍然需要提交 mutation，组件中调用这些 action 使用 dispatch 方法派发。最后是模块化，通过 modules 选项组织拆分出去的各个子模块，在访问状态时注意添加子模块的名称，如果子模块有设置 namespace，那么在提交 mutation 和派发 action 时还需要额外的命名空间前缀
  - vuex 在实现单项数据流时需要做到数据的响应式，通过源码的学习发现是借用了 vue 的数据响应化特性实现的，它会利用 vue 将 state 作为 data 对其进行响应化处理，从而使得这些状态发生变化时，能够导致组件重新渲染
