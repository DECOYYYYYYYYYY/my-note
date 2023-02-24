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
  // 底层实现
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
    * 检查SQL和编译SQL花费的时间比执行SQL的时间还要长

    * 开启预编译并使用preparedStatement以提高性能（仅需检查编译一次）

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

