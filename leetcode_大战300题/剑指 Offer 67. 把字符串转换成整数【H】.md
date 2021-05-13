# [剑指 Offer 67. 把字符串转换成整数](https://leetcode-cn.com/problems/ba-zi-fu-chuan-zhuan-huan-cheng-zheng-shu-lcof/)

<img src="pic\image-20210512230742736.png" alt="image-20210512230742736" style="zoom:67%;" />

<img src="pic\image-20210512230757431.png" alt="image-20210512230757431" style="zoom:67%;" />

这题难度不是很大，就是要用简单的方法写出来比较有技巧。

1. 首先，前面是存在空格的，所以我们可以用trim将最前面的空格去掉（也可以自己写，这样时间消耗会少一点，不需要去看最后的空格）
2. 如果string去掉空格/本身为空了，那么直接返回0
3. 如果第一个字符是非数字、非符号，那么应该直接返回的。但是这边简单，直接判断如果第一个是符号-/+的，那么i++；并且如果是-的，那么标志位sign=-1
4. 开始遍历数组，如果如果遇到了非数字的，那么终止循环，将已有的数字输出；如果是数字那么常规的就是**num * 10 + arr[i]**；但是还存在一个大数的问题。所以**在每轮数字拼接前，判断在此轮拼接后是否超过 2147483647**：即拼接之前进行判断（否则拼接了就会越界了），所以需要维护的是**bound = Integer.MAX_VALUE / 10**，如果拼接前的值已经超过了它，那么直接返回最大值（最小值），如果正好相等，那么看当前位是否有可能超过，即**num == Integer.MAX_VALUE / 10 && arr[j] > 7**

```java
class Solution {
    public int strToInt(String str) {
        char[] arr = str.trim().toCharArray();
        if(arr.length == 0) return 0;
        int sign = 1, num = 0, i = 0;
        int bound = Integer.MAX_VALUE / 10;
        if(arr[0] == '-'){
            sign = -1;
            i++;
        }
        else if(arr[0] == '+'){
            i++;
        }
        for(int j = i; j < arr.length; j++){
            if(arr[j] < '0' || arr[j] > '9') break;
            if(num > bound || (num == bound && arr[j] > '7')) 
                return sign == -1 ? Integer.MIN_VALUE : Integer.MAX_VALUE;
            num = num * 10 + arr[j] - '0';
        }
        return sign == -1 ? -num : num;
    }
}
```

——总结技巧性并不强，理清楚思路就好做了