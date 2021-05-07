# [剑指 Offer 06. 从尾到头打印链表](https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/)

三刷：还是直觉想用递归做，没有想到更简单的栈思想

<img src="pic\image-20210502102926084.png" alt="image-20210502102926084" style="zoom:67%;" />

链表题两种方法：递归、迭代

迭代的话：我们遍历肯定需要从头到尾的，但是输出是尾巴先输出，头最后——典型的：**后进先出**——用栈（Stack，本质上就是LinkedList）

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public int[] reversePrint(ListNode head) {
        Stack<Integer>stack = new Stack<>();
        while(head != null){
            stack.push(head.val);		// 压栈
            head = head.next;
        }
        int[]res = new int[stack.size()];
        for(int i = 0; i < res.length; i++){
            res[i] = stack.pop();		// 出栈
        }
        return res;
    }
}
```

递归的话：每次访问该结点的时候，都递归输出后面的节点，再输出自身，那么结果就是反的

但是由于本题的返回值是数组，所以用递归时还需要放到一个数组里面（那么还需要事先知道数组的长度，提前遍历，然后每次递归还需要确定该结点在数组中的位置 or 先存放在可变数组ArrayList中，然后转换成数组）——所以，不能很好的发现优势

如果是需要直接print的话：更加直观

```
// 伪代码
{
    if(head == null) return;		// 递归终止条件
    dfs(head.next);		// 先去找后面的结点——当前结点的后面结点会先输出
    print(head.val);	// 返回之后，然后将当前值输出
}
```

符合本题的解法：

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution { 
    private void dfs(ListNode head, int[] res, int count){
        if(head == null) return;
        dfs(head.next, res, count - 1);		// 先去访问该结点的后面结点
        res[count] = head.val;		// 在访问当前结点
    }
    public int[] reversePrint(ListNode head) {
        ListNode cur = head;
        int count = 0;
        while(cur != null){		// 先遍历一遍，统计链表长度
            count++;
            cur = cur.next;
        }
        int[] res = new int[count];
        dfs(head, res, count-1);		// 当前结点，结果数组，当前结点在数组中的位置
        return res;
    }
}
```

递归存在的一个问题：链表十分长时，递归深度很深容易溢出——本质上还是栈，所以迭代更优（递归主要看思想）