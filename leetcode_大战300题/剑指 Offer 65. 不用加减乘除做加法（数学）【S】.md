## [剑指 Offer 65. 不用加减乘除做加法](https://leetcode-cn.com/problems/bu-yong-jia-jian-cheng-chu-zuo-jia-fa-lcof/)

<img src="pic\image-20210512221935837.png" alt="image-20210512221935837" style="zoom:67%;" />

感觉这题主要还是用了数学。

可以发现：1 + 0 = 1，0 + 1 = 1， 0 + 0 = 0， 1+1=0+进位1，所以可以发现，**不带进位的位相加，就是异或运算**

而进位就只有1同时出现时才有，且进位是前进到前一位上的，所以用的是**(1&1)<<1**

```java
class Solution {
    public int add(int a, int b) {
        while(b != 0){
            int c = (a & b) << 1;       // 进位
            a ^= b;     // 非进位，用异或
            b = c;      // 更新b
        }
        return a;
    }
}
```

