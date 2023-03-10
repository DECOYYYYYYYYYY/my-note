# MySQL

从MySQL 8.0开始，系统表全部换成事务型的InnoDB表，默认的MySQL实例将不包含任何MyISAM表，除非手动创建MyISAM表

窗口函数

Using filesort是MySQL中一种速度较慢的外部排序，可优化索引来尽量避免出现，提高执行速度。

## 综述

### 数据库

数据库系统（Database System）构成：

- 数据库（Database）：用于存储数据的地方
- 数据库管理系统（Database Management System，DBMS）：用于管理数据库的软件，位于用户与操作系统之间，对数据库进行统一管理
- 数据库应用程序（Database Application）：为了提高数据库系统的处理能力所使用的管理数据库的软件补充



数据库分类：

- 关系型数据库管理系统：RDBMS(Relational Database Management System)
  - MySQL、Oracle、Sql Server、DB2、SQLlite
  - 数据以表格的形式出现，每行为各种记录名称，每列为记录名称所对应的数据域，许多的行和列组成一张表单，若干的表单组成database
  - MySQL是一个小型关系型数据库管理系统
- 非关系型数据库：NoSQL（Not Only SQL）
  - Redis、MongoDB
  - 指数据以对象的形式存储在数据库中，而对象之间的关系通过每个对象自身的属性来决定

### RDBMS术语

- **数据库**： 数据库是一些关联表的集合。
- **数据表**：表是数据的矩阵。在一个数据库中的表看起来像一个简单的电子表格。
- **列**：一列（数据元素）包含了相同类型的数据，例如邮政编码的数据。
- **行**：一行（=元组，或记录）是一组相关的数据，例如一条用户订阅的数据。
- **冗余**：存储两倍数据，冗余降低了性能，但提高了数据的安全性。
- **主键**：主键是唯一的，不得为空。可定义表中一列或多列为主键。
- **外键**：外键用于建立表连接，是表中的一个字段，对应另一个表的主键
- **存储引擎**：外键约束不能跨引擎使用
- **复合键**：复合键（组合键）将多个列作为一个索引键，一般用于复合索引。
- **索引**：使用索引可快速访问数据库表中的特定信息。索引是对数据库表中一列或多列的值进行排序的一种结构。类似于书籍的目录。
- **参照完整性**：参照的完整性要求关系中不允许引用不存在的实体。与实体完整性是关系模型必须满足的完整性约束条件，目的是保证数据的一致性。
  - 存在外键关联时，不允许直接删除父表

### 基本命令

Windows管理MySQL：

```cmd
cd mysql/bin #进入MySQL安装目录的bin目录
mysqld --console #启动服务
mysqladmin -uroot shutdown #关闭服务
```

Linux管理MySQL：

```cmd
ps -ef | grep mysqld #检查服务是否已经启动
root@host# cd /usr/bin
./mysqld_safe & #启动服务
./mysqladmin -u root -p shutdown #关闭服务
```

MySQL连接命令：`mysql 可选参数`

- `-h 主机名`：指定主机名或ip，默认是localhost
- `-u 用户名`：指定用户名
- `-p密码`：指定登录密码
  - 如果该参数后面有一段字段，则该段字符串将作为用户的密码直接登录。如果后面没有内容，则登录的时候会提示输入密码
  - 注意：该参数后面的字符串和-p之前不能有空格
- `-P 端口号`：指定服务器的端口号，默认为3306
- `数据库名`：指定数据库名
- `-e SQL语句`：在登录后执行-e后面的命令或SQL语句并退出

### 存储引擎

> 外键约束不能跨引擎使用

- `MyISAM`
  - 最好使用定长数据列而非变长数据列，使检索更快，空间换时间
  - 采用表级锁，其表锁是使用最为广泛的锁类型
  - 不支持事务
  - 可以索引BLOB和TEXT列
- `InnoDB`
  - InnoDB数据表存储不分固定长度和可变长度，因此推荐使用变长数据列，节省空间
  - 采用行级锁
  - 支持事务、`ROW_FORMAT`表设置、外键完整性约束
  - CPU效率高，需要高性能首选
  - 支持在线添加主键、在线增长VARCHAR列长、在线重命名索引
- `MEMORY`
  - 数据存储到内存中，为查询和引用其他表数据提供快速访问
  - 采用表级锁
  - 不支持BLOB或TEXT列
  - 当不再需要表中的内容时，需要删除或格式化表
- `MERGE`
  - 是一组MyISAM表的组合，对MERGE表的插入操作通过INSERT_METHOD子句完成
  - 格式：`ENGINE=MERGE UNION=(已有MyISAM表名列表) INSERT_METHOD=FIRST|LAST`
    - FIRST：数据添加到第一个表
- `BerkeleyDB`
  - 简称BDB，不由MySQL开发
  - 默认采用页面锁，也支持表级锁

## 数据类型&运算符

### 数据类型

整数型：

- `TINYINT`
  - 大小：1字节
  - 范围（有符号）：`(-128, 127)`
  - 范围（无符号）：`(0, 255)`
- `SMALLINT`
  - 大小：2字节
  - 范围（有符号）：` (-32768, 32767)`
  - 范围（无符号）：`(0, 65535)`
- `MEDIUMINT`
  - 大小：3字节
  - 范围（有符号）：`(-8 388 608, 8 388 607)`
  - 范围（无符号）：`(0, 16 777 215)`
- `INT`或`INTEGER`
  - 大小：4字节
  - 范围（有符号）：`(-2.147E9, 2.147E9)`
  - 范围（无符号）：` (0, 4.294E9)`
- `BIGINT`
  - 大小：8字节
  - 范围（有符号）：`(-9.223E18, 9.223E18)`
  - 范围（无符号）：`(0, 1.845E19)`
- `整数型(整数n)` 指定数据类型的显示宽度
  - 显示宽度：与存储无关，数值最小显示位数，数值位数不足时用空格填充
  - 默认显示宽度：`TINYINT(4)`、`SMALLINT(6)`、`MEDIUMINT(9)`、`INT(11)`、`BIGINT(20)`

浮点型和定点数型：

- `FLOAT`
  - 大小：4字节
  - 范围（有符号）：`(-3.402E38, -1.175E-38), 0, (1.175E-38, 3.402E38)`
  - 范围（无符号）：`0, (1.175E-38, 3.402E38)`
- `DOUBLE`
  - 大小：8字节
  - 范围（有符号）：`(-1.798E308, -2.225E-308), 0, (2.225E-308, 1.798E308)`
  - 范围（无符号）：`0, (2.225E-308, 1.798E308)`

- `DECIMAL(M,N)` 定点数型
  - 大小：M+2字节
- `浮点型或定点数型(整数M, 整数N)` 指定精度范围
  - M为精度，表示有效数字位数，N为标度，表示小数的位数
  - 超出精度范围会四舍五入
- 浮点数容易在减法和比较时产生误差，对精确度要求较高时，使用定点存储

文本字符串型：

- `CHAR(整数M)`
  - 固定长度字符串，保存时在右侧填充空格，检索得到的值将删除空格（传入时右侧的空格会被删除）
  - 大小：M字节，1≤M≤255
- `VARCHAR(整数M)`
  - 变长类型字符串，最大长度为M
  - 大小：L+1字节，L≤M且1≤M≤65535
  - 变长类型的大小由实际存储值的长度L决定
- `TEXT`，变长类型字符串，TEXT类型分为：
  - `TINYTEXT`，大小L+1字节，最大长度 `255(2^8–1)`
  - `TEXT`，大小L+2字节，最大长度 `65535(2^16–1)`
  - `MEDIUMTEXT`，大小L+3字节，最大长度 `16777215(2^24–1)`
  - `LONGTEXT`，大小L+4字节，最大长度 `4294967295(2^32–1)`或 `4GB`
- `ENUM('值1','值2'...)`
  - 枚举可以有`0~65535`个元素，大小为1或2字节
  - 传入值尾部的空格会被删除，每个枚举值有一个索引值
  - NULL值的索引为NULL，排在所有值前，空字符串索引为0，排在NULL后
  - 若ENUM列被声明为NOT NULL，默认值为允许值列表的第一个元素
- `SET('值1','值2'...)`
  - SET可以有`0~64`个成员，大小为1、2、3、4、8字节
  - 与ENUM的区别：SET可以从定义的列值中选择多个字符的联合，并会自动去重
    - `SET('a','b','c')`，插入`'a, b, a'`，最终值为`'a, b'`

二进制字符串型：

- `BIT(整数M)`
  - 位字段类型，M为每个值的位数，范围为`1~64`，默认为1。大小为`(M+7)/8`字节
  - 自动在值的左侧填充0
  - 可传入类型：十进制数值
- `BINARY(整数M)`
  - 二进制字节字符串，固定长度，大小为M字节
  - 自动在右侧补`\0`至最大长度
- `VARBINARY(整数M)`
  - 变长类型，大小为L+1字节
- `BLOB`
  - 二进制大对象，变长类型，有以下类型：
  - `TINYBLOB`，大小L+1字节，最大长度 `255(2^8–1)`
  - `BLOB`，大小L+2字节，最大长度 `65535(2^16–1)`
  - `MEDIUMBLOB`，大小L+3字节，最大长度 `16777215(2^24–1)`
  - `LONGBLOB`，大小L+4字节，最大长度 `4294967295(2^32–1)`或 `4GB`
- `JSON`
  - 内部会被转换成BLOB型

日期与时间型：

> 任何标点符号都可以替代`-`或`:`作为日期或时间的分隔符

- `YEAR`
  - 大小：1字节
  - 日期格式：`YYYY`
  - 日期范围：`1901 ~ 2155`
  - 可以插入两位字符串或数字，00会被转换为0000，70及以上转换为19xx，01-69转换为20xx
- `TIME`
  - 大小：3字节
  - 日期格式：`HH:MM:SS`
  - 日期范围：`-838:59:59 ~ 838:59:59`
  - 传入数据可以使用的格式：
    - 字符串：`'HH:MM:SS'`、`'HH:MM'`、`'D HH:MM'`、`'D HH'`、`'SS'`、`'HHMMSS'`
      - D表示日，可取0~34之间的值
      - 使用`'D HH'`格式时，小时必须使用双位数值
      - `'HHMMSS'` 不足6位时，假定最右边两位为秒
    - `CURRENT_DATE`或`NOW()`
- `DATE`
  - 大小：3字节
  - 日期格式：`YYYY-MM-DD`
  - 日期范围：`1000-01-01 ~ 9999-12-3`
  - 传入数据可以使用的格式：
    - 字符串：`'YYYY-MM-DD'`、`'YYYYMMDD'`、`‘YY-MM-DD’`、`‘YYMMDD’`
    - 数值：`YY-MM-DD`、`YYMMDD`
    - `CURRENT_DATE`或`NOW()`
- `DATETIME`
  - 大小：8字节
  - 日期格式：`YYYY-MM-DD HH:MM:SS`
  - 日期范围：`1000-01-01 00:00:00 ~ 9999-12-31 23:59:59`
  - 传入数据可以使用的格式：
    - 字符串：`'YYYY-MM-DD HH:MM:SS'`、`'YYYYMMDDHHMMSS'`、`‘YY-MM-DD HH:MM:SS’`、`‘YYMMDDHHMMSS’`
    - 数值：`YYYYMMDDHHMMSS`、`YYMMDDHHMMSS`
    - `CURRENT_DATE`或`NOW()`
- `TIMESTAMP`
  - 大小：4字节
  - 日期格式：`YYYY-MM-DD HH:MM:SS`
  - 日期范围：`1970-01-01 00:00:01 UTC ~ 2038-01-19 03:14:07 UTC`
  - 传入数据可以使用的格式：同`DATETIME`
  - 与`DATETIME`的区别：`TIMESTAMP`在存储时根据当前时区转换为UTC保存，检索时再转换回当前时区，在插入记录且没有指定TIMESTAMP的值时，自动设为当前时间
  - 指定微秒精度：`TIMESTAMP(位数)`，保留微秒数，位数取值为0-6，默认值为0

### 运算符

> 在Windows平台下，MySQL不区分大小写，因此字符串比较函数也不区分大小写。若要进行区分大小写的比较，在字符串前添加`BINARY`关键字。例如，`BINARY‘a’=‘A’`

- 算术运算符：`+`、` -`、` *`、` /`、` %`
  - 对0的除法和取余返回NULL
- 比较运算符：比较为真返回1，为假返回0
  - `> `、`< `、`=`、`<=>`安全等于、` >=`、` <=`、` != 或 <>`
    - `=`和`<=>`会进行类型转换再比较，字符串向数字方向转换；
    - `<=>`可用于NULL比较；以上除`<=>`外运算符：若有一个参数为NULL，结果为NULL
  - ` IS NULL 或 ISNULL`、`IS NOT NULL`
  - `值 BETWEEN 下界 AND 上界`：区间判断
    - 上下界也包含在区间内，对于字符串：按字母顺序比较
  - `LEAST(值1, ...)` 返回最小值，有NULL则返回NULL
  - ` GREATEST(值1, ...)` 返回最大值，有NULL则返回NULL
  - `值 IN (值1, ...)` 判断是否为列表的值
  - `NOT IN` 同上取反
  - ` 值 LIKE '匹配条件'` 判断是否满足条件
    - 可用通配符有：`%` 匹配任意数目的字符（包括零字符），`_`匹配一个字符
  - ` 值 REGEXP '匹配条件'` 判断是否正则匹配
- 逻辑运算符：
  - `NOT 或 !`
    - 操作数为0，结果为1；操作数为非零值，结果为0；
    - 操作数为NULL时，返回值为NULL
  - `AND 或 &&`
    - 所有操作数均为非零值并且不为NULL时，结果为1；
    - 当一个或多个操作数为0时，结果为0；
    - 其余情况返回值为NULL
  - `OR 或 ||`
    - 两个操作数均为非NULL值且任意一个操作数为非零值时，结果为1，否则结果为0；
    - 有一个操作数为NULL，且另一个操作数为非零值时，则结果为1，否则结果为NULL；
    - 两个操作数均为NULL时，则所得结果为NULL
  - `XOR` 异或
    - 任意一个操作数为NULL时，返回值为NULL；
    - 如果两个操作数都是非0值或者都是0值，则返回结果为0；
    - 如果一个为0值、另一个为非0值，返回结果为1
- 位运算符：`&`、`|`、`~`取反、`^`异或、`<<`左移、`>>`右移
  - 左移右移空位用0补齐

## SQL语言

```sql
--    --单行注释
#   --单行注释
/*...*/   --多行注释
/*! ... */  --多行注释（MySQL会执行这些语句，其他数据库管理系统会忽略注释）
```

### 数据定义语言

> DDL，Data Definition Language，用于定义数据库对象（数据库，表，字段）

数据库操作：

- `SHOW DATABASES;` 查询所有数据库
- `SELECT DATABASE();` 
- `USE 数据库名;` 使用数据库
- `CREATE DATABASE [IF NOT EXISTS] 数据库名 [DEFAULT CHARSET 字符集][COLLATE 排序规则];` 创建数据库
  - `IF NOT EXISTS` 如果不存在相同名称的数据库则创建
- `DROP DATABASE [IF EXISTS] 数据库名;` 删除数据库
  - `IF EXISTS` 若存在则删除



创建表：

```mysql
CREATE TABLE 表名 
(
    字段名1 类型 [列级别约束条件] [列...], 
    ...
    [表级别约束条件/索引]
    [表...]
)[表设置] [表...];
```

- 列级别约束：

  - 主键约束：`[PRIMARY] KEY`
  - 非空/可空约束：`NOT NULL | NULL`
  - 唯一性约束：`UNIQUE` 不得出现重复值，能且只能出现一次空值
    - 一个表中可以有多个字段声明唯一性，但只能有一个主键约束声明
    - 主键不允许有空值，但唯一性字段允许空值存在
  - 默认值约束：`DEFAULT 默认值`
  - 自增约束：`AUTO_INCREMENT` 为该字段插入NULL值，系统自动为其分配唯一编号
    - 该编号从1开始，每新增一条记录，值加1
  - `VIRTUAL | STORED`
  - 注释：`COMMENT 注释`

- 表级别约束：

  > 表级别约束前可以添加`[CONSTRAINT 约束名]`声明约束名

  - 主键约束：`PRIMARY KEY(字段名)`
    - 联合主键：字段名之间用`,`分隔
  - 外键约束：`FOREIGN KEY(外键字段名) REFERENCES 主表名(主键名)` 
    - 将该表（子表）的指定字段，关联至主表的对应主键，类型必须一致
    - 联合主键：主键名之间用`,`分隔
  - 唯一性约束：`UNIQUE(字段名)`
  
- [索引](##索引)：定义形式同表约束

- 计算列：其值始终通过其他列计算而来

  - 语法：`字段名 类型 [GENERATED ALWAYS] AS (表达式) [列约束]`
  - 使用方式同普通列

- 表设置：

  - 指定存储引擎：`ENGINE=存储引擎名`
  - 设置字符集：`[DEFAULT] CHARSET=字符集` 或 `CHARACTER SET=字符集`
    - v8+默认字符集为`utf8mb4`
  - 自增键起始序号：`AUTO_INCREMENT=1`
  - 设置比较规则：`COLLATE=utf8mb4_general_ci`
  - 设置表注释：`COMMENT='用户信息表'`
  - 设置行格式：`ROW_FORMAT=行格式名称`
    - 行格式：`DEFAULT`、`FIXED`、`DYNAMIC`、`COMPRESSED`、`REDUNDANT`、`COMPACT`
      - FIXED：静态表，表中不存在变长类型的字段
      - DYNAMIC：动态表



表信息：

- 查看表：`SHOW TABLES;`
- 查看表结构：`DESCRIBE|DESC 表名;`
  - 描述属性如下：
    - `NULL`：该列是否可以存储NULL值
    - `Key`：该列是否已编制索引
      - `PRI`：该列是表主键的一部分
      - `UNI`：该列是UNIQUE索引的一部分
      - `MUL`：在列中某个给定值允许出现多次
    - `Default`：该列是否有默认值，有的话指定值是多少
    - `Extra`：表示可以获取的与给定列有关的附加信息，如AUTO_INCREMENT等
- 查看表详细结构：`SHOW CREATE TABLE 表名[\G];`

  - 显示创建表时的`CREATE TABLE`语句，表名后可加`\G`参数，优化显示结果
- 查看系统支持的存储引擎：`SHOW ENGINES;`
- [索引相关](###索引相关SQL)



表操作：

- 修改表名：`ALTER TABLE 表名 RENAME [TO] 新表名;`
- 更改表的存储引擎：`ALTER TABLE 表名 ENGINE=存储引擎名`
- 删除表：`DROP TABLE [IF EXISTS] 表1,表2...;`
- 格式化表：`TRUNCATE TABLE 表名;` 删除并重新创建
- `ALTER TABLE 表名 子句` 搭配子句进行操作，子句可有：
- 添加字段：`ADD 字段名 类型[列约束][FIRST|AFTER 已存在字段名];`
  - `FIRST`：添加为表的第一个字段
    - `AFTER 已存在字段名`：添加至指定字段后
    - 不指定FIRST或AFTER：默认添加至最后
  - 修改字段数据类型：`MODIFY 字段名 新数据类型[列约束][FIRST|AFTER 字段名];`
  - 修改字段名和类型：`CHANGE 旧字段名 新字段名 新数据类型[列约束];`
  - 修改表设置：`变量名=值;`
  - 删除字段：`DROP 字段名;`
  - 添加约束/索引：`ADD 表级别约束/索引;`
  - 删除外键约束：`DROP FOREIGN KEY 约束名;`
  - 删除唯一性约束：`DROP INDEX 约束名;`
  - 删除主键约束：`DROP PRIMARY KEY;`
  - [删除索引](###索引相关SQL)：`DROP INDEX 索引名;`

### 数据操作语言

> DML，Data Manipulation Language，用来对数据库表中的数据进行增删改
>

插入记录：

```
INSERT INTO 表名[(字段列表)] VALUES(值列表),(...)...;
```

- 该语句仅在MySQL中可用
- 值列表和字段列表的顺序需对应。可以省略字段列表，此时值列表按字段定义顺序排列（表修改导致字段顺序改变也将影响顺序）
- 单次插入多条比多次插入效率更高
- 使用SELECT的结果插入：`INSERT INTO 表名[(字段列表1)] SELECT (字段列表2)...;`
  - MySQL不关心字段名，会将字段列表2按顺序插入字段列表1



更新记录：`UPDATE 表名 SET 列名=值,列名=值... [WHERE 表达式];`

- 若缺省WHERE子句，将应用于表中所有行

删除记录：`DELETE FROM 表名 [WHERE 表达式];`

- 若缺省WHERE子句，将删除表中所有记录。此时使用`TRUNCATE TABLE`性能会更好

### 数据查询语言

> DQL，Data Query Language，用来查询数据库中表的记录
>

```sql
SELECT [DISTINCT] {*|字段列表}
  [
        FROM 表列表
        [ON 表达式]
        [WHERE 表达式]
        [GROUP BY 字段列表 [WITH ROLLUP]]
        [HAVING 表达式]
        [ORDER BY 字段列表]
        [LIMIT [偏移,] 行数]
    ];
```

- `DISTINCT`：去除重复的值
- `{*|字段列表}`：查询的字段
  - 字段别名：`列名/表达式/(SELECT语句) AS 列别名` 表示，表达式中可用集合函数
    - 例如：`COUNT(*) AS Total`
- `FROM 表列表`：查询数据的来源
  - 表别名：`表名 AS 别名`
  - 若这些表中有相同的字段，需要完全限定表名 `表名.字段`
  - 连接查询：多张表通过一个相同的字段进行连接
    - 方法一：`select t1.id,name from t1,t2 WHERE t1.id=t2.id;`
    - 方法二：`select t1.id,name from t1 [INNER] JOIN t2 ON t1.id=t2.id;`
      - 使用JOIN子句时，连接的条件使用ON子句而非WHERE
      - JOIN语法是ANSI SQL的规范，且WHERE可能影响性能，推荐方法二
    - 内连接：`[INNER] JOIN` 两表中都符合连接条件和WHERE的数据才保留
    - 左连接：`LEFT JOIN` 返回 **左表中的所有记录** 和 右表中匹配的记录，如果右表中没有匹配的记录，则对应列用 null 填充
    - 右连接：`RIGHT JOIN` 同上，左右互换
- `WHERE 表达式`：限定查询行必须满足的查询条件
- `GROUP BY 字段列表`：分组查询，按照指定的字段分组（v8+不再自动排序）
  - 通常与集合函数一起使用，如：`SELECT s_id, COUNT(*) AS Total FROM fruits GROUP BY s_id;`
  - `HAVING 表达式` 过滤数据，保留符合表达式的分组
    - HAVING在分组后过滤选择分组；WHERE在分组前选择记录
  - `WITH ROLLUP` 在所有分组后追加一条记录，计算查询出的所有记录的总和
  - 多字段分组：按第1个字段分组，再按第2个字段分组...
- `ORDER BY 字段列表`：查询结果排序，不指定该子句则按插入顺序排序
  - 排序方向：在字段后添加关键字改变排序方向
    - `ASC` 默认，升序；`DESC` 降序。例：`ORDER BY f_price DESC, f_name;`
  - 多字段排序：从第1个字段开始排序，相同的记录再按第2个字段排序...
  - 与分组一起使用：先分组后排序，`ORDER BY`与`WITH ROLLUP`互斥
- `LIMIT [偏移,] 行数`：每次显示查询出来的数据条数
  - 偏移：偏移值为0对应第1条记录，默认值为0
  - 等价语法：`LIMIT 偏移 OFFSET 行数`

集合函数：每个分组可视为独立的表

- `COUNT(*)`  返回表中总的行数，不管某列是否有数值或者为空值
- `COUNT(字段)` 返回指定列下总的行数（忽略NULL行）
- `SUM(字段)` 返回指定列值的总和（忽略NULL行）
- `AVG(字段)`、`MAX(字段)`、`MIN(字段)`

子查询：适用于SELECT、UPDATE和DELETE语句中的所有条件表达式中

- `ANY(SELECT语句) 或 SOME(...)` 
  - 用于比较运算符后，若与子查询返回的某一值比较为TRUE，该比较运算符返回TRUE
- `ALL(...)` 同上，需要所有值都为TRUE
- `EXISTS(...)` 若子查询返回至少一行，EXISTS结果为TRUE
- `字段 IN(...)` 将子查询得到的值列表作为IN操作符的值列表，之后逻辑同IN操作符
- `字段 NOT IN(...)`

合并查询：`SELECT语句 UNION [ALL] SELECT语句`

- 使用ALL关键字：不删除重复行也不对结果自动排序。效率更高

通用表达式（CTE）：是一个临时结果集，仅在单个SQL语句的执行范围内存在

- 可用在所有SELECT、UPDATE、DELETE语句的开头：`WITH ... SELECT ...`

- 创建：`WITH [RECURSIVE] cte名[(字段列表)] AS (SELECT语句), cte名2...`

  - 查询中的列数必须与字段列表的列数相同，若省略字段列表，将使用定义CTE的查询的列列表

  - cte可以引用其他cte

  - 递归CTE：需指定RECURSIVE关键字，包含seed查询和recursive查询，中间用union分隔。seed查询只执行一次，recursive查询会被多次执行。

    ```sql
    WITH RECURSIVE cte(n) AS (
      SELECT 1
      UNION ALL
      SELECT n+1 FROM cte WHERE n<8)
    SELECT * FROM cte;
    ```

    

### 数据控制语言

> DCL，Data Control Language，用来创建数据库用户，控制数据库的访问权限

连接相关：

- 退出连接：`EXIT`
- 重新加载权限：`FLUSH PRIVILEGES`
- 查看连接线程信息：`SHOW [FULL] PROCESSLIST;`
  - 不加FULL关键字只显示前100条
  - 信息中的列：ID、User、Host、db、Command、Time、State、Info
    - Command：当前正在执行的命令，一般取值有Sleep休眠、Query查询、Connect连接
    - Time：状态持续秒数
    - State：当前连接的SQL语句状态
    - Info：显示这个SQL语句



用户管理：

- 创建新用户：`CREATE USER 用户标识列表;`
  - 用户标识：`'用户名'@'允许登录的主机名'[IDENTIFIED BY [PASSWORD] '密码' | IDENTIFIED WITH '插件名' [AS '字符串参数']];`
    - 主机名：默认值为`'%'` 表示对所有主机开放权限
    - IDENTIFIED BY：用于设置密码，若指定PASSWORD参数表示使用哈希值设置密码
    - IDENTIFIED WITH：指定身份验证插件，插件名是带单/双引号的字符串。参数会被传递给插件
    - 若缺省IDENTIFIED子句，表示无需密码
  - 通过操作用户表创建：`INSERT INTO MySQL.user(Host, User, authentication_string) VALUES('host', 'username', MD5('password'));`
- 删除用户：
  - `DROP USER ['用户名'@'主机名'[,'用户名'@'主机名']];`
  - `DELETE FROM MySQL.user WHERE host='hostname' and user='username'`
- 修改密码：更新user表的authentication_string字段（值用MD5函数加密）
- 设置密码过期时间：`ALTER USER '用户名'@'主机' PASSWORD EXPIRE INTERVAL 260 DAY;`
  - 设置永不过期：`ALTER USER '用户名'@'主机' PASSWORD EXPIRE DEFAULT;`



权限管理：

- 授权：`GRANT 权限列表 ON [授权作用对象] 表名列表 TO '用户名'@'主机' [WITH 选项] `
  - 权限：`权限类型[(字段列表)]`，权限类型有：CREATE、DROP、GRANT OPTION等
  - 授权作用对象：可取值有`TABLE`、`FUNCTION`、`PROCEDURE`
  - 表名：`数据库名.表名`
    - 数据库/表名都可用`*`表示所有数据库/表。`*.*`：使权限作用于所有数据库
  - 选项：可以指定多个选项
    - `GRANT OPTION`：被授权的用户可以将这些权限赋予别的用户
    - `MAX_QUERIES_PER_HOUR count`：设置每个小时可以执行count次查询
    - `MAX_UPDATES_PER_HOUR count`：设置每小时可以执行count次更新
    - `MAX_CONNECTIONS_PER_HOUR count`：设置每小时可以建立count个连接
    - `MAX_USER_CONNECTIONS count`：设置单个用户可以同时建立count个连接
  - 使用该语句需要`GRANT OPTION`权限
- 收回权限：`REVOKE 权限列表 ON 表名列表 FROM '用户名'@'主机'列表;`
  - 说明同授权
  - 收回所有权限：`REVOKE ALL PRIVILEGES, GRANT OPTION FROM '用户名'@'主机'列表;`
  - 使用该语句需要数据库全局的 `CREATE USER` 或 `UPDATE` 权限
- 查看权限：
  - `SHOW GRANTS FOR '用户名'@'主机';`
  - `SELECT privileges_list FROM user WHERE user='username', host= 'hostname';`



角色管理：v8+，角色是一些权限的集合

- 创建角色：`CREATE ROLE 角色名;`
- 给用户赋予角色：`GRANT 角色名 TO '用户名'@'主机';`
- 给角色授权：`GRANT 权限列表 ON [授权作用对象] 表名列表 TO 角色名 [WITH 选项] `
- 收回角色权限：`REVOKE 权限列表 ON 表名列表 FROM 角色名;`
- 查看角色信息：`SELECT * FROM mysql.default_roles;`
- 查看角色和用户关系：`SELECT * FROM mysql.role_edges;`
- 删除角色：`DROP ROLE 角色名;`

## MySQL函数

### 数学函数

运算：

- `ABS(x)` 绝对值
- `PI()`返回π
- `SIGN(x)` 根据参数符号，返回`-1、0、1`
- `SQRT(x)` 算术平方根、负数的算术平方根为`NULL`
- `MOD(x, y)` 取x被y除后的余数
- `POW(x, y) 或 POWER(x, y)` x^y
- `EXP(x)` e^x
- `LOG(x)` 底数为e、`LOG10(x)` 底数为10

角度：

- `SIN(x)`、`COS(x)`、`TAN(x)`、`COT(x)` x为弧度
- `ASIN(x)`、`ACOS(x)`、`ATAN(x)` 返回弧度
- `RADIANS(角度x)` 转弧度、`DEGREES(弧度x)` 转角度

取整相关：

- `CEIL(x) 或 CEILING(x)` 向上取整
- `FLOOR(x)` 向下取整
- `ROUND(x[,y])` 四舍五入（保留y位小数）
  - y可以为负数，表示小数点左边`ABS(y)`位都置为0
  - `ROUND (232.38,-2)`，结果为`200`
- `TRUNCATE(x,y)` 保留y位小数，其余部分截去。y为负数时同`ROUND`处理

随机：

- `RAND([整数n])` 返回`[0, 1]`的随机值，可传入n作为种子值

### 字符串函数

长度统计：

- `CHAR_LENGTH(str)` 返回字符数
- `LENGTH(str)` 返回字节数（utf8中，一个汉字3字节，数字或字母1字节）

拼接：

- `CONCAT(s1, ...)` 返回连接后的字符串
  - 参数有NULL则结果为NULL，参数有二进制字符串，则结果为二进制字符串
- `CONCAT_WS(x,s1, ...)` 返回连接后的字符串
  - 第一个参数作为分隔符。分隔符为NULL则结果为NULL。忽略其他为NULL的参数
- `REPEAT(s,n)` 重复n个字符串s，拼接并返回
- `SPACE(n)` 返回由n个空格组成的字符串

大小写转换：

- `LOWER(str) 或 LCASE(str)` 转小写
- `UPPER(str) 或 UCASE(str)` 转大写

截取子串：

- `LEFT(s, n)`、`RIGHT(s, n)` 从s的左/右截取长度为n的子串
- `SUBSTRING(s,n[,len]) 或 MID(s,n,len)`
  - 从s的位置n处（支持负索引）截取长度为len的子串（缺省len则截到结尾）
- `LTRIM(s)`、`RTRIM(s)`、`TRIM(s)` 删除左、右、左右的空格
- `TRIM([s1] FROM s)` 从s的左右删除所有s1子串，s1默认为空格

内容变更：

- `INSERT(s1, x, len, s2)` 
  - 从s1的索引x处开始，长度为len的部分替换为s2
  - 索引从1开始计，索引超出s1长度时，直接返回s1
  - 若有参数为NULL，返回NULL
- `LPAD(s1,len,s2)`、`RPAD(s1,len,s2)` 
  - 返回s1，在s1的左/右侧用s2填充至len长度
  - 若len小于s1的长度，从左/右侧删除字符至len长度
- `REPLACE(s,s1,s2)` 用s2替代s中的所有s1
- `REVERSE(s)` 反转字符串

其他：

- `STRCMP(s1,s2)` 大小比较，若s1小于/等于/大于s2，返回-1/0/1
- `LOCATE(s1,s) 或 POSITION(s1 IN s) 或 INSTR(s, s1)`
  - 返回s1在s中的开始位置
- `ELT(整数n,s1,s2...)` 返回sn，要求n≥1
- `FIELD(s,s1,s2...)` 返回s在后续字符串列表中第一次出现的位置
  - 例：`FIELD('b','a','b')` 返回 2
- `FIND_IN_SET(s1,ls2)` 返回字符串s1在字符串列表ls2中出现的位置
  - 字符串列表为用逗号分隔的字符串，如`'a,b,c'`
- `MAKE_SET(x,s1,s2...)` 按x的二进制选择字符串
  - 例：x为5，二进制为0101，从右往左有1的位数为1和3，则取s1、s3
  - 若取的值为NULL，忽略它
- `CHARSET(str)` 返回字符串使用的字符集（默认gbk）
- `COLLATION(str)` 返回字符串的排列方式
- `CONVERT(str USING 字符集)` 转换字符集

### 日期时间函数

> type：用于EXTRACT函数和计算相关函数
>
> - `MICROSECOND`6位微秒 、`SECOND` 、`MINUTE` 、`HOUR`、`DAY`、`WEEK`、`MONTH`、`QUARTER`、`YEAR`
> - `SECOND_MICROSECOND` 'SS.微秒'、
> - `MINUTE_MICROSECOND` 'mm.微秒'、`MINUTE_SECOND` 'mm:SS'
> - `HOUR_MICROSECOND` 'HH.微秒'、`HOUR_SECOND` 'HH:SS'、`HOUR_MINUTE` 'HH:mm'
> - `DAY_MICROSECOND` 'DD.微秒'、`DAY_SECOND` 'DD HH:mm:SS'、`DAY_MINUTE` 'DD HH:mm'、`DAY_HOUR` 'DD HH'
> - `YEAR_MONTH` 'YY-MM'
>

获取当前：

- `CURDATE() 或 CURRENT_DATE()`
  - 返回`'YYYY-MM-DD'` 或 `YYYYMMDD`
- `CURTIME() 或 CURRENT_TIME()`
  - 返回`'HH:MM:SS'` 或 `HHMMSS`
- `CURRENT_TIMESTAMP()、LOCALTIME()、NOW() 或 SYSDATE()`
  - 返回`'YYYY-MM-DD HH:MM:SS'` 或 `YYYYMMDDHHMMSS`
- `UTC_DATE()`、`UTC_TIME()`、`UTC_TIMESTAMP()` 返回UTC日期/时间/日期与时间，格式同上

格式转换：

- `UNIX_TIMESTAMP(date)` 返回UNIX时间戳（秒数）

- `FROM_UNIXTIME(unix)` 返回普通格式时间

- `TIME_TO_SEC(time)` 转换为秒数，例`01:10:00`返回4200

- `SEC_TO_TIME(seconds)` 秒数转换为时间

  - 返回`'HH:MM:SS'` 或 `HHMMSS`

- `DATE_FORMAT(date, format)` 格式化时间

  - format：`'%D %y %a %d'`，可选说明符如下：

    - | 格式   | 描述                | 格式   | 描述                                       |
      | :----- | :------------------ | ------ | ------------------------------------------ |
      | %a     | 缩写英文星期名      | %p     | AM 或 PM                                   |
      | %b     | 缩写英文月名        | %r     | 12小时制时间（hh:mm:ss AM/PM）             |
      | %c     | 月（0-12）          | %S，%s | 秒（00-59）                                |
      | %D     | 月中天数（1st）     | %T     | 24小时制时间（hh:mm:ss）                   |
      | %d     | 月中天数（00-31）   | %U     | 周（00-53） 周日是一周的第一天             |
      | %e     | 月中天数（0-31）    | %u     | 周（00-53） 周一是一周的第一天             |
      | %f     | 6位微秒             | %V     | 周（01-53） 周日是一周的第一天，与 %X 使用 |
      | %H     | 小时（00-23）       | %v     | 周（01-53） 周一是一周的第一天，与 %x 使用 |
      | %h，%I | 小时（01-12）       | %W     | 英文星期名                                 |
      | %i     | 分钟，数值（00-59） | %w     | 周的天（0-6，0为周日）                     |
      | %j     | 年的天（001-366）   | %X     | 4位年，周日是周的第一天，与 %V 使用        |
      | %k     | 小时（0-23）        | %x     | 4位年，周一是周的第一天，与 %v 使用        |
      | %l     | 小时（1-12）        | %Y     | 4 位年                                     |
      | %M     | 英文月名            | %y     | 2 位年                                     |
      | %m     | 月（00-12）         | %%     | 转义%                                      |
  
- `GET_FORMAT(data/time/datetime, 格式化类型)`

  - 格式化类型包括：`EUR`、`INTERVAL`、`ISO`、`JIS`、`USA`

日期时间提取：

- `EXTRACT(type FROM date)` 自定义提取，返回数字形式（去除所有分隔符）

  - 若用NOW等获取当前时间的函数作为date，会忽略day

- `SECOND(time)` 返回秒数（0-59）

- `MINUTE(time)` 返回分钟数（0-59）

- `DAYOFYEAR(date)` 返回是一年中的第几天（1-366）

- `DAYOFMONTH(date)` 返回是一个月中的第几天（1-31）

- `MONTH(date) 或 MONTHNAME(date)` 返回月份（1-12）

- `QUARTER(date)` 返回季度（1-4）

- `YEAR(date)` 返回年份（0-9999）

- `DAYNAME(date)` 返回星期几（英文，如Sunday）

- `DAYOFWEEK(date)` 返回星期几（1-7，1为周日）

- `WEEKDAY(date)` 返回星期几（0-6，0为周一）

- `WEEK(date[,mode])` 返回是一年中的第几周，mode默认为0

  - | mode | 一周的第一天 | 周数 | 哪周作为第一周        |
    | ---- | ------------ | ---- | --------------------- |
    | 0    | 周日         | 0-53 | 本年第一个周日所在周  |
    | 1    | 周一         | 0-53 | 本年第一个超过3天的周 |
    | 2    | 周日         | 1-53 | 本年第一个周日所在周  |
    | 3    | 周一         | 1-53 | 本年第一个超过3天的周 |
    | 4    | 周日         | 0-53 | 本年第一个超过3天的周 |
    | 5    | 周一         | 0-53 | 本年第一个周一所在周  |
    | 6    | 周日         | 1-53 | 本年第一个超过3天的周 |
    | 7    | 周一         | 1-53 | 本年第一个周一所在周  |

- `WEEKOFYEAR(date)` 相当于mode为3的WEEK函数

计算：

- `DATE_ADD(date, INTERVAL 时间 type)、ADDDATE(...)` 加
  - 时间可以负号开头
- `DATE_SUB(...)、SUBDATE(...)` 减
- `ADDTIME(date, time)、SUBTIME(...)` 
- `DATE_DIFF(date1, date2)` 返回间隔天数（后减前）

### 条件判断函数

- `IF(表达式,v1,v2)` 若表达式为真值，返回v1，否则返回v2
  - 真值判断：`表达式 <> 0 and 表达式 <> NULL`
- `IFNULL(v1,v2)` 若v1不为NULL，返回v1，否则返回v2
- `CASE 表达式 WHEN v1 THEN r1 [WHEN v2 THEN r2]...[ELSE rn+1] END`
  - case语句，表达式等于某个vn，返回对应then后的结果，都不匹配返回else后的结果

### 系统信息函数

- `VERSION()` 返回MySQL服务器版本字符串（utf8）
- `CONNECTION_ID()` 返回MySQL当前连接数
  - 连接详细信息：使用`SHOW PROCESSLIST`
- `DATABASE() 或 SCHEMA()` 返回默认（当前）数据库名字符串（utf8）
- `USER()`、`CURRENT_USER`、`CURRENT_USER()`、`SYSTEM_USER()`和`SESSION_USER()`
  - 返回当前登录用户的用户名和主机名。格式：`用户名@主机名`
- `LAST_INSERT_ID()` 获取最后生成的AUTO_INCREMENT值
  - 若最后一次同时插入多条数据，则返回这些数据的第一条产生的值

### 加密函数

- `MD5(str)` MD5加密（128比特），返回32位十六进制数字的二进制字符串
- `SHA(str)` 相比MD5更安全
- `SHA2(str, hash_length)` 指定长度进行加密
  - `hash_length`可选值：`224、256、384、512、0`。0等同256
- `AES_ENCODE/AES_DECODE('str', '密钥')` AES 256加密/解密

### 其他函数

- `FORMAT(数值x,n)` 格式化数值x，保留n为小数（四舍五入），返回字符串
- `CONV(数值x, from_base, to_base)` 进制转换，返回字符串
- `INET_ATON('4或8bit ip地址')` 转换ip地址，返回整数
- `INET_NTOA(整数n)` 转换ip地址，返回字符串 
- `GET_LOCK(str,timeout)` 加锁
  - str为锁名，timeout为加锁等待时间（秒），若超时没加锁成功则事件回滚，返回0；加锁成功返回1；发生错误返回NULL
- `RELEASE_LOCK(str)` 解锁
  - 成功返回1，若该线程尚未创建锁，则返回0（此时锁没有被解开）；若命名的锁不存在，则返回NULL
- `IS_FREE_LOCK(str)` 检查锁是否可用
  - 若可用返回1；若正在被使用，返回0；出现错误返回NULL
- `IS_USED_LOCK(str)` 检查锁是否正在被使用
  - 正在被使用返回使用的客户端的connection ID，否则返回NULL
- `BENCHMARK(整数n,表达式)` 执行表达式n次
- `CONVERT(str USING 字符集)` 转换字符集
- `CAST(x, AS type) 或 CONVERT(x, type)` 数据类型转换
  - 可选换的type：BINARY、CHAR(n)、DATE、TIME、DATETIME、DECIMAL、SIGNED、UNSIGNED

## 配置与全局变量

### 配置

- `DELIMITER //` 将MySQL结束符设为`//`，默认结束符为`;`
  - 使用该命令时，应避免使用`\`符

### 全局变量

修改变量：

- 更改：`SET 变量名=值;`
- 更改全局变量：
  - 方法一：`SET GLOBAL 变量名=值;`
    - 临时生效，重启数据库后会从配置文件中读取变量默认值
  - 方法二：`SET PERSIST 变量名=值;`
    - 持久生效，配置保存至数据目录下的mysqld-auto.cnf文件中，重启会读取该文件配置

查看变量：`SHOW VARIABLES LIKE '匹配条件';`



变量：

- `histogram_generation_max_mem_size`：建立直方图时读取至内存的数据上限
- `max_execution_time`：服务器语句超时时间
- `default_password_lifetime`：密码过期时间（NULL表示永不过期）
- `innodb_buffer_pool_size`：v8+，InnoDB的动态缓冲池大小（字节）
- `storage_engine`：默认存储引擎

## 索引

### 简介

索引的特点：

- 索引是对数据库表中一列或多列的值进行排序的一种结构，可提高数据库中特定数据的查询速度
- 索引是在存储引擎中实现的，每种存储引擎的索引都不一定完全相同
  - 所有存储引擎支持每个表至少16个索引，总索引长度至少为256字节
  - MySQL中索引的存储类型有BTREE和HASH两种：MyISAM和InnoDB存储引擎只支持BTREE索引；MEMORY/HEAP存储引擎可以支持HASH和BTREE索引

索引的优点：

- 通过创建唯一索引，保证数据库表中数据的唯一性
- 大大加快数据的查询速度
- 在实现数据的参考完整性方面，可以加速表和表之间的连接
- 使用分组和排序子句进行数据查询时，显著减少查询中分组和排序的时间

索引的缺点：

- 创建索引和维护索引要耗费时间，并且随着数据量的增加所耗费的时间也会增加
- 索引需要占磁盘空间，如果有大量的索引，索引文件可能比数据文件更快达到最大文件尺寸
- 对表中的数据进行增删改时，索引也要动态地维护，降低了数据的维护速度

索引的类型：

- 普通索引：仅用于加快访问速度，允许在定义索引的列中插入重复值和空值
- 唯一索引：索引列的值必须唯一，允许有空值。对于组合索引，列值组合必须唯一
- 单列索引：索引仅包含单个列
- 组合索引：在表的多个字段组合上创建的索引，只有在查询条件中使用了这些字段的左边字段时，索引才会被使用。使用组合索引时遵循最左前缀集合
- 全文索引：在索引列上支持值的全文查找。可以在CHAR、VARCHAR、TEXT类型的列上创建。索引总是对整个列进行，不支持局部（前缀）索引）。仅MyISAM支持全文索引
- 空间索引：对空间数据类型的字段建立的索引（GEOMETRY、POINT、LINESTRING和POLYGON）。创建空间索引的列，必须将其声明为NOT NULL，仅MyISAM支持空间索引

索引设计原则：

- 索引并非越多越好，索引不仅占用磁盘空间，还会影响增删改语句的性能
- 避免对经常更新的表进行过多的索引，并且索引中的列要尽可能少
- 数据量小的表最好不要使用索引，查询花费的时间可能比遍历索引的时间还要短
- 在条件表达式中经常用到的不同值较多的列上建立索引，例如性别只有两种不同值，无需建立索引
- 当唯一性是某种数据本身的特征时，指定唯一索引。确保列数据的完整性，以提高查询速度
- 在频繁进行排序或分组的列上建立索引，如果待排序的列有多个，可以建立组合索引

### 索引相关SQL

创建索引：`[索引类型] INDEX|KEY [索引名] (列名[(索引长度)][ASC|DESC], 列名...) `

- 将上述语句添加至建表语句的表约束处
- 索引类型，可选值如下：
  - 不指定索引类型：普通索引
  - `UNIQUE`：唯一性索引
  - `FULLTEXT`：全文索引
  - `SPATIAL`：空间索引
- 索引名：所缺省索引名，会将列名作为索引名
- 组合索引：指定多个列名时，创建为组合索引
  - “最左前缀”：利用索引中最左边的列集来匹配行，这样的列集称为最左前缀
    - 索引列为`id,name,age`，可搜索`(id)`或`(id,name)`或`(id,name,age)`，不能搜索`(age)`或`(name,age)`
- 索引长度：只有字符串类型的字段可以指定
- 升降序：v8+支持降序索引



为已有表添加索引：

- `ALTER TABLE 表名 ADD [索引类型] INDEX|KEY ...;` 详细格式同上
- `CREATE [索引类型] INDEX [索引名] ON 表名(列名[(索引长度)],...) [ASC|DESC]`
  - 会被映射到ALTER TABLE语句上



删除索引：

- `ALTER TABLE 表名 DROP INDEX 索引名;`
- `DROP INDEX 索引名 ON 表名;`



查看索引信息：

- 查看表中索引信息：`SHOW INDEX FROM 表名[\G];`
  - 描述属性如下：
    - `Table` 创建索引的表
    - `Non_unique` 1代表是非唯一索引，0代表唯一索引
    - `Key_name` 索引名称
    - `Seq_in_index` 该字段在索引中的位置，单列索引该值为1，组合索引为每个字段在索引定义中的顺序
    - `Column_name` 定义索引的字段
    - `Sub_part` 索引长度
    - `Null` 该字段是否能为空值
    - `Index_type` 索引类型
- 查看查询语句的索引信息：`EXPLAIN SELECT ... [\G];`
  - 描述属性如下：
    - `select_type`：所使用的SELECT查询类型，可能的取值有SIMPLE、PRIMARY、UNION、SUBQUERY等
    - `table`：读取的数据表的名字，按被读取的先后顺序排列
    - `type`：本数据表与其他数据表之间的关联关系，可能的取值有system、const、eq_ref、ref、range、index和All
    - `possible_keys`：在搜索数据记录时可选用的各个索引
    - `key`：实际选用的索引
    - `key_len`：索引按字节计算的长度，key_len数值越小，表示越快
    - `ref`：关联关系中另一个数据表里的数据列名
    - `rows`：执行这个查询时预计会从这个数据表里读出的数据行的个数
    - `Extra`：与关联操作有关的信息

### 直方图

> 直方图能近似获得一列的数据分布情况，从而让数据库知道它含有哪些数据
>
> MySQL支持了两种格式：等宽直方图（singleton）和等高直方图（equi-height）
>
> 直方图的共同点是，它们都将数据分到了一系列的buckets中去
>
> MySQL会自动将数据划到不同的buckets中，也会自动决定创建哪种类型的直方图

特点：

- 直方图在创建后不会自动更新，不影响增删改的性能
- 直方图会将所有数据读到内存中操作，根据`histogram_generation_max_mem_size`计算读多少数据

创建：`ANALYZE TABLE 表名 UPDATE HISTOGRAM ON 多个列名 WITH 整数值 BUCKETS;`

- buckets值：默认为100，可选值为1到1024。可先设低一些，不满足需求再增大
  - buckets值相关的因素：这一列的行数，数据分布情况，需要的准确性

删除：`ANALYZE TABLE 表名 DROP HISTOGRAM ON 多个列名;`

## 存储过程和函数

存储过程是一条或者多条SQL语句的集合

### 创建

创建存储过程：`CREATE PROCEDURE 存储过程名 ([参数列表]) [特性1 特...] SQL代码内容;`

- 参数列表：`[IN|OUT|INOUT] 参数名 参数类型` 
  - IN/OUT/INOUT：表示参数仅可输入/仅可输出/可输入也可输出
- 特性：
  - `LANGUAGE SQL`：说明代码是由SQL语句组成的，SQL是该特性的唯一值
  - `[NOT] DETERMINISTIC`：表示结果是否确定。即相同的输入是否得到相同的输出。默认值为NOT DETERMINISTIC
  - `{CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA}`：指明子程序使用SQL语句的限制
    - `CONTAINS SQL`：默认值，子程序包含SQL语句，但是不包含读写数据的语句
    - `NO SQL`：表明子程序不包含SQL语句
    - `READS SQL DATA`：说明子程序包含读数据的语句
    - `MODIFIES SQL DATA`：表明子程序包含写数据的语句
  - `SQL SECURITY {DEFINER | INVOKER}`：指明谁有权限来执行
    - `DEFINER`：默认值，表示只有定义者才能执行
    - `INVOKER`：表示拥有权限的调用者可以执行
  - `COMMENT 'string'`：注释信息，可以用来描述存储过程或函数
- SQL代码内容：可以用`BEGIN...END`表示代码的开始和结束



创建存储函数：`CREATE FUNCTION 函数名 ([参数列表]) [特性1 特...] SQL代码内容;`

- 默认参数为IN参数，无法指定为其他
- `RETURNS 类型 RETURN (值/Select语句);` 指定返回值类型和返回值
  - RETURNS子句只能用于函数中，且函数体必须包含一个返回语句
  - 返回的值不符合类型时，会被强制进行转换



变量：作用域为存储过程和函数的代码块中

- 声明：`DECLARE 变量名列表 类型 [DEFAULT 值];`
  - 不指定默认值则默认值为NULL
- 赋值：`SET 变量名=值[,变量名=值]...;`
- 查询赋值：`SELECT 字段列表 INTO 变量名列表 FROM 表名 WHERE 表达式;`



错误处理：作用域为存储过程和函数的代码块中

- 定义条件：`DECLARE 条件名 CONDITION FOR 条件类型;`
  - 条件类型：`SQLSTATE [VALUE] sqlstate_value` 或 `MySQL_error_code`
    - `sqlstate_value`：长度为5的字符串型错误代码
    - `MySQL_error_code`：数值型错误代码
- 定义处理程序：`DECLARE 错误处理方式 HANDLER FOR 错误类型 代码内容;`
  - 错误处理方式可取参数：
    - `CONTINUE`：不处理错误，继续执行
    - `EXIT`：遇到错误马上退出（仍会执行该行代码内容），未定义处理程序的错误会采取该路径
    - `UNDO`：遇到错误撤回之前的操作，MySQL中不支持该参数
  - 错误类型：指定遇到指定错误后进行处理
    - `SQLSTATE [VALUE] sqlstate_value`：包含5个字符的字符串错误值
    - `MySQL_error_code`：匹配数值类型错误代码
    - `条件名`：DECLARE CONDITION定义的错误条件名
    - `SQLWARNING`：匹配所有以01开头的SQLSTATE错误代码
    - `NOT FOUND`：匹配所有以02开头的SQLSTATE错误代码
    - `SQLEXCEPTION`：匹配所有没有被SQLWARNING或NOT FOUND捕获的SQLSTATE错误代码



光标：用于读取查询结果集中的记录。仅能用于存储过程和函数的代码块中

- 声明光标：`DECLARE 光标名 CURSOR FOR Select语句;`
- 打开光标：`OPEN 光标名;`
- 使用光标：`FETCH 光标名 INTO 变量名列表;` 查询单条记录，将查询得到的信息存入参数中
- 关闭光标：`CLOSE 光标名;`



流程控制：用于存储过程和函数的代码块中

- IF：`IF 条件表达式 THEN 语句块 [ELSEIF 条件表达式 语句块] [ELSE 语句块] END IF;`
- CASE：`CASE 表达式 WHEN 值 THEN 语句块 [WHEN...] [ELSE 语句块] END CASE;`
- LOOP：`[标注:] LOOP 语句块 END LOOP [标注];`
  - 重复执行直至执行LEAVE语句
- LEAVE：`LEAVE 标注;` 退出任何被标注的流程控制构造
- ITERATE：`ITERATE 标注;` 转到语句块的开头，仅可用于LOOP、REPEAT、WHILE语句中
- REPEAT：`[标注:] REPEAT 语句块 UNTIL 条件表达式 END REPEAT [标注];`
- WHILE：`[标注:] WHILE 条件表达式 DO 语句块 END WHILE [标注];`

### 使用

调用存储过程：`CALL 存储过程名([参数列表]);`

调用存储函数：与内部函数的使用方法一样



查看存储过程和函数：

- 方法一：`SHOW PROCEDURE|FUNCTION STATUS [LIKE '匹配条件'][\G];`
- 方法二：`SHOW CREATE PROCEDURE|FUNCTION 存储过程或函数名[\G];`
- 方法三：`SELECT * FROM information_schema.Routines WHERE ROUTINE_NAME='存储过程或函数名'[\G];`
  - 存储过程和函数的信息被存储于information_schema数据库下的Routines表中



修改存储过程和函数：`ALTER PROCEDURE|FUNCTION 存储过程或函数名 [特性1 特...];`

删除存储过程和函数：`DROP PROCEDURE|FUNCTION [IF EXISTS] 存储过程或函数名;`

### 触发器

触发器（trigger）是一个特殊的存储过程，只要当一个预定义的事件发生的时候，就会被自动调用

创建：`CREATE TRIGGER 触发器名 触发时机 触发事件 ON 表名 FOR EACH ROW 语句块;`

- 触发时机：可指定为 `before` 或 `after`
- 触发事件：可指定为 `INSERT`、`UPDATE` 或 `DELETE`
- 语句块中可以使用`NEW.字段名`获取INSERT插入的数据

查看触发器：

- `SHOW TRIGGERS[\G];`

- `SELECT * FROM INFORMATION_SCHEMA.TRIGGERS[\G];`

删除：`DROP TRIGGER [数据库名.]触发器名;`

## 视图

> 视图是一个虚拟表，是从数据库中一个或多个表中导出来的表。视图还可以从已经存在的视图的基础上定义。
>
> 可以像基本表一样对视图进行增删改查。
>
> 视图一经定义便存储在数据库中，没有实际的物理记录，基本表的数据发生的变化会自动反映到视图中。对视图的操作也会反映到基本表中
>
> 视图的优点：简单化，安全性（限制使用权限），逻辑数据独立性

创建：`CREATE [OR REPLACE] [ALGORITHM=算法] VIEW 视图名 [(字段列表)] AS Select语句 [WITH [CASCADED|LOCAL] CHECK OPTION]`

- 加上`OR REPLACE`：替换已创建的视图
- 算法可取值：
  - `UNDEFINED`：自动选择算法
  - `MERGE`：将使用的视图语句与视图定义合并起来，使得视图定义的某一部分取代语句对应的部分
  - `TEMPTABLE`：视图的结果存入临时表，然后用临时表来执行语句
- `[WITH [CASCADED|LOCAL] CHECK OPTION]`：视图更新权限
  - 不指定该子句：更新视图时（增加行，修改行等）不对值做检查
  - CASCADED：默认值，更新视图时要满足所有相关视图和表的条件
  - LOCAL：更新视图时仅需满足该视图本身定义的条件



查看视图信息：

- 方法一：`DESCRIBE|DESC 视图名;`
- 方法二：`SHOW TABLE STATUS LIKE '视图名匹配条件' [\G];`
- 方法三：`SHOW CREATE VIEW 视图名 [\G]`
- 方法四：`SELECT * FROM information_schema.views;`



修改视图：

- 方法一：CREATE OR REPLACE语句
- 方法二：`ALTER [ALGORITHM=算法] ...;`



更新视图：像表一样更新视图，对视图的更新会映射到基本表中，包含以下内容时，无法更新：

1. 视图中不包含基表中被定义为非空的列
2. 在定义视图的SELECT语句后的字段列表中使用了数学表达式或聚合函数
3. 在定义视图的SELECT语句中使用了DISTINCT、UNION、TOP、GROUP BY或HAVING子句



删除视图：`DROP VIEW [IF EXISTS] 视图名列表 [RESTRICT|CASCADE];`

## 权限控制

### 权限表

> 权限表存放于MySQL数据库，用于控制用户对数据库的访问，由MySQL_install_db脚本初始化

**user表**：记录允许连接到服务器的账号信息，权限是全局级的（对所有数据库生效）

- 用户列：`Host` 主机名、`User` 用户名、`authentication_string` 密码
  - Host和User是user表的联合主键
  - 匿名账户：User字段为空字符串，会允许任何人连接到数据库
- 权限列：所有权限列的类型均为ENUM，可取值为`Y`（有权限）和`N`（无权限），默认值为N
- 安全列：有6个字段，两个ssl相关，两个x509相关，两个授权插件相关。
  - ssl用于加密；x509标准可用于标识用户；Plugin字段标识可以用于验证用户身份的插件，如果该字段为空，服务器使用内建授权验证机制验证用户身份
  - 查询服务器是否支持ssl功能：`SHOW VARIABLES LIKE 'have_openssl';`
- 资源控制列：类型为int(11) unsigned，默认值为0
  - `max_questions`：用户每小时允许执行的查询操作次数
  - `max_updates`：用户每小时允许执行的更新操作次数
  - `max_connections`：用户每小时允许执行的连接操作次数
  - `max_user_connections`：用户允许同时建立的连接次数



**db表**：存储了用户对某个数据库的操作权限，决定用户能从哪个主机存取哪个数据库

- 用户列：`Host` 主机名、`User` 用户名、`Db` 
  - 这三个字段构成db表的主键
- 权限列：类型同user表



**tables_priv表**：控制表权限

**columns_priv表**：控制列权限

**procs_priv表**：控制存储过程和存储函数权限

### 访问控制

连接核实阶段：

- 连接MySQL服务器时，使用user表中的Host、User和authentication_string字段执行身份检查
- 只有在user表记录的Host和User字段匹配客户端主机名和用户名并且提供正确的密码时才接受连接
- 接受连接后进入请求核实阶段等待用户请求

请求核实阶段：

- 对在此连接上的每个请求，服务器检查要执行的操作是否有权限来执行。这些权限可以来自user、db、host、tables_priv或columns_priv表
- 确认权限时，MySQL首先检查user表
- 如果指定的权限没有在user表中被授权；MySQL将检查db表，db表是下一安全层级，其中的权限限定于数据库层级，在该层级的SELECT权限允许用户查看指定数据库所有表中的数据
- 如果在该层级没有找到限定的权限，则MySQL继续检查tables_priv表以及columns_priv表
- 如果所有权限表都检查完毕，但还是没有找到允许的权限操作，MySQL将返回错误信息，用户请求的操作不能执行，操作失败。

## 备份

待补充

## 并发

LOCK TABLES为当前线程锁定表。 UNLOCK TABLES释放被当前线程持有的任何锁。当线程发出另外一个LOCK TABLES时，或当服务器的连接被关闭时，当前线程锁定的所有表会自动被解锁。

- 如果一个线程获得在一个表上的一个READ锁，该线程和所有其他线程只能从表中读。
- 如果一个线程获得一个表上的一个WRITE锁，那么只有持锁的线程READ或WRITE表，其他线程被阻止。

### 事务

事务定义：

- 事务是一个最小的不可在分的工作单元；通常一个事务对应一个完整的业务(例如银行账户转账业务，该业务是一个最小的工作单元)
- 一个完整的业务需要批量的DML(insert、update、delete)语句共同联合完成
- 事务只和DML语句有关，或者说DML语句才有事务。这个和业务逻辑有关，业务逻辑不同，DML语句的个数不同



事务的性质：

- 原子性：事务是一个不可分割的工作单位，要么同时成功，要么同时失败
  - 例：当两个人发起转账业务时，如果A转账发起，而B因为一些原因不能成功接受，事务最终将不会提交，则A和B的请求最终不会成功
  - DDL操作回滚日志写入到data dictionary数据字典表mysql.innodb_ddl_log中，用于回滚操作。
- 持久性：一旦事务提交，他对数据库的改变就是永久的
- 隔离性：多个事务之间相互隔离的，互不干扰
- 一致性：事务执行接收之后，数据库完整性不被破坏



事务的分类：

- 隐式事务：没有明显的开启和结束标记，具有自动提交事务的功能；例如update语句修改数据
- 显式事务：具有明显的开启和结束标记
  - 需要首先禁用自动提交事务功能：`SELECT @@autocommit;`，`SET autocommit=0;`
  - 开启事务：`START TRANSACTION;`
  - 提交事务：`COMMIT;`
  - 回滚事务：`ROLLBACK;`



事务并发可能出现的问题：对于两个事务T1和T2

- 更新丢失：T1和T2更新同一行数据，但是T2中途失败退出了，导致两个修改都失效了
- 脏读：T1读取了已经被T2更新但还没有被提交的字段之后，若T2回滚，T1读取的内容就是临时且无效的
- 不可重复读 ：T1读取了一个字段，然后T2更新了该字段之后，T1在读取同一个字段，值就不同了
- 幻读：T1在A表中读取了一个字段，然后T2又在A表中插入了一些新的数据时，T1再读取该表时，就会发现多出几行



设置事务隔离级别：

- 查看：`select @@tx_isolation` ，v8+改用`transaction_isolation`变量
- 设置隔离级别：`set session|global transaction isolation level 隔离级别;`
  - session表示在当前mysql连接中生效，global表示在数据库全局生效



事务隔离级别：

- `read uncommitted`：读未提交数据，允许事务读取未被其他事务提交的变更。可能出现脏读、不可重复读和幻读
- `read committed`：读已提交数据，只允许事务读取已经被其他事务提交的变更。可以避免脏读，可能出现不可重复读和幻读问题
- `repeatable read`：可重复读，确保事务可以多次从一个字段中读取相同的值，在这个事务持续期间，禁止其他事务对这个字段进行更新。（可以避免脏读和不可重复读，但幻读仍然存在）
- `serializable`：序列化，确保事务可以从一个表中读取相同的行，在这个事务持续期间，禁止其他事务对该表执行插入、更新和删除操作，所有并发问题都可避免，但性能十分低下

### 锁定机制

行级锁定

- 锁定对象粒度小，并发能力强，但获取锁和释放锁消耗较大，加锁慢容易死锁

- InnoDB的行级锁：

  > 使用行级锁的条件：关闭autocommit（或手动开启事务，在事务里执行操作），仅在锁定事务规定的时间内使用
  >
  > InnoDB行级锁是通过给索引上的索引项加锁实现的，只有通过索引条件检索数据，InnoDB才使用行级锁，否则，InnoDB将使用表锁
  >
  > 间隙锁：更新某个区间数据时，将会锁定这个区间的所有记录。例，`update ... where id between 1 and 100`，会锁住id从1到100之间所有的记录。即使其中某条记录并不存在，该条记录也会被锁住。这时，如果另外一个Session往这个表中添加一条记录，就必须要等到上一个事务释放锁资源

  - 共享锁（S）：允许一个事务读一行数据时阻止其他的事务读取相同数据的排他锁
  - 排他锁（X）：允许获得排他锁的事务更新数据，阻止其他事务取得相同数据的共享锁和排他锁
  - 意向共享锁（IS）：事务打算给数据行加共享锁。事务在给一个数据行加共享锁前必须先取得该表的IS锁
  - 意向排他锁（IX）：事务打算给数据行加排他锁。事务在给一个数据行加排他锁前必须先取得该表的IX锁。

- 加锁：`SELECT ... LOCK IN SHARE MODE | FOR UPDATE;` 加共享锁/排它锁

  - 跳过锁等待（v8+）：在上述语句的末尾加`NOWAIT | SKIP LOCKED``
    - ``NOWAIT;` 立即报错返回
    - `SKIP LOCKED;`立即返回，结果中不包含被锁定的行

- 解锁：事务提交后锁定解除

- 查看争用情况：`SHOW STATUS LIKE '%innodb_row_lock%';`

- 设置锁等待时间：`innodb_lock_wait_timeout`



表级锁定

- 锁定粒度最大，系统开销小，并发度小，不容易死锁
- MyISAM的表级锁：
  - 读锁：一个线程获得读锁后，该线程和所有其他线程只能从表中读
    - 手动加锁，若写锁已被获取，则等待锁释放再获取
  - 写锁：一个线程获得写锁，只有持锁的线程可读写表，其他线程被阻止
    - 进行增删改操作时，自动尝试获取写锁，若读锁或写锁已被获取，则等待锁释放再获取
- 获取表的锁：`LOCK TABLE 表名 [AS 别名] [READ[LOCAL] | WRITE];`
  - READ：获取读锁；READ LOCAL：允许在锁定时执行非冲突性INSERT
  - WRITE：获取写锁
  - **lock table与事务之间相互影响。因此建议除了在事务中禁用了AUTOCOMMIT, 可以使用lock table之外。其他任何时候都不要显示执行lock table,不管是使用任何存储引擎。**
- 释放锁：`UNLOCK TABLES;`
- 查看表级锁的争用情况：`SHOW STATUS LIKE 'table%'`
  - Table_locks_immediate：产生表级锁定的次数
  - Table_locks_waited：出现比较锁定争用而发生等待的次数



页级锁定

- 锁定对象粒度介于行锁定和表锁定之间，并发能力和资源开销也介于两者之间
- 存在读锁和写锁的概念
  - 对于写锁，如果表没有加锁，就对其表加写锁；如果表已经加了写锁，此时会将写操作的请求放入写锁的队列中。
  - 对于读锁，如果没有加入读锁，那么请求会加入一个读操作的锁，其他读操作的请求会放到读锁的队列中

## 规范

- 表名、字段名使用反引号`` ` ``包裹；注释使用单引号`''`包裹
- SQL语言用全大写，表名字段名等用下划线分隔写法
- 尽量使用短索引