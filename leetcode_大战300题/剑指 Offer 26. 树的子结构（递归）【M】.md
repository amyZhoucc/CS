# [剑指 Offer 26. 树的子结构](https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/)

<img src="pic\image-20210504144356207.png" alt="image-20210504144356207" style="zoom:67%;" />

这个题目也是很有意思的，感觉是之前做过，现在还存着一点印象，所以能够想到解法。

整体思路是用两个DFS方法，在递归的时候，如果满足条件，顺便去执行另一个递归程序。——这是难得遇到的一个**需要两个递归的方法**

- 第一个递归程序，基于给定的方法，主要是参数正好，所以可以直接使用

  该方法主要是用来匹配B的根节点的，只有找到B根节点匹配的，才能开始子树匹配。且A当前结点不匹配，则递归找其左右孩子继续匹配，只要有一个返回true即可了

- 第二个递归程序，参数同前面，但是逻辑不一样

  主要是用来匹配整棵树的

  首先需要匹配当前结点，只有当前结点相等，才去递归匹配当前结点的左右子树；如果当前结点不相等，可以直接返回

  当前结点存在5种情况：

  - A == null && B == null：说明遍历到头了，返回true

  - A != null && B == null：说明B到头了，返回true

  - A != null && B == null：返回false

  - A != null && B != null：需要看A的val和B的val是否一致

    ——一致，就继续遍历下去；不一致，就返回

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
    private boolean dfs(TreeNode A, TreeNode B){    // 表示在匹配子树了
        if(B == null) return true;		// B匹配到头了，那么返回true
        else if(A == null) return false;   // A到头了，但是B没有，那么返回false
        // 如果B和A都不为null，那么看是否相等，不相等就直接返回false，相等则继续匹配左右子节点——只有全部返回true才能为true
        return A.val == B.val ? dfs(A.left, B.left) && dfs(A.right, B.right) : false;
    }
    public boolean isSubStructure(TreeNode A, TreeNode B) {	// 匹配B的根节点，如果匹配
        if(A == null || B == null) return false;	// 递归终止条件，顺便处理特殊情况
        if(A.val == B.val && dfs(A, B)){  // 如果根节点相等，向下匹配；如果匹配了为false，那么继续找左右子结点继续匹配
            return true;
        }
        else return isSubStructure(A.left, B) || isSubStructure(A.right, B);
    }
}
```

