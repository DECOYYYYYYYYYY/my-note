# Node.js

## 简介

Node.js是一个能够在服务器端运行JavaScript的开放源代码、跨平台JavaScript运行环境。

Node在处理请求时是单线程的，但是在后台拥有一个I/O线程池。

在Node中，每个js文件中的js代码都是独立运行在一个函数中，而非全局作用域，其他模块无法访问变量和函数，该函数会自动创建：

- `function(exports, require, module, __filename, __dirname){...}`
- `module`：代表当前模块本身          `exports`：exports = module.exports

模块化：见[JavaScript.md](./JavaScript.md) - CommonJS模块章节

## 包规范

包（Packages）是一个目录，是方便分发推广基于 CommonJS 规范实现的**应用程序**或**类库**

包的基本规范：

- 包都要以一个单独的目录而存在
- package.json（描述文件）必须在包的顶层目录下
  - 文件必须符合JSON格式，可包含以下属性：`name`（必须，包名），`version`（必须，版本），`main`（必须，入口文件），`description`，`keywords`
- 可选目录（在顶层目录下）：bin（可执行二进制文件）、lib（js代码）、doc（文档）、test（单元测试）

包描述文件：是一个json文件，位于包的根目录下（不能写注释）

字段：“name”、“description”、“version”、“keywords”、“maintainers”、“dependencies”。。。

## NPM

帮助完成第三方模块的发布、安装、依赖。

### 常用命令

- `npm -v`

- `npm search 包名`

- `npm init` 在当前目录初始化`package.json`文件

- `npm install [包名] [操作符]` 在当前项目安装指定依赖包（`install`可以简写为`i`）

  - 包名：
    - 若不指定包名，则安装`package.json`中的所有依赖
    - 包名后可紧跟 `@版本号` 表示安装指定版本的包
  - 操作符：
    - `--save 或 -S` 添加到生产依赖
    - `--save-dev 或 -D` 添加到开发依赖
    - `-g` 全局安装
    - `--registry url` 使用指定url作为镜像源进行安装（仅当次安装有效）

- `npm remove 包名 [操作符]` 移除包（`remove`可以简写为`r`）


  - `npm config set registry url` 设置镜像源

    - 淘宝源：`http://registry.npm.taobao.org`

## Buffer

Buffer：缓冲区，Node.js内置的类。

- 结构与数组类似，Buffer中可以存储二进制文件，性能强于传统数组。可以通过索引操作元素。
- 在Buffer中以二进制存储，在显示时都是以十六进制的形式呈现，每个元素占8位，范围从00-ff。超出范围的截取二进制最后8位
- Buffer的大小一旦确定，则不能再修改

创建：`let buf = Buffer.from(str)`

实例属性：

- `buf.length` 占用内存的大小（Bytes）

静态方法：

- `Buffer.from(二进制数据)` 将数据保存在buffer中，返回Buffer对象
- `Buffer.alloc(字节数)` 返回指定大小的Buffer对象
- `Buffer.allocUnsafe(字节数)` 返回指定大小的Buffer对象，创建时不清空内存的数据，有残留数据
- `Buffer.toString()` 返回转换出来的字符串
- `Buffer.concat(Buffer数组, 生成buffer的字节数)`：合并生成大buffer实例



## fs

fs模块用于操作文件系统

引入：`const fs = require("fs")`

同步文件写入：通过返回值获取文件

- `fs.openSync(路径, '操作类型'[, mode])` 同步打开文件，返回文件的描述符，可用于操作文件
  - 操作类型：`w` 写（不存在文件则创建）、`r` 读（不存在则异常）、`a` 追加（不存在则创建）
  - `mode`：操作权限，一般不传

- `fs.writeSync(文件描述符, '内容'[, 写入起始位置[, '编码类型']])` 同步写入文件
  - 编码类型默认为`'utf-8'`

- `fs.closeSync(文件描述符)` 保存并关闭文件（同步）

异步文件写入：通过回调函数的参数获取文件

- `fs.open(路径,'操作的类型'[, mode], (err, fd)=>{})` 异步打开文件，无返回值
  - 回调函数参数：
    - `err` 错误对象，若没有错误则为`null`
    - `fd` 文件的描述符
- `fs.write(文件描述符, '内容' [, 写入起始位置[, '编码']], (err, fd)=>{})` 异步写入文件
- `fs.close(文件描述符, (err)=>{})` 关闭文件

简单文件写入：

- `fs.writeFile(文件路径, 数据[, 配置对象], (err)=>{})`   写入数据并保存关闭
  - 配置对象：默认为`{encoding: 'utf-8', flag: 'w', mode: 0o666}`

简单文件读取：

- `fs.readFile('文件路径'[, 配置对象], (err, data)=>{})` 读取文件，data为`Buffer`对象

流式文件写入：同步、异步、简单写入不适合大文件的写入，性能差，容易导致内存溢出

- `let ws = fs.createWriteStream(文件路径[, 配置对象])` 创建一个可写流
- `ws.on/once('事件名', fn)` 监听流，执行回调函数。once绑定的事件触发一次后失效
  - 可选事件名：`'open'`、`'close'`
- `ws.write(内容) / ws.end(内容)` 写入内容 / 关闭流

流式文件读取：适合用于大文件

- `let rs = fs.createReadStream('文件路径')`
- `rs.on('data', (data)=>{})` 绑定data事件后会自动读取，读取完自动关闭
- `rs.pipe(ws)` 将可读流中的内容直接输出到可写流，完成后自动关闭两个流

其他方法：

- `fs.existsSync('文件路径')` 检查文件是否存在（异步方法已废弃）
- `fs.stat('文件路径', (err, stat)=>{}}) / fs.statSync('文件路径')` 获取文件信息
  - `stat`参数 / 同步方法返回值为对象，保存了文件状态的相关信息：
    - `stat.isFile()`
    - `stat.isDirectory()`
- `fs.unlink('文件路径',fn) / fs.unlinkSync('文件路径')` 删除文件，建议使用同步方法
- `fs.readdir('路径'[, 配置对象], (err, files)=>{}) / fs.readdirSync('路径'[, 配置对象])` 读取一个目录的目录结构，返回一个字符串数组
- `fs.truncate('路径', 字节数, callback)/ fs.truncateSync('路径', 字节数)` 截断文件（将文件修改为指定大小）
- `fs.mkdir('路径'[, 配置对象], callback)/ fs.mkdirSync('路径'[, 配置对象])` 创建一个目录
- `fs.rmdir('路径'[, 配置对象], callback)/ fs.rmdirSync('路径'[, 配置对象])` 删除一个目录
- `fs.rename('老路径', '新路径', (err)=>{})/ fs.renameSync('老路径', '新路径')` 重命名文件
- `fs.watchFile('文件路径'[, 配置对象],(curr, prev)=>{})` 当文件发生变化时，执行回调函数
  - 回调函数参数：`curr` 当前文件的状态；`prev` 修改前文件的状态。两个参数都是stats对象



## path

`const {resolve} = require('path');`

拼接绝对路径：`resolve(__dirname, '路径')` 返回字符串，变量`__dirname`代表当前文件所在目录的绝对路径
