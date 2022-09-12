---
date created: 2022-07-12 10:52
---
#JavaScript 
## 参考文章

[async 是如何被 JavaScript 实现的](https://mp.weixin.qq.com/s/c7HUZ0LNLuVXukzG56RShA)

# 引言

无论是面试过程还是日常业务开发，相信大多数前端开发者可以熟练使用 Async/Await 作为异步任务的终极处理方案。

但是对于 Async 函数的具体实现过程只是知其然不知所以然，仅仅了解它是基于 Promise 和 Generator 生成器函数的语法糖。

提及 JavaScript 中 Async 函数的内部实现原理，大多数开发者并不清楚这一过程。甚至从来没有思考过 Async 所谓语法糖是如何被 JavaScript 组合而来的。

别担心，文中会带你分析 Async 语法在低版本浏览器下的 polyfill 实现，同时我也会手把手带你基于 Promise 和 Generator 来实现所谓的 Async 语法。

我们会从以下方面来逐步攻克 Async 函数背后的实现原理：

- Promise 章节梳理，从入门到源码带你掌握 Promise 应用。

- 什么是生成器函数？Generator 生成器函数基本特征梳理。

- Generator 是如何被实现的，Babel 如何在低版本浏览器下实现 Generator 生成器函数。

- 作为通用异步解决方案的 Generator 生成器函数是如何解决异步方案。

- 开源 Co 库的基本原理实现。

- Async/Await 函数为什么会被称为语法糖，它究竟是如何被实现的。

**相信读完文章的你，对于 Async/Await 真正可以做到 “知其然，知其所以然”。**

# Promise

所谓 Async/Await 语法我们提到本质上它是基于 **Promise 和 Generator 生成器函数的语法糖**。

关于 Promise 这篇文章中我就不过于展开他的基础和原理部分了，网络中对于介绍 Promise 相关的文章目前已经非常优秀了。如果有兴趣深入 Promise 的同学可以查看：

- 🌟 JavaScript Promise MDN Docs：<https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise>

Promise 基础使用准则，MDN 上给出了详尽的说明和实例，强烈建议对于 Promise 陌生的同学可以查阅 MDN 巩固 Promise 基础知识。

- 🌟 Promise A+ 规范：<https://promisesaplus.com/>

Promise A+ 实现准则，不同浏览器 / 环境下对于 Promise 都有自己的实现，它们都会依照同一规范标准去实现 Promise 。

我在 ➡️ 这个地址：<https://github.com/19Qingfeng/notes/blob/master/promise/core/index.js> 按照规范实现过一版完整的 Promise ，有兴趣的通许可以自行查阅代码进行 Promise 原理巩固。

- 🌟 V8 Promise 源码全面解读：<https://juejin.cn/post/7055202073511460895>

关于 Promise 中各种边界应用以及深层次 Promise 原理实现，笔者强烈建议有兴趣更深层次的同学结合月夕的这篇文章去参照阅读。

# 生成器函数

关于 Generator 生成器函数与 Iterator 迭代器，大多数开发者在日常应用中可能并不如 Promise 那么常见。

所以针对于 Generator 我会稍微和大家从基础开始讲起。

# Generator 概念

## Generator 基础入门

所谓 Generator 函数它是协程在 ES6 的实现，最大特点就是可以交出函数的执行权（即拥有暂停函数执行的效果）。

```js
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}

let g = gen();

g.next(); // { value: 1, done: false }
g.next(); // { value: 2, done: false }
g.next(); // { value: 3, done: false }
g.next(); // { value: undefined, done: true }
g.next(); // { value: undefined, done: true }

```

上述的函数就是一个 Generator 生成器函数的例子，我们通过在函数声明后添加一个 * 的语法创建一个名为 gen 的生成器函数。

调用创建的生成器函数会返回一个 `Generator { }` 生成器实例对象。

初次接触生成器函数的同学，看到上面的例子可能稍微会有点懵。什么是 Generator 实例对象，函数中的 yield 关键字又是做什么的，我们应该如何使用它呢？

别着急，接下来我们来一步一揭开这些迷惑。

所谓返回的 `g` 生成器对象你可以简单的将它理解成为类似这样的一个对象结构:

```js
{
    next: function () {
        return {
            done:Boolean, // done表示生成器函数是否执行完毕 它是一个布尔值
            value: VALUE, // value表示生成器函数本次调用返回的值
        }
    }
}

```

- 首先，我们通过 `let g = gen()` 调用生成器函数创建了一个生成器对象 `g` ，此时 g 拥有 next 上述结构的 next 方法。

这一步，我们成为 g 为返回的生成器对象， gen 为生成器函数。通过调用生成器函数 gen 返回了生成器对象 g 。

- 之后，生成器对象中的 next 方法每次调用会返回一次 `{ value:VALUE, done:boolean }`的对象。

每次调用生成器对象的 next 方法会返回一个上述类型的 object:

**其中 done 表示生成器函数是否执行完毕, 而 value 表示生成器函数中本次 yield 对应的值。**

部分没有接触过的同学可能不太了解这一过程，我们来详细拆开上述函数的执行过程来看看：

- 首先调用 gen() 生成器函数返回 g 生成器对象。

- 其次返回的 g 生成器对象中拥有一个 next 的方法。

- **每当我们调用 g.next() 方法时，生成器函数紧跟着上一次进行执行，直到函数碰到 yield 关键值。**

- yield 关键字会停止函数执行并将 yield 后的值返回作为本次调用 next 函数的 value 进行返回。

- 同时，如果本次调用 g.next() 导致生成器函数执行完毕，那么此时 done 会变成 true 表示该函数执行完毕，反之则为 false 。

比如当我们调用 `let g = gen()` 时，会返回一个生成器函数，它拥有一个 next 方法。

之后当第一次调用 `g.next()` 方法时，会执行生成器函数 gen 。函数会进行执行，直到碰到 yield 关键字会进行暂停，此时函数会暂停到 `yield 1` 语句执行完毕，将 1 赋给 value

同时因为生成器函数 gen 并没有执行完毕，所以此时 done 应该为 false 。所以此时首次调用 `g.next()` 函数返回的应该是 `{ value: 1, done: false }`。

之后，我们第二次调用 `g.next()` 方法时，函数会从上一次的中断结果后进行执行。也就是会继续 `yield 2` 语句。

当遇到 `yield 2` 时，又因为碰到了 `yield` 语句。此时函数又会被中断，因为此时函数并没有执行完成，并且`yield` 语句后紧挨着的是 `2` 所以第二个 `g.next()` 会返回 `{ value: 2 , done: false }`。

同样，`yield 3;` 回和前两次执行逻辑相同。

需要额外注意的是，当我们第四次调用迭代器 g.next() 时，因为第三次 g.next() 结束时生成器函数已经执行完毕了。所以再次调用 g.next() 时，由于函数结束 done 会变为 false 。同时因为函数不存在返回值，所以 value 为 undefined。

上边是一个基于 Generator 函数的简单执行过程，其实它的本质非常简单：

**调用生成器函数会返回一个生成器对象，每次调用生成器对象的 next 方法会执行函数到下一次 yield 关键字停止执行，并且返回一个 `{ value: Value, done: boolean }`的对象。**

上述执行过程，我稍稍用了一些篇幅来描述这一简单的过程。如果看到这里你还是没有上面的 Demo 含义，那么此时请你停下往下的进度，会到开头一定要搞清楚这个简单的 Demo 。

## Generator 函数返回值

在掌握了基础 Generator 函数和 yield 关键字后，趁热打铁让我们来一举攻克 Generator 生成器函数的进阶语法。

老样子，我们先来看这样一段代码：

```js
function* gen() {
  const a = yield 1;
  console.log(a,'this is a')
  const b = yield 2;
  console.log(b,'this is b')
  const c = yield 3;
  console.log(c,'this is c')
}

let g = gen();

g.next(); // { value: 1, done: false }
g.next('param-a'); // { value: 2, done: false }
g.next('param-b'); // { value: 3, done: false }
g.next('param-c'); // { value: undefined, done: true }

// 控制台会打印:
// param-a this is a
// param-b this is b
// param-c this is c

```

这里，我们着重来看看调用生成器对象的 next 方法传入参数时究竟会发生什么事情，理解 next() 方法的参数是后续 Generator 解决异步的重点实现思路。

上文我们提到过，生成器函数中的 yield 关键字会暂停函数的运行，简单来说比如我们第一次调用 g.next() 方法时函数会执行到 `yield 1` 语句，此时函数会被暂停。

当第二次调用 `g.next()` 方法时，生成器函数会继续从上一次暂停的语句开始执行。这里有一个需要注意的点：**当生成器函数恢复执行时，因为上一次执行到 `const a = yield 1` 语句的右半段并没有给 `const a`进行赋值。**

那么此时的赋值语句 `const a = yield 1`，a 会被赋为什么值呢？细心的同学可能已经发现了。我们在 g.next('param-a') 传入的参数 `param-a` 会作为生成器函数重新执行时，上一次 `yield` 语句的返回值进行执行。

简单来说，也就是调用 `g.next('param-a')`恢复函数执行时，相当于将生成器函数中的 `const a = yield 1;` 变成 `const a = 'param-a';` 进行执行。

这样，第二次调用 `g.next('param-a')`时自然就打印出了 `param-a this is a` 。

同样当我们第三次调用 `g.next('param-b')` 时，本次调用 next 函数传入的参数会被当作 `yield 2` 运算结果赋值给 `b` 变量，执行到打印时会输出 `param-b this is b`。

同理 g.next('paramc') 会输出 `param-c this is b`。

**总而言之，当我们为 next 传递值进行调用时，传入的值会被当作上一次生成器函数暂停时 yield 关键字的返回值处理。**

自然，第一次调用 g.next() 传入参数是毫无意义的。因为首次调用 next 函数时，生成器函数并没有任何执行自然也没有 yield 关键字处理。

接下来我们来看看所谓的生成器函数返回值:

```js
function* gen() {
  const a = yield 1;
  console.log(a, 'this is a');
  const b = yield 2;
  console.log(b, 'this is b');
  const c = yield 3;
  console.log(c, 'this is c');
  return 'resultValue'
}

let g = gen();

g.next(); // { value: 1, done: false }
g.next('param-a'); // { value: 2, done: false }
g.next('param-b') // { value: 3, done: false }
g.next() // { value: 'resultValue', done: true }
g.next() // { value: undefined, done: true }

```

当生成器函数存在 `return` 返回值时，我们会在第四次调用 g.next() 函数恢复执行，此时生成器函数继续执行函数执行完毕。

此时自然 done 会变为 true 表示生成器函数已经执行完毕，之后，由于函数存在返回值所以随之本次的 value 会变为'resultValue' 。

也就是当生成器函数执行完毕时，原本本次调用 next 方法返回的 `{done:true,value:undefined}` 变为了`{ done:true,value:'resultValue'}`。

关于 Generator 函数的基本使用我们就介绍到这里，接下来我们来看看它是如何被 JavaScript 实现的。

# Generator 原理实现

关于 Generator 函数的原理其实和我们后续的异步关系并不是很大，但是本着 “知其然，知其所以然” 的出发点。

希望大家可以耐心去阅读本小结，其实它的内部运行机制并不是很复杂。笔者自己也在某电商大厂面试中被问到过如何实现 Generator 的 polyfill。

首先，你可以打开链接查看我已经编辑好的 ➡️ Babel Generator Demo[6]。

![](https://mmbiz.qpic.cn/mmbiz/pfCCZhlbMQQzpCSibaCHCenbDI0ZEC9oy7wFzWUCnCx7jiaOzLZREfib3LicCQ9VyJQL0Fbh9X4ibicRP4PUDsQsHcFQ/640?wx_fmt=jpeg)image.png

**乍一看也许很多同学会稍微有点懵，没关系。这段代码并不难，难的是你对未知恐惧的心态。**

这是 Babel 在低版本浏览器下为我们实现的 Generator 生成器函数的 polyfill[7] 实现。

左侧为 ES6 中的生成器语法，右侧为转译后的兼容低版本浏览器的实现。

首先左侧的 gen 生成器函数被在右侧被转化成为了一个简单的普通函数，具体 gen 函数内容我们先忽略它。

在右侧代码中，对于普通 gen 函数包裹了一层 regeneratorRuntime.mark(gen) 处理，在源码中这一步其实为了将普通 gen 函数继承 GeneratorFunctionPrototype 从而实现将 gen() 返回的对象变成 Generator 实例对象的处理。

这一步对于我们理解 Generator 显得并不是那么重要，所以我们可以简单的将 regeneratorRuntime.mark 改写成为这样的结构:

```js
// 自己定义regeneratorRuntime对象
const regeneratorRuntime = {
    // 存在mark方法，接受传入的fn。原封不懂的返回fn
    mark(fn) {
        return fn
    }
}

```

我们自己定义了 regeneratorRuntime 对象，并且为他定义了一个 mark 方法。它的作用非常简单，接受一个函数作为入参并且返回函数，仅此而已。

之后我们再来进入 gen 函数内部，在左侧源代码中当我们调用 `gen()` 时，是会返回一个 Iterator 对象（它拥有 next 方法，并且每次调用 next 都会返回 `{value:VALUE,done:boolean}`）。

所以在右侧我们了解到了，我们通过调用编译后的普通 gen() 函数应该和右侧返回一致，所谓 regeneratorRuntime.wrap() 方法也应该返回一个具有 next 属性的迭代器对象。

关于 regeneratorRuntime.wrap() 方法，这里传入了两个参数，第一个参数是一个接受传入 context 的函数，第二个参数是我们之前处理的 _marked 对象。

同样关于 wrap() 方法的第二个参数我们不用过多纠结，它仍然是对于编译后的生成器函数作为继承使用的一个参数，并不影响函数的核心逻辑。所以我们暂时忽略它。

此时，我们了解到，regeneratorRuntime 对象上应该存在一个 wrap 方法，并且 wrap 方法应该返回一个满足迭代器协议 [8] 的对象。

```js
// 自己定义regeneratorRuntime对象
const regeneratorRuntime = {
    // 存在mark方法，接受传入的fn。原封不懂的返回fn
    mark(fn) {
        return fn
  },
  wrap(fn) {
    // ...
    return {
      next() {
          done: ...,
          value: ...
      }
    }
  }
}

```

让我们进入 regeneratorRuntime.wrap 内部传入的具体函数来看看：

```js
function gen() {
  var a, b, c;
  return regeneratorRuntime.wrap(function gen$(_context) {
    // while(1) 配合函数的return没有任何实际意义
    // 通常在编程中使用 while(1) 来表示while中的内部会被多次执行
    while (1) {
      switch ((_context.prev = _context.next)) {
        case 0:
          _context.next = 2;
          return 1;

        case 2:
          a = _context.sent;
          console.log(a, 'this is a');
          _context.next = 6;
          return 2;

        case 6:
          b = _context.sent;
          console.log(b, 'this is b');
          _context.next = 10;
          return 3;

        case 10:
          c = _context.sent;
          console.log(c, 'this is c');

        case 12:
        case 'end':
          return _context.stop();
      }
    }
  }, _marked);
}

```

regeneratorRuntime.wrap 内部传入的函数，使用了 while(1) 内部逻辑，因为我们在 while 循环中配合了函数的 return 语句，所以这里 while(1) 其实并没有任何含义。

通常，在编程中我们用 while(1) 来表示内部的逻辑会被执行很多次，的确在函数内部的 while 循环每次调用 next 方法其实都会进入这段逻辑执行。

首先我们来看看 传入的 _context 参数存在哪些属性：

- _context.prev 表示本次生成器函数执行时的指针位置。

- _context.next 表示下次生成器函数调用时的指针位置。

- _context.sent 表示调用 g.next(params) 时，传入的 params 参数。

- _context.stop 表示当调用 g.next() 生成器函数执行完毕调用的方法。

在解释了 _context 对象上的属性含义之后，也许你还是不太明白它们各自的含义。我们先来看看简化后的实现代码：

```js
const regeneratorRuntime = {
  // 存在mark方法，接受传入的fn。原封不懂的返回fn
  mark(fn) {
    return fn;
  },
  wrap(fn) {
    const _context = {
      next: 0, // 表示下一次执行生成器函数状态机switch中的下标
      sent: '', // 表示next调用时候传入的值 作为上一次yield返回值
      done: false, // 是否完成
      // 完成函数
      stop() {
        this.done = true;
      },
    };
    return {
      next(param) {
        // 1. 修改上一次yield返回值为context.sent
        _context.sent = param;
        // 2.执行函数 获得本次返回值
        const value = fn(_context);
        // 3. 返回
        return {
          done: _context.done,
          value,
        };
      },
    };
  },
};

```

完整的 regeneratorRuntime 对象就像上边实现的那样，它看起来非常简单对吧。

在 wrap 函数中，我们接受传入的一个状态机函数。每次调用 wrap() 方法返回的 next(param) 方法时，会将 next(param) 中传入的参数传递给 wrap 函数中的维护的 _context.sent 作为模拟上一次 yield 返回值而出现。

同时在 wrap(fn) 中传入的 fn 你可以将它看作一个小型状态机 ，每次调用 next() 方法都会执行状态机 fn 函数。

不过，因为状态机中每次执行时 _context.prev 的值都是不同的造成了每次调用 next 函数都执行的是状态机中不同的逻辑。

直到，状态机函数 fn 中的 switch 语句匹配完成返回 _context.stop() 函数，此时将_context.done 变为 true 并且返回对应对象。

**所谓 Generator 核心代码简化后不过上述短短几十行，它的内部核心思想本质上就是通过 regeneratorRuntime.wrap 函数包裹一个状态机函数 fn 。**

**wrap 函数内部维护一个 _context 对象，从而每次调用返回的生成器对象的 next 方法时，被包裹的状态机函数根据 _context 的对应属性匹配对应状态来完成不同的逻辑。**

> Generator 核心原理其实就是这样，有兴趣了解完整代码的同学可以查看 facebook/regenerator[9]。

# Generator 异步解决方案

在讲述完 Generator 的基础概念和 polyfill 原理之后，我们来步入异步瞧瞧它是如何被应用在异步编程中的。

大多数情况下，我们会直接使用 Promise 来处理异步问题。Promise 帮助我们解决了非常糟糕的 “回调地狱” 式的异步解决方案。

但是 Promise 中仍然需要存在不停的 .then .then 当异步嵌套过多的情况下，Promise 中的 then 式调用也显得不是那么一种直观。

每当问题的产生一定会伴随解决方案的出现，在 Promise 处理异步问题时，Generator 显然作为解决方案出现在了我们的视野中。

```js
function promise1() {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve('1');
    }, 1000);
  });
}

function promise2(value) {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve('value:' + value);
    }, 1000);
  });
}

function* readFile() {
  const value = yield promise1();
  const result = yield promise2(value);
  return result;
}

```

我们来看看上述的代码，readFile 函数是不是稍微有一些 async 函数的影子。

假如我期望所谓 readFile() 方法和 async 函数行为一致，返回的 result 同样是一个 Promise 并且保持上诉代码的写法，我们应该如何做？

看到这里，你可以稍微思考下应该如何利用 Generator 函数的特性去实现。

---

**上边我们提到过生成器函数具有可暂停的特点，当调用生成器函数后会返回一个生成器对象。每次调用生成器对象的 next 方法，生成器函数才会继续往下执行直到碰到下一个 yield 语句，同时每次调用生成器对象的 next(param) 方法时，我们可以传入一个参数作为上一次 yield 语句的返回值。**

利用上述特性，我们可以写出如下的代码：

```js
function promise1() {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve('1');
    }, 1000);
  });
}

function promise2(value) {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve('value:' + value);
    }, 1000);
  });
}

function* readFile() {
  const value = yield promise1();
  const result = yield promise2(value);
  return result;
}

function asyncGen(fn) {
  const g = fn(); // 调用传入的生成器函数 返回生成器对象
  // 期望返回一个Promise
  return new Promise((resolve) => {
    // 首次调用 g.next() 方法，会执行生成器函数直到碰到第一个yield关键字
    // 这里会执行到 yield promise1() 同时将 promise1() 返回为迭代器对象的 value 值
    const { value, done } = g.next();
    // 因为value为Promise 所以可以等待promise完成后，在then函数中继续调用 g.next(res) 恢复生成器函数继续执行
    value.then((res) => {
      // 同时第二次调用g.next() 时是在上一次返回的promise.then中
      // 我们可以拿到上一次Promise的value值也就是 '1'
      // 传入g.next('1') 作为上一次yield的值 这里相当于 const value = '1'
      const { value, done } = g.next(res);
      // 同理，继续上述过程
      value.then(resolve);
    });
  });
}

asyncGen(readFile).then((res) => console.log(res)); // value: 1

```

我们通过定义了一个 asyncGen 函数来包裹 readFile 生成器函数，利用了生成器函数结合 yield 关键字可以被暂停的特点，同时结合 Promise.prototype.then 方法的特性实现了类似于 async 函数的异步写法。

看上去它和 async 很像对吧，不过目前的代码存在一个致命的问题：

asyncGen 函数并不具备通用性，上边的例子中 readFile 函数内部包裹了两层 yield 处理的 promise，我们在 asyncGen 函数内部同样两次调用 g.next() 方法。

如果我们包裹三层 yield 处理的 Promise ，那么我是不是重新书写 asyncGen 函数逻辑。又或者 readFile 中存在比如 `yield '1'` 并不是 Promise 的语句，那么我们当作 Promise 使用 then 方法处理肯定是会报错。

这样的方法不具备任何通用性，所以在实际项目中没有人会这样去组织异步代码。但是通过这个例子我相信你已经可以初步了解 Generator 生成器函数是如何结合 Promise 来作为异步解决方案的。

# tj/co

上边我们使用 Generator 作为异步解决方案，我们编写了一个名为 asyncGen 的包裹函数，可是它并不具备任何通用性。

接下来我们来思考如何让这个方法变得更加通用，从而在各种不同场景下去更好的解决异步问题：

同样，我希望我的 readFile 方法书写时方式和之前一样直观：

```js
function promise1() {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve('1');
    }, 1000);
  });
}

function promise2(value) {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve('value:' + value);
    }, 1000);
  });
}

function* readFile() {
  const value = yield promise1();
  const result = yield promise2(value);
  return result;
}

```

在这之前我们使用 Generator 的特性来处理 Promise 的异步问题，每次都傻傻的去根据 yeild 关键字去嵌套函数逻辑处理。

一些同学可能之前就想到了，对于无穷无尽的嵌套调用逻辑同时又存在边界停止条件。那么当我们需要封装出一个具有通用性的函数时，使用递归来处理不是更好吗？

或许根据这个思路，你先可以尝试自己封装一下。

---

不卖关子了，我们来看看具有通用性且更加优雅的 Generator 异步解决方案：

```js
function co(gen) {
  return new Promise((resolve, reject) => {
    const g = gen();
    function next(param) {
      const { done, value } = g.next(param);
      if (!done) {
        // 未完成 继续递归
        Promise.resolve(value).then((res) => next(res));
      } else {
        // 完成直接重置 Promise 状态
        resolve(value);
      }
    }
    next();
  });
}

co(readFile).then((res) => console.log(res));

```

我们定义了一个 co 函数来包裹传入的 generator 生成器函数。

在函数 co 中，我们返回了一个 Promise 来作为包裹函数的返回值，同时首次调用 co 函数时会调用 gen() 得到对应的生成器对象。

之后我们定义了一次 next 方法，在 next 函数内部只要迭代器未完成那么此时我们就会在 value 的 then 方法中在此递归调用该 next 函数。

**其实关于异步迭代时，大多数情况下都可以使用类似该函数中的递归方式来处理。**

函数中稍微有三点需要大家额外注意：

- 首先我们可以看到 next 函数接受传入的一个 param 的参数。

这是因为我们使用 Generator 来处理异步问题时，通过 `const a = yield promise` 将 promise 的 resolve 值交给 a ，所以我们需要在每次 `then` 函数中将 res 传递给下一次的 next(res) 作为上次 yield 的返回值。

- 其次，细心的同学可以留意到这一句代码`Promise.resolve(value).then((res) => next(res));`。

我们使用 Promise.resolve 将 value 进行了一层包裹，这是因为当生成器函数中的 `yield` 方法后紧挨的并不是 Promise 时，此时我们需要统一当作 Promise 来处理，因为我们需要统一调用 .then 方法。

- 最后，首次调用 next() 方法时，我们并没有传入 param 参数。

相信这个并不难理解，当我们不传入 param 时相当于直接调用 `g.next()` ，上边我们提到过当调用生成器对象的 next 方法传入参数时，该参数会当作上一次 yield 语句的返回值来处理。

因为首次调用 g.next() 时，生成器函数内部之前并不存在 yield ，所以传入参数是没有任何意义的。

它看来并不是很难对吧，但是通过这一小段代码让我们的 Generator 拥有了可以让异步代码同步调用的书写方式来使用。

其实这一小段代码也是所谓 co[10] 库的核心原理，当然所谓 co 远远不止这些，但是这段代码足够我们了解所谓在 Async/Await 未出现之前我们是如何使用所谓的 Generator 来作为终极异步解决方案了。

# Async/Await

铺垫了这么久终于来到现阶段 JavaScript 解决异步的最终方案了。

在前边我们聊到过所谓 Generator 的基础用法以及 Babel 是如何在 EcmaScript 5 中使用 Generator 生成器。

同时我们顺带聊了下，在 Async 没有出现之前我们如何使用 Generator 结合 Promise 来处理异步问题。

虽然之前的异步调用方式看起来已经非常类似于 async 语法了：

```js
function* readFile() {
  const value = yield promise1();
  const result = yield promise2(value);
  return result;
}

```

但是在使用 readFile 时，我们仍然需要使用 co 函数来将 Generator 来单独处理一层：

```js
co(readFile).then((res) => console.log(res));

```

那么此时，async 的出现解决了这一问题，照旧，我们先来看看在不支持 async 语法的低版本浏览器下 Babel 是如何处理它的：

![](https://mmbiz.qpic.cn/mmbiz/pfCCZhlbMQQzpCSibaCHCenbDI0ZEC9oy1EFpWd2oExKjHFqicNvic4cYl9ciaibwgFibAmv4uWAywrTPwmObDyiay9icQ/640?wx_fmt=jpeg)image.png

乍一看可能信息有点多，别担心。其实这些都是我们之前分析甚至实现过的代码。

在深入这段代码之前，我先告诉你所谓 Async 语法是如何被实现的结论：

在这之前，我们通过 Generator 和 Promise 解决异步问题时，需要将 Generator 函数额外使用 co 来包裹一层从而实现类似同步的异步函数调用。

那么如果我们想要实现低版本浏览器下的 Async 语法，那么我们将 co 函数伴随 Generator 的 polyfill 一起编译出来不就可以了吗？

**所谓 Async 其实就是将 Generator 包裹了一层 co 函数，所以它被称为 Generator 和 Promise 的语法糖。**

接下来我们一起来看看右边的 polyfil 实现。

![](https://mmbiz.qpic.cn/mmbiz/pfCCZhlbMQQzpCSibaCHCenbDI0ZEC9oy4KgJY2cvSGKp76icPKu01Jm2UdYqFRtofx0UjlWeFKHrKnicmz1YnZZw/640?wx_fmt=jpeg)image.png

这段函数是不是很眼熟，我们在之前简单聊过关于它的实现原理。

唯一有一点不同的是，它将 Generator 的实现额外包裹了一层 _asyncToGenerator 函数进行返回。

```js
function _asyncToGenerator(fn) {
  return function () {
    var self = this,
      args = arguments;
    return new Promise(function (resolve, reject) {
      var gen = fn.apply(self, args);
      function _next(value) {
        asyncGeneratorStep(gen, resolve, reject, _next, _throw, 'next', value);
      }
      function _throw(err) {
        asyncGeneratorStep(gen, resolve, reject, _next, _throw, 'throw', err);
      }
      _next(undefined);
    });
  };
}
复制代码


```

来看看所谓的 _asyncToGenerator 函数，它内部接受一个传入的 fn 函数。这个 fn 正是所谓的 Generator 函数。

当调用 hello() 时，实质上最终相当于调用 _asyncToGenerator 内部的返回函数，它会返回一个 Promise。

```js
return new Promise(function (resolve, reject) {
      var gen = fn.apply(self, args);
      function _next(value) {
        asyncGeneratorStep(gen, resolve, reject, _next, _throw, 'next', value);
      }
      function _throw(err) {
        asyncGeneratorStep(gen, resolve, reject, _next, _throw, 'throw', err);
      }
      _next(undefined);
    });

```

首先 Promise 中会进行：

```js
var gen = fn.apply(self, args);

```

它会调用我们传入的 fn（生成器函数） 返回生成器对象，之后它定义了两个方法：

```js
function _next(value) {
        asyncGeneratorStep(gen, resolve, reject, _next, _throw, 'next', value);
      }
function _throw(err) {
         asyncGeneratorStep(gen, resolve, reject, _next, _throw, 'throw', err);
     }

```

这两个方法内部都基于 asyncGeneratorStep 来进行函数调用：

```js
function asyncGeneratorStep(gen, resolve, reject, _next, _throw, key, arg) {
  try {
    var info = gen[key](arg);
    var value = info.value;
  } catch (error) {
    reject(error);
    return;
  }
  if (info.done) {
    resolve(value);
  } else {
    Promise.resolve(value).then(_next, _throw);
  }
}

```

所谓 asyncGeneratorStep 其实你完全可以将它当作我们之前实现的异步解决方案 co 的原理，它们内部的思想和实现是完全一致的，唯一的区别就是这里 Babel 编译后的实现考虑了 error 情况而我们当时并没有考虑出现错误时的情况。

本质上还是利用 Generator 函数内部可以被暂停执行的特性结合 Promise.prototype.then 中进行递归调用从而实现 Async 的语法糖。

其实看到这里，经过前边知识点的铺垫我相信最终 Async/Await 的实现原理对你来说一点都不会陌生。

之所以被称为语法糖因为本质上它也并没有任何针对于 Generator 生成器函数和 Promise 的其他知识拓展点，恰恰是更好的结合了这两种语法的特点衍生出了更加优雅简单的异步解决方案。

# 结尾

文章的结尾，首先感谢每一个可以读到这里的朋友。

我们讲述了从 Generator 函数发展到 Async/Await 的异步解决方案以及它们是如何在低版本浏览器中的 polyfill 最终延伸到它们的实现原理。

其实文章中的很多代码都是精简的实现版本，如果对哪个步骤阅读完文章还存在疑惑的话你也可以在评论区留下你的见解我们共同探讨，或者你可以去查看每个章节末尾对应的源码地址。
