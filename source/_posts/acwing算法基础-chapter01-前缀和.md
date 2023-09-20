## 一维数组的前缀和

```c++
s[i] =s[i-1]+a[i]
```



## 二维数组的前缀和

> 总体思想：先计算以（1,1）为起始点 到各个点的区域总和，再用差值法计算给定的起始点到终点的区域总和

### 计算以（1,1）起始点的区域的总和

```c++
//计算s[i][j]的值
s[i][j] = s[i-1][j]+s[i][j-1]-s[i-1][j-1]+a[i][j];
```

![image-20230920144354911](https://raw.githubusercontent.com/zhaodong462502/markdownImage/master/blogingimage-20230920144354911.png)



### 计算给定区域的总和

```c++
//计算（x1,y1）到（x2,y2）区域的值
s[x2][y2]-s[x1-1][y2]-s[x2][y1-1]+s[x1-1][y1-1]; //中间-1的部分都是起始点（x1,y1）的坐标

```

![image-20230920143950495](https://raw.githubusercontent.com/zhaodong462502/markdownImage/master/blogingimage-20230920143950495.png)

