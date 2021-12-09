---
date created: 2021-12-09 22:55
---

#编程题 #JavaScript
**new 运算符**
创建一个用户定义的对象类型的实例或具有构造函数的内置对象的实例

1. 创建新的对象, 如果该函数不是 JS 内置, 则创建一个新的 Object 对象

2. 将这个空对象的 `__proto__` 属性链接到**构造函数的原型对象**, 构造原型链找原型, 实现继承

3. 绑定 `this` , 将步骤 1 创建的对象作为 `this` 的上下文 , 执行构造函数中的代码, 也就是给这个新对象添加属性

4. 返回一个新的对象, 如果函数没有返回其它对象, 则自动返回这个新对象; 如果函数有返回的是非对象成员, 则返回 `this`

如果函数没有使用 `new` 运算符, 构造函数会像其它常规函数一样被调用, 并不会创建一个对象, 这种情况加 `this` 的指向也是不一样的.

## new 运算符符实现

[[编程题]]

```javascript
// 自实现 new 操作符
function _new() {
  // 1.创建一个新对象
  const obj = {}
  // 2. 绑定原型
  let [constructor, ...args] = [...arguments]
  obj.__proto__ = constructor.prototype
  // 3. 创建this上下文
  const result = constructor.apply(obj, args)
  // 4. 返回新的对象
  return Object.prototype.toString.call(result) === "[object Object]" ? result : obj
}
// use 
function car(name, color) {
  this.name = name
  this.color = color
}
let bmw = _new(car,'bmw','red')
console.log(bmw)
```
