### CORS

CORS 是一个 W3C 标准，全称是跨域资源共享（Cross-origin resource sharing），它允许浏览器向跨源服务器，发出 XMLHttpRequest 请求。

整个 CORS 通信过程，都是浏览器自动完成，不需要用户参与。对于开发者来说，CORS 通信与普通的 AJAX 通信没有差别，代码完全一样。浏览器一旦发现 AJAX 请求跨域，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感知。因此，实现 CORS 通信的关键是服务器。只要服务器实现了 CORS 接口，就可以跨域通信。

#### 服务器端配置

CORS 常用的配置项有以下几个：

* **Access-Control-Allow-Origin**（必含） – 允许的域名，只能填 `*`（通配符）或者单域名。

* **Access-Control-Allow-Methods**（必含） – 这允许跨域请求的 http 方法（常见有 `POST、GET、OPTIONS`）。

* **Access-Control-Allow-Headers**（当预请求中包含 `Access-Control-Request-Headers` 时必须包含） – 这是对预请求当中 `Access-Control-Request-Headers` 的回复，和上面一样是以逗号分隔的列表，可以返回所有支持的头部。

* **Access-Control-Allow-Credentials**（可选） – 表示是否允许发送 Cookie，只有一个可选值：true（必为小写）。如果不包含 cookies，请略去该项，而不是填写 false。这一项与 XmlHttpRequest 对象当中的 `withCredentials` 属性应保持一致，即 withCredentials 为 true 时该项也为 true；withCredentials 为 false 时，省略该项不写。反之则导致请求失败。

* **Access-Control-Max-Age**（可选） – 以秒为单位的缓存时间。在有效时间内，浏览器无须为同一请求再次发起预检请求。

#### CORS 跨域的判定流程

1. 浏览器先根据同源策略对前端页面和后台交互地址做匹配，若同源，则直接发送数据请求；若不同源，则发送跨域请求。

2. 服务器收到浏览器跨域请求后，根据自身配置返回对应文件头。若未配置过任何允许跨域，则文件头里不包含 `Access-Control-Allow-origin` 字段，若配置过域名，则返回 `Access-Control-Allow-origin + 对应配置规则里的域名的方式`。

3. 浏览器根据接受到的 响应头里的 `Access-Control-Allow-origin` 字段做匹配，若无该字段，说明不允许跨域，从而抛出一个错误；若有该字段，则对字段内容和当前域名做比对，如果同源，则说明可以跨域，浏览器接受该响应；若不同源，则说明该域名不可跨域，浏览器不接受该响应，并抛出一个错误。

上面说到的两种类型的报错，控制台输出是不一样的：

1. 服务器允许跨域请求，但是 Origin 指定的源，不在许可范围内，服务器会返回一个正常的 HTTP 回应。浏览器发现，这个回应的头信息没有包含 `Access-Control-Allow-Origin` 字段，就知道出错了，从而抛出一个错误，被 XMLHttpRequest 的 onerror 回调函数捕获。注意，这种错误无法通过状态码识别，因为 HTTP 回应的状态码有可能是 200。

```jsx
<!--控制台返回结果-->
 XMLHttpRequest cannot load http://localhost/city.json.
 The 'Access-Control-Allow-Origin' header has a value 'http://segmentfault.com' that is not equal to the supplied origin. 
 Origin 'http://www.zhihu.com' is therefore notallowed access.

```

2. 服务器不允许任何跨域请求

```jsx
<!--控制台返回结果-->
XMLHttpRequest cannot load http://localhost/city.json.
No 'Access-Control-Allow-Origin' header is present on the requested resource. 
Origin 'http://www.zhihu.com' is therefore not allowed access.

```

#### 简单请求

实际上浏览器将 CORS 请求分成两类：简单请求（`simple request`）和非简单请求（`not-so-simple request`）。

简单请求是指满足以下条件的（一般只考虑前面两个条件即可）：

1. 使用 `GET、POST、HEAD` 其中一种请求方法。
2. HTTP 的头信息不超出以下几种字段：
    * Accept
    * Accept-Language
    * Content-Language
    * Last-Event-ID
    * Content-Type：只限于三个值 `application/x-www-form-urlencoded、multipart/form-data、text/plain`
3. 请求中的任意 XMLHttpRequestUpload 对象均没有注册任何事件监听器；
4. XMLHttpRequestUpload 对象可以使用 XMLHttpRequest.upload 属性访问。 请求中没有使用 ReadableStream 对象。

对于简单请求，浏览器直接发起 CORS 请求，具体来说就是服务器端会根据请求头信息中的 `origin` 字段（包括了协议 + 域名 + 端口），来决定是否同意这次请求。

如果 `origin` 指定的源在许可范围内，服务器返回的响应，会多出几个头信息字段：

```jsx
Access-Control-Allow-Origin: http://xxx.xxx.com
Access-Control-Allow-Credentials: true
Access-Control-Expose-Headers: FooBar
Content-Type: text/html; charset=utf-8

```

#### 非简单请求

非简单请求时指那些对服务器有特殊要求的请求，比如请求方法是 `put` 或 `delete`，或者 `content-type` 的类型是 `application/json`。其实简单请求之外的都是非简单请求了。

非简单请求的 CORS 请求，会在正式通信之前，使用 `OPTIONS` 方法发起一个预检（preflight）请求到服务器，浏览器先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可以使用哪些 HTTP 动词和头信息字段。只有得到肯定答复，浏览器才会发出正式的 XMLHttpRequest 请求，否则就报错。

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/4/11/16a0ab6de903b2b0~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

下面是一个预检请求的头部：

```jsx
OPTIONS /cors HTTP/1.1
Origin: http://api.bob.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: X-Custom-Header
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
```

一旦服务器通过了 "预检" 请求，以后每次浏览器正常的 CORS 请求，就都跟简单请求一样了。

关于为什么要有简单请求和非简单请求，可参考知乎上的一个回答 [为什么跨域的 post 请求区分为简单请求和非简单请求和 content-type 相关？](https://link.juejin.cn?target=https%3A%2F%2Fwww.zhihu.com%2Fquestion%2F268998684%2Fanswer%2F344949204 "https://www.zhihu.com/question/268998684/answer/344949204")
