# [剑指 Offer 66. 构建乘积数组](https://leetcode-cn.com/problems/gou-jian-cheng-ji-shu-zu-lcof/)

<img src="pic\image-20210505162411422.png" alt="image-20210505162411422" style="zoom:67%;" />

本来可以直接for循环一下，将总乘积计算出来，然后每次除去本身即可，但是**不能使用除法**。

所以需要额外的思路：

根据题目发现：是除了本身以外的所有值均相乘，那么就是`[0, i-1] *[i+1, n-1]`，所以需要计算出左半边的值，和右半边的值。

然后发现规律：

i = 0：无左半边，[1, n -1]

i = 1：[0, 0] , [2, n-1]

i = 2：[0, 1], [3, n-1]

....

i = n - 2：[0, n - 3], [n - 1, n - 1]

i = n - 1：[0, n - 2], 无右半边

——所以我们可以构建一个矩阵：

|      | 1     | 2     | 3     | 4     | 5     |
| ---- | ----- | ----- | ----- | ----- | ----- |
| 1    | **1** |       |       |       |       |
| 2    | 1     | **1** | 3     | 4     | 5     |
| 3    | 1     | 2     | **1** | 4     | 5     |
| 4    | 1     | 2     | 3     | **1** | 5     |
| 5    | 1     | 2     | 3     | 4     | **1** |

我们可以先算下三角，求出所有的左半边值——可以根据前面的值乘上前一个数即可

然后再计算上三角，求出右半边的值

然后将左半边和右半边乘上，就是结果。

```java
class Solution {
    public int[] constructArr(int[] a) {
        if(a.length == 0) return new int[0];
        int[] b = new int[a.length];
        b[0] = 1;
        for(int i = 1; i < b.length; i++){		// 先算下三角（左半边的解）——存储在结果数组中
            b[i] = b[i - 1] * a[i - 1];
        }
        int temp = 1;	
        for(int i = a.length - 2; i >= 0; i--){	// 再计算上三角（右半边的解）——存储在temp中，然后直接和b相乘
            temp *= a[i + 1];
            b[i] *= temp;
        }
        return b;
    }
}
```

——稍微注意一下边界情况。

参考：

https://leetcode-cn.com/problems/gou-jian-cheng-ji-shu-zu-lcof/solution/mian-shi-ti-66-gou-jian-cheng-ji-shu-zu-biao-ge-fe/