## CDN 缓存
在浏览器本地缓存失效后，浏览器会像CDN边缘节点发起请求，类似浏览器缓存，CDN边缘节点也存在一套缓存机制，

-   CDN边缘节点缓存策略因服务商不同而不同，通过http响应头中的cache-control：max-age字段设置CDN边缘节点数据缓存时间。
-   当浏览器向CDN节点请求数据时，CDN节点会判断缓存数据是否过期，若缓存数据过期，CDN会向服务器发出回源请求，从服务器拉取最新数据，更新本地缓存，并将最新数据返回给客户端，CDN服务商一般会提供基于文件后缀，目录多个维度来指定CDN缓存时间，为用户提供更精细化的缓存管理。

## CDN 工作方式
1. 当点击网站页面的URL 时, 经过本地 DNS 解析, DNS 解析后会把域名的解析权交给 `cname()` 指向的内容分发专用的 DNS 服务器. 内容分发专用的DNS 服务器把内容分发的全局负载均衡设备的IP地址返回给用户
2. 当向 CDN 的全局负载均衡设备的 IP 地址发起 URL 访问请求, CDN 的全局负载均衡设备会为客户端选择合适的缓存服务器提供服务
	1. **选择的依据:** 用户的 IP 地址, 判断哪台服务器距离用户最近, 根据用户请求的 URL 中携带的内容名称判断哪台服务器上有相应数据, 查询各个服务器当前的负载情况, 判断哪台服务器有服务能力
	2. **分配:** 基于以上条件分析后, 区域负载均衡设备会向全局负载均衡设备请求返回一台缓存服务器的 IP 地址. 全局负载均衡设备返回服务器 IP 地址, 用户向目标服务器发起请求, 缓存服务器返回用户所需内容, 如果没有相应内容, 缓存服务器会向更上一级的缓存服务器请求内容, 直至内容可以触达并返回给用户. 

## CDN 优势
- CDN 节点解决了跨运营商和跨地域访问的问题, 访问延时大大降低
- 大部分请求在 CDN 边缘节点完成, CDN 起分流作用, 减轻了服务器的负载

[从输入URL到渲染全过程 - 掘金](https://juejin.cn/post/6844904194801926157#heading-9)
