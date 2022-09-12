
笔者目前所在团队是使用 Monorepo 的方式管理**所有**的业务项目，而随着项目的增多，稳定性以及开发体验受到挑战，诸多问题开始暴露，可以明显感受到现有的 Monorepo 架构已经不足以支撑日渐庞大的业务项目。

现有的 Monorepo 是基于 yarn workspace 实现，通过 link 仓库中的各个 package，达到跨项目复用的目的。package manager 也理所当然的选择了 yarn，虽然依赖了 Lerna，由于发包场景较为稀少，基本没有怎么使用。

可以总结为以下三点：

* 通过 yarn workspace link 仓库中的 package
* 使用 yarn 作为 package manager 管理项目中的依赖
* 通过 lerna 在应用 app 构建前按照依赖关系构建其依赖的 packages

存在的问题
-----

### 命令不统一

存在三种命令

1. yarn
2. yarn workspace
3. lerna

新人上手容易造成误解，部分命令之间功能存在重叠。

### 发布速度慢

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0f4cbb24ac8d47c8b889f35c60746fb7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

如果我们需要发布 app1，则会

1. 全量安装依赖，app1、app2、app3 以及 package1 至 package6 的依赖都会被安装；
2. package 全部被构建，而非仅有 app1 依赖的 package1 与 package2 被构建。

### Phantom dependencies

> 一个库使用了不属于其 dependencies 里的 Package 称之为 Phantom dependencies（幻影依赖、幽灵依赖、隐式依赖），在现有 Monorepo 架构中该问题被放大（依赖提升）。

![](https://raw.githubusercontent.com/worldzhao/blog/master/images/monorepo-2.png)

由于无法保证幻影依赖的版本正确性，给程序运行带来了不可控的风险。app 依赖了 lib-a，lib-a 依赖了 lib-x，由于依赖提升，我们可以在 app 中直接引用 lib-x，这并不可靠，我们能否引用到 lib-x，以及引用到什么版本的 lib-x 完全取决于 lib-a 的开发者。

### NPM doppelgnger

相同版本的 Package 可能安装多份，打包多份。

假设存在以下依赖关系

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c5e17811bfac42de8cfa7edbb182898f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

最终依赖安装可能存在两种结果：

1. lib-x@^1 *1 份，lib-x@^2* 2 份
2. lib-x@^2 *1 份，lib-x@^1* 2 份

最终本地会安装 3 份 lib-x，打包时也会存在三份实例，如果 lib-x 要求单例，则可能会造成问题。

### Yarn duplicate

> [Yarn duplicate 及解决方案](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fworldzhao%2Fblog%2Fissues%2F10 "https://github.com/worldzhao/blog/issues/10")

假设存在以下依赖关系

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/51bd30880bbd4d5d9ad4c4a516ee1c30~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

当 (p)npm 安装到相同模块时，判断已安装的模块版本是否符合新模块的版本范围，如果符合则跳过，不符合则在当前模块的 node_modules 下安装该模块。即 lib-a 会复用 app 依赖的 [lib-b@1.1.0](https://link.juejin.cn?target=mailto%3Alib-b%401.1.0 "mailto:lib-b@1.1.0")。

然而，使用 Yarn v1 作为包管理器，lib-a 会单独安装一份 [lib-b@1.2.0](https://link.juejin.cn?target=mailto%3Alib-b%401.2.0 "mailto:lib-b@1.2.0")。

* [difference between npm and yarn behavior with nested dependencies #3951](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fyarnpkg%2Fyarn%2Fissues%2F3951 "https://github.com/yarnpkg/yarn/issues/3951")
* [Yarn installing multiple versions of the same package](https://link.juejin.cn?target=https%3A%2F%2Fstackoverflow.com%2Fquestions%2F49072905%2Fyarn-installing-multiple-versions-of-the-same-package "https://stackoverflow.com/questions/49072905/yarn-installing-multiple-versions-of-the-same-package")
* [yarn-deduplicate-Cleans up yarn.lock by removing duplicates.](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fatlassian%2Fyarn-deduplicate "https://github.com/atlassian/yarn-deduplicate")
* Yarn v2 supports package deduplication [natively](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fyarnpkg%2Fberry%2Fpull%2F1558 "https://github.com/yarnpkg/berry/pull/1558")

### peerDependencies 风险

Yarn 依赖提升，在 peerDependencies 场景下可能导致 BUG。

1. app1 依赖 [A@1.0.0](https://link.juejin.cn?target=mailto%3AA%401.0.0 "mailto:A@1.0.0")
2. app2 依赖 [B@2.0.0](https://link.juejin.cn?target=mailto%3AB%402.0.0 "mailto:B@2.0.0")
3. [B@2.0.0](https://link.juejin.cn?target=mailto%3AB%402.0.0 "mailto:B@2.0.0") 将 [A@2.0.0](https://link.juejin.cn?target=mailto%3AA%402.0.0 "mailto:A@2.0.0") 作为 peerDependency，故 app2 也应该安装 [A@2.0.0](https://link.juejin.cn?target=mailto%3AA%402.0.0 "mailto:A@2.0.0")

若 app2 忘记安装 [A@2.0.0](https://link.juejin.cn?target=mailto%3AA%402.0.0 "mailto:A@2.0.0")，那么结构如下

```bash
--apps
    --app1
    --app2
--node_modules
    --A@1.0.0
    --B@2.0.0
```

此时 [B@2.0.0](https://link.juejin.cn?target=mailto%3AB%402.0.0 "mailto:B@2.0.0") 会错误引用 [A@1.0.0](https://link.juejin.cn?target=mailto%3AA%401.0.0 "mailto:A@1.0.0")。

### Package 引用规范缺失

目前项目内存在三种引用方式：

1. 源码引用：使用包名引用。需要配置宿主项目的构建脚本，将该 Package 纳入构建流程。类似于直接发布一个 TypeScript 源码包，引用该包的项目需要做一定的适配。
2. 源码引用：使用文件路径引用。可以理解 “宿主在自身 src 之外的源文件”，即宿主项目源代码的一部分，而非 Package。宿主需要提供该所有依赖，在 Yarn 依赖提升的前提下达到了跨项目复用，但存在较大风险。
3. 产物引用。打包完成，直接通过包名引用产物。

### Package 引用版本不确定性

假设一个 Monorepo 中的 package1 发布至了 npm 仓库，那么 Monorepo 中的 app1 应当如何在 package.json 中编写引用 package1 的版本号？

**package1/packag.json**

```jsx
{
  "name": "package1",
  "version": "1.0.0"
}
```

**app1/package.json**

```jsx
{
  "name": "app1",
  "dependencies": {
    "package-1": "?" // 这里的版本号应该怎么写？`*` or `1.0.0`
  }
}

```

在处理 Monorepo 中项目的互相引用时，Yarn 会进行以下几步判断：

1. 判断当前 Monorepo 中，是否存在匹配 app1 所需版本的 package1；
2. 若存在，执行 link 操作，app1 直接使用本地 package1；
3. 若不存在，从远端 npm 仓库拉取符合版本的 package1 供 app1 使用。

> 需要特别注意的是：`*` 无法匹配 `prerelease 版本` 👉 [Workspace package with prerelease version and wildcard dep version #6719](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fyarnpkg%2Fyarn%2Fissues%2F6719 "https://github.com/yarnpkg/yarn/issues/6719")。

假设存在以下场景：

1. package1 此前已经发布了 `1.0.0` 版本，此时远端仓库与本地 Monorepo 中代码一致；
2. 产品同学提了一个只服务于 Monorepo 内部应用的需求；
3. package1 在 `1.0.0` 版本下迭代，无需变更版本号发布；
4. Yarn 判断 Monorepo 中的 package1 版本满足了 app1 所需版本（`*` 或 `1.0.0`）；
5. app1 顺利使用上 package1 的最新特性。

直到某天，该需求特性需要提供给外部业务方使用。

1. pacakge1 将版本改为 `1.0.0-beta.0` 并进行发版；
2. Yarn 判断当前 Monorepo 中的 package1 版本不满足 app1 所需版本；
3. 从远端拉取 `package1@1.0.0` 供 app1 使用；
4. 远端 `package@1.0.0` 已经落后 app1 先前使用的本地 `package@1.0.0` 太多；
5. 准备事故通报以及复盘。

这种不确定性，导致引用此类 package 时会经常犯嘀咕：我到底引用的是本地版本还是远端版本？为什么有时候是本地版本，有时候是远端版本？我想用上 package1 的最新内容还需要时刻保持与 package1 的版本号保持一致 ，那我干嘛用 Monorepo ？

### yarn.lock 冲突

(p)npm 支持自动化解决 lockfile 冲突，yarn 需要手动处理，在大型 Monorepo 场景下，几乎每次分支合并都会遇到 yarn.lock 冲突。

* 不解决冲突无脑 `yarn`，`yarn.lock` 会直接失效，全部版本更新到 `package.json` 最新，风险太大，失去 `lockfile` 的意义；
* 人工解决冲突往往会出现 `Git conflict with binary files`，只能使用 `master` 的提交再重新 `yarn`，流程繁琐。

[Automatically resolve conflicts in lockfile · Issue #2036 · pnpm/pnpm](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fpnpm%2Fpnpm%2Fissues%2F2036 "https://github.com/pnpm/pnpm/issues/2036")

可以发现，现有 Monorepo 管理方式缺陷过多，随着其内项目的不断增加，构建速度会越来越慢，同时程序的健壮性无法得到保证。仅凭开发人员自觉是不可靠的，我们需要一套解决方案。

推荐阅读：[node_modules 困境](https://link.juejin.cn?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F137535779 "https://zhuanlan.zhihu.com/p/137535779")

解决方案
----

### [pnpm](https://link.juejin.cn?target=https%3A%2F%2Fpnpm.io%2F "https://pnpm.io/")

> Fast, disk space efficient package packageManager

在 npm@3 之前， node_modules 的结构是干净且可预测的，因为 node_modules 中的每个依赖项都有其自己的 node_modules 文件夹，其所有依赖项都在 package.json 中指定。

```jsx
node_modules
└─ foo
   ├─ index.js
   ├─ package.json
   └─ node_modules
      └─ bar
         ├─ index.js
         └─ package.json
         
```

但是这样带来了两个很严重的问题：

1. 依赖层级过深在 Windows 下会出现问题；
2. 同一 Package 作为其他多个不同 Package 的依赖项时，会被拷贝很多次。

为了解决这两个问题，npm@3 重新思考了 node_modules 的结构，引入了平铺的方案。于是就出现了下面我们所熟悉的结构。

```jsx
node_modules
├─ foo
|  ├─ index.js
|  └─ package.json
└─ bar
   ├─ index.js
   └─ package.json

```

与 npm@3 不同，pnpm 使用另外一种方式解决了 npm@2 所遇到的问题，而非平铺 node_modules。

在由 pnpm 创建的 node_modules 文件夹中，所有 Package 都与自身的依赖项分组在一起（隔离），但是依赖层级却不会过深（软链接到外面真正的地址）。

```
-> - a symlink (or junction on Windows)

node_modules
├─ foo -> .registry.npmjs.org/foo/1.0.0/node_modules/foo
└─ .registry.npmjs.org
   ├─ foo/1.0.0/node_modules
   |  ├─ bar -> ../../bar/2.0.0/node_modules/bar
   |  └─ foo
   |     ├─ index.js
   |     └─ package.json
   └─ bar/2.0.0/node_modules
      └─ bar
         ├─ index.js
         └─ package.json
```

1. 基于非扁平化的 node_modules 目录结构，解决 Phantom dependencies。Package 只可触达自身依赖。
2. 通过软链复用相同版本的 Package，避免重复打包（相同版本），解决 NPM doppelgnger（顺带解决磁盘占用）。

可以发现，很多与包管理器相关的问题就此迎刃而解。

* [Why should we use pnpm?](https://link.juejin.cn?target=https%3A%2F%2Fwww.kochan.io%2Fnodejs%2Fwhy-should-we-use-pnpm.html "https://www.kochan.io/nodejs/why-should-we-use-pnpm.html")
* [平铺 node_modules 不是唯一的路](https://link.juejin.cn?target=https%3A%2F%2Fpnpm.io%2Fzh%2Fblog%2F2020%2F05%2F27%2Fflat-node-modules-is-not-the-only-way "https://pnpm.io/zh/blog/2020/05/27/flat-node-modules-is-not-the-only-way")

### [Rush](https://link.juejin.cn?target=https%3A%2F%2Frushjs.io%2F "https://rushjs.io/")

> a scalable monorepo manager for the web

1. 命令统一。

`rush(x) xxx` 一把梭，减少新人上手成本。同时 Rush 除了 `rush add` 以及 `rushx xxx` 等命令需要在指定项目下运行，其他命令均为全局命令，可在项目内任意目录执行，避免了在终端频繁切换项目路径的问题。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2b59b54b21f94ffc8757962eb45bbebe~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

2. 强大的依赖分析能力。

Rush 中的许多命令支持分析依赖关系，比如 `-t`(to) 参数：

```bash
rush install -t @monorepo/app1
```

该命令只会安装 app1 的依赖及其 app1 依赖的 package 的依赖，即按需安装依赖。

```bash
rush build -t @monorepo/app1
```

该命令会执行 app1 以及 app1 依赖的 package 的构建脚本。

类似的，还有 `-f`(from) 参数，可以使命令只作用于当前 package 以及依赖了该 package 的 package。

3. 保证依赖版本一致性

Monorepo 中的项目应当尽量保证依赖版本的一致性，否则很有可能出现重复打包以及其他的问题。

Rush 则提供了许多能力来保证这一点，如`rush check`、`rush add -p package-name -m` 以及 `ensureConsistentVersions`。

有兴趣的同学可以自行翻阅 Rush 的官方文档，十分详尽，对于一些常见问题也有说明。

### Package 引用规范

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e58a470fde4646d681a1e032a35b57f5~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

#### 产物引用

传统引用方式，构建完成后，app 直接引用 package 的构建产物。开发阶段可以通过构建工具提供的能力保证实时构建（如 tsc --watch）

* 优点：规范，对 app 友好。
* 缺点：随着模块增多，package 热更新速度可能变得难以忍受。

#### 源码引用

package.json 中的 `main` 字段配置为源文件的入口文件，引用该 package 的 app 需要将该 package 纳入编译流程。

* 优点：借助 app 的热更新能力，自身没有生成构建产物的过程，热更新速度快
* 缺点：需要 app 进行适配， `alias` 适配繁琐；

#### 引用规范

1. 对于项目内部使用的 packages ，称为 features，不应当向外发布，直接将 `main` 字段设置为源文件入口并配置 app 项目的 webpack，走后编译形式。
2. 对于需要对外发布的 packages，不应该也不允许引用 features，必须要有构建过程，如果需要使用源码开发增加热更新速度，可以新增一个自定义的入口字段，app 的 webpack 配置中优先识别该入口字段即可。

> 补充：rush build 命令是支持构建产物缓存的，如果 app 拆分粒度够小，可复用的包足够多，同时打包镜像支持构建产物缓存的 set 与 get，就可以做到增量构建 app。

### Workspace protocol (workspace:)

> 在 PNPM 和 Yarn 支持 Workspace 能力之前，Rush 就诞生了。 Rush 的方法是将所有软件包集中安装在 common / temp 文件夹中，然后 Rush 创建从每个项目到 common / temp 的符号链接。与 PNPM Workspace 本质上是等效的。

开启 [PNPM workspace](https://link.juejin.cn?target=https%3A%2F%2Fpnpm.io%2Fzh%2Fworkspaces "https://pnpm.io/zh/workspaces") 能力从而可以使用 workspace: 协议保证引用版本的确定性，使用了该协议引用的 package 只会使用 Monorepo 中的内容。

```jsx
{
  "dependencies": {
    "foo": "workspace:*",
    "bar": "workspace:~",
    "qar": "workspace:^",
    "zoo": "workspace:^1.5.0"
  }
}
```

推荐引用 Monorepo 内的 package 时统一使用该协议，引用本地最新版本内容，保证更改能够及时扩散同步至其他项目，这也是 Monorepo 的优势所在。

若一定要使用远端版本，需要在 `rush.json` 中配置具体 project （增加 `cyclicDependencyProjects` 配置），详见 [rush_json](https://link.juejin.cn?target=https%3A%2F%2Frushjs.io%2Fpages%2Fconfigs%2Frush_json%2F "https://rushjs.io/pages/configs/rush_json/")。

**很幸运的是 PNPM workspace 中 `workspace:*` 可以匹配 prerelease 版本** 👉 [Local prerelease version of packages should be linked only if the range is *](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fpnpm%2Fpnpm%2Fpull%2F2259 "https://github.com/pnpm/pnpm/pull/2259")

问题记录
----

### Monorepo Project Dependencies Duplicate

这个问题类似于前面提到的 Yarn duplicate，但并不是 Yarn 独有的。

假设存在以下依赖关系（将 Yarn duplicate 的例子进行改造，放在 Monorepo 场景中）

**app1 以及 package1 同属于 Monorepo 内部 project。**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a22b53c2c1f74030a23924e95da9e7a0~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

在 Rush(pnpm)/Yarn 项目中，会严格按照 Monorepo 内 project 的 package.json 所声明的版本进行安装，即 app1 安装 [lib-a@1.1.0](https://link.juejin.cn?target=mailto%3Alib-a%401.1.0 "mailto:lib-a@1.1.0")，package1 安装 [lib-a@1.2.0](https://link.juejin.cn?target=mailto%3Alib-a%401.2.0 "mailto:lib-a@1.2.0")。

此时对 app1 进行打包，则 [lib-a@1.1.0](https://link.juejin.cn?target=mailto%3Alib-a%401.1.0 "mailto:lib-a@1.1.0") 和 [lib-a@1.2.0](https://link.juejin.cn?target=mailto%3Alib-a%401.2.0 "mailto:lib-a@1.2.0") 都会被打包。

对这个结果你也许会有一些意外，但仔细想想，又很自然。

换一种方式理解，整个 Monorepo 是一个大的虚拟 project，我们所有的 project 都作为这个虚拟 project 的直接依赖存在。

```jsx
{
  "name": "fake-project",
  "version": "1.0.0",
  "dependencies": {
    "@fake-project/app1": "1.0.0",
    "@fake-project/package1": "1.0.0"
  }
}
```

安装依赖时，(p)npm 首先下载**直接依赖项**，然后再下载**间接依赖项**，并且在安装到相同模块时，判断已安装的模块版本（直接依赖项）是否符合新模块（间接依赖项）的版本范围，如果符合则跳过，不符合则在当前模块的 node_modules 下安装该模块。

而 app1 和 package1 的直接依赖关系 lib-a 是该 fake-project 的间接依赖项，无法满足上述判断条件，于是按照对应 package.json 中描述的版本安装。

解决方案：[Rush: Preferred versions](https://link.juejin.cn?target=https%3A%2F%2Frushjs.io%2Fpages%2Fadvanced%2Fpreferred_versions%2F "https://rushjs.io/pages/advanced/preferred_versions/")

Rush 可以通过手动指定 `preferredVersions` 的方式，避免两个可兼容版本的重复。这里将 Monorepo 中 lib-a 的 `preferredVersions` 指定为 1.2.0，相当于在该虚拟 project 下直接安装了指定的版本的模块，作为直接依赖项。

```jsx
{
  "name": "fake-project",
  "version": "1.0.0",
  "dependencies": {
    "@fake-project/app1": "1.0.0",
    "@fake-project/package1": "1.0.0",
    "lib-a": "1.1.0"
  }
}
```

对于 Yarn，由于 Yarn duplicate 的存在，就算在根目录指定安装确定版本的 lib-a 也是无效的。 但是依旧有两种方案可以进行处理：

1. 通过 [yarn-deduplicate](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fatlassian%2Fyarn-deduplicate "https://github.com/atlassian/yarn-deduplicate") 针对性的修改 `yarn.lock`；
2. 使用`resolutions` 字段。过于粗暴，不像 `preferredVersions` 可以允许不兼容版本的存在，不推荐。

需要谨记：在 Yarn 下消除重复依赖，也应该一个 Package 一个 Package 的去进行处理，小心使得万年船。

1. 对于存在副作用的公共库，版本最好保持统一；
2. 对于其他的体积小（或支持按需加载）、无副作用的公共库，重复打包在一定程度上可以接受的。

### prettier

由于根目录不再存在 node_modules，故需要每个项目安装一个 `prettier` 作为 devDependency 并编写 `.prettierrc.js` 文件。

本着偷懒的原则，根目录新建 `.prettierrc.js`（不依赖任何第三方包），全局安装 `prettier` 解决该问题。

### eslint

先看一个场景，若在项目中使用 `eslint-config-react-app`，除了需要安装 `eslint-config-react-app`，还需要安装一系列 peerDependencies 插件。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d8e05c0db8be430ea65d141192a923fa~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aa92e0cbbcfd4bd08eb3e313f661c865~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

为什么 `eslint-config-react-app` 不将这一系列插件作为 dependencies 内置，而是作为 peerDependencies？使用者并不需要关心预设配置内引用了哪些插件。

具体讨论可以查看该 issue，里面有相关问题的讨论： [Support having plugins as dependencies in shareable config #3458](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Feslint%2Feslint%2Fissues%2F3458 "https://github.com/eslint/eslint/issues/3458") 。

总而言之：和 eslint 插件的具体查找方式有关，如果因为依赖提升失败（多版本冲突），导致需要的插件被装在了非根目录 node_modules 下，就可能产生问题，而用户自行安装 peerDependencies 可以保证不会出现该问题。

当然，我们也发现一些开源的 `eslint` 预设配置不需要安装 peerDependencies，这些预设利用了 yarn 和 npm 的扁平 node_modules 结构，也就是依赖提升，装的包都被提升至根目录 node_modules，故可以正常运作。即便如此，在基于 Yarn 的 Monorepo 中，依赖一旦复杂起来，就可能出现插件无法被查找到的情况，能够正常运转就像一个有趣的巧合。

在 Rush 中，不存在依赖提升（提升也不一定靠谱），装一系列的插件又过于繁琐，则可以通过打[补丁](https://link.juejin.cn?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2F%40rushstack%2Feslint-patch "https://www.npmjs.com/package/@rushstack/eslint-patch")的方式绕过。

### git hooks

通常会在项目中使用 `husky` 注册 `pre-commit` 和 `commit-msg` 钩子，用于校验代码风格以及 commit 信息。

很明显，在 Rush 项目的结构下，根目录是没有 node_modules 的，无法直接使用 `husky`。

我们可以借助 [rush init-autoinstaller](https://link.juejin.cn?target=https%3A%2F%2Frushjs.io%2Fpages%2Fcommands%2Frush_init-autoinstaller%2F "https://rushjs.io/pages/commands/rush_init-autoinstaller/") 的能力来达到一样的效果，本节主要参考官方文档 [Installing Git hooks](https://link.juejin.cn?target=https%3A%2F%2Frushjs.io%2Fpages%2Fmaintainer%2Fgit_hooks%2F "https://rushjs.io/pages/maintainer/git_hooks/") 以及 [Enabling Prettier](https://link.juejin.cn?target=https%3A%2F%2Frushjs.io%2Fpages%2Fmaintainer%2Fenabling_prettier%2F "https://rushjs.io/pages/maintainer/enabling_prettier/") 的内容。

```bash
# 初始化一个名为 rush-lint 的 autoinstaller

$ rush init-autoinstaller --name rush-lint

$ cd common/autoinstallers/rush-lint

# 安装 lint 所需依赖

$ pnpm i @commitlint/cli @commitlint/config-conventional @microsoft/rush-lib eslint execa prettier lint-staged

# 更新 rush-lint 的 pnpm-lock.yaml

$ rush update-autoinstaller --name rush-lint

```

在 `rush-lint` 目录下新增 `commit-lint.js` 以及 `commitlint.config.js`，内容如下

**commit-lint.js**

```jsx
const path = require('path');
const fs = require('fs');
const execa = require('execa');

const gitPath = path.resolve(__dirname, '../../../.git');
const configPath = path.resolve(__dirname, './commitlint.config.js');
const commitlintBinPath = path.resolve(__dirname, './node_modules/.bin/commitlint');

if (!fs.existsSync(gitPath)) {
    console.error('no valid .git path');
    process.exit(1);
}

main();

async function main() {
    try {
        await execa('bash', [commitlintBinPath, '--config', configPath, '--cwd', path.dirname(gitPath), '--edit'], {
            stdio: 'inherit',
        });
    } catch (\_e) {
        process.exit(1);
    }
}
```

**commitlint.config.js**

```jsx
const rushLib = require("@microsoft/rush-lib");

const rushConfiguration = rushLib.RushConfiguration.loadFromDefaultLocation();

const packageNames = [];
const packageDirNames = [];

rushConfiguration.projects.forEach((project) => {
  packageNames.push(project.packageName);
  const temp = project.projectFolder.split("/");
  const dirName = temp[temp.length - 1];
  packageDirNames.push(dirName);
});
// 保证 scope 只能为 all/package name/package dir name
const allScope = ["all", ...packageDirNames, ...packageNames];

module.exports = {
  extends: ["@commitlint/config-conventional"],
  rules: {
    "scope-enum": [2, "always", allScope],
  },
};
```

注意：此处不需要新增 `prettierrc.js`（根目录已存在） 以及 `eslintrc.js`（各项目已存在）。

根目录新增 `.lintstagedrc 文件`

**.lintstagedrc**

```jsx
{
  "{apps,packages,features}/**/*.{js,jsx,ts,tsx}": [
    "eslint --fix --color",
    "prettier --write"
  ],
  "{apps,packages,features}/**/*.{css,less,md}": ["prettier --write"]
}
```

完成了相关依赖的安装以及配置的编写，我们接下来将相关命令执行注册在 `rush` 中。

修改 `common/config/rush/command-line.json` 文件中的 `commands` 字段。

```jsx
{
  "commands": [
    {
      "name": "commitlint",
      "commandKind": "global",
      "summary": "Used by the commit-msg Git hook. This command invokes commitlint to lint commit message.",
      "autoinstallerName": "rush-lint",
      "shellCommand": "node common/autoinstallers/rush-lint/commit-lint.js"
    },
    {
      "name": "lint",
      "commandKind": "global",
      "summary": "Used by the pre-commit Git hook. This command invokes eslint to lint staged changes.",
      "autoinstallerName": "rush-lint",
      "shellCommand": "lint-staged"
    }
  ]
}
```

最后，将 `rush commitlint` 以及 `rush lint` 两个命令分别与 `commit-msg` 以及 `pre-commit`钩子进行绑定。 `common/git-hooks` 目录下增加 `commit-msg` 以及 `pre-commit` 脚本。

**commit-msg**

```bash
#!/bin/sh

node common/scripts/install-run-rush.js commitlint || exit $? #++
```

**pre-commit**

```bash
#!/bin/sh

node common/scripts/install-run-rush.js lint || exit $? #++
```

如此，便完成了需求。

### 避免全局安装 eslint 以及 prettier

经过上一节的处理，在 `rush-lint` 目录下安装了 `eslint` 以及 `prettier` 后，我们便无需全局安装了，只需要配置一下 VSCode 即可。

```jsx
{
  // ...
  "npm.packageManager": "pnpm",
  "eslint.packageManager": "pnpm",
  "eslint.nodePath": "common/autoinstallers/rush-lint/node_modules/eslint",
  "prettier.prettierPath": "common/autoinstallers/rush-lint/node_modules/prettier"
  // ...
}
```

附录
--

### 常用命令

<table><thead><tr><th>yarn</th><th>rush(x)</th><th>detail</th></tr></thead><tbody><tr><td>yarn install</td><td>rush install</td><td>安装依赖</td></tr><tr><td>yarn upgrade</td><td>rush update</td><td>rush update 安装依赖，基于 lock 文件<br>rush update --full 全量更新到符合 package.json 的最新版本</td></tr><tr><td>yarn add package-name</td><td>rush add -p package-name</td><td>yarn add 默认安装版本号为 ^ 开头，可接受小版本更新<br>rush add 默认安装版本号为 ~ 开头，仅接受补丁更新<br>rush add 可通过增加 --caret 参数达到与 yarn add 效果一致<br>rush add 不可一次性安装多个 package</td></tr><tr><td>yarn add package-name --dev</td><td>rush add -p package-name --dev</td><td>-</td></tr><tr><td>yarn remove package-name</td><td>-</td><td>rush 不提供 remove 命令</td></tr><tr><td>-</td><td>rush build</td><td>执行文件存在变更（基于 git）的项目的 build 脚本<br>rush build -t @monorepo/app1 表示只构建 @monorepo/app1 及其依赖的 package<br>rush build -T @monorepo/app1 表示只构建 @monorepo/app1 依赖的 package，不包含其本身</td></tr><tr><td>-</td><td>rush rebuild</td><td>默认执行所有项目的 build 脚本</td></tr><tr><td>yarn xxx(自定义脚本)</td><td>rushx xxx(自定义脚本)</td><td>yarn xxx 执行当前目录下 package.json 中的 xxx 脚本 (npm scripts)<br>rushx xxx 同理。可以直接执行 rushx 查看当前项目所支持的脚本命令。</td></tr></tbody></table>

### 工作流

```bash
# 从 git 拉取最新变更
$ git pull

# 更新 NPM 依赖
$ rush update

# 重新打包 @monorepo/app1 依赖的项目（不含包其本身）
$ rush rebuild -T @monorepo/app1

# 进入指定项目目录
$ cd ./apps/app1

# 启动项目 ​
$ rushx start # or rushx dev

```

### 参考文章

* [Rush.js](https://link.juejin.cn?target=https%3A%2F%2Frushjs.io%2F "https://rushjs.io/")
* [node_modules 困境](https://link.juejin.cn?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F137535779 "https://zhuanlan.zhihu.com/p/137535779")
* [Why should we use pnpm?](https://link.juejin.cn?target=https%3A%2F%2Fwww.kochan.io%2Fnodejs%2Fwhy-should-we-use-pnpm.html "https://www.kochan.io/nodejs/why-should-we-use-pnpm.html")
* [平铺 node_modules 不是唯一的路](https://link.juejin.cn?target=https%3A%2F%2Fpnpm.io%2Fzh%2Fblog%2F2020%2F05%2F27%2Fflat-node-modules-is-not-the-only-way "https://pnpm.io/zh/blog/2020/05/27/flat-node-modules-is-not-the-only-way")
