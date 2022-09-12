---
date created: 2022-07-04 13:32
---

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [segmentfault.com](https://segmentfault.com/a/1190000022112887)

> 一句话介绍 HOC 高阶组件就是接受一个组件作为参数并返回一个新组件（功能增强的组件）的函数。这里需要注意高阶组件是一个函数，并不是组件，这一点一定要注意...

**一句话介绍 HOC**\
高阶组件就是接受一个组件作为参数并返回一个新组件（功能增强的组件）的函数。这里需要注意高阶组件是一个函数，并不是组件，这一点一定要注意。\
**使用场景**\
将几个功能相似的组件里面的方法和 react 特性（如生命周期里面的副作用）提取到 HOC 中，然后向 HOC 传入需要封装的组件。最后将公用的方法传给组件。\
**优势**\
使代码简洁优雅、代码量更少\
**代码示例：**\
高阶组件：

```jsx
//定义高阶组件 //放在common公用方法里面
import React, { Component} from 'react';
export default (WrappedComponent) => {  
  return class extends Component {
    constructor(props) {
      super(props)
      this.state = { //定义可复用的状态
          count:1
      }
      this.getCode = this.getCode.bind(this) 
    }

    componentDidMount() {
      alert('111')
    }

　　　　//定义可复用的方法
    getCode(mobile) {
      // ...
      this.setState({count : mobile})
      console.log(mobile)
    }
    postVcode(mobile) {
      // ...
    }
    render() {
      return (
        <div>
          <WrappedComponent getCode={this.getCode} state={this.state} {...this.props}/>
        </div>
      )
    }
  }
}

// 高阶组件就是把可复用的逻辑放在定义的高阶组件中，然后把需要调用高阶组件的组件作为参数传给高阶组件，然后在高阶组件中
// 实例化该组件，再把需要复用的方法和state传给次组件，然后在此组件中的props中就可以拿到高阶组件里定义的逻辑方法和state了

```

传入的组件：

```jsx
import React, { Component} from 'react';
import HOC from './index'
//高阶组件的使用
class Register extends Component{
  render() {
    return (
      <div>
        <button onClick={()=>{this.props.getCode('17682334636')}}>使用高阶组件里复用的方法</button>
        {this.props.state.count}
      </div>
    )
  }
}
export default HOC(Register)
//高阶组件其实就是将复用的方法传给组件，使代码简洁、代码量更少

```

看到这里，hoc 基本就会用了。
