# [剑指 Offer 59 - II. 队列的最大值](https://leetcode-cn.com/problems/dui-lie-de-zui-da-zhi-lcof/)

<img src="pic\image-20210505234145785.png" alt="image-20210505234145785" style="zoom:67%;" />

设计题，这个和前面的**包含min函数的栈**类似，包含max肯定也是要额外维护一个数据结构的。

**包含min函数的栈**维护的是一个单调栈：就是**只有小于等于栈顶的结点才会入栈，大于的不会入栈**；当栈顶元素需要出栈时，单调栈如果有对应，则栈顶元素也会出栈——单调减栈

而本题**包含max函数的队列**维护的是一个单调队列：

- 每个结点均会入队列（从队尾入），维护一个队头最大的情况（类似于单调减栈），即如果要加入的结点比队尾节点要大，则不断从尾部出队，直到将当前结点插入；——维护是按照单调减栈来维护的
- 获取当前队列的最大值——从队头获取
- 如果队头出队时，和单调队列队头一致的，则单调队列对应出队——出队是按照队列习惯来（队头出）

```java
class MaxQueue {
    Queue<Integer>queue;
    Deque<Integer>deque;
    public MaxQueue() {
        queue = new LinkedList<Integer>();
        deque = new LinkedList<Integer>();
    }
    
    public int max_value() {
        if(queue.size() == 0) return -1;
        return deque.peekFirst();
    }
    
    public void push_back(int value) {
        queue.offer(value);
        while(!deque.isEmpty() && deque.peekLast() < value){
            deque.removeLast();
        }
        deque.addLast(value);
    }
    
    public int pop_front() {
        if(queue.size() == 0) return -1;
        int res = queue.poll();
        if(res == deque.peekFirst()) deque.removeFirst();
        return res;
    }
}

/**
 * Your MaxQueue object will be instantiated and called as such:
 * MaxQueue obj = new MaxQueue();
 * int param_1 = obj.max_value();
 * obj.push_back(value);
 * int param_3 = obj.pop_front();
 */
```

——目前来说：遇到的包含min函数的栈、包含max函数的队列都可以考虑用单调栈解决：

栈：如果要包含min函数的栈，可以用单调减栈（并且如果大于栈顶的结点不入栈）；如果要包含max函数的栈，可以用单调增栈（如果小于栈顶的结点不入栈）

队列：如果要包含min函数的队列，可以用单调增双向队列，插入是从队列尾部插入（每个入队列的结点都会加入到队列中，如果队尾不符合要求就会从尾部循环出队），如果对应的队头也是单调增的双向队列的队头，则也需要对应出队；如果要包含max函数的队列，可以用单调减双向队列