# 第三阶段


1. [高阶组件（Higher-Order Components）](https://github.com/LbhFront-end/About-React/blob/master/stageⅢ.md#高阶组件higher-order-components)
2. [React.js 的 context](https://github.com/LbhFront-end/About-React/blob/master/stageⅢ.md#reactjs-的-context)
3. [动手实现 Redux（一）：优雅地修改共享状态](https://github.com/LbhFront-end/About-React/blob/master/stageⅢ.md#动手实现-redux一优雅地修改共享状态)
4. [动手实现 Redux（二）：抽离 store 和监控数据变化](https://github.com/LbhFront-end/About-React/blob/master/stageⅢ.md#动手实现-redux二抽离-store-和监控数据变化)
5. [动手实现 Redux（三）：纯函数（Pure Function）简介](https://github.com/LbhFront-end/About-React/blob/master/stageⅢ.md#动手实现-redux三纯函数pure-function简介)
6. [动手实现 Redux（四）：共享结构的对象提高性能](https://github.com/LbhFront-end/About-React/blob/master/stageⅢ.md#动手实现-redux四共享结构的对象提高性能)
7. [动手实现 Redux（五）：不要问为什么的 reducer](https://github.com/LbhFront-end/About-React/blob/master/stageⅢ.md#动手实现-redux五不要问为什么的-reducer)
8. [动手实现 Redux（六）：Redux 总结](https://github.com/LbhFront-end/About-React/blob/master/stageⅢ.md#动手实现-redux六redux-总结)
9. [动手实现 React-redux（一）：初始化工程](https://github.com/LbhFront-end/About-React/blob/master/stageⅢ.md#动手实现-react-redux一初始化工程)
10. [动手实现 React-redux（二）：结合 context 和 store](https://github.com/LbhFront-end/About-React/blob/master/stageⅢ.md#动手实现-react-redux二结合-context-和-store)
11. [动手实现 React-redux（三）：connect 和 mapStateToProps](https://github.com/LbhFront-end/About-React/blob/master/stageⅢ.md#动手实现-react-redux三connect-和-mapstatetoprops)
12. [动手实现 React-redux（四）：mapDispatchToProps](https://github.com/LbhFront-end/About-React/blob/master/stageⅢ.md#动手实现-react-redux四mapdispatchtoprops)
13. [动手实现 React-redux（五）：Provider](https://github.com/LbhFront-end/About-React/blob/master/stageⅢ.md#动手实现-react-redux五provider)
14. [动手实现 React-redux（六）：React-redux 总结](https://github.com/LbhFront-end/About-React/blob/master/stageⅢ.md#动手实现-react-redux六react-redux-总结)
15. [使用真正的 Redux 和 React-redux](https://github.com/LbhFront-end/About-React/blob/master/stageⅢ.md#使用真正的-redux-和-react-redux)
16. [Smart 组件 vs Dumb 组件](https://github.com/LbhFront-end/About-React/blob/master/stageⅢ.md#smart-组件-vs-dumb-组件)





## 高阶组件（Higher-Order Components）

------

（本文未审核）

有时候人们很喜欢造一些名字很吓人的名词，让人一听这个名词就觉得自己不可能学会，从而让人望而却步。但是其实这些名词背后所代表的东西其实很简单。

我不能说高阶组件就是这么一个东西。但是它是一个概念上很简单，但却非常常用、实用的东西，被大量 React.js 相关的第三方库频繁地使用。在前端的业务开发当中，你不掌握高阶组件其实也可以完成项目的开发，但是如果你能够灵活地使用高阶组件，可以让你代码更加优雅，复用性、灵活性更强。它是一个加分项，而且加的分还不少。

本章节可能有部分内容理解起来会有难度，如果你觉得无法完全理解本节内容。可以先简单理解高阶组件的概念和作用即可，其他内容选择性地跳过。

了解高阶组件对我们理解各种 React.js 第三方库的原理很有帮助。

### 什么是高阶组件

*高阶组件就是一个函数，传给它一个组件，它返回一个新的组件。*

```
const NewCompoent = higherOrderComponent(OldComponent)
```

高阶组件是一个函数（而不是组件），它接受一个组件作为参数，返回一个新的组件。这个新的组件会使用你传给它的组件作为子组件。

```
import React,{Component} from 'react'

export default (WrappedComponent) => {
    class NewComponent extends Component {
    // 可以做自定义逻辑
        render() {
        	return <WrappedComponent />
    	}
	}
	return NewComponent
}
```

看起来好像就是简单的狗将了一个新的组件类 `NewComponent`,然后把传进去的 `WrappedComponent`渲染出来。但是我们可以给 `NewComponent`做一些数据启动工作

```
import React , { Component } from 'react'
export default (WrappedComponent, name) => {
    class NewComponent extends Component {
        constructor() {
            super()
            this.state = {data:null}
        }
        componentWillMout(){
            let data = localStorage.getItem(name)
            this.setState({ data })
        }
        render() {
            return <WrappedComponent data = {this.state.data} />
        }
    }
    return NewComponent
}
```

现在 `NewComponent` 会根据第二个参数 `name` 在挂载阶段从 LocalStorage 加载数据，并且 `setState` 到自己的 `state.data` 中，而渲染的时候将 `state.data` 通过 `props.data` 传给 `WrappedComponent`。

这个高阶组件有什么用呢？假设上面的代码是在 `src/wrapWithLoadData.js` 文件中的，我们可以在别的地方这么用它：

```
import wrapWithLoadData from './wrapWithLoadData'

class InputWithUserName extends Component {
    render(){
        return <input value = {this.props.data}/>
    }
}
InputWithUserName = new wrapWithLoadData(InputWithUserName,'username')
export default InputWithUserName
```

假如 `InputWithUserName` 的功能需求是挂载的时候从 LocalStorage 里面加载 `username` 字段作为 `<input />` 的 `value` 值，现在有了 `wrapWithLoadData`，我们可以很容易地做到这件事情。

只需要定义一个非常简单的 `InputWithUserName`，它会把 `props.data` 作为 `<input />` 的 `value` 值。然把这个组件和 `'username'` 传给 `wrapWithLoadData`，`wrapWithLoadData` 会返回一个新的组件，我们用这个新的组件覆盖原来的 `InputWithUserName`，然后再导出去模块。

别人用这个组件的时候实际是用了*被加工过*的组件：

```
import InputWithUserName from './InputWithUserName'

class Index extends Component {
    render() {
        return (
        	<div>
        		用户名： <InputWithUserName />
        	</div>
        )
    }
}
```

根据 `wrapWithLoadData` 的代码我们可以知道，这个新的组件挂载的时候会先去 LocalStorage 加载数据，渲染的时候再通过 `props.data` 传给真正的 `InputWithUserName`。

如果现在我们需要另外一个文本输入框组件，它也需要 LocalStorage 加载 `'content'` 字段的数据。我们只需要定义一个新的 `TextareaWithContent`：

```
import wrapWithLoadData from './wrapWithLoadData'

class TextareaWithContent extends Component {
    render() {
        return <textarea value = this.props.data/>
    }
}
TextareaWithContent = new wrapWithLoadData(TextareaWithContent,'content');
export default TextareaWithContent
```

起来非常轻松，我们根本不需要重复写从 LocalStorage 加载数据字段的逻辑，直接用 `wrapWithLoadData` 包装一下就可以了。

我们来回顾一下到底发生了什么事情，对于 `InputWithUserName` 和 `TextareaWithContent` 这两个组件来说，它们的需求有着这么一个相同的逻辑：“挂载阶段从 LocalStorage 中加载特定字段数据”。

如果按照之前的做法，我们需要给它们两个都加上 `componentWillMount` 生命周期，然后在里面调用 LocalStorage。要是有第三个组件也有这样的加载逻辑，我又得写一遍这样的逻辑。但有了 `wrapWithLoadData` 高阶组件，我们把这样的逻辑用一个组件包裹了起来，并且通过给高阶组件传入 `name` 来达到不同字段的数据加载。充分复用了逻辑代码。

到这里，高阶组件的作用其实不言而喻，*其实就是为了组件之间的代码复用*。组件可能有着某些相同的逻辑，把这些逻辑抽离出来，放到高阶组件中进行复用。*高阶组件内部的包装组件和被包装组件之间通过 props 传递数据*。

## React.js 的 context

------

## 动手实现 Redux（一）：优雅地修改共享状态

------

## 动手实现 Redux（二）：抽离 store 和监控数据变化

------

## 动手实现 Redux（三）：纯函数（Pure Function）简介

------

## 动手实现 Redux（四）：共享结构的对象提高性能

------

## 动手实现 Redux（五）：不要问为什么的 reducer

------

## 动手实现 Redux（六）：Redux 总结

------

## 动手实现 React-redux（一）：初始化工程

------

## 动手实现 React-redux（二）：结合 context 和 store

------

## 动手实现 React-redux（三）：connect 和 mapStateToProps

------

## 动手实现 React-redux（四）：mapDispatchToProps

------

## 动手实现 React-redux（五）：Provider

------

## 动手实现 React-redux（六）：React-redux 总结

------

## 使用真正的 Redux 和 React-redux

------

## Smart 组件 vs Dumb 组件

------