---
date created: 2021-12-09 22:51
date updated: 2021-12-09 22:55
---

#Vue

## keepAlive

keep-alive 是 vue 中一个抽象组件: 它自身不会渲染一个 DOM 元素, 也不会出现在父组件链中; 使用 keep-alive 包裹动态组件时, 会缓存不活动的组件实例, 而不会销毁它们

- 可以在动态组件中使用 包括 component
- 可以在 vue-router 中应用

3 个参数:

- `include` : 定义缓存白名单, keep-alive 会缓存命中的组件
- `exclude` : 定义缓存黑名单, 命中的组件不会被缓存
- `max` : 定义缓存组件的上限, 超过上限使用 [[LRU 缓存策略 - 手写实现]] 置换缓存数据

[彻底揭秘 keep-alive](https://github.com/qiudongwei/blog/issues/4)

## keepAlive 生命周期

组件一旦被 keep-alive 缓存, 那么再次渲染组件的时候就不会执行 `created` , `mounted` 等钩子函数, 但是业务场景需要我们在组件渲染时做一些事情, 所以 Vue 提供了 `activated` 钩子函数, 它的执行时机是 keep-alive 包裹的组件执行渲染的时候.

有 `activated` 钩子函数，也就有对应的 `deactivated` 钩子函数，它是发生在 `vnode` 的 `destory` 钩子函数，定义在 `src/core/vdom/create-component.js` 中：

参考资料:
[Vue.js 技术揭秘](https://ustbhuangyi.github.io/vue-analysis/v2/extend/keep-alive.html#%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F)

## 源码分析

```javascript
export default {
  name: "keep-alive",
  // 抽象组件属性 ,它在组件实例建立父子关系的时候会被忽略,发生在 initLifecycle 的过程中
  abstract: true,
  props: {
    // 被缓存组件
    include: patternTypes,
    // 不被缓存组件
    exclude: patternTypes,
    // 指定缓存大小
    max: [String, Number]
  },
  created() {
    // 初始化用于存储缓存的 cache 对象
    this.cache = Object.create(null);
    // 初始化用于存储VNode key值的 keys 数组
    this.keys = [];
  },
  destroyed() {
    for (const key in this.cache) {
      // 删除所有缓存
      pruneCacheEntry(this.cache, key, this.keys);
    }
  },
  mounted() {
    // 监听缓存（include）/不缓存（exclude）组件的变化
    // 在变化时，重新调整 cache
    // pruneCache：遍历 cache，如果缓存的节点名称与传入的规则没有匹配上的话，就把这个节点从缓存中移除
    this.$watch("include", val => {
      pruneCache(this, name => matches(val, name));
    });
    this.$watch("exclude", val => {
      pruneCache(this, name => !matches(val, name));
    });
  },
  render() {
    // 获取第一个子元素的 vnode
    const slot = this.$slots.default;
    const vnode: VNode = getFirstComponentChild(slot);
    const componentOptions: ?VNodeComponentOptions =
      vnode && vnode.componentOptions;
    if (componentOptions) {
      // name 不在 inlcude 中或者在 exlude 中则直接返回 vnode，否则继续进行下一步
      // check pattern
      const name: ?string = getComponentName(componentOptions);
      const { include, exclude } = this;
      if (
        // not included
        (include && (!name || !matches(include, name))) ||
        // excluded
        (exclude && name && matches(exclude, name))
      ) {
        return vnode;
      }

      const { cache, keys } = this;
      // 获取键，优先获取组件的 name 字段，否则是组件的 tag
      const key: ?string =
        vnode.key == null
          ? // same constructor may get registered as different local components
          // so cid alone is not enough (#3269)
          componentOptions.Ctor.cid +
          (componentOptions.tag ? `::${componentOptions.tag}` : "")
          : vnode.key;

      // --------------------------------------------------
      // 下面就是 LRU 算法了，
      // 如果在缓存里有则调整，
      // 没有则放入（长度超过 max，则淘汰最近没有访问的）
      // --------------------------------------------------
      // 如果命中缓存，则从缓存中获取 vnode 的组件实例，并且调整 key 的顺序放入 keys 数组的末尾
      if (cache[key]) {
        vnode.componentInstance = cache[key].componentInstance;
        // make current key freshest
        remove(keys, key);
        keys.push(key);
      }
      // 如果没有命中缓存,就把 vnode 放进缓存
      else {
        cache[key] = vnode;
        keys.push(key);
        // prune oldest entry
        // 如果配置了 max 并且缓存的长度超过了 this.max，还要从缓存中删除第一个
        if (this.max && keys.length > parseInt(this.max)) {
          pruneCacheEntry(cache, keys[0], keys, this._vnode);
        }
      }

      // keepAlive标记位
      vnode.data.keepAlive = true;
    }
    return vnode || (slot && slot[0]);
  }
};

// 移除 key 缓存
function pruneCacheEntry(
  cache: VNodeCache,
  key: string,
  keys: Array<string>,
  current?: VNode
) {
  const cached = cache[key]
  if (cached && (!current || cached.tag !== current.tag)) {
    cached.componentInstance.$destroy()
  }
  cache[key] = null
  remove(keys, key)
}

// remove 方法（shared/util.js）
/**
 * Remove an item from an array.
 */
export function remove(arr: Array<any>, item: any): Array<any> | void {
  if (arr.length) {
    const index = arr.indexOf(item)
    if (index > -1) {
      return arr.splice(index, 1)
    }
  }
}

```
