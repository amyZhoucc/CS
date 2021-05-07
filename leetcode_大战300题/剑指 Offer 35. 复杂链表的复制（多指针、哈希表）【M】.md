# [剑指 Offer 35. 复杂链表的复制](https://leetcode-cn.com/problems/fu-za-lian-biao-de-fu-zhi-lcof/)

<img src="pic\image-20210505102545227.png" alt="image-20210505102545227" style="zoom:67%;" />

<img src="pic\image-20210505102602350.png" alt="image-20210505102602350" style="zoom:67%;" />

第一次拿到题目的时候，死活想不明白为啥不能直接复制一份普通数组的，然后再将random结点连接上，后面写的时候才发现，我们要找到**random指向的结点是需要去遍历整个链表的**，即我们从原链表中的random结点，找到了指向的结点，但是在新数组中，我们无法定位，所以我们**需要知道该指向的结点在原链表中的相对位置**（相对开头的index/相对结尾的index，注意不能根据val找，可能存在重复val值），然后**根据该相对位置在新链表中重新走一遍**，然后才能找到——该找+重复走的过程O(2N)

所以总体时间复杂度为O(N^2)

所以，暴力法时间复杂度上不过关

## 复制-拆分法

这个方法很巧妙，一般理解并且手写一次之后就能记住整体思路了：

- 在原链表中，每个结点之后都新插入一个副本结点：——副本结点只设置了next结点，random结点均为null

<img src="pic\image-20210505104139312.png" alt="image-20210505104139312" style="zoom:67%;" />

- 然后：更新副本结点的random结点，copy结点的random，就是origin结点的random的next（random指向null的另外处理）

  <img src="pic\image-20210505104506084.png" alt="image-20210505104506084" style="zoom:67%;" />

- 最后，将原链表和副本链表分割：用双指针断开即可

```java
/*
// Definition for a Node.
class Node {
    int val;
    Node next;
    Node random;

    public Node(int val) {
        this.val = val;
        this.next = null;
        this.random = null;
    }
}
*/
class Solution {
    public Node copyRandomList(Node head) {
        if(head == null) return head;
        Node cur = head;
        while(cur != null){            // 遍历链表，然后对每个结点复制一个，紧跟在该结点之后
            Node temp = new Node(cur.val);
            temp.next = cur.next;
            cur.next = temp;
            cur = cur.next.next;
        }
        cur = head;
        while(cur != null){			// 遍历链表，填上random指针
            if(cur.random != null) cur.next.random = cur.random.next;
            else cur.next.random = null;	// 处理random指向null的结点
            cur = cur.next.next;
        }
        Node res = head.next;
        cur = res;
        while(head != null){			// 链表拆分，双指针：head指向原链表；cur指向新链表
            head.next = head.next.next;
            if(head.next != null) cur.next = cur.next.next;	
            else cur.next = null;	// 处理尾结点
            head = head.next;
            cur = cur.next;
        }
        return res;
    }
}
```

——需要注意的是：对于cur.next==null的额外处理，注意画图模拟一下即可。

## 哈希表

这个是比较自然能够想到的解法。

哈希表存储原链表的每个结点，和对应新创建的结点——key、value，然后遍历哈希表，将key对应的next、random对应的value赋值给value对应的next、random，具体看代码

```java
/*
// Definition for a Node.
class Node {
    int val;
    Node next;
    Node random;

    public Node(int val) {
        this.val = val;
        this.next = null;
        this.random = null;
    }
}
*/
class Solution {
    public Node copyRandomList(Node head) {
        if(head == null) return head;			// 处理特殊情况
        HashMap<Node, Node>store = new HashMap<>();	// key-原链表结点，value-新链表结点
        Node cur = head;
        while(cur != null){			// 遍历一遍存储到哈希表中
            Node temp = new Node(cur.val);
            store.put(cur, temp);
            cur = cur.next;
        }
        for(Map.Entry<Node, Node>entry: store.entrySet()){		// 遍历哈希表
            Node orig = entry.getKey();
            Node copy = entry.getValue();
            copy.next = store.get(orig.next);		// 核心代码——next和random的赋值
            copy.random = store.get(orig.random);
        }
        return store.get(head);
    }
}
```

——写的时候存在一个疑问：如果原链表结点的val、next的val、random的val都一样，哈希表是否会认为是一样的——不会，因为Node是一个类，而该类的哈希函数并没有被重写，所以默认比较的是内存中的地址值。