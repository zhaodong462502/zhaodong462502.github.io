- [ ] 区间问题和贪心章节的区间选点、区间最大不相交、区间分组、区间覆盖归纳总结

## sort函数

### 用法

Sort(start,end)

> 排序范围 [start,end)，左闭右开。

### struct 

和普通数组类似，排序0，n-1范围的元素，写成sort(range,range+n);

### Vector 

begin()函数，返回的是首元素的迭代器

end()函数，返回的是末尾元素的下一个位置的迭代器

排序vector，写成sort(vector.begin(),vector.end());



## c++操作符重载

```c++
struct Range{
	int l,r;
  const bool operator <(const Range &w) const{
    return l<w.l;
  } 
}range[N];

```

### 三个const的含义

#### 第一个const

返回值是const，不允许对该值直接赋值，此处小于号重载用不到，加号的重载可能用到

#### 第二个const

表示扩大入参范围，入参除了是Range 还可以是 const 修饰的Range

#### 第三个const

表示const 修饰的Range可以调用此方法





