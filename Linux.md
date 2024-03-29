# Linux复习总结

> by LHZ

## 1.Linux 概述

### Linux操作系统特点:

1. 完全免费
2. 完全兼容POSIX1.0标准
3. 多用户、多任务
4. 良好的界面
5. 安全稳定
6. 支持多平台
7. 强大的网络功能

### Linux操作系统的结构

结构：

- 内核；
- shell系统的用户界面；
- 文件系统；
- 应用程序

#### Linux内核和Linux操作系统的不同

- 内核：Linux内核主要负责管理系统中的硬件和软件资源

- 操作系统：Linux操作系统是指在Linux内核的平台上添加各种应用

### 查询帮助

- `man`:(manual)查阅联机手册,给的内容比较多 eg: `man pwd`
- `whatis`:获取命令简介(简要的) eg:`whatis uname`
- 选项`--help`:显示命令的用法 eg:`ls --help`



## 2.目录和文件

### 查看文件

- `pwd` - (Print working directory) 打印当前工作目录的绝对路径

- `ls `- (list files) 列出当前目录下的目录与文件

  > -l : 以详细信息的形式展示
  >
  > -a : 展示全部

- `cd`(change directory): 移动工作目录
  
  > `cd ..` :切换到上一级目录
  >
  > `cd ../..`:切换到上上级目录
  >
  > `cd /etc/yum`:切换到绝对路径 /etc/yum的目录
  > `cd ~`:切换到当前用户的家目录(/home/xx (xx通常为用户名))
  >
  > `cd /`:切换到根目录 /
  
- `cat`(concatenate) 打印文件内容

- `more/less` 一页一页显示文件内容,less支持往前翻页

- `head`:看文件前几行 `head -n num`看文件前num行

- `tail`:看文件后几行`tail -n num`同上

- `wc`-(word count) 字计数列出多少行,多少单词,多少字符

- `sort` 对文件内容进行排序

- `uniq` 筛选文件中的重复行

  > -u(unique):只保留不重复的行
  >
  > -d(duplicated):只保留重复的行(重复的只打印一次)
  >
  > -c(count): 计数重复行重复次数

- `grep`(global regular expression): 查找文件里符合条件的字符串或正则表达式

  > `grep [options] pattern [files]`
  >
  > eg: `grep include 2.c`:在2.c中查找include并输出

- `cmp`(compare) 两文件逐字节比较是否相同,完全相同不打印任何提示,不同打印出第一个不同之处

- `diff` 求出两文件差别

### 文件目录管理

- `mkdir`-(make directory) 创建一个新目录

  > | 选项 | 说明                                 |
  > | ---- | ------------------------------------ |
  > | -p   | 若路径中的目录不存在，则先创建目录； |
  > | -v   | 查看文件创建过程。                   |
- `cp`-(copy) 复制文件或目录

  > | 选项 | 说明                                                     |
  > | ---- | -------------------------------------------------------- |
  > | -r   | 递归处理，将指定目录下的文件及子目录一并处理             |
  > | -p   | 拷贝的同时不修改文件属性，包括所有者、所属组、权限和时间 |
  > | -f   | 强行复制文件或目录，无论目的文件或目录是否已经存在       |
- `rm`-(remove)删除文件或目录
- `mv`-(move)修改名称
- `rmdir`-(remove directory)删除空目录



### 了解系统状态

- `date`:读取系统日期和时间.

  > 可以根据需要定制输出格式:
  > date "+%Y-%m-%d %H:%M:%S Day %j"
  >
  > 2024-01-08 13:19:51 Day 8

- `cal`:打印日历. 用法:`cal` , `cal year` , `cal month year`

- `who`:列出当前已登陆入系统的用户

- `uptime`:系统自启动到现在的运行时间,当前登陆如系统的用户数,近期系统cpu负载,平均调度队列长度

- `top`:列出资源占用排名靠前的进程

- `ps`:查阅进程状态

- `free`:了解内存情况

- `vmstat`(Virtual Meomory Statistics):了解系统负载

### 压缩解压缩

- `gzip`:压缩文件,获取.gz格式的压缩包,压缩后不保存源文件.若同时列出多个文件则每个文件单独压缩. `gunzip`解压缩

  > -d   解压缩
  >
  > -k   保留原文件
  >
  > -r	递归压缩指定目录及子目录下的所有文件
  >
  > -v	显示指令执行过程

- `bzip`:压缩文件.bz2格式的压缩包,压缩后不保存源文件.若同时列出多个文件则每个文件单独压缩. `bunzip2`解压缩

  > -d   解压缩
  >
  > -k   保留原文件
  >
  > -r	递归压缩    **没有此选项！！！**
  >
  > -v	显示指令执行过程

- `zip`:打包并压缩文件,获得.zip格式的文件,压缩后保留源文件. `unzip`解压缩

  > -r	递归处理指定目录下的所有文件和子目录
  >
  > -v	显示指令的执行过程
  >
  > -d	删除压缩包内的文件
  >
  > -u	更新或追加文件到压缩包内

- `tar`(tape archive):打包多个目录或文件

  > | 选项 | 说明                                          |
  > | ---- | --------------------------------------------- |
  > | -c   | 产生.tar打包文件                              |
  > | -x   | 从打包文件中还原文件。                        |
  > | -f   | 指定压缩后的文件名                            |
  > | -z   | 通过gzip指令压缩备份文件，压缩后格式为.tar.gz |
  > | -j   | 使用bzip2压缩，压缩后格式为.tar.bz2           |
  > | -v   | 打包时显示详细信息                            |

  > 1、`tar -cvf boot.tar /boot` ：把/boot目录打包为boot.tar。
  >
  > 2、`tar -zcvf boot.tar.gz /boot`：把/boot目录打包并且压缩为gzip格式，命名为boot.tar.gz
  >
  > 3、`tar -jcvf  boot.tar.bz2  /boot`把/boot目录打包并且压缩为bzip2格式，命名为boot.tar.bz2
  >
  > 4、`tar xvf boot.tar`    //直接解压tar包
  >
  > 5、`tar xzvf boot.tar.gz` //解压gzip2压缩
  >
  > 6、`tar xjvf / tmp / boot.tar.bz2` //解压gzip2压缩

## 3.重定向

重定向是指将命令的输入或输出从默认的位置（通常是终端）改变为其他位置（文件、设备或管道）的过程。在Linux中，有三种常见的重定向方式：标准输入重定向、标准输出重定向和标准错误重定向。

1. 标准输入重定向（`<`）：使用 `<` 符号将文件作为命令的输入。例如，可以使用以下命令将文件作为输入传递给命令：

   ````shell
   command < input.txt
   
   # 上述命令将文件 input.txt的内容作为command命令的输入。
   ````

2. 标准输出重定向（`>` 和 `>>`）：使用 `>` 符号将命令的输出重定向到文件。如果文件已存在，将会被覆盖；如果文件不存在，将会创建一个新文件。例如：

   ````shell
   command > output.txt
   

#上述命令将 `command` 命令的输出保存到文件 `output.txt` 中。

#另外，如果要将输出追加到文件而不是覆盖文件，可以使用 `>>` 符号。例如：

command >> output.txt

   #上述命令将 `command` 命令的输出追加到文件 `output.txt` 的末尾。
   ````

3. 标准错误重定向（`2>` 和 `2>>`）：使用 `2>` 符号将命令的错误输出重定向到文件。类似于标准输出重定向，如果文件已存在，将会被覆盖；如果文件不存在，将会创建一个新文件。例如：

   ````shell
   command 2> error.txt
   
   #上述命令将 `command` 命令的错误输出保存到文件 `error.txt` 中。
   
   #同样，也可以使用 `2>>` 符号将错误输出追加到文件而不是覆盖文件。
   ````

除了上述的重定向方式，还有一种特殊的重定向方式是管道（`|`），它将一个命令的输出作为另一个命令的输入。例如：

```shell
command1 | command2
```

上述命令将 `command1` 的输出作为 `command2` 的输入。

## 4.Linux目录结构

![img](file:///C:/Users/86139/Downloads/efa17c9d-5f10-40a9-b793-6ce8eb1841fe.jpeg)

### 目录说明

- `/bin`: (binaries(二进制文件)),存放最经常使用的命令

- `/boot`: 存放启动Linux时使用的一些核心文件,包括一些连接文件以及镜像文件.

- `/dev`: (device),存放Linux的外部设备.**Linux中访问设备和访问文件的方式是相同的**

- `/etc`: (etcetera(等等,表省略)),存放所有的系统管理所需要的配置文件和子目录.

- `/home`:用户主目录,一般该目录名是以用户的账号命名的

- `/lib`:(library(库)),存放系统最基本的动态连接共享库,**几乎所有的应用程序都需要用到这些共享库**

- `/lost+found`:一般情况下为空,当系统非法关机后就存放一些文件

- `/media`:Linux会自动识别一些设备,如U盘光驱等,当你插入这些设备Linux识别后会把它们挂载到这个目录下

- `/mnt`:系统提供该目录是为了让用户临时挂载别的文件系统的，我们可以将光驱挂载在 /mnt/ 上，然后进入该目录就可以查看光驱里的内容了。

- `/opt`:(optional(可选)),给主机额外安装软件所摆放的目录。比如你安装一个ORACLE数据库则就可以放到这个目录下。默认是空的。

- `/proc`:(Processes(进程)),是一种伪文件系统（也即虚拟文件系统），存储的是当前内核运行状态的一系列特殊文件

- `/root`:该目录为系统管理员，也称作超级权限者的用户主目录。

- `/sbin`: (Superuser Binaries (超级用户的二进制文件) ),存放系统管理员使用的系统管理程序。

- `/srv`:存放一些服务启动之后需要提取的数据。

- `/sys`: 和`/proc`目录很相似，数据也都在内存中而不在磁盘中，存放系统相关信息。

- `/tmp`:(temporary(临时)),存放一些临时文件

- `/usr`:(unix shared resources(共享资源)),用户的很多应用程序和文件都放在这个目录下，类似于 windows 下的 program files 目录。

  -  `/usr/bin`:系统用户使用的应用程序。

  - `usr/sbin`:超级用户使用的比较高级的管理程序和系统守护程序。 
  - `/usr/src`:内核源代码默认的放置目录。

- `/var`:(variable(变量)),存放着在不断扩充着的东西，我们习惯将那些**经常被修改**的目录放在这个目录下。包括各种日志文件。
- `/run`:是一个临时文件系统，存储系统启动以来的信息。当系统重启时，这个目录下的文件应该被删掉或清除。如果你的系统上有 /var/run 目录，应该让它指向 run。

### 文件类型

在终端对某个目录或文件使用`ls -l`,查看其属性.
![img](file:///C:/Users/86139/AppData/Local/Packages/Microsoft.Windows.Photos_8wekyb3d8bbwe/TempState/ShareServiceTempFolder/055d5520-bf8b-40d6-93fe-3d0bd9b12099.jpeg)

每行文件信息的第1组字符串就是文件的属性信息。

- **首个字符**:用来表示该文件的类型。
  - **d**表示目录；
  - **–**表示普通文件；
  - **b**表示块输入输出设备文件，通常是磁盘驱动器；
  - **c**表示连续输入输出设备，通常是声卡、调制解调器等；
  - **l**表示是链接文件
  - **p**表示管道文件。

- 第2个至第4个字符:用来确定文件的**所有者权限**
- 第5个至第7个字符用来确定文件的**所属组权限**
- 第8个至第10个字符用来确定文件的**其它用户的权限**
- r控制文件的读权限，w控制文件的写权限，x用来控制文件的执行权限。**权限的加权数是将读、写、执行分别用数值4、2、1代表**，用户获得的权限是**数值之和**。

## 5.用户和用户组

Linux操作系统中设立了用户和用户组的概念，在使用系统资源时必须有身份，因此用户需要先向系统管理员申请一个账号。Linux允许多个用户同时登陆操作系统，针对系统中的多名用户，Linux还设计了用户组的概念，为用户指定用户组，可以在需要时方便地对多个用户进行管理。

用户包括 管理员,系统用户,普通用户:

- 管理员：id为0
- 系统用户：保障系统运行的用户，其用户id为1~499
- 普通用户：保障系统运行的用户，其用户id为500~60000

### 用户管理

用户管理即对用户账号进行的管理，包括账号的添加、密码设置、修改和删除；用户组的管理包括用户组的添加、更改和删除操作、用户组切换等。

- `su`:(substitude), `su -选项 username` 切换用户

  > | 参数  | 说明                                                         |
  > | ----- | ------------------------------------------------------------ |
  > | -c    | 执行完指定的指令后，切换回原来的用户                         |
  > | -l    | 切换用户的同时，切换到对应用户的工作目录，环境变量也会随之改变 |
  > | -m,-p | 切换用户时，不改变环境变量                                   |
  > | -s    | 指定要执行的shell                                            |

  > ①使用su命令切换用户时，需要输入的是目标用户的密码。
  >
  > ②选项与用户名缺省的情况下，只切换用户，但不改变用户环境。
  >
  > ③由root用户切换到其它用户，可以不输入密码。
  >
  > ④在命令行输入“exit”或“su - user”，可退出目标用户。

- `sudo`:(super user do),允许系统管理员分配给普通用户一些合理的权限，让他们去执行一些只有超级用户或者其他特许用户才能完成的任务。sudo的流程：当前用户切换到root（或指定用户) -> 以root（或指定用户）身份执行命令 -> 直接退回到当前用户。

  > | 选项 | 说明                                                         |
  > | ---- | ------------------------------------------------------------ |
  > | -b   | 在后台执行命令                                               |
  > | -h   | 显示帮助                                                     |
  > | -H   | 将HOME环境变量设置为新身份的HOME环境变量                     |
  > | -k   | 结束密码的有效期限                                           |
  > | -l   | 列出目前用户可执行与无法执行的命令                           |
  > | -p   | 改变询问密码的提示符号                                       |
  > | -s   | 执行指定的shell                                              |
  > | -u   | 以指定的用户作为新的身份，即切换到指定用户。默认切换到root用户 |

- `useradd`:添加用户.只能以管理员身份运行,创建的用户都在`/etc/passwd`文件中

  > | 选项 | 说明                                                     |
  > | ---- | -------------------------------------------------------- |
  > | -d   | 指定用户登入时的目录                                     |
  > | -g   | 指定用户所属组                                           |
  > | -G   | 指定用户所属的附加用户组                                 |
  > | -m   | 自动建立用户的登入目录                                   |
  > | -r   | 创建系统账号                                             |
  > | -s   | 指定用户的登陆shell                                      |
  > | -u   | 指定用户的用户ID。若添加-o选项，则用户ID可与其它用户重复 |

- `groupadd`:默认情况下新建用户的用户组与用户名相同，在创建用户的同时被创建。主动添加用户组时使用的命令为groupadd.创建的用户组都在`/etc/group`文件中。

  > | 选项 | 说明                                |
  > | ---- | ----------------------------------- |
  > | -g   | 指定新建用户组的组ID                |
  > | -r   | 创建系统用户组，组ID取值范围为1~499 |
  > | -o   | 允许创建组ID已存在的用户组          |
  
  

- `userdel`:删除用户.

  > | 选项 | 说明                                     |
  > | ---- | ---------------------------------------- |
  > | -f   | 强制删除用户，即便该用户为当前用户       |
  > | -r   | 删除用户的同时，删除与用户相关的所有文件 |

- `groupdel`:删除用户组

- `passwd`:更换用户密码等信息.系统管理员可以修改所有用户的密码,普通用户只能修改自己的密码.

  > | 选项 | 说明                                                 |
  > | ---- | ---------------------------------------------------- |
  > | -l   | 锁定密码，锁定后密码失效，无法登陆（新用户默认锁定） |
  > | -d   | 删除密码，仅系统管理员可使用                         |
  > | -S   | 列出密码相关信息，仅系统管理员可使用                 |
  > | -f   | 强行执行                                             |
  >
  > > 用户密码保存在 /etc/shadow文件里

  

- `usermod`:修改用户账号信息(用户ID、主目录、用户组、登陆shell...)

  > | 选项 | 说明                                         |
  > | ---- | -------------------------------------------- |
  > | -c   | 修改用户账号的备注信息                       |
  > | -d   | 修改用户的登入目录                           |
  > | -e   | 修改账号的有效期限                           |
  > | -f   | 修改缓冲天数，即修改密码过期后关闭账号的时间 |
  > | -g   | 修改用户所属组                               |
  > | -l   | 修改用户账号名称                             |
  > | -L   | 锁定用户密码，使密码失效                     |
  > | -s   | 修改用户登陆后使用的shell                    |
  > | -u   | 修改用户ID                                   |
  > | -U   | 解除密码锁定                                 |

- `id`:查看用户的UID,GID及其所属的组.

- `groups`: 查看用户组信息.`groups`查看当前用户所属组. `groups username`查看指定用户所属组

  > 直接查看组文件: cat/etc/group

- `chown`:(change the owner of file),  变更文件或目录的所有者.

- `chgrp`:(change file group), 变更文件或目录的所有者

- `chmod`:改变文件或目录权限. 格式:`chmod {用户}{+-=} {权限}文件或目录`

  > | 权限     | 对应字符 | 文件           | 目录                       |
  > | -------- | -------- | -------------- | -------------------------- |
  > | 读权限   | r        | 可查看文件内容 | 可以列出目录中的内容       |
  > | 写权限   | w        | 可修改文件内容 | 可以在目录中创建、删除文件 |
  > | 执行权限 | x        | 可执行该文件   | 可以进入目录               |

  > | 用户     | 对应字符 |
  > | -------- | -------- |
  > | 文件属主 | u        |
  > | 文件属组 | g        |
  > | 其他人   | o        |
  > | 所有人   | a        |

  eg: `chmod u+x, g-w ls_out.txt`
  还有一种8进制写法,读(r)、写(w)、执行(x)分别用数值**4、2、1**代表，用户获得的权限是**数值之和**。
  ![img](https://www.runoob.com/wp-content/uploads/2014/08/file-permissions-rwx.jpg)
  eg: `chmod 777 ls_out.txt`



## 6.Shell程序设计

Shell是一个命令，它解释由用户输入的命令并且把它们送到内核。不仅如此，Shell有自己的编程语言用于对命令的编辑，它允许用户编写由shell命令组成的程序。

Shell是一种命令语言，又是一种程序设计语言。

### 常见的Shell

- Bsh
- Csh
- Ksh
- bash
- ....

### Shell的特点:

- Shell是一种解释性语言

  > 用Shell语言写的程序不需编译，可以直接由Shell进程解释执行
  >
  > 解释性语言的特点是快捷方便，可以即编即用，但与编译性语言的目标程序来比，解释性语言程序的运行速度要低一些



- Shell是基于字符串的语言

  > Shell只是做字符串处理，不支持复杂的数据结构和运算
  >
  > Shell的输出也全部是字符方式的



- Shell是命令级语言

  > Shell程序全部由命令而不是语句组成，几乎所有的Shell命令和可执行程序都可用来编写Shell程序
  >
  > Shell命令十分丰富，命令的组合功能也十分强大

### 第一个Shell脚本

```shell
#!/bin/bash        
echo  “hello, world”
```

第一行的`#!`是Shell脚本的起始符号，它告诉Linux 系统这个文件的执行需要指定一个解释器。`#!`后面是一个路径名，指明**解释器在系统中的位置**。第二行是具体的命令语句，这一行的作用是在显示器上显示”hello,world”

Shell中注释以`#`开头

执行脚本有两种方式:

1. 将脚本文件作为Shell参数的执行方式

   > `sh 脚本名` or `bash 脚本名`

2. 将脚本本身作为一个可执行文件，修改权限后，直接执行
   ![img](data:image/png;base64,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)

### 变量

变量包括以下几种:

- 用户自定义变量:这些变量由用户在Shell脚本中显式定义，用于存储自定义值。

  > `name="John"` `count=10`

- 环境变量: 由操作系统或用户在Shell会话开始时设置的全局变量。它们对于整个系统或用户是可见的。

  > `PATH`：指定可执行程序的搜索路径。
  >
  > `HOME`：指定当前用户的主目录路径。

- 系统变量(位置参数变量+其他变量):

  ![img](file:///C:/Users/86139/AppData/Local/Packages/Microsoft.Windows.Photos_8wekyb3d8bbwe/TempState/ShareServiceTempFolder/4fa7d803-6ab1-4a6b-b966-defcbda2db10.jpeg)

#### 引用变量时要加引号

- 在一个`$`变量表达式外加**双引号**：输出变量中存储的值

- 在一个`$`变量表达式外加**单引号**：原样输出，不会发生替换现象，等价于在`$`字符前加\字符

可以用`readonly`命令将变量设置为只读的. 格式: `readonly 变量名`

#### 清除变量:

- 用unset命令清除变量，格式为：`unset 变量名`
- 清除后的变量变为未定义变量，引用其值将得到空字符串
- 只读变量是不能被清除的

#### 变量运算

整型变量的运算要放在**双括号**之间或者放在**内置命令let**的后面

eg:

```shell
x=20  y=30	     #同一行对多个变量赋值，用空格分开。	
((w=x+y))	     #运算放在双括号里面	
echo  $w	
50
```

```shell
let k=3*8		#用let 进行运算。	
echo $k	
24
```

如果没用双括号，也没用let，运算的结果会变成字符串。
eg:

```shell
u=10-6	
echo  $u	
10-6 #显示结果为 10-6
```

跟C语言一样，如果两个整数相除，结果有小数时，小数部分被丢掉。



### 程序控制结构语句

#### if语句

格式:

```shell
if [ condition ]
then
    command1 
    command2
    ...
elif [ condition2 ]
then
	command3
else
	commandN
fi
```

`if`中的判断语句要用`[]`括起来,括号跟中间条件必须有空格.
操作符与操作变量之间要有空格
字符串比较中, `> ` `<` 需要写成 `\>` ` \<` 进行转义(双中括号[[ ]]不需要)
其中的**算术比较**

##### if条件中的算术比较

- `=`, `!=`:检查两个**字符串**相等与否
- `-eq`:(equal) 检查两个**整数**是否相等
- `-ne`:(not equal) 检查两个整数是否不相等
- `-lt`:(less than) 是否小于
- `-gt`:(greater than) 是否 大于
- `-le`:(less than or equal to) 小于等于
- `-ge`:(greater than or equal to) 大于等于

但是如果使用`(())`括住条件,就可以直接用>, <, >=, <=. eg:`if(( a >= b))` 

##### 文件测试

###### 文件类型

- `-e`: (exists) 检查文件或目录是否存在。

  > eg: `if [ -e "$file" ]; then echo "存在"; fi`

- `-a`:检查文件是否存在

- `-f`: (regular file) 检查是否存在且为常规文件。

  > eg: `if [ -f "$file" ]; then echo "是文件"; fi`

- `-d`:(directory) 检查是否为目录。

  > eg: `if [ -d "$directory" ]; then echo "是目录"; fi`

- `-s`:(size) 检查文件是否为非空文件。

  > eg: `if [ -s "$file" ]; then echo "非空文件"; fi`

- `-r`:(readable) 检查文件是否可读。

  > eg: `if [ -r "$file" ]; then echo "可读"; fi` 

- `-w`: (writable) 检查文件是否可写。

  > eg: `if [ -w "$file" ]; then echo "可写"; fi `

- `-x`: (executable) 检查文件是否可执行。

  > eg: `if [ -x "$file" ]; then echo "可执行"; fi`

- `-b`:(block) 检查文件是否存在，并且是块设备文件。块设备文件是一种特殊类型的文件，用于访问块设备（如硬盘驱动器）。

  > eg: `if [ -b "$file" ]; then echo "存在且是块设备文件"; fi` 

- `-c`:(character) 检查文件是否存在，并且是字符设备文件。字符设备文件是一种特殊类型的文件，用于访问字符设备（如终端设备）。

  > eg: `if [ -c "$file" ]; then echo "存在且是字符设备文件"; fi` 

- `-p`:(pipe) 检查文件是否存在，并且是管道文件（命名管道）。管道文件是一种特殊类型的文件，用于进程间通信。

  > eg: `if [ -p "$file" ]; then echo "存在且是管道文件"; fi`

- `-S`:(socket) 检查文件是否存在，并且是套接字文件。套接字文件是一种特殊类型的文件，用于进程间的网络通信。
  eg: `if [ -S "$file" ]; then echo "存在且是套接字文件"; fi`

- `-L`: (symbolic link)检查文件是否存在，并且是符号链接文件（软链接）。符号链接文件是一种特殊类型的文件，它包含指向另一个文件或目录的符号链接。

  > eg: `if [ -L "$file" ]; then echo "存在且是符号链接文件"; fi` 

###### 文件权限

- `-r`: (read) 检查文件是否可读。如果指定的文件对当前用户可读，则条件为真。
- `-w`: (write)检查文件是否可写。如果指定的文件对当前用户可写，则条件为真。
- `-x`: (execute) 检查文件是否可执行。如果指定的文件对当前用户可执行，则条件为真。
- `-u`:(user id)检查文件是否设置了 SUID（Set User ID）位。如果指定的文件设置了 SUID 位，则条件为真。
- `-g`: (group id)检查文件是否设置了 SGID（Set Group ID）位。如果指定的文件设置了 SGID 位，则条件为真。
- `-k`: (sticky bit)检查文件是否设置了粘着位（sticky bit）。如果指定的文件设置了粘着位，则条件为真。

##### 文件比较

- `-nt`: (newer than) 检查文件是否比另一个文件新。如果指定的文件比另一个文件更新（修改时间更晚），则条件为真。
- `-ot`: (older than)检查文件是否比另一个文件旧。如果指定的文件比另一个文件更旧（修改时间更早），则条件为真。
- `-et`: (equal to) 检查文件是否与另一个文件的修改时间相同。如果指定的文件与另一个文件的修改时间相同，则条件为真。

##### 逻辑操作符

- `-a`: (and) 逻辑与 `if[ -r myfile1 -a -w myfile1 ]` 等价于 `if[ -r myfile1 ] && [ -w myfile1 ]` 等价于 `if[[ -r myfile1 && -w myfile1 ]]` 
- `-o`:(or) 逻辑或. 与 || 关系类比上面



#### case语句

语法格式:

```shell
case 值 in
模式1)
    command1
    command2
    ...
    commandN
    ;;
模式2)
    command1
    command2
    ...
    commandN
    ;;
 *)
	commandX
	;;
esac

```

#### for循环

语法格式:

```shell
for var in item1 item2 ... itemN
do
    command1
    command2
    ...
    commandN
done

```

or

```shell
for (( initialization; condition; increment/decrement ))
do
    # 执行的操作
done
```

注意:使用for循环的shell文件运行要用`bash xx.sh` or `./xx.sh` 而不是`sh xx.sh`,这样更稳妥



#### while语句

语法格式:

```shell
while [ condition ]
do
    command
done
```

while 后面也可以使用[[  ]]或者((  )),区别跟if里的情况类似



#### until循环

语法格式:

```shell
until [ condition ]
do
    command
done
```

使用[[  ]]与((  ))的区别同上

#### 退出循环指令

break continue 类比C即可,但注意break命令和continue命令**只能应用在for循环、while循环、until循环结构中**。



### 函数

定义格式:

```shell
function funname() #function加不加没区别

{

    action;

    [return int;]

}
```

#### 函数参数

函数的返回值(return的值)可以在主函数中在调用该函数后通过`$?`获取
eg:

```shell
#!/bin/bash

funWithReturn(){
    echo "这个函数会对输入的两个数字进行相加运算..."
    echo "输入第一个数字: "
    read aNum
    echo "输入第二个数字: "
    read anotherNum
    echo "两个数字分别为 $aNum 和 $anotherNum !"
    return $(($aNum+$anotherNum))
}
funWithReturn
echo "输入的两个数字之和为 $? !"

#输出为:
这个函数会对输入的两个数字进行相加运算...
输入第一个数字: 
1
输入第二个数字: 
2
两个数字分别为 1 和 2 !
输入的两个数字之和为 3 !
```

函数的第n个参数通过 `$n`获取

eg:

```shell
#!/bin/bash

funWithParam(){
    echo "第一个参数为 $1 !"
    echo "第二个参数为 $2 !"
    echo "第十个参数为 $10 !"
    echo "第十个参数为 ${10} !"
    echo "第十一个参数为 ${11} !"
    echo "参数总数有 $# 个!"
    echo "作为一个字符串输出所有参数 $* !"
}
funWithParam 1 2 3 4 5 6 7 8 9 34 73
```



#### 局部与全局变量

bash中函数内的变量默认为全局变量.如果想让它成为局部变量需要使用`local`修饰



## 7.通配符与正则表达式

### 通配符

| 通配符                  | 含义                                        | 实例                                                       |
| ----------------------- | ------------------------------------------- | ---------------------------------------------------------- |
| `*`                     | 匹配0到多个任意字符                         | `a*b`:可以是`ab`,`aabcb`,`a012b`                           |
| `?`                     | 匹配任意**一个**字符                        | `a?b`:ab之间必须也只能有一个字符,可以是`aab`,`abb`,`a0b`   |
| `[list]`                | 匹配list中的任意单个字符                    | `a[xyz]b`:`axb`或`ayb`或`azb`                              |
| `[!list]`或`[^list]`    | 匹配除list外的任意一个字符                  | `a[!0123456789]b`:ab之间加上除了阿拉伯数字外的任意一个字符 |
| `[c1-c2]`               | 匹配c1-c2中的任意单一字符,如`[0-9]`,`[a-z]` | `a[0-9]b`:ab间有一个0-9的阿拉伯数字                        |
| `[!c1-c2]`或`[^c1-c2]`  | 匹配不在c1-c2的任意一个字符                 | `a[!0-9]b`:ab间加0-9外的任意一个字符                       |
| `{string1,string2,...}` | 匹配string1,string2...中的任一字符串        | `a{abc,xyz,123}b`:`aabcb`或`axyzb`或 `a123b`               |



### 正则表达式

格式:`grep [options] PATTERN [FILES]`
如果没有提供文件名，则grep命令将搜索标准输入

#### 限定符

| 限定符                    | 效果                                                         | 实例                                               |
| ------------------------- | ------------------------------------------------------------ | -------------------------------------------------- |
| `?`                       | 前面的字符出现1次或0次(可有可无)                             | `used?`可以匹配use或者used                         |
| `*`                       | 前面的字符可以没有或者出现**多次** 相当于(?plus)             | `ab*c`可以匹配ac,abc,abbbbbbc                      |
| `+`                       | 出现一次以上                                                 | `ab+c` 匹配 abc abbbc但是没有ac                    |
| `{c1,c2}`or`{c}`or`{c1,}` | 精确指定前面的字符出现c1到c2次 / c次 / c1次以上              | 懒得举例了                                         |
| `[]`                      | 指定匹配的字符,`[x|y]`是与x或y匹配;`[xyz]`是与出现的其中任意一个字符匹配;`[a-z]`是与a到z范围内的任意一个 | `[a-zA-Z]`匹配包括大写小写的字母,匹配1234abc2中的a |
| `[^xyz]`                  | `^`相当于**逆**,即匹配非`[]`里的任意字符                     |                                                    |
|                           |                                                              |                                                    |

#### 元字符

| 元字符 | 使用                                                         |
| ------ | ------------------------------------------------------------ |
| `\d`   | digit,匹配一个长度大于等于1的数字,等同于`[0-9]`; 对应的`\D`代表非数字字符 |
| `\w`   | word,匹配一个英文字符(字母,数字或下划线_); 对应的`\W`代表非英文字符 |
| `\s`   | space,匹配一个空白字符(空格,制表符等);对应的`\S`代表非空白字符 |
| `\b`   | boundary,匹配一个单词的边界(字与空格间的位置).如`er\b`可以匹配`never x    `中的er,但不会匹配`verb`中的er |
| `.`    | 匹配任意单个字符,但不包括换行符                              |
| `^x`   | 匹配行首字符.如`^a`可以匹配abs的a但不能匹配bas的a            |
| `x$`   | 匹配行尾字符.如`a$`可匹配`bba`的a                            |

