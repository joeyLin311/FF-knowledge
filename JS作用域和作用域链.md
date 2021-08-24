#面试 #JavaScript 
avaScript 在 ES6 之前是使用**函数作用域**和**全局作用域**, 在 ES6 之后, 有了**块级作用域.** JavaScript 采用的是静态作用域又叫词法作用域, 在JS编译时静态确定的作用域. 一个程序的静态结构就决定了一个变量的作用域, 这个作用域不会因函数的位置改变而改变.

**作用域:** **即函数或变量的可见区域**

**函数作用域**: 用函数形式`function(){ ... }`包裹起来的区域即是函数作用域.

与函数作用域对应的是**全局作用域.** 就是定义在函数体最外部的变量或者函数, 可以在任何地方访问到.

```javascript
var a = '123' // 全局作用域 
function func() { 
  var b = '456' // 函数作用域 
} 
console.log(b) // 无法在函数作用域外部访问到变量b
```
![作用域图](https://cdn.nlark.com/yuque/0/2021/png/223223/1616510459969-58d4d9af-871d-4708-b6d4-81d04ed9d144.png)
**VO: 变量对象(Variable Object), AO: 活动对象(Activation Object)**

在 JavaScript 中有一个内部属性 Scope 来记录函数的作用域。在函数调用的时候，JavaScript 会为这个函数所在的新作用域创建一个环境，这个环境有一个外层域，它通过 `Scope`  创建并指向了外部作用域的环境. 因此在JavaScript 中存在一个作用域链, 它以当前作用域为起点, 链接的外部的作用域, 每个作用域链最终会在全局环境里终结. 全局作用域的外部作用域指向了 null.

**但是函数的 `scope` 属性并不是完成的作用域链**

```javascript
function foo() { 
	function bar() { ...} 
} // 函数创建后各自的[[scope]]为 
foo.[[scope]] = [ globalContext.VO ]; 
bar.[[scope]] = [ fooContext.AO, globalContext.VO ]; 
// 所以对于某个特定的函数,单看[[scope]]并不是完整的作用域链
```
**作用域链, 是由当前环境与上层环境的一系列变量对象组成, 它保证了当前执行环境对符合访问权限的变量和函数的有序访问.**

**作用域是一套规则, 是在JavaScript 引擎编译的时候确定的.**

**作用域链是在执行上下文的创建阶段创建的, 这是在 JavaScript 引擎解释执行阶段确定的.**

[从 JavaScript 作用域说开去](https://halfrost.com/javascript_scope/)