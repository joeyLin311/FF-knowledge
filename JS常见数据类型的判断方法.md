#JavaScript

## typeof

```javascript
typeof '123' // "string"
typeof 123 // "number"
typeof [1, 2, 3] // "object"
typeof new Function() // "function"
typeof new Date() // "object"
// 引用类型判断出来的都是"object"
typeof Symbol() // "symbol"
typeof true // "true"
typeof null // "object"
typeof undefined // "undefined"
typeof true // "true"
```

## instance of

instanceof 后面一定要是**对象类型**, 并且严格大小写, 返回 true/false

```javascript
function instanceOf(left, right) {
  let leftValue = left.__proto__ // 取隐式原型
  let rightValue = right.prototype //  取显式原型
  white(true) {
    if (leftValue === null) {
      return false
    }
    // 当右边的显式原型严格等于左边的隐式原型时，返回true
    if (leftValue === rightValue) {
      return true
    }
    leftValue = leftValue.__proto__
  }
}

```

## 根据对象的 constructor 内部构造属性判断

constructor 判断方法跟 instanceof 相似,但是 constructor 检测 Object 与 instanceof 不一样,constructor 还可以处理基本数据类型的检测,不仅仅是对象类型
**constructor 不能判断 undefined 和 null，并且使用它是不安全的，因为 contructor 的指向是可以改变的**

## 使用 `Object.prototype.toString.call()`

(引用自红宝书): 在任何值上调用 Object 原生的 toString() 方法，都会返回一个 `object NativeConstructorName` 格式的字符串。每个类在内部都有一个 `[[Class]]` 属性，这个属性中就指定了上述字符串中的构造函数名。 但是它不能检测非原生构造函数的构造函数名。

| 类型     | toString                                     | 结果              |
| -------- | -------------------------------------------- | ----------------- |
| Map      | Object.prototype.toString.call(new Map())    | [object Map]      |
| Set      | Object.prototype.toString.call(new Set())    | [object Set]      |
| Array    | Object.prototype.toString.call([])           | [object Array]    |
| Object   | Object.prototype.toString.call({})           | [object Object]   |
| Symbol   | Object.prototype.toString.call(Symbol())     | [object Symbol]   |
| RegExp   | Object.prototype.toString.call(new RegExp()) | [object RegExp]   |
| Function | Object.prototype.toString.callFunction() {}) | [object Function] |

```js
// 可遍历的类型
const mapTag = '[object Map]';
const setTag = '[object Set]';
const arrayTag = '[object Array]';
const objectTag = '[object Object]';

// 不可遍历类型
const symbolTag = '[object Symbol]';
const regexpTag = '[object RegExp]';
const funcTag = '[object Function]';
```
