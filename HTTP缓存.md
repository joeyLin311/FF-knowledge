#Network
[[浏览器缓存]]
[[缓存运用]]

### HTTP缓存流程
![[HTTP缓存流程.png]]

#### `HTTP`缓存机制要点如下：

1.  `HTTP`缓存机制分为**强制缓存**和**协商缓存**两类。
2.  **强制缓存**的意思就是不要问了(不发起请求)，直接用缓存吧。
3.  **强制缓存**常见技术有`Expires`和`Cache-Control`。
4.  `Expires`的值是一个时间，表示这个时间前缓存都有效，都不需要发起请求。
5.  `Cache-Control`有很多属性值，常用属性`max-age`设置了缓存有效的时间长度，单位为`秒`，这个时间没到，都不用发起请求。
6.  `immutable`也是`Cache-Control`的一个属性，表示这个资源这辈子都不用再请求了，但是他兼容性不好，`Cache-Control`其他属性可以参考[MDN的文档](https://link.segmentfault.com/?url=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FHTTP%2FHeaders%2FCache-Control)。
7.  `Cache-Control`的`max-age`优先级比`Expires`高。
8.  **协商缓存**常见技术有`ETag`和`Last-Modified`。
9.  `ETag`其实就是给资源算一个`hash`值或者版本号，对应的常用`request header`为`If-None-Match`。
10.  `Last-Modified`其实就是加上资源修改的时间，对应的常用`request header`为`If-Modified-Since`，精度为`秒`。
11.  `ETag`每次修改都会改变，而`Last-Modified`的精度只到`秒`，所以`ETag`更准确，优先级更高，但是需要计算，所以服务端开销更大。
12.  **强制缓存**和**协商缓存**都存在的情况下，先判断**强制缓存**是否生效，如果生效，不用发起请求，直接用缓存。如果**强制缓存**不生效再发起请求判断**协商缓存**。