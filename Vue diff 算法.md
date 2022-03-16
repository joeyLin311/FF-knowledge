---
date created: 2021-12-09 22:57
---

#Vue

## diff 算法执行过程

执行过程:

### 在进行同级别节点比较的时候, 首先会对新老节点的 `dom` 数组的开始和结尾节点设置标记索引, 遍历的过程中移动索引值
### 在对开始节点和结束节点比较的时候, 总共有四种情况:
  1. `oldStartVnode` / `newStartVnode` (旧开始节点 / 新开始节点)
  2. `oldEndVnode` / `newEndVnode` (旧结束节点 / 新结束节点)
  3.  `oldStartVnode` / `oldEndVnode` (旧开始节点 / 新结束节点)
  4. `oldEndVnode` / `newStartVnode` (旧结束节点 / 新开始节点)
### 开始节点和结束节点比较, 有两种情况类似
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

-   当数据发生改变时，订阅者`watcher`就会调用`patch`给真实的`DOM`打补丁
-   通过`isSameVnode`进行判断，相同则调用`patchVnode`方法
-   `patchVnode`做了以下操作：
    -   找到对应的真实`dom`，称为`el`
    -   如果都有都有文本节点且不相等，将`el`文本节点设置为`Vnode`的文本节点
    -   如果`oldVnode`有子节点而`VNode`没有，则删除`el`子节点
    -   如果`oldVnode`没有子节点而`VNode`有，则将`VNode`的子节点真实化后添加到`el`
    -   如果两者都有子节点，则执行`updateChildren`函数比较子节点
-   `updateChildren`主要做了以下操作：
    -   设置新旧`VNode`的头尾指针
    -   新旧头尾指针进行比较，循环向中间靠拢，根据情况调用`patchVnode`进行`patch`重复流程、调用`createElem`创建一个新节点，从哈希表寻找 `key`一致的`VNode` 节点再分情况操作

## Vue3 对 diff 算法的改进

vue2.x中的虚拟dom是进行**全量的对比**，在运行时会对所有节点生成一个虚拟节点树，当页面数据发生变更好，会遍历判断virtual dom所有节点（包括一些不会变化的节点）有没有发生变化；虽然说diff算法确实减少了多DOM节点的直接操作，但是这个**减少是有成本的**，如果是复杂的大型项目，必然存在很复杂的父子关系的VNode,**而Vue2.x的diff算法，会不断地递归调用patchVNode，不断堆叠而成的几毫秒，最终就会造成 VNode 更新缓慢**。

