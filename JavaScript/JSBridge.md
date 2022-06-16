---
date created: 2021-12-09 22:55
date updated: 2021-12-17 16:14
---

# JSBridge

在原生开发领域, 移动端操作系统中都包含 JavaScript 运行环境, 比如 Android 的 WebView 和 iOS 的 JSCore. 使用 JS 编写运行在移动平台的 APP 称为可能. 在这过程中, 编写的 JS 应用如何与原生系统进行通信交互, JSBridge 应运而生.
移动端的混合开发中的 JSBridge 主要被应用在两种性质的技术方案:

- 基于 Web 的 Hybrid 解决方案: 如微信浏览器, 各公司的 Hybrid 方案
- 非基于 Web UI 但是业务逻辑基于 JS 的结局方案: React-Native
- 微信小程序: 基于 Web UI, 不过为了追求运行效率, 对视图层和逻辑层进行了隔离, 实现了双线程模型的移动端解决方案.

![[jsbridge1.png]]

![[jsbridge2.png]]

## JSBridge 用途

JSBridge 顾名思义的就是在 Native 和 JS 代码端构建一个桥梁, 一是提供给 JS 端调用系统原生 API, 二是实现 Native 端和非 Native 端的数据和消息通信, 而且是双向通信.

**关于双向通信:**

- JS 向 Native 发送消息: 调用相关功能, 通知 Native 当前的 JS 运行状态等
- Native 向 JS 发送消息: 提供回调结果, 消息推送, 通知 JS 当前 Native 状态等

## JSBridge 实现原理

在移动平台上 JS 运行在一个单独的 JS Context 环境中, 如 WebView 的 Webit 引擎, JSCore, 这些 Context 与原生的运行环境天然隔离, 所以两者之间的通讯类似与 RPC(Remote Procedure Call) 调用.
所以 JSBridge 主要逻辑就是: **通信调用** 和 **句柄解析调用**.

## JSBridge 的通信原理

JSBridge 有两种调用 Native 的方式: **注入 API** 和 **拦截 URL SCHEME**

**1. 注入 API**

注入 API 的主要原理是通过 WebView 暴露的接口, 向 JS context 中注入对象或者方法, 使其在 JS 嗲用是, 直接执行相应的 Native 代码

**2. 拦截 URL SCHEME**

URL SCHEME 是一种类似于 url 的链接, 是为了方便 app 直接互相调用设计的, 形式和普通的 url 近似, 主要区别是 protocol 和 host 一般是自定义的, 例如: `qunarhy://hy/url?url=ymfe.tech`, `protocol` 是 `qunarhy`, `host` 则是 `hy`.
拦截 URL SCHEME 的主要流程是: Web 端通过某种方式（例如 iframe.src）发送 URL Scheme 请求, 之后 Native 拦截到请求并根据 URL SCHEME（包括所带的参数）进行相关操作.
在时间过程中, 这种方式有一定的**缺陷**:

- 使用 iframe.src 发送 URL SCHEME 会有 url 长度的隐患
- 创建请求, 需要一定的耗时, 比注入 API 的方式调用同样的功能, 耗时会较长

> 【注】: 有些方案为了规避 url 长度隐患的缺陷,在 iOS 上采用了使用 Ajax 发送同域请求的方式,并将参数放到 head 或 body 里.这样,虽然规避了 url 长度的隐患,但是 WKWebView 并不支持这样的方式.
> 【注 2】: 为什么选择 iframe.src 不选择 locaiton.href ？因为如果通过 location.href 连续调用 Native,很容易丢失一些调用

JSBridge 接口实现
从上面的剖析中,可以得知,JSBridge 的接口主要功能有两个: 调用 Native（给 Native 发消息） 和 接被 Native 调用（接收 Native 消息）.因此,JSBridge 可以设计如下:

```jsx
window.JSBridge = {
  // 调用 Native
  invoke: function (msg) {
    // 判断环境,获取不同的 nativeBridge
    nativeBridge.postMessage(msg)
  },
  receiveMessage: function (msg) {
    // 处理 msg
  },
}
```

在上面的文章中,提到过 RPC 中有一个非常重要的环节是 句柄解析调用 ,这点在 JSBridge 中体现为 句柄与功能对应关系.同时,我们将句柄抽象为 桥名（BridgeName）,最终演化为 一个 BridgeName 对应一个 Native 功能或者一类 Native 消息. 基于此点,JSBridge 的实现可以优化为如下:

```jsx
window.JSBridge = {
  // 调用 Native
  invoke: function (bridgeName, data) {
    // 判断环境,获取不同的 nativeBridge
    nativeBridge.postMessage({
      bridgeName: bridgeName,
      data: data || {},
    })
  },
  receiveMessage: function (msg) {
    var bridgeName = msg.bridgeName,
      data = msg.data || {}
    // 具体逻辑
  },
}
```

JSBridge 大概的雏形出现了.现在终于可以着手解决这个问题了: **消息都是单向的,那么调用 Native 功能时 Callback 怎么实现的？**
对于 JSBridge 的 Callback ,其实就是 RPC 框架的回调机制.当然也可以用更简单的 JSONP 机制解释:

> 当发送 JSONP 请求时,url 参数里会有 callback 参数,其值是 当前页面唯一 的,而同时以此参数值为 key 将回调函数存到 window 上,随后,服务器返回 script 中,也会以此参数值作为句柄,调用相应的回调函数.

由此可见,callback 参数这个 唯一标识 是这个回调逻辑的关键.这样,我们可以参照这个逻辑来实现 JSBridge: 用一个自增的唯一 id,来标识并存储回调函数,并把此 id 以参数形式传递给 Native,而 Native 也以此 id 作为回溯的标识.这样,即可实现 Callback 回调逻辑.

```jsx
(function () {
    var id = 0,
        callbacks = {};

    window.JSBridge = {
        // 调用 Native
        invoke: function(bridgeName, callback, data) {
            // 判断环境,获取不同的 nativeBridge
            var thisId = id ++; // 获取唯一 id
            callbacks[thisId] = callback; // 存储 Callback
            nativeBridge.postMessage({
                bridgeName: bridgeName,
                data: data || {},
                callbackId: thisId // 传到 Native 端
            });
        },
        receiveMessage: function(msg) {
            var bridgeName = msg.bridgeName,
                data = msg.data || {},
                callbackId = msg.callbackId; // Native 将 callbackId 原封不动传回
            // 具体逻辑
            // bridgeName 和 callbackId 不会同时存在
            if (callbackId) {
                if (callbacks[callbackId]) { // 找到相应句柄
                    callbacks[callbackId](msg.data); // 执行调用
                }
            } elseif (bridgeName) {

            }
        }
    };
})();
```

当然,这段代码片段只是一个示例,主要用于剖析 JSBridge 的原理和流程,里面存在诸多省略和不完善的代码逻辑,读者们可以自行完善.

> 【注】: 这一节主要讲的是,JavaScript 端的 JSBridge 的实现,对于 Native 端涉及的并不多.在 Native 端配合实现 JSBridge 的 JavaScript 调用 Native 逻辑也很简单,主要的代码逻辑是: 接收到 JavaScript 消息 => 解析参数,拿到 bridgeName、data 和 callbackId => 根据 bridgeName 找到功能方法,以 data 为参数执行 => 执行返回值和 callbackId 一起回传前端. Native 调用 JavaScript 也同样简单,直接自动生成一个唯一的 ResponseId,并存储句柄,然后和 data 一起发送给前端即可.

## JSBridge 如何引用

### 由 Native 端进行注入

注入方式和 Native 调用 JavaScript 类似,直接执行桥的全部代码.
它的优点在于: 桥的版本很容易与 Native 保持一致,Native 端不用对不同版本的 JSBridge 进行兼容；与此同时,它的缺点是: 注入时机不确定,需要实现注入失败后重试的机制,保证注入的成功率,同时 JavaScript 端在调用接口时,需要优先判断 JSBridge 是否已经注入成功.

### 由 JavaScript 端引用

直接与 JavaScript 一起执行.
与由 Native 端注入正好相反,它的优点在于: JavaScript 端可以确定 JSBridge 的存在,直接调用即可；缺点是: 如果桥的实现方式有更改,JSBridge 需要兼容多版本的 Native Bridge 或者 Native Bridge 兼容多版本的 JSBridge.

## WeixinJSBridge
