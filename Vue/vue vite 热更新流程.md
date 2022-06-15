---
date created: 2022-06-15 13:21
---

# 热更新流程

热更新的主要流程，需要关注的步骤的：

1.  修改代码
2.  重新编译（怎么编译，编译产物是什么，先不管）
3.  告诉前端要热更新了（怎么告诉，先不管）
4.  前端执行热更新代码进行热更新（怎么更新，先不管）

实际上，也就是这么几个过程

下面是我画的热更新的主要流程的时序图，大家一开始可能是看不懂的，这不重要，后面会逐一细讲，只要大概清晰各个部分的**时序关系**即可

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a5934586ff024233b4f70265943e5c54~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

`vite server`：指 vite 在开发时启动的 `server`

`vite client`：`vite dev server` 会在 `index.html` 中，注入路径为 `@vite/client` 的脚本，这个脚本是运行在浏览器的

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/decb906a8da14d4e8a780d1a74bdc24c~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

暂时先记住这个核心流程：

1.  修改代码，`vite server` 监听到代码被修改
2.  vite 计算出热更新的边界（即受到影响，需要进行更新的模块）
3.  `vite server` 通过 `websocket` 告诉 `vite client` 需要进行热更新
4.  浏览器拉取修改后的模块
5.  执行热更新的代码

我们先从离我们最近的浏览器端，开始介绍

---

# 热更新 API 简介

该小节主要讲这两部分：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/406912cfa3f14bb5bee46ae70d77636d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

这里主要涉及到两个 API：

- [hot.dispose](https://link.juejin.cn?target=https%3A%2F%2Fcn.vitejs.dev%2Fguide%2Fapi-hmr.html%23hot-dispose-cb "https://cn.vitejs.dev/guide/api-hmr.html#hot-dispose-cb")
- [hot.accept](https://link.juejin.cn?target=https%3A%2F%2Fcn.vitejs.dev%2Fguide%2Fapi-hmr.html%23hot-acceptcb "https://cn.vitejs.dev/guide/api-hmr.html#hot-acceptcb")

这两个 API 定义了拉取到新的代码之后，如何进行老代码的退出，和新代码的更新

我们先来看看，没有使用热更新 API 的代码被修改时，会发生什么？

## 不使用热更新 API

> 该小节对应的项目代码在 /package/ts-file-test，对应的文件为 [no-hrm.ts](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fcandy-Tong%2Fhrm-test%2Fblob%2Fmaster%2Fpackages%2Fts-file-test%2Fsrc%2Fno-hrm.ts "https://github.com/candy-Tong/hrm-test/blob/master/packages/ts-file-test/src/no-hrm.ts")

下图主要是一个 ts 文件，直接获取到一个 DOM，并替换其 innerHTML

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c76854be60ce4e0599e6f1f78e7af6e6~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

我们可以看到，该文件没有定义热更新，当文件被修改时，整个页面都重新刷新了。因为 **vite 不知道如何进行热更新，所以只能刷新页面**

## 使用 hot.accept API

> 该小节对应的项目代码在 /package/ts-file-test，对应的文件为 [accept.ts](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fcandy-Tong%2Fhrm-test%2Fblob%2Fmaster%2Fpackages%2Fts-file-test%2Fsrc%2Faccept.ts "https://github.com/candy-Tong/hrm-test/blob/master/packages/ts-file-test/src/accept.ts")

`import.meta.hot.accept` API 用于传入一个回调函数，来定义该模块修改后，需要怎么去热更新

```js
// src/accept.ts
export const render = () => {
  const el = document.querySelector<HTMLDivElement>('#accept')!;
  el.innerHTML = `
    <h1>Project: ts-file-test</h1>
    <h2>File: accept.ts</h2>
    <p>accept test</p>
  `;
};

if (import.meta.hot) {
  // 调用的时候，调用的是老的模块的 accept 回调
  import.meta.hot.accept((mod) => {
    // 老的模块的 accept 回调拿到的是新的模块
    console.log('mod', mod);
    console.log('mod.render', mod.render);
    mod.render();
  });
}
复制代码

```

当我们将修改该文件时（将 `<p>accept test</p>` 改成 `<p>accept test2</p>` ），之前老的模块注册的 accept 的回调就会被执行

mod 就是修改后的模块对象，在该文件中，mod 就是一个导出了 render 函数的对象

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb0293c689004a1883d16a070e6fdd51~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

当模块被修改时，重新执行 render 函数，设置 innerHTML 更新界面。

这时候我们**定义了如何进行热更新，vite 就不会刷新页面**了（刷新页面会清空所有请求，而下图没有清空请求）

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9126e1182c8a4fd79edfdabb194bc002~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

dispose 类似 hot，只是 **dispose 定义的是老模块如何退出，而 hot 定义的是新模块如何更新**

> 什么时候老模块需要退出？

假如你的页面有个定时器，就要在老模块退出时，将**定时器清除**，否则每次修改，页面会新增一个定时器，页面上的定时器会越来越多，造成内存泄露

dispose 主要用来做一些模块的退出工作

> 写热更新代码非常麻烦，应该没有人会在业务中写？

热更新代码的确很麻烦，业务中基本上也不会有人写，但我们在写 vue 代码时，确实有热更新的。

那是因为， **vite 的 `vite-plugin` 插件，在编译模块时加入了 vue 热更新的代码**。

**vite 本身只提供热更新 API，不提供具体的热更新逻辑**，具体的热更新行为，由 vue、react 这些框架提供

# 热更新边界

该小节主要讲这一部分

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6d095857d43149eea473fc5a92f727a3~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

> 什么是热更新边界？作用是什么？

假设有两个文件，关系如下

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/31a4ba25628549c3a256970924e82ccd~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

从上一小节，我们可以知道，vue 自带了热更新逻辑，而我们写的 ts 文件，没有热更新逻辑

当 `useData.ts` 被修改时，这时候是会刷新页面吗？

答案是不会的。vue 组件依赖的 ts 文件被修改，可以对这个 vue 文件进行热更新，重新加载组件。如果刷新页面，那开发体验就不太好了。

这时候，`index.vue` 就被称为热更新边界——**最近的可接受热更新的模块**

**沿着依赖树，往上找到最近的一个可以热更新的模块**，即热更新边界，对其进行热更新即可

> 为什么有时候修改代码可以热更新，有时候却是刷新页面？例如在 vue 项目中修改 main.ts

修改 `main.ts` 时，因为**往上找不到可以热更新的模块**了，vite 不知道如何进行热更新，因此只能刷新页面

如果其他 ts 文件，能找到热更新边界，就可以直接进行热更新

> 文件跟模块不是一一对应的吗？为什么需要遍历文件对应的模块？

**在 vite 中，文件跟模块不是一一对应**

因为 vite 可以加入查询参数，可查看 vite 文档【[更改资源被引入的方式](https://link.juejin.cn?target=https%3A%2F%2Fcn.vitejs.dev%2Fguide%2Ffeatures.html%23static-assets "https://cn.vitejs.dev/guide/features.html#static-assets")】

```js
// 显式加载资源为一个 URL
import assetAsURL from './asset.js?url'

// 以字符串形式加载资源
import assetAsString from './shader.glsl?raw'

// 加载为 Web Worker
import Worker from './worker.js?worker'

// 在构建时 Web Worker 内联为 base64 字符串
import InlineWorker from './worker.js?worker&inline'
复制代码

```

同一个文件，可能作为多个模块，例如 raw 时的编译产出的模块跟 worker 时编译产出的模块就是两个不同的模块

因为，**一个文件，是对应多个模块的。这些模块都需要找到他们的热更新边界，并进行热更新**

# 浏览器接收热更新信号

该小节主要讲这一部分

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/931a2a553a9b4319b1a3d94cc955f16b~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

> websocket 是什么创建的？

vite dev server 会在 `index.html` 中，注入路径为 `@vite/client` 的脚本，当访问 `index.html` 时，就会拉取该脚本

client.ts 在加载时，会创建 websocket 并监听 message 事件

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/208995a7f38e4445b4a3a14980e2ab9c~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

handleMessage 负责处理各种信号, 由于篇幅有限，我们不会展开讲细节

```js
async function handleMessage(payload: HMRPayload) {
  switch (payload.type) {
    case 'connected':
      // 连接信号
      console.log(`[vite] connected.`)
      setInterval(() => socket.send('ping'), __HMR_TIMEOUT__)
      break
    case 'update':
      // 模块更新信号
      break
    case 'custom': {
      // 自定义信号
      break
    }
    case 'full-reload':
      // 页面刷新信号
      break
    case 'prune':
      // 模块删除信号
      break
    case 'error': {
      // 错误信号
      break
    }
  }
}
复制代码

```

我们可以通过抓包的方式，看到 vite dev server 跟 client 之前的通信

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4951b969330f4e2c8d8b01d5c338d7c1~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

# server 模块转换

该小节主要讲这一部分

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca48e918fe3d41198101503211d125c7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

模块代码转换 vite 的核心，这部分足以开一个大的主题去讲，同样的，本文只会介绍个大概，只需要知道 vite 会转换代码即可，转换细节暂时可以不关注，把 vite server 当做一个黑箱

之前说的到，vite 的 `plugin-vue` 插件，将热更新代码注入到模块中，就是在编译转换模块的过程中处理

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/63bf85f0eb3b4fb3aae5a38e24a27a38~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

从图中可以看出，`index.vue` 经过编译后，内容是 js 代码，其中还能看到 `import.meta.hot.accept` 定义热更新的回调

> 时序图中，有个循环条件，直到动态 import 的模块没有模块依赖，是什么意思？

假如有以下两个文件：

```js
index.vue
  - useData.ts
复制代码

```

`index.vue` 依赖（import）了 `useData.ts`

当修改 `useData.ts` 时，会执行以下的步骤：

1.  vite 沿着依赖树，往上找到 `index.vue`，作为热更新边界
2.  server 将热更新边界信息，通过 websocket 传递到 client
3.  client 执行老的 `index.vue` 的 `import.meta.hot.dispose` 回调
4.  client 动态 `import(index.vue)`，vite 会重新编译 `index.vue`
5.  执行 `index.vue` 的代码（此时请求到 `index.vue` 虽然是 vue 后缀，但是它的内容经过编译后，是 js 代码），执行过程中遇到 import `useData.ts`

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/02b40b0a36de40da97afd14a9afcb493~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

5.  动态拉取 `useData.ts` 模块，vite 会重新编译 `useData.ts`
6.  执行 `useData.ts` 的代码
7.  client 执行新的 `index.vue` 的 `import.meta.hot.accept` 回调

因为**热更新边界的模块，可能会存在依赖**，import 了其他模块，这些模块都需要 import 拉取，直到动态 import 的模块没有模块依赖

# 参考资料

- [vite HRM API](https://link.juejin.cn?target=https%3A%2F%2Fcn.vitejs.dev%2Fguide%2Fapi-hmr.html "https://cn.vitejs.dev/guide/api-hmr.html")
- [vite 源码](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fvitejs%2Fvite "https://github.com/vitejs/vite")
