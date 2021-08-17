## 字符串：415. 字符串相加

**描述：给定两个字符串形式的非负整数** `**num1**` **和** `**num2**` **，计算它们的和。**

模拟竖式计算方法，对齐末尾分别计算数位，超过10就进一位。需要注意两个数字位数不同的话就需要对较短的进行补零操作，解决两个数字位数不同的情况。

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

有两种办法可以实现反转链表的操作：一是循环迭代每个节点并且把节点的prev和next指针通过第三方变量进行互换，二是使用递归， 需要注意 `head.next.next` 是指向它本身，还有反转的第一个item必须指向null

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