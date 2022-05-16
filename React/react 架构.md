---
date created: 2022-05-16 22:05
date updated: 2022-05-17 01:23
---
#React 
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

## 总结

React 16 重构了核心算法, 在调度器 Reconciler 中采用了全新的 Fiber Reconciler , 关于 Fiber 详看 [[react Fiber 架构]]
