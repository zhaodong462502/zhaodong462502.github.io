## 前导0计数

### 问题

1的前导0个数为31，对应16进制数为0x0000 0001。  
2的前导0个数位30，对应16进制数为0x0000 0002。

如何计算一个int型数对应二进制数前导0的位数呢？

### 解决思路

#### 轮询bit位算法

开始想到的最简单的办法是从左边最高位开始，向低位轮训，一直到遇见位1。

```java
// 轮询bit位 int numberOfLeadingZerors(int x) { int n = 0; for (int i = 31; i >= 0; i--) { if ( x & (1 << i) != 0) { break; } else n++; } return n; }
```

这样，最多需要循环32次，每个循环执行4个指令：判断、按位与、移位、递增。这样可能需要执行32 x 4条指令。

#### 二分搜索算法

有没有更快的算法？  
可以使用二分搜索技术计算前导0。这里二分搜索，没有用索引指示器，而是用到了进制数的特性，直接利用二进制数 + 移位操作，来作为0个数的判断点。比如0x0000FFFF，x <= 0x0000FFFF，意味着x至少有16个前导0。

从下面的代码可以看出，二分搜索法所需要的指令最多为20个（6个if，10个赋值4个移位）指令操作。

```java
// 二分搜索计算前导0 int numberOfLeadingZeros(int x) { if (x == 0) return 32; int n = 0; if (x <= 0x0000FFFF) { n = n + 16; x = x << 16; } // 判断高16位是否为0：先通过高16位，判断该数是否位于较小的一半，如果是，说明至少会有16个0，再把该数左移16位，相当于增大模的一半 if (x <= 0x00FFFFFF) { n = n + 8; x = x << 8; } // 判断高8位 if (x <= 0x0FFFFFFF) { n = n + 4; x = x << 4; } // 判断高4位 if (x <= 0x3FFFFFFF) { n = n + 2; x = x << 2; } // 判断高2位 if (x <= 0x7FFFFFFF) { n = n + 1; } // 判断高1位 return n; }
```

按位与版本，二分搜索计算前导0

```java
// 二分搜索计算前导0 int numberOfLeadingZeros(int x) { if (x == 0) return 32; int n = 0; if ((x & 0xFFFF0000) == 0) { n = n + 16; x = x << 16; } // 判断高16位是否为0：先通过高16位，判断该数是否位于较小的一半，如果是，说明至少会有16个0，再把该数左移16位，相当于增大模的一半 if ((x & 0xFF000000) == 0) { n = n + 8; x = x << 8; } // 判断高8位 if ((x & 0xF0000000) == 0) { n = n + 4; x = x << 4; } // 判断高4位 if ((x & 0xC0000000) == 0) { n = n + 2; x = x << 2; } // 判断高2位 if ((x & 0x80000000) == 0) { n = n + 1; } // 判断高1位 return n; }
```

最后一个if语句，可以改写成 : `n = n + 1 - (x >> 31)`  
如果将n初值由0设为1，那么最后一个if语句可以改成成：`n = n - (x >> 31)`，其他语句保持不变。

右移位按位与版本，二分搜索计算前导0

```java
// 二分搜索计算前导0 int numberOfLeadingZeros(int x) { if (x == 0) return 32; int n = 1; if ((x >> 16) == 0) { n = n + 16; x = x << 16; } if ((x >> 24) == 0) { n = n + 8; x = x << 8; } if ((x >> 28) == 0) { n = n + 4; x = x << 4; } if ((x >> 30) == 0) { n = n + 2; x = x << 2; } n = n - (x >> 31); return n; }
```

如果将n初值设为32，通过减法，上面的算法可以改写成：

```java
// 二分搜索计算前导0，做减法 int numberOfLeadingZeros(int x) { if (x == 0) return 32; int n = 32; int y; y = x >>> 16; if (y != 0) { n -= 16; x = y; } // 验证高16位（bit31 ~ 16）是否为0，如果是，就不用从总计数n - 16；如果不是，就需要 n - 16，因为该区域之前已经算作是0 了 y = x >>> 8; if (y != 0) { n -= 8; x = y; } // 验证bit31 ~ 8 是否为0，此时bit31 ~ 16 必为0 y = x >>> 4; if (y != 0) { n -= 4; x = y; } // 验证bit31 ~ 4 是否为0，此时bit31 ~ 8 必为0 y = x >>> 2; if (y != 0) { n -= 2; x = y; } // 验证bit31 ~ 2 是否为0，此时bit31 ~ 2必为0 y = x >>> 1; if (y != 0) return n - 2; // 验证bit31 ~ 1是否为0，当y = x >>> 1且y ≠ 0 时，说明bit1非0，也就是说n多计了bit1和bit0，需要减去 return n - x; }
```

改写成循环形式

```java
int nunberOfLeadingZeros(int x) { int y; int n = 32; int c = 16; do { y = x >>> c; if ( y != 0) { n -= c; x = y; } c = c >> 1; }while( c != 0); return n - x; }
```

### 应用 & JDK源码

在解析Java源码Integer.toHexString过程中， 碰到了这个函数Integer.numberOfLeadingZeros(int)，用于计算前导0的个数。  
见[Java源码解析：Integer.toHexString](https://www.cnblogs.com/fortunely/p/14150969.html)

查看其源码，

```undefined
public static int numberOfLeadingZeros(int i) { // HD, Figure 5-6 if (i == 0) return 32; int n = 1; if (i >>> 16 == 0) { n += 16; i <<= 16; } if (i >>> 24 == 0) { n += 8; i <<= 8; } if (i >>> 28 == 0) { n += 4; i <<= 4; } if (i >>> 30 == 0) { n += 2; i <<= 2; } n -= i >>> 31; return n; }
```

## 后缀0计数

### 二分搜索算法

同前导0计数，可以通过构造后缀0数目的掩码，写出后缀0计数的二分搜索算法。

核心思想： 如果右半部分为0，则在左半部继续查找，累计后缀0个数 + 右半部位数；否则，继续在右半部的右半部继续查找，累计后缀0个数不变。

```java
int numberOfTailingZeros(int x) { if (x == 0) return 32; int n = 1; if ((x & 0x0000FFFF) == 0) { n += 16; x = x >>> 16; } if ((x & 0x000000FF) == 0) { n += 8； x = x >>> 8; } if ((x & 0x0000000F) == 0) { n += 4; x = x >>> 4; } if ((x & 0x00000003) == 0) { n += 2; x = x >>> 2; } /* 到这里x最多无符号右移了30位，也就是说，只有原始的2个bit位，还没有判断，不过此时作为最新x的bit1、0。 x = x1 x0 (二进制表示形式，x0, x1 = 0, 1) 例如，如果此时x = 0b01，那么最终后缀0的个数是 n - 1, 因为n初值是1，所以需要减去1。 如果此时x = 0b10，那么最终后缀0的个数是 n。 综上，后缀0个数最终值为 n - (x & 1) */ return n - (x & 1); // 也可以 if (x & 0x00000001) != 0) { n = n-1; } return n; }
```

### 应用 & JDK源码

位于Integer.java中的Integer.numberOfTrailingZeros(int)，用于计算后缀0个数，其源码：

```java
public static int numberOfTrailingZeros(int i) { // HD, Figure 5-14 int y; if (i == 0) return 32; int n = 31; y = i <<16; if (y != 0) { n = n -16; i = y; } y = i << 8; if (y != 0) { n = n - 8; i = y; } y = i << 4; if (y != 0) { n = n - 4; i = y; } y = i << 2; if (y != 0) { n = n - 2; i = y; } return n - ((i << 1) >>> 31); }
```

### 参考

1.  《算法心得 高效算法的奥秘(第二版》
