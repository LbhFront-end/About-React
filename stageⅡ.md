# 第二阶段


1. [前端应用状态管理-状态提升](https://github.com/LbhFront-end/About-React/blob/master/stageⅡ.md#前端应用状态管理-状态提升)
2. [挂载阶段的组件生命周期（一）](https://github.com/LbhFront-end/About-React/blob/master/stageⅡ.md#挂载阶段的组件生命周期（一）)
3. [挂载阶段的组件生命周期（二）](https://github.com/LbhFront-end/About-React/blob/master/stageⅡ.md#挂载阶段的组件生命周期（二）)
4. [更新阶段的组件生命周期](https://github.com/LbhFront-end/About-React/blob/master/stageⅡ.md#更新阶段的组件生命周期)
5. [ref 和 React.js 中的 DOM 操作](https://github.com/LbhFront-end/About-React/blob/master/stageⅡ.md#ref-和-react.js-中的-dom-操作)
6. [props.children 和容器类组件](https://github.com/LbhFront-end/About-React/blob/master/stageⅡ.md#props.children-和容器类组件)
7. [dangerouslySetHTML 和 style 属性](https://github.com/LbhFront-end/About-React/blob/master/stageⅡ.md#dangerouslysethtml-和-style-属性)
8. [PropTypes 和组件参数验证](https://github.com/LbhFront-end/About-React/blob/master/stageⅡ.md#propTypes-和组件参数验证)







## 前端应用状态管理-状态提升

上一个评论功能的案例中，可能会有些同学会对一个地方感到疑惑： `CommentList` 中显示的评论列表数据为什么要通过父组件 `CommentApp` 用 `props` 传进来？为什么不直接存放在 `CommentList` 的 `state` 当中？例如这样做也是可以的： 

[具体StageⅠ 实战代码点击这里](https://github.com/LbhFront-end/React-Project-Comment)

```
class CommentList extends Component {
    constructor(){
        this.state = {comments:[]}
    }
    addComment(comment){
        this.state.comment.push(comment);
        this.setState({ comments:this.state.comments });
    }
    render(){
        return(
        	<div>
        		{this.state.comments.map((comment,i) => <Comment comment = {comment} key={i}>)}
        	</div>
        )
    }
}
```

如果把这个 `comments` 放到 `CommentList` 当中，*当有别的组件也依赖这个 comments数据或者有别的组件会影响这个数据*，那么就带来问题了。举一个数据依赖的例子：例如，现在我们有另外一个和 `CommentList` 同级的 `CommentList2` ，也是需要显示同样的评论列表数据。 

![](https://huzidaha.github.io/static/assets/img/posts/85B8A2B7-288F-4FC2-A0AB-C4E153BB3854.png)

`CommentList2` 和 `CommentList` 并列为 `CommentApp` 的子组件，它也需要依赖 `comments` 显示评论列表。但是因为 `comments` 数据在 `CommentList` 中，它没办法访问到。

遇到这种情况，我们*将这种组件之间共享的状态交给组件最近的公共父节点保管*，然后通过 `props` 把状态传递给子组件，这样就可以在组件之间共享数据了。

![](https://huzidaha.github.io/static/assets/img/posts/C547BD3E-F923-4B1D-96BC-A77966CDFBEF.png)

在我们的例子当中，如果把 `comments` 交给父组件 `CommentApp` ，那么 `CommentList`和 `CommentList2` 都可以通过 `props` 获取到 `comments`，React.js 把这种行为叫做“状态提升”。

但是这个 `CommentList2` 是我们临时加上去的，在实际案例当中并没有涉及到这种组件之间依赖 `comments` 的情况，为什么还需要把 `comments` 提升到 `CommentApp`？那是因为有个组件会影响到 `comments` ，那就是 `CommentInput`。`CommentInput` 产生的新的评论数据是会插入 `comments` 当中的，所以我们遇到这种情况也会把状态提升到父组件。

总结一下：当某个状态被多个组件*依赖*或者*影响*的时候，就把该状态提升到这些组件的最近公共父组件中去管理，用 `props` 传递*数据或者函数*来管理这种*依赖*或着*影响*的行为。

我们来看看状态提升更多的例子，假设现在我们的父组件 `CommentApp` 只是属于更大的组件树 `PostApp` 的一部分：

![](https://huzidaha.github.io/static/assets/img/posts/5.007.png)

而这个更大的组件树的另外的子树的 `CommentsCount` 组件也需要依赖 `comments` 来显示评论数，那我们就只能把 `comments` 继续提升到这些依赖组件的最近公共父组件 `PostApp` 当中。

现在继续让我们的例子极端起来。假设现在 `PostApp` 只是另外一个更大的父组件 `Index` 的子树。而 `Index` 的某个子树的有一个按钮组件可以一键清空所有 `comments`（也就是说，这个按钮组件可以影响到这个数据），我们只能继续 `commenets` 提升到 `Index` 当中。

你会发现这种无限制的提升不是一个好的解决方案。一旦发生了提升，你就需要修改原来保存这个状态的组件的代码，也要把整个数据传递路径经过的组件都修改一遍，好让数据能够一层层地传递下去。这样对代码的组织管理维护带来很大的问题。到这里你可以抽象一下问题：

> 如何更好的管理这种被多个组件所依赖或影响的状态？

你可以看到 React.js 并没有提供好的解决方案来管理这种组件之间的共享状态。在实际项目当中状态提升并不是一个好的解决方案，所以我们后续会引入 Redux 这样的状态管理工具来帮助我们来管理这种共享状态，但是在讲解到 Redux 之前，我们暂时采取状态提升的方式来进行管理。

对于不会被多个组件依赖和影响的状态（例如某种下拉菜单的展开和收起状态），一般来说只需要保存在组件内部即可，不需要做提升或者特殊的管理。

------



## 挂载阶段的组件生命周期（一）

我们在[讲解 JSX 的章节](https://github.com/LbhFront-end/About-React/blob/master/Stage%E2%85%A0.md#使用jsx描述ui信息)中提到，下面的代码：

```
ReactDOM.render(
 <Header />, 
  document.getElementById('root')
)
```

会编译成：

```
ReactDOM.render(
  React.createElement(Header, null), 
  document.getElementById('root')
)
```

其实我们把 `Header` 组件传给了 `React.createElement` 函数，又把函数返回结果传给了 `ReactDOM.render`。我们可以简单猜想一下它们会干什么事情：

```
// React.createElement 中实例化一个 Header
const header = new Header(props,children);

//React.createElement 中调用 header.render 方法渲染组件的内容
const headerJsxObject = header.render();

//ReactDOM 用渲染后的 JavaScript 对象来来构建真正的DOM元素
const headerDOM = createDOMFromObject(headerJsxObject)
//ReactDOM把DOM元素塞到页面上
document.getElementById('root').appendChild(headerDOM);
```

上面过程其实很简单，看代码就能理解。

我们把 *React.js 将组件渲染，并且构造 DOM 元素然后塞入页面的过程称为组件的挂载*（这个定义请好好记住）。其实 React.js 内部对待每个组件都有这么一个过程，也就是初始化组件 -> 挂载到页面上的过程。所以你可以理解一个组件的方法调用是这么一个过程：

```
-> constructor()
-> render()
// 然后构造 DOM 元素插入页面
```

这当然是很好理解的。React.js 为了让我们能够更好的掌控组件的挂载过程，往上面插入了两个方法：

```
-> constructor()
-> componentWillMount()
-> render()
// 然后构造 DOM 元素插入页面
-> componentDidMount()
```

`componentWillMount` 和 `componentDidMount` 都是可以像 `render` 方法一样自定义在组件的内部。挂载的时候，React.js 会在组件的 `render` 之前调用 `componentWillMount`，在 DOM 元素塞入页面以后调用 `componentDidMount`。

我们给 `Header` 组件加上这两个方法，并且打一些 Log：

```
class Header extends Component {
  constructor () {
    super()
    console.log('construct')
  }

  componentWillMount () {
    console.log('component will mount')
  }

  componentDidMount () {
    console.log('component did mount')
  }

  render () {
    console.log('render')
    return (
      <div>
        <h1 className='title'>React 小书</h1>
      </div>
    )
  }
}
```

在控制台你可以看到依次输出：

```
construct
component will mount
render
component did mount
```

可以看到，React.js 确实按照我们上面所说的那样调用了定义的两个方法 `componentWillMount` 和 `componentDidMount`。

一个组件可以插入页面，当然也可以从页面中删除。

```
-> constructor()
-> componentWillMount()
-> render()
// 然后构造 DOM 元素插入页面
-> componentDidMount()
// ...
// 从页面中删除
```

React.js 也控制了这个组件的删除过程。在组件删除之前 React.js 会调用组件定义的 `componentWillUnmount`：

```
-> constructor()
-> componentWillMount()
-> render()
// 然后构造 DOM 元素插入页面
-> componentDidMount()
// ...
// 即将从页面中删除
-> componentWillUnmount()
// 从页面中删除
```

看看什么情况下会把组件从页面中删除，继续使用上面例子的代码，我们再定义一个 `Index` 组件：

```
class Index extends Component {
  constructor() {
    super()
    this.state = {
      isShowHeader: true
    }
  }

  handleShowOrHide () {
    this.setState({
      isShowHeader: !this.state.isShowHeader
    })
  }

  render () {
    return (
      <div>
        {this.state.isShowHeader ? <Header /> : null}
        <button onClick={this.handleShowOrHide.bind(this)}>
          显示或者隐藏标题
        </button>
      </div>
    )
  }
}

ReactDOM.render(
  <Index />,
  document.getElementById('root')
)
```

`Index` 组件使用了 `Header` 组件，并且有一个按钮，可以控制 `Header` 的显示或者隐藏。下面这行代码：

```
...a
{this.state.isShowHeader ? <Header /> : null}
...
```

相当于 `state.isShowHeader` 为 `true` 的时候把 `Header` 插入页面，`false` 的时候把 `Header` 从页面上删除。这时候我们给 `Header` 添加 `componentWillUnmount` 方法：

```
...
  componentWillUnmount() {
    console.log('component will unmount')
  }
...
```

这时候点击页面上的按钮，你会看到页面的标题隐藏了，并且控制台打印出来下图的最后一行，说明 `componentWillUnmount` 确实被 React.js 所调用了.你可以多次点击按钮，随着按钮的显示和隐藏，上面的内容会按顺序重复地打印出来，可以体会一下这几个方法的调用过程和顺序。 

### 总结

React.js 将组件渲染，并且构造 DOM 元素然后塞入页面的过程称为组件的挂载。这一节我们学习了 React.js 控制组件在页面上挂载和删除过程里面几个方法：

- `componentWillMount`：组件挂载开始之前，也就是在组件调用 `render` 方法之前调用。
- `componentDidMount`：组件挂载完成以后，也就是 DOM 元素已经插入页面后调用。
- `componentWillUnmount`：组件对应的 DOM 元素从页面中删除之前调用。

但这一节并没有讲这几个方法到底在实际项目当中有什么作用，下一节我们通过例子来讲解一下这几个方法的用途。

------

## 挂载阶段的组件生命周期（二）

这一节我们来讨论一下对于一个组件来说，`constructor`、`componentWillMount`、`componentDidMount`、`componentWillUnmount` 这几个方法在一个组件的出生到死亡的过程里面起了什么样的作用。

一般来说，所有关于组件自身的状态的初始化工作都会放在 `constructor` 里面去做。你会发现本书所有组件的 `state` 的初始化工作都是放在 `constructor` 里面的。假设我们现在在做一个时钟应用：

我们会在 `constructor` 里面初始化 `state.date`，当然现在页面还是静态的，等下一会让时间动起来。 

```
class Clock extends Component {
    constructor() {
        super();
        this.state = {
            date :new Date()
        }
    }
    render() {
        return(
        	<div>
        		<h1>
        			<p>现在的时间是</p>
        			{this.state.date.toLocaleTimeString()}
        		</h1>
        	</div>
        )
    }
}
```

一些组件启动的动作，包括像 Ajax 数据的拉取操作、一些定时器的启动等，就可以放在 `componentWillMount` 里面进行，例如 Ajax： 

```
...
  componentWillMount () {
    ajax.get('http://json-api.com/user', (userData) => {
      this.setState({ userData })
    })
  }
...
```

当然在我们这个例子里面是定时器的启动，我们给 `Clock` 启动定时器：

```
class Clock extends Component {
  constructor () {
    super()
    this.state = {
      date: new Date()
    }
  }
  componentWillMount () {
      this.timer = setInterval(() =>{
          this.setState({data: new Date()})
      },1000);
  }
  ...
}
```

我们在 `componentWillMount` 中用 `setInterval` 启动了一个定时器：每隔 1 秒更新中的 `state.date`，这样页面就可以动起来了。我们用一个 `Index` 把它用起来，并且插入页面：

```
class Index extends Component {
  render () {
    return (
      <div>
        <Clock />
      </div>
    )
  }
}

ReactDOM.render(
  <Index />,
  document.getElementById('root')
)
```

像上一节那样，我们修改这个 `Index` 让这个时钟可以隐藏或者显示：

```
class Index extends Component {
  constructor () {
    super()
    this.state = { isShowClock: true }
  }

  handleShowOrHide () {
    this.setState({
      isShowClock: !this.state.isShowClock
    })
  }

  render () {
    return (
      <div>
        {this.state.isShowClock ? <Clock /> : null }
        <button onClick={this.handleShowOrHide.bind(this)}>
          显示或隐藏时钟
        </button>
      </div>
    )
  }
}
```

现在页面上有个按钮可以显示或者隐藏时钟。你试一下显示或者隐藏时钟，虽然页面上看起来功能都正常，在控制台你会发现报错了.

这是因为，*当时钟隐藏的时候，我们并没有清除定时器*。时钟隐藏的时候，定时器的回调函数还在不停地尝试 `setState`，由于 `setState` 只能在已经挂载或者正在挂载的组件上调用，所以 React.js 开始疯狂报错。

多次的隐藏和显示会让 React.js 重新构造和销毁 `Clock` 组件，每次构造都会重新构建一个定时器。而销毁组件的时候没有清除定时器，所以你看到报错会越来越多。而且因为 JavaScript 的闭包特性，这样会导致严重的内存泄漏。

这时候`componentWillUnmount` 就可以派上用场了，它的作用就是在组件销毁的时候，做这种清场的工作。例如清除该组件的定时器和其他的数据清理工作。我们给 `Clock`添加 `componentWillUnmount`，在组件销毁的时候清除该组件的定时器：

```
...
  componentWillUnmount () {
    clearInterval(this.timer)
  }
...
```

这时候就没有错误了。

### 总结

我们一般会把组件的 `state` 的初始化工作放在 `constructor` 里面去做；在 `componentWillMount` 进行组件的启动工作，例如 Ajax 数据拉取、定时器的启动；组件从页面上销毁的时候，有时候需要一些数据的清理，例如定时器的清理，就会放在 `componentWillUnmount` 里面去做。

说一下本节没有提到的 `componentDidMount` 。一般来说，有些组件的启动工作是依赖 DOM 的，例如动画的启动，而 `componentWillMount` 的时候组件还没挂载完成，所以没法进行这些启动工作，这时候就可以把这些操作放在 `componentDidMount` 当中。`componentDidMount` 的具体使用我们会在接下来的章节当中结合 DOM 来讲。

------



## 更新阶段的组件生命周期

从之前的章节我们了解到，组件的挂载指的是将组件渲染并且构造 DOM 元素然后插入页面的过程。*这是一个从无到有的过程*，React.js 提供一些生命周期函数可以给我们在这个过程中做一些操作。

除了挂载阶段，还有一种“更新阶段”。说白了就是 `setState` 导致 React.js 重新渲染组件并且把组件的变化应用到 DOM 元素上的过程，*这是一个组件的变化过程*。而 React.js 也提供了一系列的生命周期函数可以让我们在这个组件更新的过程执行一些操作。

这些生命周期在深入项目开发阶段是非常重要的。而要完全理解更新阶段的组件生命周期是一个需要经验和知识积累的过程，你需要对 Virtual-DOM 策略有比较深入理解才能完全掌握，但这超出了本书的目的。*本书的目的是为了让大家快速掌握 React.js 核心的概念，快速上手项目进行实战*。所以对于组件更新阶段的组件生命周期，我们简单提及并且提供一些资料给大家。

这里为了知识的完整，补充关于更新阶段的组件生命周期：

1. `shouldComponentUpdate(nextProps, nextState)`：你可以通过这个方法控制组件是否重新渲染。如果返回 `false` 组件就不会重新渲染。这个生命周期在 React.js 性能优化上非常有用。
2. `componentWillReceiveProps(nextProps)`：组件从父组件接收到新的 `props` 之前调用。
3. `componentWillUpdate()`：组件开始重新渲染之前调用。
4. `componentDidUpdate()`：组件重新渲染并且把更改变更到真实的 DOM 以后调用。

大家对这更新阶段的生命周期比较感兴趣的话可以查看[官网文档](https://facebook.github.io/react/docs/react-component.html)。

*但这里建议大家可以先简单了解 React.js 组件是有更新阶段的，并且有这么几个更新阶段的生命周期即可*。然后在深入项目实战的时候逐渐地掌握理解他们，现在并不需要对他们放过多的精力。

有朋友对 Virtual-DOM 策略比较感兴趣的话，可以参考这篇博客：[深度剖析：如何实现一个 Virtual DOM 算法 ](https://github.com/livoras/blog/issues/13)。对深入理解 React.js 核心算法有一定帮助。

------

## ref 和 React.js 中的 DOM 操作

在 React.js 当中你基本不需要和 DOM 直接打交道。React.js 提供了一系列的 `on*`方法帮助我们进行事件监听，所以 React.js 当中不需要直接调用 `addEventListener`的 DOM API；以前我们通过手动 DOM 操作进行页面更新（例如借助 jQuery），而在 React.js 当中可以直接通过 `setState` 的方式重新渲染组件，渲染的时候可以把新的 `props` 传递给子组件，从而达到页面更新的效果。

React.js 这种重新渲染的机制帮助我们免除了绝大部分的 DOM 更新操作，也让类似于 jQuery 这种以封装 DOM 操作为主的第三方的库从我们的开发工具链中删除。

但是 React.js 并不能完全满足所有 DOM 操作需求，有些时候我们还是需要和 DOM 打交道。比如说你想进入页面以后自动 focus 到某个输入框，你需要调用 `input.focus()` 的 DOM API，比如说你想动态获取某个 DOM 元素的尺寸来做后续的动画，等等。

React.js 当中提供了 `ref` 属性来帮助我们获取已经挂载的元素的 DOM 节点，你可以给某个 JSX 元素加上 `ref`属性：

```
class AutoFocusInput extends Component {
    componentDidMount () {
        this.input.focus();
    }
    render() {
        return(
        	<input ref = {(input) => this.input = input} />
        )
    }
}
ReactDOM.render(
  <AutoFocusInput />,
  document.getElementById('root')
)
```

可以看到我们给 `input` 元素加了一个 `ref` 属性，这个属性值是一个函数。当 `input` 元素在页面上挂载完成以后，React.js 就会调用这个函数，并且把这个挂载以后的 DOM 节点传给这个函数。在函数中我们把这个 DOM 元素设置为组件实例的一个属性，这样以后我们就可以通过 `this.input` 获取到这个 DOM 元素。

然后我们就可以在 `componentDidMount` 中使用这个 DOM 元素，并且调用 `this.input.focus()` 的 DOM API。整体就达到了页面加载完成就自动 focus 到输入框的功能（大家可以注意到我们用上了 `componentDidMount` 这个组件生命周期）。

我们可以给任意代表 HTML 元素标签加上 `ref` 从而获取到它 DOM 元素然后调用 DOM API。但是记住一个原则：*能不用 ref 就不用*。特别是要避免用 `ref` 来做 React.js 本来就可以帮助你做到的页面自动更新的操作和事件监听。多余的 DOM 操作其实是代码里面的“噪音”，不利于我们理解和维护。

顺带一提的是，其实可以给组件标签也加上 `ref` ，例如：

```
<Clock ref={(clock) => this.clock = clock} />
```

这样你获取到的是这个 `Clock` 组件在 React.js 内部初始化的实例。但这并不是什么常用的做法，而且也并不建议这么做，所以这里就简单提及，有兴趣的朋友可以自己学习探索。

------

## props.children 和容器类组件

有一类组件，充当了容器的作用，它定义了一种外层结构形式，然后你可以往里面塞任意的内容。这种结构在实际当中非常常见，例如这种带卡片组件： 

![](https://huzidaha.github.io/static/assets/img/posts/45A7AD7E-CC88-4957-B1EF-09DFE7755590.png)

组件本身是一个不带任何内容的方形的容器，我可以在用这个组件的时候给它传入任意内容： 

![](https://huzidaha.github.io/static/assets/img/posts/6BD73C14-60FE-44BA-A93C-B637BD07DE59.png)

基于我们目前的知识储备，可以迅速写出这样的代码：

```
class Card extends Component {
  render () {
    return (
      <div className='card'>
        <div className='card-content'>
          {this.props.content}
        </div>
      </div>
    )
  }
}

ReactDOM.render(
  <Card content={
    <div>
      <h2>React.js 小书</h2>
       <div>开源、免费、专业、简单</div>
      订阅：<input />
    </div>
  } />,
  document.getElementById('root')
)
```

我们通过给 `Card` 组件传入一个 `content` 属性，这个属性可以传入任意的 JSX 结构。然后在 `Card` 内部会通过 `{this.props.content}` 把内容渲染到页面上。

这样明显太丑了，如果 `Card` 除了 `content` 以外还能传入其他属性的话，那么这些 JSX 和其他属性就会混在一起。很不好维护，如果能像下面的代码那样使用 `Card` 那想必也是极好的：

```
ReactDOM.render(
  <Card>
    <h2>React.js 小书</h2>
    <div>开源、免费、专业、简单</div>
    订阅：<input />
  </Card>,
  document.getElementById('root')
)
```

如果组件标签也能像普通的 HTML 标签那样编写内嵌的结构，那么就方便很多了。实际上，React.js 默认就支持这种写法，所有嵌套在组件中的 JSX 结构都可以在组件内部通过 `props.children` 获取到：

```
class Card extends Component {
  render () {
    return (
      <div className='card'>
        <div className='card-content'>
          {this.props.children}
        </div>
      </div>
    )
  }
}
```

把 `props.children` 打印出来，你可以看到它其实是个数组 

这种嵌套的内容成为了 `props.children` 数组的机制使得我们编写组件变得非常的灵活，我们甚至可以在组件内部把数组中的 JSX 元素安置在不同的地方：

```
class Layout extends Component {
  render () {
    return (
      <div className='two-cols-layout'>
        <div className='sidebar'>
          {this.props.children[0]}
        </div>
        <div className='main'>
          {this.props.children[1]}
        </div>
      </div>
    )
  }
}
```

这是一个两列布局组件，嵌套的 JSX 的第一个结构会成为侧边栏，第二个结构会成为内容栏，其余的结构都会被忽略。这样通过这个布局组件，就可以在各个地方高度复用我们的布局。

### 总结

使用自定义组件的时候，可以在其中嵌套 JSX 结构。嵌套的结构在组件内部都可以通过 `props.children` 获取到，这种组件编写方式在编写容器类型的组件当中非常有用。而在实际的 React.js 项目当中，我们几乎每天都需要用这种方式来编写组件。

------

## dangerouslySetHTML 和 style 属性

这一节我们来补充两个之前没有提到的属性，但是在 React.js 组件开发中也非常常用，但是它们也很简单。

### dangerouslySetHTML

出于安全考虑的原因（XSS 攻击），在 React.js 当中所有的表达式插入的内容都会被自动转义，就相当于 jQuery 里面的 `text(…)` 函数一样，任何的 HTML 格式都会被转义掉：

```
class Editor extends Component {
  constructor() {
    super()
    this.state = {
      content: '<h1>React.js 小书</h1>'
    }
  }

  render () {
    return (
      <div className='editor-wrapper'>
        {this.state.content}
      </div>
    )
  }
}
```

假设上面是一个富文本编辑器组件，富文本编辑器的内容是动态的 HTML 内容，用 `this.state.content` 来保存。我希望在编辑器内部显示这个动态 HTML 结构，但是因为 React.js 的转义特性，页面上会显示：

![](https://huzidaha.github.io/static/assets/img/posts/A67A4660-604A-4F42-A743-A059A96344C8.png)

表达式插入并不会把一个 `<h1>` 渲染到页面，而是把它的文本形式渲染了。那要怎么才能做到设置动态 HTML 结构的效果呢？React.js 提供了一个属性 `dangerouslySetInnerHTML`，可以让我们设置动态设置元素的 innerHTML：

```
...
  render () {
    return (
      <div
        className='editor-wrapper'
        dangerouslySetInnerHTML={{__html: this.state.content}} />
    )
  }
...
```

需要给 `dangerouslySetInnerHTML` 传入一个对象，这个对象的 `__html` 属性值就相当于元素的 `innerHTML`，这样我们就可以动态渲染元素的 `innerHTML` 结构了。

有写朋友会觉得很奇怪，为什么要把一件这么简单的事情搞得这么复杂，名字又长，还要传入一个奇怪的对象。那是因为设置 `innerHTML` 可能会导致跨站脚本攻击（XSS），所以 React.js 团队认为把事情搞复杂可以防止（警示）大家滥用这个属性。这个属性不必要的情况就不要使用。

### style

React.js 中的元素的 `style` 属性的用法和 DOM 里面的 `style` 不大一样，普通的 HTML 中的：

```
<h1 style='font-size: 12px; color: red;'>React.js 小书</h1>
```

在 React.js 中你需要把 CSS 属性变成一个对象再传给元素：

```
<h1 style={{fontSize: '12px', color: 'red'}}>React.js 小书</h1>
```

`style` 接受一个对象，这个对象里面是这个元素的 CSS 属性键值对，原来 CSS 属性中带 `-` 的元素都必须要去掉 `-` 换成驼峰命名，如 `font-size` 换成 `fontSize`，`text-align` 换成 `textAlign`。

用对象作为 `style` 方便我们动态设置元素的样式。我们可以用 `props` 或者 `state`中的数据生成样式对象再传给元素，然后用 `setState` 就可以修改样式，非常灵活：

```
<h1 style={{fontSize: '12px', color: this.state.color}}>React.js 小书</h1>
```

只要简单地 `setState({color: 'blue'})` 就可以修改元素的颜色成蓝色。

------

## PropTypes 和组件参数验证

我们来了到了一个非常尴尬的章节，很多初学的朋友可能对这一章的知识点不屑一顾，觉得用不用对程序功能也没什么影响。但其实这一章节的知识在你构建多人协作、大型的应用程序的时候也是非常重要的，不可忽视。

都说 JavaScript 是一门灵活的语言 —— 这就是像是说“你是个好人”一样，凡事都有背后没有说出来的话。JavaScript 的灵活性体现在弱类型、高阶函数等语言特性上。而语言的弱类型一般来说确实让我们写代码很爽，但是也很容易出 bug。

变量没有固定类型可以随意赋值，在我们构建大型应用程序的时候并不是什么好的事情。你写下了 `let a = {}` ，如果这是个共享的状态并且在某个地方把 `a = 3`，那么 `a.xxx` 就会让程序崩溃了。而这种非常隐晦但是低级的错误在强类型的语言例如 C/C++、Java 中是不可能发生的，这些代码连编译都不可能通过，也别妄图运行。

大型应用程序的构建其实更适合用强类型的语言来构建，它有更多的规则，可以帮助我们在编写代码阶段、编译阶段规避掉很多问题，让我们的应用程序更加的安全。JavaScript 早就脱离了玩具语言的领域并且投入到大型的应用程序的生产活动中，因为它的弱类型，常常意味着不是很安全。所以近年来出现了类似 TypeScript 和 Flow 等技术，来弥补 JavaScript 这方面的缺陷。

React.js 的组件其实是为了构建大型应用程序而生。但是因为 JavaScript 这样的特性，你在编写了一个组件以后，根本不知道别人会怎么使用你的组件，往里传什么乱七八糟的参数，例如评论组件：

```
class Comment extends Component {
  const { comment } = this.props
  render () {
    return (
      <div className='comment'>
        <div className='comment-user'>
          <span>{comment.username} </span>：
        </div>
        <p>{comment.content}</p>
      </div>
    )
  }
}
```

但是别人往里面传一个数字你拿他一点办法都没有：

```
<Comment comment={1} />
```

JavaScript 在这种情况下是不会报任何错误的，但是页面就是显示不正常，然后我们踏上了漫漫 debug 的路程。这里的例子还是过于简单，容易 debug，但是对于比较复杂的成因和情况是比较难处理的。

于是 React.js 就提供了一种机制，让你可以*给组件的配置参数加上类型验证*，就用上述的评论组件例子，你可以配置 `Comment` 只能接受对象类型的 `comment` 参数，你传个数字进来组件就强制报错。我们这里先安装一个 React 提供的第三方库 `prop-types`：

```
npm install --save prop-types
```

它可以帮助我们验证 `props` 的参数类型，例如：

```
import React, { Component } from 'react'
import PropTypes from 'prop-types'

class Comment extends Component {
  static propTypes = {
    comment: PropTypes.object
  }

  render () {
    const { comment } = this.props
    return (
      <div className='comment'>
        <div className='comment-user'>
          <span>{comment.username} </span>：
        </div>
        <p>{comment.content}</p>
      </div>
    )
  }
}
```

注意我们在文件头部引入了 `PropTypes`，并且给 `Comment` 组件类添加了类属性 `propTypes`，里面的内容的意思就是你传入的 `comment` 类型必须为 `object`（对象）。

这时候如果再往里面传入数字，浏览器就会报错.

出错信息明确告诉我们：你给 `Comment` 组件传了一个数字类型的 `comment`，而它应该是 `object`。你就清晰知道问题出在哪里了。

虽然 `propTypes` 帮我们指定了参数类型，但是并没有说这个参数一定要传入，事实上，这些参数默认都是可选的。可选参数我们可以通过配置 `defaultProps`，让它在不传入的时候有默认值。但是我们这里并没有配置 `defaultProps`，所以如果直接用 `<Comment />` 而不传入任何参数的话，`comment` 就会是 `undefined`，`comment.username` 会导致程序报错

这个出错信息并不够友好。我们可以通过 `isRequired` 关键字来强制组件某个参数必须传入：

```
...
static propTypes = {
  comment: PropTypes.object.isRequired
}
...
```

那么会获得一个更加友好的出错信息，查错会更方便

React.js 提供的 `PropTypes` 提供了一系列的数据类型可以用来配置组件的参数：

```
PropTypes.array
PropTypes.bool
PropTypes.func
PropTypes.number
PropTypes.object
PropTypes.string
PropTypes.node
PropTypes.element
...
```

更多类型及其用法可以参看官方文档： [Typechecking With PropTypes - React](https://facebook.github.io/react/docs/typechecking-with-proptypes.html)。

组件参数验证在构建大型的组件库的时候相当有用，可以帮助我们迅速定位这种类型错误，让我们组件开发更加规范。另外也起到了一个说明文档的作用，如果大家都约定都写 `propTypes` ，那你在使用别人写的组件的时候，只要看到组件的 `propTypes`就清晰地知道这个组件到底能够接受什么参数，什么参数是可选的，什么参数是必选的。

### 总结

通过 `PropTypes` 给组件的参数做类型限制，可以在帮助我们迅速定位错误，这在构建大型应用程序的时候特别有用；另外，给组件加上 `propTypes`，也让组件的开发、使用更加规范清晰。

这里建议大家写组件的时候尽量都写 `propTypes`，有时候有点麻烦，但是是值得的。

------