# [剑指 Offer 50. 第一个只出现一次的字符](https://leetcode-cn.com/problems/di-yi-ge-zhi-chu-xian-yi-ci-de-zi-fu-lcof/)

<img src="pic\image-20210505180128164.png" alt="image-20210505180128164" style="zoom:67%;" />

用哈希表或者自制的哈希表就可以完成（没有更优的解了）

```java
class Solution {
    public char firstUniqChar(String s) {
        int[] alp = new int[26];
        for(int i = 0; i < s.length(); i++){
            alp[s.charAt(i) - 'a']++;
        }
        for(int i = 0; i < s.length(); i++){
            if(alp[s.charAt(i) - 'a'] == 1) return s.charAt(i);
        }
        return ' ';
    }
}
```

——需要统计次数，且限定在小写/大写字母等条件时，可以用自制的哈希表进行统计。

ps：

https://leetcode-cn.com/problems/di-yi-ge-zhi-chu-xian-yi-ci-de-zi-fu-lcof/solution/mian-shi-ti-50-di-yi-ge-zhi-chu-xian-yi-ci-de-zi-3/

在哈希表的基础上，有序哈希表中的键值对是 **按照插入顺序排序** 的。基于此，可通过遍历有序哈希表，实现搜索首个 “数量为 1的字符”。