# Servlet



## 核心接口和类

### 1. `Servlet`接口

最重要的.包括以下五个方法(都需要重写):

- `void init(ServletConfig servletConfig)`
- `ServletConfig getServletConfig()`
- `void service(ServletRequest servletRequest, ServletResponse servletResponse)`
- `String getServletInfo()`
- `void destroy()`

### 2. `GenericServlet`抽象类
它提供生命周期方法`init`和`destroy`的简单实现,要编写一般的`Servlet`.只需重写抽象`service`方法即可

### 3. `HttpServlet`抽象类(推荐)
它继承`GenericServle`并进一步拓展.它自动判断请求为`post/get/put/delete`并调用对应`doPost() doGet() doPut() doDelete()`方法.我们写的子类需要至少重写其中的一个方法



## 两种配置方法

### 1. 使用web.xml

`url-pattern`定义匹配规则:

- 精确匹配:	/具体名称	只有url路径完全相同才会触发`Servlet`
- 后缀匹配:    *.xxx            只要是以xxx结尾的就匹配触发`Servlet`
- 通配符匹配: /*                 啥样的都能匹配
- 通配符匹配: /                   匹配所有请求,不包括.jsp

`load-on-startup`:

- 元素标记容器是否应该在web应用程序启动的时候就加载这个servlet
- 值必须为整数,代表servlet加载的先后顺序,从0开始
- 如果为负数或没有设置,则容器会在当前servlet被请求时再加载

### 2. 使用注解(Servlet3.0后支持,推荐)

`@WebServlet`:

- name: Servlet名字(可选)
- value: 配置url路径,可配置多个
- urlPatterns: 配置url路径,和value作用一样,这俩不能同时使用
- loadOnStartup: 配置Servlet的创建时机,跟web.xml里面的一样



## Servlet应用

### request对象

在Servlet中用来处理客户端请求需要用`doGet`或`doPost`方法的`request`对象

####  get和post区别

`get`请求

- `get`请求提交的数据会放在URL之后,以`?`分割URL和传输数据,参数之间以`&`相连
- `get`方式明文传递,数据量小,不安全
- 效率高,浏览器默认请求方式为get
- 对应的Servlet的方法是`doGet`

`post`请求

- `post`方法是把提交的数据放在HTTP包的`body`中
- 密文传递数据,数据量大,安全
- 效率相对没有GET高
- 对象的Servlet方法为`doPost`



#### request主要方法

| 方法名                                      | 说明                         |
| ------------------------------------------- | ---------------------------- |
| `String getParameter(String name)`          | 根据表单组件名称获取提交数据 |
| `void setCharacterEncoding(String charset)` | 指定每个请求的编码           |



#### request应用

##### 使用get接收数据

html页面代码:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>注册页面</title>
</head>
<body>
<form action="/WebProject_war_exploded/rs" method="get">
    <label>用户名: </label><input type="text" name="username"/><br/>
    <label>密码: </label><input type="password" name="password"/><br/>
    <input type="submit" value="注册"/>
</form>
</body>
</html>
```

Servlet代码:

```java
package servlet;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet(value = "/rs")
public class RegisterServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
//        1.获取用户请求发送的数据
        String username = req.getParameter("username");
        String password = req.getParameter("password");

        System.out.println("提交的数据: " + username + "\t" + password);
    }
}
```



##### 使用post接收数据

与用`get`几乎一样,就是`html`里面`method`改成`post`,servlet代码里多重写一个`doPost`方法.但要注意,post传输中文会出现乱码,可以通过在`doPost`方法中**对`request`请求对象设置统一的编码来解决**(使用`setCharacterEncoding`方法):

```java
 @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
//        对request请求对象设置统一的编码
        req.setCharacterEncoding("utf-8");//就是这里
//        获取用户请求发送的数据
        String username = req.getParameter("username");
        String password = req.getParameter("password");

        System.out.println("提交的数据: " + username + "\t" + password);
    }
```



### response对象

response对象用于响应客户请求并向客户端输出信息

#### response主要方法

| 方法名称                       | 作用                              |
| ------------------------------ | --------------------------------- |
| `setHeader(name, value)`       | 设置响应信息头                    |
| `setContentType(String)`       | 设置响应文件类型,响应式的编码格式 |
| `setCharacterEncoding(String)` | 设置服务端响应内容编码格式        |
| `getWriter()`                  | 获取字符输出流                    |

`getWriter`例子:

```java
  protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
//        对request请求对象设置统一的编码
        req.setCharacterEncoding("utf-8");
//        获取用户请求发送的数据
        String username = req.getParameter("username");
        String password = req.getParameter("password");

//        响应数据给客户端
        resp.setContentType("text/html;charset=UTF-8");//同时设置服务端的编码格式和客户端响应的文件类型及响应时的编码格式(推荐)
/*       resp.setCharacterEncoding("utf-8"); 仅仅设置服务端的编码格式
        resp.setHeader("Content-Type", "text/html;charset=utf-8");
        不推荐
 */
        PrintWriter printWriter = resp.getWriter();
        printWriter.println("登陆成功!");
    }
```

### 综合案例(Servlet + JDBC)

> - 要求:实现登录功能,展现所有用户功能
> - 仅展示关键代码

#### utils层

DbUtils:

```java
package servletProject.utils;

import com.alibaba.druid.pool.DruidDataSource;
import com.alibaba.druid.pool.DruidDataSourceFactory;

import java.io.IOException;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Properties;

public class DbUtils {
    private static DruidDataSource ds;
    private static final ThreadLocal<Connection> THREAD_LOCAL = new ThreadLocal<>();

    static {
        Properties properties = new Properties();
        InputStream inputStream = DbUtils.class.getResourceAsStream("/database.properties");
        try {
            properties.load(inputStream);
            ds = (DruidDataSource) DruidDataSourceFactory.createDataSource(properties);
        } catch (IOException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        }

    }

    public static Connection getConnection(){
        Connection connection = THREAD_LOCAL.get();
        if (connection == null){
            try {
                connection = ds.getConnection();
                THREAD_LOCAL.set(connection);
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        return connection;
    }

    public static void begin(){
        Connection connection = null;
        connection = getConnection();
        try {
            connection.setAutoCommit(false);
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }

    public static void commit(){
        Connection connection = null;
        try {
            connection = getConnection();
            connection.commit();
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }finally {
            closeAll(connection,null,null);
        }
    }

    public static void rollback(){
        Connection connection = null;
        try {
            connection = getConnection();
            connection.rollback();
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }finally {
            closeAll(connection,null,null);
        }
    }

    public static void closeAll(Connection connection, Statement statement, ResultSet resultSet){
        try {
            if (resultSet != null){
                resultSet.close();
            }
            if (statement != null){
                statement.close();
            }
            if (connection != null){
                connection.close();
                THREAD_LOCAL.remove();
            }
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }
}

```

#### entity层

Admin:

```java
package servletProject.entity;

public class Admin {
    private String username;
    private String password;
    private String phone;
    private String address;

    public Admin(String username, String password, String phone, String address) {
        this.username = username;
        this.password = password;
        this.phone = phone;
        this.address = address;
    }

    public Admin() {

    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getPhone() {
        return phone;
    }

    public void setPhone(String phone) {
        this.phone = phone;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return "Admin{" +
                "username='" + username + '\'' +
                ", password='" + password + '\'' +
                ", phone='" + phone + '\'' +
                ", address='" + address + '\'' +
                '}';
    }
}
```



#### Dao层

AdminDaoImpl:

```java
package servletProject.dao.impl;

import org.apache.commons.dbutils.QueryRunner;
import org.apache.commons.dbutils.handlers.BeanHandler;
import org.apache.commons.dbutils.handlers.BeanListHandler;
import servletProject.dao.AdminDao;
import servletProject.entity.Admin;
import servletProject.utils.DbUtils;

import java.sql.SQLException;
import java.util.List;

public class AdminDaoImpl implements AdminDao {
    private QueryRunner queryRunner = new QueryRunner();
    @Override
    public int insert(Admin admin) {
        return 0;
    }

    @Override
    public int delete(String username) {
        return 0;
    }

    @Override
    public int update(Admin admin) {
        return 0;
    }

    @Override
    public Admin select(String username) {
        try {
            Admin admin = queryRunner.query(DbUtils.getConnection(), "select * from admin where username=?;",new BeanHandler<Admin>(Admin.class),username);
            return admin;
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }

    @Override
    public List<Admin> selectAll() {
        try {
            List<Admin> admins =  queryRunner.query(DbUtils.getConnection(), "select * from admin", new BeanListHandler<Admin>(Admin.class));
            return admins;
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }
}
```

#### service层

AdminServiceImpl:

```java
package servletProject.service.impl;

import servletProject.dao.AdminDao;
import servletProject.dao.impl.AdminDaoImpl;
import servletProject.entity.Admin;
import servletProject.service.AdminService;
import servletProject.utils.DbUtils;

import java.util.List;

public class AdminServiceImpl implements AdminService {
    private AdminDao adminDao = new AdminDaoImpl();
    @Override
    public Admin login(String username, String password) {
        Admin result = null;
        try {
            DbUtils.begin();
            Admin admin = adminDao.select(username);
            if (admin != null){
                if (admin.getPassword().equals(password)){
                    result = admin;
                }
            }
            DbUtils.commit();
        } catch (Exception e) {
            DbUtils.rollback();
            throw new RuntimeException(e);
        }
        return result;
    }

    @Override
    public List<Admin> showAllAdmin() {
        List<Admin> admins = null;
        try {
            DbUtils.begin();
            admins = adminDao.selectAll();
            DbUtils.commit();
        } catch (Exception e) {
            DbUtils.rollback();
            throw new RuntimeException(e);
        }

        return admins;

    }
}
```

#### servlet层

LoginServlet:

```java
package servletProject.servlet;

import servletProject.entity.Admin;
import servletProject.service.AdminService;
import servletProject.service.impl.AdminServiceImpl;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

@WebServlet(value = "/login")
public class LoginServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        req.setCharacterEncoding("UTF-8");
        resp.setContentType("text/html;charset=UTF-8");
//        1.收参
        String username = req.getParameter("username");
        String password = req.getParameter("password");
//        2.调用业务逻辑
        AdminService adminService = new AdminServiceImpl();
        Admin admin = adminService.login(username, password);
//        3.处理结果
        PrintWriter printWriter = resp.getWriter();
        if (admin != null){
            printWriter.println("<html>");
            printWriter.println("<head>");
            printWriter.println("<meta charset='UTF-8'>");
            printWriter.println("<title>结果页面</title>");
            printWriter.println("</head>");
            printWriter.println("<body>");
            printWriter.println("<h1>登陆成功!</h1>");
            printWriter.println("</body>");
            printWriter.println("</html>");
        }
        else {
            printWriter.println("<html>");
            printWriter.println("<head>");
            printWriter.println("<meta charset='UTF-8'>");
            printWriter.println("<title>结果页面<title>");
            printWriter.println("</head>");
            printWriter.println("<body>");
            printWriter.println("<h1>登陆失败!</h1>");
            printWriter.println("</body>");
            printWriter.println("</html>");
        }
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);//doPost比doGet安全,doGet比doPost处理速度快,在doPost里调用doGet可以同时实现安全与速度快
    }
}

```

ShowAllAdminServlet:

```java
package servletProject.servlet;

import servletProject.entity.Admin;
import servletProject.service.AdminService;
import servletProject.service.impl.AdminServiceImpl;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
import java.util.List;

@WebServlet(value = "/showall")
public class ShowAllAdminServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html;charset=utf-8");
        AdminService adminService = new AdminServiceImpl();

        List<Admin> adminList = adminService.showAllAdmin();

        PrintWriter printWriter = resp.getWriter();
        if (adminList != null){
            printWriter.println("<html>");
            printWriter.println("<head>");
            printWriter.println("<meta charset='UTF-8'>");
            printWriter.println("<title>显示所有</title>");
            printWriter.println("</head>");
            printWriter.println("<body>");
            printWriter.println("<table>");

            printWriter.println("   <tr>");
            printWriter.println("       <td>username</td>");
            printWriter.println("       <td>password</td>");
            printWriter.println("       <td>phone</td>");
            printWriter.println("       <td>address</td>");
            printWriter.println("   </tr>");

            for (Admin admin: adminList){
                printWriter.println("   <tr>");
                printWriter.println("       <td>" + admin.getUsername() + "</td>");
                printWriter.println("       <td>" + admin.getPassword() + "</td>");
                printWriter.println("       <td>" + admin.getPhone() +  "</td>");
                printWriter.println("       <td>" + admin.getAddress() + "</td>");
                printWriter.println("   </tr>");
            }
            printWriter.println("</table>");
            printWriter.println("</body>");
            printWriter.println("</html>");
        }else {
            printWriter.println("<html>");
            printWriter.println("<head>");
            printWriter.println("<meta charset='UTF-8'>");
            printWriter.println("<title>显示所有</title>");
            printWriter.println("</head>");
            printWriter.println("<body>");
            printWriter.println("<h3>当前没有用户</h3>");
            printWriter.println("</body>");
            printWriter.println("</html>");
        }
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}
```

## 转发与重定向

在之前的案例中,调用业务逻辑和显示结果页面都在同一个servlet里,会产生设计问题:

- 不符合单一职能原则,各司其职的思想
- 不利于后续的维护

应该将业务逻辑与显示结果分离开

![img](file:///C:/Users/86139/AppData/Local/Packages/Microsoft.Windows.Photos_8wekyb3d8bbwe/TempState/ShareServiceTempFolder/屏幕截图(24).jpeg)

- 问题: 业务逻辑和显示结果分离后,如何跳转到显示结果的Servlet?
- 业务逻辑得到的数据结果如何传递给显示结果的Servlet?

### 转发

转发的作用再服务器端,将**请求**发送给服务器上的其他资源,以共同完成一次请求的处理

#### 页面跳转

在调用业务逻辑的Servlet中,编写以下代码:
`request.getRequestDispatcher("/目标URL-pattern").forward(request,response);`

使用forward跳转时,是在服务器内部跳转,地址栏不发生变化,属于同一次请求.

eg:
AServlet:

```java
package servletProject.servlet;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet(value = "/a")
public class AServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        req.getRequestDispatcher("/b").forward(req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}
```

BServlet:

```java
package servletProject.servlet;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet(value = "/b")
public class BServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("B executed...");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}
```

然后访问 /a 控制台会显示"B executed...",说明AServlet将请求转发给了BServlet.但此时地址栏仍然显示为"/a"

#### 数据传递

forward表示一次请求,是在服务器内部跳转,可以共享同义词request作用域中的数据.

- request作用域:拥有存储数据的空间,作用范围是一次请求有效(一次请求可以经过多次转发)

  - 可以将数据存入request后,在一次请求过程中的任何位置进行获取

  - 可传递任何数据(基本数据类型,对象,数组,集合等)

- 存数据:`request.setAttribute(key,value);`

  - 以键值对形式存储在request作用域中,key为String类型,value为Object类型

- 取数据:`request.getAttribute(key);`

  - 通过key访问value

eg:
AServlet中的doget:

```java
@Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String s = (String) req.getAttribute("username");
        System.out.println(s);
    }
```

BServlet获取A分享的数据:

```java
@Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String s = (String) req.getAttribute("username");
        System.out.println(s);
    }
```

#### 优化后的ShowAllAdmin:

ShowAllAdminController:

```java
package servletProject.servlet;

import servletProject.entity.Admin;
import servletProject.service.AdminService;
import servletProject.service.impl.AdminServiceImpl;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.List;

@WebServlet(value = "/showallcontroller")
public class ShowAllAdminController extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
//        只负责调用业务逻辑功能
        AdminService adminService = new AdminServiceImpl();
        List<Admin> adminList = adminService.showAllAdmin();

//        request作用域存储数据
        req.setAttribute("admins", adminList);
//        通过转发,跳转到显示结果的Servlet
        req.getRequestDispatcher("/showalljsp").forward(req,resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}
```

ShowAllAdminJSP:

```java
package servletProject.servlet;

import servletProject.entity.Admin;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
import java.util.List;

@WebServlet(value = "/showalljsp")
public class ShowAllAdminJSP extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html;charset=utf-8");
        PrintWriter printWriter = resp.getWriter();
        List<Admin> adminList = (List) req.getAttribute("admins");

        if (adminList != null){
            printWriter.println("<html>");
            printWriter.println("<head>");
            printWriter.println("<meta charset='UTF-8'>");
            printWriter.println("<title>显示所有</title>");
            printWriter.println("</head>");
            printWriter.println("<body>");
            printWriter.println("<table>");

            printWriter.println("   <tr>");
            printWriter.println("       <td>username</td>");
            printWriter.println("       <td>password</td>");
            printWriter.println("       <td>phone</td>");
            printWriter.println("       <td>address</td>");
            printWriter.println("   </tr>");

            for (Admin admin: adminList){
                printWriter.println("   <tr>");
                printWriter.println("       <td>" + admin.getUsername() + "</td>");
                printWriter.println("       <td>" + admin.getPassword() + "</td>");
                printWriter.println("       <td>" + admin.getPhone() +  "</td>");
                printWriter.println("       <td>" + admin.getAddress() + "</td>");
                printWriter.println("   </tr>");
            }
            printWriter.println("</table>");
            printWriter.println("</body>");
            printWriter.println("</html>");
        }else {
            printWriter.println("<html>");
            printWriter.println("<head>");
            printWriter.println("<meta charset='UTF-8'>");
            printWriter.println("<title>显示所有</title>");
            printWriter.println("</head>");
            printWriter.println("<body>");
            printWriter.println("<h3>当前没有用户</h3>");
            printWriter.println("</body>");
            printWriter.println("</html>");
        }
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}
```

### 重定向

重定向作用在客户端,客户端将请求发送给服务器后,服务器响应给客户端一个新的请求地址,客户端重新发送新请求.

#### 页面跳转

在调用业务逻辑的Servlet中,编写:`response.sendRedirect("目标URI");`

> URI:统一资源标识符(Uniform Resource Identifier), 用来表示服务器中定位一个资源,资源在web项目中的路径(/project/source)

eg:
AServlet中的doGet:

```java
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.sendRedirect("/WebProject_war_exploded/b");
    }
```

BServlet中的doGet:

```java
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("B executed...");
    }
```

运行访问/a,发现地址栏变成了/b,然后控制台输出了"B execeted",说明跳转成功了.
![img](file:///C:/Users/86139/AppData/Local/Packages/Microsoft.Windows.Photos_8wekyb3d8bbwe/TempState/ShareServiceTempFolder/屏幕截图(26).jpeg)

> 相当于两次请求

#### 数据传递

sendRedirect跳转时,地址栏改变,代表客户端重新发送的请求.属于两次请求

- response没有作用域,两次request请求中的数据无法共享

- 传递数据: 可以用一种特殊的方式传递数据.通过URI的拼接进行数据传递(`"/WebProject_war_exploded/b?username=tom"`);

  > 只能传递**String类型**的数据,并且是**明文传递**,拼接在地址后面

- 获取数据:`String username = req.getParameter("username")`



## Servlet生命周期

### 生命周期的四个阶段

#### 1.实例化

用户第一次访问Servlet时,由容器调用Servlet的构造器创建具体的Servlet对象,也可以在容器启动之后立刻创建实例,使用如下代码可以设置Servlet是否在服务器启动时就创建: `<load-on-startup>1</load-on-startup>`

> 正整数:一启动就创建. 负数或不写: 被请求访问时再创建.
> Servlet只创建一个

#### 2.初始化

初始化阶段,`init()`方法会被调用.这个方法在`javax.servlet.Servlet`接口中定义.其中,方法以一个`ServletConfig`类型的对象作为参数.

> 注意:init()方法只被执行一次

#### 3.服务

当客户端有一个请求时,容器就会请求ServletRequest与响应ServletResponse对象转给Servlet,以参数的形式传给service方法

> 此方法会执行多次

#### 4.销毁

当Servlet容器停止或者重新启动都会引起销毁Servlet对象并调用destroy方法

> destory方法执行一次

#### 例子

```java
package servlet3;

import javax.servlet.*;
import javax.servlet.annotation.WebServlet;
import java.io.IOException;

@WebServlet(value = "/ls")
public class LifeServlet implements Servlet {
    public LifeServlet(){
        System.out.println("1. 实例化");
    }

    @Override
    public void init(ServletConfig servletConfig) throws ServletException {
        System.out.println("2. 初始化");
    }

    @Override
    public ServletConfig getServletConfig() {
        return null;
    }

    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        System.out.println("3. 接收请求,响应结果");
    }

    @Override
    public String getServletInfo() {
        return null;
    }

    @Override
    public void destroy() {
        System.out.println("4. 销毁");
    }
}

```

访问/ls,控制台打印了

```markdown
1. 实例化
2. 初始化
3. 接收请求,响应结果
```

刷新页面,控制台继续输出`3. 接收请求,响应结果`,说明实例化与初始化都只执行一次.
重新启动服务器就会显示`4.销毁`

![img](file:///C:/Users/86139/AppData/Local/Packages/Microsoft.Windows.Photos_8wekyb3d8bbwe/TempState/ShareServiceTempFolder/屏幕截图(27).jpeg)



## Servlet线程安全问题

Servlet在访问之后,会执行实例化操作,创建一个Servlet对象.而我们Tomcat容器可以同时多个线程并发访问同一个Servlet.如果在方法中对成员变量做修改操作,就会有线程安全问题.

### 如何保证线程安全

- `synchronized`

  - 将存在线程安全问题的代码放在同步代码块中

  > 但性能太低,不推荐使用

- 实现`SingleThreadModel`接口

  - servlet实现SingleThreadModel接口后,每个线程都会创建servlet实例,这样每个客户端请求就不存在共享资源的问题,但是servlet响应客户端请求的效率就太低了,所以已经淘汰

- 尽可能使用**局部变量**



##  状态管理

### 现有问题

- HTTP协议时无状态的,不能保存每次提交的信息
- 如果用户发来一个新请求,服务器无法知道他是否与上次请求有联系
- 对于那些需要多次提交数据才能完成的Web操作(比如登录),就成问题了

### 概念

将浏览器与web服务器之间多次交互当作一个整体来处理,并且将多次交涉所涉及的数据(即状态)保存下来.

### 状态管理分类

- 客户端状态管理技术: 将状态保存在客户端.代表性的是:Cookie技术
- 服务器状态管理技术: 将状态保存在服务器端.代表性的是session技术(服务器传递session时需要使用Cookie的方式)和application



## Cookie的使用

- Cookie是在浏览器访问Web服务器的某个资源时,由Web服务器在HTTP响应消息头中附带传送给浏览器的一小段数据
- 一旦Web浏览器保存了某个Cookie,那么它以后每次访问该Web服务器时,都应在HTTP请求头中将这个Cookie回传给Web服务器
- 一个Cookie主要由标识该信息的名称(name)和值(value)组成.

eg:

```java
@WebServlet(value = "/cs")
public class CookieServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doPost(req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //    1. 服务端创建Cookie对象
        Cookie cookie = new Cookie("username", "gavin");

//        2.将Cookie响应给客户端
        resp.addCookie(cookie);
    }
}
```

### 创建Cookie

```java
//创建Cookie
Cookie ck = new Cookie("code", code);
ck.setPath("/webs");//设置Cookie的路径
ck.setMaxAge(-1);//设置保存的生命周期 取值有三种: >0的正数x:有效期x秒; =0:浏览器关闭时失效; <0浏览器默认将cookie保存在内存里，当浏览器关闭时，cookie从内存中释放
```

### 获取Cookie

```java 
//获取所有的Cookie
Cookie[] cks = request.getCookies();
	//遍历Cookie
	for(Cookie ck:cks){
        //检索出自己的Cookie
        if(ck.getName().equals("code"))
        {
            //记录Cookie的值
            code = ck.getvalue();
            break;
        }
    }
```

### 修改Cookie

 就是创建一个同名同路径的Cookie即可修改.

```java
//创建Cookie
Cookie ck = new Cookie("code", code);
ck.setPath("/webs");
ck.setMaxAge(-1);
response.addCookie(ck);//让浏览器添加Cookie
```

> 注意: 如果改变Cookie的name和有效路径会新建Cookie,而改变Cookie的值和有效期会覆盖原有cookie

### Cookie编码与解码

Cookie默认不支持中文,只能包括ASCII字符,所以Cookie需要对Unicode字符进行编码,负责会出现乱码.

- 编码可以使用`java.net.URLEncoder`类的`encode(String str, String encoding)`方法
- 解码使用`java.net.URLDecoder`类的`decode(String str, String encoding)`方法

eg:

```java
Cookie cookie = new Cookie(URLEncoder.encode("姓名", "UTF-8"), URLEncoder.encode("张三", "UTF-8"));

cookie.setPath("/WebProject_war_exploded/get");

cookie.setMaxAge(600);

resp.addCookie(cookie);
```

```java
Cookie[] cookies = req.getCookies();

if(cookies != null){
    for(Cookie cookie : cookies){
        System.out.println(URLDecoder.decode(cookie.getName(), "UTF-8") + ":" + URLDecoder.decode(cookie.getvalue(), "UTF-8"));
    }
}
```

### Cookie优缺点

#### 优点

1. 可配置到期规则
2. 简单性: Cookie是一种基于文本的轻量结构,包含简单的键值对.
3. 数据持久性: Cookie默认在过期之前是可以一直存在客户端浏览器上的

#### 缺点

1. 大小受到限制: 大多数浏览器对Cookie的大小有 4k, 8k字节的限制
2. 用户配置为禁用: 有些用户仅用了浏览器或客户端设备接受Cookie的能力,因此限制了这一功能.
3. 潜在的安全风险: Cookie可能会被篡改.会对安全性造成潜在风险或者导致依赖于Cookie的应用程序失败



## Session对象(重点)

### Session概述

- Session用于记录用户的状态.Session指的是在一段时间内,单个客户端与Web服务器的一连串相关的交互过程
- 在一个Session中,客户可能会多次请求访问同一个资源,也有可能请求访问各种不同的服务器资源

### Session原理

- 服务器会为每一次会话分配一个Session对象
- 同一个浏览器发起的多次请求,同属于一次会话(Session)
- 首次使用到Session时,服务器会自动创建Session,并创建Cookie存储SessionID发送回客户端

> 注意: session是由服务端创建的

### Session使用

- Session作用域: 拥有存储数据的空间,作用范围是一次会话有效
  - 一次会话时使用同一浏览器发送的多次请求.一旦浏览器关闭,则结束会话
  - 可以将数据存入Session中.在一次会话的任意位置进行获取
  - 可传递任何数据(基本数据类型,对象,集合,数组)

#### 获取Session

```java
// 获取Session对象
HttpSession session = request.getSession();
System.out.println("ID: " + session.getId());//唯一标记
```

eg:

```java
package sessions;

import javax.servlet.*;
import javax.servlet.http.*;
import javax.servlet.annotation.*;
import java.io.IOException;

@WebServlet(value = "/ss")
public class SessionServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
//        1.通过request对象获取Session对象
        HttpSession session = request.getSession();

        System.out.println(session.getId());
    }
}
```

运行后访问 /ss,控制台返回SessionId,关闭浏览器再打开访问/ss,返回的SessionId变化.打开控制台发现:**服务端第一次会自动创建Session,并创建一个Cookie保存SessionId传给客户端**

#### Session保存数据

>  `setAttribute(属性名, object)`保存数据到session中

`session.setAttribute("key", value); //以键值对的形式存储在session作用域中`

#### Session获取数据

> `getAttribute(属性名);`获取session中数据

`session.getAttribute("key"); //通过String类型的Key访问Object类型的value`



#### Session移除数据

> `removeAttribute(属性名);`从session中删除数据

`session.removeAttribute("key"); //通过键移除session作用域中的值`



### Session与Request应用区别

- request是一次请求有效,请求改变,则request改变
- session是一次会话有效,浏览器改变,则session改变



### Session的生命周期

- 开始: 第一次使用到Session的请求产生,则创建Session
- 结束:
  - 浏览器关闭,则失效
  - Session超时,则失效
    - `session.setMaxInactiveInterval(seconds);//设置最大有效时间(单位:秒)`
  - 手工销毁,则失效
    - `session.invalidate();//登录退出,注销`



### 浏览器禁用Cookie的解决方案

服务器在默认情况下,会使用Cookie的方式将Session发送给浏览器,如果用户禁止Cookie,则Session不会被浏览器保存,此时服务器可以使用如URL重写这样的方式来发送SessionID.

#### URL重写

浏览器在访问服务器上的某个地址时,不再使用原来那个地址,而是使用经过改写的地址(即在原来的地址上加上了sessionID)

#### 实现URL重写

> `response.encodeRedirectURL(String url)//生成重写的URL`

```java
HttpSession session = request.getSession();
//重写URL追加SessionID
String newUrl = response.encodeRedirectURL("/WebProject_war_exploded/cs");
System.out.println(newUrl);

response.sendRedirect(newUrl);
```



### Session实战:权限验证

![img](file:///C:/Users/86139/AppData/Local/Packages/Microsoft.Windows.Photos_8wekyb3d8bbwe/TempState/ShareServiceTempFolder/屏幕截图(29).jpeg)



## ServletContext对象(重点)

### ServletContext概述

- 全局对象,也拥有作用域,对应一个Tomcat中的Web应用
- 当Web应用启动时,会为每一个Web应用创建一块共享的存储区域(ServletContext)
- ServletContext在Web服务器启动时创建,服务器关闭时销毁

### 获取ServletContext对象

- `GenericServlet`提供了`getServletContext()`方法. (**推荐**) `this.getServletContext()`

  > HttpServlet继承了GenericServlet

- `HttpServletRequest`提供了`getServletContext()`方法(**推荐**)

- `HttpSession`提供了`getServletContext()`方法

### ServletContext作用

- 获取项目真实路径

  > `String realpath =  servletContext.getRealPath("/");`

- 获取项目上下文路径

  > `System.out.println(servletContext.getContextPath());//上下文路径(应用程序名称)`
  >
  > `System.out.println(request.getContextPath());`

- 全局容器

  > ServletContext拥有作用域,可以存储数据到全局容器中
  >
  > - 存储数据: `servletContext.setAttrribute("name", value);`
  > - 获取数据: `servletContext.getAttrribute("name");`
  > - 移除数据: `servletContext.removeAttrribute("name");`



### ServletContext特点

- 唯一性: 一个应用对应一个ServletContext
- 生命周期: 只要容器不关闭或者应用不卸载,ServletContext就一直存在

### ServletContext应用场景

ServletContext统计当前项目访问次数

```java
package counter;

import javax.servlet.*;
import javax.servlet.http.*;
import javax.servlet.annotation.*;
import java.io.IOException;

@WebServlet(name = "CounterController", value = "/counterController")
public class CounterController extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
//        1.获取ServletContext对象
        ServletContext servletContext = request.getServletContext();

//        2.获取计数器
        Integer counter = (Integer) servletContext.getAttribute("counter");

        if (counter == null){
            counter = 1;
            servletContext.setAttribute("counter", counter);
        }else {
            counter++;
            servletContext.setAttribute("counter", counter);
        }
        System.out.println("counter: "+ counter);
    }
}
```



### 作用域总结

- HttpServletRequest: 一次请求,请求响应之前有效
- HttpSession: 一次会话开始,浏览器不关闭或不超时之前有效
- ServletContext: 服务器启动开始, 服务器停止之前有效.



## 过滤器(重点)

Servlet中有许多冗余重复的代码,多个Servlet都要进行编写.过滤器可以解决

过滤器(Filter)是处于客户端与服务器目标资源之间的一道过滤技术

![img](file:///C:/Users/86139/AppData/Local/Packages/Microsoft.Windows.Photos_8wekyb3d8bbwe/TempState/ShareServiceTempFolder/屏幕截图(30).jpeg)

### 过滤器作用

- 执行地位在Servlet之前,客户端发送请求时,会先经过Filter,再达到目标Servlet中,响应时,会根据执行流程再反向执行Filter
- 可以解决多个Servlet共性代码的冗余问题(例如: 乱码处理, 登录验证等)


### 过滤器编写

Servlet API中提供了一个Filter接口,开发人员编写一个Java类实现这个接口即可,这个Java类称为过滤器

#### 实现过程

- 编写Java类实现Filter接口
- 在doFilter方法中编写拦截逻辑
- 设置拦截路径

```java
package filter;

import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import java.io.IOException;
import java.util.logging.LogRecord;

@WebFilter(value = "/t")// "/t"的servlet里输出"Target".
public class MyFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        Filter.super.init(filterConfig);
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("--MyFilter--");
//        让请求继续
        filterChain.doFilter(servletRequest, servletResponse);

        System.out.println("--end--");
    }

    @Override
    public void destroy() {
        Filter.super.destroy();
    }
}
```

控制台输出:
--MyFilter--
--Target--
--end--

#### 过滤器配置

##### 注解配置

在Filter类上使用注解: `@WebFilter(value = "/目标资源")`

##### xml配置

在web.xml中进行过滤器的配置

```xml
<filter>
        <filter-name>名字: xml</filter-name>
        <filter-class>filter的路径: filter.XmlFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>xml</filter-name>
        <url-pattern>过滤的Servlet的url: /test</url-pattern>
    </filter-mapping>
```

##### 拦截路径

- 精确过滤匹配: 如`/index.jsp`, `/myservlet1`
- 后缀过滤匹配: 如`*.jsp`, `*.html`
- 通配符过滤匹配: `/*`表示拦截所有

#### 过滤器链和优先级

##### 过滤器链

客户端对服务器请求之后，服务器调用Servlet之前会执行一组过滤器(多个过滤器)，那么这组过滤器就称为一条过滤器链。
每个过滤器实现某个特定的功能，当第一个Filter的doFilter方法被调用时，Web服务器会创建一个代表Filter链的FilterChain对象传递给该方法。在doFilter方法中， 开发人员如果调用了FilterChain对象的doFiter方法，则Web服务器会检查FilterChain对象中是否还有filter,如果有，则调用第2个filter,如果没有，则调用目标资源。

![img](https://img-blog.csdnimg.cn/ad39b71ad8594d6c9a6d5bc30b61f2af.png)



##### 过滤器优先级

在一个Web应用中，可以开发编写多个Filter,这些Filter组合 起来称之为一个Filter链。 优先级:
●如果为注解的话，是按照类全名称的字符串顺序决定作用顺序
●如果web.xml, 按照filter-mapping注册顺序，从上往下
●web.xml配置 高于注解方式
●如果注解和web.xml同时配置，会创建多个过滤器对象，造成过滤多次。

#### 过滤器应用

##### 解决编码问题

```java
import jakarta.servlet.*;
import jakarta.servlet.annotation.WebFilter;
import jakarta.servlet.annotation.WebServlet;

import java.io.IOException;

@WebFilter(value = "/*")
public class EncodingFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        Filter.super.init(filterConfig);
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        //统一处理请求和响应的乱码
       	servletRequest.setCharacterEncoding("UTF-8");
        							servletResponse.setContentType("text/html;charset=utf-8");
        filterChain.doFilter(servletRequest, servletResponse);
    }

    @Override
    public void destroy() {
        Filter.super.destroy();
    }
}
```

##### 权限验证

```java
import com.wyy.servletProject.entity.Manager;
import jakarta.servlet.*;
import jakarta.servlet.annotation.WebFilter;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import jakarta.servlet.http.HttpSession;

import java.io.IOException;

@WebFilter(value = "/showallcontroller")
public class checkfilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        Filter.super.init(filterConfig);
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        //权限验证 验证管理员是否登录
        //向下转型 拆箱
        HttpServletRequest request = (HttpServletRequest)servletRequest;
        HttpServletResponse response = (HttpServletResponse) servletResponse;
        HttpSession session = request.getSession();
        Manager mgr = (Manager) session.getAttribute("mgr");
        if (mgr != null){
            filterChain.doFilter(request,response);
        }else {
            response.sendRedirect(request.getContextPath()+"/loginMgr.html");
        }
    }

    @Override
    public void destroy() {
        Filter.super.destroy();
    }
}
```

