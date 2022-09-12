---
date created: 2022-05-25 22:43
date updated: 2022-05-30 22:26
---
#React 

<https://juejin.cn/post/6955636911214067720>

## 相关问题

- 说一下 react 的事件机制
- react 为什么需要合成事件
- react 合成事件是什么, 和原生事件的区别
- react 合成事件与原生 DOM 事件的区别
- react 事件如何解决浏览器的兼容问题

## 概括

React 的事件处理机制可以分为两个阶段:

- 初始化渲染时在 root 节点上注册原生事件;
- 原生事件触发时模拟捕获, 目标和冒泡阶段派发合成事件

通过这种机制, 冒泡的原生事件类型最多在 root 节点上注册一次, 节省内存开销. 而且 React 为不同类型的时间定义了不同的处理优先级, 从而让用户代码及时响应高优先级的用户交互, 提升用户体验.
React 的事件机制中依赖合成事件这个核心概念, 合成事件在符合 W3C 规范的前提下, 做了各个浏览器兼容. 并且简化了事件处理逻辑, 对关联事件进行合成. 比如: `onChange=> ['blur', 'change', 'input', 'keydown', 'keyup']` 就集合了不同的原生事件. 它在 react 中记录为 `registrationNameDependencies`

```jsx
/**
 * Mapping from registration name to event name
 */
export const registrationNameDependencies = {
  onClick: ["click"],
  onMouseEnter: ["mouseout", "mouseover"],
  onChange: [
    "change",
    "click",
    "focusin",
    "focusout",
    "input",
    "keydown",
    "keyup",
    "selectionchange",
  ],
  // ...
};
```

## 详情

### 原生事件和合成事件

#### 原生事件

浏览器基于 DOM2 DOM3 规范实现了标准化的 DOM 事件, 基于 Event 实现了浏览器中常见的用户事件.
以下是浏览器 Event 事件属性:

```jsx
// Event 属性
boolean bubbles
boolean cancelable
DOMEventTarget currentTarget
boolean defaultPrevented
number eventPhase
boolean isTrusted
void preventDefault()
void stopPropagation()
void stopImmediatePropagation()
DOMEventTarget target
number timeStamp
string type
```

#### React 合成事件

React 的事件机制在遵循规范的前提下, 引入新的事件类型:  **SyntheticEvent** 合成事件, 该实例会传递给事件处理函数, 它是浏览器的原生事件的跨浏览器包装器, 提供不同浏览器上一致的属性.

事件触发时, 相关信息会被存储在 `SyntheticEvent` 的实例对象中, 对象包含原生事件对象类似的属性:

```jsx
// SyntheticEvent 属性
boolean bubbles
boolean cancelable
DOMEventTarget currentTarget
boolean defaultPrevented
number eventPhase
boolean isTrusted
DOMEvent nativeEvent        // 不同
void preventDefault()
boolean isDefaultPrevented()
void stopPropagation()
boolean isPropagationStopped()
void persist()
DOMEventTarget target
number timeStamp
string type
```

## react 事件要点

- fiber 节点通过 `memoizedProps` 和 `pendingProps` 保存了在 JSX 中添加的事件函数

### react 事件绑定过程

- 在 diff fiber 节点阶段, 如果发现是 React 合成事件, 比如 `onClick` , 会按照事件系统逻辑单独处理
- 根据 React 合成事件类型, 找到对应的原生事件的类型, 然后调用判断原生事件类型, 大部分事件都按照冒泡逻辑处理, 少部分事件会按照捕获阶段逻辑处理, 比如 `scroll` 事件
- 调用 `addTrappedEventListener` 方法进行真正的事件绑定, 绑定在 `document` 上, `dispatchEvent` 为统一的事件处理函数
- **只有上述几个特殊事件 `scroll` `focus` `blur` 等等, 是在事件捕获阶段发生的, 其它事件都是在事件冒泡阶段发生的. 无论是 `onClick` 还是 `onClickCapture` 都是发生在冒泡阶段**

### `dispatchEvent` 函数工作原理

1. 首先根据真实的事件源对象, 找到 `e.target` 真的 DOM 元素
2. 然后根据 DOM 元素找对与之对应的 fiber 节点中的 `targetList`
3. 然后正式进入 `legacy` 模式的事件处理系统, 进行批量更新 [[react setState 机制#1 钩子函数 合成事件中]]

### react 16 事件池

1. 首先形成 React 事件系统独有的合成事件源对象, 它保存了整个事件的信息, 作为参数传递给真正的事件处理函数 `handleCLick`
2. 然后声明事件执行队列, 按照**冒泡**和**捕获**逻辑, 从事件源开始逐渐向上, 查找 DOM 元素类型 `HostComponent` 对应的 fiber 阶段, 收集 React 合成事件. 对于 `onclick` 的冒泡阶段事件会 push 到执行队列后面, 对于  `onClickCapture` 的捕获阶段事件, 会 unshift 到执行队列的前面

**例子:**

```jsxx
handleClick1 = () => console.log(1)
handleClick2 = () => console.log(2)
handleClick3 = () => console.log(3)
handleClick4 = () => console.log(4)
render() {
  return <div onClick={handleClick3} onClickCaptrue={handleClick4}>
    <button onClick={handleClick1} onClickCapture={handleClick2}>点击</button>
  </div> 
}
// 打印: 4 2 1 3
```

![[事件池执行顺序 1.jpg]]
来看一个例子:

```jsx
 handerClick = (e) => {
    console.log(e.target) // button 
    setTimeout(()=>{
        console.log(e.target) // null
    },0)
}
```

**事件池的意义在于, 在事件函数执行之后, 可以通过 `releaseTopLevelCallbackBookKeeping` 等方法将事件源对象释放到事件池中. 好处在于每次我们不比再创建事件源对象, 可以从事件池中取出相关源对象进行复用, 在事件处理函数执行完毕后, 会释放事件源对象到事件池中, 清空属性**

### react 事件触发总结

1. 首先通过统一的时间处理函数 `dispatchEvent` 进行批量更新 batchUpdate
2. 然后执行事件对应的处理插件中的 `extractEvent`, 合成事件源对象, 每次 React 都会从事件源开始, 从上遍历类型为 `hostComponent` 的 fiber 节点, 判断 props 中是否有当前事件, 最终形成一个事件队列, React 利用该队列, 模拟了 ` 事件捕获-> 事件源 -> 事件冒泡 ` 这一过程
3. 最后通过 `runEventsInBatch` 执行事件队列, 如果发现阻止冒泡, 就跳出循环, 最后重置事件源, 放回事件池中, 完成整个流程.

![[事件触发流程.jpg]]

## React 17 版本事件系统变更

1. **事件统一绑定在 container 上(也就是 div id: root) , ReactDOM.render(app, container);** 而不是绑定在 `document` 上, 这样的好处是有利于微前端, 因为微前端一个文档可能有多个应用, 如果绑在 document 上会引发问题
2. 对齐原生浏览器事件.  React 17 升级支持了原生捕获事件, 对齐了浏览器原生标准. 同时 `onScroll` 事件不再进行事件冒泡. `onFocus` 和 `onBlur` 使用原生 `focusin` , `focusout` 合成
3. **取消事件池** 解决了上述 `setTimeout` 打印 `e.target` 为 null 的问题

## 还有另一个表述

详情看 [[react 事件机制原理]]
