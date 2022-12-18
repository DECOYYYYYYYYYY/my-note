# JDBC

> Java Data Base Connectivity,它是可以执行SQL语句的Java API
>
> sun公司只是提供了JDBC API（接口），由各数据库厂商负责实现。这样就可以用一套API操作所有数据库。使用不同的数据库，只要用数据库厂商提供的数据库驱动程序即可

## 基本程序结构

```java
Connection connection = null;
Statement statement = null;
ResultSet resultSet = null;

try {

    // 加载驱动方式1: 会导致驱动会注册两次，过度依赖于mysql的api，脱离的mysql的开发包，程序则无法编译
    //DriverManager.registerDriver(new com.mysql.jdbc.Driver());

    // 加载驱动方式2: 驱动只会加载一次，不需要依赖具体的驱动，灵活性高
    Class.forName("com.mysql.jdbc.Driver");

    //获取与数据库连接的对象-Connetcion
    connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/zhongfucheng", "root", "root");

    //获取执行sql语句的statement对象
    statement = connection.createStatement();

    //执行sql语句,拿到结果集
    resultSet = statement.executeQuery("SELECT * FROM users");

    //遍历结果集，得到数据
    while (resultSet.next()) {
        System.out.println(resultSet.getString(1));
        System.out.println(resultSet.getString(2));
    }
} catch (SQLException e) {
    e.printStackTrace();
} catch (ClassNotFoundException e) {
    e.printStackTrace();
} finally {
    // 关闭资源，后调用的先关闭。关闭之前，要判断对象是否存在
    if (resultSet != null) {
        try {
            resultSet.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    if (statement != null) {
        try {
            statement.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    if (connection != null) {
        try {
            connection.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```
