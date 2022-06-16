---
date created: 2022-05-24 23:29
date updated: 2022-05-25 22:43
---

# React

## 相关问题

- react setState 是同步还是异步的?
- 为什么 react 两次 setState 只执行了一次?

## 看图

![[setState 机制.jpg]]

### 字段说明

- `partialState`: `setState` 传入的第一个参数，对象或函数
- `_pendingStateQueue`: 当前组件等待执行更新的 `state` 队列
- `isBatchingUpdates`: react 用于标识当前是否处于批量更新状态，所有组件公用
- `dirtyComponent`: 当前所有处于待更新状态的组件队列
- `transcation`: react 的事务机制，在被事务调用的方法外包装 n 个 `waper` 对象，并一次执行: `waper.init`、被调用方法、`waper.close`
- `FLUSH_BATCHED_UPDATES`: 用于执行更新的 `waper`，只有一个 `close` 方法

## 执行过程

1. 将 `setState` 传入的 `partialState` 参数存储在当前组件实例的 state 暂存队列中, 调用的是 `this.updater.enqueueSetState` 方法

- `enqueueSetState` 主要做了两件事:
  1. 将新的 state 添加进租金的状态队列里
  2. 调用 `enqueueUpdate` 处理将要更新的实例对象

2. 判断当前 React 是否处于批量更新状态(`isBatchingUpdates`), 如果是, 则将当前组件加入待更新的组件队列中
3. 如果未处于批量更新状态 (`isBatchingUpdates === false`), 则将批量更新状态标识设置为 true , 用事务 `Transaction` 再次调用前一步方法, 保证当前组件加入到了待更新的组件队列中
4. 调用事务 `Transaction` 的 `waper` 方法, 遍历待更新组件队列, 依次执行更新.
5. 执行生命周期 `componentWillReceiveProps`
6. 将组件的 state 暂存队列中的 `state` 进行合并, 获得最终需要更新的 state 对象, 并将队列置空
7. 执行声明周期 `componentShouldUpdate` , 根据返回值判断是否需要更新
8. 执行生命周期 `componentWillUpdate`
9. 执行真正的更新, `render`
10. 执行生命周期 `componentDidUpdate`

## 总结

### 1. 钩子函数&合成事件中

在 React 的生命周期与合成事件中, react 仍然处于它的更新机制中, 这时 `isBatchingUpdates` 为 `true`.
按照上述执行, 这时无论调用多少次 `setState` , 都将不会执行更新, 而是将要更新的 state 存入 `_isPendingStateQueue` , 将要更新的组件存入 `dirtyComponent`
当上一次更新机制执行完毕, 以生命周期为例, 所有组件, 即最顶层组件 `didmount` 后会将 `isBranchUpdate` 设置为 false. 这时将执行之前积累的 `setState`

### 2. 异步函数和原生事件中

从执行机制看, `setState` 本身并不是异步的. 但是如果在调用 `setState` 过程中, 如果 react 处于更新过程, 当前的更新就会被暂存, 等上一次更新之后再执行, 这个过程给人一种异步的假象.
在生命周期中, 根据 JS 的异步机制, 会将异步执行函数先暂存, 等所有同步代码执行完毕后再执行, 这时上一次更新过程已经执行完毕, `isBatchUpdate` 被设置为 false, 根据流程图, 此时调用 `setState` 即可执行更新并拿到更新结果

### 3. `partialState` 合成机制

主要关注 `_processPendingState` 方法, 该函数是用来合并 `state` 暂存队列, 最后返回一个合并后的 `state`

```jsx
_processPendingState: function(props, context) {
  var inst = this._instance; 
  var queue = this._pendingStateQueue; 
  var replace = this._pendingReplaceState;
  // ... 省略代码
  for(var i = replace ? 1 : 0; i < queue.length; i++) {
    var partial = queue[i];
    /* 主要关注这一行代码 */
    _assign(
    nextState, 
    typeof partial === 'function' ? 
        partial.call(inst, nextState, props, context) : 
        partial
    );
    
  }
  
}
// 如果是对象只会被合并成一次
Object.assgin(
  nextState,
  {index: state.index + 1},
  {index: state.index + 1}
)

```

可以看到如果传入的是函数, 函数的参数 preState 是前一次合并后的结果. 如果传入的是对象, 很明显会被合并成一次.

### 3. `componentDidMount` 调用 `setState`

> 在 componentDidMount() 中, 你可以立即调用 setState(). 它将会触发一次额外的渲染, 但是它将在浏览器刷新屏幕之前发生. 这保证了它在此情况下即使 render() 会调用两次, 用户也不会看到中间状态. 慎用此模式, 因为它经常导致性能问题. 在大多数情况下, 你可以在 constructor() 中使用赋值初始状态来代替. 然而, 有些情况下必须这样, 比如像模态框和工具提示框. 这时, 你需要先测量这些 DOM 节点, 才能渲染依赖尺寸或者位置的某些东西.

以上是官方的文档说明, 不推荐在 `componentDidMount` 中直接调用 `setState`, 由上面的分析, `componentDidMount` 本身处于一次更新中, 我们又调用了一次 `setState`, 就会在未来再进行一次 `render`, 造成不必要的性能浪费, 多数情况下可以设置初始值来搞定.
当然我们可以在 `componentDidMount` 中调用接口, 再在回调中修改 `state`, 是可行的
当 `state` 初始值依赖 DOM 属性时, 在 `componentDidMount` 中调用 `setState` 是不可避免的

### 5. `componentWillUpdate` & `componentDidUpdate`

在这两个生命周期中不能调用 `setState` , 因为通过上述流程图可以看出, 如果调用 `setState` 会造成死循环

### 6. 推荐使用方式

在调用 `setState` 时使用函数传递 `state` 值, 在回调函数中获取最新更新后的 `state`
