---
date created: 2021-12-09 22:57
date updated: 2021-12-17 16:06
---

#Network

## Nginx 的 Etag

Nginx 官方默认的 ETag 计算方式是为"文件最后修改时间 16 进制-文件长度 16 进制"。例：ETag： “59e72c84-2404”

## Express 框架

Express 框架使用了 serve-static 中间件来配置缓存方案，其中，使用了一个叫 [etag](https://links.jianshu.com/go?to=https%3A%2F%2Flink.juejin.cn%2F%3Ftarget%3Dhttps%253A%252F%252Fgithub.com%252Fjshttp%252Fetag) 的npm包来实现etag计算。从其源码可以看出，有两种计算方式：

- 方式一：使用文件大小和修改时间

```javascript
function stattag (stat) {
  var mtime = stat.mtime.getTime().toString(16)
  var size = stat.size.toString(16)

  return '"' + size + '-' + mtime + '"'
}
```

- 方式二：使用文件内容的 hash 值和内容长度

```javascript
function entitytag (entity) {
  if (entity.length === 0) {
    // fast-path empty
    return '"0-2jmj7l5rSw0yVb/vlWAYkK/YBwk"'
  }

  // compute hash of entity
  var hash = crypto
    .createHash('sha1')
    .update(entity, 'utf8')
    .digest('base64')
    .substring(0, 27)

  // compute length of entity
  var len = typeof entity === 'string'
    ? Buffer.byteLength(entity, 'utf8')
    : entity.length

  return '"' + len.toString(16) + '-' + hash + '"'
}
```

## 为什么要有 etag？

你可能会觉得使用 last-modified 已经足以让浏览器知道本地的缓存副本是否足够新，为什么还需要 etag 呢？HTTP1.1 中 etag 的出现（也就是说，etag 是新增的，为了解决之前只有 If-Modified 的缺点）主要是为了解决几个 last-modified 比较难解决的问题：

1. 一些文件也许会周期性的更改，但是他的内容并不改变(仅仅改变的修改时间)，这个时候我们并不希望客户端认为这个文件被修改了，而重新 get；
2. 某些文件修改非常频繁，比如在秒以下的时间内进行修改，(比方说 1s 内修改了 N 次)，if-modified-since 能检查到的粒度是秒级的，这种修改无法判断(或者说 UNIX 记录 MTIME 只能精确到秒)；
3. 某些服务器不能精确的得到文件的最后修改时间。

- ETag 在标识前面加 `W/` 前缀表示用弱比较算法（If-None-Match 本身就只用弱比较算法）。
- ETag 还可以配合 If-Match 检测当前请求是否为最新版本，若资源不匹配返回状态码 412 错误。（If-Match 不加 `W/` 时使用强比较算法）。
