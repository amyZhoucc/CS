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

所以实际上n个骰子的所有概率，只和n-1个骰子的概率有关。

1. 首先，可以发现1个骰子的出现范围在[1, 6]，而2个骰子的范围在[2, 12]....，所以n个骰子的有效范围为**[n, 6n]**，所以实际上有效数字个数是**6n-n+1=5n+1**

   ——所以为了表示方便，每次循环可以用6*n+1长度的数组

2. 

实现主要参考：

<img src="pic\image-20210512204406977.png" alt="image-20210512204406977" style="zoom:67%;" />

<img src="pic\image-20210512215047602.png" alt="image-20210512215047602" style="zoom:67%;" />

```java
class Solution {
    public double[] dicesProbability(int n) {
        int[] dp = new int[6 + 1];		// 0~6，第0个是无效的，主要是用[1,6]，方便处理
        Arrays.fill(dp, 1);			// 即一个骰子，所以每个数字都是出现1次
        
        for(int i = 2; i <= n; i++){
            int[] temp = new int[6 * i + 1];	// 创建一个新的数组，范围为[0, 6i], 有效范围为[i, 6i]
            for(int j = 0; j < dp.length; j++){     // 复制整个数组
                temp[j] = dp[j];
            }
            for(int j = 6 * i; j >= i; j--){		// 从最后一个数字开始
                temp[j] = 0;			// 当前数字内容需要清零
                for(int k = 1; k <= 6; k++){	// 从[j - 6, j - 1]范围内可以组成j的出现总数
                    if(j - k < i - 1) break;	// i-1，就是上一次dp中的最小有效位
                    temp[j] += temp[j - k];
                }
            }
            dp = temp;		// 更新dp
        }
        double radix = Math.pow(6, n);
        double[] res = new double[5 * n + 1];
        int j = dp.length - 1;
        for (int i = res.length - 1; i >= 0 ; i--) {		// 只取有效位即可
            res[i] = (double)dp[j--] / radix;
        }
        return res;
    }
}
```

这个我是没有想到的，希望下一次能够想到可以用dp

参考：

https://leetcode-cn.com/problems/nge-tou-zi-de-dian-shu-lcof/solution/nge-tou-zi-de-dian-shu-dong-tai-gui-hua-ji-qi-yo-3/