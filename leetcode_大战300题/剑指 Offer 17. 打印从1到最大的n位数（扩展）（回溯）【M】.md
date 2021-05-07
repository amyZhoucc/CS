# [剑指 Offer 17. 打印从1到最大的n位数](https://leetcode-cn.com/problems/da-yin-cong-1dao-zui-da-de-nwei-shu-lcof/)

<img src="pic\image-20210307100913233.png" alt="image-20210307100913233" style="zoom:80%;" />

本题求解很简单，不是关键，关键在于对大数的求解

## 1. 本题解法

由于限定死了输出的数组是`int[]`，所以结果一定是int范围内的，而int的最大值是**21亿多一点**，所以最多是9位。直接设置一个辅助数组，该数组内存放1~9位数的十进制最大值。然后这个就是遍历上限：

```java
class Solution {
    public int[] printNumbers(int n) {      // n=1~9
        int[] helper = new int[]{9, 99, 999, 9999, 99999, 999999, 9999999, 99999999, 999999999};
        int maxValue = helper[n - 1];
        int[] res = new int[maxValue];
        for(int i = 0; i < maxValue; i++){
            res[i] = i + 1;
        }
        return res;
    }
}
```

## 2. 扩展——大数遍历

上面的解法太过于局限，所以一般会要求扩展到大数范围，即n是一个正整数，那么当n>=10后，就存在int无法表达的数字，所以需要完全切换方法：

首先明确，不管是存储的数组还是数字本身都已经不能简单的用int来表示，而统一用string来表示。

有两种方法：模拟，模拟++的过程和进位过程，但是进位效率不高，需要从最低位一位一位的判断过去。

​						**全排列**：可以发现每一位的取值都是限制在0~9，我们可以用全排列的方式求解

而全排列的解法：用回溯法：——只要n不要太大递归的深度不会太深，但是广度很大

```java
class Solution{
    private static void dfs(StringBuilder res, StringBuilder sb, char[] num, int len){
        if(sb.length() == len){             // 递归终止条件
            res.append(sb.toString() + ",");
            return;
        }
        for(char c: num){
            sb.append(c);
            dfs(res, sb, num, len);
            sb.deleteCharAt(sb.length() - 1);
        }
    }
    public static void main(String[] args){
        int len = 9;
        char[] num = {'0','1','2','3','4','5','6','7','8','9'};
        StringBuilder res = new StringBuilder();
        dfs(res, new StringBuilder(), num, len);
        res.deleteCharAt(res.length() - 1);
        System.out.println(res.length());
    }
}
```

但是在实际运行的时候，n值过大，使产生的string太大会报错

但是仍存在一个问题：所有打印出来的值位数都一样，eg：00001，而实际上我们会把最高位之前的0全部去掉：

[大佬解法](https://leetcode-cn.com/problems/da-yin-cong-1dao-zui-da-de-nwei-shu-lcof/solution/mian-shi-ti-17-da-yin-cong-1-dao-zui-da-de-n-wei-2/)

```java
class Solution{
	private static int nine, start;     //  // nine标记的是此次dfs中出现9的次数，start标记的是最高位之前0的个数
    private static void dfs(StringBuilder res, StringBuilder sb, char[] num, int len){
        if(sb.length() == len){             // 递归终止条件
            res.append(sb.toString().substring(start) + ",");
            if(len - start == nine) start--;	// 满足了低位全为9，即0099，那说明之后是从0100开始的了，所以start要向前移（当前还不用移动）	
            return;
        }
        for(char c: num){
            sb.append(c);
            if(c == '9') nine++;			// 要发生进位了，09， 下一个就是10了
            dfs(res, sb, num, len);
            sb.deleteCharAt(sb.length() - 1);
        }
        nine--;			// 回溯之后需要恢复
    }
    public static void main(String[] args){
        int len = 5;
        char[] num = {'0','1','2','3','4','5','6','7','8','9'};
        StringBuilder res = new StringBuilder();
        start = len - 1;        // 初始设置为n-1，表示除了最低位其余都是0
        dfs(res, new StringBuilder(), num, len);
        res.deleteCharAt(res.length() - 1);
        System.out.println(res);
    }
}
```

增加两个全局变量：nine：用来记录当前9的个数（由于是递归，所以如果一直都是9，那么会不断增加上去）

eg：len=6， 

- 前面高位为00000，固定最低位为9，此时nine=1，而此时start=5，所以从000009变成9，根据回溯的原理，下次遍历到的结点是000010，此时start更新：start=4；返回之后，nine=0；继续执行dfs
- 此时前面高位为0000，已经固定了次低位为9，而再此基础上会进行dfs，当固定最低位为9，nine=2，此时000099，下次遍历的结点是000100，此时start更新：start：3；一次递归返回，nine=0
- ....

作者tql

我的解法：在dfs的if里面设置一个while循环，将前面的0全部删除（如果最后全部删没了，那说明原来的值是0，补上即可）——效率不行

参考：

1. https://leetcode-cn.com/problems/da-yin-cong-1dao-zui-da-de-nwei-shu-lcof/solution/mian-shi-ti-17-da-yin-cong-1-dao-zui-da-de-n-wei-2/