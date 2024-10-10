> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7280747221510144054)

前面一篇文章介绍了如何做 Vite 的插件开发 [# 20 分钟掌握 Vite 插件开发](https://juejin.cn/post/7276260308515389480 "https://juejin.cn/post/7276260308515389480")，这篇文章介绍清晰 Vite 的构建原理，为什么比 Webpack 构建速度快这么多。

## 一句话解释 Webpack 的构建原理
======================

**前端之所有需要 类似于 Webpack 这样的构建工具，是为了提高项目的开发效率，Webpack 通过分析 js 中的 require 语句，分析出当前 js 文件所有的依赖文件，通过递归的方式层层分析后，得到整个项目的依赖关系图，对图中不同的文件执行不同的 loader，比如使用 css-loader 解析 css 代码，最后基于这个依赖关系图读取到整个项目中的所有文件代码，进行打包处理之后交给浏览器执行。**

这个过程一气呵成，顺理成章。这里我们只用文字简单描述 Webpack 的工作原理。想要弄懂其中的详细过程，关注一点，我们下一篇文章再详细剖析。

这样的构建构过程，导致了在我们调试代码之前，需要等待 Webpack 的依赖收集过程，而当项目代码体量很大时，这个依赖收集的过程往往需要我们等待几十秒甚至一两分钟，程序员在这个时间段内只能摸鱼，开发体验非常差，有什么办法解决这个问题？

如果有办法能够做到更少的代码打包就好了！！！，于是 bundless 的打包思路就诞生了，Vite 便是这个这种思路的遥遥领先。

## script 的模块化
==============

需要打包工具的核心原因，就是浏览器在执行代码的时候，本身没有一个很好的方式去读懂我们的项目中的各个文件引入关系。

所以

`Webpack 对浏览器说`：“你别纠结了，我把所有的文件引入关系都梳理好了，并且将项目中所有文件的代码打包在了一起，你就去执行找一个文件吧！”

`浏览器开开心的说`：“好的大哥，谢谢大哥”，然后毫无压力的哐哐运行

但是随着浏览器的进步，它开始能慢慢读的懂一些模块化的引入语法了，比如：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b60d49675bbf4fd3ab20341e7368b9dc~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1040&h=534&s=240794&e=png&b=c09fec)

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8de8a65c4f5f42cda082ffb6b22923fc~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1040&h=534&s=244260&e=png&b=c09fec)

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3aef291e3bd445dab4e29a99b2bc3323~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1502&h=766&s=388369&e=png&b=322a3f)

浏览器是可以正常那运行这份代码的： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4255774767634af6853c5c30a70bb861~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2876&h=844&s=119163&e=png&b=ffffff)

**再看一眼浏览器的是怎么处理这些文件引入关系的：**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f779ca9e761946cb8d5739d86f9123e8~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2874&h=1296&s=290780&e=png&b=fdfdfd)

浏览器会将 `import` 语句处理成一个个 HTTP 网络请求，去获取 `import` 引入的各种模块， 就因为浏览器现在可以通过 `type="module"` 这种方式读懂项目中文件的模块化引入，所以，bundless 的思想得以发展

## Vite 的一步一步实现
===============

Vite 正是借助了刚刚我们说的浏览器的这一特性，将项目中的各种文件引入处理成网络请求，当浏览器执行到了模块依赖的代码，便自己去请求所需要的代码资源。这么简单？没错！那接下来我们一步一步自己打造一个 Vite。

首先，这里我们不使用构建工具，自己创建一个 Vue 的项目（参考 vite 的构建目录自己一个一个创建）

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4cc194c7d47b4606ac09474a399a7f55~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=630&h=938&s=68740&e=png&b=f3f3f3)

在 index.html 中写入以下引入

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/05a769ef180844ed9cb6e42ac15939f2~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2864&h=1530&s=320464&e=png&b=fbfbfb)

但是现在浏览器是无法运行这份 html 的，因为这其中还有一个问题

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ada225ff8cdb471e8da6de80dc160e71~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1646&h=540&s=302164&e=png&b=322a3f)

来自于 node_modules 的模块浏览器无法读懂它的路径，进而无法请求到正确的资源，所以我们需要手动解决这一问题

在项目的根目录下创建 simple-vite.js 文件，通过 node 启动一个 web 服务，向浏览器响应资源

```
const http = require('http')
const fs = require('fs')

const server = http.createServer((req, res) => {
  const { url, query } = req

  if (url === '/') {
    // 设置响应头的Content-Type是为了让浏览器以html的编码方式去加载这份资源
    res.writeHead(200, { 
      'Content-Type': 'text/html'
    })

    let content = fs.readFileSync('./index.html', 'utf8')
    res.end(content)
  }

})

server.listen(8080, () => {
  console.log('listening on port 8080');
})


```

用 node 运行这份 js 后：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0af3b23ef20346a89cd8479e164134d9~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2872&h=1512&s=355148&e=png&b=ffffff) 这个应该就不用再详细解释了，node 没有基础的看官只能自己先学习一下了

因为 html 代码中有 main.js 这个模块的引入，所以浏览器会自动发这个请求，但是浏览器访问的是我们自己的后端，我们没有写这个对应的响应，所以是看不到东西的

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5f44c4ed11ed44caad85949c0f932153~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2830&h=1320&s=194838&e=png&b=ffffff)

继续处理 js 文件的请求

```
const http = require('http')
const fs = require('fs') 
const path = require('path') // ++++

const server = http.createServer((req, res) => {
  const { url, query } = req

  if (url === '/') {
    // 设置响应头的Content-Type是为了让浏览器以html的编码方式去加载这份资源
    res.writeHead(200, { 
      'Content-Type': 'text/html'
    })
    let content = fs.readFileSync('./index.html', 'utf8')
    res.end(content)
  } else if (url.endsWith('.js')) { // +++++++新增代码
    const p = path.resolve(__dirname, url.slice(1)) // '/main.js' ==> 'main.js'的绝对路径
    res.writeHead(200, {
      'Content-Type': 'application/javascript'
    })
    const content = fs.readFileSync(p, 'utf8')
    res.end(content)
  }

})

server.listen(8080, () => {
  console.log('listening on port 8080');
})


```

我们将 js 文件的请求都读取到资源代码返回给浏览器后：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8403efb5892e4a4a8a760da69199ef4d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2770&h=1202&s=270918&e=png&b=fefefe)

main.js 的资源请求正常了，但是现在项目还是无法运行，因为 mian.js 中有浏览器无法处理的模块路径'vue'，所以我们需要实现，当浏览器加载到来自 node_modules 中的模块时，我们告诉浏览器该模块的正常路径是什么

```
const http = require('http')
const fs = require('fs') 
const path = require('path') 

function rewriteImport(content) { // ++++新增
  return content.replace(/ from ['|"]([^'"]+)['|"]/g, function(s0, s1) { // 找到  from 'vue' 中的  'vue'
    if (s1[0] !== '.' && s1[1] !== '/') {
      return ` from '/@modules/${s1}'`
    } else {
      return s0
    }
  })
}

const server = http.createServer((req, res) => {
  const { url, query } = req

  if (url === '/') {
    // 设置响应头的Content-Type是为了让浏览器以html的编码方式去加载这份资源
    res.writeHead(200, { 
      'Content-Type': 'text/html'
    })

    let content = fs.readFileSync('./index.html', 'utf8')
    res.end(content)
  } else if (url.endsWith('.js')) {
    const p = path.resolve(__dirname, url.slice(1))
    res.writeHead(200, {
      'Content-Type': 'application/javascript'
    })
    const content = fs.readFileSync(p, 'utf8')
    res.end(rewriteImport(content))  // +++++修改
  }

})

server.listen(8080, () => {
  console.log('listening on port 8080');
})


```

再次启动后端，访问浏览器后：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c546a99f43947b69e58dde21e64e8e1~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2858&h=1130&s=299031&e=png&b=fefefe)

我们给所有的从 node_modules 中引入的模块，路径中打上一个特殊标记 `@modules` ，让浏览器知道这是个路径，因为在浏览器眼里， "./"， "/"， "../" 这种才叫路径。同时顺势浏览器便执行 main.js 中的代码，并发起了新的请求

但是

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6eda50d8b1954c3085d1025d3f766cd6~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2256&h=738&s=130071&e=png&b=fefefe)

这个路径依然不是最终的有效路径，不过现在我们可以在 node 中进行判断，如果出现请求的 url 是 `/@modules/xxx` 这种类型的，我们就去 node_modules 中为其读取源代码

```
const http = require('http')
const fs = require('fs') 
const path = require('path') 

function rewriteImport(content) { // ++++新增
  return content.replace(/ from ['|"]([^'"]+)['|"]/g, function(s0, s1) { // 找到  from 'vue' 中的  'vue'
    if (s1[0] !== '.' && s1[1] !== '/') {
      return ` from '/@modules/${s1}'`
    } else {
      return s0
    }
  })
}

const server = http.createServer((req, res) => {
  const { url, query } = req

  if (url === '/') {
    res.writeHead(200, { 
      'Content-Type': 'text/html'
    })

    let content = fs.readFileSync('./index.html', 'utf8')
    res.end(content)
  } else if (url.endsWith('.js')) {
    const p = path.resolve(__dirname, url.slice(1))
    res.writeHead(200, {
      'Content-Type': 'application/javascript'
    })
    const content = fs.readFileSync(p, 'utf8')
    res.end(rewriteImport(content))
    
  } else if (url.startsWith('/@modules/')) { // ++++++ 新增代码
  
    const prefix = path.resolve(__dirname, 'node_modules', url.replace('/@modules/', ''))
    const module = require(prefix + '/package.json').module
    const p = path.resolve(prefix, module)
    const content = fs.readFileSync(p, 'utf8')
    res.writeHead(200, {
      'Content-Type': 'application/javascript'
    })
    res.end(rewriteImport(content))
  }

})

server.listen(8080, () => {
  console.log('listening on port 8080');
})


```

> 小知识：要读取到一个公共库有效的源码，可以在该库的 package.json 文件中找到源码有效地址

比如 node_modules 中的 Vue 的源码： ![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eed616b14cb14ed7a275d9a1253ea8ad~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2758&h=1716&s=530612&e=png&b=fbfbfb)

如此一来，我们就帮浏览器解决了第三方模块源码资源请求不到的问题：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a461027eefb4409aecb71f73321b3e9~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2758&h=1564&s=427957&e=png&b=fefefe)

Vue 源码中涉及到的各种其他的源码资源也顺势被请求到了

现在还剩最后一个问题：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fad1429c11144864b335aff7e7b9c5e2~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2272&h=938&s=103268&e=png&b=fefefe)

关于 .vue 后缀的文件资源，浏览器确实请求到了，比如上图的 App.vue，但是，浏览器是读不懂 vue 的语法的这我们都知道，所以在 Response 响应中啥也看不见，就是因为浏览器拿到了代码解析不出来，它懵了，我们还得帮它读懂这份代码

众所周知，vue 是有自己的编译器的，它能将自己的 vue 独有的语法编译成 js 代码，所以我们只需要在浏览器请求 .vue 后缀的文件时用上 vue 自己的编译器就好了

```
const http = require('http')
const fs = require('fs') 
const path = require('path') 
const { URL } = require('url');
const compilerSfc = require('@vue/compiler-sfc') // +++++新增
const compilerDom = require('@vue/compiler-dom')

function rewriteImport(content) { 
  return content.replace(/ from ['|"]([^'"]+)['|"]/g, function(s0, s1) { // 找到  from 'vue' 中的  'vue'
    if (s1[0] !== '.' && s1[1] !== '/') {
      return ` from '/@modules/${s1}'`
    } else {
      return s0
    }
  })
}

const server = http.createServer((req, res) => {
  const { url } = req
  const query = new URL(req.url, `http://${req.headers.host}`).searchParams;  // ++++ node 新版本，读取get请求的参数写法

   // 省略部分代码... 
   else if (url.indexOf('.vue') !== -1) { // +++++ 返回.vue文件的js部分
    const p = path.resolve(__dirname, url.split('?')[0].slice(1))
    const { descriptor } = compilerSfc.parse(fs.readFileSync(p, 'utf8'))
    if (!query.get('type')) {
      res.writeHead(200, {'Content-Type': 'application/javascript'})
      const content = `
        ${rewriteImport(descriptor.script.content.replace('export default', 'const __script = '))} 
        import { render as __render } from "${url}?type=template" 
        __script.render = __render 
        export default __script
      `
      res.end(content)
    } else if (query.get('type') === 'template') { // 返回.vue文件的html部分
      const template = descriptor.template
      const render = compilerDom.compile(template.content, {mode: 'module'}).code
      res.writeHead(200, {'Content-Type': 'application/javascript'})
      res.end(rewriteImport(render))
    }
  }

})

server.listen(8080, () => {
  console.log('listening on port 8080');
})


```

我们都知道，vue 的组件由 tempalte，script，style，三部分构成，以上代码主要是将浏览器请求的 App.vue 中的 template 部分打造成新的请求，让浏览器再多请求一次，而 js 部分是不需要的

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dbb79ed684a041e9bb53605073089905~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2262&h=852&s=257953&e=png&b=fefefe)

> 注意 App.vue 中不要使用 setup 的语法糖，用语法糖需要额外的编译手段，上述代码没有包含该功能

到这里还剩一个 css 部分的请求，目前是这样的：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d8bc91cd2bf46048c1738b613034198~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2478&h=1012&s=121875&e=png&b=fefefe)

添加 css 的响应

```
// 省略部分代码...
else if (url.endsWith('.css')) {
    const p = path.resolve(__dirname, url.slice(1))
    const file = fs.readFileSync(p, 'utf8')
    const content = `
      const css = "${file.replace(/\n/g, '')}"
      let link = document.createElement('style')
      link.setAttribute('type', 'text/css')
      document.head.appendChild(link)
      link.innerHTML = css
      export default css
    `
    res.writeHead(200, {'Content-Type': 'application/javascript'})
    res.end(content)
  }
// 省略部分代码...


```

将 css 代码改写成 js 响应给浏览器执行，最后我们试一试运行项目

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ada6fc50bc9c465793b51a4fd4b0a075~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1894&h=836&s=283870&e=png&b=fcf1f1)

别慌！这是因为在浏览器环境中找不到 process 进程（这是 node 中的），我们去最外面的 index.html 文件中加一点声明就好了

```
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta  />
    <title>Vite + Vue</title>
  </head>
  <body>
    <div id="app"></div>

    <script> // ++++
      window.process = {
        env: {
          NODE_ENV: 'dev'
        }
      }
    </script>
    <script type="module" src="/src/main.js"></script>
  </body>
</html>



```

再次访问 http:localhost:8080 看一眼效果

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f70c3cbb75a743ee83feca3b04636011~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1056&h=570&s=56360&e=png&b=ffffff)

长这样子是因为我的 App.vue 中写了个 demo

终于，整个项目在不使用构建工具的情况下，我们自己将它运行起来了，也处理好了各个模块引入的问题

## 总结
=======

Vite 相比于 Webpack 之所以构建快是因为，Vite 借助新版本浏览器可以读懂模块化语法的特点，将项目中的模块化引入统一以一个又一个 http 请求的方式响应给浏览器，这样做的好处就是省去了 Webpack 构建过程中递归做依赖收集的耗时步骤，又因为 Vite 是开发环境的工具，绝大多数情况下我们不用不考虑兼容性，不会有人开发时还用老版本的浏览器吧 >_<

[完整代码在这里](https://link.juejin.cn?target=https%3A%2F%2Fgitee.com%2Fsnail_wn%2Fsimple-vite "https://gitee.com/snail_wn/simple-vite")