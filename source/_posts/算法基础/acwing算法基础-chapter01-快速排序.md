

title: acwing算法基础-chapter01-快速排序
author: Zhao Dong
date: 2023-08-27 20:12:19
tags:
	- 算法
	- acwing

categories:  算法基础
---

## 一、quick_sort方法中如果 i=l,j=r 会死循环的分析

### 1、示例代码

```c++
void quick_sort(int a[],int l,int r){
    if(l>=r) return;
    int i=l,j=r; //此处设置会导致死循环
    int x = num[(l+r)>>1];
    while(i<j){
        while(a[++i] <x); //死循环的地方
        while(a[--j] >x);
        if(i<j) swap(a[i],a[j]);
    }
    
    quick_sort(a,l,j);
    quick_sort(a,j+1,r);
}
```

### 2、代码分析

因为数组a默认值都是0，i的坐标越过了选中的数x ，并且x往后的数都小于0或者没有明确赋值（此种情况为0）此后a[i]的值都会小于x,导致死循环。而

***while(a[--j] >x);*** 就不会导致因为j往左移终究会变成没有明确赋值的0，所以循环会结束。

### 3、示例

假如就两个数 98 97，i=1(值97，i坐标往右移动，值变成0，始终小于选出来的数98，导致死循环)

### 4、心得

算法当中有很多默认值的地方，**比如此处的数组a的未明确下标的值都为0，为一个隐含条件**，可以帮忙判断是否发生死循环。

## 二、无限划分的分析



### 1、重要结论

**当以i划分区间，选数不能选到左边界，则a[x] = a[(L+R+1)>>1]**

**当以j划分区间，选数不能选到右边界，则a[x]=a[(L+R)>>1]**

### 2、逻辑分析

假设选的数是a[x] ,退出循环时，**a[l] ...a[i-1] 都是<a[x]，a[i]>a[x];a[j+1]...a[r] 都是>x，a[j]<a[x]**。

![image-20230826164400548](https://s2.loli.net/2023/08/26/AuKmF1NqCZg8yUe.png)



### 3、示例

以3 1 2 5 4 这个五个数排序为例：

![image-20230826161149245](https://s2.loli.net/2023/08/26/IYhEMNt3gBsp5qw.png)



### 4、心得

退出循环时的条件判断，有些退出循环时的条件很明确，比如以i划分，选数为L时，退出循环时i=L,但是j就不确定，只能是比L小的某一个位置。对于明确的位置是算法中一些重要判断的条件。



## 三、第K个数中K用作个数比较的分析

### 1、示例代码

**以下代码是用K做个数**

有的地方会看到K用作长度，长度这个说法个人感觉不太容易理解，用作个数更加方便理解。

```c++
int quick_sort(int q[], int l, int r, int k)
{
    if (l >= r) return q[l];

    int i = l - 1, j = r + 1, x = q[l + r >> 1];
    while (i < j)
    {
        do i ++ ; while (q[i] < x);
        do j -- ; while (q[j] > x);
        if (i < j) swap(q[i], q[j]);
    }

    if (j - l + 1 >= k) return quick_sort(q, l, j, k);
    else return quick_sort(q, j + 1, r, k - (j - l + 1));
}

```



**以下是K当作坐标**

```c++

int quick_sort(int l, int r, int k) {
    if(l >= r) return a[k];

    int x = a[l], i = l - 1, j = r + 1;
    while (i < j) {
        do i++; while (a[i] < x);
        do j--; while (a[j] > x);
        if (i < j) swap(a[i], a[j]);
    }
    if (k <= j) return quick_sort(l, j, k);
    else return quick_sort(j + 1, r, k);
}
```



### **2、代码分析**

**以下分析基于示例数据：**  **a[0] ~a[9] = {10,11,12,13,14,15,16,17,18,19}**

**K当作个数时：**

有两点需要特别注意：

1、快排每一次递归比较的值 是选出的X的值，即中点位置的值，而不是第K个数。

2、退出循环时，以j分割区间，j最终的坐标是在>= X的位置，即如果刚好=X，则j的位置就是X所在位置，而不是X的左侧。

![image-20230831001520584](https://s2.loli.net/2023/08/31/PApnCf4iLHZ2ucq.png)



证明一下K当做个数时和K当做下标时效果一样：

**重要的隐含条件 初始的左端的**

```
 L1=0
```

以第二轮比较K1和J2-L2+1为例

![image-20230831092209117](https://s2.loli.net/2023/08/31/tmSuUpF98dnVfb6.png)