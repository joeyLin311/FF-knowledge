---
date created: 2022-07-15 00:22
---

# 摘要

LCP (Largest Contentful Paint) 是一个以用户为中心的性能指标，可以测试用户感知到的页面加载速度，因为当页面主要内容可能加载完成的时候，它记录下了这个时间点。一个快速的 LCP，可以让用户感受到这个页面的可用性。

以往，对开发者而言，要测试一个页面主要内容加载并呈现给用户的速度是一个很大的挑战。

旧指标，像 `load` 和 `DOMContentLoaded` 并不是很好，因为它不一定跟用户屏幕上看到的内容相对应。然而新的以用户为中心的指标，比如 `FCP (First Contentful Paint)` 只是记录了加载体验的最开始。如果页面显示的是启动图片或者 loading 动画，这个时刻对用户而言没有意义。

在过去，我们也有推荐的性能指标，如：`FMP (First Meaningful Paint)` 和 `SI (Speed Index)` 可以帮我们捕获更多的首次渲染之后的加载性能，但这些过于复杂，而且很难解释，也经常出错，没办法确定主要内容什么时候加载完。

有时候越简单越好。经过 W3C 性能工作小组的讨论和谷歌的研究，我们发现了一个更精确的测量方式，当最大的内容块渲染完的时候，基本上主内容都加载完了。

# 什么是 LCP

LCP 指标代表的是视窗最大可见图片或者文本块的渲染时间。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ba47728ead9a4975ba66b42194ddb312~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

## 一个好的 LCP 得分是多少？

一般是至少 75% 用户能够在 2.5 秒内，包括移动和桌面设备。

## 如何确定元素类型？

跟 [Largest Contentful Paint API](https://link.juejin.cn?target=https%3A%2F%2Fwicg.github.io%2Flargest-contentful-paint%2F "https://wicg.github.io/largest-contentful-paint/") 里面定义的一致，包含以下几种元素类型:

- `<img>` 元素
- `<svg>` 中的 `<image>` 元素
- `<video>` 元素（如果定义了封面图，会影响 LCP）
- 带 `url()` 背景图的元素
- 块级元素带有文本节点或者内联文本子元素

要注意的是，限制元素在这些范围内只是为了一开始简单一点，以后可能会加入更多的元素。

## 如何确定元素的大小？

LCP 中元素尺寸的定义就是用户视窗所见到的尺寸。如果元素在视窗外面，或者如果元素被 overflow 裁剪了，这些部分不计算入 LCP 的元素尺寸。

- 对于已经被设置过大小的图片元素而言，LCP 的尺寸就是设置的尺寸，并非图片原始尺寸。

- 对于文本元素而言，只有包含所有文本节点的最小矩形才是 LCP 的尺寸。

- 对于其他元素而言，css 样式里的 margin、padding 和 border 都不算。

## LCP 什么时候上报？

由于 Web 页面都是分阶段加载的，所以最大元素可能随时会发生变化。

为了捕获这种变化，浏览器会派发一个类型是 `largest-contentful-paint` 的 `PerformanceEntry` 对象，表示浏览器绘制第一帧的时候最大的元素。在后来的渲染帧中，如果最大元素发生变化，会再次派发一个 `PerformanceEntry` 对象。

举个例子，一个页面包含一个文本标题和一张图片，首帧渲染的时候，文本是最大元素，随着图片加载完成，图片或成为最大元素。

很重要的一点是，只有元素已经渲染，而且对用户可见，这个元素才可以被当成最大元素。未加载的图片未被渲染，所以不会被考虑在内。如果用了自定义字体的文本，在字体文件加载之前，也不会被当成最大元素。

后来被 JS 添加到 dom 树上的节点，如果尺寸大于之前的最大元素，浏览器也会派发一个 `PerformanceEntry` 对象。

如果元素被移除，这个元素不会被当成最大元素。如果图片元素的 src 属性更改了，在新图片加载之前，这个元素也不会被当成最大元素。

> 未来，移除元素可能还会被当成最大元素，具体的需要等待 google 最新研究

如果用户产生了交互行为（如点击、滚动、按键等），浏览器会停止上报新的 entry，因为用户的交互行为可能会导致页面元素的可见性变化。

对统计而言，你只需要统计最近上报的一次 `PerformanceEntry` 即可。

> 需要注意的是，如果用户打开页面之后，切换到后台，等再次回到前台的时候，用户产生了交互行为，该埋点就不准确了，会比真实的时间延长一些。

## 加载时间 vs 渲染时间

因为安全的因素，如果跨域图片缺少 `Timing-Allow-Origin` 的 header，图片渲染的时间无法拿到，相应的，可以用图片加载时间来替代。不过推荐还是尽可能的加上这个 header。

## 元素的布局和尺寸的变化如何捕获？

为了使计算和派发新的条目的性能消耗保持较低，元素的大小和位子的改变不会创建新的 LCP 条目。只有元素在视窗内的初始大小和位置才会被考虑。

这也意味着图片如果在屏幕外渲染，后面被移动到了屏幕内不会上报。反之，如果图片开始在屏幕内渲染，后被移到了屏幕外，也会被上报。

## 示例

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ccc0722d7be74e79baf4d2dcd6ef96e3~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ecf8feb2d15b4d9a9d53802b27f06ecf~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

上面两张图都是在页面加载过程中，最大元素发生变化。第一张图，新的内容被加入到 dom，而且这个元素成为了最大元素。第二张图，布局发生了变化，之前在视窗中的元素被移出了视窗外。

一般来说，后加载的内容比之前已经渲染的内容要更大，但也有特例。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/698ff44c4e8c4790a7d5b7d507bcc18a~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/17a8030606714c0fbbc92d8fbc7ac821~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

第一张图，Instagram logo 加载的比较早，而且直到页面完全加载，它依旧是最大元素。第二张图，一开始是输入框中的文本元素最大，后来是段落文本最大，之后加载的每一张图片元素都没段落大。

# 如何测量 LCP？

最简单的方式就是使用 [webvitals](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FGoogleChrome%2Fweb-vitals "https://github.com/GoogleChrome/web-vitals") 的 js 库，代码如下:

```js
import {getLCP} from 'web-vitals';

// Measure and log the current LCP value,
// any time it's ready to be reported.
getLCP(console.log);
复制代码

```

## 如果最大元素并非主要内容怎么办？

使用 [Element Timing API](https://link.juejin.cn?target=https%3A%2F%2Fwicg.github.io%2Felement-timing%2F "https://wicg.github.io/element-timing/") 来设置自己的自定义指标。

# 如何改善 LCP？

LCP 主要受以下四方面影响:

- 较慢的服务器响应时间
- 阻塞渲染的 js 或者 css
- 资源加载时间
- 客户端渲染性能

具体的优化敬请期待后面的文章。

# 总结

LCP 对我们来说是一个非常直观的可以看到真实用户的体验的数据，不过因为设备兼容性问题，目前仅在 Android 8 及其以上系统，pc 上 Chrome 浏览器等支持，如果使用 webvitals 的 js 库，如果不支持，也不会有影响，库本身不会报错，也不会给你回调。在现有的安卓和 iOS 的市场占有率来看，如果项目中集成 LCP 指标统计，可以看到大概 50% 用户的体验数据，还是有集成的必要。

# 参考

[web.dev/lcp/](https://link.juejin.cn?target=https%3A%2F%2Fweb.dev%2Flcp%2F "https://web.dev/lcp/")
