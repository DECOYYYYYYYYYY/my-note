# JavaSE


## 语言基础

JDK、JRE与JVM：

- JDK（Java Development Kit，Java开发工具包）
- JRE（Java Runtime Environment，Java运行环境）
- JVM（Java Virtual Machine，Java虚拟机）
- JDK = JRE + 开发工具集（例如Javac编译工具等）
- JRE = JVM + Java SE标准类库

### 主类结构

- 每一个应用程序都必须包含一个main()方法，含有main()方法的类称为主类。
- 使用package语句声明类所在的包
- 通常将类的属性称为类的全局变量（成员变量），将方法中的属性 称为局部变量。
- 通过import关键字导入相关的类

### 基本数据类型

- 整型：
  - `byte`（单字节）[-128, 127]
  - `short`（2字节）[-32768, 32767]
  - `int`（4字节，默认）[-2.14e9, 2.14e9]
  - `long`（8字节）[9.22e18, 9.22e18]
    - 对于`long`型值，若赋给的值超过`int`型的范围，必须在数字后加`l`或`L`
    - 二进制： 0b或0B开头；八进制：以数字0开头；十六进制：以0x或0X开头
- 字符型：
  - `char`（Unicode码，2字节）
    - 使用一对单引号，内部只能写一个字符
    - 也可以将unicode码赋给`char`型变量
  - 转义字符：特殊的字符
    - `\ddd` 1-3位八进制数据所表示的字符
    - `\uxxxx` 4位十六进制数据所表示的字符
    - `\'、\\、\t、\r、\n、\b、\f`
- 浮点型：
  - `float`（4字节，6位有效数字）[1.4E-45, 3.4E38]
  - `double`（8字节，15位有效数字，默认）[4.9E-324, 1.79E308]
    - 声明`float`型变量，必须以`f`或`F`结尾，否则jvm默认先赋给`double`再转成`float`，可能损失精度。
- 布尔型：`boolean`（小写true、false）
- 包装类：`Byte`、`Short`、`Integer`、`Long`、`Float`、`Double`、`Boolean`、`Character`

类型转换：

- 隐式类型转换：系统自动完成，只有转换过程不损失信息时才会转换（不涉及boolean）
  - 不同数据类型运算时会自动按照不损失信息的方向将所有类型转换为同一类型
  - 类型精度：`byte < short < int < long < float < double`
  - 当不同精度的数据类型进行运算时，结果自动提升为精度高的数据类型
- 强制类型转换：`(目标类型) 值`，强制将值转换为目标类型
- `String` 与8种基本数据类型间的运算
  - String属于引用数据类型，翻译为：字符串
  - 声明String类型变量时，使用一对 `""`
  - String可以和8种基本数据类型变量做运算，且运算只能是连接运算：`+`
  - 运算的结果仍然是String类型

### 变量与常量

标识符：一个名字，用来标识类名、变量名、方法名、数组名、文件名的有效字符序列

标识符规则：（必须要遵守。否则，编译不通过）

- 由`unicode字符`，`0-9`，`_`或`$`组成。
- 开头不可以是数字。
- 不可以使用关键字和保留字，但能包含关键字和保留字。
- 标识符不能包含空格。

标识符命名规范：

- 包名：全小写
- 类名、接口名：大驼峰
- 变量名、方法名：小驼峰
- 常量名：所有字母都大写。每个单词用下划线连接： `XXX_YYY_ZZZ`

关键字：被Java语言赋予了特殊含义，用做专门用途的字符串（单词）

保留字：现Java版本尚未使用，但以后版本可能会作为关键字使用，包括：`goto` 、`const`



声明变量：为变量分配内存`数据类型 变量名;`

- 变量名必须是一个有效的标识符

变量初始化：

- `数据类型 变量名 = 值;` 
- `数据类型 变量名1,变量名2 = 值;` 
- `数据类型 x = 3.5, y=1.2`

常量：只能被赋值一次。初始化方式：`final 数据类型 变量名 = 值;`

变量的有效范围：

1. 成员变量

   在类体中所定义的变量被称为成员变量，成员变量在整个类中都有 效。类的成员变量又可分为两种，即静态变量和实例变量。静态变量的有效范围可以跨类。

2. 局部变量

   在类的方法体中定义的变量称为局部变量。局部变量只在当前代码块中有效。

### 运算符

算术运算符：`+ -(正负) + -(加减) * / % ++ -- +(连接符)`

- 相同类型的数据进行运算，结果类型不变。如：`int / int = int`，向0取整
- 乘方没有运算符，通过`Math.pow(底数，幂数)`实现或左右移幂数位

赋值运算符：`= += -= *= /= %=`

关系运算符: `== != > < >= <= instanceof`

- `==`运算符
  - 可比较基础数据类型（比较值）、引用数据类型（比较地址）
  - 需保证符号两边的变量类型一致，引用数据类型推荐用equals

逻辑运算符：`& && | || ! ^`

- 当符号左边是`false`时，`&`继续执行符号右边的运算。`&&`不再执行符号右边的运算。

位运算符：`<< >> >>>【无符号右移】 & | ^【异或】 ~【取反】`

- 操作对象为整型或字符型数据
- 左移空位补0；右移时最高位为0，则空位补0，否则补1；无符号右移空位补0

三元运算符：`条件表达式 ? 表达式1 : 表达式2`

`instanceof`运算符：`对象 instanceof 类或接口`

- 若对象是指定类/接口的实例 或 对象兼容于右侧类型，返回true

优先级：`括号→一元运算符（逻辑非！）→算数运算符→移位→关系运算符→逻辑与&&→逻辑或||`

### 代码注释

单行注释：`//`

多行注释：`/* */`

文档注释：`/** */`

### 流程控制

if-else、switch-case、for、while、do-while

break和continue关键字：

```java
标签名: 循环体{
    break 标签名; // 跳出外层循环
}
```

for-each循环：`for(声明语句: 表达式){}` 表达式需返回数组


Scanner类：

1. 导包：`import java.util.Scanner;`
2. Scanner的实例化：`Scanner scan = new Scanner(System in);`
3. 调用Scanner类的相关方法（`next()、nextInt()、nextFloat()...`），来获取指定类型的变量；

## 集合

### 数组

数组（Array）是相同类型数据的集合。

数组名 元素 角标、下标、索引 数组的长度：元素的个数

特点：

- 数组是序排列的
- 数组属于引用数据类型的变量。数组的元素，既可以是基本数据类型，也可以是引用数据类型
- 创建数组对象会在内存中开辟一整块连续的空间
- 数组的长度一旦确定，就不能修改

创建：

- 声明：`元素类型[] 数组名; `（推荐）或 ` 元素类型 数组名[]; `
- 赋值：`数组名 = new 元素类型[数组长度];`
- 初始化：`元素类型[] 数组名 = new 元素类型[数组长度];`
- 直接通过元素初始化：`元素类型[] 数组名 = {值0, 值1, ...};`
- 多维数组初始化：`元素类型[][] 数组名 = new 元素类型[长度][长度];`

实例属性：

- `int length`



#### Arrays工具类

导包：`java.util.Arrays`

用于操作数组，提供静态方法：

- `public static int binarySearch(Object[] a, Object key)`
  - 二分查找给定值，若存在，返回索引，否则返回`(-(插入点)-1)`
  - 数组必须有序
  - `public static int binarySearch(Object[] a, int fromIndex, int toIndex, type key)` 指定搜索区域
- `public static boolean equals(long[] a, long[] a2)`
  - 判断两个数组是否相等，相等则返回true
  - 若两个数组内的元素都相等，则认为数组相等
- `public static void fill(int[] a, int val)`
  - 将指定的 int 值分配给指定 int 型数组指定范围中的每个元素
  - 适用于所有基本数据类型
- `public static void sort(Object[] a)`
  - 根据数组元素的自然顺序进行升序排列。适用于所有基本数据类型



### Stream流

> 以声明式的方式处理数据，对stream的操作不会对源数据产生影响

获取流：

```java
// List集合获取流
Stream<String> stream1 = new ArrayList<>().add("张老三").stream();

// Set集合获取流
Stream<String> stream2 = new HashSet<>().add("张老三").stream();

// Map集合获取流
Map<Integer,String> map = new HashMap<>().put(1,"张老三");
Stream<Integer> stream3 = map.keySet().stream(); // 根据键获取流
Stream<String> stream4 = map.values().stream(); // 根据值获取流
Stream<Map.Entry<Integer, String>> stream5 = map.entrySet().stream(); // 根据键值对对象获取流

// 数组获取流
String[] arr = {"张颜宇","张三","李四","赵五","刘六","王七"};
Stream<String> stream6 = Stream.of(arr);
```



流的方法：

> 函数式接口会接收一个参数，为每次迭代时流中的元素

```java
long count(); // 返回流中元素个数

void forEach(函数式接口); // 逐一处理流中的元素

Stream<T> filter(函数式接口); // 过滤元素，保留函数返回值为true的元素

Stream<T> limit(long maxSize); // 当流的长度大于参数时，截取流至maxSize长度

Stream<T> map(函数式接口); // 映射元素，将函数返回值作为新元素

Stream<T> skip(long n); // 跳过前n个元素

Stream<T> distinct(); // 去重，根据元素的hashCode和equals方法

stream.collect(Collectors.toList()) // 将stream收集到List中

public static Stream<T> concat(Stream<T> a, Stream<T> b); // 合并流，要求类型一致

```



## 面向对象

类方法声明：`访问权限修饰符 非访问权限修饰符 返回类型 方法名(参数类型 参数名,...)`

- 例：`public static void main(String[] args)`
- 修饰符为可选项

类变量声明：`访问权限修饰符 非访问权限修饰符 变量类型 变量名 = 值;`



### 修饰符

访问权限修饰符

| 修饰符    | 类内部 | 同一个包 | 不同包的子类 | 同一个工程 |
| --------- | ------ | -------- | ------------ | ---------- |
| private   | Yes    |          |              |            |
| （缺省）  | Yes    | Yes      |              |            |
| protected | Yes    | Yes      | Yes          |            |
| public    | Yes    | Yes      | Yes          | Yes        |

- 4种权限都可以用来修饰**类的内部结构**：属性、方法、构造器、内部类
- 修饰类时只能使用：缺省、public

非访问权限修饰符

- `static`：修饰类方法和类变量，声明 方法/变量 为 静态方法/变量
- `final`：修饰类、方法和变量
  - 修饰的类不能够被继承
  - 修饰的方法不能被继承类重新定义
  - 修饰的变量为常量，是不可修改的
- `abstract`：修饰类方法和类变量，创建抽象类和抽象方法
- `synchronized`：声明的方法同一时间只能被一个线程访问
- `volatile`：修饰类变量
  - 在每次被线程访问时，都强制从共享内存中重新读取该成员变量的值
  - 当成员变量发生变化时，会强制线程将变化值回写到共享内存
  - 这样在任何时刻，两个不同的线程总是看到某个成员变量的同一个值。
- `transient`：修饰类变量，序列化的对象包含被修饰的实例变量时，JVM跳过该变量



### 继承

`class 子类A extends 父类B{}`

super关键字：

- super作为非方法使用时，可以看做父类的this；作为方法时，可视为父类的构造函数
- 子类必须在构造函数中调用`super()`方法
  - 子类通过`super()`方法获取了父类中声明的所有的属性和方法
  - 虽然调用了父类的构造器，但始终只创建了一个对象（子类对象）

类的继承特点：

- 子类继承父类以后，还可以声明自己特有的属性或方法，实现功能的拓展。
- 子类继承父类以后，认为同时获取了父类中的私有属性和方法。只因为封装性的影响，使得子类不能直接调用父类的结构而已。

- 已被重写的父类成员，可以通过super对象访问

- 类不支持多继承、接口支持多继承



### 多态

重写（overriding）：子类中定义了与父类中的同名的新方法

- 参数列表、方法名必须完全相同，返回的类型必须是原返回类型或其子类
- 访问权限不能小于原方法，不能抛出更宽泛的检查型异常，构造函数不能被重写
- 在运行时确定（运行时多态）

重载（overloading）：在同一个类中，定义了同名的新方法

- 方法名必须完全相同，参数列表必须不一样
- 在编译时确定（编译时多态）



### Object类

即`java.lang.Object`类。

特点：

- 是java中所有类的根父类
- 若类声明中未使用extends关键字指明其父类，则默认父类为Object类

相关方法：

- `public Object()` 构造器
- `public boolean equals(Object obj)` 对象比较，仅适用于引用数据类型。
  - 和`==`的作用相同：比较两个对象的地址值是否相同
  - `String`、`Date`、`File`、`包装类`等重写了该方法：比较实体内容是否相同
- `public int hashCode()`获取Hash码
- `public String toString()`对象打印时调用
  - `String`、`Date`、`File`、`包装类`等重写了该方法：返回对象的实体内容



### 抽象类

抽象类不能被实例化为对象

使用abstract关键字定义抽象类：`public abstract class Employee{}`

抽象方法：`public abstract double computePay();`

- 如果一个类包含抽象方法，那么该类必须是抽象类
- 任何子类必须重写父类的抽象方法，或者声明自身为抽象类



### 接口

特点：

- 接口可以使用`extends`关键字继承接口，且允许多继承
- 接口是隐式抽象的，当声明一个接口的时候，不必使用 abstract 关键字
- 接口中每一个方法也是隐式抽象的，声明时同样不需要 abstract 关键字
- 接口中的方法都是公有的
- 当类实现接口的时候，类要实现接口中所有的方法。否则，类必须声明为抽象类

声明：`public interface NameOfInterface{ 声明变量、声明抽象方法 }`

接口实现：`public class 类名 implements 多个接口{}`

标记接口：没有任何方法的接口被称为标记接口，主要用于：

- 建立一个公共的父接口：

  正如EventListener接口，这是由几十个其他接口扩展的Java API，你可以使用一个标记接口来建立一组接口的父接口。例如：当一个接口继承了EventListener接口，Java虚拟机(JVM)就知道该接口将要被用于一个事件的代理方案。

- 向一个类添加数据类型：

  这种情况是标记接口最初的目的，实现标记接口的类不需要定义任何接口方法(因为标记接口根本就没有方法)，但是该类通过多态性变成一个接口类型。

函数式接口：有且只有一个抽象方法，但可以有多个非抽象方法的接口



### 枚举

是一个特殊的类，表示一组常量

特点：

- 枚举跟普通类一样可以用自己的变量、方法和构造函数，构造函数只能使用 private 访问修饰符，所以外部无法调用。
- 枚举既可以包含具体方法，也可以包含抽象方法。 如果枚举类具有抽象方法，则枚举类的每个实例都必须实现它。



创建：

```java
enum Color { RED, GREEN, BLUE;}  // 未显式声明基础类型的枚举，默认基础类型为int

// 使用时
Color c1 = Color.RED; // c1的值为RED
c1 == RED; // true
```



枚举类相关方法：

- `static List<T> values()`：返回枚举类中所有的成员。
- `static T valueOf(String value)`：返回指定字符串所对应的枚举实例。如`Color.valueOf("RED")`返回Color.RED实例
- `ordinal()`：返回当前枚举实例的索引，与数组索引相似。



自定义枚举类属性：

```java
enum WeekDay {
    
    // 成员声明
    Mon("Monday", 1),Tue("Tuesday", 2),Wed("Wednesday", 3),Thu("Thursday", 4);
    
    // 属性声明
    private String day;
    private Integer index;
    
    // 构造函数
    private WeekDay(String day, Integer index) {
        this.day = day;
        this.index = index;
    }

    // getter
    public String getDay() {
        return day;
    }
    public String getIndex() {
        return index;
    }
}

WeekDay.Mon.getIndex(); // 结果：1
```



## 注解

> 注解是附加在代码中的一些元信息，用于一些工具在编译、运行时进行解析和使用，起到说明、配置的功能
>
> 注解传值：
>
> - 简单传值：`@...(值)` 将值传给value属性
> - 完整传值：`@...(属性名1=值1, 属性名2=值2)` 传给指定属性

### 标准注解

- `@Override`：标记一个方法为重写方法
- `@Deprecated`：标记类、成员、方法已废弃、过时，在编译时发出警告
- `@SuppressWarnings("警告类型")`：关闭对类、方法、成员编译时产生的警告
  - 指定多个警告类型：`@SuppressWarnings(value={"...", "..."}) `
  - 可选警告类型：
    - `deprecation`：使用了过时方法的警告
    - `unchecked`：执行了未检查的转换时的警告
    - `fallthrough`：Switch语句直接通往下一个情况而没有Break的警告
    - `path`：类路径、源文件路径有不存在路径的警告
    - `serial`：在可序列化的类上缺少serialVersionUID定义时的警告
    - `finally`：任何finally子句不能正常完成时的警告
    - `all`：所有警告
- `@FunctionalInterface`：标记接口是函数式接口，JDK8+

### 元注解

用于修饰注解的注解

- `@Retention(RetentionPolicy的枚举值)`：标记注解在哪个级别可用
  - `RetentionPolicy.SOURCE` 源代码中、`.CLASS` 默认值，类文件中、`.RUNTIME` 运行时
- `@Documented`：生成文档信息时保留注解
- `@Target(ElementType的枚举值 或 {多个值})`：标记注解的使用范围
  - `.TYPE`：应用于类、接口（包括注解类型）、枚举
  - `.CONSTRUCTOR`：应用于构造函数
  - `.PARAMETER`：应用于方法的参数
  - `.FIELD`：应用于字段或属性
  - `.METHOD`：应用于方法
  - `.PACKAGE`：应用于包
  - `.LOCAL_VARIABLE`：应用于局部变量
  - `.TYPE_PARAMETER`：v1.8+，应用于类型变量
  - `.TYPE_USE`：v1.8+，应用于任何类型的语句中
- `@Inherited`：标记子类自动继承父类中的该注解
- `@Repeatable`：标记注解可以重复使用

### 自定义注解

```
// 元注解
public @interface 注解名称{
    // 属性列表
}
```

与创建一个接口相似，但是注解的interface关键字需要以@符号开头

## 方法

可变参数：它必须是方法的最后一个参数，在该参数的类型后加`...`，参数值为由所有余下值组成的数组

`finalize()`方法：在对象销毁前会调用该方法，通常格式为`protected void finalize(){}`



### Lambda表达式

> 函数式接口：内部有且只有一个抽象方法的接口，Lambda表达式可以创建该接口的对象
>
> 类型推断机制：在上下文信息足够的情况下，编译器可以推断出参数表的类型，而不需要显式指名

语法格式：

```java
([类型 ]参数名, ...) -> { 执行语句 } // 完整形式
[类型 ]参数名 -> { 执行语句 } // 只有一个参数时，可以省略括号
([类型 ]参数名, ...) -> 表达式或单条语句 // 此时可以省略花括号，将表达式或语句的值作为返回值
```

方法引用：

```java
// 静态方法和实例方法的引用
([变量1, 变量2, ...]) -> 类名或实例.方法名([变量1, 变量2, ...]) // 原
类名或实例::方法名 // 简化形式

// 调用了参数1的实例方法时
(变量1[, 变量2, ...]) -> 变量1.实例方法([变量2, ...]) // 原
变量1的类名::实例方法名 // 简化形式
// 如：Collections.sort(list, (o1, o2) -> o1.compareTo(o2));
// 可简化为：Collections.sort(list, Integer::compareTo);

// 引用构造方法
([变量1, 变量2, ...]) -> new 类名([变量1, 变量2, ...]) // 原
类名::new // 简化形式

```



## 内置类

### Number

Integer、Long、Byte、Double、Float、Short 都是抽象类 Number 的子类（派生类）

实例方法：

- `xxxValue()` 返回转换为xxx类型后的值（xxx：byte / double / float ...）
- `public int compareTo(NumberSubClass num)` 比较数据
  - 相等返回`0`、实例大于参数返回`1`、实例小于参数返回`-1`
- `public boolean equals(Object o)` 比较是否类型和值是否都相等
- `String toString()`

静态方法：

- `static Integer valueOf(int i)`解析并返回整数
  - 参数也可以是：`String s` 或 `String s, int radix`
  - 除了Integer外，其他派生类也有该方法，返回对应类型的值
- `static String toString(int i)` 所有派生类都有该方法
- `static int parseInt(String s)` 所有派生类都有该方法，其函数名为`parseDouble`等

### Character

是char的包装类

装箱：`Character ch = 'a'; 或 Character ch = new Character('a');`

拆箱：`char c = Character实例`

静态方法：

- `public static boolean isLetter(char ch)` 判断字符是否为字母
  - 相似方法：`isDigit`、`isWhitespace` 是否为空白符（空格、tab 键、换行符）、`isUpperCase`、`isLowerCase`
- `public static char toUpperCase(char ch)` 转换为大写，若不能转换，返回字符本身
  - 相似方法：`toLowerCase`
- `public static String toString(char ch)` 返回字符串

### String

创建字符串：

1. `String str = "str"`
2. ` String str = new String("str");`
   - 参数也可以是：字符数组

注意：

- String对象在创建后不可改变，修改变量时会创建新的对象，并更新引用
- 字符串为引用类型

实例方法：

- `int length()` 返回长度
- `String concat(String str2)` 返回 实例后拼接了str2 的新字符串
  - 等效于`字符串1 + 字符串2`
- `int indexOf(int ch [,int fromIndex])` 返回字符第一次出现时的索引
  - 参数可也为：`(String str [,int fromIndex])`
  - 相似方法：`lastIndexOf` 返回最后一次出现时的索引
- `boolean matches(String regex)` 字符串匹配给定正则时，返回true
- `equals(String str2)` 判断字符串是否相等
- `String replace(char searchChar, char newChar)` 返回替换所有指定字符后的新字符串
- `String replaceAll(String regex, String replacement)` 使用正则替换所有匹配字符串，成功返回替换的字符串，失败返回原字符串
  - 相似方法：`replaceFirst`仅替换第一个匹配字符串
- `String[] split(String regex [,int limit])` 拆分字符串
- `String substring(int beginIndex [,int endIndex])` 返回子串（不包括结束索引）

静态方法：

- `String format(String template, any args)` 返回格式化后的字符串
  - 在模板字符串中写入占位符，后续参数会被解析至占位符处，占位符如下：
    - `%f` 浮点型、`%d` 十进制整型、`%o` 八进制整型、`%x` 十六进制整型、`%s` 字符串型、`%c` 字符型、`%b` 布尔
    - 浮点数格式控制：`%5.2f` `5`表示数据宽度为5，`.2`表示保留2为小数，两个部分均可选
  - 例：`String.format("年龄为：%d", 18);`

### StringBuffer 和 StringBuilder

> 需要修改字符串时，使用 StringBuffer 和 StringBuilder
>
> 与String类不同，StringBuffer 和 StringBuilder 的对象能够被多次的修改，并且不产生新的对象
>
> StringBuilder 的方法不是线程安全的（不能同步访问），且有速度优势，建议使用 StringBuilder 
>
> StringBuffer 和 StringBuilder 具有相同的方法

创建：`StringBuilder sb = new StringBuilder();` 内部为空，默认容量为16字符

- 参数也可为：`(int capacity)` 指定字符数为容量且内部为空、`(CharSequence seq)` 、`(String str)`

实例方法：

- `int capacity()` 返回当前容量、`int length()` 返回长度
- `indexOf`、`lastIndexOf`
- `StringBuffer append(String s)` 追加指定字符串
- `StringBuffer reverse()` 反转
- `StringBuffer delete(int start, int end)` 移除
- `StringBuffer insert(int offset, char c)` 插入字符
  - 参数也可为：`(int offset, String str)`
- `StringBuffer replace(int start, int end, String str)` 替换

- 

### Math

静态属性：

- `E` 自然底数

静态方法：

> `NS`意为任意Number派生类

- `NS abs(NS i)` 返回绝对值
- `double ceil(double d)` 向上取整
  - 相似方法：`floor` 向下取整、`rint` 向最接近的整数取整（0.5向最近的偶数取整）
- `int/long round(double d)` 四舍五入
- `NS min(NS arg1, NS arg2)` 返回最小值，`max`方法同理
- `double exp(double d)` 返回自然底数的d次方
  - 相似方法：`log` 参数的自然数底数的对数值
- `double pow(NS base, NS exponent)` 返回第一个参数的第二个参数次方
- `double sqrt(double d)` 返回参数的算术平方根
  - 相似方法：`sin`、`cos`、`tan`、`asin`、`acos`、`atan`、`toDegrees` 弧度转角度、`toRadians`角度转弧度
- `double random()` 返回随机数，取值范围为`[0, 1)`

## 异常处理

`java.lang.Error`类和`java.lang.Exception`是`java.lang.Throwable`的子类

- Error类表示严重的问题，不需要程序去捕获
- Exception类表示程序应该捕获的错误

检查异常与非检查异常：

- JVM强制程序捕获并处理检查异常，而对非检查异常不作限制，但异常发生时，程序终止
- 除`java.lang.RuntimeException`外，其他的异常均为检查异常

### 基本使用

抛出异常：

- `throws`关键字：用于方法声明处，指出该方法可能发生的异常

  ```java
  public void function() throws NumberFormatException{
  	// 方法体
  }
  ```

- `throw`关键字：用于在语句执行处，抛出一个异常 `throw new NumberFormatException(); `

捕获异常：`try...catch...`

```java
try {
    char c = (char)System.in.read();
    System.out.println(c);
} catch (IOException e) {
    System.out.println(e);
} catch (ArrayIndexOutOfBoundsException e) {
    System.out.println(e);
} finally {
    ...
}
```

自定义异常：继承Exception类或其子类

```java
class SimpleException extends Exception {}

public class SimpleExceptionDemo {
    public void f() throws SimpleException {
        System.out.println("Throw SimpleException from f()");
        throw new SimpleException();
    }
    public static void main(String[] args) {
        SimpleExceptionDemo sed = new SimpleExceptionDemo();
        try {
            sed.f();
        } catch(SimpleException e) {
		System.out.println(e);
             System.out.println("Caught it!");
        }
	 }
} 
```

### 异常类

`java.lang.Exception`的子类有：

- java.lang.ClassNotFoundException
- java.lang.RuntimeException (JVM正常运行时抛出的错误)
  - java.lang.ArithmeticException
  - 
- java.io.IOException
- java.awt.AWTException
- java.sql.SQLException

## 文件操作

### 标准IO流

控制台输出：

- `System.out.println([任何数据类型]);` 输出的字符串末尾自带换行符
- `System.out.print([任何数据类型]);`  
- `System.out.printf(“年龄=%d”,age);` 格式化输出

控制台输入：借助[Scanner](###Scanner)实例

### 字节流与字符流

> 字节流：一次读写一个字节。抽象类：`java.io.InputStream`、`java.io.OutputStream`
>
> 字符流：一次读写一个字符。抽象类：`java.io.Reader`、`java.io.Writer`
>
> 字符流 = 字节流 + 编码表
>
> 所有字节流/字符流的构造方法均可能抛出`FileNotFoundException`
>
> 所有读写方法均可能抛出`IOException`



**字节流**

InputStream类的方法：

- `public void close() ` 关闭流并释放系统资源
- `public int read()` 读取下一个字节的数据并返回，若到结尾返回-1
  - 参数也可以是：`(byte[] r)` 读取r.length长度的字节，存储数据于r中，返回读取的字节数
- `public int available()` 返回该流剩余可读取的字节数

OutputStream类的方法：

- `public void close() `
- `public int write(int w)` 将指定字节写入输出流
  - 参数也可以是：`(byte[] w)` 将w.length长度的字节写入

创建：`OutputStream os = new FileOutputStream(url/File实例);`

示例代码：

```java
import java.io.*;
 
public class fileStreamTest {
    public static void main(String[] args) {
        try {
            byte bWrite[] = { 11, 21, 3, 40, 5 };
            OutputStream os = new FileOutputStream("test.txt");
            for (int x = 0; x < bWrite.length; x++) {
                os.write(bWrite[x]); // 写入
            }
            os.close();
 
            InputStream is = new FileInputStream("test.txt");
            int size = is.available();
            for (int i = 0; i < size; i++) {
                System.out.print((char) is.read() + "  ");
            }
            is.close();
        } catch (IOException e) {
            System.out.print("Exception");
        }
    }
}
```

缓冲加速：

- `BufferedInputStream(FileInputStream实例)`  缓冲加速输入流
- `BufferedOutputStream(...)` 缓冲加速输出流

流的复制工具：

```xml
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.6</version>
</dependency>
```

```java
IOUtils.copy(fis,os); // fis:输入流, os:输出流
```



**字符流**

`FileReader/FileWriter(文件路径字符串)`  字符流读写文件



缓冲加速：`BufferedReader/BufferedWriter(FileReader/FileWriter实例)`

bufferedReader方法：

- `public Stream<String> lines()` 按行读取文本，并返回stream流



**字符流与字节流转换**

- 字节流到字符流：`InputStreamReader/OutputStreamWriter`

- 字符流到字节流：从字符流中获取char[]数组，转换为String，然后调用String的API函数getBytes() 获取到byte[]，然后就可以通过ByteArrayInputStream、ByteArrayOutputStream来实现到字节流的转换



### File类

java.io.File

创建：`File f = new File(String url);`

实例方法：

- `boolean isDirectory/isFile/isHidden()` 判断对象是否为目录/文件/被隐藏文件
- `boolean mkdir()` 根据对象url创建目录，false表示指定目录已存在或整个路径不存在
- `boolean mkdirs()` 根据对象url创建目录，包括其必要的父目录
- `String[] list()` 返回目录下的文件或目录
- `File[] listFiles()`
- `boolean delete()` 删除文件或目录

## 多线程

> 程序（program）：静态的代码
>
> 进程（process）：程序的一次执行过程，或是正在运行的一个程序
>
> 线程（thread）：进程可进一步细化为线程，是程序内部的一条执行路径
>
> - 线程作为调度和执行的单位，每个线程拥有独立的运行栈和程序计数器，线程切换的开销小
> - 同一进程中的多个线程共享内存地址空间，可以访问相同的变量和对象

创建线程类：

- 方法一（实现Runnable接口）：继承`java.lang.Thread`类，重写`run`方法
- 方法二：

开启线程：调用线程类的 `start()` 方法开启新线程，JVM会去调用线程类的 `run()` 方法

线程生命周期：

- 新建
- 就绪：调用start之后，具备运行条件但未分配cpu资源。
- 运行：线程被调度并获得cpu资源，会在该生命周期中执行run方法。
  - 就绪状态获取cpu执行权则进入运行状态，运行状态失去cpu状态或调用yield方法则回到就绪状态。线程会频繁地在这两个状态之间切换
- 阻塞：被人为挂起或执行输入输出操作时，让出cpu并临时中止自己的执行
  - 阻塞状态结束后，回到就绪状态
- 死亡

线程同步：解决线程安全问题，操作同步代码时，只能有一个线程参与。

- 方法一：同步代码块

  ```java
  synchronized(同步监视器){
  	// 需要同步的代码
  }
  ```

  同步监视器：也称锁，任何类的对象都可以作为锁。多个线程必须共用同一把锁

### Thread类

静态方法：

- `static void yield()` 线程让步，暂停该线程，将执行机会让给更高优先级的线程
- `static void sleep(long millis)` 线程在指定时间内放弃对cpu的控制
- `static Thread currentThread()` 返回当前线程

实例方法：

- `void start()` 启动线程，并执行对象的run方法
  - 一个线程实例只能`start()`一次
- `void run()` 线程在被调度时执行的操作
- `void join()` 某线程中调用其他线程的join方法时，阻塞当前线程，直至加入的线程执行完毕
- `void stop()` 强制线程结束（不建议使用）
- `String getName()` 返回线程名
- `void setName(String name)` 设置线程名
- `boolean isAlive()` 判断线程是否还活着
- `int getPriority()` 获取线程优先级
  - 线程优先级等级：`MAX_PRIORITY`：10、`MIN_PRIORITY`：1 、`NORM_PRIORITY`：5
  - 线程会继承其父线程的优先级
  - 高优先级仅表示高概率被执行，并不代表先执行高优先级再执行低优先级
- `void setPriority(int newPriority)` 设置线程优先级

### 优先级

## util包

### Date

### Calendar

#### GregorianCalendar

### Scanner

