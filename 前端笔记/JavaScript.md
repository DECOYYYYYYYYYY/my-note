# JavaScript

## 基本

### 数据类型

- 原始类型：`String、Number、Boolean、null、undefined、Symbol、BigInt`
- 引用类型：`Object(Object、Function、Array)`

#### 数值

- BigInt 类型：`123456789n` 支持`+ - * ** /`
- 除法会舍去小数部分，返回整数；不能与普通数值进行混合运算
- `0o767 0b1011101`二进制和八进制表示法
- `1_000_000` 数值分隔符
- `Number.EPSILON` JavaScript 能够表示的最小精度
- `Number.isFinite(变量) / Number.isNaN(...) / Number.isInteger(...) / Number.isSafeInteger(...)`
- `Number.parseInt('字符串') / Number.parseFloat(...)`
- `Math.trunc(数值)` 去除小数部分
- `Math.sign(数值)` 根据数值符号返回 1、0、-0、-1、NaN
- `Math.sqrt(数值) / Math.cbrt(数值)` 计算一个数的 平方根/立方根
- `Math.fround(数值)` 返回数值最接近的单精度(32 位)浮点值表示

### 操作符

- 运算操作符：`+ - * / % ++ -- **`

- 位操作符：
  - 按位非：`~` 用于数值前，返回数值的一补数(所有位取反，相当于对数值取相反数并减 1）
  - 按位与/或/异或：`& | ^` 需要左右两个操作数，对两个数的每一位进行逻辑与/或/异或操作，返回数值
  - 左移：`<<` 左操作数表示数值，右操作数表示移位位数。将数值的所有位向左移动，用 0 补充空位
  - 有/无符号右移：`>> >>>` 将数值的 除符号位外所有位/所有位 向右移动指定位数，用 0 补充空位
  
- 布尔操作符：`! && ||`

- 关系操作符：`< > <= >= == != === !==`

- 赋值操作符：`= *= /= %= += -= **= <<= >>= >>>= ||= &&= ??=`
  
  - 逻辑赋值运算符：`x ||= y`等同于`x || (x = y)`
  
- 条件操作符:`条件表达式 ? 表达式1：表达式2`

- 可选链操作符：`const firstName = message?.body?.user?.firstName || 'default'` 左结合，当左侧对象为 null 或 undefined 时，不再向下运算(短路)，返回 undefined。
  - 右侧不得为十进制数值、模板字符串。不得用于构造函数、用于 super 右侧。
  - 检测方法是否有定义：`if (myForm.checkValidity?.() === false) {}`
  - 正则匹配判断：`"#C0FFEE".match(/#([A-Z]+)/i)?.[1];`
  
- 空值合并操作符：`const name = user.name ?? 'Chen'` 当左侧值为 null 或 undefined 时，返回右侧值

- 优先级：
	
	> 括号 > 成员访问、带参数的 new、函数调用、可选链 > 无参数 new > 后置递增减 > 一元操作符 > 指数操作符 > 乘性操作符 > 移位操作符 > 关系操作符、in、instanceof > 相等操作符 > 按位与 > 按位异或 > 按位或 > 逻辑与 > 逻辑或 > 空值合并 > 条件操作符 > 赋值操作符 > yield > 逗号操作符


### 语句

- if 语句：`if(表达式)语句1或{语句块1｝ [ else 语句2或{语句块2｝ ] `

- do-while 语句：`do 语句或{循环体} while(表达式)；`

- while 语句：`while(表达式) 语句或{循环体}`

- for 语句：`for([初始化表达式]；[条件表达式]；[循环后表达式]) 语句或循环体`

- for-in 语句：枚举可以通过对象访问的非符号可枚举属性的属性名(字符串型)
  
  - `for(变量 in 表达式) 语句或循环体`
  
- for-of 语句：枚举可迭代对象的元素，按 next()方法产生值的顺序迭代元素
  - `for(变量 of 表达式) 语句或循环体`
  - `...可迭代对象`转换为逗号分隔的参数序列
  
- 标签语句：给语句添加标签。`标签：语句`

- break 和 continue 语句：`break [标签]；continue [标签]；`

- with 语句：`with(表达式) 语句或循环体`

- switch 语句：`switch(条件表达式)｛case 表达式1：语句块1 break；...default：语句块}`

- try/catch/finally 语句：`try{} catch(err){} finally{}`

- void语句：返回undefined的原始值。`void 表达式` 或 `void(表达式)`

  1. 立即执行函数的`()`可以用`void`替代

     ```javascript
     (function fn(){...})() // 普通的立即执行函数
     void function fn(){...}() // 用void表示
     ```

  2. 判断值是否为undefined

     ```javascript
     if(txt === undefined) // 存在名为undefined的变量时，会判断错误
     if(txt === void(0))
     ```

### 作用域

#### LHS与RHS

- LHS(Left-hand Side）：赋值操作的左侧，试图找到变量的容器本身，以为其赋值
- RHS(Right-hand Side）：赋值操作的右侧，仅查找并获取它的值

注意：

- `function foo(){}`不能等效于`var foo = function(){}`，前者没有LHS步骤。

#### 作用域

- for 循环设置变量的部分是一个父作用域，循环体内部是一个单独的子作用域
- 暂时性死区：代码块内，使用 let、const 命令声明变量之前，该变量都是不可用的。(即使外部已用 var 声明该变量)
- `GlobalThis` 获取顶层对象(防止模块中在全局环境 this 不指向顶层对象)

### 变量

- 预解析：所有的 var 和 function 声明会被提升到当前函数/全局作用域的顶部
- 声明：

  - var：声明的变量被添加到最接近的函数/全局上下文，存在变量提升
  - let：不能声明已被 let、const 声明的当前作用域的变量，块级作用域，不存在变量提升。let、const、class 在全局作用域中声明的变量不会成为 Global 对象的属性。
  - const：具有 let 的特点，外加：声明时必须同时初始化，常量的值不能修改
- 类型判断：

  - `typeof 变量`返回数据类型的字符串表达，小写
  - `实例 instanceof 构造函数名`
  - `Object.prototype.toString.call(变量)`返回"[object 大驼峰类型名]"
- 解构赋值：用于可迭代对象，根据迭代协议进行解构
  - 扩展运算符：取出对象/数组中的可遍历属性
    - 浅复制对象：`let obj2 = {...{a:1, b:2}};`
    - 浅复制数组：`let arr2 = [...[1, 2]];`
  - 接收rest参数：`(...args)=>{}` args为数组，变量名不作限制
  - 数组解构赋值：
    - `let [a, b, c=10] = [1, 2, 3];`
    - 等号右边需要是可遍历结构
    - 解构不成功则变量值为 undefined
    - 仅当将要赋的值严格等于 undefined 时，被赋值变量的默认值才会生效
  - 对象解构赋值：

    - `let { bar = 'abc', foo:f} = { foo:'aaa', bar: 'bbb'}`
    - 'aaa'会被赋值给 f(变量部分) 而非 foo(模式部分)
    - 对象的解构赋值可以取到继承的属性
    - `{x} = {x:1}` 错误写法，大括号写在行首，引擎会将{x}解释成代码块。在语句外部加()避免
    - 等号右边若为字符串、数值、布尔，会尝试转换为对象再赋值
    - 解构赋值内部仅在赋值语句的非模式部分可以使用圆括号

### 事件循环

事件循环的过程：

1. 若存在宏任务，取一个宏任务执行，执行完毕后，进入下一步。
2. 取一个微任务执行，执行完毕后再取一个微任务，直到微任务队列为空，进入下一步。
3. 更新UI渲染。回到第一步。

- 宏任务：script（全局任务）,setTimeout, setInterval, setImmediate, I/O, UI rendering

- 微任务：process.nextTick, Promise, Object.observer…

### 原型

- 原型对象：创建函数时，自动生成一个对应的原型对象。自定义构造函数的原型对象默认只有 constructor 属性，指回自定义构造函数。
  - Object()函数的原型对象的原型对象是 null
- 显式原型属性：即每个函数都有的 prototype 属性，在函数创建时自动添加，指向原型对象
- 隐式原型属性：即实例的`[[Prototype]]`特性，实例创建时该特性自动赋值为其构造函数的原型对象，可用实例对象的`_proto_`属性访问
  - 该属性为非标准属性，建议改用 Object.getPrototypeOf()和 Object.setPrototypeOf()操作`[[Prototype]]`特性
- 原型链：任何原型对象也有其原型对象。从实例开始沿着隐式原型属性层层向上，直到 null 为止的链条，称为原型链
  - 读取对象的属性时，会沿着原型链层层查找，若找不到，则返回 undefined；设置对象的属性则直接设置在该对象上

## 字符串

### 字符编码

- `\u{41}\u{42}\u{20BB7}` 可以通过大括号表示非四位的十六进制数值

### 字符串

构造函数：`new String(值)` 值会自动通过 `String()`转换后存于实例中

特性：

- 字符串若要跨行，可在每行末端加`\`后换行
- 字符串可按照码点迭代（可识别大于`0xFFFF`的码点）

实例属性：`length` 表示字符串中码元的数量

静态方法：

- `fromCharCode(多个数值)` 每个数值对应一个码元，按顺序拼接为字符串返回
- `fromCodePoint(多个数值)` 每个数值对应一个码点，按顺序拼接为字符串返回
- `` raw`模板字符串` `` 返回原始字符串
- `raw({raw:[多个字符串],多个值})`
  - `raw({raw:['name:', ', age:', ', sex:']}, 'CHEN'.toLowerCase(), 17+2, 'male')`

实例方法：

- `valueOf()、toLocaleString()、toString()` 

  - 返回对象的原始字符串值

- `charAt(索引号) / str[索引号]` 

  - 返回指定索引位置的码元对应的字符

- `charCodeAt(索引值)` 

  - 返回指定索引位置的码元的值的十进制整数表示

- `codePointAt(索引值)` 
  
  - 返回指定索引位置的码元所在码点的值的十进制整数表示
    - 若指定码元不是码点的开头，会返回错误的码点值；可以解构字符串再遍历
  
- `normalize(“NFD”/“NFC”/“NFKD”/“NFKC”)` 
  
  - 返回规范化后的字符串，可选四种规范化形式之一
    - 有些 Unicode 字符具有多个编码，通过采用相同规范化形式将这些编码转换为一致的编码。
  
- `concat(多个字符串)` 

  - 返回拼接后的字符串(将参数按顺序追加在实例字符串后方)，等效于+

- `substr(start, 长度) / slice(start, end) / substring(start, end)` 
  
  - 返回从指定索引截取的子串
    - 三个方法都可以省略第二个参数，若省略则取到末尾
    - substr 把 start 负参数转换为字符串长度加 start、把长度负参数转换为 0；slice 转换所有负参数为字符串长度加该值；substring 转换所有负参数为 0。所有负参数若转换后仍为负，则转换为 0
    - slice 第二个参数小于第一个时，返回“”；substring 自动取较小的参数为 start，较大的为 end
  
- `indexOf(‘子串’ [, start]) / .lastIndexOf(...)` 

  - 返回从前往后/从后往前第一次匹配到子串的位置，若匹配失败，返回-1

- `startsWith(‘子串’ [, start])/ endsWith(‘子串’ [, end]) ` 

  - 返回布尔，表示字符串是否以子串开头/以子串结尾。end 相当于将字符串末尾定于此处

- `includes(‘子串’ [, start])` 

  - 返回布尔，表示字符串是否包含子串

- `trim()/ trimStart()/ trimEnd()` 
  
  - 返回删除前后所有/前面所有/后面所有空格符的字符串副本
    - trimLeft 和 trimRight 方法分别为 trimStart 和 trimEnd 方法的别名
  
- `repeat(整数)` 

  - 将字符串复制指定次，返回拼接所有副本后的字符串

- `padStart(长度[, ‘填充字符串’=' '])/ padEnd(...)` 
  
  - 返回新字符串，以填充字符串在调用字符串 前/后 填充至指定长度
    
    ```javascript
    '12'.padStart(10, 'YYYY-MM-DD') // "YYYY-MM-12"
    '09-12'.padStart(10, 'YYYY-MM-DD') // "YYYY-09-12"
    ```
  
- `toLowerCase()/ toUpperCase()` 

  - 返回按照通用规则转换为小写/大写后的字符串

- `toLocaleLowerCase()/ toLocaleUpperCase()` 

  - 返回按照各语言规则转换为大写/小写后的字符串(推荐)

- `match(字符串/正则表达式)` 

  - 返回值同 RegExp 的实例方法 exec()

- `matchAll(正则表达式)`

  - 返回值迭代器，包含多个匹配项的匹配结果(参考 match())

- `search(字符串/正则表达式)` 

  - 返回第一个匹配的索引，若没有搜索到返回-1

- `replace(字符串/正则表达式，‘替换字符串’/ fn())` 
  
  - 返回修改过的字符串，用替换字符串/fn 的返回值替换匹配到的子串
    - 替换字符串中可用特殊的字符序列表示指定值：\$\$【\$】、\$&【匹配整个模式的字符串】、\$’【匹配的子串之前的字符串】、\$`【匹配的子串之后的字符串】、\$n【匹配第 n 个捕获组的字符串，n 为 0-9】、\$nn【匹配第 nn 个捕获组的字符串】
    - fn 参数：与整个模式【一个匹配项】/捕获组【多个匹配项】匹配的字符串、匹配项在字符串中的起始位置、源字符串
  
- `replaceAll(字符串/正则表达式，‘替换字符串’/ fn())` 

  - 返回修改过的字符串，替换全部

- `split(字符串/正则表达式，数组长度上限)` 

  - 返回将字符串按分隔符分隔后的字符串数组

- `localeCompare(字符串)` 按字母表顺序逐位比较，排在后面的字母大【各语言有对应的字母表】
  
  - 若字符串小于参数字符串，返回负值【通常为-1】；等于返回 0；大于返回正值【通常为 1】

### 模板字符串

`` `Hello ${userName}!` ``

- 模板字符串保留空格、缩进、换行
- 大括号内部可以放入任意的 JS 表达式，会将值转换为字符串

标签函数：`` 函数名`模板字符串` `` 是一种特殊的函数调用形式

- 函数接收到的参数依次为：由插值记号分隔的模板组成的数组、第一个表达式的值、第二个表达式的值。。。
  - 第一个参数有 raw 属性，获取由原始字符串组成的数组

### 正则表达式

#### 位置匹配符

- `^`：匹配字符串的起始位置。若开启多行匹配模式，也将匹配换行符后紧跟的位置
- `$`：匹配字符串的结束位置。若开启多行匹配模式，也将匹配换行符前紧跟的位置
- `\b`：匹配单词边界（“字”与非“字”之间的位置）
- `\B`：匹配非单词边界



#### 元字符

- `.`：匹配除换行符外，任意单个字符。若开启dotAll模式，也将匹配换行符
- `\w`：匹配一个“字”字符（字母、数字、下划线），等价于 `[A-z0-9_]`
- `\W`：所有 \w 不匹配的单个字符
- `\s`：单个空白符，包括制表符、换行符等。
  - 等价于`[\f\n\r\t\v\u0020\u00a0\u1680\u180e\u2000-\u200a\u2028\u2029\u202f\u205f\u3000\ufeff]`。但经测试 \s 不匹配 \u180e
- `\S`：所有 \s 不匹配的单个字符
- `\d`：匹配一个数字字符，等价于 `[0-9]`
- `\D`：所有 \d 不匹配的单个字符



#### 限定符

> 贪婪匹配：`* + ? {}` 限定符默认为贪婪匹配（匹配尽量多的字符），在它们后面加 `?` ，将改为非贪婪匹配（匹配尽量少的字符）

- `*`：匹配前面的字符 0 次或多次。等价于 `{0,}`
- `+`：匹配前面的字符 1 次或多次。等价于 `{1,}`
- `?`：匹配前面的字符 0 次或 1 次。等价于 `{0,1}`
- `{n}`：匹配前面的字符 n 次
- `{n,}`：匹配前面的字符最少 n 次
- `{n,m}`：匹配前面的字符最少 n 次最多 m 次。



#### 字符集合

> 字符集合匹配单个字符，匹配范围在 [] 内部指定

- 匹配方括号内所有字符：`[aeiou]`、`[a-z]`、`[A-z]`、`[0-9]`
- 匹配除方括号内所有字符：`[^aeiou]`、`[^a-z0-9]`
- 匹配中文：`[\u4e00-\u9fa5]`
- 匹配双字节字符：`[^\x00-\xff]`



#### 逻辑或

`a|b`：用于两个字符之间，表示匹配字符a或字符b

```js
/abc|d/.test('abd') // true
```



#### 特殊字符

- 转义：`{ } [ ] ( ) \ ^ $ | ? * + .`   上述字符前可加 `\` 进行转义
- `\cX`：匹配一个控制符（X 是处于 A 到 Z 之间的字符）
- `[\b]`：匹配一个退格符
- `\f`：匹配一个换页符 (U+000C)
- `\n`：匹配一个换行符 (U+000A)
- `\r`：匹配一个回车符 (U+000D)
- `\t`：匹配一个水平制表符 (U+0009)
- `\v`：匹配一个垂直制表符 (U+000B)
- `\0`：匹配一个 NULL（U+0000）字符
- `\0<digits>`：匹配一个八进制数表示的字符
- `\xhh`：匹配一个两位十六进制数（\x00-\xFF）表示的字符
- `\uhhhh`：匹配一个四位十六进制数表示的 UTF-16 代码单元
- `\u{hhhh}` 或 `\u{hhhhh}`：需开启 Unicode 匹配模式，匹配一个十六进制数表示的Unicode字符



#### 分组

> 括号整体可视为一个字符，以使用限定符、逻辑或等

- `(pattern)`：捕获括号。根据模式匹配，并记住匹配项
- `(?<组名>pattern)`：命名捕获括号，作用同捕获括号，但可以指定组名
- `(?:pattern)`：非捕获括号。根据模式匹配但不记住匹配项
- `(?=pattern)`：先行断言。当前位置**之后**的字符序列必须**能**与pattern匹配
- `(?!pattern)`：先行否定断言。当前位置**之后**的字符序列必须**不能**与pattern匹配
- `(?<=pattern)`：后行断言。当前位置**之前**的字符序列必须**能**与pattern匹配
- `(?<!pattern)`：后行否定断言。当前位置**之前**的字符序列必须**不能**与pattern匹配
- `\n`：反向引用。返回第 n（n≥1）个捕获括号的匹配项

## 数组

创建：`new Array() or []`

数组空位：`[, , , ]`空位处理规则不统一，避免出现空位。

### 静态方法

- `from(类数组对象或可迭代对象)` 返回新数组实例
- `of(多个变量)` 返回新数组实例，将变量按顺序作为其元素
- `isArray(变量)` 返回布尔，判断变量是否为数组

### 实例方法

获取信息：

- `at(整数)` 
  - 返回对应位置的成员，支持负索引
- `keys()` / `values()` / `entries()` 
  - 返回 数组索引/数组元素/索引值对 的迭代器
- `includes(变量)` 
  - 返回布尔，检测变量是否存在于数组
- `indexOf(变量)` / `lastIndexOf(变量)`
  - 返回变量在数组中 第一次/最后一次 出现时的索引值，不存在则返回-1
- `join("分隔符"=',')` 
  - 返回字符串，对所有元素调用 toString 后用分隔符拼接



简单加工：

- `push(多个变量)` / `unshift(...)`
  - 返回修改后数组的长度，在 数组末尾/头部 按原顺序添加元素
- `pop()` / `shift()` 
  - 返回删除的元素(修改原数组)，删除数组 最后/最前 一个元素
- `slice([start[, end]])` 
  - 返回新数组，包含实例指定区间的元素
- `splice(start, 删除个数[, 多个变量])`
  - 返回被删除元素组成的数组(修改原数组)，删除数组元素，并在删除处添加元素
- `fill(元素[, start[, end]])` 
  - 返回修改后的数组，由元素填充（支持负索引，忽略反向索引）
- `copyWithin(索引值[, start[, end]])`
  - 返回修改后的数组，将实例[start, end)区间的元素复制到索引处；支持负索引，忽略反向索引
- `concat(多个变量)` 
  - 返回新数组，按顺序添加参数(自动解构数组参数)到数组末尾
- `reverse()` 
  - 返回修改后的数组，翻转数组
- `sort([(a, b) => 值])` 
  - 返回修改后的数组，排序数组
    - 若不写参数，根据每一个元素调用 String()的结果字符串升序排列
    - 若比较函数返回负值，则 a 排在 b 之前
- `flat([整数n])` 
  - 返回新数组，将数组转换为低 n 维(默认为 1）的数组
- `flatMap(...)` 
  - 相当于 `arr.map(...).flat(1)`



迭代：

- `find((el, i, arr) => {}[, thisArg])` / `findIndex(...)` / `some(...)`
  - 遍历元素执行函数，若子函数返回真值，结束遍历并返回 该元素/该元素的索引/true，找不到返回 undefined/-1/false
  - `findLast()` / `findLastIndex()`：使用方法同上，从右往左迭代
- `every(同find)`
  - 遍历元素执行函数，若对于每个元素 子函数都返回真值，返回 true，否则返回 false
- `forEach(同find)` 
  - 返回 undefined，遍历元素执行函数
- `map(同find)` / `filter(...)`
  - 遍历元素执行函数，返回由 检测函数返回结果/检测函数返回 true 的项 组成的新数组
- `reduce(fn(acc, current, i, arr){}[, 初始值])` / `reduceRight(...)`
  - 从左向右/从右向左 遍历元素执行函数(若不传初始值，从第二项开始迭代，第一项作为 acc），最后一次的返回值作为返回值
  - acc 为累积器，在第一次调用时为初始值，之后为前一次调用的返回值。current 为当前项

## 对象

- 属性名表达式

### 内部特性

内部特性用来描述属性或实例的特征。不能在 JS 中直接访问，规范中会用两个中括号将其名称括起来

- `[[Prototype]]` 指向该实例对象的构造函数的原型对象
- `[[Configurable]]` 描述属性，值为布尔值，默认为 true
  - 表示属性是否可以通过 delete 删除并重新定义，其特性是否可以被修改，是否可以被改为访问器/数据属性
- `[[Enumerable]]` 属性，布尔值，默认为 true，表示属性是否可枚举，即是否可以通过 for-in 循环返回(符号属性除外)
- `[[Writable]]` 属性，布尔值，默认为 true，表示属性的值是否可被修改
- `[[Value]]` 描述属性，包含属性实际的值，默认为 undefined
- `[[Get]]` 获取函数，在读取属性时调用，默认为 undefined
- `[[Set]]` 设置函数，在写入属性时调用，默认为 undefined

数据属性由以下特性描述：`[[Configurable]]、[[Enumerable]]、[[Writable]]、[[Value]]`

访问器属性由以下特性描述(后两者非必须)：`[[Configurable]]、[[Enumerable]]、[[Get]]、[[Set]]`

### 静态方法

- `fromEntries(可迭代对象)` 将键值对列表(如[["name", "chen"]])转换为一个对象，返回新对象
- `keys(对象) / values(对象) / entries(对象)`
  - 返回包含该对象自身可枚举 属性名的字符串数组/属性值的字符串数组/键值对数组
- `is(变量1, 变量2)` 返回布尔，判断两个变量是否为同一个值
- `isExtensible(对象) / isSealed(对象) / isFrozen(对象)`
  - 返回布尔，判断对象是否 可扩展/被密封/被冻结
- `preventExtensions(对象)` 使对象不可扩展(无法添加属性)，返回该对象
- `seal(对象)`
  - 密封对象(不可扩展，`[[Configurable]]`特性为 false，可修改值)，返回该对象
- `freeze(对象)`
  - 冻结对象(不可扩展，被密封，`[[Writable]]`特性为 false），返回该对象
- `defineProperty(对象, "属性名", 属性描述对象) / defineProperties(对象, {属性名: 属性描述对象, ...})`
  - 在对象上定义新属性，或修改现有属性，返回此对象
  - 属性描述对象的属性：
    - value：值(默认为 undefined）
    - enumerable：布尔值(默认 false）、writable：...、configurable：...
    - get(){}、set(newValue){}
- `getOwnPropertyNames(对象) / getOwnPropertySymbols(...)`
  - 返回包含该对象自身实例属性 非符号/符号 属性名的数组
- `getOwnPropertyDescriptor(对象, "属性名") / getOwnPropertyDescriptors(对象)`
  - 返回 对象指定属性的属性描述对象/{属性名: 属性描述对象, ...}
- `assign(目标对象, 多个源对象)`
  - 将每个源对象中可枚举的自有属性复制到目标对象，返回目标对象(浅拷贝)
- `create(原型对象[, {属性名: 属性描述对象, ...}])`
  - 创建新对象，指定原型并返回新对象，属性描述对象见 defineProperty()
- `setPrototypeOf(对象, 原型对象) / getPrototypeOf(对象)`
  - 设置/获取实例的对象；修改原型对象的性能不佳，使用 create()和 assign()替代

### 实例方法

- `hasOwnProperty("属性名")` 返回布尔，判断对象自身是否有指定的属性
- `isPrototypeOf(对象)` 返回布尔，判断调用对象是否在指定对象的原型链上
- `propertyIsEnumerable("属性名")` 返回布尔，判断对象自身指定属性是否可枚举

## 函数

- 函数的 length 属性：未设定默认值的参数个数
- 函数的 name 属性：函数名的字符串形式
- rest 参数：`function add(...变量名) {}` 将多余的参数放入数组赋值给该变量
- 在函数参数中使用默认值、解构赋值、扩展运算符，不能在函数内部显式设定为严格模式，否则报错

箭头函数：`let fn = (参数) => {函数体}` 相当于`let fn = function(参数){函数体}`

- 箭头函数没有自己的 this，它的 this 捕获为函数声明时所处的执行上下文，无法被 call 等函数改变
- 不能作为构造函数实例化对象
- 不能使用 arguments 变量，可通过解构赋值获取可变参数
- 不可以使用 yield 命令，因此箭头函数不能用作 Generator 函数
- 当形参只有一个时，可以省略小括号
- 当函数体只有一条 return 语句时，可以省略花括号和 return，语句的执行结果就是返回值(此时返回对象时要在对象外部加上小括号)

尾调用：在函数的最后一步操作调用了其他函数`fn1(){return fn2()}`会保留外层函数的调用帧

- 尾调用优化：只保留内层函数的调用帧(ES6 支持尾调用优化，仅在严格模式下可用)

柯里化：将多参数函数转换为单参数形式

## DOM

### 元素

#### 获取元素

- `document.body / document.head / document.documentElement` 获取 body/head/html 元素
- `document.getElementById("id")` 返回第一个匹配到 ID 的 Element 对象，找不到返回 null
- `document.querySelector("选择器")` 根据选择器返回第一个匹配的 Element 对象，找不到返回 null
- `document.getElementsByTagName("标签名") / document.getElementsByName("name") / document.getElementsByClassName("类名") / document.querySelectorAll("选择器")` 
  - 返回匹配元素的 HTML 集合(HTMLCollection）
- `element.getElementsByTagName("标签名") / element.getElementsByClassName("类名")` 
  - 在元素的子树中搜索(不含元素自身)，返回匹配元素的 HTML 集合(HTMLCollection）

#### 创建元素

- `document.createElement("标签名")` 创建并返回元素
- `document.write("HTMLText")` 解析 HTMLText 并将元素写入由 document.open() 打开的文档流末尾
  - 在页面加载后调用`write`会自动调用`document.open()`

#### 元素属性

只读属性:

- `tagName` 标签名、`nodeType` 1 为元素节点，2 为属性节点，3 为文本节点

可读写属性：

- `innerText`、`innerHTML`、`outerHTML` innerText 不识别 html 标签并去除空格和换行
- `id`、`src`、`href`、`alt`、`title`、`type`、`value`、`className`
- `checked`、`selected`、`disabled` 值为布尔型

元素的自定义特性：`<div data-age="18"></div>` 自定义特性的格式为`data-xxx-xxx-...`

- `getAttribute('data-xxx-xxx')` 
- 元素的`dataset`属性：一个对象，键名为自定义特性名去掉`"data-"`，再将剩余部分改为小驼峰，键值为特性值

方法：

- `getAttribute("特性名") / setAttribute("特性名",值)` 获取/设置 元素指定特性(可获取自定义特性)
- `removeAttribute("特性名")` 移除特性

#### 样式

- `style` 可读写属性，一个对象，键名为小驼峰的 CSS 属性名，键值为属性值。用于设置样式
- `window.getComputedStyle(元素[, "伪元素字符串"])` 
  - 返回一个只读对象，以键值对的方式存储 CSS 属性(键名为小驼峰)。用于检查样式
- `className` 可读写属性，元素类名字符串
- `classList` 只读属性，包含元素类名字符串的`DOMTokenList`对象。其实例方法如下：
  - `add(class1:str, class2:str...)` 添加类名，若已存在不会添加
  - `contains(class:str):bool` 判断类是否存在
  - `item(index:int):str` 返回索引值对应的类名
  - `remove(同add)` 移除类名，若不存在不会移除
  - `toggle(class:str[, bool]):bool` 切换类名，移除返回false，添加返回true。
    - 第二个参数用于强制添加或移除，true为添加，false为移除

#### 布局

只读布局属性(单位为像素)：

- `clientHeight / clientWidth` 整数，包含内容区+内边距的**可视** 高度/宽度
  - 可视宽高：内容即使超出容器也按不超过时计算宽高
- `clientTop / clientLeft` 整数，元素距离它 上边界的高度/左边界的宽度(即边框大小)
- `offsetParent` 获取最近的包含该元素的定位元素或最近的`table,td,th,body`元素
- `offsetHeight / offsetWidth` 整数，包含内容区+内边距+边框的**可视** 高度/宽度
- `offsetTop / offsetLeft` 整数，元素与其`offsetParent`元素内容区 顶部/左部 的距离

可读写布局属性(单位为像素)：

- `scrollHeight / scrollWidth` 整数，包含内容区+内边距的**实际** 高度/宽度
  - 实际宽高：不论内容是否超出容器，都按实际内容区+内边距计算宽高
- `scrollTop / scrollLeft` 数值，被卷去 上侧/左侧 的距离(元素溢出 上界/左界 的距离)

window 的布局属性：

- `window.scrollY / window.scrollX` 只读整数，页面被卷去 上部/左部 的距离。可读写的相似属性：
  - 声明了 DTD 的情况下：`document.documentElement.scrollTop`
  - 未声明 DTD 的情况下：`document.body.scrollTop`

#### 节点

只读属性：

- `childNodes` 所有子节点集合(包含元素节点和文本节点)
- `children` 所有子元素节点集合
- `firstChild / lastChild` 第一个/最后一个 子节点
- `firstElementChild / lastElementChild` 第一个/最后一个 子元素节点
- `nextSibling / previousSibling` 下一个/上一个 兄弟节点
- `nextElementSibling / previousElementSibling` 下一个/上一个 兄弟元素节点

方法：

- `appendChild(节点2)` 返回节点 2，将节点 2 添加为调用节点的最后一个子节点
- `insertBefore(节点2, 节点3)` 返回节点 2，将节点 2 添加到调用节点的子元素节点 3 之前
- `insertAdjacentHTML("position", "HTMLText")` 无返回，解析 HTMLText 为元素并添加到调用节点的指定位置。position 的取值：
  - `'beforebegin'` 调用节点前
  - `'afterbegin'` 调用节点内部的开头
  - `'beforeend'` 调用节点内部的末尾
  - `'afterend'` 调用节点后
- `insertAdjacentElement("position", 元素)` 无返回，将元素添加到调用节点的指定位置
- `insertAdjacentText("position", 文本节点)` 无返回，将文本节点添加到调用节点的指定位置
- `removeChild(节点2)` 返回节点 2，删除调用节点的子节点 2
- `remove()` 无返回，删除调用节点
- `cloneNode([布尔=false])` 返回调用节点的副本，若参数为 true，同时复制标签和内容，否则只复制标签

- `appendChild(节点2)` 返回节点 2，将节点 2 添加为调用节点的最后一个子节点

### 事件

DOM 事件流：捕获阶段、当前目标阶段、冒泡阶段
onclick 定义的监听器和 attachEvent 只能捕获冒泡阶段、addEventListener 可以选择冒泡或捕获
事件委托：不在每个子节点单独设置事件监听器，而是监听器设置在父节点上，然后利用冒泡原理影响每个子节点

#### 注册事件

- `eventTarget.on事件名 = callback`
- `eventTarget.addEventListener('事件类型字符串', callback[, useCapture = false])`
  - useCapture 为 ture 表示在捕获阶段调用，false 为冒泡阶段调用
- `eventTarget.attachEvent('带on的事件类型字符串', callback)` 非标准，ie9-

#### 删除事件

- `eventTarget.on事件名 = null`
- `eventTarget.removeEventListener(...)`
- `eventTarget.detachEvent(...)`

#### 事件大全

- 鼠标：`click, mousemove, mouseup, mousedown, mouseenter, mouseover, mouseout, mouseleave, wheel`
  - mouseenter 和 mouseover：mouseover 在鼠标进入自身和子元素时都会触发；mouseenter 只在进入自身触发(无冒泡)。mouseleave 和 mouseout 同理，mouseleave 不会冒泡
- 键盘：`keydown(按键按下), keyup(按键弹起), keypress(按键按下，每次有文本输入时触发，不识别功能键)`
  - 执行顺序：keydown – keypress – keyup
- 表单：`submit(表单提交), change(表单元素的 value 发生改变)`
- 窗体：`scroll(滚动条滚动), contextmenu(开启右键菜单), selectstart(拖动左键选择), resize(窗体大小发生改变), load(页面完全加载完毕), DOMContentLoaded(DOM 加载完毕并执行完 js 代码)`
- 注：若要监视 DOM 元素的大小变化，可使用 `ResizeObserver` API

#### 事件对象

事件回调定义时可将事件对象 e 作为第一个形参，系统自动传入
低版本浏览器需要通过`window.event`获取事件对象，兼容性写法：`e = e || window.event`

属性：

- `target` 触发事件的对象(ie678：`srcElement`，非标准)
- `currentTarget` 绑定事件的对象(即 this，ie678 不可用)
- `type` 事件类型

阻止默认事件：

- `e.preventDefault()` ie9+
- `e.returnValue = false` ie678
- `return false` 传统方式，事件回调函数返回false

阻止冒泡：

- `e.stopPropagation()` ie9+
- `e.cancelBubble = true` ie678

鼠标事件：

- 禁止鼠标右键菜单：阻止 `contextmenu` 事件的默认行为
- 禁止鼠标选中：阻止 `selectstart` 事件的默认行为
- `clientX / clientY` 鼠标相对于浏览器窗口可视区的 X / Y 坐标，不带单位
- `pageX / pageY` 鼠标相对于文档页面的 X / Y 坐标，不带单位，IE9+
- `screenX / screenY` 鼠标相对于电脑屏幕的 X / Y 坐标，不带单位
- `offsetX / offsetY` 鼠标相对于事件元素的 X / Y 坐标，不带单位

键盘事件：

- `keyCode` 该键的 ASCII 码值(keyup 和 keydown 不区分字母大小写，默认大写；keypress 区分大小写)

## ES6

### Symbol

符号，ES6 新增的原始数据类型。符号实例唯一、不可变，不能与其他数据运算，用来创建唯一记号，用作非字符串形式的对象属性

创建：

- `Symbol([‘描述字符串’]);` 创建并返回符号实例，即使描述字符串相同，值也不同
- `Symbol.for([‘描述字符串’])` 返回符号实例，以描述字符串为键，在全局符号注册表中创建并重用符号实例
  - 即使描述字符串相等，注册表中定义的符号实例跟用 Symbol()定义的符号实例也不等同

使用符号作为属性：任何可以使用字符串或数值作为属性的地方，都可以使用符号。

静态方法：

- `keyFor(symbol实例)` 在全局符号注册表中查询 symbol 实例的字符串键，若查询不到，返回 undefined

常用内置符号：用于暴露语言内部行为，这些内置符号以全局函数 Symbol 的字符串属性存在，值为符号

- `Symbol.hasInstance` 用作对象的键，值为方法，instanceOf
- `Symbol.toStringTag` 对象，字符串，定义Object.prototype.toString.call返回值的标签名
- `Symbol.unscopables` 对象，配置对象，配置对象中设置同名属性，值为true，将属性从 with 环境排除
- `Symbol.toPrimitive` 对象，方法，转换为原始值时，以该方法的返回值为结果
- `Symbol.iterator` 对象，方法，该方法返回对象默认的迭代器
- `Symbol.asyncIterator` 对象，方法，返回默认 AsyncIterator
- `Symbol.isConcatSpeadable` 对象，布尔值，表示对象在被 concat 用作参数来合并时，选择是否展开
  - 对于数组对象，默认为真值；对于类数组对象，默认为假值
- `Symbol.match` 正则对象，方法，String.prototype.match()会调用该方法对正则表达式求值
  - 方法接收参数 target，即调用 match()方法的字符串实例，方法的返回值无限制
- `Symbol.replace / Symbol.search / Symbol.split` 对象，方法。String.prototype.replace()/search()/split()。replace 接收参数 target、replacement(替换字符串)，后两者接受 target，返回值无限制
- `Symbol.species` 对象，方法，创建衍生对象时，会使用该方法
  - 会创建衍生对象的方法：`Array.prototype.map() .filter() .concat()`

### Set

Set(集合)，用于存储任意类型的唯一值，有序(按插入顺序)

创建：`new Set([可迭代对象])`

实例属性：`size` 成员个数

实例方法：

- `add(值)` 返回集合实例
- `has(值)` 返回布尔，表示是否有该值
- `delete(值)` 返回布尔，表示是否删除成功
- `clear()` 清空集合，无返回
- `keys() / values()` keys 为 values 的别名，默认迭代器，按插入顺序生成集合内容
- `entries()` 返回迭代器，按插入顺序生成[元素 n，元素 n]
- `forEach(fn(val, key){}[, thisArg])` val 与 key 完全相等

WeakSet: 弱集合，不能使用非对象作为值。不可迭代。weak 指值不属于正式的引用，不会阻止垃圾回收

创建：`new WeekSet(...)`

实例方法：`add、has、delete`

应用：保存 DOM 节点元数据：节点删除后，自动从弱集合中删除

### Map

Map(映射)，用于存储有序(按插入顺序)键值对，是可迭代对象，可以将任何 JS 数据类型作为键

创建：`new Map([成员是键值对的可迭代对象])` 如：`[[“key”,”val”]]`，自定义迭代器等

实例属性：`size`键值对的个数

实例方法：

- `set(键名,键值)` 添加，返回映射实例
- `get(键名)` 获取，若不存在返回 undefined
- `has(键名)、delete(键名)、clear()`
- `keys()/ values()/ entries()` entries 返回默认迭代器，按插入顺序生成[key, value]为成员的数组
- `forEach(fn(val,key){}[,this])`
  相比 Object：节约内存、插入性能更好、查找速度稍慢、删除性能更好

WeakMap：弱映射，不可迭代，不能使用非对象作为键，键不属于正式引用，不阻止垃圾回收，键被回收后值也会被回收

创建：`new WeakMap(...)`

静态属性：`length` 永远为 0

实例方法：`set、get、has、delete`

应用：保存 DOM 节点元数据；实现私有变量：

```JavaScript
const User = (()=>{
    const wm = new WeakMap()
    class User{
        setData(data){
        const container = wm.get(this) || []
        container.push(data)
        wm.set(this, container)
        }
    }
    return User
})()
```

### WeakRef 与 FinalizationRegistry

**WeakRef**：直接创建对象的弱引用，在本轮事件循环不会清除原始对象

创建：`new WeakRef(原始对象)`

实例方法：`deref()` 若原始对象存在，返回原始对象。若原始对象已被回收，返回 undefined

**FinalizationRegistry**：清理器注册表，弱引用

创建：`const registry = new FinalizationRegistry(heldValue => {});` 指定清除后，执行回调

实例方法：

- `register(目标对象, 参数1[, 对象])` 被回收后，将参数 1 作为 heldValue 传入创建时的回调并执行函数
  - 若要取消注册，需要先向 register 传入第三个参数，一般用目标对象，然后调用 unregister()
- `unregister(目标对象)` 取消注册

### Proxy

拦截器(代理)，对对象的访问需要先通过这层拦截

创建：`const p = Proxy(源对象，{配置项})` 对 p 的属性的增删改查操作会映射到源对象上，并触发 getter、setter()等。配置项如下：

- `get(target【源对象】,propName【属性名字符串】,receiver){return xx)}` 拦截读取属性
  - receiver 参数：指向当前的 Proxy 实例
- `set(target,propName,value,receiver){target.propName = value}` 拦截修改或追加属性
- `deleteProperty(target,propName){return Reflect.deleteProperty(target，propName)}` 拦截删除属性
- `construct、ownKeys、getOwnPropertyDescriptor、defineProperty、preventExtensions、getPrototypeOf、isExtensible、setPrototypeOf、apply`

this 问题：在通过 Proxy 实例访问对象时，该对象内部的 this 会指向 Proxy 实例

### Reflect

Reflect 是一个内置的对象，它提供拦截 JavaScript 操作的方法。

静态方法：

- `Reflect.apply(target, thisArg, args)`
- `Reflect.construct(target, args)`
- `Reflect.get(target, name, receiver)` 获取对应属性的值
  - receiver 参数：当读取的属性部署了 getter， receiver 参数为 setter 调用时的 this 值
- `Reflect.set(target, name, value, receiver)` 修改或追加属性的值
- `Reflect.defineProperty(target, name, desc)` 添加或修改属性
  - `Object.defineProperty` 重复添加已有属性时，会抛出异常；`Reflect.defineProperty`不抛异常，返回布尔值判断是否成功
- `Reflect.deleteProperty(target, name)` 删除指定属性
- `Reflect.has(target, name)`
- `Reflect.ownKeys(target)` 返回一个数组，包含对象自身的(不含继承的)所有键名，不管键名是 Symbol 或字符串，也不管是否可枚举
- `Reflect.isExtensible(target)`
- `Reflect.preventExtensions(target)`
- `Reflect.getOwnPropertyDescriptor(target, name)`
- `Reflect.getPrototypeOf(target)`
- `Reflect.setPrototypeOf(target, prototype)`

### Promise

异步编程的解决方案。一旦创建就会立即执行(同步)。

Promise 有三种状态：pending(等待)、fulfilled(成功)、rejected(失败)

- 当 Promise 实例 p 状态为等待时，resolve(p)和 then(p)会等待状态变更后执行

创建：`const p = new Promise(function(resolve,reject){})`

- 调用`resolve(value)`，会将 Promise 实例的状态更改为成功；调用`reject(error)`，更改为失败

实例方法：

- `then([function(value){}][, function(error){}])` 异步执行，若状态为成功，执行第一个方法，为失败执行第二个方法。then 方法返回一个 Promise 对象，对象状态由传入函数的执行结果决定：
  - 若传入函数返回的结果是非 promise 类型的属性，或没有返回，则返回对象状态为成功，其值属性为传入函数的返回值
  - 若传入函数返回的结果为 promise 对象，则外实例状态为内实例状态，外实例的值属性为内实例的值属性
- `catch(function(){})` 当对象状态为失败时执行回调
- `finally(function(){})` 在 Promise 结束时，不论结果时 fulfilled 或 rejected，都执行回调

静态方法：

- `all(成员为Promise实例的数组)` 将多个 Promise 实例包装成一个实例并返回
  - 当所有成员实例的状态都变为成功时，返回的实例的状态才会变为成功，其值为成员实例返回值组成的数组
  - 只要有一个成员实例的状态变为失败，返回实例的状态就会变为失败，其值为该成员实例返回值
- `any(成员为Promise实例的数组)` 返回一个 Promise 实例
  - 成员实例有一个实例状态变为成功，返回实例的状态也变为成功，其值为该成员实例的返回值
  - 成员实例状态都变为失败，返回实例的状态变为失败，其值为 AggregateError 实例
- `race(成员为Promise实例的数组)` 返回一个 Promise 实例
  
  - 成员实例有一个实例状态改变，返回实例的状态跟着改变，其值为该成员实例的返回值
- `allSettled(成员为Promise实例的数组)` 返回一个 Promise 实例
  - 只有所有成员实例状态改变时，返回实例的状态才改变，其状态只会变为成功
- `resolve(value)` 等同于`new Promise(resolve => resolve(value))`
- `reject(error)` 等同于`new Promise((resolve, reject) => reject(value))`

报错：

- 若不报错，只有在对象状态为失败，且不存在 then 的第二个方法时，才会进入 catch

- 若 then 执行时报错，可以被 catch 捕获报错信息；then 的第二个方法无法捕获第一个方法的报错

- 若 Promise 对象执行时报错，Promise 对象状态变更为失败，可被 then 的第二个方法或 catch 捕获错误

- 若 catch 执行时报错，可被链式调用的下一个 catch 捕获

- Promise 对象内部抛出的错误不会传递到外层代码

### Iterator

- 可迭代协议(Iterable 接口)：实现该协议需要：实现@@iterator 方法，即具有默认迭代器属性
  - 默认迭代器属性：一种属性，以 Symbol.iterator 为键，值为一个无参数函数，其返回值为一个迭代器
  - 实现了可迭代协议的内置类型：String、Array、TypedArray、Set、Map、arguments 对象、NodeList 对象
- 可迭代对象：实现了可迭代协议的对象
  - 接收可迭代对象的原生语言特性会在后台调用默认迭代器属性创建迭代器来获取值，它们有：
    - for-of、数组解构、扩展操作符、Array.from()、创建集合和映射、Promise.all()/ race()、yield\*操作符
- 迭代器协议(Iterator 接口)：实现了 next()方法，每次调用返回一个具有 value 和 done 属性的对象
  - value 属性：下一个将要返回的值，默认值为 undefined
  - done 属性：布尔值，默认值为 false。迭代器已将序列迭代完毕时，done 为 true；否则为 false
- 迭代器(Iterator）：实现了迭代器协议的，按需创建的一次性对象
  - 迭代器维护对可迭代对象的引用，阻止垃圾回收程序回收可迭代对象
  - 原生迭代器都实现了可迭代协议，调用其默认迭代器属性会返回迭代器本身
  - return 方法：迭代器可定义一个会被自动调用的 return()方法，调用时关闭迭代器
    - 会调用 return 方法的场景：for-of 循环通过 break、continue、return、throw 提前退出；解构赋值未用到所有值时
    - return 方法必须返回一个具有 `done:true` 的对象

```JavaScript
function readLinesSync(file) {
  return {
    [Symbol.iterator]() {
      return {
        next() {
          return { done: false };
        },
        return() {
          file.close();
          return { done: true };
        }
      };
    },
  };
}
```

### Generator

生成器是一个特殊函数，可以在函数块中暂停和恢复代码执行

- 创建：在任何可定义函数(箭头函数除外)的地方均可创建。函数名前(function 关键字后)加 \* 表示它是一个生成器。
- yield 关键字：`yield [表达式]` 只能在生成器中使用，在执行完表达式后，立即暂停生成器，除非通过 next()传参，否则整个表达式的值为 undefined
- yield\*关键字：`yield* 可迭代对象` 依次迭代该对象并在每次迭代时暂停生成器。将可迭代对象返回 done：true 时的 value 值作为自身值

- 生成器对象：调用生成器会返回一个处于暂停状态(suspended）的生成器对象。生成器对象实现了迭代器协议。同一个生成器创建的多个生成器对象不相同，具有不同的作用域。生成器对象使用对应生成器的原型。其实例方法有`next、return、throw`

  - `next([值])` 若传入参数，作为上一次使生成器暂停的 yield 关键字的值
    
    - 调用生成器对象的`next([值])`，可以使暂停状态的生成器恢复执行，返回一个对象：
      - 若执行中通过 yield 关键字退出，返回对象的 value 属性值为 yield 的值，done 为 false
      - 若执行中通过 yield\*关键字退出，返回对象的 value 属性值为该次迭代返回的 value 值，done 为 false
      - 若执行中通过 return 关键字退出，返回对象的 value 属性值为 return 的值，done 为 true
      - 若在函数结尾无return退出，返回 `{ value: undefined, done: true }`
    
  - ```javascript
    function* fn() {
      console.log("开始执行");
      const y1 = yield 1;
      console.log("y1：", y1);
      const y2 = yield 2;
      console.log("y2：", y2);
    }
    
    const g = fn();
    console.log("创建生成器");
    console.log("next1：", g.next("传入参数1"));
    console.log("next2：", g.next("传入参数2"));
    console.log("next3：", g.next("传入参数3"));
    
    /*
    创建生成器
    开始执行
    next1：	{ value: 1, done: false }
    y1：  	传入参数2
    next2：	{ value: 2, done: false }
    y2：  	传入参数3
    next3：	{ value: undefined, done: true }
    */
    
    ```

    

  - `return([值])` 强制生成器进入关闭状态。返回一个对象：`｛value：传入的参数，done：true｝`
    
    - 生成器进入关闭状态后，后续的 next()调用都返回`｛value：undefined，done：true｝`
    
  - `throw([值])` 在生成器暂停时使用，将提供的错误注入到生成器中(由上一次使生成器暂停的 yield 关键字抛出)
    
    - 若错误未被处理，关闭生成器
    - 若生成器还没有开始执行，调用 throw 视为从 throw()方法处抛出错误

- 应用：可迭代对象可以是生成器本身，以此实现递归；生成器很适合作为默认迭代器使用；Thunk 函数

### async

- 创建：`async function 函数名(){}` async 声明的函数表明内部含有异步操作，调用该函数时，会立即返回一个 Promise 对象(pending 状态)，并同步执行内部语句直到遇到await
- async 函数的返回值将作为 then 方法回调函数的参数：
  - 若返回非 Promise 对象或是一个成功的 promise 对象，则类型为成功，值为 ruturn 值。
  - 若返回一个失败的 promise 或抛出异常，则类型为失败。
- await 表达式：`await promise对象`
  - await 表达式必须在 async 函数中，await 返回 promise 对象成功的值。若 promise 对象状态为失败，就会抛出异常，值为失败值
  - 遇到 await 语句时，async 函数会在执行完 await 语句后再继续向下执行

### Class

注意：

- Class 和模块内部，默认为严格模式
- 类的方法内部若含有 this，默认指向类的实例

定义：`class 类名 [extends 父类名] {...}`

- Class 内可以定义 constructor 方法、实例属性、实例方法以及 getter 和 setter
  - `constructor(参数){该方法为构造方法，在构造时自动执行}` this 指向实例对象
  - `属性 = xx`
  - `方法名(){方法体}`
  - `get 属性名(){} / set 属性名(newVal）{}`

创建：`new 类名()` 由 class 创建的类本质是一个特殊的构造函数，不通过 new 调用时会报错，不能调用 call

- 在 class 中声明的非静态方法都保存在原型对象中

继承：定义时指定 extends。子类若有构造函数，必须执行一次 super()，此时 super() 中的 this 指向子类的实例

- 继承后子类的原型对象即为父类

super 关键字：

- 作为函数调用，代表父类的构造函数，只能在子类的构造函数内使用
- 作为对象使用，指向父类的原型对象；在静态方法中使用，super 指向父类。子类方法中通过 super 调用父类方法时，方法内部的 this 指向当前子类实例
- super 单独使用时会报错

静态成员：在类声明体中，成员声明前添加 static 关键字，可以指定成员为类的成员而非实例的成员
私有属性：属性名的第一位为`'#'`。如：`#name = 'Chen'` 仅能从定义它的类的内部访问该属性
in 运算符：`属性 in 实例` 返回布尔，判断是否能访问到指定属性
new.target 属性：一般用于构造函数/类中，返回 new 命令作用于的那个构造函数，不通过`new`或`Reflect.construct`调用则会返回 undefined。用于类中返回当前类

### 模块

#### ES6 模块

注意：

- 模块默认使用严格模式

特点：

- ES6 模块中的值属于【动态只读引用】。
- 只读，即不允许修改引入变量的值，import 的变量是只读的，但可修改复杂引用类型的属性。当模块遇到 import 命令时，就会生成一个只读引用。等到使用时，再根据这个只读引用，到被加载的那个模块里面去取值
- 动态，即原始值发生变化，import 加载的值也会发生变化。不论是基本数据类型还是复杂数据类型。

export：`export 接口`

- export 必须提供一个接口(已声明变量用对象包裹，未声明变量在 export 处声明)`export function f(){}`   `let m = 1; export {m};`
- 默认输出：`export default 变量或方法` 本质是输出一个叫做 default 的变量或方法，系统允许为它取任意名字

import：`import {start, readFile} from "fs"` 加载同级目录下的 fs.js 模块

- import 语句先于模块内其他语句执行。import 会执行所加载的模块，多次加载只执行一次
- 整体加载：`import * as fs from 'fs'`
- 针对默认加载：`import 别名 from “fs” `
- HTML 加载：`<script src=“xx.js” type=“module”>` 异步加载，相当于打开了 defer 特性

as 关键字：`变量 as 别名` 重命名输出或输入的变量

import()函数：`import(加载模块的位置)`动态加载指定模块，返回 Promise 对象，值属性为模块对象(成功)/错误信息(失败)

- import 函数为异步加载，且与所加载模块没有静态连接关系

export 与 import 的复合写法：

```JavaScript
export { foo, bar } from 'my_module';
// 等同于
import { foo, bar } from 'my_module';
export { foo, bar };
```

- 接口改名：`export { foo as myFoo } from 'my_module';`
- 整体输出：`export * from 'my_module';` 会忽略 my_module 模块中的 default 方法
- 默认接口：`export { default } from 'foo';`
- 具名接口改为默认接口:

```JavaScript
export { es6 as default } from './someModule';
// 等同于
import { es6 } from './someModule';
export default es6;
```

- 默认接口改名为具名接口:`export { default as es6 } from './someModule';`

循环加载：ES6 模块在编译时输出接口，但使用该接口时发现未定义会报错。

```JavaScript
// a.mjs
import {bar} from './b';
console.log(bar);
export let foo = 'foo';
// b.mjs
import {foo} from './a';
console.log(foo); // 报错
export let bar = 'bar';
```

- 解决方法：将接口写作函数(变量提升)

```JavaScript
// a.mjs
import {bar} from './b';
console.log(bar());
function foo() { return 'foo' }
export {foo};
// b.mjs
import {foo} from './a';
console.log(foo());
function bar() { return 'bar' }
export {bar};
```

ES6 模块加载 CommonJS 模块：`import packageMain from 'commonjs-package';` 只能整体加载(通过 Node.js 的内置方法`module.createRequire()`也可实现，但写法会混合 ES6 和 CommonJS，不建议使用)

#### CommonJS 模块

- 特点：

  - CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。
  - CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。
    - CommonJS 加载的是一个对象，该对象只有在脚本运行完才会生成
  - CommonJS 模块的`require()`是同步加载模块(多次加载仅加载一次)，ES6 模块的 import 命令是异步加载，有一个独立的模块依赖的解析阶段。

- 导出：将 exports 对象导出(默认为空对象)

  - 方法一：`exports.变量 = xx`
  - 方法二：`module.exports = 对象` 该属性指向 exports 对象

- 引入：`require('文件名')` 加载模块，返回模块导出的 exports 对象

- 循环加载：所有模块只会执行一次，因此当某个已执行模块被再次加载时，就只读取缓存中的值并输出，不会继续执行未执行部分。

模块的 package.json 文件：

- main 字段：`"main": "./src/index.js"` 指定入口文件
- exports 字段：
  - 子目录别名：`"exports": {"./submodule": "./src/submodule.js"}` 指定`src/submodule.js`别名为`submodule`
  - main 的别名：`"exports": {".": "./main.js"}` `.`代表模块的主入口，这种写法优先级高于 main 字段
    - 可简写为`"exports": "./main.js"`
  - 条件加载：`"exports": {".": {"require": "./main.cjs", "default": "./main.js"}}` require 指定 CommonJS 的入口，default 指定其他情况的入口
    - 需要打开`--experimental-conditional-exports`标志

CommonJS 模块加载 ES6 模块：

```JavaScript
(async () => {
  await import('./my-app.mjs');
})();
```

### 异步迭代器与生成器

#### 异步迭代器

普通迭代器在调用时必须立刻返回结果，对于异步操作只能通过将 value 属性设为 Promise 对象来实现。异步迭代器解决了这一痛点。

- 异步迭代器部署在对象的`Symbol.asyncIterator`属性上
- 异步迭代器也有`next`方法，调用时返回一个 Promise 对象，其状态变为成功后的回调函数的参数为具有 value, done 属性的对象

- `for await...of`：只能在 async 函数内使用，用于遍历异步的 Iterator 接口。(也可用于同步迭代器)
  - 自动调用该对象异步迭代器的 next 方法，当返回的 Promise 对象状态变为成功时，将 value 属性赋值给指定变量，并进入循环体
  - 若返回的 Promise 对象状态变为失败，`for await...of`就会报错

#### 异步生成器

`async function* gen() {}`

- 调用时返回一个异步生成器对象，对该对象调用 next 方法，返回一个 Promise 对象

### 二进制数组

- ElementType：`Int8`(8 位整数)、`Uint8`(8 位无符号整数)、`Int16`、`Uint16`、`Int32`、`Uint32`、`Float32`、`Float64`

- 字节序：字节序由 JS 运行时所在系统决定，大端/小端字节序表示高位字节处于低位字节的前侧/后侧。

#### ArrayBuffer

缓冲，代表储存二进制数据的一段内存，不能直接读写，是所有定型数组及视图引用的基本单位，可被垃圾回收机制回收。

- 创建：`new ArrayBuffer(字节数)` 在内存中分配一段连续区域，创建后不能调整大小，初始值为 0
- 实例属性：`byteLength`
- 实例方法：`slice(...)`
- 静态方法：`isView(变量)` 返回布尔，表示参数是否为 ArrayBuffer 的视图实例(DataView 和 TypedArray）
- 与字符串相互转换：`TextEncoder`、`TextDecoder`

#### TypedArray

定型数组，一种视图，特定于一种 ElementType 并且遵循系统原生的字节序

- 定型数组一共有 9 种类型，以下为其构造函数名：`Int8Array`(8 位整数)、`Uint8Array`(8 位无符号整数)、`Uint8ClampedArray`(8 位无符号整数，溢出处理不同)、`Int16Array`、`Uint16Array`、`Int32Array`、`Uint32Array`、`Float32Array`、`Float64Array`

- 创建：

  - `new TypedArray(ArrayBuffer实例，开始的字节序号，长度)`
  - `new TypedArray(长度)`
  - `new TypedArray(普通数组 / 定型数组)`
  - 传入其他类型定型数组，生成定型数组长度不变，各元素自动转换格式，缓冲字节数会自动调整

- 实例属性：

  - `length`、`byteLength`、`byteOffset` 定型数组是从底层 ArrayBuffer 对象的哪个字节开始的
  - `buffer` 返回整段内存区域对应的 ArrayBuffer 对象(只读)

- 静态属性：`BYTES_PER_ELEMENT` 每个元素的字节数

- 实例方法：

  - 具有数组的大部分实例方法，除了可能修改数组大小的方法(concat、pop、push、shift、splice、unshift）
  - `set(定型数组/普通数组[，start])` 复制传入数组的值到指定索引处
  - `subarray([start][，end])` 返回子定型数组

- 静态方法：`from/of`方法：`Int8Array.from(普通数组)/ of(参数序列)`

- 溢出：上溢和下溢表示超过类型的最大/最小值。上溢和下溢不影响其他成员。上溢时舍去溢出部分

#### DataView

一种视图，专为文件和网络 I/O 设计，支持对缓冲数据的高度控制，相比其他视图性能略差，对缓冲内容无预设，无法迭代

- 所有 DataView 的 API 可接受一个可选的布尔值，若为 true，启用小端字节序。默认为 false
- 创建：`new DataView(ArrayBuffer实例 [，开始的字节序号][，字节数])` 默认使用全部 ArrayBuffer
- 实例属性：`buffer`、`byteLength`、`byteOffset`
- 实例方法：`getInt8(索引)/ setInt8(start，值)` 从指定索引的字节处开始，获取/设置一个 8 位整数。支持所有 ElementType

### 修饰器

> 修饰器提案暂未定案，语法可能会改变，避免使用修饰器

修饰类：

```JavaScript
@decorator
class A {}
// 等同于
class A {}
A = decorator(A) || A;
```

修饰方法：在每次方法执行前，执行修饰器函数

```JavaScript
@readonly
  name() { return `${this.first} ${this.last}` }
```

- 类的修饰器函数的`function testable(target) {}` target 为被修饰的类
- 方法的修饰器函数：`function readonly(target, name, descriptor){}`
  - 方法装饰器可以返回一个新方法，取代原来的方法，也可以不返回值，表示依然使用原来的方法。如果返回其他类型的值，会报错。
  - target 为类的原型、name 为装饰的属性名、descriptor 为该属性的描述对象
- 若需要接受参数，可以在修饰器外再封装一层函数，该函数返回修饰器

```JavaScript
@testable(true)
class MyTestableClass {}
```

- 有多个方法修饰器时，从外到内进入，在从内向外执行

```JavaScript
function dec(id){
  console.log('evaluated', id);
  return (target, property, descriptor) => console.log('executed', id);
}
class Example {
    @dec(1)
    @dec(2)
    method(){}
}
// evaluated 1
// evaluated 2
// executed 2
// executed 1
```

## 网络

### Ajax

Asynchronous JavaScript And XML（Ajax），即异步的JS和XML。可以无刷新获取数据

优缺点：

- 无需刷新花去数据、可以根据用户事件更新部分页面内容
- 没有浏览历史不能回退、存在跨域问题、SEO不友好
  - 跨域解决方案：JSONP、CORS、代理服务器

使用： 

```javascript
const xhr = new XMLHttpRequest() // 创建对象
xhr.open('Get', 'http://127.0.0.1:8000/server') // 设置请求方法和url
xhr.send() // 发送
// 处理服务器返回的结果
xhr.onreadystatechange = function (){
    // 判断服务端是否返回了所有的结果
	if(xhr.readyState === 4){
        // 判断响应状态码是否为成功
		if(xhr.status >= 200 && xhr.status < 300){  
            ...
		}
	}
}
```

实例属性：

- `status` 响应状态码
- `statusText` 响应字符串
- `response` 响应体
- `timeout` 设置超时毫秒数      

实例方法：

- `open('请求类型', 'url')` 初始化
- `setRequesetHeader('属性名', '值')` 设置请求头的属性
- `send(['参数'])` 发送请求，POST可以传参
- `abort()` 取消请求
- `getAllResponseHeaders()` 返回整个响应头

实例事件：

- `timeout` 超时
- `readystatechange` readyState属性发生变化（abort时不会触发）

IE缓存问题：

- IE会把ajax请求缓存，再次向服务器请求时，直接读取。会有时效性信息无法获取的情况。
- 解决方法：传输一个属性，t=date.now（）

`window.fetch('url', 配置对象)` 发送请求，返回Promise对象

- 配置对象：
  - `method`：`'GET'`、`'POST'`
  - `headers`：请求头信息，为对象
  - `body`：请求体信息，可为对象可为字符串（get方法的参数）
- 特点：
  - 收到一个代表错误的HTTP状态码（2xx）时，返回的Promise状态仍为fulfilled，但返回值的ok属性为false，仅网络故障或请求被阻止时，才会标记为rejected
  - 可以接受跨域cookies
  - 不发送跨域cookies

### WebSocket

- WebSocket 是 HTML5 提供的一种浏览器与服务器进行全双工通讯的网络技术，属于应用层协议，基于 TCP 传输协议，并复用 HTTP 的握手通道。

- 特点：支持双向通信、可以发送文本及二进制数据、建立在 TCP 协议上服务端实现容易、数据格式轻量性能开销小、没有同源限制、协议标识ws（加密则为wss），默认端口也为80和443

- 创建：`new WebSocket('url' [, '协议'])`

  - 例：`new WebSocket('ws：//localhost：9999')`

- 事件：事件绑定到实例上，回调函数接受 event 事件对象

  - `close` 连接关闭时
  - `error` 连接因错误而关闭时
  - `message` 收到数据时
  - `open` 连接成功时

- 实例属性：

  - `onclose` 等(每个事件都有)。返回一个事件监听器，该属性接受一个回调函数
  - `binaryType` 返回实例连接所传输二进制数据的类型(`"blob"、"arraybuffer"`)

- 只读实例属性：

  - `bufferedAmount` 返回已经被 send 方法放入队列中但还没有被发送的数据的字节数
  - `extensions` 返回服务器已选择的扩展值
  - `protocol` 返回服务器端选中的子协议的名字
  - `url` 返回对象实例化时传入 URL 的绝对路径
  - `readyState` 返回当前实例的链接状态，值为四个常量之一
    - `WebSocket.CONNECTING = 0`
    - `WebSocket.OPEN = 1`
    - `WebSocket.CLOSING = 2`
    - `WebSocket.CLOSED = 3`

- 实例方法：

  - `close([code[, 'reason']])` 关闭连接或停止连接尝试
    - code 为状态码，reason 为自然语言描述的原因字符串
  - `send(data)` 将需要传输的数据排入队列。data 需要是以下类型之一：USVString、ArrayBuffer、Blob、ArrayBufferView

### URLSearchParams

该对象可用于处理URL查询字符串，可迭代。**该API尚在试验中，不同环境实现可能不同，谨慎使用。**

创建：`new URLSearchParams(url: str)` 输入的URL若有`？`则会去除`？`及其后方的内容

实例方法：

- `append(name: str, value)` 添加参数
- `delete(name: str)` 、`get(name)`、`has(name)`、`set(name, value)`等

### 文件上传与下载

#### 上传

```html
<input id="file" onChange="FilesUpload" type="file" />
```

逻辑部分：

```js
function FilesUpload() {
    const fileRef = document.getElementById('file');
	let param = new FormData();
	let data = {
        fileSize: fileRef.files[0].size,  //文件大小 :单位B
        fileName: fileRef.files[0].name,  //文件名
        fileType: fileRef.files[0].name.split('.')[1], //文件类型：pdf,docx
        file: fileRef.files[0]  //file表单数据
	};
    
	// 利用for循环将所有准备好的文件信息遍历到params中
    for (const key in data) {
        param.append([key], data[key]);
    }

    // 使用axios上传
    getUpload(param).then((res) => {...});
},
```



#### 下载

1. 适用于浏览器无法识别文件，如果是html、jpg、pdf等会直接解析展示，而不会下载
```javascript
window.top.location.href = url 
window.open(url)
```

2. a标签`< a href="/images/download.jpg" download="myFileName">`
3. 文件流



## Canvas

`<canvas id="tutorial" width="150" height="150">替换内容</canvas>`

- width与height为可选属性。canvas也能用css控制大小，但会进行缩放，导致画面扭曲。
- 当浏览器不支持canvas时，会显示替换内容

### 2D渲染上下文

获取实例：`const ctx = canvas元素.getContext('2d'[, option])`

- ​	option可选值：
  - `alpha: bool` 是否开启透明度

实例属性与方法：

#### 矩形绘制

- `fillRect(x, y, width, height)` 绘制填充的矩形
- `strokeRect(x, y, width, height)` 绘制矩形边框
- `clearRect(x, y, width, height)` 清除矩形区域，使其完全透明

#### 路径

> 路径是通过不同颜色和宽度的线段或曲线相连形成的不同形状的点的集合。一个路径，甚至一个子路径，都是闭合的。
>
> 使用路径绘制图形的步骤：
>
> 1. 创建路径起始点
> 2. 使用画图命令画出路径
> 3. 路径封闭
> 4. 一旦路径生成，就能通过描边或填充路径来渲染图形
>
> 笔触：所有绘制方法均会移动笔触至结束点
>
> [Path2D](#Path2D)

- `beginPath()` 新建一条路径，生成之后，图形绘制命令被指向到路径上生成路径

- `closePath()` 闭合路径，之后图形绘制命令又重新指向到上下文中。

- `stroke()` 通过线条来绘制图形轮廓

- `fill()` 自动闭合路径，通过填充路径的内容区域生成实心的图形

- `moveTo(x, y)` 移动笔触

- `lineTo(x, y)` 绘制直线

- `arc(x, y, radius, startAngle, endAngle, anticlockwise)` 绘制圆弧路径

  > 以（x,y）为圆心，以radius为半径，从startAngle开始到endAngle结束，按照 anticlockwise 给定的方向（默认为false，即顺时针）生成圆弧。
  >
  > 绘制前自动调用lineTo，目标点为圆弧起始点

- `arcTo(x1, y1, x2, y2, radius)` 绘制圆弧路径

  > 当前笔触与点 1 连接的直线，和点 1 与点 2 连接的直线，作为使用指定半径的圆的**切线**，画出两条切线之间的弧线路径。

- `quadraticCurveTo(cp1x, cp1y, x, y)` 以（x, y）为结束点绘制二次贝塞尔曲线路径。（cp1为控制点）

- `bezierCurveTo(cp1x, cp1y, cp2x, cp2y, x, y)` 绘制三次贝塞尔曲线路径

- `rect(x, y, width, height)` 绘制矩形路径



**Path2D**

> Path2D对象用于保存路径，**具有和2D渲染上下文一样的路径方法和属性**，每个Path2D对象具有独立的笔触，克隆时，会同时克隆当前笔触位置。

创建：

```javascript
new Path2D();     // 空的 Path 对象
new Path2D(path); // 克隆 Path 对象
new Path2D(d);    // 从 SVG 建立 Path 对象
```

[使用SVG路径](https://www.runoob.com/svg/svg-path.html)：`d = "M10 10 h 80 v 80 h -80 Z"`

实例方法：

- `addPath(path)` 添加一条路径到当前路径

#### 样式

- `fillStyle = color`

- `strokeStyle = color`

  > `color` 可以是表示 CSS 颜色值的字符串，渐变对象或者图案对象。默认情况下，线条和填充颜色都是黑色（CSS 颜色值 `#000000`）
  >
  > 例如：`"orange", "#FFA500", "rgb(255,165,0)", "rgba(255,165,0,1)";`
  >
  > 所有传入`color`的地方均可用渐变对象、图案样式对象替代

- `globalAlpha = 不透明度` 设置全局不透明度

- `shadowOffsetX / shadowOffsetY = float` 阴影延伸距离

- `shadowBlur = float` 阴影模糊程度

- `shadowColor = color` 阴影颜色

- `lineWidth = num` 线宽

- `lineCap = type` 线条末端样式
  
  - `type`：`'butt'`以方形结束、`'round'`以圆形结束、`'square'`以方形结束，但是增加了一个宽度和线段相同，高度是线段厚度一半的矩形区域
  
- `lineJoin = type` 线条与线条间接合处的样式
  
  - `type`：`'round'`圆角连接、`'bevel'`三角形填充连接、`'miter'` 矩形填充连接（默认）
  
- `miterLinit = num` 两条线相交时交接处最大长度（两线夹角较小时，交接处会很长，默认值为10），超过最大长度后，线条结合处样式变为`'bevel'`

- `setLineDash(segments)` 设置当前线型为虚线，并设置虚线样式
  
  - `segements` num数组。奇数位为表示线段长度，偶数位表示线段间距长度。若数组长度为奇数，数组元素会被复制。例如， `[5, 15, 25]` 会变成 `[5, 15, 25, 5, 15, 25]`
  
- `getLineDash()` 获取当前虚线样式。若数组长度为奇数，数组元素会被复制。

- `lineDashOffset = num` 虚线样式的起始偏移量

- `createLinearGradient(x1, y1, x2, y2)` 创建线型渐变对象

  > 点1为起始点，点2为终点，其连线表示了渐变的方向。与渐变方向垂直的直线上所有点颜色相同，颜色取决于直线与渐变方向的相交点，相交点颜色取决于定义的色标。

- `createRadialGradient(x1, y1, r1, x2, y2, r2)` 以两个圆创建渐变对象

- `渐变对象.addColorStop(位置，color)` 定义色标

  > 位置为取值0-1的数字，渐变起点位置为0，终点位置为1。色标类似于关键帧，定义的位置之间的颜色由计算机自动计算。

- `createPattern(image, repetition)` 创建图像模式，可代替color传给fillStyle等
  
  - `image`：图像对象。见[图片](#图片)
  - `repetition`：重复模式。可选值有：`repeat`，`repeat-x`，`repeat-y` 和 `no-repeat`。

#### 文本

- `fillText(text, x, y [, maxWidth])` 绘制填充文本
- `strokeText(text, x, y [, maxWidth])` 绘制文本边框
- `font = "10px sans-serif"` 文本样式
  - 值使用和 CSS font 属性相同的语法。
- `textAlign = "start" `文本对齐选项。值同CSS text-align 属性。
- `textBaseline = "alphabetic"` 基线对齐选项
  - 可选的值包括：`top`, `hanging`, `middle`, `alphabetic`, `ideographic`, `bottom`。
- `direction = "inherit"` 文本方向。值同CSS direction 属性。取值：`rlt`, `ltr`, `inherit`
- `measureText(string)` 预测量文本。返回一个包含文本宽度等属性的对象

#### 图片

> canvas 的 API 可以使用下面这些类型中的一种作为图片的源：
> `HTMLImageElement`, `HTMLVideoElement`, `HTMLCanvasElement`, `ImageBitmap`

- `drawImage(image, x, y)` 绘制图片
  - 若调用绘制函数时，图片尚未加载完毕，则什么都不会发生
- `drawImage(image, x, y, width, height)` 绘制图片
  - `width`和`height`属性用于控制缩放
- `drawImage(image, sx, sy, sWidth, sHeight, dx, dy, dWidth, dHeight)` 绘制图片（切片）
  - s表示裁剪选择框在源图片中的位置和大小，d表示切片在上下文中的位置和大小

开启/关闭平滑缩放：

```javascript
ctx.mozImageSmoothingEnabled = false;
ctx.webkitImageSmoothingEnabled = false;
ctx.msImageSmoothingEnabled = false;
ctx.imageSmoothingEnabled = false;
```

#### 变形

- `save() / restore()` 保存 / 恢复canvas所有状态

> Canvas 状态存储在栈中，每当`save`方法被调用后，当前的状态就被推送到栈中保存。调用 `restore` 方法，上一个保存的状态就从栈中弹出，所有状态都恢复。一个绘画状态包括：
>
> - 当前应用的变形（即移动，旋转和缩放）
> - 以及`strokeStyle`, `fillStyle`, `globalAlpha`, `lineWidth` 等属性
> - 当前的裁切路径（clipping path）

- `translate(x, y)` 移动canvas的原点
- `rotate(rad)` 以原点为中心旋转canvas坐标系，单位为弧度
- `scale(x, y)` 缩放canvas坐标系，缩放因子比1大时会方法图形
- `transform(a, b, c, d, e, f)` 将变形矩阵乘上传入参数构成的矩阵
  - $\begin{bmatrix}a & b & 0 \\ c & d & 0 \\ e & f & 1  \end{bmatrix}$
- `setTransform(a, b, c, d, e, f)`  重置变形矩阵为单位矩阵，然后用相同的参数调用transform方法
- `resetTransform()` 重置变形矩阵为单位矩阵
  - $\begin{bmatrix}1 & 0 & 0 \\ 0 & 1 & 0 \\ 0 & 0 & 1  \end{bmatrix}$

#### 组合

- `globalCompositeOperation = type` 设置画新图形时采用的遮盖策略，其取值有如下字符串：[参考](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Compositing)

  - `"source-over"`：默认，在旧图形上覆盖绘制
  - `"source-in"`：只在旧图形与画布重叠的区域绘制，旧图形变为透明
  - `"source-out"`：只在旧图形不与画布重叠的区域绘制，旧图形变为透明
  - `"source-atop"`：同`"source-in"`，但旧图形正常显示
  - `"destination-over"`：在旧图形后面绘制新图形
  - `"destination-in"`：旧图形在新图形与画布重叠的区域绘制，新图形变为透明
  - `"destination-out"`：旧图形在新图形不与画布重叠的区域绘制，新图形变为透明
  - `"destination-atop"`：同`"destination-in"`，但新图形正常显示
  - `"lighter"`：新旧图形重叠的颜色通过色值相加确定
  - `"copy"`：只显示新图形
  - ......

- `clip()` 将当前正在构建的路径转换为当前的裁剪路径

  > 裁切路径的作用是遮罩，只绘制路径内的内容，会被保存为canvas的状态

#### 保存图片

`canvas.toDataURL([type][, quality])` 返回一个包含图片展示的data URI

- `type` 图片格式，默认为`'image/png'`，可选值有`'image/jpeg'`等
- `quality` jpeg或webp图片的质量，可选值从0到1，默认值为0.92
- 所获取的数据链接，可以用于任何`img`元素，或者将它放在一个有 download 属性的超链接里用于保存到本地。

`canvas.toBlob(callback[, type][, quality])` 创建画布代表图片的Blob对象。callback函数接受blob参数

### ImageData对象

创建：

- `ctx.createImageData(width, height)` 创建新的ImageData对象，所有像素被预设为透明黑。
- `ctx.createImageData(anotherImageData)` 创建与传入对象宽高相同的ImageData对象，所有像素预设为透明黑。
- `ctx.getImageData(left, top, width, height)` 创建包含了画布像素数据的ImageData对象

只读实例属性：

- `width / height` 对象宽度/高度（单位为像素）
- `data` Uint8ClampedArray类型的一维数组，包含RGBA格式的整型数据，值从0至255
  - 获取x行y列像素的R/G/B/A值：`imageData.data[((x * (imageData.width * 4)) + (y * 4)) + 0/1/2/3]`

向场景写入像素数据：

- `ctx.putImageData(ImageData, dx, dy)`

### 优化

- 在离屏 canvas 上预渲染频繁使用的图形，然后将其渲染到主画布上而无需重复生成图像。
- 避免使用浮点数坐标
- 不要使用drawImage缩放图像（在离屏canvas中缓存图片的不同尺寸，而不用drawImage缩放）
- 使用多层画布渲染复杂场景
- 用 CSS 设置大的背景图
- 用 CSS transforms 特性缩放画布（最好不直接缩放画布，或者将较小的画布按比例放大，而不是将较大的画布按比例缩小）
- 如果画布不需要透明度，在使用getContext创建绘图上下文时，将alpha设置为false
- 避免使用shadowBlur特性和text rendering
- 使用合适的方法清除画布（`clearRect()`, `fillRect()`, 调整canvas大小）
- 动画使用`window.requestAnimationFrame()`

