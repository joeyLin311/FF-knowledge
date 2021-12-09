---
date created: 2021-12-09 22:53
---

#Vue

## vue2 响应式实现原理

- Vue 的响应式从 Vue 的实例 `init()` 方法中开始, 在 `init()` 方法中调用 `initData()`, 把 data 属性注入到 Vue 实例上, 并调用 `observe(data)` 将 data 对象转换成响应式对象
- `observe(value, asRootData)` 是响应式的入口, 其中
  - 先判断 value 是否是对象, 如果不是直接 return
  - 再判断 value 是否有 **ob**(observer 对象) 属性, 有则该对象已经做了响应式处理, 直接返回
  - 如果没有则创建一个 `Observer` 对象
- 创建 `Observer` 对象, 将实例挂载到 `value` 的 `__ob__` 属性, 设置为不可枚举, 接着进行对象或数组的响应式处理
  - 数组的响应式处理: 即设置操作数组的几个特殊方法, `push`, `pop`, `sort` 等, 这些方法会改变原数组, 所以调用的时候需要发送数据更改通知
    - 找到数组对象中的 `__ob__` 中的 `dep` 属性, 调用 `dep` 的 `notice()` 方法
    - 接着遍历数组中的每一个成员, 对每个成员调用 `observe()`, 如果该成员是对象的话, 也会转换成响应式对象
  - 对象的响应式处理: 调用 `walk()` 方法, 遍历对象中的每一个属性，调用 `defineReactive()` 方法, 设置 setter/getter
- `defineReactive(obj, key, val, customSetter, shallow)` 会为每一个对象创建对应的 `dep` 对象, `dep` 收集依赖, 如果当前值是对象调用 `observe`. `defineReactive()` 的核心方法是 `getter` 和 `setter`
  - `getter` 的作用是为每一个属性收集依赖, 如果属性值是对象, 那也为属性中的子对象收集, 最后返回属性的值.
  - `setter` 会先保存新值, 如果值是对象, 也调用 `observe` 将新设置的对象转换成响应式对象, 然后调用 `dep.notify()` 发送通知
- 收集依赖阶段
  - 在 `watcher` 对象的 `get` 方法中调用 `pushTarget` , 记录 `Dep.target` 属性, 如果 `target` 存在, 把 `dep` 对象添加到 `watcher` 的依赖中
  - 访问 `data` 中的属性的时候收集依赖, `defineReactive` 的 `getter` 中收集依赖
  - `dep.depend()` 内部调用 `Dep.target.addDep(this)`, 也就是 `watcher` 的 `addDep()` 方法，它内部最调用 `dep.addSub(this)`, 把 `watcher` 对象, 添加到 `dep.subs.push(watcher)` 中, 也就是把订阅者添加到 `dep` 的 `subs` 数组中, 当数据变化的时候调用 `watcher` 对象的 `update()` 方法
  - `get()` 方法内部调用 `pushTarget(this)`, 把当前 `Dep.target = watcher`, 同时把当前 `watcher` 入栈, 有父子组件嵌套的时候先把父组件对应的 `watcher` 入栈, 再去处理子组件的 `watcher`, 子组件的处理完毕后，再把父组件对应的 `watcher` 出栈，继续操作
  - 调用每个订阅者的 `update` 方法实现更新
- 数据发生变化时
  - 调用 `dep.notice()` 发送通知, `dep.notify()` 调用上述中的 `watcher` 对象中的 `update` 方法
  - `update()` 中调用 `queueWatcher()`, 判断 `watcher` 是否已经处理, 如果没有被处理则添加到 `queue` 队列中, 并调用 `flushScheduleQueue()`
    - 在 `flushScheduleQueue()` 触发 `beforeUpdate` 钩子函数
    - 调用 `watcher.run()`, 顺序: `run()` --> `get()` --> `getter()` --> `updateComponent()`
    - 清空之前的依赖
    - 触发 `actived` 钩子函数
    - 触发 `updated` 钩子函数

## Observer, Dep, Watcher

先总结, Vue 的响应式原理核心就是 `Observer`, `Dep`, `Watcher` 三者
`Observer` 中进行响应式的绑定，在数据被读的时候，触发 `get` 方法，执行 `Dep` 来收集依赖，也就是收集 `Watcher`。

在数据被改的时候，触发 `set` 方法，通过对应的所有依赖(`Watcher`)，去执行更新。比如 `watch` 和 `computed` 就执行开发者自定义的回调方法。

## 关于监听数组的变化

[为什么 Vue 不能检测数组变化](https://juejin.cn/post/6844903917898186766)
Vue 也是不能检测到以下变动的数组：
1.当你利用索引直接设置一个项时，例如：`vm.items[indexOfItem] = newValue`
2.当你修改数组的长度时，例如：`vm.items.length = newLength`
对于第一种情况，可以使用：`Vue.set(example1.items, indexOfItem, newValue)`；而对于第二种情况，可以使用 `vm.items.splice(newLength)`。

我们刚才也分析到，对于 `Vue.set` 的实现，当 `target` 是数组的时候，也是通过 `target.splice(key, 1, val)` 来添加的，那么这里的 `splice` 到底有什么黑魔法，能让添加的对象变成响应式的呢。

其实之前我们也分析过，在通过 `observe` 方法去观察对象的时候会实例化 `Observer`，在它的构造函数中是专门对数组做了处理，它的定义在 `src/core/observer/index.js` 中。

只需要关注 `value` 是 Array 的情况，首先获取 `augment`，这里的 `hasProto` 实际上就是判断对象中是否存在 `__proto__`，如果存在则 `augment` 指向 `protoAugment`， 否则指向 `copyAugment`，来看一下这两个函数的定义

`protoAugment` 方法是直接把 `target.__proto__` 原型直接修改为 `src`，而 `copyAugment` 方法是遍历 keys，通过 `def`，也就是 `Object.defineProperty` 去定义它自身的属性值。对于大部分现代浏览器都会走到 `protoAugment`，那么它实际上就把 `value` 的原型指向了 `arrayMethods`，`arrayMethods` 的定义在 `src/core/observer/array.js` 中：

`arrayMethods` 首先继承了 `Array`，然后对数组中所有能改变数组自身的方法，如 `push、pop` 等这些方法进行重写。重写后的方法会先执行它们本身原有的逻辑，并对能增加数组长度的 3 个方法 `push、unshift、splice` 方法做了判断，获取到插入的值，然后把新添加的值变成一个响应式对象，并且再调用 `ob.dep.notify()` 手动触发依赖通知，这就很好地解释了之前的示例中调用 `vm.items.splice(newLength)` 方法可以检测到变化。

## 原理图

![[vue 响应式原理.png]]

## 原理图 2

![[vue 响应式原理 2.png]]

## vue 双向数据绑定原理

**Vue 的双向数据绑定是通过数据劫持结合发布者订阅者模式来实现的**
要实现这种双向数据绑定，必要的条件有：

1. 实现一个数据监听器 Observer，能够对数据对象的所有属性进行监听，如有变动可拿到最新值并通知订阅者
2. 实现一个指令解析器 Compile，对每个元素节点的指令进行扫描和解析，根据指令模板替换数据，以及绑定相应的更新函数
3. 实现一个 Watcher，作为连接 Observer 和 Compile 的桥梁，能够订阅并收到每个属性变动的通知，执行指令绑定的相应回调函数，从而更新视图
4. MVVM 入口函数，整合以上三者

[[Proxy, defineProperty]]
**个人理解：在 new Vue 的时候，在 Observer 中通过 Object.defineProperty()达到数据劫持，代理所有数据的 getter 和 setter 属性，在每次触发 setter 的时候，都会通过 Dep 来通知 Watcher，Watcher 作为 Observer 数据监听器与 Compile 模板解析器之间的桥梁，当 Observer 监听到数据发生改变的时候，通过 Updater 来通知 Compile 更新视图**
**而 Compile 通过 Watcher 订阅对应数据，绑定更新函数，通过 Dep 来添加订阅者，达到双向绑定**

### 实现数据绑定的做法有大致有几种：

#### 发布者-订阅者模式

一般通过 sub,pub 的方式实现数据和视图的绑定监听，
更新数据方式通常做法是 `vm.set('property', value)`。
这种方式现在毕竟太 low 了，我们更希望通过 vm.property = value 这种方式更新数据，同时自动更新视图，于是有了下面两种方式。

#### 脏值检查

angular.js 是通过脏值检测的方式比对数据是否有变更，来决定是否更新视图，
最简单的方式就是通过 setInterval() 定时轮询检测数据变动，
angular 只有在指定的事件触发时进入脏值检测，大致如下：

> 1、DOM 事件，譬如用户输入文本，点击按钮等。( `ng-click` )

2、XHR 响应事件 ( `$http` )
3、浏览器 Location 变更事件 ( `$location` )
4、Timer 事件( `$timeout` , `$interval` )
5、执行 `$digest()` 或 `$apply()`

##### 数据劫持

vue.js 则是采用数据劫持结合发布者-订阅者模式的方式，
通过 `Object.defineProperty()` 来劫持各个属性的 `setter，getter`，
在数据变动时发布消息给订阅者，触发相应的监听回调。
