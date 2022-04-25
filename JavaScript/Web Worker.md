---
date created: 2021-12-09 22:53
date updated: 2021-12-17 16:25
---

#JavaScript

# Web Worker

JavaScript 语言采用单线程模型, 所有的任务只能在一个线程上完成, 一次只能做一件事. 这种情况制约了发挥现代计算机的计算能力. 所以 Web Worker 被提出来.

**Web Worker 的作用就是为 JavaScript 创造多线程环境,** 允许主线程创建 Worker 线程, Web 引用程序可以在独立于主线程的后台线程中, 运行一个脚本操作. Worker 线程与主线程互不干扰, Worker 中完成任务后的结果, 可以将结果使用 `postMessage()` 返回给主线程.

Web Worker 有点在于可以在一个单独的线程中执行一些耗时较长的任务, 从而允许主线程中的其它任务运行不被该耗时任务阻塞到.

## Web Worker 特点

1. **在 Worker 内部无法访问主线程的任何资源**, 包括全局变量, DOM 对象 或者其它资源, 因为这是一个完全独立的线程.
2. **Worker 和主线程间的数据传递通过消息机制进行**. 使用 `poseMessage()` 方法发送消息; 使用 `onMessage()` 事件处理函数来响应消息.
3. **Worker 可以创建新的 Worker, 新的 Worker 和父页面同源.** Worker 在使用 `XMLHttpRequest` 进行网络 I/O 时, `XMLHttpRequest` 的 `responseXML` 和 `channel` 属性会返回 null.
4. **Worker 线程无法读取本地文件**, 即不能打开本机的文件系统 `file://` , 它所加载的脚本必须来自网络

## Web Worker 应用场景

- 处理密集型数学计算任务
- 大数据序列化操作
- 媒体数据处理: 文件压缩, 音频分析, 图像处理
- 高流量网络通信
