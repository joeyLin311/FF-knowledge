# 写在前面

说到 cookie、token 应该没有人不知道的吧，而如果说让大家说出 cookie、token 到底是什么关系，大家能说出来吗，往往这种看似简单的东西，一到关键的场面，就很容易让我们陷入尴尬，明明知道是什么，却解释不清楚，被 Pass 后一脸冤枉，因此，本篇我们就来稍微回顾下 cookie 相关的基础知识，给自己充充电吧！！！！

# Cookie

Cookie，有时也用其复数形式 Cookies。类型为小型文本文件，是某些网站为了辨别用户身份，进行 Session 跟踪而储存在用户本地终端上的数据（通常经过加密），由用户客户端计算机暂时或永久保存的信息。 看完 “百度百科” 的概念解释，依然是流于文字，我们来从原理方面解释下。

## 为什么要有 Cookie 呢？

我们都知道一般接口但是通过 HTTP 协议来进行数据交换的，而 HTTP 协议的特点是，无状态，工作前通过三次握手建立连接，工作完成后立刻通过四次挥手断开连接，每次连接都是独立存在的，没有任何状态将请求串联成一个整体，因此每次都需要重新验证是身份，即耗费了性能，也给黑客的攻击留下隐患。

那么 Cookie 的作用是什么呢，它的出现，就是来弥补 HTTP 无状态的问题的，Cookie 可以作为一个状态保存的状态机，用来保存用户的相关登录状态，当第一次验证通过后，服务器可以通过 set-cookie 令客户端将自己的 cookie 保存起来，当下一次再发送请求的时候，直接带上 cookie 即可，而服务器检测到客户端发送的 cookie 与其保存的 cookie 值保持一致时，则直接信任该连接，不再进行验证操作。大致的过程如下图

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4210970fd4f641ce860eccfde440e16d~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

# Token

Token, 令牌，代表执行某些操作的权利的对象，看了概念的我白了一眼，还是好好介绍一下吧。Token，简单来说，就是类似 cookie 的一种验证信息，客户端通过登录验证后，服务器会返回给客户端一个加密的 token，然后当客户端再次向服务器发起连接时，带上 token，服务器直接对 token 进行校验即可完成权限校验。

## 有了 Cookie 为什么还需要 Token？

Cookie 作为 HTTP 规范，其出现历史久远，因此存在一些历史遗留问题，比如跨域限制等，并且 Cookie 作为 HTTP 规范中的内容，其存在默认存储以及默认发送的行为，存在一定的安全性问题。相较于 Cookie，token 需要自己存储，自己进行发送，不存在跨域限制，因此 Token 更加的灵活，没有 Cookie 那么多的 “历史包袱” 束缚，在安全性上也能够做更多的优化。

## Token 传递过程

Token 的传递过程和 Cookie 差不多，依然是通过验证后返回，然后存储到客户端，当下一次再次发起请求时，携带该验证信息进行快速验证。如下图 ![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/126f7a69cf6c43e7bd413a9f520b0d10~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

## Token 有什么 优势？

从上面对于 Token 和 Cookie 的分析，我们知道了 Cookie 由于存储的内存空间只有 4kb，因此存储的主要是一个用户 id，其他的用户信息都存储在服务器的 Session 中，而 Token 没有内存限制，用户信息可以存储 Token 中，返回给用户自行存储，因此可以看出，采用 Cookie 的话，由于所有用户都需要在服务器的 Session 中存储相对应的用户信息，所以如果用户量非常大，这对于服务器来说，将是非常大的性能压力，而 Token 将用户信息返回给客户端各自存储，也就完全避开这个问题了。

# 总结

虽然 Token 作为更加现代的存储方式被广泛采用，但是 Cookie 仍然是非常重要的验证方式，因此，我们不仅需要掌握 Token 的验证方式，也需要掌握 Cookie ，通过对比两种方式，了解其差异和优缺点，我们才能够更好地理解客户端验证的方式，也能够为我们学习和分析相关安全问题提供很好的底层原理支撑！

# 往期好文推荐

======

[面试官：了解低代码平台吗？（数字孪生低代码平台探索）（三）](https://juejin.cn/post/7112503610977550343 "https://juejin.cn/post/7112503610977550343")

[面试官：了解低代码平台吗？（数字孪生低代码平台探索）（二）](https://juejin.cn/post/7112272235120820260 "https://juejin.cn/post/7112272235120820260")

[面试官：了解低代码平台吗？（数字孪生低代码平台探索）（一）](https://juejin.cn/post/7112092802200109069 "https://juejin.cn/post/7112092802200109069")

[面试官：说说从输入 URL 到页面显示到底经历了什么，体现一下你的知识广度](https://juejin.cn/post/7111719131233943566 "https://juejin.cn/post/7111719131233943566")

[面试官：作为前端，服务器相关了解多少？](https://juejin.cn/post/7109828845124976647 "https://juejin.cn/post/7109828845124976647")

[面试官：HTTPS 采用的是对称加密还是非对称加密？具体说说其加密过程](https://juejin.cn/post/7110967367097647117 "https://juejin.cn/post/7110967367097647117")

[面试官：说说 Cookie 和 Token 的区别？](https://juejin.cn/post/7111349594625146887 "https://juejin.cn/post/7111349594625146887")

[面试官：网络安全了解多少，简单说说？（一）](https://juejin.cn/post/7107636958045667364 "https://juejin.cn/post/7107636958045667364")

[面试官：网络安全了解多少，简单说说？（二）](https://juejin.cn/post/7107977566438293512 "https://juejin.cn/post/7107977566438293512")

[面试官：网络安全了解多少，简单说说？（三）](https://juejin.cn/post/7108282361426477069 "https://juejin.cn/post/7108282361426477069")

[面试官：网络安全了解多少，简单说说？（四）](https://juejin.cn/post/7108385355673370654 "https://juejin.cn/post/7108385355673370654")

[面试官：网络安全了解多少，简单说说？（五）](https://juejin.cn/post/7108775208848195620 "https://juejin.cn/post/7108775208848195620")

[面试官：网络安全了解多少，简单说说？（六）](https://juejin.cn/post/7109148004627529765 "https://juejin.cn/post/7109148004627529765")

[面试官：网络安全了解多少，简单说说？（七）](https://juejin.cn/post/7110235170279522334 "https://juejin.cn/post/7110235170279522334")

[面试官：网络安全了解多少，简单说说？（八）](https://juejin.cn/post/7110518244498210853 "https://juejin.cn/post/7110518244498210853")

[浅尝 | 从 0 到 1 Vue 组件库封装](https://juejin.cn/post/7077364973218824222 "https://juejin.cn/post/7077364973218824222")

[面试官：这么简单的正则表达式都不会？](https://juejin.cn/post/7077921971912048647 "https://juejin.cn/post/7077921971912048647")

[Webpack 打包类库踩坑](https://juejin.cn/post/7078324751634006023 "https://juejin.cn/post/7078324751634006023")

[面试官：你就只会 npm run build 吗？（Webpack 配置 Vue+Ts）](https://juejin.cn/post/7078679010376417288 "https://juejin.cn/post/7078679010376417288")

[面试官：连 VuePress 都没搭过还说开发过组件库？（VuePress 搭建）](https://juejin.cn/post/7079065629109518367 "https://juejin.cn/post/7079065629109518367")

[面试官: 连 Vue 视图更新都不会写？（Vue 视图更新原理【一】）](https://juejin.cn/post/7079437430230614024 "https://juejin.cn/post/7079437430230614024")

[面试官: 能不能手写 Vue 响应式？（Vue2 响应式原理【完整版】）](https://juejin.cn/post/7079807948830015502 "https://juejin.cn/post/7079807948830015502")

[面试官：能不能手写 Vue3 响应式（Vue3 原理解析之响应系统的实现）](https://juejin.cn/post/7084915514434306078 "https://juejin.cn/post/7084915514434306078")

[JS 优雅之道（JS 代码优化小 Tip）](https://juejin.cn/post/7102809229878099976 "https://juejin.cn/post/7102809229878099976")

[面试官：你真的会用 SVG 吗? (SVG 应用实战)](https://juejin.cn/post/7103570138154139679 "https://juejin.cn/post/7103570138154139679")

[面试官：说一下这个 Loading 动画实现思路 (CSS3 实现 Loading 动画)](https://juejin.cn/post/7104299683077423117 "https://juejin.cn/post/7104299683077423117")

[JS 扫盲题 (面试题梳理系列 （一）)](https://juejin.cn/post/7104668456875720741 "https://juejin.cn/post/7104668456875720741")

[面试官：你确定你说的防抖不是节流吗？(面试题梳理系列 （二）)](https://juejin.cn/post/7104912661926084639 "https://juejin.cn/post/7104912661926084639")

[面试官：除了 HTTP，你还用过什么通信协议？(Websocket 在数字孪生中的应用)](https://juejin.cn/post/7105412132266573831 "https://juejin.cn/post/7105412132266573831")

[面试官：你真的理解 Event Loop 吗?(JS 事件循环)](https://juejin.cn/post/7105678175237046309 "https://juejin.cn/post/7105678175237046309")

[面试官：v-for 中 key 为什么不能用 index，从原理层面聊聊？](https://juejin.cn/post/7106154428771926046 "https://juejin.cn/post/7106154428771926046")

[面试官：vue-router 的 hash 与 history 哪个模式会刷新页面?](https://juejin.cn/post/7106513582413578248 "https://juejin.cn/post/7106513582413578248")

[面试官：说说你平时用过的自适应方案（数字孪生可视化自适应方案）](https://juejin.cn/post/7106898885293015077 "https://juejin.cn/post/7106898885293015077")

[面试官：说一下如何优化过渡动画（数字孪生可视化过渡动画）](https://juejin.cn/post/7107268828786065445 "https://juejin.cn/post/7107268828786065445")

写在最后
====

博主接下来将持续更新好文，欢迎关注博主哟！！  
如果文章对您有帮助麻烦亲**点赞、收藏 + 关注**和博主一起成长哟！！❤️❤️❤️
