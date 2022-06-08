---
date created: 2022-05-16 22:05
date updated: 2022-06-08 21:52
---

#React

## 相关问题

- react 16 版本更新了什么, 主要有哪些变化?
- react 优化了哪些问题

## React 16 架构可分为三层:

- Scheduler (调度器) ---- 调度任务优先级, 高优先级任务先进入 Reconciler
- Reconciler (协调器) ---- 负责找出变化的组件
- Renderer (渲染器) ---- 负责将变化的组件渲染到页面上

## Scheduler 调度器

部分浏览器实现了 `requestIdleCallback()` 的 Web API, 该 API 的作用同 React 的 Scheduler 异曲同工: 以此实现在JS执行事件循环过程中主动协调任务调度.

但是该API并没有得到广大浏览器厂商的支持, 至今仍旧是实验性API([MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestIdleCallback)).

React 放弃了该API的使用, 进而实现了功能更加完备的 `requestIdleCallBack()` polyfill, 就是 Scheduler, 它除了在空闲时触发回调的功能外, 还提供了多种调度优先级供与任务设置.

## Reconciler 协调器

在React 15 中 Reconciler 是递归处理虚拟 DOM 的, React 16 中, 协调器的工作方式从递归变成了可以终端的循环过程. 每次循环都会调用 `shouldYield` 判断当前是否有剩余时间.

在 React 16 中 Reconciler 和 Renderer 不再是交替工作, 而是通过上述的 Scheduler 派发任务,  再由协调器统一处理组件的协调工作, 再交给 Renderer.

## Renderer 渲染器

Renderer 根据 Reconciler 为虚拟DOM打的标记, 同步执行对应DOM 的操作
![[react-renderer.png]]

## React 启动模式

React 有三种模式进入主体函数的入口,他们会影响整个应用的工作方式, 具体可以参阅官方文档:

- `legacy` : 当前 React 版本采用的方式, 但是这个模式可能不支持一些新功能
- `blocking` : 开启部分 `concurrent` 模式特种的中间模式, 目前正在实验中
- `concurrent` : 未来 React 的开发模式, Fiber 架构中的 `任务中断/任务优先级` 都是针对 `concurrent` 模式提出的.

```js
// 三者的入口函数
// legacy 
ReactDOM.render(<App />, rootNode);
// blocking
ReactDOM.createBlockingRoot(rootNode).render(<App />);
// concurrent
ReactDOM.createRoot(rootNode).render(<App />)
```

三者有个重要的区别是**自动批处理功能**, `legacy` 模式在合成事件中有自动批处理的功能, 但仅限于一个浏览器任务. 非 React 事件想使用这个功能必须使用 `unstable_batchedUpdates` . 在 `blocking` 和 `concurrent` 模式里, `setState` 都是模式自动批处理的.

## 总结

React 16 重构了核心算法, 在调度器 Reconciler 中采用了全新的 Fiber Reconciler , 关于 Fiber 详看 [[react Fiber 架构]]
