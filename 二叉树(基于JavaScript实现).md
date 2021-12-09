---
date created: 2021-12-09 23:00
---

# 二叉树(基于 JavaScript 实现)

## 定义

Binary Tree 是一种树形结构的数据类型, 二叉树的特点是每个节点最多只能有两颗子树, 且有左右次序之分, 不能颠倒. 二叉树是 n 个有限元素的集合, 通常分支被称为[左子树]和[右子树].

## 二叉排序树

二叉排序树又称为 **二叉搜索树** 或 **二叉查找树**

**特点:**

> 1. 若它的左子树不为空, 则右子树上所有节点的值均小于它的根节点的值
> 2. 若它的右子树不为空, 则右子树上所有节点的值均大于它的根节点的值
> 3. 它的左右子树也分别为二叉搜索树

### JavaScript 实现

```js
function BinarySearchTree(keys) {
  // 构造节点
  let node = function(key) {
    this.key = key;
    this.left = null;
    this.right = null;
  }
  // 根节点, 最终返回一颗树结构
  let root = null;
  // 插入节点方法
  let insertNode = (node, newNode) => {
    if(newNode.key < node.key) {
      if(node.left === null) {
        node.left = newNode;
      } else {
        insertNode(node.left, newNode);
      }
    } else {
      if(node.right === null) {
        node.right = newNode;
      } else {
        insertNode(node.right, newNode);
      }
    }
  }
  // 构造树方法
  this.insert = (key) => {
    let newNode = new Node(key);
    if(root === null) {
      root = newNode;
    } else {
      insertNode(root, newNode);
    }
  }
  // 接收参数调用方法
  keys.forEach(key => {
    this.insert(key)
  })
  return root
}
// 调用:
const keys = [8, 3, 10, 1, 6, 14, 4, 7, 13];
BinarySearchTree(keys)
```

输出如下:

```bash
## 这里有一张图
      8
    /   \
   3     10
  / \     \
 1   6     14
    / \    /
   4   7  13
```

[遍历方法的非递归实现](https://juejin.cn/post/6844903456277266446)

### 中序遍历

中序遍历的递归定义: 先左子树, 后根节点, 再右子树

```js
// BST 中序遍历
let inOrderTraverseFn = (node, cb) => {
  if(node !== null) {
    inOrderTraverseFn(node.left, cb);
    cb(node.key);
    inOrderTraverseFn(node.right, cb);
  }
}
let callBack = (key) => {
  console.log(key)
}
// 调用
inOrderTraverseFn(BinarySearchTree(keys), callBack);
```

输出结果: 整颗**二叉树的节点以从小到大依次**打印出来

```js
1, 3, 4, 6, 7, 8, 10, 13, 14
```

### 前序遍历

前序遍历的递归定义: 先根节点, 后左子树, 再右子树

```js
// BST 前序遍历
let preOrderTraverseFn = (node , cb) => {
  if(node !== null) {
    cb(node.key);
    preOrderTraverseFn(node.left, cb);
    preOrderTraverseFn(node.right, cb);
  }
}
// 调用
preOrderTraverseFn(BinarySearchTree(keys), callBack);
```

输出结果:

```js
8, 3, 1, 6, 4, 7, 10, 14, 13
```

### 后序遍历

后序遍历的递归定义: 先左子树, 后右子树, 再根节点

```js
// BST 后序遍历
let postOrderTraverseFn = (node, cb) => {
  if(node !== null) {
    postOrderTraverseFn(node.left, cb);
    postOrderTraverseFn(node.right, cb);
    cb(node.key);
  }
}
// 调用
postOrderTraverseFn(BinarySearchTree(keys), callBack);
```

输出结果:

```js
1, 4, 7, 6, 3, 13, 14, 10, 8
```

## 应用 1: 查找 BST 最小值

也就是找二叉树左子树最左侧没有子节点的节点值

```js
// 查找 BST 最小值
let minNode = (node) => {
  if(node) {
    while(node && node.left !== null) {
      node = node.left;
    }
    return node.key
  }
  return null
}
console.log('minNode is' + minNode(BinarySearchTree(keys))) // 1
```

## 应用 2: 查找 BST 最大值

也就是查找二叉树右子树最右侧没有子节点的节点值

```js
// 查找 BST 最大值
let maxNode = (node) => {
  if(node) {
    while(node && node.right !== null) {
      node = node.right
    }
    return node.key
  }
  return null
}
console.log('maxNode is'+ maxNode(binarySearchTree(keys))) // 14
```

## 应用 3: 查找 BST 某个值

就是将该值与二叉树的每个节点进行比较, 如果比此节点小, 则进入左子树递归比较, 如果该值比此节点大, 则进入右子树递归比较

```js
// BST 查找某个值
let searchNode = (node, key) => {
  if(node === null){
    return false
  }
  if(key < node.key){
    return searchNode(node.left, key)
  } else if(key > node.key) {
    return searchNode(node.right, key)
  } else {
    return true
  }
}
// 
console.log(searchNode(BinarySearchTree(keys),3)?'node 3 is found':'node 3 is not found')
console.log(searchNode(BinarySearchTree(keys),5)?'node 5 is found':'node 5 is not found')
```

## 应用 4: 删除某个独立节点

也就是删除某个没有左子树和右子树的节点

```js
// 删除某个独立节点
let removeNode = (node, key) => {
  if(node === null) {
    return null
  }
  if(key < node.key) {
    node.left = removeNode(node.left, key)
    return node
  } else if(node > node.key) {
    node.right = removeNode(node.right, key)
    return node
  } else {
    if(node.left === null){
      return node.right;
    }
    if(node.right === null){
      return node.left;
    }
    if(node.left === null $$ node.right === null) {
      node = null
      return node
    }
  }
}
console.log(removeNode(BinarySearchTree(keys), 1), BinarySearchTree(keys))
```

结果:

```bash
## 这里有一张图
      8                            
    /   \
   3     10
  / \     \
 1   6     14
    / \    /
   4   7  13
```

```bash
      8
    /   \
   3     10
  / \     \
删除  6     14
    / \    /
   4   7  13
```

```js
class _LazyMan {
  constructor(name) {
    this.tasks = []
    const task = () => {
      console.log(`Hi! This is ${name}`)
      this.next()
    }
    this.tasks.push(task)
    setTimeout(() => {
      this.next()
    }, 0)
  }
  next() {
    const task = this.tasks.shift()
    task && task()
  }
  sleep(time) {
    this.sleepWrapper(time, false)
    return this
  }
  sleepFirst(time) {
    this.sleepWrapper(time, true)
    return this
  }
  sleepWrapper(time, first) {
    const task = () => {
      setTimeout(() => {
        console.log(`Wake up after ${time}`)
        this.next()
      }, time * 1000)
    }
    if (first) {
      this.tasks.unshift(task)
    } else {
      this.tasks.push(task)
    }
  }
  eat(food) {
    const task = () => {
      console.log(`Eat ${food}`);
      this.next();
    };
    this.tasks.push(task);
    return this;
  }
}

// 测试
const lazyMan = (name) => new _LazyMan(name)

lazyMan('Hank').sleep(1).eat('dinner')

lazyMan('Hank').eat('dinner').eat('supper')

lazyMan('Hank').eat('supper').sleepFirst(5)

```
