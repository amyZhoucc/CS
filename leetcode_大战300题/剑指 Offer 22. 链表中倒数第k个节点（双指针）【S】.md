# [剑指 Offer 22. 链表中倒数第k个节点](https://leetcode-cn.com/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/)

<img src="pic\image-20210504100151440.png" alt="image-20210504100151440" style="zoom:67%;" />

本题很简单，最普通的双指针问题，但是需要处理好fast提前移动的次数，然后配合fast到底的条件

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
    public ListNode getKthFromEnd(ListNode head, int k) {
        ListNode fast = head, slow = head;
        while(k != 0){
            fast = fast.next;
            k--;
        }
        while(fast != null){
            fast = fast.next;
            slow = slow.next;
        }
        return slow;
    }
}
```

