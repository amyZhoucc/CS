# [剑指 Offer 07. 重建二叉树](https://leetcode-cn.com/problems/zhong-jian-er-cha-shu-lcof/)

<img src="pic\image-20210502132122363.png" alt="image-20210502132122363" style="zoom:67%;" />

遇到的第一个稍微有点意思的题目。这一题比较经典，如果在比较紧张的情况下，很有可能不能马上完全写对。

二叉树的前中后序遍历之间存在关系。

前序遍历，一定是根节点开头，后面紧跟左子树，最后右子树；而中序遍历，一定是先左子树、根、右子树。那么可以**从前序遍历中知道根节点，然后在中序遍历中找到该根节点，从而将中序遍历划分成左右子树，那么中序遍历就可以划分了，并且计算出左子树的个数，那么前序遍历的左右子树也可以进行划分了**，之后在划分好的左子树、右子树中继续上述步骤——将大的数组划分成小数组，然后递归求解

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
    private int find(int[] inorder, int target){	// 由于限定了里面出现的数字不会重复，直接遍历找即可
        for(int i = 0; i < inorder.length; i++){
            if(inorder[i] == target) return i;
        }
        return -1;		// 一般不会到这一步
    }
    // dfs来创建二叉树
    private TreeNode build(int[] preorder, int[] inorder, int pre_l, int pre_r, int in_l, int in_r){
        if(pre_l > pre_r) return null;	// 递归终止条件
        TreeNode head = new TreeNode(preorder[pre_l]);	// pre数组的当前第一个pre_l就是该子树的根结点
        int mid = find(inorder, preorder[pre_l]); // 找到根节点在中序遍历的位置，根节点在中序遍历时将左右子树分开了
        int left_length = mid - in_l + pre_l;  // 左子树个数+pre_l，是前序中左子树的范围，可以将前序遍历划分开来
        // 根节点的位置：前序遍历pre_l, 中序遍历mid
        // 左子树：前序遍历[pre_l+1,left_length]，中序遍历[in_l, mid-1]
        // 右子树：前序遍历[left_length+1, pre_r, mid+1, in_r]
        head.left = build(preorder, inorder, pre_l + 1, left_length,in_l, mid - 1);
        head.right = build(preorder, inorder, left_length + 1, pre_r, mid + 1, in_r);
        return head;

    }
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        TreeNode head = build(preorder, inorder, 0, preorder.length - 1, 0, inorder.length - 1);
        return head;
    }
}
```

**关键点：`pre_l,pre_r,in_l,in_r`这个4个边界需要写对**，这里面的最关键的是pre_r要写对。其余的应该不存在问题——慢慢推应该能写出来。

——注意，该解法只能用于每个结点值均是唯一的。

# 扩展：

二叉树的下一个结点：

```
给定一棵二叉树和其中的一个结点（TreeNode类型），找出按照中序遍历中该结点的下一个结点？
（该二叉树除了有左右孩子结点的指针外，还有指向父结点的指针）
```

该题目需要分类讨论：（最好就是画一个树，然后看一下如何求解）：

1. 如果该结点存在右子树，那么**右子树的最左孩子**就是next结点

2. 如果该结点不存在右子树，那么需要找到其父结点

   - 如果该节点是其**父结点的左孩子**，那么父结点就是next

   - 如果不是左孩子，那么找父结点的父结点，看父结点是不是父父结点的左孩子，如果是，那么父父结点就是next；如果不是继续向上找

     .....直到找到根节点都没有找到/找到了身为左子树的父结点，就是next

```java
/*
public class TreeLinkNode {
    int val;
    TreeLinkNode left = null;
    TreeLinkNode right = null;
    TreeLinkNode next = null;

    TreeLinkNode(int val) {
        this.val = val;
    }
}
*/
public class Solution {
    public TreeLinkNode GetNext(TreeLinkNode pNode) {
        if(pNode == null) return null;
        TreeLinkNode pNext = null;
        if(pNode.right != null){			// 存在右孩子，那么找右孩子的最左结点
            TreeLinkedNode cur = pNode.right;	// 当前节点
            while(cur.left != null){			// 找到了最左结点
                cur = cur.left;
            }
            pNext = cur;		// 最左结点就是下一个结点
        }
        else if(pNode.next != null){	// pNode是父结点的右孩子，且当前结点存在父结点（非根结点）
            TreeLinkedNode parent = pNode.next;
            TreeLinkedNode cur = pNode;
            while(parent != null && parent.left != cur){	// 存在父结点，且当前节点是父结点的右孩子则一直向上找
                cur = parent;
                parent = cur.next;		// 向上
            }
            pNext = parent;
        }
        return pNext;
    }
}
```

