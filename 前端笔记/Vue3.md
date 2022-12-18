- # Vue3

  ## 规范

  组合式函数：

  - 自定义的组合式函数以`use`开头，小驼峰命名
  - 推荐返回一个包含多个 ref 的普通的非响应式对象，这样该对象在被解构后仍保持响应性

  内置组件：

  - Vue内置组件均改为大驼峰形式命名，但在DOM模板中仍采用短横分隔形式

  ## 综述

  ### 响应式原理

  `reactive()`返回原始对象的Proxy

  数组响应式同Vue2

  ## 模板语法

  - 插值语法同Vue2，支持动态参数
  - `v-text`、`v-html`、`v-show`、`v-if`、`v-else`、`v-else-if`、`v-for`、`v-on`、`v-bind`、`v-model`、`v-slot`、`v-pre`、`v-once`、`v-cloak`
    - v-on：$event不再可用，给v-on绑定一个箭头函数，函数会接收到事件附带的参数
    - v-model：所添加的修饰符可以通过 `modelModifiers` prop 在组件内访问到，其默认值为空对象
      - 组件自定义修饰符：`modelModifiers` 对象的键名为已添加的修饰符名，键值为true
  - `v-memo` 

  ### 自定义指令

  由setup函数暴露一个变量，命名规范为`vMyDirective`，模板中通过`v-my-directive`使用

  该变量的值是一个对象，定义对象的可选钩子函数有：

  - `created` 绑定元素的attribute前或事件监听器应用前调用
  - `beforeMount` 元素被插入到 DOM 前调用
  - `mounted` 绑定元素的父组件及自己的所有子节点都挂载完成后调用
  - `beforeUpdate` 绑定元素的父组件更新前调用
  - `updated` 绑定元素的父组件及自己的所有子节点都更新后调用
  - `beforeUnmount` 绑定元素的父组件卸载前调用
  - `unmounted` 绑定元素的父组件卸载后调用

  钩子函数的参数：

  - `el`：指令绑定的元素
  - `binding`：一个对象，包含以下属性
    - `value`：传递给指令的值。例如在 `v-xx="1 + 1"` 中，值是 `2`
    - `oldValue`：之前的值，仅在 `beforeUpdate` 和 `updated` 中可用。无论值是否更改，它都可用
    - `arg`：传递给指令的参数 (如果有的话)。例如在 `v-xx:foo` 中，参数是 `"foo"`
    - `modifiers`：一个包含修饰符的对象 (如果有的话)。例如在 `v-xx.foo.bar` 中，修饰符对象是 `{ foo: true, bar: true }`
    - `instance`：使用该指令的组件实例
    - `dir`：指令的定义对象
  - `vnode`：代表绑定元素的底层 VNode
  - `prevNode`：之前的渲染中代表指令所绑定元素的 VNode。仅在 `beforeUpdate` 和 `updated` 钩子中可用

  函数式定义：`app.directive('指令名', (...)=>{})` 指令名不带`v-`前缀，仅在`mounted` 和 `updated` 生效

  ## 全局API

  - `createApp(根组件选项[,要传递的props]): App` 创建应用实例
    - 应用根组件的内容将会被渲染在容器元素里面（相比Vue2，Vue3不会替换要挂载的元素）。容器元素自己将不会被视为应用的一部分
    - 没有设置 `template` 选项时，将自动使用容器的 `innerHTML` 作为模板
  - `createSSRApp(...)` 以 SSR 激活模式创建一个应用实例，用法同createApp

  ## 应用实例

  ```js
  import { createApp } from 'vue'
  const app = createApp(根组件选项)
  app.mount('#app')
  ```

  ### 实例方法

  - `mount('选择器' 或 DOM元素)` 挂载应用，返回根组件实例
  - `unmount()` 卸载应用
  - `component('组件名'[, 组件])` 全局注册，可链式调用
    - 若不传第二个参数，返回已有的该组件
  - `provide(...)` [提供依赖](###其他API)

  ## 组合式API

  组件选项对象中定义特殊钩子函数`setup()`，用于处理组合式API

  组合式函数应该始终被同步调用，在某些场景下，也可以在生命周期钩子中使用他们

  - `<script setup>` 是唯一在调用 await 之后仍可调用组合式函数的地方。编译器会在异步操作之后自动恢复当前的组件实例

  ### setup函数

  setup函数接收参数`(props, context)`

  - props由props选项声明，props对象为响应式，但解构后失去响应式
  - setup函数需要返回一个对象，其属性将会被暴露给组件实例
  - context上下文的属性：
    - attrs 非响应式

  

  `<script setup>` 特有的编译宏命令：见[单文件组件](###单文件组件)

  - `defineProps(option)` 获取props，返回一个响应式对象

    - option同props选项

    - TS的用法：`defineProps<泛型>()`

      ```vue
      const props = defineProps<{
        foo: string
        bar?: number
      }>()
      ```

    - 响应性语法糖：`const {name, count= 100} = defineProps(...)` [需要显式地开启](https://cn.vuejs.org/guide/extras/reactivity-transform.html#explicit-opt-in)

  - `defineEmits(option)` 定义事件

    - option同emits选项

    - 返回值：若只定义一个事件，返回一个等同于 `$emit` 方法的函数；定义多个事件时，返回对象，键为事件名，值为函数

    - TS的用法：

      ```vue
      const emit = defineEmits<{
        (e: 'change', id: number): void
        (e: 'update', value: string): void
      }>()
      ```
    - 若定义了一个原生事件的名字，原生事件将不再响应
    
  - `withDefaults(props, {属性:默认值})`  用于定义props默认值，返回props对象响应式对象

  - `defineExpose(对象)` 暴露属性至组件实例

  - `useSlots()`/`userAttrs()` 返回值与`context.slots`/`context.attrs` 等价

  ### 响应式核心函数

  `ref(值)` 定义响应式变量，将值包裹为ref对象并返回

  - ref对象只有一个属性 value ，指向传入ref的值，对值的操作应通过value属性进行，以确保响应式
  - 如果值是一个对象：
    - 值将通过`reactive()`转为深层次响应式的对象
    - 若要避免深层转换，使用 `shallowRef()`来替代
  - 解包：在保持响应式的同时，无需使用`.value`。ref对象在以下情景会自动解包：
    - 在模板中作为顶层属性访问时
    - 作为插值的最终值时 `{{ object.foo }}`
    - 被嵌套在一个响应式对象中，作为属性被访问或更改时（在响应式数组或 `Map` 这样的原生集合类型中，不会解包）
  - 模板引用：`const xx = ref(null)` 
    - 声明变量名必须与元素的ref属性值相同，元素挂载后，通过该ref对象获取DOM引用或组件实例
    - 对v-for元素使用时：ref值为数组，不保证顺序
    - 若不使用`<script setup>`，需从`setup()`返回该变量
    - 函数式模板引用：为元素的ref属性绑定函数`(el)=>{}`，在每次组件更新时和元素卸载时调用
    - 对组件的引用：使用了`<script setup>`的组件是默认私有的，需要通过defineExpose暴露

  

  `reactive(对象)` 返回一个响应式对象

  - 若要避免深层响应式转换，使用 `shallowReactive()`作替代
  - 返回的对象以及其中嵌套的对象都是通过 `Proxy` 包裹的

  

  `computed(函数/对象)` 计算属性，自动追踪响应式依赖并更新，返回一个ref对象

  - 只接收一个getter函数：ref对象只读
  - 接收带有getter和setter函数的对象：ref对象可读写
  - 特点：计算属性有缓存机制

  

  `readonly(对象/ref)` 返回一个原值的只读代理

  - 传入对象可以是普通对象也可以是响应式对象
  - 只读代理是深层的。它的 ref 解包行为与 `reactive()` 相同。
  - 要避免深层的转换，使用 `shallowReadonly()`作替代。

  

  `watchEffect(fn(cleanupFn)[, option])` 立即运行函数，同时响应式追踪其依赖，依赖更改时重新执行。

  - 返回值：一个用来停止该副作用函数的函数。在异步调用watchEffect时，不会随组件销毁停止监听，需手动停止。
  - 参数：
    - 第一个参数是要运行的副作用函数。这个副作用函数的参数也是一个函数，用来注册清理回调。清理回调会在该副作用下一次执行前被调用，可以用来清理无效的副作用。
    - option：默认情况下，侦听器将在组件渲染之前执行
      - 设置`flush: 'post'`使侦听器在组件渲染后再执行
      - 设置 `flush: 'sync'`在响应式依赖发生改变时立即触发侦听器
      - `onTrack / onTrigger`选项：调试侦听器的依赖
  - `watchPostEffect()`/`watchSyncEffect()` 设置flush选项为`post/sync`时的别名

  

  `watch(源, 回调函数[, option])` 侦听源发生变化时执行回调函数

  - 源可以是：
    - 一个函数，返回一个值
    - 一个 ref
    - 一个响应式对象，此时会自动进行深层监听
    - 由以上类型的值组成的数组
  - 回调函数：`fn(新值, 旧值, cleanup)`
    - 侦听多个源时，新值旧值为数组
    - cleanup为函数，用来注册清理回调。清理回调会在下一次执行前被调用
  - option，选项如下：
    - `immediate`：布尔值，是否在侦听器创建时立即触发回调。第一次调用时旧值是 undefined
    - `deep`：布尔值，如果源是对象，强制深度遍历，以便在深层级变更时触发回调
    - `flush`：调整回调函数的刷新时机。参考回调的刷新时机及 watchEffect()
    - `onTrack / onTrigger`：调试侦听器的依赖

  ### 响应式工具函数

  `isRef(值)` 判断值是否为ref

  `unref(值)` 若参数为ref，返回内部值，否则返回值本身

  ```
  toRef(响应式对象, '属性名')
  ```

  > 将响应式对象的指定属性包装为Ref对象并返回

  ```
  toRefs(响应式对象)
  ```

  > 将响应式对象转换为一个普通对象，该对象的每个属性都是指向源对象相应属性的 ref。每个单独的 ref 都是使用 `toRef()` 创建的。

  ### 其他API

  **生命周期钩子**

  ```
  onMounted
  ```

  - 生命周期钩子API不允许被异步注册

  **依赖注入**

  `provide(注入名:str|Symbol, 值)` 提供依赖，可以被后代组件使用

  - 依赖值可以是ref对象，以保持响应式
    - 建议响应式变更都保持在提供方组件，若要在注入方更改，建议提供方提供修改的方法

  `inject(注入名[, 默认值])` 注入依赖，返回依赖的值

  - `provide()`和`inject()`必须在组件`setup()`阶段同步调用

  ## 组件

  ### 组件基础

  透传：传递给一个组件，却没有被该组件声明为 props 或 emits 的 attribute 或者 **v-on 事件监听器**，会被自动添加到组件根元素上

  - 透传属性保留原始的大小写，不会像prop一样转换为小驼峰
  - 对于多根节点的组件，不会自动透传
  - 对组件使用的自定义指令也会透传

  递归引用：组件可以引用自己，但引入后优先级较低，推荐引入后添加别名

  命名空间组件：

  ```vue
  import * as Form from './form-components'
  <Form.Input>
    <Form.Label>label</Form.Label>
  </Form.Input>
  ```

  

  ### 生命周期

  1. 调用setup函数
  2. `beforeCreate`钩子
  3. 初始化选项式API
  4. `created`钩子
  5. 判断是否存在预编译模板，若不存在则等待动态编译模板
  6. `beforeMount`钩子
  7. 初始化渲染，创建并挂载DOM
  8. `mounted`钩子
  9. 进入更新循环
  10. `beforeUpdate` 钩子
  11. 根据新数据，生成新的虚拟DOM，进行比较后完成页面更新
  12. `updated` 钩子
  13. 回到更新循环开头（第9步），除非组件将要卸载
  14. `beforeUnmount`钩子
  15. 卸载组件
  16. `unmounted`钩子

  ### 单文件组件

  单文件组件的子组件推荐使用大驼峰命名，以区分HTML元素

  通常采用`<script setup>`定义js：

  ```vue
  <script>
  import { reactive } from 'vue'
  export default {
    setup() {
      const state = reactive({ count: 0 })
      return {state}
    }
  }
  </script>
  
  // 可写为:
  <script setup>
  import { reactive } from 'vue'
  const state = reactive({ count: 0 })
  </script>
  ```

  `<script setup>`写法的特点：

  - 内部代码会被编译为setup函数的内容
  - 顶层导入和变量声明可在模板中直接使用
  - 具有一些无需导入，无法在子函数中使用的编译宏命令，见[setup函数](###setup函数)
  - 可以使用顶层 `await`，结果代码会被编译成 `async setup()`
    - 顶层await必须配合Suspense组件使用
    - `<script setup>` 是唯一在调用 await 之后仍可调用组合式函数的地方。编译器会在异步操作之后自动恢复当前的组件实例
  - 可以与普通`<script>`一起使用，通常用于以下情景：
    - 声明额外选项
    - 声明模块的具名导出
    - 运行只需要在模块作用域执行一次的副作用，或是创建单例对象

  ### 事件

  抛出事件：

  - 模板中：`$emit`方法
  - setup中：通过emits选项定义事件，通过`context.emit(...)`方法抛出
  - `<script setup>`中：`defineEmits`方法

  ### 内置组件

  `<Teleport></Teleport>` 将内部的DOM结构传送至其他位置（不影响逻辑关系）

  - `to`属性：值为选择器字符串或DOM元素，将模板传送至该元素下
  - `disabled`属性：值为true时，禁用传送

  ### 动态组件&异步组件

  相比Vue2：

  - is属性用于原生HTML元素时，其值必须加上前缀`vue:`

  

  **异步组件**

  ```js
  const AsyncComp = defineAsyncComponent(() => {
    return new Promise((resolve, reject) => {
      resolve(组件) // 加载成功
      reject(reason) // 加载失败
    })
  }) // 像使用正常组件一样使用异步组件
  defineAsyncComponent(函数或对象)
  ```

  - 函数式：方法接收一个函数，该函数需要返回一个Promise
    - `resolve(组件)`表示加载成功，`reject(reason)`表示加载失败
    - ES模块动态导入也返回Promise，在Vite和Webpack中支持：`defineAsyncComponent(()=> import('url'))`
  - 对象的选项：
    - `loader: () => import('url')` 加载函数
    - `loadingComponent: 组件` 加载异步组件时使用的组件
    - `delay: 毫秒=200` loadingComponent显示前的延迟时间（防止闪烁）
    - `errorComponent: 组件` 加载失败后展示的组件
    - `timeout: 毫秒=Infinity` 超时时间，超时后算作加载失败

  #### Suspense

  ## 过渡

  相比Vue2：

  - 单元素过渡的css类名：`v-enter` 改为 `v-enter-from`，`v-leave` 改为 `v-leave-from`
  - `<transition>`自定义过渡的属性：`enter-class`改为`enter-from-class`，`leave-class`改为`leave-from-class`
  - `<transition>`新增属性：`type:"animation"|"transition"` 同时触发动画和过渡时，哪个优先

  ## 选项

  - `emits:[]`
    - 事件校验
  - `components`
  - `inheritAttrs`
    - 在使用`<script setup/>`时，需要一个额外的`<script>`块声明这个选项
  - `钩子名(){}` 定义钩子函数
    - 生命周期钩子：`beforeCreate`、`created`、`beforeMount`、`mounted`、`beforeUpdate`、`updated`、`beforeDestroy`、`destroyed`
    - keep-alive钩子：`activated` 、`deactivated`
    - `errorCaptured(err, vm, info: str)`钩子：
      - 捕获一个来自后代组件的错误时被调用，返回false阻止错误继续向上传播。 
      - 错误传播规则：沿继承链向上传播并唤起`errorCaptured`，直至全局错误处理`config.errorHandler`。若`errorCaptured`抛出错误，则该错误和原捕获的错误都会发送给全局错误处理

  ## 插件

  插件可以是包含install方法的一个对象，也可以是install函数本身

  - install方法：`install(app, options){}`
    - 通过 `app.component()` 和 `app.directive()` 注册一到多个全局组件或自定义指令
    - 通过 `app.provide()` 使一个资源可被注入进整个应用
    - 向 `app.config.globalProperties` 中添加一些全局实例属性或方法

  应用插件：`app.use(插件[, {选项}])` 

  - 在应用实例化之后调用
  - 选项会传递给install方法的options对象