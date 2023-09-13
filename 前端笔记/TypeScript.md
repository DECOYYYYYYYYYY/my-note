# TypeScript

## 类型

### 基础类型

- 布尔值：`boolean`
- 数值：`number`
- 字符串：`string`
- 数组：
  1. 元素类型后面加上`[]`：`number[]`
  2. 数组泛型：`Array<元素类型>`
- 元组：表示一个已知元素数量和类型的数组，各元素的类型不必相同
  - 例：`[string, number]`
  - 访问已知索引的元素时，会得到正确的类型。例：上述数组索引`0`的元素为`string`，索引`1`的元素为`number`
  - 访问越界元素时，用联合类型替代。例：赋值给索引`3`的元素，其类型需为`(string | number)`
- 枚举：为一组数值赋予名字见[枚举](##枚举)
- Any：`any`。类型检查器不对值进行检查
- Unknown：`unknown`。比any更安全，只能赋给unknown和any
- Void：`void`。表示没有任何类型，只能为它赋`undefined`和`null`
- `null`与`undefined`：
  - 默认情况下，`null`和`undefined`是所有类型的子类型。可以把它们赋给任何类型的变量。若指定`--strictNullChecks`标记，`null`和`undefined`只能赋值给`void`和它们自身
- Never：`never`。表示永不存在值的类型
  - 用于类型限制：在switch语句的default中，将联合类型变量赋给一个never变量。若case中没有完全列举联合类型的所有类型，则会报错。
  - 用于函数：该函数无返回值，一般会抛出异常
- Object：`object`。表示非原始类型，即除`number`，`string`，`boolean`，`symbol`，`null`或`undefined`之外的类型
- 类型断言：向编译器断定某个值的类型
  1. 尖括号语法：`<string>value`
  2. `as`语法：`value as string`（在ts中使用JSX时，只能使用as语法）

### 其他类型

- `ReadonlyArray<T>`：只读数组，与`Array<T>`相似，但数组创建后再也不能被修改

### 自动类型推断

赋值类型推断：`let x = [0, 1, null];` 自动推断x的类型为`(number | null)[]`

按上下文归类：按照赋值类型推断相反的方向进行。通常包含函数的参数，赋值表达式的右边，类型断言，对象成员和数组字面量和返回值语句。

```typescript
window.onmousedown = function(e) {
    console.log(e.button);  //<- Error,需要给e指定明确的类型
};
```

### 类型兼容性

```typescript
interface Named {
    name: string;
}
class Person {
    name: string;
}
let p: Named;
p = new Person(); // OK, because of structural typing
```

在java或c#中，这段代码会报错，因为Person类没有明确说明其实现了Named接口。

TypeScript结构化类型系统的基本规则是：如果`x`要兼容`y`，那么`y`至少具有与`x`相同的属性。

```typescript
interface Named {
    name: string;
}

let x: Named;
// y's inferred type is { name: string; location: string; }
let y = { name: 'Alice', location: 'Seattle' };
x = y;
```

函数兼容性判断：函数`x`赋值给函数`y`。（传入参数和返回值可多不可少）

- `x`的每个参数必须能在`y`里找到对应类型的参数。（参数名可以不相同，类型必须相同） 
- `x`的返回值必须是`y`返回值的子类型
- 可选参数不进行兼容性判断
- `y`的每一个重载都要在`x`上找到对应的函数签名

其他兼容性：

- 枚举类型与数字类型相互兼容，但不同的枚举类型之间不兼容
- 类的兼容性判断：
  - 只比较实例部分的成员，不比较静态成员和构造函数
  - 若目标类型有私有/受保护成员，源类型必须包含来自同一个类的私有/受保护成员

### 类型检查

忽略类型检查：

- 忽略下一行：

  ```js
  // @ts-ignore
  ```

- 忽略全文：

  ```js
  // @ts-nocheck
  ```

- 取消忽略全文

  ```js
  // @ts-check
  ```

## 变量声明

`let a: number = 1` 声明变量a，并设置a的类型为`number`

- 若声明和赋值同时进行，且没有指定类型，则默认把值的类型作为变量的类型
- 隐式any：若声明时不赋值不指定类型，则默认视为any

`function xx(a: number): number{}` 指定形参和函数返回值的类型

`let obj: {a: string, b?: number};` 指定对象属性类型

### 别名

`import 别名 = 任意标识符` 为标识符起别名

特点：

- 同时适用于类型和导入的具有命名空间含义的符号。 
- 会生成与原始符号不同的引用，所以改变别名的值并不会影响原始变量的值。

## 操作符

- `typeof 值` 取值的类型。若值为类名，则取其构造函数的类型
- `xx as xx` 将左侧类型断言为右侧类型
  - `对象/数组 as const` 在类型中固定属性值及其类型，转换所有属性为只读 
- `类型A extends 类型B`
  - 用于class：继承
  - 用于接口/泛型：继承/扩展类型
  - 用于代码块：条件类型。若A可以分配给B，返回true

## 接口

### 接口描述普通对象

```typescript
interface LabelledValue {
    label: string;
    width?: number;
    readonly x: number;
}
```

> 上述接口代表了有一个 `label`属性且类型为`string`的对象。类型检查器要求传入的对象存在相应的属性并且类型正确。
>

**可选属性**：在属性名字定义后面加`?`符号

**只读属性**：在属性名前加`readonly`

**额外属性检查**：当对象字面量赋值给变量或作为参数传递时，如果字面量存在任何接口定义时不包含的属性时，会抛出异常。绕过检查的方法有三种：

1. 使用类型断言`{字面量} as 接口`

2. 接口定义时，添加一个索引签名

3. 将字面量赋值给另一个变量，然后使用另一个变量

### 接口描述函数

```typescript
interface SearchFunc {
  (source: string, subString: string): boolean;
}

let mySearch: SearchFunc;
mySearch = function(src: string, sub: string): boolean {
  return src.search(sub) > -1;
}
```

对于函数类型的类型检查来说：

- 函数的参数名不需要与接口里定义的名字相匹配，见上述例子重写了参数名。
- 但要求对应位置上的参数类型时兼容的
- 当函数未指定类型时，会自动根据接口推断参数类型

### 索引签名

```typescript
interface StringArray {
  [index: number]: string;
  [propName: string]: any;
}
```

> 上述例子表示：当用 `number`去索引`StringArray`时会得到`string`类型的返回值。
>
> ts支持两种索引签名：字符串和数字。可以同时使用两种类型的索引，但是数字索引的返回值必须是字符串索引返回值类型的子类型。
>
> 因为当使用 `number`来索引时，JavaScript会将它转换成`string`然后再去索引对象，因此两者需要保持一致。

### 接口描述类

```typescript
interface ClockInterface {
    currentTime: Date;
    setTime(d: Date);
}

class Clock implements ClockInterface {
    currentTime: Date;
    setTime(d: Date) {
        this.currentTime = d;
    }
    constructor(h: number, m: number) { }
}
```

接口只描述类的公共部分，不会检查类的私有成员

描述构造函数时，使用new作为函数名

当类实现了一个接口时，只对其实例部分进行类型检查。 不检查constructor等静态部分。

静态部分类型检查：借用新定义的函数实现

```typescript
interface ClockConstructor {
    new (hour: number, minute: number): ClockInterface;
}
interface ClockInterface {
    tick();
}

function createClock(ctor: ClockConstructor, hour: number, minute: number): ClockInterface {
    return new ctor(hour, minute);
}

class AnalogClock implements ClockInterface {
    constructor(h: number, m: number) { }
    tick() { }
}

let analog = createClock(AnalogClock, 7, 32);
```



例子：

```ts
interface ParseTreeNodeConstructorObj {
    text: string | null;
    parent: ParseTreeNode | null;
    type?: string;
    props?: anyObj;
    children?: Array<ParseTreeNode | string>;
}

class ParseTreeNode {
    text;
    parent;
    type;
    props;
    children; // 构造函数中，该属性有默认值，根据自动类型推导，会剔除undefined类型

    constructor({
        text,
        parent,
        type,
        props = {},
        children = [],
    }: ParseTreeNodeConstructorObj) {
        this.text = text;
        this.parent = parent;
        this.type = type;
        this.props = props;
        this.children = children;
    }
}
```



### 接口继承

```typescript
interface Shape {
    color: string;
}

interface PenStroke {
    penWidth: number;
}

interface Square extends Shape, PenStroke {
    sideLength: number;
}
```

接口继承了一个类时，会继承类的成员但不包括其实现（会同时继承类的private和protechted成员）

### 混合类型

一个对象可以同时作为函数和对象使用

```typescript
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}

function getCounter(): Counter {
    let counter = <Counter>function (start: number) { };
    counter.interval = 123;
    counter.reset = function () { };
    return counter;
}

let c = getCounter();
```

### 接口合并

同名接口会合并，其中相同的函数成员会被视为重载。重载时，后写的函数优先级高。（若有一个参数类型为非联合的字符串字面量，它会被提到重载列表最前端）

```typescript
interface Cloner {
    clone(animal: Animal): Animal;
}
interface Cloner {
    clone(animal: Sheep): Sheep;
}
interface Cloner {
    clone(animal: Dog): Dog;
    clone(animal: Cat): Cat;
}
// 合并为
interface Cloner {
    clone(animal: Dog): Dog;
    clone(animal: Cat): Cat;
    clone(animal: Sheep): Sheep;
    clone(animal: Animal): Animal;
}
```

## 类

继承、静态属性等语法同JavaScript

### 访问权限修饰符

- `public`（默认）：可以自由访问该成员
- `private`：无法在声明它的类的外部访问
- `protected`：只能在类内部及派生类中访问
  - 构造函数标记为 `protected`：这个类不能在包含它的类外被实例化，但是能被继承。
- `readonly`：属性为只读，必须在声明时或构造函数里被初始化
- 在构造函数形参前，添加一个访问限定符可以进行成员声明

### 对象

对象的类型描述：

- 通过接口描述

- 通过字面量描述

  ```ts
  type Props = {
      name: string;
      age: number;
  };
  ```



### 抽象类

抽象类做为其它派生类的基类使用，一般不会直接被实例化。不同于接口，抽象类可以包含成员的实现细节。抽象类用`abstract`关键字定义。

```typescript
abstract class Animal {
    abstract makeSound(): void;
    move(): void {
        console.log('roaming the earch...');
    }
}
```

抽象类中的抽象方法不包含具体实现并且必须在派生类中实现。抽象方法必须包含 `abstract`关键字并且可以包含访问修饰符。

### 高级技巧

类定义会创建两个东西：类的实例类型和一个构造函数。 

```typescript
let greeter1: Greeter; // greeter1的实例类型为Greeter类
let greeterMaker: typeof Greeter = Greeter;  // greeterMaker的实例类型为函数
```

因为类可以创建出类型，所以能够在允许使用接口的地方使用类。

```typescript
interface Point3d extends Point {
    z: number;
}
```



将类作为接口：mixins，创建占位属性，仅适用类型而非其实现

```typescript
class Activatable {
    isActive: boolean;
    activate() {
        this.isActive = true;
    }
    deactivate() {
        this.isActive = false;
    }
}
class SmartObject implements Activatable {}

// 占位属性
// isActive: boolean = false;
// activate: () => void;
// deactivate: () => void;
```

## 函数

### 类型定义

**函数的类型定义：**

```typescript
function add(x: number, y: number): number {
    return x + y;
}
```

- 不指定函数返回值类型时，ts会自动根据返回值推断类型

**完整的函数类型定义：**

```typescript
let myAdd: (x: number, y: number) => number =
    function(x: number, y: number): number { return x + y; };
```

- 完整的定义必须包含参数类型和返回值类型
- 用`=>`分隔参数类型和返回值类型
- 按上下文归类：只在赋值语句的一边指定了类型时，ts编译器会自动识别类型

### 函数参数

普通参数：对于函数，编译器会检查用户是否为每个参数都传入了值。

可选参数：在参数名后加`?`表示参数可选。

- 可选参数必须在普通参数之后

默认参数：默认参数都是可选参数。语法：`参数名=默认值`。

- 默认参数若在普通参数之前，使用时必须明确地传入`undefined`来获取默认值

  ```typescript
  function buildName(firstName: string, lastName?: string, nickname="rookie") {}
  ```

剩余参数：用法同rest参数

`this`参数：`this`参数是个假的参数，它定义在参数列表的最前面，用于指示`this`的类型。`fn(this: 类型)`

### 重载

```typescript
function fn(x: number): number; // 重载1
function fn(x: string): string; // 重载2
function fn(x): any {
    if (typeof x == "number") {
        return 1;
    }
    else if (typeof x == "string") {
        return "1";
    }
}
```

`function fn(x): any`并不是重载列表的一部分，因此这里只有两个重载：一个是接收字符串，另一个接收数字。 以其它参数调用 `fn`会产生错误。

## 泛型

### 泛型函数

```typescript
function identity<T>(arg: T): T {
    return arg;
}
identity<string>('str');
```

类型变量：一种特殊的变量，只用于表示类型而不是值，定义在函数的括号前。

使用泛型函数时：用尖括号传入类型变量，若不传入，ts会自动从入参判断类型。

### 泛型接口

```typescript
interface genericFn<T> {
    (arg: T): T;
}
function fn<T>(arg: T): T {
    return arg;
}
let numberFn: genericFn<number> = fn;
```

### 泛型类

```typescript
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };
```

### 泛型约束

```typescript
interface Lengthwise {
    length: number;
}
function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);
    return arg;
}
```

继承一个接口，约束泛型的结构，传入的类型变量必须符合要求

在泛型约束中使用类型参数：

```ts
function getProperty(obj: T, key: K) {
    return obj[key];
}
let x = { a: 1, b: 2, c: 3, d: 4 };
getProperty(x, "a"); // okay
getProperty(x, "m"); // error: Argument of type 'm' isn't assignable to 'a' | 'b' | 'c' | 'd'.
```

使用泛型创建工厂函数：需要引用构造函数的类类型。

```ts
function create<T>(c: {new(): T; }): T {
    return new c();
}
```

## 枚举

### 创建

数字枚举：

```typescript
enum Direction {
    Up = 1,
    Down, // 2
    Left, // 3
    Right // 4
}
```



字符串枚举：

每个成员都必须用字符串字面量，或另外一个字符串枚举成员进行初始化。

```typescript
enum Direction {
    Up = "UP",
    Down = "DOWN"
}
```



异构枚举：枚举可以混合字符串和数字成员，但不推荐这么做

### 特性

特性：

- 枚举是在运行时真正存在的对象
- 反向映射：
  - 获取枚举成员的值：`枚举名.枚举成员名`
  - 获取值对应的枚举成员名：`枚举名[值]`
  - 字符串枚举成员不支持反向映射

枚举成员的值：

- 位于首位且未初始化的成员，其值为`0`
- 未初始化且上一位成员的值是数字常量，其值为上一位成员的值加1
- 枚举成员使用**常量枚举表达式**初始化，其值为对应常量
- 所有其它情况的枚举成员被当作是需要计算得出的值。

字面量枚举成员：未被初始化的成员，或值被初始化为以下值的成员

- 任何数字、字符串字面量
- 应用了一元 `-`负号的数字字面量

联合枚举与枚举成员的类型：

- 当所有枚举成员的值都为字面量枚举值时：

  - 枚举成员将成为类型

    ```typescript
    enum Test {a,b}
    let a: Test.a = Test.a; // a=0
    let a: Test.a = Test.b; // 报错
    ```

  - 枚举类型本身变成了每个枚举成员的联合

    ```typescript
    enum Test {a,b}
    let a: Test = Test.a; // a=0
    let b: Test = Test.b; // b=1
    ```

常量枚举：`const enum...`使用`const`修饰符定义枚举，其成员值只能是常量

- 常量枚举会在编译阶段被删除

外部枚举：描述已经存在的枚举类型的形状。

```ts
declare enum Enum {
    A = 1,
    B,
    C = 2
}
```

外部枚举和非外部枚举之间有一个重要的区别，在正常的枚举里，没有初始化方法的成员被当成常数成员。 对于非常数的外部枚举而言，没有初始化方法时被当做需要经过计算的。

## 高级类型

交叉类型：表示值包含了所需的所有类型的特性。`类型1 & 类型2`

联合类型：表示值是几种类型之一。`类型1 | 类型2`

### 类型保护与区分类型

对于联合类型，默认情况下只能访问联合类型中共同拥有的成员，若要访问其中某个类型的成员，需要进行类型断言。

```typescript
let pet = getSmallPet();

if ((<Fish>pet).swim) {
    (<Fish>pet).swim();
}
else {
    (<Bird>pet).fly();
}
```



自定义类型保护：定义一个函数，其返回值是一个类型谓词`当前函数的某个参数 is 类型`。变量调用该函数时，变量会缩减为那个类型

```typescript
function isFish(pet: Fish | Bird): pet is Fish {
    return (<Fish>pet).swim !== undefined;
}

if (isFish(pet)) {
    pet.swim();
}
else {
    pet.fly();
}
```



`typeof`类型保护：ts会把`typeof 值 === "类型"`或`typeof 值 !== "类型"`识别为一个类型保护，从而无需按照上述定义函数。

- 会被视作类型保护的字符串：`"number"`， `"string"`， `"boolean"`或 `"symbol"`



`instanceof`类型保护：同`typeof`类型保护

- js要求`instanceof`右侧为一个构造函数，而ts要求：该构造函数的prototype属性类型（不可为any）或构造签名所返回的类型的联合



`--strictNullChecks`标记：

- 声明一个变量时，它不会自动地包含 `null`或 `undefined`
- 可选参数会被自动地加上`| undefined`



去除`null`与`undefined`：在变量名结尾添加符号`!`表示从该变量可能的类型中去除`null`和`undefined`

### 类型别名

`type 别名 = 任何类型;`

类型别名也可以是泛型：`type Container<T> = { value: T };`

特点：

- 类型别名是对类型的引用，不会创建新类型。因此错误信息不会显示类型别名
- 类型别名无法被`extends`和`implements`，也无法使用`extends`和`implements`

### 字面量类型

字符串字面量类型：`type 类型名 = "字符串1" | "字符串2";` 指定字符串必须的固定值

- 可用于区分重载

  ```typescript
  function createElement(tagName: "img"): HTMLImageElement;
  function createElement(tagName: "input"): HTMLInputElement;
  // ... more overloads ...
  function createElement(tagName: string): Element {
      // ... code goes here ...
  }
  ```

数字字面量类型：同字符串字面量类型

### 可辨识联合

> 单例类型，多数是指枚举成员类型和数字/字符串字面量类型

辨识联合的三要素：

1. 具有普通的单例类型属性 — 可辨识的特征
2. 一个类型别名包含了那些类型的联合 — 联合
3. 此属性上的类型保护

要联合的接口需要有可辨识标签：即下例中的`kind`属性

```typescript
interface Square {
    kind: "square";
    size: number;
}
interface Rectangle {
    kind: "rectangle";
    width: number;
    height: number;
}
interface Circle {
    kind: "circle";
    radius: number;
}
type Shape = Square | Rectangle | Circle; // 联合
// 使用可辨识联合
function area(s: Shape) {
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.height * s.width;
        case "circle": return Math.PI * s.radius ** 2;
    }
}
```

### 多态this类型

多态的this类型表示的是实例本身

```typescript
class BasicCalculator {
    public constructor(protected value: number = 0) {}
    public add(operand: number): this {
        this.value += operand;
        return this;
    }
}
class ScientificCalculator extends BasicCalculator {
    public constructor(value = 0) {
        super(value);
    }
    public sin() {
        this.value = Math.sin(this.value);
        return this;
    }
}
let v = new ScientificCalculator(2).sin().add(1)
```

### 索引类型

```typescript
function pluck<T, K extends keyof T>(o: T, names: K[]): T[K][] {
  return names.map(n => o[n]);
}
interface Person {
    name: string;
    age: number;
}
let person: Person = {
    name: 'Jarid',
    age: 35
};
let strings: string[] = pluck(person, ['name']); // ok, string[]
```

索引类型查询操作符：`keyof 类型`

- 结果为指定类型上已知的公共属性名的联合
- 见上例：`keyof Person` 完全等效于 `'name' | 'age'`

索引访问操作符：`T[K]`

- `T[K]`操作符所表示的类型与`T`中`K`属性的类型相同

### 映射类型

在映射类型里，新类型以相同的形式去转换旧类型里每个属性

```typescript
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
}
type Partial<T> = {
    [P in keyof T]?: T[P];
}
type PersonPartial = Partial<Person>; // 转换所有属性为可选
type ReadonlyPerson = Readonly<Person>; // 转换所有属性为只读
```

使用 `in` 关键字：内部使用 `for ... in` 依次绑定属性

```typescript
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
}
type Record<K extends string, T> = {
    [P in K]: T;
}
```

预定义的条件类型：（已被包含进了ts标准库中）

- `Readonly<T>`、 `Partial<T>`、`Pick`、`Record`

- `Exclude<T, U>` -- 从`T`中剔除可以赋值给`U`的类型。

- `Extract<T, U>` -- 提取`T`中可以赋值给`U`的类型。

- `NonNullable<T>` -- 从`T`中剔除`null`和`undefined`。

- `Omit<T, K extends keyof any>`：从接口T中剔除K所描述的属性

  ```ts
  interface Person {
    name: string;
    age: number;
    address: string;
  }
  type WithoutAge = Omit<Person, 'age'>; // 移除age属性
  type WithoutAgeAndAddress = Omit<Person, 'age' | 'address'>; // 移除多个属性
  ```

- `ReturnType<T>` -- 获取函数返回值类型。

- `InstanceType<T>` -- 获取构造函数类型的实例类型。

### 类型声明

https://blog.csdn.net/to_the_Future/article/details/127217913

declare关键字：

- 用来为已存在的 JS 库提供类型信息
- 类型声明在编译的时候都会被删除，不会影响真正的代码
- declare 是给编译器读取用的，可以暂时不调用

类型声明：

```typescript
declare let age: number;
declare function getAge(): number | string;
declare class Person { };
declare enum Seasons {...};
declare namespace $ {...};
declare interface XX {...};
```

- 为已有类型进行declare声明时，会将声明的类型扩展给原有类型

类型声明文件：类型声明通常单独放在`xxx.d.ts`中

- package.json中： `"typings": "xxx.d.ts"`

## 模块

`import type` 类似import，仅导入类型，在运行时不存在



TS同样支持ES5模块语法

为了支持CommonJS和AMD的`exports`, TypeScript提供了`export =`语法。

`export =`语法定义一个模块的导出`对象`。 这里的`对象`一词指的是类，接口，命名空间，函数或枚举。

若使用`export =`导出一个模块，则必须使用TypeScript的特定语法`import module = require("module")`来导入此模块。ZipCodeValidator.ts

**ZipCodeValidator.ts**

```ts
let numberRegexp = /^[0-9]+$/;
class ZipCodeValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
export = ZipCodeValidator;
```

**Test.ts**

```ts
import zip = require("./ZipCodeValidator");

// Some samples to try
let strings = ["Hello", "98052", "101"];

// Validators to use
let validator = new zip();

// Show whether each string passed each validator
strings.forEach(s => {
  console.log(`"${ s }" - ${ validator.isAcceptable(s) ? "matches" : "does not match" }`);
});
```

编译为指定的模块目标：`tsc --module 模块目标 Test.ts`

- `commonjs`、`amd`等

### 命名空间

使用`namespace`关键字声明命名空间，使用`export`关键字暴露对象，未导出的变量在外部无法访问

```typescript
namespace Validation {
    const lettersRegexp = /^[A-Za-z]+$/;
    export interface StringValidator {
        isAcceptable(s: string): boolean{
            return lettersRegexp.test(s);
        };
    }
}

let validators: Validation.StringValidator = new Validation.StringValidator;
```

使用时：`命名空间名.xxx`

命名空间引用：

- 方法一：将命名空间所在文件引入`import 'url';`
- 方法二：向编译器声明关联`/// <reference path="url" />`
  - 为了确保代码正常加载：将所有输入文件编译为一个输出文件`tsc --outFile 输出文件名 内部声明了关联的ts文件名`
  - 不可以在模块中使用`/// <reference>`

模块与命名空间：不应该对模块使用命名空间

合并命名空间：后来的命名空间中的导出成员 会被加到 已有的命名空间中（命名空间之间仍不可互相访问非导出成员）