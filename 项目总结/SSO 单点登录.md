 [共享 Cookie 实现 SSO 单点登录](https://juejin.cn/post/7083319128144445477)

由于 `HTTP` 协议具有 ` 无状态 ` 的特点，对于事物的处理没有记忆能力，每一次请求都是独立的，这样就无法保持客户端与服务端的会话状态

也就是说无法根据之前的请求状态进行本次的请求处理，以此来减少服务器的 `CPU` 占用以及内存资源的消耗

在这样的背景下，`Cookie` 技术应运而生

`Cookie` 是服务端保存在浏览器端的文本信息（一般不超过 `4KB`），浏览器每次向服务器端发出请求，都会附带上这段文本信息

遵循 ` 同源策略 `，默认情况下，跨域请求不附带 `Cookie`

### Cookie 的设置

- 概览

    一个简单的、轻量的处理 `cookies` 的 `JS API`

- 基本用法

    import Cookies from 'js-cookie'
    Cookies.set('name', '老夫子', { expires: 7 }) // 有效期 7 天
    Cookies.get('name') // => '老夫子'
    复制代码

    [详细使用文档](https://link.juejin.cn/?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Fjs-cookie "https://www.npmjs.com/package/js-cookie")


### Cookie 的缺点

- 存储空间小

    最大只有 `4KB`，当 `Cookie` 超过 `4KB` 时会被自动裁切，所以只能用来存取少量的信息

- 性能浪费

    同一个域名下的所有请求，都会携带 `Cookie`，而有些请求是不需要携带 `Cookie` 的，这就会造成性能浪费和安全性隐患

- 其它缺点

    通过更改浏览器设置可以禁用 `Cookie`

    由于原生接口不友好，导致数据操作不方便


### Cookie 的特点

- 存储时如果未设置 `Expires` 和 `Max-Age` 失效时间，则在浏览器关闭后自动删除

- 客户端的存储数据会自动通过请求头 `Cookie` 传输到服务端

- 服务端可以通过 `Set-Cookie` 响应头写 `Cookie` 到客户端

- `Cookie` 失效浏览器会自动将其删除


### Cookie 的作用域

- 设置 Domain 属性

    `Domain` 属性指定了哪些主机可以接受 `Cookie`，如果不指定，默认为 `Origin`，但不包含子域名

    当多个子域名需要共享 `Cookie` 信息的时候，就必须要指定 `Domain` 属性值为一级域名，指定 `Path` 属性值为根路径

    Set-Cookie: uid=5; Domain=taobao.com; Path=/
    复制代码

- 设置 Path 属性

    `Path` 属性指定了主机下的哪些路径可以接受 `Cookie`，子路径也可以被匹配

    Set-Cookie: uid=5; Domain=taobao.com; Path=/docs
    复制代码

- 共享登录状态

    ![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/83f67241274c4c29bfa30ccb5eed9308~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

    子域会自动继承父域中的 `Cookie` ，利用这个特点可以实现 ` 单点登录 `

    ` 前提是应用系统的域名必需得建立在一个共同的主域名之下，不支持跨主域名 `


### Cookie 的安全性

- 设置 Secure 属性

    `Secure` 属性指定浏览器只有在 `HTTPS` 加密协议下，才会将 `Cookie` 发送到服务器端，以此来保护 `Cookie` 在传输过程中不被窃取和篡改。如果是 `HTTP` 非加密协议，浏览器会自动忽略服务器端发来的 `Secure` 属性

- 设置 HttpOnly 属性

    `HttpOnly` 属性可以防止客户端脚本通过 `document.cookie` 等方式访问 `Cookie`，有助于避免 `XSS` 攻击


### Cookie 的 SameSite 属性

- 属性作用

    从版本 51 的 Chrome 浏览器开始，其 `Cookie` 新增了一个 `SameSite` 属性。这个属性可以让 `Cookie` 在跨站请求时不会被发送，用来阻止跨站请求伪造攻击 `CSRF` ，从而减少安全风险和用户追踪

- 属性值

    `Strict` 浏览器将只在访问相同站点时发送 Cookie

    `None` 版本 80 以下的 Chrome 浏览器默认规则，无论是否跨站都会发送 Cookie，必须同时设置 Secure 属性，否则无效

    `Lax` 版本 80 以上的 Chrome 浏览器默认规则，只有导航到目标地址的 Get 请求才会发送 Cookie，即 超链接、预加载请求、GET 表单


### Cookie 和 Session 的区别

- 安全性

    Cookie 数据保存在客户端，Session 数据保存在服务端，所以 Session 比 Cookie 安全

- 存取值的类型不同

    Cookie 只支持存字符串数据，想要设置其他类型的数据，需要将其转换成字符串；Session 可以存任意数据类型

- 有效期不同

    Cookie 可设置为长时间保持，比如我们经常使用的默认登录功能；Session 一般失效时间较短，客户端关闭（默认情况下）或者 Session 超时都会失效

- 存储大小不同

    单个 Cookie 保存的数据不能超过 4K；Session 可存储数据远高于 Cookie，但是当访问量过多，会占用过多的服务器资源


### Cookie、Session 鉴权原理

- Session 概览

    `Session` 是一种记录服务器和客户端会话状态的机制

    `Session` 是基于 `Cookie` 实现的，Session 存储在服务器端，SessionId 会被存储到客户端的 Cookie 中

- 认证流程

    1、客户端使用账号和密码请求登录

    2、服务端收到请求，去验证账号与密码

    3、验证通过后，服务端会生成一个随机字符串（Session Id），既要在服务端保存，又要以 `Set-Cookie` 的形式写给客户端

    4、签名，通过秘钥对 `Sid` 进行签名处理，避免 `Sid` 被篡改

    5、客户端每次向服务端请求资源时都需要携带 `Sid`

    6、服务端收到请求后解析请求 `Cookie` 中的 Sid，然后与服务端存储的 `Sid` 进行比对，来确定请求的合法性


![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/45629589ab204dd6ace2ecfb84a73e51~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

### 基于 Session 的优化

- Session 的缺点

    服务端必须存储所有在线用户的 `SessionId`

    同时在线的人数越多开销就越大，影响服务器性能

- 优化方案

    扩展服务器做集群，这样会出现分布式 `SessionId` 无法共享的问题。

    可以采用 `Session` 集中式管理（如 redis），在服务器上使用缓存技术，集中管理所有的 `Session`

    所有的服务器都从这个 ` 存储介质 ` 中存取对应的 `Session`，从而实现 `Session` 共享
