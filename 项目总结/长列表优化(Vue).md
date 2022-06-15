---
date created: 2022-06-13 17:11
---

## 虚拟列表

核心思想就是处理在页面滚动式, 只改变列表在可视区域的渲染 DOM 节点.

1. 计算可视区域起始索引 `startIndex` 和当前可视区域结束的索引 `endIndex`
   1. `startIndex = Math.floor(scrollTop/itemHeight)`
   2. `endIndex = startIndex + (clientHeight/itemHeight)`
2. 再根据 `startIndex` 和 `endIndex` 的取值范围截取列表长度, 渲染到可视区域
3. 然后计算 `startOffset` 和 `endOffset` 增加上下部分列表内容, 起缓冲作用, 缓冲区域是为了解决白屏问题
4. 监听 onScroll 事件根据滚动位置动态改变可视列表, 还有缓冲区域,

### 优化处理

1. 滚动事件使用节流函数优化加载, 节约性能损耗
2. 滚动事件可以添加 [[requestAnimationFrame]] 函数减少丢帧现象
	1. [[requestAnimationFrame#如何渲染几万条数据并不卡住界面]]
3. 对于**不定高度的列表项**,
   1. 可以对组件属性进行扩展, 增加高度属性传参
   2. 渲染至屏幕外, 计算列表高度后再渲染至可视区域内, 但是增加了渲染性能损耗,
   3. 预估高度先行渲染, 缓存渲染后列表项的高度信息, 在滚动的时候通过缓存信息获取列表项高度进行设置列表总高度.
   4. 如果列表项包含的内容复杂, 比如图片类的, 无法计算真实高度, 可以使用`ResizeObserver` 监听列表项内容区域的高度改变, 获得实际的列表项高度, 但是浏览器兼容情况一般
4. 关于缓冲区白屏的解决方法:
   1. 设置缓冲区比例, 
   2. 上下区域都增加一定的渲染条数
5. 之前使用`监听 onScroll 事件` 来触发可视区域的渲染, `onScroll` 触发的频率过快可能会导致重复计算的问题, 性能影响比较大.
   1. 可以使用 `IntersectionObserver` 代替 `onScroll` 事件, 它可以监听目标元素是否出现在可是区域内, 在监听的回调函数中执行可视区域的更新, 而且该 API 的回调是异步触发 , 不随着目标元素的滚动而触发, 效能消耗较低.

## 参考链接

[高性能渲染十万条列表](https://juejin.cn/post/6844903982742110216)
[IntersectionObserver-MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver)
[ResizeObserver-MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/ResizeObserver)