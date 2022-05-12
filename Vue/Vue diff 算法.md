---
date created: 2021-12-09 22:57
date updated: 2022-05-11 15:12
tags:
  - '#Vue'
---

#Vue

## diff 算法概述

`diff` 算法是一种通过同层的树节点进行比较的高效算法.
有两个特点:

- 节点比较只会在同层级进行, 不会跨层级比较
- 在diff 比较的过程中, 循环从两边靠拢至中间进行比较

## diff 算法执行过程

执行过程:

**在进行同级别节点比较的时候, 首先会对新老节点的 `dom` 数组的开始和结尾节点设置标记索引, 遍历的过程中移动索引值
在对开始节点和结束节点比较的时候, 总共有四种情况:**

1. `oldStartVnode` / `newStartVnode` (旧开始节点 / 新开始节点)
2. `oldEndVnode` / `newEndVnode` (旧结束节点 / 新结束节点)
3. `oldStartVnode` / `oldEndVnode` (旧开始节点 / 新结束节点)
4. `oldEndVnode` / `newStartVnode` (旧结束节点 / 新开始节点)

**开始节点和结束节点比较, 有两种情况类似**

- `oldStartVnode` / `newStartVnode` (旧开始节点 / 新开始节点)
- `oldEndVnode` / `newEndVnode` (旧结束节点 / 新结束节点)
- 如果 `oldStartVnode` 和 `newStartVnode` 是 `sameVnode` (key 和 sel 相同)
  - 调用 `patchVnode()` 对比和更新节点
  - 把旧开始和新开始索引往后移动 `oldStartIdx++` / `oldEndIdx++`
- `oldStartVnode` / `newEndVnode` (旧开始节点 / 新结束节点) 相同时
  - 调用 `patchVnode()` 对比和更新节点
  - 把 `oldStartVnode` 对应的 DOM 元素, 移动到右边
  - 更新索引
- `oldEndVnode` / `newStartVnode` (旧结束节点 / 新开始节点) 相同时
  - 调用 `patchVnode()` 对比和更新节点
  - 把 `oldEndVnode` 对应的 DOM 元素, 移动到左边
  - 更新索引
- 如果不是以上的四种情况下
  - 遍历新节点, 使用 `newStartNode` 的 key 在老节点数组中找相同节点
  - 如果没有找到, 说明 `newStartNode` 是新节点
  - 如果找到了, 判断新节点和找到的老节点的 sel 选择器是否相同
  - 如果不相同, 说明节点被修改了, 此时重新创建对应的 DOM 元素, 插入到 DOM 树中
  - 如果相同, 把 `elmToMove` 对应的 DOM 元素, 移动到左边
- 循环结束的条件
  - 老节点的所有子节点先遍历完 (`oldStartIdx` >`oldEndIdx`), 循环结束
  - 新节点的所有子节点先遍历完 (`newStartIdx` >`newEndIdx`), 循环结束
- 如果老节点的数组先遍历完(`oldStartIdx` >`oldEndIdx`), 说明新节点有剩余, 把剩余节点批量插入到右边
- 如果新节点的数组先遍历完(`newStartIdx` >`newEndIdx`), 说明老节点有剩余, 把剩余节点批量删除

## 小结

- 当数据发生改变时，订阅者`watcher`就会调用`patch`给真实的`DOM`打补丁
- 通过`isSameVnode`进行判断，相同则调用`patchVnode`方法
- `patchVnode`做了以下操作：
  - 找到对应的真实`dom`，称为`el`
  - 如果都有都有文本节点且不相等，将`el`文本节点设置为`Vnode`的文本节点
  - 如果`oldVnode`有子节点而`VNode`没有，则删除`el`子节点
  - 如果`oldVnode`没有子节点而`VNode`有，则将`VNode`的子节点真实化后添加到`el`
  - 如果两者都有子节点，则执行`updateChildren`函数比较子节点
- `updateChildren`主要做了以下操作：
  - 设置新旧`VNode`的头尾指针
  - 新旧头尾指针进行比较，循环向中间靠拢，根据情况调用`patchVnode`进行`patch`重复流程、调用`createElem`创建一个新节点，从哈希表寻找 `key`一致的`VNode` 节点再分情况操作

## Vue2 diff 算法

传统的 diff 算法通过循环递归对节点进行一次对比效率较为底下, 算法复杂度达到 O(n^3) , 主要原因在于其追求完全比对和最小修改, 而 React Vue 则是放弃完全比较和最小修改, 才实现复杂度从 O(n^3) => O(n)

Vue2 的优化措施是使用**分层diff**, 不考虑跨层级移动节点, 让新旧两个 VDom 树的比对无需循环递归(复杂度大幅优化, 直接下降一个数量级的首要条件). 这个前提也是Web UI 中 DOM 节点跨层级的移动操作比较少, 所以才能采用这个策略进行diff优化

Vue 2 采用的**双端比较Diff**算法可以概括为: oldChild 和 newChild 头尾各有两个指针, 他们的两个变量相互比较, 一共有4中比较方式. 如果 4 种比较都没有比配, 且设置了 key 就会采用 key 进行比较. 在比较过程中, 指针会向中间靠拢, 一旦 StartIndex > EndIndex 表示 oldChild 和 newChild 至少有一个已经遍历完成, 就会结束比较;

当发生一下情况则跳过比较, 变为插入或删除节点操作:

- **组件的Type(Tagname)不一致**, 原因是绝大多数情况拥有相同type 的两个组件将会生成相似的树形结构, 拥有不同 type 的两个组件将会生成不同的树形结构, 所以 type 不一致时可以放弃继续比较
- **列表组件的Key 不一致**, 旧VDom 树没有新Key或相反. Key 是元素的id, 能直接对应上是否为同一个节点, 所以跳过比较, 直接删改节点
- 对触发了 getter/setter 的组件进行diff, 精确减少diff范围

## Vue3 对 diff 算法的改进

**Vue2中diff 的通点**
vue2.x中的虚拟dom是进行**全量的对比**，在运行时会对所有节点生成一个虚拟节点树，当页面数据发生变更好，会遍历判断virtual dom所有节点（包括一些不会变化的节点）有没有发生变化；虽然说diff算法确实减少了多DOM节点的直接操作，但是这个**减少是有成本的**，如果是复杂的大型项目，必然存在很复杂的父子关系的VNode,**而Vue2.x的diff算法，会不断地递归调用patchVNode，不断堆叠而成的几毫秒，最终就会造成 VNode 更新缓慢**。

### 新增 patchFlag 区别动静节点

vue3 在模板编译时, **编译器会在生成 VNode 的时候为动态标签的末尾上打上 patchFlag, 再在此基础上进行核心 diff 算法.** 并且 patchFlag 会标识动态的属性类型有哪些

````js
/**
 *
 * Patch flags can be combined using the | bitwise operator and can be checked
 * using the & operator, e.g.
 *
 * ```js
 * const flag = TEXT | CLASS
 * if (flag & TEXT) { ... }
 * ```
 */
export const enum PatchFlags {
  TEXT = 1,                    // 动态文本元素
  CLASS = 1 << 1,              // 2 动态绑定 class 的元素
  STYLE = 1 << 2,              // 4 动态绑定 style 的元素
  PROPS = 1 << 3,              // 8 动态 props 的元素, 且不含有 class style 的元素
  FULL_PROPS = 1 << 4,         // 16 动态 props 和带有 key 值绑定的元素
  HYDRATE_EVENTS = 1 << 5,     // 32 事件监听的元素
  STABLE_FRAGMENT = 1 << 6,    // 64 子元素的订阅不会改变的 Fragment 元素
  KEYED_FRAGMENT = 1 << 7,     // 128 自己或子元素带有 key 值绑定的 Fragment 元素
  UNKEYED_FRAGMENT = 1 << 8,   // 256 没有 key 值绑定的 Fragment 元素
  NEED_PATCH = 1 << 9,         // 512 带有 ref, 指令的元素, 有响应需求的
  DYNAMIC_SLOTS = 1 << 10,     // 1024 动态 slot 的组件元素
  DEV_ROOT_FRAGMENT = 1 << 11, // 
  HOISTED = -1,                // 静态元素
  BAIL = -2                    // 非 render 函数生成的元素不需要优化, 例如: renderSlot
}
````

具体的 patchFalg 有不少类型, 其中大致可分为两大类:

- 当 patchFlag 的值**大于** 0 时, 代表所对应的元素在 patchVNode 时或 render 时是可以被优化生成或更新的
- 当 patchFlag 的值**小于**0 时, 代表所对应的元素在 patchVNode 时, 是需要被 full diff的, 即是需要进行递归遍历 VNode tree 去进行比较更新

```js
// 这是render 函数中标记的参数
export function render(_ctx, _cache, $props, $setup, $data, $options) {
 return (_openBlock(), _createBlock("div", null, [
  _createVNode("p", null, "'HelloWorld'"),
  _createVNode("p", null, _toDisplayString(_ctx.msg), 1 /* TEXT */)
 
 ]))
}
```

**Vue3 对不参与更新的元素, 做静态标记, 这些元素只会被创建一次, 在渲染的时候直接复用.**

### Vue3 的diff方法
Vue 2 中进行diff的方法为`updateChildren` 

Vue3 中的是 `patchKeyedChildren`, 它的步骤是:
- 头和头对比
- 尾和尾对比
- **基于最长递增子序列进行移动/添加/删除**
- [Vue3 diff最长子序列](https://toutiao.io/posts/9vvtgt7/preview)

看个例子，比如:

> -   老的 children：`[ a, b, c, d, e, f, g ]`
> -   新的 children：`[ a, b, f, c, d, e, h, g ]`

1.  先进行头和头比，发现不同就结束循环，得到 `[ a, b ]`
2.  再进行尾和尾比，发现不同就结束循环，得到 `[ g ]`
3.  再保存没有比较过的节点 `[ f, c, d, e, h ]`，并通过 `newIndexToOldIndexMap` 拿到在数组里对应的下标，生成数组 `[ 5, 2, 3, 4, -1 ]`，`-1` 是老数组里没有的就说明是新增
4.  然后再拿取出数组里的最长递增子序列，也就是 `[ 2, 3, 4 ]` 对应的节点 `[ c, d, e ]`
5.  然后只需要把其他剩余的节点，基于 `[ c, d, e ]` 的位置进行移动/新增/删除就可以了

## Vue3 diff 的性能优化

Vue 函数式组件和有状态的组件(类组件)更新有什么区别?
1. 类组件在实例上挂载更新方法, 可以在外部执行更新. 函数式组件只能由props数据驱动更新, 判断是否更新的条件是: 是否有preNode
2. VNode 的 children 属性本应该用来存储子节点, 但是对于组件类型的 VNode 来说, 它的子节点都应该作为插槽存在, 并且我们选择将插槽内容存储在单独的 slots 属性中, 而非存储在 children 属性中
	1. 函数式组件的 使用 children 属性存储产出的 VNode , slots属性存储插槽数据
	2. 有状态组件类型使用其 children 属性存储组件实例，用 slots 属性存储插槽数据
