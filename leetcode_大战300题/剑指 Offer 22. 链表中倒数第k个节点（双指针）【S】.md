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



## 扩展

前面限定了传递的k一定是符合规范的，那么如果k==0，k超过了链表的长度，那么要如何考虑呢：

```java
import java.util.*;

/*
 * public class ListNode {
 *   int val;
 *   ListNode next = null;
 *   public ListNode(int val) {
 *     this.val = val;
 *   }
 * }
 */

public class Solution {
    /**
     * 代码中的类名、方法名、参数名已经指定，请勿修改，直接返回方法规定的值即可
     *
     * 
     * @param pHead ListNode类 
     * @param k int整型 
     * @return ListNode类
     */
    public ListNode FindKthToTail (ListNode pHead, int k) {
        // write code here
        ListNode fast = pHead;
        while(k > 0 && fast != null){		// fast!=null，防止k越界后，导致报错
            fast = fast.next;
            k--;
        }
        if(fast == null && k != 0) return null;	//如果遍历到尾部，仍没有找到倒数k个，那么直接返回null
        // 存在fast==null，且k==0的情况，那么其实找的是链表头结点
        while(fast != null){
            fast = fast.next;
            pHead = pHead.next;
        }
        return pHead;
    }
}
```

