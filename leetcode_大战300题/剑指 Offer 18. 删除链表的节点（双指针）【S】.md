# [剑指 Offer 18. 删除链表的节点](https://leetcode-cn.com/problems/shan-chu-lian-biao-de-jie-dian-lcof/)

<img src="pic\image-20210503181608744.png" alt="image-20210503181608744" style="zoom:67%;" />

这题很简单，但是需要注意删除头结点的问题，所以我选择增加一个dummy结点.

本质上是使用双指针：一个指针遍历，指向当前遍历的结点，另一个指针指向当前节点的前一个结点。当找到目标结点时，就能根据prev.next = cur.next，从而将cur删除了

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
    public ListNode deleteNode(ListNode head, int val) {
        ListNode dummy = new ListNode(0);
        dummy.next = head;
        ListNode cur = head;
        head = dummy;
        while(cur.val != val){
            head = cur;
            cur = cur.next;
        }
        head.next = cur.next;
        return dummy.next;
    }
}
```

## 其他方法

不需要使用双指针，

**可以找到该结点i之后，将后一个结点的内容复制给i，然后将后一个结点删除。**

```
ListNode cur;  // 目标结点
cur.val = cur.next.val;		// 将内容复制
cur.next = cur.next.next;	// 将后一个结点删除
```

——那么头结点删除的问题也包含在内了。

也还是需要处理特殊情况：

- 如果cur结点是最后一个结点，那么需要重新遍历一遍，找到倒数第二个结点，然后将该结点的next指向为null
- 但是，如果有且只有一个结点，且该结点也需删除，那么整个链表为空——提前判断`if(head.next = null && head.val == val) return null;`

如果是一个完整的题目，还需要处理临界情况：如果val不在链表内，那么遍历到尾部都没有找到，那么上面的while循环需要再加条件`while(cur != null && cur.val != val)`，当遍历到尾部就跳出了，然后链表保持原样返回。 

# 扩展

删除有序链表中，重复的结点

那么用到双指针，prev、cur

```java
while (cur != null){
    if(cur.val == prev.val){
        prev.next = cur.next;
        cur = prev.next;
    }
    else {
        prev = cur;
        cur = cur.next;
    }
}
```

