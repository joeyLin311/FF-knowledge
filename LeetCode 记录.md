---
date created: 2021-12-09 22:55
---

## 字符串：415. 字符串相加

#算法 #leetcode
**描述：给定两个字符串形式的非负整数** `**num1**` **和** `**num2**` **，计算它们的和。**

模拟竖式计算方法，对齐末尾分别计算数位，超过 10 就进一位。需要注意两个数字位数不同的话就需要对较短的进行补零操作，解决两个数字位数不同的情况。

```javaScript
var addStrings = function(num1, num2) {
  // 设置i, j指向字符串末尾，记录当前位数相加是否有进位
  let i = num1.length - 1, j = num2.length - 1, add = 0;
  // 记录每个数位的计算结果
  const ans = [];
  while (i >= 0 || j >= 0 || add != 0) {
    // 下标位置大于0就把当前位数转为number 如果小于0 就把当前下标位置补0
    const x = i >= 0 ? num1.charAt(i) - '0' : 0;
    const y = j >= 0 ? num2.charAt(j) - '0' : 0;
    // 计算当前数位的结果（记得进位的add变量）
    const result = x + y + add;
    // 用10取当前位数的余数 例如当前数位得出12，该数位就是2
    ans.push(result % 10);
    // 记录进位，如: 12 => floor(1.2) => 1 
    add = Math.floor(result / 10);
    // 移动下标
    i -= 1;
    j -= 1;
  }
  // 返回值应该翻转，因为ans数组存储的顺序是从个位开始的所以reverse后再返回
  return ans.reverse().join('');
};

```

## 链表：206. 反转链表

**描述：给定一个单链表和一个头节点** `**head**` **反转它并返回**

有两种办法可以实现反转链表的操作：一是循环迭代每个节点并且把节点的 prev 和 next 指针通过第三方变量进行互换，二是使用递归， 需要注意 `head.next.next` 是指向它本身，还有反转的第一个 item 必须指向 null

```
// 循环迭代方法
var reverseList = (head) => {
  // 初始化前节点
  let prev = null
  // 记录迭代的当前节点
  let curr = head
  // 执行循环迭代，进行前后节点替换
  while (curr) {
    // 声明中间变量 存放当前节点的下一个节点
    const tmpNext = curr.next
    // 第一个next转换也就是初始化的 prev = null
    curr.next = prev
    // 把 prev 移动到当前项
    prev = curr
    // 此时 curr 来到了 curr[n+1]
    // 然后把之前的下个节点赋值给当前节点，完成指针互换
    curr = tmpNext
  }
  // 返回节点
  return prev
}

// 递归
var reverseList = (head) => {
  // 边界条件，head或者head.next 是null证明已经到头了
  if(head === null || head.next === null){ return head }
  // 声明新的节点，递归调用传入节点的下一节点
  const newHead = reverseList(head.next)
  // 然后操作指针, 其实就是把当前传入的节点的下一个节点变成自己
  head.next.next = head
  // 这里必须指向null，不然链表会产生环
  head.next = null
  return newHead
}
```

## n 数之和

[github](<[https://github.com/sisterAn/JavaScript-Algorithms/issues/128](https://github.com/sisterAn/JavaScript-Algorithms/issues/128) >)
[blog with Golang](<[https://set.sh/post/200719-how-to-solve-n-sum-problem](https://set.sh/post/200719-how-to-solve-n-sum-problem) >)
[leetCode](<[https://leetcode-cn.com/problems/3sum/solution/jsban-jie-ti-fang-an-ke-neng-shi-dai-ma-liang-zui-/](https://leetcode-cn.com/problems/3sum/solution/jsban-jie-ti-fang-an-ke-neng-shi-dai-ma-liang-zui-/) >)

### 三数之和

```javascript
给定数组, (nums = [-1, 0, 1, 2, -1, -4])
满足要求的三元组合集为: [
  [-1, 0, 1],
  [-1, -1, 2],
]
```

**思路:**

1. 题目中可能会出现多组结果, 所以需要考虑去重
2. 为了方便去重, 需要先将数组进行排序
3. 对数组进行遍历, 取当前遍历的数 `num[i]` 为一个基准数, 遍历数后面的数组为寻找数组
4. 再寻找数组中设定两个起点, 最左侧的 `left(i+1)` 和最右侧的 `right(length-1)`
5. 判断 `nums[i] + nums[left] + nums[right]` 是否等于 0, 如果等于 0 就加入结果, 并将 `left` 和 `right` 移动一位
6. 如果结果大于 0, 将 `right` 向左移动一位, 向结果逼近
7. 如果结果小于 0, 将 `left` 向右移动一位, 向结果逼近

注意在整个过程中需要考虑元素重复的问题, 需要去重

```javascript
const threeSum = function (nums) {
  const result = []
  // 升序排序
  nums.sort((a, b) => a - b)
  // 执行遍历操作
  for (let i = 0; i < nums.length; i++) {
    // 跳过重复数字
    if (i && nums[i] === nums[i - 1]) {
      continue
    }
    // 设置左右起点下标
    let left = i + 1
    let right = nums.length - 1
    while (left < right) {
      const sum = nums[i] + nums[left] + nums[right]
      if (sum > 0) {
        right--
      } else if (sum < 0) {
        left++
      } else {
        result.push([nums[i], nums[left++], nums[right--]])
        // 判断元素重复, 跳过重复项
        while (nums[left] === nums[left - 1]) {
          left++
        }
        // 同上
        while (nums[right] === nums[right + 1]) {
          right--
        }
      }
    }
  }
  return result
}
```

### 四数之和

```javascript
var fourSum = function (nums, target) {
  if (nums.length < 4) {
    return []
  }
  nums.sort((a, b) => a - b)
  const result = []
  for (let i = 0; i < nums.length - 3; i++) {
    if (i > 0 && nums[i] === nums[i - 1]) {
      continue
    }
    if (nums[i] + nums[i + 1] + nums[i + 2] + nums[i + 3] > target) {
      break
    }
    // 比三数之和多一个循环,找出第四个数
    for (let j = i + 1; j < nums.length - 2; j++) {
      if (j > i + 1 && nums[j] === nums[j - 1]) {
        continue
      }
      let left = j + 1,
        right = nums.length - 1
      while (left < right) {
        const sum = nums[i] + nums[j] + nums[left] + nums[right]
        if (sum === target) {
          result.push([nums[i], nums[j], nums[left], nums[right]])
        }
        if (sum <= target) {
          while (nums[left] === nums[++left]);
        } else {
          while (nums[right] === nums[--right]);
        }
      }
    }
  }
  return result
}
```

### 有无更加通用的 n 数求和的方法呢?

#### 解法一: 使用 DFS 深度优先 Depth-First Search

```javascript
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[][]}
 */
var nSum = function (nums, target) {
  const helper = (index, N, temp) => {
    // 如果下标越界了或者 N < 3 就没有必要在接着走下去了
    if (index === len || N < 3) {
      return
    }
    for (let i = index; i < len; i++) {
      // 剔除重复的元素
      if (i > index && nums[i] === nums[i - 1]) {
        continue
      }
      // 如果 N > 3 的话就接着递归
      // 并且在递归结束之后也不走下边的逻辑
      // 注意这里不能用 return
      // 否则循环便不能跑完整
      if (N > 3) {
        helper(i + 1, N - 1, [nums[i], ...temp])
        continue
      }
      // 当走到这里的时候，相当于在求「三数之和」了
      // temp 数组在这里只是把前面递归加入的数组算进来
      let left = i + 1
      let right = len - 1
      while (left < right) {
        let sum = nums[i] + nums[left] + nums[right] + temp.reduce((prev, curr) => prev + curr)
        if (sum === target) {
          res.push([...temp, nums[i], nums[left], nums[right]])
          while (left < right && nums[left] === nums[left + 1]) {
            left++
          }
          while (left < right && nums[right] === nums[right - 1]) {
            right--
          }
          left++
          right--
        } else if (sum < target) {
          left++
        } else {
          right--
        }
      }
    }
  }
  let res = []
  let len = nums.length
  nums.sort((a, b) => a - b)
  helper(0, 4, [])
  return res
}
```

#### 解法二: 回溯递归

```javascript
function getAllCombin(array, n, sum, temp) {
  if (temp.length === n) {
    if (temp.reduce((t, c) => t + c) === sum) {
      return temp
    }
    return
  }
  for (let i = 0; i < array.length; i++) {
    const current = array.shift()
    temp.push(current)
    const result = getAllCombin(array, n, sum, temp)
    if (result) {
      return result
    }
    temp.pop()
    array.push(current)
  }
}
const arr = [1, 2, 3, 4, 5, 6]

console.log(getAllCombin(arr, 3, 10, []))
```

## 二叉树

[[二叉树(基于 JavaScript 实现)]]

## 杨辉三角

[[杨辉三角]] 杨辉三角

## 连续区间类算法

[[连续区间]] 连续区间
