> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7349971654590857216)

【前端面试复习系列文章】

[2024 前端高频面试题 -- html 篇](https://juejin.cn/post/7316349850855211046 "https://juejin.cn/post/7316349850855211046")

[2024 前端高频面试题 -- CSS 篇](https://juejin.cn/post/6844904013620592654 "https://juejin.cn/post/6844904013620592654")

[2024 前端高频面试题 -- JS 篇](https://juejin.cn/post/7330065707358208010 "https://juejin.cn/post/7330065707358208010")

[2024 前端高频面试题 -- VUE 篇](https://juejin.cn/post/7343484473184698405 "https://juejin.cn/post/7343484473184698405")

[2024 前端高频面试题 -- 前端工程化篇](https://juejin.cn/post/7350535815132659749 "https://juejin.cn/post/7350535815132659749")

[2024 前端高频面试题 -- HTTP 和浏览器篇](https://juejin.cn/post/7351301328206331939 "https://juejin.cn/post/7351301328206331939")

[2024 前端高频面试题 -- 手写代码篇](https://juejin.cn/post/7353456468094599205 "https://juejin.cn/post/7353456468094599205")

[2024 前端高频面试题 -- 数据结构与算法篇](https://juejin.cn/post/7356060104565997605 "https://juejin.cn/post/7356060104565997605")

【导读】本文总结了前端面试中经常问到的 react 高频面试题，仅供参考。

下图为思维导图：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1793f8bbadc549aeb9a94b6f89a4f539~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1332&h=3312&s=572350&e=jpg&b=303030)

### 1.React18 有哪些更新？

*   并发模式
    
*   更新 render API
    
*   自动批处理
    
*   Suspense 支持 SSR
    
*   startTransition
    
*   useTransition
    
*   useDeferredValue
    
*   useId
    
*   提供给第三方库的 Hook
    

### 2. JSX 是什么，它和 JS 有什么区别

JSX 是 JavaScript 语法的扩展，它允许编写类似于 HTML 的代码。它可以编译为常规的 JavaScript 函数调用，从而为创建组件标记提供了一种更好的方法。

JSX 代码如下：

```
<div class />


```

它转换为以下 JS 代码：

```
React.createElement(
  'div',
  {className: 'sidebar'}
)


```

**进一步阅读**

*   [JSX 简介](https://link.juejin.cn?target=https%3A%2F%2Fzh-hans.reactjs.org%2Fdocs%2Fintroducing-jsx.html "https://link.juejin.cn?target=https%3A%2F%2Fzh-hans.reactjs.org%2Fdocs%2Fintroducing-jsx.html")
*   [深入理解 JSX](https://link.juejin.cn?target=https%3A%2F%2Fzh-hans.reactjs.org%2Fdocs%2Fjsx-in-depth.html "https://link.juejin.cn?target=https%3A%2F%2Fzh-hans.reactjs.org%2Fdocs%2Fjsx-in-depth.html")

### 3. 简述 React 的生命周期

React 的生命周期主要分为三个阶段：MOUNTING、RECEIVE_PROPS、UNMOUNTING

*   组件挂载时（组件状态的初始化，读取初始 state 和 props 以及两个生命周期方法，只会在初始化时运行一次）
    
    *   componentWillMount 会在 render 之前调用（在此调用 setState，是不会触发 re-render 的，而是会进行 state 的合并。因此此时的 this.state 不是最新的，在 render 中才可以获取更新后的 this.state。）
    *   componentDidMount 会在 render 之后调用
*   组件更新时（组件的更新过程是指父组件向下传递 props 或者组件自身执行 setState 方法时发生的一系列更新的动作）
    
    *   组件自身的 state 更新，依次执行
        
        *   shouldComponentUpdate（会接收需要更新的 props 和 state，让开发者增加必要的判断条件，在其需要的时候更新，不需要的时候不更新。如果返回的是 false，那么组件就不再向下执行生命周期方法。）
        *   componentWillUpdate
        *   render（能获取到最新的 this.state)
        *   componentDidUpdate（能获取到最新的 this.state)
    *   父组件更新 props 而更新
        
        *   componentWillReceiveProps（在此调用 setState，是不会触发 re-render 的，而是会进行 state 的合并。因此此时的 this.state 不是最新的，在 render 中才可以获取更新后的 this.state。
        *   shouldComponentUpdate
        *   componentWillUpdate
        *   render
        *   componentDidUpdate
*   组件卸载时
    
    *   componentWillMount（我们常常会在组件的卸载过程中执行一些清理方法，比如事件回收、清空定时器）

新版的生命周期函数增加了 getDerivedStateFromProps，这个生命周期其实就是将传入的 props 映射到 state 中。在 React 16.4 之后，这个函数每次会在 re-render 之前调用，

getDerivedStateFromProps 的作用是

*   无条件的根据 prop 来更新内部 state，也就是只要有传入 prop 值， 就更新 state
*   只有 prop 值和 state 值不同时才更新 state 值。

但是盲目使用这个生命周期会有一些问题

*   直接复制 props 到 state 上
*   如果 props 和 state 不一致就更新 state

### 4.React 事件机制和原生 DOM 事件流有什么区别

react 中的事件是绑定到 document 上面的，

而原生的事件是绑定到 dom 上面的，

因此相对绑定的地方来说，dom 上的事件要优先于 document 上的事件执行

### 5.Redux 工作原理

Redux 是 React 的第三方状态管理库，创建于上下文 API 存在之前。它基于一个称为存储的状态容器的概念，组件可以从该容器中作为 props 接收数据。更新存储区的唯一方法是向存储区发送一个操作，该操作被传递到一个 reducer 中。reducer 接收操作和当前状态，并返回一个新状态，触发订阅的组件重新渲染。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d8a88ac3ab3e4964a14f8fce520ac9d5~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=687&h=330&s=6192&e=webp&b=fefefe)

**进一步阅读**

*   [Redux 入门](https://link.juejin.cn?target=https%3A%2F%2Fredux.js.org%2Fintroduction%2Fgetting-started "https://link.juejin.cn?target=https%3A%2F%2Fredux.js.org%2Fintroduction%2Fgetting-started")
*   [深入研究 Redux](https://link.juejin.cn?target=https%3A%2F%2Fwww.sitepoint.com%2Fredux-deep-dive%2F "https://link.juejin.cn?target=https%3A%2F%2Fwww.sitepoint.com%2Fredux-deep-dive%2F")

### 6.React-Router 工作原理? react-router-dom 有哪些组件

路由器组件，路由匹配组件，导航组件

react-router 的依赖库 history

```
1、BrowserHistory：用于支持 HTML5 历史记录 API 的现代 Web 浏览器（请参阅跨浏览器兼容性） \
2、HashHistory：用于旧版Web浏览器\
3、MemoryHistory：用作参考实现，也可用于非 DOM 环境，如 React Native 或测试
​
BrowserHistory：pushState、replaceState
HashHistory：location.hash、location.replace


```

### 7.React hooks 解决了什么问题? 函数组件与类组件的区别

Hooks 解决了我们五年来编写和维护成千上万的组件时遇到的各种各样看起来不相关的问题。无论你正在学习 React，或每天使用，或者更愿尝试另一个和 React 有相似组件模型的框架，你都可能对这些问题似曾相识。

在 React 16.8 版本（引入钩子）之前，使用基于类的组件来创建需要维护内部状态或利用生命周期方法的组件（即`componentDidMount`和`shouldComponentUpdate`）。基于类的组件是 ES6 类，它扩展了 React 的 Component 类，并且至少实现了`render()`方法。

类组件：

```
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}


```

函数组件是无状态的（同样，小于 React 16.8 版本），并返回要呈现的输出。它们渲染 UI 的首选只依赖于属性，因为它们比基于类的组件更简单、更具性能。

函数组件：

```
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}


```

注意：在 React 16.8 版本中引入钩子意味着这些区别不再适用（请参阅 14 和 15 题）。

**进一步阅读**

*   [React 中对比函数式组件和类组件](https://link.juejin.cn?target=https%3A%2F%2Fwww.freecodecamp.org%2Fnews%2Ffunctional-components-vs-class-components-in-react%2F "https://link.juejin.cn?target=https%3A%2F%2Fwww.freecodecamp.org%2Fnews%2Ffunctional-components-vs-class-components-in-react%2F")
*   [React 中函数与类组件比对](https://link.juejin.cn?target=https%3A%2F%2Fmedium.com%2F%40Zwenza%2Ffunctional-vs-class-components-in-react-231e3fbd7108 "https://link.juejin.cn?target=https%3A%2F%2Fmedium.com%2F%40Zwenza%2Ffunctional-vs-class-components-in-react-231e3fbd7108")

### 8.SetState 是同步还是异步的，setState 做了什么

在 React 中，`setState()`函数通常被认为是异步的，这意味着调用 setState() 时不会立刻改变 react 组件中 state 的值，setState 通过触发一次组件的更新来引发重汇，多次 setState 函数调用产生的效果会合并

调用 setState 时，React 会做的第一件事情是将传递给 setState 的对象合并到组件的当前状态。这将启动一个称为和解（reconciliation）的过程。和解（reconciliation）的最终目标是以最有效的方式，根据这个新的状态来更新 UI。 为此，React 将构建一个新的 React 元素树（您可以将其视为 UI 的对象表示）。

一旦有了这个树，为了弄清 UI 如何响应新的状态而改变，React 会将这个新树与上一个元素树相比较

### 9. 什么是 fiber，fiber 解决了什么问题

React15 的 `StackReconciler` 方案由于递归不可中断问题，如果 Diff 时间过长（JS 计算时间），会造成页面 UI 的无响应（比如输入框）的表现，`vdom` 无法应用到 `dom` 中。

为了解决这个问题，React16 实现了新的基于 `requestIdleCallback` 的调度器（因为 `requestIdleCallback` 兼容性和稳定性问题，自己实现了 `polyfill`），通过任务优先级的思想，在高优先级任务进入的时候，中断 `reconciler`。

为了适配这种新的调度器，推出了 `FiberReconciler`，将原来的树形结构（vdom）转换成 Fiber 链表的形式（child/sibling/return），整个 Fiber 的遍历是基于循环而非递归，可以随时中断。

更加核心的是，基于 Fiber 的链表结构，对于后续（React 17 lane 架构）的异步渲染和 （可能存在的）worker 计算都有非常好的应用基础

### 10.React 中在哪捕获错误？

官网例子：

```
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }
​
  static getDerivedStateFromError(error) {
    // 更新 state 使下一次渲染能够显示降级后的 UI
    return { hasError: true };
  }
​
  componentDidCatch(error, errorInfo) {
    // 你同样可以将错误日志上报给服务器
    logErrorToMyService(error, errorInfo);
  }
​
  render() {
    if (this.state.hasError) {
      // 你可以自定义降级后的 UI 并渲染
      return <h1>Something went wrong.</h1>;
    }
​
    return this.props.children; 
  }
}


```

使用

```
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>


```

但是错误边界不会捕获：

```
try{}catch(err){}
///异步代码（例如 setTimeout 或 requestAnimationFrame 回调函数）
///服务端渲染
///它自身抛出来的错误（并非它的子组件)


```

### 11. React 组件传值有哪些方式

父传子：props 子传父：通过在父组件引入的子组件中传递一个函数并传参数，子组件去触发这个函数更改参数完成数据更新

跨多层组件传值：通过 context api 完成

### 12. react 无状态组件和 class 类组件的区别

*   直观区别，函数组件代码量较少，相比类组件更加简洁
*   函数组件看似只是一个返回 react 元素的函数，其实体现的是无状态组件的思想，函数组件中没有 this， 没有 state，也没有生命周期，这就决定了函数组件都是展示性组件，接收 props，渲染 dom，而不关注其他逻辑
*   因为函数组件不需要考虑组件状态和组件生命周期方法中的各种比较校验，所以有很大的性能提升空间

### 13.react 如何做到和 vue 中 keep-alive 的缓存效果

React Keep Alive 提供了 ，你必须把 放在 Provider 里面，并且每个 组件都必须拥有一个唯一的 key

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/db46937914b840b08f53b48ec44dcac6~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=554&h=148&s=14404&e=webp&b=fbfafa)

### 14.React 如何做路由监听

componentDidMount(){ this.context.router.history.listen((route)=>{ if(route.pathname==='/xxx'){ console.log(1); } }); } 还可以用高阶组件

### 15.React 有哪几种方式改变 state

this.forceUpdate this.setState :key 值传递不同也可以 replaceState 也可以改变

### 16.React 有哪几种创建组件方法

React 有三种构建组件的方式

*   React.createClass
*   ES6 class
*   无状态函数

React.createClass 是 React 最传统、兼容性最好的方法。该方法构建一个组件对象, 当组件被调用时，就会创建几个组件实例

ES6 class 方式和 createClass 类似，只是从调用内部方法变成了用类来实现。

无状态组件创建时始终保持一个实例，避免了不必要的检查和内存分配。

### 17.react 中 props 和 state 有什么区别

props 是传递给组件的（类似于函数的形参），而 state 是在组件内被组件自己管理的（类似于在一个函数内声明的变量）

props 是不可修改的，所有 React 组件都必须像纯函数一样保护它们的 props 不被更改。 由于 props 是传入的，并且它们不能更改，因此我们可以将任何仅使用 props 的 React 组件视为 pureComponent，也就是说，在相同的输入下，它将始终呈现相同的输出。 state 是在组件中创建的，一般在 constructor 中初始化 state state 是多变的、可以修改，每次 setState 都异步更新的。

*   [深入理解 Props & State](https://link.juejin.cn/?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F44784850 "https://link.juejin.cn/?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F44784850")

### 18.React 中 keys 的作用是什么？

Keys 是 React 用于追踪哪些列表中元素被修改、被添加或者被移除的辅助标识。

在 React 中渲染集合时，向每个重复的元素添加关键字对于帮助 React 跟踪元素与数据之间的关联非常重要。key 应该是唯一 ID，最好是 UUID 或收集项中的其他唯一字符串：

```
<ul>
  {todos.map((todo) =>
    <li key={todo.id}>
      {todo.text}
    </li>
  )};
</ul>


```

在集合中添加和删除项目时，不使用键或将索引用作键会导致奇怪的行为。

**进一步阅读**

*   [列表 & key](https://link.juejin.cn?target=https%3A%2F%2Fzh-hans.reactjs.org%2Fdocs%2Flists-and-keys.html%23keys "https://link.juejin.cn?target=https%3A%2F%2Fzh-hans.reactjs.org%2Fdocs%2Flists-and-keys.html%23keys")
*   [React 中 key 属性](https://link.juejin.cn?target=https%3A%2F%2Fkentcdodds.com%2Fblog%2Funderstanding-reacts-key-prop "https://link.juejin.cn?target=https%3A%2F%2Fkentcdodds.com%2Fblog%2Funderstanding-reacts-key-prop")

### 19.React 中 refs 的作用是什么？

Refs 是 React 提供给我们的安全访问 DOM 元素或者某个组件实例的句柄。

我们可以为元素添加 ref 属性然后在回调函数中接受该元素在 DOM 树中的句柄，该值会作为回调函数的第一个参数返回：

```
class CustomForm extends Component {
  handleSubmit = () => {
    console.log("Input Value: ", this.input.value)
  }
  render () {
    return (
      <form onSubmit={this.handleSubmit}>
        <input
          type='text'
          ref={(input) => this.input = input} />
        <button type='submit'>Submit</button>
      </form>
    )
  }
}


```

上述代码中的 input 域包含了一个 ref 属性，该属性声明的回调函数会接收 input 对应的 DOM 元素，我们将其绑定到 this 指针以便在其他的类函数中使用。

另外值得一提的是，refs 并不是类组件的专属，函数式组件同样能够利用闭包暂存其值：

```
function CustomForm ({handleSubmit}) {
  let inputElement
  return (
    <form onSubmit={() => handleSubmit(inputElement.value)}>
      <input
        type='text'
        ref={(input) => inputElement = input} />
      <button type='submit'>Submit</button>
    </form>
  )
}


```

**进一步阅读**

*   [何时使用 Refs](https://link.juejin.cn?target=https%3A%2F%2Fzh-hans.reactjs.org%2Fdocs%2Frefs-and-the-dom.html%23gatsby-focus-wrapper "https://link.juejin.cn?target=https%3A%2F%2Fzh-hans.reactjs.org%2Fdocs%2Frefs-and-the-dom.html%23gatsby-focus-wrapper")

### 20.React diff 原理

*   把树形结构按照层级分解，只比较同级元素。
*   列表结构的每个单元添加唯一的 key 属性，方便比较。
*   React 只会匹配相同 class 的 component（这里面的 class 指的是组件的名字）
*   合并操作，调用 component 的 setState 方法的时候, React 将其标记为 dirty 到每一个事件循环结束, React 检查所有标记 dirty 的 component 重新绘制.
*   选择性子树渲染。开发人员可以重写 shouldComponentUpdate 提高 diff 的性能。

**进一步阅读**

*   [不可思议的 React diff](https://link.juejin.cn?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F20346379 "https://link.juejin.cn?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F20346379")
*   [手把手教你学会 React diff 原理](https://link.juejin.cn?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F140489744 "https://link.juejin.cn?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F140489744")

### 21. 受控组件和非受控组件有什么区别？

在 HTML 文档中，许多表单元素（例如`、、`）都保持自己的状态。不受控制的组件将 DOM 视为这些输入状态的真实源。在受控组件中，内部状态用于跟踪元素值。当输入值改变时，React 会重新渲染输入。

在与非 React 代码集成时，不受控制的组件非常有用（例如，如果您需要支持某种 jQuery 表单插件）。

**进一步阅读**

*   [受控组件](https://link.juejin.cn?target=https%3A%2F%2Fzh-hans.reactjs.org%2Fdocs%2Fforms.html%23controlled-components "https://link.juejin.cn?target=https%3A%2F%2Fzh-hans.reactjs.org%2Fdocs%2Fforms.html%23controlled-components")
*   [非受控组件](https://link.juejin.cn?target=https%3A%2F%2Fzh-hans.reactjs.org%2Fdocs%2Funcontrolled-components.html "https://link.juejin.cn?target=https%3A%2F%2Fzh-hans.reactjs.org%2Fdocs%2Funcontrolled-components.html")
*   [受控输入与非受控输入](https://link.juejin.cn?target=https%3A%2F%2Fgoshakkk.name%2Fcontrolled-vs-uncontrolled-inputs-react%2F "https://link.juejin.cn?target=https%3A%2F%2Fgoshakkk.name%2Fcontrolled-vs-uncontrolled-inputs-react%2F")

### 22. 为什么虚拟 dom 会提高性能?

虚拟 dom 相当于在 js 和真实 dom 中间加了一个缓存，利用 dom diff 算法避免了没有必要的 dom 操作，从而提高性能。

具体实现步骤如下：

用 JavaScript 对象结构表示 DOM 树的结构；然后用这个树构建一个真正的 DOM 树，插到文档当中

当状态变更的时候，重新构造一棵新的对象树。然后用新的树和旧的树进行比较，记录两棵树差异

把 2 所记录的差异应用到步骤 1 所构建的真正的 DOM 树上，视图就更新了。

参考链接： [如何理解虚拟 DOM?](https://link.juejin.cn?target=https%3A%2F%2Fwww.zhihu.com%2Fquestion%2F29504639%3Fsort%3Dcreated "https://www.zhihu.com/question/29504639?sort=created")

PS. 未完待续，文中有错误的地方也欢迎评论指出或评论分享自己的面试题。

**另外作者也在找工作，欢迎公司有 HC 的同学内推，base 地：深圳、广州或长沙。**