---
date created: 2022-05-24 15:46
date updated: 2022-05-25 22:44
---
#React 

## useState

用于创建一个新的状态, 参数为一个固定的值或者一个有返回值的方法. 钩子执行后的结果为一个数组, 分别生成状态以及改变该状态的方法, 通过解构赋值的方法拿到对应的值与方法

## useEffect

执行副作用钩子, 有一下两种情况:

1. 函数式组件中不存在传统 class component 中的生命周期概念,如果我们需要在一些特定的生命周期或者值变化后做一些操作的话, 必须借助 `useEffect` 的一些特性去实现
2. `useEffect` 产生的 `changeState` 方法并没有提供类似于 `setState` 的第二个参数一样的功能,  因此如果需要在 state 改变后执行一些方法, 必须通过 `useEffect` 去实现

该方法接收两个参数, 第一个参数为副作用需要执行的回调函数, 生成的回调方法可以返回一个函数(`在组件卸载时运行`); 第二个参数为该副作用钩子监听的状态数组, 当对应的状态发生改变时, 执行前面的回调函数, **如果第二个参数为空, 那么在每一个 state 变化时都会执行该副作用函数的回调函数**

[[react useEffect 经典问题和实践总结]]
### 如何通过 useEffect 实现类似生命周期的功能?

1. `componenDidMount` & `componentWillMount` 这两个生命周期只在页面挂载/卸载后执行一次, `useEffect` 的执行时机是在组件卸载时运行. 所以可以让副作用函数的回调作用相关联的 state 为空数组, 不管其它状态如何变动, 该副作用函数都不会再执行. 这样就实现了相似的 class component 生命周期.
2. `componentDidUpdate`: 该生命周期在每次页面更新后都会被调用. 按照之前的逻辑, 可以把所有的状态都放在 `useEffect` 的第二个参数中, 所以可以不传递第二个参数, 让每一个 state 变化时都执行该副作用函数, 这样就实现了 `componentDidUpdate` 的平替.

## useCallback

生成 callback 的钩子函数, 用于对不同的 `useEffect` 中存在相同的逻辑封装, 减少代码冗余, 配合 `useEffect` 使用.

```jsx
const [count1, changeCount1] = useState(0);  
const [count2, changeCount2] = useState(10);  
  
const calculateCount = useCallback(() => {  
  if (count1 && count2) {  
    return count1 * count2;  
  }  
  return count1 + count2;  
}, [count1, count2])  
  
useEffect(() => {  
    const result = calculateCount(count, count2);  
    message.info(`执行副作用，最新值为${result}`);  
}, [calculateCount])
```

和单独使用 `useEffect` 不同, `useCallback` 生成计算的回调函数后, 会传入 `useEffect` 的第二个参数中.

## useRef

`useRef` 接受一个参数, 为 ref 的初始值. 它通常可以保存 DOM 之类的数据, 需要被"引用" 的数据都可以保存在 ref 中. 该钩子会返回一个对象, 对象中的 `current` 字段为我们 **指向的实例/保存的变量**, 可以实现获得目标节点实例或保存状态的功能.
`useRef` 保存的保存的变量不会随着每次数据的变化而重新生成, 而是保持在我们最后一次赋值时的状态, 依靠这种特性, 配合 `useCallback` 和 `useEffect` 可以实现 `preState/preProps` 的功能.

```jsx
const [count, changeCount] = useState(0);  
const [count1, changeCount1] = useState(0);  
// 创建初始值为空对象的prestate  
const preState = useRef({});  
// 依赖preState进行判断时可以先判断，最后保存最新的state数据  
useEffect(() => {  
  const { ... } = preState.current;  
  if (// 条件判断) {  
    // 逻辑  
  }  
  // 保存最新的state  
  preState.current = {  
    count,  
    count1,  
  }  
});
```

`useRef` 仅仅返回一个包含 `current` 属性的对象. 修改 ref 的值并不会引起视图的变化.

## useMemo

Memo 是 Memory 的缩写, `useMemo` 即是使用记忆的内容. 该函数钩子主要用于做性能的优化.
当 state 变化时, 没有设置关联状态的 `useEffect` 会全部执行,  同样的, 通过计算出来的值或者引入的组件也会重新计算/挂载一遍, 即使与其关联的状态没有发生任何变化.

> 为了帮助我们实现性能优化,  `useMemo` 可以处理计算结果的缓存或引入组件的防止重复挂载优化. 它接受两个参数: 第一个参数为 Getter 方法, 返回值为要缓存的数据或组件, 第二个参数为该返回值相关联的状态, 当其中任何一个状态发生变化时就会重新调用 Getter 方法生成新的返回值.

`useMemo` 并不关心传入的数据是什么, 只关心关联状态发生变动时生成新的返回值, 也就是**它生成的是 Getter 方法与依赖数组的关联关系**, 所以通常可以传入一个组件, **实现对组件挂载/重载的性能优化**  [[useMemo 使用场景以及解读]]

![[reactHooks 总结.jpg]]
