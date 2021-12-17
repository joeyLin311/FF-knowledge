---
date created: 2021-12-09 22:56
date updated: 2021-12-17 16:11
---

#Network

## HTTP3 来源

由于 TCP 和 UDP 两者在运输层存在一定差异，TCP 的传递效率与 UDP 相比有天然劣势，于是 Google 基于 UDP 开发出了新的协议 QUIC(Quick UDP Internet Connections)，希望取代 TCP 提高传输效率，后经过协商将 QUIC 协议更名为 HTTP/3。

## QUIC 概述

TCP、UDP 是我们所熟悉的传输层协议，UDP 比 TCP 相比效率更高但并不具备传输可靠性。而 QUIC 便是看中 UDP 传输效率这一特性，并结合了 TCP、TLS、HTTP/2 的优势，加以优化。

于是在 QUIC 上层的应用层所运行的 HTTP 协议也就被称为 HTTP/3。

## HTTP3 新特性

### 1. 0RTT 建立连接

![[http3-0RTT.png]]
保证 0RTT 的核心是 DH 密钥交换算法
**Step1**：首次连接时，客户端发送 Inchoate Client Hello 给服务端，用于请求连接；
**Step2**：服务端生成 g、p、a，根据 g、p 和 a 算出 A，然后将 g、p、A 放到 Server Config 中再发送 Rejection 消息给客户端；
**Step3**：客户端接收到 g、p、A 后，自己再生成 b，根据 g、p、b 算出 B，根据 A、p、b 算出初始密钥 K。B 和 K 算好后，客户端会用 K 加密 HTTP 数据，连同 B 一起发送给服务端；
**Step4**：服务端接收到 B 后，根据 a、p、B 生成与客户端同样的密钥，再用这密钥解密收到的 HTTP 数据。为了进一步的安全（前向安全性），服务端会更新自己的随机数 a 和公钥，再生成新的密钥 S，然后把公钥通过 Server Hello 发送给客户端。连同 Server Hello 消息，还有 HTTP 返回数据；
**Step5**：客户端收到 Server Hello 后，生成与服务端一致的新密钥 S，后面的传输都使用 S 加密。
这样，QUIC 从请求连接到正式接发 HTTP 数据一共花了 1 RTT，这 1 个 RTT 主要是为了获取 Server Config，后面的连接如果客户端缓存了 Server Config，那么就可以直接发送 HTTP 数据，实现 0 RTT 建立连接。

### 2. 连接迁移

- 传统连接通过源 IP、源端口、目的 IP、目的端口进行连接，当网络发生更换后连接再次建立时延较长。
- HTTP/3 使用 Connection ID 对连接保持，只要 Connection ID 不改变，连接仍可维持。

### 3. 队头阻塞/多路复用

- TCP 作为面向连接的协议，对每次请求序等到 ACK 才可继续连接，一旦中间连接丢失将会产生队头阻塞。
- HTTP/1.1 中提出 Pipelining 的方式，单个 TCP 连接可多次发送请求，但依旧会有中间请求丢失产生阻塞的问题。
- HTTP/2 中将请求粒度减小，通过 Frame 的方式进行请求的发送。但在 TCP 层 Frame 组合得到 Stream 进行传输，一旦出现 Stream 中的 Frame 丢失，其后方的 Stream 都将会被阻塞。
- 对于 HTTP/2 而言，浏览器会默认采取 TLS 方式传输，TLS 基于 Record 组织数据，每个 Record 包含 16K，其中有 12 个 TCP 的包，一旦其中一个 TCP 包出现问题将会导致整个 Record 无法解密。这也是网络环境较差时 HTTP/2 的传输速度比 HTTP/1.1 更慢的原因。
- HTTP/3 基于 UDP 的传输，不保证连接可靠性，也就没有对头阻塞的后果。同样传输单元与加密单元为 Packet，在 TLS 下也可避免对头阻塞的问题。

### 4. 拥塞控制

- 热拔插：TCP 对于拥塞控制在于传输层，QUIC 可在应用层操作改变拥塞控制方法。
- 前向纠错(FEC)：将数据切割成包后可对每个包进行异或运算，将运算结果随数据发送。一旦丢失数据可据此推算。(带宽换时间)
- 单调递增的 Packet Number：TCP 在超时重传后的两次 ACK 接受情况并不支持的很好。导致 RTT 和 RTO 的计算有所偏差。HTTP/3 对此进行改进，一旦重传后的 Packet N 会递增。

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/6/7/1728dbde7e87843c~tplv-t2oaga2asx-watermark.awebp)

- ACK Delay

HTTP/3 在计算 RTT 时健壮的考虑了服务端的 ACK 处理时延。
![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/6/7/1728dbe831911ad6~tplv-t2oaga2asx-watermark.awebp)

- 更多地 ACK 块

一般每次请求都会对应一个 ACK，但这样也会浪费(下载场景只需返回数据即可)。
于是可设计成每次返回 3 个 ACK block。在 HTTP/3 将其扩充成最多可携带 256 个 ACK block。

### 5. 流量控制

- TCP 使用滑动窗口的方式对发送方的流量进行控制。而对接收方并无限制。在 QUIC 中便补齐了这一短板。
- QUIC 中接收方从单挑 Stream 和整条连接两个角度动态调整接受的窗口大小。

<https://juejin.cn/post/6855470356657307662#heading-18>
