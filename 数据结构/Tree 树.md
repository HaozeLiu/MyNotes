# Tree 树



## 大体的定义与性质

树可以被定义为：链接在一起的被称为节点的实体的集合，用来模拟层级结构。

最上面的节点为**根节点（root）**，每个节点衍生出来的节点称为它的**子节点（children）**，而它是子节点的**双亲节点（parent）**，有着同一个双亲结点的节点互为**兄弟节点（sibling）**，没有子节点的节点称为**叶子节点（leaf）**，其它所有有孩子的节点称为**内部节点**。

==注意，树中每个节点之间的链接都是**单向**的，只能**从上到下**==

某个节点的**深度（depth）**定义为：该点和根节点之间的路径的长度（按初始层为第零层，就是在第几层深度就为几）。根节点的深度为0.

某个节点的**高度（height）**定义为：该点到叶子节点的最长路径（注意链接是单向的，只能往下找叶子节点）。叶子节点的高度为0. **一棵树的高度就是它的根节点的高度**。



## 二叉树（Binary Tree）

每个节点最多有**两个**子节点的树被称为**二叉树（Binary Tree）** 每个节点只有两个孩子或者没有孩子的二叉树被称为**严格二叉树**。除了最后一层每层节点都有两个子节点，而且最后一层的节点都靠左侧集中，那么这个树为**完全二叉树（Complete Binary tree）** 所有层级全被填满的二叉树称为**完美二叉树（Perfect Binary tree）**

我们可以创建一个结构体，其中一个整型变量来存放数据，两个指针用来指向俩孩子，这样就可以实现二叉树了。

```cpp
struct Node
{
    int data;
    Node* left;
    Node* right;
};//二叉树
```



我们也可以用数组实现完全二叉树，头节点编号为0，它的左孩为1右孩为2，依次编号放到数组里存储。对于编号为`i`的点，它的左孩编号为`2i + 1`,右孩编号为`2i + 2`。

### 性质：

**叶结点个数等于度为2的结点个数加1**





### 二叉搜索树（Binary Search Tree）

二叉搜索树的每一个节点的左子树上的节点值都比该点小，右子树上的节点值都比该点大。

二叉搜索树进行查找，插入，删除三个操作的时间复杂度都为`O(logn)`，较为优异。但必须让二叉搜索树**平衡（左子树和右子树的高度差不大于1）**才能保证平均时间复杂度为`O(logn)`。

#### 代码实现

```cpp
//Binary Search Tree
#include<iostream>
using namespace std;
struct BstNode
{
    int data;
    BstNode* left;
    BstNode* right;
};

BstNode* GetNewNode(int data)
{
    BstNode* newNode = new BstNode();
    newNode->data = data;
    newNode->left = newNode->right = NULL;
    return newNode;
}

BstNode* Insert(BstNode* root, int data)
{
    if(root == NULL)
        root = GetNewNode(data);
    else if (data <= root->data)
        root->left = Insert(root->left, data);
    else
        root->right = Insert(root->right, data);

    return root;
}

bool Search(BstNode* root, int data)
{
    if (root == NULL) return false;
    else if (root->data == data) return true;
    else if (data <= root->data) return Search(root->left, data);
    else return Search(root->right, data);
}
```



#### 二叉搜索树查找最大值最小值

##### 迭代法

```cpp
struct BstNode
{
    int data;
    BstNode* left;
    BstNode* right;
};

int FindMin(BstNode* root)
{
    if(root == NULL)
    {
        cout<<"Error: Tree is empty\n";
        return -1;
    }
   
    //这里root是形式参数，可以直接用，不会影响其他函数中的root
    while(root->left != NULL)
        root = root->next;
    
    return root->data;
}//迭代找 最大值 只需要把left改成right即可
```



##### 递归法

```cpp
struct BstNode
{
    int data;
    BstNode* left;
    BstNode* right;
};

int FindMin(BstNode* root)
{
    if(root == NULL)
    {
        cout<<"Error: Tree is empty\n";
        return -1;
    }
    else if(root->left == NULL)
        return root->data;
    
    return FindMin(root->left);
}
//要找 最大值 同理只需把left改为right
```



### 查找二叉树的高度 / 最大深度

回顾概念  高度（height）：该节点到**叶子节点**的**最长**路径。整棵树的高度即为根节点的高度

​				深度（depth）：该节点到**根节点**的路径的长度

#### 代码实现

每个点的高度是它的左右子树高度较大者+1，即`h(x) = max( h(x.l), h(x.r) ) + 1`

明显我们可以用递归解决

```cpp
struct BT
{
    int data;
    BT* left;
    BT* right;
};

int FindHeight(Node * root)
{
    if(root == NULL)
        return -1;//因为后面计算高度要 + 1，这里用-1为了保证叶子节点高度为0
    return max(FindHeight(root->left), FindHeight(root->right)) + 1;
}
```



### 二叉树的遍历

- 1 广度优先（Breadth-first)  / 层序遍历（Level-order）
- 2 深度优先（Depth-first）
- - `<root> -> <left> -> <right> `- 前序遍历（Preorder）即先根节点，再左子树，再右子树（左右子树以相同的方式**递归**）
  - `<left> -> <root> -> <right> `- 中序遍历（Inorder）
  - `<left> -> <right> -> <root>`- 后序遍历（Postorder）

#### 层序遍历

每次遍历完一整层再继续对下一层遍历。显然，再像往常那样用一个指针一个个迭代是行不通的，因为迭代到一个子树之后就没法到达另一个子树了。我们可以这样做：每遍历到一个节点后，把它的所有子树地址都存到**队列**里，然后继续对队列里的数据进行相同的操作，如此循环往复。

```cpp
/* Binary tree - Level Order Traversal */
void LevelOrder(Node *root)
{
    if (root == NULL) return;
    queue<Node*> Q;
    Q.push(root);
    while(!Q.empty())
    {
        Node* current = Q.front();
        cout<<current->data<<" ";
        if (current->left != NULL) Q.push(current->left);
        if (current->right != NULL) Q.push(current->right);
        Q.pop();
    }
}
```

时间复杂度`O(n)` 空间复杂度 `O(1) - best`  `O(n) - Worst / Avg`



#### 深度优先

也就是DFS

- 1 前序遍历（Preorder）：`<root> -> <left> -> <right> `

```cpp
struct Node
{
    char data;
    Node* left;
    Node* right;
};
void Preorder(Node* root)
{
    if(root == NULL) return;
   
    cout << root->data << " ";
    Preorder(root->left);
    Preorder(root->right);
}
```



- 2  中序遍历（Inorder) ：`<left> -> <root> -> <right> `

  ```cpp
  void Inorder(Node *root)
  {
      if(root == NULL) return;
      
      Inorder(root->left);
      cout << root->data << " ";
      Inorder(root->right);
  }
  ```

  

- 3 后序遍历（Postorder）:  `<left> -> <right> -> <root>` 

```cpp
void Postorder(Node *root)
{
    if(root == NULL) return;
    
    Postorder(root->left);
    Poseorder(root->right);
    cout << root->data << " ";
}
```



由上可见得其实三种不同次序的遍历其实就是两个对左右子树的递归语句与打印语句的不同顺序组合。时间复杂度均为`O(n)`,  空间复杂度 `O(h) // h 即为树的高度,worst:O(n), best / average: O(logn)`



### 判断是否为二叉搜索树

这是一个著名的面试问题，问你一个二叉树是否为二叉搜索树 `（Check if a given binary tree is BST）`。

回顾一下二叉搜索树的定义：每个节点的左子树上的节点都比它小，每个右子树上的节点都比它大。显然在BST中，一个节点的左子树也是BST，右子树同样如此。因此我们可以用递归

首先我们看一个简单粗暴的方法：

```cpp
bool IsSubtreeLesser(Node* root, int value)
{
    if(root == NULL) return true;
    if(root->data <= value
       && IsSubtreeLesser(root->left,value)
       && IsSubtreeLesser(root->right, value))
        return true;
    else
        return false;
}

bool IsSubtreeGreater(Node* root, int value)
{
    if(root == NULL) return true;
    if(root->data >value
       && IsSubtreeGreater(root->left, value)
       && IsSubtreeGreater(root->right, value))
        return true;
    else
        return false;
}

bool IsBinarySearchTree(Node* root)
{
    // return true if BST, false otherwise
    if(root == NULL) return true;

    if(IsSubtreeLesser(root->left, root->data)
       && IsSubtreeGreater(root->right, root->data)
       && IsBinarySearchTree(root->left)
       &&IsBinarySearchTree(root->right))
        return true;
    else
        return false;
}
```

我们可以看出这个方法太过粗暴，时间复杂度过高

我们可以在`IsBinarySearchTree`函数中多加数据上下限两个参数，即`maxValue  & minValue`，这样在判断时可以省去`IsSubtreeGreater, IsSubtreeLesser`两个时间消耗过大的函数， 将这两个条件改为`root->data > minValue && root->data < maxValue` 具体代码如下：

```cpp
bool IsBinarySearchTree(Node* root, int minValue, int maxValue)
{
    // return true if BST, false otherwise
    if(root == NULL) return true;

    if(root->data > minValue
       && root->data < maxValue
       && IsBinarySearchTree(root->left, minValue, root->data)
       &&IsBinarySearchTree(root->right, root->data, maxValue))
        return true;
    else
        return false;
}
```

不过我们每次调用函数都需要传三个值有点麻烦，为了省点劲我们可以将`IsBinarySearchTree`重命名为一个工具函数`IsBstUtil`,然后再搞一个`IsBinarySearchTree`函数，参数只有`root`,然后在里面调用`IsBstUtil(root, INT_MIN, INT_MAX), ` `(PS:INT_MIN, INT_MAX都是宏，为整型的最小和最大值)`

```cpp
bool IsBstUtil(Node* root, int minValue, int maxValue) {
    // return true if BST, false otherwise
    if (root == NULL) return true;

    if (root->data > minValue
        && root->data < maxValue
        && IsBstUtil(root->left, minValue, root->data)
        && IsBstUtil(root->right, root->data, maxValue))
        return true;
    else
        return false;
}

bool IsBinarySearchTree(Node* root)
{
    return IsBstUtil(root, INT_MIN, INT_MAX);
}
```

不过我们上面的代码都是建立在二叉树没有重复值的基础上的，如果有重复值，判断大小条件里加上等于就得了。



### 二叉搜索树删除一个节点

我们删除二叉搜索树的节点后**仍然要保持二叉搜索树的特性**

1. 删除叶子节点时：直接删除之后BST的特性仍然保留，无需其他处理。因此只需要

   > 1. 删除父节点的引用  2.擦除内存 

2. 删除有一个孩子的节点时： 先让待删除结点的子节点全部链接到它的父节点上，然后再把它删除。

3. 删除有两个孩子的节点时：找到右子树中最小的点，将待删除点值改为这个Rmin值，然后根据Rmin节点情况（为叶子节点 / 只有一个孩子）按照前两种方法删除。

> Rmin点是右子树中的**最小值**，所以它**必然没有左孩**，也就是说它**最多只有一个子节点**，这样就可以将情况**降价**为前两种删除情形

当然我们也可以取左子树中最大的点Lmax，与Rmin情况同理。



代码实现如下：

```cpp
/* Deleting a node from Binary search tree */
#include "iostream"
using namespace std;
struct Node{
    int data;
    Node *left;
    Node *right;
};

Node* FindMin(Node* root)
{
    while(root->left != NULL) root = root->left;
    return root;
}

Node* Delete(Node *root, int data) {
    if (root == NULL) return root;
    else if (data < root->data) root->left = Delete(root->left, data);
    else if (data > root->data) root->right = Delete(root->right, data);
    else // Found it, now ready to delete
    {
        // Case 1: No child
        if (root->left == NULL && root->right == NULL) {
            delete root;
            root == NULL;
        }
        // Case 2: One child
        else if (root->left == NULL) {
            Node *temp = root;
            root = root->right;
            delete temp;
        }
        else if (root->right == NULL) {
            Node *temp = root;
            root = root->left;
            delete temp;
        }
        // Case 3: 2 children
        else {
            Node *temp = FindMin(root->right);
            root->data = temp->data;
            root->right = Delete(root->right, temp->data);
        }
    }
    return root;
}
```





### 二叉搜索树的中序后继节点

如果我们对二叉搜索树进行中序遍历（`<left> -> <root / visit> -> <right>`），将得到按升序排列的节点。

**前驱节点**：中序遍历后某个节点的前一个节点为它的前驱节点

**后继节点**：中序遍历后某个节点的后一个节点为它的后继节点

我们如何快速的找出某个节点的后继节点？首先，我们可以先中序遍历完之后直接找出来，但这样做的时间复杂度太高了。

Case 1：有右子树 

在右子树中一直向左找到最后一个节点 （BST中即找到右子树中最小的点）

Case 2：没有右子树

下一个访问（`visit`）的节点必定是它的后继节点 (也就是往上找第一个还没有visited的点)

代码如下：

```cpp
/* Deleting a node from Binary search tree */
#include "iostream"
using namespace std;
struct Node{
    int data;
    Node *left;
    Node *right;
};

Node* Find(Node* root, int data)
{
    if (root == NULL) return NULL;
    else if (root->data == data) return root;
    else if (root->data < data) return Find(root->right, data);
    else return Find(root->left, data);
}

//Function to find successor
Node* Getsuccessor(Node* root, int data)
{
    // Search the Node
    Node* current = Find(root, data);
    if (current == NULL) return NULL;
    // Case 1: Node has right subtree
    if (current->right != NULL)
    {
        Node* temp = current->right;
        while (temp->left != NULL) temp = temp->left;
        return temp;
    }
    // Case2: No right subtree
    else
    {
        Node* successor = NULL;
        Node* ancestor = root;
        while(ancestor != current)
        {
            if (current->data < ancestor->data)
            {
                successor = ancestor;
                ancestor = ancestor->left;
            } else
                ancestor = ancestor->right;
        }
        return successor;
    }
}
```

