# [剑指 Offer 12. 矩阵中的路径](https://leetcode-cn.com/problems/ju-zhen-zhong-de-lu-jing-lcof/)

<img src="pic\image-20210502184607767.png" alt="image-20210502184607767" style="zoom:67%;" />

<img src="pic\image-20210502184620825.png" alt="image-20210502184620825" style="zoom:67%;" />

这题并不难，需要注意测试用例有一个大的例子，可能会存在越界，所以需要进行剪枝

思路：

1. 遍历矩阵，找到第一个结点

```java
class Solution {
    // 向四个方向查找的辅助数组
    private int[][] help = new int[][]{{-1, 0}, {1, 0}, {0, -1}, {0, 1}};;
    private boolean isExist = false;	// 全局判断是否已经找到符合的路径了——找到了就可以直接返回，不需要找其他路径了
    // 参数：矩阵，矩阵每个点是否访问标记，当前结点的横纵坐标（已经找到的匹配），当前结点对应的字符串下标（即将要找匹配），字符串
    private void backtrace(char[][] board, boolean[][]flags, int row, int col, int cur, String word){
        if(cur == word.length()){	// 已经找到字符串尾部了，表示所有均匹配，可以返回了
            isExist = true;
            return;
        }
        for(int i = 0; i < 4; i++){
            if(isExist) return;		// 如果已经成功找到，可以直接返回了
            int j = row + help[i][0];	// row的上下左右点的横纵坐标
            int k = col + help[i][1];
            if(j >= 0 && j < board.length && k >= 0 && k < board[0].length){	// 不能越界
                // 如果匹配，且未访问过
                if(board[j][k] == word.charAt(cur) && flags[j][k] == false){
                    flags[j][k] = true;	// 设置为访问过
                    backtrace(board, flags, j, k, cur + 1, word);	// 递归
                    flags[j][k] = false;	// 回溯，清除为未访问过
                }
            }
        }
    }
    public boolean exist(char[][] board, String word) {
        boolean[][] flags = new boolean[board.length][board[0].length];
        for(int i = 0; i < board.length; i++){
            for(int j = 0; j < board[0].length; j++){
                if(board[i][j] == word.charAt(0)){  // 需要先找到起始的点
                    flags[i][j] = true;		// 该结点也需要标记为访问过
                    backtrace(board, flags, i, j, 1, word);
                    flags[i][j] = false;
                    if(isExist) return true;
                }
            }
        }
        return false;
    }
}
```

做这种递归的题目时，我一般都是确定一个大致的思路，想好dfs需要哪些核心的数据，然后开始写，在这个过程中需要补哪些数据就给补上，最后才完善出一个完整的dfs传参，因为参数比较多，不可能马上就想到所有的传参。



参考：

https://leetcode-cn.com/problems/ju-zhen-zhong-de-lu-jing-lcof/solution/mian-shi-ti-12-ju-zhen-zhong-de-lu-jing-shen-du-yo/

他的解法更优：我按照理解重新实现了一下：

1. 不需要额外的flags数组，直接修改board即可
2. 不需要全局变量`isExist`，而是将dfs的方法设置为有返回值的

```java
class Solution {
    private boolean dfs(char[][] board, String word, int i, int j, int cur){
        if(cur == word.length()){		// 说明已经遍历到字符串尾部了，已经匹配完毕了
            return true;
        }
        if(i < 0 || i >= board.length || j < 0 || j >= board[0].length || 
           board[i][j] != word.charAt(cur)) return false;	// 越界的，值不等的直接返回
        // 到这一步，说明i、j均时合法的，且值相等
        board[i][j] = '\0';		// 防止重复访问
        // 只要有一个匹配就不会再匹配下去，直接返回（也是剪枝的一部分）
        boolean res = dfs(board, word, i + 1, j, cur + 1) || dfs(board, word, i - 1, j, cur + 1) || 
                      dfs(board, word, i, j + 1, cur + 1) || dfs(board, word, i, j - 1, cur + 1);
        board[i][j] = word.charAt(cur);	// 恢复原来的值
        return res;

    }
    public boolean exist(char[][] board, String word) {
        for(int i = 0; i < board.length; i++){
            for(int j = 0; j < board[0].length; j++){
                boolean res = dfs(board, word, i, j, 0);
                if(res) return true;
            }
        }
        return false;
    }
}
```

