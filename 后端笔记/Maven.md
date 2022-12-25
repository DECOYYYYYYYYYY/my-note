# Maven

一个采用纯Java编写的项目管理工具

用于管理项目的整个生命周期，包括清除、编译，测试，报告、打包、部署等等

## Maven目录结构

- conf/Settings.xml 配置文件

  - 更改默认用户库：`<settings>`节点下添加

    ```xml
    <localRepository>D:\maven\maven_repository</localRepository>
    ```

    - 通过maven下载的jar包都会存储到指定的个人仓库中

## 项目目录结构

https://segmentfault.com/a/1190000013582912

- `src目录`
  - `main目录`
    - `java目录` 
    - `resources目录`
  - `test目录` 测试用
- `pom.xml` maven配置文件

## 生命周期

maven对项目构建的生命周期划分：

* clean ：清除。
* default ：核心工作，包含编译→测试→报告→打包→安装
* site ： 产生报告，发布站点等。这套声明周期一般不会使用

生命周期常用命令：

> 在同一套生命周期内，执行后面的命令，前面的所有命令会自动执行：执行install时，会依次执行mvn compile、mvn test、mvn package、mvn install

- `mvn clean`：删除编译好的项目信息（target目录）
- `mvn compile`：编译项目（将src/main的代码编译，放置在target目录下）
- `mvn test`
- `mvn package`
- `mvn install`

## pom.xml

project object model 项目对象模型 ，它是maven核心配置文件

project节点内的节点：

- `<modelVersion>4.0.0</modelVersion>` 必须项
  - 4.0.0，是当前仅有的可以被Maven2&3同时支持的POM版本
- `<groupId>otowa.user.dao</groupId>` 必须项，组织标识
- `<artifactId>user-dak</artifactId>` 必须项，唯一标识符
- `<version>0.0.1-SNAPSHOT</version>` 必须项，项目版本号
  - maven坐标：使用groupId、artifactId和version来唯一确定一个项目
- `<dependencies></dependencies>` 依赖
- `<packaging>war</packaging>` 设置打包方式为war包格式（默认为jar包）

  - 若报错需添加插件：

    ```xml
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-war-plugin</artifactId>
    <version>3.3.1</version>
    ```

- `<build></build>` 部署

## 依赖管理

> **本地仓库**：Maven会把项目所构建出来的jar包等资源存放在本地仓库中。需要jar包的时候，Maven首先去本地仓库中寻找jar包
>
> **私有服务器**：存储一些jar包的服务器
>
> **中心仓库**：当Maven在本地仓库和私服找不到需要的jar包的时候，就去中心仓库中下载对应的jar包。中心仓库的URL配置于：`apache-maven-3.6.2\lib\maven-model-builder-3.6.2.jar\org\apache\maven\model\POM.xml`文件中

添加依赖：在dependencies节点内添加

```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
    <scope>compile</scope>
</dependency>
```

- scope节点：指定该依赖的作用范围，默认值为compile

  | **依赖范围** | 编译classpath | 测试classpath | 运行classpath | 例子              |
  | ------------ | ------------- | ------------- | ------------- | ----------------- |
  | **compile**  | Y             | Y             | Y             | logback           |
  | **test**     | -             | Y             | -             | Junit             |
  | **provided** | Y             | Y             | -             | servlet-api       |
  | **runtime**  | -             | Y             | Y             | jdbc驱动          |
  | **system**   | Y             | Y             | -             | 存储在本地的jar包 |

  

## 插件

格式：在build节点下添加

```xml
<plugins>
    <plugin>
        <groupId>...</groupId>
        <artifactId>...</artifactId>
        <version>...</version>
        <configuration>可选的配置项</configuration>
    </plugin>
</plugins>
```

常见插件：

- Tomcat插件：由maven集成tomcat

  ```xml
  <groupId>org.apache.tomcat.maven</groupId>
  <artifactId>tomcat7-maven-plugin</artifactId>
  <version>2.2</version>
  <configuration>
  	<port>端口号，默认为8080</port>
      <path>访问路径</path>
  </configuration>
  ```
  

## IDEA相关

快捷执行命令：插件市场内安装Maven Helper插件，在选中项目右键，可以通过Run Maven执行命令

快捷导入jar包坐标：pom.xml中按alt+insert，选择Dependency，搜索并选中坐标