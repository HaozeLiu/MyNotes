# Little tips

## 1.**数组作为函数参数传值时**

编译器不会拷贝整个数组，只会创建一个同名的指针存放数组首元素地址。（因为可能数组很大，如果直接把数组全部都存入函数计算效率会很低）

## 2.**互质的两个数不能凑出的最大数**

有互质的两个数 `a, b`，问`k1 * a + k2 * b` 不能等于的最大数`n`是多少？

**`n = a * b - a - b = (a - 1) * (b - 1) - 1`** 

## 3. 字符串相关

-  1 如果想建立一个字符数组 `c[]` 来存放字符串`"john"` ，那么==`c.size() >= 5`==。Why? 字符数组会在字符串末尾加一个`\0` 以表示字符串已经结束。

- 2`strlen` 计算字符串长度不包括`\0`

```cpp
char c[20];
c[0] = 'J', c[1] = 'O', c[2] = 'H', c[3] = 'N', c[4] = '\0';
int len = strlen(c);
printf("Length = %d\n", len);//Length = 4
```

- 3 `char c[20] = "JOHN";` 直接这样声明并初始化一个字符数组会自动在末尾加上`'\0'`, 但如果写`c[4] = "JOHN";`将编译错误。

```cpp
char c[] = "JOHN";
printf("Size in bytes = %d\n", sizeof(c));
int len = strlen(c);
printf("Length = %d\n", len);
/* Size in bytes = 5
   Length = 4 */
```

使用`sizeof(c)`计算将囊括`'\0'`, 得出5个字节，但`strlen(c)` 仍然不包括`'\0'`，显示为4。

## 4. 动态内存分配

动态内存分配操作 (在堆区(heap)里)

- C：
  - `malloc` 作用是申请一块内存，它会返回分配给我们的内存块的第一个字节的地址，是一个`void`指针（注意，**`void`指针无法直接解引用**），如果不初始化会随机得到一些垃圾值。 eg: `int* p = (int*)malloc(sizeof(int));`
  
  - `clloc` 类似于`malloc`，需要两个参数，一个是数量，一个是大小，同时默认初始化为0。eg:`int* p = (int*)calloc(3, sizeof(int));`
  
  - `realloc` 作用是给一块已经分配好的内存改变大小。接受两个参数 ，第一个是已分配内存起始地址的指针，第二个是新内存块的大小。
  
  - `free` 任何已分配的内存会在程序结束前一直存在，除非你显式的释放它。
  
    应用举例：
  
    ```c
    #include<stdio.h>
    #include<stlib.h>
    int main()
    {
        int n;
        printf("Enter size of array\n");
        scanf("%d", &n);
        //动态变长数组
        int *A = (int*)malloc(n*sizeof(int));//或者(int*)calloc(n, sizeof(int));
        for(int i = 0; i < n; i++)
        {
            A[i] = i + 1;
        }
        
        int* B = realloc(A, 2*n*sizeof(int))
        
        free(A);
        A = NULL;//free后，调整指针指向NULL
    }
    ```
  
    
  
    
  
- Cpp:  
  - `new`  想要申请一块数组这样的连续内存时，要 `p = new int[20];`
  - `delete` 删除一块数组这样的连续内存时， 要` delete[] p;`

## 5.**`srand(), rand()`**

计算机并不能产生真正的随机数，而是已经编写好的一些无规则排列的数字存储在电脑里，把这些数字划分为若干相等的N份，并为每份加上一个编号用srand()函数获取这个编号，然后rand()就按顺序获取这些数字，当srand()的参数值固定的时候，rand()获得的数也是固定的，所以一般srand的参数用time(NULL)，因为系统的时间一直在变，所以rand()获得的数，也就一直在变，相当于是随机数了。只要用户或第三方不设置随机种子，那么在默认情况下随机种子来自系统时钟。

一个赌博程序  

```c
//三张牌："Jack Queen King" - 计算机随机洗牌
//用户需要猜测 Queen 的位置。
//猜对了，他获得三倍赌注"bet"的钱，错了赌注失去，直到没钱
#include<stdio.h>
#include<stdlib.h>
#include<time.h>
int cash = 100;

void play(int bet)
{
    char c[3] = {'J', 'Q', 'K'};
    printf("\nShuffling ...\n");
    
    srand(time(NULL));
    for(int i = 0; i < 5; i++)
    {
        int x = rand() % 3;
        int y = rand() % 3;
        int temp = c[x];
        c[x] = c[y];
        c[y] = temp;
    }//两两随机交换5次以达到洗牌的效果
    int playersGuess;
    printf("What's the position of queen - 1, 2 or 3?\n");
    scanf("%d", &playersGuess);
    if(c[playersGuess - 1] == 'Q') 
    {
        cash += 3 * bet;
        printf("You win ! Total cash = $%d", cash);
    }
    else 
    {
        cash -= bet;
        printf("You lose ! Total cash = %d", &cash);
    }
}

int main()
{
    int bet;
    while(cash > 0)
    {
        printf("What's your bet? $")
        scanf("%d", &bet);
        if(bet == 0 || bet > cash) break;
        play(bet); 
    }
}
```



## 6.**`memset`只能赋值 -1 和 0**

## 7. 判断闰年 

1582年以来公历的**置闰规则**：

*普通闰年*：公历年份**是4的倍数，且不是100的倍数**的，为闰年（如2004年、2020年等就是闰年）。

*世纪闰年*：公历年份**是整百数的，必须是400的倍数**才是闰年（如1900年不是闰年，2000年是闰年）。

````cpp
if((year % 4 == 0 && year % 100 != 0) || year % 400 == 0) 
    return true;
else return false;
````

## 8. 判断奇偶

用位运算比较好

```cpp
//n & 1 = 1则为奇数，n & 1 = 0则为偶数
if(n & 1) == 1
    return true;
else return false;
```



## 9.  Java的逻辑运算符 

java 中的 `&`和`|`相当于 c 里面的 `&& , ||`, 但 java 中也有`&&, ||`,它们与单个的区别是**当逻辑运算前半部分的结果足以推导出最终结果时，会直接停止运算以节省资源**，
