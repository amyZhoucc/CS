# [重排链表](https://www.nowcoder.com/practice/3d281dc0b3704347846a110bf561ef6b?tpId=117&&tqId=37712&rp=1&ru=/activity/oj&qru=/ta/job-code-high/question-ranking)

网易面试真题，总体难度我觉得不大，就是要综合运用多个题的知识——如果是第一次接触到的，是需要考虑一会时间

**还有，zcc不要再看错题了！！！**——这都看错几次了！！！

<img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20210901101253407.png" alt="image-20210901101253407" style="zoom:50%;" />

<img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20210901101306467.png" alt="image-20210901101306467" style="zoom:50%;" />

就是链表的插入，就是在原来顺序上：Ln插入到L1和L2之间，Ln-1插入到L2和L3之间....

分析之后可以发现：就是将链表的后半部分逆序插入到前半部分

后半部分：如果链表是奇数长度，那么是中点之后，不包括中点；如果是偶数长度，那么是中点及其之后。

所以主要思路是：

1. 找到链表的中点：**快慢指针**

   需要注意的，快慢指针一般都是找到的中点，即如果是奇数长度，那么找到的就是中点；如果是偶数长度，找到的是后半部分的中点

   `fast != null && fast.next != null`

   所以对于奇数，还需要向后移动一个

   ——此时，可以用一个prev存放slow的前一个，那么`prev.next = null`

2. 翻转后半部分链表：翻转链表——用递归，之间将后半部分翻转，slow为起点

3. 前半部分和后半部分插空：合并链表，就是简单的用链表的next...

```java
/**
 * Definition for singly-linked list.
 * class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    // 翻转链表_将后半部分进行翻转
    private ListNode reverse(ListNode head){			// 初始的head=slow
        if(head == null || head.next == null) return head;
        ListNode res = reverse(head.next);
        head.next.next = head;
        head.next = null;
        return res;
    }
    public void reorderList(ListNode head) {
        if(head == null || head.next == null || head.next.next == null) return;    // 边界情况
        
        ListNode fast = head, slow = head, prev = slow;
        while(fast != null && fast.next != null){		// 快慢指针
            fast = fast.next.next;
            prev = slow;
            slow = slow.next;
        }
        // 如果是偶数个，那么fast最后一定指向null，此时slow就是指向了链表后半部分的起点
        // 如果是奇数个，那么fast最后一定之前null的前一个，此时slow指向了链表的中点，还需要后移一个，指向后半部分的起点
        if(fast != null){		
            prev = slow;
            slow = slow.next;        // 指向后半部分的起点
        }
        prev.next = null;		// 前后链表断掉——避免出现循环
        ListNode right = reverse(slow);		// 翻转后半部分链表
        ListNode left = head;
        while(left != null && right != null){
            ListNode temp = left.next;			// 保存临时的值
            ListNode temp2 = right.next;
            left.next = right;			// 画图即可得出
            right.next = temp;
            left = temp;
            right = temp2;
        }
    }
}
```

——本题是一个很好的对多个链表做题技巧的综合，**用到了快慢指针、翻转链表、简单的链表合并操作**