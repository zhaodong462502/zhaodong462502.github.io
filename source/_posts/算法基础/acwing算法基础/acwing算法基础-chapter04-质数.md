质数定义

质数是指在大于1的自然数中，除了1和它本身不再有其他因数的自然数。

质数相关题型



筛质数

| 题型大类     | 题型小类           | 区别                                                         |                                                              | 例题                                                         |
| ------------ | ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 求质数       | 试除法求质数       | 1、给定的数是不连续的数  2、数的***个数***的范围小 1<n<1000 3、数本身的范围大 1<a<10^9 | 1、遍历a时，只能遍历到a/i，否则会超时                        | [试除法判定质数](https://www.acwing.com/activity/content/problem/content/935/) |
|              | 求质因数           | 同上                                                         | 1、遍历a时，只能遍历到a/i，否则会超时                        | [分解质因数](https://www.acwing.com/activity/content/problem/content/936/) |
|              | 筛法求质数         | 1、给定的数是连续的数  2、数的***个数***的范围大 1<n<10^ 6  3、数本身的范围大1<a<10^9 | 不需要单独遍历具体的数                                       | [筛质数](https://www.acwing.com/activity/content/problem/content/937/) |
| 求约数       | 试除法求约数       | 约数是成对出现，且最多只有1个大于根号n的约数                 | 遍历时只需要遍历到n/i即可                                    | [试除法求约数](https://www.acwing.com/activity/content/problem/content/938/) |
|              | 约数的个数         | 如果a=p0^a0+...pk^ak 则约数个数=(a0+1)\*...\*(ak+1)，p为a的质因数 |                                                              | [约数的个数](https://www.acwing.com/activity/content/problem/content/939/) |
|              | 约数的和           | 如果a=p0^a0+...pk^ak 则约数的和=(p0^0+p0^1...+p0^a0)\*...\*(pk^0+pk^1+...+...pk^ak)，p为a的质因数 |                                                              | [约数的和](https://www.acwing.com/activity/content/problem/content/940/) |
|              | 最大公约数         | gcd(a,b) = gcd(b,a%b)                                        |                                                              | [最大公约数](https://www.acwing.com/activity/content/problem/content/941/) |
| 欧拉函数     | 求给定数的欧拉函数 | phi(n) = n\*(1-1/p0)\*(1-1/p1)\*...\*(1-1/pk);               | 给定的数不连续                                               | [欧拉函数](https://www.acwing.com/activity/content/problem/content/942/) |
|              | 筛法求欧拉函数     | phi(n) = n\*(1-1/p0)\*(1-1/p1)\*...\*(1-1/pk);   phi(p) = p -1,为质数。因为求的数是连续的，所以可以用筛法。 |                                                              | [筛法求欧拉函数](https://www.acwing.com/activity/content/problem/content/943/) |
| 快速幂       | 求逆元             | 计算a^b ，将b转化为二进制求解b=2^0+2^1+...+2^n               | 逆元证明：a/b = a*x (mod p)，则称x是b在模p上的逆元。当p为质数时，费马小定理 a^phi(p) = 1(mod p )，即a^(p-1) = 1(mod p),则逆元x为a^(p-2)。 | [快速幂求逆元](https://www.acwing.com/activity/content/problem/content/945/) |
| 扩展欧几里得 | 求逆元             | a*x = 1(mod p),转化为二项式 a\*x+p\*y = 1,求出a、p的最大公约数d,d=1即存在逆元 |                                                              |                                                              |
|              | 线性同余           | a1\*x1=b1(mod p1) ，a2 \* x2 = b2(mod p2)  ; a1\*x1+p1\*y1=b1。a1\*x1+p1*\y1=gcd(a1,p1), gcd(a1,p1) = gcd(p1,a1%p1); x1=y11,y1 = y1-d/b\*x1; | 利用gcd(a1,p1) = gcd(p1,a1%p1)构造两个二项式相等从而推导递推公式 | [线性同余](https://www.acwing.com/activity/content/problem/content/947/) |
|              | 中国剩余定理       | 两个二项式直接根据题意可以得到，a1\*k1+p1= a2\*k2+p2，k1 和k1+a2/d*k都是 这个等式的解可以推导出新的k和p的取值 从何循环计算 | k1 和k1+a2/d*k都是 这个等式的解                              | [表达整数的奇怪方式](https://www.acwing.com/activity/content/problem/content/948/) |
| 组合数       | 组合数1            | c(a,b),a和b的范围在1<n<10^4,1<b<a<10^4，结果对10^9+7取余，用公式c(a,b) = c(a-1,b)+c(a-1,b-1) 递推求解 |                                                              | [组合数1](https://www.acwing.com/activity/content/problem/content/955/) |
|              | 组合数2            | c(a,b),a和b的范围在1<n<10^4，1<b<a<10^5,结果对10^9+7取余，用组合数定义a!/(b!*(a-b)!)。因为结果取余，所以除数部分需要求逆元 |                                                              | [组合数2](https://www.acwing.com/activity/content/problem/content/956/) |
|              | 组合数3            | c(a,b),a和b的范围在1<n<20，1<b<a<10^18,结果对10^9+7取余。c(a,b) = c(a0,b0)\*...\*c(ak,bk),a=a0\*p^0+...+ak*p^ak,b=b0\*p^0+..+bk*p^bk。a0 = a/p % p,a1=a/p^2%p...ak = a/p^k%p;b0=b/p%p...bk=a/p^k%p。lucas定理 |                                                              | [组合数3](https://www.acwing.com/activity/content/problem/content/957/) |
|              | 组合数4            | c(a,b),a和b的范围在1<b<a<5000 ，结果不取余。利用公式a!/(b!*(a-b)!)分别算出a、b、a-b的质因子和其指数，通过消去多余指数 最终结果就是质因子大数和质因数的指数相乘的高精度算法。质因子部分用筛法。 |                                                              | [组合数4](https://www.acwing.com/activity/content/problem/content/958/) |
|              | 卡特兰数           | n个0和n个1满足0比1多的排列的个数。结果就是卡特兰数c(2n,n)-c(2n,n-1) = c(2n,n)/(n+1)。其中c(2n,n)的含义是 从2n个数中挑选横向n个数 。 |                                                              | [满足条件的01序列](https://www.acwing.com/activity/content/problem/content/959/) |

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
