#JavaScript 

## 预编译发生时机

JS 的预编译分为**全局预编译** 和 **函数预编译**
- 全局预编译发生在页面加载完成时执行
- 函数预编译发生在函数执行时的前一刻

## 全局预编译的步骤

1. 创建 GO (Global Object, 全局执行器上下文, 在浏览器中为 windows) 对象
2. 寻找 `var` 变量声明, 并赋值为 `undefined`
3. 寻找 `function` 函数声明, 并赋值为函数体
4. 执行代码

## 函数预编译的步骤

1. 创建 AO 对象, 执行期上下文
2. 寻找函数的形参和变量声明, 将变量和形参名作为 AO 对象的属性名, 值设为 `undefined`
3. 将形参和实参相统一, 即更改形参后的 `undefined` 设置为具体的形参值
4. 寻找函数中的函数声明, 将函数名作为 AO 属性名, 值为函数体

## 综合案例

```js
var a = 1;
console.log(a);
function test(a) {
  console.log(a);
  var a = 123;
  console.log(a);
  function a() {}
  console.log(a);
  var b = function() {}
  console.log(b);
  function d() {}
}
var c = function (){
console.log("I at C function");
}
console.log(c);
test(2);
```

1. 全局预编译
```js
GO{
    a: undefined,
    c: undefined，
    test: function(a) {
        console.log(a);
        var a = 123;
        console.log(a);
        function a() {}
        console.log(a);
        var b = function() {}
        console.log(b);
        function d() {}
    }
}
```
2. 按顺序执行代码
```js
GO{
    a: 1,
    c: function (){
        console.log("I at C function");
    }，
    test: function(a) {
        console.log(a);
        var a = 123;
        console.log(a);
        function a() {}
        console.log(a);
        var b = function() {}
        console.log(b);
        function d() {}
    }
}
```
3. 执行到 `test(2)` , 函数开始预编译
	- 3.1
	```js
	AO{
    a: undefined,
    b: undefined
	}
	```
	- 3.2
	```js
	AO{
    a: 2,
    b: undefined
	}
	```
	- 3.3
	```js
	AO{
    a: function(){},
    b: undefined,
    d: function(){}
	}
	```
4. 执行结果
```js
var a = 1;
console.log(a); // 1
function test(a) {
  console.log(a); // function a() {}
  var a = 123;
  console.log(a); // 123
  function a() {}
  console.log(a); // 123
  var b = function() {}
  console.log(b); // function b() {}
  function d() {}
}
var c = function (){
console.log("I at C function");
}
console.log(c); // function c(){ console.log("I at C function"); }
test(2);
```

## 总结

1. 未经声明就赋值, 此变量就是全局变量所有, 一切声明的全局变量, 都是 windows 属性
2. 预编译发生在变量声明和函数声明阶段, 匿名函数和函数表达式不参与预编译
3. 变量声明提升: 无论变量调用和声明的位置是前是后, V8 引擎总会把声明移动到调用前, 注意仅仅是声明, 所以值是 `undefined`
4. 函数声明整体提升: 无论函数调用和声明的位置是前是后, V8 引擎总会把函数声明移动到调用前面. 如果函数名同形参或变量名同名, 则会赋值为函数体
5. 只有在解析执行才会对变量初始化

结合 [[JS 作用域和作用域链]] 