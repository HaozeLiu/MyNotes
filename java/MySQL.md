# MySQL



数据库中最基本的单元是表（`table`）

(数据库)`database` -> (表) `table`

## 简单命令

- `\c`：终止一条命令的输入

- `show databases;`:显示数据库列表
- `use name;`:使用数据库
- `create database name;`：建新库
- `select database();`:查看当前数据库
- `drop database name`：删库



- `show tables`显示库中表
- `create table name`：建表
- `drop table name`：删表
- `select * from name`:显示表中记录
- `delete from name`:清空表中记录
- `desc name`: 查看表的结构



## 简单查询



- 查询一个字段？

> `select 字段名 from 表名;`

- 查询多个字段：

> 用逗号隔开：`select xx,xx from 表名;`

- 查询所有字段：

  > 把所有字段名都写上。
  >
  > 用星号*：`select * from 表名;`（但效率低，可读性差，实际开发中不要用）

- 给查询的列起别名

  > 用as：`select xx as 别名 from 表名; `（只是将显示的查询结果列名显示为别名，原表名不变）

**select 语句永远不会进行修改操作（它只负责查询）**

as也可以省略

> `select xx 别名 from 表名；`



## 条件查询

查询出表中符合条件的数据

格式：

`select	字段1, 字段2, 字段3...... from 表名 where 条件；`

条件：

1. `>, <, >=, <=, =` 等比较大小
1. `&&， || `与`and， or`均可
1. `between ... and ...` 相当于`>= ... and <= ...`
1. `null`不能用`xx = null`查找，因为数据库中`null`代表什么也没有，应该用`xx is null`。查询不为`null`就用 `xx is not null`

### 模糊查询

使用`like`。

`%, _`是占位符，`%`代表任意个(包括0个)字符，`_`代表任意一个字符。

eg：

1. 找出名字中带 o 的:

   `select ename form emp where ename like '%o%';`

2. 找出名字以 t 结尾的：

   `select ename from emp where ename like '%t';`

3. 找出名字以 k 开始的

​	  `select ename from emp where ename like 'k%'`

4. 找出第二个字母是 a 的

   `select ename from emp where ename like '_a%';`

注意，\ 是转义字符。如果要找出名字带下划线_的，我们需要对下划线转义`select ename from emp where ename like '%\_%';`



## 排序

如按薪资排序查询员工：

 `select ename, sal from emp order by sal;`

默认是升序的，若想降序要在`order by xxx`后面加个`desc`。

`desc`是降序，`asc`是升序



### 按多个字段排序

按薪资升序排名，如果薪资相同，再按找名字升序排列：

` select ename, sal from emp order by sal asc, ename asc;`

就是加个逗号后面继续写就行

**注意**，除了用`sal，ename`这种字段名字代表外也可以用数字代替：

` select ename, sal from emp order by 2;`(`sal`是`ename`，`sal`中的第2个)



## 数据处理函数

又叫单行处理函数：一个输入对应一个输出。

- `lower(), upper()`转换大小写

>  `select lower(ename) from emp;`

- `substr(对象, 起始下标, 截取长度)` 取子串

  > `select substr(ename, 1, 3) from emp;` 注意，**不是从0开始是从1开始**

- `length()`取长度
- `trim()` 去除字符串前后的空格
- `round(数字, 四舍五入到第几位小数)` 四舍五入
- `rand()`生成随机数
- `ifnull(数据, null转化的值)`空处理函数，将`null`转化为一个具体值（有null参与的数学运算结果都为null，为了避免这个现象要用到这个函数）

>  `select ename, (sal + ifnull(comm, 0)) * 12 from emp;`



### `case.. when.. then.. else..end`:

当员工工作岗位为MANAGER时，工资上调10%，SALESMAN上调50%，其他正常。（不修改数据库，只是查询结果显示为上调）

` select ename, job, sal, (case job when 'MANAGER' then sal * 1.1 when 'SALESMAN' then sal * 1.5 else sal end) as newsal from emp;`





## 分组函数（多行处理函数）

特点：输入多行，最终输出一行

5个：

1. `count`
2. `sum`
3. `avg`
4. `max`
5. `min`

eg：找出最高工资

`select max(sal) from emp;`

### 分组查询***

`select ... from ... group by ...`

如：计算每个部门的工资和？ 计算每个工作岗位的平均薪资？。。。

将之前关键字组合在一起，**执行顺序**：

```sql
select
	...
from
	...
where
	...
group by
	...
order by
	...
```

顺序不能颠倒，**执行顺序是**：

1. `from`
2. `where`
3. `group by`
4. `select`
5. `order by`



注意点：

- **分组函数会直接忽略掉NULL，不需要提前对NULL进行处理**
- 分组函数使用的时候必须先进行分组，然后再使用。如果没分组整张表默认为一组。分组函数不能使用在`where`子句中

> 找到比最低工资高的员工信息：`select ename, sal from emp where sal > min(sal)` **会报错**，因为where执行时还没有分组，所以报错了。
>
> 为啥`select max(sal) from emp;`不报错？因为select前面已经执行group by了，没写相当于整个表当一组。



- 找出每个工作岗位的工资和？

  > ​	按照工作岗位分组，然后对工资求和:`select job, sum(sal) from emp group by job;`

- 找出每个部门的最高薪资？

>  `select deptno, max(sal) from emp group by deptno;`

- 找出每个部门，不同工作岗位的最高薪资？

>  `select deptno, job, max(sal) from emp group by deptno, job;`



在`group by`后面可以加`having xxx`条件再一次过滤

- 找出每个部门最高薪资，而且要大于3000

>  `select deptno, max(sal) from emp group by deptno having max(sal) > 30
> 00;` 
>
> `select deptno, max(sal) from emp where sal > 3000 group by deptno;`(用`where` 先一步筛选，效率更高)





### 去除重复记录

（把查询结果中的重复去掉，数据库原数据不变）

要使用关键字：`distinct`

eg: ` select distinct job from emp;`

> `distinct`必须在所有字段前方。
>
> 如果`distinct`后面有多个字段，那就是多个字段都相同的数据才去掉





## 连接查询

从一张表中单独查询，称为单表查询。多张表联合起来查询数据，叫做连接查询。

根据表连接的方式分类：

- 内连接：
  - 等值连接
  - 非等值连接
  - 自连接
- 外连接
  - 左外连接（左连接）
  - 右外连接（右连接）
- 全连接



### 内连接



#### 等值连接

 查找每个工作人员的工作地点：

> 显然需要emp表中的ename与dept表中的dname连接
>
> SQL92 语法： ` select ename, dname from emp, dept where emp.deptno = dept.deptno;`
>
> 可以给表起别名提高效率：` select ename, dname from emp e, dept d where e.deptno = d.deptno;`

> SQL99语法：` select ename, dname from emp e inner join dept d on e.deptno = d.deptno;` //inner代表内连接，可以省略。

一定要加限制条件，如果表连接不加限制条件，会出现**笛卡尔积**现象



#### 非等值连接

找出每个员工的薪资等级，要求显示员工名，薪资，薪资等级。

> ` select ename, sal, grade from emp e join salgrade s on e.sal between
> s.losal and s.hisal;`



#### 自连接

查询员工的上级领导，要求显示员工名和对应的领导名

> 信息都在emp表中，员工对应的mgr即为领导编号，同时每个员工都有自己的编号empno。
>
> 我们可以把emp看成两张表a， b。a表中的mgr与b表的empno，则为对应领导
>
> ` select a.ename as '员工名', b.ename as '领导名' from emp a join emp b on a.mgr = b.empno;`



### 外连接

在内连接中，我们只显示完全符合限定条件的数据，如查询员工工作地点：`  select e.ename, d.dname from emp e join dept d on e.deptno = d. deptno;`

在`dept`表中，有的地点没有工作人员，那么查询的结果就不会显示，但如果我们想让这些数据也显示出来呢？在`join`前加个`right`（因为`dept`在`join`右边）



### 三张表，四张表怎么连接

语法：

```mysql
select
	...
from
	a
join
	b
on
	a和b的连接条件
join
	c
on
	a和c的连接条件
join
	d
on
	a和d的连接条件
```

eg：找出每个员工的部门名称以及工资等级

> ` select e.ename,e.sal,d.dname,s.grade from emp e join dept d on e.deptno = d.deptno join salgrade s on e.sal between s.losal and s.hisal;`





## 子查询

`select`语句中嵌套`select`语句，被嵌套的`select`语句称为子查询

子查询可以出现在哪：

```mysql
select
	..(select).
from 
	..(select).
where
	..(select).
```





### where中的子查询

找出比最低工资高的员工姓名和工资：

是不是想这样：`select ename,sal from emp where sal > min(sal);`

**错了！！**==`where`子句中不能直接用分组函数！==

应该如何？：

```mysql
//第一步：查询最低工资是多少
select min(sal) from emp;

//第二步：查出来是800了，找出大于800的
select ename,sal from emp where sal > 800;

//第三步：合并
select ename,sal from emp where sal > (select min(sal) from emp);

//括号中的就是子查询
```



### from中的子查询

注意：`from`后面的子查询可以将子查询的查询结果当作一张临时表

eg：找出每个岗位的平均工资的薪资等级

```mysql
//第一步：找出每个岗位的平均工资
select job, avg(sal) from emp group by job;

//第二步：把上面的查询结果当作一张表t
select 
	t.*,s.grade 
from 
	(select job, avg(sal) as avgsal from emp group by job) t 
join 
	salgrade s 
on 
	t.avgsal between s.losal and s.hisal;
```



### select后的子查询

找出每个员工的部门名称，要求显示员工名，部门名：

除了我们之前学过的写法，还可以这样嵌套

```mysql
 select 
 	e.ename, 
 	(select d.dname from dept d where e.deptno = d.deptno) as dname 
 from 
 	emp e;
```





### union合并查询结果集

eg：查询工作岗位是MANAGER和SALESMAN的员工

` select ename, job from emp where job = 'manager' or job = 'salesman';`

`select ename, job from emp where job in('manager','salesman');`

我们也可以使用`union`：

 `select ename, job from emp where job = 'manager' union select ename, job from emp where job = 'salesman';`

> `union`效率更高，可以减少匹配的次数。
>
> 例如要a连接b连接c，abc各有10条记录：
>
> 如果用正常方法，a连接b：10 * 10 = 100次，结果再次连接c，100 * 10 = **1000**；
>
> 如果使用union，a连接b：10 * 10 = 100； a连接c： 10  * 10 = 100；100 + 100 = **200**；

union使用限制：

- 拼接的两个结果集必须列数相同
- oracle里要求拼接的列和列的数据类型也要一致，mysql不做要求





## limit

limit是将查询结果集的一部分拿出来，通常被使用在分页查询中；

分页的作用是提升用户体验。

eg：按照薪资降序取出前五名的员工

> ` select ename, sal from emp order by sal desc limit 5;`

**`limit startIndex(可省略), length`**

取出工资排名3到5的：

> `select ename, sal from emp order by sal desc limit 2, 3;`

通用公式：`limit (pageNo - 1) * pageSize, pageSize`



## 表的创建

建表的语法格式：（建表属于DDL，DDL包括：`creat, drop, alter`）

`create table 表名(字段名1 数据类型， 字段名2 数据类型， 字段名3 数据类型);`

数据类型：

- varchar: 可变长的字符串（会根据实际的数据长度动态分配空间）
- char：定长字符串
- int
- bigint：类似于long
- double
- float
- date：短日期(只包括年月日) mysql默认格式：`%y-%m-%d`
- datetime：长日期类型（包括年月日时分秒）mysql默认格式：`%y-%m-%d %h:%i:%s`
- clob：字符大对象(Character Large Object)，最多可以存储4G的字符串，可以存储文章，说明
- blob：二进制大对象(Binary Large Object)专门存储图片，声音， 视频等流媒体数据。需要使用io流

eg：创建一个学生表

> ` create table t_student(no int, name varchar(32), sex char(1), age int(3), email varchar(255));`

若想删除，则`drop table t_student;`//若删除的表不存在则会报错

`drop table if exists t_student;`//若存在则删除

**表的复制**：`create table emp2 as select * from emp;` 相当于把查询结果当作一张表新建



### 插入数据（insert）

格式：`insert into 表名(字段名1,字段名2,字段名3...) values(值1,值2,值3);`

eg:` insert into t_student(no, name,sex,age,email) values(1,'zhangsan','m',20,'zhangsan@xx.com');`

建表时可以指定默认值，在字段后面加个`default xxx`就行。eg：` create table t_student(no int, name varchar(32), sex char(1) default 'm', age int(3), email varchar(255));`

`insert`语句的字段名可以省略，但是后面数据必须全写上。



`str_to_date('01-10-1990', '%d-%m-%y')` 

`%y-年 %m-月 %d-日 %h-时 %i分 %s-秒 `

但如果字符串格式符合`%y-%m-%d`可以不用`str_to_date`函数



#### 插入多条记录

` insert into t_student() values(2, 'liuhaoze', 'm',19, 'haozee@foxmail.com'), (3, 'ergo', 'f', 18, 'xxx@123.com');`



#### 将查询结果插入一张表中

`insert into xxx select * from xx;`

### 修改update

`update 表名 set 字段名1=值1,字段名2=值2,字段名3=值3...where条件;` 若无条件则全部数据都会更新



### 删除delete

`delete from 表名 where 条件;` 若没有条件就会删除整张表的内容

delete支持回滚，但是删除效率比较低



#### truncate删除

`truncate table xxx`

物理删除，不支持回滚但是快速。





## 约束constraint

约束包括：

- 非空约束: `not null`
- 唯一性约束: `unique`
- 主键约束: `primary key`（PK）
- 外键约束: `foreign key`（FK）
- 检查约束: `check`(mysql不支持， oracle支持)



### 非空约束

非空约束`not null`约束的字段不能为null

`create table t_vip(id int, name varchar(255) not null);`



### 唯一性约束

`unique`约束的字段不能重复

`create table t_vip(id int, name varchar(255) unique);`

#### 联合约束

限制多个字段联合约束（即这几个字段全部一样时才算重复）

`create table t_vip(id int, name varchar(255), email varchar(255), unique(name, email));`  name和Email联合唯一



unique和 not  null可以联用：`create table t_vip(id int, name varchar(255) not null uique);`  

> mysql中，一个字段如果同时被notnull和unique约束，会自动变为主键字段（oracle中不会）



#### 主键约束（primary key）

主键是**每一行记录**的唯一标识，不能为空且值不能重复。（`not null & unique`）

任何一张表都应该有主键，不然无效。

怎么添加主键？：加上`primary key`

```mysql
create table t_vip(
	id int primary key,
    name varchar(255)
);
```

或者用表级约束

```mysql
create table t_vip(
	id int,
    name varchar(255),
    primary key(id)
);
```



也可以让多个字段当作主键（**复合主键**， 几个主键全一样才算做重复，实际开发中不建议使用复合主键）

主键值建议使用`int, bigint, char`等类型，不建议使用`varchar`来做主键，主键值一般都是数字且定长



主键分为自然主键和业务主键：

- 自然主键：主键值是一个自然数，和业务没关系。
- 业务主键：主键值和业务紧密关联，例如拿银行卡号当主键

实际开发中使用自然主键多，因为主键只要做到不重复就行，如果与业务挂钩要发生变动时就不方便了

##### 自动维护主键值的机制

```mysql
create table t_vip(
	id int primary key auto_increment, //auto_increment表示从1开始自增
    name varchar(255)
);
```



#### 外键约束（foreign key）***

业务背景：请设计数据库表，描述班级和学生的信息。

- 方案1：班级和学生存储在一张表中
  no(pk)  name  classno  classname
  缺点：数据冗余，空间浪费（同一个班级的classname和classno都一样，完全重复）
- 方案2：班级一张表，学生一张表
  t_class
  classno(pk)  classname
  t_student
  no(pk)  name  cno

在方案2中，如果cno没有约束，可能出现一个t——class中没有的数据，就会导致数据无效，这时候就要用到**外键约束**了

```mysql
create table t_class(
	classno int primary key,
    classname varchar(255)
);

create table t_student(
	no int primary key auto_increment,
    name varchar(255),
    cno int,
    foreign key(cno) references t_class(classno)//外键约束
);
```

外键引用的字段必须unique，外键值可以为NULL



## 存储引擎

存储引擎是一个表存储 / 组织数据的方式

### 如何给表指定存储引擎？

`show create table t_student`展示完整建表语句，我们发现在建表的时候可以在最后小括号右边使用：`engine = xxx(默认为InnoDB) charset(字符编码方式) = xxx(默认为utf8) `

eg:

```mysql
create table  t_product(
	id int primary key,
    name varchar(255)
)engine = InnoDB;
```



### Mysql支持哪些存储引擎

命令：`show engines \g` 可以显示出mysql支持的9个引擎，版本不同对这9个的支持情况不同

常用引擎：

- `MyISAM`:使用三文件表示每个表：

  - 格式文件 — 存储表结构的定义（`mytable.frm`）

  - 数据文件 — 存储表行的内容（`mytable.MYD`）

  - 索引文件 — 存储表上索引（`mytable.MYI`） 

    

    灵活的`AUTO_INCREMENT`字段处理

    可被转换为压缩、只读表来节省空间

- `InnoDB`:Mysql的缺省引擎，同时也是个重量级的存储引擎。
  支持事务，支持数据库崩溃后自动恢复机制。他的主要特点：非常安全，支持事务机制

- `MEMORY`:数据存储在内存当中，行的长度固定，使得它速度非常之快，但不安全



## 事务

一个事务其实就是一个完整的业务逻辑

假设要从A账户向B账户转账1000，则：

- 将A账户的钱减去1000（update语句）
- 将B账户的钱加上1000（update语句）

这就是一个完整的业务逻辑，以上的操作是一个最小的工作单元，要么同时成功，要么同时失败，不可再分。



### 事务怎么做到全部成功或失败？

`InnoDB`存储引擎：提供一组用来**记录事务性活动的日志文件**，在事务执行过程中，每一条DML的操作都会记录到这个日志文件中，在事务执行过程中我们可以**提交事务**，也可以**回滚事务**

> **提交事务**：清空事务性活动的日志文件，将数据全部彻底持久化到数据库表中。
> 提交事务标志着事物的结束，并且是一种全部**成功**的结束。
>
> **回滚事务**：将之前所有的DML操作全部撤销，并且清空日志文件。
> 回滚事务标志着事物的结束，并且是一种全部**失败**的结束。



### 具体实现

提交事务：`commit`

回滚事务：`rollback`   注意事务的单词是：`transaction`

mysql默认情况下每执行一条语句都是默认提交，怎么关闭自动提交？—— `start transaction`



### 事物的四个特性

1. A:原子性
   事务是最小的工作单元，不可再分
2. C：一致性
   所有事务要求，在同一个事务中，所有操作必须同时成功或同时失败，以保持数据的一致性
3. I：隔离性
   A事务与B事务之间具有一定的隔离。如果A事务在操作一张表的时候，B事务也在操作这张表会出事
4. D：持久性
   事务最终结束的一个保障。事务提交，就相当于将没有保存到硬盘上的数据保存到硬盘上

#### 隔离性

事务间存在隔离级别：

- 读未提交：`read uncommitted`（最低级别）
  事务A可以读取到事务B未提交的数据，存在问题：**脏读**现象（`Dirty Read`）
  实际中很少用到这个级别，都是第2级起步

- 读已提交：`read committed`
  A只能读取到B已提交的数据。解决了 脏读。但是存在**不可重复读取数据**的问题

  > 不同的时刻读到的同一批数据可能是不一样的，可能会受到其他事务的影响，比如其他事务改了这批数据并提交了。

  oracle数据库默认隔离级别就是这个。

- 可重复读：`rapeatable read`
  事务A开始之后，不管多久，每次在A中读取到的数据都是一样的。(另个事务就算提交了，在A中还是读不到B中修改的数据)可能存在**幻影读 / 幻读**现象：每次读到的数据都可能是虚幻

  Mysql的默认隔离级别

- 序列化 / 串行化：`serializable`(最高级别)
  事务必须都排队，不能并发。



设置隔离级别：`set global transaction isolation level xxxx;`



## 索引

索引是在数据库表的字段上添加的，目的是为了提高查询效率。一张表的一个字段或者多个字段联合可以添加一个索引。索引相当于一本书的目录，是为了缩小扫描范围而存在的一种机制。

底层是`B-Tree`（自平衡二叉树）数据结构，遵循左小右大存放，采用中序遍历方式取数据。



### 索引分类

- 单一索引：一个字段上添加索引
- 复合索引：两个或更多字段上添加索引
- 主键索引：主键上添加索引
- 唯一性索引：具有`unique`约束的字段上添加索引

注意：唯一性比较弱的字段上添加索引用处不大

### 索引的实现原理

1. 主键会自动添加索引，被unique约束的字段也会自动创建索引对象。
2. 任何一张表的任何一条记录在硬盘存储上都有一个硬盘的物理存储编号
3. Mysql中索引是一个单独的对象，不同的存储引擎以不同的形式存在。



什么条件我们会考虑给字段添加索引？

1. 数据量庞大
2. 该字段经常出现在`where`后以条件的形式存在，即这个字段总是被扫描
3. 该字段很少的DML（增删改查）操作。因为DML后索引需要重新排序。



### 索引的创建与删除

创建索引：
	`create index index_name on table_name(column1[asc(默认) / desc], column2...);`

删除索引：
	`drop index index_name on table_name;`



### 索引的失效

1. 模糊查询以 '%'开头 如：`select * from emp where ename like '%T';`
   所以尽量避免模糊查询的时候以'%'开头
2. 使用'or'的时候。如果使用'or'那么两边的条件字段都有索引才会走索引。因此不建议使用'or'。
3. 使用复合索引时没有使用左侧的列查找。
4. `where`当中索引列参加了运算
5. 在`where`当中索引列使用了函数





## 视图

`view`：站在不同的角度区块带同一份数据



### 视图的创建与删除

创建：`create view view_name as select x from table_name;`

删除：`drop view view_name;`

注意：只有DQL语句才能以view的形式创建。



### 视图可以做什么？

对视图增删改查，也会影响到源表的数据。





## 数据库设计三范式

1. 要求任何一张表必须有主键，每一个字段原子性不可再分。

2. 建立在第一范式的基础之上，村求所有非主键字段必须完全依赖主键，不要产生部份依赖。

   > 部份依赖缺点：数据冗余，空间浪费

3. 建立在第二范式的基础上，所有非主键字段必须直接依赖主键，不要产生传递依赖。
