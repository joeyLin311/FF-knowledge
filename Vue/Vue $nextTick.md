---
date created: 2021-12-09 22:55
date updated: 2021-12-17 16:15
---

#Vue 

[Vue2 技术揭秘-nextTick](https://ustbhuangyi.github.io/vue-analysis/v2/reactive/next-tick.html#js-%E8%BF%90%E8%A1%8C%E6%9C%BA%E5%88%B6)

## nextTick

> 官方对 nextTick 的描述: 在下次 DOM 更新循环结束之后执行延迟回调. 在修改数据之后立即使用这个方法, 获取更新后的 DOM

```jsx
vm.msg = 'abc';
// DOM 还未更新
Vue.nextTick(function(){
  // DOM 更新了
})

// 作为一个Promise 使用
Vue.nextTick().then(function(){
  // DOM 更新
})
```

当你设置 `vm.msg = 'abc'`, 该组件不会立即重新渲染. 当刷新队列时, 组件会在事件循环队列清空时的下一个“tick”更新. 多数情况我们不需要关心这个过程, 但是如果你想在 DOM 状态更新后做点什么, 这就可能会有些棘手. 虽然 Vue.js 通常鼓励开发人员沿着“数据驱动”的方式思考，避免直接接触 DOM, 但是有时我们确实要这么做. 为了在数据变化之后等待 Vue 完成更新 DOM ,  可以在数据变化之后立即使用 Vue.nextTick(callback) 这样回调函数在 DOM 更新完成后就会调用.

Vue 异步执行 DOM 更新. 只要观察到数据变化, Vue 将开启一个队列, 并缓冲在同一事件循环中发生的所有数据改变. 如果同一个 watcher 被多次触发, 只会被推入到队列中一次. 这种在缓冲时去除重复数据对于避免不必要的计算和 DOM 操作上非常重要. 然后在下一个的事件循环"tick"中, Vue 刷新队列并执行实际 (已去重的) 工作.
Vue 在内部尝试对异步队列使用原生的 `Promise.then` 和 `MessageChannel`, 如果执行环境不支持，会采用 `setTimeout(fn, 0)` 代替

## 为什么 nextTick 要尽可能地 microTask 优先

详情看连链接
[Vue异步更新 && nextTick源码解析](https://juejin.cn/post/6844903918472790023)

## nextTick 使用场景

- **如果要在 created()钩子函数中进行的 DOM 操作，由于 created()钩子函数中还未对 DOM 进行任何渲染，所以无法直接操作，需要通过**`$nextTick()` **来完成。在 created()钩子函数中进行的 DOM 操作，不使用**`$nextTick()` **会报错**
- **更新数据后，想要使用 js 对新的视图进行操作时**
- **在使用某些第三方插件时 ，这些插件需要 dom 动态变化后重新应用该插件，这时候就需要使用$nextTick()来重新应用插件的方法**
