> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7350478703317762088)

> 在 TypeScript 中经常会遇到应该使用 Type 还是 Interface 的问题，本篇文章将讲解应该使用 Type 而不是 Interface 的好处

以下是最常见的使用场景，可以看到`Type`和`Interface`都能用作组件属性类型

```ts
type UserProps = {
  name: string;
  age: number;
}
   
// interface UserProps {
//   name: string;
//   age: number;
// }
   
export default function User({}: UserProps){
  return <div>Card</div>;
}


```

**给一个基础类型增加其他属性**

`type`使用的是交叉

```ts
type UserProps = {
  name: string;
  ages: number;
};

type AdminProps = UserProps & {
  role: string;
}


```

`interface`使用的是继承

```ts
interface UserProps {
  name: string;
  age: number;
}

interface AdminProps extends UserProps {
  role: string;
}


```

**interface 只能描述对象，而 type 可以描述对象以及其他任何东西 (例如：像 string,number,boolean 等原生类型)**

为了更语义化，定义一个`Address`类型，这个类型实际是`string`类型

```ts
type Address = string;

const address: Address = "广东省深圳市南山区";


```

`interface`为了实现这种效果，需要这样使用

```ts
interface Address {
  address: string;
}

const address: Address = {
  address: "广东省深圳市南山区"
}


```

可以看出使用`type`更加方便

**type 可以描述联合类型，而 interface 不行**

```ts
type Address = string | string[];

const address: Address = ["广东省深圳市南山区", "广东省深圳市福田区"]


```

**type 可以轻松使用工具类型，interface 也可以，不过只能用丑陋的语法**

使用`Omit`取出`UserProps`中除某几个属性外的其他属性作为一个新的类型

```ts
type UserProps = {
  name: string;
  age: number;
  createdBy: Date;
}

type GuestProps = Omit<UserProps, "name" | "age">;


```

`interface`需要使用继承，并会有一个空的括号

```ts
interface GuestProps extends Omit<UserProps, "name" | "age"> {}


```

**type 可以轻松描述元组，interface 也可以，不过只能用丑陋的语法**

```ts
type Address = [number, string];

// interface Address extends Array<number | string> {
//   0: number;
//   1: string;
// }

const address: Address = [1, '广东省深圳市南山区'];


```

**从其他类型提取类型**

```ts
const project = {
  title: "Project 1",
  specification: {
    areaSize: 100,
    rooms: 3
  },
}

// const project = {
//   title: "Project 1",
//   specification: {
//     areaSize: 100,
//     rooms: 3
//   },
// } as const

type Specification = typeof project["specification"];


```

甚至可以给`project`加上`as const`，这样`specification`中的`areaSize`和`rooms`都固定为常量了

**interface 可以被合并，“interface 是开放的” 和 “type 是收敛的”**

```ts
interface User {
  name: string;
  age: number;
}

interface User {
  role: string;
}

let user: User = {
}


```

两个`User`的`interface`可以同时定义，最终`User`将会是两者的结合，这在使用第三方库并存在相同`interface`的时候会造成困扰

```ts
type User = {
  name: string;
  age: number;
}

// type User = {
//   role: string;
// }

type User2 = User & {
  role: string;
}

let user: User2 = {
  
}


```

定义两个相同名称的`User`将会报错，可以使用交叉类型来增加属性，清晰明了

**type 也能被 class 使用**

```ts
interface IUser {
  name: string;
  age: number;
}

type TUser = {
  name: string;
  age: number;
}

class User implements TUser {
  name: string;
  age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
}


```

**结语**

当我们能用`type`实现`interface`的所有功能，并且方式更优雅，而且还具备`interface`不能实现的功能时，我们应该毫不犹豫选择使用`type`而不是`interface`，毕竟我们用的是`TypeScript`而不是`InterfaceScript`