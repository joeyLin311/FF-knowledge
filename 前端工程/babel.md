---
date created: 2021-12-09 22:57
date updated: 2021-12-17 16:02
---

# babel

## babel 编译流程

babel 三大编译步骤:

- **解析 parser**,
- **转换 transformer**,
- **生成 generator**

1. **解析阶段:**  babel 默认使用 @babel/parser 将代码转换为 AST.解析一般分为两个阶段: 词法分析和语法分析
2. **词法分析:** 对输入的字符序列做标记优化(tokenization)操作
3. **语法分析:** 处理标记与标记之间的关系, 最终形成一颗完整的 AST 抽象语法树

> 将代码解析成抽象语法树（AST）,每个 js 引擎（比如 Chrome 浏览器中的 V8 引擎）都有自己的 AST 解析器,而 Babel 是通过 Babylon 实现的.在解析过程中有两个阶段：词法分析和语法分析,词法分析阶段把字符串形式的代码转换为令牌（tokens）流,令牌类似于 AST 中节点；而语法分析阶段则会把一个令牌流转换成 AST 的形式,同时这个阶段会把令牌中的信息转换成 AST 的表述结构.

2. **转换阶段:** babel 使用 @babel/traverse 提供的方法对 AST 进行深度优先遍历, 调用插件对关注节点的梳理函数, 按需对 AST 节点进行增删改操作

> 在这个阶段,Babel 接受得到 AST 并通过 `babel-traverse` 对其进行深度优先遍历,在此过程中对节点进行添加、更新及移除操作.这部分也是 Babel 插件介入工作的部分.

3. **生成阶段:** babel 默认使用 `@babel/generator` 将上一阶段处理后的 AST 转换为代码字符串

> 将经过转换的 AST 通过 `babel-generator` 再转换成 js 代码,过程就是深度优先遍历整个 AST,然后构建可以表示转换后代码的字符串.

[babel 官网流程](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md#toc-stages-of-babel)

## babel 插件系统

babel 的核心模块 `@babel/core`, `@babel/parser`, `@babel/generator` 和 `@babel/traverse` 提供了完成的编译流程, 但是具体的转换逻辑需要插件来完成

使用 babel 时我们可以通过配置文件指定 `plugin` 和 `preset`, 而 `preset` 可以是 `plugin` 和 `preset` 以及其它配置的集合.babel 会递归读取 `preset` , 最终获取一个大的 `plugin` 数组用于后续.

**插件总共分为两种：**

当我们添加 **语法插件** 之后, 在解析这一步就使得 babel 能够解析更多的语法.(顺带一提, babel 内部使用的解析类库叫做 babylon, 并非 babel 自行开发)

举个简单的例子, 当我们定义或者调用方法时, 最后一个参数之后是不允许增加逗号的, 如 `callFoo(param1, param2,)` 就是非法的.如果源码是这种写法, 经过 babel 之后就会提示语法错误.
但最近的 JS 提案中已经允许了这种新的写法(让代码 diff 更加清晰).为了避免 babel 报错, 就需要增加语法插件 `babel-plugin-syntax-trailing-function-commas`

当我们添加 转译插件 之后, 在转换这一步把源码转换并输出.这也是我们使用 babel 最本质的需求.

比起语法插件, 转译插件其实更好理解, 比如箭头函数 `(a) => a` 就会转化为 `function (a) {return a}`.完成这个工作的插件叫做 `babel-plugin-transform-es2015-arrow-functions`

## babel 执行顺序

很简单的几条原则：

- Plugin 会运行在 Preset 之前.
- Plugin 会从前到后顺序执行.
- Preset 的顺序则 刚好相反(从后向前).

preset 的逆向顺序主要是为了保证向后兼容, 因为大多数用户的编写顺序是 ['es2015', 'stage-0'].这样必须先执行 stage-0 才能确保 babel 不报错.因此我们编排 preset 的时候, 也要注意顺序, 其实只要按照规范的时间顺序列出即可.
插件和 preset 的配置项
简略情况下, 插件和 preset 只要列出字符串格式的名字即可.但如果某个 preset 或者插件需要一些配置项(或者说参数), 就需要把自己先变成数组.第一个元素依然是字符串, 表示自己的名字；第二个元素是一个对象, 即配置对象.
最需要配置的当属 env, 如下：

```jsx
"presets": [
    // 带了配置项, 自己变成数组
    [
        // 第一个元素依然是名字
        "env",
        // 第二个元素是对象, 列出配置项
        {
          "module": false
        }
    ],

    // 不带配置项, 直接列出名字
    "stage-2"
]
```

## bebel env (重点)

因为 env 最为常用也最重要, 所以我们有必要重点关注.
env 的核心目的是通过配置得知目标环境的特点, 然后只做必要的转换.例如目标浏览器支持 es2015, 那么 es2015 这个 preset 其实是不需要的, 于是代码就可以小一点(一般转化后的代码总是更长), 构建时间也可以缩短一些.
如果不写任何配置项, env 等价于 latest, 也等价于 es2015 + es2016 + es2017 三个相加(不包含 stage-x 中的插件).env 包含的插件列表维护在这里
下面列出几种比较常用的配置方法：

```jsx
{
  "presets": [
    ["env", {
      "targets": {
        "browsers": ["last 2 versions", "safari >= 7"]
      }
    }]
  ]
}
```

如上配置将考虑所有浏览器的最新 2 个版本(safari 大于等于 7.0 的版本)的特性, 将必要的代码进行转换.而这些版本已有的功能就不进行转化了.这里的语法可以参考 browserslist

```jsx
{
  "presets": [
    ["env", {
      "targets": {
        "node": "6.10"
      }
    }]
  ]
}
```

如上配置将目标设置为 nodejs, 并且支持 6.10 及以上的版本.也可以使用 `node: 'current'` 来支持最新稳定版本.例如箭头函数在 nodejs 6 及以上将不被转化, 但如果是 nodejs 0.12 就会被转化了.
另外一个有用的配置项是 `modules`.它的取值可以是 `amd`, `umd`, `systemjs`, `commonjs` 和 `false`.这可以让 babel 以特定的模块化格式来输出代码.如果选择 `false` 就不进行模块化处理.

## babel 7.x 新变更

### preset 的变更：淘汰 es201x, 删除 stage-x, 强推 env (重点)

淘汰 es201x 的目的是把选择环境的工作交给 env 自动进行, 而不需要开发者投入精力.凡是使用 es201x 的开发者, 都应当使用 env 进行替换.但这里的淘汰 (原文 deprecated) 并不是删除, 只是不推荐使用了, 不好说 babel 8 就真的删了.
与之相比, stage-x 就没那么好运了, 它们直接被删了.这是因为 babel 团队认为为这些 “不稳定的草案” 花费精力去更新 preset 相当浪费.stage-x 虽然删除了, 但它包含的插件并没有删除(只是被更名了, 可以看下面一节), 我们依然可以显式地声明这些插件来获得等价的效果.完整列表
为了减少开发者替换配置文件的机械工作, babel 开发了一款 babel-upgrade 的工具, 它会检测 babel 配置中的 stage-x 并且替换成对应的 plugins.除此之外它还有其他功能, 我们一会儿再详细看.(总之目的就是让你更加平滑地迁移到 babel 7)

### npm package 名称的变化 (重点)

这是 babel 7 的一个重大变化, 把所有 `babel-*` 重命名为 `@babel/*`, 例如：

- `babel-cli` 变成了 `@babel/cli`.
- `babel-preset-env` 变成了 `@babel/preset-env`.进一步, 还可以省略 `preset` 而简写为 `@babel/env`
- `babel-plugin-transform-arrow-functions` 变成了 `@babel/plugin-transform-arrow-functions`.和 `preset` 一样, plugin 也可以省略, 于是简写为 `@babel/transform-arrow-functions` .

这个变化不单单应用于 package.json 的依赖中, 包括 .babelrc 的配置 (plugins, presets) 也要这么写, 为了保持一致.例如

```jsx
{
  "presets": [
-   "env"
+   "@babel/preset-env"
  ]
}
```

顺带提一句, 上面提过的 babel 解析语法的内核 `babylon` 现在重命名为 `@babel/parser`, 看起来是被收编了.
上文提过的 `stage-x` 被删除了, 它包含的插件虽然保留, 但也被重命名了.babel 团队希望更明显地区分已经位于规范中的插件 (如 es2015 的 `babel-plugin-transform-arrow-functions`) 和仅仅位于草案中的插件 (如 `stage-0` 的 `@babel/plugin-proposal-function-bind`).方式就是在名字中增加 `proposal`, 所有包含在 `stage-x` 的转译插件都使用了这个前缀, 语法插件不在其列.
最后, 如果插件名称中包含了规范名称 (`-es2015-`, `-es3-` 之类的), 一律删除.例如 `babel-plugin-transform-es2015-classes` 变成了 `@babel/plugin-transform-classes`

### 不再支持低版本 node

babel 7.0 开始不再支持 nodejs 0.10,0.12,4, 5 这四个版本, 相当于要求 nodejs >= 6

### only 和 ignore 匹配规则的变化

在 babel 6 时, ignore 选项如果包含 `*.foo.js`, 实际上的含义 (转化为 glob) 是 `./**/*.foo.js`, 也就是当前目录 包括子目录 的所有 foo.js 结尾的文件.这可能和开发者常规的认识有悖.
于是在 babel 7, 相同的表达式 `*.foo.js` 只作用于当前目录, 不作用于子目录.如果依然想作用于子目录的, 就要按照 glob 的完整规范书写为 `./**/*.foo.js` 才可以.only 也是相同.
这个规则变化只作用于通配符, 不作用于路径.所以 node_modules 依然包含所有它的子目录, 而不单单只有一层.(否则全世界开发者都要爆炸)

## 编写 Babel 插件

Babel 插件的写法是借助访问者模式（Visitor Pattern）对关注的节点定义处理函数.参考一个简单 Babel 插件例子：

```jsx
module.exports = function () {
  return {
    pre() {},
    // 在 visitor 下挂载各种感兴趣的节点类型的监听方法
    visitor: {
      /**
       * 对 Identify 类型的节点进行处理
       * @param {NodePath} path
       */
      Identifier(path) {
        path.node.name = path.node.name.toUpperCase();
      },
    },
    post() {},
  };
};
```

使用该 Babel 插件的效果如下：

```jsx
// input

// index.js
function hzfe() {}

// .babelrc
{
  "plugins": ["babel-plugin-yourpluginname"]
}
// output
function HZFE() {}
```

## 深入 Babel 转换阶段

在转换阶段,Babel 的相关方法会获得一个插件数组变量,用于后续的操作.插件结构可参考以下接口.

```jsx
interface Plugin {
  key: string | undefined | null;
  post: Function | void;
  pre: Function | void;
  visitor: Object;
  parserOverride: Function | void;
  generatorOverride: Function | void;
  // ...
}
```

转换阶段,Babel 会按以下顺序执行.详细逻辑可查看源码：

执行所有插件的 `pre` 方法.
按需执行 `visitor` 中的方法.
执行所有插件的 `post` 方法.
一般来说,写 Babel 插件主要使用到的是 `visitor` 对象,这个 `visitor` 对象中会书写对于关注的 AST 节点的处理逻辑.而上面执行顺序中的第二步所指的 `visitor` 对象,是整合自各插件的 visitor,最终形成一个大的 `visitor` 对象,大致的数据结构可参考以下接口：

```jsx
// 书写插件时的 visitor 结构
interface VisitorInPlugin {
  [ASTNodeTypeName: string]:
    | Function
    | {
        enter?: Function;
        exit?: Function;
      };
}

// babel 最终整合的 visitor 结构
interface VisitorInTransform {
  [ASTNodeTypeName: string]: {
    // 不同插件对相同节点的处理会合并为数组
    enter?: Function[];
    exit?: Function[];
  };
}
```

在对 AST 进行深度优先遍历的过程中,会创建 `TraversalContext` 对象来把控对 `NodePath` 节点的访问,访问时调用对节点所定义的处理方法,从而实现按需执行 visitor 中的方法.详细实现请看 `babel-traverse` 中的源码.
