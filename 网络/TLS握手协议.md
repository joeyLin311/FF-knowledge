---
date created: 2022-05-30 22:52
---

#Network

1.  客户端发出一个 `client hello` 消息，携带的信息包括：所支持的 SSL/TLS 版本列表；支持的与加密算法；所支持的数据压缩方法；随机数 A。
2.  服务端响应一个 server hello 消息，携带的信息包括：协商采用的 SSL/TLS 版本号；会话 ID；随机数 B；服务端数字证书 serverCA；由于双向认证需求，服务端需要对客户端进行认证，会同时发送一个` client certificate request`，表示请求客户端的证书。
3.  客户端校验服务端的数字证书；校验通过之后发送随机数 C，该随机数称为 `pre-master-key`，使用数字证书中的公钥加密后发出；由于服务端发起了 `client certificate request`，客户端使用私钥加密一个随机数 `clientRandom` 随客户端的证书 clientCA 一并发出。
4.  服务端校验客户端的证书，并成功将客户端加密的随机数 `clientRandom` 解密；根据随机数 A/随机数 B/随机数 C（`pre-master-key`） 产生动态密钥 `master-key`，加密一个 finish 消息发至客户端。
5.  客户端根据同样的随机数和算法生成 `master-key`，加密一个 finish 消息发送至服务端。
6.  服务端和客户端分别解密成功，至此握手完成，之后的数据包均采用` master-key` 进行加密传输。
