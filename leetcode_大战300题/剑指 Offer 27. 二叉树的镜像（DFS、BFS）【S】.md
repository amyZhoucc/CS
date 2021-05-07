# [剑指 Offer 27. 二叉树的镜像](https://leetcode-cn.com/problems/er-cha-shu-de-jing-xiang-lcof/)

<img src="pic\image-20210504160641160.png" alt="image-20210504160641160" style="zoom:67%;" />

本质上就是将左右互换了。

## DFS

主要思想是，当前结点左右结点互换的时候，该左右结点为根节点的树已经镜像。

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
    public TreeNode mirrorTree(TreeNode root) {
        if(root == null) return null;		// 递归终止条件
        TreeNode temp = root.left;		// 需要暂存左子树
        root.left = mirrorTree(root.right);
        root.right = mirrorTree(temp);
        return root;
    }
}
```

## BFS

思路很简单：就是在BFS遍历的基础上，将每个节点的左右子结点互换即可。

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
    public TreeNode mirrorTree(TreeNode root) {
        if(root == null) return root;       // 排除特殊情况
        Queue<TreeNode>queue = new LinkedList<>();
        queue.offer(root);
        while(!queue.isEmpty()){
            TreeNode cur = queue.poll();
            TreeNode left = cur.left, right = cur.right;
            cur.left = right;		// 左右结点互换
            cur.right = left;
            if(cur.left != null) queue.offer(cur.left);	// 后面都是一样的（互换之后还是按照原来的顺序遍历下一层）
            if(cur.right != null) queue.offer(cur.right);
        }
        return root;
    }
}
```

