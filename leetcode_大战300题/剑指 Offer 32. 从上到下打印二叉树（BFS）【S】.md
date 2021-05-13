# [剑指 Offer 32 - I. 从上到下打印二叉树](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/)

<img src="pic\image-20210504214621415.png" alt="image-20210504214621415" style="zoom: 67%;" />

层序遍历，没有说的必要（连面试都不会让你写的简单题）

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
    public int[] levelOrder(TreeNode root) {
        if(root == null) return new int[0];
        ArrayList<Integer>value = new ArrayList<>();
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        while(!queue.isEmpty()){
            TreeNode cur = queue.poll();
            value.add(cur.val);
            if(cur.left != null) queue.offer(cur.left);
            if(cur.right != null) queue.offer(cur.right);
        }
        int[] res = new int[value.size()];
        for(int i = 0; i < res.length; i++){
            res[i] = value.get(i);
        }
        return res;
    }
}
```

# [剑指 Offer 32 - II. 从上到下打印二叉树 II](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-ii-lcof/)

<img src="pic\image-20210504214608241.png" alt="image-20210504214608241" style="zoom:67%;" />

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
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> res = new ArrayList<>();
        if(root == null) return res;
        Queue<TreeNode> queue = new LinkedList<TreeNode>();
        queue.offer(root);
        while(!queue.isEmpty()){
            int size = queue.size();
            ArrayList<Integer> temp = new ArrayList<>();
            while(size > 0){
                TreeNode cur = queue.poll();
                temp.add(cur.val);
                if(cur.left != null) queue.offer(cur.left);
                if(cur.right != null) queue.offer(cur.right);
                size--;
            }
            res.add(temp);
        }
        return res;
    }
}
```

# [剑指 Offer 32 - III. 从上到下打印二叉树 III](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-iii-lcof/)

<img src="pic\image-20210504231510805.png" alt="image-20210504231510805" style="zoom:67%;" />

可以利用双端队列的优势，在奇数行将值头插；在偶数行将值尾插，其余不变

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
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> res = new LinkedList<>();
        if(root == null) return res;
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        while(!queue.isEmpty()){
            int size = queue.size();
            LinkedList<Integer>temp = new LinkedList<>();
            while(size > 0){
                TreeNode cur = queue.poll();
                if(res.size() % 2 == 0) temp.addLast(cur.val);	// 偶数行，尾插
                else temp.addFirst(cur.val);	// 奇数行，头插
                if(cur.left != null) queue.offer(cur.left);
                if(cur.right != null) queue.offer(cur.right);
                size--;
            }
            res.add(temp);
        }
        return res;
    }
}
```

当然，也可以都按照正序来，但是对于奇数行，每次结果出之后都reverse一下。

——这种题要是做错了，都对不起自己

由于看面经说面试官不满意上面的解法，说太暴力了，再实现一个解法。

用两个栈实现：odd栈存放的是奇数行，even栈存放的是偶数行：偶数行遍历的时候就是偶数栈出栈，然后按照左、右的顺序入奇数栈；奇数行遍历的时候就是奇数栈出栈，然后按照右、左的顺序入偶数栈

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
    public List<List<Integer>> levelOrder(TreeNode root) {
        if(root == null) return new ArrayList<>();
        List<List<Integer>> res = new ArrayList<>();
        Stack<TreeNode>even = new Stack<>();
        Stack<TreeNode>odd = new Stack<>();
        even.push(root);
        while(!even.isEmpty() || !odd.isEmpty()){
            Stack<TreeNode>stack, stack2;
            if(res.size() % 2 == 0){
                stack = even;
                stack2 = odd;
            }
            else{
                stack = odd;
                stack2 = even;
            }
            ArrayList<Integer>temp = new ArrayList<>();
            while(!stack.isEmpty()){
                TreeNode cur = stack.pop();
                temp.add(cur.val);
                if(res.size() % 2 != 0){
                    if(cur.right != null) stack2.push(cur.right);
                    if(cur.left != null) stack2.push(cur.left);
                }
                else{
                    if(cur.left != null) stack2.push(cur.left);
                    if(cur.right != null) stack2.push(cur.right);
                }
            }
            res.add(temp);
        }
        return res;
    }
}
```

