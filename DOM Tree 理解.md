---
date created: 2021-12-09 22:57
---

#browser

# DOM tree 的理解

> 所属, 浏览器-> 浏览器渲染 -> new article

## 什么是 DOM

DOM, Document Object Model, 文档对象模型, 是 HTML 和 XML 文档的编程接口. 它提供了对文档的结构化的表述, 并定义了一种方式可以使从程序中对该结构进行访问, 从而改变文档的 **结构** , **样式** 和 **内容**.

提炼 3 点:

- 核心 DOM : 针对任何结构化文档的标准模型
- XML DOM : 针对 XML 文档的标准模型
- HTML DOM: 针对 HTML 文档的标准模型

在渲染引擎中, DOM 有三个层面的作用:

- 从页面的视角来看, DOM 是生成页面的基础数据结构
- 从 JavaScript 脚本来看, DOM 提供给 JavaScript 操作的接口, 通过这套接口, JS 可以以对 DOM 结构进行访问, 从而改变文档的结构, 样式和内容
- 从安全视角看, DOM 是一道安全防护线, 一些不安全的内容在 DOM 解析阶段就被过滤掉了

## DOM 的解析和生成

[DOM 概述 - Web API 接口参考 | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/API/Document_Object_Model/Introduction)
[理解 DOM 结构 | 前端工程师手册 (gitbooks.io)](https://leohxj.gitbooks.io/front-end-database/content/html-and-css-basic/learn-dom-tree.html)
浏览器内部通过一个 HTML 解析器(HTMLParser) 模块, 将 HTML 字节流转换为 DOM 结构.

- 在网络进程接收到响应头之后, 会根据头部中的 content-type 字段值为 `text/html`, 浏览器根据它识别资源为 HTML 类型的文件, 然后为该资源创建一个渲染进程. 通关建立网络进程和渲染进程的数据管道, 将网络进程中获取的数据传输给渲染进程进行解析
- 渲染进程大致通过以下几个步骤将 HTML 字节流解析成 DOM:
  - Conversion(转换):  浏览器读取原始数据, 根据文件编码将字节转成字符
  - Tokenizing(分词): 渲染进程根据 HTML 规范将字符转换为一个个 Token, 分为 Tag Token 和 文本 text Token
  - Lexing (词法分析): 分词之后产生的标记被转换成 nodes, 包含 HTML 语法的各种信息, 如属性, 属性值, 文本等
  - DOM construction (DOM 构造): 因为 HTML 标记定义了不同标签之间的关系, 上一步产生的对象会链接在一个树状的数据结构中, 以标识节点的父子, 兄弟关系.

### DOM 节点

根据 W3C 的 HTML DOM 标准, HTML 文档中所有的内容都是节点, 在标准提供的 Document 接口中, 包含了节点类型常量

| 常量                               | 值  | 描述                     |
| -------------------------------- | -- | ---------------------- |
| Node.ELEMENT_NODE                | 1  | 元素节点, div, p           |
| Node.TEXT_NODE                   | 3  | element 或者 attr 中的文字   |
| Node.CDATA_SECTION_NODE          | 4  | 一个 CDATASection 节点     |
| Node.PROCESSING_INSTRUCTION_NODE | 7  |                        |
| Node.COMMENT_NODE                | 8  | 一个 Comment 节点          |
| Node.DOCUMENT_NODE               | 9  | 一个 Document 节点         |
| Node.DOCUMENT_TYPE_NODE          | 10 | 描述文档类型的节点              |
| Node.DOCUMENT_FRAGMENT_NODE      | 11 | 一个 DocumentFragment 节点 |

## 所谓的线程互斥

当 HTML 解析器遇到 script 标签时, 会暂停 HTML 的解析, 转而交由 JavaScript 引擎执行 script 标签中的脚本. 因为脚本中可能会涉及改变当前 DOM 树的数据或结构. 当执行完 script 标签中的脚本之后, HTML 解析器回复解析构成, 继续往后解析, 直至生成最终的 DOM 树.

引申出的问题的是, 如果 script 标签中的脚本是引用自其它地方的 JS 文件, 就需要先下载这段代码. JavaScript 文件的下载过程也会阻塞 DOM 的解析, 而下载动作一般认为都是耗时操作. 针对这种情况, Chrome 浏览器做了相关的优化, 其中一个主要优化师预解析操作: 当渲染引擎收到字节流之后, 会开启一个预解析线程, 用来分析 HTML 文件中包含的 JavaScript, CSS 等相关文件, 然后针对这些文件进行提前下载.

针对 JavaScript 线程会阻塞 DOM 的情况, 我们可以使用一些相关的策略进行规避:

- 使用 CDN 来加速 JavaScript 文件的加载
- 压缩 JavaScript 文件的体积
- 在 script 标签中设置 async 或者 defer 属性来标记代码, 改变标签的加载变成异步的
  - async 标志的 JS 一旦加载完成, 会立即执行
  - defer 标志的 JS 文件, 需要在 DOMContentLoaded 事件之前执行
