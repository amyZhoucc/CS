# [剑指 Offer 14- I. 剪绳子](https://leetcode-cn.com/problems/jian-sheng-zi-lcof/)

<img src="pic\image-20210503095610215.png" alt="image-20210503095610215" style="zoom:67%;" />

求一个整数分解的最大积。

## 动态规划

比较直观，递推方程式可以写出来

$f(n) = max(f(i) * f(n-i)), (i \in (0, n))$

很明显，可以发现有很多重叠的解，那么可以从1开始计算，然后计算到n

```java
class Solution {
    public int cuttingRope(int n) {
        if(n < 2) return n;				
        else if(n == 2) return 1;
        else if(n == 3) return 2;		// 对于特殊的解的处理
        int[] store = new int[n + 1];
        store[0] = 0;
        store[1] = 1;
        store[2] = 2;
        store[3] = 3;			// 特殊解作为因子（即不拆分）的值：就是其本身
        for(int i = 4; i <= n; i++){
            int max = i;
            for(int j = 1; j <= i / 2; j++){
                int cur = store[j] * store[i - j];
                if(max < cur){
                    max = cur;
                }
            }
            store[i] = max;
        }
        return store[n];
    }
}
```

## 贪心

代码简单，但是证明最优比较难。

通过数学推导可以得出：划分成每段为3最优。

每段长度一样长是最优的，且$x^a = x^{\frac{n}{x}} = (x^{\frac{1}{x}})^n$

本质上就是求$x^{\frac{1}{x}}$，在x=e处有极大值

```java
class Solution {
    public int cuttingRope(int n) {
        if(n < 2) return n;
        else if(n == 2) return 1;
        else if(n == 3) return 2;	// 处理特殊情况
        int radix = n / 3;		// 尽量切分成3大小的
        int rest = n % 3;
        if(rest == 1){		// 如果最后余1，那么还是划分成3*4
            return (int)Math.pow(3, radix - 1) * 4;
        }
        if(rest == 2){
            return (int)Math.pow(3, radix) * 2;
        }
        else{
            return (int)Math.pow(3, radix);
        }
    }
}
```

证明：

https://leetcode-cn.com/problems/jian-sheng-zi-lcof/solution/mian-shi-ti-14-i-jian-sheng-zi-tan-xin-si-xiang-by/

# [剑指 Offer 14- II. 剪绳子 II](https://leetcode-cn.com/problems/jian-sheng-zi-ii-lcof/)

<img src="pic\image-20210503103013319.png" alt="image-20210503103013319" style="zoom:67%;" />

就是加上了一个溢出判断

```java
class Solution {
    private int pow(int radix){
        long count = 1;
        for(int i = 1; i <= radix; i++){
            count *= 3;		// 如果是int可能存在溢出，类型为long，就不会溢出了
            count %= 1000000007;
        }		
        return (int)count;	//最后类型强转一下就可以了
    }
    public int cuttingRope(int n) {
        if(n == 2) return 1;
        else if(n == 3) return 2;
        int count = 1;
        int radix = n / 3;
        if(n % 3 == 0){
            return pow(radix);
        }
        else if(n % 3 == 1){
            // *4还可能溢出，所以乘之前先转换为long，然后之后强转回来
            return (int)((pow(radix - 1) * (long)4) % 1000000007);
        }
        else{
            return (int)((pow(radix) * (long)2) % 1000000007);
        }
    }
}
```

