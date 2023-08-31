title: acwing算法基础-chapter01-归并排序
author: Zhao Dong
date: 2023-08-31 23:54:19
tags:

- 算法
- acwing

---



## **一、归并排序的形象理解**

[原题链接](https://www.acwing.com/problem/content/description/789/)

### **示例代码**

```c++
void merge_sort(int q[], int l, int r)
{
    if (l >= r) return;

    int mid = l + r >> 1;

    merge_sort(q, l, mid), merge_sort(q, mid + 1, r);

    int k = 0, i = l, j = mid + 1;
    while (i <= mid && j <= r) //第一处
        if (q[i] <= q[j]) tmp[k ++ ] = q[i ++ ];
        else tmp[k ++ ] = q[j ++ ];
    while (i <= mid) tmp[k ++ ] = q[i ++ ]; // 第二处
    while (j <= r) tmp[k ++ ] = q[j ++ ]; //第三处

    for (i = l, j = 0; i <= r; i ++, j ++ ) q[i] = tmp[j]; //数组回填也很重要
}

```

### **代码分析**

**大佬做的示意图，有助于形象理解**

![](https://raw.githubusercontent.com/zhaodong462502/markdownImage/master/bloging/1130_4cf170747a-3.gif)

**个人理解**：

几个重要的点

1、*先递归后排序*的目的是为了**深入到最小区间开始排序**，最小区间即两个元素，如上图示意。

2、选取的mid是向下取整，所以左半部区间都是2的整数倍

3、递归的顺序都是从左到右开始

**三处循环的理解：**

1、循环第一处两个区间相等如果排序的数总数为偶数，则只会在循环第一处处理结束，如果排序的数总数为奇数，则还有可能进入循环第二处、循环第三处处理。

2、 i和j循环中可以如此交换赋值前提是最小区间（区间内个数为2）都是排好序的，不然无法按照此方法排序，示例如下图

![image-20230831234028867](https://raw.githubusercontent.com/zhaodong462502/markdownImage/master/bloging/image-20230831234028867.png)
