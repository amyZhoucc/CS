# [剑指 Offer 59 - I. 滑动窗口的最大值](https://leetcode-cn.com/problems/hua-dong-chuang-kou-de-zui-da-zhi-lcof/)

<img src="pic\image-20210512180048277.png" alt="image-20210512180048277" style="zoom:67%;" />

其实这题不是典型的滑动窗口题，人家都限定了滑动窗口，且该滑动窗口大小是给定的，所以其本质上是一个考察单调队列的题。

类似于下一题，都是维护一个双向队列，该队列是满足**单调减的**，且队列中元素的个数不会超过窗口k的大小。

- 对于窗口右侧是新加入的结点：
  - 首先会和队尾的节点进行比较，如果队尾结点>=num[i]，那么直接插入队尾即可；
  - 如果队尾结点<num[i]，那么不断从队尾出队，直到找到>=num[i]的队尾结点，然后将该num[i]插入到队尾；
- 对于窗口左侧，除去的结点，如果该结点和队头结点的值一样，那么队头需要出队

​	在实现的时候，我们还需要模拟窗口移动，分为两步：

1. 窗口还未形成的时候，那么遍历0~k-1，然后将结点按照单调栈的规则加入到队列中
2. 窗口形成了，那么先移动左边窗口，将i-k先移出去，然后移动右边窗口，将i移进来，然后查看队列能够获得i-k+1为左窗口头的最大值。

```java
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        if(nums.length == 0 || k == 0) return new int[0];		// 处理特殊情况
        int[] res = new int[nums.length - k + 1];		
        Deque<Integer>queue = new LinkedList<>();		// 单调队列
        for(int i = 0; i < k; i++){
            while(!queue.isEmpty() && queue.peekLast() < nums[i]) queue.pollLast();
            queue.offerLast(nums[i]);
        }
        res[0] = queue.peekFirst();
        for(int i = k; i < nums.length; i++){
            if(nums[i - k] == queue.peekFirst()) queue.pollFirst();
            while(!queue.isEmpty() && queue.peekLast() < nums[i]) queue.pollLast();
            queue.offerLast(nums[i]);
            res[i - k + 1] = queue.peekFirst();
        }
        return res;
    }
}
```

——注意特殊情况的处理：

如果k>len，那么直接返回；k=0，那么也要直接返回；

我在牛客上面的解法：

```java
import java.util.*;
public class Solution {
    public ArrayList<Integer> maxInWindows(int [] num, int size) {
        Deque<Integer>queue = new LinkedList<>();        // 双向链表，模拟单调减队列
        ArrayList<Integer>res = new ArrayList<>();        // 返回值
        int len = num.length;
        if(len < size || size == 0) return res;        // 处理特殊情况
        for(int i = 0; i < size; i++){        // 先构建出滑动窗口，将前size个值都遍历一遍，放入队列中
            while(!queue.isEmpty() && queue.peekLast() < num[i]) queue.removeLast();
            queue.addLast(num[i]);
        }
        for(int i = 0; i < len - size; i++){
            int temp = queue.peekFirst();
            res.add(temp);
            if(num[i] == temp) queue.removeFirst();        // 左滑窗向右移动
            // 右滑窗向右移动
            while(!queue.isEmpty() && queue.peekLast() < num[i + size]) queue.removeLast();
            queue.addLast(num[i + size]);
        }
        res.add(queue.peekFirst());
        return res;
    }
}
```

