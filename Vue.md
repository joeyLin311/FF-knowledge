#Vue
- [[diff算法]]
- [[Vue2响应式原理]]
- [[vue2模板编译过程]]
- [[Vue2 生命周期]]
- [[vue router原理]]
- [[keepAlive原理]]
- [[nextTick]]
- [[vue2 vs vue3]]
- [[computedVSwatch]]
- **vue组件通信的常用方式有那些?**
	1. [[props]]
	2. 自定义事件[[vue event]]
	3. [[eventBus]]
	4. [[vuex]]
	5. 边界情况的 `$parent` `$root` `$refs`, 依赖注入provide/inject
	6. `$attrs` `$listeners`
## 什么是mvvm，mvc？区别？

### MVC（Model-View-Controller）

MVC是比较直观的架构模式，用户操作
->View（负责接收用户的输入操作） ->Controller（业务逻辑处理）-> Model（数据持久化）->View（将结果反馈给View）。MVC使用非常广泛，比如JavaEE中的SSH框架。


### MVVM（Model-View-ViewModel）

如果说MVP是对MVC的进一步改进，那么MVVM则是思想的完全变革。
它是将“数据模型数据双向绑定”的思想作为核心，因此在View和Model之间没有联系，
通过ViewModel进行交互，而且Model和ViewModel之间的交互是双向的，
因此视图的数据的变化会同时修改数据源，而数据源数据的变化也会立即反应view。
微信小程序前端使用mvvm。
