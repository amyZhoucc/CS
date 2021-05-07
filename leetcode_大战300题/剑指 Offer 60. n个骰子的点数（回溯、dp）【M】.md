# [剑指 Offer 60. n个骰子的点数](https://leetcode-cn.com/problems/nge-tou-zi-de-dian-shu-lcof/)

<img src="pic\image-20210505192502772.png" alt="image-20210505192502772" style="zoom:67%;" />

很自然想到全排列的问题，但是递归深度是个问题，且由于不要列出所有可能，所以是大材小用的，时间复杂度太大

n再大一点就不行了

## 回溯

其实就是存在大量的重复计算的问题：

```java
class Solution {
    private void dfs(int n, int cur, double[] res, int sum){
        if(cur == n){
            res[sum - n]++;
            return;
        }        // 深度已经达到了
        for(int i = 1; i <= 6; i++){
            dfs(n, cur + 1, res, sum + i);
        }
    }
    public double[] dicesProbability(int n) {
        double[] res = new double[n * 5 + 1];
        dfs(n, 0, res, 0);
        double base = Math.pow(1/(double)6, n);
        for(int i = 0; i < res.length; i++){
            res[i] *= base;
        }
        return res;
    }
}
```

模拟计算点数 4 和 点数 6 ，这两种点数各自出现的次数：

<img src="pic\image-20210505195522323.png" alt="image-20210505195522323" style="zoom:67%;" />

所以，想到用dp来减少重复计算次数

## dp

可以想到，已知有n-1个骰子的所有可能解`f(n-1,x)`，然后再添加一颗，每个点的概率是1/6，那么对应乘上概率1/6——$f(n, x) = \sum^{6}_{i=1}\frac{1}{6}f(n - 1, x - i)$

所以递推方程式就有了。

```java
class Solution {
    public double[] dicesProbability(int n) {
        double[] dp = new double[6];
        Arrays.fill(dp, 1.0/6.0);
        for(int i = 2; i <= n; i++){
            double[] temp = new double[5 * i + 1];	// 当前状态只和上一状态有关
            for(int k = 0; k < 6; k++){
                for(int j = 0; j < dp.length; j++){
                    temp[j + k] += dp[j] / 6.0;
                }
            }
            dp = temp;
        }
        return dp;
    }
}
```

这个我是没有想到的，希望下一次能够想到可以用dp

参考：

https://leetcode-cn.com/problems/nge-tou-zi-de-dian-shu-lcof/solution/jian-zhi-offer-60-n-ge-tou-zi-de-dian-sh-z36d/