---
date created: 2022-05-25 22:43
---
#React 

## 概览

react 的 render 阶段开始于, `performSyncWorkOnRoot` 或者 `performConcurrentWorkOnRoot` 方法的调用, 取决于本次更新是同步更新还是异步更新.

`performUnitOfWork` 是其中调用的关键方法, 它会创建下一个 Fiber 节点并赋值给 `workInProgress` , 并将 `workInProgress` 与已创建的 Fiber 节点连接起来构成 Fiber 树.

总体来看, render 阶段是通过遍历的方式实现可中断的递归, 所以 `performUnitOfWork` 的工作可以分为: "递" 和 "归" 两个过程.

## "递" 和 "归"

### "递" 阶段

首先从 `rootFiber` 开始向下深度有限遍历. 为遍历到的每个 Fiber 节点 调用 `beginWork` 方法. 该方法会根据传入的 Fiber 节点创建 Fiber 子节点, 并将这两个 Fiber 节点连接起来.

`beginWork` 的工作可以分为两部分:

- `update` 时: 如果 `current` 存在, 在满足一定条件时可以复用 `current` 节点, 这样就能克隆 `current.child` 作为 `workInProgress.child`, 而不需要新建 `workInProgress.child`
- `mount` 时: 除了 `fiberRootNode` 以外, `current === null`. 会根据 `fiber.tag` 的不同, 创建不同类型的 子 Fiber 节点
- [详情-react 技术揭秘](https://react.iamkasong.com/process/beginWork.html#%E6%96%B9%E6%B3%95%E6%A6%82%E8%A7%88)

![[beginWork.png]]

### "归" 阶段

"归" 阶段会调用 `completeWork` 处理 Fiber 节点. 当某个 Fiber 节点执行完 `completeWork` , 如果它存在兄弟 Fiber 节点, 会进入其兄弟 Fiber 的"递" 阶段, 如果不存在兄弟节点, 会进入父 Fiber 节点的"归" 阶段.

递归阶段会交错执行直到访问处理了 rootFiber, 整个 render 阶段的工作就结束了.

`completeWork` 也针对不同的 `fiber.tag` 调用不同的处理逻辑. 然后也根据 `current === null ?` 判断是 `mount` 还是 `update`. 在 `update` 阶段会针对 `HostComponent` 判断是否 Fiber 节点存在对应的 DOM 节点

`update` 时如果 Fiber 节点已经存在对应的 DOM 节点, 不需要生成 DOM 节点, 需要处理 `props` , 比如:

- `onClick` , `onChange` 等回调函数的注册
- 处理 `style prop`
- 处理 `DANGEROUSLY_SET_INNER_HTML prop`
- 处理 `children prop`

然后调用 `updateHostComponent` 方法. 该方法内部, 被处理完的 `props` 会被赋值给 `workInProgress.updateQueue` , 并最终在 `commit` 阶段渲染在页面上

`mount` 时主要的逻辑包括一下三个:

- 为 Fiber 节点生成对应的 DOM 节点
- 将子孙 DOM 节点插入刚生成的 DOM 节点中
- 与 `update` 逻辑中的 `updateHostComponent` 类似地处理 `props` 的过程

`mount` 时只会在 `rootFiber` 存在 `Placement effectTag`。那么 `commit 阶段` 是如何通过一次插入 `DOM` 操作（对应一个 `Placement effectTag`）将整棵 `DOM 树` 插入页面的呢？

原因就在于 `completeWork` 中的 `appendAllChildren` 方法。

由于 `completeWork` 属于“归”阶段调用的函数，每次调用 `appendAllChildren` 时都会将已生成的子孙 `DOM 节点` 插入当前生成的 `DOM 节点` 下。那么当“归”到 `rootFiber` 时，我们已经有一个构建好的离屏 `DOM 树`。

## effectList

至此 `render 阶段` 的绝大部分工作就完成了。

**还有一个问题:** 作为 `DOM` 操作的依据，`commit 阶段` 需要找到所有有 `effectTag` 的 `Fiber 节点` 并依次执行 `effectTag` 对应操作。难道需要在 `commit 阶段` 再遍历一次 `Fiber 树` 寻找 `effectTag !== null` 的 `Fiber 节点` 么？

这显然是很低效的。

为了解决这个问题，在 `completeWork` 的上层函数 `completeUnitOfWork` 中，每个执行完 `completeWork` 且存在 `effectTag` 的 `Fiber 节点` 会被保存在一条被称为 `effectList` 的单向链表中。

`effectList` 中第一个 `Fiber 节点` 保存在 `fiber.firstEffect`，最后一个元素保存在 `fiber.lastEffect`。

类似 `appendAllChildren`，在“归”阶段，所有有 `effectTag` 的 `Fiber 节点` 都会被追加在 `effectList` 中，最终形成一条以 `rootFiber.firstEffect` 为起点的单向链表。

```jsx
                       nextEffect         nextEffect
rootFiber.firstEffect -----------> fiber -----------> fiber
```

这样，在 `commit 阶段` 只需要遍历 `effectList` 就能执行所有 `effect` 了。
![[completeWork.png]]
