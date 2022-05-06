---
date created: 2021-12-09 22:54
date updated: 2021-12-17 16:16
---

#JavaScript

## 示例代码
### 例一:
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
 * 执行结果:
 * script start
 * async1 start
 * async2
 * promise1
 * script end
 * async1 end
 * promise2
 * setTimeout
 */
```
**代码分析:**
> 注意: new Promise() 内部的回调函数是当成同步函数执行的
> 注意: 执行到 await code 时, 会先执行 code , 再执行 await
1. 先执行同步代码
2. 首先执行 `console.log('script start')`
3. `setTimeout` 为宏任务, 先不执行
4. 执行 `async1` 函数 `console.log('async1 start')`; 以及 `async2()` ; `await` 由于是 `Promise.then` 的语法糖是异步代码, 暂不执行
5. `new Promise()` 内部代码执行, 后面的 `then` 内容先不执行
6. 执行 `console.log('script end')`
7. 同步代码执行结束, 开始按书写顺序执行微任务
8. 先执行 `console.log('async1 end')` ; 前面说过, `await` 下面的代码相当于 `then` 里的回调内容
9. 执行 `new Promise.then` 里面的内容 `console.log('promise2')`
10. 最后执行 宏任务的代码 `setTimeout()`

### 例二:
```js
setTimeout(() => {
  console.log('0');
}, 0)
new Promise((resolve, reject) => {
  console.log('1');
  resolve();
}).then(() => {
  console.log('2');
  new Promise((resolve, reject) => {
    console.log('3');
    resolve();
  }).then(() => {     // 📌
    console.log('4');
  }).then(() => {
    console.log('5');
  })
}).then(() => {
  console.log('6');   // 📌
})

new Promise((resolve, reject) => {
  console.log('7');
  resolve()
}).then(() => {
  console.log('8');
})
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
**代码分析:**
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

[参考链接](http://heyang.xyz/2021/10/08/%E4%B8%80%E7%AF%87%E6%96%87%E7%AB%A0%E8%A7%A3%E5%86%B3Promise-then%EF%BC%8Casync-await%E6%89%A7%E8%A1%8C%E9%A1%BA%E5%BA%8F%E7%B1%BB%E5%9E%8B%E9%A2%98/)