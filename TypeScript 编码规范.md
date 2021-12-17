---
date created: 2021-12-09 22:54
date updated: 2021-12-17 16:19
---

## 1. 简介

### [TypeScript 入门教程](https://ts.xcatliu.com/)

### 术语

**驼峰命名法,** 指的是混合使用字母大写来构成变量或者函数名。

例如：`addNewNumber()`

**帕斯卡(pascal)命名法,** 帕斯卡命名法指当变量名和函式名称是由二个或二个以上单字连结在一起，而构成的唯一识别字时，用以增加变量和函式的可读性。

例如：`AddNewNumber()`

- 与驼峰命名法的区别是: 帕斯卡命名法命名的变量或函数名的第一个单词首字母也是大写

**TypeScript 语法糖**, 如无特别说明, 本篇皆使用 TypeScript 3.x 以上的语法糖。除了编译之后, 还会配置 telint 进行辅助检查。

**JavaScript 语法,** 如无特别说明, 本篇采用 ECMAScript 2015 (ES6)以上版本的语法糖。在配置 `tsconfig.json` 文件时, 配置为 ES2017。要求默认支持 `async...await` , 要求配置 tslint 辅助检查。

**Node.js 版本要求,** 要求使用 nodeJs 版本为 TLS 版本。

### 总体原则

所有使用 TypeScript 和 JavaScript 作为开发语言的软件产品都须遵照本规范的内容进行编码。

规则后面括号里面标识的是严重级别。主要分为规则、建议、参考三个级别。其中规则是要强制要遵守的；建议是在优先遵守；参考可以有自己的规则，但可以参考本规范中的规则。

## 2. 概述

对于 TypeScript 会有两个文件, 一个是 TypeScript 的源代码文件 `.ts`, 一个是编译后生成 JavaScript 的 `.js` 文件。对于 JavaScript 则只有 `.js` 文件。

### 规则: 版权和版版本的声明

/*************************************************************************

- Copyright(c) 2021-20xx, XXXXX 有限公司

- XXX Co., Ltd.

- All rights reserved.

-

- @file：Filename.h

- @summary：文件描述信息

- @version: 1.1

---

- revision             author            reason             date
- 1.1                  张三              增加功能 1      2019-03-01
- 1.0                  李四              增加功能 2      2019-03-02

*************************************************************************/

### 规则: 文件名和路径名

因为在 Linux 系统下面, 文件名和路径名是大小写敏感的, 而在 Windows 系统下面, 是大小写不敏感。在 TypeScript 或 JavaScript 引用模块的时候, 有可能因为大小写的问题造成错误:

- 文件名与路径名:

- 文件名使用小写字母、数字和下划线组成 `.js` 或 `.ts` 文件

- 对于多个单词使用 _ 分隔, 不能使用驼峰命名法

### 规则: 文件编码格式要求使用 utf-8 格式

注意请不要使用 Microsoft 的 UTF-8·BOM+ 格式

## 3. 代码格式

### 建议: 程序块要采用缩进风格编写, 缩进的空格数为 2 个, 不要使用 tab 键进行缩进

- 程序块要采用缩进风格编写, 缩进空格为 2 个。函数或过程的开始、结构的定义以及循环、判断语句中的代码都要采用缩进风格, case 语句下的情况处理语句也要遵循语句缩进要求。
- 代码对齐要求使用空格键, 不要使用 tab 键。以免不同的编辑器阅读程序时, 因 tab 键设置的空格数目不同而造成代码布局不整齐。在 vscode 中可将 tab 键定义为 "自动替换为 2 个空格" 方式, 提高编写效率。

### 建议: 相对独立的程序块之间用空行隔开

以下情况要求使用空行隔开:

1. 函数方法之间应该使用空行隔开

2. 不同逻辑的程序块之间用空行隔开, **如有必要的情况下,** 应当加上程序块描述注释

3. 每个类型声明之后应该使用空行与其它代码隔开

示例:

// bad
if (! isValid (bFlag))
{
... // 程序代码
}
let age: number     = data[i].age;
let name: string = data[i].name;

// good!
if (! isValid (bFlag)){
... //程序代码
}

let age: number     = data[i].age;
let name: string = data[i].name;

### 建议: 每行代码的宽度不超过 120 字符

注意: 以下情况应分为多行

- 长表达式要在低优先级操作符处划分出新行, 操作符放在新行之首, 划分出的新行要进行适当的缩进, 使排版整齐, 语句可读。

- 若函数或过程中的参数较长，则要进行适当的划分。

- 循环、判断等语句中若有较长的表达式或语句, 则要进行适当的划分, 长表达式要在低优先级操作符处划分出新行, 操作符放在新行之首。

### 规则: 每行只能写一行语句

### 建议: 相应运算符等符号之间应在适当处插入空格, 增加可读性

- `+ - * /`, `=`, `==`, `===`, `!=`, `!==`, `>=`, `<=`, `>`, `<`, `<<`, `>>`, `&&`, `&`, `^`, `|`, `||` 等运算符两边需要加上空格
- `,` , `;` 等符号后面如果有内容, 需要加上空格

### 规则: 程序块分界符(TypeScript 和 JavaScript 的 {} )

对于 `function`、`if`、`switch`、`for`、`do…while`、`try…catch` 等语句块，{ 要紧跟后面，而 } 要求和定义同一列。

示例:

funciton fun(){
for(let i = 0; i < 100; i++ ){
console.log("num=" + i);
}

if(…){
// code...
}

switch(...) {
case ...: {
// code...
}
break;
default: {
// code...
}
}
do {
// ...
}while(...);
return 0;
}

### 建议: 多分支跳出语句使用 do{}while(false);格式

使用它的好处有以下两点:

- 避免多个函数出口
- 避免程序嵌套层次过多

示例:

let num = 999;
do
{
if( num == 100 ) break;
if( num == 99 ){
num = 1999;
break;
}
// code...
} while (false);

### 规则: 严禁在代码中使用有意义的常数, 要求用常量替代

if(n === 1) {
}
else if(n === 2) {
}
else if(n === 3) {
}
// 这样的 1，2，3 是不允许出现在逻辑代码中的。
// 要求改为：
/** 定义演示用的 enum 类型 */
enum YearType{
Normal:1,
Raw:2,
Other:3,
}
if(n === YearType.Normal) {
}
else if(n === YearType.Raw) {
}
else if(n === YearType.Other) {
}

### 规则: 要求在每个语句的最后，加上;号

增加可读性

## 4. 命名规范

### 常量定义与命名

对 JavaScript 要求使用 `const` 定义

**规则: 常量类型名称和常量值名由大写字母+数字+ _ 组成(或使用驼峰命名法)**

示例:

/** 好年 */
const GOOD_YEAR = 2019;

/** 状态定义 _/
const STATUS = {
/_* 普通状态:1 _/
NORMAL:1,
/_* 高:2 _/
HIGH:2,
/_* 低:3 _/
LOW:3,
/_* 太高:4 */
TOO_HIGH:4,
};

// 其中: STATUS 是名称，NORMAL 是值名

// 对于 TypeScript,则要求使用 enum
/** 常量定义 _/
export enum STATUS {
/_* 普通 _/
NORMAL = 1,
/_* 高级 _/
HIGH,
/_* 低级 _/
LOW,
/_* 太高级 */
TOO_HIGH
}

### 局部变量与命名

#### 规则: 非函数参数局部变量使用驼峰命名法。

#### 规则: 变量定义要求使用 `let` 或 `const`，不能使用 `var`

#### 规则: 函数参数变量要加 param 前缀

使用 驼峰命名法法，但需要加上前缀 param，用以明确说明这个是一个函数参数

例如：

/**

- 演示用函数
- @param {string} paramOne 参数 1
- @param {string} paramTow 参数 2
- @return {void}

*/
function sample(paramOne: string, paramTwo: string): void {
let testNickname = "test";
...
}

### 规则: 类或者接口命名使用帕斯卡命名法, 接口需要加上前缀 `I`

interface IFruit {
}

class RedApple implements IFruit {
}

class BigOrange implements IFruit {
}

#### 规则: 公开类成员函数名, 使用骆驼命名法(包括静态类成员函数)

#### 规则: 类的成员变量使用 m_为前缀, 使用骆驼命名法。(包括静态类成员变量)

#### 建议: 类的成员变量使用 `get` 和 `set` 方法访问

### 规则: 函数名使用骆驼命名法

/**

- 计算年份
- @param {number} 计算的时间，单位: 秒
- @return {number} 计算后的年份

*/
function calcYear(paramTime) {
...
}

### 规则: 使用函数常见动词约定

动词

含义

返回值

can

判断是否可执行某个动作(权限)

true: 可执行

false: 不可执行

has

判断是否含有某个值

true: 有

false: 没有

is

判断是否为某个值

true: 是

false: 不是

get

获取某个值

返回该值

set

设置某个值

无返回, 返回是否设置成功, 返回链式对象

load

需要加载

无返回或者返回加载后的结果

...

...

...

## 5. 注释

### 注释说明:

**目的: 使代码有更好的可读性, 并通过 vscode 编辑器的代码提示功能, 使代码协作者可以更高效率协同开发。**

- 注释是一个比较大的内容, 主要分为版权相关注释, 变量注释, 代码块注释, 类注释, 函数注释, 常量注释

- 注释视注释内容的长短, 自行选择单行或者多行注释

- 可使用 vscode 中一款名为 "Add jsdoc comments" 的插件, 对于变量, 类, 函数, 常量, 请使用该插件来生成注释。直达链接: [Add jsdoc comments](https://marketplace.visualstudio.com/items?itemName=stevencl.addDocComments)

### 建议: 变量注释

- 对于有特殊定义或使用的变量, 需要增加注释
- 如果需要代码提示, 则使用上述插件进行格式注释

### 建议: 代码块注释

- 对于一些比较长的代码块或者有特别需要注意的代码块, 需要加上注释, 方便开发者理解代码含义

![](https://cdn.nlark.com/yuque/0/2021/png/223223/1620660898312-87a90844-b3a7-423c-94a1-2ecf1e63ea2c.png)

### 类或接口的注释

#### 类的注释是必须的

/**

- 这是一个测试类
- 每个类，必需要注释，不可少

*/
export class TestSimple {
...
}

#### 如果这个类不建议使用了, 需要标注

/**

- 这是一个测试类
- 每个类，必需要注释，不可少
- @deprecated 已经放弃，请不要再使用

*/
export class TestSimple {
...
}

#### 建议: 其它标注, 可以适当使用如: `@date`, `@author`, `@example`, `@exports`, `@copyright` 等

/**

- 这是一个测试类
- 每个类，必需要注释，不可少
- @date 2019-03-08
- @author rex
- @example
- let a = new TestSimple();
- @exports
- @copyright  2017-2019 北京希为科技版权所有

*/
export class TestSimple {
...
}

### 函数注释

#### 规则: 对于非常简单、容易理解的函数不需要特别注释

#### 规则: 对于 Typescript 中 `@param` 以及 `@return` 标注，不需要指定类型

因为在函数定义中已经指定了类型，不需要额外注释

#### 规则: 有具体功能或存在多行、或不易理解、或存在使用时需要注意的函数, 需要加上注释

/**

- 部署企业申请子流程
- @param {ThinkController} paramController ThinkController 对象
- @param {DBManager} paramDB 数据库管理对象
- @param {Application} paramApp 应用系统对象
- @param {String} paramName 应用名称
- @return {void}

*/
function financingJd(paramController: ThinkController, paramDB: DBManager,
paramApp: Application, paramName: String): void {
}

#### 规则: 对于类的 `static` 函数，要求加上 `@static` 标注

![](https://cdn.nlark.com/yuque/0/2021/png/223223/1620661382592-7081affa-30e9-435b-8d12-30a7512b77e6.png)

#### 规则: 对于 `async` 函数，要求加上 `@async` 标注

/**

- 部署企业申请子流程
- @async
- @param {ThinkController} paramController ThinkController 对象
- @param {DBManager} paramDB 数据库管理对象
- @param {Application} paramApp 应用系统对象
- @param {String} paramName 应用名称
- @return {Promise<void>}

*/
async function financingJd(paramController: ThinkController, paramDB: DBManager,
paramApp: Application, paramName: String): Promise<void> {
}
// 在 typescript 下面, 对于 async 返回类型, 需要 Promise<>定义

#### 建议: JavaScript 中参数或返回是 的, 需要标注(TypeScript 不需要)

![](https://cdn.nlark.com/yuque/0/2021/png/223223/1620661574352-cd125bc2-c3bc-48cb-a66e-85eccbc77367.png)

## 6. 其它说明

### 建议: 对于字符串, 使用 `'` 或 `` ` ``,尽量不要使用 `"`

### 规则: 对于类的成员变量, 要求使用 `private` 描述，访问用 `get` 和 `set` 开放访问

- 对于企业软件这是必须项

如下图是个简单的例子, 图中 `m_aa` 和 `m_bb` 就是类 `TestSimple` 的成员变量

![](https://cdn.nlark.com/yuque/0/2021/png/223223/1620661751754-675b2499-38f9-44c9-b47a-506b05f452bb.png)

### 规则: 对于保护类成员变量, 要求使用 `protected` 描述。访问用 `get` 和 `set` 开放访问。

![](https://cdn.nlark.com/yuque/0/2021/png/223223/1620661843359-931b6e52-7705-4907-8ea0-daf1801b62e6.png)

### 建议: 函数不超过 100 行

### 建议: 参数过多的时候，要用 `object` 做为传入

## 7. 附录

- 🔲tsconfig.json 配置

- 🔲jsconfig.json 配置, 仅限当前项目使用，用于当前 TypeScript 与 JavaScript 混合编程。将来的新项目将全部使用 TypeScript.

- 🔲TSLint 配置

- 🔲JSLint 配置

- 🔲ThinkJS 的 Typescript 编译
