## 盛最多水的容器

暂存代码

```c++
class Solution {
public:
    int maxArea(vector<int>& height) {
        int size=height.size();
        int maxA = 0;

        for(int i=0;i<size;i++){
            int maxH=0;
            for(int j=size-1;j>=0;j--){
                if(height[j]>maxH){
                    maxH = height[j];
                    int tempA = (j-i) * min(height[i],height[j]) ;
                     maxA = max(tempA,maxA);
                }

            }
        }

        return maxA;
    }
};
```

暂存代码二

```
class Solution {
public:
    int maxArea(vector<int>& height) {
        int size=height.size();
        int maxA = 0;

        for(int i=0,j=size-1;i<size;i++){
           // int minH = min(height[i],height[j])
            int tempA = abs(j-i) * min(height[i],height[j])  ;
          cout<< tempA<<endl;
            maxA = max(tempA,maxA);
            while(j>i && height[i]>= height[j]){
                j--;     
            }   
        }

        return maxA;
    }
};


```



## leetcode题库【双指针】盛最多水的容器

### 原题链接

[盛最多水的容器](https://leetcode.cn/problems/container-with-most-water/)

> 没看题解，全部自己调试，耗时2h。

![image-20230922113612875](https://raw.githubusercontent.com/zhaodong462502/markdownImage/master/blogingimage-20230922113612875.png)



### AC代码

```
class Solution {
public:
    int maxArea(vector<int>& height) {
        int size=height.size();
        int maxA = 0;
        int i=0,j=size-1;
        while(i<size && j>=0 && j>i){
            int tempA = abs(j-i) * min(height[i],height[j])  ;
           // cout<< tempA<<endl;
            maxA = max(tempA,maxA);

             if(height[i]>= height[j]){
                 j--;
             }else{
                 i++;
             }
        }

        return maxA;
    }
};
```

