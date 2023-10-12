筛质数

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
