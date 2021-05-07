# [剑指 Offer 16. 数值的整数次方](https://leetcode-cn.com/problems/shu-zhi-de-zheng-shu-ci-fang-lcof/)

<img src="pic\image-20210503153842569.png" alt="image-20210503153842569" style="zoom:67%;" />

拿到这个题目，很直观想到的暴力解：就是将底数不断地相乘，乘n次就是要的答案，但这肯定不是最优的时间复杂度。

然后，可以想到二分求解，即 x = 2, n = 10，那么是$2^5 * 2^5$，而$2^5$，可以拆分成$2^2 * 2^2 * 2$，那么很自然的采用递归的解法

时间复杂度为O(logN)

递归解法：

能够想到用一个二分来求解。

```java
class Solution {
    public double myPow(double x, int n) {
        if(n == 0) return 1;		// 边界条件——base^0，为1
        else if(n == 1) return x;		// 递归终止的条件
        // 防止溢出，所以先额外乘一个
        else if(n < 0) return (1 / x) * myPow(1 / x, -(n + 1));
        else{
            double res = myPow(x, n >> 1);
            res *= res;
            if((n & 1) == 1) res *= x;		// 如果n是奇数，则还需要再乘上一个base
            return res;
        } 
    }
}
```

位运算的解法：

思想：eg：10 = b1010，那么可以变成$2^8 * 2^2$，所以我们只需要判断n对应二进制有哪几位为1，乘上对应的radix即可

这个时间复杂更小，限制在32次内：

```java
class Solution {
    public double myPow(double x, int n) {
        long nn = n;			// 为了处理边界情况：n = Integer.MIN_VALUE,-n然后就溢出了，所以用long存储
        if(n == 0) return 1;
        else if(nn < 0){        // 当n为负数时，x是倒数
            x = 1 / x;
            nn = -nn;
        }
        double res = 1;
        double radix = x;
        while(nn != 0){
            if((nn & 1) == 1) res *= radix;		// 如果该位存在数的话，那么需要乘上，不存在的话，那么掠过
            radix *= radix;		// 更新2次 - 4次 - 8次...
            nn >>= 1;
        }
        return res;
    }
}
```

——可以得出一个结论，**任何一个数都可以被2的幂次表示**，本质上就是**任何一个数都有二进制表示，就能被表示成2的幂次的多项式（且系数只能为0/1）**