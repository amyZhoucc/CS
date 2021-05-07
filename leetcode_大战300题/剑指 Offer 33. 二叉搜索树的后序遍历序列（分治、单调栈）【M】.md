# [剑指 Offer 33. 二叉搜索树的后序遍历序列](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-hou-xu-bian-li-xu-lie-lcof/)

<img src="pic\image-20210504234601709.png" alt="image-20210504234601709" style="zoom:67%;" />



这题总体来说难度不大，总想着有简单的方法，所以纠结了很久。

主要思想：分治

## 分治

首先：分析一下后序遍历，那么**最后一个结点肯定是根节点**，而该二叉树又是二叉搜索树，**如果序列正确的话，可以根据最后一个结点找到前面第一个大于该值的结点，然后将序列一分为二**，然后子序列继续按照上面的操作——这样可以直接构建出二叉搜索树的结构。

而现在的问题是：看该序列是否能够构建出二叉搜索树

所以还需要增加一个性质：**二叉搜索树的左子树所有结点一定都小于根节点，右子树所有结点一定都大于根节点**，所以上面划分出左右子树后，遍历右子树序列，看是否有小于root的结点，如果有直接返回false。

```java
class Solution {
    private int mid(int[] postorder, int val, int left, int right){  // 找到以val划分的中线
        int i = left;
        for(; i <= right; i++){
            if(postorder[i] > val) return i;    // 找到第一个大于val的值，就是中线
        }
        return i;		// 如果没有找到，说明根节点最大，那么是只有左子树的树
    }
    private boolean dfs(int[]postorder, int left, int right){
        if(left >= right) return true;          // 只有一个结点，那么返回true
        int root = postorder[right];    // 最后一个结点就是根节点
        int index = mid(postorder, root, left, right - 1);	// 在[left, right-1]范围内找分界，如果找不到，那么就返回index
        for(int i = index; i < right; i++){		// 如果存在右子树，去看右子树是否有小于root的结点
            if(postorder[i] < root) return false;
        }
        // 如果序列正常，则dfs其左右孩子结点
        return dfs(postorder, left, index - 1) && dfs(postorder, index, right - 1);
    }
    public boolean verifyPostorder(int[] postorder) {
        return dfs(postorder, 0, postorder.length - 1);
    }
}
```

易错的点：

- 找分界的时候，如果没有找到，说明是一个只有左子树的树，那么返回根节点的index
- 递归终止条件：**`left >= right`**，left==right表示只有一个结点，所以直接返回true；left>right，该树只有左子树，那么右子树的范围变成了[right, right-1]，所以需要合起来处理。

## 单调栈

维护一个单调递增的栈。

后序遍历的逆序：根-右-左，那么根之后就是连续大于根的结点，然后都将其压入栈中。如果遇到小于栈顶的结点（表明是某个根节点的左结点，且是**当前栈中最接近于它的且大于该值的结点**），那么就将当前栈顶循环出栈，并且更新root。

——模拟一下吧

```java
class Solution {
    public boolean verifyPostorder(int[] postorder) {
        LinkedList<Integer>stack = new LinkedList<>();
        int root = Integer.MAX_VALUE;
        for(int i = postorder.length - 1; i >= 0; i--){
            if(root < postorder[i]) return false;
            while(!stack.isEmpty() && postorder[i] < stack.peek()){
                root = stack.pop();
            }
            stack.push(postorder[i]);
        }
        return true;
    }
}
```

参考：https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-hou-xu-bian-li-xu-lie-lcof/solution/mian-shi-ti-33-er-cha-sou-suo-shu-de-hou-xu-bian-6/

——第二种方法还不是完全理解，需要再看看。