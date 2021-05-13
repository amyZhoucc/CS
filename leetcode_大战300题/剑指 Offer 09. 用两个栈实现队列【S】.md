# [剑指 Offer 09. 用两个栈实现队列](https://leetcode-cn.com/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/)

<img src="pic\image-20210502143823308.png" alt="image-20210502143823308" style="zoom:67%;" />

这个也是一个典型题，用栈模拟队列

栈特点：后进先出

队列特点：先进先出

那么可以将入栈的元素出栈放到另外一个栈中，那么顺序自然就反过来了。

所以，我们可以让一个栈接收输入，然后一个栈接收输出，**只有在输出栈为空的时候，才会将输入栈的内容全部放到输出栈中**

```java
class CQueue {
    Stack<Integer> stack_in;
    Stack<Integer> stack_out;
    public CQueue() {
        stack_in = new Stack<Integer>();
        stack_out = new Stack<Integer>();
    }
    
    public void appendTail(int value) {
        stack_in.push(value);
    }
    
    public int deleteHead() {
        if(stack_in.size() == 0 && stack_out.size() == 0) return -1;
        if(stack_out.size() == 0){				// 输出栈为空时，将输入栈的内容放到输出栈中，然后又可以pop了
            while(stack_in.size() != 0){
                stack_out.push(stack_in.pop());
            }
        }
        return stack_out.pop();
    }
}

/**
 * Your CQueue object will be instantiated and called as such:
 * CQueue obj = new CQueue();
 * obj.appendTail(value);
 * int param_2 = obj.deleteHead();
 */
```

# 扩展

## [225. 用队列实现栈](https://leetcode-cn.com/problems/implement-stack-using-queues/)

<img src="pic\image-20210509195832753.png" alt="image-20210509195832753" style="zoom:67%;" />

<img src="pic\image-20210509195848968.png" alt="image-20210509195848968" style="zoom:67%;" />

不要局限在上面的栈模拟队列中，因为栈存在着负负得正的特点——就是先进后出，然后全部pop出去之后，之前的栈顶就在栈底了，恰好符合队列的特性。

而队列不管你翻来覆去几次，都还是一样的存储顺序，所以和上面的基本思想就不一样了。

这边的思路是：**每次入队的时候，都将除了新入队的结点外都出队然后再入队，那么原来的队尾（新入队的结点）就变成了队头结点**，每次都这样做，那么整个队列就是一个逆序的，最早入队的就一直在队尾了，就满足了栈的特点

```java
class MyStack {
    Queue<Integer>store;
    /** Initialize your data structure here. */
    public MyStack() {
        store = new LinkedList<Integer>();
    }
    
    /** Push element x onto stack. */
    public void push(int x) {
        int cur_size = store.size();
        store.offer(x);
        for(int i = 0; i < cur_size; i++){		// 核心——出了刚入队的结点外都出队再入队
            store.offer(store.poll());
        }
    }
    
    /** Removes the element on top of the stack and returns that element. */
    public int pop() {
        return store.poll();
    }
    
    /** Get the top element. */
    public int top() {
        return store.peek();
    }
    
    /** Returns whether the stack is empty. */
    public boolean empty() {
        return store.size() == 0? true : false;
    }
}

/**
 * Your MyStack object will be instantiated and called as such:
 * MyStack obj = new MyStack();
 * obj.push(x);
 * int param_2 = obj.pop();
 * int param_3 = obj.top();
 * boolean param_4 = obj.empty();
 */
```

时间复杂度：入栈操作：O(N)，出栈操作和其他操作都是O(1)，所以均摊起来就是O(1)