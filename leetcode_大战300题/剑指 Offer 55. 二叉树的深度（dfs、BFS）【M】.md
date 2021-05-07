# [剑指 Offer 55 - I. 二叉树的深度](https://leetcode-cn.com/problems/er-cha-shu-de-shen-du-lcof/)

<img src="pic\image-20210506191603253.png" alt="image-20210506191603253" style="zoom:67%;" />

不解释，可以用DFS，也可以用BFS，看需要遍历多少层。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public int maxDepth(TreeNode root) {
        if(root == null) return 0;
        return Math.max(maxDepth(root.left), maxDepth(root.right)) + 1;
    }
}
```



# [剑指 Offer 55 - II. 平衡二叉树](https://leetcode-cn.com/problems/ping-heng-er-cha-shu-lcof/)

<img src="pic\image-20210506200204737.png" alt="image-20210506200204737" style="zoom:67%;" />

求二叉树深度的基础上，并且判断该树是否平衡

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    private boolean flag;
    private int dfs(TreeNode root){
        if(flag || root == null) return 0;	// 可以剪枝
        int leftDepth = dfs(root.left);
        int rightDepth = dfs(root.right);
        if(Math.abs(leftDepth - rightDepth) > 1) flag = true;
        return Math.max(leftDepth, rightDepth) + 1;
    }
    public boolean isBalanced(TreeNode root) {
        dfs(root);
        return !flag;
    }
}
```

主要是想了解额外的解法：

利用返回值，来进行剪枝，即如果遍历过程中树一直都是平衡的，则返回值返回的就是普通的树的深度；而如果不平衡了，那么开始返回-1，而如果左右子树只要返回-1，则认为出现不平衡直接返回。

——**利用负数来实现异常判断**

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    private int dfs(TreeNode root){
        if(root == null) return 0;
        int leftDepth = dfs(root.left);
        if(leftDepth == -1) return -1;
        int rightDepth = dfs(root.right);
        if(rightDepth == -1) return -1;
        return Math.abs(leftDepth-rightDepth) > 1 ? -1 : Math.max(leftDepth, rightDepth) + 1;
    }
    public boolean isBalanced(TreeNode root) {
        return dfs(root) != -1;
    }
}
```

参考：

https://leetcode-cn.com/problems/ping-heng-er-cha-shu-lcof/solution/mian-shi-ti-55-ii-ping-heng-er-cha-shu-cong-di-zhi/