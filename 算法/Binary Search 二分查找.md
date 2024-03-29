# Binary Search 二分查找

## 概述

二分的本质是：在一个区间上定义某种性质，整 个区间可以分为满足该性质不满足该性质两部分，则二分法可以查找到这两部分的交界处两个边界点。（有单调性一定可以二分，二分不一定非要有单调性）

## 整数二分代码模板

```cpp
bool check(int x) {/* ... */} // 检查x是否满足某种性质

// 区间[l, r]被划分成[l, mid]和[mid + 1, r]时使用（查找右半部分的左边界）：
int bsearch_1(int l, int r)
{
    while (l < r)
    {
        int mid = l + r >> 1;
        if (check(mid)) r = mid;    // check()判断mid是否满足性质
        else l = mid + 1;
    }
    return l;
}
// 区间[l, r]被划分成[l, mid - 1]和[mid, r]时使用（查找左半部分的右边界）：
int bsearch_2(int l, int r)
{
    while (l < r)
    {
        int mid = l + r + 1 >> 1; //这里的 + 1是为了防止死循环
        if (check(mid)) l = mid;
        else r = mid - 1;
    }
    return l;
}
```





## 浮点数二分代码模板

```cpp
bool check(double x) {/* ... */} // 检查x是否满足某种性质

double bsearch_3(double l, double r)
{
    const double eps = 1e-6;   // eps 表示精度，取决于题目对精度的要求
    while (r - l > eps)
    {
        double mid = (l + r) / 2;
        if (check(mid)) r = mid;
        else l = mid;
    }
    return l;
}
```


