# HTML

对于cdn引用的标签，添加crossorigin="anonymous"属性，在想url发送请求时将不携带当前域名的cookie 



网站favicon图标：

png转换为ico格式图标：http://www.bitbug.net/ 、  在head标签中放入代码：<link rel="shortcut icon" href="图标路径" />

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

`<span></span>`：行内容器，无特殊样式与功能



`<div></div>`：块级容器，无特殊样式与功能，相似的语义标记元素：

- header：文档的标题、标志或导航内容
- nav：文档的导航部分
- aside：文档的侧边栏
- main：文档的主要内容，通常是文档的最大内容块
- footer：文档的脚注、版权信息等
- section：将文档分为不同部分，通常用于组织文章或页面结构
- article：通常用于表示一篇文章或博客
- hgroup：关联标题与次要内容，通常将一个标题元素和任意个段落元素置于其中



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

`<br />`：换行，相当于 `\n`



### 链接

`<link />`：外部资源链接元素，可选属性：

- `href="url"`：资源地址（hypertext reference）
- `rel="可选值1 可选值2"`：relationship，描述资源与当前文档的关系，常用可选值：
  - stylesheet（样式表）、icon（文档图标）、shortcut icon（兼容写法）、preload
- `sizes="64x64"`：定义icons大小，单位为px，也可指定属性该为any，表示可伸缩为任意尺寸
- `as="可选值"`：仅在rel属性设置为preload或prefetch时可用，规定加载内容的类型
  - 可选值：audio、document【用于iframe元素】、image、style、script、video
- `crossorigin="枚举值"`：定义元素如何处理跨域请求
  - `anonymous` | `""`：请求使用 CORS 标头，跨域请求不交换用户凭据
  - `use-credentials`：请求使用了 CORS 标头，总是交换用户凭据



`<a></a>`：超链接（anchor），可选属性：

- `href="目标URL"`：目标URL，也可以传入 `#元素ID` 用于导航到该元素，`#` 或 `#top` 表示返回文档顶部
- `target="值"`：指定在何处显示链接的资源。可选值：
  - window、tab、iframe等浏览器上下文的name属性值：在指定上下文中加载
  - `_self`：默认值，当前页面加载
  - `_blank`：新窗口打开
  - `_parent`：当前浏览上下文的父上下文中加载，如iframe中的a元素指定该属性，在iframe所在文档中加载资源
  - `_top`：在顶级浏览上下文中加载
- `download`：指示下载URL而非导航到它，可给属性传入值，作为下载的文件名，该属性仅在同源时生效
- `crossorigin`：[详见link的同名属性](#链接)



`<script></script>`：嵌入或引用可执行脚本（如JS），可选属性：

- `src="资源URI"`：引用外部脚本
- `type="可选值"`：指定script元素包含或src引用的脚本的语言，值为MIME类型
  - 可选值：text/javascript（默认值）、text/ecmascript、application/ecmascript、application/javascript、module（视为js模块执行）
- `text="xx"`：元素的文本内容，元素插入到dom后，该属性会被解析为代码
- `async`：异步加载脚本，加载完后立刻执行（执行时可能阻塞HTML加载）
- `defer`：异步加载脚本，加载完后，在DOMContentLoaded事件前执行脚本（仅对外部脚本有效，即带有src属性的脚本）
- `crossorigin`：[详见link的同名属性](#链接)



`<noscript></noscript>`：页面中无法使用脚本时，显示该元素。元素可包含：

- 若该元素在head元素中，仅能包含link、style、meta元素
- 其他情况下，能包含任意元素，但后代中不能有 noscript 元素



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

```html
<table>
    <thead>
        <tr>
            <th colspan="2">The table header</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>The table body</td>
            <td>with two columns</td>
        </tr>
    </tbody>
</table>
```

`<table></table>`：表格元素，允许包含的内容（按顺序）：

- 一个 caption 元素
- 多个 colgroup 元素
- 一个 thead 元素
- 多个 tbody 或 tr 元素（两者不能同时作为 table 的子元素出现）
- 一个 tfoot 元素



`<caption></caption>`：标题容器



`<colgroup></colgroup>`：定义表格的列组，内部只能包含 col 元素

- 给 该元素 或 col元素 设置样式，可应用到对应列上
- 可选属性：
  - `span="正整数"`：该元素所跨越的列数，若设置该属性，该元素必须为空元素
- `<col></col>`：定义列组，可选属性为 span，作用同上



`<thead></thead>`：表头，可包含任意个tr元素

`<tbody></tbody>`：表格主体，可包含任意个tr元素

`<tfoot></tfoot>`：汇总行，可包含任意个tr元素



`<tr></tr>`：表格行，可混合包含任意个 td 或 th 元素

`<th></th>`：表头单元格容器，可选属性：

- `colspan/rowspan="正整数"`：单元格所占的 列数/行数
- `headers="空格分隔id字符串"`：与其他th元素关联，字符串是对应元素的id
- `scope="枚举值"`：定义该元素关联的单元格，枚举值有：
  - auto
  - row：表头关联一行中所有单元格
  - col：表头关联一列中所有单元格
  - rowgroup：表头属于一个行组并与其中所有单元格相关联
  - colgroup：表头属于一个列组并与其中所有单元格相关联

`<td></td>`：数据单元格容器，可选属性同 th（scope属性除外）



### 列表

`<ul></ul>`：无序列表，只能包含任意个li元素

`<ol></ol>`：有序列表，只能包含任意个li元素

`<li></li>`：列表项容器



`<dl></dl>`：描述列表（小标题+列表项），可包含任意个指定DOM结构，DOM结构可以是：

1. 一个或多个 dt 元素，以及一个或多个 dd 元素
2. 一组或多组div元素，div内包含上述dt dd 结构

`<dt></dt>`：术语定义（小标题）容器

`<dd></dd>`：术语描述（列表项）容器



### 表单

表单中只有一个单行文本输入字段时，按下回车会自动触发submit事件

#### 表单域

`<form></form>`：表单域，可选属性：

- `name="表单名"`：不得为空，且是唯一值
- `accept-charset="空格分隔列表"`：服务器支持的字符编码，默认值为UNKNOWN，表示与该文档编码相同
- `autocomplete="on|off"`：开启表单内input元素的自动补全
- `action="URL"`：表单提交地址 
- `method="post|get|dialog"`：提交方法
  - dialog：若表单在dialog中，在提交时关闭对话框
- `enctype="可选值"`：若提交方法为post，该属性表示提交给服务器的MIME类型，可选值：
  - `application/x-www-form-urlencoded`：默认值
  - `multipart/form-data`：用于提交文件，当表单包含 type=file 的 input 元素时使用此值
  - `text/plain`：用于调试
- `novalidate`：表单提交时不需要验证表单
- `target="xx"`：提交表单后，在哪显示响应信息，[详见a元素同名属性](#链接)



#### 表单控件

`<label></label>`：标签文本，与表单控件关联

- form：详见input元素
- `for="控件id"`：与指定控件关联，也可以直接将控件作为label元素的子元素来建立关联



`<select></select>`：选项菜单，内部可包含任意option或optgroup元素，可选属性：

- name、disabled、form、autocomplete、required：详见input元素
- `multiple`：支持多选，此时会显示为一个滚动列表框
- `size`：显示滚动列表框时，同时可见的行数

`<option></option>`：选项，可选属性：

- `disabled`、`selected`：选项禁用 / 在一开始就选中
- `label="xx"`：表示选项含义的文本，默认值为元素内的文本内容
- `value="xx"`：选项在提交时的值，默认值为元素内的文本内容

`<optgroup></optgroup>`：选项分组，可包含任意个option元素，属性：

- `disabled`：可选属性，选项组禁用
- `label="xx"`：必选属性，选项组名



`<textarea></textarea>`：文本域，可选属性：

- 元素内的内容：作为value的初始值，可通过DOM的value属性更改
- name、disabled、form、maxlength、minlength、placeholder、readonly、required、autocomplete：详见input元素
- `cols="正数"`：可视宽度（列数），默认为20
- `rows="正数"`：显示的文本行数
- `spellcheck="default|true|false"`：是否开启拼写检查
- `warp="soft|hard"`：是否自动换行，hard会在文本到达最大宽度时，自动插入换行符；soft则不会，默认值为soft



`<input />`：表单输入元素，可选属性：

- `value="xx"`：初始值，DOM的value属性表示当前值，提交时会提交当前value值
- `name="名称"`：控件名，提交时随着值一起提交
- `autocomplete="on|off"`：是否开启自动补全，优先级比表单域设置的同名属性高
- `disabled`：禁用
- `form="表单域id"`：关联表单域，若作为表单子元素，则无需填写该属性
- 表单验证属性：[详见MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/input#%E5%B1%9E%E6%80%A7)
- `type="枚举值"`：指定类型，枚举值：
  - text：默认值，单行输入文本框
  - email、number、password、tel、url、search：类似text，浏览器与设备有特殊支持
  - button：无默认行为的按钮，上面显示 value 属性的值
  - reset：重置按钮（不推荐使用） 
  - submit、image：提交按钮 / 图像化的提交按钮
    - image可设置img元素相同的属性
    - 两者的额外属性：formaction、formenctype、formmethod、formnovalidate、formtarget。这些属性用于覆盖表单域中的同名属性（去掉前缀后），取值也与对应属性相同。
  - radio、checkbox：单选框 / 复选框
    - 单选框在name值相同的选项中只能选中一个
    - 两个额外属性：`checked`，表示是否默认选中。DOM实例的checked属性表示当前选中状态
    - 两者在提交时，会将选中状态的input元素的value属性提交
  - range：数字拖动条，默认值为正中间的值，可用min和max规定范围
  - month、week：年月输入 / 年与周数输入
  - date、time、datetime-local：日期输入（年月日）/ 时间输入（时分）/ 日期与时间输入
  - color：指定颜色（取色器）
  - file：文件选择，其value属性为已选择文件的路径，可通过dom实例的files属性获取已选择文件信息。额外属性如下：
    - `accept="逗号分隔的文件类型说明符"`：接受的文件类型，说明符可以是以下类型
      1. 扩展名：如 `.jpg`、`.pdf` 等
      2. MIME类型字符串
      3. `audio/*`、`video/*`、`image/*`：任意音频 / 视频 / 图片文件
    - `capture="可选值"`：不选择文件，改为通过相机或媒体选择器获取（accept需要是图片或视频）
      - 可选值有：user / environment（使用前置/后置摄像头）
    - `multiple`：是否允许多选文件
    - `directory` / `webkitdirectory` ：非标准属性。是否切换为文件夹选择模式
  - hidden：不显示的控件，但值仍会提交到服务器



### 多媒体

`<img />`：图像元素（行内）

- 可选属性： 
  - `src="图像URL"`
  - `alt="文本"`：alt为替换文本，图像无法显示时显示的文字
  - `width/height="像素格式"`：图像宽高
  - `decoding="枚举值"`：图像解码方式，枚举值：sync（同步解码），async（异步解码），auto（默认值，浏览器自动判断）
  - `loading="枚举值"`：图像加载方式，枚举值：eager（立即加载），lazy（当图片和视口接近到某个距离时才加载）
  - `sizes="(max-height: 1500px) 1000px, 800px"`：根据媒体查询调整图片的预期宽度，需要同时设置srcset，否则无效
  - `srcset="URL 条件, URL2 条件2"`：达成指定条件时，改变URL，可选条件：
    - `2x`：在屏幕密度为2x时，该条件无需设置sizes，较为方便
    - `256w`：单位为像素，w仅做关键字，根据sizes指定的预期宽度匹配该条件（取最接近的）
  - `crossorigin`：[详见link的同名属性](#链接)
- 服务器端图像映射：待补充



`<video></video>`：视频，可包含任意个source元素，可选属性：

- `crossorigin`：[详见link的同名属性](#链接)
- `src="url"`：视频源，也可由source元素提供
- `poster="url"` ：封面图片
- `width/height="数值"` 设置宽高（单位：像素），不支持百分比
- `controls`：显示播放控制面板
- `muted`：静音播放
- `autoplay`：视频就绪自动播放（chrome需添加muted才生效）
- `loop`：循环播放
- `preload="枚举值"`：视频播放前的动作（若设置了autoplay则忽略此属性）
  - auto：预加载
  - none：不预加载视频
  - metadata：仅预先获取视频的元数据（如长度）



`<source />`：多媒体资源，可选属性：

- `src="url"`：资源的源
- `type="MIME类型"`：资源类型，如：`video/mp4`、`video/ogg` 等



`<audio></audio>`：音频，可包含任意个source元素

- 可选属性：src、controls、autoplay（chrome自动禁用）、loop、crossorigin、muted、preload：详见video元素



### 其他

按钮元素：`<button></button>`

水平分割线（horizontal rule）：`<hr />`

ruby注释（音标）：`<ruby></ruby>`

- `<ruby>夼<rp>(</rp><rt>kuang</rt><rp>)</rp></ruby>`
- rt内表示注释，rp内表示兼容性不好时显示内部内容，兼容性好时不显示内部内容

度量衡（类似磁盘空间指示条）：`<meter></meter>`

- `<meter value="5" min="0" low="2" high="7" max="10">5 out of 10</meter>`
  当value大于high值时会变色警告

进度条：`<progress max="100" value="0"></progress>`

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



## 字符实体

XML和HTML的字符实体引用：[参考W3school](https://www.w3school.com.cn/html/html_entities.asp)

| 原义字符 | 等价字符实体 |
| :------- | :----------- |
| space    | `&nbsp;`     |
| <        | `&lt;`       |
| >        | `&gt;`       |
| "        | `&quot;`     |
| '        | `&apos;`     |
| &        | `&amp;`      |