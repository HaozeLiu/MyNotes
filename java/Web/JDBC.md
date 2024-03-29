# JDBC

## JDBC常规6步骤：

1. 注册驱动
2. 获取连接
3. 创建statement
4. 发送查询SQL语句，并获取查询结果
5. 关闭资源



## 静态SQL路线(没有动态语句)：

`DriverManager -> Connection -> Statement -> Result`

```java
import com.mysql.cj.jdbc.Driver;

import java.sql.*;

public class JDBC_test01 {
    public static void main(String[] args) throws SQLException {
//        1.注册驱动
        DriverManager.registerDriver(new Driver());
//        2.获取链接
        Connection connection = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/atguigu", "root", "123456");
//        3.创建statement
        Statement statement = connection.createStatement();
//        4.发送sql语句，并获取返回结果
        String sql = "select * from t_user;";
        ResultSet resultSet = statement.executeQuery(sql);
//        5.进行结果集解析
        while(resultSet.next()){
            int id = resultSet.getInt("id");
            String account = resultSet.getString("account");
            String password = resultSet.getString("password");
            String nickname = resultSet.getString("nickname");
            System.out.println(id + "--" + account + "--" + password + "--" + nickname);
//        6.关闭资源
            resultSet.close();
            statement.close();
            connection.close();
        }
    }
}
```





### 1.注册驱动：

`Class.forName("com.mysql.cj.jdbc.Driver");`



### 2.获取连接

使用方法：`getConnection`

核心属性：

1. 数据库软件所在的ip地址：localhost | 127.0.0.1
2. 数据库软件所在的主机的端口号：3306
3. 连接的具体库
4. 连接的账号
5. 连接的密码
6. 可选的信息

可选择传入三个参数，两个参数：

- 三个参数：
  - `String url` 数据库所在的信息，连接的具体库及其他
    语法：`jdbc:数据库管理软件名称[mysql,oracle...]://ip地址|主机名:port端口号/数据库名`
    具体：`jdbc:mysql://127.0.0.1:3306/atguigu`
    本机的省略写法(数据库安装在本机，且端口号为3306)：`jdbc:mysql:///atguigu`
  - `String user`
  - `String password`

```java
      Connection connection = DriverManager.getConnection("jdbc:mysql:///atguigu", "root", "123456");
```



- 两个参数：
  - `String url`
  - `Properties info`:存储账号和密码
    `Properties`类似于`Map`，只不过`key`，`value`都是字符串

```java
Properties info = new Properties();
        info.put("user", "root");
        info.put("password", "123456");
        Connection com = DriverManager.getConnection("jdbc:mysql://localhost:3306/atguigu", info);
```



- 一个参数：
  - `String url`：`jdbc:mysql://localhost:3306/atguigu ? user = root & password = 123456`



### 3.创建发送SQL语句的statement对象 并发送SQL语句

```java
Statement statement = connection.createStatement();

String sql = "SELECT * FROM t_user WHERE account = '" + account + "' AND PASSWORD = '" + password + "';";
```

发送SQL语句有两种方式：

1. `int i = statement.executeUpdate(sql);`

   参数：`sql` 非`DQL`语句即可

   > `SQL`分类：`DDL`(容器创建，修改，删除)，`DML`(插入，修改，删除)， `DQL`(查询)， `DCL`(权限控制)，`TCL`(事务控制语言)

​	   返回：int
​						1.若`DML`语句，则返回影响的行数
​						2.若非`DML`，则返回0

2. `ResultSet resultSet = statement.executeQuery(sql);`
   参数：`sql` 需要为`DQL`
   返回：`resultSet` 结果封装对象

```java
 while (resultSet.next()){
            int id = resultSet.getInt(1);
            String account1 = resultSet.getString("account");
            String password1 = resultSet.getString(3);
            String nickname = resultSet.getString("nickname");
            System.out.println(id + "--" + account1 + "--" + password1 + "--" + nickname);
        }
```



我们可以这样检查登录账号：

```java
public class UserLogin {
    public static void main(String args[]) throws ClassNotFoundException, SQLException {
//        获取用户输入信息
        Scanner scanner = new Scanner(System.in);
        System.out.println("请输入账号");
        String account = scanner.nextLine();
        System.out.println("请输入密码");
        String password = scanner.nextLine();

//        注册驱动
        Class.forName("com.mysql.cj.jdbc.Driver");

//        获取数据库连接
        Connection connection = DriverManager.getConnection("jdbc:mysql:///atguigu", "root", "123456");

//        创建statement对象
        Statement statement = connection.createStatement();
        String sql = "SELECT * FROM t_user WHERE account = '" + account + "' AND PASSWORD = '" + password + "';";

        ResultSet resultSet = statement.executeQuery(sql);
        
        if (resultSet.next())
            System.out.println("登录成功！");
        else System.out.println("登陆失败。。。");
        resultSet.close();
        statement.close();
        connection.close();
    }

}

```

但是会出现一个问题：**注入攻击**。如果密码输入`' or '1' = '1`，总是能登陆成功。
这是什么原理？把代码中sql语句这段和密码这里连起来就懂了：`.... AND PASSWORD = '' or '1' = '1'`,必定判断为`true`所以就会登陆成功。

基于`statement`实现模拟登录存在以下问题：

1. SQL语句需要拼接，比较麻烦

2. 只能拼接字符串类型，其他的数据库类型无法处理

3. 可能发生注入攻击

   > 动态值充当SQL语句结构，影响原有的查询结果。



## 预编译SQL路线(没有动态值语句): DriverManager -> Connection -> PreparedStatement -> Result

```java
package preparedstatement;

import com.mysql.cj.jdbc.Driver;

import java.sql.*;
import java.util.Scanner;

/*
* TODO 防止注入攻击
* */
public class PSUserLoginPart {
    public static void main(String args[]) throws ClassNotFoundException, SQLException {
//        获取用户输入信息
        Scanner scanner = new Scanner(System.in);
        System.out.println("请输入账号");
        String account = scanner.nextLine();
        System.out.println("请输入密码");
        String password = scanner.nextLine();

//        1.注册驱动
        Class.forName("com.mysql.cj.jdbc.Driver");
//
//        2.获取连接
        Connection connection = DriverManager.getConnection("jdbc:mysql:///atguigu?user=root&password=123456");

        /*
* statement
*   1.创建statement
*   2.拼接SQL语句
*   3.发送SQL语句，并且获得返回结果
*
* preparedStatement
*   1.编写SQL语句结构，不包含动态值部分的语句，动态值部分使用占位符 ? 替代。
*   2.创建preparedStatement，并传入动态值
*   3.动态值 占位符 赋值 ? 单独赋值
*   4.发送SQL语句，并获取返回结果
*/
//        3.编写SQL语句结构
        String sql = "select * from t_user where account = ? and password = ? ;";

//        4.创建预编译statement并且设置SQL语句结果
        PreparedStatement preparedStatement = connection.prepareStatement(sql);

//        5.单独为占位符赋值
        /*
        *两个参数：
        * 参数1：index 占位符的位置，从左往右从1开始
        * 参数2： object 占位符的值 可以设置任何类型的数据，避免了拼接并且类型更加丰富
        */
        preparedStatement.setObject(1,account);
        preparedStatement.setObject(2,password);

//        6.发送SQL语句，并获取结果。
        ResultSet resultSet = preparedStatement.executeQuery();

//        7.结果集解析：检查是否登录成功
        if (resultSet.next()) {
            System.out.println("登陆成功！");
        }
        else{
            System.out.println("登陆失败");
        }

//        8.关闭资源
        resultSet.close();
        preparedStatement.close();
        connection.close();
    }
}

```



PreparedStatement_CRUD:
```java
package preparedstatement;

import org.junit.Test;

import java.sql.*;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class PS_CRUD {
    @Test
    public void testInsert() throws ClassNotFoundException, SQLException {
        /*
        * TODO:t_user插入一条数据
        *   account test
        *   password test
        *   nickname 二狗子
        * */
//        1.注册驱动
        Class.forName("com.mysql.cj.jdbc.Driver");
//        2.获取连接
        Connection connection = DriverManager.getConnection("jdbc:mysql:///atguigu", "root", "123456");
//        3.编写SQL语句结构，动态值部分用 ? 代替
        String sql = "insert into t_user(account,password,nickname) values(?,?,?);";
//        4.创建preparedStatement，并传入SQl语句结果
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
//        5.占位符赋值
        preparedStatement.setObject(1,"test");
        preparedStatement.setObject(2,"test");
        preparedStatement.setObject(3,"二狗子");
//        6.发送SQL语句
//        DML类型
        int rows = preparedStatement.executeUpdate();
//        7.输出结果
        if(rows > 0)
            System.out.println("数据插入成功！");
        else System.out.println("数据插入失败。。。");

        preparedStatement.close();
        connection.close();
    }

    @Test
    public void testUpdate() throws ClassNotFoundException, SQLException {
        /*
         * TODO:修改 id = 3 的用户nickname = 三狗子
         * */
//        1.注册驱动
        Class.forName("com.mysql.cj.jdbc.Driver");
//        2.获取连接
        Connection connection = DriverManager.getConnection("jdbc:mysql:///atguigu", "root", "123456");
//        3.编写SQL语句结构，动态值部分用 ? 代替
        String sql = "update t_user set nickname = ? where id = ?;";
//        4.创建preparedStatement，并传入SQl语句结果
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
//        5.占位符赋值
        preparedStatement.setObject(1,"三狗子");
        preparedStatement.setObject(2, "3");
//        6.发送SQL语句
        int i = preparedStatement.executeUpdate();
//        7.输出结果
        if(i > 0)
            System.out.println("修改成功！");
        else System.out.println("修改失败。。。");

        preparedStatement.close();
        connection.close();
    }

    @Test
    public void testDelete() throws ClassNotFoundException, SQLException {
        /*
        * TODO:删除 id = 3 的数据
        *  */
//        1.注册驱动
        Class.forName("com.mysql.cj.jdbc.Driver");
//        2.获取连接
        Connection connection = DriverManager.getConnection("jdbc:mysql:///atguigu", "root", "123456");
//        3.编写SQL语句结构，动态值部分用 ? 代替
        String sql = "delete from t_user where id = ?";
//        4.创建preparedStatement，并传入SQl语句结果
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
//        5.占位符赋值
        preparedStatement.setObject(1,"3");
//        6.发送SQL语句
        int i = preparedStatement.executeUpdate();
//        7.输出结果
        if(i > 0)
            System.out.println("数据删除成功");
        else
            System.out.println("数据删除失败");
    }

    @Test
    public void testSelect() throws ClassNotFoundException, SQLException {
        /*
        * TODO:查询所有用户数据，并且封装到一个List<map> list集合中
        *
        * 数据库 -> resultSet -> map(key=列名,value=列的内容) -> List<map> list
        * */
//        1.注册驱动
        Class.forName("com.mysql.cj.jdbc.Driver");
//        2.获取连接
        Connection connection = DriverManager.getConnection("jdbc:mysql:///atguigu", "root", "123456");
//        3.编写SQL语句结构，动态值部分用 ? 代替
        String sql = "select id,account,password,nickname from t_user ;";
//        4.创建preparedStatement，并传入SQl语句结果
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
//        5.占位符赋值(没有占位符,直接省略)
//        6.发送SQL语句
        ResultSet resultSet = preparedStatement.executeQuery();
//        7.结果集解析
        List<Map> list = new ArrayList<>();

//        获取列的信息对象
        ResultSetMetaData metaData = resultSet.getMetaData();
        int columnCount = metaData.getColumnCount();

        while(resultSet.next()){
           Map map = new HashMap<>();

//         纯手动取值,不推荐.
//           map.put("id", resultSet.getInt("id"));
//           map.put("account", resultSet.getString("account"));
//           map.put("password", resultSet.getString("password"));
//           map.put("nickname", resultSet.getString("nickname"));

            for (int i = 1; i <= columnCount; i++) {
//                获取指定下角标的值 resultSet
                Object value = resultSet.getObject(i);
//                获取指定下角标的列名 ResultSetMetaData
                String columnLabel = metaData.getColumnLabel(i);
                map.put(columnLabel,value);
            }
            list.add(map);
        }

    }

}
```



## 主键回显

java程序获取插入数据时 mysql维护自增长维护的主键id值,这就是主键回显

作用:在多表关联插入数据时,一般主表的主键都是自动生成的,所以在插入数据之前无法知道这条数据的主键,但是需要在插入数据之前就绑定主表的主键,这时可以用主键回显技术.

```java
package preparedstatement;

import org.junit.Test;

import java.sql.*;

public class PSOtherPart {

    /*
    * TODO:
    *   t_user插入一条数据,并且获取数据库自增长的主键
    * 1.创建preparedStatement时,告知,携带回数据库自增长的主键 (sql,Statement.RETURN_GENERATED_KEYS)
    * 2.获取司机装主键值的结果集对象,一行一列,获取对应的数据即可 ResultSet resultSet = preparedStatement.getGeneratedKeys();
    * */
    @Test
    public void returnPrimaryKey() throws ClassNotFoundException, SQLException {
//    1.注册驱动
        Class.forName("com.mysql.cj.jdbc.Driver");
//    2.获取连接
        Connection connection = DriverManager.getConnection("jdbc:mysql:///atguigu?user=root&password=123456");
//    3.编写SQL语句
        String sql = "insert into t_user(account,password,nickname) values(?,?,?);";
//    4.创建Statement
        PreparedStatement preparedStatement = connection.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS);//传的时候告诉司机,去的时候别忘了把主键传回来
//    5.占位符赋值
        preparedStatement.setObject(1, "test1");
        preparedStatement.setObject(2, "123456");
        preparedStatement.setObject(3, "驴蛋蛋");
//    6.发送SQL语句,并获得结果
        int i = preparedStatement.executeUpdate();
//    7.结果解析
        if(i > 0) {
            System.out.println("数据插入成功!");

//            可以获取回显的主键
            ResultSet resultSet = preparedStatement.getGeneratedKeys();
            resultSet.next();
            int id = resultSet.getInt(1);
            System.out.println("id = " + id);
        }else {
            System.out.println("数据插入失败");
        }
//    8.关闭资源
        preparedStatement.close();
        connection.close();
    }
}
```



## 批量插入数据优化

使用普通方式插入10000条数据

```java
package preparedstatement;

import org.junit.Test;

import java.sql.*;

public class PSOtherPart {
/*使用普通的方式插入10000条数据
* */
    @Test
    public void testInsert() throws ClassNotFoundException, SQLException {
//    1.注册驱动
        Class.forName("com.mysql.cj.jdbc.Driver");
//    2.获取连接
        Connection connection = DriverManager.getConnection("jdbc:mysql:///atguigu?user=root&password=123456");
//    3.编写SQL语句
        String sql = "insert into t_user(account,password,nickname) values(?,?,?);";
//    4.创建Statement
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
//    5.占位符赋值
        long start = System.currentTimeMillis();

        for (int i = 0; i < 10000; i++) {
            preparedStatement.setObject(1, "dd"+i);
            preparedStatement.setObject(2, "dd"+i);
            preparedStatement.setObject(3, "驴蛋蛋"+i);
//    6.发送SQL语句,并获得结果
            preparedStatement.executeUpdate();
        }

        long end = System.currentTimeMillis();
//    7.结果解析
        System.out.println("执行10000次数据插入花费的时间=" + (end - start));
//    8.关闭资源
        preparedStatement.close();
        connection.close();
    }
}
```

输出结果:`执行10000次数据插入花费的时间=2629`



使用批量插入方式插入10000条数据

```java
package preparedstatement;

import org.junit.Test;

import java.sql.*;

public class PSOtherPart {
/*TODO:使用批量插入方式插入10000条数据
       1.路径后面添加?rewriteBatchedStatements=true 允许批量插入
       2.insert into values[注意必须是values]语句最后不能加分号;
       3.不是执行每条语句,是批量添加 addBatch();
       4.遍历添加完毕后,统一批量执行 executeBatch()
* */
    @Test
    public void testInsert() throws ClassNotFoundException, SQLException {
//    1.注册驱动
        Class.forName("com.mysql.cj.jdbc.Driver");
//    2.获取连接(注意要批量插入数据需要多传一个参数)
        Connection connection = DriverManager.getConnection("jdbc:mysql:///atguigu?rewriteBatchedStatements=true", "root", "123456");
//    3.编写SQL语句
        String sql = "insert into t_user(account,password,nickname) values(?,?,?)";
//    4.创建Statement
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
//    5.占位符赋值
        long start = System.currentTimeMillis();

        for (int i = 0; i < 10000; i++) {
            preparedStatement.setObject(1, "ddd"+i);
            preparedStatement.setObject(2, "ddd"+i);
            preparedStatement.setObject(3, "驴蛋蛋"+i);

            preparedStatement.addBatch();//不执行,输入先追加到values后面
        }
        preparedStatement.executeBatch();//执行批量操作

        long end = System.currentTimeMillis();
//    7.结果解析
        System.out.println("执行10000次数据插入花费的时间=" + (end - start));
//    8.关闭资源
        preparedStatement.close();
        connection.close();
    }
}
```

输出结果:`执行10000次数据插入花费的时间=414` **明显得到优化**



## jdbc事务





### 转账类事务设计

`BankDao`

```java
package transaction;



import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;

/*
* Description: bank表的数据库操作方法存储类
* */
public class BankDao {

    /*
    * 加钱的数据库操作方法(jdbc)
    * */
    public void add(String account, int money, Connection connection) throws ClassNotFoundException, SQLException {

//        3.编写SQL语句
        String sql = "update t_bank set money = money + ? where account = ?;";

//        4.创建statement
        PreparedStatement preparedStatement = connection.prepareStatement(sql);

//        5.占位符赋值
        preparedStatement.setObject(1, money);
        preparedStatement.setString(2, account);

//        6.发送SQL语句
        preparedStatement.executeUpdate();

//        7.关闭资源
        preparedStatement.close();

        System.out.println("加钱成功!");
    }

    /*
     * 减钱的数据库操作方法(jdbc)
     * */
    public void sub(String account, int money, Connection connection) throws Exception{

//        3.编写SQL语句
        String sql = "update t_bank set money = money - ? where account = ?;";

//        4.创建statement
        PreparedStatement preparedStatement = connection.prepareStatement(sql);

//        5.占位符赋值
        preparedStatement.setObject(1, money);
        preparedStatement.setString(2, account);

//        6.发送SQL语句
        preparedStatement.executeUpdate();

//        7.关闭资源
        preparedStatement.close();


        System.out.println("减钱成功!");
    }
}

```



`BankService`

```java
package transaction;

import org.junit.Test;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

/**
 * @Author LiuHaoze
 * @Description TODO 银行卡业务方法,调用Dao方法
 * @Date 2023/12/6 20:11
 * @Version 1.0
 */
public class BankService {

    @Test
    public void start() throws Exception {
        //二狗子给驴蛋蛋转五百
        transfer("lvdandan", "ergouzi", 500);

    }

    public void transfer(String addAccount, String subAccount, int money) throws Exception {
        BankDao bankDao = new BankDao();

//        一个事务最基本的要求,必须是同一个对象 connection

//        1.注册驱动
        Class.forName("com.mysql.cj.jdbc.Driver");

//        2.获取连接
        Connection connection = DriverManager.getConnection("jdbc:mysql:///atguigu?user=root&password=123456");

        try {
            //开启事务
            //关闭事务提交
            connection.setAutoCommit(false);

            //执行数据库动作
            bankDao.add(addAccount, money, connection);
            System.out.println("--------");
            bankDao.sub(subAccount, money, connection);

            //事务提交
            connection.commit();
        }catch (Exception e){
            //事务回滚
            connection.rollback();
            throw e;
        }finally {
            connection.close();
        }

    }
}

```





## 连接池

- 如果不使用数据库连接池,每次都通过`DriverManager`获取新连接,用完直接断开,连接的利用率太低太浪费
- 对于数据库服务器来说压力太大了

可以建立一个连接池,池中容纳一定数量的连接对象,使用时直接拿,用完再放回去

### 硬编码方式

```java
package druid;

import com.alibaba.druid.pool.DruidDataSource;
import com.alibaba.druid.pool.DruidPooledConnection;

import java.sql.Connection;
import java.sql.SQLException;

/**
 * @Author LiuHaoze
 * @Description TODO
 * @Date 2023/12/7 16:55
 * @Version 1.0
 */
public class DruidUsePart {

    /*
    * 直接使用代码设置连接池连接参数方式
    *
    * 1.创建一个druid连接池对象
    *
    * 2.设置连接池参数 [必须 | 非必须]
    *
    * 3.获取连接 [通用方法,所有连接池都一样]
    *
    * 4.回收链接 [通用方法,所有连接都一样]
    * */

    public void testHard() throws SQLException {
//        连接池对象
        DruidDataSource dataSource = new DruidDataSource();

//        设置参数
//        必须: 连接数据库驱动类的全限定符[注册驱动] | url | user | password
        dataSource.setUrl("jdbc:mysql://localhost:3306/atguigu");
        dataSource.setUsername("root");
        dataSource.setPassword("123456");
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
//        非必须: 初始化连接数量,最大的连接数量
        dataSource.setInitialSize(5);
        dataSource.setMaxActive(10);

//        获取连接
        Connection connection = dataSource.getConnection();

//        数据库curd


//        回收连接
        connection.close();//连接池提供的连接,close释放会变成直接回收
    }
}

```



### 软编码方式

配置文件:

```properties
# key = value => java Properties读取 (key | value)
driverClassName=com.mysql.cj.jdbc.Driver
username=root
password=123456
url=jdbc:mysql:///atguigu
```

代码:

```java
 /*
    通过读取外部配置文件的方法,实例化druid连接池对象
    * */
    @Test
    public void testSort() throws Exception {

//        1.读取外部配置文件 Properties
        Properties properties = new Properties();

//       src下的文件可以使用类加载器提供的方法
        InputStream ips = DruidUsePart.class.getClassLoader().getResourceAsStream("druid.properties");


        properties.load(ips);

//        2.使用连接池的工具类的工程模式,创建连接池
        DataSource dataSource = DruidDataSourceFactory.createDataSource(properties);

        Connection connection = dataSource.getConnection();

//        数据库curd

        connection.close();

    }
```



但如果每次都这么搞就太费劲了,我们封装成一个工具类

### 工具类`v2.0`

```java
package utils;

import com.alibaba.druid.pool.DruidDataSourceFactory;

import javax.sql.DataSource;
import java.io.IOException;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.SQLException;
import java.util.Properties;

/**
 * @Author LiuHaoze
 * @Description TODO 工具类,内部包含一个连接池对象,并且对外提供获取连接和回收连接的方法
 * 工具类的方法,推荐写成静态的,外部调用会更加方便
 *
 * 实现:
 *      属性 连接池对象[实例化一次]
 *          可以用单例模式/静态代码块
 *      方法
 *          对外提供连接的方法
 *          回收外部传入连接方法
 * @Date 2023/12/9 13:14
 * @Version 1.0
 */
public class JdbcUtils {

    private static DataSource dataSource = null;//连接池对象

    static {
        //初始化连接池对象
        Properties properties = new Properties();
        InputStream ips = JdbcUtils.class.getClassLoader().getResourceAsStream("druid.properties");
        try {
            properties.load(ips);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }

        try {
            dataSource = DruidDataSourceFactory.createDataSource(properties);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    public static Connection getConnection() throws SQLException {

        return dataSource.getConnection();
    }

    public static void freeConnection(Connection connection) throws SQLException {
        connection.close();
    }
}

```



但这个工具类并不能保证同一个线程中使用相同的一个连接.

### 工具类`v2.0`

```java
package utils;

import com.alibaba.druid.pool.DruidDataSourceFactory;

import javax.sql.DataSource;
import java.io.IOException;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.SQLException;
import java.util.Properties;

/**
 * @Author LiuHaoze
 * @Description TODO 利用线程池本地变量,存储连接信息.确保一个线程的多个方法可以获取同一个connection
 * 优势: 事务操作的时候 service 和 dao 属于同一个线程,不在传递参数了
 * @Date 2023/12/11 13:56
 * @Version 1.0
 */
public class JdbcUtilsV2 {

        private static DataSource dataSource = null;//连接池对象
        private static ThreadLocal<Connection> tl = new ThreadLocal<>();

        static {
            //初始化连接池对象
            Properties properties = new Properties();
            InputStream ips = utils.JdbcUtils.class.getClassLoader().getResourceAsStream("druid.properties");
            try {
                properties.load(ips);
            } catch (IOException e) {
                throw new RuntimeException(e);
            }

            try {
                dataSource = DruidDataSourceFactory.createDataSource(properties);
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        }

        public static Connection getConnection() throws SQLException {

//            线程本地变量中是否存在
            Connection connection = tl.get();
//            第一次没有
             if (connection == null){
//                 线程本地变量没有,连接池获取
                 connection = dataSource.getConnection();
                 tl.set(connection);
             }

            return connection;
        }

        public static void freeConnection() throws SQLException {
            Connection connection = tl.get();
            if (connection != null){
                tl.remove();//清空线程本地变量
                connection.setAutoCommit(true);//事务状态回归
                connection.close();
            }
        }

}
```



### 工具类BaseDao

我们知道JDBC分以下几步:

1. 注册驱动
2. 获取连接
3. 编写SQL语句结构
4. 创建statement
5. 占位符赋值
6. 发送SQL语句
7. 结果解析
8. 回收资源

但上面的工具类只封装了第1,2步和最后一步.中间的几步需要继续封装

基本上每一个数据表都应该有一个对应的DAO接口及其实现类, 所有表的操作(增删改查)代码重复率很高, 所以可以抽取公共代码, 给这些DAO的实现类可以抽取一个公共的父类,我们称其为`BaseDao`

#### 非DQL

```java
package utils;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

/**
 * @Author LiuHaoze
 * @Description TODO 封装Dao数据库重复代码.
 * 封装两个方法: 一个简化非DQL
 *             一个简化DQL
 * @Date 2023/12/11 15:20
 * @Version 1.0
 */
public abstract class BaseDao {

    /*
    * 封装简化非DQL语句
    * @param sql    带占位符的SQL语句
    * @param params 占位符的值
    * @return  执行影响的行数
    * */
    public int executeUpdate(String sql, Object...params) throws SQLException {
//        获取连接
        Connection connection = JdbcUtilsV2.getConnection();

        PreparedStatement preparedStatement = connection.prepareStatement(sql);

//        占位符赋值
//        可变参数可以当作数组使用
        for (int i = 0; i < params.length; i++) {
            preparedStatement.setObject(i, params[i]);
        }

//        发送SQL语句
        int rows = preparedStatement.executeUpdate();

        preparedStatement.close();
//        是否回收连接要看是不是事务
        if (connection.getAutoCommit()) {
//            没有开启事务,正常回收
            JdbcUtilsV2.freeConnection();
        }
//        connection.setAutoConnection(false);//开启事务了,不管连接即可,业务层处理

        return rows;
    }
}
```

使用BaseDao插入数据操作:

```java
public class PS_CRUD extends BaseDao {
    @Test
    public void testInsert() throws ClassNotFoundException, SQLException {
        /*
        * TODO:t_user插入一条数据
        *   account test
        *   password test
        *   nickname 二狗子
        * */

        String sql = "insert into t_user(account,password,nickname) values(?,?,?);";

        int i = executeUpdate(sql, "test", "test", "ergouzi");
        System.out.println("i = " + i);
    }
}
```



#### DQL

我们之前进行DQL操作时,返回的是一个`List<Map>`,但是`Map`**没有数据校验机制**,也**不支持反射操作**,所以这里我们返回一个`List<Java实体类> list`

```java
    public <T> List<T> executeQuery(Class<T> clazz, String sql, Object... paras) throws SQLException, InstantiationException, IllegalAccessException, NoSuchFieldException {

//        获取连接
        Connection connection = JdbcUtilsV2.getConnection();

        PreparedStatement preparedStatement = connection.prepareStatement(sql);
//        占位符赋值

        if (paras != null && paras.length != 0) {
            for (int i = 1; i < paras.length; i++) {
                preparedStatement.setObject(i, paras[i - 1]);
            }
        }

//      发送SQL语句
        ResultSet resultSet = preparedStatement.executeQuery();

//       结果集解析
        List<T> list = new ArrayList<>();

//        获取列的信息对象
//        TODO: metaData 装的当前结果集列的信息对象(它可以获取列的名称根据下角标,可以获取列的数量)
        ResultSetMetaData metaData = resultSet.getMetaData();

        int columnCount = metaData.getColumnCount();

        while (resultSet.next()) {
            T t = clazz.newInstance();

            for (int i = 1; i <= columnCount; i++) {

//              对象的属性值
                Object value = resultSet.getObject(i);

                String propertyName = metaData.getColumnLabel(i);

//                反射,给对象的属性赋值
                Field field = clazz.getDeclaredField(propertyName);
                field.setAccessible(true);//属性可以设置,打破private的修饰限制
                /*
                 * 参数1: 要赋值的对象   如果属性是静态,可以为null
                 * 参数2: 具体的属性值
                 * */
                field.set(t, value);
            }
            list.add(t);
        }
        resultSet.close();
        preparedStatement.close();

        if (connection.getAutoCommit()) {
            JdbcUtilsV2.freeConnection();
        }

        return list;
    }
```

