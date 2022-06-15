---
date created: 2022-05-30 22:49
---

#React

##  v16

16版本进行了大量升级, 最显著的更新就是 Fiber 架构的引入, 双缓存系统, 调度器更新, 

##  v17

17 版本没哟显著更新, 主要更新在事件系统, React 17 版本不再对文档级别添加事件处理程序, 而是将事件系统绑定在 React 的根节点
```js
// v16
document.addEventListener()
// v17
rootNode.addEvenetListener()
```

17版本还支持新的 JSX 转换

## v18

全新的concurrentMode 根API, `createRoot` 创建APP. 该模式通过渲染可中断来修复阻塞渲染限制, React 可以在 concurrent 模式中同事更新多个状态. 

更好的服务端渲染支持, 提供 `hydrateRoot` 水合 , 
`setState` 全面支持批处理, 在之前的版本中, React 只能在组件的生命周期函数或合成事件函数中批处理, v18 版本将 `setState` 默认将所有更新都进行批处理. 之前版本在 `Promise` `setTimeout` 以及原生事件中还是不会进行批处理
新增 API `flushSync` 提供退出批处理, 获得回调函数中的更新值

新增 `startTransition` 标记非紧急更新, 用于手动分配高低优先级任务. 与 concurrent mode配合实现可中断渲染机制

组件支持试试更新, server component 在服务端执行, 组件能拥有完整的服务端能力
### 新 Hooks
`useDeferredValue` : 让一个 state 延迟生效, 只有当前没有紧急更新时, 该值才会变为最新值, 也是跟 `startTransition` 一样标记非紧急更新
新增 `useId()` , 用于生成唯一ID, 避免 hydration 不兼容, id代表组件在组件树中的层级结构

`useSyncExternalStore`: 比较安全的读取外接数据源, 一般第三方状态库使用该 hook

`useInsertionEffec`: 作用在 DOM 节点生成之后, `useLayoutEffect` 之前, 一般用于提前注入 style 脚本
``