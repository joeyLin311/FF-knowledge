#browser 
## 什么是ajax? ajax的步骤？

```
ajax(异步javascript xml) 能够刷新局部网页数据而不是重新加载整个网页。
如何使用ajax?
第一步，创建xmlhttprequest对象，var xmlhttp =new XMLHttpRequest（);XMLHttpRequest对象用来和服务器交换数据。
var xhttp;
if (window.XMLHttpRequest) {
//现代主流浏览器
xhttp = new XMLHttpRequest();
} else {
// 针对浏览器，比如IE5或IE6
xhttp = new ActiveXObject("Microsoft.XMLHTTP");
}
第二步，使用xmlhttprequest对象的open()和send()方法发送资源请求给服务器。
第三步，使用xmlhttprequest对象的responseText或responseXML属性获得服务器的响应。
第四步，onreadystatechange函数，当发送请求到服务器，我们想要服务器响应执行一些功能就需要使用onreadystatechange函数，每次xmlhttprequest对象的readyState发生改变都会触发onreadystatechange函数。
```