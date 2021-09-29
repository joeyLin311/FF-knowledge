#Vue #JavaScript 
## Object.defineProperty
MDN定义: `Object.defineProperty()` 方法会直接在一个对象上定义一个新属性, 或者修改一个对象的现有属性, 并返回此对象.
方法传入三个参数:
> obj: 要定义属性的对象, 就是目标对象
> prop: 要定义或修改的属性名称或 Symbol
> descriptor: 要定义或修改的属性描述符

```javascript
var obj = {}
Object.defineProperty(obj, "key", {
  configurable: true,
  enumerable: false,
  writable: true,
  value: 'value',
  get(){ return obj.key},
  set(val){ return obj.key = val }
})
```
| 属性名 | 作用 | 默认值 |
| --- | --- | --- |
| configurable | 只有该属性的 `configurable`为true, 该属性的描述符才能够被改变, 同时该属性也能从对应的对象上被删除 | false |
| enumerable | 只有该属性的 `enumerable`为true, 该属性才会出现在对象的可枚举属性中, 就是能不能 `for...in` 和 `Object.keys()` | false |
| writable | 只有该属性的 `enumerable`为 true, 才能被赋值运算符改变 | false |
| value | 该属性对应的值 | undefined |
| get | 属性的 getter 函数, 当访问该属性时会调用此函数 | undefined |
| set | 当属性值被修改时, 会调用此函数.该方法接收一个参数, 会传入赋值时的this对象 | undefined |

**数据描述符和存取描述符不能混用, 但是均可以和 comfigurable enumerable 搭配使用,如下表格**

|  | configurable | enumerable | value | writable | get | set |
| --- | --- | --- | --- | --- | --- | --- |
| 数据描述符 | YES | YES | YES | YES | NO | NO |
| 存取描述符 | YES | YES | NO | NO | YES | YES |

### Object.defineProperty() 的缺陷
虽然 `Object.defineProperty()` 能够劫持对象的属性进行改写操作, 但是需要对对象上的每一个属性进行遍历劫持; 如果对象上有新增的属性, 则需要对新增的属性再次进行劫持; 如果对象的属性是对象, 还需要进行深度遍历. **这就是为什么Vue给对象新增.属性需要通过 **`**$set**` 操作的原因, 其原理也是通过 `Object.defineProperty` 对新增的属性再次进行劫持
`Object.defineProperty()` 除了可以劫持对象的属性, 还可以劫持数组; 虽然数组没有属性, 但是我们可以把数组的索引看成是属性. 虽然通过这样我们监听到了数组的变化, 但是监听对象属性面临着同样的问题: 新增的元素并不会触发监听事件.
**为此, Vue 的解决方案是劫持 `Array.prototype` 原型链上的7个方法, 重新绑定到实例对象的  `__proto__`, 以实现劫持**
总结就是: 
1. defineProperty 无法检测到对象属性的添加或删除
1. defineProperty 无法检测数组元素的变化, 需要进行数组方法的重写
1. defineProperty 无法检测数组的长度修改

## Proxy
相较于 `Object.defineProperty` 去劫持某个属性, Proxy更加彻底, 是直接对整个对象进行代理. MDN对Proxy的描述是: 
> `Proxy`可以理解成, 在目标对象之前架设一层"拦截" , 外界对该对象的访问, 都必须先通过这层拦截, 因此提供了一种机制, 可以对外界的访问进行过滤和改写

### 语法: `var obj = new Proxy(target, handler)`
Proxy 本身是一个构造函数, 通过 `new Proxy` 生成拦截的实例对象, 让外界进行访问; 构造函数中的 `target`就是我们需要代理的目标对象, 可以是对象或者是数组; `handler` 参数和 defineProperty 中的 description 描述符有些类似, 也是一个对象, 用来定制代理规则
```javascript
var target = {}
var proxyObj = new Proxy(
  target,
  {
    get: function(target, propKey, receiver) {
      console.log(`getting ${propKey}!`)
      return Reflect.get(target, propKey, receiver)
    },
    set: function(target, propKey, value, receiver) {
      console.log(`setting ${propKey}!`)
      return Reflect.set(target, propKey, value, receiver)
    },
    deleteProperty: function(target, propKey) {
      console.log(`delete ${propKey}!`)
      delete target[propKey]
      return true
    }
  }
)
//setting count!
proxyObj.count = 1;
//getting count!
//1
console.log(proxyObj.count)
//delete count!
delete proxyObj.count
```
上述代码可见: Proxy 直接代理了 `target`整个对象, 并且返回了一个新的对象, 通过监听代理对象上属性的变化来获取目标对象属性的变化; 而且Proxy 不仅能够监听到对象属性的增加, 还能监听属性的删除, 比 defineProperty 功能更加强大.
proxy 代理数组的例子: 
```javascript
var list = [1,2]
var proxyObj = new Proxy(list, {
  get: function (target, propKey, receiver) {
    console.log(`getting ${propKey}!`);
    return Reflect.get(target, propKey, receiver);
  },
  set: function (target, propKey, value, receiver) {
    console.log(`setting ${propKey}:${value}!`);
    return Reflect.set(target, propKey, value, receiver);
  },
})

//setting 1:3!
proxyObj[1] = 3
//getting push!
//getting length!
//setting 2:4!
//setting length:3!
proxyObj.push(4)
//setting length:5!
proxyObj.length = 5
```
## Vue2 的响应式实现原理
Vue2 在初始化的时候, 会递归调用Object.defineProperty. 当第一层对象属性定义后, 会继续递归调用下一层属性的 Obejct.defineProperty, 目的是为了依赖收集, 所以Vue2在初始化上花费了较多时间去同步递归定义 Object.defineProperty操作. 
另外从 Obejct.defineProperty 的缺陷可以看出, 初始化时吧 data 的属性递归遍历收集了, 当data在代码运行过程中如果有动态增加属性的话, 需要用户主动告知Vue那些属性需要进行依赖收集变成响应式数据, 所以才有了 `Vue.$set` 和` Vue.delete`
以下是精简源码, Vue2 中定义响应式对象的API是 `Vue.observer(data)`
```javascript
// https://github.com/vuejs/vue/blob/dev/src/core/observer/index.js
export function observe (value: any, asRootData: ?boolean): Observer | void {
  let ob = new Observer(value)
  return ob
}

export class Observer {
  constructor (value: any) {
      this.walk(value)
  }

  walk (obj: Object) {
    const keys = Object.keys(obj)
    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i])
    }
  }
}

export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: ?Function,
  shallow?: boolean // default: false
) {
  const dep = new Dep()
  const getter = property && property.get
  const setter = property && property.setter

  // 同步实时：递归子层级
  let childOb = !shallow && observe(val)
  
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      const value = getter ? getter.call(obj) : val
      // ... dep依赖收集
      return value
    },
    set: function reactiveSetter (newVal) {
      setter.call(obj, newVal)
      // ... dep触发更新
    }
  })
}
```
## Vue3 中响应事件处理
Vue3 中定义响应式对象的API是: `reactive(data)`
Proxy 提供的对JS的元数据编程, 可以直接创建新的JS语法, Vue3 中定义响应式相较为简单, 就是原生的 `new Proxy(target, handler)`. 此时这里没有递归调用初始化, 即可看成是懒加载去依赖收集, 也就是用到的时候才去依赖收集.
源码中代理对象 `baseHandlers` 中 `get` 的操作定义
```javascript
// https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/reactive.ts
export function reactive(target: object) {
  return createReactiveObject(target)
}

function createReactiveObject(target: Target) {
  const observed = new Proxy(
    target,
    baseHandlers // {get, set, deleteProperty} // 其中的get
  )
  return observed
}
```
get 定义, 主要用来依赖收集吗同时如果属性的value 为Object 对象时, 则自动进行Proxy代理
```javascript
// https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/baseHandlers.ts
const get = createGetter()

function createGetter(isReadonly = false, shallow = false) {
  return function get(target: Target, key: string | symbol, receiver: object) {
    const res = Reflect.get(target, key, receiver)

    // ...依赖收集

    // 懒加载，当访问响应式对象时，再去构造下一个Proxy(res, { getter })
    if (isObject(res)) {
      return reactive(res)
    }

    return res
  }
}
```
​

参考资料:
[Vue3js.cn](https://vue3js.cn/es6/)
[springleo's Blog](https://lq782655835.github.io/blogs/vue/vue3-code-4.why-proxy-faster.html)