---
date created: 2021-12-09 22:54
date updated: 2021-12-17 16:18
---

## 列出使用 Typescript 的一些优点

- 它提供了可选静态类型的优点。在这里，Typescript 提供了可以添加到变量、函数、属性等的类型。
- Typescript 能够编译出一个能在所有浏览器上运行的 JavaScript 版本。
- TypeScript 总是在编译时强调错误，而 JavaScript 在运行时指出错误。
- TypeScript 支持强类型或静态类型，而这不是在 JavaScript 中。
- 它有助于代码结构。
- 它使用基于类的面向对象编程。
- 它提供了优秀的工具支持和智能感知，后者在添加代码时提供活动提示。
- 它通过定义模块来定义名称空间概念。

## Typescript 的缺点是什么?

- TypeScript 需要很长时间来编译代码。
- TypeScript 不支持抽象类。
- 如果我们在浏览器中运行 TypeScript 应用程序，需要一个编译步骤将 TypeScript 转换成 JavaScript。
- Web 开发人员使用了几十年的 JavaScript，而 TypeScript 不是都是新东西。
- 要使用任何第三方库，必须使用定义文件。并不是所有第三方库都有可用的定义文件。
- 类型定义文件的质量是一个问题，即如何确保定义是正确的?

## JavaScript 不支持函数重载，但 TypeScript 是否支持函数重载？

是的，TypeScript 支持函数重载。但是它的实现很奇怪，当我们在 TypeScript 中执行函数重载时，我们只能实现一个带有多个签名的函数。

```js
//带有字符串类型参数的函数  
function add(a:string, b:string): string;    
  
//带有数字类型参数的函数
function add(a:number, b:number): number;    
  
//函数定义
function add(a: any, b:any): any {    
    return a + b;    
}
```

在上面的例子中，前两行是函数重载声明。它有两次重载，第一个签名的参数类型为 string，而第二个签名的参数类型为 number。第三个函数包含实际实现并具有 any 类型的参数。任何数据类型都可以接受任何类型的数据。然后，实现检查所提供参数的类型，并根据供应商参数类型执行不同的代码段。
