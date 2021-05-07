# [剑指 Offer 68 - I. 二叉搜索树的最近公共祖先](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-zui-jin-gong-gong-zu-xian-lcof/)

<img src="pic\image-20210505144548831.png" alt="image-20210505144548831" style="zoom:67%;" />

二叉树的题目几乎都可以用递归 or 迭代的方法做。

审题后发现：是在二叉搜索树中，找到指定的两个结点的最近公共结点。而二叉搜索树的特点：根结点的左孩子的val均小于根节点；根节点的右孩子均大于根节点。

所以对于两个结点：只有3种情况：

- 结点A<root，在左子树，B>root，在右子树（或者两个反一下）——那么最近公共节点就是根节点
- 结点A<root，B<root，均在左子树，那么递归去查左子树
- 结点A>root，B>root，均在右子树，那么递归去查右子树

## 递归

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
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if(root.val < p.val && root.val < q.val) return lowestCommonAncestor(root.right, p, q);
        else if(root.val > p.val && root.val > q.val) return lowestCommonAncestor(root.left, p, q);
        else return root;
    }
}
```

——递归可以实现很优雅的代码

## 迭代

总体思路和递归一致，因为最多只需要访问一侧的子树（要么访问一侧子树，要么直接返回了）

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
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        while(root != null){		// 包含了树为空的情况和结点找不到的情况
            if(p.val < root.val && q.val < root.val) root = root.left;
            else if(p.val > root.val && q.val > root.val) root = root.right;
            else break;
        }
        return root;
    }
}
```

——其实也很优雅



# [剑指 Offer 68 - II. 二叉树的最近公共祖先](https://leetcode-cn.com/problems/er-cha-shu-de-zui-jin-gong-gong-zu-xian-lcof/)

<img src="pic\image-20210505152155392.png" alt="image-20210505152155392" style="zoom:67%;" />

<img src="pic\image-20210505152207563.png" alt="image-20210505152207563" style="zoom:67%;" />

就是在上面一题上，将二叉搜索树修改成了普通的二叉树，那么不能利用二叉搜索树的特性进行快速查找了。

## 笨蛋解法

第一次做的时候，使用了笨蛋解法，现在做还是这个笨蛋解法:weary:毫无进步

应该是之前没有看题解的巧妙解法所以还是用了这个方法。

总体思路：分别获得结点p和q的路径（包含自身），然后从头开始比较，找到第一个不一样的结点，那么它前一个结点就是最深匹配的结点了

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
    List<TreeNode>rigth_path;
    private void pathFind(TreeNode root, TreeNode target, List<TreeNode>path){	// 找路径
        if(root == null) return;
        if(root == target){             // 找到了
            path.add(root);
            rigth_path = new ArrayList<>(path);
            path.remove(path.size() - 1);
            return;
        }                           // 两个递归终止条件
        path.add(root);
        pathFind(root.left, target, path);
        pathFind(root.right, target, path);
        path.remove(path.size() - 1);
    }
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        pathFind(root, p, new ArrayList<>());
        List<TreeNode>p_path = new ArrayList<>(rigth_path);	// 保存路径（否则就会被覆盖掉）
        pathFind(root,q, new ArrayList<>());
        List<TreeNode>q_path = new ArrayList<>(rigth_path);
        int i = 0;
        while(i < p_path.size() && i < q_path.size()){
            if(p_path.get(i) != q_path.get(i)) break;
            i++;
        }
        return p_path.get(i - 1);
    }
}
```

## 巧妙解法

它根据题目限定，只有两个结点，那么结点的位置只有两种可能：

- 同在左子树
- 一个在做一个在右
- 同在右子树

然后我们在dfs遍历的时候，如果找到该结点了，那么就返回该结点；如果直到null都没有找到就返回null。

- 如果根节点发现左右结点的返回值都为null，表示该root结点的树中不存在任何一个结点，那么也返回null

- 如果左右结点只有一个结点返回非null，那么返回该非null结点

- 如果左右结点均返回非null，那么在该root结点的树中，则返回root，则其他子树均返回了null

  （隐含的，该root结点在返回到父结点时，另外一个结点返回一定为null，所以解就是该root结点）

——特殊的，如果p和q存在祖先关系，那么只要最先一个找到了，则不会再遍历下去，这样还是符合条件的

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
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if(root == null || root == p || root == q) return root;
        TreeNode left = lowestCommonAncestor(root.left, p, q);
        TreeNode right = lowestCommonAncestor(root.right, p, q);
        if(left == null && right == null) return null;	// 该root的树中不存在任何一个结点
        else if(left == null) return right;
        else if(right == null) return left;
        else return root;    
    }
}
```

——这个方法太优雅了，主要就是靠返回值来获得祖先结点，要多学学！