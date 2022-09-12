---
date created: 2022-07-14 23:23
---

## postMessage 的定义

postMessage 是 html5 引入的 API,postMessage() 方法允许来自不同源的脚本采用异步方式进行有效的通信, 可以实现跨文本文档, 多窗口, 跨域消息传递. 多用于窗口间数据通信, 这也使它成为跨域通信的一种有效的解决方案.

## postMessage 的兼容性

下图是在 caniuse 上面搜到的 postMessage 兼容性截图, 除 IE 浏览器的支持度比较低外,, 其他浏览器的支持度良好.

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9a6d0c949e3a4afa81ab9f15a491d65c~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

## postMessage API 介绍

### 发送数据

```js
otherWindow.postMessage(message, targetOrigin, [transfer]);
```

### otherWindow

窗口的一个引用, 比如 iframe 的 contentWindow 属性, 执行 window.open 返回的窗口对象, 或者是命名过的或数值索引的 window.frames.

### message

要发送到其他窗口的数据, 它将会被 [结构化克隆算法]([developer.mozilla.org/en-US/docs/…](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FDOM%2FThe%255C_structured%255C_clone%255C_algorithm "https://developer.mozilla.org/en-US/docs/DOM/The%5C_structured%5C_clone%5C_algorithm")) 序列化. 这意味着你可以不受什么限制的将数据对象安全的传送给目标窗口而无需自己序列化.

### targetOrigin

通过窗口的 origin 属性来指定哪些窗口能接收到消息事件, 指定后只有对应 origin 下的窗口才可以接收到消息, 设置为通配符 "*" 表示可以发送到任何窗口, 但通常处于安全性考虑不建议这么做. 如果想要发送到与当前窗口同源的窗口, 可设置为 "/"

### transfer | 可选属性

是一串和 message 同时传递的 **Transferable** 对象, 这些对象的所有权将被转移给消息的接收方, 而发送一方将不再保有所有权.

### 接收数据: 监听 message 事件的发生

```js
window.addEventListener("message", receiveMessage, false) ;
function receiveMessage(event) {
     var origin= event.origin;
     console.log(event);
}
```

event 对象的打印结果截图如下:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c58d14cb21ce481cb3f348e7fe55abb7~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

这里重点介绍 event 对象的四个属性

- **data :**   指的是从其他窗口发送过来的消息对象;

- **type:**   指的是发送消息的类型;

- **source:**   指的是发送消息的窗口对象;

- **origin:** 指的是发送消息的窗口的源

## postMessage 的使用场景

### 场景一 跨域通信 (包括 GET 请求和 POST 请求)

>  我们都知道 JSONP 可以实现解决 GET 请求的跨域问题, 但是不能解决 POST 请求的跨域问题. 而 postMessage 都可以. 这里只是列举一个示例, 仅供参考, 具体的代码如何编写要以具体的场景而定奥~

**父窗体创建跨域 iframe 并发送信息**

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <title>跨域POST消息发送</title>
        <script type="text/JavaScript">    
            // sendPost 通过postMessage实现跨域通信将表单信息发送到 moweide.gitcafe.io上,
            // 并取得返回的数据    
            function sendPost() {        
                // 获取id为otherPage的iframe窗口对象        
                var iframeWin = document.getElementById("otherPage").contentWindow;        
                // 向该窗口发送消息        
                iframeWin.postMessage(document.getElementById("message").value, 
                    'http://moweide.gitcafe.io');    
            }    
            // 监听跨域请求的返回    
            window.addEventListener("message", function(event) {        
                console.log(event, event.data);    
            }, false);
        </script>
    </head>
    <body> 
        <textarea id="message"></textarea> 
        <input type="button" value="发送" onclick="sendPost()"> 
        <iframe
            src="http://moweide.gitcafe.io/other-domain.html" id="otherPage"
            style="display:none"></iframe>
    </body>

</html>
```

**子窗体接收信息并处理**

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <title>POST Handler</title>
        <script src="//code.jquery.com/jquery-1.11.0.min.js"></script>
        <script type="text/JavaScript">
            window.addEventListener("message", function( event ) {
                // 监听父窗口发送过来的数据向服务器发送post请求
                var data = event.data;
                $.ajax({
                    // 注意这里的url只是一个示例.实际练习的时候你需要自己想办法提供一个后台接口
                    type: 'POST', 
                    url: 'http://moweide.gitcafe.io/getData',
                    data: "info=" + data,
                    dataType: "json"
                }).done(function(res){        
                    //将请求成功返回的数据通过postMessage发送给父窗口        
                    window.parent.postMessage(res, "*");    
                }).fail(function(res){        
                    //将请求失败返回的数据通过postMessage发送给父窗口        
                    window.parent.postMessage(res, "*");    
                });
            }, false);
        </script>
    </head>

    <body></body>
</html>
```

### 场景二  WebWorker

JavaScript 语言采用的是单线程模型, 通常来说, 所有任务都在一个线程上完成, 一次只能做一件事, 后面的任务要等到前面的任务被执行完成后才可以开始执行, 但是这种方法如果遇到复杂费时的计算, 就会导致发生阻塞, 严重阻碍应用程序的正常运行. Web Worker 为 web 内容在后台线程中运行脚本提供了一种简单的方法, 线程可以执行任务而不干扰用户界面. 一旦创建, 一个 worker 可以将消息发送到创建它的 JavaScript 代码, 通过消息发布到改代码指定的事件处理程序.

一个 woker 是使用一个构造函数创建一个对象, 运行一个命名的 JavaScript 文件 - 这个文件将包含在工作线程中运行的代码, woker 运行在另一个全局上下文中, 不同于当前的 window, 不能使用 window 来获取全局属性.

**一些局限性**

- 只能加载同源脚本文件, 不能直接操作 DOM 节点

- Worker 线程不能执行 `alert()` 方法和 `confirm()` 方法，但可以使用 XMLHttpRequest 对象发出 AJAX 请求

- 无法读取本地文件, 只能加载网络文件

- 也不能使用 window 对象的默认方法和属性, 然而你可以使用大量 window 对象之下的东西, 包括 webSocket,indexedDB 以及 FireFoxOS 专用的 DataStore API 等数据存储机制. 查看 [Functions and classes available to workers](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FWeb%2FAPI%2FWorker%2FFunctions_and_classes_available_to_workers "https://developer.mozilla.org/en-US/docs/Web/API/Worker/Functions_and_classes_available_to_workers") 获取详情。

workers 和主线程间的数据传递通过这样的消息机制进行——双方都使用 postMessage() 方法发送各自的消息，使用 onmessage 事件处理函数来响应消息（消息被包含在 `[Message](https://developer.mozilla.org/zh-CN/docs/Web/Reference/Events/Message "/zh-CN/docs/Web/Reference/Events/Message")`事件的 data 属性中）。这个过程中数据并不是被共享而是被复制; woker 分为专用 worker 和共享 worker, 一个专用 worker 仅仅能被首次生成它的脚本使用, 而共享 woker 可以同时被多个脚本使用.

专用 woker 使用示例:

```js
// main.js
if(window.Worker) {
    var myWorker = new Worker('http://xxx.com/worker.js');
    // 发送消息
    first.onchange = function() {
        myWorker.postMessage([first.value, second.value]);
        console.log("Message posted to worker");
    }
    second.onchange = function() {
      myWorker.postMessage([first.value,second.value]);
      console.log('Message posted to worker');
    }
    // 主线程 监听onmessage以响应worker回传的消息
    myWorker.onmessage = function (e) {
      var textContent = e.data;
      console.log("message received from worker");  
    }
}

// worker.js

// 内置selfduixiang,,代表子线程本身, worker内部要加载其他脚本,可通过importScripts()方法
onmessage = function(e) {
    console.log("message received from main script");
    var workerResult = "Result: " + (e.data[0] * e.data[1]);
    console.log("posting message\back to main script");
    postMessage(workerResult);
}
```

Web Worker 的使用场景, 用于收集埋点数据, 可以用于大量复杂的数据计算, 复杂的图像处理, 大数据的处理. 因为它不会阻碍主线程的正常执行和页面 UI 的渲染.

埋点数据采集下的使用: 可在 main.js 中收集数据, 将收集到的信息通过 postMessage 的方式发送给 worker.js, 在 woker.js 中进行相关运算和整理并发送到服务器端; 当然, 不使用 Web Woker, 通过在单页面应用中的 index.html 中创建 iframe 也可以实现页面间切换, 页面停留时长等数据的采集, 具体的实现我就不细讲了, 感兴趣的同学可在网上搜索解决方案, 有什么疑问欢迎私信我~~~

### 场景三  Service Worker

可在浏览器控制台的 application 中里看到 Service Worker 的存在

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ea95b6935c2a4022adbfbf647fa281d7~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

Service Worker 是 web 应用做离线存储的一个最佳的解决方案, Service Worker 和 Web Worker 的相同点是在常规的 js 引擎线程以外开辟了新的 js 线程去处理一些不适合在主线程上处理的业务, 不同点主要包括以下几点:

- Web Worker 式服务于特定页面的, 而 Service Worker 在被注册安装之后能够在多个页面使用

- Service Worker 常驻在浏览器中, 不会因为页面的关闭而被销毁. 本质上, 它是一个后台线程, 只有你主动终结, 或者浏览器回收, 这个线程才会结束.

- 生命周期, 可调用的 API 也不同

我们可以使用 Service Worker 来进行缓存, 用 js 来拦截浏览器的 http 请求, 并设置缓存的文件, 从而创建离线 web 应用. 关于 Service Worker 的概念的介绍就到这里~~, 感兴趣的可以找相关文章学习, 有疑问的欢迎私信与我探讨~, 这里我们主要介绍的是使用 postMessage 方法进行 Service Worker 和页面之间的通讯.

**从页面发送信息到 Service Worker**

需要注意一点, 这个页面如果直接扔进浏览器里 (使用的是 file 协议) 打开是会报错的, 要使用 nginx 做端口映射, 或者用 node 搭建一个服务器 (使用 http 协议) 来访问该页面(目前我猜测的原因是浏览器对 file 协议打开的文件做了一些服务的限制, 如果有大佬知道具体原因还望告知). 下文也将附上我的 nginx 做端口映射的配置.

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<title>Service Worker跨窗口通信</title>
</head>
<body>
    <textarea id="showArea"></textarea>
    <script src="sw1.js"></script> 
    <script src="sw2.js"></script> 
    <script type="text/JavaScript">
        if('serviceWorker' in window.navigator) {
            // 对于多个不同scope的多个Service Worker,我们也可以给指定的Service Worker发送消息
            navigator.serviceWorker.register('./sw1.js', { scope:'./sw1'})
                .then(function(reg) {
                    console.log('success', reg);
                    return new Promise((resolve, reject) => {
                        const interval = setInterval(function() {
                            if(reg.active) {
                                clearInterval(interval);
                                resolve(reg.active);
                            }    
                        }, 1000);
                    }).then(sw => {
                        sw.postMessage("this message is from page to sw1");
                    })
                    
                })
            navigator.serviceWorker.register('./sw2.js', { scope:'./sw2'})
                .then(function(reg) {
                    console.log('success', reg);
                    return new Promise((resolve, reject) => {
                        const interval = setInterval(function() {
                            if(reg.active) {
                                clearInterval(interval);
                                resolve(reg.active);
                            }    
                        }, 1000);
                    }).then(sw => {
                        sw.postMessage("this message is from page to sw2");

                    })
                    
                });
                navigator.serviceWorker.addEventListener('message', function (event) {
                    console.log(event.data);
                    // 接受数据，并填充在 DOM 中
                    document.getElementById('showArea').value = event.data ;
                });
        }
    
    </script>
</body>
</html>

// sw1.js
self.addEventListener("message", function(event) {
    console.log("sw1.js " + event.data);
    event.source.postMessage('this message is from sw1.js, to page');
});

// sw2.js

self.addEventListener("message", function(event) {    
    console.log("sw2.js " + event.data); 
     // event.source是消息来源页面对象的引用   
    event.source.postMessage('this message is from sw2.js, to page');
});

```

nginx 做端口映射的相关配置:

```bash
// nginx.conf
// 因为有多个项目会用到nginx服务做端口映射
// 所以我在nginx的目录下新建了conf.d的文件来存放每个项目的配置.
// 然后在主配置文件里通过include引入

 http {    
    # 这里省略了一些你本机电脑上的nginx服务的配置    
    include conf.d/*.conf; 
}

// testHtml.conf
server {    
    listen 9090;    
    server_name       localhost;
    location / {        
        root  C:/Users/hzljie/Desktop/test/testb;  
        # 这是我的测试页面的存放路径,读者用的时候记得根据自己的来更改奥        
        index  test.html test.htm;    
    }
}
```

运行的效果的截图:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bd6378f6862445bdb6b0155f72111e27~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

这样就实现了 Service Worker 与其他页面的通信, 感兴趣的小伙伴可以一起来试一试奥. 如果你出现报错的情况, 要仔细检查自己的代码奥~

## 往期文章

[在 vue 中使用 SockJS 实现 webSocket 通信](https://juejin.cn/post/6844903664721592327 "https://juejin.cn/post/6844903664721592327")

[Expires, Last-Modified, Etag 缓存机制](https://juejin.cn/post/6844903816664449031 "https://juejin.cn/post/6844903816664449031")
