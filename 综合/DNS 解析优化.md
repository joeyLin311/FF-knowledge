---
date created: 2022-05-30 22:53
---

#Network

## 为什么需要 DNS 解析优化

通常情况下, 主机域名加载一个页面只需要解析一次 DNS, 但是如果页面上的资源文件比如: fonts, images, scripts 等都是位于不同的主机上, DNS 会对每一个资源进行解析. 这个行为对于页面加载性能影响是个问题, 特别是对于移动网络来说. 所以我们可以开启 DNS Prefetch

## DNS Prefetch

DNS Prefetch, 即通过浏览器定义的规则, 提前解析资源对应的域名, 让解析结果缓存在系统缓存中. 这样就缩短了 DNS 的解析时间, 提高网页访问速度.

打开 DNS Prefetch 之后, 浏览器会在空闲时间提前将这些域名解析成对应的 IP 地址. 这个环节为了防止 DNS Prefetch 阻塞正常页面的渲染, Chrome 不会将此行为放在网络堆栈进程里去与解析, 而是单独开辟 8 个完全异步的 Worker 线程专门负责 DNS 预解析. 所以预解析操作在 Chrome 里并不会对首屏加载产生太大的副作用.

## 如何使用 DNS Prefetch

- **自动解析**
  - 使用 `<meta>` 标签对 `href` 中的域名开启预解析操作. 这个行为是与网页渲染并行的, 但是 HTTPS 的页面不会自动解析

```html
<!-- http 页面开启 DNS Prefetch -->
<meta http-equiv="x-dns-prefetch-control" content="on" >

<!-- http 页面关闭 DNS Prefetch -->
<meta http-equiv="x-dns-prefetch-control" content="off" >
```

- **手动解析**

```html
<!-- 手动指定资源的目标IP地址 -->
<link rel="dns-prefetch" href="//xx.xx.xx">
```
