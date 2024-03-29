# java笔记



## java最低学习路线：

1. 语言基础
2. 框架：javaweb~简单过一下~ + spring~比较重要~ + springmvc~简单了解~ + mabatis~简单了解~ + springboot~比较重要，简单学完~ 后面还可以学微服务
3. 直接做项目（两个）

> 项目技术栈最少涉及springboot + redis

4. 复习八股文：

重点：java（集合 + 面对对象概念 + 多线程基础） jvm 基础 mysql + redis 计算机网络

##  1. Java的逻辑运算符

java 中的 `&`和`|`相当于 c 里面的 `&& , ||`, 但 java 中也有`&&, ||`,它们与单个的区别是**当逻辑运算前半部分的结果足以推导出最终结果时，会直接停止运算以节省资源**



## 2. 静态

关键字`static`：某个类公用的，普遍的属性，创建时加上前缀`static`,使用时可以直接对类使用。

> ```java
> public class Main{
>     public static void main(String[] args){
>         Bird test = new Bird();
>         test.name = "jiujiu";
>         Bird.fly();
>         System.out.println(test.name);
>     }
> }
> 
> class Bird{
>     String name;
>     static void fly(){
>         System.out.println("Fly away......");
>     }
> }
> ```
>
> 

**注意**：先有类，再有对象 ： 

1. **成员方法可以访问静态属性和静态方法**  
2. **静态方法不可以访问成员属性和成员方法**。

eg:

```java
class Test{
    String name;
    static void s_test(){
        System.out.println(name);//这里会报错，因为在s_test这个静态方法里不能访问name这个成员属性。
    }
}
```

```java
class Test{
    String name;
    void test(){
        s_test();//这里就没事，因为成员方法可以访问静态方法
    }
    static void s_test(){
        System.out.println("s_test...");
    }
}
```



### 静态代码块

形如`static{......}`为静态代码块。**类的信息加载完成后，会自动运行静态代码块**。

eg:

```java
public class Main{
    public static void main(String[] args){
        Test.test();//如果是 new Test(); 则运行结果为 static code block
    }
}

class Test{
    static{
        System.out.println("static code block");
    }
    static void test(){
        System.out.println("test");
    }
}
/*运行结果为：
static code block
test
*/
```

对应的，对象准备创建时，会自动调用**代码块**（要和前面的静态代码块做区分，就是不带`static`直接一个花括号括住）

eg：

```java
public class Main{
    public static void main(String[] args){
        Test a = new Test();
    }
}

class Test{
    static{
        System.out.println("static code block");
    }
    {
        System.out.println("code block");
    }
    static void test(){
        System.out.println("test");
    }
}/*输出结果为
static code block
code block
*/
```





## 3. 继承(extend)

面对对象有三个重要特点：1. 继承 2. 封装 3. 多态

类存在父子关系，子类可以直接获取父类的成员属性和成员方法，一个父类可以有多个子类。

### super & this

如果父类子类有同名的属性，则`this`代表子类的 ，`super`代表父类的。

eg：

```java
public class Main{
    public static void main(String args[]){
        
    }
}

class Parent{
    String name = "zhangsan";
}

class Child extends Parent{
    String name = "lisi";
    
    void test(){
        System.out.println(super.name);//这个是zhangsan
        System.out.println(this.name);//这个是李四
        System.out.println(name);//无前缀默认子类，所以李四
    }
}
```



- 构建子类对象时会先调用一遍父类的构造方法
- 如果父类没有提供构造方法，JVM会自动默认使用无参的构造方法，不需要自己写
- 如果父类提供构造方法，那么JVM不会提供默认的构造方法，子类应该显式地调用super方法构建父类对象

eg：

```java
public class Main{
    public static void main(String[] args){
        Child c1 = new Child();
        Child c2 = new Child();
    }
}

class Parent{
    String username;

    Parent(String name){
        username = name;
        System.out.println("parent...");
    }
}

class Child extends Parent{
    Child(){
        super("test");
        System.out.println("child...");
    }
    String name;
}
/*
输出结果：
test
child...
test
child...
*/
```





## 4. 多态



- 所谓多态，就是一个对象在不同的场景下表现出来的不同状态和形态
- 多态语法其实就是对对象的使用场景进行了约束
- 一个对象可以使用的功能取决于引用变量的类型

eg：

```java
public class Main{
    public static void main(String args[]){
        Person p = new Person();
        p.testPerson();
        
        Person p1 = new Boy();//可以编译过
        p1.testPerson();//可以
        p1.testBoy();//错误，这里体现了多态对对象的使用场景的约束
        
        Person p2 = new Girl();//可以
        p2.testPerson();//可以
        p2.testGirl();//错误，同上。
    }
}

class Person{
    void testPerson(){
        System.out.println("test person......");
    }
}

class Boy extends Person{
    void testBoy(){
        System.out.println("test boy......");
    }
}

class Girl extends Person{
    void testGirl(){
        System.out.println("test girl......");
    }
}//Girl & Boy 相当于 Person 的不同形态
```





## 5.方法重写

- 一个对象能使用什么方法，取决于引用变量的类型。（也就是左边的前缀类型） 对象的属性也一样

  **编译看左边，运行看右边**

- 一个对象方法的具体使用需要看具体的对象（也就是右边初始化的类型）|| **但属性是在哪里声明的就在哪里使用**



eg：

```java
public class Main{
    public static void main(String[] args){
        Parent p = new Child();
        System.out.println(p.sum());//sum方法的具体使用要看具体的对象（右边初始化的类型），即Child，所以最后结果是40；
    }
}

class Parent{
    int i = 10;
    int sum(){
        return i + 10;
    }
}

class Child extends Parent{
    int i = 20;

    int sum() {
        return i + 20;
    }//如果删掉Child的sum，只保留 i = 20;程序最后结果为20
}
```





```java
public class Main{
    public static void main(String[] args){
        Parent p = new Child();
        System.out.println(p.sum());//运行sum方法时，由于子类没有，所以走父类的sum(),而sum()中的getI()的具体使用要看右边具体的对象，即Child，所以getI为20，然后+10得30，最后结果为30.
    }
}

class Parent{
    int i = 10;
    int sum(){
        return getI() + 10;
    }
    int getI(){
        return i;
    }
}

class Child extends Parent{
    int i = 20;
    int getI(){
        return i;
    }
}
```







## 6.访问权限

- java的源码中，公共类(前面带`public`的)只能有一个，而且必须跟源码文件名相同。

java中的访问权限主要分四种：

1. `private`：私有的，只有同一个类中可以使用
2. `(default)`：默认权限 / 包路径访问权限，当不设定任何权限时，JVM会默认提供一个**包权限**（即同一个包里才可以使用，其他的包或者**子包**都不行）
3. `protected`：受保护的权限，子类可以访问
4. `public`：公共的，任意使用。

> java不允许外部类(源码中直接声明的类)使用`private || protected`修饰
>
> 只有内部类才能用，即类中声明的类。内部类可以看作外部类的属性，所以需要**构建外部类对象**才可以使用。

eg:

```java
public class Main{
    public static void main(String[] args){
        OuterClass outer = new OuterClass();
        OuterClass.InnerClass inner = outer.new InnerClass();
    }

}

class OuterClass{
    public class InnerClass{

    }
}
```



## 7. equals()

Java的equals方法是用来比较两个对象是否相等的。它是Object类的一个方法，所以所有的类都可以继承和重写它。equals方法的一般语法是：

```java
public boolean equals(Object obj)
```

 

其中，obj是要比较的另一个对象。equals方法返回一个布尔值，表示两个对象是否相等。

要正确地使用equals方法，需要遵循以下几个原则：

- 自反性：对于任何非空对象x，x.equals(x)应该返回true。
- 对称性：对于任何非空对象x和y，x.equals(y)应该返回true当且仅当y.equals(x)也返回true。
- 传递性：对于任何非空对象x，y和z，如果x.equals(y)返回true，并且y.equals(z)也返回true，那么x.equals(z)也应该返回true。
- 一致性：对于任何非空对象x和y，如果没有修改它们的状态，那么多次调用x.equals(y)应该始终返回相同的结果。
- 非空性：对于任何非空对象x，x.equals(null)应该返回false。

不同的类可以根据自己的需求来实现equals方法。例如，String类的equals方法是用来比较两个字符串是否包含相同的字符序列的。如果两个字符串包含相同的字符序列，那么它们就被认为是相等的。例如：

```java
String s1 = "Hello";
String s2 = "Hello";
String s3 = "World";
System.out.println(s1.equals(s2)); // 输出true
System.out.println(s1.equals(s3)); // 输出false
```

 

注意，String类的equals方法是区分大小写的。如果要忽略大小写进行比较，可以使用equalsIgnoreCase方法。例如：

```java
String s1 = "Hello";
String s2 = "HELLO";
System.out.println(s1.equals(s2)); // 输出false
System.out.println(s1.equalsIgnoreCase(s2)); // 输出true
```

 

另一方面，Number类的equals方法是用来比较两个数值是否相等的。它会考虑数值的类型和精度。如果两个数值类型不同，或者精度不同，那么它们就被认为是不相等的。例如：

```java
Integer i1 = 10;
Integer i2 = 10;
Double d1 = 10.0;
Double d2 = 10.0;
System.out.println(i1.equals(i2)); // 输出true
System.out.println(i1.equals(d1)); // 输出false
System.out.println(d1.equals(d2)); // 输出true
```



注意，Number类的equals方法和 "`==` " 运算符有所不同。"`==`" 运算符只能用来比较基本类型的数值，而不适用于包装类型的数值。如果要比较包装类型的数值是否相等，可以使用equals方法或者将其转换为基本类型后再使用”==”运算符。例如：

```java
Integer i1 = 10;
Integer i2 = 10;
Double d1 = 10.0;
Double d2 = 10.0;
System.out.println(i1 == i2); // 输出true
System.out.println(i1 == d1); // 编译错误
System.out.println(i1.doubleValue() == d1); // 输出true
System.out.println(d1 == d2); // 输出false
```

 



## 8. final

使用关键词`final`可以使数据初始化后不被修改（类似于 cpp 中的`const`）

- `final`可以修饰变量也可以修饰属性，但它修饰的变量或属性必须手动初始化
- `final`可以修饰方法，这个方法不能被子类重写（`final`不能修饰构造方法）
- `final`可以修饰类，这个类不能被继承
- `final`可以修饰方法参数，这个参数传入之后无法修改



## 9. 抽象  - abstract

- 抽象类：不完整的类，就是抽象类

> `abstract class Name_of_class`

因为抽象类不完整，所以无法构造对象，但可以通过子类间接构造对象

- 抽象方法：只有声明，没有实现的方法

> `abstract 返回值类型 方法名 （参数)`

注意：如果抽象类中有抽象方法，那么子类继承抽象类，需要重写抽象方法，将方法补充完整。

==又因为`final` 修饰的方法不能被子类重写，所以`final` & `abstract`不能同时使用==



## 10. 接口 - interface

- 所谓的接口，可以简单理解为规则
- 基本语法：`interface 接口名称 { 规则属性， 规则的行为 }`
- 规则的属性必须为固定的，且不能修改
- 属性和行为的访问权限必须为公共的
- 属性应该是静态的
- 行为应该是抽象的 **(接口中的类和方法前面不用加`abstract`关键字，因为 JVM 默认加上了)**
- 接口可以继承其他接口
- 类的对象需要遵循接口，在java中 ,**“遵循”**称之为**实现**,类需要实现接口，且可以实现多个接口。
- 接口中的方法必须是：1. 抽象方法 2. 默认实现方法（前面加`default`关键字）3.静态方法

eg：

创建接口

```java
public class Main{
    public static void main(String[] args){
    }
}

interface USBInterface{ //接口
    public void start();
    public void stop();
}
```



实现接口 (实现接口的类： `public class 类名 implements 接口名`)

```java
public class Phone implements USBInterface{
    @Override
    public void stop() {
        System.out.println("手机停止工作");
    }

    @Override
    public void start() {
        System.out.println("手机开始工作");
    }
}
```

```java
public class Camera implements USBInterface{
    @Override
    public void start() {
        System.out.println("相机开始工作。。。");
    }

    @Override
    public void stop() {
        System.out.println("相机停止工作...");
    }
}
```



使用接口

```java
public class Computer {
    public void work(USBInterface usbInterface) {
        usbInterface.start();
        usbInterface.stop();
    }
}//相当于使用接口的方法
```





## 11.枚举 enum

- Java 枚举是一个特殊的类，一般表示一组常量，比如一年的 4 个季节，一年的 12 个月份，一个星期的 7 天，方向有东南西北等。
- 所有的枚举值都是 public static final 的,所以一般大写命名

eg:

```java
enum Color
{
    RED, GREEN, BLUE;
}
```



- 枚举跟普通类一样可以用自己的变量、方法和构造函数，构造函数只能使用 private 访问修饰符，所以外部无法调用。

  枚举既可以包含具体方法，也可以包含抽象方法。 如果枚举类具有抽象方法，则枚举类的每个实例都必须实现它。

  

- ### values(), ordinal() 和 valueOf() 方法

  enum 定义的枚举类默认继承了 java.lang.Enum 类，并实现了 java.lang.Serializable 和 java.lang.Comparable 两个接口。

  values(), ordinal() 和 valueOf() 方法位于 java.lang.Enum 类中：

  - values() 返回枚举类中所有的值。

  - ordinal()方法可以找到每个枚举常量的索引，就像数组索引一样。

  - valueOf()方法返回指定字符串值的枚举常量。

    ```java
    enum Color 
    { 
        RED, GREEN, BLUE; 
    } 
      
    public class Test 
    { 
        public static void main(String[] args) 
        { 
            // 调用 values() 
            Color[] arr = Color.values(); 
      
            // 迭代枚举
            for (Color col : arr) 
            { 
                // 查看索引
                System.out.println(col + " at index " + col.ordinal()); 
            } 
      
            // 使用 valueOf() 返回枚举常量，不存在的会报错 IllegalArgumentException 
            System.out.println(Color.valueOf("RED")); 
            // System.out.println(Color.valueOf("WHITE")); 
        } 
    }

输出结果：

```markdown
RED at index 0
GREEN at index 1
BLUE at index 2
RED
```





## 12. 匿名类

假如我们写一个抽象类 “人”，其中有抽象方法“活着”，后面我们要写诸如“老师”，“工人”等类来继承“人”

抽象类 People

```java
public abstract class People
{
    public abstract void live();
}
```



新建 Teacher 类继承 People，并重写其live方法

```java
public class Teacher extends People {
    @Override
    public void live() {
        System.out.println("老师在努力地活着");
    }
}
```



最后

```java
public class Demo {
 	public static void main(String[] args) {
  		Teacher t = new Teacher();
  		t.run();
 	}
}
```

结果为`老师在努力地活着`



如果我们想再加`学生在努力地活着 ， 工人在努力地活着。。。。。`等等等等，每次都要再新建个学生类，工人类，太过麻烦，这时我们的**匿名类**堂堂登场

people类不变，不再需要新建Teacher类，而是直接在Demo中如此：

```java
public class Demo{
    public static void main(String[] args){
        People teacher = new People() {
            @Override
            public void live() {
                System.out.println("老师在努力地活着");
            }
        };
        teacher.live();
    }
}
```





## 13.异常(Exception)

1. 可以通过代码恢复正常逻辑执行的异常，称之为运行期异常：`RuntimeException`
2. 不可以通过代码恢复正常逻辑执行的异常，称之为编译期异常：`Exception`

语法：

捕捉多个异常的时候，需要先捕捉范围小的异常，然后范围大的异常

```java
/*try{
    放可能会出现异常的代码
    如果出现异常，JVM会将异常进行封装，形成一个具体的异常类，然后将这个异常抛出
} catch(抛出的异常对象 对象引用){
    
} catch(多个catch是为了处理多个异常，可以有任意个){
    
}finally{
    最终执行的代码
}
*/

int i = 0;
int j = 10 / i;

try{
    j = 10 / i;
} catch(ArithmeticException e1){
    //e.getMessage(); 错误的消息
    //e.getCause();错误的原因
    //e.printStackTrace();打印异常信息在程序中出错的位置及原因。
    i = 10;
    j = 10 / i;
} finally{
    System.out.println("最终执行的代码，不管是否异常")
}
```



### 常见异常

1. 除数为0的算数异常 (**运行期异常**)  `java.long.ArithmeticException`

   ```java
   int i = 3;
   int j = 0;
   int x = i / j;
   ```

   

2. 空指针异常(调用了一个为空的对象的成员属性或成员方法)**运行期异常** `java.long.NullPointerException`

   ```java
   User3 user = null;
   System.out.println(user.toString());
   
   class User3{
       
   }
   ```

   

3. 数组越界`ArrayIndexOutOfBoundsException`**运行期异常**

   ```java
   String[] names = new String[3];
   names[0] = "zhangsan";
   names[1] = "lisi";
   names[2] = "wangwu";
   names[3] = "zhaoliu";//越界了
   ```

   

4. 字符串索引越界`StringIndexOutOfBoundsException`

   ```java
   String s = "abc";
   System.out.println(s.CharAt(3));
   ```

   

5. 格式化异常`NumberFormatException`

   ```java
   String s = "a123";
   Inter i = Inter.parseInt(s);
   
   System.out.println(i);
   ```

6. 类型转换错误`java.long.ClassCastException`

```java
main ...	
	Object obj = new User5();
	Emp5 emp =(Emp5) obj;

    
class user5{
        
}

class Emp5{
    
}
```



### throws

自己写的方法，如果可能会出问题，则需要提前声明，告诉其他人我的方法可能会出问题。此时需要关键字`throws`。

它相当于**“甩锅”**，将异常抛到调用该方法的地方处理，调用这个方法的地方就需要使用**`try catch`**来处理解决了。

```java
public class Main {
    public static void main(String[] args) {

        int a = 5;
        int b = 0;
        try {
            Test.test(a, b);
        }
        catch (Exception e){
            System.out.println(e.fillInStackTrace());
        }
    }
}

class Test{
    static void test(int i, int j) throws Exception{
        System.out.println(i / j);
    }
}

```



### throw

`throw` ：手动抛出异常

```java
public void checkNumber(int num) {
  if (num < 0) {
    throw new IllegalArgumentException("Number must be positive");
  }
}
```



### 自定义异常类

写一个类，继承一个已有的异常类就行了。





## 多线程

- **程序(program)**：为完成特定任务，用某种语言编写的一组指令的集合。即指一段静态的代码，静态对象。
- **进程(process)**：程序的一次执行过程，或是正在内存中运行的应用程序。

> 每个进程都有一个独立的内存空间，系统运行一个程序即是一个进程从创建，运行到消亡的过程（生命周期）
>
> 程序是静态的，进程是动态的
>
> 进程作为**操作系统调度和分配资源的最小单位**（亦是系统运行的基本单位），系统在运行时会为每个进程分配不同的内存区域

- **线程(thread)**：进程可进一步细化为线程，是程序内部的**一条执行路径**，一个进程中至少有一个线程

> 线程是**CPU调度和执行的最小单位**
>
> 一个进程中的多个线程共享相同的内存单元，他们从一个堆中分配对象，可以访问相同的变量和对象。

### 抢占式调度

JAVA使用抢占式调度，让优先级高的线程以较大的概率优先使用CPU，如果现成的优先级相同，那么随机选择一个（线程随机性）



### 继承`Thread`类创建线程

1. 定义`Thread`类的子类，并重写该类的`run()`方法~该方法的方法体就代表线程需要完成的任务，所以又叫**线程执行体**~
2. 创建`Thread`类的实例，即创建了线程对象
3. 调用线程对象的`start()`方法来启动该线程

```java
import java.util.Scanner;

public class Main {
    public static void main(String args[]) {
            PrintNumber temp = new PrintNumber();
            temp.start();
        }
    }

class PrintNumber extends Thread{
    @Override
    public void run() {
        for (int i = 0; i <= 100; i++)
            if (i % 2 == 0) System.out.println(Thread.currentThread().getName() + i);
    }
}
```



### 实现Runnable接口创建线程

1. 定义`Runnable`接口的实现类，并重写该接口的`run()`方法
2. 创建`Runnable`实现类的实例，并以此实例作为参数传入`Thread`的构造器中，来创建`Thread`类的实例。

```java
import java.util.Scanner;

public class Main {
    public static void main(String args[]) {
            PrintNumber temp = new PrintNumber();
            Thread t1 = new Thread(temp);
            t1.start();
        }
    }

class PrintNumber implements Runnable{
    @Override
    public void run() {
        for (int i = 0; i <= 100; i++)
            if (i % 2 == 0) System.out.println(Thread.currentThread().getName() + i);
    }
}
```



### 线程的常用结构

1. 线程中的构造器

- `public Thread()`:分配一个新的线程对象
- `public Thread(String name)`：分配一个指定名字的新的线程对象
- `public Thread(Runnable target)`：指定创建线程的目标对象，它实现了`Runnable`接口中的`run()`方法
- `public Thread(Runnable target, String name)`：分配一个带有指定目标新的线程对象并指定名字

2. 线程中的常用方法

- `start()`：-> 启动线程 -> 调用线程的`run()`
- `run()`：将线程要执行的操作，声明在`run()`中
- `currentThread()` ：获取当前执行代码对应的线程
- `getName()`：获取线程名
- `setName()`：设置线程名
- `sleep(long millis)`：静态方法，调用时，可以使当前线程睡眠指定的毫秒数

> 注意，`sleep`默认会抛出一个非运行时异常，需要处理。若要在`run`方法中处理只能**`try catch`**处理而不能再`throws`，因为他继承的`Thread`中的`run`方法没有抛出异常

- `yield()`：静态方法。执行此方法时，释放当前进程CPU的执行权。
- `join()`：在线程a中通过线程b调用`join()`，意味着线程a进入阻塞状态，直到线程b执行结束，a才继续进行。（其实就相当于插队）

- `isAlive()` 判断线程是否存活  

线程的优先级相关

- `getPriority()`：获取线程的优先级

> 线程默认的优先级为5；`Thread`类中有三个优先级常量
>
> - `MAX_PRIORITY`：为10，最高优先级
> - `MIN_PRIORITY`：为1，最低优先级
> - `NORM_PRIORITY`：为5，普通优先级

- `setPriority(int newPriority)`：改变线程的优先级



### 线程的生命周期

**JDK1.5之前：5种状态**

新建(New)  就绪(Runnable)   运行(Running)  阻塞(Blocked)  死亡(Dead) 

CPU需要在多条线程之间切换，于是线程状态会多次在运行，阻塞，就绪之间切换。

**JDK1.5后： 六种状态**

1.新建(`New`) 2. 可运行(`Runnable`) 3. 终止(`Terminated`) 4. 阻塞(`Blocked`) 5. 等待(`Waiting`) 6. 定时等待(`Timed Waiting`)



### 通过同步解决线程安全问题

假设我们要卖100张票，建立三个线程分别代表三个窗口卖票。当某个线程进入到**判断是否有票后，票数减这一操作还没进行时**这一线程就**阻塞**，然后进行另一个线程了，这时就会造成卖重票的问题。如何解决呢？

1. **同步代码块**

```java
synchronized(同步监视器){
    //需要被同步的代码
}
```

说明：

- 需要被同步的代码，即为操作共享数据的代码
- 共享数据：即多个线程都需要操作的数据，比如：多个售票口线程共同卖的`ticket`
- 需要被同步的代码，在被`synchronized`包裹以后，就使得一个线程在操作这些代码的过程中，其他线程必须等待 
- 同步监视器，俗称锁，哪个线程获取了锁，哪个线程就能执行需要被同步的代码
- 同步监视器：可以使用任何一个类的对象充当，但多个线程必须共用一个同步监视器

```java
public class Main {
    public static void main(String args[]) {
        SaleTicket sale = new SaleTicket();
        Thread t1 = new Thread(sale);
        t1.setName("窗口1");
        Thread t2 = new Thread(sale);
        t2.setName("窗口2");
        Thread t3 = new Thread(sale);
        t3.setName("窗口3");

        t1.start();
        t2.start();
        t3.start();
    }
}

class SaleTicket implements Runnable {
    int ticket = 100;
    Object obj = new Object();

    @Override
    public void run() {
        while (true) {
            synchronized (this) {
                if (ticket > 0) {
                    System.out.println(Thread.currentThread().getName() + "售票，票号为：" + ticket);
                    ticket--;
                }
                else break;
            }
        }
    }
}

```

在实现`Runnable`接口的方式中，同步监视器可以考虑使用`this`

在继承`Thread`类的方式中，同步监视器要慎用`this`。可以考虑使用`当前类.class`



2. **同步方法**

如果操作共享数据的代码完整声明在了一个方法中，那么我们就可以将此方法声明为同步方法

```java
 pubilc synchronized void test() {  
     
    System.out.println("方法运行");
}
```

> 非静态的同步方法，默认同步监视器是`this`
>
> 静态的同步方法，默认的同步监视器是当前类本身



`synchronized`**好处**：解决了线程的安全问题

**弊端**：在操作共享数据时，多线程其实是串行的，意味着性能低。







### 死锁

Java的线程锁是可重入的锁。

什么是可重入的锁？我们还是来看例子：

```java
public class Counter {
    private int count = 0;

    public synchronized void add(int n) {
        if (n < 0) {
            dec(-n);
        } else {
            count += n;
        }
    }

    public synchronized void dec(int n) {
        count += n;
    }
}
```

观察`synchronized`修饰的`add()`方法，一旦线程执行到`add()`方法内部，说明它已经获取了当前实例的`this`锁。如果传入的`n < 0`，将在`add()`方法内部调用`dec()`方法。由于`dec()`方法也需要获取`this`锁，现在问题来了：

对同一个线程，能否在获取到锁以后继续获取同一个锁？

答案是肯定的。JVM允许同一个线程重复获取同一个锁，这种能被同一个线程反复获取的锁，就叫做可重入锁。

由于Java的线程锁是可重入锁，所以，获取锁的时候，不但要判断是否是第一次获取，还要记录这是第几次获取。每获取一次锁，记录+1，每退出`synchronized`块，记录-1，减到0的时候，才会真正释放锁。

### 



一个线程可以获取一个锁后，再继续获取另一个锁。例如：

```java
public void add(int m) {
    synchronized(lockA) { // 获得lockA的锁
        this.value += m;
        synchronized(lockB) { // 获得lockB的锁
            this.another += m;
        } // 释放lockB的锁
    } // 释放lockA的锁
}

public void dec(int m) {
    synchronized(lockB) { // 获得lockB的锁
        this.another -= m;
        synchronized(lockA) { // 获得lockA的锁
            this.value -= m;
        } // 释放lockA的锁
    } // 释放lockB的锁
}
```

在获取多个锁的时候，不同线程获取多个不同对象的锁可能导致死锁。对于上述代码，线程1和线程2如果分别执行`add()`和`dec()`方法时：

- 线程1：进入`add()`，获得`lockA`；
- 线程2：进入`dec()`，获得`lockB`。

随后：

- 线程1：准备获得`lockB`，失败，等待中；
- 线程2：准备获得`lockA`，失败，等待中。

此时，两个线程各自持有不同的锁，然后各自试图获取对方手里的锁，造成了双方无限等待下去，这就是死锁。

死锁发生后，没有任何机制能解除死锁，只能强制结束JVM进程。

因此，在编写多线程应用时，要特别注意防止死锁。因为死锁一旦形成，就只能强制结束进程。

那么我们应该如何避免死锁呢？答案是：线程获取锁的顺序要一致。即严格按照先获取`lockA`，再获取`lockB`的顺序，改写`dec()`方法如下：

```java
public void dec(int m) {
    synchronized(lockA) { // 获得lockA的锁
        this.value -= m;
        synchronized(lockB) { // 获得lockB的锁
            this.another -= m;
        } // 释放lockB的锁
    } // 释放lockA的锁
}
```





#### ReentrantLock

除了使用`synchronized`解决线程安全问题，我们还可以使用JDK5.0开始提供的`java.util.concurrent` 包中的`ReentrantLock`解决

使用`synchronized`上锁的代码：

```java
public class Counter {
    private int count;

    public void add(int n) {
        synchronized(this) {
            count += n;
        }
    }
}
```



使用`reentrantLock`代替：

```java
if (lock.tryLock(1, TimeUnit.SECONDS)) {
    try {
        ...
    } finally {
        lock.unlock();
    }
}//为了unlock()一定能得到执行，我们使用try finally包住
```

**注意，一定要确保多个线程同用一个Lock实例**





### wait() & notify()

我们看下面代码：

```java
class TaskQueue {
    Queue<String> queue = new LinkedList<>();

    public synchronized void addTask(String s) {
        this.queue.add(s);
    }

    public synchronized String getTask() {
        while (queue.isEmpty()) {
        }
        return queue.remove();
    }
}
```

上述代码看上去没有问题：`getTask()`内部先判断队列是否为空，如果为空，就循环等待，直到另一个线程往队列中放入了一个任务，`while()`循环退出，就可以返回队列的元素了。

但实际上`while()`循环永远不会退出。因为线程在执行`while()`循环时，已经在`getTask()`入口获取了`this`锁，其他线程根本无法调用`addTask()`，因为`addTask()`执行条件也是获取`this`锁。

因此，执行上述代码，线程会在`getTask()`中因为死循环而100%占用CPU资源。



我们想要的执行效果是：

- 线程1可以调用`addTask()`不断往队列中添加任务；

- 线程2可以调用`getTask()`从队列中获取任务。如果队列为空，则`getTask()`应该等待，直到队列中至少有一个任务时再返回。

  

  我们先改造`getTask()`方法，在条件不满足时，线程进入等待状态：

  ```java
  public synchronized String getTask() {
      while (queue.isEmpty()) {
          this.wait();
      }
      return queue.remove();
  }
  ```

**`wait()`方法必须在当前获取的锁对象上调用，这里获取的是`this`锁，因此调用`this.wait()`。**

注意：**`wait()`方法调用时，会*释放*线程获得的锁，`wait()`方法返回后，线程又会重新试图获得锁。**与`sleep()`不同，sleep会占着锁一直等着。

如何让等待的线程被重新唤醒，然后从`wait()`方法返回？答案是在相同的锁对象上调用`notify()`方法。我们修改`addTask()`如下：

```java
public synchronized void addTask(String s) {
    this.queue.add(s);
    this.notify(); // 唤醒在this锁等待的线程
}
```

注意到在往队列中添加了任务后，线程立刻对`this`锁对象调用`notify()`方法，这个方法会唤醒一个正在`this`锁等待的线程（就是在`getTask()`中位于`this.wait()`的线程），从而使得等待线程从`this.wait()`方法返回。

对应`notify()`还有`notifyAll()`:唤醒所有被`wait`的程序

**注意**，==`wait(), notify()`都只能在`synchronized`代码块中使用，而不能在`lock`中使用。==



### 线程创建的方式3：实现Callable

`Runnable`接口有个问题，它的方法没有返回值。Java标准库还提供了一个`Callable`接口，和`Runnable`接口比，它多了一个返回值：

```java
//1. 创建Callable的实现类，并实现call()方法
class Task implements Callable<String> {
    public String call() throws Exception {
        return longTimeCalculation(); 
    }
}

public class Main {
    public static void main(String args[]) {
        //2.创建Callable接口实现类的对象
        Task task = new Task();
        //3. 将此Callable接口实现类的对象作为参数传递到FutureTask的构造器中，创建FutureTask的对象
        FutureTask futureTask = new FutureTask(task);
        //4.使用FutureTask对象作为参数传递到Thread类的构造器中，创造Thread对象，并调用start();
        Thread t1 = new Thread(futureTask);
        t1.start();                                        
    }
}
```

并且`Callable`接口是一个泛型接口，可以返回指定类型的结果。与`Runnable`不同的是实现`Callable`接口的类要重写`call()`方法

与`Runnable`方式的对比：

好处：

- `call()`可以有返回值，更灵活；
- `call()`可以使用`throws`的方式处理异常，更灵活
- `Callable`使用了泛型参数，可以指定具体的`call()`的返回值类型，更灵活

缺点：如果主线程需要获取分线程`call()`的返回值，则此时的主线程时阻塞状态的。



### 使用线程池

如果并发的线程数量很多，并且每个线程都是执行一个时间很短的任务就结束了，这样频繁创建线程会大大降低系统的效率，因为**创建线程和销毁线程都需要时间**

**思路**：提前创建好多个线程，放入线程池中，使用时直接获取，使用完放回池中。可以避免频繁创建销毁，实现重复利用。

**好处**：

- 提高响应速度（减少了创建新线程的时间）
- 降低资源消耗，提高资源复用率（重复利用线程池中线程，不需要每次都创建）
- 可以设置相关的参数，对线程池中的线程进行管理

具体使用：

```java
public class ThreadPool{
    
    public static void main(String[] args){
        //1.提供指定线程数量的线程池
        ExecutorService service = Executors.newFixedThreadPool(10);
        ThreadPoolExecutor service1 = (ThreadPoolExecutor) service;
        //设置线程池的属性
        sevice1.setMaximumPoolSize(50);//设置线程池中线程数上限
        
        //2.执行指定的线程的操作。需要提供实现Runnable接口或Callable接口实现类的对象
        service.execute(new NumberThread());//适用于RUnnable
        service.submit(Callable callable);//适用于Callable
        
        //3.关闭连接池
        service.shutdown();
    }
}
```







## I/O流

**I/O流的分类**：

- 流向的不同：输入流，输出流
- 处理单位的不同：字节流，字符流
- 流的角色的不同：节点流，处理流

**基础I/O流的框架**：

| 抽象基类       | 4个节点流（也称为文件流） |
| -------------- | ------------------------- |
| `InputStream`  | `FileInputStream`         |
| `OutputStream` | `FileOutputStream`        |
| `Reader`       | `FileReader`              |
| `Writer`       | `FileWriter`              |



### 输入、输出的标准化过程

4.1 输入过程

1. 创建File类的对象，指明读取的数据的来源。（要求此文件一定要存在）
2. 创建相应的输入流，将File类的对象作为参数，传入流的构造器中
3. 具体的读入过程：
   创建相应的`byte[]` 或 `char[]`。
4. 关闭流资源
   说明：程序中出现的异常需要使用`try-catch-finally`处理。

4.2 输出过程
① 创建File类的对象，指明写出的数据的位置。（不要求此文件一定要存在）
② 创建相应的输出流，将File类的对象作为参数，传入流的构造器中
③ 具体的写出过程：
`write(char[]/byte[] buffer,0,len)`
④ 关闭流资源

说明：程序中出现的异常需要使用`try-catch-finally`处理。



### 节点流（文件流）



#### FileReader` \ `FileWriter的使用

（处理字符）

数据写入 / **FileReader**的使用

```java
public static void main(String[] args){
    FileReader fR=null;
    try {
        //1、实例化File类的对象，指明要操作的文件
        File file = new File("hello.txt");
        System.out.println(file.getAbsolutePath());
        //2、提供具体的流
        fR = new FileReader(file);
        //3、数据的读入
        //read()返回读入的一个字符，如果达到文件末尾返回-1
        int data;
        while ((data=fR.read())!=-1){
            System.out.println(data);
        }
    }catch (IOException e){
        e.printStackTrace();
    }finally {
        //4、流的关闭(必须关闭，否则会内存泄漏)
        try {
            if (fR!=null)fR.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

这段代码中输入部分的`read()`每次只读一个字符，与磁盘交互次数过多，效率很低，下面进行优化

```java
char[] cbuffer = new char[5];
int len;
while((len = fr.read(cbuffer)) != -1){
    //遍历数组
	for(int i = 0; i < len; i++){
        System.out.print(cbuffer[i]);
    }    
}
```



数据写出到指定的文件中 / **FileWriter**的使用

```java
/*
从内存中写出数据到硬盘的文件里。

说明：
1. 输出操作，对应的File可以不存在的。并不会报异常
2.
     File对应的硬盘中的文件如果不存在，在输出的过程中，会自动创建此文件。
     File对应的硬盘中的文件如果存在：
           如果流使用的构造器是：FileWriter(file,false) / FileWriter(file):对原文件的覆盖
           如果流使用的构造器是：FileWriter(file,true):不会对原文件覆盖，而是在原文件基础上追加内容

 */
@Test
public void testFileWriter() {
    FileWriter fw = null;
    try {
        //1.提供File类的对象，指明写出到的文件
        File file = new File("hello1.txt");

        //2.提供FileWriter的对象，用于数据的写出
        fw = new FileWriter(file,false);

        //3.写出的操作
        fw.write("I have a dream!\n");
        fw.write("you need to have a dream!");
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        //4.流资源的关闭
        if(fw != null){
            try {
                fw.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}

```





#### FileInputStream和FileOutpStream的使用

1. 对于文本文件`(.txt,.java,.c,.cpp)`，使用字符流处理 
2. 对于非文本文件`(.jpg,.mp3,.mp4,.avi,.doc,.ppt,...)`，使用字节流处理 

```java
/*
实现对图片的复制操作
 */
@Test
public void testFileInputOutputStream()  {
    FileInputStream fis = null;
    FileOutputStream fos = null;
    try {
        //1.造文件
        File srcFile = new File("爱情与友情.jpg");
        File destFile = new File("爱情与友情2.jpg");

        //2.造流
        fis = new FileInputStream(srcFile);
        fos = new FileOutputStream(destFile);

        //3.复制的过程
        byte[] buffer = new byte[5];
        int len;
        while((len = fis.read(buffer)) != -1){
            fos.write(buffer,0,len);
        }

    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if(fos != null){
            //4.关闭流
            try {
                fos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if(fis != null){
            try {
                fis.close();
            } catch (IOException e) {
                e.printStackTrace();
            }

        }
    }
}
```



### 缓冲流

四种：1. `BufferedInputStream` 2.`BufferedOutputStream` 3. `BufferedReader` 4. `BufferedWriter`

作用：提高流的读取、写入的速度
提高读写速度的原因：内部提供了一个缓冲区，减少与磁盘交互的次数。缓冲区默认情况下是8kb





#### 使用BufferedInputStream和BufferedOutputStream:处理非文本文件

```java
//实现文件复制的方法
    public void copyFileWithBuffered(String srcPath,String destPath){
        BufferedInputStream bis = null;
        BufferedOutputStream bos = null;    
	try {
        //1.造文件
        File srcFile = new File(srcPath);
        File destFile = new File(destPath);
        //2.造流
        //2.1 造节点流
        FileInputStream fis = new FileInputStream((srcFile));
        FileOutputStream fos = new FileOutputStream(destFile);
        //2.2 造缓冲流
        bis = new BufferedInputStream(fis);
        bos = new BufferedOutputStream(fos);

        //3.复制的细节：读取、写入
        byte[] buffer = new byte[1024];
        int len;
        while((len = bis.read(buffer)) != -1){
            bos.write(buffer,0,len);
        }
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        //4.资源关闭
        //要求：先关闭外层的流，再关闭内层的流
        if(bos != null){
            try {
                bos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }

        }
        if(bis != null){
            try {
                bis.close();
            } catch (IOException e) {
                e.printStackTrace();
            }

        }
        //说明：关闭外层流的同时，内层流也会自动的进行关闭。关于内层流的关闭，我们可以省略.
//        fos.close();
//        fis.close();
        }
    }
```


#### 使用BufferedReader和BufferedWriter：处理文本文件

```java
@Test
    public void testBufferedReaderBufferedWriter(){
        BufferedReader br = null;
        BufferedWriter bw = null;
        try {
            //创建文件和相应的流
            br = new BufferedReader(new FileReader(new File("dbcp.txt")));
            bw = new BufferedWriter(new FileWriter(new File("dbcp1.txt")));

            //读写操作
            //方式一：使用char[]数组
//            char[] cbuf = new char[1024];
//            int len;
//            while((len = br.read(cbuf)) != -1){
//                bw.write(cbuf,0,len);
//    //            bw.flush();
//            }

            //方式二：使用String
            String data;
            while((data = br.readLine()) != null){
                //方法一：
//                bw.write(data + "\n");//data中不包含换行符
                //方法二：
                bw.write(data);//data中不包含换行符
                bw.newLine();//提供换行的操作

            }


        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //关闭资源
            if(bw != null){

                try {
                    bw.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(br != null){
                try {
                    br.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
        }

    }
```

**注意：关于`flush()`**：

`BufferedWriter`是缓冲输入流，意思是调用`BufferedWriter`的`write()`方法时候。数据是先写入到缓冲区里，并没有直接写入到目的文件里。必须调用`BufferedWriter`的`flush()`方法。这个方法会刷新一下该缓冲流，也就是会把数据写入到目的文件里。或者你可以调用`BufferedWriter`的`close()`方法，该方法会在关闭该输入流之前先刷新一下该缓冲流。也会把数据写入到目的文件里。





## 网络编程

要想实现网络通信，需要解决的三个问题：

1. 如何准确地定位网络上一台或多台主机？

> 使用IP地址

2. 如何定位主机上的特定应用

> 使用端口号

3. 找到主机后，如何可靠，高效地进行数据传输

> 规范网络通信协议



### 通信要素1：IP地址

定义：**互联网协议地址(`Internet Protocol Address`)** 俗称IP。用来给网络中的一台计算机设备做唯一的编号

#### IP地址分类一：

- `IPv4`：32位的二进制数，通常被分为4个字节，表示成`a,b,c,d`的形式，其中a,b,c,d都是 0~255之间的**十进制**(Why?因为二进制太长了不方便，就用十进制表示)整数。如`192.168.65.100`

- `IPv6`：因为IP地址需求量越来越大，为扩大地址空间使用128位地址长度，共16个字节，写成8个无符号整数。每个整数用四个十六进制数表示。如`ABCD:EF01:2345:6789:ABCD:EF01:2345:6789`



#### IP地址分类方式二：

公网地址(万维网)和私有地址(局域网) `192.168`开头的就是私有地址



#### 本地回路地址(`localhost`)：表示自己(本地主机的地址)，`127.0.0.1`，每个计算机都是这样



#### 域名：

便捷的记录IP地址

`www.baidu.com`......



### 通信要素2：端口号

网络的通信，本质上是两个进程的通信。每台计算机都有很多进程，那么网络通信时，如何区分这些进程呢？

**IP地址**可以唯一标识网络中的设备，那么**端口号**就可以唯一标识设备中的进程

- 公认端口：0~1023.。被预先定义的服务通信占用，如`HTTP(80), FTP(21), Telnet(23)`
- 注册端口：1024~49151.分配给用户进程或应用程序。如`Tomcat(8080), MySQL(3306), Oracle(1521)`
- 动态 / 私有端口：49152~65535



### 通信要素3：网络通信协议（网络模型）

 网络通信协议目的：为了实现可靠，高效的数据传输

| OSI参考模型    |各层作用| TCP/IP参考模型     |TCP/IP参考模型各层对应协议 | 对应结构|
| :----- | :--------- | :----------| :--------| :----------------|
| 应用层 |为应用程序提供服务| 应用层 |HTTP，FTP，Tenlnet，DNS..... | 应用程序 |
| 表示层 |数据格式转化，数据加密|  | | |
| 会话层 |建立，管理和维护对话|  | | |
| 传输层 |建立，管理和维护端对端的连接| 传输层 |TCP,UDP...... | 操作系统 |
| 网络层 |IP地址及路由选择| 网络层 |IP, ICMP, ARP...... | |
| 数据链路层 |提供介质访问和链路管理| 物理+数据链路层 |Link | 设备驱动程序和网络接口 |
| 物理层 |物理传输|  | | |

OSI参考模型：将网络分为7层，过于理想化，没有实施起来。

TCP/IP参考模型：事实上使用的标准。将网络协议分为4层：应用层，传输层，





### InetAddress的使用

作用：`InetAddress`类的一个实例就代表一个具体的IP地址。



实例化方式：

- `InetAddress getByName(String host)`
- `InetAddress getLocalHost()`

```java
try{
    //getByName(String host)
    InetAddress inet1 = InetAddress.getByName("192.168.23.31");
    
    InetAddress inet2 = InetAddress.getByName("www.baidu.com");
    
    //getLocalHost()
    InetAddress inet3 = InetAddress.getLocalHost();//本机IP
} catch(UnknownHostException e){
    e.printStackTrace();
}
```



### 传输层协议：TCP与UDP协议

#### TCP协议

- TCP协议进行通信的两个应用进程：客户端，服务端
- 使用TCP协议前，须先**建立TCP连接**，形成基于字节流的传输数据通道
- 传输前，采用"三次握手"方式，点对点通信，是**可靠的**
- - TCP协议使用**重发机制**，当一个通信实体发送一个消息给另一个通信实体后，需要收到另一个通信实体确认信息，如果没有收到另一个通信实体确认信息，则会再次重复发送

- 在连接中可以进行**大数据量的传输**
- 传输完毕，需**释放已建立的连接，效率低**

#### UDP协议

- UDP协议进行通信的两个应用进程：发送端，接收端
- 将数据，源，目的封装成数据报（传输的基本单位），接收方收到也不确认，不能保证数据的完整性，所以是**不可靠的**
- 每个数据报的大小限制在**64K**内
- 发送数据结束**无需释放资源，开销小，通信效率高**
- 适用场景：音频，视频和普通数据的传输，如视频会议
