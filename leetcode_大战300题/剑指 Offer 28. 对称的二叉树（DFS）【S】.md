# [剑指 Offer 28. 对称的二叉树](https://leetcode-cn.com/problems/dui-cheng-de-er-cha-shu-lcof/)

<img src="pic\image-20210504163446202.png" alt="image-20210504163446202" style="zoom:67%;" />

这个也算是一个经典的树的递归题。

根结点肯定是对称的，只需要看其左右子树即可，而左右子树的话，先看根结点值是否一样，一样才有比下去的意义。

如果不一样，则直接返回；如果一样，则左子树的左孩子需要和右子树的右孩子进行比较，左子树的右孩子和右子树的左孩子进行比较。

和第26一样，看结点是否一样有4种情况：

- A == null && B == null：返回true
- A != null && B == null：返回false
- A == null && B != null：返回false
- A != null && B != null：看值，如果不相等，直接返回；如果相等，继续比较下去

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
    private boolean dfs(TreeNode A, TreeNode B){
        if(A == null && B == null) return true;
        else if(A == null || B == null) return false;		// 只要存在一个为null，就是不满足（对比offer26）
        return (A.val == B.val) && dfs(A.left, B.right) && dfs(A.right, B.left);
    }
    public boolean isSymmetric(TreeNode root) {
        if(root == null) return true;
        return dfs(root.left, root.right);
    }
}
```

——要学会将判断条件优化，这样能够节省代码