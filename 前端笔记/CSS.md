# CSS

## CSS&DOM树

浏览器渲染流程：

1. HTML解析文件，生成DOM Tree，解析CSS文件生成CSSOM Tree。（两者并行）
2. 将Dom Tree和CSSOM Tree结合，生成Render Tree(渲染树)
3. 根据Render Tree渲染绘制，将像素渲染到屏幕上。





提高css加载速度，可以使用:

1. 使用CDN
2. 对css进行压缩(webpack,gulp等，也可以通过开启gzip压缩)
3. 合理的使用缓存(设置cache-control，expires，以及E-tag都是不错的，注意文件更新后，要避免缓存而带来的影响，可在文件名字后面加一个版本号)
4. 减少http请求数，将多个css文件合并，或者是干脆直接写成内联样式(内联样式的一个缺点就是不能缓存)

## 概念

CSS：层叠样式表，实现了结构（HTML）与样式分离


元素显示模式：

- 块元素：高度、宽度、内外边距可控制；宽度默认是父级容器宽度的100%，前后带有换行符；块级容器可以容纳任何元素（文字标签除外，不能容纳块级元素）
  - `<h1-6><p><div><ul><ol><li>`
- 行内元素：相邻行内元素在一行上，一行可以显示多个；高宽无法直接设置；默认宽度为它本身内容的宽度；行内元素只能容纳文本或其他行内元素
- 行内块元素：可和相邻的行内块元素在一行上，但之间会有空白缝隙；默认宽度为它本身内容的宽度；高度，行高，内外边距可控制。
  - `img、input、td`
- 显示模式的转换：`display：block;` `display：inline;` `display：inline-block;`

块级格式化上下文（Block Formatting Contexts，BFC)：是一个环境，里面的元素不会影响外面的元素

- 布局规则：只要元素满足下面任一条件即可触发 BFC 特性
  - body 根元素
  - 浮动元素：float 除 none 以外的值
  - 绝对定位元素：position (absolute、fixed) 
  - display 为 inline-block、table、inline-table及各种table-...、flex
  - overflow 不为 visible 以外的值 (hidden、auto、scroll)

- BFC特性：多个BFC之间不会发生外边距合并、BFC可以包含浮动的元素且高度不塌陷（用于清除浮动）、BFC不与浮动元素重叠（两栏自适应布局）

## 综述

CSS引入方式：

- 内部样式：放入style标签内，style标签可以放在文档的任意地方，但一般放在head标签中，控制范围为整个页面
- 行内样式：在元素标签内部的sytle属性内设置CSS样式，只控制当前的标签样式
- 外部样式表：在html文件中使用link标签引入外部CSS文件。
  - `<link rel="stylesheet" type="text/css" href="css文件路径">`

CSS三大特性：

- 层叠性：给相同的选择器的相同样式设置不同属性会覆盖（后执行的代码覆盖先执行的代码）
- 继承性：子标签会继承父标签的部分属性（text-，font-，line-和color）
- 优先级：不同选择器给同一元素的相同样式设置不同属性，按优先级设置

CSS属性推荐的书写顺序：

- 布局定位属性：display / position / float / clear / visibility / overflow
- 自身属性：width / height / margin / padding / border / background
- 文本属性：color / font / text-decoration / text-align / vertical-align / white-space / break-word
- 其他属性（CSS3）：content / cursor / border-radius / box-shadow / text-shadow / background：linear-gradient。。。

CSS初始化：

> 所有标签内外边距清除、em与i的文字不倾斜、去除li的小圆点、设置图片边框为0（照顾兼容性）及vertical-align：middle（消除图片底侧空白缝隙）、设置按钮cursor：pointer、设置链接颜色并取消下划线、设置body文字样式及抗锯齿（-webkit-font-smoothing：antialiased；）、清除浮动。

## 选择器

> 选择器的优先级：
>
> - 单选择器的优先级：!important > 行内样式 > ID选择器 > 类选择器、伪类选择器、属性选择器 > 元素选择器、伪元素选择器 > 继承或*
> - 复合选择器的优先级：分别统计该复合选择器中的行内样式、ID选择器、类选择器、元素选择器的个数。按优先级顺序依次比较数量，数量多的优先级高。
> - 提升优先级： `#foo` → `#foo:not(#bar)`


选择器格式：`选择器 {属性: 值;...}`

### 基础选择器

- 标签选择器：`标签名` 选择所有该标签
- 类选择器：`.类名` 选择所有class属性为类名的标签
- id选择器：`#id名` 选择**第一个**id属性为id名的标签
- 通配符选择器：`*` 选择页面中所有元素

### 复合选择器

- 后代选择器：`元素1 元素2...`   选择所有 空格后选中元素 中作为 空格前选中元素 **后代**的元素
- 子选择器：`元素1>元素2`  选择元素1中作为直接子代的所有元素2
- 并集选择器：`元素1, 元素2, ...` 选择元素1和元素2
- 交集选择器：`h3.class1`  选择类名为class1的h3标签
- 相邻兄弟选择器：`h1+h2`  若h1和h2为兄弟，选择所有h1后面的第一个h2标签
- 后续兄弟选择器：`h1~h2`  若h1和h2为兄弟，选择所有h1后面的所有h2标签

### 属性选择器

- `[属性名]` 具有某属性
- `[属性名=值]` 有属性且属性值等于指定值
- `[属性名^=值]` 有属性且属性值开头为指定值
- `[属性名$=值]` 有属性且属性值结尾为指定值
- `[属性名*=值]` 有属性且属性值包含指定值
- `[属性名=值 i]` 右侧加字母`i或I`，在匹配值时忽略大小写
- `[属性名=值 s]` 右侧加字母`s或S`，在匹配值时区分大小写

### 伪类选择器

链接伪类选择器：
- `a:link`  选择所有未被访问的链接
- `a:visited`  选择所有已被访问的链接
- `a:hover`  选择鼠标指针位于其上的链接
- `a:active`  选择活动链接（鼠标按下未弹起的链接）
- 为了确保生效，按照上述自上而下的顺序声明

focus伪类选择器：选取焦点，一般只有input才能获取

- `input:focus`  选择获得输入光标的表单元素

结构伪类选择器：
- `:first-child` 元素是它父元素的第一个元素
- `:last-child` 元素是它父元素的最后一个元素
- `:nth-child(n)` 元素是它父元素的第n个元素
  - n可以是数字、关键字（`even`偶数、`odd`奇数）和公式（公式中的n从0开始递增1直到结束，如n+5为第五个到最后一个）
- `标签名:first-of-type` 元素是它父元素的第一个指定类型的元素
- `标签名:last-of-type` 元素是它父元素的最后一个指定类型的元素
- `标签名:nth-of-type(n)` 元素是它父元素的第n个指定类型的元素

- 反选伪类选择器：`:not(选择器)` 将不被选择器选中的元素作为结果

### 伪元素选择器

利用CSS创建新标签元素

- `::before` 在父元素内部的最前面插入元素
  ```css
  div::before{
      content: '';
  }
  ```

- `::after` 在父元素内部的最后面插入元素

- 创建的元素为行内元素、此元素无法在文档树中找到、必须有content属性

### 媒体查询选择器

`@media 查询条件{}`

css选择器写在媒体查询选择器`{}`内部

查询条件：条件之间用`and`（条件与）或  `,`（条件或）分隔

- 媒体类型：
  - `all` 所有设备
  - `print` 打印设备
  - `screen` 带屏幕的设备
  - `speech` 屏幕阅读器
  - 可在媒体类型前加`only`关键字，表示只有这个媒体类型应用
- 媒体特性：键值对形式，需写于`()`内
  - 视窗宽高属性：`width`、`height`、`min-width`、`max-width`
  - `max`不包括本数，`min`包括本数
  - 例：`@media (width: 500px){}`
- `not`关键字用于所有查询条件的开头，对整个规则取反，需要指定媒体类型

屏幕大小常用断点：

- 0-768：超小屏幕     `(max-width: 768px)`
- 768-992：小屏幕     `(min-width: 768px)`
- 992-1200：中型屏幕  `(min-width: 992px)`
- 1200-∞：大屏幕     `(min-width: 1200px)`

## 基本属性

继承属性：`某属性: inherit` 规定应该从父元素继承该属性的值

### 文本

- 字体系列：`font-family: “Microsoft yahei”, Arial, “宋体”;` 
  - 字体中自带空格的必须加引号，优先使用前面的字体，若无法显示则下一个
- 字体大小：`font-size: 20px;`
- 字体粗细：`font-weight：normal | bold | bolder | lighter | 100、200...900);`
  - 400等同于normal，700等同于bold
- 字体样式：`font-style：normal | italic(斜体)| oblique(倾斜)`
- 字体复合属性：`font：font-style font-weight font-size/-height font-family` 
  - `font-size`和`font-family`属性为必须项
- 文本域大小的拖拽：`resize:none;` 



### 段落

- 对齐：`text-align: center | left | right | justify(两段对齐); `
  - 设置子元素的行内元素或行内块元素的水平对齐方式
- 首行缩进：`text-indent: 2em; ` 缩进2字符
- 修饰：`text-decoration: none | underline | line-through(删除线);` 
- 文字行高：`line-height：40px | 1.5;`  若不带单位，表示为文字大小的1.5倍
- 换行：
  - `word-break: normal | break-all | keep-all`
    - `break-all` 对于非中日韩文的文本，可在任意字符间断行。
    - `keep-all` 中日韩文不断行，其他文本同normal
  - `word-wrap: normal | break-word` 
    - `break-word` 内容将在边界内换行。如果需要，单词内部允许断行
    - css3中更名为`overflow-wrap`，建议同时写`word-wrap`保证兼容



### 列表

- 列表元素标记：`list-style-type: none | disc | circle ...` 
  - `disc`：实心圆，默认；`circle`：空心圆；`square`：实心方块；
  - `decimal`：从1开始的十进制阿拉伯数字；`\1F44D`：任意Unicode字符；
- 图片作为列表元素标记：`list-style-image: url()` 
- 标记框位置：`list-style-position: outside(默认，在主块盒外) | inside`
- 复合属性：`list-style: <...-type> <...-image> <...-position>`



### 表格

- 表的列宽：`table-layout: auto | fixed`

  - auto：表的列宽会根据包含的内容大小而变化
  - fixed：表的列宽与列首单元格相同

- 表的边框是否合并：`border-collapse: collapse | separate`

  



### 背景

- 背景颜色：`background-color: 颜色;` 设置背景色为透明，默认为透明
- 背景图片：`background-image: none | url(...);`  背景图片显示在背景颜色之上
- 背景平铺：`background-repeat: repeat | no-repeat | repeat-x | repeat-y;`
- 背景图片位置：`background-position：x y;`           
  - xy可用像素表示；也可用百分比表示，50%表示居中；
  - xy也可用方位名词（`left`、`top`等）替代。若x和y都被替代，则两个值前后顺序无关。若只指定一个方位名词，第二个值默认居中。
- 背景图像固定：`background-attachment: scroll | fixed`  设置图片是否随其他元素一起运动，可用于实现视差滚动
- 背景属性复合写法：`background：背景颜色 背景图片地址 背景平铺 背景图像滚动 背景图片位置;` 可以省略任意项



生成图像的函数：可以直接用在background-image中

- `radial-gradient([<形状>], <多个色值描述>)`：生成渐变图形
  - 形状：默认值为 `ellipse at center ` ，表示渐变形状为椭圆，渐变中心为图形中心。可以指定渐变形状为 `circle` 等
  - 色值描述：`<color> <position>`，例：`#ccc 0%`，表示渐变起始点的颜色为#ccc



### 空格处理

`white-space: normal(默认) | nowrap | pre | pre-wrap | pre-line | inherit`

- normal：按浏览器默认方式处理空格
- nowrap：不因超出容器宽度而换行
- pre：按pre标签的方式处理（保留空格与换行符）
- pre-wrap：按pre标签的方式处理，超出容器宽度时，换行
- pre-line：保留换行符，其他符号同normal



### 对齐

`text-align: 可选值`：用于块级容器，控制其内部所有行内内容的水平对齐

- 可选值：left、right、center、justify（两侧对齐，对最后一行无效）



行内、行内块元素的垂直对齐：`vertical-align: 属性值;`

- `baseline` 默认，元素位于父元素的基线（英文字母上往下的第三行）上  
- `top` 元素顶部与行中最高元素的顶部对齐
- `middle` 元素对齐于父元素的中部  
- `bottom` 元素底部与行内最低元素的底部对齐

使块级盒子水平居中：在给盒子指定了宽度的情况下，设置左右外边距为auto

定位元素居中：`left: 50%; transform: translateX(-50%);`

块级元素水平垂直居中：父元素设置为flex容器，设置该元素 `margin: auto;` 



### 元素可见性

`display：none | block；`   不占位置，触发回流和重绘，有株连性（子孙节点全隐藏）

`visibility：hidden | visible；`    占原先所在位置，触发重绘，伪株连性（可以给子孙节点设置visibility：visible可见）

`pointer-events: none`        设置元素不可用（阻止hover、active触发事件，阻止js点击动作触发事件等）



### 多媒体

图片比例：`object-fit: 属性值;`

- `contain`：保持图片比例并确保容器内能放下整张图片
- `fill`：默认值，拉伸图片填充容器
- `cover`：保持图片比例，确保图片宽高至少有一个和容器一致
- `none`：保持图片比例和尺寸
- `scale-down`：等效于 `none` 和 `contain` 中尺寸较小的那个



### 界面样式

`cursor: 可选值;` 设置在对象上移动鼠标指针的光标形状

- `default` 默认  
- `pointer` 小手（链接）  
- `move` 十字箭头  
- `text` 工字  
- `not-allowed` 禁止

`outline:none;`  去除表单选中时的轮廓线

盒子阴影：`box-shadow: h-shadow v-shadow blur spread color inset;`

- `h-shadow`、`v-shadow` 必填，值为像素值，表示水平和垂直阴影的位置，允许负值
- `blur` 选填，值为像素值，模糊距离
- `spread` 选填，值为像素值，阴影尺寸
- `inset` 选填，值为`inset` 若写，则表示内阴影

文字阴影：`text-shadow: h-shadow v-shadow blur color` 属性参考盒子阴影



## 盒子模型

盒子模型`box-sizing`：

`box-sizing: content-box` 默认，盒子大小为`width`+`padding`+`border`

`box-sizing: border-box` 盒子大小为`width`



### 边框

`border: border-width border-style border-color;`  属性没有顺序要求

- `border-width`：边框宽度，像素值
- `border-style`：`none`、`solid` 实线、`dashed` 虚线、`dotted`点线

`border-top: ...`  单独设置上边框

设置相邻边框合并在一起：

- `border-collapse: collapse;`（仅表格有效） 或  使得右元素的边框压住左元素的边框

`border-radius: 像素值 | 百分比;`  圆角边框

- 百分比为盒子宽高乘以指定百分比所得的值，可用于生成椭圆
- 若指定两个值，分别为左上右下、右上左下的圆角大小
- 若指定三个值，分别为左上、右上左下、右下的圆角大小
- 若指定四个值，分别为左上、右上、右下、左下的圆角大小
- 单独指定某个圆角的大小：`border-top-left-radius: 像素值;` 



### 内容

`width: 像素值;`、`height: 像素值;` 设置内容区的宽高

- 行内或行内块元素可以通过设置`padding`并且不设置`width`的方法来通过内容撑大盒子

`overflow: 可选值` 设置内容溢出时的处理。`overflow-x`、`overflow-y`同理

- `visible` 默认，超出部分可见
- `hidden` 超出部分隐藏
- `scroll` 不管是否超出，添加滚动条
- `auto` 超出则添加滚动条



### 边距

`padding: 多个像素值;` 设置盒子的内边距

- 若设置一个值，表示各边内边距；
- 若设置两个值，分别表示`上下`与`左右`的内边距
- 若设置三个值，分别表示`上`、`左右`、`下`的内边距
- 若设置四个值，分别表示`上`、`右`、`下`、`左`的内边距
- `padding-right: ...`  单独设置各内边距

`margin: 多个像素值;` 设置盒子的外边距，格式同内边距

外边距合并：同一个BFC中，相邻块级元素外边距会自动合并，取大值。

外边距塌陷：子元素指定垂直外边距时，子元素不生效，父元素的该外边距变为父子元素中该外边距的较大值。若子盒子为浮动，则无此问题。

- 解决方案：
  - 方法一：为父元素添加边框
  - 方法二：父元素添加内边距
  - 方法三：为父元素添加`overflow:hidden;`



## 浮动与定位

### 浮动

浮动元素会尽量向左或向右移动，直到左边缘或右边缘触及包含块或另一个浮动框的边缘

`float: none(默认) | left | right);`

浮动特性：

- 脱离标准流（不保留原先位置，漂浮在后面的标准流之上）
- 在一行内显示并且元素顶部对齐（父级宽度不够则另起一行）
- 具有行内块元素的特性
- 浮动的元素会压住下面标准流的盒子，但不会压住盒子里的文字或图片

#### 清除浮动

`clear: none(默认) | left | right | both;` 元素的指定方向不出现浮动元素

浮动的元素脱离标准流，不占位置，若父元素不设置高度，则无法匹配浮动元素的高度。

清除浮动的方法：

- 额外标签法（隔墙法）：在最后的浮动元素后面添加块元素空标签，此标签设置clear属性

- 父级添加overflow（应用BFC特性）：给父级添加`overflow`属性，属性值设置为`hidden`、`auto`或`scroll`

- after伪元素法，给父元素添加：

  ```css
  .clearfix::after{
      content: "";
      display: block;
      height: 0;
      clear: both;
      visibility: hidden;
  }
  .clearfix { 
      *zoom: 1; // IE6、7兼容用
  }
  ```

- 双伪元素清除浮动，给父元素添加：

  ```css
  .clearfix::before, .clearfix::after {
      content: “”;
      display: table;
  }
  .clearfix::after {
      clear: both;
  }
  .clearfix { 
      *zoom: 1;
  }
  ```

### 定位

定位方式：`position: static(默认) | relative | fixed | absolute | sticky` 

- `static`：无定位（标准流）。

- `relative`：相对定位，定位基点为元素的默认位置。

- `absolute`：绝对定位
  - 定位基点为最近的带定位的上级元素，若都没有定位，则为根元素html
  - 脱离标准流
- `fixed`：固定定位，定位基点为浏览器窗口。不随页面滚动而变化，脱离标准流。
- `sticky`：粘性定位，一些时候为`relative`定位，另一些时候为`fixed`定位。
  - 元素与浏览器各边的距离不会小于设置的边偏移（转为`fixed`），但元素不会超出定位基点的内容区（转回`relative`）

边偏移：有`top`、`bottom`、`left`和`right`属性。表示元素与定位基点各边线的距离

定位叠放顺序：`z-index: int;`  数值越大越靠上，压住数值较小的元素，默认值为auto（小于1）。属性值相同，则后来居上。

扩展：

- 绝对定位居中对齐：`left: 50%; margin-left: -自身宽度一半 px;`
- 行内元素添加了定位后，可以直接设置宽高

## Grid布局

### 容器属性

指定网格布局：`display: grid | inline-grid;`

指定列宽/行高：`grid-template-columns / grid-template-rows: 100px 30% repeat(3, 10%);` 列/行数为属性数

- 列宽及行高的属性内，可用方括号设置**网格线**名字，用于引用。如`[c1] 100px [c2] 100px [c3]`
- repeat 方法：`grid-template-columns: repeat(2, 10px, 15px, 20px);`
- auto-fill 关键字：`repeat(auto-fill, 100px)` 填充至无法容纳更多行/列
- `grid-auto-columns / grid-auto-rows: 50px; ` 单元格指定在划分好的网格外时，自动生成的额外网格列宽 / 行高

fr 关键字：作为单位`grid-template-columns: 150px 1fr 2fr;` 指定第三列宽度为第二列宽度的两倍

minmax 方法：`minmax(100px, 1fr);` 指定最小最大值

指定行/列间距：`grid-row-gap / grid-column-gap: 20px;`

- 合并简写：`grid-gap: <grid-row-gap> <grid-column-gap>;` 第二个值默认等于第一个值
- 最新标准中，上述三个属性的`grid-`前缀已被移除

指定区域：

```css
/* 空区域用点表示，区域必须是矩形
区域的起始 / 终止网格线自动命名为 区域名-start / 区域名-end */
grid-template-areas: 
'a . b'
'c c c'
'd d e';
```

填充顺序：`grid-auto-flow: row | column | row dense | column dense;` 

- 先行后列（默认） / 先列后行 / 某些项目指定位置后，剩下项目的填充顺序（当前面的行/列有空时，是否插入）

单元格内容对齐：`align-items / justify-items: start | end | center | stretch;`

- 垂直 / 水平对齐。stretch：拉伸，占满单元格（默认值）
- 合并简写：`place-items: <align-items> <justify-items>;` 第二个值默认等于第一个值

内容对齐：`align-content / justify-content: start（默认） | end | center | stretch | space-around | space-between | space-evenly;`

- 内容相对于容器的对齐方式
- 合并简写：`place-content: <align-content> <justify-content>;`  第二个值默认等于第一个值

### 项目属性

项目位置：`grid-column-start / grid-column-end / grid-row-start / grid-row-end: 网格线`

- 未命名网格线用索引获取，从左到右/从上到下 为1开始的整数
- 属性还可以用`span`关键字，`span 整数n`表示左右边框（上下边框）跨越n个网格
- 合并简写：
  - `grid-column: <grid-column-start> / <grid-column-end>;`
  - `grid-row: <grid-row-start> / <grid-row-end>;`

项目放置区域：`grid-area: 区域名;`

- 可用作简写形式：`grid-area: <grid-row-start> / <grid-column-start> /  <grid-row-end> / <grid-column-end>;`

单元格内容对齐：`align-self / justify-self: start | end | center | stretch(默认);`

- 合并简写：`place-self: <align-self> <justify-self>;`  第二个值默认等于第一个值

## flex布局

用来代替浮动完成页面布局，使元素具有弹性，使其跟随页面的大小改变大小

### 弹性容器

`display: flex / inline-flex` 将元素设置为块级/行内弹性容器

`flex-direction: 可选值` 指定容器中弹性元素的排列方式（改变主轴方向）

- `row`：默认值。弹性元素在容器内水平从左向右排列
- `row-reverse`：水平从右向左
- `column`：纵向自上向下
- `column-reverse`：纵向自下向上

`flex-wrap: 可选值` 设置弹性元素在弹性容器中是否自动换行（当容器大小不足时）

- `nowrap`：不换行（默认值）
- `wrap`：沿辅轴方向换行
-  `wrap-reverse`：沿辅轴反方向换行

`flex-flow: flex-direction flex-wrap` 复合设置两个属性

`justify-content: 可选值` 设置如何分配主轴上的空白空间

- `flex-start`：元素沿主轴起边排列。新标准中改名为`start`。
- `flex-end`：沿主轴终边排列。新标准中改名为`end`。
- `center`：元素居中排列
- `space-around`：空白分配到元素两侧
- `space-evenly`：空白平均分配，元素与元素，元素与容器边缘之间的空白均相等
- `space-between`：空白分配到元素间

`align-items: 可选值` 设置元素在辅轴上如何对齐

- `stretch`：将同一行元素的高度设置为相同的值
- `flex-start`：元素不会拉伸，沿辅轴起边对齐，新标准中改名为`start`。
- `flex-end`：元素不会拉伸，沿辅轴终边对齐，新标准中改名为`end`。
- `center`
- `baseline`：基线对齐

`align-content: 可选值` 设置如何分配辅轴上的空白空间（辅轴将一行视为一个元素）

- 可选值同`justify-content` 

### 弹性元素

是弹性容器的子元素，弹性元素也可以是弹性容器

`flex-grow: 数值` 指定弹性元素伸展的系数

- 默认值为0，将父元素剩余的空间按子元素系数大小分配后给子元素

`flex-shrink: 数值` 指定弹性元素收缩的系数

- 默认值为1。指定为0时，父元素装不下子元素时，该元素不收缩

`flex-basis: 像素值` 指定元素在主轴上的基础长度

- 默认值为auto，若指定了具体值，会覆盖width或height属性

`flex: flex-grow flex-shrink flex-basis` 复合指定样式

- 也可指定为`initial(0 1 auto)`、`auto(1 1 auto)`、`none(0 0 auto)`

`order: 整数` 指定弹性元素的排列顺序，越小越靠前，默认值为0

`align-self: 可选值` 设置单个元素的`align-items`，可选值同`align-items`

## CSS3

### 变量

执行计算：`calc(表达式);` 

- 表达式中可用`+-*/`，运算符前后必须加空格。例：`width: calc(100% - 80px);` 



声明全局变量：

```css
:root{ 
  --bg-color: #cccccc;
}
```

行内声明变量：

```html
<div style="--bg-color:#cccccc"></div>
```

使用变量：

```css
color: var(--font-color);
```



### 滤镜

`filter: 滤镜函数; ` 为该元素添加图形效果

- 多个过滤函数之间用空格分割

`backdrop-filter: 滤镜函数;` 为该元素后面区域添加图形效果



滤镜函数：

- `filter(<长度>)`：高斯模糊，长度越大越模糊
- `brightness(<百分比>)`：亮度调整，值越大越亮，100%为原亮度
- `contrast(<百分比>)`：对比度调整，值越大对比度越大，100%为原始对比度
- `drop-shadow(<x偏移>,<y偏移>,[<模糊半径>],[<color>])`：生成阴影，属性参考text-shadow
- `grayscale(<百分比>)`：灰度调整，0%无变化，100%转为灰度图像
- `hue-rotate(<角度>)`：色相旋转
- `invert(<百分比>)`：图像反转，0%无变化，100%完全反转
- `opacity(<百分比>)`：不透明度调整，0%全透明，100%无变化
- `saturate(<百分比>)`：饱和度调整，0%完全不饱和，100%无变化
- `sepia(<百分比>)`：转为深褐色，0%无变化，100%完全深褐色

### 过渡与动画

#### 过渡

为元素设置`transition`属性，指定元素变化时，会根据指定的规律变化

`transition: 要过渡的属性 花费时间 运动曲线 触发延迟时间;`

- 要过渡的属性：若要让所有属性同时变化，写`all`关键字
- 花费时间：必选项，单位s
- 运动曲线：`linear` 匀速、`ease`（默认）渐慢、`ease-in` 加速、`ease-out` 减速、`ease-in-out` 先加速后减速
- 触发延迟时间：单位s，默认值为`0s`
- 同时修改多个属性：在transition冒号后面写多个表达式，用`,`隔开

#### 2D转换

转换：处于标准流而不影响其他元素的位置

`transform: 变换函数;`

- `translate(x值, y值)`、`translateX(值);`、`translateY(值);` 平移变换
  - translate所需值可以是像素值，也可以是百分比（参照自身宽高计算）
  - 平移对行内元素无效
- `rotate(值);` 旋转变换
  - 值可以是`0`、`3.14rad`弧度值、`45deg`角度值、`0.25turn`圈数，顺时针为正
- `scale(x值, y值);` 缩放变换
  - 值为放大倍数，没有单位；可以只写一个参数表示两个参数相等
- 若要同时添加多种效果，函数之间加空格隔开，需要按平移、旋转、缩放的顺序写，否则会影响坐标轴方向

`transform-origin: x y;`  设置旋转/缩放中心

- 值可为像素值、百分比或方位名词，默认为50% 50%

#### 动画

设置关键帧控制动画效果，页面加载后自动播放

定义动画：

```css
@keyframes 动画名称 {
    0% { width: 100px; } 
    100% { width: 200px; }
}
```

- 关键帧可用整数百分比表示，也可用关键字`from`（即0%）`to`（即100%）

使用动画：

为元素设置属性：

- `animation-name: 动画名称;`
- `animation-duration: 持续时间;` 单位为s

可选属性：

- `animation-timing-function: 运动曲线 | steps函数;`
  - 运动曲线：`linear` 匀速、`ease`（默认）渐慢、`ease-in` 加速、`ease-out` 减速、`ease-in-out` 先加速后减速
  - steps函数：`steps(总帧数[, 跳跃选项])` 动画按指定帧数播放
- `animation-delay: 动画何时开始;` 
  - 默认为0s
- `animation-iteration-count: 动画被播放的次数;` 
  - 默认`1`，可规定`infinite`无限
- `animation-direction: 是否在下一周期逆向播放;` 
  - 默认为`normal`，可选 `alternate`逆播放
- `animation-play-state: 运行状态;` 
  - 默认为`running`，可规定`paused`
- `animation-fill-mode: 结束后状态;` 
  - `forwards`保持、`backwards`回到初始
- `animation: 动画名称 持续时间 运动曲线 何时开始 播放次数 是否反方向 动画状态;`
  - 所有动画属性的简写方式
  - 前两项必写，持续时间在何时开始之前，其他无顺序关系
  - 可以添加多个动画，用逗号隔开

#### 3D转换

z轴垂直屏幕向外为正

`transform: 变换函数;`

- `translateZ(像素值);`  z方向平移，一般不用百分比
- `translate3d(x, y, z);`  3D复合平移，不能省略xyz任意一项
- `rotateZ(值);` 旋转，旋转方向按左手准则
- `rotate3d(x, y, z, 旋转值);`  xyz表示旋转轴的矢量，旋转方向按左手准则
  - 例：`rotate(1, 1, 0, 60deg)`

`perspective: 像素值;`  透视（视距），眼睛到屏幕的距离

- 该属性写在被观察元素的父元素上

3D呈现：控制子元素是否开启三维立体环境

- `transform-style: flat | preserve-3d;`   
  - 设为flat会把子元素投影到父元素所在平面，不遮挡元素

## Less

一门css的预处理语言，以更少的代码实现更强大的样式，less会被编译为css。

单行注释：`//`

选择器：

```less
选择器1 {
	选择器2 {...}
    &选择器3 {...}
} // less写法

选择器1 选择器2{...} // 编译为css
选择器1选择器3{...}
```

- &表示外层选择器的引用

运算符：

- `+ - *` 可以直接使用   `/` 需要在括号内使用，或转义为 `./`
- `~` 防止less编译后续值。例：`@a = ~"xxx"`
- `when` 当when后面括号中的条件需满足。例：`.col(@b)when (@a > 1){}`

引入：`@import "xxx.less"`

变量：

- 声明变量：`@a: 100px;` 
  - 变量值可以是像素值、颜色、类名。
- 使用变量：`width: @a;`
  - 为选择器的类名或值的一部分使用时：`.@{a}{...}`、`url("@{a}/1.png");`
  - 变量有作用域，内部更改变量不影响外部
- 引用其他属性的值：`width: 100px; height: $width;`
- 变量有延迟加载特性，即先加载完声明变量再赋值到样式，可以看做把声明部分提前到作用域最前

扩展：

- 方法一：`选择器1: extend(选择器2){...}` 将选择器2中设置的属性复制到选择器1中
  - 若在选择器2后面加上关键词all，则会把选择器2所选元素的伪类同样添加给选择器1
- 方法二：`选择器1{选择器2();...}` 同上，使用混合函数，性能较方法一差

混合函数：

- 创建一个`mixins`混合函数：函数名以`.`开头，括号内可接收形参，仅能被混合引用，不会作为样式生效

  ```css
  .test(@w, @h) when(...) {}
  ```

- 使用混合函数：将混合函数内的选择器解包在调用函数的位置

  ```less
  // 传参时必须严格按参数数量传递
  .test(200px, 300px); // 按顺序传参
  .test(@h: 300px, @w: 200px); // 按名字传参
  ```

函数：

- `average(颜色1, 颜色2)` 返回两个颜色的中间值
- `darken(颜色, 百分数)` 返回加深的颜色
- `percentage(数字)` 将数字换为百分比数

循环：用递归实现

```less
.generate(@num) {
    .sub(@count) when(@count>0) {
        .test-@{count} {
            background-color: red;
        }
        .sub(@count - 1);
    }
    .sub(@num);
}
.generate(24);
```

媒体查询：

```less
@media screen {
    @media (min-width: 1023px) {
        .selector {
            color: blue;
        }
    }
}
```

## CSS Modules

用于提供局部作用域和模块依赖



### 局部作用域

定义css module文件

```css
.title {
	color: red;
}
:global(.title) {
	color: green;
}
```

在JSX中使用CSS Modules

```jsx
import style from './App.module.css';

export default () => {
    return (
        <h1 className={style.title}>
            Hello World
        </h1>
    );
};
```

> `.title`会被编译为哈希字符串，以此实现局部作用域。
>
> `:local(.title)`：等同于`.title`
>
> 使用`:global`伪类，可以将括号内的选择器声明为全局有效（不编译为哈希字符串）



### 模块依赖

class组合：

```css
.blue {
  background-color: blue;
}

.title {
  composes: blue;
  color: red;
}
```

> title会继承blue选择器的规则



跨文件组合：

```css
composes: blue from './another.css';
```



## 常见问题

### 消除图片底部空白缝隙

`vertical-align: top / middle / bottom` 或将图片转换为块级元素

### 溢出文字省略号显示

单行文本：

```css
white-space: nowrap; // 禁止换行
overflow: hidden; 
text-overflow:ellipsis; // 省略号替代超出的部分
```

多行文本（适用于webKit浏览器或移动端，注意兼容性问题）：

```css
overflow:hidden;
text-overflow:ellipsis;
display:-webkit-box;
-webkit-line-clamp:2; // 行数
-webkit-box-orient:vertical;
```

### 隐藏鼠标关注焦点时的outline

focus-visible选择器，表示键盘伪类焦点选择器，在规范中定义为:元素聚焦，同时浏览器认为聚焦轮廓应该显示。

有时候我们希望键盘触发聚焦轮廓，而鼠标关注焦点则不触发轮廓，此时用以下一行CSS代码搞定。

```css
:focus:not(:focus-visible) {
    outline: 0;
}
```

### flex弹性元素被内容撑大

```css
弹性元素 {
    flex-grow: 1;
    height: 1px; // 固定大小即可
}
```

这样盒子就不会被撑开，但是内容会溢出，给弹性元素再添加overflow: hidden;

还有一点，上层的每一个盒子都需要有高度，不然还是会被撑开