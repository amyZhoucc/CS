# [剑指 Offer 42. 连续子数组的最大和](https://leetcode-cn.com/problems/lian-xu-zi-shu-zu-de-zui-da-he-lcof/)

<img src="pic\image-20210506220110713.png" alt="image-20210506220110713" style="zoom:67%;" />

能用动态规划我是没有想到的（想到最多可以用贪心）

可以从简单的开始考虑：如果不需要是连续的，那么可以选择数组中所有的正数相加就是最大值，但是要求是连续的，且中间可能存在负数，所以可以用动态规划考虑。——**从数组开始到当前结点范围内，求最大和的子数组**。

主要思想：当前的结点可以选择是否加入之前的序列，**如果之前的序列和为负数，那么会越加越小**，eg：之前序列=-5，当前=-2，那么加入=-7，而不加入=-2，所以选择重新开始。

所以递推方程式：**$dp(i) = num[i] + max(dp[i-1], 0)$——注意这个的含义是是包含当前节点，且以当前结点截止的最大和子序列，还需要再维护一个maxValue来记录整个过程中的最大值**

eg：

[-2]：那么自己构成子序列，maxValue=-2

[-2, 1]：prev=-2，为负数，所以会越加越小，则抛弃，从1开始，maxValue=1

[-2, 1, -3]：prev=1，为正数，[1, -3]，maxValue=1

[-2,1,-3,4]：prev=-2，为负数，抛弃，从4开始，maxValue=4

[-2,1,-3,4,-1]：prev=4，[4, -1] ,maxValue=4

[-2,1,-3,4,-1,2]：prev=3，[4,-1,2]，maxValue=5

[-2,1,-3,4,-1,2,1]：prev=5, [4,-1,2,1], maxValue=6

[-2,1,-3,4,-1,2,1,-5]：prev=6, [4,-1,2,1,-5]，maxValue=6 

[-2,1,-3,4,-1,2,1,-5, 4]：prev=1, [4,-1,2,1,-5,4]，maxValue=6

根据递推方程式写出的解法：

```java
class Solution {
    public int maxSubArray(int[] nums) {
        int maxValue = nums[0];
        int[] dp = new int[nums.length];
        dp[0] =  nums[0];
        for(int i = 1; i < nums.length; i++){
            if(dp[i - 1] < 0) dp[i] = nums[i];
            else dp[i] = dp[i - 1] + nums[i];
            maxValue = Math.max(maxValue, dp[i]); 
        }
        return maxValue;
    }
}
```

——相比于下面的更为直观

其实还可以优化：

```java
class Solution {
    public int maxSubArray(int[] nums) {
        int maxValue = nums[0];
        int prev = nums[0];
        for(int i = 1; i < nums.length; i++){
            if(prev < 0) prev = nums[i];
            else prev += nums[i];
            maxValue = Math.max(prev, maxValue);
        }
        return maxValue;
    }
}
```

——本题的dp不是很明显，但是知道用dp之后，推递推方程式还是较为容易的（**该dp方程式和常规的不同，它并不能直接获得最大值，而它的含义是：以当前节点作为子数组的尾节点，看最大和子数组**——所以维护的是包含当前节点，且以当前结点截止的最大和子序列）

参考：

https://leetcode-cn.com/problems/lian-xu-zi-shu-zu-de-zui-da-he-lcof/solution/dong-tai-gui-hua-jie-jue-zui-hao-liao-ji-ede6/