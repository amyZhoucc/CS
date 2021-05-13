# [剑指 Offer 19. 正则表达式匹配](https://leetcode-cn.com/problems/zheng-ze-biao-da-shi-pi-pei-lcof/)

<img src="pic\image-20210509231847030.png" alt="image-20210509231847030" style="zoom:67%;" />

<img src="pic\image-20210509231904172.png" alt="image-20210509231904172" style="zoom:67%;" />

理解题目：s是字符串，p是匹配。对于普通的字符都要求一一匹配，但是

- 如果p中出现了*，那么是前面字符的n倍
- 如果出现了.，那么匹配一个随意字符

主要思想是DP。

设定A是给定的，然后用B去核验是否满足匹配，都是从最后一个开始匹配：

B的最后一个可能是：普通字符、*、.

1. 普通字符，那么s的最后一个字符也必须完全一样才行，即`A[N-1]==B[M-1]?`，如果相等就看`[0, N-2]`和`[0, M-2]`范围内内容
2. *，那么直接跳过A的最后一个字符，即看`[0, N-2]`和`[0, M-2]`范围内内容
3. .，那么就是`B[M-2]`可以重复0~n次，`B[M-2]*`是一个整体考量
   1. 如果$A[N-1] \ne B[M-2]$，那么就看`[0, N-2]`和`[0, M-3]`范围内内容
   2. 如果相等，那么看`[0, N-2]`和`[0, M-1]`范围内的内容

——所以能发现能变成子问题，所以可以用dp

那么递推方程式：`f[i][j]`

- 场景1、2：`f[i][j] = f[i-1][j-1]`
- 场景3：可以选择是`f[i][j]=f[i][j-2] / f[i-1][j]`

对于dp来说，我们肯定可以从字符串头开始进行匹配的，所以我们需要初始化

处理特殊情况：

1. A=空，B=空，那么`f[0][0]=true`
2. A=空，B=!空，那么需要计算的
3. A=!空，B=空，那么`f[0][0]=false`
4. 均非空，那么需要计算

我们需要申请一个len+1，len2+1的二维数组。

首先处理边界条件，即s==null，p!=null，均填在第一行：

遍历一遍p，如果当前是*，那么将 *和前一个合并看成一个整体跳过，所以直接看i-2时的结果

```java
for(int i = 0; i < len2; i++){
    // i=*，那么将i-1和i看成一个整体，然后dp[0][i + 1] = dp[0][i - 1](i结果存放在i+1中，所以i+2存放的是i-1的结果)
    if(p.charAt(i) == '*' && dp[0][i - 1]) dp[0][i + 1] = true;
}      
```

接着我们来看匹配的：

对于两边都是普通字符，那么直接看字符是否相等：如果相等直接将`dp[i][j]`的结果拿来即可

```
dp[i + 1][j + 1] = dp[i][j] if s.i = p.j
```

如果p是特殊字符.，那么上面的判断肯定不成立，那么可以跳过本次比较，直接看(i, j)结果

```
dp[i + 1][j + 1] = dp[i][j] if s.i != p.j && p.j==.
```

如果p是特殊字符*，那么上面的判断肯定不成立，那么需要看s的当前字符和p的前一个字符是否匹配（s肯定是普通字符，p是普通字符或者是.）即s.i == p.j-1，如果两者不相等，那么需要将j-1和j看成一个整体进行消去（即出现零次）

还需要注意一种特殊情况，p.j-1=.，那么虽然不匹配，它实际上可以匹配任意字符，所以我们需要将其排除

```
dp[i + 1][j + 1] = dp[i + 1][j - 1]; if s.i != p.j-1 && p.j-1 !='.'
```

如果i和j-1匹配了/p.j-1=.，那么需要考虑如下3种情况：

1. eg：s=abc， p=abcc*，那么虽然能够匹配，但是还是要当整体消除

   `dp[i + 1][j + 1] = dp[i + 1][j - 1]`

2. eg：s=abc，p=abc*，能够匹配且只匹配了一个，那么将 *忽略即可，就看前面的

   `dp[i + 1][j + 1] = dp[i + 1][j]`

3. eg：s=abccccc，p=abc*，能够匹配多个，那么将s的最后一个字符去掉，然后看之前的，直到变成了第2种。

   `dp[i + 1][j + 1] = dp[i][j + 1]`

——只要3个里面满足一个就可以返回了

```java
class Solution {
    public boolean isMatch(String s, String p) {
        if(p == null) return s == null;     // 处理特殊情况
        int len1 = s.length(), len2 = p.length();
        boolean[][]dp = new boolean[len1 + 1][len2 + 1];
        dp[0][0] = true;
        for(int i = 0; i < len2; i++){
            if(p.charAt(i) == '*' && dp[0][i - 1]) dp[0][i + 1] = true;
        }           // 先处理第一行的问题——也算是边界条件：如果s==null，而p!=null时，也还是需要计算的
        for(int i = 0; i < len1; i++){
            for(int j = 0; j < len2; j++){
                if(s.charAt(i) == p.charAt(j) || p.charAt(j) == '.') dp[i + 1][j + 1] = dp[i][j];
                else if(p.charAt(j) == '*'){
                    if(s.charAt(i) != p.charAt(j - 1) && p.charAt(j - 1) != '.' ) dp[i + 1][j + 1] = dp[i + 1][j - 1];
                    else{
                        dp[i + 1][j + 1] = dp[i + 1][j] || dp[i + 1][j - 1] || dp[i][j + 1];
                    }
                }
            }
        }
        return dp[len1][len2];
    }
}
```

参考：

https://leetcode-cn.com/problems/zheng-ze-biao-da-shi-pi-pei-lcof/solution/dong-tai-gui-hua-he-di-gui-liang-chong-fang-shi-ji/

——这题真的太难了，感觉智商被按在地上摩擦，多看几遍，记结果吧。

# 扩展

这个题就是对两个字符串进行匹配，并且要考虑特殊字符的情况，其本质上也是进行最大匹配，直到不能匹配为止，那么后面就都是false

可以看到，对于这种最大匹配来说，可以用dp，那么可以做几题经典的匹配题来熟悉一下

# [1143. 最长公共子序列](https://leetcode-cn.com/problems/longest-common-subsequence/)

这个是一个中等题，相对上面的就比较简单。

<img src="pic\image-20210510112653648.png" alt="image-20210510112653648" style="zoom:67%;" />

<img src="pic\image-20210510112708090.png" alt="image-20210510112708090" style="zoom: 67%;" />

**求两个数组或者字符串的最长公共子序列问题，肯定是要用动态规划的**

**子序列：是可以不连续的**，只需要相对顺序保证即可。

如何想到用dp呢：

假设我们从最后一个节点开始比较：

- eg：abcde、ace：最后一个相等，那么**结果就是之前的匹配结果+1**，即`dp[i][j] = dp[i-1][j-1]+1`；
- eg：ace、bc，如果最后一个不相等，那么它们不能同时移动了，所以要么A向前移动，再和B比较，要么B向前移动，再和A比较，即`dp[i - 1][j]、dp[i][j - 1]`，然后就是选择其中的最大值即可，例子里面dp是0，1，所以选择1，即`dp[i][j] = max(dp[i-1][j], dp[i][j-1])`

——所以既然能写出递推方程式了，那么肯定是能用dp的

在实现的时候，我们需要增加初始的行，即a==null对应的一行；b=null对应的一列，默认都是0

```java
class Solution{
	public int longestCommonSubsequence(String text1, String text2) {
        if(text1 == null || text2 == null) return 0;		// 处理特殊情况
        int len1 = text1.length(), len2 = text2.length();
        int[][]dp = new int[len1 + 1][len2 + 1];
        for(int i = 1; i <= len1; i++){
            for(int j = 1;j <= len2; j++){
                if(text1.charAt(i) == text2.charAt(j)) dp[i][j] = dp[i - 1][j - 1] + 1;
                else dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
        return dp[len1][len2];
    }
}
```

如果要求用递归求解：

```java
class Solution {
    private int dfs(String text1, String text2, int i, int j){
        if(i < 0 || j < 0) return 0;    // 递归终止条件
        if(text1.charAt(i) == text2.charAt(j)) return dfs(text1, text2, i - 1, j - 1) + 1;
        else return Math.max(dfs(text1, text2, i - 1, j), dfs(text1, text2, i, j - 1));
    }
    public int longestCommonSubsequence(String text1, String text2) {
        if(text1 == null || text2 == null) return 0;        
        return dfs(text1, text2, text1.length() - 1, text2.length() - 1);
    }
}
```

——存在重复求解的问题，所以性能不好，会超时。

参考：

https://leetcode-cn.com/problems/longest-common-subsequence/solution/fu-xue-ming-zhu-er-wei-dong-tai-gui-hua-r5ez6/

它提供了一个动态规划的套路：

单个数组or字符串需要用dp时，可以定义一个一维数组；当两个数组进行dp时，需要定义一个二维数组`dp[i][j]`，表示串A的[0, i]和串B的[0, j]的dp结果。