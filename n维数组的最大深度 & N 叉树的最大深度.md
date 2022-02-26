---
date created: 2021-12-16 17:22
date updated: 2021-12-17 16:15
---

# n维数组的最大深度

#编程题 #leetcode

## 解法一: 普通版

```js
function maxDepth(arr, count = 0, newArr = []) {
  for(const child of arr){
	  if(Array.isArray(child)) {
		  count++
			maxDepth(child, count, newArr)
		} else {
		  newArr.push(count)
		}
	}
	if(newArr.length > 0) {
	  return Math.max(...newArr) + 1
	}
}
```

## 解法二:  简化版

 DFS 深度优先
```js
const getArrayDepth = (arr) => {
 return Array.isArray(arr) ? 
	 Math.max(...arr.map(item => getArrayDepth(item))) + 1 : 0;
}
```
