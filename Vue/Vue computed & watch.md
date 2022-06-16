---
date created: 2021-12-09 22:57
date updated: 2021-12-17 16:05
---

# Vue

# 计算属性 VS 侦听属性

## computed 计算属性

计算属性的初始化是发生在 Vue 实例初始化阶段的 `initState` 函数中，执行了 `if (opts.computed) initComputed(vm, opts.computed)`，`initComputed` 的定义在 `src/core/instance/state.js` 中

### computed 原理

首先得讲 vue 响应式原理，因为 computed 的实现是基于 Watcher 对象的。 那么 vue 的响应式原理是什么呢，众所周知，vue 是基于 Object.defineProperty 实现监听的。在 vue 初始化数据 data 和 computed 数据过程中。会涉及到以下几个对象：

1. Observe 对象
2. Dep 对象
3. Watch 对象 Observe 对象是在 data 执行响应式时候调用，因为 computed 属性基于响应式属性，所以其不需要创建 Observe 对象。 Dep 对象主要功能是做依赖收集，有个属性维护多个 Watch 对象，当更新时候循环调用每个 Watch 执行更新。 Watch 对象主要是用于更新，而且是收集的重点对象。

这里谈到 computed 计算属性，首先要知道，其有两种定义方式，一种是方法，另一种是 get，set 属性。而且，其内部监听的对象必须是已经定义响应式的属性，比如 data 的属性 vuex 的属性。
vue 在创建 computed 属性时候，会循环所有计算属性，每一个计算属性会创建一个 watch，并且在通过 defineProperty 定义监听，在 get 中，计算属性工作是做依赖收集，在 set 中，计算属性重要工作是重新执行计算方法，这里需要多补充一句，因为 computed 是懒执行，也就是说第一次初始化之后，变不会执行计算，下一次变更执行重新计算是在 set 中。

另一个补充点是依赖收集的时机，computed 收集时机和 data 一样，是在组件挂载前，但是其收集对象是自己属性对应的 watch，而 data 本身所有数据对应一个 watch。

## watch 侦听属性

侦听属性的初始化也是发生在 Vue 的实例化初始阶段的 `initState` 函数中, 在 `computed` 初始化后, 执行了

```jsx
if(opts.watch && opts.watch !== nativeWatch) {
  initWatch(vm, opts.watch)
}
```

`initWatch` 的实现, 定义在 `src/core/instance/state.js` 中

主要就是对 `watch` 对象进行遍历, 拿到每一个 `handler` , 因为 Vue 是支持 `watch` 的同一个 `key` 对应多个 `handler` , 所以如果 `handler` 是一个数组, 则遍历这个数组, 调用 `createWatcher` 方法, 否则直接调用 `creatWatcher`

### 关于 deep watcher

在 watch 的表达式中设置 `deep: true` 就可以实现对该对象的深层侦听. 在对 watch 的表达式或函数求值后, 会调用 `traverse` 函数, 它的定义在 `src/core/observer/traverse.js` 中.
它的作用是对一个对象做深层递归遍历, 遍历的过程中对子对象的访问触发了 getter , 这样就可以收集到以来, 也就是订阅到它们的 `watcher` , 该方法中有个小优化: 遍历过程中会把子响应式对象通过它们的 `dep id` 记录到 `seenObjects` , 避免以后重复访问.
