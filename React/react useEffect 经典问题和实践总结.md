> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7077757011013664776)

一、背景
----

接触`Hooks`5 个月，从之前的毫无基础、照着老代码一步一步模仿，现在已经能应付大部分工作中的问题。这期间遇到过很多问题，虽然用这样那样的手段满足了需求，但是总不能理解这些手段背后的原理，为此也找了很多教程来看，由于这些教程都聚焦在如何使用，对于背后的心智模型没有做更多的解读，直到读了 Dan 的《useEffect 完整指南》，才有了” 哦😯，原来如此的感觉 “。

接下来我将从`useEffect`五个经典问题及自己实践两个维度来阐述我的理解。

二、`useEffect`五个经典问题
-------------------

## 1. 🤔 如何用`useEffect`模拟`componentDidMount`生命周期？

对于这个问题，大家会习惯性的想到`useEffect(()=>{fetchData()}),[])`, 将依赖项数组置为空即可，这是一个管用的方法，可是这个问题是个陷阱。函数组件和`class`组件的心智模型是不一样的，不能把他们混为一谈，在我看来，他们最大的区别是当状态变化时，函数组件相当于经历了轮回，从种子长成花朵，再从花朵结成种子，不同年份的花朵看起来一样，但终究不是同一朵，而`class`组件相当于一朵花，从鲜艳的花朵变成干花束，形态不同，但始终是那一朵。

## 2. 🤔 如何正确地在`useEffect`里请求数据？`[]`又是什么？

每个`useEffect`都是不同的，随着状态的变化，一个个`effect`出生，一个个`effect`死去，怎么保证出生的`effect`是我们想要的呢，这就需要借助依赖数组来注入遗传因子，请求过程中用到的所有的遗传因子都要写到数组里，当然这是最保险的方法，如果想更优雅一些，可以看引用文献及下面的实践总结。

## 3. 🤔 我应该把函数当做`effect`的依赖吗？

可以把函数作为`effect`的依赖。更多关于函数的最佳实践看下面的实践总结。

## 4. 🤔 为什么有时候会出现无限重复请求的问题？

分为两种情况，一种是没有依赖数组，那么每次渲染都会触发这个副作用，例如

```js
useEffect(()=>{
    fetchData()
})
```

还有一种是设置了依赖数组，但是依赖数组里的变量一直在变，例如

```js
const [data,setData] = useState()

useEffect(()=>{
    const fetchData = async() => {
        const res = await fetchNewData()
        setData(res.data)
    }
    fetchData()
},[data])
```

## 5. 🤔 为什么有时候在`effect`里拿到的是旧的`state`或`prop`？

`effect`里获取的始终应该是最新的`state`或`prop`，如果出现旧的`state`或`prop`，一定是因为依赖数组中的依赖项不完整，没能触发`effect`更新。

三、`useEffect`最佳实践总结
-------------------

## 1.`class`组件和函数组件的区别

* 每一个函数、每一个`useEffect`中的`props`和`state`都是独立的。不是`state`在不变的`effect`中变化，而是每一个`effect`中有不变的`state`
* 函数组件的状态会停留在某个状态，改变后的状态跟前者不是同一个，所以不同时机不同状态；`class`组件中，状态始终指向`this.state`，不同时机但是同一个状态
* `useEffect`并不等于`componentDidmount` + `componentDieUpdate`，将函数传入到子组件，如何仅在需要的时候触发这个请求，是分辨两者的最佳办法，函数式组件传入的函数是类的一个属性，是永远不会变的，他永远只会请求一次数据，如果传入它的依赖参数，依赖参数默认是一直变动的，所以就会一直重新请求数据。如果用`hooks`传递，设置好适当的依赖项，就只会在依赖项变动的时候重新请求数据
* `useEffect`先渲染，后执行`preState`和`curState`的副作用

## 2. 如何定义函数和请求

* 某些函数只在`effect`中使用的话，那就在`effect`里定义
* 某些函数在多个地方用到，就独立定义，最好用`useCallBack`包裹，并且在依赖数组里把依赖项写全
* `useEffect`回调函数不能用`async`修饰，但是可以在回调函数内部重新写一个`async`函数，然后调用

```jsx
//不可以
useEffect(async()=>{
    const res = await fetchNewData(id)
    setData(res.data)
},[id])


//推荐
useEffect(()=>{
const fetchData = async() => {
    const res = await fetchNewData(id)
    setData(res.data)
}
fetchData()
},[id])
```

四、参考文献
------

1. [useEffect 完整指南](https://link.juejin.cn?target=https%3A%2F%2Foverreacted.io%2Fzh-hans%2Fa-complete-guide-to-useeffect%2F%23tldr "https://overreacted.io/zh-hans/a-complete-guide-to-useeffect/#tldr")
2. [[译] 如何用 React Hooks 获取数据](https://link.juejin.cn?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F104946710 "https://zhuanlan.zhihu.com/p/104946710")
