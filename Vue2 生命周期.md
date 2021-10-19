#Vue
https://ustbhuangyi.github.io/vue-analysis/v2/components/lifecycle.html#beforecreate-created
![[Vuelifecycle.png]]
## beforeCreate&created
`beforeCreate` 和 `created` 函数都是在实例化 Vue 的阶段, 在 `_init` 方法中执行的. 定义在 `src/core/instance/init.js` 中
可以看到 `beforeCreate` 和 `created` 的钩子调用是在 `initState` 的前后，`initState` 的作用是初始化 `props`、`data`、`methods`、`watch`、`computed` 等属性，之后我们会详细分析。那么显然 `beforeCreate` 的钩子函数中就不能获取到 `props`、`data` 中定义的值，也不能调用 `methods` 中定义的函数。
在这俩个钩子函数执行的时候，并没有渲染 DOM，所以我们也不能够访问 DOM，一般来说，如果组件在加载的时候需要和后端有交互，放在这俩个钩子函数执行都可以，如果是需要访问 `props`、`data` 等数据的话，就需要使用 `created` 钩子函数。之后我们会介绍 vue-router 和 [[vuex]] 的时候会发现它们都混合了 beforeCreate 钩子函数。

## beforeMount&mounted
`beforeMount` 钩子函数发生在 `mount`，也就是 DOM 挂载之前，它的调用时机是在 `mountComponent` 函数中. 定义在 `src/core/instance/lifecycle.js`
### beforeUpdate
`beforeUpdate` 的执行时机是在渲染 Watcher 的 `before`函数中, 也就是在组件已经 `mounted`之后, 才会去调用这个钩子函数
### updated
`update` 的执行时机是在 `flushSchedulerQueue` 函数调用的时候，它的定义在 `src/core/observer/scheduler.js` 中

## beforeDestroy& destroyed
`beforeDestroy` 和 `destroyed` 钩子函数的执行时机在组件销毁的阶段，组件的销毁过程之后会详细介绍，最终会调用 `$destroy` 方法，它的定义在 `src/core/instance/lifecycle.js` 中：
`beforeDestroy` 钩子函数的执行时机是在 `$destroy` 函数执行最开始的地方，接着执行了一系列的销毁动作，包括从 `parent` 的 `$children` 中删掉自身，删除 `watcher`，当前渲染的 VNode 执行销毁钩子函数等，执行完毕后再调用 `destroy` 钩子函数。
在 `$destroy` 的执行过程中，它又会执行 `vm.__patch__(vm._vnode, null)` 触发它子组件的销毁钩子函数，这样一层层的递归调用，所以 `destroy` 钩子函数执行顺序是先子后父，和 `mounted` 过程一样。

## activated & deactivated
`activated` 和 `deactivated` 钩子函数是专门为 `keep-alive` 组件定制的钩子，我们会在介绍 `keep-alive` 组件的时候详细介绍 [[keepAlive原理#keepAlive生命周期]]

## Vue中组件生命周期调用顺序说一下

组件的调用顺序都是`先父后子`,渲染完成的顺序是`先子后父`。
组件的销毁操作是`先父后子`，销毁完成的顺序是`先子后父`。
### 加载渲染过程
```
父beforeCreate->父created->父beforeMount->子beforeCreate->子created->子beforeMount- >子mounted->父mounted
```

### 子组件更新过程
```
父beforeUpdate->子beforeUpdate->子updated->父updated
```

### 父组件更新过程
```
父 beforeUpdate -> 父 updated
```

### 销毁过程
```
父beforeDestroy->子beforeDestroy->子destroyed->父destroyed
```