---
date created: 2021-12-09 22:54
---

#Network

# SYN cookie

SYN Cookies 的应用允许服务器当 SYN 队列被填满时避免丢弃连接。相反，服务器会表现得像 SYN 队列扩大了一样。服务器会返回适当的 SYN+ACK 响应，但会丢弃 SYN 队列条目。如果服务器接收到客户端随后的 ACK 响应，服务器能够使用编码在 TCP 序号内的信息重构 SYN 队列条目。
