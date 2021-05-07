# [剑指 Offer 49. 丑数](https://leetcode-cn.com/problems/chou-shu-lcof/)

<img src="pic\image-20210505225609361.png" alt="image-20210505225609361" style="zoom:67%;" />

能够想到是动态规划，之前的数肯定是丑数，然后用丑数乘上2/3/5肯定还是丑数，但是存在的是顺序问题，

eg：2是丑数，2*2，2 * 3，2*5也都是丑数，但是他们并不是连续的丑数

所以，我们可以**维护三指针**，分别指向2对应的被乘数、3对应的被乘数、5对应的被乘数，然后每次插入的时候都判断一下，哪个最小

```java
class Solution {
    public int nthUglyNumber(int n) {
        int[] res = new int[n];
        res[0] = 1;
        int a = 0, b = 0, c = 0;		// 三指针
        for(int i = 1; i < n; i++){
            res[i] = Math.min(Math.min(res[a] * 2, res[b] * 3), res[c] * 5);// 选择最小值插入
            if(res[i] == res[a] * 2) a++;		// 更新a++，也可以用来去重，eg:2*5,5*2，肯定只有一个能存在
            if(res[i] == res[b] * 3) b++;
            if(res[i] == res[c] * 5) c++;
        }
        return res[n - 1];
    }
}
```

