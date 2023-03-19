# SSM框架

## Spring Framework

### 相关概念

控制反转IOC（Inversion of Control）：

- 使用对象时，由主动new产生对象转换为由外部提供对象，此过程中对象创建控制权由程序转移到外部，此思想称为控制反转
- Spring提供了一个容器，称为IOC容器，用来充当IOC思想中的"外部"，负责对象的创建、初始化等一系列工作，其中包含了数据层和业务层的类对象
- IOC容器中存放的对象被称为Bean或Bean对象

依赖注入DI（Dependency Injection）：绑定对象与对象之间的依赖关系，用于为IOC容器和需要使用对象的对象建立联系

依赖坐标：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.10.RELEASE</version>
</dependency>
```

### 控制反转IOC

#### bean的基础配置

在resources目录下创建配置文件 `applicationContext.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="bookDao" name="dao" class="com.spring.dao.impl.BookDaoImpl"/>
    <bean id="bookService" name="service service4" class="..." />

    <bean id="orderDao" class="com.spring.factory.OrderDaoFactory" factory-method="getOrderDao" />
    
    <bean id="userFactory" class="com.itheima.factory.UserDaoFactory"/>
	<bean id="userDao" factory-method="getUserDao" factory-bean="userFactory"/>
    
    <bean id="bookDao" class="..." init-method="init" destroy-method="destory"/>
</beans>
```

bean标签：

- id属性：容器可以通过id值获取对应的bean，id在一个容器中需唯一
- class属性：bean的类型，即配置的bean的全路径类名
- name属性：别名，容器可以通过name值获取对应的bean
  - 可设置多个别名，用逗号、分号或空格分隔。`"service service4"`
- scope属性：bean的作用范围。可选：`singleton` 单例（默认值）、`prototype` 非单例
- factory-method属性：用于工厂实例化bean，属性值为工厂类中创建对象的方法名
  - 若为静态工厂创建，需将class指定为工厂类
- factory-bean属性：用于实例工厂创建bean，指定工厂的实例对象
- init-method/destory-method属性：bean创建之后/销毁之前调用的方法
  - 也可以通过bean添加接口`InitializingBean`， `DisposableBean`并实现接口中的两个方法`afterPropertiesSet`和`destroy`来实现
- lazy-init属性：设置为true表示懒加载，只有获取bean对象时才创建



bean的实例化：

> 注意：spring能访问到bean的私有构造方法，因为使用的是反射

- 构造方法实例化：bean提供一个无参数构造方法供spring调用

- 静态工厂实例化：bean设置factory-method属性，class指定为静态工厂，工厂提供创建对象的方法：

  ```java
  public static OrderDao getOrderDao(){
  	return new OrderDaoImpl();
  }
  ```

- 实例工厂实例化：提供两个bean，一个表示工厂实例，设置id和class属性。另一个表示要创建的实例，设置id、factory-method、factory-bean属性。工厂提供创建对象的方法：

  ```java
  public UserDao getUserDao(){
  	return new UserDaoImpl();
  }
  ```



FactoryBean的使用：用于简化实例工厂的配置

- 创建一个UserDaoFactoryBean的类，实现FactoryBean接口，重写接口的方法

  ```java
  public class UserDaoFactoryBean implements FactoryBean<UserDao> {
      // 代替原始实例工厂中创建对象的方法
      public UserDao getObject() throws Exception {
          return new UserDaoImpl();
      }
      // 返回所创建类的Class对象
      public Class<?> getObjectType() {
          return UserDao.class;
      }
      // 可重写该方法返回false，来指定为非单例模式。若不重写默认为单例模式
      public boolean isSingleton() {
          return false;
      }
  }
  ```

- 配置文件中进行配置：不再需要创建工厂的bean

  ```xml
  <bean id="userDao" class="com.itheima.factory.UserDaoFactoryBean"/>
  ```




#### IOC容器的使用

简易使用：

```java
// 获取IOC容器
ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
// 获取bean
BookDao bookDao = (BookDao) ctx.getBean("bookDaoFactory");
```

- 可能抛出的异常：NoSuchBeanDefinitionException
- ApplicationContext没有close方法，更换为ClassPathXmlApplicationContext可以调用ctx.close()在JVM退出前关闭容器
- 也可以调用ClassPathXmlApplicationContext实例的registerShutdownHook()方法，在JVM退出前关闭容器



容器的创建方式：

- 方式一：基于类路径，推荐

  ```java
  ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
  ```

- 方式二：基于文件系统，不推荐

  ```java
  ApplicationContext ctx = new FileSystemXmlApplicationContext("D:\\workspace\\spring\\spring_10_container\\src\\main\\resources\\applicationContext.xml"); 
  ```



bean的获取方式：

- 方式一：

  ```java
  BookDao bookDao = (BookDao) ctx.getBean("bookDao");
  ```

- 方式二：

  ```java
  BookDao bookDao = ctx.getBean("bookDao", BookDao.class);
  ```

- 方式三：必须保证IOC容器中该类型对应的bean对象只有一个

  ```java
  BookDao bookDao = ctx.getBean(BookDao.class);
  ```



BeanFactory的使用：是IOC的顶层接口

```java
Resource resources = new ClassPathResource("applicationContext.xml");
BeanFactory bf = new XmlBeanFactory(resources);
BookDao bookDao = bf.getBean(BookDao.class);
```

- BeanFactory是延迟加载，只有在获取bean对象的时候才会去创建
- ApplicationContext默认情况下是立即加载，容器加载的时候就会创建bean对象
  - ApplicationContext若要设置延迟加载，需为bean设置`lazy-init="true"`

### 依赖注入DI

#### setter注入

```xml
<bean id="userDao" class="..."/>
<bean id="bookService" class="...">
    <property name="userDao" ref="userDao"/>
</bean>
<bean id="bookDao" class="...">
    <property name="databaseName" value="mysql"/>
    <property name="connectionNum" value="10"/>
</bean>
```

property标签：

- ref属性：引用数据类型注入，ref值使用另一个beanB的id或name，将beanB注入property所在beanA。此时在beanA中无需new创建beanB，而是使用setter方法获取beanB。

  ```java
  // 未使用ref时
  private BookDao bookDao = new BookDaoImpl();
  // 使用ref后
  private BookDao bookDao;
  public void setBookDao(BookDao bookDao) {
  	this.bookDao = bookDao;
  }
  ```

- name属性：上述set方法的方法名和参数名取决于name属性

- value属性：简单数据类型注入，值会在注入时自动转换为set方法的参数类型

#### 构造器注入

```xml
<bean id="bookDao" class="..."/>
<bean id="bookService" class="...">
    <constructor-arg name="bookDao" ref="bookDao"/>
</bean>
```

constructor-arg标签：将值注入bean的构造函数的参数中，ref、name、value属性用法同上

解耦：

- 方法一：name属性改用type属性，按类型注入参数

  ```xml
  <bean id="bookDao" class="...">
      <constructor-arg type="int" value="10"/>
      <constructor-arg type="java.lang.String" value="mysql"/>
  </bean>
  ```

- 方法二：name属性改用index属性，按索引下标注入参数

  ```xml
  <bean id="bookDao" class="...">
      <constructor-arg index="1" value="100"/>
      <constructor-arg index="0" value="mysql"/>
  </bean>
  ```



#### 注入方式的选择

- 强制依赖使用构造器进行，使用setter注入有概率不进行注入导致null对象出现
  - 强制依赖指对象在创建的过程中必须要注入指定的参数
- 可选依赖使用setter注入进行，灵活性强
  - 可选依赖指对象在创建过程中注入的参数可有可无
- Spring框架倡导使用构造器，第三方框架内部大多数采用构造器注入的形式进行数据初始化，相对严谨
- 如果有必要可以两者同时使用，使用构造器注入完成强制依赖的注入，使用setter注入完成可选依赖的注入
- 实际开发过程中还要根据实际情况分析，如果受控对象没有提供setter方法就必须使用构造器注入
- 自己开发的模块推荐使用setter注入



#### 自动装配

> IOC容器根据bean所依赖的资源在容器中自动查找并注入到bean中
>
> 需要注入的类中对应属性的setter方法不能省略，被注入对象必须被IOC容器管理
>
> 自动装配仅用于引用类型注入，且与setter、构造器注入同时出现时，自动配置失效

配置方式：

- 去除property标签，为需要注入的bean添加autowire属性，其取值有：
  - `byType`：按类型装配，通常使用该模式
    - 必须保障容器中想同类型的bean唯一
  - `byName`：按名称装配
    - 必须保障容器中具有指定名称的bean，因变量名与配置耦合，不推荐使用
- 为被注入类设置setter方法，格式同[setter注入](####setter注入)



#### 集合注入

以下以setter注入演示，构造器注入的使用方式也相同：

- list底层通过数组实现，所以list和array标签可以混用
- 若要添加引用类型，将value标签换为ref标签

```xml
<property name="array">
    <array>
        <value>100</value>
        <value>200</value>
        <value>300</value>
    </array>
</property>

<property name="list">
    <list>
        <value>itcast</value>
    </list>
</property>

<property name="set">
    <set>
        <value>itcast</value>
    </set>
</property>

<property name="map">
    <map>
        <entry key="country" value="china"/>
    </map>
</property>

<property name="properties">
    <props>
        <prop key="country">china</prop>
    </props>
</property>
```



### 第三方bean

以druid库为例，将DruidDataSource交由IOC控制：

```xml
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/spring_db"/>
    <property name="username" value="root"/>
    <property name="password" value="1234"/>
</bean>
```

硬编码优化：

- resources目录下创建配置文件`jdbc.properties`：

  ```properties
  jdbc.driver=com.mysql.jdbc.Driver
  jdbc.url=jdbc:mysql://127.0.0.1:3306/spring_db
  jdbc.username=root
  jdbc.password=1234
  ```

- applicationContext.xml中开启context命名空间，并使用`${key}`注入属性：

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:context="http://www.springframework.org/schema/context"
         xsi:schemaLocation="
              http://www.springframework.org/schema/beans
              http://www.springframework.org/schema/beans/spring-beans.xsd
              http://www.springframework.org/schema/context
              http://www.springframework.org/schema/context/spring-context.xsd">
  	<context:property-placeholder location="jdbc.properties"/>
      <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
          <property name="driverClassName" value="${jdbc.driver}"/>
          <property name="url" value="${jdbc.url}"/>
          <property name="username" value="${jdbc.username}"/>
          <property name="password" value="${jdbc.password}"/>
      </bean>
  </beans>
  ```



`<context:property-placeholder/>`标签：

- 注意：context优先加载系统环境变量，若配置文件存在同名变量，会导致值出错
  - 解决方法：为标签添加属性`system-properties-mode="NEVER"`

- 加载多个配置文件：按下列方法设置location属性
  - 方法一：`location="jdbc.properties,jdbc2.properties"`
  - 方法二：`location="*.properties"` 不标准
  - 方法三：`location="classpath:*.properties"` 标准写法
    - `classpath:`：代表从根路径下开始查找，但只能查询当前项目的根路径
  - 方法四：`location="classpath*:*.properties"`
    - 还可以加载当前项目所依赖的所有项目的根路径下的配置文件



### 注解开发

#### 基本使用

注解定义bean：

- 将配置文件的bean节点删除

- 为BookDaoImpl类上添加`@Component("bookDao")`注解，定义bean

- 在配置文件的beans节点下添加包扫描（会同时扫描所有子包中的类，用于发现bean类）

  ```xml
  <context:component-scan base-package="com.spring"/>
  ```

  

纯注解开发：spring3.0+

- 删除配置文件，使用类替代，用`@Configuration`注解标注为配置类，用`@ComponentScan`注解配置包扫描

  ```java
  @Configuration
  @ComponentScan("com.itheima")
  public class SpringConfig {
  }
  ```

- 获取IOC容器时改用

  ```java
  ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
  ```

  - 若要使用close方法，将ApplicationContext改为AnnotationConfigApplicationContext



#### bean注解

- `@Component("bookDao")`：用于类，定义bean

  - 注解的值为该bean的名称，默认值为类名小驼峰

  - 获取bean：

    ```java
    // 按id获取
    BookDao bookDao = (BookDao) ctx.getBean("bookDao");
    // 按类型获取
    BookDao bookDao = ctx.getBean(BookDao.class);
    ```

  - 衍生注解：`@Controller`、`@Service`、`@Repository`

    - 作用与@Component一致，仅用于区分表现层、业务层和数据层

- `@Scope`：用于类，设置该类创建对象的作用范围

  - 可选值有：singleton（单例，默认值），prototype（非单例）

- `@PostConstruct`：用于方法，在创建后执行，替换 init-method

- `@PreDestroy`：用于方法，在销毁前执行，替换 destroy-method

- `@Autowired`：用于成员和setter方法，自动装配注入，用于成员时可以删除setter方法

  ```java
  @Autowired
  private BookDao bookDao;
  ```

  - 默认使用按类型装配，若有多个同类bean，则按照成员变量名与bean的名称匹配

- `@Qualifier`：注入指定名称的bean，必须与@Autowired一起使用，用于变量名也无法匹配的时候

  ```java
  @Autowired
  @Qualifier("bookDao1")
  private BookDao bookDao;
  ```

- `@Value("${配置文件的属性}")`：用于成员，简单类型注入，将属性的值注入变量



#### 配置类注解

```java
@Configuration
@ComponentScan("com.spring")
@PropertySource("jdbc.properties")
@Import({JdbcConfig.class})
public class SpringConfig {
    @Bean
    public DataSource dataSource(){}
}
```

- `@Configuration`：标注为配置类

- `@ComponentScan`：配置包扫描

  - 会扫描包下所有子包中的类，解析注解，发现注解bean类和配置类，发现的配置类自动加载至该配置类

  - value属性：接收包名字符串，或包名字符串数组

  - excludeFilters属性：设置扫描加载bean时，排除的过滤规则

    ```java
    excludeFilters=@ComponentScan.Filter(
        type = FilterType.ANNOTATION,
        classes = Controller.class
    )
    ```

    - type属性：设置排除规则，接收FilterType

      * `ANNOTATION`：按照注解排除，搭配classes
      * `ASSIGNABLE_TYPE`：按照指定的类型过滤
      * `ASPECTJ`：按照Aspectj表达式排除，基本上不会用
      * `REGEX`：按照正则表达式排除
      * `CUSTOM`：按照自定义规则排除

    - classes属性：设置排除的具体注解类，当前设置排除@Controller定义的bean

- `@PropertySource`：加载配置文件，加载后Value可用 `${key}` 获取键值

  - 加载多个：`@PropertySource({"jdbc.properties","xxx"})`

  - 不支持用`*`通配符，可以加上`classpath:`，表示从当前项目的根路径找文件

    ```java
    @PropertySource({"classpath:jdbc.properties"})
    ```

- `@Import`：引入第三方配置类，参数需要一个`class[]`

- `@Bean`：用于方法，表示将返回值作为bean交给spring管理

  - 注解值的含义同@component
  - 该方法可以直接设置形参，容器会自动根据类型装配对象



#### 第三方bean（注解开发）

以druid库为例，在配置类中添加方法，返回要创建的bean对象，并添加`@Bean`注解

```java
public class JdbcConfig {
    @Value("${jdbc.driver}")
    private String driver;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.userName}")
    private String userName;
    @Value("${jdbc.password}")
    private String password;

	@Bean
    public DataSource dataSource(){
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName(driver);
        ds.setUrl(url);
        ds.setUsername(userName);
        ds.setPassword(password);
        return ds;
    }
}
```

- 需在主配置类或该配置类中加载`jdbc.properties`配置文件

### spring整合

#### 整合Mybatis

```xml
<!-- 引入mybatis整合依赖 -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>1.3.0</version>
</dependency>
```

步骤1：在配置类中完成数据源的创建，见上述[druid例子](####第三方bean（注解开发）)

步骤2：创建Mybatis配置类并配置SqlSessionFactory

```java
public class MybatisConfig {
    //定义bean，SqlSessionFactoryBean，用于产生SqlSessionFactory对象
    @Bean
    public SqlSessionFactoryBean sqlSessionFactory(DataSource dataSource){
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        //设置模型类的别名扫描
        factoryBean.setTypeAliasesPackage("com.spring.model.dto");
        //设置数据源
        factoryBean.setDataSource(dataSource);
        return factoryBean;
    }
    //定义bean，返回MapperScannerConfigurer对象
    @Bean
    public MapperScannerConfigurer mapperScannerConfigurer(){
        MapperScannerConfigurer msc = new MapperScannerConfigurer();
        msc.setBasePackage("com.spring.mapper");
        return msc;
    }
}
```

* 封装SqlSessionFactory

  * SqlSessionFactoryBean是FactoryBean的一个子类
  * 当前Spring容器中已经创建了Druid数据源，类型刚好是DataSource类型，自动装配了DruidDataSource对象

* 使用MapperScannerConfigurer加载Dao接口，配置Mapper代理开发

  * 属性basePackage，用来设置所扫描的包路径

  * 获取mapper：

    ```java
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
    EmployeeMapper mapper = ctx.getBean(EmployeeMapper.class);
    ```

    

步骤3：将配置类引入主配置类中

步骤4：编写运行类

```java
public class App {
    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
        AccountService accountService = ctx.getBean(AccountService.class);
        Account ac = accountService.findById(1);
        System.out.println(ac);
    }
}
```



#### 整合Junit

在test\java下创建一个任意名字的类

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {SpringConfiguration.class})
//@ContextConfiguration(locations={"classpath:applicationContext.xml"})
public class AccountServiceTest {
    // 注入bean
    @Autowired
    private AccountService accountService;
    @Test
    public void testFindById(){
        System.out.println(accountService.findById(1));
    }
}
```

* @RunWith：设置类运行器
* @ContextConfiguration：加载Spring环境对应的配置类
  * 加载配置类：`@ContextConfiguration(classes = {xx.class})`
  * 加载配置文件：`@ContextConfiguration(locations={"xx.xml"})`



### AOP

> 连接点(JoinPoint)：程序执行过程中的任意位置，粒度为执行方法、抛出异常、设置变量等，在SpringAOP中，理解为方法的执行
>
> 切入点(Pointcut)：匹配连接点的式子
>
> * 在SpringAOP中，一个切入点可以描述一个具体方法，也可也匹配多个方法
> * 连接点范围要比切入点范围大，是切入点的方法也一定是连接点，但是是连接点的方法就不一定要被增强，所以可能不是切入点
>
> 通知(Advice)：在切入点处执行的操作，也就是共性功能，在SpringAOP中，功能最终以方法的形式呈现
>
> 通知类：定义通知的类
>
> 切面(Aspect)：描述通知与切入点的对应关系

#### 简易实现

- 定义通知类和通知（名称无要求），假设通知为当前系统时间的打印

  ```java
  @Component
  @Aspect
  public class MyAdvice {
      @Pointcut("execution(void com.itheima.dao.BookDao.update())")
      private void pt(){}
      
      @Before("pt()")
      public void method(){
          System.out.println(System.currentTimeMillis());
      }
  }
  ```

  - 切入点依托一个无实际意义的方法进行（无参数，无返回值，无逻辑）
  - @Aspect：定义当前类为切面类
  - @Pointcut：值为切入点表达式
  - @Before：通知方法在切入点方法前运行

- 开启AOP功能：为主配置文件添加注解`@EnableAspectJAutoProxy`



#### 工作流程

1. Spring容器启动：加载通知类和需要增强的类，bean还未创建
2. 读取所有切面配置中的切入点
3. 初始化bean：将要实例化为bean对象的类中的方法与切入点匹配
   - 若匹配成功，表示需要增强，会使用动态代理创建代理对象
   - 若匹配失败，则返回原始对象
4. 执行方法时：
   - 若为代理对象，根据代理对象的运行模式运行原始方法和增强的内容
   - 若为原始对象，直接调用方法



#### 切入点表达式

格式：

```
execution(public User com.itheima.service.UserService.findById(int))
```

* execution：动作关键字，描述切入点的行为动作，例如execution表示执行到指定切入点
* public：访问修饰符，还可以是public，private等，可省略
* User：返回值类型
* com.itheima.service：包名，多级包使用点连接
* UserService：类/接口名称
* findById：方法名
* int：参数，直接写参数的类型，多个类型用逗号隔开
* 异常名：方法定义中抛出指定异常，可以省略



通配符：

- `*`：单个独立的任意符号，可以独立出现，也可以作为前缀或者后缀的匹配符出现

  ```java
  execution(public * com.itheima.*.UserService.find*(*))
  ```

  匹配com.itheima包下的任意包中的UserService类或接口中所有find开头的带有一个参数的方法

- `..`：多个连续的任意符号，可以独立出现，常用于简化包名与参数的书写

  ```java
  execution(public User com..UserService.findById(..))
  ```

  匹配com包下的任意包中的UserService类或接口中所有名称为findById的方法

- `+`：专用于匹配子类类型

  ```java
  execution(* *..*Service+.*(..))
  ```

  匹配所有以Service结尾的接口的子类



书写规范：

- 描述切入点通常描述接口，而不描述实现类，防止紧耦合
- 访问控制修饰符针对接口开发均采用public描述（也可省略）
- 返回值类型对于增删改类使用精准类型加速匹配，对于查询类使用`*`快速描述
- 包名书写尽量不使用`..`匹配，效率低，常用`*`做单个包描述匹配，或精准匹配
- 接口名/类名 书写名称与模块相关的采用`*`匹配，例如`UserService`书写成`*Service`
- 方法名书写以动词进行精准匹配，名词采用`*`匹配，例如`getById`书写成`getBy*`
- 通常不使用异常作为匹配规则



#### 通知

> AOP通知描述了抽取的共性功能，根据共性功能抽取的位置不同，最终运行代码时要将其加入到合理的位置
>
> 通知类也需添加@Component注解，交由Spring管理
>
> 

通知添加的位置：

```java
public int methodName(){
	// 前置通知
    try {
        // 前置通知
        // 需要增强的方法
        // 返回后通知
    } catch (Exception e) {
        // 抛出异常后通知
    }
    // 后置通知
}
```

通知注解的value值：`"切入点方法名()"` 或 `"当前类名.切入点方法名()"` 不推荐使用后者

通知类型：

- 前置通知：`@Before`

- 后置通知：`@After`

- 环绕通知：`@Around`，环绕通知的使用：注意环绕通知方法的返回值需与原始方法一致

  ```java
  @Around("pt()")
  public Object around(ProceedingJoinPoint pjp) throws Throwable{
      // 操作前逻辑, 相当于前置通知
      Object ret = pjp.proceed(); //表示对原始操作的调用
      // 操作后逻辑, 相当于后置通知
      // return xx; // 可对切入点的返回值进行修改
  }
  ```

  - pjp.proceed方法也可传入Object[]作为参数，表示覆盖切入点方法的参数

  - 获取方法信息：

    ```java
    // 获取执行签名信息
    Signature signature = pjp.getSignature();
    // 通过签名获取执行操作名称(接口名)
    String className = signature.getDeclaringTypeName();
    // 通过签名获取执行操作名称(方法名)
    String methodName = signature.getName();
    ```

- 返回后通知：`@AfterReturning`

- 抛出异常后通知：`@AfterThrowing`

  

通知方法获取数据：

- 获取切入点方法参数：

  - 环绕通知：接收参数 `ProceedingJoinPoint pjp`

    ```java
    Object[] args = pjp.getArgs();
    ```

  - 其他通知：通知方法接收参数 `JoinPoint jp`

    ```java
    Object[] args = jp.getArgs();
    ```

- 获取切入点方法返回值：

  - 返回后通知：为注解传入参数 `returning="ret"`，再在通知方法的参数中接收`Object ret`
    - 若有接收 JoinPoint 参数，需将 JoinPoint 放在参数的第一位
  - 环绕通知：接收pjp.proceed方法的返回值

- 获取切入点方法异常信息：

  - 抛出异常后通知：为注解传入参数 `throwing="t"`，再在通知方法的参数中接收`Throwable t`
  - 环绕通知：使用try...catch包裹pjp.proceed方法，捕获异常



#### 事务

为了管理事务，Spring提供了：

- 平台事务管理器接口：`PlatformTransactionManager` ，接口需实现commit方法和rollback方法
- 具体的实现：`DataSourceTransactionManager`，可用于管理JDBC的事务



使用：

- 引入依赖：

  ```xml
  <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>5.3.12</version>
  </dependency>
  ```

- 主配置类添加注解：`@EnableTransactionManagement`

- 配置类添加方法：用于配置事务管理器

  ```java
  @Bean
  public PlatformTransactionManager transactionManager(DataSource dataSource){
      DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();
      transactionManager.setDataSource(dataSource);
      return transactionManager;
  }
  ```

- 为需要被事务管理的方法添加注解：`@Transactional`

  ```java
  @Transactional
  public void transfer(String out,String in ,Double money) {
      accountDao.outMoney(out,money); // 操作1
      accountDao.inMoney(in,money); // 操作2
  }
  ```

  - 原本操作1和2会开启两个事务，添加注解后，该方法会创建一个事务，操作1和2会合并到方法的事务中



相关注解：

- `@EnableTransactionManagement`：开启注解事务支持
- `@Transactional`：
  - 作用范围：建议写在实现类或实现类的方法上
    - 写在接口类上，该接口的所有实现类的所有方法都会有事务
    - 写在接口方法上，该接口的所有实现类的该方法都会有事务
    - 写在实现类上，该类中的所有方法都会有事务
    - 写在实现类方法上，该方法上有事务
  - 注解参数：
    - `readOnly`：true只读事务，false读写事务，增删改要设为false，查询设为true
    - `timeout`：超时时间，单位秒，指定时长内事务没有提交成功就自动回滚，-1表示不设置超时时间
    - `rollbackFor`：传入class数组，当出现指定异常进行事务回滚
      - Spring事务默认只会对Error异常和RuntimeException异常及其子类进行自动回滚，IOException不会自动回滚
    - `rollbackForClassName`：传入类全名字符串数组，作用同上
    - `noRollbackFor`：传入class数组，当出现指定异常不进行事务回滚
    - `noRollbackForClassName`：传入类全名字符串数组，作用同上
    - `isolation`：设置事务的隔离级别，传入Isolation的枚举项：
      * `DEFAULT` 默认、`READ_UNCOMMITTED` 读未提交、`READ_COMMITTED` 读已提交、`REPEATABLE_READ` 重复读取、`SERIALIZABLE` 串行化
    - `propagation`：设置事务的传播行为，传入Propagation的枚举项：
      - `REQUIRED`：默认值，业务方法需要在一个容器里运行。
        - 在事务中运行：加入到这个事务
        - 在事务外运行：新建一个新的事务
      - `REQUIRES_NEW`：不管是否存在事务，该方法总会发起一个新的事务
      - `SUPPORTS`：
        - 在事务中运行：加入到这个事务
        - 在事务外运行：在没有事务的环境下执行
      - `NOT_SUPPORTED`：不管是否存在事务，该方法总在没有事务的环境下执行
      - `MANDATORY`：
        - 在事务中运行：加入到这个事务
        - 在事务外运行：容器抛出Error
      - `NEVER`：
        - 在事务中运行：容器抛出Error
        - 在事务外运行：在没有事务的环境下执行
      - `NESTED`：如果一个活动的事务存在，则运行在一个嵌套的事务中。如果没有活动事务，则按REQUIRED属性执行。它使用了一个单独的事务，这个事务拥有多个可以回滚的保存点。内部事务的回滚不会对外部事务造成影响。它只对DataSourceTransactionManager事务管理器起效。 

### 进阶

#### InitializingBean

```java
public interface InitializingBean {
    void afterPropertiesSet() throws Exception;
}
```

bean实现该接口后，会在spring初始化时调用afterPropertiesSet()方法



## SpringMVC

基于Java实现MVC模型的轻量级Web框架，是对Servlet进行了封装

### 基本概念

后端服务器Servlet拆分成三层，分别是`web`、`service`和`dao`

* web层主要由servlet来处理，负责页面请求和数据的收集以及响应结果给前端
* service层主要负责业务逻辑的处理
* dao层主要负责数据的增删改查操作

servlet处理请求和数据的时候，存在的问题是一个servlet只能处理一个请求

同步调用：采用MVC设计模式，将其设计为`controller`、`view`和`Model`

* controller负责请求和数据的接收，接收后将其转发给service进行业务处理
* service根据需要会调用dao对数据进行增删改查
* dao把数据处理完后将结果交给service,service再交给controller
* controller根据需求组装成Model和View,Model和View组合起来生成页面转发给前端浏览器
* 这样做的好处就是controller可以处理多个请求，并对请求进行分发，执行不同的业务操作

异步调用模式：不再需要view，将json返回至前端，性能比同步调用强



### 基本使用

> 目录结构：com.xx包下
>
> - config目录：存放配置类
> - controller目录：存放SpringMVC的controller类
> - service目录：存放service接口和实现类
> - dao目录：存放dao/Mapper接口

- 导入依赖坐标：

  ```xml
  <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.2.10.RELEASE</version>
  </dependency>
  <!-- 也需要导入servlet和tomcat插件 -->
  ```

- 创建Controller：

  ```java
  @Controller
  public class UserController {
      @RequestMapping("/save")
      @ResponseBody
      public void save(){
          System.out.println("user save ...");
          return "{'info':'springmvc'}";
      }
  }
  ```

- 创建SpringMVC配置类：

  ```java
  @Configuration
  @ComponentScan("com.itheima.controller")
  public class SpringMvcConfig {}
  ```

- 创建servlet配置类：用于替换web.xml

  ```java
  public class ServletContainersInitConfig extends AbstractDispatcherServletInitializer {
      // 加载springmvc配置类
      protected WebApplicationContext createServletApplicationContext() {
          AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
          ctx.register(SpringMvcConfig.class); // 加载指定配置类
          return ctx;
      }
  
      // 设置由springmvc控制器处理的请求映射路径
      protected String[] getServletMappings() {
          return new String[]{"/"};
      }
  
      // 加载spring配置类
      protected WebApplicationContext createRootApplicationContext() {
          AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
          ctx.register(SpringConfig.class);
          return ctx;
      }
  }
  ```

- AbstractDispatcherServletInitializer类是SpringMVC提供的快速初始化Web3.0容器的抽象类
  AbstractDispatcherServletInitializer提供了三个接口方法供用户实现：

  - createServletApplicationContext方法：用来加载SpringMVC环境，创建Servlet容器时，加载SpringMVC对应的bean并放入WebApplicationContext对象范围中，而WebApplicationContext的作用范围为ServletContext范围，即整个web容器范围
  - getServletMappings方法，设定SpringMVC对应的请求映射路径，即SpringMVC拦截哪些请求
  - createRootApplicationContext方法：用来加载Spring环境，如果创建Servlet容器时需要加载非SpringMVC对应的bean,使用当前方法进行，使用方式和createServletApplicationContext相同



### 工作流程

> 层次结构：web容器 → ServletContext → WebApplicationContext → UserController

启动服务器初始化过程：

1. 服务器启动，执行ServletContainersInitConfig类，初始化web容器
2. 执行createServletApplicationContext方法，创建WebApplicationContext对象

3. 加载SpringMvcConfig配置类

4. 执行@ComponentScan加载对应的bean，扫描指定包及其子包下所有类上的注解

5. 加载UserController，每个@RequestMapping的名称对应一个具体的方法

   * 此时就建立了 `/save` 和 save方法的对应关系

6. 执行getServletMappings方法，设定SpringMVC拦截请求的路径规则

   * `/`代表所拦截请求的路径规则，只有被拦截后才能交给SpringMVC来处理请求



单次请求过程：

1. 发送请求`http://localhost/save`
2. web容器发现该请求满足SpringMVC拦截规则，将请求交给SpringMVC处理
3. 解析请求路径/save
4. 由/save匹配执行对应的方法save(）



### 控制类

相关注解：

- `@Controller`：类注解，设定SpringMVC的控制器bean
- `@RequestMapping`：类注解或方法注解，根据value值设置控制器方法请求访问路径
  - 设置在类上时，内部所有@ReqestMapping方法的路径前都会拼接上类设置的路径
  - method属性：指定请求方式，需传入`RequestMethod.PO`
- `@ResponseBody`：类注解或方法注解，设置控制器方法响应内容为返回值，无需解析
  - 设置在类上时，对所有方法生效



### 配置类

SpringMVC和Spring为两个不同的环境

* SpringMVC需加载其相关bean(表现层bean)，也就是controller包下的类
* Spring需加载：业务bean(Service)和功能bean(DataSource等)



加载控制：若Spring的包扫描包含了controller，会报错

- 方法一：将Spring配置类的@ComponentScan改为精准扫描，扫描除controller的包
- [方法二](####配置类注解)：设置Spring配置类的@ComponentScan，排除controller包



简化servlet配置类：

```java
public class ServletConfig extends AbstractAnnotationConfigDispatcherServletInitializer {

    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{SpringConfig.class};
    }

    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{SpringMvcConfig.class};
    }

    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
}
```



SpringMVC配置类相关注解：

- `@EnableWebMvc`：开启SpringMVC的注解支持（包括json支持



静态资源放行：创建配置类，放置在config包下，并在SpringMVC配置类添加包扫描

- pages、js等目录均放置于webapp目录下

```java
@Configuration
public class SpringMvcSupport extends WebMvcConfigurationSupport {
    @Override
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {
        //当访问/pages/????时候，从/pages目录下查找内容
        registry.addResourceHandler("/pages/**").addResourceLocations("/pages/");
        registry.addResourceHandler("/js/**").addResourceLocations("/js/");
        registry.addResourceHandler("/css/**").addResourceLocations("/css/");
        registry.addResourceHandler("/plugins/**").addResourceLocations("/plugins/");
    }
}

@Configuration
@ComponentScan({"com.itheima.controller","com.itheima.config"})
// 或 @ComponentScan("com.itheima")
@EnableWebMvc
public class SpringMvcConfig {}
```



### 请求

#### 接收参数

接收param参数：

- 普通类型：url中的每个参数由方法的同名参数对应接收

  ```java
  // http://.../...?name=itcast&age=15
  
  @RequestMapping("/commonParam")
  @ResponseBody
  public String commonParam(String name,int age){} // 参数名与url中一致
  ```

- POJO类型：方法的参数为POJO类，会将请求参数注入POJO的同名属性（要求必须同名）

- 嵌套POJO类型：上述POJO的某属性为另一个POJO类，按对象层次接收

  ```java
  // ?age=15&address.city=beijing
  public class Address {
      private String city;
      //setter...getter...略
  }
  public class User {
      private int age;
      private Address address;
      //setter...getter...略
  }
  ```

- 数组类型：方法的参数为数组，要求请求参数有多个且与数组同名

- 集合类型：使用方法同上，但list是接口无法创建对象，需用`@RequestParam`注解

  ```java
  public String listParam(@RequestParam List<String> likes){}
  ```

- 日期参数：前SpringMVC支持的字符串转日期的格式为`yyyy/MM/dd`

  - 控制方法的参数添加注解`@DateTimeFormat`，并指定字符串

    ```java
    public String fn(@DateTimeFormat(pattern="yyyy-MM-dd HH:mm:ss") Date date)
    ```




接收请求体参数：

- 引入jackson依赖：

  ```xml
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-databind</artifactId>
  <version>2.9.0</version>
  ```

- SpringMVC配置类添加注解 `@EnableWebMvc`

- 控制方法的参数添加注解 `@RequestBody`

  - 该注解修饰集合或POJO，将json参数注入集合（json数组）或POJO（json对象）



#### 获取session/request

在controller方法中接收一个HttpSession / HttpServletRequest 类型的参数即可

```java
@RequestMapping("/test")
public String test(HttpServletRequest request) {
    HttpSession session = request.getSession();
    // ...
}

@RequestMapping("/test")
public String test(HttpSession session) {
    // ...
}
```



#### 中文乱码

tomcat8.5+已解决该问题

- get请求：tomcat插件的configuration节点下添加：`<uriEncoding>UTF-8</uriEncoding>`

- post请求：servlet配置类中添加过滤器方法

  ```java
  @Override
  protected Filter[] getServletFilters() {
      CharacterEncodingFilter filter = new CharacterEncodingFilter();
      filter.setEncoding("UTF-8");
      return new Filter[]{filter};
  }
  ```



#### 参数注解

`@RequestParam("url中的参数名")`，用于方法参数，将指定参数注入

```java
// http://.../...?name=itcast&age=15
public String commonParam(@RequestParam("name") String userName , int age)
```

- value属性：url中的参数名
- required属性：布尔，是否为必传属性
- defaultValue属性：参数默认值



### 响应

> 同样需要引入jackson依赖

响应页面：page.jso文件置于webapp目录下

```java
@RequestMapping("/toJumpPage")
// 方法需要返回String
public String toJumpPage(){
    return "page.jsp";
}
```

响应文本：

```java
@RequestMapping("/toText")
@ResponseBody
public String toText(){
    return "response text";
}
```

响应JSON：

```java
@RequestMapping("/toJsonPOJO")
@ResponseBody
// 需为配置类开启@EnableWebMvc注解
public Object toJsonPOJO(){
    User user = new User();
    user.setAge(15);
    return user; // 也可以返回集合,会转换为json数组
}
```



### Rest风格

> 与传统风格的区别：
>
> * 传统风格资源描述形式
>   * `http://localhost/user/getById?id=1` 查询id为1的用户信息
>   * `http://localhost/user/saveUser` 保存用户信息
> * REST风格描述形式
>   * `http://localhost/user/1` 
>   * `http://localhost/user`
>
> 优点：隐藏资源的访问行为，无法通过地址得知对资源是何种操作；书写简化
>
> 约定：
>
> * 发送GET请求是用来做查询
> * 发送POST请求是用来做新增
> * 发送PUT请求是用来做修改
> * 发送DELETE请求是用来做删除

获取参数：`http://localhost/user/1` ，[请求体传参](###请求)

- 修改@RequestMapping的value属性，将其中修改为`/users/{id}`，目的是和路径匹配

- 在方法的形参前添加@PathVariable注解

  ```java
  @RequestMapping(value = "/users/{id}",method = RequestMethod.DELETE)
  @ResponseBody
  public String delete(@PathVariable("id") Integer id) {}
  ```

  - @PathVariable注解可以不传值，会自动将变量名与@RequestMapping的value中的变量名进行匹配



#### 简化开发

- 问题1：每个方法的@RequestMapping注解中都定义了访问路径/books，重复性太高。

  ```
  将@RequestMapping提到类上面，用来定义所有方法共同的访问路径。
  ```

- 问题2：每个方法的@RequestMapping注解中都要使用method属性定义请求方式，重复性太高

  ```
  使用@GetMapping  @PostMapping  @PutMapping  @DeleteMapping代替
  ```

- 问题3：每个方法响应json都需要加上@ResponseBody注解，重复性太高

  ```
  1.将ResponseBody提到类上面，让所有的方法都有@ResponseBody的功能
  2.使用@RestController注解替换@Controller与@ResponseBody注解，简化书写
  ```


```java
@RestController //@Controller + ReponseBody
@RequestMapping("/books")
public class BookController {
	//@RequestMapping(method = RequestMethod.POST)
    @PostMapping
    public String save(@RequestBody Book book){}

    //@RequestMapping(value = "/{id}",method = RequestMethod.DELETE)
    @DeleteMapping("/{id}")
    public String delete(@PathVariable Integer id){}

    //@RequestMapping(method = RequestMethod.PUT)
    @PutMapping
    public String update(@RequestBody Book book){}

    //@RequestMapping(value = "/{id}",method = RequestMethod.GET)
    @GetMapping("/{id}")
    public String getById(@PathVariable Integer id){}

    //@RequestMapping(method = RequestMethod.GET)
    @GetMapping
    public String getAll(){}
}
```



### 异常处理器

> 常用异常分类
>
> - 业务异常（BusinessException）
>   - 规范的用户行为产生的异常：用户在页面输入内容的时候未按照指定格式进行数据填写
>
>   - 不规范的用户行为操作产生的异常：如用户故意传递错误数据
>
> - 系统异常（SystemException）
>
>   - 项目运行过程中可预计但无法避免的异常，比如数据库或服务器宕机
> - 其他异常（Exception）
>
>   - 编程人员未预期到的异常，如用到的文件不存在

基本

- 创建异常处理器类，并确保SpringMvc配置类能扫描到该类

  ```java
  @RestControllerAdvice
  public class ProjectExceptionAdvice {
      // 除了自定义的异常处理器，保留对Exception类型的异常处理，用于处理非预期的异常
      @ExceptionHandler(Exception.class)
      public void doException(Exception ex){
          return new Result(...); // 返回结果给前端
      }
  }
  ```

  - @RestControllerAdvice：标识当前类为REST风格对应的异常处理器
    - 自带@ResponseBody注解与@Component注解，具备对应的功能
  - @ExceptionHandler：方法注解，指定异常的处理方案，功能等同于控制器方法



自定义异常：

- 创建自定义异常类：

  ```java
  public class SystemException extends RuntimeException{
      private Integer code;
      public Integer getCode() {
          return code;
      }
      public void setCode(Integer code) {
          this.code = code;
      }
      public SystemException(Integer code, String message) {
          super(message);
          this.code = code;
      }
      public SystemException(Integer code, String message, Throwable cause) {
          super(message, cause);
          this.code = code;
      }
  }
  ```

  - 让自定义异常类继承`RuntimeException`的好处是，后期在抛出这两个异常的时候，就不用在try...catch...或throws了
  - 自定义异常类中添加`code`属性的原因是为了更好的区分异常是来自哪个业务的

- 将其他异常包成自定义异常：

  ```java
  public Book getById(Integer id) {
      // 模拟业务异常，包装成自定义异常
      if(id == 1){
          throw new BusinessException(Code.BUSINESS_ERR,"xx");
      }
      // 模拟系统异常，将可能出现的异常进行包装，转换成自定义异常
      try{
          int i = 1/0;
      }catch (Exception e){
          throw new SystemException(Code.SYSTEM_TIMEOUT_ERR,"服务器访问超时，请重试!",e);
      }
      return bookDao.getById(id);
  }
  ```

- 异常处理器添加方法：

  ```java
  @ExceptionHandler(BusinessException.class)
  public Result doBusinessException(BusinessException ex){
      return new Result(ex.getCode(),null,ex.getMessage());
  }
  ```

  

### 拦截器

> 在指定的方法调用前后执行预先设定的代码，并可以阻止原始方法的执行
>
> 代码顺序：过滤器 → MVC核心 → 拦截器 → 原始方法（处理器） → 拦截器 → 过滤器
>
> 拦截器仅针对SpringMVC的访问进行增强，过滤器对所有动态资源访问进行增强

- 创建拦截器类：实现HandlerInterceptor接口

  ```java
  @Component
  public class ProjectInterceptor implements HandlerInterceptor {
      @Override
      //原始方法调用前执行的内容
      public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
          return true;
      }
  
      @Override
      //原始方法调用后执行的内容
      public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
      }
  
      @Override
      //原始方法调用完成后执行的内容
      public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
      }
  }
  ```

- 配置拦截器类：需被SpringMvc配置类扫描到

  ```java
  @Configuration
  public class SpringMvcSupport extends WebMvcConfigurationSupport {
      @Autowired
      private ProjectInterceptor projectInterceptor;
  
      @Override
      protected void addInterceptors(InterceptorRegistry registry) {
          //配置拦截器
          registry.addInterceptor(projectInterceptor).addPathPatterns("/books");
      }
  }
  ```

  - addPathPatterns可传入多个参数，参数可使用`*`通配符



#### 执行顺序

1. preHandle：返回true则执行handle，返回false则跳过后面方法的执行

   - handler参数：获取方法的相关信息

     ```java
     HandlerMethod hm = (HandlerMethod)handler;
     String methodName = hm.getMethod().getName();//可以获取方法的名称
     ```

2. handle：原始方法

3. postHandle

4. afterCompletion



#### 拦截器链

在配置类的addInterceptors方法中多次调用addInterceptor方法，形成拦截器链

* 拦截器链的运行顺序参照拦截器添加顺序为准
* 当拦截器中出现对原始处理器的拦截（preHandle返回false），后面的拦截器均终止运行
* 当拦截器运行中断，仅运行配置在前面的拦截器的afterCompletion操作



#### 简化开发

SpringMvc配置类实现WebMvcConfigurer接口，无需创建SpringMvcSupport类

```java
@Configuration
@ComponentScan({"com.itheima.controller"})
@EnableWebMvc
//实现WebMvcConfigurer接口可以简化开发，但具有一定的侵入性
public class SpringMvcConfig implements WebMvcConfigurer {
    @Autowired
    private ProjectInterceptor projectInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(projectInterceptor).addPathPatterns("/books","/books/*");
    }
}
```



## SpringBoot基础

用于简化spring开发

### 基本使用

- 创建maven空项目，创建引导类：类名任意

  ```java
  @SpringBootApplication
  public class Application {
      public static void main(String[] args) {
          SpringApplication.run(Application.class, args);
      }
  }
  ```

- pom中指定父工程：

  ```xml
  <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-parent</artifactId>
      <version>2.5.0</version>
  </parent>
  ```

- 注意：springboot打包方式要求jar包

- pom中添加依赖：

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  ```

- 完成上述步骤后，无需配置servlet、无需本地tomcat、无需springmvc配置类，直接运行引导类即可启动服务器（默认端口为8080）



#### 起步依赖

spring-boot-starter-parent：锁定了依赖和插件的版本

spring-boot-starter-web：该依赖引入了spring-web、spring-webmvc 和 tomecat依赖



案例：切换web服务器

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <!-- 排除tomcat依赖 -->
    <exclusions>
        <exclusion>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <groupId>org.springframework.boot</groupId>
        </exclusion>
    </exclusions>
</dependency>
<!-- 引入jetty依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```



### 配置文件

#### 文件类型

在resources目录下创建配置文件，文件名必须是application

* `application.properties`

  ```
  server.port=80
  ```

* `application.yml` / `application.yaml`

  ```yaml
  server:
  	port: 80
  ```

* 配置方式的优先级：properties > yml > yaml



#### 读取配置数据

- 方法一：直接使用 `@Value("${一级属性.二级属性}")` 即可读取

  - 注意：${}实际可用于任何注解；${}会被maven导致在非默认配置文件中无法使用，可采用 `@@一级属性.二级属性` 替代

- 方法二：配置文件的所有数据会被封装到 `Environment` 对象中，使用 `@Autowired` 获取

- 方法三：自定义对象注入，创建一个自定义bean

  ```java
  @Component
  @ConfigurationProperties(prefix = "前缀")
  public class Enterprise {
      private String name;
      // 省略了getter、setter和toString
  }
  ```

  - prefix为可选属性，会根据 `prefix前缀.name` 查找配置数据

  - 若出现configuration anootation processor...警告，添加如下依赖：

    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-configuration-processor</artifactId>
        <optional>true</optional>
    </dependency>
    ```

    

#### 多环境配置

- yml单文件：在`application.yml`中使用`---`来分割不同的配置

  ```yaml
  spring:
    profiles:
      active: dev # 设置启用的环境
  ---
  # 开发环境
  spring:
    profiles: dev # 给开发环境起的名字
  server:
    port: 80
  ```

  - 上述起名的配置已过时，最新起名的配置为：

    ```yaml
    spring:
      config:
        activate:
          on-profile: dev
    ```

- yml多文件：

  - `application.yml`

    ```yaml
    spring:
      profiles:
        active: dev # 设置启用的环境
    ```

  - 新建 `application-dev.yml` ，该配置文件内无需为环境起名

- properties多文件：

  - 新建 `application-dev.yml`

  - `application.properties` 配置文件中设置启用的环境

    ```properties
    spring.profiles.active=dev
    ```



#### 配置覆盖

命令行配置覆盖：

```shell
java –jar xxx.jar –-server.port=88 –-spring.profiles.active=test
```



配置文件优先级：

> 级别数字越大优先级越高，file表示项目根目录，即pom所在目录
>

* 1级：classpath：application.yml
* 2级：classpath：config/application.yml
* 3级：file ：application.yml
* 4级：file ：config/application.yml



#### 配置项

```yml
server:
	port: 8080 # 开放端口
	servlet:
		session:
			timeout: 3600 # session超时秒数，若小于60，以60为准，超过该时间没有访问系统，则session过期
			cookie:
				name: SESSIONID # sessionid在cookie中的名字
```



### 打包

将springboot项目打成jar包，该jar包的运行不依赖tomcat等工具

打包步骤：

- pom配置打包插件

  ```xml
  <build>
      <plugins>
          <plugin>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-maven-plugin</artifactId>
          </plugin>
      </plugins>
      <!-- 排除sql文件 -->
      <resources>
          <resource>
              <directory>src/main/resources</directory>
              <excludes>
                  <exclude>**/*.sql</exclude>
              </excludes>
          </resource>
      </resources>
  </build>
  ```
  
- 运行命令：`mvn package`，会在target目录下生成jar包

- 运行jar包，启动服务器：`java -jar xxx.jar`



### 注解

#### 配置类注解

- `@ConditionalOnProperty`：在配置文件中某属性为指定值时，才加载该类
  - prefix属性：字符串，可选，属性前缀，用于拼接在name属性前
  - name属性：字符串数组，属性名
  - havingValue属性：字符串，属性值



### 整合junit

添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <scope>test</scope>
</dependency>
```



整合junit

* 在测试类上添加 `@SpringBootTest` 、`@RunWith` 注解
* 测试类中使用 `@Autowired` 注入要测试的资源
* 定义测试方法进行测试

```java
@RunWith(SpringRunner.class)
@SpringBootTest
class ApplicationTests {
    @Autowired
    private BookService bookService;

    @Test
    public void save() {
        bookService.save();
    }
}
```

注意：引导类所在包必须是测试类所在包及其子包

- 例如：引导类所在包是 `com.spring`、测试类所在包是 `com.spring`
- 如果不满足这个要求的话，需要为 `@SpringBootTest` 注解传入 `classes` 属性指定引导类的字节码对象
  - 如 `@SpringBootTest(classes = SpringbootApplication.class)`



### 整合mybatis

- 定义实体类、dao接口，dao接口需用 `@Mapper` 修饰

- 配置文件中添加数据库配置：

  ```yml
  spring:
    datasource:
      driver-class-name: com.mysql.jdbc.Driver
      url: jdbc:mysql://localhost:3306/ssm_db?serverTimezone=UTC
      username: root
      password: 1234
      type: com.alibaba.druid.pool.DruidDataSource
  ```
  
  - 时区问题：`SpringBoot`版本低于2.4.3(不含)，Mysql驱动版本大于8.0时，需要在url需传递参数`serverTimezone=UTC`，或在MySQL数据库端配置时区
  - type属性：指定数据源



## SpringBoot进阶

### 参数校验

引入依赖：springboot v2.3+需引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```



为DTO的属性标注注解：为属性添加校验

> 属性校验注解均有message可选属性，值为字符串，标识校验不通过时的提示信息，在字符串中可用`{属性名}`引用注解中的其他属性值

```java
// 空值校验
@NotNull // 不得为null
@NotEmpty // 用于String与集合，不得为null，且长度必须大于0
@NotBlank // 用于String，不能为null，且调用trim()后，长度必须大于0

// 长度校验
@Length(min = 2, max = 10) // 用于String
@Size(...) // 用于对象（Array,Collection,Map,String)

// 大小校验(number)
@Min(value = 1)
@Max(...)
@Range(min = 1, max = 100)
@PositiveOrZero // 值不得小于0

// 大小校验(number或string)
@DecimalMax(value = 1, inclusive = true) // true表示小于等于，为false表示小于
@DecimalMin(...) // 用于number或string

// 数字格式校验
@Digist(integer = 9, fraction = 2) // 分别指定整数部分和小数部分的最大长度

// 字符串格式校验
@Pattern(regexp = "^男|女$") // 必须匹配正则
@Email
@URL
@SafeHtml

// 布尔校验
@AssertTrue // 为true
@AssertFalse // 为false

// 时间
@Past // 时间必须在过去
@Future // 时间必须在未来
```



为controller方法中接收的DTO参数标注注解：

> - `@Valid`：添加后额外接收一个参数BindingResult，并在代码中校验
> - `@Validated`：会自动校验，校验失败时抛出`org.springframework.validation.BindException`

```java
@RestController
public class TestController {
    
    @RequestMapping("/add")
	public String add(@Valid Employee employee, BindingResult bindingResult) {
        if (bindingResult.hasErrors()){
            return bindingResult.getAllErrors().get(0).getDefaultMessage();
        }
    }
    
    @RequestMapping("/add2")
	public String add2(@Validated Employee employee) {}
    
}
```



### 全局Controller处理

```java
@RestControllerAdvice
public class GlobalController {

    //（1）全局数据绑定
    // 应用到所有@RequestMapping注解方法，方法都可以获得此键值对
    @ModelAttribute 
    public void addUser(Model model) {   
        model.addAttribute("键名", "值");  
    }    
    //（2）全局数据预处理
    // 应用到所有@RequestMapping注解方法，在其执行之前初始化数据绑定器WebDataBinder 
    @InitBinder("user")
    public void initBinder(WebDataBinder binder) {
    }    
    
    //（3）全局异常处理
    // 应用到所有@RequestMapping注解的方法，在其抛出Exception异常时执行
    // value属性可以过滤拦截指定异常，此处拦截所有的Exception
    @ExceptionHandler(Exception.class)    
    public String handleException(Exception e) {
        return "error"; // 可以返回字符串或JSON
    }

}
```

`@RestControllerAdvice`：组合注解，由@ControllerAdvice、@ResponseBody组成

`@ControllerAdvice`：继承自@Controller，用来处理Controller全局数据，搭配@ExceptionHandler、@ModelAttribute以及@InitBinder使用

- basePackages属性：字符串数组，字符串为包名，接管指定包及其子包下的Controller
- basePackageClasses属性：Controller类数组，接管这些类所属的包及其子包下的Controller
- assignableTypes属性：Controller类数组，接管这些类
- annotations属性：注解类数组，接管所有被这些注解标记的Controller



### 自定义配置文件属性

> 为springboot配置文件中的自定义配置项生成元数据信息，可以在idea中编辑配置文件时出现提示信息，并可以将对应属性的值加载进配置类中。

引入依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

创建配置类：

```java
@Configuration
@ConfigurationProperties(prefix = "custom") // prefix与value同义
@Setter
@Getter
public class CustomConfiguration {

    /**
     * 最大大小
     */
    private int maxCoreSize;

    private boolean awaitTimeout = false; // 设置默认值

    private Host host;

    @Getter
    @Setter
    public static class Host {

        private String ip;
        
    }
}
```

- 字段的注释也会被写入元数据，并提示
- @ConfigurationProperties的值为配置项前缀，类中的类也可以添加@ConfigurationProperties
- 类中的属性会转为短横分隔形式，如上例中，会提示 `custom.max-core-size`
- 配置文件中对应的值会写入配置类实例的属性中，可通过Getter读取



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
  - 在package节点中设置包名后，在SQL映射文件中的resultType属性可以直接写该包下的类名，如`User`。
  - 但需注意，若类并非直接在该包下，而是在该包内的其他包中，则仍需在type属性中指定全类名。

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
    	update tb_brand set brand_name = #{brandName};
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
  - 方法返回主键值：insert节点设置属性 `useGeneratedKey="true" keyProperty="id"`
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

  

## MyBatisPlus

用于简化mybatis开发、增强mybatis的功能

### 在springboot中使用

- 添加依赖：

  ```xml
  <dependency>
      <groupId>com.baomidou</groupId>
      <artifactId>mybatis-plus-boot-starter</artifactId>
      <version>3.4.1</version>
  </dependency>
  <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid</artifactId>
      <version>1.1.16</version>
  </dependency>
  ```

  - druid数据源也可以不加，SpringBoot有内置的数据源，可以配置成使用Druid数据源

- 配置文件中添加数据库信息

- 定义实体类、Mapper接口和引导类。让Mapper被容器扫描到的两种方法

  - 方案一：在Mapper上添加`@Mapper`注解，并且确保Mapper处在引导类所在包或其子包中

  - 方案二：在引导类上添加`@MapperScan`注解，其属性为所要扫描的Mapper所在包

    ```java
    @MapperScan("com.spring.mapper")
    ```

- 此时会自动实现Mapper，其实现类将作为Bean可被装载。Mapper和SQL映射文件之间的关系仍存在

  - SQL映射文件中查询得到的下划线格式的字段，在通过resultMap转换为PO时，会自动匹配小驼峰格式的属性，所以无需指定列映射，但在resultMap中仍需指定id映射



### 配置文件

applicaiton.yml中添加

```yml
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl # 日志控制台输出
  global-config:
    banner: off # 关闭启动图标
    db-config:
    	id-type: assign_id # 设置全局主键id生成策略
    	table-prefix: tbl_ # 设置表名前缀，会和模型类的类名小写拼接作为表名
    	logic-delete-field: deleted # 逻辑删除字段名
    	logic-delete-value: 1 # 逻辑已删除值
        logic-not-delete-value: 0 # 逻辑未删除值
  mapper-locations: classpath:mapper/**/*.xml # 配置mapper.xml文件路径
  type-aliases-package: com.spring.model.po # 配置类型别名路径
```



### 标准数据层开发

Mapper可以继承BaseMapper类

```java
@Mapper
public interface UserMapper extends BaseMapper<UserPO> {}
```

以获取下列方法：

```java
// 新增
int insert (T t);

// 删除
int deleteById (Serializable id);

// 修改
int updateById(T t);

// 按ID查询
T selectById (Serializable id);

// 查询所有
List<T> selectList(Wrapper<T> queryWrapper);

// 分页查询
IPage<T> selectPage(IPage<T> page, Wrapper<T> queryWrapper);

...
```

- 增删改查的返回值：成功返回1，失败返回0
- serializable类型：String和Number是其子类，能作为主键的数据类型都已经是其子类



### 查询

> 注意：PO中的小驼峰属性在去表中查询时，会被自动转为下划线分隔形式

#### 基本使用

模型类：

```java
@Data
@TableName("user")
public class UserPO {
    private Long id;
    private String name;
    private Integer age;
}
```

mapper：

```java
@Mapper
public interface UserMapper extends BaseMapper<UserPO> {}
```

使用：

```java
QueryWrapper qw = new QueryWrapper();
qw.lt("age",18);
List<UserPO> userList = UserMapper.selectList(qw);
```

- 不再需要mapper.xml，上述代码相当于 

  ```sql
  SELECT id,name,age FROM user WHERE (age < 18);
  ```



#### 条件查询

QueryWrapper：用于构建查询条件

```java
QueryWrapper qw = new QueryWrapper();

// 简单使用，可能出现字段名写错的情况
qw.lt("age",18); // 添加条件: age小于18

// 配合lambda表达式使用
qw.lambda().gt(User::getAge, 10); // 添加条件:age大于10

// 使用LambdaQueryWrapper,简化调用
LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper<User>();
lqw.gt(User::getAge, 10);
```



多条件构建：条件方法支持链式调用

- 逻辑与：多次调用条件方法
- 逻辑或：调用完一个条件方法后，先调用or()方法，再调用下一个条件方法



#### 查询条件方法

```java
// 条件方法中的字段参数为函数式接口(一般用lambda表达式)
// 条件方法可在最前面添加一个布尔参数, 若为true则添加该条件, 为false则不添加
// 所有条件方法均返回LambdaQueryWrapper, 支持链式调用

// 范围查询
eq(字段, 值); // 等于(=)
gt(字段, 值); // 大于(>)
ge(字段, 值); // 大于等于(>=)
lt(字段, 值); // 小于(<)
lte(字段, 值); // 小于等于(<=)

// 模糊查询
like(字段, 值); // 前后加百分号，如 %值%
likeLeft(字段, 值); // 前面加百分号，如 %值
likeRight(字段, 值); // 后面加百分号，如 值%

// 结果排序
orderBy(boolean isAsc, 多个字段); // isAsc: true升序, false降序
orderByAsc(多个字段); // 根据字段升序
orderByDesc(多个字段); // 根据字段降序

// 包含查询
notIn(字段, 集合); // 数据不在集合中
in(字段, 集合); // 数据在集合中

// select方法
select(多个字段); // 仅查询指定字段
select("count(*) as count"); // 聚合函数, SELECT count(*) as count FROM user

// 分组查询
groupBy("tel"); // GROUP BY tel
```



#### 相关注解

- `@TableField`：属性注解，用于模型类的属性
  - value属性：字段映射，值为数据库表中的字段名
  - exist属性：传入布尔，若为false表示该属性在数据库中不存在，不会查询该属性
  - select属性：传入布尔，若为false则不查询该字段，与select()映射配置不冲突
- `@TableName("表名")`：类注解，用于模型类，指定数据库表名



#### 分页查询

配置类添加分页拦截器：

```java
@Configuration
public class MybatisPlusConfig {
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor(){
        MybatisPlusInterceptor mpInterceptor=new MybatisPlusInterceptor();
        mpInterceptor.addInnerInterceptor(new PaginationInnerInterceptor());
        return mpInterceptor;
    }
}
```

分页查询：

```java
IPage<User> page=new Page<>(1,10); // 参数分别为当前页码，和每页记录数
IPage<User> result = userMapper.selectPage(page, queryWrapper); // 执行分页查询

// 获取分页结果(IPage的方法)
long getCurrent(); // 当前页码值
long getSize(); // 每页显示数
long getPages(); // 总页数
long getTotal(); // 总记录数
List<T> getRecords(); // 记录
```



### 增删改

#### 基本使用

添加：

```java
User user = new User();
user.setName("xx");
user.setAge(12);
userDao.insert(user);
```

删除：

```java
userDao.deleteById(2L); // 删除单条记录

//  删除多条数据
List<Long> list = new ArrayList<>();
list.add(1L);
list.add(3L);
userDao.deleteBatchIds(list);
```

修改：

```java
User user = new User();
user.setId(3L);
user.setName("Jock bbb");
userDao.updateById(user);
```



#### 主键生成策略

`@TableId(type = IdType.AUTO)`：修饰模型类的主键属性，指定主键生成策略

- value属性：设置数据库中的主键字段名
- type属性：
  - `AUTO`：使用数据库主键自增，需确保数据库设置了主键自增
  - `NONE`：不设置id生成策略，约等于INPUT
  - `INPUT`：用户手工输入id，需将表的自增策略删除，并给模型类设置id
  - `ASSIGN_ID`：雪花算法生成id（64位整数）(可兼容数值型与字符串型)
    - 基于时间戳、机器id、序列号生成。若修改系统时间可能生成重复主键
  - `ASSIGN_UUID`：以UUID生成算法作为id生成策略，uuid为32字符的字符串
    - 占用空间较大、性能差、不能排序
  - 其他的几个策略均已过时，都将被ASSIGN_ID和ASSIGN_UUID代替掉



#### 逻辑删除

> 物理删除：业务数据从数据库中丢弃，执行的是delete操作
>
> 逻辑删除：为数据设置是否可用状态字段，删除时设置状态字段为不可用状态，数据保留在数据库中，执行的是update操作

模型类添加逻辑删除属性，并使用`@TableLogic`

```java
@TableLogic(value="0",delval="1")
private Integer deleted;
```

- value为正常数据的值，delval为删除数据的值
- 完成后运行删除方法，会自动使用逻辑删除而非物理删除
- 在配置文件中设置相关信息后，无需设置@TableLogic注解的值，但仍需添加该注解



### 服务类简化

原：

```java
public interface UserService{
	public List<User> findAll();
}

@Service
public class UserServiceImpl implements UserService{
    @Autowired
    private UserDao userDao;
    
	public List<User> findAll(){
        return userDao.selectList(null);
    }
}
```

简化：借助IService和ServiceImpl

```java
public interface UserService extends IService<User>{}

@Service
public class UserServiceImpl extends ServiceImpl<UserDao, User> implements UserService{}
```



#### service实现类

> 下列中的泛型T代指创建service时传入的实体类

```java
// 插入数据至数据库，会将自动生成的id回写至实体类中
boolean save(T entiry)
```



### 乐观锁

> mybatis-plus会将version作为条件进行更新，更新时将version加1，防止并发冲突

实现方式：

- 表中添加version列（可设置默认值为1），并在模型类添加属性

  ```java
  @Version
  private Integer version;
  ```

- 配置类中添加乐观锁拦截器

  ```java
  @Configuration
  public class MpConfig {
      @Bean
      public MybatisPlusInterceptor mpInterceptor() {
          // 定义Mp拦截器
          MybatisPlusInterceptor mpInterceptor = new MybatisPlusInterceptor();
          // 添加乐观锁拦截器
          mpInterceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
          return mpInterceptor;
      }
  }
  ```

- 在更新操作时携带version数据

  ```java
  User user = userDao.selectById(3L);
  user.setName("Jock bbb");
  userDao.updateById(user);
  ```

  

### 代码生成器

导入依赖：

```xml
<!--代码生成器-->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.4.1</version>
</dependency>
<!--velocity模板引擎-->
<dependency>
    <groupId>org.apache.velocity</groupId>
    <artifactId>velocity-engine-core</artifactId>
    <version>2.3</version>
</dependency>
```

创建代码生成类：[模板](https://baomidou.com/pages/779a6e)

```java
public class CodeGenerator {
    public static void main(String[] args) {
        // 获取代码生成器的对象
        AutoGenerator autoGenerator = new AutoGenerator();

        //设置数据库相关配置
        DataSourceConfig dataSource = new DataSourceConfig();
        dataSource.setDriverName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/mybatisplus_db?serverTimezone=UTC");
        dataSource.setUsername("root");
        dataSource.setPassword("root");
        autoGenerator.setDataSource(dataSource);

        //设置全局配置
        GlobalConfig globalConfig = new GlobalConfig();
        //设置代码生成位置
		globalConfig.setOutputDir(System.getProperty("user.dir")+"/mp_demo/src/main/java"); 
        globalConfig.setOpen(false); // 设置生成完毕后是否打开生成代码所在的目录
        globalConfig.setAuthor("chen"); // 设置作者
        globalConfig.setFileOverride(true); // 设置是否覆盖原始生成的文件
        globalConfig.setMapperName("%sDao"); // 设置数据层接口名，%s为占位符，指代模块名称
        globalConfig.setIdType(IdType.ASSIGN_ID); // 设置Id生成策略
        autoGenerator.setGlobalConfig(globalConfig);

        //设置包名相关配置
        PackageConfig packageInfo = new PackageConfig();
        packageInfo.setParent("com.aaa"); //设置生成的包名，与代码所在位置不冲突，二者叠加组成完整路径
        packageInfo.setEntity("domain"); //设置实体类包名
        packageInfo.setMapper("dao"); //设置数据层包名
        autoGenerator.setPackageInfo(packageInfo);

        //策略设置
        StrategyConfig strategyConfig = new StrategyConfig();
        strategyConfig.setInclude("tbl_user"); //设置当前参与生成的表名，参数为可变参数
        strategyConfig.setTablePrefix("tbl_"); //设置数据库表的前缀
        strategyConfig.setRestControllerStyle(true); //设置是否启用Rest风格
        strategyConfig.setVersionFieldName("version"); //设置乐观锁字段名
        strategyConfig.setLogicDeleteFieldName("deleted"); //设置逻辑删除字段名
        strategyConfig.setEntityLombokModel(true); //设置是否启用lombok
        autoGenerator.setStrategy(strategyConfig);
        
        // 执行生成操作
        autoGenerator.execute();
    }
}
```

运行代码生成类完成代码生成

- 
