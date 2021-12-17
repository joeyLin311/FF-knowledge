---
date created: 2021-12-09 22:57
date updated: 2021-12-17 16:04
---

## BFC 是什么

BFC 全称为 block formatting context，中文为“块级格式化上下文”。它是一个只有块级盒子参与的独立块级渲染区域，它规定了内部的块级盒子如何布局，且与区域外部无关。

## BFC 有什么用

- 修复浮动元素造成的高度塌陷问题。
- 避免非期望的外边距折叠。
- 实现灵活健壮的自适应布局。

## 触发 BFC 的常见条件

- <html> 根元素。
- float 的值不为 none。
- position 的值不为 relative 或 static。
- overflow 的值不为 visible 或 clip（除了根元素）。
- display 的值为 table-cell，table-caption，或 inline-block 中的任意一个。
- display 的值为 flow-root，或 display 值为 flow-root list-item。
- flex items，即 display 的值为 flex 或 inline-flex 的元素的直接子元素（该子元素 display 不为 flex，grid，或 table）。
- grid items，即 display 的值为 grid 或 inline-grid 的元素的直接子元素（该子元素 display 不为 flex，grid，或 table）。
- contain 的值为 layout，content，paint，或 strict 中的任意一个。
- column-span 设置为 all 的元素。

**提示**：`display: flow-root`，`contain: layout` 等是无副作用的，可在不影响已有布局的情况下触发 BFC。

## 盒模型

box model, 盒模型描述了一个由元素生成的矩形盒子, **视觉格式化模型**决定这些盒子的大小, 位置以及属性(例如颜色, 背景, 边框尺寸等等)

### 盒模型图示

![[boxdim.png]]

> 一般盒模型分为**标准盒子模型**和**IE 盒子模型**, 前者的 height 计算 content 部分, 后者 height 包含至 border 属性. 我们可以使用 `display: content-box` 和 `display: border-box` 去设置这两种模型

## 视觉格式化模型

由于视觉格式化模型描述中，有许多相近的术语，在此先行列出解释。

- box（盒子）：一个抽象概念，在画布上占据一块空间，通常由元素（element）生成。一个元素可能生成多个盒子（如伪元素、list-item 元素）。
- principal box（主要盒子）：元素生成的多个盒子中，用来包含它的后代盒子和它生成的内容的盒子，也是参与任何定位方案的盒子。
- block-level box（块级盒子）：参与 BFC 布局的盒子，通常由块级元素生成。
- block container box（块容器）：在 CSS2.2 中，除了 table box 或替换元素的主要盒子，一个块级盒子也是块容器，但不是所有的块容器都是块级盒子（如非替换内联块盒子）。块容器主要侧重于其子元素的定位、布局。
- block box（块盒子）：如果一个块级盒子同时也是块容器，则可以称作块盒子。
- block（块）：当没有歧义时，作为 block box, block-level box 或 block container box 的简写。

（注：盒子有“块盒子”和“块级盒子”，元素只有“块级元素”，而没有“块元素”）

### 视觉格式化模型（visual formatting model）

视觉格式化模型描述了用户代理（如浏览器）在可视化媒体（如显示器）上处理文档树（document tree）的过程。下面各小节是视觉格式化模型中与 BFC 强相关的描述：

#### 包含块（containing block）

大部分情况下，包含块是一个由元素最近的祖先块容器的内容区域（content area）确定的矩形区域，包含块本身不是盒子，是一个矩形区域。元素的大小，位置，及偏移等布局信息根据包含块的尺寸进行计算。

#### 正常流（normal flow）

正常流是浏览器的默认布局方式。在正常流中的盒子，属于 BFC，IFC，或其他格式化上下文之一。

#### 格式化上下文（formatting context）

**格式化上下文**是一系列相关盒子进行布局的环境。不同的格式化上下文有不同的布局规则。目前常见的格式化上下文有以下这些：

- 块级格式化上下文（BFC，block formatting context）。
- 内联格式化上下文（IFC，inline formatting context）。
- 弹性格式化上下文（FFC，flex formatting context），在 CSS3 中定义。
- 栅格格式化上下文（GFC，grid formatting context），在 CSS3 中定义。

#### 独立格式化上下文（independent formatting context）

一个盒子要么建立一个新的独立格式化上下文，要么延续其包含块的格式化上下文。除了特殊说明，建立新的格式化上下文就是在建立一个独立格式化上下文。

当一个盒子建立一个独立格式化上下文时，它创建了一个新的独立布局环境。除了通过改变盒子本身的大小，盒子内部的后代不会影响格式化上下文外部的规则和内容，反之亦然。

#### 块级格式化上下文规范及解析

根据 W3C CSS2.1 视觉格式化模型一章的定义，BFC 相关规范描述如下：

1. 浮动元素，绝对定位元素，不是块级盒子的块级容器（如 inline-block，table-cells，table-captions）以及 overflow 值不为 visible 的块级盒子，都会为他们的内容创建 BFC（注：触发 BFC 的条件）。
2. 在 BFC 中，盒子从包含块的顶部开始，在垂直方向上一个接一个的排列。相邻盒子之间的垂直间隙由它们的 margin 值决定。在同一个 BFC 中，相邻块级盒子的垂直外边距会合并（注：参与 BFC 布局的都是块级元素）。
3. 在 BFC 中，每一个盒子的左外边缘（margin-left）会触碰到包含块的左边缘。即使是存在浮动元素也是如此，除非该盒子建立了一个新的 BFC。

结合规范第三点与**独立格式化上下文**的知识，我们可以有以下推论：

1. BFC 内外互不影响。
2. BFC 包含内部的浮动（解决内部浮动元素导致的高度塌陷）。
3. BFC 排斥外部的浮动（触发 BFC 的元素不会和外部的浮动元素重叠）。
4. 外边距折叠的计算不能跨越 BFC 的边界。
5. 各自创建了 BFC 的兄弟元素互不影响（注：在水平方向上多个浮动元素加一个或零个触发 BFC 的元素可以形成多列布局）。

通过 BFC 可以实现灵活健壮的自适应布局，在一行中达到类似 flexbox 的效果，示例如下：

**两栏自适应布局**

```html
<div class="container">
  <div class="left">
    <p>LEFT:</p>
    这是一段自适应宽度的标注
  </div>
  <div class="right">
    <p>RIGHT:</p>
    占据剩下空间的主体内容，当空间不够的时候将缩小直至换行
  </div>
</div>
<style>
  div {
    margin: 1em;
    padding: 0.5em;
    border: 2px dashed #36fd;
  }

  .left {
    float: left;
    background: #73d5;
  }

  .right {
    display: flow-root;
    background: #5df5;
    min-width: 100px;
  }
</style>
```

**多列自适应布局**

```html
<div class="container">
  <div class="col">
    <p>COL:</p>
    自适应内容宽度
  </div>
  <div class="col">
    <p>COL:</p>
    自适应内容宽度
  </div>
  <div class="col fr">
    <p>COL:</p>
    自适应内容宽度
  </div>
  <div class="col">
    <p>COL:</p>
    自适应剩余空间宽度
  </div>
</div>
<style>
  div {
    margin: 1em;
    padding: 0.5em;
    border: 2px dashed #36fd;
  }

  .fr {
    float: right !important;
  }

  .col {
    float: left;
    background: #73d5;
  }

  .col:nth-child(4) {
    float: none;
    display: flow-root;
    background: #5df5;
    min-width: 100px;
  }
</style>
```
