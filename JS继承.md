---
date created: 2021-12-09 22:55
---

#面试 #JavaScript

# JS 继承

**继承方式可分为是否使用 Object 函数**

## 1. 不使用 Object.create()

### 原型链继承

```javascript
// 父类
function superType() {
  console.log('I\'m father')
}
function subType() {}
subType.prototype = new superType()

// 所有涉及到原型链继承的继承方式都要修改子类构造函数的指向, 否则子类实例的构造函数会指向父类
subType.prototype.constructor = subType
```

把子类 prototype 设置为父类的实例, 就完成简单的继承, 称为原型链继承

**优点**: 继承父类所有方法

**缺点**:

- 父类的引用属性会被所有子类实例共享
- 子类构建实例时不能向父类传递参数

### 构造函数继承

```javascript
//在子类中写入
superType.call(this, subType)
```

将父类构造函数的内容复制给子类的构造函数, 唯一一个继承中不涉及 prototype 的继承

- **优点**: 实例的引用类型不共享, 也就是不继承父类的引用属性, 子类的构建实例可以向父类传递参数
- **缺点**: 父类方法不能复用, 子类实例的方法每次都是单独创建的

### 组合继承

就是原型链继承和构造函数继承的组合, 兼具两者**优点**

```javascript
// 父类
function superType() {
  this.name = 'parent'
  this.arr = [1, 2, 3]
}
// 父类原型方法
superType.prototype.say = function () {
  console.log('this is parent')
}

function subType() {
  // 构造函数继承
  superType.call(this) // 第二次调用superType
}
// 第一次调用superType, 属于原型链继承
subType.prototype = new superType()
```

**缺点**: 调用了两次父类构造函数, 第二次覆盖掉子类继承自父类的同名参数, 这种情况造成了一定程度上的性能浪费(内部是怎么工作的呢?)

## 2. 使用 Object.create

### 原型式继承

使用 ES6 新增的 `Object.create()` 规范了原型式继承, 接收两个参数, 1: 新创建对象的原型对象, 2: 可选参数, 可定义对象额外属性的对象参数, configurable, enmuerable 之类这种

### 寄生式继承

```javascript
function createObj(o) {
  var clone = Object.create(o)
  clone.sayName = function () {
    console.log('hi')
  }
  return clone
}
```

创建一个仅用于封装继承过程的函数，该函数在内部以某种形式来做增强对象，最后返回对象

### 寄生组合继承

```javascript
// 封装对象继承方法
function prototype(child, parent) {
  var prototype = object(parent.prototype) // 创建父类原型的浅复制
  prototype.constructor = child // 修正原型的构造函数
  child.prototype = prototype // 将子类的原型替换为该原型
}

// 当我们使用的时候：
prototype(Child, Parent)
```

用 `prototype` 函数替换了 `Child.prototype = new Parent()` ,从而避免了执行 `new Parent()`. MDN 给出的推荐语是: 这种方式的高效率体现它只调用了一次父类构造函数，并且因此避免了在 Parent.prototype 上面创建不必要的、多余的属性。与此同时, 原型链还能保持不变；因此, 还能够正常使用 instanceof 和 isPrototypeOf。开发人员普遍认为寄生组合式继承是引用类型最理想的继承范式

## 3. ES6 Class Extend

ES6 Class Extend 本质上是一种继承的语法糖, 跟组合寄生组合继承类似. 但是寄生组合继承是先创建子类实例 this 对象, 然后对其扩展. 而 Class Extend 是先将父类实例对象的属性方法添加到 `this` 上面(所以必须调用 `super` 方法), 然后用子类的构造函数修改 `this`

**class 关键字只是原型的语法糖, JavaScript 继承仍然是基于原型实现的**

```js
class Pet {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  showName() {
    console.log("调用父类的方法");
    console.log(this.name, this.age);
  }
}

// 定义一个子类
class Dog extends Pet {
  constructor(name, age, color) {
    super(name, age); // 通过 super 调用父类的构造方法
    this.color = color;
  }

  showName() {
    console.log("调用子类的方法");
    console.log(this.name, this.age, this.color);
  }
}
```
