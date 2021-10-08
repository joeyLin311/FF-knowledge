#Vue 
# 计算属性VS侦听属性
## computed 计算属性
计算属性的初始化是发生在 Vue 实例初始化阶段的 `initState` 函数中，执行了 `if (opts.computed) initComputed(vm, opts.computed)`，`initComputed` 的定义在 `src/core/instance/state.js` 中

## watch 侦听属性
侦听属性的初始化也是发生在 Vue 的实例化初始阶段的 `initState` 函数中, 在 `computed` 初始化后, 执行了
```js
if(opts.watch && opts.watch !== nativeWatch) {
  initWatch(vm, opts.watch)
}
```

`initWatch` 的实现, 定义在 `src/core/instance/state.js` 中

主要就是对 `watch` 对象进行遍历, 拿到每一个 `handler` , 因为 Vue 是支持 `watch` 的同一个 `key` 对应多个 `handler` , 所以如果 `handler` 是一个数组, 则遍历这个数组, 调用 `createWatcher` 方法, 否则直接调用 `creatWatcher`

### 关于 deep watcher
在watch的表达式中设置 `deep: true` 就可以实现对该对象的深层侦听. 在对 watch 的表达式或函数求值后, 会调用 `traverse` 函数, 它的定义在 `src/core/observer/traverse.js` 中.
它的作用是对一个对象做深层递归遍历, 遍历的过程中对子对象的访问触发了 getter , 这样就可以收集到以来, 也就是订阅到它们的 `watcher` , 该方法中有个小优化: 遍历过程中会把子响应式对象通过它们的 `dep id` 记录到 `seenObjects` , 避免以后重复访问.
