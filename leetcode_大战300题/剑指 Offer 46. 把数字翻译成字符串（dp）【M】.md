# [剑指 Offer 46. 把数字翻译成字符串](https://leetcode-cn.com/problems/ba-shu-zi-fan-yi-cheng-zi-fu-chuan-lcof/)

<img src="pic\image-20210505201639482.png" alt="image-20210505201639482" style="zoom:67%;" />

很自然能够想到DP，就是`dp(i) = dp(i - 1) + dp(i - 2)`，但是有些不同的是，dp(2)不一定能加上去，还需要满足条件：

看当前的数字和前面一个数字构成的十位数是否满足10~25范围内，如果满足才能加上去

```java
class Solution {
    public int translateNum(int num) {
        int first = 1, second = 1;
        String s = Integer.toString(num);
        for(int i = 1; i < s.length(); i++){
            int two = (s.charAt(i - 1) - '0') * 10 + (s.charAt(i) - '0');   // 两位数
            if(two <= 25 && two >= 10){
                int temp = first + second;
                first = second;
                second = temp;
            }
            else first = second;
        }
        return second;
    }
}
```

——首先需要能够看出是dp，类似于走台阶的问题，再稍微加上一个限制条件。

当然，这边我是将数字转换成了字符串，但是需要用到额外的空间，而我们可以使用求余和求整来实现空间节省

——**动态规划的对称性**从后向前计算机解也是一样的结果

```java
class Solution {
    public int translateNum(int num) {
        int a = 1, b = 1, x, y = num % 10;
        while(num != 0) {
            num /= 10;
            x = num % 10;
            int tmp = 10 * x + y;
            int c = (tmp >= 10 && tmp <= 25) ? a + b : a;
            b = a;
            a = c;
            y = x;
        }
        return a;
    }
}
```

参考：https://leetcode-cn.com/problems/ba-shu-zi-fan-yi-cheng-zi-fu-chuan-lcof/solution/mian-shi-ti-46-ba-shu-zi-fan-yi-cheng-zi-fu-chua-6/