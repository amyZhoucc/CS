# [剑指 Offer 36. 二叉搜索树与双向链表](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/)

<img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20210506230410980.png" alt="image-20210506230410980" style="zoom:67%;" />

<img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20210506230442747.png" alt="image-20210506230442747" style="zoom:67%;" />

题目要求我们原地修改二叉搜索树，将其修改为双向链表，链表顺序就是二叉搜索树的中序遍历。其中left指向前驱结点，right指向后继结点，且链表头的left结点指向链表尾，链表尾的right结点指向链表头，从而构成循环双向链表。

很显然，我们需要用到DFS，且是基于中序遍历的即：左、根、右，然后在根这边进行操作。

此外我们需要保存链表头结点head、前驱结点prev

前序遍历，找到的第一个非null结点，一定是最左结点，即链表头

```JAVA
/*
// Definition for a Node.
class Node {
    public int val;
    public Node left;
    public Node right;

    public Node() {}

    public Node(int _val) {
        val = _val;
    }

    public Node(int _val,Node _left,Node _right) {
        val = _val;
        left = _left;
        right = _right;
    }
};
*/
class Solution {
    Node prev, head;    // 分别表示链表的前驱结点和头结点
    private void dfs(Node root){
        if(root == null) return;
        dfs(root.left);
        if(prev == null){		// 如果prev还未赋值，那么就是第一次执行到这边，就是树的最左结点，那么就是链表头
            head = root;
        }
        else{
            prev.right = root;		// prev的后继和当前结点的前驱均可以更新了
            root.left = prev;
        }
        prev = root;		// 更新prev结点
        dfs(root.right);
        
    }
    public Node treeToDoublyList(Node root) {
        dfs(root);
        if(prev != null && head != null){	// 将首尾连接起来
            prev.right = head;
            head.left = prev;
        }
        return head;
    }
}
```

