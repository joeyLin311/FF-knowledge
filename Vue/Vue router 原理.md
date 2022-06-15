---
date created: 2021-12-09 22:53
date updated: 2022-06-14 01:19
---

#Vue

# Vue Router

 [知乎](https://zhuanlan.zhihu.com/p/37730038)
 [思否](https://segmentfault.com/a/1190000023662742)
 [vueJs 技术揭秘](https://ustbhuangyi.github.io/vue-analysis/v2/vue-router/)

 [关于 $ router 和  $route](https://segmentfault.com/a/1190000022666268)

## 概览

**Vue-Router 支持三种路由方式: `hash`, `history`, `abstract`, 并且提供了 `<router-link>` 和 `<router-view>`2 种组件, 还有简单的配置和方便的 API**

## 路由注册

路由对象通过 `Vue.use` 进行插件注册, 利用 `Vue.mixin` 方法, 在 Vue-Router 内部的 `install` 方法给每个组件注入 `beforeCreate` 和 `destoryed` 钩子函数，在 `beforeCreate` 做一些私有属性定义和路由初始化工作
接着给 Vue 原型上定义了 `$router` 和 `$route` 两个属性的 get 方法, 方便我们在 vue 文件中直接使用 `this.$router` 和 `this.$route` 访问路由 API
然后又通过 `Vue.component` 方法定义了全局的 `<router-link>` 和 `<router-view>` 2 个组件, 方便在模板处直接使用路由.

## VueRouter 对象

VueRouter 对象实现是一个 Class, 其中定义了相关的属性和方法, 在 `new VueRouter()` 时, 执行到 `beforeCreate` 钩子函数会执行 `router.init` 方法, 然后执行 `history.transitionTo` 方法, 在该方法中又有 `match` 方法进行路由规则匹配

## matcher

matcher 的数据结构表现如下:

```js
export type Matcher = {
  match: (raw: RawLocation, current?: Route, redirectedFrom?: Location) => Route;
  addRoutes: (routes: Array<RouteConfig>) => void;
};
```

`window.location` 在 VueRouter 中是以 `type Location` 形式声明, 其中结构化描述了 URL 的组成部分. `Route` 是返回的结果, 它表示的是路由中的一条线路, 除了描述 `location` 意外, 还包含了 `matcher` 匹配到的所有路由信息 `RouterRecord`.

在 `matcher` 内部, 通过 `createRouteMap` 函数构建路由映射表, 其中包含 `pathList` 存储所有 `path` , `pathMap` 表示 `patch` 到 `RouterRecord` 的映射关系, `nameMap` 表示 `name` 到 `RouterRecord` 的映射关系.
`RouterRecord` 是通过 `addRouteRecord` 生成的, 它通过遍历 `routes` 数组为每个 `route` 生成一条记录. 其中:

- 通过 `path-to-regexp` 库把 `path` 解析成正则表达式的扩展.
- `component` 对象记录 `routes` 中组件的实例, 还有组件的依赖关系, 并通过深度遍历形成 `routes` 的树型结构
### addRoutes
`addRoutes` 方法的作用是动态添加路由配置, 直接调用 `createRouteMap` 传入新的 `routes` 配置
### match
`match` 方法返回的是一个路径, 它的作用的是根据传入的 `raw` 和当前的路径 `currentRoute` 计算出一个新的路径并返回
 [`matcher` 相关的主流程](https://ustbhuangyi.github.io/vue-analysis/v2/vue-router/matcher.html#match)

## 路径切换
VueRouter 中路径切换的方法是 `history.transitionTo`, 经过上述 `match` 方法返回的新路径, `transitionTo` 内部根据目标 `location` 和当前路径 `current` 执行 match 方法匹配目标路径.  然后执行 `confirmTransition` 方法执行切换 ( 里面调用 `updateRoute` 进行组件更新)

然后拿到  `updated`、`activated`、`deactivated` 3 个 `ReouteRecord` 数组, 执行一些列钩子函数, 期间还有路由守卫的参与

所以当执行 `route.push` 方法的时候, 内部会执行 `transitionTo` 方法做路径切换, 然后在完成的回调函数中, 执行 `pushHash` 函数(以 hash 模式为例), 如果浏览器支持, 就执行 `pushState` 方法, 它会调用浏览器原生的 `history` 对象的 `pushState` 或者 `replaceState` 接口, 更新 URL 地址, 并把当前 URL 压入历史栈中.

如果浏览器页面返回, 则触发 `popState` 事件, 拿到当前要跳转的 hash , 执行 `transitionTo` 再做路径切换.

### 小总结
路由变化的过程中, 路由始终会维护当前的线路, 路由切换的时候会把当前线路切换到目标线路, 切换过程中会执行一系列的导航守卫钩子函数, 会更改 url, 同样也会渲染对应的组件, 切换完毕后会把目标线路更新替换至当前线路, 这样作为下一次的路径切换时的依据.
## 模拟 VueRouter 的 hash 模式的实现

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
