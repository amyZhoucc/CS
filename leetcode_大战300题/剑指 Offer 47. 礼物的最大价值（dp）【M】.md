# [剑指 Offer 47. 礼物的最大价值](https://leetcode-cn.com/problems/li-wu-de-zui-da-jie-zhi-lcof/)

<img src="pic\image-20210506155442248.png" alt="image-20210506155442248" style="zoom:67%;" />

很显然的用DP，但是要有技巧，需要维护一个memory，可以申请一个m*n的辅助数组，来存储每个结点的最大礼物值，但是空间还可以节省。

分析题意：每个结点(i, j)只能从[i - 1, j]、[i, j-1]中的较大值获得，而[i-1, j]的是从[i-2, j]和[i-1, i -1]获得。所以可以得出递推方程式：$dp(i, j) = max(dp(i-1, j), dp(i, j-1))$，然后[i, 0]时，只能从[i-1,0]获得。

然后我们可以发现：[i, j]的值只可能从它前一个和上一个获得，所以我们只需要维护两个数组：prev数组和cur数组，然后就能根据prev来更新cur数组

```java
class Solution {
    public int maxValue(int[][] grid) {
        if(grid.length == 0) return 0;          // 处理特殊情况
        int row = grid.length, col = grid[0].length;
        int[] prev = new int[col];          // 存储前一行的最大值，初始全部为0
        for(int i = 0; i < row; i++){
            int[] temp = new int[col];      // 当前行
            temp[0] = prev[0] + grid[i][0];     // 处理当前行的第一个元素（只能是上面的加上自己）
            for(int j = 1; j < col; j++){
                temp[j] = Math.max(temp[j - 1], prev[j]) + grid[i][j];
            }
            prev = temp;
        }
        return prev[col - 1];
    }
}
```

