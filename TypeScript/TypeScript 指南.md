
TS 是什么 ？
========

**TS**：是 **TypeScript** 的简称，是一种由微软开发的自由和开源的编程语言。

TS 和 JS 的关系
-----------

对比于 JS，TS 是 JS 的**超集**，简单的说就是在 `JavaScript` 的基础上加入了**类型系统**，让每个参数都有明确的意义，从而带来了更加**智能**的提示。

相对于`JS`而言，`TS`属于**强类型**语言，所以对于项目而言，会使代码更加规范，从而解决了大型项目代码的复杂性，其次，浏览器是不识别`TS`的，所以在编译的时候，`TS`文件会先编译为`JS`文件。

TS 的基本数据类型
==========

这里将 TS 的数据类型简单的进行下归类：

* 基本类型：`string`、`number`、`boolean`、`symbol`、`bigint`、`null`、`undefined`
* 引用类型：`array`、 `Tuple`(元组)、 `object`(包含`Object`和`{}`)、`function`
* 特殊类型：`any`、`unknow`、`void`、`never`、`Enum`(枚举)
* 其他类型：`类型推理`、`字面量类型`、`交叉类型`

注：案例中有可能用到`type`和`interface`，在下面会详细讲解，有比较模糊的可以先看看

基本类型
----

```
    //字符串
    let str: string = "Domesy"
    
    // 数字
    let num: number = 7
    
    //布尔
    let bool: boolean = true
    
    //symbol
    let sym: symbol = Symbol();
     
    //bigint
    let big: bigint = 10n
        
    //null
    let nu: null = null
    
    //undefined
    let un: undefined = undefined


```

需要注意：

* `null` 和 `undefined` 两个类型一旦赋值上，就不能在赋值给任何其他类型
* `symbol`是独一无二的，假设再定义一个 `sym1`，那么 **sym === sym1 为 false**

引用类型
----

### Array

两种方式：

* 类型名称 + []
* Array <数据类型>

```ts
    let arr1: number[] = [1, 2, 3]
    
    let arr2: Array<number> = [1, 2, 3]
    
    let arr2: Array<number> = [1, 2, '3'] // error
    
     //要想是数字类型或字符串类型，需要使用 ｜
    let arr3: Array<number | string> = [1, 2, '3'] //ok


```

### Tuple(元组)

**Tuple** 可以说是 `Array` 的一种特殊情况, 针对上面的 `arr3`, 我们看他的类型可以是`string`也可以是`number`，但对每个元素没有作出具体的限制。

那么 `Tuple` 的作用就是限制**元素的类型**并且**限制个数**的数组, 同时 `Tuple`这个概念值存在于`TS`，在`JS`上是不存在的

这里存在一个问题：在`TS`中, 是允许对 **Tuple** 扩增的（也就是允许使用 `push`方法），但在**访问上不允许**

```ts
    let t: [number, string] = [1, '2'] // ok
    let t1: [number, string] = [1, 3] // error
    let t2: [number, string] = [1] // error
    let t3: [number, string] = [1, '1', true] // error


    let t5: [number, string] = [1, '2'] // ok
    t.push(2)
    console.log(t) // [1, '2', 2]

    let a =  t[0] // ok
    let b = t[1] // ok
    let c = t[2] // error


```

### object

* `object` 非原始类型，在定义上直接使用 object 是可以的，但你要更改对象的属性就会报错，原因是并没有使对象的内部具体的属性做限制，所以需要使用 **{}** 来定义内部类型

```ts
    let obj1: object = { a: 1, b: 2}
    obj1.a = 3 // error

    let obj2: { a: number, b: number } = {a: 1, b: 2}
    obj2.a = 3 // ok


```

* `Object`(大写的 O）, 代表所有的原始类型或非原始类型都可以进行赋值, 除了`null`和 `undefined

```ts
    let obj: Object;
    obj = 1; // ok
    obj = "a"; // ok
    obj = true; // ok
    obj = {}; // ok
    obj = Symbol() //ok
    obj = 10n //ok
    obj = null; // error
    obj = undefined; // error


```

### function

#### 定义函数

* 有两种方式，一种为 `function`， 另一种为`箭头函数`
* 在书写的时候，也可以写入返回值的类型，如果写入，则必须要有对应类型的返回值，**但通常情况下是省略**，因为`TS`的类型推断功能够正确推断出返回值类型

```ts
    function setName1(name: string) { //ok
      console.log("hello", name);
    }
    setName1("Domesy"); // "hello",  "Domesy"

    function setName2(name: string):string { //error
      console.log("hello", name);
    }
    setName2("Domesy");

    function setName3(name: string):string { //error
      console.log("hello", name);
      return 1
    }
    setName3("Domesy");

    function setName4(name: string): string { //ok
      console.log("hello", name);
      return name
    }
    setName4("Domesy"); // "hello",  "Domesy"

    //箭头函数与上述同理
    const setName5 = (name:string) => console.log("hello", name);
    setName5("Domesy") // "hello",  "Domesy"


```

#### 参数类型

* 可选参数： 如果函数要配置可有可无的参数时，可以通过 **?** 实现，切可选参数一定要在最后面
* 默认参数：函数内可以自己设定其默认参数，用 **=** 实现
* 剩余参数：仍可以使用扩展运算符 **...**

```ts
    // 可选参数
    const setInfo1 = (name: string, age?: number) => console.log(name, age)
    setInfo1('Domesy') //"Domesy",  undefined
    setInfo1('Domesy', 7) //"Domesy",  7

    // 默认参数
    const setInfo2 = (name: string, age: number = 11) => console.log(name, age)
    setInfo2('Domesy') //"Domesy",  11
    setInfo2('Domesy', 7) //"Domesy",  7

    // 剩余参数
    const allCount = (...numbers: number[]) => console.log(`数字总和为：${numbers.reduce((val, item) => (val += item), 0)}`)
    allCount(1, 2, 3) //"数字总和为：6"


```

#### 函数重载

**函数重载**：是使用相同名称和不同参数数量或类型创建多个方法的一种能力。 在 TypeScript 中，表现为给同一个函数提供多个函数类型定义。 简单的说：**可以在同一个函数下定义多种类型值，最后汇总到一块**

```ts
    let obj: any = {};
    function setInfo(val: string): void;
    function setInfo(val: number): void;
    function setInfo(val: boolean): void;
    function setInfo(val: string | number | boolean): void {
      if (typeof val === "string") {
        obj.name = val;
      } else {
        obj.age = val;
      }
    }
    setInfo("Domesy");
    setInfo(7);
    setInfo(true);
    console.log(obj); // { name: 'Domesy', age: 7 }


```

特殊类型
----

### any

在 TS 中，任何类型都可以归于 `any` 类型，所以`any`类型也就成了所有类型的**顶级类型**，同时，**如果不指定变量的类型，则默认为 any 类型**, 当然不推荐使用该类型，因为这样丧失了 TS 的作用

```ts
    let d:any; //等价于 let d 
    d = '1';
    d = 2;
    d = true;
    d = [1, 2, 3];
    d = {}


```

### unknow

与`any`一样，都可以作为所有类型的**顶级类型**，但 `unknow`更加**严格**，那么可以说除了`any` 之下的第二大类型，接下来对比下`any`, 主要严格于一下两点：

* `unknow`会对值进行检测，而类型`any`不会做检测操作，说白了，`any`类型可以赋值给任何类型，但`unknow`只能赋值给`unknow`类型和`any`类型
* `unknow`不允许定义的值有任何操作（如 方法，new 等），但 any 可以

```ts
    let u:unknown;
    let a: any;

    u = '1'; //ok
    u = 2; //ok
    u = true; //ok
    u = [1, 2, 3]; //ok
    u = {}; //ok

    let value:any = u //ok
    let value1:any = a //ok
    let value2:unknown = u //ok
    let value3:unknown = a //ok
    let value4:string = u //error
    let value5:string = a //ok
    let value6:number = u //error
    let value7:number = a //ok
    let value8:boolean = u //error
    let value9:boolean = a //ok

    u.set() // error
    a.set() //ok
    u() // error
    a() //ok
    new u() // error
    new a() //ok


```

### void

当一个函数，没有返回值时，TS 会默认他的返回值为 `void` 类型

```ts
    const setInfo = ():void => {} // 等价于 const setInfo = () => {}

    const setInfo1 = ():void => { return '1' }  // error
    const setInfo2 = ():void => { return 2 } // error
    const setInfo3 = ():void => { return true } // error
    const setInfo4 = ():void => { return  } // ok
    const setInfo5 = ():void => { return undefined } //ok 


```

### never

表示一个函数永远不存在返回值，TS 会认为类型为 `never`，那么与 `void` 相比, `never`应该是 `void`子集， 因为 `void`实际上的返回值为 `undefined`，而 `never` 连 `undefined`也不行

符合`never`的情况有：当抛出异常的情况和无限死循环

```ts
    let error = ():never => { // 等价约 let error = () => {}
            throw new Error("error");
    };

    let error1 = ():never => {
        while(true){}
    }


```

### Enum(枚举)

可以定义一些带名字的常量，这样可以**清晰表达意图**或**创建一组有区别的用例**

注意：

* 枚举的类型只能是 `string` 或 `number`
* 定义的名称不能为**关键字**

同时我们可以看看翻译为 ES5 是何样子

### 数字枚举

* 枚组的类型默认为**数字类型**，默认从 0 开始以此累加，如果有设置默认值，则**只会对下面的值产生影响**
* 同时支持**反向映射**（及从成员值到成员名的映射），但智能映射无默认值的情况，并且只能是默认值的前面

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7559b8e5b7b94f8fbafc95fd37eeffbd~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

### 字符串枚举

字符串枚举要注意的是必须要有**默认值**，不支持**反向映射**

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0df3876ecc6a474085e468cbd3f1929f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

### 常量枚举

除了`数字类型`和`字符串类型`之外，还有一种特殊的类型，那就是**常量枚举**，也就是通过`const`去定义`enum`，但这种类型不会编译成任何 `JS`, 只会编译对应的值

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/160ac2e75cab457d99093ce48c34756a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

### 异构枚举

包含了 `数字类型` 和 `字符串类型` 的混合，反向映射一样的道理

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7d5f1433ab2c48bebc65ca5f8ace0da4~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

类型推论
----

我们在学完这些基础类型，我们是不是每个类型都要去写字段是什么类型呢？其实不是，在`TS`中如果不设置类型，并且不进行赋值时，将会推论为 **any** 类型，如果进行赋值就会默认为类型

```ts
    let a; // 推断为any
    let str = '小杜杜'; // 推断为string
    let num = 13; // 推断为number
    let flag = false; // 推断为boolean

    str = true // error Type 'boolean' is not assignable to type 'string'.(2322)
    num = 'Domesy' // error
    flag = 7 // error


```

字面量类型
-----

**字面量类型**：在`TS`中，我们可以指定参数的类型是什么，目前支持`字符串`、`数字`、`布尔`三种类型。比如说我定义了 `str 的类型是 '小杜杜'` 那么 str 的值只能是**小杜杜**

```ts
    let str:'小杜杜' 
    let num: 1 | 2 | 3 = 1
    let flag:true

    str = '小杜杜' //ok
    str = 'Donmesy' // error

    num = 2 //ok
    num = 7 // error

    flag = true // ok
    flag = false // error


```

交叉类型（&）
-------

**交叉类型**：将多个类型合并为一个类型，使用`&`符号连接，如：

```ts
    type AProps = { a: string }
    type BProps = { b: number }

    type allProps = AProps & BProps

    const Info: allProps = {
        a: '小杜杜',
        b: 7
    }


```

### 同名基础属性合并

我们可以看到`交叉类型`是结合两个属性的属性值，那么我们现在有个问题，要是两个属性都有相同的属性值，那么此时总的类型会怎么样，先看看下面的案列：

```ts
    type AProps = { a: string, c: number }
    type BProps = { b: number, c: string }

    type allProps = AProps & BProps

    const Info: allProps = {
        a: '小杜杜',
        b: 7,
        c:  1, // error (property) c: never
        c:  'Domesy', // error (property) c: never
    }


```

如果是相同的类型，合并后的类型也是此类型，那如果是不同的类型会如何：

我们在`Aprops`和`BProps`中同时加入`c属性`，并且`c属性`的类型不同，一个是`number`类型，另一个是`string`类型

现在结合为 `allProps` 后呢? 是不是`c属性`是 `number` 或 `string` 类型都可以，还是其中的一种？

然而在实际中， `c` 传入`数字类型`和`字符串类型`都不行，我么看到报错，现实的是 **c 的类型是 never**。

这是因为对应 `c`属性而言是 `string & number`, 然而这种属性明显是不存在的，所以`c`的属性是`never`

### 同名非基础属性合并

```ts
    interface A { a: number }
    interface B { b: string }

    interface C {
        x: A
    }
    interface D {
        x: B
    }
    type allProps = C & D

    const Info: allProps = {
      x: {
        a: 7,
        b: '小杜杜'
      }
    }

    console.log(Info) // { x: { "a": 7, "b": "小杜杜" }}


```

我们来看看案例，对于混入多个类型时，若存在相同的成员，且成员类型为非基本数据类型，那么是可以成功合。

如果 接口 A 中的 也是 b，类型为 number，就会跟**同名基础属性合并**一样

Class（类）
========

在`ES6`中推出了一个叫 `class（类）` 的玩意，具体定义就不说了，相信用过`React`的小伙伴一定不陌生.

基本方法
----

在基本方法中有：`静态属性`，`静态方法`、`成员属性`、`成员方法`、`构造器`、`get set方法`，接下来逐个看看：

需要注意的是： 在成员属性中，如果不给默认值, 并且不使用是会报错的，如果不想报错就给如 **!**，如：`name4!:string`

```ts
    class Info {
      //静态属性
      static name1: string = 'Domesy'

      //成员属性，实际上是通过public上进行修饰，只是省略了
      nmae2:string = 'Hello' //ok 
      name3:string //error
      name4!:string //ok 不设置默认值的时候必须加入 !

      //构造方法
      constructor(_name:string){
        this.name4 = _name
      }

      //静态方法
      static getName = () => {
        return '我是静态方法'
      }

      //成员方法
      getName4 = () => {
        return `我是成员方法:${this.name4}`
      }

      //get 方法
      get name5(){
        return this.name4
      }

      //set 方法
      set name5(name5){
        this.name4 = name5
      }
    }

    const setName = new Info('你好')
    console.log(Info.name1) //  "Domesy" 
    console.log(Info.getName()) // "我是静态方法" 
    console.log(setName.getName4()) // "我是成员方法:你好" 


```

让我们看看上述代码翻译成 ES5 是什么样：

```ts
    "use strict";
    var Info = /** @class */ (function () {
        //构造方法
        function Info(_name) {
            var _this = this;
            //成员属性
            this.nmae2 = 'Hello'; //ok
            //成员方法
            this.getName4 = function () {
                return "\u6211\u662F\u6210\u5458\u65B9\u6CD5:".concat(_this.name4);
            };
            this.name4 = _name;
        }
        Object.defineProperty(Info.prototype, "name5", {
            //get 方法
            get: function () {
                return this.name4;
            },
            //set 方法
            set: function (name5) {
                this.name4 = name5;
            },
            enumerable: false,
            configurable: true
        });
        //静态属性
        Info.name1 = 'Domesy';
        //静态方法
        Info.getName = function () {
            return '我是静态方法';
        };
        return Info;
    }());
    var setName = new Info('你好');
    console.log(Info.name1); //  "Domesy" 
    console.log(Info.getName()); // "我是静态方法" 
    console.log(setName.getName4()); // "我是成员方法:你好" 


```

私有字段 (#)
--------

在 TS 3.8 版本便开始支持 **ECMACMAScript** 的私有字段。

需要注意的是`私有字段`与常规字段不同，主要的区别是：

* 私有字段以 `#` 字符开头，也叫私有名称；
* 每个私有字段名称都**唯一**地限定于其包含的类；
* 不能在私有字段上使用 TypeScript 可访问性修饰符（如 public 或 private）；
* 私有字段不能在包含的类之外访问，甚至不能被检测到。

```ts
    class Info {
      #name: string; //私有字段
      getName: string;

      constructor(name: string) {
        this.#name = name;
        this.getName = name
      }

      setName() {
        return `我的名字是${this.#name}`
      }
    }

    let myName = new Info("Domesy");


    console.log(myName.setName()) // "我的名字是Domesy" 
    console.log(myName.getName) // ok "Domesy" 
    console.log(myName.#name) // error 
    // Property '#name' is not accessible outside class 'Info' 
    // because it has a private identifier.(18013)


```

只读属性（readonly）
--------------

**只读属性**：用 `readonly`修饰，只能在**构造函数**中初始化，并且在 TS 中，只允许将`interface`、`type`、`class`上的属性标识为`readonly`

* `readonly`实际上只是在`编译阶段`进行代码检查
* 被`radonly`修饰的词只能在 `constructor`阶段修改，其他时刻不允许修改

```ts
    class Info {
      public readonly name: string; // 只读属性
      name1:string

      constructor(name: string) {
        this.name = name;
        this.name1 = name;
      }

      setName(name:string) {
        this.name = name // error
        this.name1 = name; // ok
      }
    }


```

继承（extends）
-----------

**继承**：是个比较重要的点，指的是子可以继承父的思想，也就是说 `子类` 通过继承`父类`后，就拥有了`父类`的属性和方法，这点与`HOC`有点类似

这里又个`super`字段，给不知道的小伙伴说说，其作用是**调用父类上的属性和方法**

```ts
    // 父类
    class Person {
      name: string
      age: number

      constructor(name: string, age:number){
        this.name = name
        this.age = age
      }

      getName(){
        console.log(`我的姓名是：${this.name}`)
        return this.name
      }

      setName(name: string){
        console.log(`设置姓名为：${name}`)
        this.name = name
      }
    }

    // 子类
    class Child extends Person {
      tel: number
      constructor(name: string, age: number, tel:number){
        super(name, age)
        this.tel = tel
      }

      getTel(){
        console.log(`电话号码是${this.tel}`)
        return this.tel
      }
    }

    let res = new Child("Domesy", 7 , 123456)
    console.log(res) // Child {."name": "Domesy", "age": 7, "no": 1 }
    console.log(res.age) // 7
    res.setName('小杜杜') // "设置姓名为：小杜杜" 
    res.getName() //   "我的姓名是：小杜杜"
    res.getTel() //  "电话号码是123456" 


```

修饰符
---

主要有三种修饰符：

* **public**：类中、子类内的任何地方、外部**都能调用**
* **protected**：类中、子类内的任何地方都能调用, 但**外部不能调用**
* **private**：类中可以调用，子类内的任何地方、外部**均不可调用**

```ts
    class Person {
      public name: string
      protected age: number
      private tel: number

      constructor(name: string, age:number, tel: number){
        this.name = name
        this.age = age
        this.tel = tel
      }
    }

    class Child extends Person {
      constructor(name: string, age: number, tel: number) {
        super(name, age, tel);
      }

      getName(){
        console.log(`我的名字叫${this.name},年龄是${this.age}`) // ok name 和 age可以
        console.log(`电话是${this.tel}`) // error 报错 原因是 tel 拿不出来
      }
    }


    const res = new Child('Domesy', 7, 123456)
    console.log(res.name) // ok Domesy
    console.log(res.age) // error
    console.log(res.tel) // error


```

abstract
--------

**abstract**: 用 abstract 关键字声明的类叫做**抽象类**，声明的方法叫做**抽象方法**

* **抽象类**：指不能被实例化，因为它里面包含一个或多个抽象方法。
* **抽象方法**：是指不包含具体实现的方法；

注：抽象类是不能直接实例化，只能实例化实现了所有抽象方法的子类

```ts
    abstract class Person {
      constructor(public name: string){}

      // 抽象方法
      abstract setAge(age: number) :void;
    }

    class Child extends Person {
      constructor(name: string) {
        super(name);
      }

      setAge(age: number): void {
        console.log(`我的名字是${this.name},年龄是${age}`);
      }
    }

    let res = new Person("小杜杜") //error
    let res1 = new Child("小杜杜");

    res1.setAge(7) // "我的名字是小杜杜,年龄是7"


```

重写和重载
-----

* **重写**：子类重写继承自父类中的方法
* **重载**：指为同一个函数提供多个类型定义，与上述函数的重载类似

```ts
    // 重写
    class Person{
      setName(name: string){
        return `我的名字叫${name}`
      }
    }

    class Child extends Person{
      setName(name: string){
        return `你的名字叫${name}`
      }
    }

    const yourName = new Child()
    console.log(yourName.setName('小杜杜')) // "你的名字叫小杜杜" 

    // 重载
    class Person1{
      setNameAge(name: string):void;
      setNameAge(name: number):void;
      setNameAge(name:string | number){
        if(typeof name === 'string'){
          console.log(`我的名字是${name}`)
        }else{
          console.log(`我的年龄是${name}`)
        }
      };
    }

    const res = new Person1()
    res.setNameAge('小杜杜') // "我的名字是小杜杜" 
    res.setNameAge(7) // "我的年龄是7"


```

TS 断言和类型守卫
==========

TS 断言
-----

分为三种：`类型断言`、`非空断言`、`确定赋值断言`

当断言失效后，可能使用到：**双重断言**

### 类型断言

在特定的环境中，我们会比 TS 知道这个值具体是什么类型，不需要 TS 去判断，简单的理解就是，**类型断言会告诉编译器，你不用给我进行检查，相信我，他就是这个类型**

共有两种方式：

* **尖括号**
* **as**：推荐

```ts
   //尖括号
   let num:any = '小杜杜'
   let res1: number = (<string>num).length; // React中会 error

   // as 语法
   let str: any = 'Domesy';
   let res: number = (str as string).length;


```

但需要注意的是：尖括号语法在 **React** 中会报错，原因是与`JSX`语法会产生冲突，所以只能使用 **as 语法**

### 非空断言

在上下文中当类型检查器无法断定类型时，一个新的后缀表达式操作符 `!` 可以用于断言操作对象是**非 null 和非 undefined 类型。**

我们对比下`ES5`的代码

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7fa80b9ac2a54614a129c5724678254a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

我们可以看出来 `!`可以帮助我们过滤 `null`和 `undefined`类型，也就是说，编译器会默认我们只会传来`string`类型的数据，所以可以赋值为`str1`

但变成`ES5`后 `!`会被移除，所以当传入 `null` 的时候，还是会打出 `null`

### 确定赋值断言

在`TS` 2.7 版本中引入了确定赋值断言，即允许在实例属性和变量声明后面放置一个 `!` 号，以告诉`TS`该属性会被明确赋值。

```ts
    let num: number;
    let num1!: number;

    const setNumber = () => num = 7
    const setNumber1 = () => num1 = 7

    setNumber()
    setNumber1()

    console.log(num) // error 
    console.log(num1) // ok


```

### 双重断言

**断言失效后，可能会用到，但一般情况下不会使用**

失效的情况：基础类型不能断言为接口

```ts
    interface Info{
      name: string;
      age: number;
    }

    const name = '小杜杜' as Info; // error, 原因是不能把 string 类型断言为 一个接口
    const name1 = '小杜杜' as any as Info; //ok


```

类型守卫
----

**类型守卫**：是**可执行运行时检查的**一种表达式，用于确保**该类型在一定的范围内**。

我个人的感觉是，类型守卫就是你可以设置多种类型，但我默认你是什么类型的意思

目前，常有的类型守卫共有 4 种：**in 关键字**、**typeof 关键字**、**instanceof** 和**类型谓词（is)**

### in 关键字

**用于判断这个属性是那个里面的**

```ts
    interface Info {
      name: string
      age: number
    }

    interface Info1{
      name: string
      flage: true
    }

    const setInfo = (data: Info | Info1) => {
      if("age" in data){
        console.log(`我的名字是：${data.name}，年龄是：${data.age}`)
      }

       if("flage" in data){
        console.log(`我的名字是：${data.name}，性别是：${data.flage}`)
      }
    }

    setInfo({name: '小杜杜', age: 7}) // "我的名字是：小杜杜，年龄是：7" 
    setInfo({name: '小杜杜', flage: true}) // "我的名字是：小杜杜，性别是：true"


```

### typeof 关键字

**用于判断基本类型，如 string ｜ number 等**

```ts
    const setInfo = (data: number | string | undefined) => {
      if(typeof data === "string"){
        console.log(`我的名字是：${data}`)
      }

      if(typeof data === "number"){
        console.log(`我的年龄是：${data}`)
      }

      if(typeof data === "undefined"){
        console.log(data)
      }
    }

    setInfo('小杜杜') // "我的名字是：小杜杜"  
    setInfo(7) // "我的年龄是：7" 
    setInfo(undefined) // undefined" 


```

### instanceof 关键字

**用于判断一个实例是不是构造函数，或使用类的时候**

```ts
    class Name {
      name: string = '小杜杜'
    }

    class Age extends Name{
      age: number = 7
    }

    const setInfo = (data: Name) => {
      if (data instanceof Age) {
        console.log(`我的年龄是${data.age}`);
      } else {
        console.log(`我的名字是${data.name}`);
      }
    } 

    setInfo(new Name()) // "我的名字是小杜杜"
    setInfo(new Age()) // "我的年龄是7" 


```

### 类型谓词（is)

```ts
function isNumber(x: any): x is number { //默认传入的是number类型
  return typeof x === "number"; 
}

console.log(isNumber(7)) // true
console.log(isNumber('7')) //false
console.log(isNumber(true)) //false


```

两者的区别
-----

通过上面的介绍，我们可以发现`断言`与`类型守卫`的概念非常相似，都是确定参数的类型，但`断言`更加**霸道**，它是直接告诉编辑器，这个参数就是这个类型，而类型守卫更像确定这个参数具体是什么类型。（个人理解，有不对的地方欢迎指出～）

类型别名、接口
=======

类型别名（type）
----------

**类型别名**：也就是`type`，用来给一个类型起个新名字

```
    type InfoProps = string | number
    
    const setInfo = (data: InfoProps) => {}


```

接口（interface）
-------------

**接口**：在面向对象语言中表示行为抽象，也可以用来描述对象的形状。

使用 **interface** 关键字来定义接口

### 对象的形状

接口可以用来描述`对象`，主要可以包括以下数据：`可读属性`、`只读属性`、`任意属性`

* **可读属性**：当我们定义一个接口时，我们的属性可能不需要全都要，这是就需要 **?** 来解决
* **只读属性**：用 **readonly** 修饰的属性为只读属性，意思是指允许定义，不允许之后进行更改
* **任意属性**：这个属性极为重要，它是可以用作就算没有定义，也可以使用，比如 **[data: string]: any**。比如说我们对组件进行封装，而封装的那个组件并没有导出对应的类型，然而又想让他不报错，这时就可以使用任意属性

```ts
    interface Props {
        a: string;
        b: number;
        c: boolean;
        d?: number; // 可选属性
        readonly e: string; //只读属性
        [f: string]: any //任意属性
    }
    let res: Props = {
        a: '小杜杜',
        b: 7,
        c: true,
        e: 'Domesy',
        d: 1, // 有没有d都可以
        h: 2 // 任意属性，之前为定义过h
    }

    let res.e = 'hi' // error, 原因是可读属性不允许更改


```

### 继承

**继承**：与类一样，接口也存在继承属性，也是使用`extends`字段

```ts
    interface nameProps {
        name: string
    }

    interface Props extends nameProps{
        age: number
    }

    const res: Props = {
        name: '小杜杜',
        age: 7
    }


```

### 函数类型接口

同时，可以定义函数和类，加`new`修饰的事**类**，不加 new 的事**函数**

```ts
    interface Props {
        (data: number): number
    }

    const info: Props = (number:number) => number  //可定义函数

    // 定义函数
    class A {
        name:string
        constructor(name: string){
            this.name = name
        }
    }

    interface PropsClass{
        new (name: string): A
    }

    const info1 = (fun: PropsClass, name: string) => new fun(name)

    const res = info1(A, "小杜杜")
    console.log(res.name) // "小杜杜" 


```

type 和 interface 的区别
--------------------

通过上面的学习，我们发现`类型别名`和`接口`非常相似，可以说在大多数情况下，`type`与`interface`是等价的

但在一些特定的场景差距还是比较大的，接下来逐个来看看

### 基础数据类型

* `type`和`interface`都可以定义 **对象** 和 **函数**
* `type`可以定义其他数据类型，如字符串、数字、元祖、联合类型等，而`interface`不行

```ts
    type A = string // 基本类型

    type B = string | number // 联合类型

    type C = [number, string] // 元祖

    const dom = document.createElement("div");  // dom元素
    type D = typeof dom


```

### 扩展

`interface` 可以扩展 `type`，`type` 也可以扩展为 `interface`，但两者实现扩展的方式不同。

* `interface` 是通过 `extends` 来实现
* `type` 是通过 `&` 来实现

```ts
    // interface 扩展 interface
    interface A {
        a: string
    }
    interface B extends  A {
        b: number
    }
    const obj:B = { a: `小杜杜`, b: 7 }

    // type 扩展 type
    type C = { a: string }
    type D = C & { b: number }
    const obj1:D = { a: `小杜杜`, b: 7 }

    // interface 扩展为 Type
    type E = { a: string }
    interface F extends E { b: number }
    const obj2:F = { a: `小杜杜`, b: 7 }

    // type 扩展为 interface
    interface G { a: string }
    type H = G & {b: number}
    const obj3:H = { a: `小杜杜`, b: 7 }

T extends String
```

重复定义
----

`interface` 可以多次被定义，并且会进行合并，但`type`不行

```ts
    interface A {
        a: string
    }
    interface A {
        b: number
    }
    const obj:A = { a: `小杜杜`, b: 7 }

    type B = { a: string }
    type B = { b: number } // error


```

联合类型 (Union Types)
==================

**联合类型 (Union Types)**: 表示取值可以为多种类型中的一种, 未赋值时联合类型上只能访问两个类型共有的属性和方法，如：

```
    const setInfo = (name: string | number) => {}

    setInfo('小杜杜')
    setInfo(7)


```

从上面看 `setInfo`接收一个`name`，而 `name` 可以接收 `string`或`number`类型，那么这个参数便是联合类型

可辨识联合
-----

**可辨识联合**：包含三个特点，分别是`可辨识`、`联合类型`、`类型守卫`,

这种类型的本质是：结合**联合类型**和**字面量类型**的一种类型保护方法。

**如果一个类型是多个类型的联合类型，且多个类型含有一个公共属性，那么就可以利用这个公共属性，来创建不同的类型保护区块。**

也就是上面一起结合使用，这里写个小例子：

```ts
    interface A {
      type: 1,
      name: string
    }

    interface B {
      type: 2
      age: number
    }

    interface C {
      type: 3,
      sex: boolean
    }

    // const setInfo = (data: A | B | C) => {
    //   return data.type // ok 原因是 A 、B、C 都有 type属性
    //   return data.age // error，  原因是没有判断具体是哪个类型，不能确定是A，还是B，或者是C
    // }

    const setInfo1 = (data: A | B | C) => {
      if (data.type === 1 ) {
        console.log(`我的名字是${data.name}`);
      } else if (data.type === 2 ){
        console.log(`我的年龄是${data.age}`);
      } else if (data.type === 3 ){
        console.log(`我的性别是${data.sex}`);
      }
    }

    setInfo1({type: 1, name: '小杜杜'}) // "我的名字是小杜杜"
    setInfo1({type: 2, age: 7}) // "我的年龄是7" 
    setInfo1({type: 3, sex: true}) // "我的性别是true" 


```

定义了 `A`、`B`、`C` 三次接口，但这三个接口都包含`type`属性，那么`type`就是`可辨识的属性`, 而其他属性只跟特性的接口相关。

然后通过可辨识属性`type`，才能使用其相关的属性

泛型
==

**泛型**：Generics，是指在定义函数、接口或类的时候，不预先指定具体的类型，而在使用的时候再指定类型的一种特性

也就是说，泛型是**允许同一个函数接受不同类型参数的一种模版**，与`any`相比，使用泛型来创建可服用的组件要更好，因为**泛型会保留参数类型**（PS：泛型是整个 TS 的重点，也是难点，请多多注意～）

为什么需要泛型
-------

我们先看看一个例子：

```ts
    const calcArray = (data:any):any[] => {
        let list = []
        for(let i = 0; i < 3; i++){
            list.push(data)
        }
        return list
    }

    console.log(calcArray('d')) // ["d", "d", "d"]


```

上述的例子我们发现，在`calcArray`中传任何类型的参数，返回的数组都是`any`类型

由于我们不知道传入的数据是什么，所以返回的数据也为`any的数组`

但我们现在想要的效果是：**无论我们传什么类型，都能返回对应的类型**，针对这种情况怎么办？所以此时`泛型`就登场了

泛型语法
----

我们先用泛型对上面的例子进行改造下，

```ts
    const calcArray = <T,>(data:T):T[] => {
        let list:T[] = []
        for(let i = 0; i < 3; i++){
            list.push(data)
        }
        return list
    }

    const res:string[] = calcArray<string>('d') // ok
    const res1:number[] = calcArray<number>(7) // ok

    type Props = {
        name: string,
        age: number
    }
    const res3: Props[] = calcArray<Props>({name: '小杜杜', age: 7}) //ok


```

经过上面的案例，我们发现传入的`字符串`、`数字`、`对象`，都能返回对应的类型，从而达到我们的目的，接下来我们再看看`泛型语法`：

```
    function identity <T>(value:T) : T {
        return value
    }


```

第一次看到这个`<T>`我们是不是很懵，实际上这个`T`就是**传递的类型**, 从上述的例子来看，这个`<T>`就是`<string>`, 要注意一点，这个`<string>`实际上是可以省略的，因为 TS 具有**类型推论**，可以自己推断类型

多类型传参
-----

我们有多个未知的类型占位，我们可以定义任何的字母来表示不同的参数类型

```ts
    const calcArray = <T,U>(name:T, age:U): {name:T, age:U} => {
        const res: {name:T, age:U} = {name, age}
        return res
    }

    const res = calcArray<string, number>('小杜杜', 7)
    console.log(res) // {"name": "小杜杜", "age": 7}


```

泛型接口
----

定义接口的时候，我们也可以使用泛型

```ts
    interface A<T> {
        data: T
    }

    const Info: A<string> = {data: '1'}
    console.log(Info.data) // "1"


```

泛型类
---

同样泛型也可以定义类

```ts
    class clacArray<T>{
        private arr: T[] = [];

        add(value: T) {
            this.arr.push(value)
        }
        getValue(): T {
            let res = this.arr[0];
            console.log(this.arr)
            return res;
        }
    }

    const res = new clacArray()

    res.add(1)
    res.add(2)
    res.add(3)

    res.getValue() //[1, 2, 3] 
    console.log(res.getValue) // 1


```

泛型类型别名
------

```ts
    type Info<T> = {
        name?: T
        age?: T
    }

    const res:Info<string> = { name: '小杜杜'}
    const res1:Info<number> = { age: 7}


```

泛型默认参数
------

所谓默认参数，是指定类型，如默认值一样，从实际值参数中也无法推断出类型时，这个默认类型就会起作用。

```ts
    const calcArray = <T = string,>(data:T):T[] => {
        let list:T[] = []
        for(let i = 0; i < 3; i++){
            list.push(data)
        }
        return list
    }


```

泛型常用字母
------

用常用的字母来表示一些变量的代表：

* **T**：代表 **Type**，定义泛型时通常用作第一个类型变量名称
* **K**：代表 **Key**，表示对象中的**键类型**；
* **V**：代表 **Value**，表示对象中的**值类型**；
* **E**：代表 **Element**，表示的**元素类型**；

常用技巧
====

在 TS 中有许多关键字和工具类型，在使用上，需要注意**泛型**上的应用，有的时候结合起来可能就有一定的问题

在此特别需要注意 `extends`、`typeof`、`Partial`、`Record`、`Exclude`、`Omit`这几个工具类型

extends
-------

**extends**：检验是否拥有其属性 在这里，举个例子，我们知道`字符串`和`数组`拥有`length`属性，但`number`没有这个属性。

```ts
    const calcArray = <T,>(data:T): number => {
      return data.length // error 
    }


```

上述的 `calcArray`的作用只是获取`data的数量`, 但此时在`TS`中会报错，这是因为 **TS 不确定传来的属性是否具备 length 这个属性**，毕竟每个属性都不可能完全相同

那么这时该怎么解决呢？

我们已经确定，要拿到传过来数据的 `length`，也就是说传过来的属性必须具备`length`这个属性，如果没有，则不让他调用这个方法。

换句话说，`calcArray`需要具备检验属性的功能，对于上述例子就是检验是否有`length`的功能，这是我们就需要 **extends** 这个属性帮我们去鉴定：

```ts
    interface Props {
        length: number
    }

    const calcArray = <T extends Props,>(data:T): number => {
      return data.length // error
    }

    calcArray('12') // ok
    calcArray([1,3]) //ok
    calcArray(2) //error 


```

可以看出`calcArray(2)`会报错，这是因为`number`类型并不具备`length`这个属性

typeof
------

**typeof 关键字**：我们在类型保护的时候讲解了 typeof 的作用，除此之外，这个关键字还可以实现**推出类型**，如下图，可以推断中 **Props 包含的类型**

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c01f7f01bc449d78530bc5e38ee2a5a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

keyof
-----

**keyof 关键字**: 可以获取一个对象接口的所有`key`值, 可以检查对象上的键是否存在

```ts
    interface Props {
        name: string;
        age: number;
        sex: boolean
    }

    type PropsKey = keyof Props; //包含 name， age， sex

    const res:PropsKey = 'name' // ok
    const res1:PropsKey = 'tel' // error

    // 泛型中的应用
    const getInfo = <T, K extends keyof T>(data: T, key: K): T[K] => {
        return data[key]
    }

    const info = {
        name: '小杜杜',
        age: 7,
        sex: true
    }

    getInfo(info, 'name'); //ok
    getInfo(info, 'tel'); //error


```

索引访问操作符
-------

**索引访问操作符**：通过 **[]** 操作符可进行索引访问, 可以访问其中一个属性

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39784ee772554968b70a7274924e2f11~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

in
--

**in**：映射类型, 用来映射遍历枚举类型

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/717b932d27054e7bba535c606b5910a9~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

infer
-----

**infer**：可以是使用为条件语句，可以用 `infer` 声明一个类型变量并且对它进行使用。如

```ts
    type Info<T> = T extends { a: infer U; b: infer U } ? U : never;

    type Props = Info<{ a: string; b: number }>; // Props类： string | number

    type Props1 = Info<number> // Props类型： never


```

Partial
-------

**Partial 语法**：`Partial<T>` 作用：将所有属性变为可选的 **?**

```ts
    interface Props {
        name: string,
        age: number
    }

    const info: Props = {
        name: '小杜杜',
        age: 7
    }

    const info1: Partial<Props> = { 
        name: '小杜杜'
    }


```

从上述代码上来看，name 和 age 属于必填，对于 info 来说必须要设置 name 和 age 属性才行，但对于 info1 来说，只要是个对象就可以，至于是否有 name、 age 属性并不重要

Required
--------

**Required 语法**：`Required<T>` 作用：将所有属性变为必选的，与 `Partial`相反

```ts
    interface Props {
        name: string,
        age: number,
        sex?: boolean
    }

    const info: Props = {
        name: '小杜杜',
        age: 7
    }

    const info1: Required<Props> = { 
        name: '小杜杜',
        age: 7,
        sex: true
    }


```

Readonly
--------

**Readonly 语法**：`Readonly<T>` 作用：将所有属性都加上 readonly 修饰符来实现。也就是说无法修改

```ts
    interface Props {
        name: string
        age: number
    }

    let info: Readonly<Props> = {
        name: '小杜杜',
        age: 7
    }

    info.age = 1 //error read-only 只读属性


```

从上述代码上来看, `Readonly`修饰后，属性无法再次更改，智能使用

Record
------

**Record 语法**：Record<K extends keyof any, T>

作用：将 `K` 中所有的属性的值转化为 `T` 类型。

```ts
    interface Props {
        name: string,
        age: number
    }

    type InfoProps = 'JS' | 'TS'

    const Info: Record<InfoProps, Props> = {
        JS: {
            name: '小杜杜',
            age: 7
        },
        TS: {
            name: 'TypeScript',
            age: 11
        }
    }


```

从上述代码上来看, `InfoProps`的属性分别包含`Props`的属性

需要注意的一点是：`K extends keyof any`其类型可以是:`string`、`number`、`symbol`

Pick
----

**Pick 语法**：Pick<T, K extends keyof T>

作用：将某个类型中的子属性挑出来，变成包含这个类型部分属性的子类型。

```ts
    interface Props {
        name: string,
        age: number,
        sex: boolean
    }

    type nameProps = Pick<Props, 'name' | 'age'>

    const info: nameProps = {
        name: '小杜杜',
        age: 7
    }


```

从上述代码上来看, `Props`原本属性包括`name`、`age`、`sex`三个属性，通过 **Pick** 我们吧`name`和`age`挑了出来，所以不需要`sex`属性

Exclude
-------

**Exclude 语法**：Exclude<T, U>

作用：将 T 类型中的 U 类型剔除。

```ts
    // 数字类型
    type numProps = Exclude<1 | 2 | 3, 1 | 2> // 3
    type numProps1 = Exclude<1, 1 | 2> // nerver
    type numProps2 = Exclude<1, 1> // nerver
    type numProps3 = Exclude<1 | 2, 7> // 1 2

    // 字符串类型
    type info = "name" | "age" | "sex"
    type info1 = "name" | "age" 
    type infoProps = Exclude<info, info1> //  "sex"

    // 类型
    type typeProps = Exclude<string | number | (() => void), Function> // string | number

    // 对象
    type obj = { name: 1, sex: true }
    type obj1 = { name: 1 }
    type objProps = Exclude<obj, obj1> // nerver


```

从上述代码上来看, 我们比较了下类型上的，当 T 中有 U 就会剔除对应的属性，如果 U 中又的属性 T 中没有，或 T 和 U 刚好一样的情况都会返回 **nerver**，且对象永远返回`nerver`

Extra
-----

**Extra 语法**：Extra<T, U>

作用：将 T 可分配给的类型中提取 U。与 **Exclude** 相反

```ts
    type numProps = Extract<1 | 2 | 3, 1 | 2> // 1 | 2


```

Omit
----

**Omit 语法**：Omit<T, U>

作用：将已经声明的类型进行属性剔除获得新类型

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7afb178ecc2b4427b82b7a5ebe3ab035~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

与 **Exclude** 的区别：Omit 返回的是新的类型，原理上是在 `Exclude`之上进行的，`Exclude`是根据自类型返回的

NonNullable
-----------

**NonNullable 语法**：`NonNullable<T>` 作用：从 T 中排除 `null` 和 `undefined` ![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/825ae5b2e0704d4aba9b143fdf6c6835~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

ReturnType
----------

**ReturnType 语法**：`ReturnType<T>`

作用：用于获取 **函数 T 的返回类型。**

```ts
    type Props = ReturnType<() => string> // string
    type Props1 = ReturnType<<T extends U, U extends number>() => T>; // number
    type Props2 = ReturnType<any>; // any
    type Props3 = ReturnType<never>; // any


```

从上述代码上来看， ReturnType 可以接受 any 和 never 类型，原因是这两个类型属于顶级类型，包含函数

Parameters
----------

**Parameters**：`Parameters<T>` 作用：用于获取 **获取函数类型的参数类型**

```ts
    type Props = Parameters<() => string> // []
    type Props1 = Parameters<(data: string) => void> // [string]
    type Props2 = Parameters<any>; // unknown[]
    type Props3 = Parameters<never>; // never


```

End
===

参考
---

* [TypeScript 4.0](https://link.juejin.cn?target=https%3A%2F%2Fwww.typescriptlang.org%2Fdocs%2Fhandbook%2Frelease-notes%2Ftypescript-4-0.html "https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-0.html")
* [深入理解 TypeScript](https://link.juejin.cn?target=https%3A%2F%2Fjkchao.github.io%2Ftypescript-book-chinese%2F "https://jkchao.github.io/typescript-book-chinese/")
* [一份不可多得的 TS 学习指南（1.8W 字）](https://juejin.cn/post/6872111128135073806 "https://juejin.cn/post/6872111128135073806")
