name
----

如果项目是需要发版为 npm 包的，则`name`字段是必须的。  
因为它涉及到 npm 包的命名。

举个例子

笔者曾发布过开源的 npm 包，名字是 [`ping-url`](https://link.juejin.cn?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Fping-url "https://www.npmjs.com/package/ping-url")。  
对应的源代码 [`package.json`](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fwall-wxk%2Fping-url%2Fblob%2Fmaster%2Fpackage.json "https://github.com/wall-wxk/ping-url/blob/master/package.json")的定义如下：

```jsx
{
  "name": "ping-url",
  "version": "1.0.3",
  "description": "Check the url is normally accessible or not"
  
  // other
}

```

如果项目是不需要发版成 npm 包的，则`name`字段是可选的，不一定要设置。

### name 命名规范

* `name`字符串长度，必须小于或等于 214 个字符。
* 同一作用域内的包，可以用`.`或`_`作为开始字符
* 不能使用大写字母命名
* 因为`name`字段，在下载 npm 包时，会应用于`url`中，所以不能带任何不安全的 URL 字符。

### 不安全的 URL 字符

* 空格`" "`
* 大于小于号`<>`
* 方括号`[]`
* 花括号`{}`
* 竖线`|`
* 反斜杠`\`
* 插入号`^`
* 百分比`%`

### 私源 npm 包怎么命名？

格式：`@[scope]/[name]`。

举个例子：

笔者想要发布个私源是`@leon`，包名是`ping-url`的包，则`name`需要命名为：`@leon/ping-url`。

```jsx
{
  "name": "@leon/ping-url",
  "version": "1.0.3"
}
```

安装命令：`npm install @leon/ping-url`。

version
-------

`version`字段用于定义版本号。

如果项目是为发布 npm 包，则必须包含此字段。如果是普通的项目，则此字段是可选的。

每次发布的`version`，必须是唯一的，之前发布的时候没使用过的。  
`version` 的命名规则和注意点，可以参考笔者的另一篇文章 [package.json 怎么管理依赖包版本？](https://juejin.cn/post/7108688424818704398 "https://juejin.cn/post/7108688424818704398")，这里就不展开了。

description
-----------

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/71a1841863c24c3c80e4054f5d6fd02d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

`description`用于描述当前项目的概况。  
如上图所示，发布的 npm 包，在搜索结果中，可以直接显示`description`内容，方便使用方直接了解包的功能。

keywords
--------

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c23cc89ab4f47108db9ff620a26f0cb~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

图中标签在`package.json`中对应的定义：

```jsx
"keywords": [
    "ping url",
    "ping host",
    "ping"
  ]
```

从上述例子可以很清晰地看出，`keywords`是标签，用于标记当前项目的重点词汇。同时，可以作为搜索关键词，提供给资源平台使用，进行索引。

homepage
--------

项目的官网主页地址。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b6544004a9bc412cbd03661097df592b~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

```jsx
"homepage": "https://github.com/wall-wxk/ping-url"
```

项目有对应官网地址的话，可以在`homepage`中声明。如果没有的话，也可以像笔者这样，放个 github 项目源码入口。

repository
----------

项目的源码地址。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4eb5e96a2e074aaf821eb029fc38ce8b~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

```jsx
"repository": {
    "type": "git",
    "url": "https://github.com/wall-wxk/ping-url.git"
}
```

开源项目，这个字段很重要。  
因为有意向的协作者，可以通过字段信息，便捷地进入查看项目源码。

license
-------

项目的协议类型。

这个项目涉及到知识产权方面的知识。所以开源项目的时候，要重点考虑到底要用哪个协议，而不是无脑用`MIT`。 具体的可选协议列表，可查看 [SPDX License List](https://link.juejin.cn?target=https%3A%2F%2Fspdx.org%2Flicenses%2F "https://spdx.org/licenses/")

author
------

作者信息。

```jsx
"author": {
    "name": "leon",
    "email": "582104384@qq.com",
    "url": "https://wangxiaokai.vip"
}
```

如果有兴趣让别人知道你是谁，这个字段是必不可少的。  
当然如果你的代码是`shit mountain`，这个字段也可以让别人知道是你写的....

contributors
------------

协作者信息。

格式是一个对象数组。对象内容和`author`一致。

```jsx
"contributors": [{
    "name": "hanmeimei",
    "email": "hanmeimei@qq.com"
},{
    "name": "lihua",
    "email": "lihua@qq.com"
}]
```

files
-----

声明有哪些文件，是需要作为依赖项，保留下来。  
不然，执行`npm publish`进行发布时，这些文件是会自动屏蔽上传的。  
同理，也可以使用`.npmignore`文件进行配置。

```jsx
"files": [
    "dist/*.js",
    "lib"
]
```

如果没有`files`字段声明，则这些文件，都不会保留，npm 包将不能使用。

main
----

使用 npm 包时，需要进行`require(..)`的操作。这个操作，会查看`main`字段，找到程序的主入口。

bin
---

工具性质的 npm 包，一定有`bin`字段，对外暴露脚本命令。

举个例子

```jsx
"bin": {
    "npg-cli": "bin/cli"
}
```

笔者发布的`npm-package-cli`包，是用于生成 npm 包脚手架的工具，对外暴露了脚本命令：`npg-cli`。详情可查看 [npm-package-cli](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fwall-wxk%2Fnpm-package-cli "https://github.com/wall-wxk/npm-package-cli")。  
使用方安装`npm-package-cli`包后，`npg-cli`命令会进行注册，可以在`CMD`中识别并运行。

scripts
-------

项目脚本命令。（PS：这个就不需要解释了：））

需要注意的，一定要有约定俗成的规范脚本命令，降低维护成本。  
比如：

* `npm run start` 项目启动
* `npm run build` 打包

dependencies、devDependencies、peerDependencies
---------------------------------------------

依赖的使用性质划分。详细可查看笔者的文章 [package.json 怎么管理依赖包版本？](https://juejin.cn/post/7108688424818704398 "https://juejin.cn/post/7108688424818704398")。

需要提及的一点：`peerDependencies`在 npm 包的依赖关系处理中，很重要。

举个例子

UI 组件库`leon-ui` 依赖 React 版本 16，那么`package.json`中，应该用`peerDependencies`告知使用方，需要 React 16。

```jsx
"peerDependencies": {
    "react": ">=16.8.0"
}
```

大家还可以注意项目`npm install`时，控制台会输出一些依赖的`WARN`，就是`peerDependencies`在起作用。

private
-------

`private`和发布 npm 包相关。

当`private: true`时，npm 会拒绝发布当前项目。这是防止意外发布个人仓库的一种保护方式。

publishConfig
-------------

用于定义发布 npm 时，设置相关信息。

举个例子：  
笔者发布私源 npm 包，到自己的 npm 服务，可以配置如下：

```jsx
"publishConfig": {
  "registry": "http://npm.wangxiaokai.vip/repository/",
  "access": "public",
  "tag": "leon-tag"
}
```

* `registry` 发布的 npm 私源地址
* `access` 发布有作用域的包（比如`@leon/ping-url`），必须要设置`access`。
* `tag` 指定当前版本对应的标签

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b7cdd6c22b6c4908ab7c2b5329886ef0~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

如图所示，右侧即是标签 tag。

> 注意  
> 如果没有显式指定 tag，默认 tag 是`latest`

types
-----

项目如果是用`TypeScript`写的，则需要`types`字段，对外暴露相关的类型定义。

module
------

性质等同于`main`字段。`module`用于 ES6 规范的模块，只要支持 ES6，会优先使用`module`入口。  
这样，代码也可以启用`tree shaking`机制。

unpkg
-----

CDN 方式下，引入当前 npm 包的链接。

sideEffects
-----------

`sideEffects`格式：`boolean | string[]`。

`sideEffects: false`用于告知打包工具（webpack），当前项目`无副作用`，可以使用`tree shaking`优化。

`sideEffects`的值，也可以是一个文件路径组成的数组。告知哪些文件`无副作用`，可以使用`tree shaking`优化。

```jsx
"sideEffects": [
    "a.js",
    "b.js"
]
```

### 注意点

`"import xxx;"`语句，只引入未使用，如果声明了 sideEffects，则会被`tree shaking`删除掉。

并且，由于`tree shaking`只在`production`模式生效，所以本地开发会一切正常，生产环境很难及时发现这个问题。

当然， 样式文件使用`"import xxx;"`的方式引入，会进行保留。

engines
-------

项目运行环境的要求声明。

```jsx
"engines": {
    "node": ">=0.10.3 <15"
}
```

上述代码，告知 node 版本需要在`0.10.3`与`15`之间，才可以运行当前项目。  
在不符合条件的环境中运行项目时，控制台会有报错输出。

os
--

操作系统的要求声明。

```jsx
"os": [
    "darwin",
    "linux"
]
```

cpu
---

CPU 的要求声明。

```jsx
"cpu": [
    "x64",
    "ia32"
]
```

workspaces
----------

`monorepo`类型的项目，需要用到`workspaces`。它可以告知其他工具，当前项目的工作区间在哪里。

```jsx
{
  "name": "workspace-example",
  "workspaces": [
    "./packages/*"
  ]
}
```

具体怎么在实战中使用，可查看笔者的另一篇文章[使用 Yarn 与 Lerna 管理 monorepo](https://juejin.cn/post/7073285203728269343 "https://juejin.cn/post/7073285203728269343")。

bugs
----

开源项目用于接收 bug 反馈。

```jsx
{
  "url" : "https://github.com/owner/project/issues",
  "email" : "project@hostname.com"
}
```

参考
--

* [npm Docs](https://link.juejin.cn?target=https%3A%2F%2Fdocs.npmjs.com%2Fcli%2Fv7%2Fconfiguring-npm%2Fpackage-json "https://docs.npmjs.com/cli/v7/configuring-npm/package-json")
