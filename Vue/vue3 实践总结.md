---
date created: 2022-07-13 12:47
---
#Vue
## 一、Vue3

**Vue3 中文文档** [1]

### Vue3 是什么，与 Vue2 区别（What）

1.  `Performance`：性能更强。
2.  `Tree shaking support`：可以将无用模块 “剪辑”，仅打包需要的。
3.  `Composition API`：组合式 `API`
4.  `Fragment, Teleport, Suspense`：“碎片”，`Teleport` 即 `Protal 传送门 `，“悬念”
5.  `Better TypeScript support​`​：更优秀的 Ts 支持
6.  `Custom Renderer API`：暴露了自定义渲染 `API`
### 为什么要大版本迭代 （Why）

1.  主流浏览器对新的 JavaScript 语言特性的普遍支持。
2.  当前 Vue 代码库随着时间的推移而暴露出来的设计和体系架构问题。

### 他是如何提升的(How)

1.  **响应式系统提升：** 使用 Proxy 提升了响应式的性能和功能
2.  **编译优化：** 标记和提升所有的静态节点，diff 时只需要对比动态节点内容
3.  **事件缓存：** 提供了事件缓存对象 cacheHandlers，无需重新创建函数直接调用缓存的事件回调
4.  **打包和体积优化：** 按需引入，Tree shaking 支持（ES Module）

## 二、编码

### 全局 API

1.  【**新增】createApp：**入口文件（main.ts）挂载方式

```js
import { createApp } from "vue";

import App from "./App.vue";

// Vue2

new Vue({

   render: (h) => h(App)

}).$mount("#app");

// Vue3

createApp(App)

  .use(**)

  .mount("#app");

```

2.  **【修改】**

<table data-id="t7216feb-KVa9eDU0" data-transient-attributes="class" data-width="872px"><colgroup data-id="c82d01ad-cEAZKfF2"><col data-id="c416895f-ATLUM2N8" span="1" width="436"><col data-id="c416895f-NXmB89h5" span="1" width="436"></colgroup><tbody data-id="tc1e2dd5-4g7eTAYj"><tr data-id="tb3b27aa-146YLIF8"><td data-id="t0b07364-9k7B7SHh" data-transient-attributes="table-cell-selection"><p data-id="pd157317-eDc2RKdh">2.x 全局 API</p></td><td data-id="tf57bfb8-bKQ0ICda" data-transient-attributes="table-cell-selection"><p data-id="pd157317-EiNXQCTX">3.x 实例 API (app)</p></td></tr><tr data-id="tb3b27aa-0VECgYl7"><td data-id="t0b07364-eGIm2nA0" data-transient-attributes="table-cell-selection"><p data-id="pd157317-Bi00jWQD">Vue.config</p></td><td data-id="tf57bfb8-dUNhZZJL" data-transient-attributes="table-cell-selection"><p data-id="pd157317-oj1Kb4SX">app.config</p></td></tr><tr data-id="t0992843-e5F53KGf"><td data-id="t6039d04-4fdhk6mL" data-transient-attributes="table-cell-selection"><p data-id="pd157317-Vauu70uu">Vue.config.productionTip</p></td><td data-id="tb48c6d2-WGAP1R2X" data-transient-attributes="table-cell-selection"><p data-id="pd157317-Kx1EbKq3">无</p></td></tr><tr data-id="tb3b27aa-lDZSO3BX"><td data-id="t0b07364-8LjTJAGe" data-transient-attributes="table-cell-selection"><p data-id="pd157317-bCr71KZ4">Vue.config.ignoredElements</p></td><td data-id="tf57bfb8-nMM97W15" data-transient-attributes="table-cell-selection"><p data-id="pd157317-0sU2Swiu">app.config.isCustomElement</p></td></tr><tr data-id="t0992843-HfWfkXbc"><td data-id="t6039d04-mZh9RO88" data-transient-attributes="table-cell-selection"><p data-id="pd157317-93fZSYfi">Vue.component</p></td><td data-id="tb48c6d2-UkmaiWZ7" data-transient-attributes="table-cell-selection"><p data-id="pd157317-JDVADjrz">app.component</p></td></tr><tr data-id="tb3b27aa-De6Shd7d"><td data-id="t0b07364-cfnU1QbT" data-transient-attributes="table-cell-selection"><p data-id="pd157317-imB6EO0H">Vue.directive</p></td><td data-id="tf57bfb8-01PWW2ZH" data-transient-attributes="table-cell-selection"><p data-id="pd157317-Wlk2pz11">app.directive</p></td></tr><tr data-id="t0992843-e09TKImU"><td data-id="t6039d04-EaQ3i3Un" data-transient-attributes="table-cell-selection"><p data-id="pd157317-ehzT6kAs">Vue.mixin</p></td><td data-id="tb48c6d2-YEcjSJAL" data-transient-attributes="table-cell-selection"><p data-id="pd157317-UiIG6X5M">app.mixin</p></td></tr><tr data-id="tb3b27aa-H549KLiR"><td data-id="t0b07364-G5iR011k" data-transient-attributes="table-cell-selection"><p data-id="pd157317-8Ndd4HMm">Vue.use</p></td><td data-id="tf57bfb8-hqFVgW5H" data-transient-attributes="table-cell-selection"><p data-id="pd157317-iPZTHmgm">app.use</p></td></tr><tr data-id="t0992843-oigH5JVS"><td data-id="t00b07f2-5K2OdOFl" data-transient-attributes="table-cell-selection"><p data-id="pd157317-TFgRmNer">Vue.prototype</p></td><td data-id="ta9e714b-cZB8AEL3" data-transient-attributes="table-cell-selection"><p data-id="pd157317-C2RmBqA6">app.config.globalProperties</p></td></tr></tbody></table>

### config.ignoredElements` 替换为 `config.isCustomElement

引入此配置选项的目的是支持原生自定义元素，因此重命名可以更好地传达它的功能，新选项还需要一个比旧的 string/RegExp 方法提供更多灵活性的函数：

```js
// Vue2

Vue.config.ignoredElements = [

  // 用一个 `RegExp` 忽略所有“ion-”开头的元素

  // 仅在 2.5+ 支持

  /^ion-/

]

// Vue3

const app = Vue.createApp({})

app.config.isCustomElement = tag => tag.startsWith('ion-')

```

### `Vue.prototype` 替换为 `config.globalProperties`

在 Vue 2 中，Vue.prototype 通常用于添加可在所有组件中访问的属性。

Vue 3 中的等效项是 config.globalProperties。在实例化应用程序内的组件时，将复制这些属性

```js
// Vue2

Vue.prototype.$http = () => {}

// Vue3

const app = Vue.createApp({})

app.config.globalProperties.$http = () => {}

```

### 生命周期

<table data-id="t7216feb-CZR62NJ5" data-transient-attributes="class" data-width="872px"><colgroup data-id="c82d01ad-CaBmBKM1"><col data-id="c416895f-2k1gaZ2D" span="1" width="436"><col data-id="c416895f-apK0BjcG" span="1" width="436"></colgroup><tbody data-id="tc1e2dd5-Q5OkMkUK"><tr data-id="tb3b27aa-rLbLZIEa"><td data-id="t0b07364-Ph6Y4FOe" data-transient-attributes="table-cell-selection"><p data-id="pd157317-JsAQynSv">2.0 生命周期</p></td><td data-id="tf57bfb8-3V3OnIkc" data-transient-attributes="table-cell-selection"><p data-id="pd157317-ElNHycY8">3.0 生命周期</p></td></tr><tr data-id="tb3b27aa-eKcdamQo"><td data-id="t0b07364-Li8pRK44" data-transient-attributes="table-cell-selection"><p data-id="pd157317-8d0Crdni">beforeCreate(组件创建之前)</p></td><td data-id="tf57bfb8-UMVgCsEM" data-transient-attributes="table-cell-selection"><p data-id="pd157317-Inatvq8i">setup()</p></td></tr><tr data-id="t0992843-i1HSGE66"><td data-id="t6039d04-9KHZKUjp" data-transient-attributes="table-cell-selection"><p data-id="pd157317-1f32B2PU">created(组件创建完成)</p></td><td data-id="tb48c6d2-8GUpVp6f" data-transient-attributes="table-cell-selection"><p data-id="pd157317-76CHnx6z">setup()</p></td></tr><tr data-id="tb3b27aa-akqVeULd"><td data-id="t0b07364-6BAsj6VH" data-transient-attributes="table-cell-selection"><p data-id="pd157317-8W6hOs5b">beforeMount(组件挂载之前)</p></td><td data-id="tf57bfb8-WGB6Ze0q" data-transient-attributes="table-cell-selection"><p data-id="pd157317-gERihshg">onBeforeMount(组件挂载之前)</p></td></tr><tr data-id="t0992843-pe9MQMLc"><td data-id="t6039d04-N5D9E0F8" data-transient-attributes="table-cell-selection"><p data-id="pd157317-anB5TU2q">mounted(组件挂载完成)</p></td><td data-id="tb48c6d2-LgheR3JD" data-transient-attributes="table-cell-selection"><p data-id="pd157317-6A4GA72a">onMounted(组件挂载完成)</p></td></tr><tr data-id="tb3b27aa-OlpmqoZK"><td data-id="t0b07364-u4t5RreS" data-transient-attributes="table-cell-selection"><p data-id="pd157317-buUYRQdR">beforeUpdate(数据更新，虚拟 DOM 打补丁之前)</p></td><td data-id="tf57bfb8-eOXKV189" data-transient-attributes="table-cell-selection"><p data-id="pd157317-vijZ75N9">onBeforeUpdate(数据更新，虚拟 DOM 打补丁之前)</p></td></tr><tr data-id="t0992843-4iVisRMF"><td data-id="t6039d04-8TgPZr0F" data-transient-attributes="table-cell-selection"><p data-id="pd157317-CHe6iKQi">updated(数据更新，虚拟 DOM 渲染完成)</p></td><td data-id="tb48c6d2-GRjURhhp" data-transient-attributes="table-cell-selection"><p data-id="pd157317-I6U02VJV">onUpdated(数据更新，虚拟 DOM 渲染完成)</p></td></tr><tr data-id="tb3b27aa-0ch2bZMO"><td data-id="t0b07364-NIgt34CH" data-transient-attributes="table-cell-selection"><p data-id="pd157317-65f2C7yQ">beforeDestroy(组件销毁之前)</p></td><td data-id="tf57bfb8-8ERQQW7k" data-transient-attributes="table-cell-selection"><p data-id="pd157317-uEvJyYW6">onBeforeUnmount(组件销毁之前)</p></td></tr><tr data-id="t0992843-lOAcWkjg"><td data-id="t6039d04-QZ7CFFRF" data-transient-attributes="table-cell-selection"><p data-id="pd157317-mInz1d8I">destroyed(组件销毁之后)</p></td><td data-id="tb48c6d2-bfaWEnhP" data-transient-attributes="table-cell-selection"><p data-id="pd157317-ERzbr2Xb">onUnmounted(组件销毁之后)</p></td></tr><tr data-id="tb3b27aa-jRk2N7bK"><td data-id="t0b07364-g9Yh0BvK" data-transient-attributes="table-cell-selection"><p data-id="pd157317-ZEowxVzs">activated(被 keep-alive 缓存的组件激活时调用)</p></td><td data-id="tf57bfb8-fDldhYZX" data-transient-attributes="table-cell-selection"><p data-id="pd157317-54d03Fug">onActivated(被激活时执行)</p></td></tr><tr data-id="t0992843-QWAZ9SDr"><td data-id="t6039d04-QTGH20bR" data-transient-attributes="table-cell-selection"><p data-id="pd157317-TmW2eit8">deactivated(被 keep-alive 缓存的组件停用时调用)</p></td><td data-id="tb48c6d2-8gTOr010" data-transient-attributes="table-cell-selection"><p data-id="pd157317-hyQdMcqb">onDeactivated(比如从 A 组件，切换到 B 组件，A 组件消失时执行)</p></td></tr><tr data-id="tb3b27aa-XBlOZGMs"><td data-id="t4f6cead-V3BFB6SW" data-transient-attributes="table-cell-selection"><p data-id="pd157317-Kjx4LE3D">errorCaptured(当捕获一个来自子孙组件的错误时被调用)</p></td><td data-id="t8fcdd92-MO0OO3QB" data-transient-attributes="table-cell-selection"><p data-id="pd157317-5z16i1V7">onErrorCaptured(当捕获一个来自子孙组件的异常时激活钩子函数)</p></td></tr></tbody></table>

### 新特性

#### Options API => Composition API

![](https://s2.51cto.com/images/blog/202202/16150726_620ca2ae1d16868957.gif?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)![](https://s2.51cto.com/images/blog/202202/16150726_620ca2ae384f729831.gif?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### setup()

```js
import {toRefs} from 'vue'

export default {

    name: 'demo',

    props:{

      name: String,

    },

    // setup()作为在组件内使用Composition API的入口点。

    // 执行时机是在beforeCreate和created之间,不能使用this获取组件的其他变量，

    // 而且不能是异步。setup返回的对象和方法，都可以在模版中使用。

    setup(props, context){

      // 这里需要使用toRefs来进行解构

      // 这里的props与vue2基本一致,当然这里的name也可以直接在template中使用

      const { name }=toRefs(props);

      console.log(name.value);

            // context是一个上下文对象

      //【从原来 2.x 中 this 选择性地暴露了一些 property（attrs/emit/slots）】

      // 属性，同vue2的 $attrs

      console.log(context.attrs);

      // 插槽

      console.log(context.slots);

      // 事件，同vue2的 $emit

      console.log(context.emit);

      // 生命周期钩子

      onMounted(() => {})

  }

}
```

**注意点：**

- 注意​`​props​`​​ 对象是响应式的，​`​watchEffect​`​​ 或​`​watch​`​​ 会观察和响应​`​props​`​ 的更新，**不要**解构​`​props​`​ 对象，那样会使其失去响应性
- ​`​attrs​`​​ 和​`​slots​`​ 都是内部组件实例上对应项的代理，可以确保在更新后仍然是最新值。所以可以解构，无需担心后面访问到过期的值
- ​`​this​`​**在**​`​setup()​`​**中不可用**。由于​`​setup()​`​​ 在解析 2.x 选项前被调用，​`​setup()​`​​ 中的​`​this​`​​ 将与 2.x 选项中的​`​this​`​​ 完全不同。同时在​`​setup()​`​​ 和 2.x 选项中使用​`​this​`​ 时将造成混乱

#### 响应式 reactive，ref

```js
import { ref, reactive, toRefs } from 'vue';

setup() {

    // ref

    // ref 对我们的值创建了一个响应式引用

    const counter = ref(0); 

        // reactive

    // 接收一个普通对象然后返回该普通对象的响应式代理

    const obj = {a:1};

    const objReactive = reactive(obj);

    const add = () => {

      counter.value++;

      objReactive.a++;

    }

        return {

      counter,  // return返回会自动解套【在模板中不需要.value】

      objReactive,

      add,

    }; // 这里返回的任何内容都可以用于组件的其余部分

}

```

<table data-id="t4a16ab5-Hj4mH0Pj" data-transient-attributes="class" data-width="870px"><colgroup data-id="c82d01ad-TQaN99gb"><col data-id="c5a5f748-X8IG8hSB" span="1" width="290"><col data-id="c5a5f748-ZS9c9GlE" span="1" width="290"><col data-id="c5a5f748-LPcfAchP" span="1" width="290"></colgroup><tbody data-id="tc1e2dd5-ciQZgUdR"><tr data-id="tb3b27aa-MQ1bk9Ki"><td data-id="t0b07364-8ImA1CYE" data-transient-attributes="table-cell-selection"></td><td data-id="t0b07364-9UCSB6uQ" data-transient-attributes="table-cell-selection"><p data-id="pd157317-AXq9cp5G"><strong>ref</strong></p></td><td data-id="tf57bfb8-8Qt2eVUG" data-transient-attributes="table-cell-selection"><p data-id="pd157317-WVfRo973"><strong>reactive</strong></p></td></tr><tr data-id="tb3b27aa-KrGw3jFD"><td data-id="t0b07364-uUAmzhqf" data-transient-attributes="table-cell-selection"><p data-id="pd157317-28g3TDvS">入参</p></td><td data-id="t0b07364-qkbTCKfB" data-transient-attributes="table-cell-selection"><p data-id="pd157317-lbSH7FnB">基本类型</p></td><td data-id="tf57bfb8-vsdQ2GbH" data-transient-attributes="table-cell-selection"><p data-id="pd157317-MQlJiFI6">引用类型</p></td></tr><tr data-id="t0992843-RaSrCqbj"><td data-id="t6039d04-c8FjquZV" data-transient-attributes="table-cell-selection"><p data-id="pd157317-pzvUJsai">返回值</p></td><td data-id="t6039d04-TiSLsRLD" data-transient-attributes="table-cell-selection"><p data-id="pd157317-CgX7idjf">响应式且可变的 ref 对象</p></td><td data-id="tb48c6d2-98b6VWHb" data-transient-attributes="table-cell-selection"><p data-id="pd157317-KvEOxnKw">响应式代理（Proxy）</p></td></tr><tr data-id="tb3b27aa-5jTiQtzm"><td data-id="t4f6cead-SIlN6hlb" data-transient-attributes="table-cell-selection"><p data-id="pd157317-bJ7sbTjy">访问方式</p></td><td data-id="t4f6cead-eLgmDD8V" data-transient-attributes="table-cell-selection"><p data-id="pd157317-nno31hMt">1.ref 对象拥有一个指向内部值的单一属性 ​<code>​.value​</code>​</p><p data-id="pd157317-CiOENBYM">2. 在 dom 和 setup() 的 return 中会自动解套</p><p data-id="pd157317-PXONA5Io">3.ref 作为 reactive 对象的 property 被访问或修改时，也将自动解套</p></td><td data-id="t8fcdd92-lEC2bMc2" data-transient-attributes="table-cell-selection"><p data-id="pd157317-IwGfu1Ia">直接. 访问即可</p></td></tr></tbody></table>

**问题 & 注意点：** 因为 reactive 是组合函数【**对象**】，所以必须始终保持对这个所返回对象的引用以保持响应性，不能解构该对象或者展开

例如：

​`​const { a } = objReactive​`​​或者​`​return { ...objReactive }​`​

**解决方法：**

​`​toRefs​`​ API

用来提供解决此约束的办法——它将响应式对象的每个 property 都转成了相应的 ref【把对象转成了 ref】。

```js
import { reactive, toRefs } from 'vue';

setup() {

    // reactive

    // 接收一个普通对象然后返回该普通对象的响应式代理

    const obj = {a:1};

    const objReactive = reactive(obj);

        // toRefs

    // 将响应式对象转换为普通对象，其中结果对象的每个 property 都是指向原始对象相应 property 的ref

    const objRef = toRefs(objReactive);

        const { a } = objRef;

    const addObj = () => {

      a.value++;

      console.log(a.value, objRef, objReactive, obj);

    }

    return {

      ...objRef,

      addObj

    }; 

}

```

- Hooks 方式

1.  counter.js

```js
import { ref } from 'vue';

export function useCounter() {

    const count = ref(0);

    const decrement = () => {

        count.value--;

    }

    const increment = () => {

        count.value++;

    }

    return {

        count,

        decrement,

        increment

    }

}
```

2.  父组件

```vue
<template>

    <h2>{{ count }}</h2>

    <button @click="increment">increment</button>

    <button @click="decrement">decrement</button>

</template>

<script>

import { useCounter } from './counter.js';

export default {

    setup() {

        return {

          ...useCounter(),

        };

    }

}

</script>
```

### 响应式计算和侦听

1.  computed

```js
const count = ref(1)

/*不支持修改【只读的】 */

const plusOne = computed(() => count.value + 1)

plusOne.value++ // 错误！

/*【可更改的】 */

const plusOne = computed({

  get: () => count.value + 1,

  set: (val) => {

    count.value = val - 1

  },

})

```

2.  watch

```js
// 直接侦听一个 ref

const count = ref(0)

watch(count, (count, prevCount) => {

  /* ... */

})

// 也可以使用数组来同时侦听多个源

watch([fooRef, barRef], ([foo, bar], [prevFoo, prevBar]) => {

  /* ... */

})
```

3.  watchEffect

- 定义：在响应式地跟踪其依赖项时立即运行一个函数，并在更改依赖项时重新运行它
- 第一个参数：​`​effect​`​​，顾名思义，就是包含副作用的函数。如下代码中，副作用函数的作用是：当​`​count​`​ 被访问时，旋即（随即）在控制台打出日志。
- 返回值：也是一个函数，显式调用可以清除 watchEffect，组件卸载时会被隐式调用

```js
const count = ref(0);

const stop = watchEffect(() => console.log(count.value)); // -> logs 0

setTimeout(() => {

  count.value++; // -> logs 1

}, 100);

// 清除watchEffect

stop();
```

- 清除副作用（onInvalidate）

​`​watchEffect​`​​ 的第一个参数——​`​effect​`​​函数——自己也有参数：叫​`​onInvalidate​`​​，也是一个函数，用于清除 ​`​effect​`​ 产生的副作用。

​`​onInvalidate​`​ 被调用的时机很微妙：它只作用于异步函数，并且只有在如下两种情况下才会被调用：

1.  当​`​effect​`​ 函数被重新调用时
2.  当监听器被注销时（如组件被卸载了）

```js
import { asyncOperation } from "./asyncOperation";

const id = ref(0);

watchEffect((onInvalidate) => {

  const token = asyncOperation(id.value);

  // onInvalidate 会在 id 改变时或停止侦听时，取消之前的异步操作（asyncOperation）

  onInvalidate(() => {

    token.cancel();

  });

});
```

- 第二个参数：options

主要作用是指定调度器，即何时运行副作用函数。

```js
// fire before component updates

watchEffect(

  () => {

    /* ... */

  },

  {

    flush: "pre",

    onTrigger(e) {

      // 依赖项变更导致副作用被触发时，被调用

      debugger;

    },

    onTrack(e) {

      // 依赖项变更会导致重新追踪依赖时，被调用【调用次数为被追踪的数量】

      debugger

    },

  }

);
```
