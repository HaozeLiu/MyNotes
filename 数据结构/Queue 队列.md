# Queue 队列

队列与栈相反，具有先进先出（First-In-First-Out) 的特点。

插入要在队尾操作，删除在队头操作。队列有以下几种操作：

1. `EnQueue(x)  / Push(x)` 插入一个元素
2. `Dequeue() / Pop()` 删除队头一个元素
3. `Front() / Peek()` 返回队列头部的元素
4. `IsEmpty()` 检查队列是否为空

以上操作时间复杂度一定要为`O(1)`



## 使用数组实现队列

```cpp
#define MAX_SIZE 101

class Queue
{
private:
    int A[MAX_SIZE];
    int front, rear;
public:
    Queue()
    {
        front = -1;
        rear = -1;
    }

    bool IsEmpty()
    {
        return (front == -1 && rear == -1);
    }

    bool IsFull()
    {
        return (rear + 1) % MAX_SIZE == front ? true : false;
    }

    void Enqueue(int x)
    {
        cout<<"Enqueuing"<<x<<" \n";
        if (IsFull())
        {
            cout<<"Error: Queue is full\n";
            return;
        }
        if (IsEmpty())
        {
            front = rear = 0;
        }
        else
        {
            rear = (rear + 1) % MAX_SIZE;
        }
        A[rear] = x;
    }

    void Dequeue()
    {
        cout<<"Dequeuing \n";
        if (IsEmpty())
        {
            cout<<"Error: Queue is empty\n";
            return;
        }
        else if(front == rear) //代表队列中只有一个数
        {
            rear = front = -1;
        }
        else
        {
            front = (front + 1) % MAX_SIZE;
        }
    }

    int Front()
    {
        if (front == -1)
        {
            cout<<"Error: cannot return front from empty queue\n";
            return -1;
        }
        return A[front];
    }
```

代码中几处**取模运算**的含义是：利用**循环数组**，使得在 rear 到顶且 front 不在 0 处时数组中的几处删除过的废弃空间得以利用。



## 使用链表实现队列

我们使用链表实现队列，想在队尾插入新元素时，需要从头节点遍历到链表末尾然后进行插入操作，这样就会是`O(n)` 的复杂度。但如果我们在创建链表时**多设一个指向尾部的指针**，并且随着链表的变化而变化，这样队列的任何操作就都是`O(1)`的复杂度了。

```c
typedef struct Node
{
    int data;
    struct Node* next;
}Node;

Node* front = NULL;
Node* rear = NULL;

void Enqueue(int x)
{
    Node* temp = (Node*) malloc(sizeof(Node));
    temp->data = x;
    temp->next = NULL;
    if(front == NULL && rear == NULL)//队列为空时
    {
        front = rear = temp;
        return;
    }
    rear->next = temp;
    rear = temp;
}

void Dequeue()
{
    Node* temp = front;
    if (front == NULL)
    {
        printf("Queue is empty\n");
        return;
    }
    if (front == rear)
        front = rear = NULL;//仅剩一个元素时
    else front = front->next;

    free(temp);
}

int Front()
{
    if (front == NULL)
    {
        printf("Queue is empty/n");
        return -1 ;
    }
    return front->data;
}

void Print()
{
    Node *temp = front;
    while (temp != NULL)
    {
        printf("%d ",temp->data);
        temp = temp->next;
    }
    printf("\n");
}

```

