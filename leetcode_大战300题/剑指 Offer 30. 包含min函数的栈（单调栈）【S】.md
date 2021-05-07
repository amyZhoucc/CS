# [剑指 Offer 30. 包含min函数的栈](https://leetcode-cn.com/problems/bao-han-minhan-shu-de-zhan-lcof/)

<img src="pic\image-20210504210442501.png" alt="image-20210504210442501" style="zoom:67%;" />

这个题之前做过，但是用啥数据结构完全没印象（之前没有整理），所以用了很多辅助数据结构才实现

时间复杂度并不能达到O(1)，大概在O(NlogN)左右——堆排序的时间复杂度

## 笨蛋解法

主要就是用最小堆和哈希表，维持最小的堆顶，并且由于存入的数可能会出现重复，而删除又是不确定的，所以用哈希表存储出现的次数。

- 当压栈时，如果该结点出现过，就直接val++；如果结点没有出现过，存入到哈希表中，并且加入到堆中
- 当出栈时，如果该结点减少到1了，那么需要将其从哈希表和堆中删除；否则，就更新val

```java
class MinStack {
    private HashMap<Integer, Integer>hashMap;	// 哈希表存储，每个结点的出现次数
    private LinkedList<Integer>stack;		// 存储栈的
    private PriorityQueue<Integer>queue;		// 最小堆，栈顶就是最小值
    /** initialize your data structure here. */
    public MinStack() {
        hashMap = new HashMap<Integer, Integer>();
        stack = new LinkedList<Integer>();
        queue = new PriorityQueue<Integer>();
    }
    public void push(int x) {
        stack.push(x);
        if(hashMap.containsKey(x)){
            hashMap.put(x, hashMap.get(x) + 1);
        }
        else{
            hashMap.put(x, 1);
            queue.add(x);
        }
    }
    public void pop() {
        int res = stack.pop();
        if(hashMap.get(res) == 1){
            hashMap.remove(res);
            queue.remove(res);
        }
        else{
            hashMap.put(res, hashMap.get(res) - 1);
        }
    }
    public int top() {
        return stack.peek();
    }
    public int min() {
        return queue.peek();
    }
}

/**
 * Your MinStack object will be instantiated and called as such:
 * MinStack obj = new MinStack();
 * obj.push(x);
 * obj.pop();
 * int param_3 = obj.top();
 * int param_4 = obj.min();
 */
```

## 单调栈解法

单调减栈，就是维持一个单调递减的栈，栈顶就是当前的最小值。

原理，如果新入栈的元素大于单调栈栈顶元素，那么该结点没有必要入，因为即使它存在，也不会是最小值；如果是小于栈顶元素，那么需要入栈——所以，就维护了当前栈结构下的最小栈。

```java
class MinStack {
    private LinkedList<Integer>stack;
    private LinkedList<Integer>singleStack;
    /** initialize your data structure here. */
    public MinStack() {
        stack = new LinkedList<>();
        singleStack = new LinkedList<>();
    }

    public void push(int x) {
        stack.push(x);
        if(singleStack.isEmpty()) singleStack.push(x);        // 如果单调栈为空，那么将结点插入
        else{       // 栈不为空，那么需要比较再确定是否插入
            if(singleStack.peek() >= x) singleStack.push(x);
        }
    }

    public void pop() {
        int res = stack.pop();
        if(res == singleStack.peek()) singleStack.pop();
    }

    public int top() {
        return stack.peek();
    }

    public int min() {
        return singleStack.peek();
    }
}

/**
    * Your MinStack object will be instantiated and called as such:
    * MinStack obj = new MinStack();
    * obj.push(x);
    * obj.pop();
    * int param_3 = obj.top();
    * int param_4 = obj.min();
    */
```

参考：

https://leetcode-cn.com/problems/bao-han-minhan-shu-de-zhan-lcof/solution/mian-shi-ti-30-bao-han-minhan-shu-de-zhan-fu-zhu-z/（tql）

——这个解法真的有必要记一下，单调栈单调栈单调栈！