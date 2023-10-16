质数定义

质数是指在大于1的自然数中，除了1和它本身不再有其他因数的自然数。

质数相关题型



筛质数

| 题型大类 | 题型小类     | 区别                                                         |                                       |      |
| -------- | ------------ | ------------------------------------------------------------ | ------------------------------------- | ---- |
| 求质数   | 试除法求质数 | 1、给定的数是不连续的数  2、数的***个数***的范围小 1<n<1000 3、数本身的范围大 1<a<10^9 | 1、遍历a时，只能遍历到a/i，否则会超时 |      |
|          | 求质因数     | 同上                                                         | 1、遍历a时，只能遍历到a/i，否则会超时 |      |
|          | 筛法求质数   | 1、给定的数是连续的数  2、数的***个数***的范围大 1<n<10^ 6  3、数本身的范围大1<a<10^9 |                                       |      |
| 求约数   |              |                                                              |                                       |      |
|          |              |                                                              |                                       |      |

暂存代码

```



int primes[N]
bool st[N];
int cnt=0;
for(int i=2;i<=n;i++){
    
    bool flag = true;
    for(int j=0;j<cnt;j++){
        if(i % primes[j] ==0) {
            flag = false;
             break;
        }
           
    }
    if(flag) st[i]=true;
    else st[i]=false;
    
    if(!st[i]) primes[cnt++] = i;

    
    
}
```

筛质数

![image-20231009153900577](https://s2.loli.net/2023/10/09/9PdsAYFcfbNEGxS.png)



## 扩展欧几里得举例推导过程

![image-20231011192935613](C:\Users\Fujitsu\AppData\Roaming\Typora\typora-user-images\image-20231011192935613.png)



## 中国剩余定理

推导过程

![image-20231012135229797](https://s2.loli.net/2023/10/12/dB8KRhOkfzDJ2ib.png)

![image-20231012143145086](https://s2.loli.net/2023/10/12/Xu8rH2F7IqJt19j.png)
