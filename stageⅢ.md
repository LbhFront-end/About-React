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

### 高阶组件的灵活性

代码复用的方法、形式有很多种，你可以用类继承来做到代码复用，也可以分离模块的方式。但是高阶组件这种方式很有意思，也很灵活。学过设计模式的同学其实应该能反应过来，它其实就是设计模式里面的装饰者模式。它通过组合的方式达到很高的灵活程度。

假设现在我们需求变化了，现在要的是通过 Ajax 加载数据而不是从 LocalStorage 加载数据。我们只需要新建一个 `wrapWithAjaxData` 高阶组件：

```
import React , { Component } from 'react'

export default (WrappendComponent, name) => {
    class NewComponent extends Component {
        constructor() {
            super()
            this.state = { data : null }
        }
        
        componentWillMount () {
            ajax.get('/data/' + name, (data) => {
                this.setState({ data });
            });
        }
        render() {
            return <WrappedComponent data={this.state.data}>
        }
    }
}
```

其实就是改了一下 `wrapWithLoadData` 的 `componentWillMount` 中的逻辑，改成了从服务器加载数据。现在只需要把 `InputWithUserName` 稍微改一下： 

```
import wrapWithAjaxData from './WrapWithAjaxData'

class InputWithUserName extends Component {
    render() {
        return <input value={this.props.data} />
    }
}
InputWithUserName = wrapWithAjaxData(InputWithUserName,'name')
export default InputWithUserName
```

只要改一下包装的高阶组件就可以达到需要的效果。而且我们并没有改动 `InputWithUserName` 组件内部的任何逻辑，也没有改动 `Index` 的任何逻辑，只是改动了中间的高阶组件函数。

### 多层高阶组件

假如现在需求有变化了：我们需要先从 LocalStorage 中加载数据，再用这个数据去服务器取数据。我们改一下（或者新建一个）`wrapWithAjaxData` 高阶组件，修改其中的 `componentWillMount`：



```
...
    componentWillMount () {
      ajax.get('/data/' + this.props.data, (data) => {
        this.setState({ data })
      })
    }
...
```

它会用传进来的 `props.data` 去服务器取数据。这时候修改 `InputWithUserName`：

```
import wrapWithLoadData from './wrapWithLoadData'
import wrapWithAjaxData from './wrapWithAjaxData'

class InputWithUserName extends Component {
    render(){
        return <input value={this.props.data}/>
    }
}

InputWithUserName = wrapWithAjaxData(InputWithUserName);
InputWithUserName = wrapWithLoadData(InputWithUserName, 'username');
export default InputWithUserName
```

我们给 `InputWithUserName` 应用了两种高阶组件：先用 `wrapWithAjaxData` 包裹 `InputWithUserName`，再用 `wrapWithLoadData `包含上次包裹的结果。它们的关系就如下图的三个圆圈： 

![](https://huzidaha.github.io/static/assets/img/posts/A8F1DD5F-1995-419E-8551-4FC2D59F58B4.png)

实际上最终得到的组件会先去 LocalStorage 取数据，然后通过 `props.data` 传给下一层组件，下一层用这个 `props.data` 通过 Ajax 去服务端取数据，然后再通过 `props.data` 把数据传给下一层，也就是 `InputWithUserName`。大家可以体会一下下图尖头代表的组件之间的数据流向： 

![](https://huzidaha.github.io/static/assets/img/posts/8F6C1E91-B365-4919-84C3-2252223621F8.png)

### 用高阶组件改造评论功能

大家对这种在挂载阶段从 LocalStorage 加载数据的模式都很熟悉，在上一阶段的实战中，`CommentInput` 和 `CommentApp` 都用了这种方式加载、保存数据。实际上我们可以构建一个高阶组件把它们的相同的逻辑抽离出来，构建一个高阶组件 `wrapWithLoadData`：

```
export default (WrappedComponent, name) => {
    class LocalStorageAction extends Component {
        constructor() {
            super();
            this.state = { data : null}
        }
        componentWillMount() {
            let data = localStorage.getItem(name)
            try {
                // 尝试把它解析成 JSON 对象
                this.setState({ data: JSON.parse(data)})
            }catch (e){
                // 如果出错了就当作普通字符串读取
                this.setState({ data })
            }
        }
        saveData(data) {
            try{
            //尝试把它解析为JSON字符串
            localStorage.setItem(name, JSON.stringify(data))                
            }catch(e){
            //如果出错了就当作普通字符串保存
            localStorage.setItem(name, `${data}`); 
            }
        }
        render() {
            return(
            	<WrappendComponent 
            		data={this.state.data}
            		saveData={this.saveData.bind(this) 
            		//这里的意思是把其他参数原封不动地传递给被包装的组件
            		{...this.props}/>
            )
        }
    }
    return LocalStorageActions
}
```

`CommentApp` 可以这样使用： 

```
class CommentApp extends Component {
    static propTypes = {
        data: PropTypes.any,
        saveData: ProTypes.func.isRequired
    }
    constructor(){
        super()
        this.state = { comments: props.data }
    }
    render(){
        return(
        	<div className='wrapper'>
        		<CommentInput onSubmit={this.handleSubmitComment.bind(this)}/>
        		<CommentList 
        			comments={this.state.comments}
                     onDeleteComment={this.handleDeleteComment.bind(this)}/>
        	</div>
        )
    }
}
CommentApp = wrapWithLoadData(CommentApp, 'comments')
export default CommentApp
```

同样地可以在 `CommentInput` 中使用 `wrapWithLoadData`，这里就不贴代码了。有兴趣的同学可以查看[高阶组件重构的 CommentApp 版本](https://github.com/huzidaha/react-naive-book-examples/commit/0d67eab713c042301fa4992c719069e92a7243f5)。 

### 总结

*高阶组件就是一个函数，传给它一个组件，它返回一个新的组件*。新的组件使用传入的组件作为子组件。

*高阶组件的作用是用于代码复用*，可以把组件之间可复用的代码、逻辑抽离到高阶组件当中。*新的组件和传入的组件通过 props 传递信息*。

高阶组件有助于提高我们代码的灵活性，逻辑的复用性。灵活和熟练地掌握高阶组件的用法需要经验的积累还有长时间的思考和练习，如果你觉得本章节的内容无法完全消化和掌握也没有关系，可以先简单了解高阶组件的定义、形式和作用即可。

------

## React.js 的 context

这一节我们来介绍一个你可能永远用不上的 React.js 特性 —— context。但是了解它对于了解接下来要讲解的 React-redux 很有好处，所以大家可以简单了解一下它的概念和作用。

在过去很长一段时间里面，React.js 的 context 一直被视为一个不稳定的、危险的、可能会被去掉的特性而不被官网文档所记载。但是全世界的第三方库都在用使用这个特性，直到了 React.js 的 v0.14.1 版本，context 才被官方文档所记录。

除非你觉得自己的 React.js 水平已经比较炉火纯青了，否则你永远不要使用 context。就像你学 JavaScript 的时候，总是会被提醒不要用全局变量一样，React.js 的 context 其实像就是组件树上某颗子树的全局变量。

想象一下我们有这么一棵组件树：

![](https://huzidaha.github.io/static/assets/img/posts/85C81DFF-F71E-4B2B-9BAB-AF285F3DB1DB.png)

假设现在这个组件树代表的应用是用户可以自主换主题色的，每个子组件会根据主题色的不同调整自己的字体颜色或者背景颜色。“主题色”这个玩意是所有组件共享的状态，根据我们在 [前端应用状态管理 —— 状态提升](https://github.com/LbhFront-end/About-React/blob/master/stage%E2%85%A1.md#前端应用状态管理-状态提升) 中所提到的，需要把这个状态提升到根节点的 `Index` 上，然后把这个状态通过 `props` 一层层传递下去： 

![](https://huzidaha.github.io/static/assets/img/posts/03118DDD-60E3-469A-AB78-5FBE57425E30.png)

假设原来主题色是绿色，那么 `Index` 上保存的就是 `this.state = { themeColor: 'green' }`。如果要改变主题色，可以直接通过 `this.setState({ themeColor: 'red' })`来进行。这样整颗组件树就会重新渲染，子组件也就可以根据重新传进来的 `props.themeColor` 来调整自己的颜色。

但这里的问题也是非常明显的，我们需要把 `themeColor` 这个状态一层层手动地从组件树顶层往下传，每层都需要写 `props.themeColor`。如果我们的组件树很层次很深的话，这样维护起来简直是灾难。

如果这颗组件树能够全局共享这个状态就好了，我们要的时候就去取这个状态，不用手动地传：

![](https://huzidaha.github.io/static/assets/img/posts/3BC6BDFC-5772-4045-943B-15FBEC28DAC0.png)

就像这样，`Index` 把 `state.themeColor` 放到某个地方，这个地方是每个 `Index` 的子组件都可以访问到的。当某个子组件需要的时候就直接去那个地方拿就好了，而不需要一层层地通过 `props` 来获取。不管组件树的层次有多深，任何一个组件都可以直接到这个公共的地方提取 `themeColor` 状态。

React.js 的 context 就是这么一个东西，某个组件只要往自己的 context 里面放了某些状态，这个组件之下的所有子组件都直接访问这个状态而不需要通过中间组件的传递。一个组件的 context 只有它的子组件能够访问，它的父组件是不能访问到的，你可以理解每个组件的 context 就是瀑布的源头，只能往下流不能往上飞。

我们看看 React.js 的 context 代码怎么写，我们先把整体的组件树搭建起来，这里不涉及到 context 相关的内容：

```
class Index extends Component {
    render() {
        return (
        	<div>
        		<Header />
        		<Main />
        	</div>
        )
    }
}
class Header extends Component {
    render() {
        return(
        	<div>
              <h2>This is header</h2>
              <Title />        	
        	</div>
        )
    }
}
class Main extends Component {
  render () {
    return (
    <div>
      <h2>This is main</h2>
      <Content />
    </div>
    )
  }
}

class Title extends Component {
  render () {
    return (
      <h1>React.js 小书标题</h1>
    )
  }
}

class Content extends Component {
  render () {
    return (
    <div>
      <h2>React.js 小书内容</h2>
    </div>
    )
  }
}

ReactDOM.render(
  <Index />,
  document.getElementById('root')
)
```

代码很长但是很简单，这里就不解释了。

现在我们修改 `Index`，让它往自己的 context 里面放一个 `themeColor`：

```
class Index extends Component {
    static childContextTypes = {
        themeColor: PropTypes.string
    }
    constructor() {
        super()
        this.state = { themeColor:'red' }
    }
    getChildContext() {
        return { themeColor:this.state.themeColor }
    }
    render() {
        return(
        	<div>
        		<Header />
        		<Main />
        	</div>
        )
    }
}
```

构造函数里面的内容其实就很好理解，就是往 state 里面初始化一个 `themeColor` 状态。`getChildContext` 这个方法就是设置 `context` 的过程，它返回的对象就是 context（也就是上图中处于中间的方块），所有的子组件都可以访问到这个对象。我们用 `this.state.themeColor` 来设置了 context 里面的 `themeColor`。

还有一个看起来很可怕的 `childContextTypes`，它的作用其实 `propsType` 验证组件 `props` 参数的作用类似。不过它是验证 `getChildContext` 返回的对象。为什么要验证 context，因为 context 是一个危险的特性，按照 React.js 团队的想法就是，把危险的事情搞复杂一些，提高使用门槛人们就不会去用了。如果你要给组件设置 context，那么 `childContextTypes` 是必写的。

现在我们已经完成了 `Index` 往 context 里面放置状态的工作了，接下来我们要看看子组件怎么获取这个状态，修改 `Index` 的孙子组件 `Title`：

```
class Title extends Component {
    static contextTypes = {
        themeColor: PropTypes.string
    }
    render(){
        return (
        	<h1 style={{ color: this.context.themeColor}}> 哈哈哈哈</h1>
        )
    }
}
```

子组件要获取 context 里面的内容的话，就必须写 `contextTypes` 来声明和验证你需要获取的状态的类型，它也是必写的，如果你不写就无法获取 context 里面的状态。`Title` 想获取 `themeColor`，它是一个字符串，我们就在 `contextTypes` 里面进行声明。

声明以后我们就可以通过 `this.context.themeColor` 获取到在 `Index` 放置的值为 `red` 的 `themeColor`，然后设置 `h1` 的样式

如果我们要改颜色，只需要在 `Index` 里面 `setState` 就可以了，子组件会重新渲染，渲染的时候会重新取 context 的内容，例如我们给 `Index` 调整一下颜色：

```
...
  componentWillMount () {
    this.setState({ themeColor: 'green' })
  }
...
```

那么 `Title` 里面的字体就会显示绿色。我们可以如法炮制孙子组件 `Content`，或者任意的 `Index` 下面的子组件。让它们可以不经过中间 `props` 的传递获就可以获取到由 `Index` 设定的 context 内容。

### 总结

一个组件可以通过 `getChildContext` 方法返回一个对象，这个对象就是子树的 context，提供 context 的组件必须提供 `childContextTypes` 作为 context 的声明和验证。

如果一个组件设置了 context，那么它的子组件都可以直接访问到里面的内容，它就像这个组件为根的子树的全局变量。任意深度的子组件都可以通过 `contextTypes` 来声明你想要的 context 里面的哪些状态，然后可以通过 `this.context` 访问到那些状态。

context 打破了组件和组件之间通过 `props` 传递数据的规范，极大地增强了组件之间的耦合性。而且，就如全局变量一样，*context 里面的数据能被随意接触就能被随意修改*，每个组件都能够改 context 里面的内容会导致程序的运行不可预料。

但是这种机制对于前端应用状态管理来说是很有帮助的，因为毕竟很多状态都会在组件之间进行共享，context 会给我们带来很大的方便。一些第三方的前端应用状态管理的库（例如 Redux）就是充分地利用了这种机制给我们提供便利的状态管理服务。但我们一般不需要手动写 context，也不要用它，只需要用好这些第三方的应用状态管理库就行了。

------

## 动手实现 Redux（一）：优雅地修改共享状态

从这节起我们开始学习 Redux，一种新型的前端“架构模式”。经常和 React.js 一并提出，你要用 React.js 基本都要伴随着 Redux 和 React.js 结合的库 React-redux。

要注意的是，Redux 和 React-redux 并不是同一个东西。Redux 是一种架构模式（Flux 架构的一种变种），它不关注你到底用什么库，你可以把它应用到 React 和 Vue，甚至跟 jQuery 结合都没有问题。而 React-redux 就是把 Redux 这种架构模式和 React.js 结合起来的一个库，就是 Redux 架构在 React.js 中的体现。

如果把 Redux 的用法重新介绍一遍那么这本书的价值就不大了，我大可把官网的 Reducers、Actions、Store 的用法、API、关系重复一遍，画几个图，说两句很玄乎的话。但是这样对大家理解和使用 Redux 都没什么好处，本书初衷还是跟开头所说的一样：希望大家对问题的根源有所了解，了解这些工具到底解决什么问题，怎么解决的。

现在让我们忘掉 React.js、Redux 这些词，从一个例子的代码 + 问题开始推演。

用 `create-react-app` 新建一个项目 `make-redux`，修改 `public/index.html` 里面的 `body` 结构为：



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