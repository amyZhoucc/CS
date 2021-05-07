# [剑指 Offer 25. 合并两个排序的链表](https://leetcode-cn.com/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof/)

<img src="pic\image-20210504132719307.png" alt="image-20210504132719307" style="zoom:67%;" />

这题其实很简单，类似于合并两个有序的数组。

只不过数组是创建了一个新的数组进行合并（当然也可以给定一个数组足够的空间，然后将另一个数组合并过来（为了减少的数组的移动次数，用从后向前的移动方法））

这边的话：可以选择设置一个dummy结点，而我是直接增加了判断：看head结点是l1的head 还是l2的head，然后就可以开始后面的操作了。

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
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if(l1 == null) return l2;
        else if(l2 == null) return l1;      // 处理特殊情况
        ListNode head, cur;
        if(l1.val <= l2.val){
            head = l1;
            l1 = l1.next;		// 如果l1的head是新的头，那么下面遍历的时候l1应该指向head.next
        }
        else{
            head = l2;
            l2 = l2.next;
        }						// 确定新链表的头结点
        cur = head;
        while(l1 != null && l2 != null){	
            if(l1.val <= l2.val){
                cur.next = l1;
                l1 = l1.next;
            }
            else{
                cur.next = l2;
                l2 = l2.next;
            }
            cur = cur.next;
        }
        cur.next = l1 != null? l1 : l2;	// 剩下的直接链上去即可，如果两者同时结束，那么也可以完美收尾
        return head;
    }
}
```

——要会用三目运算符简化判断`cur.next = l1 != null? l1 : l2;`