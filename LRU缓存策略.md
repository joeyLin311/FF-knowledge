#browser #Vue
[从keep-alive学习LRU算法](https://juejin.cn/post/6844904116620099591)
## 代码实现
### 模拟keep-alive
```javascript
// 第一步代码
class LRUCache {
  constructor(n) {
    this.size = n // 初始化最大缓存数据条数n
    this.data = new Map() // 初始化缓存空间map
  }

  // 第二步代码
  // domain 域名
  // info 一些信息
  put(domain, info) {
    if (this.data.size >= this.size) {
      // 删除最不常用数据
      const firstKey = [...this.data.keys()][0]
      // 次数不必当心data为空，因为this.size 一般不会取0，满足this.data.size >= this.size时，this.data自然也不为空。
      this.data.delete(firstKey)
    }
    // 写入数据
    this.data.set(domain, info)
  }

  // 第三步代码
  get(domain) {
    if (!this.data.has(domain)) {
      return false
    }
    //获取结果
    const info = this.data.get(domain)
    // 移除数据
    this.data.delete(domain)
    // 重新添加该数据
    this.data.set(domain, info)
    return info
  }
}
```
## LRU纯净版
### 一  数组 + 对象
```javascript
var LRUCache = function (capacity) {
  this.keys = []
  this.cache = Object.create(null)
  this.capacity = capacity
};

LRUCache.prototype.get = function (key) {
  if (this.cache[key]) {
    // 调整位置
    remove(this.keys, key)
    this.keys.push(key)
    return this.cache[key]
  }
  return -1
};

LRUCache.prototype.put = function (key, value) {
  if (this.cache[key]) {
    // 存在即更新
    this.cache[key] = value
    remove(this.keys, key)
    this.keys.push(key)
  } else {
    // 不存在即加入
    this.keys.push(key)
    this.cache[key] = value
    // 判断缓存是否已超过最大值
    if (this.keys.length > this.capacity) {
      removeCache(this.cache, this.keys, this.keys[0])
    }
  }
};

// 移除 key
function remove(arr, key) {
  if (arr.length) {
    const index = arr.indexOf(key)
    if (index > -1) {
      return arr.splice(index, 1)
    }
  }
}

// 移除缓存中 key
function removeCache(cache, keys, key) {
  cache[key] = null
  remove(keys, key)
}
```
### 二 Map 内置对象特性
Map 对象保存键值对是有序的, 在频繁地增删键值对时性能表现更好
```javascript
var LRUCache = function (capacity) {
  this.cache = new Map()
  this.capacity = capacity
}

LRUCache.prototype.get = function (key) {
  if (this.cache.has(key)) {
    // 存在即更新
    let temp = this.cache.get(key)
    this.cache.delete(key)
    this.cache.set(key, temp)
    return temp
  }
  return -1
}

LRUCache.prototype.put = function (key, value) {
  if (this.cache.has(key)) {
    // 存在即更新（删除后加入）
    this.cache.delete(key)
  } else if (this.cache.size >= this.capacity) {
    // 不存在即加入
    // 缓存超过最大值，则移除最近没有使用的
    this.cache.delete(this.cache.keys().next().value)
  }
  this.cache.set(key, value)
}
```