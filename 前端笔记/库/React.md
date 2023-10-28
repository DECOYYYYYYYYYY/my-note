# React

## 安装

创建项目：`npx create-react-app 项目名`
构建生产版本：

- 对于通过`create-react-app`创建的项目：`npm run build`
- 单文件项目：引入`.min.js`类型的 react 文件

## 目录结构

待补充

## React 元素

- 与 DOM 元素不同，React 元素是创建开销极小的普通对象。React DOM 会负责更新 DOM 来与 React 元素保持一致。

创建：`ReactDOM.render(React元素, DOM元素[, callback])`在 DOM 元素内渲染一个 React 元素，返回对该组件的引用。

- 若该 React 元素之前已经在 DOM 元素内渲染过，会将其更新，并仅会在必要时改变 DOM 以映射最新的 React 元素。

### JSX

介绍：

- 是一个语法糖。在编译后，会被转译为 React.createElement() 函数调用，返回一个对象（React 元素）
- 在渲染所有输入内容之前，默认会进行转义。能防止 XSS 攻击

注意：

- DOM 中的 class 属性要写作 className
- style属性接收一个对象，对象属性名为小驼峰css属性名，对象属性值为css属性值
- React 组件必须以大写字母开头
- 字符串字面量赋值给 prop 时，值未转义。`<A message="&lt;3" />`等价于`<A message={'<3'} />`
- props 的默认值为 true
- 布尔类型、Null 以及 Undefined 作为子元素将会被忽略（有助于依据特定条件渲染元素）

```react
const element = <h1 title={JS表达式}>Hello, {JS表达式}</h1>;

// 使用小驼峰定义属性名
const element = <div tabIndex="0">Hello!</div>; // 使用引号，来将属性值指定为字符串字面量

// 本质为
const element = React.createElement(
  'div', // 该参数也可直接传入组件class
  {tabIndex: '0'},
  'Hello!'
);
```

动态 JSX 标签：将一个大写字母开头的变量用作 JSX 标签的类型



### 列表渲染

由 React 元素组成的数组可以直接插入到元素中: 

```jsx
<ul>
    { numbers.map( (number) => 
		<li key={number.toString()}>{number}</li>
	}
</ul>
```

- 必须为需要给每个列表元素分配一个 key 属性
- key 属性：字符串，用于识别哪些元素变更了，key 应该是该列表中的独一无二的字符串



### Protals

`ReactDOM.createPortal(任何可渲染的React元素，DOM元素)` 作为 render 方法的返回值，将指定 react 元素渲染为指定 DOM 元素的子元素

protal 的事件冒泡会沿 React 树传递，而非 DOM 树

## 组件

### 基本使用

创建：

```jsx
// 函数式组件
function Welcome(props) {
  return JSX表达式;
}
// 类组件
class Welcome extends React.Component {
  render() {
    return <div>{this.props.name}</div>;
  }
}
```

组件渲染：

- 若 render 函数返回 null，则不会渲染组件，此时不影响生命周期
- state 的变化会重新调用该组件及其后代组件的 render 函数，**props 的变化也会导致重新渲染**



### props

> props对象不允许修改

组件的 JSX 所接收的属性（attributes）以及子组件（children）会通过props对象传递给组件。

```jsx
// 使用组件时，传递属性
<Welcome userName={"abc"}></Welcome>

// 在组件中获取prop
class Welcome extends React.Component {
  render() {
    return <div>{this.props.userName}</div>;
  }
}
```

- props.children：它包含组件的开始标签和结束标签之间的内容



设置props默认值：为类添加静态属性defaultProps

```jsx
class Welcome extends React.Component {
    static defaultProps = { userName: 'xx' }
	render() {...}
}
```



### state

适用于 class 声明的组件，必须在构造函数中为 this.state 赋初值:

```react
constructor(props) {
    super(props);
    this.state = {date: new Date()};
}
```



更新 state：`this.setState({xxx:xxxxx})` 会将提供的对象合并到当前的 state（浅拷贝）。并自动调用 render 方法更新 DOM

- 默认情况下使用 setState 更新 state 是异步的，在React控制之外的事件中调用setState是同步的（如原生js绑定事件、setTimeout/setInrerval等）

- 因此可以为 setState 方法传入函数，以获取更新执行前的state和props

  ```js
  this.setState( (preState, preProps)=>({xxx:xxx}) )
  ```

- 也可为setState方法传入一个回调函数作为第二个参数，回调函数在更新完成后执行

  ```js
  this.setState( {xxx:xxx}, ()=>{...} )
  ```



触发重渲染：

- 直接更改 state 对象的属性不会触发render，若传入state的对象地址与state相同，不会触发重渲染
- 多次连续调用setState并不会每次都触发render，React会根据时机调用render以节省性能开销



状态提升：将多个组件中需要共享的 state 向上移动到它们的最近共同父组件中，便可实现共享 state。

- 父组件的 state 保存数据，定义一个函数`onXXXChange(value){}`。在函数中更新 state，然后将函数作为 props 传给子组件，即可实现共享 state



### ref

获取ref：

- 字符串形式：为HTML元素或组件的 `ref` 属性传入字符串，通过`组件实例.refs['xx']` 获取真实DOM或组件实例（不推荐使用）
- 回调式：
  - 获取DOM实例：为DOM的`ref`属性传入函数，该函数接受一个参数 currentNode，为该节点的引用（该函数会在挂载时和卸载时调用，卸载时传入 null）
  - 获取组件实例：为子组件传入一个函数，函数接收ref参数，表示子组件的实例。在子组件的生命周期中调用该函数，并传入this。
- `React.createRef()` v16.3+，创建并返回一个 ref 对象
  - 将 ref 对象绑定到 DOM元素或组件 的 ref 属性后，可以通过 ref 对象的 current 属性访问 DOM实例或组件实例
  - 不可直接用于函数组件，需将函数组件传入forwardRef包装后再使用
- [useRef](####useRef)



`React.forwardRef((props, ref) => React元素)` 创建并返回一个 React 组件，即返回值

- Ref 转发：参数 ref 为其接受的 ref 属性，可将其转发到其组件树下的另一个元素中，此时 createRef创建的ref对象 的 current 属性指向转发后的元素
- 参数 props 为其接受的 props 属性



### 生命周期

适用于 class 声明的组件，生命周期函数会在特定的时候被调用

- `constructor()`

- `getDerivedStateFromProps(nextProps, prevState)` 返回一个对象来更新state，若返回null则不更新

- `componentDidMount()` 组件已经被渲染到 DOM 中之后（挂载后）

- `shouldComponentUpdate(nextProps, nextState)` 在重新渲染前调用，返回 false 可以阻止渲染

  - React.PureComponent 相比 React.Component 基于 prop 和 state 的浅层对比实现了 shouldComponentUpdate()

- `getSnapShotBeforeUpdate(prevProps, prevState)`：用于在组件更新前捕获一些信息，返回值会被传递到componentDidUpdate的第三个参数中

- `componentDidUpdate(prevProps, prevState, snapshot)` 组件更新后

- `componentWillUnmount()` 组件从 DOM 中被移除之后

- `static getDerivedStateFromError(error)` 捕获错误后调用，返回一个对象来更新state，使得能够渲染降级 UI

- `componentDidCatch(error, info)` 捕获错误后调用，用于打印错误信息

  

> 组件的生命周期可以分为**挂载、更新、卸载**阶段
>
> **挂载**
>
> **constructor** 可以进行state和props的初始化
>
> static getDerivedStateFromProps
>
> render
>
> **componentDidMount** 第一次渲染后调用，可以访问DOM，进行异步请求和定时器、消息订阅
>
> 
>
> **更新**
>
> 当组件的props或state变化会触发更新
>
> static getDerivedStateFromProps
>
> **shouldComponentUpdate** 返回一个布尔值，默认返回true，可以通过这个生命周期钩子进行性能优化，确认不需要更新组件时调用
>
> render
>
> getSnapShotBeforeUpdate
>
> componentDidUpdate 在组件完成更新后调用
>
> 
>
> **卸载**
>
> componentWillUnmount 组件从DOM中被移除的时候调用
>
> 
>
> **错误捕获**
>
> static getDerivedStateFromError 在errorBoundary中使用
>
> componentDidCatch
>
> **render**是class组件中唯一必须实现的方法



### 事件

- React 事件采用小驼峰命名
- 不能通过返回 false 来阻止默认行为，必须显示调用`e.preventDefault()`

JSX 语法传入事件处理函数：`<button onClick={activateLasers}></button>`

组件中：

```react
  constructor(props) {
    super(props);
    this.activateLasers = this.activateLasers.bind(this);
```

this 问题：

- 若不想使用 bind 解决 this 问题，采用如下试验性语法来声明方法：`activateLasers = () => {}`
- 或在 JSX 的回调中使用箭头函数（不推荐，可能导致性能浪费）

事件处理函数传参：

```react
<button onClick={(e) => activateLasers(id, e)}></button>
<button onClick={activateLasers.bind(this, id)}></button>
// 以上两种情况，事件对象会被作为第二个参数传递
```



### 表单

state 的值随输入变化：

```react
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {value: ''};
    this.handleChange = this.handleChange.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
          名字:<input type="text" value={this.state.value} onChange={this.handleChange} />
      </form>
    );
  }
}
```

React 中与 HTML 的不同

- textarea：HTML 采用子元素定义文本，React 采用 value 属性定义文本
- select：HTML 采用子元素 selected 属性定义默认选中，React 采用根 select 标签上使用 value 属性定义（可以接收数组）
- label：HTML 的 for 属性，在 React 中应写作 htmlFor

当需要处理多个 input 元素时，可以给每个元素添加 name 属性，并根据 event.target.name 获取。

受控组件：

- 在受控组件上指定 value 的 prop 会阻止用户更改输入。将 value 设置为 undefined 或 null，则可编辑



非受控组件：

- `<input type="file" />`

受控组件的表单数据是由 React 组件来管理的。非受控组件的表单数据将交由 DOM 节点来处理。

使用 ref 从 DOM 节点中获取表单数据：`ref对象.current.value` `ref对象.current.files`

表单元素的默认值：指定`defaultValue`属性



### 高阶组件

高阶组件（HOC）就是参数为组件，返回值为新组件的函数
HOC 不应该修改传入的组件，会将组件包装在容器组件中组成新组件，传入组件应该接收来自容器组件的所有 prop：

```jsx
render() {
    return <WrappedComponent data={this.state.data} {...this.props} />;
}
```

注意：

- 不要在 render 方法中使用 HOC（导致子树每次渲染都进行卸载，重新挂载）
- 记得复制静态方法（一条条复制 或 hoist-non-react-statics）
- refs 不会被传递（ref 不是 prop 属性），解决方法：通过 React.forwardRef 方法创建组件作为 HOC 返回值



### 函数组件

函数组件是一个js函数，返回JSX

```js
const Example = (props) => {
  // 在这使用 Hook
  return <div />;
}

```

设置props默认值：

```ts
// 方法一
const Example = ({xx = 10}) => {...}

// 方法二
const Example = (props) => {...}
Example.defaultProps = {xx: 10}
```



## 高级特性

### 懒加载

动态 import()语法：仅引入需要的代码

```react
import { add } from './math'; //正常情况

import("./math").then(math => {
  console.log(math.add(16, 26));
}); // 自动进行代码分割
```

React.lazy：懒加载组件（暂不支持服务端渲染，可用 Loadable Components 代替）

```react
import OtherComponent from './OtherComponent'; //正常情况

const OtherComponent = React.lazy(() => import('./OtherComponent')); // lazy

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  );
}
```

注意：

- React.lazy 接受一个函数，这个函数需要动态调用 import()。它必须返回一个 Promise，该 Promise 需要 resolve 一个 default export 的 React 组件。

- 应在 Suspense 组件中渲染 lazy 组件，如此可以在等待加载 lazy 组件时做优雅降级（如 loading 指示器等）。
- fallback 属性接受任何在组件加载过程中你想展示的 React 元素。
- Suspense 组件可以置于懒加载组件之上的任何位置。可以用一个 Suspense 组件包裹多个懒加载组件。
- React.lazy 只支持 default export，若要使用命名导出，可以创建一个中间模块，引入目标模块并重新导出为默认模块

### Context

共享对于一个组件树而言是“全局”的数据，避免层层传递



Context 对象：`React.createContext('默认值')` 创建并返回一个 Context 对象

- 实例属性`displayName`，接受字符串，DevTools 会使用该属性作为 provider 和 consumer 的对象名
- 使用Provider和Consumer组件时，需先引入对应的Context对象



Provider 组件：`<Context对象名.Provider value={xx}><消费组件 /></Context 对象名.Provider>`

- Provider 的值变化时，会重新渲染内部的消费组件和 Consumer 组件



Consumer 组件：`<Context对象名.Consumer>{value => React节点} </Context 对象名.Consumer>` 根据函数返回值渲染

- 传递给函数的 `value` 值等同于往上组件树离这个 context 最近的 Provider 提供的 `value` 值。如果没有对应的 Provider，`value` 参数等同于传递给 `createContext()` 的 `defaultValue`。



contextType：给类挂载静态属性 contextType，将属性赋值为 Context 对象。然后在实例中就可以通过`this.context`向上获取最近 Provider 组件的值



### 错误边界

是定义了 `static getDerivedStateFromError()`或`componentDidCatch() `生命周期方法的 Class 组件，可以捕获发生在整个子组件树（不包含自身）的渲染期间、生命周期方法以及构造函数中的错误。

无法捕获的错误：事件处理、异步代码、服务端渲染、它自身抛出来的错误

### Fragments

一个组件通常只能返回一个元素，需要返回多个元素时，使用 Fragments 实现则不需在外层包裹额外节点。Fragment 通常需要设置 key 属性，key 是唯一可以传递给它的属性。

```react
render() {
    return (
      <React.Fragment>
        <td>Hello</td>
        <td>World</td>
      </React.Fragment>
    );
  }
```

短语法：使用`<></>`代替`<React.Fragment></React.Fragment>`

### 性能

虚拟化长列表：对于过长列表，可以使用“虚拟滚动”技术，仅渲染有限的内容。库：` react-window``react-virtualized `

避免重复渲染：组件的 props 或 state 变更时，会重新渲染

Profiler：测量渲染开销

创建：`<Profiler id="xx" onRender={callback}><XXXXXX {...props} /></Profiler>`

回调函数接收参数：

- `id` 发生提交的 Profiler 树的 `id`
- `phase` 判断是首次渲染还是重渲染。值为：`mount` 或 `update` 之一
- `actualDuration` 本次更新花费的渲染时间。
- `baseDuration` 估计不使用 memoization 的情况下渲染需要的时间
- `startTime` 本次更新中 React 开始渲染的时间戳
- `commitTime` 本次更新中 React committed 的时间戳，这个值在所有 profiler 之间共享
- `interactions` 属于本次更新的 interactions 的集合

Diff 算法：在重新渲染时，对比两棵树来高效更新 DOM

- 当普通元素及组件更改类型时，React 会销毁节点并重新建立，其子节点也会因此被销毁
- 当普通元素类型不变时，React 会保留节点，仅比对及更新有改变的属性
- 当组件元素类型不变时，组件实例不变，更新该组件的 props，因此即使 props 没有改变，也会重复渲染：
  - 覆盖生命周期`shouldComponentUpdate()`来提速，默认返回 true，返回 false 时跳过渲染
  - 组件继承 React.PureComponent ，会自动通过 props 和 state 的浅比较来实现 shouldComponentUpdate 方法
  - 通过解构运算符、concat、Object.assign 方法使得引用数据类型的变更正常触发 DOM 更新
- Diff 会递归子节点
- key 属性：Diff 默认按索引取元素对比，当列表元素顺序改变时，会导致 DOM 更新。为 li 设置 key 属性，使其按 key 属性对比元素，避免仅改变顺序导致的 DOM 更新

### Render Props

prop 的值为一个函数，返回 React 元素，则称之为 render prop，在子元素的render方法中调用该prop值作为返回值。

- 可以直接使用 children prop 来实现，将函数直接放在元素内部（推荐）
- 使用 Render Props 会抵消 React.PureComponent 带来的优势

### 严格模式

`<React.StrictMode></React.StrictMode>` 为内部启用严格模式

严格模式有助于：

- 识别不安全的生命周期
- 关于使用过时字符串 ref API 的警告
- 关于使用废弃的 findDOMNode 方法的警告
- 检测意外的副作用
- 检测过时的 context API

### PropTypes

可以对组件上的 props 进行类型检查

引入：`import PropTypes from 'prop-types';`

给目标组件设置静态属性`propTypes`，值为一个配置对象，配置对象属性名为要进行类型检查的属性名，值为 PropTypes 类型：`组件名.propTypes = { 属性名 : PropTypes.string }`

类型可以是 Proptypes 的静态属性：

- array、bool、func、number、object、string、symbol、any
- node：任何可被渲染的元素
- element：react 元素
- elementType：react 元素类型（即，MyComponent）

调用 PropTypes 的静态方法也可以返回类型：

- instanceOf（类名）
- oneOf（数组）：指定值只能为数组成员之一
- oneOfType（数组）：数组内需要是类型
- arrayOf（类型）：指定一个数组由某一类型的元素组成
- objectOf（类型）
- shape（{属性名：类型}）：指定一个对象由特定的类型值组成（可包含未指定的属性）
- exact（{属性名：类型}）：只可以有指定的属性，不可包含额外的属性

在类型后加上`isRequired`指定该 prop 为必须项。如：`PropType.string.isRequired`

## Router

### react-router-dom

针对 web 开发，用于实现前端路由的 react 插件库。

#### 路由器组件

`<BrowserRouter></BrowserRouter>`：路由器，将顶层元素包装在路由器内。其属性有：

- `basename：string` 基本地址，以 / 开头，
- `getUserConfirmation: (message, callback) => {}` 前置路由守卫，默认使用 window.confirm
- `forceRefresh: bool` 是否在跳转时刷新整页
- `keyLength: number` location.key 的长度，默认为 6
- `children: node` 子节点



`<HashRouter></HashRouter>`：路由器，Hash 模式（URL 中带#）。属性有：

- `basename、getUserConfirmation、children`
- `hashType:string` window.location.hash 的编码类型，可取值有：
  - `slash` 默认值，#/。。。
  - `noslash` #。。。
  - `hashbang` #!/。。。



`<MemoryRouter></MemoryRouter>`：路由器，将历史记录保存在内存中，适合没有 url 的情况。属性有：

- `getUserConfirmation、keyLength、children`
- `initialEntries: array` 初始情况下保存的位置信息，数组成员可以是 `{pathname, search, hash, state}` 的完整位置对象或简单的字符串 URL
- `initialIndex: number` 初始位置索引



`<StaticRouter></StaticRouter>`：路由器，永远不会改变位置，适用于服务器端渲染场景。属性有：

- `basename、children`
- `location: string | object` 服务器收到的 URL，值需要位置对象或简单的字符串 URL
- `context: object` 普通对象，其属性会在 route 渲染时传递给被渲染组件



`<Router></Router>`：路由器，所有 Router 组件的通用低阶接口。属性有：

- `history: object` 用于导航的历史记录对象
- `children: node`



#### 路由组件

`<Link></Link>`：用于跳转路由。属性有：

- `to: string` 链接位置。例：`/courses?sort=name#the-hash`
- `to: object` 可选属性：
  - `pathname` 要链接到的路径字符串（ / 开头）
  - `search` 查询参数字符串表示（ ？开头）
  - `hash` hash 字符串（#开头）
  - `state: 值` 状态持续到 location
- `replace: bool` 是否以 replace 模式跳转
- `innerRef: func` 访问 ref 组件的底层
- 及其他 a 标签的属性



`<NavLink></NavLink>`：用于跳转路由。属性有：

- `to`
- `activeClassName: string` 当元素处于 active 状态时的 className
- `activeStyle: object` 对象属性名为小驼峰的 css 属性名，值为字符串
- `exact: bool` 是否仅在位置完全匹配时才应用 active 的类/样式
- `strict: bool` 是否在判断位置是否匹配当前的 URL 时，考虑`pathname` 尾部的斜线
- `isActive: (match, location)=>{}` 判断 active 的额外函数
- `location: object` 用于替换 isActive 的 location 对象



`<Prompt />`：在位置跳转前给予用户确认信息。属性有：

- `message: string`、`message: func` 提示信息字符串
- `when: bool` 是否弹出提示信息



`<Redirect />`：重定向，导航到新的位置。属性有：

- `to`、`exact`、`strict`
- `push: bool` 是否将新位置推入历史记录，而非替换当前条目
- `from: string` 要进行重定向的路径名。所有匹配的 URL 参数都会提供给 `to`，必须包含在 `to` 中用到的所有参数，`to` 未使用的其它参数将被忽略。



`<Route></Route>`：该组件渲染为空，仅当 path 属性与应用程序的位置匹配时，在 route 处渲染组件。
Route 有三种渲染方式，对应三个组件属性，只能使用其中一种：

- `component: node` component 渲染，传入组件，该组件接收 route props 作为属性
- `render: (props) => {}` render 渲染，调用函数，渲染返回值
  - route props 包含 match、history、location 等属性
- `children: ({match}) => {}` children 渲染，原理与 render 相同
- `path: string` 路径
- `exact`、`strict`
- `location: object` 传入 location 将代替当前历史位置进行路径匹配的判断（优先级低于 switch 的 location）
- `sensitive: bool` 是否在匹配时区分大小写



`<Switch></Switch>`：只渲染与路径匹配的第一个子 router 或 redirect

- `location`、`children`



#### 其他

withRouter：高阶组件，将一个组件包裹进`Route`里面，`react-router`的三个对象`history, location, match`就会被放进这个组件的`props`属性中

- 与 connect 同时使用时，withRouter 应在 connect 之后调用



### history

实例化 history 对象：

```js
import { createBrowserHistory } from 'history'
const history = createBrowserHistory(); // 也可使用 createHashHistory
```

基本术语：

- `browser history` 针对 DOM 环境，用于支持 HTML5 history API 的浏览器
- `hash history` 针对 DOM 环境，用于传统的旧式（低版本） 浏览器
- `memory history` history 保存于内存，用于测试以及 React Native 等非 DOM 环境

注意：

- history 对象可变，因此其地址不变，为了保证生命周期正常执行，route 渲染的组件应通过`this.props.location`访问 location，而非`this.props.history.location`
- location 对象不可变，因此改变时，会换为新对象

location 对象：代表应用程序的位置，含有以下属性：

- `pathname: string` URL 路径
- `search: string` URL 中的查询字符串
- `hash: string` URL 中的 hash 片段
- `state: object` 存储至 `location` 中的额外状态数据， `hash history` 模式无效

history 对象的属性与方法：

- `length: number` 历史堆栈中的条目数
- `action: string` 当前的导航操作（`push`、`replace` 或 `pop`）
- `location: object` 读写 location 对象
- `push(path, [state])` 将一个新条目推入到历史堆栈中
- `replace(path, [state])` 替换历史堆栈中的当前条目
- `go(n=0)` 将历史堆栈中的指针移动 n 个条目
  - n=0 时刷新页面
- `goBack()` 返回到上一个页面，相当于 go(-1)
- `goForward()` 进入到下一个页面，相当于 go(1)
- `block((location)=>{})` 导航拦截
  - 传入函数的返回值为 true 表示允许跳转，false 表示阻止跳转，为字符串表示弹框由用户确认

match 对象：包含 route path 匹配 URL 的信息。属性有：

- `params: object` 根据 `path` 中指定的动态片段，从 URL 中解析出的键值对
- `isExact: boolean ` 如果整个 URL 匹配（不包含尾随字符），则为 `true`
- `path: string ` 用于匹配的路径模式。
- `url: string ` URL 的匹配部分。
- 匹配失败时，match 为 null

matchPath、

## Redux

JS 状态管理器，不止适用于 React

### 基本概念

state：

- 是一个普通对象，用于存储数据。初始 state 为 undefined（可给 reducer 的第一个参数设置默认值）

action：

- action 是一个普通对象，用于描述“发生了什么”。将数据从应用传到 store，是 store 数据的唯一来源。
- action 的结构：`{type: '全大写字符串'}` 除了 type 字段，action 对象的结构不作限制
- action 创建函数：返回 action 对象的函数
- 调用 `store.dispatch(action对象)` 触发 reducer

reducer：

- 根据 action 更新 state，是一个纯函数：`(previousState, action) => newState`
- 返回值一般写为：`{...previousState, ...{xx:xx}}`
- reducer 中不可执行的操作：
  - 修改传入参数（创建新对象作为 newState）
  - 执行有副作用的操作，如 API 请求和路由跳转
  - 调用非纯函数，如 `Date.now()` 或 `Math.random()`
- 合并：`combineReducers({key1: reducer1})` 返回合并后的 reducer，该 reducer 返回的 state 对象为：
  - `{key1:reducer1返回的state对象}`

store：

- 引入：`import { createStore } from 'redux'`
- 创建：`let store = createStore( reducer函数[, state的初始状态])`
- 获取 state：`store.getState()`
- 触发 reducer：`store.dispatch(action)`
- 注册监听器：`store.subscribe(()=>{})` 返回一个函数，调用该函数可以注销监听器

### react-redux

Redux 的 React 绑定库，使得 React 组件能从 store 中读取数据，并向 store 分发 actions 更新数据

`connect`方法：

- 引入：`import { connect } from 'react-redux'`
- `connect(mapStateToProps, mapDispatchToProps)(React元素)` 创建并返回容器组件
  - mapStateToProps：`(state, ownProps)=> ({key1: value1})` 一个普通函数，将 keys 注入为容器组件的 props
  - mapDispatchToProps 可以是对象或函数：
    - 函数：`(dispatch, ownProps)=> ({key1: value1})` 将 keys 注入为容器组件的 props
    - 对象：`{key1: action创建函数}` 注入为 props，key1 的值为一个函数，调用时会自动创建 action 并 dispatch

`<Provider store={store}></Provider>`：使得内部所有组件可以访问到 store

## Hook

> React v16.8+开始支持hook
>
> hook 不能在 class 组件中使用
>
> hook即js函数，但只能在函数最外层调用hook（不能在if等语句块内），且只能在React函数组件和自定义hook中调用hook
>
> hook的函数名以use开头



### 基础Hook

#### useState

```js
const [count, setCount] = useState(初始值);
```

参数：state 的初始值；可以是一个函数，将其返回值作为初始值

返回值：返回当前状态与一个用于更新它的函数

- set函数可以接收一个函数，该函数接收更新前的state，并将其返回值作为更新后的值（若返回值为对象，需要手动合并对象`(pre)=>({...pre, ...newObj})`）

注意：调用set函数的更新逻辑同[setState方法](###state)



#### useEffect

```js
useEffect( ()=>{...} , [值]);
```

参数一：副作用回调函数

- 在执行DOM更新（组件挂载和后续更新）之后调用回调函数
- 回调函数可以返回一个副作用清除函数，React会在需要时调用该函数（组件卸载时、非第一次调用回调函数时）

参数二：可选参数。仅当数组内的值变更时，执行副作用

- 可传入空数组，表示副作用仅执行一次（仅在组件挂载时执行，并在卸载时调用副作用清除函数）

特点：

- 不同于componentDidMount、componentDidUpdate，useEffect会在浏览器绘制后延迟执行，但保证在任何新渲染前执行



#### useContext

> 获取Context的值

```js
const value = useContext(Context对象);
```

参数：context对象（React.createContext 的返回值）

返回值：context的当前值，由上层组件中距离当前组件最近的`<MyContext.Provider>` value 属性决定

特点：在Provider更新时，会触发重渲染，即使祖先使用 React.memo 或 shouldComponentUpdate，也会在组件本身使用 useContext 时重新渲染



### 额外Hook

#### useReducer

> hook适配reducer

```js
const [state, dispatch] = useReducer(reducer, initialArg [, init] );
```

参数：

- initialArg：初始state。
- init：可选参数，是一个函数，接收initialArg，将返回值作为初始state



#### useCallback

> 获取memoized回调函数，将该函数传递给子组件后，仅当指定依赖项更新时，才重新生成该函数，并触发子组件重渲染

```js
const memoizedCallback = useCallback(() => {}, [a, b]);
```

参数：分别为回调函数和依赖项数组

返回值：传入回调函数的memoized版本



#### useMemo

> 返回一个memoized值，将值传递给子组件后，仅当指定依赖项更新时，才重新计算该值，并触发子组件重渲染

```js
const memoizedValue = useMemo(() => 值, [a, b]);
```

参数：分别为创建函数和依赖项数组

返回值：创建函数的返回值的memoized版本



#### useRef

> 返回一个 ref 对象，ref 对象本身在组件的整个生命周期内保持不变，因此变更 current **不会触发重渲染**

```js
const refContainer = useRef(initialValue);
```

参数：初始值

返回值：ref 对象，通过其 current 属性获取值，current 属性可读写

使用：

- 获取组件实例或DOM：将ref对象传入组件或DOM的ref属性
- 可以给current属性赋值，以此保存任意可变值



#### useImperativeHandle

> 自定义暴露给父组件的实例值

```js
useImperativeHandle(ref, createHandle, [deps])
```

参数一：父组件传入的ref

参数二：返回对象的函数，将对象的属性暴露。

- 父组件可获取 `该组件的ref.current.暴露的属性` 访问

使用：应当与 `forwardRef` 一起使用

```js
function FancyInput(props, ref) {
  const inputRef = useRef();
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    }
  }));
  return <input ref={inputRef} />;
}
FancyInput = forwardRef(FancyInput);
```



#### useLayoutEffect

> 使用方式与useEffect相同，但该函数会在DOM变更后、渲染前执行effect，可用于读取DOM布局并同步触发重渲染



#### useDebugValue

> 用于在React开发者工具中显示自定义hook的标签
>
> 通常推荐在共享hook库中使用

```js
useDebugValue(value [, (value)=>{}] )
```

参数一：标签名

参数二：可选格式化函数，接收参数一，需返回一个格式化的显示值

注意：最终传入函数的显示值需为字符串



## 类型标注

函数组件：

- 借助React.FC：

  ```ts
  interface Props {
      buttonConfig?: ButtonListConfig;
  }
  const App: React.FC<Props> = (props) => {}
  ```

- React.FC 不能与 forwardRef 同时使用，需改用如下方式

  ```ts
  // 组件中
  interface Handle {
      start: () => void;
  }
  const ButtonList: React.ForwardRefRenderFunction<Handle, Props> = (props, ref) => {
      useImperativeHandle(ref, () => ({
          start: ()=>{...}
      }));
  }
  export default React.forwardRef(ButtonList)
  ```

  ```tsx
  // 在父组件中使用时
  const buttonListRef = useRef<React.ElementRef<typeof ButtonList>>(null);
  buttonListRef.current?.start();
  ```

  