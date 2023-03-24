# HTML

对于cdn引用的标签，添加crossorigin="anonymous"属性，在想url发送请求时将不携带当前域名的cookie



网站favicon图标：

png转换为ico格式图标：http://www.bitbug.net/ 、  在head标签中放入代码：<link rel=”shortcut icon” href=”图标路径” />

## 综述

### 页面渲染原理

渲染流程：

1. 解析HTML，生成DOM树；解析CSS，生成CSSOM树（并行）
   - 解析过程中遇到link、script、img标签时，浏览器会向服务器请求资源
     - script的加载或者执行都会阻塞html解析、其他下载线程以及渲染线程
     - css的加载和解析不会阻塞html的解析，但会阻塞渲染（步骤3）
     - img的加载不会阻塞解析与渲染。未下载完的图片需等下载完后才渲染
2. 将DOM树和CSSOM树结合，生成渲染树（Render Tree）
3. Layout：根据生成的渲染树，将所有的节点在平面上合成
4. Painting：根据渲染树以及回流得到的几何信息，得到节点的绝对像素
5. Display：将像素发送给GPU，展示在页面上（GPU会将多个合成层合并为同一个层并展示在页面中。css3硬件加速的原理是新建合成层）

回流与重绘：

- 回流（reflow）：渲染树因为元素的改变而需要重新构建，重新触发上述步骤3。回流结束后一定会触发重绘
- 重绘（repaint）：渲染树的一些元素需要更新属性，重新触发上述步骤4。这些属性只会影响外观，风格，不影响布局
- 浏览器一般有队列机制，将多次修改存储起来一次性回流，但**通过JS获取布局信息**时会强制清空队列进行回流与重绘

会触发回流与重绘的操作：

1. 添加、删除元素（回流+重绘）
2. 隐藏元素，display:none（回流+重绘），visibility:hidden（只重绘）
3. 移动元素，如改变top、left或移动元素到另一个父元素中（重绘+回流）
4. 改变浏览器大小（回流+重绘）
5. 改变浏览器的字体大小（回流+重绘）
6. 改变元素的padding、border、margin（回流+重绘）
7. 改变浏览器的字体颜色（只重绘）
8. 改变元素的背景颜色（只重绘）
   



### 文档模式

标准模式（严格模式）：按照W3C标准解析代码，若文档中有DTD声明，则进入标准模式

混杂模式（兼容模式、怪异模式）：浏览器用自己的方式解析代码，通常模拟老浏览器的行为，若文档中无DTD声明，进入兼容模式

## HTML元素

```HTML
<p class="editor-note">
    My cat is very <i>grumpy</i>
</p>
```

HTML元素的主要部分有：

1. 开始标签（Opening tag）：包含元素的名称、属性，表示元素的开始。
   - 属性（Attibute）：`属性名="属性值"`，包含元素的额外信息。可以缺省值部分，写为`属性名`
     - 枚举属性：缺省时，该属性值为默认值
     - 布尔属性：缺省或传入任何值时，该属性值为 true
   - 空元素（单标签元素）：`<input />`，在开始标签中关闭，可以省略 `/`
2. 结束标签（Closing tag）：与开始标签相似，在元素名前有一个斜杠，表示着元素的结尾
3. 内容（Content）：元素的内容，可以包含文本和元素



注释元素：`<!--注释-->`



### 基本

`<!DOCTYPE html>`：文档类型声明，用于HTML文件开头，向浏览器声明解析文档所用的规范（HTML/XHTML）



`<html></html>`：根元素，包含整个页面的内容，所有其他元素必须是该元素的后代

- 可选属性：
  - `xmlns="xml命名空间"`：默认值为`"http://www.w3.org/1999/xhtml"`



`<head></head>`：用于规定文档配置信息，用作html元素的子元素

- head元素中必须包含一个title元素，除非已从更高等级协议中指定标题（iframe）



`<title>标题</title>`：页面标题



`<body></body>`：文档内容，用作html元素的子元素

- 可选属性：onload、onunload等事件属性



`<meta />`：元数据元素，通常用于head元素内，提供与页面有关的元数据

- 可选属性：

  - `charset="UTF-8"`：指定编码类型
  - `name="键名"`、`content="键值"`：两者连用，以键值对的形式提供元数据
  - `http-equiv="指令名"`、`content="值"`：两者连用，定义编译指示指令

- 常用示例：

  - `<meta name="description" content="页面描述内容" />`
  - `<meta name="keywords" content="页面关键词" />`
  - `<meta name="robots" content="可选值" />`：搜索引擎检索方式
    - 可选值：多个值用逗号分隔
      - `all`：文件将被检索，且页面上的链接可以被查询
      - `none`：文件将被检索，且页面上的链接不可以被查询
      - `index`：文件将被检索
      - `follow`：页面上的链接可以被查询
      - `noindex`：文件不被检索
      - `nofollow`：链接不被查询
  - `<meta name="viewport" content="可选值" />`：移动端视口设置
    - 可选值：多个值用逗号分隔
      - `width=数值 | device-width`：设置视口宽度为指定数值或设备屏幕可见宽度
      - `height=数值 | device-height`
      - `initial-scale=1`：设置初始缩放比例
      - `maximum-scale=数值`、`minimum-scale=数值`
      - `user-scalable=no | yes`：是否允许用户缩放
  - `<meta http-equiv="refresh" content="5;url=http://..." />`：5秒后重定向至指定页面，若content只指定时间，则只会在指定时间后刷新页面
  
  

### 容器

`<div></div>`：块级容器，无特殊样式与功能，相似的语义标记元素：

- header：文档的标题、标志或导航内容
- nav：文档的导航部分
- aside：文档的侧边栏
- main：文档的主要内容，通常是文档的最大内容块
- footer：文档的脚注、版权信息等
- section：将文档分为不同部分，通常用于组织文章或页面结构
- article：通常用于表示一篇文章或博客
- hgroup：关联标题与次要内容，通常将一个标题元素和任意个段落元素置于其中



`<span></span>`：行内容器，无特殊样式与功能



`<details></details>`：详细信息展现元素（块级），仅在展开状态显示其内容

- `open="布尔值"`：是否展开
- 事件：toggle（切换展开状态）

`<summary></summary>`：块级，用于details元素内，作为其可见标题



`<figure></figure>`：可附标题内容元素（块级），通常将img置于该元素内

`<figcaption></figcaption>`：块级，用于figure元素的第一个或最后一个子元素，用于描述figure内的其他数据



`<dialog></dialog>`：对话框元素（块级）

- `open="布尔值"`：对话框是否激活（显示），推荐使用 `show() / showModal()` 来激活

### 文本

块级元素：

`<h1></h1>`：标题，由h1-h6逐渐变小，标题自动加粗

`<p></p>`：段落，文本会根据浏览器大小自动换行，段落之间有一定间距





行内元素：

`<strong></strong>` / `<b></b>`：加粗

`<em></em>` / `<i></i>`：倾斜

`<del></del>` / `<s></s>`：删除线

`<ins></ins>` / `<u></u>`：下划线

`<br />`：换行，相当于\n



字符实体：



### 链接

`<link />`：外部资源链接元素，可选属性：

- `href="url"`：资源地址（hypertext reference）
- `rel=“可选值1 可选值2”`：relationship，描述资源与当前文档的关系，常用可选值：
  - stylesheet（样式表）、icon（文档图标）、shortcut icon（兼容写法）、preload
- `sizes=“64x64”`：定义icons大小，单位为px，也可指定属性该为any，表示可伸缩为任意尺寸
- `as=“可选值”`：仅在rel属性设置为preload或prefetch时可用，规定加载内容的类型
  - 可选值：audio、document【用于iframe元素】、image、style、script、video



`<a></a>`：超链接（anchor），可选属性：

- `href=”目标URL”`：目标URL，也可以传入 `#元素ID` 用于导航到该元素，`#` 或 `#top` 表示返回文档顶部
- `target="值"`：指定在何处显示链接的资源。取值为window、tab、
-  _self（默认）：当前页面打开     _blank：新窗口打开
- `download`：指示下载URL而非导航到它，可给属性传入值，作为下载的文件名（文件名可带路径）
- 内部链接：直接链接内部页面名称即可，如”index.html”          跳转到当前页面：”#”
- 下载链接：URL若为文件或压缩包，则会下载文件    锚点链接：URL为”#id”  点击会跳转到页面的指定位置
- 跳转需要：在网页元素处设置id属性



`<script src=“xx”></script>`										定义客户端脚本（如JavaScript）
特性：
type=“可选值”【指定script元素包含或src引用的脚本的语言，值为MIME类型，若为js建议不写type】
text/javascript、text/ecmascript、application/ecmascript、application/javascript、module【视为js模块执行】
src=”资源URL”【从外部引用文件】				charset=”UTF-8”【指定src引用文件的字符编码】
async【HTML5，异步加载脚本，即脚本与HTML并行加载，脚本加载完后暂停HTML加载，执行脚本，然后继续加载HTML】
defer【仅对外部脚本有效，延迟加载脚本，脚本与HTML并行加载，脚本延迟到HTML加载完成后执行】
defer会在执行完脚本后触发DOMContentLoaded事件，async与DOMContentLoaded事件无关
integrity=“sha384-xxxxxx”【指定哈希值生成算法和实际哈希值，当获取资源时会进行校验，若哈希不一致，则不执行】


`<iframe></iframe>`：内联框架元素（块级），用于将另一个HTML页面嵌入当前页面

- 可选属性：
  - `src="URL"`：被嵌套页面的URL
  - `srcdoc="HTML代码"`：将代码渲染进iframe，会使src属性失效
  - `width/height="像素格式"`：元素宽高，可以是数值或百分比
  - `name="xx"`：用于定位元素的名称，可用于a/form元素的target属性
  - `allow="枚举值"`：指定特征策略，枚举值有：
    - fullscreen：允许调用 iframe 的 requestFullscreen() 激活全屏
    - payment：允许调用 payment API
- 优点：用来加载速度较慢的内容（广告）、实现脚本并行下载、实现跨子域通信
- 缺点：会阻塞主页面的onload事件、无法被一些搜索引擎识别、不好管理页面



### 表格

### 列表

### 表单

### 多媒体



`<img />`：图像元素（行内）

- 可选属性： 
  - `src="图像URL"`
  - `alt="文本"`：alt为替换文本，图像无法显示时显示的文字
  - `title=”文本”`：提示文本，鼠标悬浮在图像上时显示的文字 
  - `width/height=”像素格式”`：图像宽高
  - `decoding="枚举值"`：图像解码方式，枚举值：sync（同步解码），async（异步解码），auto（默认值，浏览器自动判断）
  - `loading="枚举值"`：图像加载方式，枚举值：eager（立即加载），lazy（当图片和视口接近到某个距离时才加载）
  - `sizes="(max-height: 1500px) 1000px, 800px"`：根据媒体查询调整图片的预期宽度，需要同时设置srcset，否则无效
  - `srcset=“URL 条件, URL2 条件2”`：达成指定条件时，改变URL，可选条件：
    - `2x`：在屏幕密度为2x时，该条件无需设置sizes，较为方便
    - `256w`：单位为像素，w仅做关键字，根据sizes指定的预期宽度匹配该条件（取最接近的）
- 服务器端图像映射：待补充

## 全局属性

> 所有HTML元素共有的属性，可用于任何元素，即使可能对元素不起作用

### 基本

`id="xx"`：定义唯一标识符（ID），该标识符在整个文档中必须唯一



`style="css语句"`：行内样式

`class="类名1 类名2"`：元素类名



`data-xxx="值"`：自定义数据属性，可通过dom获取



`is="自定义元素名"`：扩展为已注册的自定义元素



`lang="en"`：定义元素的语言，通常用于html元素，值需要符合规范，如：`zh-CN` 等

 

`part="值1 值2"`：用于 shadow DOM 的元素，允许 CSS 通过 `::part(值)`选择元素

`slot="插槽名"`：用于 shadow DOM 的元素，该元素将被分配给对应插槽



`translate="枚举值"`：指定在页面本地化时，该元素的可翻译属性值及其文本子节点是否跟随翻译，或者是保持不变。枚举值：

- 空字符串和`"yes"`：元素将被翻译
- `"no"`：元素不会被翻译



### 显示

`autofocus`：页面加载完毕或所属dialog显示时，自动聚焦元素



`hidden`：隐藏该元素，css的display属性会覆盖该属性



`dir="枚举值"`：指示元素中文本方向，枚举值：

- ltr：从左到右，用于英语等
- rtl：从右到左，用于阿拉伯语等
- auto：根据内容决定方向



`enterkeyhint="枚举值"`：定义虚拟键盘上回车的显示方式（不影响行为）

- 枚举值：enter、done、go、next、previous、search、send



`spellcheck`：检查元素内容是否存在拼写错误

- 枚举值：true、false



`title`：包含表示与其所属元素相关信息的文本



### 输入&控制

`accesskey="i"`：指定键盘快捷键，用于激活元素，属性值由空格分隔的字符列表组成

- chrome、safari等使用 `Alt + accesskey` 触发
- firefox使用 `Alt + Shift + accesskey` 触发
- 详见[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Global_attributes/accesskey)



`autocapitalize="枚举值"`：控制用户的文本输入是否和如何自动大写

- 枚举值：
  - off / none：所有字母小写
  - on / sentences：每个句子的第一个字母大写，其他字母小写
  - words：每个单词的第一个字母大写，其他字母小写
  - characters：所有字母大写
- 该属性不会影响物理键盘输入，会影响如移动设备虚拟键盘、语音输入的行为。
- 该属性不会影响 type 为url、email、password的input元素



`contenteditable="枚举值"`：元素是否可被用户编辑

- 枚举值：true、false



`inert`：指定为惰性元素，其特点有：

- 防止元素触发点击事件
- 阻止元素获得焦点
- 将元素从辅助功能树中排除



`draggable="枚举值"`：搭配拖拽API，指示是否可拖动元素

- 枚举值：true、false



`inputmode="枚举值"`：编辑元素或其内容时要使用的虚拟键盘配置类型，枚举值：

- none：无虚拟键盘，一般在需要显示自定义键盘时使用
- text：默认值，使用用户本地区域设置的标准文本输入键盘
- decimal：小数输入键盘，包含数字和分隔符（通常是“ . ”或者“ , ”）
- numeric：数字输入键盘，包含数字
- tel：电话输入键盘，包含数字、星号（*）和井号（#）键
- search：为搜索输入优化的虚拟键盘，返回键可能被重新标记为“搜索”
- email：为邮件地址输入优化的虚拟键盘，通常包含"@"符号和其他优化
- url：为网址输入优化的虚拟键盘，比如，“/”键会更加明显、历史记录访问等。



`tabindex="整数"`：指示元素是否可以通过顺序键盘导航聚焦（通常用tab键）

- 负值：元素可聚焦，但不可通过键盘导航聚焦
- 0：元素可聚焦，可通过键盘导航聚焦，相对顺序由DOM顺序决定
- 正值：元素可聚焦，可通过键盘导航聚焦，元素聚焦的顺序由该值决定，若有值相同的元素，它们按DOM顺序决定



### 元数据

```html
<dl itemscope
    itemtype="http://vocab.example.net/book"
    itemid="urn:isbn:0-330-34032-8">
    <dt>Title <dd itemprop="title">The Reality Dysfunction
    <dt>Author <dd itemprop="author">Peter F. Hamilton
    <dd><time itemprop="pubdate" datetime="1996-01-26">26 January 1996</time> 
</dl>

<div id="b" itemprop="band" itemscope itemref="c"></div>
<div id="c">
    <p>Band: <span itemprop="name">Jazz Band</span> </p>
    <p>Size: <span itemprop="size">12</span> players</p>
</div>
```

`itemscope`：将元素定义为与元数据关联的数据项



`itemtype="url"`：定义 itemprops 的词汇表，仅用于具有 itemscope 的元素

- google等搜索引擎支持 schema.org 结构化数据词汇
- 例：指定url为 `http://schema.org/Product` ，可用词汇有 brand、name 等



`itemid="url"`：数据项的唯一全局标识符，仅用于具有 itemscope、itemtype 的元素



`itemprop="词汇"`：标记元素，仅用于 itemscope 的后代元素



`itemref="id1 id2"`：将指定id的元素关联为该数据项的一部分，仅用于具有 itemscope 的元素

- 被关联元素必须：不是某个数据项的后代

