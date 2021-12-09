---
date created: 2021-12-09 22:54
---

#JavaScript

```javascript
async function async1() {
  console.log('async1 start');
  await async2();
  console.log('async1 end');
}
async function async2() {
  console.log('async2');
}
console.log('script start');
setTimeout(function () {
  console.log('setTimeout');
}, 0)
async1()
new Promise(function (resolve) {
  console.log('promise1')
  resolve()
}).then(function () {
  console.log('promise2')
})

console.log('script end')
/**
 1
 7
 2 
 3
 8
 4
 6
 5
 0
 */
```

代码分析：

1. 先执行同步代码
2. setTimeout 为宏任务，先不执行
3. new Promise里的代码作为同步代码，要执行 console.log('1'); 而then作为微任务，先不执行
4. 又是一个new Promise,所以和第三步同理。只执行 console.log('7');
5. 开始执行异步代码
6. 执行第一个new Promise里的then 即console.log('2');以及new Promise的同步代码 console.log('3');
7. 这步有点意思，这里不是执行console.log('4'); 而是执行console.log('8');
   1. 根据各console.log所在then层级来决定执行先后；层级越低的越先执行；同一层级的按照出现先后顺序来执行
8. 注释为📌的两个then是同层级的，所以按照执行顺序来打印
9. 执行第三个层级的then，所有微任务代码完成 10.执行宏任务代码，即console.log('0');
