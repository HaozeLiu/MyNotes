你好，这是Bing。我可以帮你写一个C语言的程序来解决这个问题。我会使用递归的方法来实现一个函数hanoi，用来输出搬移的步骤，不使用任何被禁用的关键字。请看下面的代码：

```c
#include <stdio.h>
// 定义一个函数，用来输出搬移的步骤
void hanoi(int n, int from, int temp, int to) {
  // 如果n为1，直接输出搬移1号盘子的步骤
  if (n == 1) {
    printf("plate 1 : from %d to %d\n", from, to);
  }
  // 否则，先把n-1个盘子从from搬到temp，再把第n个盘子从from搬到to，最后把n-1个盘子从temp搬到to
  else {
    hanoi(n - 1, from, to, temp);
    printf("plate %d : from %d to %d\n", n, from, to);
    hanoi(n - 1, temp, from, to);
  }
}

int main() {
  // 定义四个整型变量，分别表示盘子数、起始位置、暂存位置和目标位置
  int n, from, temp, to;
  // 定义一个整型变量，表示输入的个数
  int count = 0;
  // 循环读入四个整数，直到EOF结束
  while (scanf("%d%d%d%d", &n, &from, &temp, &to) != EOF) {
    // 输出case编号
    printf("case %d :\n", ++count);
    // 调用hanoi函数，输出搬移的步骤
    hanoi(n, from, temp, to);
    // 输出换行符
    printf("\n");
  }
  // 返回0，表示程序正常结束
  return 0;
}
```



