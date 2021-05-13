# [剑指 Offer 44. 数字序列中某一位的数字](https://leetcode-cn.com/problems/shu-zi-xu-lie-zhong-mou-yi-wei-de-shu-zi-lcof/)

<img src="pic\image-20210511150446160.png" alt="image-20210511150446160" style="zoom:67%;" />

找到某一位的某一个位数。

首先我们需要知道n对应的位数 - n对应的起始数字 - n对应的数字的第几位

1. n对应的位数

   通过规律：1~9：是9位；10~99：是90*2位；100~999：是900 *3位....

   所以我们可以通过一个for循环来迭代算一下对应的位数

   从而确定他的起始数字，是1 or 10 or 100.... => **start**，且能确定当前的**bit**值，即一个数字有多少位

   且此时的n已经是除去前面的所有完整位数后剩下的，eg：n-9-90*2-900 *3....

2. 找到对应的数字：可以通过找规律发现：**`(n-1)/bit`**就能找到对应的位

   | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    | 10   |
   | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
   | 1    | 0    | 1    | 1    | 1    | 2    | 1    | 3    | 1    | 4    |

3. 然后**(n - 1) % bit**就是对应的余数，所以将num转换成string，然后找到对应的位即可

```java
class Solution {
    public int findNthDigit(int n) {
        int bit = 1;			
        long count = 9;
        int start = 1;		// 开始的值，是1，10，100，1000....
        while(n > count){       // 看能否进入下一轮
            n -= count;     // 减去当前轮的总长度
            bit++;
            start *= 10;
            count = (long)9 * start * bit;       // 计算下一轮的总长度（这边可能会存在大数溢出的问题，所以需要将count强转为long）
        }	// n现在的值是除去前面的所有位数后剩下的
        int num = (int)start + (n - 1) / bit;
        return Long.toString(num).charAt((n - 1) % bit) - '0';
    }
}
```

——注意大数溢出的问题。由于count是预计下次能够减的数，所以存在溢出的可能

参考：https://leetcode-cn.com/problems/shu-zi-xu-lie-zhong-mou-yi-wei-de-shu-zi-lcof/solution/mian-shi-ti-44-shu-zi-xu-lie-zhong-mou-yi-wei-de-6/