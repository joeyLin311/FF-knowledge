---
date created: 2022-05-30 22:50
date updated: 2022-06-09 01:15
---

#React

## 相关问题

- 解释一下 React 的 Diff 算法?
- 它与 Vue 的 Diff 有什么区别?
- [为什么 Vue 不需要 fiber](https://mp.weixin.qq.com/s?__biz=Mzk0MDMwMzQyOA==&mid=2247494750&idx=1&sn=7f74887f6e550c9307c265b0eae8d5e1&chksm=c2e11975f596906366283fcb87fa2afddd9a2b54f08ab1f0539f23a06177ff26b38e4588ff2d&scene=132#wechat_redirect)

## 概括

> 一个 DOM 节点, 在某个时刻最多会有 4 个节点与它相关

1.  current Fiber. 表示该 DOM 节点对应的 Fiber 节点
2.  workInProgress Fiber. 表示该节点本次更新需要渲染到页面中, 与 DOM 相对应的 Fiber 节点
3.  DOM 节点本身
4.  JSX 对象, 表示 render 方法的返回对象, 包含描述 DOM 节点信息

Diff 算法的本质上, 就是通过对比 `1` 和 `4` , 生成 `2`. 期间 React 会遍历 Fiber 树, 寻找需要变更的节点, 对其进行相应的更新删除或者移动. 期间由于 Fiber 数据结构的特性, 又区分单节点和多节点的情况进行比较操作.

## 详细

React  diff 发生在 render 流程中 `beginWork` 中调用 `reconcileChildren` 方法的阶段. 该方法判断 `current 是否为 null` 区分了组件两个不同的情况:

- 需要 `mount` 的组件会创建新的 ` 子 Fiber 节点 ` 调用 `mountChildFibers()`
- 需要 `update` 的组件, 方法内部会酱当前组件与该组件在上次更新时对应的 Fiber 节点比较(**diff 算法**), 将比较的结果生成新的 Fiber 节点. 它调用 `reconcileChildFibers()`, 该函数会根据 `child` 类型调用不同的处理函数

分析一下 `reconcileChildFibers()` 的入参情况

```TypeScript
workInProgress.child = reconcileChildFibers(
  returnFiber: Fiber,
  currentFirstChild: Fiber | null,
  newChild: any,
  lanes: Lanes,
)
```

- **returnFiber:** 作为父节点传入, 新生成的第一个 fiber 节点的 return 会指向它
- **currentFirstChild.child:** 旧 fiber 节点, diff 生成新 fiber 节点时会用新生成的 ReactElement 和它做比较
- **newChild:** 新生成的 ReactElement , 会以它为标准生成新的 fiber 节点
- **lanes:** 本次的渲染优先级, 最终会被挂载到新 fiber 的 lanes 属性

所以很明显, React 的 diff 算法比较的两个主体是 oldFiber(`curretn.child`) 和 newChildren(`nextChildren` ), 他们是两个不一样的数据结构.

## diff 的基本原则

我们知道在 Fiber 节点中, 每个 Fiber 节点的 `child` 指向它的子节点, 而 `sibling` 指向它右侧的兄弟节点这一数据结构. 对于需要更新的 Fiber node , 共有**节点更新**, **节点增删**, **节点移动**三种情况.

针对这种情况, React 的 diff 遵循一下策略:

- 同级别元素进行 Diff , 如果一个 DOM 节点在前后两次更新中跨越了层级, React 不会尝试去复用该节点.
- 即使两个元素的子树完全一样, 但前后的父级元素不同, 依照规则: 该元素及其子树会完全销毁, 并重建一个新元素及其子树, 不会尝试复用子树.
- 使用 `tag` 和 `key` 识别节点, 区分出前后的加点是否变化, 已达到尽量复用无变化节点的目的. 开发者可以通过设置 `key` 来暗示那些子元素在不同的渲染下能保持稳定.

## diff 过程

![[diff 过程.jpg]]
在子节点 diff 过程中, React 内部会将子节点分成 ` 单节点 ` 和 ` 多节点 ` 情况进行区别 diff:

- 类型为 `object` , `number`, `string` 的节点, 代表同级只有一个节点, 调用 `reconcileSingleElement` 方法处理
- 类型为 `Array` , 表示同级有多个节点

### 单节点情况

- 上次更新是的 Fiber 节点是否存在对应的 DOM 节点, 如果就生成新 Fiber 节点返回
- 如果是的话判断 DOM 节点是否可以复用
- 如果不能进行复用标记该 DOM 节点需要被删除, 并生成一个新的 Fiber 节点返回
- 如果可以复用, 就将上次更新的 Fiber 节点的副本作为本次新生成的 Fiber 节点返回

单节点中 React 判断 DOM 节点是否可以复用有三个条件:

- 首先比较 `key` 是否相同
- 比较 `type` 是否相同
- 细节注意
  - 当 `child !== null` 且 `key 相同 ` 且 `key 不同 ` 时执行 `deleteRemainingChildren` 将 `child` 及其兄弟 fiber 节点都标记删除
  - 当 `child !== null` 且 `key 不同 ` 时仅将 `child` 标记删除

### 多节点情况

当 Fiber 节点的 `child` 为多节点是, React 采用有点判断是否需要**更新**的节点, 优化 diff 过程, 所以采用了两轮遍历.

> React 中的的 `newChildren` 为数组形式, 但是和 `newChildren` 中每个组件比较的是 `current fiber` , 同级别的 Fiber 节点是由 `sibling` 指针链接而成的单链表, **所以不能使用双指针遍历**
> 即: `newChildren[0]` 与 `fiber` 比较, `newChildren[1]` 与 `fiber.sibling` 比较

- 第一轮遍历: 处理需要**更新**的节点
- 第二轮遍历: 处理剩下的**不属于更新**的节点

#### 第一轮遍历

1.  遍历 `newChildren` 与 `oldFiber` 节点比较, 判断 DOM 节点是否能复用
2.  如果可以复用, 就继续与 `oldFiber.sibling` (兄弟节点) 比较判断是否复用
3.  如果不能复用, 分两种情况

- `key` 不同不能复用, **立即跳出遍历, 第一遍遍历结束**
- `key` 相同但是 `type` 不同导致不能用, 则标记 `oldFiber` 为 `DELETION`, 然后继续遍历

4.  当 `newChildren` 遍历完或者 `oldFiber` 遍历完, **跳出遍历, 第一轮遍历结束**

如此遍历结束后会出现四种情况:

- `newChildren` 和 `oldFiber` 同时遍历完毕
  - 理想情况, 直接在第一轮遍历进行组件更新, diff 工作结束
- `newChildren` 未遍历完毕 / `oldFiber` 遍历完毕
  - 意味着次轮遍历有新节点插入, 需要遍历剩下的 `newChildren` 为生成的 `workInProgress fiber` 一次标记 `Placement`
- `newChildren` 遍历完毕 / `oldFiber` 未遍历完毕
  - 意味着本次更新有节点被删除了, 需要把未遍历的 `oldFiber` 节点标记为 `Deletion`
- `newChildren` 和 `oldFiber` 都没遍历完毕
  - 次情况意味新旧节点在此次更新种有移动位置, 此情况需要详细讨论

### 处理移动节点

React 使用 `key` 快速寻找 `oldFiber` 中对应的节点, 将他们存进 Map 结构中 `{key: key, value: oldFiber}`, 表示已经存在的 `child` 节点

### 标记节点是否移动

React 中是以最后一个可复用的节点在 `oldFiber` 的位置索引(使用变量 `lastPlacedIndex` 表示)

本次开始更新中的节点是按照 `newChildren` 的顺序排列. 在遍历 `newChildren` 的过程中, 每个**遍历到的可复用节点** 一定是当前遍历到的**所有可复用节点**中**最靠右的那个**, 即一定在 `lastPlacedIndex` 对应的 ` 可复用的节点 ` 在本次更新中位置的最后面.

我们只需要比较**遍历到的可复用节点** 在上次更新时是否也在 `lastPlacedIndex` 对应的 `oldFiber` 后面 , 就可以知道两次更新中这两个节点的相对位置是否发生了改变.

### 两个 Demo

#### demo1

```jsx
// 之前
abcd

// 之后
acdb

===第一轮遍历开始===
a（之后）vs a（之前）  
key不变，可复用
此时 a 对应的oldFiber（之前的a）在之前的数组（abcd）中索引为0
所以 lastPlacedIndex = 0;

继续第一轮遍历...

c（之后）vs b（之前）  
key改变，不能复用，跳出第一轮遍历
此时 lastPlacedIndex === 0;
===第一轮遍历结束===

===第二轮遍历开始===
newChildren === cdb，没用完，不需要执行删除旧节点
oldFiber === bcd，没用完，不需要执行插入新节点

将剩余oldFiber（bcd）保存为map

// 当前oldFiber：bcd
// 当前newChildren：cdb

继续遍历剩余newChildren

key === c 在 oldFiber中存在
const oldIndex = c（之前）.index;
此时 oldIndex === 2;  // 之前节点为 abcd，所以c.index === 2
比较 oldIndex 与 lastPlacedIndex;

如果 oldIndex >= lastPlacedIndex 代表该可复用节点不需要移动
并将 lastPlacedIndex = oldIndex;
如果 oldIndex < lastplacedIndex 该可复用节点之前插入的位置索引小于这次更新需要插入的位置索引，代表该节点需要向右移动

在例子中，oldIndex 2 > lastPlacedIndex 0，
则 lastPlacedIndex = 2;
c节点位置不变

继续遍历剩余newChildren

// 当前oldFiber：bd
// 当前newChildren：db

key === d 在 oldFiber中存在
const oldIndex = d（之前）.index;
oldIndex 3 > lastPlacedIndex 2 // 之前节点为 abcd，所以d.index === 3
则 lastPlacedIndex = 3;
d节点位置不变

继续遍历剩余newChildren

// 当前oldFiber：b
// 当前newChildren：b

key === b 在 oldFiber中存在
const oldIndex = b（之前）.index;
oldIndex 1 < lastPlacedIndex 3 // 之前节点为 abcd，所以b.index === 1
则 b节点需要向右移动
===第二轮遍历结束===

最终acd 3个节点都没有移动，b节点被标记为移动
```

#### demo2

```jsx
// 之前
abcd

// 之后
dabc

===第一轮遍历开始===
d（之后）vs a（之前）  
key改变，不能复用，跳出遍历
===第一轮遍历结束===

===第二轮遍历开始===
newChildren === dabc，没用完，不需要执行删除旧节点
oldFiber === abcd，没用完，不需要执行插入新节点

将剩余oldFiber（abcd）保存为map

继续遍历剩余newChildren

// 当前oldFiber：abcd
// 当前newChildren dabc

key === d 在 oldFiber中存在
const oldIndex = d（之前）.index;
此时 oldIndex === 3; // 之前节点为 abcd，所以d.index === 3
比较 oldIndex 与 lastPlacedIndex;
oldIndex 3 > lastPlacedIndex 0
则 lastPlacedIndex = 3;
d节点位置不变

继续遍历剩余newChildren

// 当前oldFiber：abc
// 当前newChildren abc

key === a 在 oldFiber中存在
const oldIndex = a（之前）.index; // 之前节点为 abcd，所以a.index === 0
此时 oldIndex === 0;
比较 oldIndex 与 lastPlacedIndex;
oldIndex 0 < lastPlacedIndex 3
则 a节点需要向右移动

继续遍历剩余newChildren

// 当前oldFiber：bc
// 当前newChildren bc

key === b 在 oldFiber中存在
const oldIndex = b（之前）.index; // 之前节点为 abcd，所以b.index === 1
此时 oldIndex === 1;
比较 oldIndex 与 lastPlacedIndex;
oldIndex 1 < lastPlacedIndex 3
则 b节点需要向右移动

继续遍历剩余newChildren

// 当前oldFiber：c
// 当前newChildren c

key === c 在 oldFiber中存在
const oldIndex = c（之前）.index; // 之前节点为 abcd，所以c.index === 2
此时 oldIndex === 2;
比较 oldIndex 与 lastPlacedIndex;
oldIndex 2 < lastPlacedIndex 3
则 c节点需要向右移动

===第二轮遍历结束===
```
