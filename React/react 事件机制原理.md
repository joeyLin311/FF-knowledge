---
date created: 2022-05-30 22:44
---
#React 
## React 事件的注册

使用 `ReactDOM.createRoot` 创建 Root 时, React 会调用 `listenToAllSupportedEvents` 方法对搜索有支持的事件进行监听:

1.  `allNativeEvents` 用于收集所有合成事件相关联的原生时间名. 这个收集动作在**插件初始化阶段** 完成
2.  对每个原生事件调用 `addTrappedEventListener` 函数. 该函数最终使用 `addEventListener` 方法对原生事件进行捕获或冒泡阶段的事件监听注册

### 总结:

基于上述流程可知, 调用 `ReactDOM.createRoot` 方法的时候, 就已经在 root 节点上初始化所有原生事件的监听回调函数. 而不是在组件上写合成事件的监听阶段才开始注册事件回调.
![[react 事件注册.png]]

## React 事件的触发

- 在注册事件阶段调用的 `addTrappedEventListener` 方法中, 会使用 `createEventListenerWrapperWithPriority` 函数来创建事件回调.
- `createEventListenerWrapperWithPriority` 函数根据事件类型, 划分出若干个不同优先级的 `dispatchEvent`. 事件回调最终都进入该方法调用.

触发原生事件后 React 处理事件的流程大致如下:

1.  原生事件触发后, 进入 `dispatchEvent` 回调方法
2.  `attemptToDispatchEvent` 方法更具该原生事件查找到当前原生 DOM 节点和映射的 Fiber 节点
3.  事件和 Fiber 等信息被派发给插件系统进行处理, 插件系统调用各插件暴露的 `extractEvents` 方法
4.  `accumulateSinglePhaseListeners` 方法向上收集 Fiber 树上监听相关事件的其它回调函数, 构造出合成事件对象并加入到派发队列 `dispatchQueue` 中
5.  调用 `processDispatchQueue` 方法, 基于捕获或冒泡阶段的表示, 按倒序或顺序执行 `dispatchQueue` 中的方法

![[react 事件触发.png]]
