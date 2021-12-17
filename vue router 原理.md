---
date created: 2021-12-09 22:53
date updated: 2021-12-17 16:22
---

#Vue

## Vue Router

[知乎](https://zhuanlan.zhihu.com/p/37730038)
[思否](https://segmentfault.com/a/1190000023662742)
[vueJs 技术揭秘](https://ustbhuangyi.github.io/vue-analysis/v2/vue-router/)
**Vue-Router 支持三种路由方式: `hash`, `history`, `abstract`, 并且提供了 `<router-link>` 和 `<router-view>`2 种组件, 还有简单的配置和方便的 API**
[关于 `$router` 和 `$route`](https://segmentfault.com/a/1190000022666268)

### 路由注册

#### Vue.use() 注册插件,

#### 路由安装

执行 `install` 方法, 然后利用 `Vue.mixin()` 把 `beforeCreate` 和 `destoryed` 狗子函数注入到每一个组件中
在 Vue 原型上定义了 `$router` 和 `$route` 两个属性的 get 方法, 方便我们在 vue 文件中直接使用 `this.$router` 和 `this.$route` 访问路由 API
通过 `Vue.component` 方法定义了全局的 `<router-link>` 和 `<router-view>` 2 个组件, 方便在模板处直接使用路由

#### 小总结

Vue 编写插件的时候通常要提供静态的 `install` 方法，我们通过 `Vue.use(plugin)` 时候，就是在执行 `install` 方法。`Vue-Router` 的 `install` 方法会给每一个组件注入 `beforeCreate` 和 `destoryed` 钩子函数，在 `beforeCreate` 做一些私有属性定义和路由初始化工作，下一节我们就来分析一下 `VueRouter` 对象的实现和它的初始化工作。

### 模拟 VueRouter 的 hash 模式的实现

实现思路和 History 模式类似，把 URL 中的 # 后面的内容作为路由的地址，可以通过 hashchange 事件监听路由地址的变化。

```javascript
let _Vue = null
class VueRouter {
  static install(Vue) {
    // 1.判断当前插件是否被安装
    // 如果已经安装直接 return 掉
    if (VueRouter.install.installed) return
    VueRouter.install.installed = true
    // 2.把 Vue 的构造函数记录到全局变量
    _Vue = Vue
    // 3.把创建 Vue 的实例传入的 router 对象注入到 Vue 实例上
    // _Vue.prototype.$router = this.$options.router
    // 执行混入
    _Vue.mixin({
      beforeCreate() {
        // 判断 router 是否已经挂载到了 Vue 实例上
        if (this.$options.router) {
          _Vue.prototype.$router = this.$options.router
          this.$options.router.init()
        }
      },
    })
  }
  constructor(options) {
    this.options = options
    // routerMap 对象记录路径和对应的组件
    this.routeMap = {}
    // 响应式
    this.data = _Vue.observable({
      current: '/', // 当前默认路径
    })
    this.init()
  }
  init() {
    this.createRouteMap()
    this.initComponent(_Vue)
    this.initEvent()
  }
  createRouteMap() {
    // 目标 routes => [{ name: '', path: '', component: }]
    // 遍历所有的路由规则, 把路由规则解析成键值对的形式存储到 routeMap 中
    this.options.routes.forEach(route => {
      this.routeMap[route.path] = route.component
    })
  }
  initComponent(Vue) {
    Vue.component('router-link', {
      // 外部接收参数
      props: {
        to: String,
      },
      // render 函数
      render(h) {
        // 参数 h 创建虚拟 DOM, 在 render 函数中调用 h 函数并将结果返回, 接收三个参数
        // 参数1: 元素对应的选择器
        // 参数2: 标签设置相关属性, attrs 对象中为 DOM 对象属性
        // 参数3: 元素中如果有子元素则传递进去, 再次默认为 default
        return h(
          'a',
          {
            attrs: {
              href: this.to,
            },
            // 可以额外添加 on 配置项注册元素事件
            on: {
              click: this.clickhander,
            },
          },
          [this.$slots.default]
        )
      },
      methods: {
        clickhander(e) {
          // 时间参数 e
          // 改变浏览器地址栏 pushiState 不向服务器发送请求
          // history
          // history.pushState({}, '', this.to) // data title url
          // hash
          history.pushState({}, '', this.to)
          this.$router.data.current = this.to
          e.preventDefault()
        },
      },
      // template:"<a :href='to'><slot></slot><>"
    })
    const self = this
    Vue.component('router-view', {
      render(h) {
        // self.data.current
        // 注意 this 问题, 此时this的指向非目前的 class, 所以在外面先用变量存储 this
        const cm = self.routeMap[self.data.current]
        return h(cm)
      },
    })
  }
  // initEvent() {
  //   //
  //   window.addEventListener("popstate", () => {
  //     this.data.current = window.location.pathname
  //   })
  // }

  // 监听页面 load 和 hashchange 方法，这个地方有个判断
  // 如果当前页面的 hash 不存在，则自动加上 '#/' ,并加载 '/' 的组件
  initEvent() {
    window.addEventListener('load', this.hashChange.bind(this))
    window.addEventListener('hashchange', this.hashChange.bind(this))
  }
  ashChange() {
    if (!window.location.hash) {
      window.location.hash = '#/'
    }
    this.data.current = window.location.hash.substr(1)
  }
}
```
