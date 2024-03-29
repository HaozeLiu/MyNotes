# SSM

## Spring

### 核心概念

- **IoC(Inversion of Control) 控制反转**
  - 使用对象时,由主动new产生对象转换为由外部提供对象,此过程中对象创建控制权由程序转移到外部,此思想称为控制反转
- Spring技术对IoC思想进行了实现
  - Spring提供了一个容器,称为IoC容器,用来充当IoC思想中的"外部"
  - IoC容器负责对象的创建,初始化等一系列工作,被创建或被管理的对象在IoC容器中统称为**Bean**
- **DI(Dependency Injection) 依赖注入**
  - 在容器中建立bean与bean之间的依赖关系的整个过程,称之为依赖注入

#### IoC案例

- 配置bean:
  resources里新建applicationContext.xml文件:

```xml
<!--bean标签表示配置bean;id属性表示给bean起名字;class属性写类路径-->
<bean id="bookDao" class="com.SpringStudy.dao.impl.BookDaoImpl"></bean>

<bean id="bookService" class="com.SpringStudy.service.impl.BookServiceImpl"></bean>
```

- 获取bean

```java
public class App{
    public static void main(String[] args){
        //获取Ioc容器
        ApplicationContext ctx = new classPathXmlApplicationContext("applicationContext.xml");
        //获取bean
        BookDao bookDao = (BookDao) ctx.getBean("bookDao");
        BookService bookService == (BookService) ctx.getBean("bookService");
        //然后调用自己写的BookDaoImpl和BookServiceImpl里面的方法
    }
}
```

#### DI案例

1. 基于IoC管理bean
2. Service中使用new形式创建的Dao对象是否保留?(否)
3. Service中需要的Dao对象如何进入到Service中?(提供方法)
4. Service与Dao间的关系如何描述?(配置)

```java
public class BookServiceImpl implements BookService{
    //业务层中不能使用new的方式创建dao对象
    private BookDao bookDao;
    
    //业务方法
    public void save(){
        .....
        bookDao.save();
    }
    //提供对应的set方法
    public void setBookDao(BookDao bookDao){
        this.bookDao = bookDao;
    }
}
```

因为是在service里使用了dao,要配置service与dao的关系.要在applicationContext.xml文件里修改:

```xml
<bean id="bookDao" class="com.SpringStudy.dao.impl.BookDaoImpl"></bean>

<bean id="bookService" class="com.SpringStudy.service.impl.BookServiceImpl">
	<!--配置service与dao的关系 注意下面的name,对应着service里set方法名"setBookDao"里set后面的字,并且首字母大写; ref是对应的bean id-->
    <property name="bookDao" ref="bookDao"/>
</bean>
```



### bean配置

#### bean起别名

再加一个name属性存放别名,多个别名之间可以使用空格,逗号或者分号隔开

```xml
<bean id="bookService" name="service service2 bookEbi" class="com.SpringStudy.service.impl.BookServiceImpl"/>
```

#### bean的作用范围

Spring默认创造的bean是**单例**的.如果想改为多例模式,在bean里修改一个scope属性~注意:scope属性默认为singleton,即单例模式~:

```xml
<bean id="bookDao" name="dao" class="com.SpringStudy.dao.impl.BookDaoImpl" scope="prototype"/>
```

- 为什么bean默认为单例?

  > 如果多例,那么bean用一个造一个,会对容器造成极大的负担.

- 适合交给容器进行管理的bean
  - 表现层对象servlet
  - 业务层对象service
  - 数据层对象dao
  - 工具对象
- 不适合交给容器进行管理的对象
  - 封装实体的域对象(有状态的,记录成员属性值的对象)

### bean实例化

bean本质上就是对象,创建bean使用构造方法完成

#### 1. 构造方法实例化bean(常用)

- 提供可访问的构造方法

  ```java
  public class BookDaoImpl implements BookDao{
      public BookDaoImpl(){
          ...
      }
      public void save(){
          ...
      }
  }
  ```

- 配置

  ```xml
  <bean
        id="bookDao"
        class="com.SpringStudy.dao.impl.BookDaoImpl"
        />
  ```

无参构造方法如果不存在,将抛出异常**BeanCreationException**

> 注意,Spring报错一般从最底下依次往上看

#### 2. 使用静态工厂实例化bean(了解)

- 静态工厂

  ```java
  public class OrderDaoFactory{
      public static OrderDao getOrderDao(){
          return new OrderDaoImpl();
      }
  }
  ```

- 配置

  ```xml
  <bean
        id="orderDao"
        factory-method="getOrderDao"
  		class="com.SpringStudy.factory.OrderDaoFactory"
  />      
  ```

#### 3. 使用实例工厂实例化bean(了解)

- 实例工厂

  ```java
  public class UserDaoFactory{
      public UserDao getUserDao(){
          return new UserDaoImpl();
      }
  }
  ```

- 配置

  ```xml
  <bean id="userDaoFactory" class="com.SpringStudy.factory.UserDaoFactory"/>
  
  <bean
        id="userDao"
        factory-method="getUserDao"
        factory-bean="userDaoFactory"
        />
  ```

由于工厂bean是为了配合使用,实际上没啥意义,而且factory-method方法名不固定,每次都需要配置,所以Spring进行了优化:

#### 4. 使用FactoryBean实例化bean(务必掌握)

- FactoryBean

  ```java
  public class UserDaoFactory implements FactoryBean<UserDao>{
      public UserDao getObject() throws Exception{
          return new UserDaoImpl();
      }
      public Class<?> getObjectType(){
          return UserDao.class;
      }
      //isSingleton可以不写,默认为单例
      public boolean isSingleton(){
          return true;//ture则为单例模式,false则为非单例模式
      }
  }
  ```

- 配置

  ```xml
  <bean
        id="userDao"
        class="com.SpringStudy.factory.UserDaoFactoryBean"
        />
  ```

### bean生命周期

1. 配置文件里配置好初始化和销毁文件

   - 提供生命周期控制方法

     ```java
     public class BookDaoImpl implements BookDao{
         public void save(){
     		...
         }
         
         public void init(){
             ...
         }
         
         public void destory(){
             ...
         }
     }
     ```

   - 配置生命周期控制方法

     ```xml
     <bean id="bookDao" class="com.SpringStudy.dao.impl.BookDaoImpl" init-method="init" destroy-method="destory"/>
     ```

2. 使用接口

   - 实现InitializingBean,DisposableBean接口~这个不用再多写配置文件了~

     ```java
     public class BookServiceImpl implements BookService, InitializingBean, DisposableBean{
         public void save(){
             ...
         }
         
         public void afterPropertiesSet() throws Exception{
     		...
       	}
         
         public void destory() throws Exception{
             ...
         }
     }
     ```

   #### 生命周期流程

   - 初始化容器
     1. 创建对象(内存分配)
     2. 执行构造方法
     3. 执行属性注入(set操作)
     4. 执行bean初始化方法
   - 使用bean
     1. 执行业务操作
   - 关闭/销毁容器
     1. 执行bean销毁方法

#### bean销毁的方法

1. JVM退出之前把容器关闭了(使用close方法)
   要注意ApplicationContext接口没有close方法,得用ClassPathXmlApplicationContext

   ```java
   public class AppForLifeCycle{
       public static void main(String[] args){
           ClassPathXmlApplicationContext ctx = new 	ClassPathXmlApplicationContext("applicationContext.xml");
   		BookDao bookDao = (BookDao) ctx.getBean("bookDao");
   		bookDao.save();//业务方法
   		ctx.close();
       }
   }
   ```

   

2. 设置关闭勾子

   ```java
   public class AppForLifeCycle{
       public static void main(String[] args){
           ClassPathXmlApplicationContext ctx = new 	ClassPathXmlApplicationContext("applicationContext.xml");
   		ctx.registerShutdownHook();//这个,勾子放哪都行
           BookDao bookDao = (BookDao) ctx.getBean("bookDao");
   		bookDao.save();//业务方法
       }
   }
   ```

   相当于告诉虚拟机,关闭虚拟机前把这个容器先关掉



### 依赖注入方式

- 思考: 向一个类中传递数据的方式有几种?
  1. 普通方法(set方法)
  2. 构造方法
- 思考: 依赖注入描述了在容器中建立bean与bean之间依赖关系的过程,如果bean运行需要的是数字或字符串呢?
  1. 引用类型
  2. 简单类型(基本数据类型与String)
- 依赖注入方式
  - setter注入
    - 简单类型
    - 引用类型
  - 构造器注入
    - 简单类型
    - 引用类型

#### setter注入

##### 引用类型

- 在bean中定义引用类型属性并提供可访问的**set**方法

  ```java
  public class BookServiceImpl implements BookService{
      private BookDao bookDao;
      public void setBookDao(BookDao bookDao){
          this.bookDao = bookDao;
      }
  }
  ```

- 配置中使用**property**标签**ref**属性注入引用类型对象

  ```xml
  <bean id="bookService" class="com.SpringStudy.service.impl.BookServiceImpl">
  	<property name="bookDao" ref="bookDao"/>
  </bean>
  <bean id="bookDao" class="com.SpringStudy.dao.impl.BookDaoImpl"/>
  ```

##### 简单类型

- 在bean中定义引用类型属性并提供可访问的**set**方法

  ```java
  public class BookDaoImpl implements BookDao{
      private int connectionNumber;
      public void setConnectionNumber(int connectionNumber){
          this.connectionNumber = connectionNumber;
      }
  }
  ```

- 配置中使用**property**标签**value**属性注入简单类型数据

  ```xml
  <bean id="bookDao" class="com.SpringStudy.dao.impl.BookDaoImpl">
  	<property name="connectionNumber" value="10"/>
  </bean>
  ```

  

#### 构造器注入

跟setter注入大同小异

##### 引用类型

- bean中定义引用类型属性并提供可访问的**构造**方法

  ```java
  public class BookServiceImpl implements BookService{
      private BookDao bookDao;
      public BookDao(BookDao bookDao){
          this.bookDao = bookDao;
      }
  }
  ```

- 配置中使用**constructor-arg**标签**ref**属性注入引用类型对象

  ```xml
  <bean id="bookService" class="com.SpringStudy.service.impl.BookServiceImpl">
  	<constructor-arg name="bookDao" ref="bookDao"/>
  </bean>
  <bean id="bookDao" class="com.SpringStudy.dao.impl.BookDaoImpl"/>
  ```

  

##### 简单类型

- 在bean中定义引用类型属性并提供可访问的**set**方法

  ````java
  public class BookDaoImpl implements BookDao{
      private int connectionNumber;
      public void setConnectionNumber(int connectionNumber){
          this.connectionNumber = connectionNumber;
      }
  }
  ````

- 配置中使用**constructor-arg**标签**value**属性注入简单类型对象

  ```xml
  <bean id="bookDao" class="com.SpringStudy.dao.impl.BookDaoImpl">
  	<constructor-arg name="connectionNumber" value="10"/>
  </bean>
  ```

注意,构造器传参配置中constructor-arg的name还必须要跟构造方法的**形参**名字匹配,**耦合度太高**,所以想到下面的改善方法:

##### 构造器注入--参数适配

- 配置中使用**constructor-arg**标签**type**属性设置按**形参类型**注入

  ```xml
  <bean id="bookDao" class="com.SpringStudy.dao.impl.BookDaoImpl">
  	<constructor-arg type="int" value="10"/>
      <constructor-arg type="java.lang.String" value="mysql"/>
  </bean>
  ```

- 配置中使用**constructor-arg**标签**index**属性设置按**形参位置**注入

  ```xml
  <bean id="bookDao" class="com.SpringStudy.dao.impl.BookDaoImpl">
  	<constructor-arg index="0" value="10"/>
      <constructor-arg inrdex="1" value="mysql"/>
  </bean>
  ```

  

##### 依赖注入方式选择

1. 强制依赖~就是bean运行必须要有的东西~使用构造器运行,使用setter注入有概率不进行注入导致null对象出现
2. 可选依赖使用setter注入进行,灵活性比较强
3. Spring框架倡导使用构造器,第三方框架内部大多数采用构造器注入的形式进行数据初始化,相对严谨
4. 如果有必要可以两者同时使用,使用构造器注入完成强制依赖的注入,使用setter注入完成可选依赖的注入
5. 实际开发过程中还要根据实际情况分析,如果受控对象没有提供setter方法就要使用构造器注入
6. **自己开发的模块推荐使用setter注入**



#### 依赖自动装配

IoC容器根据bean所依赖的资源在容器中自动查找并注入到bean中的过程称为自动装配

自动装配的方式:

- **按类型(常用)**:`byType`
- 按名称:`byName`
- 按构造方法:`constructor`(不推荐)
- 不启用自动装配:`no`

配置中使用**bean**标签**autowire**属性设置自动装配的类型

```xml
<bean id="bookDao" class="com.SpringStudy.dao.impl.BookDaoImpl"/>

<bean id="bookService" class="com.SpringStudy.service.impl.BookServiceImpl" autowire="byType"/>
```

##### 依赖自动装配特征用类型依赖注入,不能对简单类型进行操作

- 使用按类型配对(byType)时,必须保障容器中相同类型的bean唯一,推荐使用
- 使用按名称配对(byName)时,必须保障容器中具有指定名称的bean,这种方法与配置耦合,不推荐使用
- 自动装配优先级低于setter注入与构造器注入,同时出现时自动装配配置失效



### 集合注入

- 注入**数组**对象

  ```xml
  <property name="array">
  	<array>
      	<value>100</value>
          <value>200</value>
          <value>300</value>
      </array>
  </property>
  ```

- 注入**List**对象

  ```xml
  <property name="list">
  	<list>
      	<value>one</value>
          <value>two</value>
          <value>three</value>
      </list>
  </property>
  ```

- 注入**Set**对象

  ```xml
  <property name="set">
  	<set>
      	<value>first</value>
          <value>second</value>
          <value>third</value>
      </set>
  </property>
  ```

- 注入**Map**对象

  ~~~xml
  <property name="map">
  	<map>
      	<entry key="country" value="china"/>
          <entry key="province" value="shandong"/>
          <entry key="city" value="dezhou"/>
      </map>
  </property>
  ~~~

- 注入**Properties**对象

  ```xml
  <property name="properties">
  	<props>
      	<prop key="country">china></prop>
          <prop key="province">shandong</prop>
          <prop key="city">dezhou</prop>
      </props>
  </property>
  ```

如果不是简单对象,就不用value等等,而是用`<ref bean="beanId"/>`



### 加载properties文件

- 开启context命名空间

  ````xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans 	         xmlns="http://www.springframework.org/schema/beans"
  	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:comtext="http://www.springframework.org/schema/context"
         xsi:schemaLocation="
              http://www.springframework.org/schema/beans
              http://www.springframework.org/schema/beans/spring-beans.xsd
              http://www.springframework.org/schema/context
              http://www.springframework.org/schema/context/spring-context.xsd">
  </beans>
  ````

- 使用context命名空间,加载指定properties文件
  `<context:property-placeholder location="jdbc.properrties"/>`

- 使用`${}`读取加载的属性值
  `<property name="username" value="${jdbc.username}"/>`



注意:

- 不加载系统属性
  `<context:property-placeholder location="jdbc.properrties" system-properties-mode="NEVER"/>`

- 加载多个properties文件
  `<context:property-placeholder location="jdbc.properrties,msg.properties"/>`

- 加载所有properties文件
  `<context:property-placeholder location="*.properrties"/>`

- 加载properties文件**标准格式**

  `<context:property-placeholder location="classpath:*.properties"/>`

- 从类路径或jar包中搜索并加载properties文件

  `<context:property-placeholder location="classpath*:*.properrties"/>`



### 容器

#### 创建容器

- 方式一:类路径加载配置文件
  `ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");`
- 方式二:文件路径加载配置文件
  `ApplicationContext ctx = new FileSystemXmlApplicationContext("D:\\applicationContext.xml");`
- 加载多个配置文件
  `ApplicationContext ctx = new ClassPathXmlApplicationContext("bean1.xml", "bean2.xml");`

#### 获取bean

- 方式一:使用bean名称获取
  `BookDao bookDao = (BookDao) ctx.getBean("bookDao");`

- 方式二:使用bean名称获取并指定类型
  `BookDao bookDao = ctx.getBean("bookDao", BookDao.class);`

- 方式三:使用bean类型获取

  `BookDao bookDao = ctx.getBean(BookDao.class);`


#### 容器类层次结构图:

![img](file:///C:/Users/86139/AppData/Local/Packages/Microsoft.Windows.Photos_8wekyb3d8bbwe/TempState/ShareServiceTempFolder/屏幕截图(34).jpeg)





### 注解开发

#### 注解开发定义bean

- 使用@Component定义bean

  ```java
  @Component("bookDao")
  public class BookDaoImpl implements BookDao {
  }
  @Component//不起名的话获取bean时就要按类型获取
  public class BookServiceImpl implements BookService {
  }
  ```

- 核心配置文件中通过组件扫描加载bean

  ```xml
  <context:component-scan base-package="org.springStudy"/>
  ```

- Spring提供`@Component`注解的三个衍生注解

  - `@Controller`:用于表现层bean定义
  - `@Service`:用于业务层bean定义
  - `@Repository`:用于数据层bean定义

  ```java
  @Repository("bookDao")
  public class BookDaoImpl implements BookDao {
  }
  @Service
  public class BookServiceImpl implements BookService {
  }
  ```

  

#### 纯注解开发

- Spring3.0开启了纯注解开发模式,使用Java类替代配置文件

- java类替代Spring核心配置文件

  ```java
  @Configuration
  @ComponentScan("com.SpringStudy")
  public class SpringConfig{
  }
  ```

  > `@Configuration`注解用于设定当前类为配置类
  >
  > `@ComponentScan`注解用于设定扫描路径,此注解只能添加一次,多个数据请用数组格式:
  > `@ComponentScan({"com.SpringStudy.service","com.SpringStudy.dao})`

- 读取Spring核心配置文件初始化容器对象切换为读取Java配置类初始化容器对象

  ```java
  //加载配置文件初始化容器
  Application ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
  //加载配置类初始化容器
  Application ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
  ```



#### bean作用范围与生命周期

- 使用`@Scope`定义bean作用范围

  ```java
  @Repository
  @Scope("singleton")
  public class BookDaoImpl implement BookDao {  
  }
  ```

- 使用`@PostConstruct`,`@PreDestroy`定义bean生命周期

  ```java
  @Repository
  @Scope("singleton")
  public class BookDaoImpl implement BookDao { 
      public BookDaoImpl(){
          ...
      }
      
      @PostConstruct
  	public void init(){
          ...
      }
      
      @PreDestroy
      public void destroy(){
          ...
      }
  }
  ```

  

#### 依赖注入

- 使用`@Autowired`注解开启自动装配模式

  ```java
  @Service
  public class BookServiceImpl implements BookService{
      @Autowired
      private BookDao bookDao;
      
      public void save(){
          bookDao.save();
      }
  }
  ```

  > - 自动装配基于反射设计创建对象并暴力反射对应属性为私有属性初始化数据,因此无需提供setter方法
  > - 自动装配建议使用无参构造方法(默认),如果不提供对应构造方法,请提供唯一的构造方法

- 使用`@Qualifier`注解开启指定名称装配bean

  ```java
  @Service
  public class BookServiceImpl implements BookService{
      @Autowired
      @Qualifier("bookDao")
      private BookDao bookDao;
  }
  ```

  > 注意,`@Qualifier`注解无法单独使用,必须配合`@Autowired`注解使用

- 使用`@Value`实现简单类型注入

  ```java
  @Repository
  
  public class BookDaoImpl implement BookDao {  
      @Value("100")
      private String connectionNum;
  }
  ```

- 使用`@PropertySource`注解加载properties文件

  ```java
  @Configuration
  @ComponentScan("com.SpringStudy")
  @PropertySource("classpath:jdbc.properties")
  public class SpringConfig{
      
  }
  ```

  > 路径仅支持单一文件配置,多文件需使用数组格式配置,不允许使用通配符*

#### 第三方bean管理

- 使用@bean配置第三方bean

  ```java
  @Configuration
  public class JdbcConfig{
      @Bean
      public DataSource dataSource(){
          DruidDataSource ds = new DruidDataSource();
          ds.setDriverClassName("com.mysql.jdbc.Driver")
              ds.setUrl("jdbc:mysql://localhost:3306/spring_db");
          ds.setUsername("root");
          ds.setPassword("root");
          return ds;
      }
  }
  ```

1. 使用独立的配置类管理第三方bean

   ```java
   public class JdbcConfig{
       @Bean
       public DataSource dataSource(){
           DruidDataSource ds = new DruidDataSource();
           ds.setDriverClassName("com.mysql.jdbc.Driver")
               ds.setUrl("jdbc:mysql://localhost:3306/spring_db");
           ds.setUsername("root");
           ds.setPassword("root");
           return ds;
       }
   }
   ```

2. 将独立的配置类加入核心配置

   - 方式1:导入式(使用`@import`注解手动加入配置类到核心配置,此注解只能添加一次,多个数据请使用数组格式)
     **第三方配置类不用加`@Configuration`注解**

     ```java
     @Configuration
     @Import(JdbcConfiig.class)
     public class SpringConfig{
     }
     ```

   - 方式2:扫描式 **第三方配置类需要加`@Configuration`注解**

     使用`@ComponentScan`注解扫描配置类所在的包,加载对应的配置类信息

     ```java
     @Configuration
     @ComponentScan({"com.SpringStudy.config","com.SpringStudy.service","com.SpringStudy.dao"})
     public class SpringConfig{
     }
     ```



#### 第三方bean依赖注入

- 简单类型依赖注入

  ```java
  public class JdbcConfig{
      @Value("com.mysql.jdbc.Driver")
      private String driver;
      @Value("jdbc:mysql://localhost:3306/spring_db")
      private String url;
      @Value("root")
      private String userName;
      @Value("root")
      private String password;
      
      @Bean
      public DataSource dataSource(){
          DruidDataSource ds = new DruidDataSource();
          ds.setDriverClassName(driver)
          ds.setUrl(url);
          ds.setUsername(userName);
          ds.setPassword(password);
          return ds;
      }
  }
  ```

- 引用类型依赖注入

  ```java
  @Bean
      public DataSource dataSource(BookService bookService){
          System.out.println(bookService);
          DruidDataSource ds = new DruidDataSource();
          //属性设置
          return ds;
      }
  ```

  > 引用类型注入只需要为bean定义方法设置形参即可,容器会根据类型自动装配对象



### AOP

AOP(Aspect Oriented Programming)面向切面编程，一种编程范式，指导开发者如何组织程序结构。

* OOP(Object Oriented Programming)面向对象编程

AOP是在不改原有代码的前提下对其进行增强。

​              

#### 核心概念

准备一个环境,先看最主要的类`BookDaoImpl`:

```java
@Repository
public class BookDaoImpl implements BookDao {
    public void save() {
        //记录程序当前执行执行（开始时间）
        Long startTime = System.currentTimeMillis();
        //业务执行万次
        for (int i = 0;i<10000;i++) {
            System.out.println("book dao save ...");
        }
        //记录程序当前执行时间（结束时间）
        Long endTime = System.currentTimeMillis();
        //计算时间差
        Long totalTime = endTime-startTime;
        //输出信息
        System.out.println("执行万次消耗时间：" + totalTime + "ms");
    }
    public void update(){
        System.out.println("book dao update ...");
    }
    public void delete(){
        System.out.println("book dao delete ...");
    }
    public void select(){
        System.out.println("book dao select ...");
    }
}
```

对于`save`方法中有计算万次执行消耗的时间。

目前只有save方法有计算时间的功能,我们想给delete,update和select方法都加上这个功能,同时不惊动(改动)原有设计(代码),这个也就是Spring的理念：

==无入侵式/无侵入式==

1. 前面一直在强调，Spring的AOP是对一个类的方法在不进行任何修改的前提下实现增强。对于上面的案例中BookServiceImpl中有`save`,`update`,`delete`和`select`方法,这些方法我们给起了一个名字叫==连接点==

2. 对于需要增强的方法(update,delete...)我们给起了一个名字叫==切入点==

3. 我们将要增强的功能单独抽出来:

   ```java
   public void method(){
       Long startTime = System.currentTimeMills();
       for(int i = 0; i < 10000; i++){
           //调用原始操作
       }
       Long endTime = Sytem.currentTimeMills();
       Long totalTime = endTime - startTime;
       System.out.println("万次耗时:" + tatalTime + "ms");
   }
   ```

   给他起名叫==通知==

4. 通知是要增强的内容，会有多个，切入点是需要被增强的方法，也会有多个，那哪个切入点需要添加哪个通知，就需要提前将它们之间的关系描述清楚，那么对于通知和切入点之间的关系描述，我们给起了个名字叫==切面==

5. 通知是一个方法，方法不能独立存在需要被写在一个类中，这个类我们也给起了个名字叫==通知类==

#### 实现步骤

1. 添加依赖
   pom.xml

   ```xml
   <dependency>
       <groupId>org.aspectj</groupId>
       <artifactId>aspectjweaver</artifactId>
       <version>1.9.4</version>
   </dependency>
   ```

   - 因为`spring-context`中已经导入了`spring-aop`,所以不需要再单独导入`spring-aop`
   - 导入AspectJ的jar包,AspectJ是AOP思想的一个具体实现，Spring有自己的AOP实现，但是相比于AspectJ来说比较麻烦，所以我们直接采用Spring整合ApsectJ的方式进行AOP开发。

2. 定义接口与实现类

   就是我们上面的BookDaoImpl

3. 定义通知类和通知

   通知就是将共性功能抽取出来后形成的方法，我们把例子简化一下,不计算运行时间,只打印当前系统时间:

   ```java
   public class MyAdvice {
       public void method(){
           System.out.println(System.currentTimeMillis());
       }
   }
   ```

   类名和方法名没有要求，可以任意。

4. 定义切入点

   BookDaoImpl中有两个方法(我们简化一下,把select之类的都删了,只保留save和update)，分别是save和update，我们要增强的是update方法，该如何定义呢?

   ```java
   public class MyAdvice {
       @Pointcut("execution(void org.spring.dao.BookDao.update())")
       private void pt(){}
       public void method(){
           System.out.println(System.currentTimeMillis());
       }
   }
   ```

5. 制作切面

   切面是用来描述通知和切入点之间的关系，如何进行关系的绑定?

   ```java
   public class MyAdvice {
       @Pointcut("execution(void org.spring.dao.BookDao.update())")
       private void pt(){}
       
       @Before("pt()")
       public void method(){
           System.out.println(System.currentTimeMillis());
       }
   }
   ```

   绑定切入点与通知关系，并指定通知添加到原始连接点的具体执行==位置==
   **说明:**@Before翻译过来是之前，也就是说通知会在切入点方法执行之前执行，除此之前还有其他四种类型，后面会讲。

6. 将通知类配给容器并标识其为切面类

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

7. 开启注解格式AOP功能

   ```java
   @Configuration
   @ComponentScan("com.itheima")
   @EnableAspectJAutoProxy
   public class SpringConfig {
   }
   ```

8. 运行程序

> 知识点:
>
> 1:@EnableAspectJAutoProxy  
>
> | 名称 | @EnableAspectJAutoProxy |
> | ---- | ----------------------- |
> | 类型 | 配置类注解              |
> | 位置 | 配置类定义上方          |
> | 作用 | 开启注解格式AOP功能     |
>
> 2: @Aspect
>
> | 名称 | @Aspect               |
> | ---- | --------------------- |
> | 类型 | 类注解                |
> | 位置 | 切面类定义上方        |
> | 作用 | 设置当前类为AOP切面类 |
>
> 3: @Pointcut   
>
> | 名称 | @Pointcut                   |
> | ---- | --------------------------- |
> | 类型 | 方法注解                    |
> | 位置 | 切入点方法定义上方          |
> | 作用 | 设置切入点方法              |
> | 属性 | value（默认）：切入点表达式 |
>
> 4: @Before
>
> | 名称 | @Before                                                      |
> | ---- | ------------------------------------------------------------ |
> | 类型 | 方法注解                                                     |
> | 位置 | 通知方法定义上方                                             |
> | 作用 | 设置当前通知方法与切入点之间的绑定关系，当前通知方法在原始切入点方法前运行 |

#### AOP切入点表达式

首先我们先要明确两个概念:

* 切入点:要进行增强的方法
* 切入点表达式:要进行增强的方法的描述方式

##### 语法格式

对于切入点的描述，我们其实是有两种方式的:

描述方式一：执行com.itheima.dao包下的BookDao接口中的无参数update方法

```java
execution(void com.itheima.dao.BookDao.update())
```

描述方式二：执行com.itheima.dao.impl包下的BookDaoImpl类中的无参数update方法

```
execution(void com.itheima.dao.impl.BookDaoImpl.update())
```

因为调用接口方法的时候最终运行的还是其实现类的方法，所以上面两种描述方式都是可以的。

对于切入点表达式的语法为:

* 切入点表达式标准格式：动作关键字(访问修饰符  返回值  包名.类/接口名.方法名(参数) 异常名）

对于这个格式，我们不需要硬记，通过一个例子，理解它:

```
execution(public User com.itheima.service.UserService.findById(int))
```

* execution：动作关键字，描述切入点的行为动作，例如execution表示执行到指定切入点
* public:访问修饰符,还可以是public，private等，可以省略
* User：返回值，写返回值类型
* com.itheima.service：包名，多级包使用点连接
* UserService:类/接口名称
* findById：方法名
* int:参数，直接写参数的类型，多个类型用逗号隔开
* 异常名：方法定义中抛出指定异常，可以省略

切入点表达式就是要找到需要增强的方法，所以它就是对一个具体方法的描述，但是方法的定义会有很多，所以如果每一个方法对应一个切入点表达式，想想这块就会觉得将来编写起来会比较麻烦，有没有更简单的方式呢?

就需要用到下面所学习的通配符。

##### 通配符

我们使用通配符描述切入点，主要的目的就是简化之前的配置，具体都有哪些通配符可以使用?

* `*`:单个独立的任意符号，可以独立出现，也可以作为前缀或者后缀的匹配符出现

  ```
  execution（public * com.itheima.*.UserService.find*(*))
  ```

  匹配com.itheima包下的任意包中的UserService类或接口中所有find开头的带有一个参数的方法

* `..`：多个连续的任意符号，可以独立出现，常用于简化包名与参数的书写

  ```
  execution（public User com..UserService.findById(..))
  ```

  匹配com包下的任意包中的UserService类或接口中所有名称为findById的方法

* `+`：专用于匹配子类类型

  ```
  execution(* *..*Service+.*(..))
  ```

  这个使用率较低，描述子类的，咱们做JavaEE开发，继承机会就一次，使用都很慎重，所以很少用它。*Service+，表示所有以Service结尾的接口的子类。

接下来,用上面的案例分析一下:

```java
package com.spring.dao
public interface BookDao{
    public void save();
    public void update();
}
```

```java
package com.spring.dao.impl
@Repository
public class BookDaoImpl implements BookDao {
    public void save() {        
        System.out.println("book dao save ...");
        System.out.println(System.currentTimeMillis());
    }
    public void update(){
        System.out.println("book dao update ...");
    }
}
```

切入点表达式:

```java
execution(void com.itheima.dao.BookDao.update())
匹配接口，能匹配到
execution(void com.itheima.dao.impl.BookDaoImpl.update())
匹配实现类，能匹配到
execution(* com.itheima.dao.impl.BookDaoImpl.update())
返回值任意，能匹配到
execution(* com.itheima.dao.impl.BookDaoImpl.update(*))
返回值任意，但是update方法必须要有一个参数，无法匹配，要想匹配需要在update接口和实现类添加参数
execution(void com.*.*.*.*.update())
返回值为void,com包下的任意包三层包下的任意类的update方法，匹配到的是实现类，能匹配
execution(void com.*.*.*.update())
返回值为void,com包下的任意两层包下的任意类的update方法，匹配到的是接口，能匹配
execution(void *..update())
返回值为void，方法名是update的任意包下的任意类，能匹配
execution(* *..*(..))
匹配项目中任意类的任意方法，能匹配，但是不建议使用这种方式，影响范围广
execution(* *..u*(..))
匹配项目中任意包任意类下只要以u开头的方法，update方法能满足，能匹配
execution(* *..*e(..))
匹配项目中任意包任意类下只要以e结尾的方法，update和save方法能满足，能匹配
execution(void com..*())
返回值为void，com包下的任意包任意类任意方法，能匹配，*代表的是方法
execution(* com.itheima.*.*Service.find*(..))
将项目中所有业务层方法的以find开头的方法匹配
execution(* com.itheima.*.*Service.save*(..))
将项目中所有业务层方法的以save开头的方法匹配
```

##### 书写技巧

对于切入点表达式的编写其实是很灵活的，那么在编写的时候，有没有什么好的技巧让我们用用:

- 所有代码按照标准规范开发，否则以下技巧全部失效
- 描述切入点通**==常描述接口==**，而不描述实现类,如果描述到实现类，就出现紧耦合了
- 访问控制修饰符针对接口开发均采用public描述（**==可省略访问控制修饰符描述==**）
- 返回值类型对于增删改类使用精准类型加速匹配，对于查询类使用\*通配快速描述
- **==包名==**书写**==尽量不使用..匹配==**，效率过低，常用\*做单个包描述匹配，或精准匹配
- **==接口名/类名==**书写名称与模块相关的**==采用\*匹配==**，例如UserService书写成\*Service，绑定业务层接口名
- **==方法名==**书写以**==动词==**进行**==精准匹配==**，名词采用*匹配，例如getById书写成getBy*,selectAll书写成selectAll
- 参数规则较为复杂，根据业务方法灵活调整
- 通常**==不使用异常==**作为**==匹配==**规则



#### AOP通知类型

前面的案例中，有涉及到如下内容:

```java
@Before("pt()")
```

它所代表的含义是将`通知`添加到`切入点`方法执行的==前面==。

除了这个注解外，还有没有其他的注解，换个问题就是除了可以在前面加，能不能在其他的地方加?

##### 类型介绍

我们先来回顾下AOP通知:

* AOP通知描述了抽取的共性功能，根据共性功能抽取的位置不同，最终运行代码时要将其加入到合理的位置

通知具体要添加到切入点的哪里?

共提供了5种通知类型:

- 前置通知
- 后置通知
- **==环绕通知(重点)==**
- 返回后通知(了解)
- 抛出异常后通知(了解)

实验:

- 添加BookDao和BookDaoImpl类

  ```java
  public interface BookDao {
      public void update();
      public int select();
  }
  
  @Repository
  public class BookDaoImpl implements BookDao {
      public void update(){
          System.out.println("book dao update ...");
      }
      public int select() {
          System.out.println("book dao select is running ...");
          return 100;
      }
  }
  ```

- 创建Spring的配置类

  ```java
  @Configuration
  @ComponentScan("com.itheima")
  @EnableAspectJAutoProxy
  public class SpringConfig {
  }
  ```

- 创建通知类

  ```java
  @Component
  @Aspect
  public class MyAdvice {
      @Pointcut("execution(void com.itheima.dao.BookDao.update())")
      private void pt(){}
  
      public void before() {
          System.out.println("before advice ...");
      }
  
      public void after() {
          System.out.println("after advice ...");
      }
  
      public void around(){
          System.out.println("around before advice ...");
          System.out.println("around after advice ...");
      }
  
      public void afterReturning() {
          System.out.println("afterReturning advice ...");
      }
      
      public void afterThrowing() {
          System.out.println("afterThrowing advice ...");
      }
  }
  ```

- 编写App运行类

  ```java
  public class App {
      public static void main(String[] args) {
          ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
          BookDao bookDao = ctx.getBean(BookDao.class);
          bookDao.update();
      }
  }
  ```

##### 前置通知

修改MyAdvice,在before方法上添加`@Before注解`

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(void com.itheima.dao.BookDao.update())")
    private void pt(){}
    
    @Before("pt()")
    //此处也可以写成 @Before("MyAdvice.pt()"),不建议
    public void before() {
        System.out.println("before advice ...");
    }
}
```

输出

```
before  advice ...
book dao update is running ...
```

##### 后置通知

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(void com.itheima.dao.BookDao.update())")
    private void pt(){}
    
    @Before("pt()")
    public void before() {
        System.out.println("before advice ...");
    }
    @After("pt()")
    public void after() {
        System.out.println("after advice ...");
    }
}
```

输出:

```
before  advice ...
book dao update is running ...
after advice ...
```

##### 环绕通知

###### 基本使用

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(void com.itheima.dao.BookDao.update())")
    private void pt(){}
    
    @Around("pt()")
    public void around(){
        System.out.println("around before advice ...");
        System.out.println("around after advice ...");
    }
}
```

显示:

```
around before advice ...
around after advice ...
```

运行结果中，通知的内容打印出来，但是原始方法的内容却没有被执行。

因为环绕通知需要在原始方法的前后进行增强，所以环绕通知就必须要能对原始操作进行调用，具体如何实现?

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(void com.itheima.dao.BookDao.update())")
    private void pt(){}
    
    @Around("pt()")
    public void around(ProceedingJoinPoint pjp) throws Throwable{
        System.out.println("around before advice ...");
        //表示对原始操作的调用
        pjp.proceed();
        System.out.println("around after advice ...");
    }
}
```

###### 注意事项

(1)原始方法有返回值的处理

* 修改MyAdvice,对BookDao中的select方法添加环绕通知，

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(void com.itheima.dao.BookDao.update())")
    private void pt(){}
    
    @Pointcut("execution(int com.itheima.dao.BookDao.select())")
    private void pt2(){}
    
    @Around("pt2()")
    public void aroundSelect(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("around before advice ...");
        //表示对原始操作的调用
        pjp.proceed();
        System.out.println("around after advice ...");
    }
}
```

* 修改App类，调用select方法

```java
public class App {
    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
        BookDao bookDao = ctx.getBean(BookDao.class);
        int num = bookDao.select();
        System.out.println(num);
    }
}
```

运行后会报错，错误内容为:

Exception in thread "main" org.springframework.aop.AopInvocationException: ==Null return value from advice does not match primitive return type for: public abstract int com.itheima.dao.BookDao.select()==
	at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:226)
	at com.sun.proxy.$Proxy19.select(Unknown Source)
	at com.itheima.App.main(App.java:12)

错误大概的意思是:`空的返回不匹配原始方法的int返回`

* void就是返回Null
* 原始方法就是BookDao下的select方法

所以如果我们使用环绕通知的话，要根据原始方法的返回值来设置环绕通知的返回值，具体解决方案为:

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(void com.itheima.dao.BookDao.update())")
    private void pt(){}
    
    @Pointcut("execution(int com.itheima.dao.BookDao.select())")
    private void pt2(){}
    
    @Around("pt2()")
    public Object aroundSelect(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("around before advice ...");
        //表示对原始操作的调用
        Object ret = pjp.proceed();
        System.out.println("around after advice ...");
        return ret;
    }
}
```

**说明:**

​	为什么返回的是Object而不是int的主要原因是Object类型更通用。

​	在环绕通知中是可以对原始方法返回值就行修改的。



##### 返回后通知

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(void com.itheima.dao.BookDao.update())")
    private void pt(){}
    
    @Pointcut("execution(int com.itheima.dao.BookDao.select())")
    private void pt2(){}
    
    @AfterReturning("pt2()")
    public void afterReturning() {
        System.out.println("afterReturning advice ...");
    }
}
```

输出:

```
book dao select is running ...
afterReturning advice ...
100
```

**注意：**返回后通知是需要在原始方法`select`正常执行后才会被执行，如果`select()`方法执行的过程中出现了异常，那么返回后通知是不会被执行。后置通知是不管原始方法有没有抛出异常都会被执行。这个案例大家下去可以自己练习验证下。



##### 异常后通知

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(void com.itheima.dao.BookDao.update())")
    private void pt(){}
    
    @Pointcut("execution(int com.itheima.dao.BookDao.select())")
    private void pt2(){}
    
    @AfterReturning("pt2()")
    public void afterThrowing() {
        System.out.println("afterThrowing advice ...");
    }
}
```

**注意：**异常后通知是需要原始方法抛出异常，可以在`select()`方法中添加一行代码`int i = 1/0`即可。如果没有抛异常，异常后通知将不会被执行。

==**环绕通知注意事项**==

1. 环绕通知必须依赖形参ProceedingJoinPoint才能实现对原始方法的调用，进而实现原始方法调用前后同时添加通知
2. 通知中如果未使用ProceedingJoinPoint对原始方法进行调用将跳过原始方法的执行
3. 对原始方法的调用可以不接收返回值，通知方法设置成void即可，如果接收返回值，最好设定为Object类型
4. 原始方法的返回值如果是void类型，通知方法的返回值类型可以设置成void,也可以设置成Object
5. 由于无法预知原始方法运行后是否会抛出异常，因此环绕通知方法必须要处理Throwable异常



#### AOP通知获取数据

目前我们写AOP仅仅是在原始方法前后追加一些操作，接下来我们要说说AOP中数据相关的内容，我们将从`获取参数`、`获取返回值`和`获取异常`三个方面来研究切入点的相关信息。

##### 获取参数

###### 非环绕获取通知方式

在方法上添加JoinPoint,通过JoinPoint来获取参数

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(* com.itheima.dao.BookDao.findName(..))")
    private void pt(){}

    @Before("pt()")
    public void before(JoinPoint jp) 
        Object[] args = jp.getArgs();
        System.out.println(Arrays.toString(args));
        System.out.println("before advice ..." );
    }
	//...其他的略
}
```

###### 环绕通知获取方式

环绕通知使用的是ProceedingJoinPoint，因为ProceedingJoinPoint是JoinPoint类的子类，所以对于ProceedingJoinPoint类中应该也会有对应的`getArgs()`方法，我们去验证下:

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(* com.itheima.dao.BookDao.findName(..))")
    private void pt(){}

    @Around("pt()")
    public Object around(ProceedingJoinPoint pjp)throws Throwable {
        Object[] args = pjp.getArgs();
        System.out.println(Arrays.toString(args));
        Object ret = pjp.proceed();
        return ret;
    }
	//其他的略
}
```

**注意:**

* pjp.proceed()方法是有两个构造方法，分别是:

  ```java
  proceed();
  proceed(Object[] objects)
  ```

  * 调用无参数的proceed，当原始方法有参数，会在调用的过程中自动传入参数

  * 所以调用这两个方法的任意一个都可以完成功能

  * 但是当需要修改原始方法的参数时，就只能采用带有参数的方法,如下:

    ```java
    @Component
    @Aspect
    public class MyAdvice {
        @Pointcut("execution(* com.itheima.dao.BookDao.findName(..))")
        private void pt(){}
    
        @Around("pt()")
        public Object around(ProceedingJoinPoint pjp) throws Throwable{
            Object[] args = pjp.getArgs();
            System.out.println(Arrays.toString(args));
            args[0] = 666;
            Object ret = pjp.proceed(args);
            return ret;
        }
    	//其他的略
    }
    ```

    有了这个特性后，我们就可以在环绕通知中对原始方法的参数进行拦截过滤，避免由于参数的问题导致程序无法正确运行，保证代码的健壮性。

##### 获取返回值

对于返回值，只有返回后`AfterReturing`和环绕`Around`这两个通知类型可以获取，具体如何获取?

###### 环绕通知获取返回值

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(* com.itheima.dao.BookDao.findName(..))")
    private void pt(){}

    @Around("pt()")
    public Object around(ProceedingJoinPoint pjp) throws Throwable{
        Object[] args = pjp.getArgs();
        System.out.println(Arrays.toString(args));
        args[0] = 666;
        Object ret = pjp.proceed(args);
        return ret;
    }
	//其他的略
}
```

上述代码中，`ret`就是方法的返回值，我们是可以直接获取，不但可以获取，如果需要还可以进行修改。

###### 返回后通知获取返回值

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(* com.itheima.dao.BookDao.findName(..))")
    private void pt(){}

    @AfterReturning(value = "pt()",returning = "ret")
    public void afterReturning(Object ret) {
        System.out.println("afterReturning advice ..."+ret);
    }
	//其他的略
}
```

==注意:==

1. 参数名的问题
   `@AfterReturning(value = "pt()",returning = "ret") `中的ret必须要与`public void afterReturning(Object ret)`中的ret名字保持一致

2. afterReturning方法参数类型的问题

   参数类型可以写成String，但是为了能匹配更多的参数类型，建议写成Object类型

3. afterReturning方法参数的顺序问题

   `public void afterReturning(JoinPoint jp, Object ret) `
   如果有JoinPoint参数,jp必须要放在第一位

##### 获取异常

对于获取抛出的异常，只有抛出异常后`AfterThrowing`和环绕`Around`这两个通知类型可以获取，具体如何获取?

###### 环绕通知获取异常

这块比较简单，以前我们是抛出异常，现在只需要将异常捕获，就可以获取到原始方法的异常信息了

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(* com.itheima.dao.BookDao.findName(..))")
    private void pt(){}

    @Around("pt()")
    public Object around(ProceedingJoinPoint pjp){
        Object[] args = pjp.getArgs();
        System.out.println(Arrays.toString(args));
        args[0] = 666;
        Object ret = null;
        try{
            ret = pjp.proceed(args);
        }catch(Throwable throwable){
            t.printStackTrace();
        }
        return ret;
    }
	//其他的略
}
```

在catch方法中就可以获取到异常，至于获取到异常以后该如何处理，这个就和你的业务需求有关了。

###### 抛出异常后通知获取异常

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(* com.itheima.dao.BookDao.findName(..))")
    private void pt(){}

    @AfterThrowing(value = "pt()",throwing = "t")
    public void afterThrowing(Throwable t) {
        System.out.println("afterThrowing advice ..."+t);
    }
	//其他的略
}
```

如何让原始方法抛出异常，方式有很多，

```java
@Repository
public class BookDaoImpl implements BookDao {

    public String findName(int id,String password) {
        System.out.println("id:"+id);
        if(true){
            throw new NullPointerException();
        }
        return "itcast";
    }
}
```

==注意:== ` @AfterThrowing(value = "pt()",throwing = "t")`中的t与`afterThrowing(Throwable t)`中的t必须一致



### Spring事务

Spring事务作用：在数据层或**==业务层==**保障一系列的数据库操作同成功同失败

举个简单的例子，

* 转账业务会有两次数据层的调用，一次是加钱一次是减钱
* 把事务放在数据层，加钱和减钱就有两个事务
* 没办法保证加钱和减钱同时成功或者同时失败
* 这个时候就需要将事务放在业务层进行处理。

```java
public interface PlatformTransactionManager{
    void commit(TransactionStatus status) throws TransactionException;
    void rollback(TransactionStatus status) throws TransactionException;
}
```

`commit`是用来提交事务，`rollback`是用来回滚事务。

`PlatformTransactionManager`只是一个接口，Spring还为其提供了一个具体的实现:

```java
public class DataSourceTransactionManager{
    ......
}
```

从名称上可以看出，我们只需要给它一个DataSource对象，它就可以帮你去在业务层管理事务。其内部采用的是JDBC的事务。所以说如果你持久层采用的是JDBC相关的技术，就可以采用这个事务管理器来管理你的事务。而Mybatis内部采用的就是JDBC的事务，所以后期我们Spring整合Mybatis就采用的这个DataSourceTransactionManager事务管理器。

#### 案例

```java
@Service
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountDao accountDao;

    public void transfer(String out,String in ,Double money) {
        accountDao.outMoney(out,money);
        int i = 1/0;//出现异常
        accountDao.inMoney(in,money);
    }

}
```

模拟了转账过程中出现异常的情况,会发现转账的人钱少了,收账的人钱却没有增多.

不管哪种情况，都是不允许出现的，对刚才的结果我们做一个分析:

①：程序正常执行时，账户金额A减B加，没有问题

②：程序出现异常后，转账失败，但是异常之前操作成功，异常之后操作失败，整体业务失败

当程序出问题后，我们需要让事务进行回滚，而且这个事务应该是加在业务层上，而Spring的事务管理就是用来解决这类问题的。

Spring事务管理具体的实现步骤为:

##### 1:在需要被事务管理的方法上添加注解

```java
public interface AccountService {
    /**
     * 转账操作
     * @param out 传出方
     * @param in 转入方
     * @param money 金额
     */
    //配置当前接口方法具有事务
    public void transfer(String out,String in ,Double money) ;
}

@Service
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountDao accountDao;
	@Transactional
    public void transfer(String out,String in ,Double money) {
        accountDao.outMoney(out,money);
        int i = 1/0;
        accountDao.inMoney(in,money);
    }

}
```

==注意:==

`@Transactional`可以写在接口类上、接口方法上、实现类上和实现类方法上

* 写在接口类上，该接口的所有实现类的所有方法都会有事务
* 写在接口方法上，该接口的所有实现类的该方法都会有事务
* 写在实现类上，该类中的所有方法都会有事务
* 写在实现类方法上，该方法上有事务
* ==建议写在实现类或实现类的方法上==

##### 2:在JdbcConfig类中配置事务管理器

```java
public class JdbcConfig {
    @Value("${jdbc.driver}")
    private String driver;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
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

    //配置事务管理器，mybatis使用的是jdbc事务
    @Bean
    public PlatformTransactionManager transactionManager(DataSource dataSource){
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();
        transactionManager.setDataSource(dataSource);
        return transactionManager;
    }
}
```

**注意：**事务管理器要根据使用技术进行选择，Mybatis框架使用的是JDBC事务，可以直接使用`DataSourceTransactionManager`

##### 3：开启事务注解

在SpringConfig的配置类中开启

```java
@Configuration
@ComponentScan("com.itheima")
@PropertySource("classpath:jdbc.properties")
@Import({JdbcConfig.class,MybatisConfig.class
//开启注解式事务驱动
@EnableTransactionManagement
public class SpringConfig {
}

```

##### 知识点1：@EnableTransactionManagement

| 名称 | @EnableTransactionManagement           |
| ---- | -------------------------------------- |
| 类型 | 配置类注解                             |
| 位置 | 配置类定义上方                         |
| 作用 | 设置当前Spring环境中开启注解式事务支持 |

##### 知识点2：@Transactional   

| 名称 | @Transactional                                               |
| ---- | ------------------------------------------------------------ |
| 类型 | 接口注解  类注解  方法注解                                   |
| 位置 | 业务层接口上方  业务层实现类上方  业务方法上方               |
| 作用 | 为当前业务层方法添加事务（如果设置在类或接口上方则类或接口中所有方法均添加事务） |



#### Spring事务角色

重点要理解两个概念，分别是**事务管理员**和**事务协调员**。

1. 未开启Spring事务之前:
   - `AccountDao`的`outMoney`因为是修改操作，会开启一个事务T1
   - `AccountDao`的`inMoney`因为是修改操作，会开启一个事务T2
   - `AccountService`的`transfer`没有事务，
     * 运行过程中如果没有抛出异常，则T1和T2都正常提交，数据正确
     * 如果在两个方法中间抛出异常，T1因为执行成功提交事务，T2因为抛异常不会被执行
     * 就会导致数据出现错误
2. 开启Spring的事务管理后
   - `transfer`上添加了`@Transactional`注解，在该方法上就会有一个事务T
   - `AccountDao`的`outMoney`方法的事务T1加入到`transfer`的事务T中
   - `AccountDao`的`inMoney`方法的事务T2加入到`transfer`的事务T中
   - 这样就保证他们在同一个事务中，当业务层中出现异常，整个事务就会回滚，保证数据的准确性。

通过上面例子的分析，我们就可以得到如下概念:

- 事务管理员：发起事务方，在Spring中通常指代业务层开启事务的方法
- 事务协调员：加入事务方，在Spring中通常指代数据层方法，也可以是业务层方法

==注意:==

目前的事务管理是基于`DataSourceTransactionManager`和`SqlSessionFactoryBean`使用的是同一个数据源。



#### 事务传播行为

事务传播行为：事务协调员对事务管理员所携带事务的处理态度。

需要用到之前我们没有说的`propagation属性`。

例子:上面的转账方法,我们要求增加一个日志记录功能,要求转账成功与否都要记录.

````java
@Service
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountDao accountDao;
    @Autowired
    private LogService logService;
	@Transactional
    public void transfer(String out,String in ,Double money) {
        try{
            accountDao.outMoney(out,money);
            accountDao.inMoney(in,money);
        }finally {
            logService.log(out,in,money);
        }
    }
}
````

这样写有问题吗?正常情况下能实现,但是转账中如果出现异常,那么就会回滚事务,记录日志也在这个事务中一起回滚,本来记录的数据就又回滚没了.要想解决这个问题，就需要用到事务传播行为,需要用到之前我们没有说的`propagation属性`。

##### 修改logService改变事务的传播行为

```java
@Service
public class LogServiceImpl implements LogService {

    @Autowired
    private LogDao logDao;
	//propagation设置事务属性：传播行为设置为当前操作需要新事务
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void log(String out,String in,Double money ) {
        logDao.log("转账操作由"+out+"到"+in+",金额："+money);
    }
}
```

运行后，就能实现我们想要的结果，不管转账是否成功，都会记录日志。

##### 2.事务传播行为的可选值

![1630254257628](D:/BaiduNetdiskDownload/基础框架8笔记/Spring笔记/spring_day03/assets/1630254257628.png)

对于我们开发实际中使用的话，因为默认值需要事务是常态的。根据开发过程选择其他的就可以了，例如案例中需要新事务就需要手工配置。其实入账和出账操作上也有事务，采用的就是默认值。



## SpringMVC

### 入门案例

因为SpringMVC是一个Web框架，将来是要替换Servlet,所以先来回顾下以前Servlet是如何进行开发的?

1.创建web工程(Maven结构)

2.设置tomcat服务器，加载web工程(tomcat插件)

3.导入坐标(Servlet)

4.定义处理请求的功能类(UserServlet)

5.设置请求映射(配置映射关系)

SpringMVC的制作过程和上述流程几乎是一致的，具体的实现流程是什么?

1.创建web工程(Maven结构)

2.设置tomcat服务器，加载web工程(tomcat插件)

3.导入坐标(==SpringMVC==+Servlet)

4.定义处理请求的功能类(==UserController==)

5.==设置请求映射(配置映射关系)==

6.==将SpringMVC设定加载到Tomcat容器中==

#### pom.xml配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.itheima</groupId>
  <artifactId>springmvc_01_quickstart</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>

  <dependencies>
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.2.10.RELEASE</version>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.tomcat.maven</groupId>
        <artifactId>tomcat7-maven-plugin</artifactId>
        <version>2.1</version>
        <configuration>
          <port>80</port>
          <path>/</path>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

**说明:**servlet的坐标为什么需要添加`<scope>provided</scope>`?

* scope是maven中jar包依赖作用范围的描述，
* 如果不设置默认是`compile`在在编译、运行、测试时均有效
* 如果运行有效的话就会和tomcat中的servlet-api包发生冲突，导致启动报错
* provided代表的是该包只在编译和测试的时候用，运行的时候无效直接使用tomcat中的，就避免冲突

#### 创建配置类

```java
@Configuration
@ComponentScan("com.itheima.controller")
public class SpringMvcConfig {
}
```

#### 创建Controller类

```java
@Controller
public class UserController {
    
    @RequestMapping("/save")
    @ResponseBody
    public String save(){
        System.out.println("user save ...");
        return "{'info':'springmvc'}";
    }
}
```

#### 使用配置类替换web.xml

将web.xml删除，换成ServletContainersInitConfig

```java
public class ServletContainersInitConfig extends AbstractDispatcherServletInitializer {
    //加载springmvc配置类
    protected WebApplicationContext createServletApplicationContext() {
        //初始化WebApplicationContext对象
        AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
        //加载指定配置类
        ctx.register(SpringMvcConfig.class);
        return ctx;
    }

    //设置由springmvc控制器处理的请求映射路径
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }

    //加载spring配置类
    protected WebApplicationContext createRootApplicationContext() {
        return null;
    }
}
```

#### 知识点1：@Controller

| 名称 | @Controller                   |
| ---- | ----------------------------- |
| 类型 | 类注解                        |
| 位置 | SpringMVC控制器类定义上方     |
| 作用 | 设定SpringMVC的核心控制器bean |

#### 知识点2：@RequestMapping

| 名称     | @RequestMapping                 |
| -------- | ------------------------------- |
| 类型     | 类注解或方法注解                |
| 位置     | SpringMVC控制器类或方法定义上方 |
| 作用     | 设置当前控制器方法请求访问路径  |
| 相关属性 | value(默认)，请求访问路径       |

#### 知识点3：@ResponseBody

| 名称 | @ResponseBody                                    |
| ---- | ------------------------------------------------ |
| 类型 | 类注解或方法注解                                 |
| 位置 | SpringMVC控制器类或方法定义上方                  |
| 作用 | 设置当前控制器方法响应内容为当前返回值，无需解析 |

总结:

- 一次性工作
  - 创建工程，设置服务器，加载工程
  - 导入坐标
  - 创建web容器启动类，加载SpringMVC配置，并设置SpringMVC请求拦截路径
  - SpringMVC核心配置类（设置配置类，扫描controller包，加载Controller控制器bean）
- 多次工作
  - 定义处理请求的控制器类
  - 定义处理请求的控制器方法，并配置映射路径（@RequestMapping）与返回json数据（@ResponseBody）

### 案例工作流程

#### 启动服务器初始化过程

1. 服务器启动，执行ServletContainersInitConfig类，初始化web容器

   * 功能类似于以前的web.xml

2. 执行createServletApplicationContext方法，创建了WebApplicationContext对象

   * 该方法加载SpringMVC的配置类SpringMvcConfig来初始化SpringMVC的容器

3. 加载SpringMvcConfig配置类

4. 执行@ComponentScan加载对应的bean

   * 扫描指定包及其子包下所有类上的注解，如Controller类上的@Controller注解

5. 加载UserController，每个@RequestMapping的名称对应一个具体的方法

   ```java
   @Controller
   public class UserController {
       
       @RequestMapping("/save")
       @ResponseBody
       public String save(){
           System.out.println("user save ...");
           return "{'info':'springmvc'}";
       }
   }
   ```

   - 此时就建立了`/save`和save方法的对应关系

6. 执行getServletMappings方法，设定SpringMVC拦截请求的路径规则

   ```java
    protected String[] getServletMappings() {
           return new String[]{"/"};
       }
   ```

   - `/`代表所拦截请求的路径规则，只有被拦截后才能交给SpringMVC来处理请求

#### 单次请求过程

1. 发送请求`http://localhost/save`
2. web容器发现该请求满足SpringMVC拦截规则，将请求交给SpringMVC处理
3. 解析请求路径/save
4. 由/save匹配执行对应的方法save(）
   * 上面的第五步已经将请求路径和方法建立了对应关系，通过/save就能找到对应的save方法
5. 执行save()
6. 检测到有@ResponseBody直接将save()方法的返回值作为响应体返回给请求方

### bean加载过程

项目中有许多bean,有些需要让Spring加载,有些需要让SpringMVC加载,如何让它们分开加载各自的内容?也就是==因为功能不同，如何避免Spring错误加载到SpringMVC的bean?==

解决方案很简单,就是加载Spring控制的bean的时候排除掉SpringMVC控制的bean.有三种方式:

1. Spring加载的bean设定扫描范围为精准范围，例如service包、dao包等

   ```java
   @Configuration@ComponentScan({"com.itheima.service","comitheima.dao"})public class SpringConfig {}
   ```

   **说明:**

   上述只是通过例子说明可以精确指定让Spring扫描对应的包结构，真正在做开发的时候，因为Dao最终是交给`MapperScannerConfigurer`对象来进行扫描处理的，我们只需要将其扫描到service包即可。

2. Spring加载的bean设定扫描范围为com.itheima,排除掉controller包中的bean

   ```java
   @Configuration
   @ComponentScan(value="com.itheima",
       excludeFilters=@ComponentScan.Filter(
       	type = FilterType.ANNOTATION,
           classes = Controller.class
       )
   )
   public class SpringConfig {
   }
   ```

   - excludeFilters属性：设置扫描加载bean时，排除的过滤规则

   - type属性：设置排除规则，当前使用按照bean定义时的注解类型进行排除

     * ANNOTATION：按照注解排除
     * ASSIGNABLE_TYPE:按照指定的类型过滤
     * ASPECTJ:按照Aspectj表达式排除，基本上不会用
     * REGEX:按照正则表达式排除
     * CUSTOM:按照自定义规则排除

     大家只需要知道第一种ANNOTATION即可

   - classes属性：设置排除的具体注解类，当前设置排除@Controller定义的bean

3. 不区分Spring与SpringMVC的环境，加载到同一个环境中[了解即可]

最后一个问题，有了Spring的配置类，要想在tomcat服务器启动将其加载，我们需要修改ServletContainersInitConfig

```java
public class ServletContainersInitConfig extends AbstractDispatcherServletInitializer {
    protected WebApplicationContext createServletApplicationContext() {
        AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
        ctx.register(SpringMvcConfig.class);
        return ctx;
    }
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
    protected WebApplicationContext createRootApplicationContext() {
      AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
        ctx.register(SpringConfig.class);
        return ctx;
    }
}
```

对于上述的配置方式，Spring还提供了一种更简单的配置方式，可以不用再去创建`AnnotationConfigWebApplicationContext`对象，不用手动`register`对应的配置类，如何实现?



```java
public class ServletContainersInitConfig extends AbstractAnnotationConfigDispatcherServletInitializer {

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



### 请求与响应

SpringMVC是web层的框架，主要的作用是接收请求、接收数据、响应结果，所以这一章节是学习SpringMVC的==重点==内容,主要有四部分内容:

* 请求映射路径
* 请求参数
* 日期类型参数传递
* 响应json数据

#### 设置请求映射路径

```java
@Controller
public class UserController {

    @RequestMapping("/save")
    @ResponseBody
    public String save(){
        System.out.println("user save ...");
        return "{'module':'user save'}";
    }
    
    @RequestMapping("/delete")
    @ResponseBody
    public String save(){
        System.out.println("user delete ...");
        return "{'module':'user delete'}";
    }
}

@Controller
public class BookController {

    @RequestMapping("/save")
    @ResponseBody
    public String save(){
        System.out.println("book save ...");
        return "{'module':'book save'}";
    }
}
```

上面的代码运行会报错,因为UserController和BookController各有一个save方法,访问路径还一样,这样就冲突了

我们应该为不同模块设置模块名作为**请求路径前置**

对于Book模块的save,将其访问路径设置`http://localhost/book/save`

对于User模块的save,将其访问路径设置`http://localhost/user/save`

我们可以对每个方法的`@RequestMapping`修改加上前缀,但是如果方法太多了,每个都加就太麻烦了.我们可以在类前加一个`RequestMapping`,相当于雷利所有的方法请求路径都加上了这个前缀:

```java
@Controller
@RequestMapping("/user")
public class UserController {

    @RequestMapping("/save")
    @ResponseBody
    public String save(){
        System.out.println("user save ...");
        return "{'module':'user save'}";
    }
    
    @RequestMapping("/delete")
    @ResponseBody
    public String save(){
        System.out.println("user delete ...");
        return "{'module':'user delete'}";
    }
}

@Controller
@RequestMapping("/book")
public class BookController {

    @RequestMapping("/save")
    @ResponseBody
    public String save(){
        System.out.println("book save ...");
        return "{'module':'book save'}";
    }
}
```

**注意:**

* 当类上和方法上都添加了`@RequestMapping`注解，前端发送请求的时候，要和两个注解的value值相加匹配才能访问到。
* @RequestMapping注解value属性前面加不加`/`都可以



#### 请求参数

关于请求参数的传递与接收是和请求方式有关系的，目前比较常见的两种请求方式为：

* GET
* POST

##### 参数传递

###### GET发送单个参数

发送请求与参数

```
http://localhost/commonParam?name=itcast
```

接收参数

```java
@Controller
public class UserController {

    @RequestMapping("/commonParam")
    @ResponseBody
    public String commonParam(String name){
        System.out.println("普通参数传递 name ==> "+name);
        return "{'module':'commonParam'}";
    }
}
```

###### GET发送多个参数

发送请求与参数:

```
http://localhost/commonParam?name=itcast&age=15
```

接收参数：

```java
@Controller
public class UserController {

    @RequestMapping("/commonParam")
    @ResponseBody
    public String commonParam(String name,int age){
        System.out.println("普通参数传递 name ==> "+name);
        System.out.println("普通参数传递 age ==> "+age);
        return "{'module':'commonParam'}";
    }
}
```

###### GET请求中文乱码

出现乱码的原因相信大家都清楚，Tomcat8.5以后的版本已经处理了中文乱码的问题，但是IDEA中的Tomcat插件目前只到Tomcat7，所以需要修改pom.xml来解决GET请求中文乱码问题

```xml
<build>
    <plugins>
      <plugin>
        <groupId>org.apache.tomcat.maven</groupId>
        <artifactId>tomcat7-maven-plugin</artifactId>
        <version>2.1</version>
        <configuration>
          <port>80</port><!--tomcat端口号-->
          <path>/</path> <!--虚拟目录-->
          <uriEncoding>UTF-8</uriEncoding><!--访问路径编解码字符集-->
        </configuration>
      </plugin>
    </plugins>
  </build>
```

###### POST发送参数

和GET一致

###### POST请求中文乱码

解决方案:配置过滤器

```java
public class ServletContainersInitConfig extends AbstractAnnotationConfigDispatcherServletInitializer {
    protected Class<?>[] getRootConfigClasses() {
        return new Class[0];
    }

    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{SpringMvcConfig.class};
    }

    protected String[] getServletMappings() {
        return new String[]{"/"};
    }

    //乱码处理
    @Override
    protected Filter[] getServletFilters() {
        CharacterEncodingFilter filter = new CharacterEncodingFilter();
        filter.setEncoding("UTF-8");
        return new Filter[]{filter};
    }
}
```

CharacterEncodingFilter是在spring-web包中，所以用之前需要导入对应的jar包。





#### 五种类型参数传递

我们来研究一些比较复杂的参数传递，常见的参数种类有:

* 普通参数
* POJO类型参数
* 嵌套POJO类型参数
* 数组类型参数
* 集合类型参数

这些参数如何发送，后台该如何接收?我们一个个来学习。

##### 普通参数

- 普通参数:url地址传参，地址参数名与形参变量名相同，定义形参即可接收参数。

  ![image-20240309140655021](C:\Users\86139\AppData\Roaming\Typora\typora-user-images\image-20240309140655021.png)

如果形参与地址参数名不一致该如何解决?

发送请求与参数:

```
http://localhost/commonParamDifferentName?name=张三&age=18
```

后台接收参数:

```java
@RequestMapping("/commonParamDifferentName")
@ResponseBody
public String commonParamDifferentName(String userName , int age){
    System.out.println("普通参数传递 userName ==> "+userName);
    System.out.println("普通参数传递 age ==> "+age);
    return "{'module':'common param different name'}";
}
```

因为前端给的是`name`,后台接收使用的是`userName`,两个名称对不上，导致接收数据失败

解决方案:使用@RequestParam注解

```java
@RequestMapping("/commonParamDifferentName")
    @ResponseBody
    public String commonParamDifferentName(@RequestPaam("name") String userName , int age){
        System.out.println("普通参数传递 userName ==> "+userName);
        System.out.println("普通参数传递 age ==> "+age);
        return "{'module':'common param different name'}";
    }
```

**注意:写上@RequestParam注解框架就不需要自己去解析注入，能提升框架处理性能**

 

##### POJO数据类型

简单数据类型一般处理的是参数个数比较少的请求，如果参数比较多，那么后台接收参数的时候就比较复杂，这个时候我们可以考虑使用POJO数据类型。

* POJO参数：请求参数名与形参对象属性名相同，定义POJO类型形参即可接收参数

此时需要使用前面准备好的POJO类，先来看下User

```java
public class User {
    private String name;
    private int age;
    //setter...getter...略
}
```

发送请求和参数:

![image-20240309141051071](C:\Users\86139\AppData\Roaming\Typora\typora-user-images\image-20240309141051071.png)

后台接收参数:

```java
//POJO参数：请求参数与形参对象中的属性对应即可完成参数传递
@RequestMapping("/pojoParam")
@ResponseBody
public String pojoParam(User user){
    System.out.println("pojo参数传递 user ==> "+user);
    return "{'module':'pojo param'}";
}
```

**注意:**

* POJO参数接收，前端GET和POST发送请求数据的方式不变。
* ==请求参数key的名称要和POJO中属性的名称一致，否则无法封装。==



##### 嵌套POJO类型参数

如果POJO对象中嵌套了其他的POJO类，如

```java
public class Address {
    private String province;
    private String city;
    //setter...getter...略
}
public class User {
    private String name;
    private int age;
    private Address address;
    //setter...getter...略
}
```

* 嵌套POJO参数：请求参数名与形参对象属性名相同，按照对象层次结构关系即可接收嵌套POJO属性参数

发送请求和参数:

![1630482363291](D:/BaiduNetdiskDownload/基础框架8笔记/SpringMVC笔记/SpringMVC_day01/assets/1630482363291.png)

后台接收参数:

```java
//POJO参数：请求参数与形参对象中的属性对应即可完成参数传递
@RequestMapping("/pojoParam")
@ResponseBody
public String pojoParam(User user){
    System.out.println("pojo参数传递 user ==> "+user);
    return "{'module':'pojo param'}";
}
```

**注意:**

==请求参数key的名称要和POJO中属性的名称一致，否则无法封装==



##### 数组类型参数

举个简单的例子，如果前端需要获取用户的爱好，爱好绝大多数情况下都是多个，如何发送请求数据和接收数据呢?

* 数组参数：请求参数名与形参对象属性名相同且请求参数为多个，定义数组类型即可接收参数

发送请求和参数:

![1630482999626](D:/BaiduNetdiskDownload/基础框架8笔记/SpringMVC笔记/SpringMVC_day01/assets/1630482999626.png)

后台接收参数:

```java
  //数组参数：同名请求参数可以直接映射到对应名称的形参数组对象中
    @RequestMapping("/arrayParam")
    @ResponseBody
    public String arrayParam(String[] likes){
        System.out.println("数组参数传递 likes ==> "+ Arrays.toString(likes));
        return "{'module':'array param'}";
    }
```

##### 集合类型参数

数组能接收多个值，那么集合是否也可以实现这个功能呢?

发送请求和参数:

![1630484283773](D:/BaiduNetdiskDownload/基础框架8笔记/SpringMVC笔记/SpringMVC_day01/assets/1630484283773.png)

后台接收参数:

```java
//集合参数：同名请求参数可以使用@RequestParam注解映射到对应名称的集合对象中作为数据
@RequestMapping("/listParam")
@ResponseBody
public String listParam(List<String> likes){
    System.out.println("集合参数传递 likes ==> "+ likes);
    return "{'module':'list param'}";
}
```

**运行会报错**

错误的原因是:SpringMVC将List看做是一个POJO对象来处理，将其创建一个对象并准备把前端的数据封装到对象中，但是List是一个接口无法创建对象，所以报错。

解决方案是:使用`@RequestParam`注解

```java
//集合参数：同名请求参数可以使用@RequestParam注解映射到对应名称的集合对象中作为数据
@RequestMapping("/listParam")
@ResponseBody
public String listParam(@RequestParam List<String> likes){
    System.out.println("集合参数传递 likes ==> "+ likes);
    return "{'module':'list param'}";
}
```

* 集合保存普通参数：请求参数名与形参集合对象名相同且请求参数为多个，@RequestParam绑定参数关系
* 对于简单数据类型使用数组会比集合更简单些。

#### 知识点：@RequestParam

| 名称     | @RequestParam                                          |
| -------- | ------------------------------------------------------ |
| 类型     | 形参注解                                               |
| 位置     | SpringMVC控制器方法形参定义前面                        |
| 作用     | 绑定请求参数与处理器方法形参间的关系                   |
| 相关参数 | required：是否为必传参数 <br/>defaultValue：参数默认值 |

#### JSON数据传输参数

前面我们说过，现在比较流行的开发方式为异步调用。前后台以异步方式进行交换，传输的数据使用的是==JSON==,所以前端如果发送的是JSON数据，后端该如何接收?

对于JSON数据类型，我们常见的有三种:

- json普通数组（`["value1","value2","value3",...]`）
- json对象（`{key1:value1,key2:value2,...}`）
- json对象数组（`[{key1:value1,...},{key2:value2,...}]`）

对于上述数据，前端如何发送，后端如何接收?

##### JSON普通数组

1. pom.xml添加依赖

   SpringMVC默认使用的是jackson来处理json的转换，所以需要在pom.xml添加jackson依赖

   ```xml
   <dependency>
       <groupId>com.fasterxml.jackson.core</groupId>
       <artifactId>jackson-databind</artifactId>
       <version>2.9.0</version>
   </dependency>
   ```

2. 发送JSON数据
   ![image-20240309144226259](C:\Users\86139\AppData\Roaming\Typora\typora-user-images\image-20240309144226259.png)

3. 开启SpringMVC注解支持

   在SpringMVC的配置类中开启SpringMVC的注解支持，这里面就包含了将JSON转换成对象的功能。

   ```java
   @Configuration
   @ComponentScan("com.itheima.controller")
   //开启json数据类型自动转换
   @EnableWebMvc
   public class SpringMvcConfig {
   }
   ```

4. 参数前添加@RequestBody

   ```java
   //使用@RequestBody注解将外部传递的json数组数据映射到形参的集合对象中作为数据
   @RequestMapping("/listParamForJson")
   @ResponseBody
   public String listParamForJson(@RequestBody List<String> likes){
       System.out.println("list common(json)参数传递 list ==> "+likes);
       return "{'module':'list common for json param'}";
   }
   ```

##### JSON对象数据

只需要关注请求和数据如何发送?后端数据如何接收?

请求和数据的发送:

```json
{
	"name":"itcast",
	"age":15
}
```

![image-20240309144700554](C:\Users\86139\AppData\Roaming\Typora\typora-user-images\image-20240309144700554.png)

后端接收数据：

```java
@RequestMapping("/pojoParamForJson")
@ResponseBody
public String pojoParamForJson(@RequestBody User user){
    System.out.println("pojo(json)参数传递 user ==> "+user);
    return "{'module':'pojo for json param'}";
}
```



##### JSON对象数组

集合中保存多个POJO该如何实现?

请求和数据的发送:

```json
[
    {"name":"itcast","age":15},
    {"name":"itheima","age":12}
]
```

![image-20240309144851471](C:\Users\86139\AppData\Roaming\Typora\typora-user-images\image-20240309144851471.png)

后端接收数据:

```java
@RequestMapping("/listPojoParamForJson")
@ResponseBody
public String listPojoParamForJson(@RequestBody List<User> list){
    System.out.println("list pojo(json)参数传递 list ==> "+list);
    return "{'module':'list pojo for json param'}";
}
```

**小结**

SpringMVC接收JSON数据的实现步骤为:

(1)导入jackson包

(2)使用PostMan发送JSON数据

(3)开启SpringMVC注解驱动，在配置类上添加@EnableWebMvc注解

(4)Controller方法的参数前添加@RequestBody注解

##### 知识点1：@EnableWebMvc

| 名称 | @EnableWebMvc             |
| ---- | ------------------------- |
| 类型 | ==配置类注解==            |
| 位置 | SpringMVC配置类定义上方   |
| 作用 | 开启SpringMVC多项辅助功能 |

##### 知识点2：@RequestBody

| 名称 | @RequestBody                                                 |
| ---- | ------------------------------------------------------------ |
| 类型 | ==形参注解==                                                 |
| 位置 | SpringMVC控制器方法形参定义前面                              |
| 作用 | 将请求中请求体所包含的数据传递给请求参数，此注解一个处理器方法只能使用一次 |

###### @RequestBody与@RequestParam区别

* 区别
  * @RequestParam用于接收url地址传参，表单传参【application/x-www-form-urlencoded】
  * @RequestBody用于接收json数据【application/json】

* 应用
  * 后期开发中，发送json格式数据为主，@RequestBody应用较广
  * 如果发送非json格式数据，选用@RequestParam接收请求参数



#### 日期类型参数传递

对于日期的格式有N多中输入方式，比如:

* 2088-08-18
* 2088/08/18
* 08/18/2088
* ......

针对这么多日期格式，SpringMVC该如何接收，它能很好的处理日期类型数据么?

1. yyyy/mm/dd 形式的,可以直接接收.

   ```java
   @RequestMapping("/dataParam")
   @ResponseBody
   public String dataParam(Date date,Date date1)
       System.out.println("参数传递 date ==> "+date);
       return "{'module':'data param'}";
   }
   ```

2. 其他格式,使用`@DateTimeFormat`

   ```java
   @RequestMapping("/dataParam")
   @ResponseBody
   public String dataParam(Date date,
                           @DateTimeFormat(pattern="yyyy-MM-dd") Date date1)
       System.out.println("参数传递 date ==> "+date);
   	System.out.println("参数传递 date1(yyyy-MM-dd) ==> "+date1);
       return "{'module':'data param'}";
   }
   ```



#### 响应

SpringMVC接收到请求和数据后，进行一些了的处理，当然这个处理可以是转发给Service，Service层再调用Dao层完成的，不管怎样，处理完以后，都需要将结果告知给用户。

比如:根据用户ID查询用户信息、查询用户列表、新增用户等。

对于响应，主要就包含两部分内容：

* 响应页面
* 响应数据
  * 文本数据
  * json数据

因为异步调用是目前常用的主流方式，所以我们需要更关注的就是如何返回JSON数据，对于其他只需要认识了解即可。

##### 响应页面

比如我们写了一个`page.jsp`页面,要在某个程序返回这个页面.:

```java
@Controller
public class UserController {
    
    @RequestMapping("/toJumpPage")
    //注意
    //1.此处不能添加@ResponseBody,如果加了该注入，会直接将page.jsp当字符串返回前端
    //2.方法需要返回String
    public String toJumpPage(){
        System.out.println("跳转页面");
        return "page.jsp";
    }
    
}
```

这样访问`/toJumpPage`就能跳转到`page.jsp`

##### 返回文本数据

```java
@Controller
public class UserController {
    
   	@RequestMapping("/toText")
	//注意此处该注解就不能省略，如果省略了,会把response text当前页面名称去查找，如果没有回报404错误
    @ResponseBody
    public String toText(){
        System.out.println("返回纯文本数据");
        return "response text";
    }
}
```

##### 响应JSON数据

###### 响应POJO对象

```java
@Controller
public class UserController {
    
    @RequestMapping("/toJsonPOJO")
    @ResponseBody
    public User toJsonPOJO(){
        System.out.println("返回json对象数据");
        User user = new User();
        user.setName("itcast");
        user.setAge(15);
        return user;
    }
    
}
```

返回值为实体类对象，设置返回值为实体类类型，即可实现返回对应对象的json数据，需要依赖==@ResponseBody==注解和==@EnableWebMvc==注解

###### 响应POJO集合对象

```java
@Controller
public class UserController {
    
    @RequestMapping("/toJsonList")
    @ResponseBody
    public List<User> toJsonList(){
        System.out.println("返回json集合数据");
        User user1 = new User();
        user1.setName("传智播客");
        user1.setAge(15);

        User user2 = new User();
        user2.setName("黑马程序员");
        user2.setAge(12);

        List<User> userList = new ArrayList<User>();
        userList.add(user1);
        userList.add(user2);

        return userList;
    }
    
}

```

##### 知识点：@ResponseBody

| 名称     | @ResponseBody                                                |
| -------- | ------------------------------------------------------------ |
| 类型     | ==方法\类注解==                                              |
| 位置     | SpringMVC控制器方法定义上方和控制类上                        |
| 作用     | 设置当前控制器返回值作为响应体,<br/>写在类上，该类的所有方法都有该注解功能 |
| 相关属性 | pattern：指定日期时间格式字符串                              |

**说明:**

* 该注解可以写在类上或者方法上
* 写在类上就是该类下的所有方法都有@ReponseBody功能
* 当方法上有@ReponseBody注解后
  * 方法的返回值为字符串，会将其作为文本内容直接响应给前端
  * 方法的返回值为对象，会将对象转换成JSON响应给前端

此处又使用到了类型转换，内部还是通过Converter接口的实现类完成的，所以Converter除了前面所说的功能外，它还可以实现:

* 对象转Json数据(POJO -> json)
* 集合转Json数据(Collection -> json)



### REST风格

* ==REST==（Representational State Transfer），表现形式状态转换,它是一种软件架构==风格==

  当我们想表示一个网络资源的时候，可以使用两种方式:

  * 传统风格资源描述形式
    * `http://localhost/user/getById?id=1` 查询id为1的用户信息
    * `http://localhost/user/saveUser` 保存用户信息
  * REST风格描述形式
    * `http://localhost/user/1` 
    * `http://localhost/user`

传统方式一般是一个请求url对应一种操作，这样做不仅麻烦，也不安全，因为会程序的人读取了你的请求url地址，就大概知道该url实现的是一个什么样的操作。

查看REST风格的描述，你会发现请求地址变的简单了，并且光看请求URL并不是很能猜出来该URL的具体功能

所以REST的优点有:

- 隐藏资源的访问行为，无法通过地址得知对资源是何种操作
- 书写简化

但是我们的问题也随之而来了，一个相同的url地址即可以是新增也可以是修改或者查询，那么到底我们该如何区分该请求到底是什么操作呢?

* 按照REST风格访问资源时使用==行为动作==区分对资源进行了何种操作
  * `http://localhost/users`	查询全部用户信息 GET（查询）
  * `http://localhost/users/1`  查询指定用户信息 GET（查询）
  * `http://localhost/users`    添加用户信息    POST（新增/保存）
  * `http://localhost/users`    修改用户信息    PUT（修改/更新）
  * `http://localhost/users/1`  删除用户信息    DELETE（删除）

请求的方式比较多，但是比较常用的就4种，分别是`GET`,`POST`,`PUT`,`DELETE`。

按照不同的请求方式代表不同的操作类型。

* 发送GET请求是用来做查询
* 发送POST请求是用来做新增
* 发送PUT请求是用来做修改
* 发送DELETE请求是用来做删除



#### RESTful入门案例

##### 增

`@RequestMapping`里多加一个参数为method,选择`RequestMethod.xxx`,增使用POST

```java
@Controller
public class UserController {
	//设置当前请求方法为POST，表示REST风格中的添加操作
    @RequestMapping(value = "/users",method = RequestMethod.POST)
    @ResponseBody
    public String save() {
        System.out.println("user save...");
        return "{'module':'user save'}";
    }
}
```

* 将请求路径更改为`/users`

  * 访问该方法使用 POST: `http://localhost/users`

* 使用method属性限定该方法的访问方式为`POST`

  * 如果发送的不是POST请求，比如发送GET请求，则会报错


##### 删

```java
@Controller
public class UserController {
    //设置当前请求方法为DELETE，表示REST风格中的删除操作
	@RequestMapping(value = "/users",method = RequestMethod.DELETE)
    @ResponseBody
    public String delete(Integer id) {
        System.out.println("user delete..." + id);
        return "{'module':'user delete'}";
    }
}
```

* 将请求路径更改为`/users`
  - 访问该方法使用 DELETE: `http://localhost/users`

访问成功，但是删除方法没有携带所要删除数据的id,所以针对RESTful的开发，如何携带数据参数?

###### 传递路径参数

前端发送请求的时候使用:`http://localhost/users/1`,路径中的`1`就是我们想要传递的参数。

后端获取参数，需要做如下修改:

* 修改`@RequestMapping`的value属性，将其中修改为`/users/{id}`，目的是和路径匹配
* 在方法的形参前添加`@PathVariable`注解

```java
@Controller
public class UserController {
    //设置当前请求方法为DELETE，表示REST风格中的删除操作
	@RequestMapping(value = "/users/{id}",method = RequestMethod.DELETE)
    @ResponseBody
    public String delete(@PathVariable Integer id) {
        System.out.println("user delete..." + id);
        return "{'module':'user delete'}";
    }
}
```

##### 改

```java
@Controller
public class UserController {
    //设置当前请求方法为PUT，表示REST风格中的修改操作
    @RequestMapping(value = "/users",method = RequestMethod.PUT)
    @ResponseBody
    public String update(@RequestBody User user) {
        System.out.println("user update..." + user);
        return "{'module':'user update'}";
    }
}
```

将请求路径更改为`/users`

- 访问该方法使用 PUT: `http://localhost/users`

##### 根据ID查

```java
@Controller
public class UserController {
    //设置当前请求方法为GET，表示REST风格中的查询操作
    @RequestMapping(value = "/users/{id}" ,method = RequestMethod.GET)
    @ResponseBody
    public String getById(@PathVariable Integer id){
        System.out.println("user getById..."+id);
        return "{'module':'user getById'}";
    }
}
```

将请求路径更改为`/users`

- 访问该方法使用 GET: `http://localhost/users/666`

##### 查询所有

```java
@Controller
public class UserController {
    //设置当前请求方法为GET，表示REST风格中的查询操作
    @RequestMapping(value = "/users" ,method = RequestMethod.GET)
    @ResponseBody
    public String getAll() {
        System.out.println("user getAll...");
        return "{'module':'user getAll'}";
    }
}
```

将请求路径更改为`/users`

- 访问该方法使用 GET: `http://localhost/users`

##### 知识点:@PathVariable

| 名称 | @PathVariable                                                |
| ---- | ------------------------------------------------------------ |
| 类型 | ==形参注解==                                                 |
| 位置 | SpringMVC控制器方法形参定义前面                              |
| 作用 | 绑定路径参数与处理器方法形参间的关系，要求路径参数名与形参名一一对应 |

关于接收参数，我们学过三个注解`@RequestBody`、`@RequestParam`、`@PathVariable`,这三个注解之间的区别和应用分别是什么?

* 区别
  * @RequestParam用于接收url地址传参或表单传参
  * @RequestBody用于接收json数据
  * @PathVariable用于接收路径参数，使用{参数名称}描述路径参数
* 应用
  * 后期开发中，发送请求参数超过1个时，以json格式为主，@RequestBody应用较广
  * 如果发送非json格式数据，选用@RequestParam接收请求参数
  * 采用RESTful进行开发，当参数数量较少时，例如1个，可以采用@PathVariable接收请求路径变量，通常用于传递id值



#### RESTful快速开发

前面的RESTful有些麻烦,体现在下面:

问题1：每个方法的@RequestMapping注解中都定义了访问路径/books，重复性太高。

问题2：每个方法的@RequestMapping注解中都要使用method属性定义请求方式，重复性太高。

问题3：每个方法响应json都需要加上@ResponseBody注解，重复性太高。

对于上面所提的这三个问题，具体该如何解决?

```java
@RestController //@Controller + ReponseBody
@RequestMapping("/books")
public class BookController {
    
	//@RequestMapping(method = RequestMethod.POST)
    @PostMapping
    public String save(@RequestBody Book book){
        System.out.println("book save..." + book);
        return "{'module':'book save'}";
    }

    //@RequestMapping(value = "/{id}",method = RequestMethod.DELETE)
    @DeleteMapping("/{id}")
    public String delete(@PathVariable Integer id){
        System.out.println("book delete..." + id);
        return "{'module':'book delete'}";
    }

    //@RequestMapping(method = RequestMethod.PUT)
    @PutMapping
    public String update(@RequestBody Book book){
        System.out.println("book update..." + book);
        return "{'module':'book update'}";
    }

    //@RequestMapping(value = "/{id}",method = RequestMethod.GET)
    @GetMapping("/{id}")
    public String getById(@PathVariable Integer id){
        System.out.println("book getById..." + id);
        return "{'module':'book getById'}";
    }

    //@RequestMapping(method = RequestMethod.GET)
    @GetMapping
    public String getAll(){
        System.out.println("book getAll...");
        return "{'module':'book getAll'}";
    }
    
}
```

对于刚才的问题，我们都有对应的解决方案：

问题1：每个方法的@RequestMapping注解中都定义了访问路径/books，重复性太高。

```
将@RequestMapping提到类上面，用来定义所有方法共同的访问路径。
```

问题2：每个方法的@RequestMapping注解中都要使用method属性定义请求方式，重复性太高。

```
使用@GetMapping  @PostMapping  @PutMapping  @DeleteMapping代替
```

问题3：每个方法响应json都需要加上@ResponseBody注解，重复性太高。

```
1.将ResponseBody提到类上面，让所有的方法都有@ResponseBody的功能
2.使用@RestController注解替换@Controller与@ResponseBody注解，简化书写
```

##### 知识点1：@RestController

| 名称 | @RestController                                              |
| ---- | ------------------------------------------------------------ |
| 类型 | ==类注解==                                                   |
| 位置 | 基于SpringMVC的RESTful开发控制器类定义上方                   |
| 作用 | 设置当前控制器类为RESTful风格，<br/>等同于@Controller与@ResponseBody两个注解组合功能 |

##### 知识点2：@GetMapping @PostMapping @PutMapping @DeleteMapping

| 名称     | @GetMapping @PostMapping @PutMapping @DeleteMapping          |
| -------- | ------------------------------------------------------------ |
| 类型     | ==方法注解==                                                 |
| 位置     | 基于SpringMVC的RESTful开发控制器方法定义上方                 |
| 作用     | 设置当前控制器方法请求访问路径与请求动作，每种对应一个请求动作，<br/>例如@GetMapping对应GET请求 |
| 相关属性 | value（默认）：请求访问路径                                  |



如果项目有静态资源,而配置文件中

```java
//设置由springmvc控制器处理的请求映射路径
    protected String[] getServletMappings() {
        return new String[]{"/"};//配置了'/',静态资源符合这个规则
    }
```

那么SpringMVC就会拦截静态资源.

解决方案?

* SpringMVC需要将静态资源进行放行。

```java
@Configuration
public class SpringMvcSupport extends WebMvcConfigurationSupport {
    //设置静态资源访问过滤，当前类需要设置为配置类，并被扫描加载
    @Override
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {
        //当访问/pages/????时候，从/pages目录下查找内容
        registry.addResourceHandler("/pages/**").addResourceLocations("/pages/");
        registry.addResourceHandler("/js/**").addResourceLocations("/js/");
        registry.addResourceHandler("/css/**").addResourceLocations("/css/");
        registry.addResourceHandler("/plugins/**").addResourceLocations("/plugins/");
    }
}

```

* 该配置类是在config目录下，SpringMVC扫描的是controller包，所以该配置类还未生效，要想生效需要将SpringMvcConfig配置类进行修改

```java
@Configuration
@ComponentScan({"com.itheima.controller","com.itheima.config"})
@EnableWebMvc
public class SpringMvcConfig {
}

或者

@Configuration
@ComponentScan("com.itheima")
@EnableWebMvc
public class SpringMvcConfig {
}
```



### 统一结果封装

我们在Controller层中有许多数据要返回给前端,对于前端开发人员在解析数据的时候就比较凌乱了，所以对于前端来说，如果后台能够返回一个统一的数据结果，前端在解析的时候就可以按照一种方式进行解析。开发就会变得更加简单。

能不能将返回结果的数据进行统一，具体如何来做，大体的思路为:

* 为了封装返回的结果数据:==创建结果模型类，封装数据到data属性中==
* 为了封装返回的数据是何种操作及是否操作成功:==封装操作结果到code属性中==
* 操作失败后为了封装返回的错误信息:==封装特殊消息到message(msg)属性中==

根据分析，我们可以设置统一数据返回结果类

```java
public class Result{
	private Object data;
	private Integer code;
	private String msg;
}
```

**注意:**Result类名及类中的字段并不是固定的，可以根据需要自行增减提供若干个构造方法，方便操作。

#### 例子

##### 1.创建Result类

```java
public class Result {
    //描述统一格式中的数据
    private Object data;
    //描述统一格式中的编码，用于区分操作，可以简化配置0或1表示成功失败
    private Integer code;
    //描述统一格式中的消息，可选属性
    private String msg;

    public Result() {
    }
	//构造方法是方便对象的创建
    public Result(Integer code,Object data) {
        this.data = data;
        this.code = code;
    }
	//构造方法是方便对象的创建
    public Result(Integer code, Object data, String msg) {
        this.data = data;
        this.code = code;
        this.msg = msg;
    }
	//setter...getter...省略
}
```

##### 2.定义返回码Code类

````java
//状态码
public class Code {
    public static final Integer SAVE_OK = 20011;
    public static final Integer DELETE_OK = 20021;
    public static final Integer UPDATE_OK = 20031;
    public static final Integer GET_OK = 20041;

    public static final Integer SAVE_ERR = 20010;
    public static final Integer DELETE_ERR = 20020;
    public static final Integer UPDATE_ERR = 20030;
    public static final Integer GET_ERR = 20040;
}

````

**注意:**code类中的常量设计也不是固定的，可以根据需要自行增减，例如将查询再进行细分为GET_OK,GET_ALL_OK,GET_PAGE_OK等。

##### 3.修改Controller类的返回值

```java
//统一每一个控制器方法返回值
@RestController
@RequestMapping("/books")
public class BookController {

    @Autowired
    private BookService bookService;

    @PostMapping
    public Result save(@RequestBody Book book) {
        boolean flag = bookService.save(book);
        return new Result(flag ? Code.SAVE_OK:Code.SAVE_ERR,flag);
    }

    @PutMapping
    public Result update(@RequestBody Book book) {
        boolean flag = bookService.update(book);
        return new Result(flag ? Code.UPDATE_OK:Code.UPDATE_ERR,flag);
    }

    @DeleteMapping("/{id}")
    public Result delete(@PathVariable Integer id) {
        boolean flag = bookService.delete(id);
        return new Result(flag ? Code.DELETE_OK:Code.DELETE_ERR,flag);
    }

    @GetMapping("/{id}")
    public Result getById(@PathVariable Integer id) {
        Book book = bookService.getById(id);
        Integer code = book != null ? Code.GET_OK : Code.GET_ERR;
        String msg = book != null ? "" : "数据查询失败，请重试！";
        return new Result(code,book,msg);
    }

    @GetMapping
    public Result getAll() {
        List<Book> bookList = bookService.getAll();
        Integer code = bookList != null ? Code.GET_OK : Code.GET_ERR;
        String msg = bookList != null ? "" : "数据查询失败，请重试！";
        return new Result(code,bookList,msg);
    }
}
```



### 统一异常处理

如果用户发送请求,结果出现异常时,前端会收到复杂的异常信息,与之前我们约定的格式不一致，这个问题该如何解决?

在解决问题之前，我们先来看下异常的种类及出现异常的原因:

- 框架内部抛出的异常：因使用不合规导致
- 数据层抛出的异常：因外部服务器故障导致（例如：服务器访问超时）
- 业务层抛出的异常：因业务逻辑书写错误导致（例如：遍历业务书写操作，导致索引异常等）
- 表现层抛出的异常：因数据收集、校验等规则导致（例如：不匹配的数据类型间导致异常）
- 工具类抛出的异常：因工具类书写不严谨不够健壮导致（例如：必要释放的连接长期未释放等）

看完上面这些出现异常的位置，你会发现，在我们开发的任何一个位置都有可能出现异常，而且这些异常是不能避免的。所以我们就得将异常进行处理。

**思考**

1. 各个层级均出现异常，异常处理代码书写在哪一层?

   ==所有的异常均抛出到表现层进行处理==

2. 异常的种类很多，表现层如何将所有的异常都处理到呢?

   ==异常分类==

3. 表现层处理异常，每个方法中单独书写，代码书写量巨大且意义不强，如何解决?

   ==AOP==

对于上面这些问题及解决方案，SpringMVC已经为我们提供了一套解决方案:

* 异常处理器:

  * 集中的、统一的处理项目中出现的异常。

    ![1630657791653](D:\BaiduNetdiskDownload\基础框架8笔记\SpringMVC笔记\SpringMVC_day02\assets\1630657791653.png)

#### 异常处理器

##### 1.创建异常处理器类

````java
//@RestControllerAdvice用于标识当前类为REST风格对应的异常处理器
@RestControllerAdvice
public class ProjectExceptionAdvice {
    //除了自定义的异常处理器，保留对Exception类型的异常处理，用于处理非预期的异常
    @ExceptionHandler(Exception.class)
    public void doException(Exception ex){
      	System.out.println("嘿嘿,异常你哪里跑！")
        return new Result(666,null,"嘿嘿,异常你哪里跑！");
    }
}
````

==确保SpringMvcConfig能够扫描到异常处理器类==

这样就算后台执行的过程中抛出异常，最终也能按照我们和前端约定好的格式返回给前端。



##### 知识点1：@RestControllerAdvice

| 名称 | @RestControllerAdvice              |
| ---- | ---------------------------------- |
| 类型 | ==类注解==                         |
| 位置 | Rest风格开发的控制器增强类定义上方 |
| 作用 | 为Rest风格开发的控制器类做增强     |

**说明:**此注解自带@ResponseBody注解与@Component注解，具备对应的功能

![1630659060451](D:\BaiduNetdiskDownload\基础框架8笔记\SpringMVC笔记\SpringMVC_day02\assets\1630659060451.png)

##### 知识点2：@ExceptionHandler

| 名称 | @ExceptionHandler                                            |
| ---- | ------------------------------------------------------------ |
| 类型 | ==方法注解==                                                 |
| 位置 | 专用于异常处理的控制器方法上方                               |
| 作用 | 设置指定异常的处理方案，功能等同于控制器方法，<br/>出现异常后终止原始控制器执行,并转入当前方法执行 |

**说明：**此类方法可以根据处理的异常不同，制作多个方法分别处理对应的异常



#### 项目异常处理方案

##### 异常分类

在项目中如何处理异常呢?因为异常的种类有很多，如果每一个异常都对应一个@ExceptionHandler，那得写多少个方法来处理各自的异常，所以我们在处理异常之前，需要对异常进行一个分类:

- 业务异常（BusinessException）

  - 规范的用户行为产生的异常

    - 用户在页面输入内容的时候未按照指定格式进行数据填写，如在年龄框输入的是字符串

      ![1630659599983](D:\BaiduNetdiskDownload\基础框架8笔记\SpringMVC笔记\SpringMVC_day02\assets\1630659599983.png)

  - 不规范的用户行为操作产生的异常

    - 如用户故意传递错误数据

      ![1630659622958](D:\BaiduNetdiskDownload\基础框架8笔记\SpringMVC笔记\SpringMVC_day02\assets\1630659622958.png)

- 系统异常（SystemException）

  - 项目运行过程中可预计但无法避免的异常
    - 比如数据库或服务器宕机

- 其他异常（Exception）

  - 编程人员未预期到的异常，如:用到的文件不存在

    ![1630659690341](D:\BaiduNetdiskDownload\基础框架8笔记\SpringMVC笔记\SpringMVC_day02\assets\1630659690341.png)

将异常分类以后，针对不同类型的异常，要提供具体的解决方案:

##### 异常解决方案

- 业务异常（BusinessException）
  - 发送对应消息传递给用户，提醒规范操作
    - 大家常见的就是提示用户名已存在或密码格式不正确等
- 系统异常（SystemException）
  - 发送固定消息传递给用户，安抚用户
    - 系统繁忙，请稍后再试
    - 系统正在维护升级，请稍后再试
    - 系统出问题，请联系系统管理员等
  - 发送特定消息给运维人员，提醒维护
    - 可以发送短信、邮箱或者是公司内部通信软件
  - 记录日志
    - 发消息和记录日志对用户来说是不可见的，属于后台程序
- 其他异常（Exception）
  - 发送固定消息传递给用户，安抚用户
  - 发送特定消息给编程人员，提醒维护（纳入预期范围内）
    - 一般是程序没有考虑全，比如未做非空校验等
  - 记录日志



##### 异常解决方案的具体实现

> 思路:
>
> 1.先通过自定义异常，完成BusinessException和SystemException的定义
>
> 2.将其他异常包装成自定义异常类型
>
> 3.在异常处理器类中对不同的异常进行处理

1. 自定义异常类

   ```java
   //自定义异常处理器，用于封装异常信息，对异常进行分类
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
   
   //自定义异常处理器，用于封装异常信息，对异常进行分类
   public class BusinessException extends RuntimeException{
       private Integer code;
   
       public Integer getCode() {
           return code;
       }
   
       public void setCode(Integer code) {
           this.code = code;
       }
   
       public BusinessException(Integer code, String message) {
           super(message);
           this.code = code;
       }
   
       public BusinessException(Integer code, String message, Throwable cause) {
           super(message, cause);
           this.code = code;
       }
   
   }
   ```

   **说明:**

   * 让自定义异常类继承`RuntimeException`的好处是，后期在抛出这两个异常的时候，就不用在try...catch...或throws了
   * 自定义异常类中添加`code`属性的原因是为了更好的区分异常是来自哪个业务的

2. 将其他异常包成自定义异常

假如在BookServiceImpl的getById方法抛异常了，该如何来包装呢?

```java
public Book getById(Integer id) {
    //模拟业务异常，包装成自定义异常
    if(id == 1){
        throw new BusinessException(Code.BUSINESS_ERR,"请不要使用你的技术挑战我的耐性!");
    }
    //模拟系统异常，将可能出现的异常进行包装，转换成自定义异常
    try{
        int i = 1/0;
    }catch (Exception e){
        throw new SystemException(Code.SYSTEM_TIMEOUT_ERR,"服务器访问超时，请重试!",e);
    }
    return bookDao.getById(id);
}
```

具体的包装方式有：

* 方式一:`try{}catch(){}`在catch中重新throw我们自定义异常即可。
* 方式二:直接throw自定义异常即可

上面为了使`code`看着更专业些，我们在Code类中再新增需要的属性

```java
//状态码
public class Code {
    public static final Integer SAVE_OK = 20011;
    public static final Integer DELETE_OK = 20021;
    public static final Integer UPDATE_OK = 20031;
    public static final Integer GET_OK = 20041;

    public static final Integer SAVE_ERR = 20010;
    public static final Integer DELETE_ERR = 20020;
    public static final Integer UPDATE_ERR = 20030;
    public static final Integer GET_ERR = 20040;
    public static final Integer SYSTEM_ERR = 50001;
    public static final Integer SYSTEM_TIMEOUT_ERR = 50002;
    public static final Integer SYSTEM_UNKNOW_ERR = 59999;

    public static final Integer BUSINESS_ERR = 60002;
}

```

##### 步骤3:处理器类中处理自定义异常

```java
//@RestControllerAdvice用于标识当前类为REST风格对应的异常处理器
@RestControllerAdvice
public class ProjectExceptionAdvice {
    //@ExceptionHandler用于设置当前处理器类对应的异常类型
    @ExceptionHandler(SystemException.class)
    public Result doSystemException(SystemException ex){
        //记录日志
        //发送消息给运维
        //发送邮件给开发人员,ex对象发送给开发人员
        return new Result(ex.getCode(),null,ex.getMessage());
    }

    @ExceptionHandler(BusinessException.class)
    public Result doBusinessException(BusinessException ex){
        return new Result(ex.getCode(),null,ex.getMessage());
    }

    //除了自定义的异常处理器，保留对Exception类型的异常处理，用于处理非预期的异常
    @ExceptionHandler(Exception.class)
    public Result doOtherException(Exception ex){
        //记录日志
        //发送消息给运维
        //发送邮件给开发人员,ex对象发送给开发人员
        return new Result(Code.SYSTEM_UNKNOW_ERR,null,"系统繁忙，请稍后再试！");
    }
}
```

小结:

以后项目中的异常处理方式为:

![1630658821746](D:\BaiduNetdiskDownload\基础框架8笔记\SpringMVC笔记\SpringMVC_day02\assets\1630658821746.png)



### 拦截器

* 拦截器（Interceptor）是一种动态拦截方法调用的机制，在SpringMVC中动态拦截控制器方法的执行
* 作用:
  * 在指定的方法调用前后执行预先设定的代码
  * 阻止原始方法的执行
  * 总结：拦截器就是用来做增强

#### 拦截器案例

1. 创建拦截器类

   让类实现HandlerInterceptor接口，重写接口中的三个方法。

   ```java
   @Component
   //定义拦截器类，实现HandlerInterceptor接口
   //注意当前类必须受Spring容器控制
   public class ProjectInterceptor implements HandlerInterceptor {
       @Override
       //原始方法调用前执行的内容
       public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
           System.out.println("preHandle...");
           return true;
       }
   
       @Override
       //原始方法调用后执行的内容
       public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
           System.out.println("postHandle...");
       }
   
       @Override
       //原始方法调用完成后执行的内容
       public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
           System.out.println("afterCompletion...");
       }
   }
   ```

   **注意:**拦截器类要被SpringMVC容器扫描到。

2. 配置拦截器类

   ```java
   @Configuration
   public class SpringMvcSupport extends WebMvcConfigurationSupport {
       @Autowired
       private ProjectInterceptor projectInterceptor;
   
       @Override
       protected void addResourceHandlers(ResourceHandlerRegistry registry) {
           registry.addResourceHandler("/pages/**").addResourceLocations("/pages/");
       }
   
       @Override
       protected void addInterceptors(InterceptorRegistry registry) {
           //配置拦截器
           registry.addInterceptor(projectInterceptor).addPathPatterns("/books" );
       }
   }
   ```

3. SpringMVC添加SpringMvcSupport包扫描

   ```java
   @Configuration
   @ComponentScan({"com.itheima.controller","com.itheima.config"})
   @EnableWebMvc
   public class SpringMvcConfig{
      
   }
   ```

   > 注意,这个例子可以拦截`/books`路径的内容,但是`/books/100`这种就不行了,需要把配置拦截器的内容改成`registry.addInterceptor(projectInterceptor).addPathPatterns("/books","/books/*" );`

==拦截器中的`preHandler`方法，如果返回true,则代表放行，会执行原始Controller类中要请求的方法，如果返回false，则代表拦截，后面的就不会再执行了。==

4. 可以简化SpringMvcSupport的编写

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
           //配置多拦截器
           registry.addInterceptor(projectInterceptor).addPathPatterns("/books","/books/*");
       }
   }
   ```

   这样就不用写`SpringMvcSupport`类了



#### 拦截器参数

##### 前置处理方法

原始方法之前运行preHandle

```java
public boolean preHandle(HttpServletRequest request,
                         HttpServletResponse response,
                         Object handler) throws Exception {
    System.out.println("preHandle");
    return true;
}
```

* `request`:请求对象
* `response`:响应对象
* `handler`:被调用的处理器对象，本质上是一个方法对象，对反射中的`Method`对象进行了再包装

使用request对象可以获取请求数据中的内容，如获取请求头的`Content-Type`

````java
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    String contentType = request.getHeader("Content-Type");
    System.out.println("preHandle..."+contentType);
    return true;
}
````

使用handler参数，可以获取方法的相关信息

```java
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    HandlerMethod hm = (HandlerMethod)handler;
    String methodName = hm.getMethod().getName();//可以获取方法的名称
    System.out.println("preHandle..."+methodName);
    return true;
}
```

##### 后置处理方法

原始方法运行后运行，如果原始方法被拦截，则不执行  

```java
public void postHandle(HttpServletRequest request,
                       HttpServletResponse response,
                       Object handler,
                       ModelAndView modelAndView) throws Exception {
    System.out.println("postHandle");
}
```

前三个参数和上面的是一致的。多了个参数:

- `modelAndView`:如果处理器执行完成具有返回结果，可以读取到对应数据与页面信息，并进行调整

  因为咱们现在都是返回json数据，所以该参数的使用率不高。

##### 完成处理方法



```java
public void afterCompletion(HttpServletRequest request,
                            HttpServletResponse response,
                            Object handler,
                            Exception ex) throws Exception {
    System.out.println("afterCompletion");
}
```

前三个参数与上面的是一致的。

- `ex`:如果处理器执行过程中出现异常对象，可以针对异常情况进行单独处理  

  因为我们现在已经有全局异常处理器类，所以该参数的使用率也不高。



这三个方法中，最常用的是==preHandle==,在这个方法中可以通过返回值来决定是否要进行放行，我们可以把业务逻辑放在该方法中，如果满足业务则返回true放行，不满足则返回false拦截。



#### 拦截器链

##### 配置成多个拦截器

1. 在前一个拦截器类的基础上,再写一个,实现接口，并重写接口中的方法

   ```java
   @Component
   public class ProjectInterceptor2 implements HandlerInterceptor {
       @Override
       public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
           System.out.println("preHandle...222");
           return false;
       }
   
       @Override
       public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
           System.out.println("postHandle...222");
       }
   
       @Override
       public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
           System.out.println("afterCompletion...222");
       }
   }
   ```

2. 配置拦截器类

   ``````java
   @Configuration
   @ComponentScan({"com.itheima.controller"})
   @EnableWebMvc
   //实现WebMvcConfigurer接口可以简化开发，但具有一定的侵入性
   public class SpringMvcConfig implements WebMvcConfigurer {
       @Autowired
       private ProjectInterceptor projectInterceptor;
       @Autowired
       private ProjectInterceptor2 projectInterceptor2;
   
       @Override
       public void addInterceptors(InterceptorRegistry registry) {
           //配置多拦截器
           registry.addInterceptor(projectInterceptor).addPathPatterns("/books","/books/*");
           registry.addInterceptor(projectInterceptor2).addPathPatterns("/books","/books/*");
       }
   }
   ``````

   

拦截器执行的顺序是和配置顺序有关,先进后出:

* 当配置多个拦截器时，形成拦截器链
* 拦截器链的运行顺序参照拦截器添加顺序为准
* 当拦截器中出现对原始处理器的拦截，后面的拦截器均终止运行
* 当拦截器运行中断，仅运行配置在前面的拦截器的afterCompletion操作

![image-20240313211738639](C:\Users\86139\AppData\Roaming\Typora\typora-user-images\image-20240313211738639.png)

preHandle：与配置顺序相同，必定运行

postHandle:与配置顺序相反，可能不运行

afterCompletion:与配置顺序相反，可能不运行。

这个顺序不太好记，最终只需要把握住一个原则即可:==以最终的运行结果为准==
