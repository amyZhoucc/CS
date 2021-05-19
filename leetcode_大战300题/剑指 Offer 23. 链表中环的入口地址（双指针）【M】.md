# [剑指 Offer 23. 链表中环的入口地址](https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/)

<img src="pic\image-20210504152400886.png" alt="image-20210504152400886" style="zoom:67%;" />

<img src="pic\image-20210504152420515.png" alt="image-20210504152420515" style="zoom:67%;" />

链表中找环，也是一个超级经典的题目。使用的方法就是双指针：快慢指针

可以很自然的想到，如果链表中存在环，那么快指针最终能和慢指针相遇。

- 第一轮：判断是否存在环：用快慢指针，判断是否
- 第二轮：如果有环，那么f指针维持刚才相遇的结点，s指针从头开始，两个指针都一步一步走，相遇的点就是环的入口地址

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
    public ListNode detectCycle(ListNode head) {
        if(head == null || head.next == null) return null;	// 排除特殊情况——只有一个结点，不存在环的情况
        ListNode fast = head, slow = head;
        while(fast != null && fast.next != null){	// 先判断是否存在环
            fast = fast.next.next;
            slow = slow.next;
            if(fast == slow) break;
        }
        if(fast != slow) return null;		// 不是相遇跳出循环的，那么直接返回null，表示不存在环
        else{				// 找入口结点
            slow = head;
            while(fast != slow){
                fast = fast.next;
                slow = slow.next;
            }
            return fast;
        }
    }
}
```

再推一边为啥两次相遇就能找到结点的入口地址：

设，从head到环的入口（不包括环入口），有a个结点；环有b个结点，总体有a+b个结点

- 第一轮相遇时：

  fast比slow每次都多走一步，那么**`fast = 2slow`**

  那么fast能和slow相遇，那么fast一定已经经过至少一轮了，所以fast肯定比slow多走n轮：**`fast = s + nb`**

  所以相遇时：`fast = 2nb, slow = nb`

- 第二轮相遇时：设置fast不动，slow从头开始走，每次都只走一步：

  由于`fast = 2nb`  -> `fast = 2nb + a`;

  `slow = a`——那么正好在环的头相遇了，且只需要走a步即可。

——这个推导还是要掌握的



## ps：

如果要求不能使用额外的内存空间，那么该如何实现呢？

可以破坏链表结构，将遍历到的链表结点指向自己，如果发现结点的next=null，那么说明链表有环；如果发现结点next==自己，那么说明遍历到环的入口结点了

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
    public ListNode detectCycle(ListNode head) {
        if(head == null) return null;
        while(head.next != null && head.next != head){	// 遍历到链表尾，说明无环 or 遍历到自己了，说明存在环
            ListNode temp = head.next;
            head.next = head;
            head = temp;
        }
        if(head.next == null) return null;
        else return head;
    }
}
```

