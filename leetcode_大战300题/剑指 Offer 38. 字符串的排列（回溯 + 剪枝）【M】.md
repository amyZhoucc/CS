# [剑指 Offer 38. 字符串的排列](https://leetcode-cn.com/problems/zi-fu-chuan-de-pai-lie-lcof/)

<img src="pic\image-20210507223055731.png" alt="image-20210507223055731" style="zoom: 80%;" />

典型的全排列问题，做出来是没有问题的，但是想要做优是比较难的。

需要遇到的问题是：

1. 去重：如果遇到aabbcc，如果不去重的话，按照回溯的执行可能存在多个aabbcc——**用hashSet存储当前层已经遍历过的字母**，如果出现同样的字母就认为是重复了，则跳过，那么重复的字母有且只会出现一种排序，而不会重复，eg：aabc：第一层：当第二个a作为首字母的时候，由于第一个a已经进入hashset中了，那么该a就跳过了，那么不会重复出现aabc、aabc
2. 如何判断哪些结点已经遍历过了——flag数组，将本次dfs深度中已经遍历的结点全部标记为true；当回溯的时候，全部重置为false。

```java
class Solution {
    private void dfs(char[]ss, boolean[] flag, StringBuilder sb, List<String>res){
        if(sb.length() == ss.length){
            res.add(sb.toString());
        }
        HashSet<Character>hashSet = new HashSet<>();
        for(int i = 0; i < ss.length; i++){
            if(flag[i]) continue;           // 已经遍历过的
            if(hashSet.contains(ss[i])) continue;       // 去重，除去遇到一样的
            sb.append(ss[i]);
            hashSet.add(ss[i]);·		// 该字母已经出现过了，则该层中不能再次进入同样的一个字母
            flag[i] = true;		// 表示该字母已经遍历过了，下面几层均不能再用
            dfs(ss, flag, sb, res);
            sb.deleteCharAt(sb.length() - 1);
            flag[i] = false;
        }
    }
    public String[] permutation(String s) {
        List<String>res = new ArrayList<>();
        char[]ss = s.toCharArray();
        boolean[] flag = new boolean[ss.length];
        dfs(ss, flag, new StringBuilder(), res);
        String[] total = new String[res.size()];
        for(int i = 0; i < res.size(); i++){
            total[i] = res.get(i);
        }
        return total;
    }
}
```

这边比较冗余的是：每一层都遍历了整个字符串数组[0, length - 1]，并且需要额外的boolean数组进行标记。存在浪费

[题解](https://leetcode-cn.com/problems/zi-fu-chuan-de-pai-lie-lcof/solution/mian-shi-ti-38-zi-fu-chuan-de-pai-lie-hui-su-fa-by/)用了超级妙的方法：

——主要是通过移动字符串来实现对字符串数组的遍历：参数中cur其实表示的是层数，实际上就是新字符串的尚未固定的结点

eg：dfs(ss, 0, res)，表示当前是第0层，那么dfs前的swap都是i和0进行互换，实际上就是将index=0，这一位固定下来

eg：char=[a, b, c]，那么第0层，分别固定了a、b、c，对应交换后的结果是[a, b, c]，[b,a,c]，[c,b,a]，然后继续接下来的操作。

```java
class Solution {
    private void swap(char[]ss, int i, int j){
        char temp = ss[i];
        ss[i] = ss[j];
        ss[j] = temp;
    }
    private void dfs(char[]ss, int cur, List<String>res){
        if(cur == ss.length -1){		// cur指向最后一个，表示前面的位已经全部固定了，则可以直接返回了
            res.add(String.valueOf(ss));
            return;
        }
        HashSet<Character>hashSet = new HashSet<>();		// 去重
        for(int i = cur; i < ss.length; i++){
            if(hashSet.contains(ss[i])) continue;
            hashSet.add(ss[i]);
            swap(ss, cur, i);		// 将i结点固定到当前层对应的位置
            dfs(ss, cur + 1, res);
            swap(ss, i, cur);			// 回溯之后变回原来的样子
        }
    }
    public String[] permutation(String s) {
        List<String>res = new ArrayList<>();
        char[]ss = s.toCharArray();
        dfs(ss, 0, res);
        return res.toArray(new String[res.size()]);
    }
}
```

参考：

https://leetcode-cn.com/problems/zi-fu-chuan-de-pai-lie-lcof/solution/mian-shi-ti-38-zi-fu-chuan-de-pai-lie-hui-su-fa-by/

# 拓展

是求集合的幂集，那么意味着集合本身是不重复的，且集合的幂集也是不能重复的。

总体思想还是回溯，但是更简单，直接for循环即可，那么能够保证不重复。

```java
class Solution {
    private void backtrack(int[] nums, List<List<Integer>>res, List<Integer>list, int start){
        if(start == nums.length) return;
        for(int i = start; i < nums.length; i++){
            list.add(nums[i]);
            res.add(new ArrayList<>(list));
            backtrack(nums, res, list, i + 1);
            list.remove(list.size() - 1);
        }
    }
    public List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> res = new ArrayList<>();
        res.add(new ArrayList<Integer>());
        backtrack(nums, res, new ArrayList<Integer>(), 0);
        return res;
    }
}
```

还可以用非递归的方法，就是每次都将一个新的结点加入到原有的列表中

```java
class Solution {
    public List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> res = new ArrayList<>();
        res.add(new ArrayList<Integer>());      // 先加入空集
        for(int num: nums){
            int len = res.size();	// 获取上一次的集合个数
            for(int i = 0; i< len; i++){	// 对每个集合都进行添加num操作
                List<Integer> temp = new ArrayList<Integer>(res.get(i));
                temp.add(num);
                res.add(temp);
            }
        }
        return res;
    }
}
```



<img src="pic\image-20210511120542875.png" alt="image-20210511120542875" style="zoom:67%;" />

全排列之后 + 数学计算。

## [51. N 皇后](https://leetcode-cn.com/problems/n-queens/)

<img src="pic\image-20210511120845351.png" alt="image-20210511120845351" style="zoom:67%;" />

这个也是一个经典题，思路就是回溯，但是为了实现快速的判断，需要很多辅助，还有输出结果。