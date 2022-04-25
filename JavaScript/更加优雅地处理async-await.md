---
date created: 2021-12-09 23:00
date updated: 2021-12-17 11:57
---

# JavaScript

一般项目中通常会使用`Promise`封装请求方法, 在实际使用中遇到多个接口嵌套调用时, `then`方法中也会出现嵌套的代码. 这个时候使用 `async/await`可以解决代码嵌套的问题, 可读性更好.
但是在`Promise`中我们可以使用`catch`捕获请求过程中出现的错误, 但是 `async/await`并没有相关的方法.

## 如何捕捉 `async/await`错误

### 使用 `try...catch()`

可以将 async 函数通过 try/catch 包裹起来, 此时 Prmise 如果达到 reject 状态, 那么 JS 将抛出一个可以被 try/catch 捕获的错误.

```javascript
async function catchErr() {
  try {
    await Promise.reject(new Error('something wrong'))
  } catch (err) {
    err.message; // 'something wrong'
  }
}
```

 但是在一些情况下使用 try/catch 也不完全可以正确捕获错误, 比如下面的代码, 在 try 语句中 return Promise, 这个时候被转变成了一个不可捕获的错误.

```javascript
async function catchErr() {
  try {
    // 此处为 return 
    return Promise.reject(new Error('something wrong'))
  } catch (err) {
    // 代码不会执行到这里
  }
}
```

还有一个缺点就是: 使用 try/catch 之后, 很难使用链式语法进行 Promise 的 then 操作.

### 使用类 Go 语言同时返回 Error 和 正确的值

npm 中有一个库叫 `await-to-js`, 它的源码揭示了这种处理方法:

```javascript
/**
 * @param { Promise } 传进去的请求函数
 * @param { Object= } errorExt - 拓展错误对象
 * @return { Promise } 返回一个Promise
 */
export function to(promise, errorExt) {
  return promise
    .then((data) => [null, data])
    .catch((err) => {
    if (errorExt) {
      const parsedError = Object.assign({}, err, errorExt);
      return [parsedError, undefined];
    }

    return [err, undefined];
  });
}

export default to;
```

其中`to`函数返回一个 Promise 且值是一个数组, 数组之中有两个元素, 遵循 error first , 如果索引为 0 的元素不为空值, 说明该请求报错, 如果索引 0 的元素为空值说明该请求没有报错.

## await-to-js

```javascript
/**
 * @param { Promise } promise
 * @param { Object= } errorExt - Additional Information you can pass to the err object
 * @return { Promise }
 */
export function to<T, U = Error> (
  promise: Promise<T>,
  errorExt?: object
): Promise<[U, undefined] | [null, T]> {
  return promise
    .then<[null, T]>((data: T) => [null, data]) // 执行成功，返回数组第一项为 null。第二个是结果。
    .catch<[U, undefined]>((err: U) => {
      if (errorExt) {
        Object.assign(err, errorExt);
      }

      return [err, undefined]; // 执行失败，返回数组第一项为错误信息，第二项为 undefined
    });
}

export default to;
```

正常情况下，await 命令后面是一个 Promise 对象，返回该对象的结果。如果不是 Promise 对象，就直接返回对应的值。

所以我们只需要利用 Promise 的特性，分别在 promise.then 和 promise.catch 中返回不同的数组，其中 fulfilled 的时候返回数组第一项为 null，第二个是结果。rejected 的时候，返回数组第一项为错误信息，第二项为 undefined。使用的时候，判断第一项是否为空，即可知道是否有错误，具体使用如下：

```js
import to from 'await-to-js';
// If you use CommonJS (i.e NodeJS environment), it should be:
// const to = require('await-to-js').default;

async function asyncTaskWithCb(cb) {
  let err, user, savedTask, notification;
  [ err, user ] = await to(UserModel.findById(1));
  if(!user) return cb('No user found');

  [ err, savedTask ] = await to(TaskModel({userId: user.id, name: 'Demo Task'}));
  if(err) return cb('Error occurred while saving task');

  if(user.notificationsEnabled) {
    [ err ] = await to(NotificationService.sendNotification(user.id, 'Task Created'));
    if(err) return cb('Error while sending notification');
  }

  if(savedTask.assignedUser.id !== user.id) {
    [ err, notification ] = await to(NotificationService.sendNotification(savedTask.assignedUser.id, 'Task was created for you'));
    if(err) return cb('Error while sending notification');
  }

    cb(null, savedTask);
}

```
