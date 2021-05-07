# [剑指 Offer 62. 圆圈中最后剩下的数字](https://leetcode-cn.com/problems/yuan-quan-zhong-zui-hou-sheng-xia-de-shu-zi-lcof/)

<img src="pic\image-20210505212356360.png" alt="image-20210505212356360" style="zoom:67%;" />

这个题目是典型的约瑟夫环问题。

我们只关注最后存活下来的每次在删除中位于的位置：它活下来，则在最后它位于数组的第0个位置；

最后一次删除（即只有两个结点），那么index = (res + m) %2（res = 0），那么就是它最后一轮删除的时的位置

倒数第二次删除（即只有三个结点），那么index=（res + m）% 3（res = 上一次的index）

....

```java
class Solution {
    public int lastRemaining(int n, int m) {
        int res = 0;
        for(int i = 2; i <= n; i++){
            res = (res + m) % i;
        }
        return res;
    }
}
```

——记住结论，从结果开始逆着推它的最初位置。