

# [剑指 Offer 58 - I. 翻转单词顺序](https://leetcode-cn.com/problems/fan-zhuan-dan-ci-shun-xu-lcof/)

<img src="pic\image-20210505173129892.png" alt="image-20210505173129892" style="zoom:67%;" />

1. 首先：需要将首尾的空格全部去掉——利用字符串自带的方法trim
2. 遍历字符串，
   - 用一个指针i从尾巴开始遍历，找到第一个空格，那么[i, j]范围内就是最后一个字符串，然后将该字符串加入到输出中
   - 由于中间可能存在多个连续的空格，所以需要用while循环将空格删掉，然后更新i和j——指向最近的非空格的字符
3. 最前面一个单词，会因为i<0，而没有加入到输出中，所以需要将其加上
4. 最后去除空格，直接返回

```java
class Solution {
    public String reverseWords(String s) {
        s.trim();      // 去除首尾多余的空格
        StringBuilder sb = new StringBuilder();
        int i = s.length() - 1, j = i;
        while(i >= 0){
            if(s.charAt(i) == ' '){
                sb.append(s.substring(i + 1, j + 1) + ' ');
                while(i >= 0 && s.charAt(i) == ' '){
                    i--;
                    j = i;
                }
            }
            else{
                i--;
            }
        }
        if(i != j) sb.append(s.substring(i + 1, j + 1) + ' ');
        return sb.toString().trim();
    }
}
```

# [剑指 Offer 58 - II. 左旋转字符串](https://leetcode-cn.com/problems/zuo-xuan-zhuan-zi-fu-chuan-lcof/)

<img src="pic\image-20210505174538346.png" alt="image-20210505174538346" style="zoom:67%;" />

这题从效率来讲，其他方法没有思考的必要。

```java
class Solution {
    public String reverseLeftWords(String s, int n) {
        StringBuilder sb = new StringBuilder();
        sb.append(s.substring(n, s.length()));
        sb.append(s.substring(0, n));
        return sb.toString();
    }
}
```

还有其他方法：

如果不能用substring的方法，那么可以将字符串存入一个数组中，然后从n开始存储`i%s.length()`

```java
class Solution {
    public String reverseLeftWords(String s, int n) {
        StringBuilder res = new StringBuilder();
        for(int i = n; i < n + s.length(); i++)
            res.append(s.charAt(i % s.length()));
        return res.toString();
    }
}
```

如果只能用String，那么将stringBuilder替换成String，然后每次都用string+=，则效率会很低