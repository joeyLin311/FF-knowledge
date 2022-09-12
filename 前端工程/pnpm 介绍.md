> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7115756767908200478)

pnpm 是什么
========

和`npm`，`yarn`一样，`pnpm`是一个包管理工具。不一样的是，`pnpm`解决了`npm`和`yarn`一直都没有解决的痛点。在许多方面比`npm`和`yarn`更优秀。

### pnpm 对比 npm/yarn 的优点

* **更快速的依赖下载**
* **更高效的利用磁盘空间**
* **更优秀的依赖管理**

更快速的依赖下载
========

根据官方提供的数据↓

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d54cac194554bf2bbac75c67c1dff83~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

从图上我们可以看出，`pnpm`平均比`npm`和`yarn`快上 2~3 倍。**这一点在依赖的下载上额外明显。**

更高效的利用磁盘空间
==========

为什么说`pnpm`会比`npm`和`yarn`更高效的利用磁盘空间?

`pnpm` 有一个`store`的概念（是一块存储文件的空间，后面会说到），内部使用 "基于内容寻址" 的文件系统来存储磁盘上所有的文件，这一套系统的优点是：

不会重复下载依赖
--------

举个例子 假设出现这么一个情况↓

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8d341f632af84e019527e5cb23838c74~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**你 4 个项目都依赖了 express.js(第三方插件)。如果是 npm/yarn 的话，express.js 就会被安装 4 个在你的磁盘空间当中**。从而出现下面这个情况↓

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0e06b6cc9d5541a9be2f0f77133f78dd~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

但是`pnpm` 得益于 "基于内容寻址" 的文件系统，**使用 pnpm 下载的文件不论被多少项目所依赖，都只会在磁盘中写入一次。后面再有项目使用的时候会采用硬链接的方式去索引那一块磁盘空间。**

所以，在同样被多个项目依赖的时候，pnpm 对磁盘的占用如下↓

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f0cc6bac7fc44db09f782de716b9ae00~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

### 更好玩的是

如果有一天你所依赖的版本提升了。假设从`express@2.0`升级到了`express@3.0`。而`express@3.0`比`express@2.0`多了 20 个文件。这个时候`pnpm`并不会删除`express@2.0`再去重新下载`express@3.0`。而是复用`express@2.0`原本的磁盘内容。再在`express@2.0`的基础上增加 20 个文件形成`express@3.0`。

更优秀的依赖管理
========

**在 npm1 和 npm2 的时候。依赖结构是这样的**↓

**代码示例（一）**

```bash
node_modules 
└─ 依赖A 
   ├─ index.js 
   ├─ package.json 
   └─ node_modules 
       └─ 依赖B 
       ├─ index.js 
       └─ package.json
 └─ 依赖C 
   ├─ index.js 
   ├─ package.json 
   └─ node_modules 
       └─ 依赖B 
       ├─ index.js 
       └─ package.json
```

上面这种结构会导致哪些问题呢？让我们来分析下。

* **依赖包会被重复安装**
* **React 实例无法共享**
* **依赖层级太多问题**

依赖包会被重复安装
---------

还是以 **代码示例（一）** 为例子。A 依赖 B，C 也依赖 B。也就是说 B 同时是 A 和 C 的依赖。**这种情况下。B 会被下载两次。**

`npm1`和`npm2`的运行逻辑是，某一个包被其他包依赖 N 次，就需要被下载 N 次。也就是我们所说的**重复安装**。

React 实例无法共享
------------

举个例子。

同一 node_modules 目录下

A 包中引入了 React （import React from 'react'）

C 包中也引入了 React （import React from 'react'）

**虽然同为 React, 但这两个 React 其实是两个实例，有各自的存储空间。这样就意味着。存储在 A 包的 React 实例内容，无法通过 C 包的 React 实例引用到。**

依赖层级太多问题
--------

假设我们有这么一个依赖↓

```bash
node_modules 
└─ 依赖A 
   ├─ index.js 
   ├─ package.json 
   └─ node_modules 
       └─ 依赖B 
       ├─ index.js 
       ├─ package.json
       └─ node_modules 
           └─ 依赖C 
           ├─ index.js 
           ├─ package.json 
           └─ node_modules 
               └─ 依赖D 
               ├─ index.js 
               └─ package.json
```

上方的代码示例就有四层。如果我们有某一个依赖了 10 层呢？他也一样会一层一层依赖下去。像是 "依赖地狱"。可读性不高。

npm3 和 yarn 中平铺的结构依然存在的问题
=========================

上述`npm1`和`npm2`的这些问题在`npm3+`和`yarn`中得到了解决

从`npm3`开始。`npm3`和`yarn`都采用了 "扁平化依赖" 来解决上述问题。

采用了扁平化管理之后，**代码示例（一）** 就从**嵌套结构**变成了**扁平化结构**, 像这样↓

**代码示例（二）**

```bash
node_modules 
└─ 依赖A  
    ├─ index.js 
    ├─ package.json 
    └─ node_modules 
└─ 依赖C   
    ├─ index.js 
    ├─ package.json 
    └─ node_modules 
└─ 依赖B 
    ├─ index.js 
    ├─ package.json 
    └─ node_modules 
```

**所有的依赖都会被平铺到同一层面**。这样，因为 require 寻找包的机制。如果 A 和 C 都依赖了 B。那么 A 和 C 在自己的 node_modules 中未找到依赖 C 的时候会向上寻找，并最终在与他们同级的 node_modules 中找到依赖包 C。 这样，**就不会出现重复下载的情况。而且依赖层级嵌套也不会太深。因为没有重复的下载，所有的 A 和 C 都会寻找并依赖于同一个 B 包。自然也就解决了实例无法共享数据的问题**

> 这里需要提一嘴 require 寻找依赖包的机制，require/import 在引入一个包的时候（比如 require("express")，不考虑使用路径引入的情况）;require、import 会从自己当前的 node_modules 的一级文件（或文件夹）中寻找这个依赖，如果当前依赖没有，则会去上一级的 node_modules 中寻找该依赖，以此类推一直到根节点。到跟节点都为找到依赖的话则会报错。

**这种平铺的结构看似完美，但其实依然存在一些细节上的问题。**

比如

* **依赖结构的不确定性**
* **扁平化算法的复杂度比较高，相对的比较耗时。**
* **项目中还是存在可以非法访问的问题**

下面我们来聊聊这三个问题。

依赖结构的不确定性
---------

上面我们已经说过，`npm3`和`yarn`已经开始采用扁平化的依赖结构。也就是说会将所有的依赖包不分几级依赖的全部都平铺到同一个层级中。然后通过包 require 引入的特性去访问。

那么问题来了。如果有那么一个依赖形式呢？↓

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26fae65877e843abbd21299b8e6218d8~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

A 包和 C 包依赖于 B 包的不同版本（也就是两个不同的包依赖与同一个包的不同版本）。

**因为同一目录下不能出现两个同名的文件，所以如果依赖于同一个包的不同版本，那么有一个版本注定还是要被嵌套依赖。**

在这种情况下，你认为扁平化的依赖会是哪种形式↓

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/deb554396f7a4799ba2c823556f54605~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

答案是：**都有可能**

这完全取决于 A 包和 C 包在 package.json 中的位置。在前面先处理的依赖会被扁平化到 A，C 两包的同级目录中。后处理的包当 npm3/yarn 打算把依赖包平铺的时候会发现目录下已经有该包的另一个版本。所以后处理的包不会扁平化处理，保持原味。

扁平化算法的复杂度比较高，相对的比较耗时
--------------------

这也是一个问题。要知道，对文件的操作可不像我们给数组和对象进行排序。这种深度嵌套的结构。npm3/yarn 在处理过程中相对的麻烦。所需要运算的时间会比较长。尤其是大项目，依赖了很多很多包的时候，我们会明显的感觉到，npm 的依赖安装变慢了。

项目中还是存在可以非法访问的问题
----------------

什么叫非法访问？

举个例子

项目 依赖 B 包

B 包 依赖 C 包

然后你就会惊讶的发现，你在项目中，居然可以访问到 C 包里的内容。

原理解释↓

> 因为扁平化的处理机制，身为 B 包依赖的 C 包，也会被放到和 B 包同级别的 node_modules 下，而我们在项目中使用 require/import 去引入 C 包的时候。require 的机制会去当前的 node_modules 下（也就是 B 包所在的那个目录下）寻找 C 包。然后，他就找到了。我们就拿来用了。这就是非法访问。

这会造成什么问题呢？

如果你在项目中使用了 C 包 @1.0，那么有一天，B 包升级了，他的依赖 C 包也跟着升级成了 @2.0 并遗弃了 [C@1.0](https://link.juejin.cn?target=mailto%3AC%401.0 "mailto:C@1.0") 在项目中使用的某个 Api。那么这个时候，项目就会报错。而这种错误，一旦出现，处理起来就会非常棘手，尤其是当我们有大量使用的时候。那就哦吼了。

pnpm 解决了这些问题
------------

我们先执行以下命令

> pnpm init

> pnpm install express

然后你就会发现，express 直接出现在了 node_modules 下面，他的依赖并没有和他处于同一目录下，而他本身目录下也不具备 node_modules。像这样↓

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/275ea10933dc4226bb61ea741050346d~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

为什么会这样？

因为 node_modules 下面的 express 仅仅只是一个软连接，`pnpm`直接抛弃了`npm3`/`yarn`原本在项目级别的扁平化结构。项目级别的 node_modules 下用`软链接`([什么是软连接？_百度知道 (baidu.com)](https://link.juejin.cn?target=https%3A%2F%2Fzhidao.baidu.com%2Fquestion%2F1546528787657412147.html "https://zhidao.baidu.com/question/1546528787657412147.html")) 代替。

那 express 的依赖又去了哪里？

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0326ee80083d4bf3bfc3dec4dda0144e~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

最后我们在`.pnpm/express@version/node_modules/` 下面找到了`express`的依赖。

**使用`pnpm`下载的所有依赖都会以`.pnpm/依赖名@版本号/node_modules/`这种形式被存储。**

乍一看，好像又从扁平化管理回到嵌套结构了。性能不是倒退了吗？

**不是的**。

一开始从`npm1`和`npm2`的嵌套结构变成扁平化结构是为了解决

1. 包被重复安装

2. 依赖示例无法共享

3. 依赖层级太深。

这三个问题。而`.pnpm/依赖名@版本号/node_modules/`下面的依赖也全部都是软连接。这些软连接指向存储在`store`中的文件。(`store`是`pnpm`的文件公共存储空间，在后面会有介绍)

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f122bacba544404cba5c46488e2b2d39~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

发现这种设计的巧妙之处了吗。

**因为`.pnpm/依赖名@版本号/node_modules/`下面都是软连接，他们指向同一块存储空间。所以也就不存了包会重复安装和依赖实例无法共享的问题。**

**而，`express`所有的依赖都会在`.pnpm/依赖名@版本号/node_modules/`这个目录下被扁平化处理，同样解决了依赖结构太深的问题。**

还有将包本身和依赖放到同一目录下，这样，利用 require 的特性也能够找到所有的依赖包。再将包本身的软连接放到外层的 node_modules 中。这样，node_modules 中的包在结构上就几乎和 package.json 中的内容保持一致。为什么说几乎一致而不是完全一致？因为有些包有变量提升，会被提升到外层 node_modules 中。但是大体上还是一样的。

**到这里，`pnpm`就`又`解决了`npm3`/`yarn`当时没有解决的依赖结构的不确定性。**

### 关于 npm3/yarn 未解决的非法访问问题

而得益于与 pnpm 的这种包管理方式，我们就会发现，呦呵，非法访问问题也解决了。怎么解决的呢？ 我画了张图↓

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7308fb8ba4f04290bec44326238dab0a~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

当我们再项目中引用的时候，他会去 node_modules 中去寻找。由上图可知，**只会在 node_modules 下面的第一层去寻找！！！** 而`pnpm`的机制会让 node_moduels 下只有一级依赖包的软链接 (也就是说如果你下载一个`express`, 那么项目级别的 node_modules 下就只有 express 的软连接而没有`express`的依赖包的软链接)。所以如果你在自己的项目中直接去引用二级依赖包的话，会报错，直接找不到 **（如上图）** 。

可能有些同学还不理解，没关系，我们再举个例子。

我们下载了一个 A 包。A 包的软连接在项目级别的 node_modules 中，但是 A 包的所有依赖包都在. pnpm / 依赖名 @版本号 / node_modules / 下面。假设 C 包是 A 包的依赖包。我们直接在项目中使用 require/import 引用 C 包的话。require/import 会去项目级别的 node_modules 中寻找。然后就会发现项目级别的 node_modules 下面只有一个 A 包的软连接，没有 A 包的依赖 C 包，找不到，就会报错。这样，只要 C 包没有在项目的`package.json`中声明，就无法访问（避免了非法访问）。

所以。到这里。`pnpm`也解决了`npm3`和`yarn`中非法访问的问题。

而且，非法访问的这个问题以当前的`npm`和`yarn`的版本，使用在项目级别的 node_modules 下进行扁平化的管理的机制，几乎无法避免。**`pnpm`却完美的解决了。**

pnpm 的 store
============

上面说到，`pnpm`都是软连接的时候，有些小伙伴就不明白了。

都是软连接，那我文件到底存哪里去了？？他链接到哪去了？？

**容我介绍一下 pnpm 的主角:`store`**

**`pnpm`下载的依赖全部都存储到`store`中去了，`store`是`pnpm`在硬盘上的公共存储空间。**

`pnpm`的`store`在 Mac/linux 中默认会设置到`{home dir}>/.pnpm-store/v3`；windows 下会设置到当前盘符的根目录下。使用名为 .pnpm-store 的文件夹名称。

项目中所有`.pnpm/依赖名@版本号/node_modules/`下的软连接都会连接到`pnpm`的`store`中去。

pnpm 如何使用
=========

这么好的一个包管理工具，要如何去使用呢？其实并不难，对于曾经学习过 npm 的我们来说。几乎等于 0 成本学习。

```bash
npm install pnpm -g pnpm安装

pnpm add <pkg> 安装软件包到 dependencies

pnpm add -D <pkg> 安装软件包到 devDependencies

pnpm add -g <pkg> 全局安装软件包

pnpm install 或 pnpm i 下载项目所有依赖项

pnpm update 或 pnpm up 遵循 package.json 指定的范围更新所有的依赖项

pnpm update -g <pkg> 从全局更新一个依赖包

pnpm remove <pkg> 从项目的 package.json 中删除相关依赖项

pnpm remove -D <pkg> 仅删除开发环境 devDependencies 中的依赖项

pnpm remove <pkg> -g 从全局删除一个依赖包

pnpm run <script> 或 pnpm <script> 运行脚本
```

这些几乎就是`pnpm`所有的命令了。和`npm`几乎保持一致。

总结
==

* **比 npm 和 yarn 更快速的依赖下载**
* **比 npm 和 yarn 更高效的利用磁盘空间**
* **比 npm 和 yarn 更优秀的依赖管理**
