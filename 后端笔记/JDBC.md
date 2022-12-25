# JDBC

> Java Data Base Connectivity，是使用Java语言操作关系型数据库的一套API
>
> sun公司只是提供了JDBC API（接口），由各数据库厂商负责实现。这样就可以用一套API操作所有数据库。使用不同的数据库，只要用数据库厂商提供的数据库驱动程序即可

## 连接路径

语法：`jdbc:mysql://ip或域名:端口号/数据库名称?参数1=值1...`

示例：`jdbc:mysql://127.0.0.1:3306/db1`

细节：

* 如果连接的是本机mysql服务器，且默认端口是3306，则url可以简写为：`jdbc:mysql:///数据库名称?参数`

* 配置 useSSL=false 参数，禁用安全连接方式，解决警告提示

## 基本程序结构

将驱动jar包导入项目：`mysql-connector-java-5.1.48.jar`

```java
try {
    Class.forName("com.mysql.jdbc.Driver"); // 加载驱动
    String url = "jdbc:mysql://127.0.0.1:3306/db1";
    String username = "root";
    String password = "1234";
    Connection conn = DriverManager.getConnection(url, username, password); // 获取连接对象
    Statement stmt = conn.createStatement(); // 获取执行sql对象
    ResultSet result = stmt.executeQuery("SELECT * FROM users"); // 执行sql语句, 获取结果集
} catch (SQLException e) {
    e.printStackTrace();
} catch (ClassNotFoundException e) {
    e.printStackTrace();
} finally {
    // 关闭资源，后调用的先关闭。关闭之前，要判断对象是否存在
    if (result != null) {
        try {
            result.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    if (stmt != null) {
        try {
            stmt.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    if (conn != null) {
        try {
            conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

## API详解

### DriverManager

驱动管理类，用于注册驱动以及获取数据库连接

- 注册驱动：

  ```java
  static {
      try {
          DriverManager.registerDriber(new Driver());
      } catch (SQLException var1){
          throw new RuntimeException("Can't register driver!")
      }
  }
  
  // 只需要加载Driver类，就会执行上面的静态代码块来加载驱动
  // MySQL5之后的驱动包可以省略注册的步骤，会自动加载jar包中的Driver类
  Class.forName("com.mysql.jdbc.Driver");
  ```

- 获取数据库连接：[连接路径](##连接路径)

  ```java
  static Connection getConnection(String url, String username, String password)
  ```

### Connection

用于获取执行SQL的对象以及管理事务

- 获取执行SQL对象

  ```java
  // 普通执行SQL对象
  Statement createStatement()
  
  // 预编译SQL的执行SQL对象，防止SQL注入
  PreparedStatement prepareStatement(String sql)
  
  // 执行存储过程的对象
  CallableStatement prepareCall(String sql)
  ```

- 事务管理

  ```java
  // 设置autoCommit
  void setAutoCommit(Boolean autoCommit)
  
  // 提交事务
  void commit()
  
  // 回滚事务
  void rollback()
  ```

### Statement

用于执行SQL语句

```java
// 执行DDL、DML语句
int executeUpdate(String sql) // UPDATE语句:返回受影响的行数

// 执行DQL语句
ResultSet executeQuery(String sql)
```

### ResultSet

```java
// 将光标从当前位置向后移动一行，并判断移动后的行是否为有效行(有无数据)
boolean next()

// 获取数据
int getInt(String 列名)
int getInt(int 列编号) // 列编号从1开始
String getString()
...
```

### PreparedStatement

> 预编译SQL语句并执行：预防SQL注入问题
>
> 使用PreparedStatement并开启预编译功能，会在获取对象时检查与编译，在重新设置参数时不再重新检查编译。这样就提高了性能。
>
> 开启预编译：创建连接时附带参数 `useServerPrepStmts=true`

- 获取对象：

  ```java
  // SQL语句中的参数值，使用?占位符替代
  String sql = "select * from user where username = ? and password = ?";
  // 通过Connection对象获取，并传入对应的sql语句
  PreparedStatement pstmt = conn.prepareStatement(sql);
  ```

- 为sql中的?赋值：index为?的位置编号，从左到右从1开始

  ```java
  void setInt(int index, int value)
  void setString(int index, String value)
  ...
  ```

- 执行SQL语句：

  ```java
  int executeUpdate() // 执行DDL、DML语句
  ResultSet executeQuery() // 执行DQL语句
  ```

## 操作数据库流程

* 将sql语句发送到MySQL服务器端

* MySQL服务端会对sql语句进行如下操作

  * 检查SQL语句的语法是否正确

  * 编译SQL语句。将SQL语句编译成可执行的函数
    * 检查SQL和编译SQL花费的时间比执行SQL的时间还要长。

    * 开启预编译并使用preparedStatement以提高性能

  * 执行SQL语句

## 数据库连接池

> 是个容器，负责分配、管理数据库连接。
>
> 它允许应用程序重复使用一个现有的数据库连接，而不是再重新建立一个
>
> 释放空闲时间超过最大空闲时间的数据库连接来避免因为没有释放数据库连接而引起的数据库连接遗漏

实现：

* 标准接口：DataSource

  官方(SUN) 提供的数据库连接池标准接口，由第三方组织实现此接口。该接口提供了获取连接的功能：

  ```java
  Connection getConnection()
  ```

  直接通过连接池（DataSource）获取 `Connection` 对象

* 常见的数据库连接池：DBCP、C3P0、Druid

  * Druid连接池是阿里巴巴开源的数据库连接池项目，性能比其他两个更好



Driud的使用：

- 导入jar包：`druid-1.1.12.jar`

- 定义配置文件：创建`druid.properties`

  ```properties
  driverClassName=com.mysql.jdbc.Driver
  url=jdbc:mysql:///db1?useSSL=false&useServerPrepStmts=true
  username=root
  password=1234
  # 初始化连接数量
  initialSize=5
  # 最大连接数
  maxActive=10
  # 最大等待时间
  maxWait=3000
  ```

- 使用代码：

  ```java
  public class DruidDemo {
      public static void main(String[] args) throws Exception {
          // 加载配置文件
          Properties prop = new Properties();
          prop.load(new FileInputStream("src/druid.properties"));
          // 获取连接池对象
          DataSource dataSource = DruidDataSourceFactory.createDataSource(prop);
          // 获取数据库连接
          Connection connection = dataSource.getConnection();
          ...
      }
  }
  ```


## Mybatis

> 是一款优秀的持久层框架，用于简化 JDBC 开发
>
> - 持久层：负责将数据到保存到数据库的那一层代码
>   * JavaEE三层架构：表现层、业务层、持久层
> - 框架：一个半成品软件，是一套可重用的、通用的、软件基础代码模型
>
> idea相关：使用MyBatisX插件优化开发

### 导入依赖

```xml
<dependencies>
    <!--mybatis 依赖-->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.5</version>
    </dependency>
    <!--mysql 驱动-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.46</version>
    </dependency>
    <!--junit 单元测试-->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13</version>
        <scope>test</scope>
    </dependency>
    <!-- 添加slf4j日志api -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>1.7.20</version>
    </dependency>
    <!-- 添加logback-classic依赖 -->
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.2.3</version>
    </dependency>
    <!-- 添加logback-core依赖 -->
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-core</artifactId>
        <version>1.2.3</version>
    </dependency>
</dependencies>
```

### 配置文件

在resources目录下创建文件：`mybatis-config.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <typeAliases>
        <package name="com.jdbc.pojo"/>
    </typeAliases>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <!--数据库连接信息-->
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql:///mybatis?useSSL=false"/>
                <property name="username" value="root"/>
                <property name="password" value="1234"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
       <!--加载sql映射文件-->
       <mapper resource="UserMapper.xml"/>
    </mappers>
</configuration>
```

- environments节点：配置数据库连接环境信息。可以配置多个environment节点，通过default属性对应切换不同的environment
  - environment节点下属性：
    - transactionManager：事务管理
    - dataSource：数据库连接信息
- typeAlias节点：类型别名
  - 在package节点中设置包名后，在SQL映射文件中的resultType属性可以直接写该包下的类名。如`User`

### sql映射文件

将sql语句写于xml文件中：名称通常为`表名Mapper.xml`，如`UserMapper.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="test">
    <select id="selectAll" resultType="com.jdbc.pojo.User">
        select * from tb_user;
    </select>
    <insert id="add">
    	insert into tb_brand (brand_name, company_name) values (#{brandName}, #{companyName});
	</insert>
    <update id="update">
    	update tb_brand set brand_name = #{brandName}
	</update>
    <delete id="deleteById">
    	delete from tb_brand where id = #{id};
	</delete>
</mapper>
```

- select节点：用于查询数据
  - id：该sql语句的唯一标识
  - resultType属性：返回的类
- insert节点：用于插入数据
  - 方法返回主键值：insert节点设置属性`useGeneratedKey="true" keyProperty="id"`
- update节点：用于更新数据，方法返回整数，表示受影响的行数
- delete节点：用于删除数据

#### SQL片段

将可复用的SQL片段抽取到mapper下的sql标签中（例如其别名使得名称与POJO类中一致）

```xml
<sql id="brand_column">
	id, brand_name as brandName, company_name as companyName, ordered, description, status
</sql>
<select id="selectAll" resultType="brand">
    select
    <include refid="brand_column" />
    from tb_brand;
</select>
```

#### resultMap

将结果的部分字段名映射为POJO类中的属性名

```xml
<resultMap id="brandResultMap" type="brand">
    <!-- id节点用于完成主键字段的映射，result用于一般字段的映射 -->
    <id .../>
	<result column="brand_name" property="brandName"/>
    <result column="company_name" property="companyName"/>
</resultMap>
<select id="selectAll" resultMap="brandResultMap">...</select>
```

#### 传入参数

- SQL语句的格式为：

  ```xml
  <select id="selectById" resultMap="brandResultMap">
      select * from tb_brand where id = #{id};
  </select>
  ```

  - mybatis提供了两种参数占位符：

    * `#{}`：执行SQL时，会将`#{}`占位符替换为`?`，底层使用的是 `PreparedStatement`

    * `${}`：拼接SQL。底层使用的是 `Statement`，会存在SQL注入问题。一般用于不固定表名和列名

- Mapper接口传入参数的方法有：

  - 方法一：为接口方法的参数添加注解 `@Param(参数名)` ，传递给sql中的对应参数
  - 方法二：接口方法的参数为一个实体类对象，会将其成员传递给sql中的对应参数
  - 方法三：接口方法的参数为一个map集合对象，会将其成员传递给sql中的对应参数

- Mapper参数传递：

  - 当传入多个参数时，Mybatis 会将这些参数封装成 Map 集合对象，值就是参数值，而键在没有使用 `@Param` 注解时有以下命名规则：
    - 以 arg 开头  ：第一个参数叫 arg0，第二个参数叫 arg1...
    - 以 param 开头 ： 第一个参数叫 param1，第二个参数叫 param2...
    - 若有键添加了`@Param`注解，会将对应的argx替换为注解值，param不替换
  - 传入单个参数时：
    - POJO和MAP：直接使用
    - Collection：封装值到map集合，键名为arg0和collection
    - List：封装值到map集合，键名为arg0、collection和list
    - Array：封装值到map集合，键名为arg0和array
    - 其他类型：直接使用

#### 特殊字符处理

- 转义：`<` 转义为 `&lt;`
- `<![CDATA[内容]]>`：内容无需转义

#### 动态SQL

根据用户输入或外部条件变化SQL语句

- if标签：当test属性值为true时拼接内部sql语句

  ```xml
  <select ...>
      select * from tb_brand where
      <if test="status != null">
      	and status = #{status}
      </if>
  </select>
  ```

- where标签：用于替换where关键字，会动态地去掉第一个条件前的and，且在没有条件时不加where

  ```xml
  <select ...>
      select * from tb_brand
      <where>
          <if test="status != null">
              and status = #{status}
          </if>
      </where>
  </select>
  ```

- choose标签：相当于java中的switch语句

  ```xml
  <select ...>
      select * from tb_brand
      <where>
          <choose><!--相当于switch-->
              <when test="status != null"><!--相当于case-->
                  status = #{status}
              </when>
              <otherwise><!--相当于default-->
                  1 = 1
              </otherwise>
          </choose>
      </where>
  </select>
  ```

- set标签：用于update节点

  ```xml
  <update id="update">
      update tb_brand
      <set>
          <if test="brandName != null and brandName != ''">
              brand_name = #{brandName},
          </if>
          <if test="status != null">
              status = #{status}
          </if>
      </set>
      where id = #{id};
  </update>
  ```

- foreach标签：用于迭代可迭代对象

  ```xml
  <delete id="deleteByIds">
      delete from tb_brand where id in
      <foreach collection="array" item="id" separator="," open="(" close=")">
          #{id}
      </foreach>
      ;
  </delete>
  ```

  - collection 属性：数组名
    * mybatis会将数组参数，封装为一个Map集合。默认情况下该集合的key为array，值为数组。所以collection属性默认写为array
    * 给接口方法的参数添加@Param注解改变map集合的默认key的名称
  - item 属性：本次迭代获取到的元素
  - separator 属性：集合项迭代之间的分隔符
  - open/close 属性：在拼接SQL语句之前/之后拼接的语句，只会拼接一次

#### 注解实现CRUD

> 注解完成简单功能，配置文件完成复杂功能。要用注解实现复杂SQL，需要使用SQL构建器完成，导致代码混乱

注解：

* 查询 ：@Select
* 添加 ：@Insert
* 修改 ：@Update
* 删除 ：@Delete

代码实现：在接口方法上添加注解：`@Select(value = "select * from tb_user where id = #{id}")`

### java编码

创建POJO类：

```java
public class User {
    private int id;
    private String username;
    private String password;
    private String gender;
    private String addr;
    //省略了 setter 和 getter
}
```

mybatis代码：

```java
// 加载mybatis的核心配置文件，获取 SqlSessionFactory
String resource = "mybatis-config.xml"; // 相对resources根目录的路径
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

// 获取SqlSession对象，用它来执行sql
SqlSession sqlSession = sqlSessionFactory.openSession();

// 执行sql，参数是一个字符串，该字符串必须是映射配置文件的namespace.id
List<User> users = sqlSession.selectList("test.selectAll"); 
System.out.println(users);

// 释放资源
sqlSession.close();
```

### Mapper代理开发

- 定义与SQL映射文件同名的Mapper接口，并且将Mapper接口和SQL映射文件放置在同一目录下

  - 不推荐直接将SQL映射文件放在接口的同一目录下。由于resources目录编译后与java目录同处于class目录下，所有可以将SQL映射文件放在resources目录的相同包结构的目录下
  - 例如：要将Mapper放置于java目录下的com.jdbc.mapper.UserMapper中，则UserMapper.xml需要置于resources/com/jdbc/mapper目录下

- 设置SQL映射文件的namespace属性为Mapper接口的全限定名。`"com.jdbc.mapper.UserMapper"`

- 在Mapper接口中定义方法，方法名是SQL映射文件中sql语句的id

  ```java
  public interface UserMapper {
  	List<User> selectAll();
  }
  ```

- 此时mybatis-config.xml可用包扫描的方式简化SQL映射文件的加载：

  ```xml
  <mappers>
      <!-- <mapper resource="com/jdbc/mapper/UserMapper.xml"/>-->
      <package name="com.jdbc.mapper"/>
  </mappers>
  ```

- java编码：

  ```java
  // 省略加载配置文件和获取sqlSession的代码
  UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
  List<User> users = userMapper.selectAll();
  ```

  