#Vue
## Vue Router
[知乎](https://zhuanlan.zhihu.com/p/37730038)
[思否](https://segmentfault.com/a/1190000023662742)
[vueJs技术揭秘](https://ustbhuangyi.github.io/vue-analysis/v2/vue-router/)
**Vue-Router 支持三种路由方式: `hash`, `history`, `abstract`, 并且提供了 `<router-link>` 和 `<router-view>`2种组件, 还有简单的配置和方便的API**
[关于`$router`和`$route`](https://segmentfault.com/a/1190000022666268)
### 路由注册
#### Vue.use() 注册插件,
#### 路由安装
执行`install` 方法, 然后利用 `Vue.mixin()` 把 `beforeCreate` 和 `destoryed` 狗子函数注入到每一个组件中
在 Vue 原型上定义了 `$router` 和 `$route` 两个属性的 get 方法, 方便我们在vue文件中直接使用 `this.$router` 和 `this.$route` 访问路由API
通过 `Vue.component` 方法定义了全局的 `<router-link>` 和 `<router-view>` 2 个组件, 方便在模板处直接使用路由
#### 小总结
Vue 编写插件的时候通常要提供静态的 `install` 方法，我们通过 `Vue.use(plugin)` 时候，就是在执行 `install` 方法。`Vue-Router` 的 `install` 方法会给每一个组件注入 `beforeCreate` 和 `destoryed` 钩子函数，在 `beforeCreate` 做一些私有属性定义和路由初始化工作，下一节我们就来分析一下 `VueRouter` 对象的实现和它的初始化工作。