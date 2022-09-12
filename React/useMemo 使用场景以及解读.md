---
date created: 2022-07-01 16:07
---
#React 
## 相关问题

- **useMemo 到底是做什么的，工作原理是什么。**
- **useMemo 是否用的越多越好。**
- **useMemo 在什么场景下使用。**

## useMemo 到底是做什么的，工作原理是什么

简而言之，useMemo 是用来缓存**计算属性的**。

计算属性其实是函数的返回值，或者说指那些以返回一个值为目标的函数。

有些函数，需要我们手动的去点击，去完成一些动作才触发。**而有些函数，则是直接在渲染的时候就执行，在 DOM 区域被当作属性值一样去使用。**

**后者，就被称为计算属性。**

**而计算属性，最后一定会使用 return 返回一个值！**

举个例子 **代码示例（一）**

```jsx
//Com组件
const Com = () => {
    const [params1,setParams1] = useState(0);
    const [params2,setParams2] = useState(0);
    
    //这种是需要我们手动去调用的函数
    const handleFun1 = () => {
        console.log("我需要手动调用，你不点击我不执行");
        setParams1(val => val +1);
    } 
    
    //这种被称为计算属性，不需要手动调用，在渲染阶段就会执行的。
    const computedFun2 = () => {
        console.log('我又执行计算了');
        return params2;
    }
    
    return <div onClick = {handleFun1}>
        //每次重新渲染的时候我就会执行
        computed:{computedFun2()}
    </div>
}

```

在上面的代码示例中，computedFun2 函数就是一个计算属性。而 handleFun1 则是一个普通函数。

看上面的例子，组件每次点击执行 handleFun1 的时候，因为组件内部状态 (params1) 的改变会让该组件重新渲染

而每次组件重新渲染都会让我们去执行 computedFun2 函数（计算属性），**"我又执行计算了" 这句话会在每次渲染的时候都被打印一次**，尽管 computedFun2 函数中只使用到了 params2 状态，与被改变的状态并没有任何关系。

如果 computedFun2 函数里面的计算过程非常的复杂，那么每次重新计算无疑的非常麻烦的。而且在我们上面的例子中，我们返回的值并不会因为 params1 的改变而产生变化。

**那么, 我们要如何让组件在改变与计算属性无关的状态的时候进行的渲染不触发我们计算属性的重新计算呢？**

为了解决这个问题，useMemo 出现了。**useMemo 有两个入参，第一个值填写我们需要缓存的计算属性，第二个值填写依赖，useMemo 会在每次需要重新计算的时候去比较依赖是否被更改，只有当依赖改变了被 useMemo 保护的函数才会重新执行，否则拒绝重新执行，直接返回旧的计算属性值。**

还是拿上面那套代码示例举个例子 **代码示例（二）**

```jsx
//Com组件
const Com = () => {
    const [params1,setParams1] = useState(0);
    const [params2,setParams2] = useState(0);
    
    //这种是需要我们手动去调用的函数
    const handleFun1 = () => {
        console.log("我需要手动调用，你不点击我不执行");
        setParams1(val => val +1);
    } 
    
    //这种被称为计算属性，不需要手动调用，在渲染阶段就会执行的。
    const computedFun2 = useMemo(() => {
        console.log('我又执行计算了');
        return params2;
    },[params2])
    
    return <div onClick = {handleFun1}>
        //现在，我被useMemo保护，只有在组件初始化和params2改变的时候会执行
        computed:{computedFun2()}
    </div>
}

```

代码示例一和代码示例二的区别仅仅在于，computedFun2 函数是否被 useMemo 保护。

而使用 useMemo 保护了 computedFun2 这个函数之后，computedFun2 函数只会在组件初始化的时候和 params2 状态改变的时候执行，然后不论 params1 这个状态如何的改变，computedFun2 函数都不会再去执行。

## useMemo 是不是用的越多越好

**不是！！！**

**缓存，需要成本！**

缓存并不是免费的，所有被 useMemo 保护的函数都会被加入 useMemo 的工作队列。

在组件进行渲染并且此组件内使用了 useMemo 之后，为了校验改组件内被 useMemo 保护的这个计算属性是否需要重新计算，**它会先去 useMemo 的工作队列中找到这个函数，然后还需要去校验这个函数都依赖是否被更改。**

这其中, **寻找到需要校验的计算属性和进行校验这两个步骤都需要成本**。

当我们大量的使用 useMemo 之后，非但不能给项目带来性能上的优化，反而会为项目增加负担，我们将这种情况戏称为：反向优化。

## useMemo 在什么情况下使用

刚刚上面说，useMemo 不能滥用，那如何用才不能算是滥用呢？

### 第一种情况

**当我们的某一个计算属性真的需要大量的计算时候**

举个例子 **代码示例（三）**

```jsx
//Com组件
const Com = () => {
    
    //这种就是完全没必要被useMemo缓存的，计算过程一共也就一个创建变量，一个加一，缓存它反而亏本
     const computedFun1 = () => {
        let number = 0;
        number = numebr +1;
        return number;
    }

    //这个就需要缓存一下了，毕竟他每次计算的计算量还是蛮大的。
    const computedFun2 = () => {
        let number =  0;
        for(let i=0;i<100000;++i){
            number = number +i-(number-i*1.1);
        }
        return number;
    }
    
    return <div onClick = {handleFun1}>
        computed1:{computedFun1()}  //这个计算量小，是不需要使用useMemo缓存的，缓存它反而亏本
        computed2:{computedFun2()} //这个计算量大，需要缓存。
    </div>
}

```

在上面的示例中。

computedFun1 这个函数是完全没必要去缓存的。计算量太小了。我们使用 useMemo 成本都比它计算的成本高。

而 computedFun2 这个函数是需要去缓存的，毕竟这个函数都计算量属实有点大了。缓存起来，能不执行就不执行。

### 第二种情况

**当子组件依赖父组件的某一个依赖计算属性并且子组件使用了 React.memo 进行优化了的时候。**

我想, 能看到这里的同学一定不需要我太详细的讲解什么是 React.memo。

简单说，React.memo() 是通过校验 props 中的数据是否改变的来决定组件是否需要重新渲染的一种缓存技术，**具体点说 React.memo() 其实是通过校验 Props 中的数据的内存地址是否改变来决定组件是否重新渲染组件的一种技术。**

假设我们往子组件传入一个**计算属性**，当父组件的**其他 State**（**与子组件无关的 state**）改变的时候。那么，因为状态的改变，父组件需要重新渲染，那被 React.memo 保护的子组件是否会被重新构建？

就这个问题, 举个栗子。有如下↓代码片段

**代码示例一**

```jsx
import {useMemo,memo} from 'react';
/**父组件**/
const Parent = () => {
    const [parentState,setParentState] = useState(0);  //父组件的state

    //需要传入子组件的函数
    const toChildComputed = () => {
        console.log("需要传入子组件的计算属性");
        return 1000;
    }

    return (<div>
            <Button onClick={() => setParentState(val => val+1)}>
                点击我改变父组件中与Child组件无关的state
            </Button>
            //将父组件的函数传入子组件
            <Child computedParams={toChildComputed()}></Child>
    <div>)
}

// 被memo保护的子组件

const Child = memo(() => {
    consolo.log("我被打印了就说明子组件重新构建了")
    return <div><div>
})

```

问: 当我点击父组件中的 Button 改变父组件中的 state。子组件会不会重新渲染。乍一看，改变的是 parentState 这个变量，和子组件半毛钱关系没有，子组件还被 React.memo 保护着，好像是不会被重新渲染。但这里的问题是，你要传个其他变量进去这也就走的通了。**但是传入的是函数，不行，走不通。子组件会重新渲染。**

React.memo 检测的是 props 中数据的栈地址是否改变。而 ** 父组件重新构建的时候，如果不缓存计算属性，那么计算属性将会被重新计算执行，并返回一个新的值（这意味这返回了一个新的存储地址）, 新的计算属性传入到子组件中被 props 检测到栈地址更新。也就引发了子组件的重新渲染。

所以，在上面的代码示例里面，子组件是要被重新渲染的。

## **那么如何才能让子组件不进行重新渲染呢？useMemo 第二种使用方法来了。**

**useMemo 会在发现依赖没有变化之后返回旧的计算属性值。**

使用 useMemo 包一下需要传入子组件的那个计算属性。那样的话，父组件重新渲染，子组件中的函数就会因为被 useMemo 保护而返回旧的计算属性值，子组件就不会检测成地址变化，也就不会重选渲染。

还是上面的代码示例，我们进行以下优化。

**代码示例二**

```jsx
import {useMemo,memo} from 'react';
/**父组件**/
const Parent = () => {
    const [parentState,setParentState] = useState(0);  //父组件的state
    
    //需要传入子组件的函数
    //只有这里和上一个示例不一样！！
    //只有这里和上一个示例不一样！！
    //只有这里和上一个示例不一样！！
    //只有这里和上一个示例不一样！！
    //只有这里和上一个示例不一样！！
    //只有这里和上一个示例不一样！！
    const toChildComputed = useMemo(() => {
       console.log("需要传入子组件的计算属性");
       return 1000;
    },[])
    
    return (<div>
          <Button onClick={() => setParentState(val => val+1)}>
              点击我改变父组件中与Child组件无关的state
          </Button>
          //将父组件的函数传入子组件
          <Child computedParams={toChildComputed()}></Child>
    <div>)
}

/**被memo保护的子组件**/
const Child = memo(() => {
    consolo.log("我被打印了就说明子组件重新构建了")
    return <div><div>
})

```

这样，子组件就不会被重新渲染了。

代码示例一和代码示例二中的区别只有被传入的子组件的计算属性（toChildComputed 函数）是否被 useMemo 保护。

我们只需要使用 useMemo 保护一下父组件中传入子组件的那个函数（toChildComputed 函数）**保证它不会在没有必要的情况下返回一个新的内存地址就好了。**

# 总结

- **useMemo 是用来缓存计算属性的，它会在发现依赖未发生改变的情况下返回旧的计算属性值的地址。**

- **useMemo 绝不是用的越多越好，缓存这项技术本身也需要成本。**

- **useMemo 的使用场景之一是: 只需要给拥有巨大计算量的计算属性缓存即可。**

- **useMemo 的另一个使用场景是：当有计算属性被传入子组件，并且子组件使用了 react.memo 进行了缓存的时候, 为了避免子组件不必要的渲染时使用**
