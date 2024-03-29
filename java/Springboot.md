# Springboot

## 整合SSMP项目



### 实体类开发(使用Lombok)

实体类的开发写`get/set`方法太过繁琐,可以使用lombok技术--一个Java类库，提供了一组注解，简化POJO实体类开发，SpringBoot目前默认集成了lombok技术，并提供了对应的版本控制，所以只需要提供对应的坐标即可，在pom.xml中添加lombok的坐标。

```XML
<dependencies>
    <!--lombok-->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
</dependencies>
```

> 实测项目为`jdk21`时直接用springboot提供的版本会有bug,把lombok版本改为`1.18.30`即可

使用lombok可以通过一个注解@Data完成一个实体类对应的`getter`，`setter`，`toString`，`equals`，`hashCode`等操作的快速添加

```java
import lombok.Data;
@Data
public class Book {
    private Integer id;
    private String type;
    private String name;
    private String description;
}
```



### 数据层开发(使用Mybatisplus,数据源使用druid)

1. 导入Mybatis和Druid的starter以及mysql驱动

   ```xml
   <dependencies>
       <dependency>
           <groupId>com.baomidou</groupId>
           <artifactId>mybatis-plus-boot-starter</artifactId>
           <version>3.4.3</version>
       </dependency>
       <dependency>
           <groupId>com.alibaba</groupId>
           <artifactId>druid-spring-boot-starter</artifactId>
           <version>1.2.6</version>
       </dependency>
       <dependency>
           <groupId>mysql</groupId>
           <artifactId>mysql-connector-java</artifactId>
           <scope>runtime</scope>
       </dependency>
   </dependencies>
   ```

2. dao接口(继承mp的BaseMapper加速开发)

   ```java
   @Mapper
   public interface BookDao extends BaseMapper<Book> {
       
   }
   ```

   继承mp的BaseMapper可以不用写每个crud的sql语句,自动就生成对应方法了.

3. 配置

   ```yaml
   server:
     port: 80
   
   spring:
     datasource:
       druid:
         driver-class-name: com.mysql.cj.jdbc.Driver
         url: jdbc:mysql://localhost:3306/ssm_db?serverTimezone=UTC
         username: root
         password: root
   
   mybatis-plus:
     global-config:
       db-config:
         table-prefix: tbl_		#设置表名通用前缀
         id-type: auto				#设置主键id字段的生成策略为参照数据库设定的策略，当前数据库设置id生成策略为自增
         configuration:
       log-impl: org.apache.ibatis.logging.stdout.StdOutImpl      #设置显示运行时日志
   ```

   `table-prefix`为设置表名前缀,因为表的名字是`tbl_book`,而实体类名字是`Book`,以此类推,为了让他们对应上就要加上这个.
   `id-type`是设置主键id生成策略,因为mp默认的是雪花算法,与我们表的自增不符合,所以要改成`auto`即自增

   

   

#### 分页功能

MP同时也给我们提供了分页功能,API如下:

```java
@Test
void testGetPage(){
    IPage page = new Page(2,5);
    bookDao.selectPage(page, null);
    System.out.println(page.getCurrent());
    System.out.println(page.getSize());
    System.out.println(page.getTotal());
    System.out.println(page.getPages());
    System.out.println(page.getRecords());
}
```

其中selectPage方法需要传入一个封装分页数据的对象，可以通过new的形式创建这个对象，当然这个对象也是MP提供的，别选错包了。创建此对象时就需要指定分页的两个基本数据

- 当前显示第几页
- 每页显示几条数据

​    可以通过创建Page对象时利用构造方法初始化这两个数据

```JAVA
IPage page = new Page(2,5);
```

​	将该对象传入到查询方法selectPage后，可以得到查询结果，但是我们会发现当前操作查询结果返回值仍然是一个IPage对象，这又是怎么回事？

```JAVA
IPage page = bookDao.selectPage(page, null);
```

​	原来这个IPage对象中封装了若干个数据，而查询的结果作为IPage对象封装的一个数据存在的，可以理解为查询结果得到后，又塞到了这个IPage对象中，其实还是为了高度的封装，一个IPage描述了分页所有的信息。下面5个操作就是IPage对象中封装的所有信息了

```JAVA
@Test
void testGetPage(){
    IPage page = new Page(2,5);
    bookDao.selectPage(page, null);
    System.out.println(page.getCurrent());		//当前页码值
    System.out.println(page.getSize());			//每页显示数
    System.out.println(page.getTotal());		//数据总量
    System.out.println(page.getPages());		//总页数
    System.out.println(page.getRecords());		//详细数据
}
```

但是直接如此操作还是不行,因为MySQL的分页操作使用limit关键字进行，而并不是所有的数据库都使用limit关键字实现的，这个时候MP为了制作的兼容性强，将分页操作设置为基础查询操作的升级版.

基础操作中有查询全部的功能，而在这个基础上只需要升级一下（PLUS）就可以得到分页操作。所以MP将分页操作做成了一个开关，你用分页功能就把开关开启，不用就不需要开启这个开关。而我们现在没有开启这个开关，所以分页操作是没有的。这个开关是通过MP的拦截器的形式存在的

**定义MP拦截器并将其设置为Spring管控的bean**

```JAVA
@Configuration
public class MPConfig {
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor(){
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor());
        return interceptor;
    }
}
```

上述代码第一行是创建MP的拦截器栈，这个时候拦截器栈中没有具体的拦截器，第二行是初始化了分页拦截器，并添加到拦截器栈中。如果后期开发其他功能，需要添加全新的拦截器，按照第二行的格式继续add进去新的拦截器就可以了。

这时候再运行分页就可以了.



#### 条件查询功能

除了分页功能，MP还提供有强大的条件查询功能。以往我们写条件查询要自己动态拼写复杂的SQL语句，现在简单了，MP将这些操作都制作成API接口，调用一个又一个的方法就可以实现各种套件的拼装。

下面的操作就是执行一个模糊匹配对应的操作，由like条件书写变为了like方法的调用

```JAVA
@Test
void testGetBy(){
    QueryWrapper<Book> qw = new QueryWrapper<>();
    qw.like("name","Spring");
    bookDao.selectList(qw);
}
```

​	其中第一句QueryWrapper对象是一个用于封装查询条件的对象，该对象可以动态使用API调用的方法添加条件，最终转化成对应的SQL语句。第二句就是一个条件了，需要什么条件，使用QueryWapper对象直接调用对应操作即可。比如做大于小于关系，就可以使用lt或gt方法，等于使用eq方法，等等，此处不做更多的解释了。

**但是这样仍然存在隐患**,比如查询字段name，当前是以字符串的形态书写的，万一写错，编译器还没有办法发现，只能将问题抛到运行器通过异常堆栈告诉开发者，不太友好。

MP针对字段检查进行了功能升级，全面支持Lambda表达式，就有了下面这组API。由QueryWrapper对象升级为LambdaQueryWrapper对象，这下就变了上述问题的出现

```JAVA
@Test
void testGetBy2(){
    String name = "1";
    LambdaQueryWrapper<Book> lqw = new LambdaQueryWrapper<Book>();
    lqw.like(Book::getName,name);
    bookDao.selectList(lqw);
}
```

​	为了便于开发者动态拼写SQL，防止将null数据作为条件使用，MP还提供了动态拼装SQL的快捷书写方式

```JAVA
@Test
void testGetBy2(){
    String name = "1";
    LambdaQueryWrapper<Book> lqw = new LambdaQueryWrapper<Book>();
    //方式一：JAVA代码控制
    //if(name != null) lqw.like(Book::getName,name);
    //方式二：API接口提供控制开关
    lqw.like(name != null,Book::getName,name);				
    bookDao.selectList(lqw);
}
```

### 业务层开发

其实标准业务层开发很多初学者认为就是调用数据层，怎么说呢？这个理解是没有大问题的，更精准的说法应该是<font color="#ff0000"><b>组织业务逻辑功能，并根据业务需求，对数据持久层发起调用</b></font>。有什么差别呢？目标是为了组织出符合需求的业务逻辑功能，至于调不调用数据层还真不好说，有需求就调用，没有需求就不调用。

一个常识性的知识普及一下，业务层的方法名定义一定要与业务有关，例如登录操作

```JAVA
login(String username,String password);
```

​	而数据层的方法名定义一定与业务无关，是一定，不是可能，也不是有可能，例如根据用户名密码查询

```JAVA
selectByUserNameAndPassword(String username,String password);
```

下面写的业务层案例就先简单一点.业务层接口:



```JAVA
public interface BookService {
    Boolean save(Book book);
    Boolean update(Book book);
    Boolean delete(Integer id);
    Book getById(Integer id);
    List<Book> getAll();
    IPage<Book> getPage(int currentPage,int pageSize);
}
```

​	业务层实现类如下，转调数据层即可

```JAVA
@Service
public class BookServiceImpl implements BookService {

    @Autowired
    private BookDao bookDao;

    @Override
    public Boolean save(Book book) {
        return bookDao.insert(book) > 0;
    }

    @Override
    public Boolean update(Book book) {
        return bookDao.updateById(book) > 0;
    }

    @Override
    public Boolean delete(Integer id) {
        return bookDao.deleteById(id) > 0;
    }

    @Override
    public Book getById(Integer id) {
        return bookDao.selectById(id);
    }

    @Override
    public List<Book> getAll() {
        return bookDao.selectList(null);
    }

    @Override
    public IPage<Book> getPage(int currentPage, int pageSize) {
        IPage page = new Page(currentPage,pageSize);
        bookDao.selectPage(page,null);
        return page;
    }
}
```

但是,这一部分都是简单的crud业务功能,太简单但也太麻烦,MP技术为我们提供的加速开发的功能.

#### MP业务层快速开发

其实MP技术不仅提供了数据层快速开发方案，业务层MP也给了一个通用接口，个人观点不推荐使用，凑合能用吧，其实就是一个封装+继承的思想，代码给出，实际开发慎用

​	业务层接口快速开发

```JAVA
public interface IBookService extends IService<Book> {
    //添加非通用操作API接口
}
```

​	业务层接口实现类快速开发，关注继承的类需要传入两个泛型，一个是数据层接口，另一个是实体类

```JAVA
@Service
public class BookServiceImpl extends ServiceImpl<BookDao, Book> implements IBookService {
    @Autowired
    private BookDao bookDao;
	//添加非通用操作API
}
```

​	如果感觉MP提供的功能不足以支撑你的使用需要，其实是一定不能支撑的，因为需求不可能是通用的，在原始接口基础上接着定义新的API接口就行了，此处不再说太多了，就是自定义自己的操作了，但是不要和已有的API接口名冲突即可。

**总结**

1. 使用通用接口（ISerivce<T>）快速开发Service
2. 使用通用实现类（ServiceImpl<M,T>）快速开发ServiceImpl
3. 可以在通用接口基础上做功能重载或功能追加
4. 注意重载时不要覆盖原始操作，避免原始提供的功能丢失



### 表现层开发

`````java
@RestController
@RequestMapping("/books")
public class BookController2 {

    @Autowired
    private IBookService bookService;

    @GetMapping
    public List<Book> getAll(){
        return bookService.list();
    }

    @PostMapping
    public Boolean save(@RequestBody Book book){
        return bookService.save(book);
    }

    @PutMapping
    public Boolean update(@RequestBody Book book){
        return bookService.modify(book);
    }

    @DeleteMapping("{id}")
    public Boolean delete(@PathVariable Integer id){
        return bookService.delete(id);
    }

    @GetMapping("{id}")
    public Book getById(@PathVariable Integer id){
        return bookService.getById(id);
    }

    @GetMapping("{currentPage}/{pageSize}")
    public IPage<Book> getPage(@PathVariable int currentPage,@PathVariable int pageSize){
        return bookService.getPage(currentPage,pageSize, null);
    }
}
`````

但是有一个问题,就是我们传给前端不同操作的结果数据格式都不同,比如:

- 增删改操作结果

  ```
  true
  ```

- 查询单个数据操作结果

  ```json
  {
      "id": 1,
      "type": "计算机理论",
      "name": "Spring实战 第5版",
      "description": "Spring入门经典教程"
  }
  ```

- 查询全部数据操作结果

  ````json
  [
      {
          "id": 1,
          "type": "计算机理论",
          "name": "Spring实战 第5版",
          "description": "Spring入门经典教程"
      },
      {
          "id": 2,
          "type": "计算机理论",
          "name": "Spring 5核心原理与30个类手写实战",
          "description": "十年沉淀之作"
      }
  ]
  ````

  每种不同操作返回的数据格式都不一样，而且还不知道以后还会有什么格式，这样的结果让前端人员看了是很容易让人崩溃的，必须将所有操作的操作结果数据格式统一起来，需要设计表现层返回结果的模型类，用于后端与前端进行数据格式统一，也称为**前后端数据协议**

  ```JAVA
  @Data
  public class R {
      private Boolean flag;
      private Object data;
  }
  ```

  ​	其中flag用于标识操作是否成功，data用于封装操作数据，现在的数据格式就变了

  ```json
  {
      "flag": true,
      "data":{
          "id": 1,
          "type": "计算机理论",
          "name": "Spring实战 第5版",
          "description": "Spring入门经典教程"
      }
  }
  ```

  这样表现层都改成 `return new R(flag)`类似的

### 业务消息一致性处理

我们目前只处理了正常使用的情况,但如果出现bug就坏事了.比如后台抛出异常,前端就会接收这样一个数据:

```json
{
    "timestamp": "2021-09-15T03:27:31.038+00:00",
    "status": 500,
    "error": "Internal Server Error",
    "path": "/books"
}
```

这样又跟之前的数据格式不一样了,看来不仅要对正确的操作数据格式做处理，还要对错误的操作数据格式做同样的格式处理

​	首先在当前的数据结果中添加消息字段，用来兼容后台出现的操作消息ss

```JAVA
@Data
public class R{
    private Boolean flag;
    private Object data;
    private String msg;		//用于封装消息
}
```

​	后台代码也要根据情况做处理，当前是模拟的错误

```JAVA
@PostMapping
public R save(@RequestBody Book book) throws IOException {
    Boolean flag = bookService.insert(book);
    return new R(flag , flag ? "添加成功^_^" : "添加失败-_-!");
}
```

​	然后在表现层做统一的异常处理，使用SpringMVC提供的异常处理器做统一的异常处理

```java
@RestControllerAdvice
public class ProjectExceptionAdvice {
    @ExceptionHandler(Exception.class)
    public R doOtherException(Exception ex){
        //记录日志
        //发送消息给运维
        //发送邮件给开发人员,ex对象发送给开发人员
        ex.printStackTrace();
        return new R(false,null,"系统错误，请稍后再试！");
    }
}
```

