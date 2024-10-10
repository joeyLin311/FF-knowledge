> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [bj.chenenkou.com](https://bj.chenenkou.com/jishu/119.html)

> 使用 Fetch API 和 SSE 实现服务器发送事件服务器发送事件（Server-Sent Events，简称 SSE）是一种使得服务器能够向客户端推送信息的机制。在传统的 Web 应用中，客户端...

使用 Fetch API 和 SSE 实现服务器发送事件
============================

服务器发送事件（Server-Sent Events，简称 SSE）是一种使得服务器能够向客户端推送信息的机制。在传统的 Web 应用中，客户端需要定期轮询服务器以获取最新数据，而 SSE 提供了一种更高效的方式，让服务器能够主动向客户端推送数据。在本篇文章中，我们将探讨如何使用 Fetch API 和 SSE 来实现服务器发送事件。

SSE 简介
------

SSE 是一种轻量级的长连接技术，它允许服务器向客户端推送数据，而不需要客户端频繁地请求。SSE 使用 HTTP 协议，并通过`text/event-stream`内容类型来传输数据。服务器保持连接打开，并可以随时向客户端发送数据。

使用 EventSource
--------------

在浏览器中，你可以使用`EventSource`API 来轻松地处理 SSE。以下是一个简单的示例代码，展示了如何使用 JavaScript 处理 SSE：

```ts
// 创建一个新的EventSource连接，指定服务器的URL
var source = new EventSource('/path/to/server-endpoint');
// 监听服务器的消息事件
source.onmessage = function(event) {
  // 当服务器发送消息时，此函数会被调用
  console.log('Received message:', event.data);
};
// 监听open事件，服务器连接成功时触发
source.onopen = function(event) {
  console.log('Connection opened:', event);
};
// 监听error事件，连接出错时触发
source.onerror = function(error) {
  console.log('Error occurred:', error);
};
// 可以自定义事件类型的监听器
source.addEventListener('myevent', function(event) {
  var data = JSON.parse(event.data);
  console.log('Received custom event:', data);
});

```

JavaScript点击复制

在上面的代码中，`/path/to/server-endpoint`是服务器提供的 SSE 接口。客户端通过`EventSource`对象与服务器建立连接，并监听不同的事件类型，如默认的`message`事件、`open`事件、`error`事件，以及自定义事件。服务器发送的消息可以通过`event.data`访问。

类（Class）封装方案
------------

首先，我们提供一个使用类来封装 SSE 处理的方案：

```ts
class SSEClient {
  constructor(url) {
    this.url = url;
    this.eventListeners = {};
    this.reader = null;
    this.buffer = '';
  }
  connect() {
    this.controller = new AbortController();
    this.signal = this.controller.signal;
    fetch(this.url, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
        // "Content-Type": "application/json",
      },
      body: 'your=post&data=here',
      signal: this.signal
    })
    .then(response => {
      if (
        response.headers
            .get("Content-Type")
            .includes("text/event-stream")
      ) {
        this.reader = response.body.getReader();
        this.decoder = new TextDecoder();
        this.read();
      } else {
        throw new Error(`Expected Content-Type "text/event-stream", but received "${response.headers.get('Content-Type')}"`);
      }
    })
    .catch(error => {
      console.error('Error fetching SSE:', error);
    });
  }
  disconnect() {
    if (this.reader) {
      this.reader.cancel();
    }
    if (this.controller) {
      this.controller.abort();
    }
  }
  read() {
    if (!this.reader) {
      throw new Error('Reader is not initialized');
    }
    this.reader.read().then(({ done, value }) => {
      if (done) {
        console.log('Stream complete');
        return;
      }
      this.buffer += this.decoder.decode(value, { stream: true });
      const lines = this.buffer.split('\n');
      this.buffer = lines.pop(); // 保留最后一个不完整的行
      lines.forEach(line => {
        if (line.trim().length === 0) return; // 忽略空行
        // const [type, data] = line.split(': ');
        // if (type === 'data') {
        //   this.dispatchEvent('message', data);
        // }
        if (line.startsWith("data:")) {
            const data = JSON.parse(line.slice(5));
            this.dispatchEvent("message", data);
        }
      });
      this.read();
    }).catch(error => {
      console.error('Error reading stream:', error);
    });
  }
  addEventListener(type, callback) {
    if (!this.eventListeners[type]) {
      this.eventListeners[type] = [];
    }
    this.eventListeners[type].push(callback);
  }
  removeEventListener(type, callback) {
    if (!this.eventListeners[type]) {
      return;
    }
    const index = this.eventListeners[type].indexOf(callback);
    if (index !== -1) {
      this.eventListeners[type].splice(index, 1);
    }
  }
  dispatchEvent(type, data) {
    if (!this.eventListeners[type]) {
      return;
    }
    this.eventListeners[type].forEach(callback => {
      callback(data);
    });
  }
}
// 使用示例
const sseClient = new SSEClient('/path/to/server-endpoint');
sseClient.addEventListener('message', data => {
  console.log('Received data:', data);
});
sseClient.connect();

```

JavaScript点击复制

在这个类中，我们封装了连接、读取事件流、以及分发事件的逻辑。你可以通过`addEventListener`方法添加事件监听器，通过`removeEventListener`方法移除事件监听器，通过`connect`方法建立连接，以及通过`disconnect`方法断开连接。

非类封装方案
------

如果你不想使用类，也可以使用模块化的方式来编写处理 SSE 的代码：

```ts
const decoder = new TextDecoder();
function createSSEClient(url) {
  let reader = null;
  let buffer = '';
  const eventListeners = {};
  function connect() {
    const controller = new AbortController();
    const signal = controller.signal;
    fetch(url, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
      },
      body: 'your=post&data=here',
      signal: signal
    })
    .then(response => {
      if (response.headers.get('Content-Type') === 'text/event-stream') {
        reader = response.body.getReader();
        read();
      } else {
        throw new Error('Expected Content-Type "text/event-stream", but received "${response.headers.get('Content-Type')}"');
      }
    })
    .catch(error => {
      console.error('Error fetching SSE:', error);
    });
    return {
      disconnect: () => controller.abort()
    };
  }
  function read() {
    if (!reader) {
      throw new Error('Reader is not initialized');
    }
    reader.read().then(({ done, value }) => {
      if (done) {
        console.log('Stream complete');
        return;
      }
      buffer += decoder.decode(value, { stream: true });
      const lines = buffer.split('\n');
      buffer = lines.pop(); // 保留最后一个不完整的行
      lines.forEach(line => {
        if (line.trim().length === 0) return; // 忽略空行
        const [type, data] = line.split(': ');
        if (type === 'data') {
          dispatchEvent('message', data);
        }
      });
      read();
    }).catch(error => {
      console.error('Error reading stream:', error);
    });
  }
  function addEventListener(type, callback) {
    if (!eventListeners[type]) {
      eventListeners[type] = [];
    }
    eventListeners[type].push(callback);
  }
  function removeEventListener(type, callback) {
    if (!eventListeners[type]) {
      return;
    }
    const index = eventListeners[type].indexOf(callback);
    if (index !== -1) {
      eventListeners[type].splice(index, 1);
    }
  }
  function dispatchEvent(type, data) {
    if (!eventListeners[type]) {
      return;
    }
    eventListeners[type].forEach(callback => {
      callback(data);
    });
  }
  return {
    connect,
    addEventListener,
    removeEventListener
  };
}
// 使用示例
const sseClient = createSSEClient('/path/to/server-endpoint');
sseClient.addEventListener('message', data => {
  console.log('Received data:', data);
});
const { disconnect } = sseClient.connect();
// 当你需要断开连接时
// disconnect();

```

JavaScript点击复制

在这个非类封装方案中，我们创建了一个`createSSEClient`函数，它返回一个对象，包含`connect`、`addEventListener`和`removeEventListener`方法。这样，你可以在不同的地方导入并使用这个函数来创建 SSE 客户端，并根据自己的需求添加事件监听器和处理逻辑。  
请注意，这个示例使用了 ES6 模块语法。如果你在不支持 ES6 模块的环境中工作，你可能需要使用 CommonJS 语法或其他模块系统。

总结
--

无论是使用类还是函数，封装 SSE 客户端的逻辑都是为了重用和模块化代码。类的方式提供了更明确的封装和对象导向的编程风格，而函数方式则提供了更灵活的模块化。选择哪种方式取决于你的个人偏好和项目的具体需求。不过，两种方式都允许你轻松地处理服务器发送的事件，并能够适应不同的使用场景。
