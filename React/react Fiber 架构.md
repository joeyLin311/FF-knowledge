---
date created: 2022-05-17 01:23
date updated: 2022-05-25 22:43
---

#React

## 相关问题

- react 为什么需要 fiber？
- react fiber 有哪些技术点, 是怎么做到的？
- [为什么Vue不需要fiber](https://mp.weixin.qq.com/s?__biz=Mzk0MDMwMzQyOA==&mid=2247494750&idx=1&sn=7f74887f6e550c9307c265b0eae8d5e1&chksm=c2e11975f596906366283fcb87fa2afddd9a2b54f08ab1f0539f23a06177ff26b38e4588ff2d&scene=132#wechat_redirect)

## 太长不看

Fiber 是 react 16 中采用的新协调引擎, 它的主要目标是支持虚拟DOM的渐进式渲染.

Fiber 将原有的栈式调度改为 Fiber 纤进程式调度, 主要达成了以下目标:

- 对大型复杂任务的分片
- 对任务划分优先级, 优先调度高优先级的任务
- 调度过程中, 可以对任务进行挂起, 恢复, 终止等操作. 以支持react 执行过程中的局部刷新
- 支持 `render()` 返回多个元素
- 更好地支持 error boundary (错误分界)

因为Fiber是全新的调度方式, 任务的更新过程可能会被打断, 所以在编写组件涉及组件更新的时候, render 以及之前的生命周期函数可能会调用多次, 因此在 `shouldComponentUpdate`  注意不能出现副作用.

## Fiber Reconciler 工作原理

Fiber 节点的大致结构如下:

```js
const Fiber Node = {
  tag: TypeOfWork,          // 标识 Fiber 类型
  type: 'div',              // 和 fiber 相关的组件类型
  return: Fiber | null,     // 父节点
  child: Fiber | null,      // 子节点
  sibling: Fiber | null,    // 右边的同级兄弟节点
  alternate: Fiber | null,  // diff 的变化记录所在节点
  ...
}
```

> React 使用**双缓存**来完成 Fiber 树的构建与替换---- 对应着 DOM 树的创建与更新.

**Fiber 主要工作流程:**

- `ReactDOM.render()` 引导 React 启动或调用 `setState()` 的时候开始创建或更新 Fiber 树
- 从根节点开始遍历 Fiber Node Tree, 并且创建 WorkInProgress Tree ( **reconciliation** 阶段)
  - 本阶段可以暂停, 终止, 和重启, 会导致 react 相关相关生命周期重复执行
  - React 会生成至多两颗树, 一颗是代表当前状态的 `current tree`, 一颗是待更新的 `workInProgress tree`
  - 遍历 `current tree`, 重用或更新 Fiber Node 到 `workInProgress tree`, `workInProgress` 完成更新后会替换 `current tree`
  - 每更新一个节点,  同时生成该节点对应的 Effect List
  - 为每个节点创建更新任务
- 将创建的更新任务加入任务队列, 等待调度
  - 调度由 Scheduler 模块完成, 其核心职责是执行回调
  - Scheduler 模块实现了跨平台兼容的 `requestIdleCallback`
  - 每处理完成一个 Fiber Node 的更新, 可以中断, 挂起, 或恢复
- 根据 Effect List 更新DOM (**commit** 阶段)
  - React 会遍历 Effect List 将所有变更一次性更新到 DOM 上
  - 这一阶段的工作会导致用户可见的变化. 因此该过程不可终端, 必须一直执行到更新完成.

![[React调度流程图.png]]

## Fiber  Tree 的数据结构

Fiber Tree 采用单链表结构, 上述的 Fiber Node 表明了每个Fiber Node都有两个指针域, 一个`child` 指向节点的第一个子节点, 一个`sibling` 指向右边的兄弟节点

## 深入解释 Fiber 的过程和策略
[[react Render 流程]]
详见 [参考链接](http://www.ayqy.net/blog/dive-into-react-fiber/#articleHeader3)
