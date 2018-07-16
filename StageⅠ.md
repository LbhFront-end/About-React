# 第一阶段

1. [React.js简介](https://github.com/LbhFront-end/About-React/blob/master/StageⅠ.md#react.js-简介)
<a href="#1">React.js简介</a>
2. [前端组件化（一）：从一个简单的例子讲起](https://github.com/LbhFront-end/About-React/blob/master/StageⅠ.md#前端组件化（一）：从一个简单的例子讲起)
3. [前端组件化（二）：优化DOM操作](https://github.com/LbhFront-end/About-React/blob/master/StageⅠ.md#前端组件化（二）：优化DOM操作)
4. [前端组件化（三）：抽象出公共组件类](https://github.com/LbhFront-end/About-React/blob/master/StageⅠ.md#前端组件化（三）：抽象出公共组件类)
5. [React.js基本环境安装](https://github.com/LbhFront-end/About-React/blob/master/StageⅠ.md#react.js基本环境安装)
6. [使用JSX描述UI信息](https://github.com/LbhFront-end/About-React/blob/master/StageⅠ.md#使用jsx描述UI信息)
7. [组件的render方法](https://github.com/LbhFront-end/About-React/blob/master/StageⅠ.md#组件的render方法)
8. [组件的组合、嵌套和组件树](https://github.com/LbhFront-end/About-React/blob/master/StageⅠ.md#组件的组合、嵌套和组件树)
9. [事件监听](https://github.com/LbhFront-end/About-React/blob/master/StageⅠ.md#事件监听)
10. [组件的state和setState](https://github.com/LbhFront-end/About-React/blob/master/StageⅠ.md#组件的state和setstate)
11. [配置组件的props](https://github.com/LbhFront-end/About-React/blob/master/StageⅠ.md#配置组件的props)
12. [stateVSprops](https://github.com/LbhFront-end/About-React/blob/master/StageⅠ.md#state-vs-props)
13. [渲染列表数据](https://github.com/LbhFront-end/About-React/blob/master/StageⅠ.md#渲染列表数据)

<a name = "1">
## React.js 简介

React.js 是一个帮助你构建页面 UI 的库。如果你熟悉 MVC 概念的话，那么 React 的组件就相当于 MVC 里面的 View。如果你不熟悉也没关系，你可以简单地理解为，React.js 将帮助我们将界面分成了各个独立的小块，每一个块就是组件，这些组件之间可以组合、嵌套，就成了我们的页面。

一个组件的显示形态和行为有可能是由某些数据决定的。而数据是可能发生改变的，这时候组件的显示形态就会发生相应的改变。而 React.js 也提供了一种非常高效的方式帮助我们做到了数据和组件显示形态之间的同步。

React.js 不是一个框架，它只是一个库。它只提供 UI （view）层面的解决方案。在实际的项目当中，它并不能解决我们所有的问题，需要结合其它的库，例如 Redux、React-router 等来协助提供完整的解决方法。

------

## 前端组件化（一）：从一个简单的例子讲起

### 一个简单的点赞功能

 假设现在我们需要实现一个点赞、取消点赞的功能。

HTML:

```
  <body>
    <div class='wrapper'>
      <button class='like-btn'>
        <span class='like-text'>点赞</span>
        <span>👍</span>
      </button>
    </div>
  </body>
```

JavaScript：

```
  const button = document.querySelector('.like-btn')
  const buttonText = button.querySelector('.like-text')
  let isLiked = false
  button.addEventListener('click', () => {
    isLiked = !isLiked
    if (isLiked) {
      buttonText.innerHTML = '取消'
    } else {
      buttonText.innerHTML = '点赞'
    }
  }, false)
```

### 结构复用

现在我们来重新编写这个点赞功能，让它具备一定的可复用。这次我们先写一个类，这个类有 `render` 方法，这个方法里面直接返回一个表示 HTML 结构的字符串：

```
  class LikeButton {
    render () {
      return `
        <button id='like-btn'>
          <span class='like-text'>赞</span>
          <span>👍</span>
        </button>
      `
    }
  }
```

然后可以用这个类来构建不同的点赞功能的实例，然后把它们插到页面中。

```
  const wrapper = document.querySelector('.wrapper')
  const likeButton1 = new LikeButton()
  wrapper.innerHTML = likeButton1.render()

  const likeButton2 = new LikeButton()
  wrapper.innerHTML += likeButton2.render()
```

### 实现简单的组件化

你一定会发现，现在的按钮是死的，你点击它它根本不会有什么反应。因为根本没有往上面添加事件。但是问题来了，`LikeButton` 类里面是虽然说有一个 `button`，但是这玩意根本就是在字符串里面的。你怎么能往一个字符串里面添加事件呢？DOM 事件的 API 只有 DOM 结构才能用。

我们需要 DOM 结构，准确地来说：*我们需要这个点赞功能的 HTML 字符串表示的 DOM 结构*。假设我们现在有一个函数 `createDOMFromString` ，你往这个函数传入 HTML 字符串，但是它会把相应的 DOM 元素返回给你。这个问题就可以解决了。

```
// ::String => ::Document
const createDOMFromString = (domString) => {
  const div = document.createElement('div')
  div.innerHTML = domString
  return div
}
```

先不用管这个函数应该怎么实现，先知道它是干嘛的。拿来用就好，这时候用它来改写一下 `LikeButton` 类：

```
  class LikeButton {
    render () {
      this.el = createDOMFromString(`
        <button class='like-button'>
          <span class='like-text'>点赞</span>
          <span>👍</span>
        </button>
      `)
      this.el.addEventListener('click', () => console.log('click'), false)
      return this.el
    }
  }
```

现在 `render()` 返回的不是一个 html 字符串了，而是一个由这个 html 字符串所生成的 DOM。在返回 DOM 元素之前会先给这个 DOM 元素上添加事件再返回。

因为现在 `render` 返回的是 DOM 元素，所以不能用 `innerHTML` 暴力地插入 wrapper。而是要用 DOM API 插进去。

```
  const wrapper = document.querySelector('.wrapper')

  const likeButton1 = new LikeButton()
  wrapper.appendChild(likeButton1.render())

  const likeButton2 = new LikeButton()
  wrapper.appendChild(likeButton2.render())
```

现在你点击这两个按钮，每个按钮都会在控制台打印 `click`，说明事件绑定成功了。但是按钮上的文本还是没有发生改变，只要稍微改动一下 `LikeButton` 的代码就可以完成完整的功能：

```
  class LikeButton {
    constructor () {
      this.state = { isLiked: false }
    }

    changeLikeText () {
      const likeText = this.el.querySelector('.like-text')
      this.state.isLiked = !this.state.isLiked
      likeText.innerHTML = this.state.isLiked ? '取消' : '点赞'
    }

    render () {
      this.el = createDOMFromString(`
        <button class='like-button'>
          <span class='like-text'>点赞</span>
          <span>👍</span>
        </button>
      `)
      this.el.addEventListener('click', this.changeLikeText.bind(this), false)
      return this.el
    }
  }
```

这里的代码稍微长了一些，但是还是很好理解。只不过是在给 `LikeButton` 类添加了构造函数，这个构造函数会给每一个 `LikeButton` 的实例添加一个对象 `state`，`state` 里面保存了每个按钮自己是否点赞的状态。还改写了原来的事件绑定函数：原来只打印 `click`，现在点击的按钮的时候会调用 `changeLikeText` 方法，这个方法会根据 `this.state` 的状态改变点赞按钮的文本。

现在这个组件的可复用性已经很不错了，你的同事们只要实例化一下然后插入到 DOM 里面去就好了。

下一节我们继续优化这个例子，让它更加通用。

## 前端组件化（二）：优化DOM操作

一个组件的显示形态由多个状态决定的情况非常常见。代码中混杂着对 DOM 的操作其实是一种不好的实践，手动管理数据和 DOM 之间的关系会导致代码可维护性变差、容易出错。所以我们的例子这里还有优化的空间：如何尽量减少这种手动 DOM 操作？ 

### 状态改变 -> 构建新的 DOM 元素更新页面

这里要提出的一种解决方案：*一旦状态发生改变，就重新调用 render 方法，构建一个新的 DOM 元素*。这样做的好处是什么呢？好处就是你可以在 `render` 方法里面使用最新的 `this.state` 来构造不同 HTML 结构的字符串，并且通过这个字符串构造不同的 DOM 元素。页面就更新了！听起来有点绕，看看代码怎么写，修改原来的代码为：

```
class LikeButton{
    constructor(){
        this.state = {isLike : false}
    }
    setState(state){
        this.state = state
        this.el = this.render();
    }
    changLikeText(){
        this.setState({
            isLiked:!this.state.isLiked
        })
    }
    render(){
        this.el = createDOMFromString(`
        	<button class= 'like-button'>
        		<span class='like-btn'>${this.state.isLike ? '取消' : '点赞'}<span>
        		<span>👍<span>
        	</button>
        `)
        this.el.addEventListener('click', this.changeLikeText.bind(this), false)
      	return this.el
    }
    
}
```

其实只是改了几个小地方：

1. `render` 函数里面的 HTML 字符串会根据 `this.state` 不同而不同（这里是用了 ES6 的模版字符串，做这种事情很方便）。
2. 新增一个 `setState` 函数，这个函数接受一个对象作为参数；它会设置实例的 `state`，然后重新调用一下 `render` 方法。
3. 当用户点击按钮的时候， `changeLikeText` 会构建新的 `state` 对象，这个新的 `state` ，传入 `setState` 函数当中。

这样的结果就是，用户每次点击，`changeLikeText` 都会调用改变组件状态然后调用 `setState` ；`setState` 会调用 `render` ，`render` 方法会根据 `state` 的不同重新构建不同的 DOM 元素。

也就是说，*你只要调用 setState，组件就会重新渲染*。我们顺利地消除了手动的 DOM 操作。

### 重新插入新的 DOM 元素

上面的改进不会有什么效果，因为你仔细看一下就会发现，其实重新渲染的 DOM 元素并没有插入到页面当中。所以在这个组件外面，你需要知道这个组件发生了改变，并且把新的 DOM 元素更新到页面当中。

重新修改一下 `setState` 方法：

```
...
    setState (state) {
      const oldEl = this.el
      this.state = state
      this.el = this.render()
      if (this.onStateChange) this.onStateChange(oldEl, this.el)
    }
...
```

使用这个组件的时候：

```
const likeButton = new LikeButton()
wrapper.appendChild(likeButton.render()) // 第一次插入 DOM 元素
likeButton.onStateChange = (oldEl, newEl) => {
  wrapper.insertBefore(newEl, oldEl) // 插入新的元素
  wrapper.removeChild(oldEl) // 删除旧的元素
}
```

这里每次 `setState` 都会调用 `onStateChange` 方法，而这个方法是实例化以后时候被设置的，所以你可以自定义 `onStateChange` 的行为。*这里做的事是，每当 setState中构造完新的 DOM 元素以后，就会通过 onStateChange 告知外部插入新的 DOM 元素，然后删除旧的元素，页面就更新了*。这里已经做到了进一步的优化了：现在不需要再手动更新页面了。

非一般的暴力，因为每次 `setState` 都重新构造、新增、删除 DOM 元素，会导致浏览器进行大量的重排，严重影响性能。不过没有关系，这种暴力行为可以被一种叫 Virtual-DOM 的策略规避掉，但这不是本文所讨论的范围。

这个版本的点赞功能很不错，我可以继续往上面加功能，而且还不需要手动操作DOM。但是有一个不好的地方，如果我要重新另外做一个新组件，譬如说评论组件，那么里面的这些 `setState` 方法要重新写一遍，其实这些东西都可以抽出来，变成一个通用的模式。[下一节](http://react.huziketang.com/blog/lesson4)我们把这个通用模式抽离到一个类当中。

------

## 前端组件化（三）：抽象出公共组件类

为了让代码更灵活，可以写更多的组件，我们把这种模式抽象出来，放到一个 `Component` 类当中：

```
  class Component {
    setState (state) {
      const oldEl = this.el
      this.state = state
      this._renderDOM()
      if (this.onStateChange) this.onStateChange(oldEl, this.el)
    }

    _renderDOM () {
      this.el = createDOMFromString(this.render())
      if (this.onClick) {
        this.el.addEventListener('click', this.onClick.bind(this), false)
      }
      return this.el
    }
  }
```

这个是一个组件父类 `Component`，所有的组件都可以继承这个父类来构建。它定义的两个方法，一个是我们已经很熟悉的 `setState`；一个是私有方法 `_renderDOM`。`_renderDOM` 方法会调用 `this.render` 来构建 DOM 元素并且监听 `onClick` 事件。所以，组件子类继承的时候只需要实现一个返回 HTML 字符串的 `render` 方法就可以了。

还有一个额外的 `mount` 的方法，其实就是把组件的 DOM 元素插入页面，并且在 `setState` 的时候更新页面：

```
  const mount = (component, wrapper) => {
    wrapper.appendChild(component._renderDOM())
    component.onStateChange = (oldEl, newEl) => {
      wrapper.insertBefore(newEl, oldEl)
      wrapper.removeChild(oldEl)
    }
  }
```

这样的话我们重新写点赞组件就会变成：

```
  class LikeButton extends Component {
    constructor () {
      super()
      this.state = { isLiked: false }
    }

    onClick () {
      this.setState({
        isLiked: !this.state.isLiked
      })
    }

    render () {
      return `
        <button class='like-btn'>
          <span class='like-text'>${this.state.isLiked ? '取消' : '点赞'}</span>
          <span>👍</span>
        </button>
      `
    }
  }

  mount(new LikeButton(), wrapper)
```

这样还不够好。在实际开发当中，你可能需要给组件传入一些自定义的配置数据。例如说想配置一下点赞按钮的背景颜色，如果我给它传入一个参数，告诉它怎么设置自己的颜色。那么这个按钮的定制性就更强了。所以我们可以给组件类和它的子类都传入一个参数 `props`，作为组件的配置参数。修改 `Component` 的构造函数为：

```
...
    constructor (props = {}) {
      this.props = props
    }
...
```

继承的时候通过 `super(props)` 把 `props` 传给父类，这样就可以通过 `this.props`获取到配置参数：

```
  class LikeButton extends Component {
    constructor (props) {
      super(props)
      this.state = { isLiked: false }
    }

    onClick () {
      this.setState({
        isLiked: !this.state.isLiked
      })
    }

    render () {
      return `
        <button class='like-btn' style="background-color: ${this.props.bgColor}">
          <span class='like-text'>
            ${this.state.isLiked ? '取消' : '点赞'}
          </span>
          <span>👍</span>
        </button>
      `
    }
  }

  mount(new LikeButton({ bgColor: 'red' }), wrapper)
```

这里我们稍微修改了一下原有的 `LikeButton` 的 `render` 方法，让它可以根据传入的参数 `this.props.bgColor` 来生成不同的 `style` 属性。这样就可以自由配置组件的颜色了。

只要有了上面那个 `Component` 类和 `mount` 方法加起来不足40行代码就可以做到组件化。如果我们需要写另外一个组件，只需要像上面那样，简单地继承一下 `Component`类就好了：

```
  class RedBlueButton extends Component {
    constructor (props) {
      super(props)
      this.state = {
        color: 'red'
      }
    }

    onClick () {
      this.setState({
        color: 'blue'
      })
    }

    render () {
      return `
        <div style='color: ${this.state.color};'>${this.state.color}</div>
      `
    }
  }
```

简单好用，现在可以灵活地组件化页面了。

### 总结

我们用了很长的篇幅来讲一个简单的点赞的例子，并且在这个过程里面一直在优化编写的方式。最后抽离出来了一个类，可以帮助我们更好的做组件化。在这个过程里面我们学到了什么？

组件化可以帮助我们解决前端结构的复用性问题，整个页面可以由这样的不同的组件组合、嵌套构成。

一个组件有自己的显示形态（上面的 HTML 结构和内容）行为，组件的显示形态和行为可以由数据状态（state）和配置参数（props）共同决定。数据状态和配置参数的改变都会影响到这个组件的显示形态。

当数据变化的时候，组件的显示需要更新。所以如果组件化的模式能提供一种高效的方式自动化地帮助我们更新页面，那也就可以大大地降低我们代码的复杂度，带来更好的可维护性。

好了，课程结束了。你已经学会了怎么使用 React.js 了，因为我们已经写了一个——当然我是在开玩笑，但是上面这个 `Component` 类其实和 React 的 `Component` 使用方式很类似。掌握了这几节的课程，你基本就掌握了基础的 React.js 的概念。

接下来我们开始正式进入主题，开始正式介绍 React.js。你会发现，有了前面的铺垫，下面讲的内容理解起来会简单很多了。

------

## React.js基本环境安装

### 安装 React.js

React.js 单独使用基本上是不可能的事情。不要指望着类似于 jQuery 下载放到 `<head />` 标签就开始使用。使用 React.js 不管在开发阶段生产阶段都需要一堆工具和库辅助，编译阶段你需要借助 Babel；需要 Redux 等第三方的状态管理工具来组织代码；如果你要写单页面应用那么你需要 React-router。这就是所谓的“React.js全家桶”。

本课程不会教大家如何配置这些东西，因为这不是课程的重点，网上有很多的资料，大家可以去参考那些资料。我们这里会直接使用 React.js 官网所推荐使用的工具 `create-react-app` 工具。它可以帮助我们一键生成所需要的工程目录，并帮我们做好各种配置和依赖，也帮我们隐藏了这些配置的细节。也就是所谓的“开箱即用”。

工具地址：<https://github.com/facebookincubator/create-react-app>

在安装之前要确认你的机器上安装了 node.js 环境包括 npm。如果没有安装的同学可以到 node.js 的官网下载自己电脑的对应的安装包来安装好环境。 

安装好环境以后，只需要按照官网的指引安装 `create-react-app` 即可。

```
npm install -g create-react-app
```

这条命令会往我们的机器上安装一条叫 `create-react-app` 的命令，安装好以后就可以直接使用它来构建一个 react 的前端工程：

```
create-react-app hello-react
```

这条命令会帮我们构建一个叫 `hello-react` 的工程，并且会自动地帮助我们安装所需要的依赖，现在只需要安静地等待它安装完。

> 额外的小贴士：
>
> 如果有些同学安装过程比较慢，那是很有可能是因为 npm 下载的时候是从国外的源下载的缘故。所以可以把 npm 的源改成国内的 taobao 的源，这样会加速下载过程。在执行上面的命令之前可以先修改一下 npm 的源：
>
> `npm config set registry https://registry.npm.taobao.org`

下载完以后我们就可以启动工程了，进入工程目录然后通过 npm 启动工程：

```
cd hello-react
npm start
```

终端提示成功,并且会自动打开浏览器，就可以看到 React 的工程顺利运行的效果.

------

## 使用JSX描述UI信息

这一节我们通过一个简单的例子讲解 React.js 描述页面 UI 的方式。把 `src/index.js` 中的代码改成： 

```
import React, { Component } from 'react'
import ReactDOM from 'react-dom'
import './index.css'

class Header extends Component {
    render(){
        return () {
            <div>
            	<h1>哈哈哈哈</h1>
            </div>
        }
    }
}

ReactDOM.render(
	<Header />,
	document.getElementById('root');
);
```

我们在文件头部从 `react` 的包当中引入了 `React` 和 React.js 的组件父类 `Component`。记住，只要你要写 React.js 组件，那么就必须要引入这两个东西。

`ReactDOM` 可以帮助我们把 React 组件渲染到页面上去，没有其它的作用了。你可以发现它是从 `react-dom` 中引入的，而不是从 `react` 引入。有些朋友可能会疑惑，为什么不把这些东西都包含在 `react` 包当中呢？我们稍后会回答这个问题。

接下来的代码你看起来会比较熟悉，但又会有点陌生。你看其实它跟我们前几节里面讲的内容其实很类似，一个组件继承 `Component` 类，有一个 `render` 方法，并且把这个组件的 HTML 结构返回；这里 return 的东西就比较奇怪了，它并不是一个字符串，看起来像是纯 HTML 代码写在 JavaScript 代码里面。你也许会说，这不就有语法错误了么？这完全不是合法的 JavaScript 代码。这种看起来“在 JavaScript 写的标签的”语法叫 JSX。

### JSX 原理

为了让大家深刻理解 JSX 的含义。有必要简单介绍了一下 JSX 稍微底层的运作原理，这样大家可以更加深刻理解 JSX 到底是什么东西，为什么要有这种语法，它是经过怎么样的转化变成页面的元素的。

思考一个问题：如何用 JavaScript 对象来表现一个 DOM 元素的结构，举个例子：

```
<div class='box' id='content'>
  <div class='title'>Hello</div>
  <button>Click</button>
</div>
```

每个 DOM 元素的结构都可以用 JavaScript 的对象来表示。你会发现一个 DOM 元素包含的信息其实只有三个：标签名，属性，子元素。

所以其实上面这个 HTML 所有的信息我们都可以用合法的 JavaScript 对象来表示：

```
{
  tag: 'div',
  attrs: { className: 'box', id: 'content'},
  children: [
    {
      tag: 'div',
      arrts: { className: 'title' },
      children: ['Hello']
    },
    {
      tag: 'button',
      attrs: null,
      children: ['Click']
    }
  ]
}
```

你会发现，HTML 的信息和 JavaScript 所包含的结构和信息其实是一样的，我们可以用 JavaScript 对象来描述所有能用 HTML 表示的 UI 信息。但是用 JavaScript 写起来太长了，结构看起来又不清晰，用 HTML 的方式写起来就方便很多了。

于是 React.js 就把 JavaScript 的语法扩展了一下，让 JavaScript 语言能够支持这种直接在 JavaScript 代码里面编写类似 HTML 标签结构的语法，这样写起来就方便很多了。编译的过程会把类似 HTML 的 JSX 结构转换成 JavaScript 的对象结构。

上面的代码：

```
mport React, { Component } from 'react'
import ReactDOM from 'react-dom'
import './index.css'

class Header extends Component {
  render () {
    return (
      <div>
        <h1 className='title'>哈哈哈哈</h1>
      </div>
    )
  }
}

ReactDOM.render(
  <Header />,
  document.getElementById('root')
)
```

经过编译以后会变成：

```
import React, { Component } from 'react'
import ReactDOM from 'react-dom'
import './index.css'

class Header extends Component {
  render () {
    return (
	 React.createElement(
	 	"div",
	 	null,
	 	React.createElement(
	 		"h1",
	 		{ className : 'title'},
	 		"哈哈哈哈"
	 	)
	 )
    )
  }
}

ReactDOM.render(
  React.createElement(Header, null), 
  document.getElementById('root')
);
```

`React.createElement` 会构建一个 JavaScript 对象来描述你 HTML 结构的信息，包括标签名、属性、还有子元素等。这样的代码就是合法的 JavaScript 代码了。所以使用 React 和 JSX 的时候一定要经过编译的过程。

这里再重复一遍：*所谓的 JSX 其实就是 JavaScript 对象*。每当在 JavaScript 代码中看到这种 JSX 结构的时候，脑子里面就可以自动做转化，这样对你理解 React.js 的组件写法很有好处。

有了这个表示 HTML 结构和信息的对象以后，就可以拿去构造真正的 DOM 元素，然后把这个 DOM 元素塞到页面上。这也是我们最后一段代码中 `ReactDOM.render` 所干的事情：

```
ReactDOM.render(
  <Header />,
  document.getElementById('root')
)
```

`ReactDOM.render` 功能就是把组件渲染并且构造 DOM 树，然后插入到页面上某个特定的元素上（在这里是 id 为 `root` 的 `div` 元素）。

所以可以总结一下从 JSX 到页面到底经过了什么样的过程： 

```
	Babel编译+React.js构造   					ReactDOM.render
JSX =======================> JavaScript对象构造 ===================> DOM元素 ===> 插入页面

```

有些同学可能会问，为什么不直接从 JSX 直接渲染构造 DOM 结构，而是要经过中间这么一层呢？

第一个原因是，当我们拿到一个表示 UI 的结构和信息的对象以后，不一定会把元素渲染到浏览器的普通页面上，我们有可能把这个结构渲染到 canvas 上，或者是手机 App 上。所以这也是为什么会要把 `react-dom` 单独抽离出来的原因，可以想象有一个叫 `react-canvas` 可以帮我们把 UI 渲染到 canvas 上，或者是有一个叫 `react-app` 可以帮我们把它转换成原生的 App（实际上这玩意叫 `ReactNative`）。

第二个原因是，有了这样一个对象。当数据变化，需要更新组件的时候，就可以用比较快的算法操作这个 JavaScript 对象，而不用直接操作页面上的 DOM，这样可以尽量少的减少浏览器重排，极大地优化性能。这个在以后的章节中我们会提到。

### 总结

要记住几个点：

1. JSX 是 JavaScript 语言的一种语法扩展，长得像 HTML，但并不是 HTML。
2. React.js 可以用 JSX 来描述你的组件长什么样的。
3. JSX 在编译的时候会变成相应的 JavaScript 对象描述。
4. `react-dom` 负责把这个用来描述 UI 信息的 JavaScript 对象变成 DOM 元素，并且渲染到页面上。

------

## 组件的render方法

React.js中一切都是组件，用的React.js写的React.js组件。我们再编写React.js组件的时候，一般都需要继承React.js的`Component。`一个组件类必须实现一个`render`方法，这个方法必须返回一个`JSX`元素。但是需要注意到的是，必须要用一个外层的`JSX`元素将所有的内容包裹起来。返回并列多个`JSX`元素时不合法的，下面是错误的做法

```
...
render () {
  return (
    <div>第一个</div>
    <div>第二个</div>
  )
}
...
```

必须要用一个外层元素把内容进行包裹：

```
...
render () {
  return (
    <div>
      <div>第一个</div>
      <div>第二个</div>
    </div>
  )
}
...
```

### 表达式插入

在 JSX 当中你可以插入 JavaScript 的表达式，表达式返回的结果会相应地渲染到页面上。表达式用 `{}` 包裹。例如： 

```
...
render () {
  const word = '哈哈哈哈哈'
  return (
    <div>
      <h1>学习 React 的感觉是 {word}</h1>
    </div>
  )
}
...
```

页面上就显示“学习 React 的感觉是哈哈哈哈哈”。你也可以把它改成 `{1 + 2}`，它就会显示 “学习 React 的感觉是 3”。你也可以把它写成一个函数表达式返回： 

```
render() {
    return(
    	<div>
    		<h1>学习 React 的感觉是 { function() { return '哈哈哈哈哈'}}</h1>
    	</div>
    )
}
```

简而言之，`{}` 内可以放任何 JavaScript 的代码，包括变量、表达式计算、函数执行等等。 `render` 会把这些代码返回的内容如实地渲染到页面上，非常的灵活。

表达式插入不仅仅可以用在标签内部，也可以用在标签的属性上，例如：

```
...
render () {
  const word = '哈哈哈哈哈';
  const className = 'header'
  return (
    <div className = {className}>
      <h1>学习 React 的感觉是 {word}</h1>
    </div>
  )
}
...
```

这样就可以为 `div` 标签添加一个叫 `header` 的类名。 

注意，直接使用 `class` 在 React.js 的元素上添加类名如 `<div class=“xxx”>` 这种方式是不合法的。因为 `class` 是 JavaScript 的关键字，所以 React.js 中定义了一种新的方式：`className` 来帮助我们给元素添加类名。

还有一个特例就是 `for` 属性，例如 `<label for='male'>Male</label>`，因为 `for` 也是 JavaScript 的关键字，所以在 JSX 用 `htmlFor` 替代，即 `<label htmlFor='male'>Male</label>`。而其他的 HTML 属性例如 `style` 、`data-*` 等就可以像普通的 HTML 属性那样直接添加上去。

### 条件返回

`{}` 上面说了，JSX 可以放置任何表达式内容。所以也可以放 JSX，实际上，我们可以在 `render` 函数内部根据不同条件返回不同的 JSX。例如：

```
...
render () {
  const isGoodWord = true
  return (
    <div>
      <h1>
        React 小书
        {isGoodWord
          ? <strong> is good</strong>
          : <span> is not good</span>
        }
      </h1>
    </div>
  )
}
...

```

上面的代码中定义了一个 `isGoodWord` 变量为 `true`，下面有个用 `{}` 包含的表达式，根据 `isGoodWord` 的不同返回不同的 JSX 内容。现在页面上是显示 `React 小书 is good`。如果你把 `isGoodWord` 改成 `false` 然后再看页面上就会显示 `React 小书 is not good`。

如果你在表达式插入里面返回 `null` ，那么 React.js 会什么都不显示，相当于忽略了该表达式插入。结合条件返回的话，我们就做到显示或者隐藏某些元素：

```
...
render () {
  const isGoodWord = true
  return (
    <div>
      <h1>
        React 小书
        {isGoodWord
          ? <strong> is good</strong>
          : null
        }
      </h1>
    </div>
  )
}
...
```

这样就相当于在 `isGoodWord` 为 `true` 的时候显示 `<strong>is good</strong>`，否则就隐藏。

条件返回 JSX 的方式在 React.js 中很常见，组件的呈现方式随着数据的变化而不一样，你可以利用 JSX 这种灵活的方式随时组合构建不同的页面结构。

如果这里有些同学觉得比较难理解的话，可以回想一下，其实 JSX 就是 JavaScript 里面的对象，转换一下角度，把上面的内容翻译成 JavaScript 对象的形式，上面的代码就很好理解了。

### JSX 元素变量

同样的，如果你能理解 JSX 元素就是 JavaScript 对象。那么你就可以联想到，JSX 元素其实可以像 JavaScript 对象那样自由地赋值给变量，或者作为函数参数传递、或者作为函数的返回值。

```
...
render () {
  const isGoodWord = true
  const goodWord = <strong> is good</strong>
  const badWord = <span> is not good</span>
  return (
    <div>
      <h1>
        React 小书
        {isGoodWord ? goodWord : badWord}
      </h1>
    </div>
  )
}
...
```

这里给把两个 JSX 元素赋值给了 `goodWord` 和 `badWord` 两个变量，然后把它们作为表达式插入的条件返回值。达到效果和上面的例子一样，随机返回不同的页面效果呈现。

再举一个例子：

```
...
renderGoodWord (goodWord, badWord) {
  const isGoodWord = true
  return isGoodWord ? goodWord : badWord
}

render () {
  return (
    <div>
      <h1>
        React 小书
        {this.renderGoodWord(
          <strong> is good</strong>,
          <span> is not good</span>
        )}
      </h1>
    </div>
  )
}
...
```

这里我们定义了一个 `renderGoodWord` 函数，这个函数接受两个 JSX 元素作为参数，并且随机返回其中一个。在 `render` 方法中，我们把上面例子的两个 JSX 元素传入 `renderGoodWord` 当中，通过表达式插入把该函数返回的 JSX 元素插入到页面上。

------

## 组件的组合、嵌套和组件树

继续拓展前面的例子，现在我们已经有了 `Header` 组件了。假设我们现在构建一个新的组件叫 `Title`，它专门负责显示标题。你可以在 `Header` 里面使用 `Title`组件： 

```
class Title extends Component {
  render () {
    return (
      <h1>React 小书</h1>
    )
  }
}

class Header extends Component {
  render () {
    return (
      <div>
        <Title />
      </div>
    )
  }
}
```

我们可以直接在 `Header` 标签里面直接使用 `Title` 标签。就像是一个普通的标签一样。React.js 会在 `<Title />` 所在的地方把 `Title` 组件的 `render` 方法表示的 JSX 内容渲染出来，也就是说 `<h1>React 小书</h1>` 会显示在相应的位置上。如果现在我们在 `Header` 里面使用三个 `<Title />` ，那么就会有三个 `<h1 />` 显示在页面上。

```
<div>
  <Title />
  <Title />
  <Title />
</div>
```

这样可复用性非常强，我们可以把组件的内容封装好，然后灵活在使用在任何组件内。另外这里要注意的是，*自定义的组件都必须要用大写字母开头，普通的 HTML 标签都用小写字母开头*。

现在让组件多起来。我们来构建额外的组件来构建页面，假设页面是由 `Header`、`Main` 、`Footer` 几个部分组成，由一个 `Index` 把它们组合起来。 

```
import React, { Component } from 'react';
import ReactDOM from 'react-dom';

class Title extends Component {
  render () {
    return (
      <h1>React 小书</h1>
    )
  }
}

class Header extends Component {
  render () {
    return (
    <div>
      <Title />
      <h2>This is Header</h2>
    </div>
    )
  }
}

class Main extends Component {
  render () {
    return (
    <div>
      <h2>This is main content</h2>
    </div>
    )
  }
}

class Footer extends Component {
  render () {
    return (
    <div>
      <h2>This is footer</h2>
    </div>
    )
  }
}

class Index extends Component {
  render () {
    return (
      <div>
        <Header />
        <Main />
        <Footer />
      </div>
    )
  }
}

ReactDOM.render(
  <Index />,
  document.getElementById('root')
)
```

组件可以和组件组合在一起，组件内部可以使用别的组件。就像普通的 HTML 标签一样使用就可以。这样的组合嵌套，最后构成一个所谓的组件树，就正如上面的例子那样，`Index` 用了 `Header`、`Main`、`Footer`，`Header` 又使用了 `Title` 。这样用这样的树状结构表示它们之间的关系： 

```
                                        ---------
                                        | Index |
                                        ---------
                                            |
                      ----------------------------------------------       
                      |                     |		              |
                  ---------             ---------              ---------
                  | Header|             |  Main |              | Footer|
                  ---------             ---------              ---------
                      |     
                  ---------             
                  | Title |            
                  ---------                         
```

这里的结构还是比较简单，因为我们的页面结构并不复杂。当页面结构复杂起来，有许多不同的组件嵌套组合的话，组件树会相当的复杂和庞大。理解组件树的概念对后面理解数据是如何在组件树内自上往下流动过程很重要。 

------

## 事件监听

在 React.js 里面监听事件是很容易的事情，你只需要给需要监听事件的元素加上属性类似于 `onClick`、`onKeyDown` 这样的属性，例如我们现在要给 `Title` 加上点击的事件监听： 

```
class Title extends Component {
  handleClickOnTitle () {
    console.log('Click on title.')
  }

  render () {
    return (
      <h1 onClick={this.handleClickOnTitle}>React 小书</h1>
    )
  }
}
```

只需要给 `h1` 标签加上 `onClick` 的事件，`onClick` 紧跟着是一个表达式插入，这个表达式返回一个 `Title` 自己的一个实例方法。当用户点击 `h1` 的时候，React.js 就会调用这个方法，所以你在控制台就可以看到 `Click on title.` 打印出来。

在 React.js 不需要手动调用浏览器原生的 `addEventListener` 进行事件监听。React.js 帮我们封装好了一系列的 `on*` 的属性，当你需要为某个元素监听某个事件的时候，只需要简单地给它加上 `on*` 就可以了。而且你不需要考虑不同浏览器兼容性的问题，React.js 都帮我们封装好这些细节了。

React.js 封装了不同类型的事件，这里就不一一列举，有兴趣的同学可以参考官网文档： [SyntheticEvent - React](https://facebook.github.io/react/docs/events.html#supported-events)，多尝试不同的事件。另外要注意的是，这些事件属性名都必须要用驼峰命名法。

没有经过特殊处理的话，*这些 on\* 的事件监听只能用在普通的 HTML 的标签上，而不能用在组件标签上*。也就是说，`<Header onClick={…} />` 这样的写法不会有什么效果的。这一点要注意，但是有办法可以做到这样的绑定，以后我们会提及。现在只要记住一点就可以了：这些 `on*` 的事件监听只能用在普通的 HTML 的标签上，而不能用在组件标签上。

### event 对象

```
class Title extends Component {
  handleClickOnTitle (e) {
    console.log(e.target.innerHTML)
  }

  render () {
    return (
      <h1 onClick={this.handleClickOnTitle}>React 小书</h1>
    )
  }
}
```

再看看控制台，每次点击的时候就会打印”React 小书“。

### 关于事件中的 this

一般在某个类的实例方法里面的 `this` 指的是这个实例本身。但是你在上面的 `handleClickOnTitle` 中把 `this` 打印出来，你会看到 `this` 是 `null` 或者 `undefined`。

```
...
  handleClickOnTitle (e) {
    console.log(this) // => null or undefined
  }
...
```

这是因为 React.js 调用你所传给它的方法的时候，并不是通过对象方法的方式调用（`this.handleClickOnTitle`），而是直接通过函数调用 （`handleClickOnTitle`），所以事件监听函数内并不能通过 `this` 获取到实例。

如果你想在事件函数当中使用当前的实例，你需要手动地将实例方法 `bind` 到当前实例上再传入给 React.js。

```
class Title extends Component {
  handleClickOnTitle (e) {
    console.log(this)
  }

  render () {
    return (
      <h1 onClick={this.handleClickOnTitle.bind(this)}>React 小书</h1>
    )
  }
}
```

`bind` 会把实例方法绑定到当前实例上，然后我们再把绑定后的函数传给 React.js 的 `onClick` 事件监听。这时候你再看看，点击 `h1` 的时候，就会把当前的实例打印出来

这种 `bind` 模式在 React.js 的事件监听当中非常常见，`bind` 不仅可以帮我们把事件监听方法中的 `this` 绑定到当前组件实例上；还可以帮助我们在在渲染列表元素的时候，把列表元素传入事件监听函数当中——这个将在以后的章节提及。

如果有些同学对 JavaScript 的 `this` 模式或者 `bind` 函数的使用方式不是特别了解到话，可能会对这部分内容会有些迷惑，可以补充对 JavaScript 的 [this](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this) 和 [bind](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)相关的知识再来回顾这部分内容。

### 总结

为 React 的组件添加事件监听是很简单的事情，你只需要使用 React.js 提供了一系列的 `on*` 方法即可。

React.js 会给每个事件监听传入一个 `event` 对象，这个对象提供的功能和浏览器提供的功能一致，而且它是兼容所有浏览器的。

React.js 的事件监听方法需要手动 `bind` 到当前实例，这种模式在 React.js 中非常常用。

------



## 组件的state和setState

### state

我们前面提到过，一个组件的显示形态是可以由它数据状态和配置参数决定的。一个组件可以拥有自己的状态，就像一个点赞按钮，可以有“已点赞”和“未点赞”状态，并且可以在这两种状态之间进行切换。React.js 的 `state` 就是用来存储这种可变化的状态的。

我们还是拿点赞按钮做例子，它具有已点赞和未点赞两种状态。那么就可以把这个状态存储在 state 中。修改 `src/index.js` 为： 

```
import React, { Component } from 'react'
import ReactDOM from 'react-dom'
import './index.css'

class LikeButton extends Component {
    constructor () {
        super();
        this.state = {isLiked:false}
    }
    
    handleClickOnLikeButton () {
        setState({ isLiked: !this.state.isLiked});
    }
    
    render () {
        return (
        	<button onClick = {this.handleClickOnLikeButton.bind(this)}>
        		{this.state.isLiked ? '取消': '点赞'} 👍
        	</button>
        )
    }
}
```

`isLiked` 存放在实例的 `state` 对象当中，这个对象在构造函数里面初始化。这个组件的 `render` 函数内，会根据组件的 `state` 的中的`isLiked`不同显示“取消”或“点赞”内容。并且给 `button` 加上了点击的事件监听。

最后构建一个 `Index` ，在它的 `render` 函数内使用 `LikeButton` 。然后把 `Index`渲染到页面上：

```
...
class Index extends Component {
  render () {
    return (
      <div>
        <LikeButton />
      </div>
    )
  }
}

ReactDOM.render(
  <Index />,
  document.getElementById('root')
)
```

### setState 接受对象参数

在 `handleClickOnLikeButton` 事件监听函数里面，大家可以留意到，我们调用了 `setState` 函数，每次点击都会更新 `isLiked` 属性为 `!isLiked`，这样就可以做到点赞和取消功能。

`setState` 方法由父类 `Component` 所提供。*当我们调用这个函数的时候，React.js 会更新组件的状态 state ，并且重新调用 render 方法，然后再把 render 方法所渲染的最新的内容显示到页面上*。

注意，当我们要改变组件的状态的时候，不能直接用 `this.state = xxx` 这种方式来修改，如果这样做 React.js 就没办法知道你修改了组件的状态，它也就没有办法更新页面。所以，一定要使用 React.js 提供的 `setState` 方法，*它接受一个对象或者函数作为参数*。

传入一个对象的时候，这个对象表示该组件的新状态。但你只需要传入需要更新的部分就可以了，而不需要传入整个对象。例如，假设现在我们有另外一个状态 `name` ：

------

```
...
  constructor (props) {
    super(props)
    this.state = {
      name: 'Tomy',
      isLiked: false
    }
  }

  handleClickOnLikeButton () {
    this.setState({
      isLiked: !this.state.isLiked
    })
  }
...
```

因为点击的时候我们并不需要修改 `name`，所以只需要传入 `isLiked` 就行了。Tomy 还是那个 Tomy，而 `isLiked` 已经不是那个 `isLiked` 了。

### setState 接受函数参数

这里还有要注意的是，当你调用 `setState` 的时候，*React.js 并不会马上修改 state*。而是把这个对象放到一个更新队列里面，稍后才会从队列当中把新的状态提取出来合并到 `state` 当中，然后再触发组件更新。这一点要好好注意。可以体会一下下面的代码：

```
...
  handleClickOnLikeButton () {
    console.log(this.state.isLiked)
    this.setState({
      isLiked: !this.state.isLiked
    })
    console.log(this.state.isLiked)
  }
...
```

你会发现两次打印的都是 `false`，即使我们中间已经 `setState` 过一次了。这并不是什么 bug，只是 React.js 的 `setState` 把你的传进来的状态缓存起来，稍后才会帮你更新到 `state` 上，所以你获取到的还是原来的 `isLiked`。

所以如果你想在 `setState` 之后使用新的 `state` 来做后续运算就做不到了，例如：

```
...
  handleClickOnLikeButton () {
    this.setState({ count: 0 }) // => this.state.count 还是 undefined
    this.setState({ count: this.state.count + 1}) // => undefined + 1 = NaN
    this.setState({ count: this.state.count + 2}) // => NaN + 2 = NaN
  }
...
```

上面的代码的运行结果并不能达到我们的预期，我们希望 `count` 运行结果是 `3` ，可是最后得到的是 `NaN`。但是这种后续操作依赖前一个 `setState` 的结果的情况并不罕见。

这里就自然地引出了 `setState` 的第二种使用方式，可以接受一个函数作为参数。React.js 会把上一个 `setState` 的结果传入这个函数，你就可以使用该结果进行运算、操作，然后返回一个对象作为更新 `state` 的对象：

```
...
  handleClickOnLikeButton () {
    this.setState((prevState) => {
      return { count: 0 }
    })
    this.setState((prevState) => {
      return { count: prevState.count + 1 } // 上一个 setState 的返回是 count 为 0，当前返回 1
    })
    this.setState((prevState) => {
      return { count: prevState.count + 2 } // 上一个 setState 的返回是 count 为 1，当前返回 3
    })
    // 最后的结果是 this.state.count 为 3
  }
...
```

这样就可以达到上述的*利用上一次 setState 结果进行运算*的效果。

### setState 合并

上面我们进行了三次 `setState`，但是实际上组件只会重新渲染一次，而不是三次；这是因为在 React.js 内部会把 JavaScript 事件循环中的消息队列的同一个消息中的 `setState` 都进行合并以后再重新渲染组件。

深层的原理并不需要过多纠结，你只需要记住的是：在使用 React.js 的时候，并不需要担心多次进行 `setState` 会带来性能问题。

------



## 配置组件的props

组件是相互独立、可复用的单元，一个组件可能在不同地方被用到。但是在不同的场景下对这个组件的需求可能会根据情况有所不同，例如一个点赞按钮组件，在我这里需要它显示的文本是“点赞”和“取消”，当别的同事拿过去用的时候，却需要它显示“赞”和“已赞”。如何让组件能适应不同场景下的需求，我们就要让组件具有一定的“可配置”性。

React.js 的 `props` 就可以帮助我们达到这个效果。每个组件都可以接受一个 `props`参数，它是一个对象，包含了所有你对这个组件的配置。就拿我们点赞按钮做例子：

```
class LikeButton  extends Component {
    constructor () {
        super();
        this.state = { isLiked : false }
    }
    
    handleClickOnLikeButton () {
        this.setState({ isLiked: !this.state.isLiked});
    }
    
    render() {
        const likedText = this.props.likedText || '取消'
        const unLikedText = this.props.unlikeText || '点赞'
        return (
        	<button onClick = {this.handleClickOnLikeButton.bind(this)}>
        	  { this.state.isLiked ? likeText : unlikeText} 👍
        	</button> 
        )
    }
}
```

从 `render` 函数可以看出来，组件内部是通过 `this.props` 的方式获取到组件的参数的，如果 `this.props` 里面有需要的属性我们就采用相应的属性，没有的话就用默认的属性。

那么怎么把 `props` 传进去呢？*在使用一个组件的时候，可以把参数放在标签的属性当中，所有的属性都会作为 props 对象的键值*：

```
class Index extends Component {
  render () {
    return (
      <div>
        <LikeButton likedText='已赞' unlikedText='赞' />
      </div>
    )
  }
}
```

就像你在用普通的 HTML 标签的属性一样，可以把参数放在表示组件的标签上，组件内部就可以通过 `this.props` 来访问到这些配置参数了。

前面的章节我们说过，JSX 的表达式插入可以在标签属性上使用。所以其实可以把任何类型的数据作为组件的参数，包括字符串、数字、对象、数组、甚至是函数等等。例如现在我们把一个对象传给点赞组件作为参数：

```
class Index extends Component {
  render () {
    return (
      <div>
        <LikeButton wordings={{likedText: '已赞', unlikedText: '赞'}} />
      </div>
    )
  }
}
```

 现在我们把 `likedText` 和 `unlikedText` 这两个参数封装到一个叫 `wordings` 的对象参数内，然后传入点赞组件中。大家看到 `{{likedText: '已赞', unlikedText: '赞'}}`这样的代码的时候，不要以为是什么新语法。之前讨论过，JSX 的 `{}` 内可以嵌入任何表达式，`{{}}` 就是在 `{}` 内部用对象字面量返回一个对象而已。

这时候，点赞按钮的内部就要用 `this.props.wordings` 来获取到到参数了：

```
class LikeButton extends Component {
  constructor () {
    super()
    this.state = { isLiked: false }
  }

  handleClickOnLikeButton () {
    this.setState({
      isLiked: !this.state.isLiked
    })
  }

  render () {
    const wordings = this.props.wordings || {
        likedText:'取消'，
        unlikeText: '点赞'
    }
    return (
      <button onClick={this.handleClickOnLikeButton.bind(this)}>
        {this.state.isLiked ? wordings.likedText : wordings.unlikedText} 👍
      </button>
    )
  }
}
```

甚至可以往组件内部传入函数作为参数：

甚至可以往组件内部传入函数作为参数：

```
class Index extends Component {
  render () {
    return (
      <div>
        <LikeButton
          wordings={{likedText: '已赞', unlikedText: '赞'}}
          onClick={() => console.log('Click on like button!')}/>
      </div>
    )
  }
}
```

这样可以通过 `this.props.onClick` 获取到这个传进去的函数，修改 `LikeButton `的 `handleClickOnLikeButton` 方法：

```
...
  handleClickOnLikeButton () {
    this.setState({
      isLiked: !this.state.isLiked
    })
    if (this.props.onClick) {
      this.props.onClick()
    }
  }
...
```

当每次点击按钮的时候，控制台会显示 `Click on like button!` 。但这个行为不是点赞组件自己实现的，而是我们传进去的。所以，一个组件的行为、显示形态都可以用 `props` 来控制，就可以达到很好的可配置性。

### 默认配置 defaultProps

上面的组件默认配置我们是通过 `||` 操作符来实现。这种需要默认配置的情况在 React.js 中非常常见，所以 React.js 也提供了一种方式 `defaultProps`，可以方便的做到默认配置。

```
class LikeButtons extends Component {
    static default props = {
        likeText ： '取消'，
        unLikeText : '点赞'
    }
    
    constructor() {
        super();
        this.state = {isLiked : false}
    }
    handleClickOnLikeButton() {
        this.setState({ isLiked : !this.state.isLiked});
    }
    render() {
        return (
        	<button onClick = {this.handleClickOnLikeButton.bind(this)}>
        		{this.state.isLiked ? this.props.likeText ： this.props.unLikeText} 👍
        	</button>
        )
    }
}
```

注意，我们给点赞组件加上了以下的代码：

```
  static defaultProps = {
    likedText: '取消',
    unlikedText: '点赞'
  }
```

`defaultProps` 作为点赞按钮组件的类属性，里面是对 `props` 中各个属性的默认配置。这样我们就不需要判断配置属性是否传进来了：如果没有传进来，会直接使用 `defaultProps` 中的默认属性。 所以可以看到，在 `render` 函数中，我们会直接使用 `this.props` 而不需要再做判断。 

### rops 不可变

`props` 一旦传入进来就不能改变。修改上面的例子中的 `handleClickOnLikeButton` ：

```
...
  handleClickOnLikeButton () {
    this.props.likedText = '取消'
    this.setState({
      isLiked: !this.state.isLiked
    })
  }
...
```

我们尝试在用户点击按钮的时候改变 `this.props.likedText` ，然后你会看到控制台报错了

你不能改变一个组件被渲染的时候传进来的 `props`。React.js 希望一个组件在输入确定的 `props` 的时候，能够输出确定的 UI 显示形态。如果 `props` 渲染过程中可以被修改，那么就会导致这个组件显示形态和行为变得不可预测，这样会可能会给组件使用者带来困惑。

但这并不意味着由 `props` 决定的显示形态不能被修改。组件的使用者可以*主动地通过重新渲染的方式*把新的 `props` 传入组件当中，这样这个组件中由 `props` 决定的显示形态也会得到相应的改变。

修改上面的例子的 `Index` 组件：

```
class Index extends Component {
  constructor () {
    super()
    this.state = {
      likedText: '已赞',
      unlikedText: '赞'
    }
  }

  handleClickOnChange () {
    this.setState({
      likedText: '取消',
      unlikedText: '点赞'
    })
  }

  render () {
    return (
      <div>
        <LikeButton
          likedText={this.state.likedText}
          unlikedText={this.state.unlikedText} />
        <div>
          <button onClick={this.handleClickOnChange.bind(this)}>
            修改 wordings
          </button>
        </div>
      </div>
    )
  }
}
```

在这里，我们把 `Index` 的 `state` 中的 `likedText` 和 `unlikedText` 传给 `LikeButton` 。`Index` 还有另外一个按钮，点击这个按钮会通过 `setState` 修改 `Index` 的 `state` 中的两个属性。

由于 `setState` 会导致 `Index` 重新渲染，所以 `LikedButton` 会接收到新的 `props`，并且重新渲染，于是它的显示形态也会得到更新。这就是通过重新渲染的方式来传入新的 `props` 从而达到修改 `LikedButton` 显示形态的效果。

### 总结

1. 为了使得组件的可定制性更强，在使用组件的时候，可以在标签上加属性来传入配置参数。
2. 组件可以在内部通过 `this.props` 获取到配置参数，组件可以根据 `props` 的不同来确定自己的显示形态，达到可配置的效果。
3. 可以通过给组件添加类属性 `defaultProps` 来配置默认参数。
4. `props` 一旦传入，你就不可以在组件内部对它进行修改。但是你可以通过父组件主动重新渲染的方式来传入新的 `props`，从而达到更新的效果。

------



## state vs props

我们来一个关于 `state` 和 `props` 的总结。

`state` 的主要作用是用于组件保存、控制、修改*自己*的可变状态。`state` 在组件内部初始化，可以被组件自身修改，而外部不能访问也不能修改。你可以认为 `state` 是一个局部的、只能被组件自身控制的数据源。`state` 中状态可以通过 `this.setState`方法进行更新，`setState` 会导致组件的重新渲染。

`props` 的主要作用是让使用该组件的父组件可以传入参数来配置该组件。它是外部传进来的配置参数，组件内部无法控制也无法修改。除非外部组件主动传入新的 `props`，否则组件的 `props` 永远保持不变。

`state` 和 `props` 有着千丝万缕的关系。它们都可以决定组件的行为和显示形态。一个组件的 `state` 中的数据可以通过 `props` 传给子组件，一个组件可以使用外部传入的 `props` 来初始化自己的 `state`。但是它们的职责其实非常明晰分明：*state 是让组件控制自己的状态，props 是让外部对组件自己进行配置*。

如果你觉得还是搞不清 `state` 和 `props` 的使用场景，那么请记住一个简单的规则：尽量少地用 `state`，尽量多地用 `props`。

没有 `state` 的组件叫无状态组件（stateless component），设置了 state 的叫做有状态组件（stateful component）。因为状态会带来管理的复杂性，我们尽量多地写无状态组件，尽量少地写有状态的组件。这样会降低代码维护的难度，也会在一定程度上增强组件的可复用性。前端应用状态管理是一个复杂的问题，我们后续会继续讨论。

React.js 非常鼓励无状态组件，在 0.14 版本引入了函数式组件——一种定义不能使用 `state` 组件，例如一个原来这样写的组件：

```
class HelloWorld extends Component {
  constructor() {
    super()
  }

  sayHi () {
    alert('Hello World')
  }

  render () {
    return (
      <div onClick={this.sayHi.bind(this)}>Hello World</div>
    )
  }
}
```

用函数式组件的编写方式就是：

```
const HelloWorld = (props) => {
    const sayHi = (e) => alert('Hello World!')
    return (
    	<div onClick = {sayHi}> hello World</div>
    )
}
```

以前一个组件是通过继承 `Component` 来构建，一个子类就是一个组件。而用函数式的组件编写方式是一个函数就是一个组件，你可以和以前一样通过 `<HellWorld />` 使用该组件。不同的是，函数式组件只能接受 `props` 而无法像跟类组件一样可以在 `constructor` 里面初始化 `state`。你可以理解函数式组件就是一种只能接受 `props`和提供 `render` 方法的类组件。

但本书全书不采用这种函数式的方式来编写组件，统一通过继承 `Component` 来构建组件。

------



## 渲染列表数据

列表数据在前端非常常见，我们经常要处理这种类型的数据，例如文章列表、评论列表、用户列表…一个前端工程师几乎每天都需要跟列表数据打交道。

React.js 当然也允许我们处理列表数据，但在使用 React.js 处理列表数据的时候，需要掌握一些规则。我们这一节会专门讨论这方面的知识。

### 渲染存放 JSX 元素的数组

假设现在我们有这么一个用户列表数据，存放在一个数组当中：

```
const users = [
  { username: 'Jerry', age: 21, gender: 'male' },
  { username: 'Tomy', age: 22, gender: 'male' },
  { username: 'Lily', age: 19, gender: 'female' },
  { username: 'Lucy', age: 20, gender: 'female' }
]
```

如果现在要把这个数组里面的数据渲染页面上要怎么做？开始之前要补充一个知识。之前说过 JSX 的表达式插入 `{}` 里面可以放任何数据，如果我们往 `{}` 里面放一个存放 JSX 元素的数组会怎么样？

```
...

class Index extends Component {
  render () {
    return (
      <div>
        {[
          <span>React.js </span>,
          <span>is </span>,
          <span>good</span>
        ]}
      </div>
    )
  }
}

ReactDOM.render(
  <Index />,
  document.getElementById('root')
)
```

我们往 JSX 里面塞了一个数组，这个数组里面放了一些 JSX 元素（其实就是 JavaScript 对象）。到浏览器中，你在页面上会看到React.js 把插入表达式数组里面的每一个 JSX 元素一个个罗列下来，渲染到页面上。所以这里有个关键点：*如果你往 {} 放一个数组，React.js 会帮你把数组里面一个个元素罗列并且渲染出来*。 

### 使用 map 渲染列表数据

知道这一点以后你就可以知道怎么用循环把元素渲染到页面上：循环上面用户数组里面的每一个用户，为每个用户数据构建一个 JSX，然后把 JSX 放到一个新的数组里面，再把新的数组插入 `render` 方法的 JSX 里面。看看代码怎么写：

```
const users = [
  { username: 'Jerry', age: 21, gender: 'male' },
  { username: 'Tomy', age: 22, gender: 'male' },
  { username: 'Lily', age: 19, gender: 'female' },
  { username: 'Lucy', age: 20, gender: 'female' }
]

class Index extends Component {
    render() {
        const usersElements  = []; // 保存每个用户渲染以后 JSX 的数组
        for(let user of users) {
            usersElements.push( //循环每个用户，构建JSX,push到数组中
            <div>
              <div>姓名：{user.username}</div>
              <div>年龄：{user.age}</div>
              <div>性别：{user.gender}</div>
              <hr />            
            </div>
            )
        }
    }
}

ReactDOM.render(
  <Index />,
  document.getElementById('root')
)
```

这里用了一个新的数组 `usersElements`，然后循环 `users` 数组，为每个 `user` 构建一个 JSX 结构，然后 push 到 `usersElements` 中。然后直接用表达式插入，把这个 `userElements` 插到 return 的 JSX 当中。因为 React.js 会自动化帮我们把数组当中的 JSX 罗列渲染出来 .

但我们一般不会手动写循环来构建列表的 JSX 结构，可以直接用 ES6 自带的 `map`（不了解 `map` 函数的同学可以先了解相关的知识再来回顾这里），代码可以简化成： 

```
class Index extends Component {
    render() {        
		return(
			<div>
				{ users.map(user) => {
                    return (
                        <div>
                          <div>姓名：{user.username}</div>
                          <div>年龄：{user.age}</div>
                          <div>性别：{user.gender}</div>
                          <hr />
                        </div>                    	
                    )
				}}
			</div>
		)
        }
    }
}
```

这样的模式在 JavaScript 中非常常见，一般来说，在 React.js 处理列表就是用 `map` 来处理、渲染的。现在进一步把渲染单独一个用户的结构抽离出来作为一个组件，继续优化代码： 

```
const users = [
  { username: 'Jerry', age: 21, gender: 'male' },
  { username: 'Tomy', age: 22, gender: 'male' },
  { username: 'Lily', age: 19, gender: 'female' },
  { username: 'Lucy', age: 20, gender: 'female' }
]

class User extends Component {
  render () {
    const { user } = this.props
    return (
      <div>
        <div>姓名：{user.username}</div>
        <div>年龄：{user.age}</div>
        <div>性别：{user.gender}</div>
        <hr />
      </div>
    )
  }
}

class Index extends Component {
  render () {
    return (
      <div>
        {users.map((user) => <User user={user} />)}
      </div>
    )
  }
}

ReactDOM.render(
  <Index />,
  document.getElementById('root')
)
```

这里把负责展示用户数据的 JSX 结构抽离成一个组件 `User` ，并且通过 `props` 把 `user` 数据作为组件的配置参数传进去；这样改写 `Index` 就非常清晰了，看一眼就知道负责渲染 `users` 列表，而用的组件是 `User`。

### key! key! key!

打开控制台看看： React.js 报错了。如果需要详细解释这里报错的原因，估计要单独写半本书。但可以简单解释一下。

React.js 的是非常高效的，它高效依赖于所谓的 Virtual-DOM 策略。简单来说，能复用的话 React.js 就会尽量复用，没有必要的话绝对不碰 DOM。对于列表元素来说也是这样，但是处理列表元素的复用性会有一个问题：元素可能会在一个列表中改变位置。例如：

```
<div>a</div>
<div>b</div>
<div>c</div>
```

假设页面上有这么3个列表元素，现在改变一下位置：

```
<div>a</div>
<div>c</div>
<div>b</div>
```

`c` 和 `b` 的位置互换了。但其实 React.js 只需要交换一下 DOM 位置就行了，但是它并不知道其实我们只是改变了元素的位置，所以它会重新渲染后面两个元素（再执行 Virtual-DOM 策略），这样会大大增加 DOM 操作。但如果给每个元素加上唯一的标识，React.js 就可以知道这两个元素只是交换了位置：

```
<div key='a'>a</div>
<div key='b'>b</div>
<div key='c'>c</div>
```

这样 React.js 就简单的通过 `key` 来判断出来，这两个列表元素只是交换了位置，可以尽量复用元素内部的结构。

这里没听懂没有关系，后面有机会会继续讲解这部分内容。现在只需要记住一个简单的规则：*对于用表达式套数组罗列到页面上的元素，都要为每个元素加上 key 属性，这个 key 必须是每个元素唯一的标识*。一般来说，`key` 的值可以直接后台数据返回的 `id`，因为后台的 `id` 都是唯一的。

在上面的例子当中，每个 `user` 没有 `id` 可以用，可以直接用循环计数器 `i` 作为 `key`：

```
...
class Index extends Component {
  render () {
    return (
      <div>
        {users.map((user, i) => <User key={i} user={user} />)}
      </div>
    )
  }
}
...
```

再看看，控制台已经没有错误信息了。但这是不好的做法，这只是掩耳盗铃（具体原因大家可以自己思考一下）。记住一点：在实际项目当中，如果你的数据顺序可能发生变化，标准做法是最好是后台数据返回的 `id` 作为列表元素的 `key`。

------

## 