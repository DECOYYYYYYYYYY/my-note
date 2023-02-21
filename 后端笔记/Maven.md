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

## 生命周期&命令

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
- `mvn install`：将当前项目安装到本地maven仓库



命令参数：`mvn 指令 [参数]`

- `-P 环境id`：[指定环境](##环境配置)
- `-D skipTests`：[跳过测试](###跳过测试)



## 配置文件

pom.xml（project object model 项目对象模型 ），它是maven核心配置文件

pom.xml（project object model 项目对象模型 ），它是maven核心配置文件

project节点内的节点：

- `<modelVersion>4.0.0</modelVersion>` 必须项

  - 4.0.0，是当前仅有的可以被Maven2&3同时支持的POM版本

- `<groupId>otowa.user.dao</groupId>` 必须项，组织标识

- `<artifactId>user-dak</artifactId>` 必须项，唯一标识符

- `<version>0.0.1-SNAPSHOT</version>` 必须项，项目版本号

  - maven坐标：使用groupId、artifactId和version来唯一确定一个项目

- `<dependencies></dependencies>` 依赖

- `<packaging>war</packaging>` 设置项目打包方式

  - `jar`：默认值

  - `war`：war包格式，若报错需添加插件：

    ```xml
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-war-plugin</artifactId>
    <version>3.3.1</version>
    ```
    
  - `pom`：[聚合或继承](##聚合&继承)

- `<build></build>`

- `<modules></modules>`：[聚合或继承](##聚合&继承)管理的模块

- `<parent></parent>`：[继承](###继承)的父项目信息

- `<dependencyManagement></dependencyManagement>`：[继承](###继承)的依赖管理

- `<properties></properties>`：[属性](##属性)

- `<profiles></profiles>`：[环境配置](##环境配置)

## 依赖管理

> **本地仓库**：Maven会把项目所构建出来的jar包等资源存放在本地仓库中。需要jar包的时候，Maven首先去本地仓库中寻找jar包
>
> **私有服务器**：存储一些jar包的服务器
>
> **中心仓库**：当Maven在本地仓库和私服找不到需要的jar包的时候，就去中心仓库中下载对应的jar包。中心仓库的URL配置于：`apache-maven-3.6.2\lib\maven-model-builder-3.6.2.jar\org\apache\maven\model\POM.xml`文件中

### 添加依赖

> 依赖也是一个项目，添加依赖至当前项目后，可以根据类全名在当前项目访问依赖项目的类。所以可以将多个项目需要的共有模块放置在一个项目里，使用install命令部署到本地仓库，并由多个项目将其设为依赖

在project节点下添加dependencies节点，内部添加dependency节点（依赖）

```xml
<dependencies>
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.1.0</version>
        <scope>compile</scope>
        <!-- <optional>true</optional> -->
        <!-- 
        <exclusions>
            <exclusion>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis</artifactId>
            </exclusion>
        </exclusions>
        -->
    </dependency>
</dependencies>
```

- scope节点：指定该依赖的作用范围，默认值为compile

  | **依赖范围** | 编译classpath | 测试classpath | 运行classpath | 例子              |
  | ------------ | ------------- | ------------- | ------------- | ----------------- |
  | **compile**  | Y             | Y             | Y             | logback           |
  | **test**     | -             | Y             | -             | Junit             |
  | **provided** | Y             | Y             | -             | servlet-api       |
  | **runtime**  | -             | Y             | Y             | jdbc驱动          |
  | **system**   | Y             | Y             | -             | 存储在本地的jar包 |
  
- optional节点：传入true表示该依赖为可选依赖

  - 可选依赖：该依赖无法通过依赖传递被外部获取

- exclusions节点：排除依赖，将已经通过依赖传递获取的依赖排除，无需指定版本号



### 依赖传递

> 当前项目A依赖B，B依赖C，则称B的层级为1度，C为2度，以此类推

依赖传递：A依赖了B和C,B和C有分别依赖了其他jar包，所以在A项目中就可以使用上面所有jar包

依赖冲突：项目依赖的某个包，有多个不同的版本，例：A依赖B和C，而C依赖B的另一个版本

依赖冲突规则：

- 特殊优先：在当前项目配置了相同资源的不同版本，后配置的覆盖先配置的
- 路径优先：当依赖中出现相同的资源时，层级越深，优先级越低，层级越浅，优先级越高
- 声明优先：当资源在相同层级被依赖时，配置顺序靠前的覆盖配置顺序靠后的




### 常用依赖坐标

```xml
<dependencies>
    <!--servlet-->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.1.0</version>
        <scope>provided</scope>
    </dependency>
    <!--spring-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.2.10.RELEASE</version>
    </dependency>
    <!-- spring-jdbc -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>5.2.10.RELEASE</version>
    </dependency>
    <!--springMVC-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.2.10.RELEASE</version>
    </dependency>
    <!--mybatis-->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.5</version>
    </dependency>
    <!--mysql驱动-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.46</version>
    </dependency>
    <!--druid-->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.2.10</version>
    </dependency>
    <!--jackson-->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.9.0</version>
    </dependency>
</dependencies>
```

## 插件

格式：在project节点下添加

```xml
<build>
    <plugins>
        <plugin>
            <groupId>...</groupId>
            <artifactId>...</artifactId>
            <version>...</version>
            <configuration>可选的配置项</configuration>
        </plugin>
    </plugins>
</build>
```



## 聚合&继承

> 聚合工程与父工程通常为一个不具有业务功能的空工程，根目录下有且仅有一个pom文件

### 聚合

> 使用聚合工程可以将多个工程编组，通过对聚合工程进行构建，实现对所包含的模块进行同步构建

使用：

- 创建仅有pom文件的空工程，作为聚合工程

- 将项目的打包方式改为pom

- 将所要管理的项目添加进pom.xml

  ```xml
  <modules>
      <module>../maven_02_ssm</module>
      <module>../maven_03_pojo</module>
      <module>../maven_04_dao</module>
  </modules>
  ```

- 执行 `mvn compile`，所有被管理的项目都会执行编译操作

### 继承

> 继承：描述的是两个工程间的关系，与java中的继承相似，子工程可以继承父工程中的配置信息，常见于依赖关系的继承

使用：

- 创建仅有pom文件的空工程，作为父工程

- 将项目的打包方式改为pom

- 将所要管理的项目（子工程）添加进pom.xml

  ```xml
  <modules>
      <module>../maven_02_ssm</module>
      <module>../maven_03_pojo</module>
      <module>../maven_04_dao</module>
  </modules>
  ```

- 抽取子工程共用的jar包，维护在父项目中

- 删除子工程中已被抽取到父项目的jar包，并在子工程pom中添加父项目信息：

  ```xml
  <!--配置当前工程继承自parent工程-->
  <parent>
      <groupId>com.itheima</groupId>
      <artifactId>maven_01_parent</artifactId>
      <version>1.0-RELEASE</version>
      <!--设置父项目pom.xml位置路径-->
      <relativePath>../maven_01_parent/pom.xml</relativePath>
  </parent>
  ```

  

子项目依赖管理：

> 依赖置于父工程，可能导致某些子项目引入了多余的依赖，需进行优化
>
> 依赖管理：父工程中声明依赖及其版本信息，子工程中可以选择不引入该依赖，也可以引入依赖且不指定版本号，将版本交由父工程统一管理

- 父项目的pom中的project节点下添加：

  ```xml
  <!--定义依赖管理-->
  <dependencyManagement>
      <dependencies>
          <dependency>
              <groupId>junit</groupId>
              <artifactId>junit</artifactId>
              <version>4.12</version>
              <scope>test</scope>
          </dependency>
      </dependencies>
  </dependencyManagement>
  ```

- 子项目添加上述依赖（无需指定版本号）

  ```xml
  <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <scope>test</scope>
  </dependency>
  ```

  

## 属性

### 基本使用

- 当前pom中定义属性

  ```xml
  <properties>
      <spring.version>5.2.10.RELEASE</spring.version>
  </properties>
  ```

- 当前工程和子工程pom中通过 `${spring.version}` 使用属性

  ```xml
  <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-core</artifactId>
      <version>${spring.version}</version>
  </dependency>
  ```



### maven中的属性

分类：

- 自定义属性：用户自定义的属性
- 内置属性：`${内置属性名}`，如`${project.basedir}`
- Setting属性：`${setting.属性名}`
- Java系统属性：`${系统属性分类.系统属性名}`
- 环境变量属性：`${env.环境变量属性名}`



内置属性查询：`mvn help:system`



### 配置文件使用属性：

- 父工程中定义属性

  ```xml
  <properties>
     <jdbc.url>jdbc:mysql://127.1.1.1:3306/ssm_db</jdbc.url>
  </properties>
  ```

- `.properties`文件中引用属性

  ```properties
  jdbc.url=${jdbc.url}
  ```

- 设置maven过滤文件范围：默认情况下Maven从当前项目的`src\main\resources`下读取文件进行打包。如果需要让子项目的资源文件被maven读取并使用该项目的属性，需对该项目的pom进行配置

  ```xml
  <build>
      <resources>
          <!--设置资源目录-->
          <resource>
              <directory>../maven_02_ssm/src/main/resources</directory>
              <!--设置能够解析${}，默认是false -->
              <filtering>true</filtering>
          </resource>
      </resources>
  </build>
  ```

  - 同时配置所有子项目：`${project.basedir}` 表示当前项目所在目录,子项目继承了父项目，相当于所有的子项目都添加了资源目录的过滤

    ```xml
    <resource>
        <directory>${project.basedir}/src/main/resources</directory>
        <filtering>true</filtering>
    </resource>
    ```

  - 打包过程可能报错：maven认为项目为web项目，但找不到web.xml

    - 方法一：为报错项目补充web.xml

    - 方法二：配置maven打包war时，忽略web.xml检查

      ```xml
      <build>
          <plugins>
              <plugin>
                  <groupId>org.apache.maven.plugins</groupId>
                  <artifactId>maven-war-plugin</artifactId>
                  <version>3.2.3</version>
                  <configuration>
                      <failOnMissingWebXml>false</failOnMissingWebXml>
                  </configuration>
              </plugin>
          </plugins>
      </build>
      ```

      

## 环境配置

父工程设置多个环境，并指定默认激活环境

```xml
<profiles>
    <!--开发环境-->
    <profile>
        <id>env_dep</id>
        <properties>
            <jdbc.url>jdbc:mysql://127.1.1.1:3306/ssm_db</jdbc.url>
        </properties>
        <!--设定是否为默认启动环境-->
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
    </profile>
    <!--生产环境-->
    <profile>
        <id>env_pro</id>
        <properties>
            <jdbc.url>jdbc:mysql://127.2.2.2:3306/ssm_db</jdbc.url>
        </properties>
    </profile>
    <!--测试环境-->
    <profile>
        <id>env_test</id>
        <properties>
            <jdbc.url>jdbc:mysql://127.3.3.3:3306/ssm_db</jdbc.url>
        </properties>
    </profile>
</profiles>
```

- 指定环境：`mvn install -P 环境id` 不指定时使用默认环境



### 跳过测试

跳过所有测试：`mvn 指令 -D skipTests`

跳过指定测试：

- 在父工程中的pom.xml中添加测试插件配置

  ```xml
  <build>
      <plugins>
          <plugin>
              <artifactId>maven-surefire-plugin</artifactId>
              <version>2.12.4</version>
              <configuration>
                  <skipTests>false</skipTests>
                  <excludes>
                      <exclude>**/BookServiceTest.java</exclude>
                  </excludes>
              </configuration>
          </plugin>
      </plugins>
  </build>
  ```

  - `skipTests`：如果为true，则跳过所有测试，如果为false，则不跳过测试
  - `excludes`：哪些测试类不参与测试，即排除，针对skipTests为false来设置的
  - `includes`：哪些测试类要参与测试，即包含，针对skipTests为true来设置的



## 私服

安装Nexus