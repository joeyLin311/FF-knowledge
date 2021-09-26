#Vue
## event
[event](https://ustbhuangyi.github.io/vue-analysis/v2/extend/event.html#%E7%BC%96%E8%AF%91)
在日常开发中, 实现组件间的通信, 原生交互都离不开事件. 对于一个组件元素, 不仅可以绑定原生的DOM事件, 还可以绑定自定义事件, 非常灵活.
### 编译
在编译阶段, `parse` 阶段会执行 `processAttrs` 方法, 定义在 `src/compiler/parser/index.js` 中.
1. 在标签属性编译过程中, 如果遇上指令, 先通过 `parseModifiers` 解析出修饰符, 然后执行 `addHandler` 函数, 该函数做了3件事情: 1. 根据 `modifier` 修饰符对时间名 `name` 做处理. 2. 根据 modifier.native 判断是原生事件还是普通事件, 分别对应 `el.natveEvents` 和 `el.events` 3. 最后按照 name 对事件做归类, 并把回调函数的字符串保留到对应的事件中
2. `genHandlers` 方法遍历事件对象 `events`，对同一个事件名称的事件调用 `genHandler(name, events[name])` 方法，它的内容看起来多，但实际上逻辑很简单，首先先判断如果 `handler` 是一个数组，就遍历它然后递归调用 `genHandler` 方法并拼接结果，然后判断 `hanlder.value` 是一个函数的调用路径还是一个函数表达式， 接着对 `modifiers` 做判断，对于没有 `modifiers` 的情况，就根据 `handler.value` 不同情况处理，要么直接返回，要么返回一个函数包裹的表达式；对于有 `modifiers` 的情况，则对各种不同的 `modifer` 情况做不同处理，添加相应的代码串。
### 自定义事件
1. `$on(event, fn)` : 添加自定义事件, 把回调函数存储在 `vm._events[event].push(fn)`
```javascript
Vue.prototype.$on = function (event: string | Array<string>, fn: Function): Component {
  const vm: Component = this
  if (Array.isArray(event)) {
    for (let i = 0, l = event.length; i < l; i++) {
      this.$on(event[i], fn)
    }
  } else {
    (vm._events[event] || (vm._events[event] = [])).push(fn)
    // optimize hook:event cost by using a boolean flag marked at registration
    // instead of a hash lookup
    if (hookRE.test(event)) {
      vm._hasHookEvent = true
    }
  }
  return vm
}
```
2. `$emit(event)` : 执行已有的自定义事件, 根据传入的事件名, 找到所有的回调函数 `vm._events[event]` 然后便利执行所有的回调函数
```javascript
Vue.prototype.$emit = function (event: string): Component {
  const vm: Component = this
  if (process.env.NODE_ENV !== 'production') {
    const lowerCaseEvent = event.toLowerCase()
    if (lowerCaseEvent !== event && vm._events[lowerCaseEvent]) {
      tip(
        `Event "${lowerCaseEvent}" is emitted in component ` +
        `${formatComponentName(vm)} but the handler is registered for "${event}". ` +
        `Note that HTML attributes are case-insensitive and you cannot use ` +
        `v-on to listen to camelCase events when using in-DOM templates. ` +
        `You should probably use "${hyphenate(event)}" instead of "${event}".`
      )
    }
  }
  let cbs = vm._events[event]
  if (cbs) {
    cbs = cbs.length > 1 ? toArray(cbs) : cbs
    const args = toArray(arguments, 1)
    for (let i = 0, l = cbs.length; i < l; i++) {
      try {
        cbs[i].apply(vm, args)
      } catch (e) {
        handleError(e, vm, `event handler for "${event}"`)
      }
    }
  }
  return vm
}

```
3. `$off(event, fn)` : 移除指定的事件名, 和指定的回调函数
```javascript
Vue.prototype.$off = function (event?: string | Array<string>, fn?: Function): Component {
  const vm: Component = this
  // all
  if (!arguments.length) {
    vm._events = Object.create(null)
    return vm
  }
  // array of events
  if (Array.isArray(event)) {
    for (let i = 0, l = event.length; i < l; i++) {
      this.$off(event[i], fn)
    }
    return vm
  }
  // specific event
  const cbs = vm._events[event]
  if (!cbs) {
    return vm
  }
  if (!fn) {
    vm._events[event] = null
    return vm
  }
  if (fn) {
    // specific handler
    let cb
    let i = cbs.length
    while (i--) {
      cb = cbs[i]
      if (cb === fn || cb.fn === fn) {
        cbs.splice(i, 1)
        break
      }
    }
  }
  return vm
}
```
4. `$once(event, fn)` : 内部就是执行 `vm.$on` , 并且当回调函数执行一次后再通过 `vm.$off` 移除事件的回调, 保证回调函数只执行一次
```javascript
Vue.prototype.$once = function (event: string, fn: Function): Component {
  const vm: Component = this
  function on() {
    vm.$off(event, on)
    fn.apply(vm, arguments)
  }
  on.fn = fn
  vm.$on(event, on)
  return vm
}

```
### 小总结
Vue 支持 2 种事件类型，原生 DOM 事件和自定义事件，它们主要的区别在于添加和删除事件的方式不一样，并且自定义事件的派发是往当前实例上派发，但是可以利用在父组件环境定义回调函数来实现父子组件的通讯。另外要注意一点，只有组件节点才可以添加自定义事件，并且添加原生 DOM 事件需要使用 `native` 修饰符；而普通元素使用 `.native` 修饰符是没有作用的，也只能添加原生 DOM 事件。
