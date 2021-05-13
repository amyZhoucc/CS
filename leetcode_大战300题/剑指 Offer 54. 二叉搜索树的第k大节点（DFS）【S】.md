# [剑指 Offer 54. 二叉搜索树的第k大节点](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-di-kda-jie-dian-lcof/)

<img src="pic\image-20210509152610700.png" alt="image-20210509152610700" style="zoom:67%;" />

这个是变种的top-k问题。

实际上二叉搜索树是有序的，只要按照中序遍历：左-根-右，那么输出的结果就是顺序递增的。但是我们需要获得的是top-k，即最大的值，所以大的数都在右边，即右子树这边，所以我们只要稍微修改一下dfs的顺序就可以先获得top-k。

我觉得本题的核心在于：创建两个实例变量，用来存储返回值、存储当前遍历的计数

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
    private int res;			// 返回值
    private int count;		// 遍历计数
    public void dfs(TreeNode root, int k){
        if(count == k || root == null) return;
        dfs(root.right, k);
        count++;			// 从最右结点返回开始才计数
        if(k == count){			// 找到第k大的数了
            res = root.val;
            return;
        }
        dfs(root.left, k);
    }
    public int kthLargest(TreeNode root, int k) {
        count = 0;
        dfs(root, k);
        return res;
    }
}
```

（如果要暴力解，将这个二叉搜索树按照中序遍历将他们的值存放在arrayList中，然后通过index就可以找到需要的结点）。