# [剑指 Offer 63. 股票的最大利润](https://leetcode-cn.com/problems/gu-piao-de-zui-da-li-run-lcof/)

<img src="pic\image-20210505214807400.png" alt="image-20210505214807400" style="zoom:67%;" />

这个题做了很多次了，第一次独立把它做出来。

主要思想是：动态规划。

**即如果在当天要卖的话，那么最大的利润就是在该天之前的最小值买入。**

所以我们可以维护一个最小值数组，存储的是从第1天开始的截止到当前天的最小值，然后和价格数组相见，最大值就是最大利润。

```java
class Solution {
    public int maxProfit(int[] prices) {
        if(prices.length <= 1) return 0;
        int[] min = new int[prices.length];
        min[0] = prices[0];
        for(int i = 1; i < prices.length; i++){
            if(min[i - 1] > prices[i]) min[i] = prices[i];
            else min[i] = min[i - 1];
        }
        int max = 0;
        for(int i = 0; i < prices.length; i++){
            if(prices[i] - min[i] > max) max = prices[i] - min[i];
        }
        return max;
    }
}
```

当然，可以优化，我们可以边遍历边维护最小值，边维护最大利润：——节省了一个数组的空间

```java
class Solution {
    public int maxProfit(int[] prices) {
        if(prices.length <= 1) return 0;
        int min = prices[0], max = 0;
        for(int i = 1; i < prices.length; i++){
            min = Math.min(min, prices[i]);     // 求价格的最小值
            max = Math.max(max, prices[i] - min);
        }
        return max;
    }
}
```

