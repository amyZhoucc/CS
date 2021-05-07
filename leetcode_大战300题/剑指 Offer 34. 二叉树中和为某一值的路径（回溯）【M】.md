# [剑指 Offer 34. 二叉树中和为某一值的路径](https://leetcode-cn.com/problems/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-jing-lcof/)

<img src="pic\image-20210504230248230.png" alt="image-20210504230248230" style="zoom: 67%;" />

很显然，需要用到回溯法，找到一条适合的路径：从根节点开始到某个叶子结点的一条路。

题目很简单，但是存在的问题是：如果按照常规的回溯法，结果添加是发生在递归终止条件时，即return之前。但是递归终止条件是root=null，那么一个叶子结点有2个null结点，那么会出现重复添加的情况。——**所以，不能发生在null判断时**

那么就在非null结点中额外增加一个判断：`root != null && root.left == null && root.right == null`——判断是否是叶子结点，如果是叶子结点看当前的剩余值是否==0，如果是将该路径加入队列；如果不是叶子结点/是叶子结点但是剩余值!=0，就跳过不添加

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    private void dfs(TreeNode root, List<List<Integer>>res, List<Integer>list, int target){
        if(root == null) return;		// 递归终止条件

        list.add(root.val);
        int want = target - root.val;
        // 找到叶子结点了
        if(root != null && root.left == null && root.right == null && want == 0){
            res.add(new ArrayList<>(list));
        }
        dfs(root.left, res, list, want);
        dfs(root.right, res, list, want);
        list.remove(list.size() - 1);
    }
    public List<List<Integer>> pathSum(TreeNode root, int target) {
        List<List<Integer>> res = new ArrayList<>();
        if(root == null) return res;
        dfs(root, res, new ArrayList<Integer>(), target);
        return res;
    }
}
```

——这题估计之前第一遍的时候是会做的，结果现在做不会做了，还是说明本题没有完全掌握。