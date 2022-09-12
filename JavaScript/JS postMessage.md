> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6844903917684260877)

首先回顾一下什么是同源策略

### 同源策略

1. 协议（通常为 https）
2. 端口号（443 为 https 的默认值）
3. 主机 (两个页面的模数 Document.domain 设置为相同的值)

提到 iframe 有很多人嗤之以鼻，iframe 的缺点很多，但是在某些特定场景下，也可以发挥不错的作用，首先看一下 iframe 的缺点：

### iframe 缺点以及解决方案

缺点:

1.iframe 会阻塞主页面的 Onload 事件；

2. 相同域 iframe 和主页面共享 http 连接池，所以如果相同域用多个 iframe 会阻塞加载

解决方案：

```js
动态生成iframe，在主页面加载完成后去生产iframe加载，从而避免阻塞的影响


```

3.iframe 对 SEO 不友好

```js
在广告位以及内部系统等适合的场景中使用iframe


```

### iframe 跨域通信

#### postMessage

##### API 简介

```js
otherWindow.postMessage(message, targetOrigin, [transfer]);


```

`otherWindow`是 window 对象的引用：

1. `window.open`/`window.opener` 使用 window.open 返回的对象

2. `HTMLIFrameElement.contentWindow` iframe 元素的 contentWindow 属性

3. `window.parent` 当前窗口的父窗口对象, 如果没有返回自身

4. `window.frames` 当前窗口的所有直接子窗口

看起来比较复杂，如果想实现主页面和 iframe 交互，只需要第二项和第三项

`targetOrigin` ：

1.`*`代表可以发送到任意 origin

2.`https:www.xxx.com:443`可以指定特定的`origin`发送，默认端口号可以省略

##### 使用

以主页面中嵌套一个 iframe 为案例说明

主页面：

```js
  //html
  <iframe src="./iframe.html" id="iframe" frameborder="0" scrolling="no" width="100%"></iframe>


```

```js
//js
//发送message
var iframe = document.getElementById('iframe')
iframe.contentWindow.postMessage(data, '*')


```

子页面：

```js
//js
//监听message事件
window.addEventListener("message", receiveMessageFromIndex, false);//注意ie中事件绑定是attachEvent
//回调函数
function receiveMessageFromIndex(event) {
//传递的data可以从event.data中获取到
}


```

```js
//发送消息给主页面
window.parent.postMessage(data, '*');
//主页面监听方式和子页面一样


```

这样就比较简单的实现了主页面和子页面的双向通信。

##### 兼容性

需要注意的是`postMessage` `API`中的`message`在 ie8/ie9 等一些低版本浏览器中，中是不支持除`String`以外的其他类型时（因为不支持结构化克隆算法），所以，如果要兼容低版本 ie，需要通过`JSON.strigify`以及`JSON.parse`去发送和接受。

这里给了一个主页面以及子页面的双向通信的 demo：

[演示地址](https://link.juejin.cn?target=http%3A%2F%2Fblog.moonduffy727.ml%2Fdemonstration-repository%2Fiframe%2FpostMessage%2Findex.html "http://blog.moonduffy727.ml/demonstration-repository/iframe/postMessage/index.html")

[仓库地址](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FMoonDuffy%2Fdemonstration-repository%2Ftree%2Fmaster%2Fiframe%2FpostMessage "https://github.com/MoonDuffy/demonstration-repository/tree/master/iframe/postMessage")
