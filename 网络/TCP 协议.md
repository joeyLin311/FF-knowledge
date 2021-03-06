---
date created: 2021-12-09 22:54
date updated: 2021-12-17 16:18
---

#Network

## TCP 概念

TCP( transmission control protocol)协议是一种**面向连接的, 可靠的, 基于字节流**的传输层通信协议, 在简化的计算机网络 OSI 模型中属于第四层传输层 # 传输层指定的功能. `OSI` 模型

- TCP 协议中仅有两方进行通信, 广播和多播不能用于 TCP
- TCP 使用校验, 确认和重传机制来保证通信的可靠
- TCP 给数据分节排序, 并使用积累确认保证数据的顺序不变和非重复
- TCP 使用**滑动窗口**机制来实现流量控制, 通过动态改变**滑动窗口**大小进行拥塞控制(?)

## 三次握手

指建立一个 TCP 连接过程时, 需要客户端和服务端总共发送 3 个数据包. 握手的主要作用是为了确认双方的接收和发送能力是否正常, 初始化序列号, 确定**滑动窗口**大小等信息.

SYN 队列: 存放完成了二次握手的结果, 队列长度由 listen 函数的参数 backlog 指定

ACCEPT 队列: 存放完成了三次握手的结果, 队列长度由 listen 函数的参数 backlog 指定

1. 一次握手(SYN = 1, seq = 1): 客户端向服务端发送一个 SYN 标志位置 1 的包, 指明客户端连接服务端的端口以及包含一个随机数 x 作为消息序列号, 即是 sequence number
2. 发送完毕后, 客户端进入 `SYN_SEND` 状态
3. 二次握手(SYN = 1, ACK = 1, seq = y, ACKnum = x + 1): 服务端确认 `SYN` 数据包并放入 SYN 队列中, 发回 ACK 确认包应答, 同时发送 `SYN` 包, ACK 确认码 x+1 , ACK/SYN 包产生 一个随机数 y. 发送完毕后服务端进入 `SYN_RCVD` 状态
4. 三次握手(SYN/ACK = 1, ACK = x + 1, ACKnum = y+1): 客户端收到 ACK/SYN 包后, 再次发送一个 ACK 包, 该包(ACK)的序号设定为 x + 1 , ACK 确认码为 y + 1. 发送完毕后, 客户端进入 `ESTABLEISHED` 状态, 服务器接收到这个包时也进入 `ESTABLEISHED` 状态, 至此三次握手结束

三次握手的目的是在不可靠的信道上创建可信的通信链接, 需要双方通知对象自己的分组的序列号的开始值

#### 为什么是三次二次握手可以吗?

1. 确认双方的收发能力
   1. TCP 建立通信三次握手的目的是在不可靠的信道上创建可信的通信链接, 需要双方通知对象自己的分组的序列号的开始值
   2. 确认双方的发送能力, 接收能力是正常的, 第一次确认客户端发送, 服务端接收, 第二次确认服务端接收/发送, 客户端接收/发送, 第三次数据包发送, 服务端收到, 才确认了双方的收发能力.
   3. TCP 使用序列号和确认号保证可靠传输, 两次握手后如果停止, 客户端无法确认服务端的确认号, 就无法保证 TCP 的可靠性
   4. TCP 有个机制是基于重复累计确认的重传: 每个数据包都有数据号和确认号, 如果一个数据包没有收到 ACK 号码就不会发生变化, 如果发送方收到 3 次以上的同一个包, 就会重传最后一个未被确认的包. 如果是两次握手, 只有发送方的 ACKnum 可以得到确认, 另一方的 ACKnum 无法得到确认, 可靠性就无法建立
2. **利用两次握手进行 SYN 攻击**
   1. 在第二次握手服务器发送 SYN/ACK 包后, 客户端还没发送 ACK 包前, 这个时候称为半连接(half-open connect). 此时服务器处于 `SYN_RCVD` 状态 . 这个时候会长生一个漏洞, 攻击端会不断发送 SYN 包给服务器, 服务器又继续回复确认包, 攻击端又是伪造的 IP 地址, TCP 建立不起来, 服务器的资源就会一直等待这些回传信息, 造成运行缓慢. 这是一种典型的 DoS/DDoS 攻击
   2. 怎么防范? 服务器端检测是否存在大量半连接的 TCP 状态且 IP 地址是随机的就可以断定是攻击
   3. 无法被完全阻止
   4. 可以缩短 TCP 超时的时间 SYN timeout
   5. 增加最大半连接数
   6. 网关保护
   7. SYN cookie(?) [[SYN cookie]]

## 四次挥手

TCP 通信的连接终止的过程就是四次挥手, 该过程中连接的每一侧都独立地被终止, 也就是客户端或服务端都可以主动发起终止连接的动作.

1. 一次挥手(FIN = 1, seq = 1): 假设由客户端发起终止连接动作, 客户端发起一个 FIN 标志为 1 的包, 表示自己没有数据可以发送, 但是仍然可以接收数据, 发送后客户端进入 `FIN_WAIT_1` 状态.

2. 二次挥手(ACK = 1, ACKnum = 1): 服务端收到客户端 FIN 包, 发送一个 ACK 包确认接受到终止连接的请求, 此时还没准备好关闭连接, 服务端进入 `CLOSE_WAIT` 状态, 客户端接收到 ACK 包后, 进入 `FIN_WAIT_2` 状态.

3. 三次挥手(FIN = 1, seq = y): 服务端准备准备好关闭连接时, 向客户端发送终止连接包, FIN 设置为 1. 发送完毕后, 服务端进入 `LAST_ACK` 状态, 等到客户端的最后一个 ACK 包

4. 四次挥手(ACK = 1, ACKnum = y + 1):

5. 客户端接收服务端的终止请求, 发送一个确认包, 进入 `TIME_WAIT` 状态, 等待可能出现的要求重传的 ACK 包.

6. 服务端接收客户端的确认包, 关闭连接, 进入 `CLOSED` 状态.

7. 客户端会等待一个固定时间, 2*MSL(Linux 是 30s), 没有收到服务端的 ACK 判定服务端已经正确关闭连接, 自己也进入 `CLOSED` 状态

#### 为什么中断连接要四次挥手? 三次不可以吗?

其实在 TCP 握手的时候，接收端发送 SYN/ACK 的包是将一个 ACK 和一个 SYN 合并到一个包中，所以减少了一次包的发送，三次完成握手。

对于四次挥手，因为 TCP 是全双工通信，在主动关闭方发送 FIN 包后，接收端可能还要发送数据，不能立即关闭服务器端到客户端的数据通道，所以也就不能将服务器端的 FIN 包与对客户端的 ACK 包合并发送，只能先确认 ACK，然后服务器待无需发送数据时再发送 FIN 包，所以四次挥手时必须是四次数据包的交互。

#### 为什么 TIME_WAIT 状态需要经过 2MSL 才能返回到 CLOSE 状态？

`MSL` 指的是报文在网络中最大生存时间。在客户端发送对服务器端的 FIN 的确认包 `ACK` 后，这个 `ACK` 包是有可能不可达的，服务器端如果收不到 `ACK` 的话需要重新发送 `FIN` 包。

所以客户端发送 `ACK` 后需要留出 2MSL 时间（ACK 到达服务器 + 服务器发送 FIN 重传包，一来一回）等待确认服务器端确实收到了 ACK 包。

也就是说客户端如果等待 2MSL 时间也没有收到服务器端的重传包 FIN，说明可以确认服务器已经收到客户端发送的 ACK。

还有第 2 个理由，避免新旧连接混淆。

在客户端发送完最后一个 ACK 报文段后，在经过 2MSL 时间，就可以使本连接持续的时间内所产生的所有报文都从网络中消失，使下一个新的连接中不会出现这种旧的连接请求报文。

你要知道，有些自作主张的路由器会缓存 IP 数据包，如果连接重用了，那么这些延迟收到的包就有可能会跟新连接混在一起。

## 总结 TCP 特性

- 乱序, 使用序号和确认号进行可靠传输
- 基于重复累计确认的重传, 后来运用**选择性确认,** **允许接收方确认它成功收到的分组的不连续的块**
- 超时重传,也就是丢包
- 流量控制
- 拥塞控制

## TCP KeepAlive

TCP 的连接，实际上是一种纯软件层面的概念，在物理层面并没有“连接”这种概念。TCP 通信双方建立交互的连接，但是并不是一直存在数据交互，有些连接会在数据交互完毕后，主动释放连接，而有些不会。在长时间无数据交互的时间段内，交互双方都有可能出现掉电、死机、异常重启等各种意外，当这些意外发生之后，这些 TCP 连接并未来得及正常释放，在软件层面上，连接的另一方并不知道对端的情况，它会一直维护这个连接，长时间的积累会导致非常多的半打开连接，造成端系统资源的消耗和浪费，为了解决这个问题，在传输层可以利用 TCP 的 KeepAlive 机制实现来实现。主流的操作系统基本都在内核里支持了这个特性。

TCP KeepAlive 的基本原理是，隔一段时间给连接对端发送一个探测包，如果收到对方回应的 ACK，则认为连接还是存活的，在超过一定重试次数之后还是没有收到对方的回应，则丢弃该 TCP 连接。

[TCP-Keepalive-HOWTO](http://www.tldp.org/HOWTO/html_single/TCP-Keepalive-HOWTO/) 有对 TCP KeepAlive 特性的详细介绍，有兴趣的同学可以参考。这里主要说一下，TCP KeepAlive 的局限。首先 TCP KeepAlive 监测的方式是发送一个 probe 包，会给网络带来额外的流量，另外 TCP KeepAlive 只能在内核层级监测连接的存活与否，而连接的存活不一定代表服务的可用。例如当一个服务器 CPU 进程服务器占用达到 100%，已经卡死不能响应请求了，此时 TCP KeepAlive 依然会认为连接是存活的。因此 TCP KeepAlive 对于应用层程序的价值是相对较小的。需要做连接保活的应用层程序，例如 QQ，往往会在应用层实现自己的心跳功能。
