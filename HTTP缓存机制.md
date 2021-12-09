---
date created: 2021-12-09 22:56
---

#Network

[[浏览器缓存]]
[[缓存运用]]

## HTTP 缓存流程

![[HTTP 缓存流程.png]]

## `HTTP` 缓存机制要点如下

`HTTP` 缓存机制分为**强制缓存**和**协商缓存**两类.

### 强制缓存

**强制缓存**的意思就是不要问了(不发起请求), 直接用缓存吧.
**强制缓存**常见技术有通过 `Expires` 和 `Cache-Control` 控制,命中强缓存时不会发起网络请求, 资源直接从本地获取, 浏览器显示状态码 200 from cache.

**Expires **的值是一个时间

- 缺点: 使用本地时间判断是否过期,而本地时间是可修改的且并非一定准确的
- 表示这个时间前缓存都有效,都不需要发起请求.
- 是 HTTP/1.0 版本提供的
- 优先级低于 `Cache-Control: max-age`

**Cache-Control**

- **max-age** 设置缓存存储的最大时长,单位秒.
- **s-max-age** 与 **max-age** 用法一致,不过仅适用于代理服务器.
- **public** 表示响应可被任何对象缓存.
- **private** 表示响应只可被私有用户缓存,不能被代理服务器缓存.
- **no-cache** 强制客户端向服务器发起请求(禁用强缓存,可用协商缓存).
- **no-store** 禁止一切缓存,包含协商缓存也不可用.
- **must-revalidate** 一旦资源过期,在成功向原始服务器验证之前,缓存不能用该资源响应后续请求.
- **immutable** 表示响应正文不会随时间改变(只要资源不过期就不发送请求)

值得注意的是,虽然以上常用字段都是响应头的字段,但是 Cache-Control 同时也支持请求头,例如 `Cache-Control: max-stale=<seconds>` 表明客户端愿意接收一个已经过期但不能超出 `<seconds>` 秒的资源.
其他属性可以参考 [MDN 的文档](https://link.segmentfault.com/?url=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FHTTP%2FHeaders%2FCache-Control) .

#### 拓展

- HTTP/1.0 Pragma
  - 在 HTTP/1.0 时期用于禁用浏览器缓存 Pragma: no-cache.
- 缓存位置
  - 从 Service Worker 中读取缓存（只支持 HTTPS）.
  - 从内存读取缓存时 network 显示 memory cache.
  - 从硬盘读取缓存时 network 显示 disk cache.
  - Push Cache（推送缓存）（HTTP/2.0）.
  - 优先级 Service Worker > memory cache > disk cache > Push Cache.
- 最佳实践：资源尽可能命中强缓存,且在资源文件更新时保证用户使用到最新的资源文件
  - 强缓存只会命中相同命名的资源文件.
  - 在资源文件上加 hash 标识（webpack 可在打包时在文件名上带上）.
  - 通过更新资源文件名来强制更新命中强缓存的资源.

### 协商缓存

- **协商缓存**常见技术有 `ETag` 和 `Last-Modified`.
- `ETag` 其实就是给资源算一个 `hash` 值或者版本号, 对应的常用 `request header` 为 `If-None-Match`.[[Etag]]
- `Last-Modified` 其实就是加上资源修改的时间, 对应的常用 `request header` 为 `If-Modified-Since`, 精度为 ` 秒 `.
- `ETag` 每次修改都会改变, 而 `Last-Modified` 的精度只到 ` 秒 `, 所以 `ETag` 更准确, 优先级更高, 但是需要计算, 所以服务端开销更大.
  - 优先级高于 Last-Modified / If-Modified-Since
  - 如果资源请求的响应头里含有 ETag,客户端可以在后续的请求的头中带上 If-None-Match 头来验证缓存.若服务器判断资源标识一致,则返回 304 状态码告知浏览器可从本地读取缓存.
  - 唯一标识内容是由服务端生成算法决定的,可以是资源内容生成的哈希值,也可以是最后修改时间戳的哈希值.所以 Etag 标识改变并不代表资源文件改变,反之亦然

**强制缓存**和**协商缓存**都存在的情况下, 先判断**强制缓存**是否生效, 如果生效, 不用发起请求, 直接用缓存.如果**强制缓存**不生效再发起请求判断**协商缓存**.

### 优缺点

**优点**

- 节省了不必要的数据传输，节省带宽.
- 减少服务端的负担，提高网站性能.
- 降低网络延迟，加快页面响应速度，增强用户体验.

**缺点**

- 不恰当的缓存设置可能会导致资源更新不及时，导致用户获取信息滞后.
