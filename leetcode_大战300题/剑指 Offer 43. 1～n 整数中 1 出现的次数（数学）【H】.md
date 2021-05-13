# [剑指 Offer 43. 1～n 整数中 1 出现的次数](https://leetcode-cn.com/problems/1nzheng-shu-zhong-1chu-xian-de-ci-shu-lcof/)

<img src="pic\image-20210511215602997.png" alt="image-20210511215602997" style="zoom:67%;" />

这题真的太难了。

先找规律：

1. 0~9，只有一个1，如果某个数是10的a倍，那么在个位上存在a个1
2. 10~99，在十位上有10个1，如果某个数是100的a倍，那么在**十位上存在a*10个1**；如果是10的b倍，那么在**个位上存在b个1**
3. 100~999，在百位上有100个1，而某个数是1000的a倍，那么在**百位上个存在a*100个1**；如果是100的b倍，那么在**十位上存在b*10个1**；如果是10的c倍，那么在**个位上存在c个1**

然后还有余数，那么该如何处理余数呢？

eg：如果某个数是100的x倍，而余数有y，那么十位上的个数：默认的就有x*10个1，但是未考虑余数：

1. 如果余数 >= 20，那么十位上的实际个数是**(x + 1) * 10**
2. 如果余数 < 20 && 余数 >= 10，那么十位上的实际个数是**(x+1) * 10  + (y-10+1)**
3. 如果余数<10，那么十位上的实际个数是**x*10**

终于找到一个可以理解的方法了。:expressionless: 

需要注意的是：radix是

```java
class Solution {
    public int countDigitOne(int n) {
        int count = 0;
        long radix = 1;
        while(n >= radix){		// 注意除数最后一定比n要大
            int div = (int)(n / (radix * 10));			// 取整
            int rest = (int)(n % (radix * 10));			// 取余
            if(rest >= 2 * radix) count += (div + 1) * radix;		// 根据上面余数的结论进行计算
            else if(rest >= radix) count += div * radix + rest - radix + 1;
            else count += div * radix;
            radix *= 10;				// 除数向前推
        }
        return count;
    }
}
```

eg：12，实际上我先是算10，后算100，分别代表个位、十位的1的个数；345，实际上也是先算10，100，1000，分别代表个位、十位、百位。

参考：
https://leetcode-cn.com/problems/1nzheng-shu-zhong-1chu-xian-de-ci-shu-lcof/solution/ji-jian-jie-fa-by-harrisliao/（终于找到能看懂的解法了）