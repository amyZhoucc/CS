# [剑指 Offer 15. 二进制中1的个数](https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/)

<img src="pic\image-20210503104017571.png" alt="image-20210503104017571" style="zoom:67%;" />

主要思路就是：位运算（在Java的Integer包装类中存在很多位运算的骚操作）

可以发现：n&(n - 1)能除去n的最低位1，eg：101 & 100，那么最低位的1不见了，100 & 011，最低位的1也不见了，直接是0；

110 & 101，最低位的1不见了

```java
public class Solution {
    // you need to treat n as an unsigned value
    public int hammingWeight(int n) {
        int count = 0;
        while(n != 0){
            count++;
            n = n & (n -1);
        }
        return count;
    }
}
```

# 扩展：

1. 一句话判断一个整数是否是2的幂次方？

   即是否是2、4、8...

   可以`n & (n - 1)`来判断，如果结果是0，那么是；否则不是

2. 输入两个整数m、n，看从m的二进制变成n的二进制表示，需要改变多少位

   对m、n做异或，得到的值，计算1的个数

   eg：m = 1001, n = 1101, m & n = 0100，那么只需要修改1位即可

注意：要考虑到负数，Java的无符号右移是>>>（左移由于不存在符号位的影响，所以没有<<<）