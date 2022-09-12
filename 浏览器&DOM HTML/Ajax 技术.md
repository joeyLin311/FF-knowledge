---
date created: 2021-12-09 22:57
date updated: 2021-12-17 16:02
---
#browser 

## 什么是ajax? ajax的步骤？

AJAX是异步的JavaScript和XML（**A**synchronous **J**avaScript **A**nd **X**ML）。简单点说，就是使用 [XMLHttpRequest](https://developer.mozilla.org/en-US/DOM/XMLHttpRequest) 对象与服务器通信。 它可以使用JSON，XML，HTML和text文本等格式发送和接收数据。AJAX最吸引人的就是它的“异步”特性，也就是说它可以在不重新刷新页面的情况下与服务器通信，交换数据，或更新页面。

## 如何使用ajax?

1. 创建`xmlhttprequest` 对象，`var xmlhttp =new XMLHttpRequest（)`;  `XMLHttpRequest` 对象用来和服务器交换数据。

```jsx
var xhttp;
if (window.XMLHttpRequest) {
  //现代主流浏览器
  xhttp = new XMLHttpRequest();
} else {
  // 针对浏览器，比如IE5或IE6
  xhttp = new ActiveXObject("Microsoft.XMLHTTP");
}
```

2. 使用xmlhttprequest对象的open()和send()方法发送资源请求给服务器。
3. 使用xmlhttprequest对象的responseText或responseXML属性获得服务器的响应。
4. onreadystatechange函数，当发送请求到服务器，我们想要服务器响应执行一些功能就需要使用onreadystatechange函数，每次xmlhttprequest对象的readyState发生改变都会触发onreadystatechange函数。
