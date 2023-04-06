# Nginx

## 介绍

Nginx（读音为“engine x”），是一个高性能的HTTP和反向代理的WEB服务器，同时也是一个POP3/SMTP/IMAP代理服务器。

代理与反向代理：都是作为服务器和客户端的中间层，转发响应和请求，用于隐藏真实的IP地址，提高网络性能等。正向代理（代理）用于代理客户端；反向代理用于代理服务器。



架构模式：Nginx包含一个master进程和多个worker进程，master进程用于管理worker进程，worker进程用于处理用户请求。



日志：

- access.log：用来记录用户所有的访问请求。
- error.log：记录nginx本身运行时的错误信息，不会记录用户的访问请求

## Nginx服务器的控制

`kill -信号 PID`：为master进程/worker进程传递信号

- 获取master进程的：
  - 方法一：`ps -ef | grep nginx`
  - 方法二：`cat /usr/local/nginx/logs/nginx.pid`
- 信号：
  - `TERM/INT`：强制关闭整个服务
  - `QUIT`：关闭整个服务
  - `HUP`：重读配置文件并使用服务对新配置项生效
  - `USR1`：重新打开日志文件，可以用来进行日志切割
  - `USR2`：平滑升级到最新版的nginx
    - 升级：直接替换sbin目录下的nginx文件为新版本，传递信号进行升级。升级后，系统中将有两个master进程，旧版本的进程记录在`/usr/local/nginx/logs/nginx.pid.oldbin`中，需给它发送QUIT信号
  - `WINCH`：所有子进程不在接收处理新连接，相当于给work进程发送QUIT指令



`nginx [选项]`：控制 nginx 服务器

- 选项：
  - `-?` / `-h`：显示帮助信息
  - `-v`：打印版本号信息
  - `-V`：打印版本号信息和配置信息
  - `-t`：测试配置文件语法是否正确
  - `-T`：测试配置文件语法是否正确并列出用到的配置文件信息
  - `-q`：在配置测试期间禁止显示非错误消息
  - `-s 信号`：传递信号，信号可取值有 ：
    - `stop`：同TERM/INT信号
    - `quit`：同QUIT信号
    - `reopen`：同USR1信号
    - `reload`：同HUP信号
  - `-p 路径`：指定Nginx的prefix路径，(默认为: /usr/local/nginx/)
  - `-c 路径`：指定Nginx的配置文件路径，(默认为: conf/nginx.conf)
  - `-g`：用来补充Nginx配置文件，向Nginx服务指定启动时应用全局的配置



## 配置文件

> Nginx的核心配置文件为 conf/nginx.conf ，conf目录下的 .default 文件均为备份文件

配置文件有三大块：全局块，events块，http块。http块中可以配置多个server块，每个server块又可以配置多个location块。

### 全局块

全局块中可以包含events块、http块、任意多个指令键值对

```conf
指令名	指令值;

events {	 
    ...
}
           
http {		
	...
}
```

指令：

- `user 用户名 [用户组];`：指定worker进程的用户和用户组
  - 默认用户：nobody
- `master_process on|off;`：指定是否开启工作进程
  - 默认值：on
- `worker_processes 整数|auto;`：生成工作进程的数量
  - 默认值：1
- `deamon on|off;`：Nginx是否以守护进程的方式启动
  - 默认值：on
- `pid 存储进程号的文件路径;`：配置master进程的进程号
  - 默认值：/usr/local/nginx/logs/nginx.pid
- `error_log 文件路径 [日志级别];`：配置错误日志存放路径
  - 日志级别：debug | info | notice | warn | error | crit | alert | emerg
  - 默认值：logs/error.log error
- `include 文件路径;`：引入其他配置的内容到此处，该指令可用于所有块中



### events块

events块中可以包含任意多个指令键值对

```conf
events {
	指令名	指令值;
}
```

指令：

- `accept_mutex on|off;`：网络连接序列化
  - 默认值：on
  - 解决惊群问题：若关闭序列化，一个请求会唤醒所有worker进程；开启序列化后，会根据请求一个个唤醒接收。
- `multi_accept on|off;`：是否允许单个worker进程同时接收多个网络连接
  - 默认值：off
- `worker_connections 整数;`：单个worker进程最大的连接数
  - 默认值：512
- `use 函数;`：设置事件驱动模型
  - 函数：可选值有 select | poll | epoll | kqueue 等，一般用epoll来优化
  - 默认值：视操作系统而定



### http块

http块中可以配置任意多个server块，和任意多个指令键值对

```conf
http {
	指令名	指令值;
	server {
		...
	}
}
```

仅适用http的指令：

- `log_format 格式名[escape=default|json|none] string....; `：指定日志输出格式
  - 默认值：combined "..."



适用于http、server、location块的指令：

- `default_type 类型;`：定义默认的MIME Type（即Content-Type字段）
  
  - 默认值：text/plain
  
- `access_log 文件路径 [格式名 [buffer=size]];`：设置用户访问日志
  
  - 默认值：logs/access.log combined
  
- `sendfile on|off;`：是否使用sendfile()传输文件（提高处理静态资源的性能）
  
  - 默认值：off
  
- `tcp_nopush on|off;`：开启后，提高网络包的传输效率（需先开启sendfile）

  - 默认值：off

- `tcp_nodelay on|off`：在keep-alive连接开启时生效，提高网络包传输的实时性

  - 默认值：on

    > nopush与nodelay：nopush会将数据缓存到缓存区，存满后一次发送，减少网络开销，提高效率；nodelay则是有数据就发送，提高实时性。
    >
    > 当两者都开启时，会视情况自动切换，因此**推荐都开启**

- `keepalive_timeout 整数;`：keepalive连接的超时时间（秒数），默认值：75
  
- `keepalive_requests 整数;`：keepalive连接的上限，默认值：100
  
- `root 目录路径;`：设置查找资源的根目录路径，默认值：html

- `index 多个文件名;`：设置默认文件，若路径中没有指定具体访问资源，则依次查找文件，将找到的第一个文件作为请求资源，默认值：index.html

- `error_page 状态码 [=[新状态码]] uri;`：出现指定错误码后，进行重定向

  - uri：
    - 可以是完整的地址，用于使网页重定向
    - 也可以是一段/开头的路径，在nginx内重定向
    - 可以是@xx，进入uri为@xx的location块中
  - =新状态码：以该状态码响应

- `add_header 键名 键值`：添加响应头

  - 跨域访问：

    ```conf
    add_header Access-Control-Allow-Origin *;
    add_header Access-Control-Allow-Methods *;
    ```

- `expires 可选值;`：控制Expires和Cache-Control响应头来控制缓存

  - `off`：默认值，不缓存
  - `[modified] 整数`：指定过期时间，传入整数单位为秒，若为负数意为不缓存，若为正数，表示在 文件最后修改时间+传入秒数 的时刻过期
  - `@15h30m`：每天的15点30分过期
  - `epoch`：设置Expires为1970年0点，即不缓存
  - `max`：设置Expires为2037最后一秒，过期时间设为10年后
  
- `autoindex on|off;`：是否将目录的文件列出，以供下载。默认值：off

- `autoindex_format html|xml|json|jsonp;`：目录列表的格式，默认值：html

- `autoindex_exact_size on|off;`：目录列表格式为html时生效，是否展示文件的详细大小

  - `on`：默认值，展示详细大小，单位为bytes
  - `off`：展示大概大小，单位为KB、MB或GB

- `autoindex_localtime on|off;`：目录列表格式为html时生效，文件展示时间格式

  - `on`：展示文件所在服务器的时间
  - `off`：默认值，展示 GMT 时间



### server块

server块中可以配置任意location块，和任意多个指令键值对

```conf
server {
	指令名	指令值;
	location ... {
		...
	}
}
```

仅适用于server的指令：

- `listen address:port [default_server];`：监听端口，可以指定多个listen指令

  - 地址:端口可以简写为以下形式

    ```
    127.0.0.1:8000 // 监听指定ip和端口
    127.0.0.1 // 监听该ip下所有端口
    *:8000 // 监听指定端口上的连接
    8000 // 监听指定端口上的连接
    ```

  - default_server：标识符，将该虚拟主机设置为默认主机，当 address:port 没有匹配到对应的server时，使用该server

  - 默认值：*:80

- `server_name 多个名称`：设置虚拟主机服务名称

  - 名称：可以是ip地址、localhost或域名。名称本身可以使用首尾`*`通配符、正则表达式（需以`~`作为开始标记）
    - 匹配优先级（当多个server同时匹配成功时）：精准匹配 > 首通配符匹配 > 尾通配符匹配 > 正则匹配 > 默认主机处理
  - 默认值：空字符串



适用于server、location的指令：

- `try_files 多个uri;`：会按顺序寻找资源，把第一个找到的资源作为目标资源

  - 前端history路由的适配：`try_files $uri $uri/ /index.html;`

- `set $开头变量名 值;`：设置变量

- `if (条件表达式) {...}`：if语句

  - 条件表达式：

    - 使用 `=`、`!=` 比较字符串是否相等（字符串不用加引号）

    - 若条件表达式只有一个变量，则仅当它为空或0时，判断为false

    - 使用正则匹配变量：`变量 连接符 正则字符串`，连接符有：

      - `~`：匹配正则表达式过程中区分大小写
      - `~*`：匹配正则表达式过程中不区分大小写
      - `!~` 和 `!~*`：同上匹配，结果取相反值

    - 判断请求的文件是否存在：

      ```
      if (-f $request_filename){...} #判断请求的文件是否存在
      if (!-f $request_filename){...} #判断请求的文件是否不存在
      ```

    - 判断请求的目录是否存在：`-d`、`!-d`，使用同上

    - 判断请求的目录或者文件是否存在：`-e`、`!-e`，使用同上

    - 判断请求的文件是否可执行：`-x`、`!-x`，使用同上

- `break;`：停止并跳出该块的执行，也可用于if中

- `return [状态码] URL;` / `return 状态码 [响应体内容];`：立即完成该请求的处理，向客户端响应，也可用于if中

  

### location块

用于设置请求的URI，其中可以包含任意多指令键值对

```conf
location [符号]uri {
	指令名	指令值;
}
```

uri：以`/`开头

符号：下例中假设uri为 `/abc`

- 不带符号：路径必须以uri开始。例：可以匹配 `/abc/`、`/abcdef` 等
- `=`：路径必须与uri精准匹配。例：仅能匹配 `/abc`、`/abc?a=1` 等
- `~`：表示uri为正则表达式，且区分大小写
- `~*`：表示uri为正则表达式，且不区分大小写
- `^~`：与不带符号一致



指令：

- root、index、error_page、sendfile、tcp_nopush、tcp_nodelay等常用指令见[http块](#http块)
- `alias 路径`：更改location路径，无默认值
  - 与root指令的区别：
    - root指定查找资源的根目录，查找路径为 `root路径 + 请求路径`
    - alias更改请求路径，查找路径为 `alias路径`
  - 若location的uri以 / 结尾，alias也必须是以 / 结尾



location块的匹配：nginx只会执行一个location块

1. 检查使用前缀字符串的 locations，在使用前缀字符串的 locations 中选择最长匹配的，并将结果进行储存。
2. 如果符合带有 = 修饰符的 URI，则立刻停止匹配
3. 如果符合带有 ^~ 修饰符的 URI，则也立刻停止匹配。
4. 如果不满足2和3，使用1中记住的最长匹配前缀字符串location，按照定义文件的顺序，检查正则表达式，匹配到就停止。
5. 当正则表达式匹配不到的时候，使用之前储存的前缀字符串
   



### 全局变量

| 变量               | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| $args              | 变量中存放了请求URL中的请求指令。比如http://192.168.200.133:8080?arg1=value1&args2=value2中的"arg1=value1&arg2=value2"，功能和$query_string一样 |
| $http_user_agent   | 变量存储的是用户访问服务的代理信息(如果通过浏览器访问，记录的是浏览器的相关版本信息) |
| $host              | 变量存储的是访问服务器的server_name值                        |
| $document_uri      | 变量存储的是当前访问地址的URI。比如http://192.168.200.133/server?id=10&name=zhangsan中的"/server"，功能和$uri一样 |
| $document_root     | 变量存储的是当前请求对应location的root值，如果未设置，默认指向Nginx自带html目录所在位置 |
| $content_length    | 变量存储的是请求头中的Content-Length的值                     |
| $content_type      | 变量存储的是请求头中的Content-Type的值                       |
| $http_cookie       | 变量存储的是客户端的cookie信息，可以通过add_header Set-Cookie 'cookieName=cookieValue'来添加cookie数据 |
| $limit_rate        | 变量中存储的是Nginx服务器对网络连接速率的限制，也就是Nginx配置中对limit_rate指令设置的值，默认是0，不限制。 |
| $remote_addr       | 变量中存储的是客户端的IP地址                                 |
| $remote_port       | 变量中存储了客户端与服务端建立连接的端口号                   |
| $remote_user       | 变量中存储了客户端的用户名，需要有认证模块才能获取           |
| $scheme            | 变量中存储了访问协议                                         |
| $server_addr       | 变量中存储了服务端的地址                                     |
| $server_name       | 变量中存储了客户端请求到达的服务器的名称                     |
| $server_port       | 变量中存储了客户端请求到达服务器的端口号                     |
| $server_protocol   | 变量中存储了客户端请求协议的版本，比如"HTTP/1.1"             |
| $request_body_file | 变量中存储了发给后端服务器的本地文件资源的名称               |
| $request_method    | 变量中存储了客户端的请求方式，比如"GET","POST"等             |
| $request_filename  | 变量中存储了当前请求的资源文件的路径名                       |
| $request_uri       | 变量中存储了当前请求的URI，并且携带请求参数，比如http://192.168.200.133/server?id=10&name=zhangsan中的"/server?id=10&name=zhangsan" |
| $uri               | 当前请求的URI，相比$request_uri，不携带请求参数，如上例将返回"/server" |

## 压缩

常用压缩配置：

```
gzip on;
gzip_static on;
gzip_types *;
gzip_comp_level 6;
gzip_min_length 1024;
gzip_vary on;
gzip_disable "MSIE [1-6]\.";
gzip_proxied off;
```

ngx_http_gzip_module模块的相关指令：

> 该模块已内置，下列指令适用于http、server、location等块

- `gzip on|off;`：开启gzip功能，需开启后下列指令才有效，默认关闭
- `gzip_types 多个mime_type;`：要压缩的类型，可用"*"代表所有，默认值为text/html
- `gzip_comp_level 级别;`：压缩级别，级别为数字1-9，默认为1。越大压缩程度越高，也越慢
- `gzip_vary on|off;`：是否携带Vary:Accept-Encoding响应头，默认关闭
- `gzip_buffers number size;`：压缩的缓冲区数量和大小（推荐使用默认值）
  - 默认值：`32 4k` 或 `16 8k`
- `gzip_disable 多个"正则表达式";`：客户端种类匹配时关闭gzip功能，无默认值
  - 正则表达式会去匹配浏览器标志（user-agent），来排除不支持Gzip的浏览器
- `gzip_http_version 1.0|1.1;`：http版本低于指定值时，关闭gzip，默认值为1.1
- `gzip_min_length size;`：传输数据小于指定值时，关闭gzip
  - 默认值为：20
  - nginx中表示数据大小：不带单位为字节，单位为k表示千字节，单位为m表示兆字节
  - 建议设置size为 `1k` 以上
- `gzip_proxied 多个值;`：根据服务器返回结果判断是否关闭gzip，默认值为off
  - 可选值如下：
    - off - 关闭Nginx服务器对后台服务器返回结果的Gzip压缩
    - expired - 启用压缩，如果header头中包含 "Expires" 头信息
    - no-cache - 启用压缩，如果header头中包含 "Cache-Control:no-cache" 头信息
    - no-store - 启用压缩，如果header头中包含 "Cache-Control:no-store" 头信息
    - private - 启用压缩，如果header头中包含 "Cache-Control:private" 头信息
    - no_last_modified - 启用压缩，如果header头中不包含 "Last-Modified" 头信息
    - no_etag - 启用压缩 ,如果header头中不包含 "ETag" 头信息
    - auth - 启用压缩 , 如果header头中包含 "Authorization" 头信息
    - any - 无条件启用压缩



ngx_http_gzip_static_module模块的相关指令：

> 下列指令适用于http、server、location等块
>
> 可通过包管理工具安装 nginx-mod-http-gzip-static 包来获取模块

- `gzip_static on|off|always;`：若服务器上已有同名.gz压缩文件，就会直接发送，提高性能，默认值为off



## 重写与转发

> 重写与转发：重写即重定向，使浏览器直接跳转到新页面；转发则是将该次请求转由目标处理

`rewrite 正则表达式 替换字符串 [flag];`：重写URI，正则会去匹配路径，若匹配成功替换URI为指定值

- 适用块：server、location、if
- 替换字符串：若以 `http://` 或 `https://` 开头，则不继续处理该请求，直接返回
  
  - 在替换字符串中可用 `$1`等 获取正则的匹配组，也可以使用$host等[变量](#全局变量)
- falg可选值：
  - last : 相当于Apache的[L]标记，表示完成rewrite
  - break : 停止执行当前虚拟主机的后续rewrite指令集
  - redirect : 返回302临时重定向，地址栏会显示跳转后的地址
  - permanent : 返回301永久重定向，地址栏会显示跳转后的地址

- 示例：重定向至其他网址

  ```
  server {
      listen 80;
      server_name zfas56dg49zxc465vs4d56fg4s56dfg.shop;
      rewrite /(.*) http://sandpile.shop/$1 permanent;
  }
  ```



`proxy_pass URL;`：指定被代理服务器的地址

- 适用块：location

- URL：包含传输协议、主机名或ip、端口等要素。

  - 若URL以/结尾，会将路径去除location匹配到的部分，作为新路径
  - 若URL不以/结尾，将原路径作为新路径

  ```
  # 把 /app/hello 转发为 http://localhost:8080/hello
  location /app/ { proxy_pass http://localhost:8080/; } 
  
  # 把 /app/hello 转发为 http://localhost:8080/app/hello
  location /app { proxy_pass http://localhost:8080; } 
  ```



`proxy_set_header 键名 键值;`：发送给代理服务器时，更改请求头

- 适用块：http、server、location

- 默认值：

  ```
  proxy_set_header Host $proxy_host;
  proxy_set_header Connection close;
  ```



`proxy_redirect 可选值`：发送给代理服务器时，重置Location、Refresh请求头

- 适用块：http、server、location
- 可选值：
  - `off`：不重置
  - `default`：默认值，将源URI替换为proxy_pass指定的URI
  - `目标代理服务器 替换值`：手动指定



`rewrite_log on|off;`：是否开启重写日志（输出于access.log）的输出，默认off

- 适用块：http、server、location



## SSL

> 以下指令需安装ngx_stream_ssl_module模块，可用于stream和server块中
>
> 设置以下指令前，需为listen中的端口号后追加ssl关键字，如`listen 443 ssl;`

示例：

```
server {
    listen              12345 ssl;

    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_certificate     /usr/local/nginx/conf/cert.pem;
    ssl_certificate_key /usr/local/nginx/conf/cert.key;
    ssl_session_cache   shared:SSL:10m;
    ssl_session_timeout 10m;
}
```

`ssl_protocols 多个协议;`：指定支持的协议

`ssl_certificate 文件路径;`：指定PEM格式证书文件

`ssl_certificate_key 文件路径;`：指定证书密钥文件

`ssl_ciphers 多个值;`：指定启用的加密方式

- 默认值：`ssl_ciphers HIGH:!aNULL:!MD5;`

`ssl_session_cache 多个值;`：指定会话缓存策略

- 默认值：`ssl_session_cache none;`

`ssl_session_timeout 时间;`：会话重用时间

- 默认值：`5m`

## 负载均衡

待补充



## 缓存

待补充