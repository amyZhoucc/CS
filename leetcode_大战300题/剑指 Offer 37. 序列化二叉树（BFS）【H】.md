# [剑指 Offer 37. 序列化二叉树](https://leetcode-cn.com/problems/xu-lie-hua-er-cha-shu-lcof/)

<img src="pic\image-20210510233417414.png" alt="image-20210510233417414" style="zoom:67%;" />

这题其实难度不大，就是对应个树进行遍历，然后根据该遍历序列反推出二叉树。

虽然题目中已经提示用层序遍历，但是为啥用层序遍历呢？（面试时可能会考）

1. 我们可以用前序遍历、中序遍历的组合推出二叉树，具体可以见[第7题](https://github.com/amyZhoucc/CS/blob/main/leetcode_%E5%A4%A7%E6%88%98300%E9%A2%98/%E5%89%91%E6%8C%87%20Offer%2007.%20%E9%87%8D%E5%BB%BA%E4%BA%8C%E5%8F%89%E6%A0%91%EF%BC%88%E5%88%86%E6%B2%BB%EF%BC%89%E3%80%90M%E3%80%91.md)

   但是这样存在的问题是，不能有重复的结点、只有所有数据都读取出来时才能开始反序列化。如果是在流里面的话，那么会要等待很长时间

2. 所以层序遍历，你可以边序列化，边反序列化

序列化：就按照普通的层序遍历即可，但是注意要将null结点也包括进去。所以这次层序遍历不需要判断cur的left、right是否是null，而是都将它们入队，而对于cur为null的结点，那么直接加入序列

但是要注意，如果arrayList.toString，是会自动带有[]，每个元素之间会有空格+逗号，所以在反序列化的时候还需要额外处理。

反序列化时：还是会创建一个队列，存放刚创建的结点，那么对应序列的前两个就是该结点的左右孩子结点。然后更新当前结点的左右孩子，然后将非null的左右孩子再入栈。

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
public class Codec {
    // Encodes a tree to a single string.
    public String serialize(TreeNode root) {
        ArrayList<Integer> store = new ArrayList<>();
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        while(!queue.isEmpty()){
            TreeNode cur = queue.poll();
            if(cur != null){
                store.add(cur.val);
                queue.add(cur.left);		// 不管是否是null，全部入队
                queue.add(cur.right);
            }
            else store.add(null);		// 如果cur==null，那么它直接加入序列中
        }
        return store.toString();
    }

    // Decodes your encoded data to tree.
    public TreeNode deserialize(String data) {
        String[] ss = data.substring(1, data.length() - 1).split(",");	//去掉首尾的[]，然后按照,分割，注意除了第一个结点，其他的结点前面都有空格，需要trim
        Queue<TreeNode>queue = new LinkedList<>();
        TreeNode root = null;
        if(ss[0].equals("null")) return root;	// 第一个结点就是null，说明该树为空
        root = new TreeNode(Integer.parseInt(ss[0]));
        queue.offer(root);
        int i = 1;
        while(!queue.isEmpty() && i < ss.length){
            TreeNode cur = queue.poll();
            if(!ss[i].trim().equals("null")){
                int left = Integer.parseInt(ss[i++].trim());
                cur.left = new TreeNode(left);
                queue.offer(cur.left);
            }
            else i++;
            if(!ss[i].trim().equals("null")){
                int right = Integer.parseInt(ss[i++].trim());
                cur.right = new TreeNode(right);
                queue.offer(cur.right);
            }
            else i++;
        }
        return root;
    }
}

// Your Codec object will be instantiated and called as such:
// Codec codec = new Codec();
// codec.deserialize(codec.serialize(root));
```

