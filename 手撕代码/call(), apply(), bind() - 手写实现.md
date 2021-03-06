---
date created: 2021-12-09 22:58
date updated: 2022-05-07 02:20
---

# JavaScript #编程题
ES6+ 相关API实现

## 实现 call()

`call(this, arg1, arg2, ...)`

1. 判断当前`this`是否为函数, 防止 `Function,prototype.myCall()` 直接调用
2. `context` 为可选参数, 如果不传的话默认上下文为 `window` 对象
3. 为 `context` 创建一个 `Symbol` 属性, 保证唯一性, 将当前函数赋值给这个属性
4. 处理参数, 传入第一参数后的剩余参数
5. 调用函数后立即删除 `Symbol` 属性

```jsx
Function.prototype.myCall = function(context = window, ...args) {
  if(this === Function.prototype) {
    // 用于防止 Function.prototype.myCall() 直接调用
 return undefined;
  }
  // 可选参数, 不传就是window对象
  context = context || window;
  // 创建 Symbol 属性
  const fn = Symbol();
  context[fn] = this;
  const result = context[fn](..args);
  delete context[fn];
  return result 
}
```

## 实现 apply()

`apply(this, [arg1, arg2, ...])`
与`call()` 实现类似, 只不过参数传递变成了数组

```javascript
Function.prototype.myApply = function(context = window, ...args) {
  if(this === Function.prototype) {
    // 防止自身直接调用
 return undefined;
  }
  const fn = Symbol();
  context[fn] = this;
  let result;
  if(Array.isArray(args)) {
    result = context[fn](..args);
  } else {
    result = context[fn]();
  }
  delete context[fn];
  return result;
}
```

## 实现 bind()

1. 处理参数，返回一个闭包
2. 判断是否为构造函数调用，如果是则使用`new`调用当前函数
3. 如果不是，使用`apply`，将`context`和处理好的参数传入

```javascript
Function.prototype.myBind = function (context,...args1) {
  if (this === Function.prototype) {
    throw new TypeError('Error')
  }
  const _this = this
  return function F(...args2) {
    // 判断是否用于构造函数
    if (this instanceof F) {
   return new _this(...args1, ...args2)
    }
    return _this.apply(context, args1.concat(args2))
  }
}
```

[JS中的call，apply，bind](https://segmentfault.com/a/1190000023445911)

## 三者的区别

- 三者都可以改变 `this` 的指向
- 三者第一个参数都是 `this` 要指向的对象, 如果没有传这个参数或者为 `undefined` 或 `null` 则默认指向全局 `window`
- `apply` 接受数组, `call` 接受参数列表, 且 `call` , `apply` 是一次性传入参数, 而 `bind` 可以分多次传入
- `bind` 是返回绑定 `this` 之后的函数, 且该函数永久改变了 `this` 的指向. `apply` 和 `call` 则是立即执行, 且 `this` 指向只会改变当前一次
