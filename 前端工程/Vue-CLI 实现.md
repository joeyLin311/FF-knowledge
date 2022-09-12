---
date created: 2022-07-01 16:35
---

## 方案对比

=======

虽然项目也没啥受重视的，但项目流程该有还是要有的。首先通过安利拉了俩个成员入伙。简单跟大家开会沟通了一下：

1. 市面主流方案调研

---

首先，提到 react 脚手架，必然离不开在 react 社区占据主流的`create-react-app`, 但出乎意料的被大家第一时间就排除了。主要是因为它配置不够灵活，对`webpack`的封装太死，这让它在应对复杂项目的时候，有点力不从心，而且自身提供的功能虽然说很通用但是也足够简陋。我猜这时候一定会有人会说，你可以`eject`一下，把源码暴露出来啊？但这样的话，我使用它的意义就没有了，而且基于它的源码二次开发, 成本肯定低不了。

接下来，我们把目标转向了`vue-cli`, 没想到获得了大家的一致好评。我认为它最优秀的地方是于: 在功能的通用性及灵活性，这两个指标上取得了一个很好的平衡。这点是与`vue.js`框架的哲学是高度一致的，所以与`vue`结合起来开发项目，会有一种行云流水的感觉，而且大家一致对它的`cli`（交互命令行工具）喜爱有加，就是下边这个东东，用过的人一定印象深刻。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/146cad965c1c448d8f293140eda6772f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

那么，我们的目标也就有了，开发一款使用方式类似于`vue-cli`的 react 脚手架。之前对`@vue/cli`的认识只停留在使用层面，所以需要先对它的实现了解一波。正所谓，知自知彼，才好实现（抄）嘛。

2. @vue/cli 源码分析

---

我 clone 的是 [4.5.14](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue-cli%2Ftree%2Fdev%2Fpackages%2F%2540vue%2Fcli "https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli") 的版本，5.0 新版本起笔时，而且还处在 beta 版阶段，略过。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1e6832120c8f4755bce111b845b1ff5e~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

整体的代码是使用`lerna + yarn-workspace`维护的，而且与 vue 正式功能有关的包都包含在了`@vue`这个 npm 域（有人也称为 npm 组织）下。

如上图所示，完整的`@vue/cli`功能主要是三部分组成的：

- `@vue/cli` 负责命令行参数收集
- `@vue/cli-service` 这个包是启动 vue 的引擎及核心，承载 webpack 配置。
- `@vue/plugin-xxx` vue-cli 的插件，一个插件对应一个功能，与上边用户自定义 feature 选项一一对应。

cli 与核心功能包分离是一种主流的拆包方式`webpack-cli & webpack`,`@babel/cli & @babel/core`皆是如此实现的，属于程序设计上的分层架构，cli（上层）需要依赖核心服务（下层），但核心服务（下层）并不依赖 cli（上层），依然可以独立运行。

阅读源码第一步，先从`package.json`入手。这种通过命令使用的包，都是通过`package.json`的 [bin](https://juejin.cn/post/6844903870578032647#heading-15 "https://juejin.cn/post/6844903870578032647#heading-15") 字段来实现。

```jsx
  "bin": {
    "vue": "bin/vue.js"
  },
```

顺藤摸瓜，打开`bin/vue.js`，这里 这里使用`Commander`这个命令行解析工具，注册了一些全局命令，这里只注意`create`这个命令就 OK。

在执行全局的`vue create xxx`命令时，运行的就是这个文件。 ![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2deb0d09f2f2431aafe91d05613c352e~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?) 当`vue create xxx`命令触发时，会执行`../lib/create`这个文件，会把项目名称`name`传进去，继续追踪`../lib/create.js`。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/153ab7f7d2364568b784e469b5565e2e~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

在`create`主函数中。首先，拼一个将要创建项目的目标路径`targetDir`（这里大部分情况是，敲`vue create`命令时的所在目录 + 传进来的项目`name`）。

紧接着，是对磁盘已存在同名项目的处理。接下来是重点,`new Creator` 这个类正式开启了创建项目的主流程。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa678fcab86f4db9bd802e969d608e8d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

找到这个`Creator`类，可以发现它继承了 node 的`EventEmiter`这个事件处理类，这点是为了实现它的整个插件机制，这类似于在编写 webpack 的插件过程中，需要订阅一些内部派发的一些 hook，实现基于发布订阅的事件模式，下边会细细讲解这块。

往下就需要开启断点调试了，帮助将它的整个运行流程看的更清楚些。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cd2544344992438e98a15b26bf93dde8~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

左侧红框中的交互选项是不是时曾相似，没错，它对应的交互界面，在上边出现过。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/68950e66520f4098b0dc3f72674bb625~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

vue 为了方便用户选择，预设了一些功能集合，`Creator`类的`constuctor`里只是初始化一些属性变量，核心的是紧接着调用的`create`实例方法。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dd18f9d9b916455c9a88c4404c8b0407~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

调用`promptAndResolvePreset` ，会弹出预设选项界面，✅ 一个默认选项，继续；

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f8b9ce54f1624e60bbcd3caac5fe0c2c~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

左侧可以看到`presets`这个表示所选预设的数据里，已经包含了预设里包含的`babel、 eslint`等插件。

换一个预设，选择手动呢？如下图：得到也是相同的数据结构。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7be6c48a7ea54c54b6c29514cb4cd03f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

接下来，就是给`presets`里的`plugins`数组里的每一个插件，初始化一些默认选项

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/06dc57cbffc64ec2b3ff8f93a60ad7c7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

接下来，初始化生成 `package.json` 需要的数据。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a78785cbca4a4c389e6d8201a3a154cf~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

将刚才预设里包含的插件，插入`devDependencies`开发依赖中。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6f3f5a5124cb40bea9710e240e1c7459~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

插入成功, 调试栈里已经可以看到包含了 3 个开发依赖

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e5d333a641a4196971b94bdf3e10c60~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

接下来，将`package.json`写入磁盘。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8562932fa31c47989c820448d821c047~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

执行`npm install`，安装依赖。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c8830fd7281a433892e9e9e3258407d3~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

接着，又做了些：写入`npmrc、yarnrc`文件，初始化`git`仓库等一些无关紧要的事情，快速掠过。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9436af55901e409cbceb88f5b554b553~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

重点来了，下面就要使用刚安装好的插件，生成项目模版了。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2fc1f993d8b847aa96dcec14e13f49df~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

`this.resolvePlugins(preset.plugins, pkg)`这个方法很关键，用来引入插件。但在介绍这个方法之前，我想先聊一聊框架的插件机制以及它在`vue/cli`中的设计与实现。

### 插件机制

就是可以将一些可选配或者用户需要自定义的功能封装起来，**按需插入**使用，这样的设计优势是：可以拆分代码复杂度和功能耦合度，并且极大的增加了框架的**可扩展性**。

插件，首先要定好怎么插。这也是插件机制最重要的一点，也就是**框架开发者与插件开发者需要约定好一套插件接入的固定方式**。体现在 vue/cli 中，这种约定如下图所示。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/616580a241db4a78a0897a446f2bb3ad~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

每一个插件都有一个`generator`文件夹，下边必须存在一个`index.js`，然后`index.js`需要导出一个函数，在函数体里可以调用外部主体注入的一些工具方法，比如，

- `api.injectImport()` 在项目里插入一个模块导入。
- `api.extendPackage()` 扩展项目的 package.json
- `api.render('./template')` 使用 ejs 渲染模版文件（条件编译）

再回来看上边的`this.resolvePlugins(preset.plugins, pkg)`方法，这里取到的 apply，其实就是插件约定导出的那个函数，但这里需要注意的是，只是**暂时将 apply 方法保存起来**，并不会调用。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/96764c9d852a453eada22d9cd6903a6a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

原`plugins`，经过此方法处理后，就转换为了新的包含具体插件功能的`plugins`。\
{id: options} => [{ id, apply, options }]

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0416bf80894947acaeb8680c880c0078~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

接下来，将整理好的 plugins 传入`Generator`这个类中，开始了最后一步，也是最实质的一步：**生成项目**。

可以这样说，之前一大堆的参数收集、整理、引入插件等准备工作都是为了最终的生成这一步。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/16c4e697af3d475998f3f2aaa7158d73~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

`new` 之后，紧接着调了`generate`这个实例方法，所以接下来，重点看下这个方法的实现

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aee0794391ea4ef0a6598cfa0620a0e7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

方法执行，首先调用了`this.initPlugins()`，这个方法中最重要的就是遍历执行所有插件默认的 apply 方法（即前边提到的插件机制中约定默认导出的那个函数），`apply`第一个参数是`api`, 包含了`render`渲染 ejs 模版等这些能力, 而这些能力都来自`GeneratorAPI`这个类，注意第二个参数，会把当前`this`传入。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9ee1add1598a456e9d54304d663b11af~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

接下来，继续看`GeneratorAPI`的实现。先排除下`@vue/cli-service`这个包，因为这个包虽然存在于`plugins`数组中，但它却很特殊，它属于核心服务，严格意义上并不属于插件的范畴，不需要当插件处理，只需参与安装就行，这里我猜只是让数据结构尽量保持简单，用时候过滤一下也没有很麻烦。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/09432debba6f4df186a459960ea59567~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

`GeneratorAPI`的整体实现采用了一种中间件的设计，其上的 `_injectFileMiddleware` 方法，会将 api 上的方法调用以 push 一个函数的方式先存起来，暂不执行。

拿上边讲到的`api.render()`渲染模版方法举例，在调用插件导出的`apply`方法时，确实会触发`api.render()`调用, 但却并不会真正的使用`ejs`编译模版文件，而是将他们暂存到`this.generator.fileMiddlewares`这个中间暂存数组中。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cadf0a885b2b406a8bd774b9e3a3ea36~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/429786c6d2374263a9cd63dfba2f5237~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

那么`api.render()`这些方法真正的起作用是在哪里呢？

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f9a1b2da69f44091bfa127ac4610e9b2~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

打开紧接着执行的 `this.resolveFiles` 方法，真相就在这里。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/238e22dfaa9e45d9b91c98eede07e451~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

用`for of`串行调用每一个`middleware`函数，得到最终需要生成的文件内容

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4b0a3c3503c942eeb813afa8b4fedfb7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

此刻，`this.files` 就保存了完整的**文件路径**到**文件内容**的映射, 如上图左侧所示。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f06b3a79953c4e86ad935fbc84061b82~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

接下来，是一些常规操作。`sortPkg` 可以对`package.json`的依赖整理下顺序，满足下一些强迫症 er 的感受。

接下来就可以根据`this.files`，调用 `writeFileTree` 往磁盘愉快的写文件了。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/46b5d2bf2d664998a6f44df836e4a45c~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

最终生成的目录结构如下，流程介绍完毕。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/41fb1a6d24d74ceab4d1f8e7147b81c2~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

3. 确定功能及目标

---

前边的对`@vue/cli`的详细分析，可以总结为关键的三点

- 1. 采用**命令行界面**的进行功能选择
- 2. `cli` 与 `核心服务` @vue/cli-service（webpack 配置）采用分层设计，**独立发包**
- 3. **插件机制**，按需生成不同的功能模版文件

这样的设计对一个面向所有 vue 开发者的脚手架而言是必要的，因为它需要具备非常大的灵活性。但对一款只满足特定团队的脚手架来说，是过于复杂的，而且开发成本也会很高，光是将各种功能插件拆分就需要非常大的工作量。其次，团队的项目开发，最重要的就是统一与规范，很多配置及功能都可以是默认内置的，比如 eslint、babel、css 处理器，这些都是团队项目开发中必备的功能，所以**配置的多样性及灵活性并不是首要考虑因素**。所以，第三点插件机制可以舍去。

cli 与 webpack 配置分离，这样可以做到两者的**独立版本更新**，有利于用户项目的持续维护，但这样做的成本在于需要自己定义一套与`webpack`配置大不一样的配置约定出来，这个在`@vue/cli-service` 中是使用`vue.config.js`作为配置文件, 其他主流的脚手架比如`create-react-app`都是走的这个路子。这样的话，就还需要有一份详细的脚手架配置的文档，通过配置脚手架去配置 webpack，这就导致我们已掌握的 webpack 配置知识无用武之地，vue 中采用的`chainWebpack`是个解决办法，但究竟它有多难用，只有用过的人才能体会到。我们也许只需要一款透明的`webpack`配置，所有的`rules、plugins`都是可以直接修改的，这样就不需要文档，只需要会 webpack 配置就能搞定一切，而且，这份 webpack 配置在我们的团队就是现成的，所以上边提到的第二点也不需要。

所以，我们需要借鉴的地方就主要在于交互命令行工具，还有按需进行 ejs 模版编译。同时结合我们团队的项目特点进行功能提炼，经过小伙伴的讨论，主要有以下几点选择：

### （1） 移动端还是 PC 端?

PC 端会内置基于`antd`的布局模版，移动端会开启`px2rem`适配，html 里内联 `flexible.js`脚本。

### （2） 生成单页 or 多页模版?

`MPA`与`SPA`的需求在我们的项目场景中都是存在的，所以在脚手架层面去区分两者还是很有意义的。

### （3） 状态管理库、react-router 版本按需安装

## 功能开发

====

### 初始化项目

- monorepo

它相比单仓库的优势是在于有利于多个 npm 之间的高效联调及`node_modules`磁盘空间共享 由于 lerna 的依赖提升对依赖版本号要求过于严格，而且缺少很多`yarn-workspace`独有的功能 (特别是对 bin、module 的本地软链处理)。目前主流的实践是，利用`yarn`的`workspace`做依赖管理，`lerna`做 npm 包的自动版本管理及发包。 这里采用`monorepo`的原因是，后续可能会基于这个脚手架开发组件库或者 webpack-plugin 等项目，方便他们之间的联调。

```bash
npm i -g lerna
lerna init
```

**lerna.json**

```bsh
{
  "packages": [
    "packages/*"
  ],
  "version": "0.1.6",
  "npmClient": "yarn",
  "useWorkspaces": true
}


```

package.json

```json
   ...
  "workspaces": [ 
    "packages/*"
  ],
  ...


```

创建 cli 包

```bash
lerna create react-booster-cli
```

初始化下项目

```bash
yarn install 
```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d3fe0923143d429ab09289ed86da400d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

接下来就是实现 cli 的功能，首先在分析 vue/cli 源码时提到了，全局包用到的全局命令，是通过`package.json`的`bin`字段实现的，咱们也一样。

```json
booster/packages/react-booster-cli/package.json
  "bin": {
    "booster": "bin/booster.js"
  },

```

接下来创建`./bin/booster.js`文件

```jsx
#!/usr/bin/env node
// 命令行解析工具
const program = require('commander');

program
  .version(require("../package").version)
  .usage("<command> [options]");
  
  
  program.command("create <project-name>")
  .description("创建一个新的项目")
  .action((projectName)=>{
    require('../lib/create')(projectName)
  })

  program.parse(process.argv);  
```

`#!/usr/bin/env node`这句非常重要，声明此文件需要使用`node`程序来执行。里边用到了`commander`这个命令行参数解析库，安装一下

```bash
yarn workspace react-booster-cli add commander
```

现在就可以测试跑通下咱们的流程了, 在 booster 根目录执行

```bash
npx booster
```

出现以下界面就算成功了

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/179d6c697f194349814340f6103909fa~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

也许有人会好奇运行成功的原因，我这里简单解释下，首先，`npx xxx`会先去当前目录下的`node_modules/.bin/`目录下找 xxx 文件。很显然，是存在的。这是为什么呢? 虽然声明了全局命令，但写的 cli 包，一没发包，二没安装。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7a8c5ad2dbc84a07bfbca1d4a37fe29d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

其实这是`yarn install` 的一个附带（超好用的）功能，准确说是通过使用`workspace`，`yarn install`会自动的帮忙解决安装和 link 问题，原理是在`node_modules/.bin`目录下创建软链（类似于 win 上的文件快捷方式），链接到了`packages/react-booster-cli/bin/booster.js`。

接下来执行 `create` 命令会走 comander 的`action`，将项目名传到 create 文件里的 create 方法中，这与 vue/cli 是一致的

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c6660fbe402f4a93bd194f0a49058d06~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

### 功能实现

#### 实现 create 方法

这里需要很多做 cli 常用的一些工具包，每个工具的用途下边都有注释说明，很多其实都是用来使命令行更加美观的。也许 npm 生态里大部分都是前端开发者，像这样装饰命令行的包种类和数量都格外丰富。比如可以使用`ora`、`chalk`很方便实现一些命令行加载效果、颜色字体以及进度条，增加命令行的用户体验。

**`lib/create.js`**

```jsx
const path = require("path");
const fs = require("fs");
// 检测目录是否存在
const exists = fs.existsSync;
// 删除文件
const rm = require("rimraf").sync;
//询问cli输入参数
const ask = require("./ask");
// 命令行交互工具
const inquirer = require("inquirer");
// 命令行loading
const ora = require("ora");
// 输出增色
const chalk = require("chalk");
// 检测版本
const checkVersion = require("./check-version");

const generate = require("./generate");

const { writeFileTree } = require("./util/file");
const runCommand = require("./util/run");

// loading
const spinner = ora();
async function create(projectName) {
  const cwd = process.cwd(); //当前运行node命令的目录
  const projectPath = path.resolve(cwd, projectName);
  // 假如当前已存在同名项目，询问是否覆盖
  if (exists(projectPath)) {
    const answers = await inquirer.prompt([
      {
        type: "confirm",
        message: "Target directory exists. Do you want to replace it?",
        name: "ok",
      },
    ]);
    if (answers.ok) {
      console.log(chalk.yellow("Deleting old project ..."));
      rm(projectPath);
      await create(projectName);
    }
  } else {

    // 收集用户输入选项
    const answers = await ask();
    spinner.start("check version");
    // 检测版本
    await checkVersion();
    spinner.succeed();
    console.log(`✨  Creating project in ${chalk.yellow(projectPath)}.`);
    // console.log(answers);
    // 更新 package.json
    const pkg = require("../template/package.json");

    // 生成项目配置文件，app.config.json
    const appConfig = {};
    const { platform, isMPA, stateLibrary,reactRouterVersion } = answers;
    if (platform === "mobile") {
      pkg.devDependencies["postcss-pxtorem"] = "^6.0.0";
      pkg.dependencies["lib-flexible"] = "^0.3.2";
    } else if (platform === "pc") {
      pkg.dependencies["antd"] = "latest";
    }
    pkg.dependencies[stateLibrary] = "latest";
    if (reactRouterVersion === "v5") {
      pkg.devDependencies["react-router"] = "5.1.2";
    } else if (reactRouterVersion === "v6") {
      pkg.dependencies["react-router"] = "^6.x";
    }

    appConfig.platform = platform;

    spinner.start("rendering template");
    const filesTreeObj = await generate(answers,projectPath);
    spinner.succeed();
    spinner.start("🚀 invoking generators...");
    await writeFileTree(projectPath, {
      ...filesTreeObj,
      "package.json": JSON.stringify(pkg, null, 2),
      "app.config.json": JSON.stringify(appConfig, null, 2),
    });
    spinner.succeed();
    console.log(`🗃  Initializing git repository...`)
    await runCommand('git init')
    
    console.log();
    console.log(
      `🎉  Successfully created project ${chalk.yellow(projectName)}.`
    );
    console.log(
        `👉  Get started with the following commands:\n\n` +
          chalk.cyan(` ${chalk.gray("$")} cd ${projectName}\n`) +
          chalk.cyan(` ${chalk.gray("$")} npm install or yarn\n`) +
          chalk.cyan(` ${chalk.gray("$")} npm run dev`)
      );
    console.log();
     
    
  }
}

module.exports = (...args) => {
  return create(...args).catch((err) => {
    spinner.fail("create error");
    console.error(chalk.red.dim("Error: " + err));
    process.exit(1);
  });
};


```

#### 检测版本

`lib/check-version.js`

```jsx
const request = require('request')
const semver = require('semver')
const chalk = require('chalk')
const packageConfig = require('../package.json')


module.exports = function checkVersion() {
    return new Promise((resolve,reject)=>{
        if (!semver.satisfies(process.version, packageConfig.engines.node)) {
            return console.log(chalk.red(
              `  You must upgrade node to >= ${packageConfig.engines.node} .x to use react-booster-cli`
            ))
          }
          request({
            url: 'https://registry.npmjs.org/react-booster-cli',
          }, (err, res, body) => {
            if (!err && res.statusCode === 200) {
              const latestVersion = JSON.parse(body)['dist-tags'].latest
              const localVersion = packageConfig.version
              if (semver.lt(localVersion, latestVersion)) {
                console.log()
                console.log(chalk.yellow('  A newer version of booster-cli is available.'))
                console.log()
                console.log(`  latest:     ${chalk.green(latestVersion)}`)
                console.log(`  installed:  ${chalk.red(localVersion)}`)
                console.log()
              }
              resolve()
            }else{
              reject()
            }
          })
    })
}



```

#### 命令行功能选择

核心是利用`inquirer`这个包，来实现命令行交互界面，这个包非常强大，提供了单选、多选、输入框等交互方式，活脱脱一个命令行`form`啊!

lib/ask.js

```jsx
const { prompt } = require('inquirer');//生成命令行交互界面

const questions = [
  {
    name: 'platform',
    type: 'list',
    message: '您的web应用需要运行在哪个端呢?',
    choices: [{
      name: 'PC端',
      value: 'pc',
    }, {
      name: '移动端',
      value: 'mobile',
    }]
  },
  {
    name: 'isMPA',
    type: 'list',
    message: '生成单页or多页模版?',
    choices: [{
      name: '单页（SPA）',
      value: false,
    }, {
      name: '多页（MPA）',
      value: true,
    }]
  },
  {
    name: 'stateLibrary',
    type: 'list',
    message: '您希望安装的状态管理库是?',
    choices: [{
      name: 'mobx',
      value: 'mobx',
    }, {
      name: 'redux',
      value: 'redux',
    }]
  },
  {
    name: 'reactRouterVersion',
    type: 'list',
    message: '选择react-router版本',
    choices: [{
      name: 'v5（推荐）',
      value: 'v5',
    }, {
      name: 'v6（对hook支持度较好,但api不够稳定）',
      value: 'v6',
    }]
  },
]

module.exports = function ask () {
  return prompt(questions)
}


```

#### 生成项目文件

`lib/generate.js`

```jsx
const { isBinaryFileSync } = require("isbinaryfile");
const fs = require("fs");
const ejs = require("ejs");
const path = require("path");

/**
 * @name 渲染文件
 * @param {*} filePath 文件路径
 * @param {*} ejsOptions ejs注入数据对象
 * @returns 文件内容
 */
function renderFile(filePath, ejsOptions = {}) {
  // 二进制文件直接返回
  if (isBinaryFileSync(filePath)) {
    return fs.readFileSync(filePath);
  }
  const content = fs.readFileSync(filePath, "utf-8");

  //src目录下需要经过ejs动态编译
  if (/[\\/]src[\\/].+/.test(filePath)) {
    return ejs.render(content, ejsOptions);
  }
  // 其他文件，比如webpack的配置文件，直接读取返回
  return content;
}

/**
 * @name 生成项目文件
 * @param {*} answers 收集的问题
 * @returns 文件树 eg { '/path/a/b' : 文件内容 }
 */
async function generate(answers, targetDir) {
  const globby = require("globby");

  // 匹配脚手架文件夹所有文件
  const fileList = await globby(["**/*"], {
    cwd: path.resolve(__dirname, "../template"),
    gitignore: true,
    dot: true,
  });
  const { isMPA } = answers;
  // ejs注入的模版变量
  const ejsData = {
    ...answers,
    projectDir: targetDir,
    pageName:'index'
  };
  // 生成文件树对象
  const filesTreeObj = {};
  fileList.forEach((oriPath) => {
    let targetPath = oriPath;
    const absolutePath = path.resolve(__dirname, "../template", oriPath);
  
    if (isMPA && /^src[\\/].+/.test(oriPath)) {
      // 针对多页场景，生成多页面模版
      const [dir, file] = oriPath.split(/[\\/]+/);
      ["index", "pageA", "pageB"].forEach((pageName) => {
        targetPath = `${dir}/pages/${pageName}/${file}`;
        filesTreeObj[targetPath] = renderFile(absolutePath, {
          ...ejsData,
          pageName,
        });
      });
    } else {
      const content = renderFile(absolutePath, ejsData);
      filesTreeObj[targetPath] = content;
    }
  });

  return filesTreeObj;
}

module.exports = generate;



```

**将命令行收集到的参数注入到 ejs 模版中**

比如在命令行通过用户交互收集到`platform`这个代表 web 平台的参数。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8c7127b65e124d0fbbe2f723f2dff370~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

ejs 渲染时， `platform = mobile`，代表选择是移动端，就在 html 模版中的 head 标签中插入`flexible.js`的脚本，PC 的话，就不需要。这样就可以将不同功能的最终生成代码区分开。 ![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/925c2180d4e74840a3815313960ecb0c~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

功能开发完毕之后，因为大部分公司都是有自己的私有 npm 库的，我这里演示下发公开包的步骤，其实都差不多。

### 发布 npm 包

```bash
npm login
lerna publish
```

发包这一步，还是挺容易踩到坑的，这里总结下我遇到的：

1. **npm 公开包的名称需要是唯一的。**

最好提前去 [npm](https://link.juejin.cn?target=https%3A%2F%2Fwww.npmjs.com%2F "https://www.npmjs.com/") 网站搜索一下，看自己将要发的包名是否已存在。或者花钱买私有域，类似于 @vue/xxx 这样的。

2. **lerna publish 重试不生效。**

运行 lerna publish 如果中途有包发布失败，再运行 lerna publish 的时候，因为 git 的 Tag 已经打上去了，所以不会再重新发布包到 NPM。

解决方法：运行 lerna publish from-git，会把当前标签中涉及的 NPM 包再发布一次，PS：不会再更新 package.json，只是执行 npm publish

3. **全局修改为淘宝 npm 源导致的问题**

- 淘宝源与官方源的同步存在延时

  ```
  npm 包已经发布成功，但因为全局设置的是淘宝源，亲测会有一定的同步延时，大概半小时到一个小时左右，所以有可能会更新不到最新的包版本或者直接首次发版成功后的一段时间内找不到包。
  ```

- 全局用淘宝源，会导致发包失败。

  ```
  因为淘宝源只能下载包，不能上传包。但是不用淘宝源，安装其他依赖又慢的不行。 解决方法： 安装依赖，统一从淘宝源拉，保证依赖安装速度。发包时借助 npm 包的 package.json 的`publishConfig`字段指定官方源，确保能发包成功。
  ```

```jsx
"publishConfig": {
	"registry": "https://registry.npmjs.org/",
	"access": "public"
 },
```

4. **devDependencies** 与 **dependencies** 的区别

   首先，在普通的业务项目中，这两者是没有任何本质区别的。也就是说在安装时，带不带 --dev，只会影响最终在`package.json`中的归类位置，最终会不会被 webpack 等的工具打包构建，只决定于是否在项目中被引用。

   但对于发到 npm 上的项目而言，这点很重要。 当用户安装你的包时，**只有生产依赖才会被一起安装**，开发依赖则不会。如果使用不当，比如不小心将一个生产依赖装入了开发依赖，安装你 npm 包的用户会运行报错，找不到 xx 模块。

## 最后

==

[npm 地址](https://link.juejin.cn?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Freact-booster-cli "https://www.npmjs.com/package/react-booster-cli")、 [github 地址](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FFEyudong%2Fbooster%2Ftree%2Fmaster%2Fpackages%2Freact-booster-cli "https://github.com/FEyudong/booster/tree/master/packages/react-booster-cli"), 欢迎大家试用， 提`issues`。这个项目是我业余时间开发的，**以上标题及故事情节也纯属虚构**。代码是完全脱敏公开的，如果你们团队也有类似的需求，可以给大家提供一个参考，这就是我最好的收获。
