---
date created: 2022-05-24 00:16
date updated: 2022-05-25 22:43
---

#react

## 相关问题
- useMemo 如何检测依赖项的变化
- hooks 的依赖项比较策略
## 动机

react Hooks 的出现源于在 react 之前的版本中, 组件的状态没有一个很好的行为去复用逻辑, 当然我们可以用 [[react HOC, render props, Hooks]] 去重新组织组件的内部行为. 但是它们的方法会让组件维护变得比较麻烦, 形成"嵌套地狱".

### 复杂组件变得难以理解

react 之前版本在维护组件过程中, 考虑状态逻辑和副作用一直是一个通点, 在生命周期中我们会使用 `componentDidMount` 或者 `componentDidUpdate` 拿到数据, 但是在同一个周期中可能会包含其它的逻辑. 这些行为可能要在在组件的生命周期中清除, 这就容易引发组件之间共享状态的不一致, 导致 bug

**Hooks 的出现可以让我们无需修改组件结构的情况下, 复用状态逻辑**. Hooks 是函数式的, 并非强制由生命周期划分不同的行为. 而且在 class component 中关于 `this` 的指向会让编写过程中增加更多的心智负担, hooks 的出现可以让我们放心地直接使用函数式组件, 使用更多的 react 特性. 毕竟 react 的设计更像函数式编程

## Hooks 的使用

Hooks 使用闭包来保存状态, 使用链表保存一系列的 Hooks, 将链表中的第一个 Hook 与 Fiber 关联. 在 Fiber 树更新时, 就能从 Hooks 中计算出最终输出的状态和执行相关的副作用.
注意在使用过程中:

- 不要在循环, 条件或嵌套函数中调用 Hooks
- 只在 React 函数组件中调用 Hooks

## Hooks 原理

hooks 保存的是组件中通过一些途径产生更新的动作, 这些更新会造成组件 render. Hook 和 update 类似, 都是通过链表链接, 但是它是一组无环的单向链表. 在 Hook 工作过程中会创建 update 动作, 然后 React 调度更新执行 render,  并且计算值的结果返回新的结果. 在这个过程中, 会判断这次更新是首次 `mount` 还是 `update`.  两者调用的是不同的 Dispatcher 方法. 他们根据 Fiber 节点中  `current === null || current.memoizedState === null` 去区分不同对象的 update 操作.

Hook 的数据结构表示如下:
```ts
const hook: Hook = {
  memoizedState: null,

  baseState: null,
  baseQueue: null,
  queue: null,

  next: null,
};
```
其中 `memoizedState` 保存的是单一 `hook` 对应的数据, 不同类型 `hook` 的 `memoizedState` 保存不同类型的数据:

- useState: 对于 `const [state, updateState] = useState(initialState)` , `memoizedState` 保存 `state` 的值
- useReducer: 对于 `const [state, dispatch] = useReducer(reducer, {});` , `memoizedState` 保存 `state` 的值
- useRef: `memoizedState` 保存包含 `useEffect 回调函数 `, ` 依赖项 ` 等的链表数据结构 `effect` , 该创建过程会同事将 effect 链表保存在 `fiber.updateQueue` 中
- useMemo: 对于 `useMemo(callback, [depA])` , `memoizedState` 保存 `[callback(), depA]`
- userCallback: 对于 `useCallback(callback, [depA])` , `memoizedState` 保存 `[callback, depA]` . 与 `useMemo` 的区别是, `useCallback` 保存的是 `callback` 函数本身, 而 `useMemo` 保存的是 `callback` 函数的执行结果
- 有些 hook 是没有 `memoizedState` 的, 比如: `useContext`

### useState 和 useReducer

两个 Hook 在 Fiber 执行 beginWork 时会调用 renderWithHooks 方法, 方法内部会执行 FunctionComponent 对应函数(fiber.type)
1.  mount 场景
    useState 调用 mountState ; useReducer 调用 mountReducer. 他们在内部会创建并返回当前的 hook, 创建 queue, 再创建 dispatch 返回. 两个 hook 区别在于 queue 参数

 [参考链接](https://juejin.cn/post/7101486767336849421)
