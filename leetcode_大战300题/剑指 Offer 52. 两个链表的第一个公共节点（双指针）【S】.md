# [剑指 Offer 52. 两个链表的第一个公共节点](https://leetcode-cn.com/problems/liang-ge-lian-biao-de-di-yi-ge-gong-gong-jie-dian-lcof/)

<img src="pic\image-20210505231814060.png" alt="image-20210505231814060" style="zoom:67%;" />

<img src="pic\image-20210505231827353.png" alt="image-20210505231827353" style="zoom:67%;" />

<img src="pic\image-20210505231840635.png" alt="image-20210505231840635" style="zoom:67%;" />

类似于链表找环，肯定可以用双指针求解，但是我就是想不到

设链表A长度为a，链表B长度为b，公共长度部分为c

那么：先遍历A，后走B走到交界处：a+(b-c)

​			先遍历B，后走A走到交界处：b+(a-c)

——两者相等了，那么可以设置两个指针A、B，同时从各自的链表开始走，每次都走一步，先遍历自己再走对方，等到指向同一个结点，那么就是交点

- 如果两者的(a -c) == (b - c)，那么还未完全遍历就相遇了
- 如果两者没有交点，a+b = b+a，则最终都指向了null，则符合结果要求
- 如果两者有交点，则返回一定是交点

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        ListNode A = headA, B = headB;
        while(A != B){
           A = A == null ? headB : A.next;
           B = B == null ? headA : B.next;
        }
        return A;
    }
}
```

参考：

https://leetcode-cn.com/problems/liang-ge-lian-biao-de-di-yi-ge-gong-gong-jie-dian-lcof/solution/jian-zhi-offer-52-liang-ge-lian-biao-de-gcruu/

我的解法：先判断两者是否存在交点——即分别遍历，看最后一个非null结点是否一样，如果一样则存在交点，否则不存在。然后A的尾巴指向B的开头——构成一个环，然后就是求题23链表中环的入口地址

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        if (headA == null || headB == null) return null;
        ListNode a = headA;
        while(a.next != null) a = a.next;
        a.next = headB;
        ListNode fast = headA, slow = headA;
        // 下面就是链表中找环的方法
        while(true){
            if(fast == null || fast.next == null) {
                a.next = null;
                return null;			// 如果没有环，则直接恢复原状，并返回
            }
            fast = fast.next.next;
            slow = slow.next;
            if(fast == slow) break;		// 相遇了
        }
        fast = headA;
        while(fast != slow){		// 找到交点
            fast = fast.next;
            slow = slow.next;
        }
        a.next = null;		// 最后恢复原链表
        return fast;
    }
}
```

