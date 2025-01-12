由于 **vue3.2** 版本的发布，`<script setup>` 的实验性标志已经去掉，这说明这个语法提案已经正式开始使用，并且我个人对这个方案表示非常喜欢，其他的[更新](https://link.zhihu.com/?target=https%3A//github.com/vuejs/vue-next/blob/master/CHANGELOG.md)请自行了解。到目前为止，我认为 vue3 已经完全可以用于生产环境。在此将我的开发体验，总结至此，分享给大家。

我认为前端架构核心工作是定制一套适合当前业务需求的解决方案，从而降低需求的增加而带来的技术实现的复杂度。下面我将从 16 个方向，逐渐带领大家搭建一套属于你自己的脚手架，制定一套合理的解决方案，为项目打下良好的基础，与同伴形成合适的开发习惯。

由于篇幅问题，以讲解实现思路为主，希望大家友善发言，共同进步！

## 1. 搭建脚手架

使用 `[vue-cli](https://zhida.zhihu.com/search?q=vue-cli&zhida_source=entity&is_preview=1)` 或 `vite` ，通过一系列的配置，初始化一个开发模板，无需从零开始搭建开发环境，可以有效的提升开发效率，相信也是大多数开发者接手一个新项目所使用的一种方式。尽管官方提供的脚手架已经足够优秀，但未必是真正符合我们自己团队的使用习惯，所以从官方的基础上，开发一款属于我们自己的脚手架，能更多的提升开发效率。

### 1.1 前端脚手架应具备哪些功能？

- 减少重复的初始化工作，不需要再复制其他类似的项目删除无关代码，或从零搭建一个项目。
- 可以根据团队需求，使用简单的交互操作生成相应的目录结构和文件。
- 统一团队的开发习惯、代码风格，保证构建结果的一致性。
- 完整的使用文档，降低新人上手、开发和后期维护成本。

### 1.2 如何开发一款自己的脚手架？

提到构建[前端工程化](https://zhida.zhihu.com/search?q=%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B%E5%8C%96&zhida_source=entity&is_preview=1)中脚手架，相信大家已经看过不少文章，几年前我也曾经写过一篇关于[脚手架构建的文章](https://link.zhihu.com/?target=https%3A//juejin.cn/post/6844903607855235079)，随便搜一下关键词可以看到很多相关的文章，在这里不做太多的介绍，主要讲一些这些文章中很少提到的如何根据选项生成文件。

### 1.3 如何根据选项生成文件？

说实话我也不知道大佬们是怎么根据各种配置编译成相应的文件，这块希望大家踊跃发言，寻求一种更佳高效简洁的方式。在这里跟大家分享一下我的方案：

交互方面，搭建过脚手架的同学一定知道 [inquirer](https://link.zhihu.com/?target=https%3A//github.com/SBoudrias/Inquirer.js%23readme)，这个库可以很方便的通过交互式操作获取到我们选择的一些自定义配置参数。那么问题来了，如何通过这些配置相应的创建对应的文件呢？

这里我推荐使用 [EJS](https://link.zhihu.com/?target=https%3A//ejs.bootcss.com/) + [Prettier](https://link.zhihu.com/?target=https%3A//prettier.io/docs/en/) 生成代码，通过 [fs-extra](https://link.zhihu.com/?target=https%3A//github.com/jprichardson/node-fs-extra) 写入最终的文件。

- **EJS**

**EJS** 是一款 JavaScript 模板引擎，我们可以通过传入参数，生成对应的代码串，例如创建一个 `package.ejs` 用来生成 `package.json` 中，如果我们选择使用了 `scss` 作为 CSS 预处理器，然后将 `[sass](https://zhida.zhihu.com/search?q=sass&zhida_source=entity&is_preview=1)` 和 `stylelint-scss` 作为项目的安装依赖：

<% if (precss === 'scss') { -%>
  "sass": "1.26.5",
  "stylelint-scss": "^3.20.1",
<% } -%>

模板引擎可以帮你通过参数生成代码，它并不会限制你生成任何类型的代码文件，因为我们生成的是纯代码，最后通过读取 `.ejs` 文件对应生成相应的类型文件即可。

- **Prettier**

**Prettier** 是一款代码格式化工具，相信大家对它并不陌生。使用 EJS 生成的目的还是给开发人员阅读和编辑，所以生成的代码应该符合最终的格式要求，因为后续我们会为脚手架添加 ESLint 和 StyleLint 等工具，刚刚创建的项目里面一堆红线报错可是十分不友好的。

import prettier = require("prettier");
prettier.format(code, { parser: 'json' }))

`parser` 是 prettier 的解析器，常见的 typescript、css、less、json 等文件都可以进行格式化。

## 2. 基于 vite 的搭建基础模板

最早搭建 [vue3 脚手架](https://zhida.zhihu.com/search?q=vue3+%E8%84%9A%E6%89%8B%E6%9E%B6&zhida_source=entity&is_preview=1)的时候，我选择的用 [vue/cli](https://zhida.zhihu.com/search?q=vue%2Fcli&zhida_source=entity&is_preview=1) 搭建，因为生态不健全，有些基于 webpack 的功能无法使用，但现在 vite 生态已经比较完善了，所以重构脚手架，由 webpack 转向 vite，这一步极大的提升了开发体验。

### 2.1 创建基本模板项目

npm init vite@latest
yarn create vite
pnpm create vite

然后按照提示操作即可，vite 提供的选项很少，只有 vue 或 vue + ts，不像 vue/cli 提供那么多的配置方式，所以剩下的东西需要我们手动配置。

当然 vite 也提供了很多模板，但是我认为做加法比做减法更加容易，在众多的模板中很难找到适合我们自己的。

### 2.2 常用插件推荐

这里先简单了解几个好用的 vite 插件：

- [unplugin-vue-components](https://link.zhihu.com/?target=https%3A//github.com/antfu/unplugin-vue-components)：组件的按需自动导入。
- [vite-plugin-svg-icons](https://link.zhihu.com/?target=https%3A//github.com/anncwb/vite-plugin-svg-icons)：用于生成 svg 雪碧图。
- [vite-plugin-compression](https://link.zhihu.com/?target=https%3A//github.com/anncwb/vite-plugin-compression)：使用 gzip 或者 [brotli](https://zhida.zhihu.com/search?q=brotli&zhida_source=entity&is_preview=1) 来压缩资源。

为什么只推荐这么几个插件？因为 `vite` 对许多 `webpack` 需要安装的 `loader` 或 `plugin` 都有着天生的支持，比如 less、sass、typescript，后续会在相应的章节说明用法。

## 3. 使用 Typescript

**vue2.x 版本对 TypeScript 的支持是硬伤**，而 TypeScript 对大型项目的保障能力是被普遍认可的。这一点在 vue3.x 版本中得到了非常友好的支持。

Vite 天然支持引入 `.ts` 文件。  

这里对 tsconfig.json 做了一些修改：

```json
{
  "compilerOptions": {
    "types": ["vite/client"],
    "baseUrl": "src",
    "paths": {
      "@/*": ["./*"]
    }
  },
  "exclude": ["node_modules"]
}
```

在初期使用 [typeScript](https://zhida.zhihu.com/search?q=typeScript&zhida_source=entity&is_preview=1) 的时候，很多人都很喜欢使用 any 类型，把 typeScript 写成了 anyScript ，虽然使用起来很方便，但是这就失去了 typeScript 的类型检查意义了，当然写类型的习惯是需要慢慢去养成的，不用急于一时。  

## 4. 配置环境变量

vite 提供了两种模式：具有开发服务器的开发模式（development）和生产模式（production）。

这里我们可以建立 4 个 `.env` 文件，一个通用配置和三种环境：开发、测试、生产。

### 4.1 配置模式

NODE_ENV=development # 开发模式
NODE_ENV=production # 生产模式

- .env 通用配置，我个人喜欢把他当作项目的配置文件，例如项目的 title，此文件不对应任何模式。
- .env.development 开发环境，使用 development 模式。
- .env.staging 测试环境，因为要部署到测试服务器，或本地使用 serve 命令预览，所以使用 production 模式。
- .env.production 生产环境，因为要部署到测试服务器，或本地使用 serve 命令预览，所以使用 production 模式。

package.json 内 script 需要增加 staging 命令

```json
"script": {
  "build": "vue-tsc --noEmit && vite build",
  "staging": "vue-tsc --noEmit && vite build --mode staging",
  "serve": "vite preview --host"
}
```

### 4.2 常用的环境变量

推荐使用以下常见的三个变量：

- **VITE_APP_BASE_URL**

接口请求地址。

通常后端会区分三种环境，部署在不同的地址下。

- **VITE_APP_STATIC_URL**

静态资源地址。

静态资源我是不建议你直接放在项目中，这会导致项目仓库变得巨大。  

本地开发和测试环境我会选在使用本地搭建的静态资源服务器，你可以找后端运维的同学帮你搭建，或者你使用 http-server 在本地启动一个服务器也可以。生产环境建议上传至 OSS。

- **VITE_PUBLIC_PATH**

构建资源公共路径。

这个与 vue/cli 中的 publicPath 同理，有的时候你构建的项目并不是存放在跟路径下，例如 `http://ip:port/{项目名}`。

### 4.3 封装静态资源文件

如果你配置了 VITE_APP_STATIC_URL 静态资源环境变量，那么你需要封装以下两个东西：

- 根据环境返回实际的资源地址函数。
- 方便使用的静态资源组件。

**baseStaticUrl.ts**

// 处理静态资源链接
export default function baseStaticUrl(src = '') {
  const { VITE_APP_STATIC_URL } = import.meta.env;
  if (src) {
    return `${VITE_APP_STATIC_URL}${src}`;
  }
  return VITE_APP_STATIC_URL as string;
}

**静态资源组件**

静态资源主要有图片、音频和视频三种常见的形式。

- 通过 src 写入相对的路径，使用上述的函数来补全完整的路径，即可在不同的环境下使用不同地址的静态资源。
- 通过 type 传入图片、音频和视频的类型。
- [autoplay](https://zhida.zhihu.com/search?q=autoplay&zhida_source=entity&is_preview=1) 是解决以视频为背景的情况下，视频无法自动播放的问题。

```vue
<script lang="ts" setup>
import { computed, ref, Ref, withDefaults, onMounted, watch } from 'vue';
import { baseStaticUrl } from '@/libs/utils';
import useDevice from '@/hooks/useDevice';

interface Props {
  src?: string;
  type?: string;
  autoplay?: boolean;
}
const props = withDefaults(defineProps<Props>(), {
  src: '',
  type: 'img',
  autoplay: true,
});

const envSrc = computed(() => baseStaticUrl(props.src));
// 处理视频自动播放（解决 chrome 无法自动播放的问题）
const { deviceType } = useDevice();
const poster = computed(() =>
  deviceType.value === 'desktop' ? '' : baseStaticUrl(props.src),
);
const videoRef: Ref<HTMLVideoElement | null> = ref(null);

// 解决移动端视频无法自动播放的问题
function videoAutoPlay() {
  if (props.type === 'video' && videoRef.value !== null) {
    videoRef.value.src = baseStaticUrl(props.src);
  }
  if (props.autoplay && videoRef.value) {
    videoRef.value.oncanplay = () => {
      if (videoRef.value) videoRef.value.play();
    };
  }
}

// 自动播放视频
onMounted(() => { videoAutoPlay();});
// 监听视频 src，如果存在则自动播放

watch(envSrc, () => { if (videoRef.value) videoRef.value.play(); });
</script>

<script lang="ts">
export default { name: 'StaticFile' };
</script>

<template>
  <img v-if="type === 'img'" :src="envSrc" />
  <video ref="videoRef" v-else-if="type === 'video'" muted :poster="poster" />
  <audio v-else :src="envSrc" />
</template>
```
### 4.4 封装 `SVG` 的图标组件

svg 图标比较小，而且都是可读的 xml 文本，所以我们把它直接放在项目中即可，通过 `vite-plugin-svg-icons` 插件，实现自动引入 svg 图标。

配置 vite.config.ts：

```json
plugins: [
  viteSvgIcons({
    iconDirs: [resolve(process.cwd(), 'src/assets/icons')],
    symbolId: 'icon-[dir]-[name]',
  }),
]
```
封装一个 [vue 组件](https://zhida.zhihu.com/search?q=vue+%E7%BB%84%E4%BB%B6&zhida_source=entity&is_preview=1)：

```vue
<script setup lang="ts">
import { computed, withDefaults } from 'vue';

interface Props {
  prefix?: string;
  name?: string;
  color?: string;
}

const props = withDefaults(defineProps<Props>(), {
  prefix: 'icon',
  name: '',
  color: '#000',
});

const symbolId = computed(() => `#${props.prefix}-${props.name}`);
</script>

<template>
  <svg aria-hidden="true">
    <use :xlink:href="symbolId" :fill="color" />
  </svg>
</template>

首先将下载的 .svg 图标放入 @/assets/icons 文件夹下

<svg-icon name="" color="" />
```

- name 放置在 @/assets/icons 文件夹下的文件名。
- color 颜色填充，使用此项会默认覆盖图标颜色。

## 5. 按需自动引入组件

[unplugin-vue-components](https://link.zhihu.com/?target=https%3A//github.com/antfu/unplugin-vue-components) 是一款非常强大的插件（极力推荐），核心功能就是帮助你自动按需引入组件，[Tree-shakable](https://zhida.zhihu.com/search?q=Tree-shakable&zhida_source=entity&is_preview=1)，只注册你使用的组件。这里说一下他的两个核心使用方式和配置方式。

此插件不仅支持 vue3，同时也支持 vue2，并且支持 Vite、Webpack、[Vue CLI](https://zhida.zhihu.com/search?q=Vue+CLI&zhida_source=entity&is_preview=1)、Rollup。  

### 5.1 安装与配置

安装：

```
npm i unplugin-vue-components -D
```
配置：

```ts
// vite.config.ts
import Components from 'unplugin-vue-components/vite'

export default defineConfig({
  plugins: [
    Components({ /* options */ }),
  ],
})
```

这里的 options 可以配置一些选项，后面提到的组件库注册会使用到。

### 5.2 改变全局组件注册方式

我们通常将全局的组件封装在 `@/src/components` 中，然后通过 `app.component()` 注册全局组件。使用此插件后，无需手写注册，直接在模板中使用组件即可：

这里引入官方的示例：

<template>
  <div>
    <HelloWorld msg="Hello Vue 3.0 + Vite" />
  </div>
</template>

```vue
<script>
export default {
  name: 'App'
}
</script>

自动编译为：

<template>
  <div>
    <HelloWorld msg="Hello Vue 3.0 + Vite" />
  </div>
</template>

<script>
import HelloWorld from './src/components/HelloWorld.vue'

export default {
  name: 'App',
  components: {
    HelloWorld
  }
}
</script>
```
### 5.3 自动引入组件库

在使用组件库时，常规组件我们也会注册到全局，如果使用局部注册由于页面中会使用到多个组件，会非常麻烦，所以这个功能绝佳，例如我们使用 [ant-design-vue 组件库](https://zhida.zhihu.com/search?q=ant-design-vue+%E7%BB%84%E4%BB%B6%E5%BA%93&zhida_source=entity&is_preview=1)。

直接在模板中使用即可，无需手动注册或局部引用：

<template>
  <a-button>按钮</a-button>
</template>

当然，你还需要在 vite 中引入它的解析器：

```ts
import Components from 'unplugin-vue-components/vite'
import { AntDesignVueResolver } from 'unplugin-vue-components/resolvers'

export default defineConfig({
  plugins: [
    Components({
      resolvers: [
        AntDesignVueResolver(),
      ]
    })
  ],
})
```

目前支持的解析器，根据你的喜好去选择：

- [Ant Design Vue](https://link.zhihu.com/?target=https%3A//github.com/antfu/unplugin-vue-components/blob/main/src/core/resolvers/antdv.ts)
- [Element Plus](https://link.zhihu.com/?target=https%3A//github.com/antfu/unplugin-vue-components/blob/main/src/core/resolvers/element-plus.ts)
- [Element UI](https://link.zhihu.com/?target=https%3A//github.com/antfu/unplugin-vue-components/blob/main/src/core/resolvers/element-ui.ts)
- [Headless UI](https://link.zhihu.com/?target=https%3A//github.com/antfu/unplugin-vue-components/blob/main/src/core/resolvers/headless-ui.ts)
- [IDux](https://link.zhihu.com/?target=https%3A//github.com/antfu/unplugin-vue-components/blob/main/src/core/resolvers/idux.ts)
- [Naive UI](https://link.zhihu.com/?target=https%3A//github.com/antfu/unplugin-vue-components/blob/main/src/core/resolvers/naive-ui.ts)
- [Prime Vue](https://link.zhihu.com/?target=https%3A//github.com/antfu/unplugin-vue-components/blob/main/src/core/resolvers/prime-vue.ts)
- [Vant](https://link.zhihu.com/?target=https%3A//github.com/antfu/unplugin-vue-components/blob/main/src/core/resolvers/vant.ts)
- [VEUI](https://link.zhihu.com/?target=https%3A//github.com/antfu/unplugin-vue-components/blob/main/src/core/resolvers/veui.ts)
- [Varlet UI](https://link.zhihu.com/?target=https%3A//github.com/antfu/unplugin-vue-components/blob/main/src/core/resolvers/varlet-ui.ts)
- [View UI](https://link.zhihu.com/?target=https%3A//github.com/antfu/unplugin-vue-components/blob/main/src/core/resolvers/view-ui.ts)
- [Vuetify](https://link.zhihu.com/?target=https%3A//github.com/antfu/unplugin-vue-components/blob/main/src/core/resolvers/vuetify.ts)
- [VueUse Components](https://link.zhihu.com/?target=https%3A//github.com/antfu/unplugin-vue-components/blob/main/src/core/resolvers/vueuse.ts)
- [Quasar](https://link.zhihu.com/?target=https%3A//github.com/antfu/unplugin-vue-components/blob/main/src/core/resolvers/quasar.ts)

## 6. 样式

项目中最好使用通用样式，可以创建 `src/styles` 目录存放，这里推荐一些分类：

styles
  ├── antd # 组件库样式覆盖，命名自取，这里以 ant design 为例
  ├── color.less # 颜色
  ├── index.less # 入口
  ├── global.less # 公共类
  ├── transition.less # 动画相关
  └── variable.less # 变量

### 6.1 预设基础样式

相信用过 [normalize](https://link.zhihu.com/?target=https%3A//github.com/necolas/normalize.css) 的同学不在少数，它可以重置 css 样式，使各浏览器效果保持一致。后面的章节会提到 tailwind.css，它内置了预设样式重置的功能，与 normalize 还是有一定的区别，有兴趣的同学可以[了解一下](https://link.zhihu.com/?target=https%3A//www.tailwindcss.cn/docs/preflight)。

### 6.2 CSS 预处理器

虽然 vite 原生支持 less/sass/scss/[stylus](https://zhida.zhihu.com/search?q=stylus&zhida_source=entity&is_preview=1)，但是你必须手动安装他们的预处理器依赖，例如：

```
npm install -D less
```

如何选择预处理器？

推荐使用你是所使用的组件库的样式语言，因为 css 预处理器学会一种后，入手其他几乎没有学习成本。

### 6.3 开启 scoped

没有加 scoped 属性，会编译成全局样式，造成全局污染。

<style scoped></style>

### 6.4 深度选择器

有时我们可能想明确地制定一个针对子组件的规则。

如果你希望 scoped 样式中的一个选择器能够作用得 “更深”，例如影响子组件，你可以使用 >>> 操作符。有些像 Sass 之类的预处理器无法正确解析 >>>。这种情况下你可以使用 /deep/ 或 ::v-deep 操作符取而代之——两者都是 >>> 的别名，同样可以正常工作。

## 7. 布局

页面整体布局是一个产品最外层的框架结构，往往会包含导航、页脚、侧边栏等。在页面之中，也有很多区块的布局结构。在真实项目中，页面布局通常统领整个应用的界面，有非常重要的作用，所以单独拆分出来也是非常有必要的。

在脚手架中，所有的通用布局组件都应该放在 src/layouts 中，这种封装比较简单，这里就不贴代码了，大家按照自己实际情况自行发挥，在此仅提供一下封装思路。

### 7.1 常规的布局

**[BasicLayout](https://zhida.zhihu.com/search?q=BasicLayout&zhida_source=entity&is_preview=1)**

基础页面布局，包含了头部导航，侧边栏等。

**BlankLayout**

空白的布局。

### 7.2 特殊的布局

**[RouteLayout](https://zhida.zhihu.com/search?q=RouteLayout&zhida_source=entity&is_preview=1)**

如果你的项目在路由切换中需要对某些二级页面进行缓存，那么推荐你创建一个 RouteLayout，通过路由 `meta` 中的配置，返回 `[router-view](https://zhida.zhihu.com/search?q=router-view&zhida_source=entity&is_preview=1)` 或者使用 `[keep-alive](https://zhida.zhihu.com/search?q=keep-alive&zhida_source=entity&is_preview=1)` 包裹的 `router-view`。

**[UserLayout](https://zhida.zhihu.com/search?q=UserLayout&zhida_source=entity&is_preview=1)**

用于用户登录注册等页面抽离出来。

**PageLayout**

基础布局，包含了面包屑等信息，内含 slot。

## 8. 集成 Tailwind.css

[Tailwind.css](https://link.zhihu.com/?target=https%3A//tailwindcss.com/) 在我第一次看到它的时候，内心是比较反感的，但实际上手之后又觉得真香。从 vue2 项目中，我已经引入了 tailwind，整体的开发结果就是，基本很少再使用 `<style>` 标签去转本定义一些 class 和样式，毕竟起名字这种事，一个是涉及到规范，一个是涉及到英语。如果你选择 tailwind，CSS 预处理器的作用就会显得微乎其微，因为你无需再自定定义各种变量和 mixins。

总体来说，学习成本并不高，花上两个小时足够上手，记住不用死记硬背那些类名。

### 8.1 效率提升

很多人总是说样式要与 HTML 分离，现在为什么又要提倡 tailwind 这种与 HTML 紧密结合的工具？这是因为现在使用 vue 这类框架已经高度组件化，样式分离是为了方便复用和维护，但在组件化面前样式分离只能是降低开发效率。

下面介绍一下 tailwind 提供了哪些提升效率的功能：

- 提供了大量的功能类，极大的提高了可维护性。
- 响应式设计，各种设备一把梭。
- 悬停、焦点和其它状态。
- 深色模式。
- 支持配置，例如颜色方面很难做到跟你的设计师统一。
- 不用为起名字而纠结???

### 8.2 JIT 模式

如果你的环境支持 postcss8（ vue/cli 构建的 vue2 项目是 postcss7 ），那么 **JIT** 模式直接带你起飞。

- 超快的构建速度。
- 支持变体，你甚至可以这么写 `sm:hover:active:disabled:opacity-75`。
- 支持任意样式，例如 `md:top-[-113px]`。
- 开发和生产环境结果是一致的，（我在 vue2 项目中就遇到过组件库构建结果不一致的问题）。

如果你使用 vscode 那你一定要安装 [Tailwind CSS IntelliSense](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dbradlc.vscode-tailwindcss) 插件，它可以自动补全类名，显著降低学习成本。

### 8.3 关于打包体积

使用默认配置，未压缩是 **3739.4kB** ，[Gzip](https://zhida.zhihu.com/search?q=Gzip&zhida_source=entity&is_preview=1) 压缩 是 293.9kB，[Brotli](https://zhida.zhihu.com/search?q=Brotli&zhida_source=entity&is_preview=1) 压缩 是 73.2kB。这似乎看起来很大，这是因为 tailwind 提供了成千上万的功能类，其中绝大部分你不会使用到。

当构建生产时，你应该使用 purge 选项来 **[tree-shake](https://zhida.zhihu.com/search?q=tree-shake&zhida_source=entity&is_preview=1)** 优化未使用的样式，并优化您的最终构建大小当使用 Tailwind 删除未使用的样式时，很难最终得到超过 **10kb** 的压缩 CSS。

还有一点，`[Atom](https://zhida.zhihu.com/search?q=Atom&zhida_source=entity&is_preview=1) CSS` 极大的提升了样式的复用程度，从而直接降低了构建体积。

## 9.vuex 替代方案 pinia

由于 `vuex 4` 对 typescript 的支持让人感到难过，所以状态管理弃用了 vuex 而采取了 [pinia](https://link.zhihu.com/?target=https%3A//pinia.esm.dev/)。

忘记在哪看到，[尤大](https://zhida.zhihu.com/search?q=%E5%B0%A4%E5%A4%A7&zhida_source=entity&is_preview=1)好像说 [pinia](https://link.zhihu.com/?target=https%3A//pinia.esm.dev/) 可能会代替 vuex，所以请放心使用。

### 9.1 为什么采用 Pinia ?

- **Pinia** 的 API 设计非常接近 `[Vuex 5](https://zhida.zhihu.com/search?q=Vuex+5&zhida_source=entity&is_preview=1)` 的[提案](https://link.zhihu.com/?target=https%3A//github.com/vuejs/rfcs/discussions/270)。（作者是 Vue 核心团队成员）
- 无需像 `Vuex 4` 自定义复杂的类型来支持 typescript，天生具备完美的类型推断。
- 模块化设计，你引入的每一个 store 在打包时都可以自动拆分他们。
- 无嵌套结构，但你可以在任意的 store 之间交叉组合使用。
- **Pinia** 与 **[Vue devtools](https://zhida.zhihu.com/search?q=Vue+devtools&zhida_source=entity&is_preview=1)** 挂钩，不会影响 Vue 3 开发体验。

下面简单的介绍一下如何使用 Pinia，并对比 vuex 有哪些区别与注意事项，具体请参考[官方文档](https://link.zhihu.com/?target=https%3A//pinia.esm.dev/)。

### 9.2 创建 Store

Pinia 已经内置在脚手架中，并且与 vue 已经做好了关联，你可以在任何位置创建一个 store：

import { defineStore } from 'pinia'

export const useUserStore = defineStore({
  id: 'user',
  state: () =>({}),
  getters: {},
  actions: {}
})

这与 Vuex 有很大不同，它是标准的 Javascript 模块导出，这种方式也让开发人员和你的 IDE 更加清楚 store 来自哪里。

Pinia 与 Vuex 的区别：

- **id** 是必要的，它将所使用 store 连接到 devtools。
- 创建方式：`new Vuex.Store(...)`(vuex3)，`createStore(...)`(vuex4)。
- 对比于 vuex3 ，state 现在是一个**函数返回对象**。
- 没有 **[mutations](https://zhida.zhihu.com/search?q=mutations&zhida_source=entity&is_preview=1)**，不用担心，state 的变化依然记录在 devtools 中。

### 9.3 State

创建好 store 之后，可以在 state 中创建一些属性了：

state: () => ({ name: 'codexu', age: 18 })

将 store 中的 state 属性设置为一个函数，该函数返回一个包含不同状态值的对象，这与我们在组件中定义数据的方式非常相似。

在模板中使用 store：

现在我们想从 store 中获取到 name 的状态，我们只需要使用以下的方式即可：

<h1>{{userStore.name}}</h1>

const userStore = useUserStore()
return { userStore }

注意这里并不需要 `userStore.state.name`。

虽然上面的写法很舒适，但是你一定不要用解构的方式去提取它内部的值，这样做的话，会失去它的响应式：

const { name, email } = useUserStore()

### 9.4 Getters

Pinia 中的 getter 与 Vuex 中的 getter 、组件中的计算属性具有相同的功能，传统的函数声明使用 this 代替了 state 的传参方法，但箭头函数还是要使用函数的第一个参数来获取 state ，因为箭头函数处理 this 的作用范围：

getters: {
  nameLength() {
    return this.name.length
  },
  nameLength: state => state.name.length,
  nameLength: ()=> this.name.length ❌ 
}

### 9.5 Actions

这里与 Vuex 有极大的不同，Pinia 仅提供了一种方法来定义如何更改状态的规则，**放弃 mutations 只依靠 Actions**，这是一项重大的改变。

Pinia 让 Actions 更加的灵活：

- 可以通过**组件**或其他 **action** 调用
- 可以从**其他 store** 的 action 中调用
- 直接在商店实例上调用
- 支持**同步**或**异步**
- 有任意数量的参数
- 可以包含有关如何更改状态的逻辑（也就是 vuex 的 mutations 的作用）
- 可以 `$patch` 方法直接更改状态属性

actions: {
  async insertPost(data){
    await doAjaxRequest(data);
    this.name = '...';
  }
}

### 9.6 Devtools

脚手架已内置下面的代码，这将添加 devtools 支持：

import { createPinia, PiniaPlugin } from 'pinia'

Vue.use(PiniaPlugin)
const pinia = createPinia()

时间旅行功能貌似已经可以使用了，这块后续会关注。

## 10. 基于 mitt 处理组件间事件联动

如果你曾经是 Vue2.x 的开发者，那么请阅读下面引用[官方文档](https://link.zhihu.com/?target=https%3A//v3.cn.vuejs.org/guide/migration/events-api.html%23_3-x-%25E6%259B%25B4%25E6%2596%25B0)的一段话：

我们从实例中完全移除了 `$on`、`$off` 和 `$once` 方法。`$emit` 仍然包含于现有的 API 中，因为它用于触发由父组件声明式添加的事件处理函数。  
在 Vue 3 中，已经不可能使用这些 API 从组件内部监听组件自己发出的事件了，该用例暂没有迁移的方法。但是该 [eventHub](https://zhida.zhihu.com/search?q=eventHub&zhida_source=entity&is_preview=1) 模式可以被替换为实现了事件触发器接口的外部库，例如 `mitt` 或 `[tiny-emitter](https://zhida.zhihu.com/search?q=tiny-emitter&zhida_source=entity&is_preview=1)`。  

### 10.1 为什么选择 mitt ？

- 足够小，仅有 200bytes。
- 支持全部事件的监听和批量移除。
- 无依赖，不论是什么框架都可以直接使用。

### 10.2 严重警告

我们已经无法在项目中使用 **[eventBus](https://zhida.zhihu.com/search?q=eventBus&zhida_source=entity&is_preview=1)**，仅推荐你在**特殊场合**下使用 mitt，**它并不是开发的常态**，你一定要确保知道自己在做什么？否则你的项目将难以维护！！！

### 10.3 如何使用 mitt ？

在使用 mitt 前建议请阅读[官方文档](https://link.zhihu.com/?target=https%3A//github.com/developit/mitt)：

脚手架默认提供一个可以直接使用的对象：

import emitter from '@/libs/emitter';

当然你也可以引入已经安装好的 mitt：

import mitt from 'mitt'

const emitter = mitt()

mitt 提供了非常简单的 API，下面代码是官方演示：

// listen to an event
emitter.on('foo', e => console.log('foo', e) )

// listen to all events
emitter.on('*', (type, e) => console.log(type, e) )

// fire an event
emitter.emit('foo', { a: 'b' })

// clearing all events
emitter.all.clear()

// working with handler references:
function onFoo() {}
emitter.on('foo', onFoo)   // listen
emitter.off('foo', onFoo)  // unlisten

## 11. 异步请求

绝大多数项目想必逃脱不了接口的对接，如果你的项目存在大量的接口，我建议做到以下几点：

- 封装请求。
- 统一的 API 接口管理。
- Mock 数据功能（根据需求斟酌使用）。

上述的主要目的就是在帮助我们简化代码和利于后期的更新维护。

### 11.1 基于 axios 的封装

相信开发过 vue2 项目的同学已经对 axios 非常熟悉的，在这里提供一些封装的思路：

- 通过 `import.meta.env.VITE_APP_BASE_URL` 获取环境变量，配置 `baseURL`，如果接口存在多个不同域名，可以通过 js 变量控制。
- 设置 `timeout` 请求超时、断网情况处理。
- 设置**请求头**，携带 `token`。
- **异常拦截处理**，后端通过你携带的 `token` 判断你是否过期，如果返回 `401` 你可能需要跳转到登录页面，并提示需要重新登录。
- **响应拦截**，通常后端返回 code、data、msg，如果是请求正常，我们可以直接返回 data 数据，如果是异常的 code，我们也可以在这里直接弹出报错提示。
- **无感刷新 token**，如果你的 token 过期，可以通过后端返回的 refreshToken 调用刷新接口，获取新的 token。当然这里涉及到很多细节，例如终端请求、重新发送请求、重新请求列队。
- **中断请求**，例如页面切换时，我们要中断正在发生的请求。

[相关代码（仅供参考）](https://link.zhihu.com/?target=https%3A//github.com/code-device/x-build/blob/next/template/src/libs/request/index.ts)

### 11.2 为 axios 增加泛型的支持

到目前为止，axios 请求返回的类型是 any，这时我们对请求后的数据进行操作时，没有享受到 ts 带来的类型提示，这显然不符合我们的预期。

这时我们要做的就是重新声明 axios 模块： 新建一个 shims.d.ts，然后在调用时加上泛型。

import { AxiosRequestConfig } from 'axios';

declare module 'axios' {
  export interface AxiosInstance {
    <T = any>(config: AxiosRequestConfig): Promise<T>;
    request<T = any>(config: AxiosRequestConfig): Promise<T>;
    get<T = any>(url: string, config?: AxiosRequestConfig): Promise<T>;
    delete<T = any>(url: string, config?: AxiosRequestConfig): Promise<T>;
    head<T = any>(url: string, config?: AxiosRequestConfig): Promise<T>;
    post<T = any>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T>;
    put<T = any>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T>;
    patch<T = any>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T>;
  }
}

做好这一步后，你就必须在创建接口时，声明请求相应数据的类型。

### 11.3 封装更方便的 useRequest

设想一下，编写请求代码时，我们通常会定义这么几个变量：

- **data**: 储存请求数据
- **loading**: 请求加载状态

尤其是 loading，我们需要在请求前设置为 true，请求结束后设置为 false。

上面的封装方式，是对基础的功能封装，因为我们在使用 vue3，所以可以进行再一次的封装成为 **hook**，我们使用起来会更加方便。

例如下面这个样子：

使用 useRequest 定义一个接口：

export default getUserInfo(id) {
  return useRequest({
    method: 'get',
    url: '/api/user',
    params: { id }
  })
}

使用此接口：

const { data, loading } = getUserInfo();

注意这里的 data 是响应式的。  

这是我想到的一种思路，目前还没有做很好的封装，[相关代码仅供参考](https://link.zhihu.com/?target=https%3A//github.com/code-device/x-build/blob/next/template/src/libs/request/index.ts%23L101)，你也可以借鉴一些成熟方案，比如 vueuse 中的 [useFetch](https://link.zhihu.com/?target=https%3A//vueuse.org/core/usefetch/%23usefetch)，但是他是基于 Fetch API 设计的，并不符合我的预期要求，有更好的方案请大家在下面留言。

### 11.4 统一的 API 接口管理

自从前端和后端分家之后，前后端接口对接就成为了常态，而对接接口的过程就离不开接口文档，比较主流就是 Swagger，但是如何在前端项目中更好的去管理跟后端对接的接口呢？

在 src 目录中 创建 api 目录，内部目录应按照后端制定的模块创建。

每个模块中创建多个 ts 文件，一个接口应对应一个 ts 文件，其中包含了以下内容：

- 请求**参数**的类型声明。
- 响应**数据**的类型声明。
- 返回定义好的请求函数（url、method、params、data 等）。

统一去定义和管理 API 接口，只要后端规范的命名和你认真的写好类型声明，对前端来说 typescript 就是最好的接口文档。

### 11.5 mock

vite 使用 mock 数据非常简单，你可以使用 [vite-plugin-mock](https://link.zhihu.com/?target=https%3A//github.com/anncwb/vite-plugin-mock) 插件，如果你了解 mockjs，你可以快速上手。

## 12. 路由

路由和菜单是组织起一个应用的关键骨架。

### 12.1 创建路由三部曲

通常一个项目需要做到这几步：

- 使用 **[createRouter](https://zhida.zhihu.com/search?q=createRouter&zhida_source=entity&is_preview=1)** 创建路由，这时候根据需求选择 **Hash** 路由或者 **History** 路由。
- 根据业务需求**配置路由**，注意这里很可能就用到前文提到过的**布局组件**。
- 如果有权限相关的业务，你需要创建 **permission.ts** 在路由钩子触发时做一些事情。

如果你的页面比较多，建议你创建 [routes](https://zhida.zhihu.com/search?q=routes&zhida_source=entity&is_preview=1) 目录，分模块声明路由。  

[参考代码](https://link.zhihu.com/?target=https%3A//github.com/code-device/x-build/tree/next/template/src/router)

### 12.2 使用 meta 丰富你的路由

vue-router4.x 支持 typescript，配置路由的类型是 [RouteRecordRaw](https://zhida.zhihu.com/search?q=RouteRecordRaw&zhida_source=entity&is_preview=1)，这里 meta 可以让我们有更多的发挥空间，这里提供一些参考：

- title: string; 页面标题，通常必选。
- icon?: string; 图标，一般配合菜单使用。
- auth?: boolean; 是否需要登录权限。
- ignoreAuth?: boolean; 是否忽略权限。
- roles?: RoleEnum[]; 可以访问的角色
- keepAlive?: boolean; 是否开启页面缓存
- hideMenu?: boolean; 有些路由我们并不想在菜单中显示，比如某些编辑页面。
- order?: number; 菜单排序。
- frameUrl?: string; 嵌套外链。

这里只提供一些思路，每个项目多多少少会涉及到这些问题，具体如何实现请查阅资料自行解决。  

## 13. 项目性能与细节优化

### 13.1 开启 gzip

开启 gzip 可以极大的压缩静态资源，对页面加载的速度起到了显著的作用。

使用 [vite-plugin-compression](https://link.zhihu.com/?target=https%3A//github.com/anncwb/vite-plugin-compression) 可以 **gzip** 或 **brotli** 的方式来压缩资源，这一步需要服务器端的配合，vite 只能帮你打包出 `.gz` 文件。此插件使用简单，你甚至无需配置参数，引入即可。

### 13.2 页面载入进度条

页面路由切换时，附带一个加载进度条会显得非常友好，不至于白屏时间过长，让用户以为页面假死。

这时候我们可以用到 [nprogress](https://link.zhihu.com/?target=https%3A//github.com/rstacruz/nprogress)，在路由切换时开启和关闭：

import NProgress from 'nprogress';

router.beforeEach(async (to, from, next) => {
  NProgress.start();
});

router.afterEach((to) => {
  NProgress.done();
});

### 13.3 Title

在不同的路由下显示不同的标题是常规的操作，我们可以通过路由钩子获取 meta 中的 title 属性改变标签页上的 title。

你可以使用 vueuse 提供的 [useTitle](https://link.zhihu.com/?target=https%3A//vueuse.org/core/usetitle/%23usetitle)，或者 window.document.title 自行封装。

你也可以通过环境变量将你的主标题拼接在路由标题的后面：

const { VITE_APP_TITLE } = import.meta.env;

### 13.4 解决移动端使用 vh 的问题

有兴趣的同学可以尝试一下 chrome 移动端浏览器上的 100vh，是真正的视口高度的 100% 嘛。

为了解决这一问题，我们可以通过 [postCss](https://zhida.zhihu.com/search?q=postCss&zhida_source=entity&is_preview=1) 插件解决。

安装 [postcss-viewport-height-correction](https://link.zhihu.com/?target=https%3A//github.com/Faisal-Manzer/postcss-viewport-height-correction) 插件：

npm install -D postcss-viewport-height-correction

在 postcss.config.js 中增加 plugin：

module.exports = {
  plugins: {
    'postcss-viewport-height-correction': {},
  },
}

添加这一段 js 代码在全局，你可以直接添加在 index.html 上即可：

const customViewportCorrectionVariable = 'vh';
function setViewportProperty(doc) {
  let prevClientHeight;
  const customVar = `--${customViewportCorrectionVariable || 'vh'}`;
  function handleResize() {
    const { clientHeight } = doc;
    if (clientHeight === prevClientHeight) return;
    requestAnimationFrame(function updateViewportHeight() {
      doc.style.setProperty(customVar, `${clientHeight * 0.01}px`);
      prevClientHeight = clientHeight;
    });
  }
  handleResize();
  return handleResize;
}
window.addEventListener('resize', setViewportProperty(document.documentElement));

### 13.5 可以常驻的 JavaScript 库

- 前文提到过的 [vueuse](https://link.zhihu.com/?target=https%3A//vueuse.org/guide/)，非常强大，强烈建议尝试。
- [lodash](https://link.zhihu.com/?target=https%3A//www.lodashjs.com/)，用了都说好，早用早下班。

## 14. 代码风格与流程规范

### 14.1 ESLint

不管是多人合作还是个人项目，代码规范都是很重要的。这样做不仅可以很大程度地避免基本语法错误，也保证了代码的可读性。

这里推荐使用 **airbnb** 规范。

[配置参考](https://link.zhihu.com/?target=https%3A//juejin.cn/post/7005001569879982087%23heading-2)

### 14.2 StyleLint

尽管前文提到过 tailwind，可以让你几乎不写 css，但是涉及到团队协作，这一点也要严谨。

StyleLint 是一个强大的、现代化的 CSS 检测工具, 与 ESLint 类似, 是通过定义一系列的编码风格规则帮助我们避免在样式表中出现错误，配合编辑器的自动修复，可以很好的统一团队项目 css 风格。

[配置参考](https://link.zhihu.com/?target=https%3A//juejin.cn/post/7005001569879982087%23heading-6)

### 14.3 代码提交规范

在多人协作的背景下，git 仓库和 workflow 的作用很重要。而对于 commit 提交的信息说明存在一定规范，现使用 **commitlint** + **[husky](https://zhida.zhihu.com/search?q=husky&zhida_source=entity&is_preview=1)** 规范 git commit -m "" 中的描述信息。我们都知道，在使用 git commit 时，git 会提示我们填入此次提交的信息。可不要小看了这些 commit，团队中规范了 commit 可以更清晰的查看每一次代码提交记录，还可以根据自定义的规则，自动生成 changeLog 文件。

**提交格式（注意冒号后面有空格）：**

<type>[optional scope]: <description>

- type ：用于表明我们这次提交的改动类型。
- optional scope：可选，用于标识此次提交主要涉及到代码中哪个模块。
- description：一句话描述此次提交的主要内容，做到言简意赅。

**Type 类型**

- build：编译相关的修改，例如发布版本、对项目构建或者依赖的改动
- chore：其他修改, 比如改变构建流程、或者增加依赖库、工具等
- ci：持续集成修改
- docs：文档修改
- feat：新特性、新功能
- fix：修改 bug
- perf：优化相关，比如提升性能、体验
- refactor：代码重构
- revert：回滚到上一个版本
- style：代码格式修改, 注意不是 css 修改
- test：测试用例修改

关于 commitlint + husky 的配置文章有很多，大同小异，请根据自己的实际情况配置。

## 15. 编写使用文档

做到这一步，你的整个脚手架开发已经接近于尾声，但是你做了这么多，你的同事并不知道如何使用，甚至你过一段时间也会忘记，所以你必须养成良好的编写文档习惯。

### 15.1 使用 vitepress 搭建文档

这里我推荐使用 [vuepress](https://zhida.zhihu.com/search?q=vuepress&zhida_source=entity&is_preview=1) 或者 vitepress，说实话你只写文档 vitepress 会让你更舒服，因为它很快。

[vitepress](https://link.zhihu.com/?target=https%3A//vitepress.vuejs.org/) 很适合构建博客网站、技术文档，就是因为它可以直接用 markdown 进行书写，所有写过博客的人，都应该对它不陌生。一个 .md 文件，即可生成一张页面，十分方便。

创建一个 vitepress 文档实在是太过于简单，你可以参考官方文档，或者参考我的[文档](https://link.zhihu.com/?target=https%3A//github.com/code-device/x-build/tree/next/documents)。

### 15.2 文档部署

如果你的团队可以帮助你搭建 CI/CD 自动部署是再好不过了，如果没有这个条件，你也可以通过 github 提供的 actions 功能，完成自动部署。

[代码参考](https://link.zhihu.com/?target=https%3A//github.com/code-device/x-build/blob/next/.github/workflows/ci.yml)

## 16. 插件

如果你想更痛快的用上述功能，建议你安装下面的插件。

### 16.1 VSCode 插件

- [Vue Language Features (Volar)](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Djohnsoncodehk.volar)，你现在查 Volar 可能找不到，你需要的是这个。
- [Vue 3 Snippets](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dhollowtree.vue-snippets)，vue3 快捷输入。
- [Tailwind CSS IntelliSense](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dbradlc.vscode-tailwindcss)，tailwind 代码提示。
- [Stylelint](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dstylelint.vscode-stylelint)
- [Prettier - Code formatter](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Desbenp.prettier-vscode)
- [ESLint](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Ddbaeumer.vscode-eslint)

### 16.2 Chrome 插件

- [Vue.js devtools](https://link.zhihu.com/?target=https%3A//chrome.google.com/webstore/detail/vuejs-devtools/ljjemllljcmogpfapbkkighbhhppjdbg)，你当然要安装支持 vue3 的版本，而且此版本对 pinia 支持的也非常友好。

## 源码

上述内容，均可在我的[开源项目](https://zhida.zhihu.com/search?q=%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE&zhida_source=entity&is_preview=1) [X-BUILD](https://link.zhihu.com/?target=https%3A//github.com/code-device/x-build) 中找到相关源码，如果可以帮到你，请给一颗 star 或点赞鼓励我贡献出更多的开源项目或文章。

## 参考

- [《基于 Vue 的前端架构，我做了这 15 点》](https://link.zhihu.com/?target=https%3A//juejin.cn/post/6901466994478940168)
- [《搭建自己的脚手架—“优雅” 生成前端工程》](https://link.zhihu.com/?target=https%3A//juejin.cn/post/6844903607855235079)
- [《Vuex4 对 TypeScript 并不友好，所以我选择 Pinia》](https://link.zhihu.com/?target=https%3A//juejin.cn/post/6981356417064108062)
- [《前端脚手架 webpack 迁移 Vite2 踩坑实践》](https://link.zhihu.com/?target=https%3A//juejin.cn/post/7005001569879982087)