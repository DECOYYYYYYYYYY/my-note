#  Vue3

## 规范

组合式函数：

- 自定义的组合式函数以`use`开头，小驼峰命名
- 推荐返回一个包含多个 ref 的普通的非响应式对象，这样该对象在被解构后仍保持响应性

内置组件：

- Vue内置组件均改为大驼峰形式命名，但在DOM模板中仍采用短横分隔形式

## 综述

### 响应式原理

> vue能检测数组的方法：`push`、`pop`、`shift`、`unshift`、`splice`、`sort`、`reverse`

主要术语：

```js
let A0 = ref(1)
let A1 = ref(2)
let A2 = computed(() => A0.value + A1.value)
```

- 副作用（effect）：是一个函数，用于更改程序里的状态。如传入computed的函数
- 依赖（dependency）：在副作用中用到的变量。如A0和A1
- 订阅者（subscriber）：将某个副作用设置为某个变量的订阅者，变量发生变化后会重新执行副作用



具体实现：

1. watchEffect、computed等函数，会在执行时将传入的副作用函数设为当前副作用
2. reactive返回原始对象的Proxy，Proxy设置了getter和setter；ref返回一个带getter和setter的对象
3. getter中，会将当前运行的副作用添加为该变量（依赖）的订阅者，会在副作用运行前被设置
4. setter中，在赋值完成后，重新执行该依赖的订阅者数组中的所有副作用



### 响应式语法糖

> 借助编译宏命令实现，编译宏命令是全局可用的，可以不导入
>
> 响应式语法糖[需要显式地开启](https://cn.vuejs.org/guide/extras/reactivity-transform.html#explicit-opt-in)

- ref无需使用 `.value` 即可访问值：

  ```js
  let count = $ref(0)
  count++
  
  // 编译为
  let count = ref(0)
  count.value++
  ```

  - 编译宏命令：`$ref`、`$computed`、`$shallowRef`、`$customRef`、`$toRef`

- 使用`$()`解构ref：从响应式对象或包含多个ref的对象中解构

  ```js
  const { x, y } = $(useMouse())
  console.log(x, y)
  
  // 编译为
  const __temp = useMouse(),
    x = toRef(__temp, 'x'),
    y = toRef(__temp, 'y')
  console.log(x.value, y.value)
  ```

- 使用`$()`将现存的 ref 转换为响应式变量：无需 `.value` 即可访问值

  ```js
  let count = $(ref(0))
  ```

- props解构：可以指定默认值和别名

  ```js
  const {name, count = 100} = defineProps(...)
  ```

- 使用上述语法糖后，若不希望某个变量被自动加上`.value`，可使用`$$(变量)`

- TS对宏函数的类型支持：全局引入，可添加进`env.d.ts`文件中

  ```ts
  /// <reference types="vue/macros-global" />
  ```

  - 若从`vue/macros`中显式引入宏函数，则不需要上述全局声明



## 模板语法

> 插值语法同Vue2，支持动态参数

### 内置指令

内置指令参考Vue2：`v-text`、`v-html`、`v-show`、`v-if`、`v-else`、`v-else-if`、`v-for`、`v-on`、`v-bind`、`v-model`、`v-slot`、`v-pre`、`v-once`、`v-cloak`



与Vue2的差别：

- `v-on`：$event不再可用，给v-on绑定一个箭头函数，函数会接收到事件附带的参数

- `v-bind`：移除`.sync`修饰符，新增修饰符：
  - `.camel`：将绑定属性从短横线命名转为驼峰命名
  - `.prop`：强制绑定为DOM属性
  - `.attr`：强制绑定为DOM特性
  
- `v-model`：
  
  - 组件使用 `v-model`：在组件内接收参数并触发事件即可
  
    ```vue
    <CustomInput v-model="serchText" />
    <!-- 会被展开为如下形式 -->
    <CustomInput
      :modelValue="searchText"
      @update:modelValue="newValue => searchText = newValue"
    />
    ```
  
  - 所添加的修饰符可以通过 `modelModifiers` prop 在组件内访问到，其默认值为空对象。有修饰符时，会将 `修饰符名:true` 的键值对添加进该对象中
  



新增的指令：

- `v-memo="数组"`：组件重新渲染时，若数组内元素不变，跳过该元素及其子元素的渲染
  - 与 `v-for` 一起使用时，必须 `v-memo` 绑定在 `v-for` 所在元素上



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



全局自定义指令：`app.directive('指令名', 函数或对象)` 指令名不带`v-`前缀

- 函数式：`(钩子函数参数)=>{}` ，仅在`mounted` 和 `updated` 生效
- 对象式：同setup的定义对象



## 应用实例

```js
import { createApp } from 'vue'
const app = createApp(根组件选项)
app.mount('#app')
```

### 全局API

> 通过import从vue中引入

- `version: string`：Vue版本号

- `nextTick(回调函数)`

- `createApp(根组件选项[,要传递的props对象]): App` 创建应用实例

  > 应用根组件的内容将会被渲染在容器元素里面（相比Vue2，Vue3不会替换要挂载的元素）。容器元素自己将不会被视为应用的一部分

  - 没有设置 `template` 和 `render` 选项时，将自动使用容器的 `innerHTML` 作为模板

- `createSSRApp(...)` 以 SSR 激活模式创建一个应用实例，用法同createApp

- `defineComponent(组件选项对象):选项对象构造函数`：为选项式API提供[类型推导](###TS相关)

- `defineAsyncComponent(异步组件选项对象 | 异步加载函数):组件`：定义[异步组件](###动态组件&异步组件)

- `defineCustomElement(组件选项对象):自定义HTML元素类的构造器`：定义自定义元素

  - 组件选项额外支持styles选项，它是一个内联（行内）CSS字符串数组

  - 注册：

    ```js
    customElements.define('my-vue-element', 返回的构造器)
    ```

### 实例属性

- `version: string`：Vue版本号

- `config: obj`：应用配置对象，对象可选属性如下

  - `errorHandler: (err, instance, info) => void`：全局错误处理函数

    - 参数分别为：错误对象、触发错误的组件实例、错误信息字符串

  - `warnHandler: (msg, instance, trace) => void`：全局运行时警告处理函数

    - 参数分别为：警告信息、来源组件实例、组件追踪字符串

  - `performance: bool`：设为true，在浏览器开发工具的性能页中启用vue支持

  - `globalProperties: Record<string, any>`：定义全局属性

    ```js
    // Vue2中使用原型定义全局属性，Vue3中已不支持，使用如下写法
    app.config.globalProperties.msg = 'hello'
    ```

### 实例方法

- `mount('选择器' 或 DOM元素)`：挂载应用，返回根组件实例
- `unmount()`：卸载应用
- `component('组件名'[, 组件])` 全局注册，可链式调用
  - 若不传第二个参数，返回已有的该组件
- `provide(...)`：提供依赖，详见[依赖注入](####依赖注入)
- `directive('指令名'[, 对象或函数])`：注册全局指令，详见[自定义指令](###自定义指令)
  - 忽略第二个参数则返回已有的该名字的指令
- `use()`：使用插件
- `mixin(选项)`：混入，不推荐使用

## 组合式API

> 组件选项对象中定义特殊钩子函数`setup()`，用于处理组合式API
>
> 组合式函数应该始终被同步调用，在某些场景下，也可以在生命周期钩子中使用他们
>
> - `<script setup>` 是唯一在调用 await 之后仍可调用组合式函数的地方。编译器会在异步操作之后自动恢复当前的组件实例
>

### setup函数

setup函数接收参数`(props, context)`

- `props` 为响应式对象

- `context` 上下文对象（非响应式）的属性：
  
  - `attrs` 对象，透传Attributes，等效于`$attrs`
  - `slots` 对象，插槽，等效于`$slots`
  - `emit` 函数，用于触发事件，等效于`$emit`
  - `expose` 函数，用于暴露公共属性。`expose(对象)`
  
- setup函数需要返回一个对象，其属性将会被暴露给组件实例

  - setup函数也可以返回一个调用了渲染函数的函数，此时通过expose暴露属性

    ```js
    setup(props, { expose }) {
        const count = ref(0)
        expose({count})
        return () => h('div', count.value)
    }
    ```

    

### 编译宏命令

`<script setup>` 的编译宏命令：特点见[单文件组件](### 单文件组件)

- `defineProps(option)` 获取props，返回一个响应式对象

  - props为一个响应式对象，若父组件传入ref属性，会将其自动解包成props的属性。此时对属性使用isRef判为false

  - option同props选项

  - TS用法：`defineProps<参数>()`

    ```vue
    const props = defineProps<{
      foo: string
      bar?: number
    }>()
    ```

    - 参数可以是：对象类型字面量、对**同一文件**的接口或对象类型的引用

- `defineEmits(数组或对象)` 定义事件

  - 参数：

    - 数组式：传入事件名字符串数组
  - 对象式：键名为事件名，键值为null或一个验证函数，验证函数会接收`$emit` 调用的额外参数，验证函数返回true表示参数通过验证
  
- 返回值：若只定义一个事件，返回一个等同于 `$emit` 方法的函数；定义多个事件时，返回对象，键为事件名，值为函数
  
  - TS用法：
  
    ```vue
    const emit = defineEmits<{
      (e: 'change', id: number): void
      (e: 'update', value: string): void
    }>()
    ```
    
  - 若定义了一个原生事件的名字，原生事件将不再响应
  
- `withDefaults(props, {属性:默认值})`  用于定义props默认值，返回props响应式对象

- `defineExpose(对象)` 暴露属性至组件实例

- `useSlots()`/`userAttrs()` 返回值与`context.slots`/`context.attrs` 等价



### 响应式函数

#### ref

`ref<T>(value: T): ref对象`：定义响应式变量，将值包裹为ref对象并返回

- 参数与返回值：

  - ref对象只有一个属性 value ，指向传入的值，对值的操作应通过value属性进行，以确保响应式
  - 如果值是一个对象，值将通过`reactive()`转为深层次响应式的对象
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
  
- 标注类型：

  ```ts
  import { ref } from 'vue'
  import type { Ref } from 'vue'
  import MyModal from './MyModal.vue'
  
  const year: Ref<string | number> = ref('2020') // 方法一
  const year = ref<string | number>('2020') // 方法二
  
  const el = ref<HTMLInputElement | null>(null) // 为模板引用标注类型
  const modal = ref<InstanceType<typeof MyModal> | null>(null) // 为组件模板引用标注
  ```



`shallowRef()`：ref方法的浅层作用形式



`triggerRef(ref: ShallowRef)`：强制触发依赖于一个浅层ref的副作用，通常在对浅引用的内部值进行深度变更后使用



`customRef(工厂函数)`：创建一个自定义ref，显式声明对其依赖追踪和更新触发的控制方式

- 工厂函数接受 `track` 和 `trigger` 两个函数作为参数，并返回一个带有 `get` 和 `set` 方法的对象。track用于跟踪依赖，trigger用于触发副作用

  ```js
  import { customRef } from 'vue'
  
  const value = 'hello'
  const valueRef = customRef((track, trigger) => {
      return {
          get() {
              track()
              return value
          },
          set(newValue) {
              value = newValue
              trigger()
          }
      }
  })
  ```

  

#### reactive

`reactive(对象)` 返回一个深层响应式对象

- 若要避免深层响应式转换，使用 `shallowReactive()`作替代

- 标注类型：不推荐使用泛型参数

  ```ts
  import { reactive } from 'vue'
  
  interface Book {
    title: string
    year?: number
  }
  const book: Book = reactive({ title: 'Vue 3 指引' })
  ```



`shallowReactive()`：reactive方法的浅层作用形式



#### computed

`computed(函数/对象[, 调试对象]): ref对象` 计算属性，自动追踪响应式依赖并更新

- 第一个参数：

  - 只接收一个getter函数：ref对象只读
  - 接收带有getter和setter函数的对象：ref对象可读写

- 第二个参数：带有 `onTrack` 和 `onTrigger` 选项的对象（调试信息仅在开发模式生效）

  - `onTrack(e){debugger}`：依赖被追踪（读取）时调用
  - `onTrigger(e){debugger}}`：依赖修改导致订阅者重新执行副作用时调用

- 特点：计算属性有缓存机制

- 标注类型：

  ```ts
  const double = computed<number>(...) // getter返回值和setter传入值需为number
  ```

  

#### readonly

`readonly(对象/ref)` 返回一个原值的只读代理

- 传入对象可以是普通对象也可以是响应式对象
- 只读代理是深层的。它的 ref 解包行为与 `reactive()` 相同。
- 要避免深层的转换，使用 `shallowReadonly()`作替代。



`shallowReadonly()`：readonly方法的浅层作用形式



#### watchEffect

`watchEffect(fn(cleanupFn)[, option])` 立即运行函数，同时响应式追踪其依赖，依赖更改时重新执行。

- 返回值：一个用来停止该副作用函数的函数。在异步调用watchEffect时，不会随组件销毁停止监听，需手动停止。
- 参数：
  - 第一个参数：要运行的副作用函数。这个副作用函数的参数也是一个函数，用来注册清理回调。清理回调会在该副作用下一次执行前被调用，可以用来清理无效的副作用。
  - option，可选项如下：
    - `flush: 'pre' | 'post' | 'sync'`：设置侦听器执行时机
      - `pre`：默认值，在组件渲染之前执行
      - `post`：在组件渲染后再执行
      - `sync`：在响应式依赖发生改变时立即触发侦听器
    - `onTrack / onTrigger`：同computed
- `watchPostEffect()`/`watchSyncEffect()` 设置flush选项为`post/sync`时的别名



#### watch

`watch(源, 回调函数[, option]): 停止函数` 侦听源发生变化时执行回调函数

- 源可以是：
  - 一个getter函数（会返回一个值的函数）
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
  - `onTrack / onTrigger`：同computed



#### 工具函数

`isRef(值):bool`：判断值是否为ref（返回值可用作ts类型守卫）

`isProxy(值):bool`：检查一个对象是否是由reactive、readonly或其shallow方法创建的代理

`isReactive(值):bool`：检查一个对象是否是由reactive或shallowReactive方法创建的代理

`isReadonly(值):bool`：检查一个对象是否是由readonly或shallowReadonly方法创建的代理



`unref(值)`：若参数为ref，返回内部值，否则返回值本身

`toRef(响应式对象, '属性名'):ref`：基于响应式对象的属性，创建一个与属性同步的ref

`toRefs(响应式对象)`：将响应式对象转换为一个普通对象，该对象的每个属性都是指向源对象相应属性的 ref。每个单独的 ref 都是使用 `toRef()` 创建的。

`toRaw()`：返回由reactive、readonly或其shallow方法创建的代理的原始对象（一般用于临时读取而不引起代理跟踪的开销）

`markRaw(对象):对象本身`：将对象标记为不可转为代理，但其子对象仍可转为代理



`effectScope():Effectscope`：创建一个effect作用域，作用域的实例方法有：

- `run(回调)`：在回调函数中创建响应式副作用（计算属性和侦听器），会被作用域捕获
- `stop()`：停止作用域，处理所有捕获的副作用

`getCurrentScope():Effectscope | und`：若有，返回当前活跃的effect作用域

`onScopeDispose(回调)`：在当前活跃的effect作用域上注册回调，在作用域停止时调用



### 生命周期钩子

> 生命周期钩子API不允许被异步注册

普通生命周期钩子：

`onBeforeMount(回调)`：组件挂载前调用

`onMounted(回调)`：组件挂载后调用

`onBeforeUpdate(回调)`：组件因响应式状态变更而更新其DOM树前调用

`onUpdated(回调)`：组件因响应式状态变更而更新其DOM树后调用

`onBeforeUnmount(回调)`：组件卸载前调用

`onUnmounted(回调)`：组件卸载后调用



keep-alive钩子：

`onActivated(回调)`：若组件实例在keepAlive缓存，在被插入DOM时调用

`onDeactivated(回调)`：若组件实例在keepAlive缓存，在从DOM中移除时调用



错误处理钩子：

`onErrorCaptured( (err,instance,info)=>bool|void )`：捕获后代组件错误时调用

- 参数分别为：错误对象、触发错误的组件实例、错误信息字符串
- 回调函数返回`false`：阻止错误继续向上传播
- 错误传播规则：沿继承链向上传播并唤起errorCaptured，直至全局错误处理config.errorHandler。若errorCaptured抛出错误，则该错误和原捕获的错误都会发送给全局错误处理



调试钩子：

`onRenderTracked( (e)=>void )`：组件渲染过程中追踪到响应式依赖时调用

`onRenderTriggered( (e)=>void )`：响应式依赖的变更触发了组件渲染时调用

- onRenderTracked和triggered钩子仅在开发模式中有效。推荐在回调中放入`debugger`语句



SSR钩子：

`onServerPrefetch( ()=>promise )`：组件实例在服务器上被渲染前调用，

- 若返回promise，会在渲染组件前等待promise完成；该钩子一般用于抓取数据



### 依赖注入

`provide(注入名:str|Symbol, 值)` 提供依赖，可以被后代组件使用

- 依赖值可以是ref对象，以保持响应式
- 建议响应式变更都保持在提供方组件，若要在注入方更改，建议提供方提供修改的方法



`inject(注入名[, 默认值]): 值|und` 注入依赖，返回依赖的值

- 默认值参数也可以传入一个工厂函数；若默认值本身是一个函数，需要将false作为第三个参数传入，表示该函数是默认值而非工厂函数
- 存在多个同名依赖时，组件链上离得近的依赖优先



标注类型：

```ts
import { provide, inject } from 'vue'
import type { InjectionKey } from 'vue'

// 方法一: 使用InjectionKey接口
const key = Symbol() as InjectionKey<string>
provide(key, 'foo') // 若提供的是非字符串值会导致错误
const foo = inject(key) // foo 的类型：string | undefined

// 方法二: 使用泛型
provide('foo', 'xxx')
const foo = inject<string>('foo') // 类型：string | undefined
const foo = inject<string>('foo', 'bar') // 指定默认值, 类型：string
const foo = inject('foo') as string // 强制转换, 类型：string
```



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

单文件组件（SFC）的子组件推荐使用大驼峰命名，以区分HTML元素

单文件组件特点：

- 递归引用：在模板中使用 `<组件所在文件名/>` 引用自己

- 语块可以使用src进行导入：

  ```vue
  <template src="./template.html"></template>
  <style src="./style.css"></style>
  <!-- script setup 不能与src一起使用 -->
  <script src="./script.js"></script>
  ```



#### script setup

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
  - 导入命名空间组件：`import * as 命名空间名 from'xxx'`，在模板中通过`<命名空间名.组件名>`使用
- 具有一些无需导入，无法在子函数中使用的[编译宏命令](###编译宏命令)
- 可以使用顶层 `await`，结果代码会被编译成 `async setup()`
  - 顶层await必须配合Suspense组件使用
  - `<script setup>` 是唯一在调用 await 之后仍可调用组合式函数的地方。编译器会在异步操作之后自动恢复当前的组件实例
- 可以与普通`<script>`一起使用，通常用于以下情景：
  - 声明额外选项
  - 声明模块的具名导出
  - 运行只需要在模块作用域执行一次的副作用，或是创建单例对象



#### CSS

- scoped属性同Vue2

- 样式穿透：

  ```css
  .a :deep(.b) {...}
  // ↓会被编译为
  .a[data-v-f3f3eg9] .b {...}
  ```

- 插槽选择器：影响父组件传递来的插槽

  ```css
  :slotted(div) {...}
  ```

- 全局选择器：影响全局

  ```css
  :global(.red) {...}
  ```

- v-bind：在style标签中，将CSS的值链接到组件的状态

  - 可以直接将变量传入v-bind，也可以使用js表达式（需要使用引号包裹）

    ```css
    color: v-bind(color); // 直接使用变量
    color: v-bind('theme.color'); // js表达式
    ```




### 事件

抛出事件：

- 模板中：`$emit`方法
- setup中：通过emits选项定义事件，通过`context.emit(...)`方法抛出
- `<script setup>`中：`defineEmits`方法



### 内置组件

`<Transition></Transition>`

`<TransitionGroup></TransitionGroup>`

`<KeepAlive></KeepAlive>`



`<Teleport></Teleport>` 将内部的DOM结构传送至其他位置（不影响逻辑关系）

- `to`属性：值为选择器字符串或DOM元素，将模板传送至该元素下
- `disabled`属性：值为true时，禁用传送



`<Suspense></Suspense>` 协调异步依赖，控制所有包裹在该组件中的异步依赖

- 状态：遇到异步依赖时，进入挂起状态；所有依赖都完成后，进入完成状态
  - 进入完成状态后，仅当默认插槽的根节点被替换时，才会回到挂起状态
- 插槽：`#default` 和 `#fallback`两个插槽都只允许一个直接子节点
  - 在挂起状态，显示fallback插槽的内容；在完成状态，显示default插槽的内容
  - 从完成状态回退到挂起状态时，不会立刻显示fallback的内容，在超时后才会切换显示
- `timeout:str|num`属性：加载超时毫秒数
- 事件：
  - `@resolve`：default插槽获取新内容时触发
  - `@pending`：进入挂起状态时触发
  - `@fallback`：fallback插槽显示时触发



组件的嵌套顺序：

```html
<RouterView v-slot="{ Component }">
  <template v-if="Component">
    <Transition mode="out-in">
      <KeepAlive>
        <Suspense>
          <component :is="Component"></component>
        </Suspense>
      </KeepAlive>
    </Transition>
  </template>
</RouterView>
```



### 动态组件&异步组件

动态组件：

`<component></component>` 根据is属性渲染为不同组件

- is属性：可传入 HTML标签名字符串/已注册组件名字符串/组件选项对象
- 相比Vue2：
  - is属性用于原生HTML元素时，其值必须加上前缀`vue:`
  - 动态生成的HTML元素上使用v-model将失效，需要手动设置v-bind和事件

`<KeepAlive></KeepAlive>`



异步组件：

> setup函数被async修饰时，该组件会自动成为异步组件

```js
const AsyncComp = defineAsyncComponent(() => {
  return new Promise((resolve, reject) => {
    resolve(组件) // 加载成功
    reject(reason) // 加载失败
  })
}) // 像使用正常组件一样使用异步组件
```

`defineAsyncComponent(函数或对象)`

- 函数式：方法接收一个函数，该函数需要返回一个Promise
  - `resolve(组件)`表示加载成功，`reject(reason)`表示加载失败
  - ES模块动态导入也返回Promise，在Vite和Webpack中支持：`defineAsyncComponent(()=> import('url'))`
- 对象的选项：
  - `loader: () => import('url')` 加载函数
  - `loadingComponent: 组件` 加载异步组件时使用的组件
  - `delay: 毫秒=200` loadingComponent显示前的延迟时间（防止闪烁）
  - `errorComponent: 组件` 加载失败后展示的组件
  - `timeout: 毫秒=Infinity` 超时时间，超时后算作加载失败



### JSX

与React JSX的区别：

- 使用 HTML 特性 `class` 和 `for` 时，不需要使用 `className` 和 `htmlFor`

- 传递插槽给组件的方式不同：

  ```jsx
  // 默认插槽
  <MyComponent>{() => 'hello'}</MyComponent>
  
  // 具名插槽
  <MyComponent>{{
    default: () => 'default slot',
    foo: () => <div>foo</div>,
    bar: () => [<span>one</span>, <span>two</span>]
  }}</MyComponent>
  ```



### 函数式组件

> 函数式组件没有状态，内部不应该使用ref、reactive等响应式api
>
> 函数式组件接收props，返回vnodes（或jsx）
>
> 函数式组件不会创建组件实例（没有this），也不会触发生命周期钩子

```js
function MyComponent(props, context) {...}
```

可用的选项：props和emits，详细用法见选项式API

```js
MyComponent.props = ['value']
MyComponent.emits = ['click']
```

- 没有定义 props 选项时：
  - 被传入函数的 props 对象会像 attrs 一样会包含所有 attribute
  - prop 的名字不会基于驼峰命名法被处理
  - 透传时只有class、style 和 事件监听器将默认从 attrs 中继承



## 过渡

相比Vue2：

- 单元素过渡的css类名：`v-enter` 改为 `v-enter-from`，`v-leave` 改为 `v-leave-from`
- `<transition>`自定义过渡的属性：`enter-class`改为`enter-from-class`，`leave-class`改为`leave-from-class`
- `<transition>`新增属性：`type:"animation"|"transition"` 同时触发动画和过渡时，哪个优先



## 选项式API

### 选项

> 使用`<script setup/>`时，仅当需要用到`inheritAttrs`选项时，才需要一个额外的`<script>`块声明该选项

- 同Vue2的选项：`name`、`components`、`template`、`data`、`props`、`computed`、`methods`、`inheritAttrs`、`provide`、`inject`、`mixins`、`extends`
- `watch: {属性: 对象/函数/字符串/数组}`：监听属性，与Vue2的区别：
  - 对象式的配置项中新增：`flush、onTrack、onTrigger`，使用参考[组合式API](####watch)
  - 回调函数新增第三个参数`onCleanup`
- `render(this){}`：用[渲染函数](###渲染函数)创建组件，函数需返回VNode
- `compilerOptions:对象`：运行时编译器选项，对象可选属性同[app.config.compilerOptions](https://cn.vuejs.org/api/application.html#app-config-compileroptions)
- `emits:数组或对象`：声明由组件触发的自定义事件，键值同[defineEmits的参数](###编译宏命令)
- `expose:字符串数组`：暴露自定义属性到组件实例上
- `钩子名(){}` 定义钩子函数，其使用参考[组合式API](###生命周期钩子)
  - 生命周期钩子：`beforeCreate`、`created`、`beforeMount`、`mounted`、`beforeUpdate`、`updated`、`beforeUnmount`、`unmouted`
  - keep-alive钩子：`activated` 、`deactivated`
  - 捕获后代组件错误钩子：`errorCaptured(...)`
  - 组件调试钩子：`renderTracked(e)`、`renderTriggered(e)`
  - SSR钩子：`serverPrefetch`



### 组件实例

仅用于选项式API，参考Vue2：`$data`、`$props`、`$el`、`$options`、`$parent`、`$root`、`$slots`、`$refs`、`$attrs`、`$watch()`、`$emit()`、`$forceUpdate()`、`$nextTick()`

- $watch的使用参考watch选项



### TS相关

开启props和emits类型推导：

```ts
import { defineComponent } from 'vue'

// 使用webpack并使用defineComponent时，需要加上PURE注解
export default /*#__PURE__*/ defineComponent({ 
  props: {
    message: String,
  },
  emits: ['change'],
  setup(props) {
    props.message // <-- 类型：string
    emit('change') // <-- 类型检查 / 自动补全
  }
})
```



为emits标注参数类型：

```ts
emits: {
  事件名(payload: { 参数1: string }) {
    return payload.参数1.length > 0 // 执行运行时校验, 返回true表示检验通过
  }
},
```



为计算属性标注类型：直接为相关函数指定返回类型

```ts
computed: {
  greeting(): string {...},
  greetingUppercased: {
    get(): string {...},
    set(newValue: string) {...}
  }
}
```



## 进阶API

### 渲染函数

`h()`：渲染函数，返回一个vnode

```jsx
import { h } from 'vue'

const vnode = h(
  'div', // type
  { id: 'foo', class: 'bar' }, // props
  [] // children
)
```

- 参数一：必须项，HTML标签字符串 或 一个组件
- 参数二：可选项，数据对象
  - attribute 和 property 都能在 prop 中书写，Vue 会自动将分辨
    - 属性名使用前缀：`.` 指定为property、`^` 指定为attribute
    - class和style特性也可使用数组或对象形式
  - 事件监听以 onClick 形式书写，键值为回调函数
- 参数三：可选项，子元素数组，也可写为字符串（表示文本节点）
  - 子元素可以是vnodes或字符串（表示文本节点）



其他函数：

- `render(vnode, 真实dom)`：挂载vnode

- `mergeProps(多个对象):合并后的对象`：合并多个props对象

  - 会将对象的class、style、事件监听函数进行合并

- `cloneVNode(vnode[, props对象])`：克隆vnode，并可以添加额外的prop

- `isVNode(值):bool`

- `resolveComponent(组件名:str):组件|str`：按名称手动解析已注册的组件

  - 必须在渲染函数内调用
  - 若未找到组件，抛出警告，返回组件名字符串

- `withDirectives(vnode, 数组):vnode`：为vnode添加自定义指令

  - 数组：`[指令对象, 值, 参数, {修饰符:true}]`

    ```js
    // <div v-pin:top.animate="200"></div>
    const vnode = withDirectives(h('div'), [
      [pin, 200, 'top', { animate: true }]
    ])
    ```

- `resolveDirective(指令名:str):指令对象|und`：按名称手动解析已注册的指令

  - 若未找到指令，抛出警告，返回undefined

- `withModifiers(回调, 修饰符字符串数组):回调`：为事件处理函数添加v-on修饰符

  ```js
  const vnode = h('button', {
    // 等价于 v-on.stop.prevent
    onClick: withModifiers(() => {...}, ['stop', 'prevent'])
  })
  ```



### TS工具类型

- `PropType<T>`：用于在用运行时 props 声明时给一个 prop 标注更复杂的类型定义

  ```ts
  import { PropType } from 'vue'
  
  interface Book {...}
  
  export default {
    props: {
      book: Object as PropType<Book>, // 提供一个比 Object 更具体的类型
      callback: Function as PropType<(id: number) => void> // 也可以标记函数
    }
  }
  ```

- `ComponentCustomProperties`：扩展全局属性

  ```ts
  declare module 'vue' {
    interface ComponentCustomProperties {
      $http: typeof axios
      $translate: (key: string) => string
    }
  }
  ```

- `ComponentCustomOptions`：扩展全局选项

  ```ts
  declare module 'vue' {
    interface ComponentCustomOptions {
  	beforeRouteEnter?(to: Route, from: Route, next: () => void): void
    }
  }
  ```

- `ComponentCustomProps`：扩展全局TSX props

  ```ts
  declare module 'vue' {
    interface ComponentCustomProps {
      hello?: string
    }
  }
  ```

  ```tsx
  // 现在即使没有在组件选项上定义过 hello 这个 prop 也依然能通过类型检查了
  <MyComponent hello="world" />
  ```

- `CSSProperties`：扩展在样式属性绑定上允许的值的类型

  ```ts
  declare module 'vue' {
    interface CSSProperties {
      [key: `--${string}`]: string
    }
  }
  ```

  ```html
  <div :style="{ '--bg-color': 'blue' }">
  ```

- 对于上述四个扩展：

  - 类型扩展必须被放置在一个模块 `.ts` 或 `.d.ts` 文件中
    - 模块文件：包含import或export，即使是`export{}`，否则将覆盖原始类型，而非扩展
  - 该模块文件需被包含在tsconfig.json中
    - 对于库或插件，该文件需在package.json的types属性中列出

## 插件

插件可以是包含install方法的一个对象，也可以是install函数本身

- install方法：`install(app, options){}`
  - 通过 `app.component()` 和 `app.directive()` 注册一到多个全局组件或自定义指令
  - 通过 `app.provide()` 使一个资源可被注入进整个应用
  - 向 `app.config.globalProperties` 中添加一些全局实例属性或方法

应用插件：`app.use(插件[, {选项}])` 

- 在应用实例化之后调用
- 选项会传递给install方法的options对象



### Vue-Router

#### 相比Vue2的变化

创建：

- `new Router({选项})` 变成 `createRouter({选项})`

- 由history选项替代mode选项

  ```js
  createRouter({
    history: createWebHistory(), // history模式
    // history: createWebHashHistory(), // hash模式
    // history: createMemoryHistory(), // abstract模式
    routes: [],
  })
  ```

- base选项改为由history函数的第一个参数传递：

  ```js
  history: createWebHistory('/base-directory/')
  ```

- scrollBehavior返回的对象中的属性名变更：x改为left，y改为top

- 移除fallback选项，移除`*`路由（使用自定义regex参数定义）



router实例：

- onReady属性变更为isReady：

  ```js
  router.onReady(onSuccess, onError)
  // 替换成
  router.isReady().then(onSuccess).catch(onError)
  ```

- match方法被合并到resolve方法中

  - 待补充

- 删除`getMatchedComponents`方法，可从`currentRoute.value.matched`中获取匹配组件

- 删除app属性



$route路由信息对象：

- 删除parent属性，可通过`$route.matched[this.$route.matched.length - 2]` 获取
- `params`、`query`和 `hash` 中的解码值现在都是一致的



内置组件：

- `transition` 和 `keep-alive` 现在必须通过 `v-slot` 在 `RouterView` 内部使用：

  ```vue
  <router-view v-slot="{ Component }">
    <transition>
      <keep-alive>
        <component :is="Component" />
      </keep-alive>
    </transition>
  </router-view>
  ```

- 必须通过v-slot将模板传递给router-view：

  ```html
  <router-view>
    <p>In Vue Router 3, I render inside the route component</p>
  </router-view>
  <!-- 替换成 -->
  <router-view v-slot="{ Component }">
    <component :is="Component">
      <p>In Vue Router 3, I render inside the route component</p>
    </component>
  </router-view>
  ```

- 删除 `router-link` 的append、event、tag、exact属性

  ```html
  <!-- 原 -->
  <router-link to="/about" tag="span" event="dblclick">About Us</router-link>
  <!-- 替代方案：使用插槽定制化 -->
  <router-link to="/about" custom v-slot="{ navigate }">
    <span @click="navigate" @keypress.enter="navigate" role="link">About Us</span>
  </router-link>
  ```



ts相关：类型重命名

| vue-router@3 | vue-router@4            |
| ------------ | ----------------------- |
| RouteConfig  | RouteRecordRaw          |
| Location     | RouteLocation           |
| Route        | RouteLocationNormalized |



#### 组合式API

- `useRouter(): router`：返回router路由实例

- `useRoute(): route`：返回当前路由元信息route，route是响应式对象

- `onBeforeRouteLeave( (to, from)=>bool )`：离开前组件路由守卫

- `onBeforeRouteUpdate( (to, from)=>bool )`：路由变更但组件被复用时的路由守卫

- `uselink(props)`：返回RouterLink组件v-slot插槽的内容

  ```js
  import { RouterLink, useLink } from 'vue-router'
  export default {
    props: {
      // 如果使用 TypeScript，添加 @ts-ignore
      ...RouterLink.props,
      inactiveClass: String,
    },
    setup(props) {
      const { route, href, isActive, isExactActive, navigate } = useLink(props)
    },
  }
  ```



#### 动态路由

添加路由：`router.addRoute(父路由名?:str, 路由对象): 函数`

- 参数与返回值：

  - 若传入父路由名，表示添加为其子路由；路由对象同createRouter - routes选项内的对象
  - 返回的函数在调用后会删除该路由

- 新增的路由若于当前位置匹配，需用 push 或 replace 方法手动导航，才能显示新路由

  ```js
  router.addRoute({ path: '/about', component: About })
  router.replace(router.currentRoute.value.fullPath)
  ```

- 若是在导航路由守卫中添加或删除路由，可以直接返回新位置来重定向：

  ```js
  router.beforeEach(to => {
    if (!hasNecessaryRoute(to)) {
      router.addRoute(generateRoute(to))
      return to.fullPath // 触发重定向
    }
  })
  ```

- 添加路由时，若已有同名路由，会删除原有的同名路由



删除路由：`router.removeRoute(路由名:str)`

- 删除路由时会删除所有别名路由和子路由



查看现有路由：

- `router.hasRoute(name: str): bool`：检查路由是否存在
- `router.getRoutes(): RouteRecord[]`：返回包含所有路由记录的数组



### Pinia

#### 基本使用

main.ts中创建pinia实例（根store）

```js
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'

const pinia = createPinia()
const app = createApp(App)

app.use(pinia)
app.mount('#app')
```

定义store：

```js
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('todos', {
  state: () => ({ count: 0 }),
  getters: {
    double: (state) => state.count * 2,
  },
  actions: {
    increment() {
      this.count++
    },
  },
})
```

使用store：

```js
import { useCounterStore } from '@/stores/counter'

const counter = useCounterStore()
counter.count++ // 操作state
counter.$patch({ count: counter.count + 1 }) // 操作state
counter.increment() // 使用 action 代替
```



#### Store

store实例：是一个响应式对象

- 定义的state属性、actions内的函数、getters的属性都会暴露在store实例上
- `storeToRefs(store实例):对象`：返回一个普通对象，为store的响应式属性（state、 getter 和 plugin 添加的 state 属性）包装为ref作为该对象的属性



定义store：`defineStore(id:str, 函数|对象): 返回store实例的函数`

- 函数式：无参函数，与setup函数类似，需返回一个对象，根据对象的属性定义store：
  - 若键值为ref，该属性归为state属性
  - 若键值为函数，该属性归为actions属性
  - 若键值为computed的返回值，该属性归为getters属性
- 对象式：传入带有 `state`、`actions` 与 `getters` 属性的对象
  - state属性的值为箭头函数，返回一个对象
  - actions属性的值为对象，键名为action名，键值为函数
  - getters属性的值为对象，键名为getter属性名，键值为**箭头函数**，函数接收state参数
    - 需要接收外部参数时，getter函数也可以返回一个函数，由该函数接收参数。此时getter不再被缓存（可以通过闭包自行缓存结果）
    - getter在使用上与state中的属性完全相同
- 函数式和对象式内部函数都可以通过this访问store实例



#### State

访问state：

- `store实例.属性名`：state中属性可通过store实例访问，可对属性进行读写

- `store实例.$patch(对象或函数)`：变更state

  - 对象式：`{属性名: 新值}`，作为补丁对象修改属性

  - 函数式：

    ```js
    store.$patch((state) => {
      // 修改state
      state.hasChanged = true
    })
    ```

    

重置state为初始值：`store实例.$reset()`



选项式API映射：`...mapState(defineStore的返回值, 数组或对象)` 该方法也可映射getter

- 添加到组件的computed选项中，添加后可以直接使用映射的属性
- 数组式：`['属性名']`
- 对象式：`{属性名: 'state中的属性名', 属性名2: (store)=>any }`
- 通过mapState映射的属性为只读属性，若要映射可修改属性，改用mapWritableState，此时无法再对象式中传递函数



订阅state：

- 方法一：在组件中 `watch(pinia.state, (state)=>{}, {deep: true})`
- 方法二：`store实例.$subscribe( (mutation, state)=>{} )`
  - 比起 watch，使用 $subscribe 在 patch 后只触发一次
  - mutation的属性：
    - `type: 'direct' | 'patch object' | 'patch function'`
    - `storeId: str`
    - `payload: 传递给$patch方法的补丁对象`
  - 若要在组件卸载后继续监听，将`{detached:true}`作为第二个参数传入$subscribe



#### Action

> Pinia舍弃了mutation
>
> action可以是异步的

选项式API映射：`...mapActions(defineStore的返回值, 数组或对象)`

- 添加到组件的 methods 选项中，添加后可以直接使用映射的属性
- 数组式：`['属性名']`
- 对象式：`{属性名: 'action名'}`



订阅action：`store实例.$onAction({name,store,args,after,onError}=>{}):函数`

- 返回值：调用后取消订阅
- 参数：
  - `name`：action名称
  - `args`：传递给action的参数数组
  - `after( (result)=>{} )`：在action返回或解决后调用回调
  - `onError( (error)=>{} )`：在action抛出或拒绝后调用回调
- 若要在组件卸载后仍保留订阅，将true作为第二个参数传入$onAction



#### 插件

使用插件：`pinia.use(插件函数)`，会在store被激活前调用插件函数



插件是一个函数：

- 插件函数接收context参数，context的属性有：
  - `pinia`：pinia实例
  - `app`：当前应用（仅Vue3）
  - `store`：要扩展的store实例
  - `options`：store定义时传入的选项
- 函数可以返回一个对象，用于为store添加属性（类似defineStore的函数式返回值）



插件常用功能：

- 添加state：

  ```js
  store.$state.hasError = hasError // 添加state
  store.hasError = toRef(store.$state, 'hasError') // 允许通过store.hasError访问
  ```

- 在插件中订阅state和action

- 添加新的选项：通过options.选项获取新选项信息

- ...



ts类型支持：

- 插件类型标注：

  ```ts
  import { PiniaPluginContext } from 'pinia'
  
  export function myPiniaPlugin(context: PiniaPluginContext) {}
  ```

- 为store添加新属性时：扩展PiniaCustomProperties接口

  ```ts
  import 'pinia'
  
  declare module 'pinia' {
    export interface PiniaCustomProperties<Id, S, G, A> {
      $options: {
        id: Id
        state?: () => S
        getters?: G
        actions?: A
      }
    }
  }
  ```

  - 泛型字母的含义：`S` State、`G` Getters、`A` Actions、`SS` Setup Store / Store

- 为state添加类型时：扩展PiniaCustomStateProperties接口，只接受S泛型

- 为新选项添加类型：扩展DefineStoreOptionsBase接口，接收 S 和 Store 泛型



## 常见问题

### TS找不到vue模块的类型声明

在全局ts类型声明文件中添加：

```ts
declare module '*.vue' {
    import { defineComponent } from 'vue';
    const Component: ReturnType<typeof defineComponent>;
    export default Component;
}
```

