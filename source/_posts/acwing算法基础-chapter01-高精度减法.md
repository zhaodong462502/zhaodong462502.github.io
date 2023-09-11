title: acwing算法基础-chapter01-高精度减法
author: Zhao Dong
date: 2023-09-11 21:11:19
tags:

- 算法
- acwing

---

[原题连接](https://www.acwing.com/problem/content/794/)

## 一、算法逻辑解读

### 整体逻辑解读

1. 逆序读入两个数A，B
2. 判断A，B大小，用大数减小数
3. 减法过程

### 减法过程的解读

``` c++
vector <int> sub(vector<int>& A, vector<int> &B)
{
    vector<int> C;
    int t = 0;
    for(int i = 0; i < A.size(); i++)
    {
        t = A[i] - t;
        if(i < B.size()) t -= B[i];
        C.push_back((t + 10) % 10 ); // 合而为1
        if(t < 0)  t = 1;
        else t = 0;

    }

    while(C.size() > 1 && C.back() == 0) C.pop_back();  //去掉前导0

    return C;
}
```


以A-B为例（A>=B）
1. 用临时变量T存储计算结果，计算结果包括下面的2和3步
2. 临时变量T先减去借位，计算公式 T=a[i]-T;
3. 如果减法过程在B的范围内 再减去B的值 T-= b[i]; 
4. 减完将A的位置上的数存储到C，存储的值计算公式 c[i]=（T+10）%10，+10的原因是X有可能是负数
5. 判断临时变量T的值，T>=0(表示a[i] >b[i],不需要借位) 则重置T = 0,T<0(表示a[i] <b[i],需要借位) 则T=1

> 技巧就是T的作用：
> 	1. 当做计算临时变量 
> 	2. 当做a[i] 和b[i] 大小的比较，需不需要借位的判断
> 	3. 借位数 1 或者 0 

## 二、数值字符串和数值Int转换

``` c++
a[i] - '0'
```

> 数值字符串的Ascii码减去0的字符串的Ascii码得到的就是该字符串对于的数值
## 三、Vector的基础知识
### back()方法
> 访问向量的最后一个元素，并返回最后一个元素的引用。

### pop_back()
> 删除向量的最后一个元素，并将向量的长度减1。

**因为A和B是倒序处理的，所以高位都在向量的末尾**
