- # React

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
    'div',
    {tabIndex: '0'},
    'Hello!'
  );
  ```

  动态 JSX 标签：将一个大写字母开头的变量用作 JSX 标签的类型

  ## 组件

  ### 基本

  创建：

  ```react
  function Welcome(props) {
    return JSX表达式;
  } // 或
  class Welcome extends React.Component {
    render() {
      return <div>{this.props.name}</div>;
    }
  }
  ```

  - React.PureComponent 相比 React.Component 基于 prop 和 state 的浅层对比实现了 shouldComponentUpdate()

  特性：

  - 自定义组件的组件名必须以大写字母开头
  - 对于自定义组件， JSX 所接收的属性（attributes）以及子组件（children）会被转换为单个对象传递给组件，这个对象被称为 “props”
  - 所有 React 组件都必须像纯函数一样保护它们的 props 不被更改。
  - 若 render 函数返回 null，则不会渲染组件，此时不影响生命周期
  - state的变化会重新调用该组件及其后代组件的render函数，**props的变化不会导致重新渲染**

  props.children：每个组件都可以获取到 props.children。它包含组件的开始标签和结束标签之间的内容

  ### state

  适用于 class 声明的组件，必须在构造函数中为 this.state 赋初值:

  ```react
    constructor(props) {
      super(props);
      this.state = {date: new Date()};
    }
  ```

  更新 state：`this.setState({xxx:xxxxx})` 会将提供的对象合并到当前的 state（浅拷贝）。会自动调用 render 方法更新 DOM

  - state 的更新可能是异步的，若要同步更新，给 setState 方法传入函数`(state, props)=>({xxx:xxxxx})`
  - 直接更改 state 对象的属性不会重新渲染组件

  状态提升：将多个组件中需要共享的 state 向上移动到它们的最近共同父组件中，便可实现共享 state。

  - 父组件的 state 保存数据，定义一个函数`onXXXChange(value){}`。在函数中更新 state，然后将函数作为 props 传给子组件，即可实现共享 state

  ### 生命周期

  适用于 class 声明的组件，生命周期函数会在特定的时候被调用

  - `componentDidMount()` 组件已经被渲染到 DOM 中之后
  - `componentDidUpdate()` 组件更新后
  - `componentWillUnmount()` 组件从 DOM 中被移除之后
  - `static getDerivedStateFromError(error)` 捕获错误后调用，内部应更新 static，使得能够渲染降级 UI
  - `componentDidCatch()` 捕获错误后调用，用于打印错误信息
  - `shouldComponentUpdate(nextProps, nextState)` 在重新渲染前调用，返回 false 可以阻止渲染

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

  ### 元素列表

  由 React 元素组成的数组可以直接插入到元素中: `<ul>{ numbers.map( (number) => <li>{number}</li> }</ul>`

  key 属性：

  - key 用于识别哪些元素变更了，key 应该是该列表中的独一无二的字符串
  - 需要给每个列表元素分配一个 key 属性：`<li key={number.toString()}>{number}</li>`

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

  ### ref

  组件内的 React 元素设置`ref`属性后，可以通过组件的 refs 属性获取对该节点的引用

  - 若节点为 HTML 元素，该引用指真实 DOM 元素
  - 若节点为自定义组件，该引用指组件的挂载实例

  创建：

  - 字符串形式：`ref`属性值为字符串，通过`组件实例.refs['xx']` 获取引用（不推荐使用）
  - 回调式：`ref`属性值为函数，该函数接受一个参数 currentNode，为该节点的引用（会在挂载时和卸载时调用，卸载时传入 null）
  - `React.createRef()` 创建并返回一个 ref 对象 。将 ref 对象绑定到元素的 ref 属性后，可以通 ref 对象的 current 属性访问真实 DOM

  Ref 转发：

  - `React.forwardRef((props, ref) => React元素)` 创建并返回一个 React 组件，回调函数返回值将作为其子元素
    - 参数 ref 为其接受的 ref 属性，可将其转发到其组件树下的另一个元素中，此时 ref 对象的 current 属性指向转发后的元素
    - 参数 props 为其接受的 props 属性
  - `<input type="file" />`是非受控组件

  ### 高阶组件

  高阶组件（HOC）就是参数为组件，返回值为新组件的函数
  HOC 不应该修改传入的组件，会将组件包装在容器组件中组成新组件，传入组件应该接收来自容器组件的所有 prop：

  ```react
  render() {
        return <WrappedComponent data={this.state.data} {...this.props} />;
      }
  ```

  注意：

  - 不要在 render 方法中使用 HOC（导致子树每次渲染都进行卸载，重新挂载）
  - 记得复制静态方法（一条条复制 或 hoist-non-react-statics）
  - refs 不会被传递（ref 不是 prop 属性），解决方法：通过 React.forwardRef 方法创建组件作为 HOC 返回值

  ### Protals

  `ReactDOM.createPortal(任何可渲染的React元素，DOM元素)` 作为 render 方法的返回值，将指定 react 元素渲染为为指定 DOM 元素的子元素

  protal 的事件冒泡会沿 React 树传递，而非 DOM 树

  ### 非受控组件

  非受控组件：

  - `<input type="file" />`

  受控组件的表单数据是由 React 组件来管理的。非受控组件的表单数据将交由 DOM 节点来处理。

  使用 ref 从DOM节点中获取表单数据：`ref对象.current.value` `ref对象.current.files`

  表单元素的默认值：指定`defaultValue`属性

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

  Provider 组件：`<Context对象名.Provider value={xx}><消费组件 /></Context 对象名.Provider>`

  - Provider 的值变化时，会重新渲染内部的消费组件和 Consumer 组件

  Consumer 组件：`<Context对象名.Consumer>{value => React节点} </Context 对象名.Consumer>` 根据函数返回值渲染

  - 传递给函数的 `value` 值等同于往上组件树离这个 context 最近的 Provider 提供的 `value` 值。如果没有对应的 Provider，`value` 参数等同于传递给 `createContext()` 的 `defaultValue`。

  contextType：给类挂载静态属性 contextType，将其赋值为 Context 对象。然后在实例中就可以通过`this.context`向上获取最近 Provider 组件的值

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

  prop 的值为一个函数，返回 React 元素，则称之为 render prop，用于告知组件需要渲染什么内容。

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

  给目标组件设置静态属性`propTypes`，值为一个配置对象，配置对象属性名为要进行类型检查的属性名，值为PropTypes类型：`组件名.propTypes = { 属性名 : PropTypes.string }`

  类型可以是Proptypes的静态属性：

  - array、bool、func、number、object、string、symbol、any
  - node：任何可被渲染的元素
  - element：react元素
  - elementType：react元素类型（即，MyComponent）

  调用PropTypes的静态方法也可以返回类型：

  - instanceOf（类名）
  - oneOf（数组）：指定值只能为数组成员之一
  - oneOfType（数组）：数组内需要是类型
  - arrayOf（类型）：指定一个数组由某一类型的元素组成
  - objectOf（类型）
  - shape（{属性名：类型}）：指定一个对象由特定的类型值组成（可包含未指定的属性）
  - exact（{属性名：类型}）：只可以有指定的属性，不可包含额外的属性

  在类型后加上`isRequired`指定该prop为必须项。如：`PropType.string.isRequired`

  ## Router

  ### react-router-dom

  针对web开发，用于实现前端路由的react插件库。

  #### 路由器组件

  `<BrowserRouter></BrowserRouter>`：路由器，将顶层元素包装在路由器内。其属性有：

  - `basename：string` 基本地址，以 / 开头，
  - `getUserConfirmation: (message, callback) => {}` 前置路由守卫，默认使用window.confirm
  - `forceRefresh: bool` 是否在跳转时刷新整页
  - `keyLength: number` location.key的长度，默认为6
  - `children: node` 子节点

  

  `<HashRouter></HashRouter>`：路由器，Hash模式（URL中带#）。属性有：

  - `basename、getUserConfirmation、children`
  - `hashType:string` window.location.hash的编码类型，可取值有：
    - `slash` 默认值，#/。。。
    - `noslash` #。。。
    - `hashbang` #!/。。。

  

  `<MemoryRouter></MemoryRouter>`：路由器，将历史记录保存在内存中，适合没有url的情况。属性有：

  - `getUserConfirmation、keyLength、children`
  - `initialEntries: array` 初始情况下保存的位置信息，数组成员可以是 `{pathname, search, hash, state}` 的完整位置对象或简单的字符串 URL
  - `initialIndex: number` 初始位置索引

  

  `<StaticRouter></StaticRouter>`：路由器，永远不会改变位置，适用于服务器端渲染场景。属性有：

  - `basename、children`
  - `location: string | object`  服务器收到的URL，值需要位置对象或简单的字符串 URL
  - `context: object` 普通对象，其属性会在route渲染时传递给被渲染组件

  

  `<Router></Router>`：路由器，所有Router组件的通用低阶接口。属性有：

  - `history: object`  用于导航的历史记录对象
  - `children: node` 

  #### 路由组件

  `<Link></Link>`：用于跳转路由。属性有：

  - `to: string` 链接位置。例：`/courses?sort=name#the-hash`
  - `to: object` 可选属性：
    - `pathname` 要链接到的路径字符串（ / 开头）
    - `search` 查询参数字符串表示（ ？开头）
    - `hash` hash字符串（#开头）
    - `state: 值` 状态持续到location
  - `replace: bool` 是否以replace模式跳转
  - `innerRef: func` 访问 ref 组件的底层
  - 及其他 a 标签的属性

  

  `<NavLink></NavLink>`：用于跳转路由。属性有：

  - `to`
  - `activeClassName: string` 当元素处于active状态时的className
  - `activeStyle: object` 对象属性名为小驼峰的css属性名，值为字符串
  - `exact: bool` 是否仅在位置完全匹配时才应用 active 的类/样式
  - `strict: bool` 是否在判断位置是否匹配当前的URL时，考虑`pathname` 尾部的斜线
  - `isActive: (match, location)=>{}` 判断active的额外函数
  - `location: object` 用于替换 isActive 的location对象

  

  `<Prompt />`：在位置跳转前给予用户确认信息。属性有：

  - `message: string`、`message: func` 提示信息字符串
  - `when: bool` 是否弹出提示信息

  

  `<Redirect />`：重定向，导航到新的位置。属性有：

  - `to`、`exact`、`strict`
  - `push: bool` 是否将新位置推入历史记录，而非替换当前条目
  - `from: string` 要进行重定向的路径名。所有匹配的 URL 参数都会提供给 `to`，必须包含在 `to` 中用到的所有参数，`to` 未使用的其它参数将被忽略。

  

  `<Route></Route>`：该组件渲染为空，仅当path属性与应用程序的位置匹配时，在route处渲染组件。
  Route有三种渲染方式，对应三个组件属性，只能使用其中一种：

  - `component: node` component渲染，传入组件，该组件接收 route props 作为属性
  - `render: (props) => {}` render渲染，调用函数，渲染返回值
    - route props包含match、history、location等属性
  - `children: ({match}) => {}` children渲染，原理与render相同
  - `path: string` 路径
  - `exact`、`strict`
  - `location: object` 传入location将代替当前历史位置进行路径匹配的判断（优先级低于switch的location）
  - `sensitive: bool` 是否在匹配时区分大小写

  

  `<Switch></Switch>`：只渲染与路径匹配的第一个子router或redirect

  - `location`、`children`

  #### 其他

  withRouter：高阶组件，将一个组件包裹进`Route`里面，`react-router`的三个对象`history, location, match`就会被放进这个组件的`props`属性中

  - 与connect同时使用时，withRouter应在connect之后调用

  ### history

  引入库：`import { createBrowserHistory } from 'history'`

  实例化history对象：`const history = createBrowserHistory();` 也可使用 createHashHistory()等

  基本术语：

  - `browser history` 针对 DOM 环境，用于支持 HTML5 history API 的浏览器
  - `hash history` 针对 DOM 环境，用于传统的旧式（低版本） 浏览器
  - `memory history` history保存于内存，用于测试以及 React Native 等非 DOM 环境

  注意：

  - history对象可变，因此其地址不变，为了保证生命周期正常执行，route渲染的组件应通过`this.props.location`访问location，而非`this.props.history.location`
  - location对象不可变，因此改变时，会换为新对象

  location对象：代表应用程序的位置，含有以下属性：

  - `pathname: string` URL 路径
  - `search: string` URL 中的查询字符串
  - `hash: string` URL 中的 hash 片段
  - `state: object` 存储至 `location` 中的额外状态数据， `hash history` 模式无效

  history对象的属性与方法：

  - `length: number` 历史堆栈中的条目数
  - `action: string` 当前的导航操作（`push`、`replace` 或 `pop`）
  - `location: object` 读写location对象
  - `push(path, [state])` 将一个新条目推入到历史堆栈中
  - `replace(path, [state])` 替换历史堆栈中的当前条目
  - `go(n=0)` 将历史堆栈中的指针移动 n 个条目
    - n=0时刷新页面
  - `goBack()` 返回到上一个页面，相当于 go(-1)
  - `goForward()` 进入到下一个页面，相当于 go(1)
  - `block((location)=>{})` 导航拦截
    - 传入函数的返回值为true表示允许跳转，false表示阻止跳转，为字符串表示弹框由用户确认

  match对象：包含route path匹配URL的信息。属性有：

  - `params: object`  根据 `path` 中指定的动态片段，从 URL 中解析出的键值对
  - `isExact: boolean ` 如果整个 URL 匹配（不包含尾随字符），则为 `true`
  - `path: string ` 用于匹配的路径模式。
  - `url: string ` URL 的匹配部分。
  - 匹配失败时，match为null

  matchPath、

  ## Redux

  JS状态管理器，不止适用于React

  ### 基本概念

  state：

  - 是一个普通对象，用于存储数据。初始state为undefined（可给reducer的第一个参数设置默认值）

  action：

  - action是一个普通对象，用于描述“发生了什么”。将数据从应用传到store，是store数据的唯一来源。
  - action的结构：`{type: '全大写字符串'}` 除了type字段，action对象的结构不作限制
  - action创建函数：返回action对象的函数
  - 调用 `store.dispatch(action对象)` 触发reducer

  reducer：

  - 根据action更新state，是一个纯函数：`(previousState, action) => newState`
  - 返回值一般写为：`{...previousState, ...{xx:xx}}`
  - reducer中不可执行的操作：
    - 修改传入参数（创建新对象作为newState）
    - 执行有副作用的操作，如 API 请求和路由跳转
    - 调用非纯函数，如 `Date.now()` 或 `Math.random()`
  - 合并：`combineReducers({key1: reducer1})` 返回合并后的reducer，该reducer返回的state对象为：
    - `{key1:reducer1返回的state对象}` 

  store：

  - 引入：`import { createStore } from 'redux'`
  - 创建：`let store = createStore( reducer函数[, state的初始状态])`
  - 获取state：`store.getState()`
  - 触发reducer：`store.dispatch(action)` 
  - 注册监听器：`store.subscribe(()=>{})` 返回一个函数，调用该函数可以注销监听器

  ### react-redux

  Redux 的 React 绑定库，使得React组件能从store中读取数据，并向store分发actions更新数据

  

  `connect`方法：

  - 引入：`import { connect } from 'react-redux'`
  - `connect(mapStateToProps, mapDispatchToProps)(React元素)` 创建并返回容器组件
    -  mapStateToProps：`(state, ownProps)=> ({key1: value1})` 一个普通函数，将keys注入为容器组件的props
    - mapDispatchToProps可以是对象或函数：
      - 函数：`(dispatch, ownProps)=> ({key1: value1})` 将keys注入为容器组件的props
      - 对象：`{key1: action创建函数}` 注入为props，key1的值为一个函数，调用时会自动创建action并dispatch

  

  `<Provider store={store}></Provider>`：使得内部所有组件可以访问到store

  ## Hook

  hook不能在class组件中使用

  ```
  const [count, setCount] = useState(0);
  ```

  - 函数传入初始state，返回当前状态与一个用于更新它的函数