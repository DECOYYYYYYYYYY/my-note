# javaWeb

## 综述

JavaWeb 三大组件(Servlet、Filter、Listener)

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

一个运行JAVA的网络服务器，底层是Socket的一个程序，支持Servlet/JSP少量JavaEE规范。也称为Web容器、Servlet容器

### 目录结构

> 访问url：`http://localhost:8080/web应用名/xxx.html`

- bin：启动和关闭tomcat的bat文件
- conf：配置文件
  - `server.xml`：用于配置server相关的信息，比如tomcat启动的端口号，配置主机(Host)
    - 修改端口号：更改Connector节点的port属性
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

### 部署

一般JavaWeb项目会被打成war包，将war包放入webapps目录下，Tomcat会自动解压war包

### maven插件

需同时指定`<packaging>war</packaging>`

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.tomcat.maven</groupId>
            <artifactId>tomcat7-maven-plugin</artifactId>
            <version>2.2</version>
            <configuration>
                <port>端口号，默认为8080</port>
                <path>访问路径</path>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### IDEA相关配置

xxx.war和 xxx.war exploded这两种部署项目模式的区别

* war模式是将WEB工程打成war包，把war包发布到Tomcat服务器上

* war exploded模式是将WEB工程以当前文件夹的位置关系发布到Tomcat服务器上
* war模式部署成功后，Tomcat的webapps目录下会有部署的项目内容
* war exploded模式部署成功后，Tomcat的webapps目录下没有，而使用的是项目的target目录下的内容进行部署
* 建议选war模式进行部署，更符合项目部署的实际情况

## Servlet

> Servlet接口是JavaEE的规范
>
> 处理浏览器带来HTTP请求，并返回一个响应给浏览器，从而实现浏览器和服务器的交互
>
> Servlet类由web服务器（tomcat）创建，Servlet方法由web服务器调用
>
> https://www.zhihu.com/question/321913492

### 基本程序结构

导入依赖坐标：Tomcat已自带servlet的jar包，所以设置scope为provided

```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
    <scope>provided</scope>
</dependency>
```

依赖引入：

```java
import javax.servlet.*;

public class MyServlet implements Servlet {...}
```

创建一个类，实现Servlet接口，重写以下方法：

- init：初始化
- destroy：销毁
- service：服务
  - 向返回体输出：`servletResponse.getWriter().wirte(...);`

- ServletConfig：Servlet配置
- getServletInfo：Servlet信息

### 生命周期

1. 加载和实例化：默认情况下，当Servlet第一次被访问时，由容器创建Servlet对象
2. 初始化：在Servlet实例化之后，容器将调用Servlet的init()方法初始化这个对象，完成一些如加载配置文件、创建连接等初始化的工作。
3. 请求处理：每次请求Servlet时，Servlet容器都会调用Servlet的service()方法对请求进行处理
4. 服务终止：当需要释放内存或者容器关闭时，容器就会调用Servlet实例的destroy()方法完成资源的释放。在destroy()方法调用之后，容器会释放这个Servlet实例，该实例随后会被Java的垃圾收集器所回收

### Servlet配置

WebServlet：（servlet v3+）给Servlet类添加注解 `@WebServlet("/访问路径")` ，注解的属性有：

- `urlPatterns:String|String[]`：访问路径，与value属性含义一致。url匹配规则如下：
  - 精确匹配：`/user/select`
  - 目录匹配：`/user/*`
  - 扩展名匹配：`*.txt`
  - 任意匹配：`/*` 或 `/`（`/`会覆盖tomcat的defaultServlet，无法访问静态资源）
- `loadOnStartup: int = -1`：指定Servlet对象的创建时机
  - 若值为负整数，则在第一次访问时创建；为0或正整数，则在服务器启动时创建，数字越小优先级越高



在web.xml配置Servlet：在web-app节点下添加

```xml
<!--  Servlet 全类名 -->
<servlet>
    <!-- servlet的名称，名字任意-->
    <servlet-name>demo13</servlet-name>
    <!--servlet的类全名-->
    <servlet-class>com.itheima.web.ServletDemo13</servlet-class>
</servlet>

<!-- Servlet 访问路径 -->
<servlet-mapping>
    <!-- servlet的名称，要和上面的名称一致-->
    <servlet-name>demo13</servlet-name>
    <!-- servlet的访问路径-->
    <url-pattern>/demo13</url-pattern>
</servlet-mapping>
```



### 继承体系

下者继承自上者

- `Servlet`：Servlet体系根接口
- `GenericServlet`：Servlet的抽象实现类
- `HttpServlet`：对HTTP协议封装的Servlet实现类



HttpServlet的使用：

```java
@WebServlet(...)
public class ServletDemo4 extends HttpServlet {
    @Override
    protected void doGet(...) throws ... {
    }
    @Override
    protected void doPost(...) throws ... {
    }
}
```

两个方法中不需要调用super方法

### Request类

继承体系：下者继承自上者

- `ServletRequest`：Java提供的请求对象根接口
- `HttpServletRequest`：Java提供的对Http协议封装的请求对象接口
- `RequestFacade`：Tomcat定义的实现类
  - 由于接口无法创建对象，servlet的service方法、doGet方法中获取的request对象均为`RequestFacade`对象



request对象的方法：

- 获取请求方式：`GET`

  ```java
  String getMethod()
  ```

- 获取虚拟目录（项目访问路径）：`/request-demo`

  ```java
  String getContextPath()
  ```

- 获取URL：`http://localhost:8080/request-demo/req1`

  ```java
  StringBuffer getRequestURL()
  ```

- 获取URI：`/request-demo/req1`

  ```java
  String getRequestURI()
  ```

- 获取请求参数（GET）：`username=zhangsan&password=123`

  ```java
  String getQueryString()
  ```

- 获取指定请求参数（GET）：`zhangsan`

  ```java
  String getParameter(String name)
  ```

- 获取指定请求头：

  ```java
  String getHeader(String name)
  ```

- 获取请求体数据（POST）：

  ```java
  ServletInputStream getInputStream() // 字节数据
      
  BufferedReader getReader() //文本数据
  ```

- 获取请求参数（GET与POST统一处理）：

  ```java
  // 获取所有参数的Map集合，若有多个同名参数，会将值收为数组的元素
  Map<String,String[]> getParameterMap() 
  
  // 根据名称获取参数值（数组）
  String[] getParameterValues(String name)
  
  // 根据名称获取参数值(单个值)
  String getParameter(String name)
  ```

- 请求转发：仅能转发至服务器的内部资源

  ```java
  // 转发请求
  req.getRequestDispatcher("资源B路径").forward(req,resp)
  
  // 存储数据至request域中
  void setAttribute(String name,Object o)
  
  // 根据key删除值
  void removeAttribute(String name)
  
  // 根据key获取值
  Object getAttribute(String name)
  ```

  



中文乱码：

- POST请求：TOMCAT获取流时采用ISO-8859-1解码

  - 更改编码：`request.setCharacterEncoding("UTF-8")`

- GET请求：发送请求时，会进行url编解码，tomcat默认按照ISO-8859-1解码

  - url编码原理：字符串转二进制，每个字节转为2个16进制数并在前面加上%

  - 解决方法：

    ```java
    // 将乱码编码并转为字节数组
    byte[] bytes = username.getBytes(StandardCharsets.ISO_8859_1);
    // 字节数组解码
    username = new String(bytes, StandardCharsets.UTF_8);
    ```



### Response类

继承体系：下者继承自上者

- `ServletResponse`：Java提供的响应对象根接口
- `HttpServletResponse`：Java提供的对Http协议封装的响应对象接口
- `ResponseFacade`：Tomcat定义的实现类
  - servlet的service方法、doGet方法中获取的request对象均为`ResponseFacade`对象



response对象的方法：

- 设置响应状态码：

  ```java
  void setStatus(int statusCode)
  ```

- 设置响应头：

  ```java
  void setHeader(String name,String value)
      
  // 设置content-type
  void setContentType(String value) // "text/html;charset=utf-8"
  ```

- 设置响应体：

  ```java
  // 获取字符输出流
  PrintWriter getWriter()
  
  // 获取字节输出流
  ServletOutputStream getOutputStream()
  ```

- 重定向：指示浏览器请求其他资源，可重定向至服务器外部资源

  ```java
  resp.setStatus(302);
  resp.setHeader("location","资源B的访问路径");
  
  // 可简化为
  resp.sendRedirect("资源B的访问路径")
  ```


## 会话

> 会话：用户打开浏览器，访问web服务器的资源，会话建立，直到有一方断开连接，会话结束
>
> 会话跟踪：一种维护浏览器状态的方法，服务器需要识别多次请求是否来自于同一浏览器，以便在同一次会话的多次请求间共享数据
>
> Cookie：客户端会话技术，将数据保存到客户端，以后每次请求都携带Cookie数据进行访问
>
> Session：服务端会话跟踪技术，将数据保存到服务端

### Cookie

* 发送：

  ```java
  Cookie cookie = new Cookie("key","value"); // 创建并设置数据
  response.addCookie(cookie); // 发送Cookie到客户端
  ```

* 接收：

  ```java
  Cookie[] cookies = request.getCookies();
  cookie.getName(); // 获取某条cookie的键名
  cookie.getValue(); // 获取某条cookie的键值
  ```

* 设置Cookie的存活时间：

  ```java
  cookie.setMaxAge(int seconds)
  ```

* Cookie存储中文：Cookie不允许直接存储中文，需进行URL编解码

  ```java
  // 编码
  Cookie cookie = new Cookie("username", URLEncoder.encode("张三", "UTF-8"));
  // 解码
  URLDecoder.decode(cookie.getValue(),"UTF-8");
  ```

### Session

> JavaEE中提供了HttpSession接口，来实现一次会话的多次请求之间数据共享功能
>
> Session是基于Cookie来实现的，于响应时将id设置为cookie，访问时根据id判断是否为同一浏览器

使用：

* 获取Session对象：

  ```java
  HttpSession session = request.getSession();
  ```

* Session对象的功能：

  ```java
  // 存储数据到 session 域中
  void setAttribute(String name, Object o)
  // 获取值
  Object getAttribute(String name)
  // 删除值
  void removeAttribute(String name)
  // 销毁session
  void invalidate()
  ```



使用细节：

- 只要服务器是正常关闭和启动，session中的数据是可以被保存下来的。由Tomcat完成

  - 钝化：在服务器正常关闭后，Tomcat会自动将Session数据写入硬盘的文件中
    - 钝化的数据路径为：`项目目录\target\tomcat\work\Tomcat\localhost\项目名\SESSIONS.ser`
  - 活化：再次启动服务器后，从文件中加载数据到Session中
    - 数据加载后，`SESSIONS.ser`文件会被删除

- session在默认情况下，无操作，30分钟自动销毁

  * 配置失效时间：在项目的web.xml的web-app节点下配置

    ```xml
    <session-config>
        <!-- 单位为分钟 -->
        <session-timeout>100</session-timeout>
    </session-config>
    ```

## Filter&Listener

### Filter

> 过滤器可以把对资源的请求拦截下来（调用doFilter方法），从而实现一些特殊的功能

使用：实现`javax.servlet.Filter`接口，设置@WebFilter注解表示拦截哪些资源

```java
@WebFilter("/*")
public class FilterDemo implements Filter {
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws ... {
        // 放行前逻辑
        chain.doFilter(request,response); // 放行
        // 放行后逻辑
    }
    @Override
    public void init(...) throws ... {}
    @Override
    public void destroy() {}
}
```

执行流程：执行放行前逻辑 → 放行 → 访问资源（执行访问资源逻辑） → 执行放行后逻辑

过滤器链：配置的多个过滤器，其优先级按照过滤器类名（字符串）排序。`AFilter`优先于`BFilter`

### Listener

监听器，在 `application`，`session`，`request` 三个对象创建、销毁或者往其中添加修改删除属性时自动执行代码的功能组件

`javax.servlet.ServletContextListener`的使用：ServletContext代表web应用，监听其发布成功或销毁

```java
@WebListener
public class ContextLoaderListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        //加载资源
    }
    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        //释放资源
    }
}
```
