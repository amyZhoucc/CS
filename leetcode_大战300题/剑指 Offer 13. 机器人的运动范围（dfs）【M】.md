# [剑指 Offer 13. 机器人的运动范围](https://leetcode-cn.com/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof/)

<img src="pic\image-20210502211009685.png" alt="image-20210502211009685" style="zoom:67%;" />

整体思路本题同之前的12题，但是并不需要回溯，只需要简单的dfs即可

```java
class Solution {
    private int[][] help = new int[][]{{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
    private int count = 0;
    private boolean overflow(int i, int j, int k){
        int count = 0;
        count += i / 10 + i % 10;
        count += j / 10 + j % 10;
        // System.out.println(count);
        return count > k;

    }
    private void dfs(int m, int n, int k, int i, int j, boolean[][] flags){
        // 越界，超过限定大小，已经遍历过
        if(i < 0 || i >= m || j < 0 || j >= n || overflow(i, j, k) || flags[i][j]) return;
        
        count++;
        flags[i][j] = true;
        for(int l = 0; l < 4; l++){
            dfs(m, n, k, i + help[l][0], j + help[l][1], flags);
        }
    }
    public int movingCount(int m, int n, int k) {
        dfs(m, n, k, 0, 0, new boolean[m][n]);
        return count;
    }
}
```

很显然，bfs也是可以的。

可以从(0, 0)开始逐步扩展，然后将符合要求的结点均放入一个队列中，然后每次都从中取出一个结点，看其周围的结点还是否符合要求，如果有则继续加入。**直到队列为空。**

```java
class Solution {
    private int[][] help = new int[][]{{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
    private boolean rangeCheck(int i, int j, int k){
        int count = i / 10 + i % 10;
        count += j / 10 + j % 10;
        return count <= k;
    }
    public int movingCount(int m, int n, int k) {
        int num = 0;
        boolean[][] flag = new boolean[m][n];
        Queue<int[]>store = new LinkedList<>();	// 可以存放数组
        store.offer(new int[]{0, 0});		// 初始存入(0,0)
        flag[0][0] = true;
        num++;
        while(!store.isEmpty()){
            int[] cur = store.poll();
            for(int l = 0; l < 4; l++){
                int i = cur[0] + help[l][0], j = cur[1] + help[l][1];
                if(i >= 0 && i < m && j >= 0 && j < n && rangeCheck(i, j, k) && !flag[i][j]){
                    store.offer(new int[]{i, j});
                    flag[i][j] = true;
                    num++;
                }
            }
        }
        return num;
    }
}
```

