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

maven软件构建的生命周期：清除→编译→测试→报告→打包（jarwar）→安装→部署

生命周期命令：

- `mvn clean`：删除编译好的项目信息（target目录）
- `mvn compile`：编译项目（将src/main的代码编译，放置在target目录下）
- `mvn test`
- `mvn package`
- `mvn install`：执行mvn compile、mvn test、mvn package所做的工作

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

- scope节点：可取值有test、compile等。默认值为complie。
  - test范围的依赖不会包含在发布的jar包中