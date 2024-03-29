# Merge Sort 归并排序



## 基本思路

1. **确定分界点**  以中间位置 mid = （l + r）/ 2 作为分界点，分出 left ，right 两部分。
2. **递归排序**   递归排序 left right，使它们分别成为有序数组。
3. **归并**   让两个有序数组合二为一形成新的一整个有序数组。==重点==

## 归并两部分的思路 

（双路归并，合二为一） 

新开辟一个数组用来储存最终合并后的数，用两个指针分别先指向left 和right的第一个数即最小数，比较两个指针所指数的大小，将较小数放到新数组里，该指针向后挪移一位，继续比较。两数相同时随便放哪个均可，我们为了稳定性就固定放所指的left中的数。一直重复该操作，直到

某个指针走完整个部分，再将另一部分指针指向的数与剩余数全部放入新数组，则完成归并。   

## 代码模板

```cpp
void merge_sort(int q[], int l, int r)
{
    if (l >= r) return;

    int mid = l + r >> 1;
    merge_sort(q, l, mid);
    merge_sort(q, mid + 1, r);

    int k = 0, i = l, j = mid + 1;
    while (i <= mid && j <= r)
        if (q[i] <= q[j]) tmp[k ++ ] = q[i ++ ];
        else tmp[k ++ ] = q[j ++ ];

    while (i <= mid) tmp[k ++ ] = q[i ++ ];
    while (j <= r) tmp[k ++ ] = q[j ++ ];

     for (i = l, j = 0; i <= r; i ++, j ++ ) q[i] = tmp[j];
} 
```

