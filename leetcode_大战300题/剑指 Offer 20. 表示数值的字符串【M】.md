# [剑指 Offer 20. 表示数值的字符串](https://leetcode-cn.com/problems/biao-shi-shu-zhi-de-zi-fu-chuan-lcof/)

<img src="pic\image-20210512232219335.png" alt="image-20210512232219335" style="zoom:67%;" />

<img src="pic\image-20210512232235828.png" alt="image-20210512232235828" style="zoom:67%;" />

这个题思想不难（不会用有限状态机，就用普通的方法吧）

这个题不用将实际的字符转换成数字，而是判断其表示是否存在问题，那么只需要列出满足要求的即可，不满足的要求就是其他

小数麻烦一点：

- 可以是前面没有数字，后面有无符号整数，即**.xxx**
- 前面有数字，后面无内容，即**xxx.**
- 前面有数字，后面有无符号整数，即**xxx.xxx**

```java
class Solution {
    private int index = 0;		// 全局变量
    private boolean unsignedInteger(String str){        // 判断是否存在无符号数（小数点后面的数字只能是无符号数）
        int before = index;
        while(str.charAt(index) >= '0' && str.charAt(index) <= '9') index++;
        return index > before;	// 出现移动，说明存在无符号数
    }
    private boolean number(String str){     // 判断是否存在数字（可以带符号）
        if(str.charAt(index) == '+' || str.charAt(index) == '-') index++;
        return unsignedInteger(str);
    }
    public boolean isNumber(String s) {
        if(s == null) return false;
        s = s + '|';		// 到尾巴做个限制，那么不需要考虑越界的问题
        while(s.charAt(index) == ' ') index++;
        boolean numeric = number(s);      // 判断是否存在整数
        if(s.charAt(index) == '.'){     // 如果存在小数
            index++;
            // 判断小数点后面是否存在无符号整数，如果不存在，那么前面需要是有整数的
            numeric = unsignedInteger(s) || numeric;  
        }
        if(s.charAt(index) == 'E' || s.charAt(index) == 'e'){	// 如果存在e
            index++;
            numeric = numeric && number(s);		// 前面是否是数字且后面是否是整数（可带符号）
        }
        while(s.charAt(index) == ' ') index++;		// 去除最后的空格
        return numeric && s.charAt(index) == '|';       // 前面是否满足是一个数，并且是否遍历到尾巴了
    }
}
```

