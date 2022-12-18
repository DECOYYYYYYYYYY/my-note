# javaWeb

## 综述

### javaWeb目录

目录结构：

- `webapps目录`
  - `web应用目录`
    - `HTML/jsp文件` 可存放多个
    - `lib目录`
      - `jar文件`
    - `WEB-INF目录`
      - `web.xml` web程序的主要配置文件
      - `taolib.tld`
      - `classes目录` 放置class文件标签处理类

[指定主页为其中一个html文件](https://segmentfault.com/a/1190000013122831)：web.xml添加

```xml
<welcome-file-list>
    <welcome-file>helloword2.html</welcome-file>
</welcome-file-list>
```

## Tomcat

一个运行JAVA的网络服务器，底层是Socket的一个程序，它也是JSP和Serlvet的一个容器

### 目录结构

> 访问url：`http://localhost:8080/web应用名/xxx.html`

- bin：启动和关闭tomcat的bat文件
- conf：配置文件
  - `server.xml`：用于配置server相关的信息，比如tomcat启动的端口号，配置主机(Host)
  - `web.xml`：配置与web应用（web应用相当于一个web站点）
  - `tomcat-user.xml`：配置用户名密码和相关权限
- lib：放置运行tomcat运行需要的jar包
- logs：存放日志
- webapps：放置web应用，见[javaWeb目录](##javaWeb目录)
- work工作目录：该目录用于存放jsp被访问后生成对应的server文件和.class文件

### 配置虚拟目录

虚拟目录映射：将目录不在webapps下的web应用目录，交给tomcat管理

配置：

- 方法一：在conf/server.xml文件中的`<Host>`节点下添加

  ```xml
  <Context path="/web1" docBase="D:\web1"/>
  ```

- 方法二：在conf/Catalina/localhost目录下创建`web应用名.xml`

  ```xml
  <?xml version="1.0" encoding="UTF-8"?> 
  <Context 
      docBase="D:\web1" 
      reloadable="true"> 
  </Context> 
  ```

### 配置虚拟主机

用同一个Tomcat服务器运行多个web应用

配置：在conf/server.xml文件中添加

```xml
<Host name="域名" appBase="D:\web1">
    <Context path="/web1" docBase="D:\web1"/>
</Host>
```

- appBase：该路径下的所有子目录自动被部署为应用

配置临时域名：C:\Windows\System32\drivers\etc\hosts文件添加

```
127.0.0.1        域名
```

## Servlet

处理浏览器带来HTTP请求，并返回一个响应给浏览器，从而实现浏览器和服务器的交互

https://www.zhihu.com/question/321913492

### 基本程序结构

```java
import javax.servlet.*;
public class MyServlet implements Servlet {...}
```

创建一个类，实现Servlet接口，重写以下方法：

- init：初始化
- destroy：销毁
- service：服务
- ServletConfig：Servlet配置
- getServletInfo：Servlet信息



service方法：

- 向浏览器输出：`servletResponse.getWriter().wirte(...);`