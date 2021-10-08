#Vue
## [[options API & composition API]]
## [[vue3生命周期变化]]
## diff算法提升
## typescript 支持
同上 [[options API & composition API]]
## [[Vite]]
## 升级概览
[升级概览](https://v3.cn.vuejs.org/guide/migration/introduction.html#%E6%A6%82%E8%A7%88)

[Vue3知识点]([https://juejin.cn/post/6977004323742220319#heading-10](https://juejin.cn/post/6977004323742220319#heading-10))

### Vue 3.0 性能提升主要是通过哪几方面体现的？
总结以下几点:
- 响应式系统升级
- 编译升级优化
- 优化源代码打包体积
在性能方面 Vue.js 3.x 又大幅度提升，使用代理对象 Proxy 重写了响应式的代码并且对编译器做了优化，重写了虚拟 DOM，从而让渲染和 Update 的性能都有了大幅度的提升。
另外，官方介绍服务端渲染的性能也提升两到三倍。
#### 响应式系统升级
- Vue 2.x 中响应式系统的核心是 `defineProperty`
  - 初始化时遍历 `data` 中的所有成员，通过 `defineProperty` 把对象的属性转换成 `getter` 和 `setter`，如果 `data` 中的属性又是对象的话，需要递归处理每一个子对象的属性。这些都是初始化时进行的，如果你未使用这些属性也会进行响应式的处理
- Vue 3.x 中使用 Proxy 对象重写响应式系统
  - Proxy 的性能本身就比 `defineProperty` 好，且代理对象可以拦截属性的访问、赋值、删除等操作，不需要初始化时遍历所有的属性，如果有多层属性嵌套只有访问某个属性时才会递归处理下一级属性
  - 使用 Proxy 对象默认可以监听动态新增的属性，而 Vue 2.x 想要动态添加响应式属性需要调用 `Vue.set` 方法来处理
  - Vue 2.x 监听不到属性的删除
  - Vue 2.x 无法对数组的索引和 `length` 属性进行监听
除了响应式系统的升级，Vue.js 3.x 通过优化编译的过程和重写虚拟 DOM 让首次渲染和更新的性能有了大幅度提升
#### 编译升级优化
Vue 2.x 模板首先需要编译成 `render` 函数，这个过程一般在构建时完成的，在编译时会编译静态根节点和静态节点，静态根节点要求节点中必须有一个静态子节点
当组件的状态发生变化后会通知 `watcher` 触发 `update` 去执行 **虚拟 DOM** 的 `patch` 操作，遍历所有的虚拟节点找到差异更新到 **真实 DOM** 上，diff 的过程中会去比较整个 **虚拟 DOM**，先对比新旧 div 以及它的属性再对比内部子节点
Vue 2.x 中渲染最小的单位是组件，diff 的过程会跳过静态根节点，因为静态根节点的内容不会发生变化，即:
- **Vue 2.x 中通过标记静态根节点，优化 diff 的过程**，但是静态节点还需要进行 diff，没有被优化
- **Vue 3.x 中标记和提升所有静态根节点，diff 的时候只需要对比动态节点内容**
  - Fragments（VS Code 需要升级 vetur 插件）：模板中不需要再创建唯一的根节点，可以直接放文本内容或者多个同级标签
  - 静态提升
  - Patch flag
  - 缓存事件处理函数

#### 优化源代码打包体积

- Vue 3.x 中移除了一些不常用的 API 例如：`inline-template`、`filter` 等
- Tree-shaking

### Vue 3.0 所采用的 Composition Api 与 Vue 2.x 使用的 Options Api 有什么区别？

- Options API
  - 包含一个描述组件选项（data、methods、props 等）的对象
  - Options API 开发复杂组件，同一个功能逻辑的代码被拆分到不同选项
- Composition API
  - Vue.js 3.x 新增的一组 API
  - 一组基于函数的 API
  - 可以更灵活的组织组件的逻辑

相对于 Options API 这样做的好处：查看某个逻辑时只需关注具体的函数即可，当前的逻辑代码都封装在函数内部，不像 Options API 时获取鼠标位置的逻辑代码分散在不同的位置，开发时需要经常跳转至文件不同位置

### Proxy 相对于 Object.defineProperty 有哪些优点？
[[Proxy, defineProperty]]
1. Proxy 更为强大
   Object.defineProperty()只能监视属性的读写，Proxy 能监视到更多对象操作。例如 delate 操作、对对象方法的调用。

```javascript
const target = {
 foo: 'aaa',
 bar: 'bbb'
}
const proxy = new Proxy(target, {
 get (target, key, receiver) {
   return Reflect.get(target, key, receiver)
 },
 set (target, key, value, receiver) {
   return Reflect.set(target, key, value, receiver)
 },
 deleteProperty (target, key) {
   return Reflect.deleteProperty(target, key)
 }
})

proxy.foo = 'ccc'
delete proxy.foo
```

2. Proxy 更好的支持数组对象的监视

   以往想通过`Object.defineProperty()`去监视数组的操作, 最常见的一种方式就是通过重写数组的操作方法(Vue使用的方式), 大体的思路就是通过自定义的方法去覆盖掉数组原先对象的push、shift等方法以此去劫持对应这个方法调用的过程. Proxy 可以如下所示:

```javascript
const list = []

const listProxy = new Proxy(list, {
 set(target, property, value) {
   console.log('set', property, value)
   target[property] = value
   return true // 表示设置成功
 }
})

listProxy.push(100)
// set 0 100
// set length 1

listProxy.push(200)
// set 1 200
// set length 2
```

3. Proxy 是以非侵入的方式监管了对象的读写

   也就是说一个已经定义好的对象，不需要对对象本身做任何操作就可以监视到内部成员的读写。而`Object.defineProperty()`就要求我们必须通过特定的方式单独定义对象中需要被监视的属性。

### Vue 3.0 在编译方面有哪些优化？

Vue 2.x 模板首先需要编译成 `render` 函数，这个过程一般在构建时完成的，在编译时会编译静态根节点和静态节点，静态根节点要求节点中必须有一个静态子节点

当组件的状态发生变化后会通知 `watcher` 触发 `update` 去执行 **虚拟 DOM** 的 `patch` 操作，遍历所有的虚拟节点找到差异更新到 **真实 DOM** 上，diff 的过程中会去比较整个 **虚拟 DOM**，先对比新旧 div 以及它的属性再对比内部子节点

Vue 2.x 中渲染最小的单位是组件，diff 的过程会跳过静态根节点，因为静态根节点的内容不会发生变化，即:

- **Vue 2.x 中通过标记静态根节点，优化 diff 的过程**，但是静态节点还需要进行 diff，没有被优化
- **Vue 3.x 中标记和提升所有静态根节点，diff 的时候只需要对比动态节点内容**
  - Fragments（VS Code 需要升级 vetur 插件）：模板中不需要再创建唯一的根节点，可以直接放文本内容或者多个同级标签
  - 静态提升
  - Patch flag
  - 缓存事件处理函数

### Vue.js 3.0 响应式系统的实现原理？

- Proxy 对象实现属性监听
  - Vue3 重写了响应式系统，和 Vue2 相比 Vue3 的响应式系统底层采用 Proxy 对象实现。在初始化的时候不需要遍历所有的属性，再把属性通过 defineProperty 转换成 getter 和 setter
- 多层嵌套，在访问属性过程中处理下一级属性
  - 如果有多层属性嵌套的话，只有访问某个属性的时候才会递归处理下一级属性，所以 Vue3 中响应式系统的性能要比 Vue2 好
- 默认监听动态添加的属性
- 默认监听属性的删除操作
- 默认监听数组索引和 length 属性
- 可以作为单独的模块使用

**核心方法：**

- reactive/ref/toRefs/computed
- effect
- track
- trigger

> watch/watchEffect 是 Vue3 的 runtime.core 中实现的，watch 函数的内部其实实现了一个底层函数 effect
> 我们会模拟实现 effect 函数以及 Vue3 中收集依赖和触发更新的函数 track 和 trigger
