# [剑指 Offer 24. 反转链表](https://leetcode-cn.com/problems/fan-zhuan-lian-biao-lcof/)

<img src="pic\image-20210504103622352.png" alt="image-20210504103622352" style="zoom: 67%;" />

一个可以说是刷了n遍的题目。需要拿到题目，马上写出解。

## 递归

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
    public ListNode reverseList(ListNode head) {
        if(head == null || head.next == null) return head;	// 第一个判断是处理特殊情况；第二个是递归的终止条件
        ListNode res = reverseList(head.next);
        head.next.next = head;		// 是前一个结点来重新指向后一个结点的next的
        head.next = null;		// 主要是针对原来的head的结点的指向问题
        return res;			// 返回的一直都是最新的头结点
    }
}
```

## 迭代

麻烦一点，但是容易想到，也要能快速写出来，注意不要弄混和写错

双指针，准确来说是三指针：prev、cur、next

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
    public ListNode reverseList(ListNode head) {
        if(head == null || head.next == null) return head;      // 处理特殊情况
        ListNode prev = head, cur = head.next;
        prev.next = null;		// 头结点指向要提前处理了
        while(cur != null){
            ListNode next = cur.next;
            cur.next = prev;
            prev = cur;
            cur = next;
        }
        return prev;
    }
}
```

