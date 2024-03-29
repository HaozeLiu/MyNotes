# Stack 栈

栈的一大特点：先进后出(LIFO)

插入，弹出操作都必须强制在栈的一端执行，这一端叫栈顶。

## 数组实现栈

建立一个数组，再用一点`top`表示栈顶，初始化为`top = -1`。

`Push(x)`:在栈顶插入数`x`

```cpp
Push(int x)
{
    top += 1;
    a[top] = x;//这里数组 a 与 均为全局变量
}
```

`Pop()`: 弹出栈顶元素

```cpp
Pop()
{
    top -= 1;//不用删掉原来top处的值，下次push时自会覆盖掉
}    
```



很明显一个问题是，当栈内存储数的数量超过数组长度时会爆栈（stack overflow），为了解决这个问题，我们可以采用动态数组的思路，**当要存的数数量超过数组长度时，再建一个更大的新数组，然后把旧数组拷贝到新数组，再把老数组删掉**

`Top()`返回栈顶元素

```cpp
Top()
{
    return a[top];
}
```

`Isempty()`检查栈是否为空

```cpp
IsEmpty()
{
    if(top == -1)
        return true;
    else return false;
}
```

### 栈的实现代码

```c
//Stack - Array based implementation
#include<stdio.h>
#define MAX_SIZE 101
int A[MAX_SIZE];
int top = -1;

void Push(int x)
{
    if(top == MAX_SIZE - 1)//爆栈的情况
    {
        printf("Error: stack overflow\n");
        return;
    }
    A[++top] = x;//就是把 top++; A[top] = x合成一句; ++top:自增会发生在赋值之前
}

void Pop()
{
    if(top == -1)
    {
        printf("Error: No element to pop\n");
        return;
    }
    top--;
}

int Top()
{
    return A[top];
}

void Print()
{
    printf("Stack: ");
    for(int i = 0; i <= top; i++)
        printf("%d ",A[i]);
    printf("\n");
}
int main()
{
    Push(2);Print();
    Push(5);Print();
    Push(10);Print();
    Pop();Print();
    Push(12);Print();
}
```



## 链表实现栈

使用链表来实现栈，我们可以不用担心内存溢出的问题。**（当然取而代之的是要注意使用的堆上的额外内存，不要忘记`free`）**

==注意==：如果让链表尾`（tail)`作栈顶，那么每次 `push, pop` 操作都要先从 `head` 遍历到` tail` 再进行操作，那么时间复杂度就为`O(n)`。但让`head`作栈顶就明显不同了，时间复杂度为`O(1)`。

### 代码实现

```c
typedef struct Node
{
    int data;
    struct Node* link;
}Node;
Node* top = NULL;

void push(int x)
{
    Node* temp = (Node*)malloc(sizeof(Node));
    temp -> data = x;
    temp -> link = top;
    top = temp;
}

void pop()
{
    Node* temp;
    if(top == NULL) return;
    temp = top;
    top = top -> link;
    free(temp);
}

int Top()
{
    return top -> data;
}

bool IsEmpty()
{
    if(top -> data == NULL)
        return true;
    else return false;
}
```



## 栈的应用

### 1. 用栈反转一个字符串 / 链表

#### 反转字符串

先建一个栈把字符串正序存入，然后进行一个循环，每次将栈顶元素加入数组后弹出栈顶数。（利用先进后出的特性反转字符）

```cpp
// string reveral using stack
#include<iostream>
#include<stack>
using namespace std;

void Reverse(char *C, int n)// char C[] 与 char *C 等价
{
    stack<char> S;
    for(int i = 0; i < n; i++)
        S.push(C[i]);
    for(int i = 0; i < n; i++)
    {
        C[i] = S.Top;
        S.pop();
    }
}

int main()
{
    cahr C[51];
    cout<<"Enter a string: "<<endl;
    gets(C);
    Reverse(C,strlen(C));
    printf("Output = %s",C);
}
```



#### 反转链表

```cpp
#include<iostream>
#include<stack>
using namespace std;

struct  Node
{
    int data;
    Node* next;
};
Node *head = NULL;

void Reverse()
{
    if(head == NULL) return;
    stack<struct Node*> S;
    Node* temp = head;
    while(temp != NULL)
    {
        S.push(temp);
        temp = temp ->next;
    }
    temp = S.top(); head = temp;
    S.pop();
    while(!S.empty())
    {
        temp -> next = S.top();
        S.pop();
        temp = temp ->next;
    }
}
```



### 2. 检查括号匹配性

检测 `(), [], {}` 的匹配性

​	思路：

​		1. 数量。`'('.count = ')'.count, '['.count = ']'.count, '{'.count = '}'.count  ` 

​		2. 顺序及匹配  必须先左括号再右括号，括号类型也要匹配

​        3. 后出现的左括号要先有相应的右括号来关闭。



#### 代码实现

 ```cpp
 #include<iostream>
 #include<stack>
 #include<string>
 using namespace std;
 
 bool ArePair(char opening, char closing)
 {
     if(opening == '(' && closing == ')') return true;
     else if(opening == '{' && closing == '}') return true;
     else if(opening == '[' && closing == ']') return true;
     return false;
 }
 
 bool AreParanthesesBalanced(string exp)
 {
     stack<char> S;
     for(int i = 0; i < exp.length(); i++)
     {
         if(exp[i] == '(' || exp[i] == '{' || exp[i] == '[')
             S.push(exp[i]);
         else if(exp[i] == ')' || exp[i] == '}' || exp[i] == ']')
         {
             if(S.empty() || !Arepair(S.top(), exp[i]))
                 return false;
             else S.pop();
         }
     }
     return S.empty()? true:false;
 }
 
 int main()
 {
     string expression;
     cout<<"Enter an expression: ";
     cin>>expression;
     if(AreParanthesesBalanced(expression))
         cout<<"Balanced\n";
     else
         cout<<"Not Balanced\n";
 }
 ```



### 3. 中缀，前缀，后缀

#### 定义

1. 中缀：**操作符**在操作数的**中间**

eg : `2 + 3`,   ` p - q`,   ` a + b * c`

2. 前缀：操作符在操作数的**前面**

eg : `+ 2 3`,   `- p q`,    `+ a * b c`

3. 后缀：操作符在操作数的**后面**

eg : `2 3 +`,   `p q -`,   `a b c * +`



#### 后缀的计算

从左到右扫描式子，**扫描到的每个数放到一个栈中**，每当扫描到运算符时，对它前两个操作数进行运算，然后在栈里用计算结果取代栈中原来的两个数，以此类推。

前缀式子类似于后缀，只是从右往左计算即可。



#### 中缀转化为后缀式

从左到右扫描式子，扫描到操作数时放到后缀式最后面，扫描到操作符时先放入栈中，再次扫描到运算符时，若栈顶运算符优先级高或相等，则先把栈顶运算符弹出放入后缀式再压入扫描到的这个运算符，若扫描到的运算符优先级高，则先将其压入栈中。式子扫描完时，讲栈中运算符全部弹出放入式子。
