# [剑指 Offer 29. 顺时针打印矩阵](https://leetcode-cn.com/problems/shun-shi-zhen-da-yin-ju-zhen-lcof/)

<img src="pic\image-20210504171356618.png" alt="image-20210504171356618" style="zoom:67%;" />

这个题没有任何技巧可言，纯粹就是考验你的思路是否清晰，然后对临界情况是否能够判断准确

如果在紧张的情况下，不知道能否正确做出该题而没有bug。

需要四个指针：分别指示最上面未访问的行、最下面未访问到的行，最左边的列、最右边的列

关键是临界条件的判断：

- 如果原矩阵为空，则直接返回空数组
- 每次修改行、列时，需要判断一下，如果已经不满足的条件，直接跳出循环

```java
class Solution {
    public int[] spiralOrder(int[][] matrix) {
        if(matrix.length == 0) return new int[0];
        int row = matrix.length, col = matrix[0].length;
        int[] res = new int[row * col];
        int r_l = 0, r_h = row - 1, c_l = 0, c_r = col - 1;
        int index = 0;
        while(r_l <= r_h && c_l <= c_r){
            for(int i = c_l; i <= c_r; i++) res[index++] = matrix[r_l][i];
            r_l++;
            if(r_l > r_h) break;	// 如果已经不满足条件，对下面的循环无影响（不会进去；但是对下下个循环有影响，会报错）
            for(int i = r_l; i <= r_h; i++) res[index++] = matrix[i][c_r];
            c_r--;
            if(c_r < c_l) break;         
            for(int i = c_r; i >= c_l; i--) res[index++] = matrix[r_h][i];
            r_h--;
            if(r_h < r_l) break;
            for(int i = r_h; i>= r_l; i--) res[index++] = matrix[i][c_l];
            c_l++;
        }
        return res;
    }
}
```

还有取名字的技巧，eg：其实可以取l、r、t、b，又短又方便记忆