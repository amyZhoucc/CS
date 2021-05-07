# [剑指 Offer 64. 求1+2+…+n](https://leetcode-cn.com/problems/qiu-12n-lcof/)

<img src="pic\image-20210505165701013.png" alt="image-20210505165701013" style="zoom: 67%;" />

只能用加减法、然后逻辑判断&&、||、！等

利用条件判断&& 和 递归可以实现：&&能够实现类似于判断的操作：只有前面的条件满足了，后面的才会执行：

`n > 0 && sumNums(n-1)>0`后面的大于零主要是要实现后面也是一个布尔值的要求

```java
class Solution {
    int res = 0;
    public int sumNums(int n) {
        boolean flag = n > 0 && sumNums(n - 1) > 0;	// 核心
        res += n;
        return res;
    }
}
```

## 其他解法

1. 用数组越界来触发返回的

   ```java
   int[] nums = new int[1];
   
   public int sumNums(int n) {
       try {
           //只有当n等于1的时候，这里才返回
           //否则就走最下面的return
           return nums[n - 1] + 1;		// 数组越界了，会抛出异常，然后执行下面的语句
       } catch (Exception e) {
   
       }
       return n + sumNums(n - 1);
   }
   ```

2. 位运算

   主要是基于n(n + 1)/2，/2可以使用移位完成，而n(n+1)

   可以使用位运算来：基于思想：乘数1 * 乘数2，只有乘数对应的位上为1，才有加的意义；如果为0，则可以跳过

   由于限定范围在n <= 10000，所以只需要移位14次即可

   ```java
   public int sumNums(int n) {
       int a = n;
       int b = n + 1;
       int res = 0;
       boolean flag;
   
       //第1次
       // 判断当前最低位是否为1，如果为1，则需要加上a的值（==0，只是用维持是一个布尔运算）
       flag = ((b & 1) == 1) && ((res += a) == 0);
       a <<= 1;		// a*2
       b >>= 1;		// b/2，主要是获得当前bit位是否为1
       //第2次
       flag = ((b & 1) == 1) && ((res += a) == 0);
       a <<= 1;
       b >>= 1;
       //第3次
       flag = ((b & 1) == 1) && ((res += a) == 0);
       a <<= 1;
       b >>= 1;
       //第4次
       flag = ((b & 1) == 1) && ((res += a) == 0);
       a <<= 1;
       b >>= 1;
       //第5次
       flag = ((b & 1) == 1) && ((res += a) == 0);
       a <<= 1;
       b >>= 1;
       //第6次
       flag = ((b & 1) == 1) && ((res += a) == 0);
       a <<= 1;
       b >>= 1;
       //第7次
       flag = ((b & 1) == 1) && ((res += a) == 0);
       a <<= 1;
       b >>= 1;
       //第8次
       flag = ((b & 1) == 1) && ((res += a) == 0);
       a <<= 1;
       b >>= 1;
       //第9次
       flag = ((b & 1) == 1) && ((res += a) == 0);
       a <<= 1;
       b >>= 1;
       //第10次
       flag = ((b & 1) == 1) && ((res += a) == 0);
       a <<= 1;
       b >>= 1;
       //第11次
       flag = ((b & 1) == 1) && ((res += a) == 0);
       a <<= 1;
       b >>= 1;
       //第12次
       flag = ((b & 1) == 1) && ((res += a) == 0);
       a <<= 1;
       b >>= 1;
       //第13次
       flag = ((b & 1) == 1) && ((res += a) == 0);
       a <<= 1;
       b >>= 1;
       //第14次
       flag = ((b & 1) == 1) && ((res += a) == 0);
   
       return res >> 1;		// /2操作
   }
   ```

