---
date created: 2021-12-09 22:58
---

#JavaScript

```javascript
// MyPromise 源码实现
const PENDING = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED = 'rejected';
class MyPromise {
  // 使用构造函数直接执行一个回调
  constructor(executor) {
    this._status = PENDING; // promise 初始状态为 pending
    this._value = undefined; // promise.then 回调的值初始
    this._resolveQueue = []; // resolove状态触发成功队列
    this._rejectQueue = []; // reject状态触发失败队列
    // resolve 回调
    const resolve = value => {
      // 把 resolve 封装进 run 函数
      const run = () => {
        // promise 规定 Promise 的状态只能从 pending 触发, 变成后两种状态
        if (this._status === PENDING) {
          this._status = FULFILLED // resolve 改变状态为 fulfilled
          this._value = value // 存储当前值用于 then 回调

          // 执行 resolve 回调 
          while (this._resolveQueue.length) {
            const callback = this._resolveQueue.shift()
            callback(value)
          }
        }
      }
      // 使用 setTimeout 实现 promise 异步调用特性
      setTimeout(run)
    }
    // reject 回调
    const reject = value => {
      const run = () => {
        if (this._status === PENDING) {
          this._status = REJECTED // 修改状态为 rejected
          this._value = value // 赋值
          while (this._rejectQueue.length) {
            const callback = this._rejectQueue.shift()
            callback(value)
          }
        }
      }
      setTimeout(run)
    }
    // 立即执行 executor 传入 resolve && reject
    executor(resolve, reject)
  }
  
  // 常用的 then 方法 接受成功回调和失败回调
  then = (onFulfilled, onRejected) => {
    // 如果 then 的参数不是 function 传 null 否则让值继续传递下去
    typeof onFulfilled !== "function" ? onFulfilled = value => value : null
    typeof onRejected !== "function" ? onRejected = error => error : null
    // then 返回一个新的 promise
    return new Mypromise((resolve, reject) => {
      const resolveFn = value => {
        try {
          const x = onFulfilled(value)
          // 判断返回值类型 如果是 promise 则等待状态变更, 否则直接 resolve
          x instanceof MyPromise ? x.then(resolve, reject) : resolve(x)
        } catch (error) {
          reject(error)
        }
      }
      const rejectFn = error => {
        try {
          const x = onRejected(error)
          // 判断返回值类型 如果是 promise 则等待状态变更, 否则直接 resolve
          x instanceOf MyPromise ? x.then(resolve, reject) : resolve(x)
        } catch (error) {
          reject(error)
        }
      }
      
      switch (this._status) {
        case PENDING:
          this._resolveQueue.push(resolveFn)
          this._rejectQueue.push(rejectFn)
          break;
        case FULFILLED:
          resolveFn(this._value)
          break;
        case REJECTED:
          rejectFn(this._value)
          break;
      }
      
    })
  }
  // .catch 
  catch(rejectFn) {
    return this.then(undefined, rejectFn)
  }
  // .finally
  finally(callback) {
    return this.then(value => MyPromise.resolve(callback()).then(() => value), error => {
      MyPromise.resolve(callback()).then(() => error)
    })
  }
  // 静态resolve方法
  static resolve(value) {
    return value instanceof MyPromise ? value : new MyPromise(resolve => resolve(value))
  }

  // 静态reject方法
  static reject(error) {
    return new MyPromise((resolve, reject) => reject(error))
  }
  // 静态all方法
  static all(promiseArr) {
    let count = 0
    let result = []
    return new MyPromise((resolve, reject) => {
      if (!promiseArr.length) {
        return resolve(result)
      }
      promiseArr.forEach((p, i) => {
        MyPromise.resolve(p).then(value => {
          count++
          result[i] = value
          if (count === promiseArr.length) {
            resolve(result)
          }
        }, error => {
          reject(error)
        })
      })
    })
  }
}
```
