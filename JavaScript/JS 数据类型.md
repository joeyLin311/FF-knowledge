数据类型分类
======

一共 8 种数据类型  
基础数据类型：`Number` `String` `Null` `Boolean` `Undefined` `Symbol` `Bigint`  
引用数据类型：`Object`

```js
注意：
function 和 array 不是数据类型 广义上，他们属于对象。

NaN 也不是数据类型 
typeof NaN   // number  
NaN === NaN  // false
```

储存方式
====

**基础数据类型：**  
存储在栈（stack）中，占据空间小、大小固定，属于被频繁使用数据，所以放入栈中存储。  
**引用数据类型：**  
同时存储在栈（stack）和堆（heap）中，占据空间大、大小不固定。 栈中存储了指针，该指针指向堆中该实体的起始地址。

区别
==

基础数据类型的赋值会重新开辟一个内存空间，两个值互不干扰，完全独立。

```js
let a = 1；
let b = a；
b = 2
console.log(a,b) // 1 2
```

引用数据类型 会复制一份引用地址给赋值的对象，它们共同指向了同一个堆内存放的对象，一个对象的值改变，另外一个也会跟着改变。如果一个对象重新赋值，则不会再影响另外一个对象。

```js
let a = {name:"jimmy"}
let b = a;
b.name = "chimmy";
console.log(a) // {name:"chimmy"}
b = {name:"jeck"}
console.log(a) // {name:"chimmy"}
console.log(b) // {name:"jeck"}
```

```js
let a = {name:'jimmy',age:22}
function change(obj){
 obj.age = 24;
 obj = {
   name:'chimmy',
   age:21
 }
 return obj;
}
let b = change(a);
console.log(b.age); // 21
console.log(a.age); // 24
```

函数传参进来的 obj，传递的是 obj 在堆中的内存地址，通过调用 obj.age = 24，改变了 a 对象的 age 属性, 但 obj = { name:'chimmy', age:21 } 却又把 obj 变成了另一个内存地址，将 { name:'chimmy', age:21 } 存入其中，最后返回 b 的值就变成了 { name:'chimmy', age:21 }。而 a 打印的结果为 {name:'jimmy',age:24}

_注意_：对于引用类型的变量，== 和 === 只会判断引用的地址是否相同，而不会判断对象具体的属性以及值是否相同。因此，如果两个变量指向相同的对象，则返回 true。  
所以 {}==={},[]===[],[1,2] ===[1,2],{name:'jimmy'}==={name:"jimmy"} 都是 false  
如果想判断两个不同的对象或者数组是否真的相同，一个简单的方法就是将它们转换为字符串然后判断。 另一个方法就是递归地判断每一个属性的值，直到类型为基本类型，然后判断是否相同。

数据类型检测
======

typeof
------

**基础数据类型**

```js
typeof "string"     // string  
typeof 1            // number  
typeof undefined   // undefined  
typeof true        // boolean  
typeof Symbol()    // symbol  
typeof null        // object 
```

基础数据类型，typeof 只有 null 的判定会有误差。

注意点：

```js
typeof NaN  // number
typeof Number(1) // number; // Number会尝试把参数解析成数值

NaN是一个比较特殊的值，它不等于自身  NaN === NaN 或者 NaN == NaN 都为fasle
要想准确判断是否为NaN，
1.可以使用 Object.is(NaN,NaN) 来判断
2.可以利用NaN不等于自身这一特性来判断
function selfIsNaN(value){
  return value !== value
}
3.ES6 在 Number 对象上提供的isNaN()方法
Number.isNaN(NaN); // true　本身是NaN
```

**引用数据类型**

```js
typeof {}           // 'object'
typeof []           // 'object'
typeof(() => {})    // 'function'
```

typeof 操作符会对对象类型及其子类型，譬如函数（可调用对象）、数组（有序索引对象）等进行判定，除了函数其他都会得到 object 的结果。  
综上：使用 typeof 检测数据类型无法准确识别 null 数组 对象等，有缺陷。

instanceof
----------

通过 instanceof 操作符可以对对象类型进行判定，其原理就是测试构造函数的 prototype 是否出现在被检测对象的原型链上。

```js
[] instanceof Array            // true
({}) instanceof Object         // true
(()=>{}) instanceof Function   // true
```

instanceof 只能准确判断出数组和方法，不能准确判断出对象

```js
let arr = [];
let fuc = () => {};
arr instanceof Object // true
fuc instanceof Object // true
```

Array 和 Function 属于 Object 子类型，所以 Object 构造函数在 arr 和 fuc 的原型链上。所以

```js
arr instanceof Object // true  
fuc instanceof Object // true
```

Object.prototype.toString
-------------------------

toString() 是 Object 的原型方法，调用该方法，可以统一返回格式为 “[object Xxx]” 的字符串，其中 Xxx 就是对象的类型。对于 Object 对象，直接调用 toString() 就能返回 [object Object]；而对于其他对象，则需要通过 call 来调用，才能返回正确的类型信息。

PS: 调用 call 是因为有些数据类型没有 toString 的方法，使用 call，可以改变 this 的指向。相当于借用了 Object.prototype.toString 这个方法。关于 call，可以参考我的另外一篇文章 [new,call,apply,bind 知多少](https://juejin.cn/post/6982067582006198280 "https://juejin.cn/post/6982067582006198280")

```js
Object.prototype.toString({})       // "[object Object]"
Object.prototype.toString.call({})  // 同上结果，加上call也ok
Object.prototype.toString.call(1)    // "[object Number]"
Object.prototype.toString.call('1')  // "[object String]"
Object.prototype.toString.call(true)  // "[object Boolean]"
Object.prototype.toString.call(function(){})  // "[object Function]"
Object.prototype.toString.call(null)   //"[object Null]"
Object.prototype.toString.call(undefined) //"[object Undefined]"
Object.prototype.toString.call(/123/g)    //"[object RegExp]"
Object.prototype.toString.call(new Date()) //"[object Date]"
Object.prototype.toString.call([])       //"[object Array]"
Object.prototype.toString.call(document)  //"[object HTMLDocument]"
Object.prototype.toString.call(window)   //"[object Window]"
```

Object.prototype.toString.call() 可以准确的判断出数据类型，甚至可以把 document 和 window 都区分开来。  
实现一个通用的判断数据类型的方法

```js
function getType(type){
  if(typeof type !== "object"){
    return typeof type;
  }
  return Object.prototype.toString.call(type).slice(8, -1).toLowerCase();
}
```
[[JS 常见数据类型的判断方法]]
数据类型转换
======

强制类型转换
------

### 转为字符串

String()

```js
String(null) // "null"
String(undefined) // "undefined"
String(true) // "true"
String([]) // " "
String([1,2]) // "1,2"
String({}) // “[object Object]”
String(function(){ }) // “function (){}”
```

### 转为数字

**Number()**

转换规则

1.  如果是布尔值，true 和 false 分别被转换为 1 和 0；
2.  如果是数字，返回自身；
3.  如果是 null，返回 0；
4.  如果是 undefined，返回 NaN；
5.  如果是字符串，遵循以下规则：如果字符串中只包含数字（或者是 0X / 0x 开头的十六进制数字字符串，允许包含正负号），则将其转换为十进制；如果字符串中包含有效的浮点格式，将其转换为浮点数值；**如果是空字符串，将其转换为 0**；如果不是以上格式的字符串，均返回 NaN；

6. 如果是 Symbol，抛出错误；  
7. 如果是对象，调用对象的 valueOf() 方法，然后依据前面的规则转换返回的值；如果转换的结果是 NaN ，则调用对象的 toString() 方法，再次依照前面的顺序转换返回对应的值。

第七点简单的规则也可以这样理解，Number 方法的参数是对象时，将返回 NaN，除非是包含单个数值的数组或者空数组。

Number 函数将字符串转为数值，要比 parseInt 函数严格很多。基本上，只要有一个字符无法转成数值，整个字符串就会被转为 NaN。

```js
Number(true);        // 1
Number(false);       // 0
Number([]);          // 0
Number({});          // NaN
Number('0111');      // 111
Number(null);        // 0
Number('');          // 0
Number('1a');        // NaN
Number(-0X11);       // -17
Number('0X11')       // 17
```

**parseInt() 和 parsefloat()**  
首先查看位置 0 处的字符，判断它是否是个有效数字；如果不是，该方法将返回 NaN(not a number)，不再继续执行其他操作。但如果该字符是有效数字，该方法将查看位置 1 处的字符，进行同样的测试。这一过程将持续到发现非有效数字的字符为止，然后把该字符之前的有效数字字符串转换成数字。如下图：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/327b571a0f194eb7a5e92f3088966c38~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 转换为布尔值

Boolean() 除了 undefined、 null、 false、 ''、 0（包括 +0，-0）、 NaN 转换出来是 false，其他都是 true。

隐式类型转换
------

通过逻辑运算符 (&&、 ||、 !)、运算符 (+、-、*、/)、关系操作符 (>、 <、 <= 、>=)、相等运算符 (==) 或者 if/while 条件的操作，如果遇到两个数据类型不一样的情况，都会出现隐式类型转换。

### !!

会隐式转化为 boolean 进行比较

### '==' 的隐式类型转换规则

1.  规则 1：如果其中一个操作值是 null 或者 undefined，那么另一个操作符必须为 null 或者 undefined，才会返回 true，否则都返回 false
2.  规则 2：两个操作值如果为 string 和 number 类型，那么就会将字符串转换为 number
3.  规则 3：如果一个操作值是 boolean，那么转换成 number
4.  规则 4：如果其中一个是 Symbol 类型，那么返回 false；
5.  规则 5：如果一个操作值为 object 且另一方为 string、number 或者 symbol，就会把 object 转为原始类型再进行判断（调用 object 的 valueOf/toString 方法进行转换）。

```js
null == undefined  // true  规则1
null == 0  // false 规则1  
null == "" // false 规则1
'123' == 123   // 规则2  
""== 0 0==false 1==true // 都为true   // 规则3     
var a = {
  value: 0,
  valueOf: function() {
    this.value++;
    return this.value;
  }
};
// 注意这里a又可以等于1、2、3
console.log(a == 1 && a == 2 && a ==3);  //true Object隐式转换 规则5
```

实际开发中，用 === 代替 == 判断是否相等

### '+' 的隐式类型转换规则

'+' 号操作符，不仅可以用作数字相加，还可以用作字符串拼接。仅当 '+' 号两边都是数字时，进行的是加法运算；如果两边都是字符串，则直接拼接，无须进行隐式类型转换。

1.  如果其中有一个是字符串，另外一个是 undefined、null 或布尔型，则调用 toString() 方法进行字符串拼接；如果是纯对象、数组、正则等，则默认调用对象的转换方法会存在优先级，然后再进行拼接。
2.  如果其中有一个是数字，另外一个是 undefined、null、布尔型或数字，则会将其转换成数字进行加法运算
3.  如果其中一个是字符串、一个是数字，则按照字符串规则进行拼接。

```js
1 + 2        // 3  常规情况
'1' + '2'    // '12' 常规情况
// 下面看一下特殊情况
'1' + undefined   // "1undefined" 规则1，undefined转换字符串
'1' + null        // "1null" 规则1，null转换字符串
'1' + true        // "1true" 规则1，true转换字符串
'1' + 1n          // '11' 比较特殊字符串和BigInt相加，BigInt转换为字符串
1 + undefined     // NaN  规则2，undefined转换数字相加NaN
1 + null          // 1    规则2，null转换为0
1 + true          // 2    规则2，true转换为1，二者相加为2
1 + 1n            // 错误  不能把BigInt和Number类型直接混合相加
'1' + 3           // '13' 规则3，字符串拼接


```

### Object 的转换规则

首先会调用原型链上的 valueOf() 方法，如果已经转换为基础类型，则返回，如果还不是基础类型，继续调用 toString() 方法，如果转换为基础类型，则返回；如果还不是基础类型，则抛出错误。

下面这个例子，重写了 obj 对象的 valueOf 和 toString 方法

```js
var obj = {
  value: 1,
  valueOf() {
    return 2;
  },
  toString() {
    return '3'
  },
}
console.log(obj + 1); // 输出3


```

`10 + {} // "10[object Object]"  `
>{} 会默认调用 valueOf 是转化为 {}，不是基础类型，则继续转换，调用 toString，返回结果 "[object Object]"，于是和 10 进行'+'运算，按照字符串拼接规则来，所以为 "10[object Object]"

> valueOf 和 toString 都是 obj 对象原型上的方法，关于原型可以参考我的这篇文章 [js 原型原型链知多少](https://juejin.cn/post/6981625867021582349 "https://juejin.cn/post/6981625867021582349")

> 关于 Object 这个构造函数有那些方法，以及详细的介绍。可以参考我的这篇文章 [Object](https://juejin.cn/post/6985344590895120420 "https://juejin.cn/post/6985344590895120420")

**自测是否已经完全掌握转换规则**

```js
1.'123' == 123   // false or true?
2.'' == null    // false or true?
3.'' == 0        // false or true?
4.[] == 0        // false or true?
5.[] == ''       // false or true?
6.[] == ![]      // false or true?
7.null == undefined //  false or true?
8.Number(null)     // 返回什么？
9.Number('')      // 返回什么？
10.parseInt('');    // 返回什么？
11.{}+10           // 返回什么？


```

答案：  
1 true  
2 false  
3 true  
4 true  
5 true  
6 true  
7 true  
8 0  
9 0  
10 NaN  
11 10

**FAQ**:

### 为什么 [] == 0,[]=="" 为 true

当判断中，发现有一边不是原始值类型，就会先调用 valueOf 方法进行转换

```js
[].valueOf()  // []


```

发现 valueOf 转化完后，依然不是原始值类型，那继续用 toString 方法转换

```js
[].toString() // ""


```

转换完毕，发现转成了原始值类型 ""  
所以 []=="" 为 true

"" == 0 判断时，会继续用 Number 进行转换判断

```js
Number("")==0 


```

所以 []==0 为 true

### 为什么 [] == ![] 结果为 true

根据运算符优先级 ，! 的优先级是大于 == 的，所以先会执行 ![]  
！可将变量转换成 boolean 类型，null、undefined、NaN 以及空字符串 ('') 取反都为 true，其余都为 false  
所以 ! [] 运算后的结果就是 false  
也就是 [] == ! [] 相当于 [] == false  
根据'==' 的隐式类型转换规则第三条  
根据上面提到的规则（如果有一个操作数是布尔值，则在比较相等性之前先将其转换为数值——false 转换为 0，而 true 转换为 1），则需要把 false 转成 0  
也就是 [] == ! [] 相当于 [] == false 相当于 [] == 0  
第一个例子解释了 []==0 为什么 为 true