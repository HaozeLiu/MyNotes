# Linkedlist链表

每个元素都是一个由两部分组成的**结构体**，一部分为**数据域**，存储数据，另一部分为**指针域**，存储下一元素的地址，每个元素这样串起来就像链子一样。

## 数组 vs 链表

1. **数据访问**：数组可以以 O(1) 的时间来访问任何一个元素， 但链表必须遍历，时间成本为 O(n)。
2. **内存占用上**：如果数据规模较大，我们可能没有那么大的一块连续内存给创建数组，但我们使用链表则可以碎片利用空间；如果想要扩充数据，那么数组就很麻烦，必须新建一个更大的数组，再把数据都拷贝过去，操作繁冗且内存占用大，而链表则操作简便且空间占用小；
3. **插入元素成本**：==1)开头插入元素== 数组O(n)，链表O(1); ==2) 末尾插入元素== 若数组大小足够则为O(1)，不够则需要创建新数组且拷贝之前所有内容，为O(n)；链表无论如何都得遍历到最后在创建一个新元素，为O(n)；==3) 在第 i 个位置插入元素== 均为O(n)
4. **实现难度** :毫无疑问,数组简单链表较难.

## 链表的 C/CPP 实现 

```Cpp
typedef struct node
{
    int data;
    struct node* next;
};

node* head; //用来指向链表头节点的指针
head = null; //链表为空时A指向空

//给空链表创建第一个新节点,数据为2
node* temp = (node*)malloc(sizeof(node));//or node* temp = new node();
(*temp).data = 2;//or temp -> data = 2;
(*temp).next = null;//or temp -> next = null;
head = temp;

//再添加一个新节点,数据为4
temp = new node();
temp -> data = 4;
temp -> next = null;

node* temp1 = head;
while(temp1 != null)
{
    temp1 = temp1 -> next;
}//遍历到链表末节点
temp -> next = temp;//这样链表串上新节点了
```



## 头部插入一个节点

```c
#include<stdlib.h>
#include<stdio.h>
typedef struct node{
    int data;
    struct node* next;
}node;
node* head;//全局变量

//头部插入节点函数
void Insert(int x)
{
    node* temp = (node*)malloc(sizeof(node));
    temp -> data = x;
    temp -> next = head;//可以覆盖掉链表为空的情况
    head = temp;//让 head 和 temp 指向同一块
}

//链表逐个打印函数
void Print()
{
    node* temp = head;//这样才能保证打出第一个节点
    
    printf("List is : ");
    while(temp != null)
    {
        printf(" %d", temp -> data);
        temp = temp -> next;
    }
    printf("\n");
}

int main()
{
    head = null;
    printf("How many numbers?\n");
    int n, x;
    scanf("%d", &n);
    
    for(int i = 0; i < n; i++)
    {
        printf("Enter the number \n");
        scanf("%d", &x);
        Insert(x);
        Print();
    }
    
    return 0;
}
```



## 任意位置插入一个节点

```c
#include<stdlib.h>
#include<stdio.h>
typedef struct Node{
    int data;
    struct Node* next;
}Node;
Node* head;

void Insert(int data, int n)
{
    Node* temp1 = (Node*)malloc(sizeof(Node));
    temp1 -> data = data;
    temp1 -> next = NULL;
    
    if(n == 1)
    {
        temp1 -> next = head;
        head = temp1;
        return;
    }
    Node* temp2 = head;
    for(int i = 0; i < n - 2; i++)
    {
        temp2 = temp2 -> next;
    }
    temp1 -> next = temp2 -> next;
    temp2 -> next = temp1;
}
void Print()
{
    Node* temp = head;
    while(temp != NULL)
    {
        printf("%d ", temp -> data);
        temp = temp -> next;
    }
    printf("\n");
}

int main()
{
    head = NULL;//empty list
    Insert(2, 1);//List : 2
    Insert(3, 2);//List : 2, 3
    Insert(4, 1);//List : 4, 2, 3
    Insert(5, 2);//List : 4, 5, 2, 3
    Print();
    
    return 0;
}
```



## 任意位置删除一个节点

```c
#include<stdio.h>
#include<stdlib.h>
typedef struct Node{
	int data;
	struct Node* next;
}Node;
Node* head;
void Insert(int data)//insert a integer at the end of list
{
    Node* temp1 = (Node*)malloc(sizeof(Node));
    temp1 -> data = data;
    temp1 -> next = NULL;
    
    if(head == NULL)
        head = temp1;
    else
    {
    	Node* temp2 = head;
    	while(temp2 -> next != NULL)
    		temp2 = temp2 -> next;
    	temp2 -> next = temp1;
    }
}
void Print()
{
    Node* temp = head;
    printf("List is : ");
    while(temp != NULL)
    {
        printf("%d ", temp -> data);
        temp = temp -> next;
    }
    printf("\n");
}
void Delete(int n)
{
    Node* temp1 = head;
    if(n == 1)
    {
        head = temp1 -> next;
        free(temp1);
        return;
    }
    for(int i = 0; i < n - 2; i++)
    	temp1 = temp1 -> next;
    Node* temp2 = temp1 -> next;//nth Node
    temp1 -> next = temp2 -> next;
    free(temp2);
}
int main()
{
    head = NULL;//empty list
    Insert(2);
    Insert(4);
    Insert(6);
    Insert(5);//List : 2, 4, 6, 5
    
    int n;
    printf("Enter a position\n");
    scanf("%d", &n);
    Delete(n);
    Print();
    
    return 0;
}
```



## 反转一个链表

### 迭代方式实现

```c
typedef struct Node
{
    int data;
    struct Node* next;
}Node;
Node* head;

void Reverse()
{
    Node *current, *prev, *next;
    current = head;
    prev = NULL;
    while(current != NULL)
    {
        next = current -> next;
        current -> next = prev;
        prev = current;
        current
    }
    head = prev;
}
```



### 递归方式实现（有点难啊 qwq ）

```c
typedef struct Node{
    int data;
    struct Node* next;
}Node;
Node* head;
void Reverse(Node* p)
{
    if(p -> next == NULL)
    {
        head = p;
        return;
    }
    
    Reverse(p -> next);
    p -> next -> next = p;
    p -> next = NULL;
}

```



### 头节点不用全局变量版本

```cpp
typedef struct Node{
    int data;
    struct Node* next;
}Node;

Node* Reverse(Node* head)
{
    if(head == NULL || head->next == NULL)
        return head;
    
    Node* last = Reverse(head->next);
    head->next->next = head;
    head->next = NULL;
    return last;
}
```



以上都是关于**单链表**的内容，下面我们要讲讲**双向链表**

---

## 双向链表

顾名思义，双向链表每个节点都有两个链接，一个指向前一个结点，一个指向下一个节点。

```c
typedef struct Node{
    int data;
    struct Node* next;
    struct Node* prev;
}Node;
```





