---
date created: 2021-12-09 22:55
date updated: 2021-12-17 16:15
---

#browser

# `<link>` 标签的 preload 和 prefetch 属性
 [Web 性能优化：Preload 与 Prefetch 的使用及在 Chrome 中的优先级](https://blog.fundebug.com/2019/04/11/understand-preload-and-prefetch/)
preload 和 prefetch 是一种简单而优秀的 web 性能优化方案, 它为开发者提供了可以更加细粒度地控制浏览器资源加载的方法.

在网络请求中, 如果遇到一些资源如: 图片, CSS, JS 等, 在它们执行前需要等待它们下载完之后开始, 那我们可以使用 preload/prefetch 进行预先加载, 不必等待网络的开销.

当使用 preload/prefetch 作为 `<link>` 标签的 rel 的属性值时, `as` 属性就成为了必选项. 它规定了 `link` 中加载资源的类型, 关乎内容的优先级, 请求匹配, 正确的内容安全策略的选择以及正确的 `Accept` 请求头的设置.

`as` 属性的值可以取: style, script, image, font, fetch, document, audio, video 等; 如果 `as` 属性在 preload 中被省略, 那么设置 preload 的资源将被当做异步请求处理.

## preload 提前加载

**preload 特点**

1.  preload 加载的资源是在浏览器渲染机制之前进行处理的, 并不会阻塞 `onload` 事件
2.  preload 可以支持加载多种类型的资源, 并且可以加载跨域资源.
3.  preload 加载 JS 脚本时, 其加载和执行的过程是分离的. 即 preload 会预加载相应的脚本代码, 待到需要时再自行调用

## prefetch 预判加载

prefetch 是一种利用浏览器的空闲时间加载一些将来可能在页面中用到的资源的一种获取机制; 通常可以用于加载非首页的其它页面所需要的资源, 以便加快后续页面的首屏打开速度.

**prefetch 特点**
prefetch 加载的资源可以获取非当前页面所需要的资源, 并且将其缓存至少 5 分钟(无论资源是否可以缓存); 并且, 当页面跳转时, 未完成的 prefetch 请求不会被中断.

## 两者补充对比

1.  preload/ prefetch 请求的资源都被缓存在浏览器的 HTTP cache 中, 如果资源不可被缓存, 那么被使用前缓存在 Memory cache 中

 > Chrome 一共有四种缓存类型: HTTP cache, memory cache, Service Worker cache, Push cache

2.  preload / prefetch 都没有同域名限制, 可以通过设置 `corssorigin` 开启跨域资源加载

3.  preload 主要用于预加载当前页面需要的资源; prefetch 主要用于加载将来页面需要的资源

4.  无论资源是否可以被缓存, prefetch 的资源会存储在 net-stack cache 至少 5 分钟

5.  preload 需要使用 `as` 属性自行特定的资源类型一边浏览器为其分配一定的优先级, 使其能正确加载资源
