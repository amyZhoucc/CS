# [剑指 Offer 31. 栈的压入、弹出序列](https://leetcode-cn.com/problems/zhan-de-ya-ru-dan-chu-xu-lie-lcof/)

<img src="pic\image-20210504221847120.png" alt="image-20210504221847120" style="zoom:67%;" />

这题比较有意思，之前做过，并没有回忆起来（估计当时只是看了一下题解，然后认为理解了，就复现了一下）

基本解题思路：模拟。

创建一个栈来模拟整个入栈和出栈的过程。

思路很简单：不断的压栈，然后遇到一个栈顶=popped的顶的数字，就出栈，直到栈顶不符合popped，或者栈为空；然后继续入栈，直到压栈的数组为空。

但是，自己实现起来还是有点难度：

- 看popped数组，实际上就是每次的栈顶数组
- 我们是需要将pushed的内容全部入栈的，所以可以**以pushed这个数组作为一个大循环**；然后，该数组入栈后，每次入栈都判断现在栈顶是否是popped需要的栈顶了，如果不是，继续入栈（直到循环结束）；如果**是，那么需要将栈中的内容不断pop，直到栈顶不匹配了——里面有一个while小循环**，用来出栈，然后继续入栈操作

——最后判断栈是否为空

```java
class Solution {
    public boolean validateStackSequences(int[] pushed, int[] popped) {
        int j = 0, len = pushed.length;
        LinkedList<Integer>store = new LinkedList<>();
        for(int i = 0; i < len; i++){
            store.push(pushed[i]);
            while(j <= len && !store.isEmpty() && popped[j] == store.peek()){
                store.pop();
                j++;
            }
        }
        return store.isEmpty();
    }
}
```

——发现，对于给定的序列模拟熟练一些；但是，对于判断是否不符合还是不会做（后面的判断是否是二叉搜索树的后续遍历序列，也是和这个一样，判断是否是合理序列）