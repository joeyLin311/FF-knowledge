---
date created: 2021-12-09 22:55
date updated: 2021-12-17 16:15
---

#JavaScript

## Map 和 WeakMap

**Map** 可以接受各种类型的值当作 `key`，对同一个对象的引用才视为同一个键，否则取不到。

- 属性：get，set，has，delete，clear
- 遍历：keys，valuse，forEach，entries（遍历顺序就是插入顺序）

**WeakMap** 只接受对象作为键值，null 除外，键名所引用的对象都是弱引用，不会计入垃圾回收机制，一旦不需要会自动删除。

- 属性：get，set，has，delete

## Set 和 WeakSet

**Set** 成员惟一，不会有重复的值，判断和 `===` 差不多效果，区别在于 NaN 是等于自身的，`===` 是不相等的

- 属性：size，add，delete，has，clear
- 遍历：keys，values，entries，forEach（keys 和 values 一致）
- Array.from 方法可以将 Set 转化为 array

**WeakSet** 只能放对象，对象的弱引用，如果在其他地方无引用则垃圾回收

- 属性：add,delete,has 不能遍历

## Symbol

**symbol** 是一种基本数据类型 （ [primitive data type](https://developer.mozilla.org/zh-CN/docs/Glossary/Primitive) ）。`Symbol()`函数会返回**symbol**类型的值，该类型具有静态属性和静态方法。它的静态属性会暴露几个内建的成员对象；它的静态方法会暴露全局的symbol注册，且类似于内建对象类，但作为构造函数来说它并不完整，因为它不支持语法：`new Symbol()`。

每个从 `Symbol()` 返回的 symbol 值都是唯一的。一个 symbol 值能作为对象属性的标识符；这是该数据类型仅有的目的。
