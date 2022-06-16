## 题目

给出原文字符串, 通过对字符串的每个字母进行改变来实现加密, 加密方式是在每一个字母`str[i]` 上偏移特定数组元素 `a[i]` 的量.

数组前三位已经赋值, `a[0]= 1` `a[1] = 2` `a[2] = 3`  当 i>= 3时, 数组元素 `a[i]= a[i-1]+a[i-2]+a[i-3]`

## 解法

```jsx
// 例子:
const cryptStr = (str, num) => {
  const arr = []
  for(let i = 0; i<str.length; i++) {
    const num1 = str.charCodeAt(i)
    const num2 = fn(i)
    const num3 = num1 +num2
    if(num3 >= 97 && num3 <= 122) {
      arr.push(String.formCharCode(num3))
    } else {
      const num4 = 97 + (num3 - 97)%26
      arr.push(String.fromCharCode(num4))
    }
  }
  function fn(num) {
    if(num==0) {
      return 1
    } else if(num === 1) {
      return 2
    } else if(num === 2) {
      return 4
    } else {
      return fn(num - 1) + fn(num - 2) + fn(num - 3)
    }
  }
}
```
