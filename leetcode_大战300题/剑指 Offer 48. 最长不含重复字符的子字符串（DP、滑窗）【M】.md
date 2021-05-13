# [剑指 Offer 48. 最长不含重复字符的子字符串](https://leetcode-cn.com/problems/zui-chang-bu-han-zhong-fu-zi-fu-de-zi-zi-fu-chuan-lcof/)

<img src="pic\image-20210511202627444.png" alt="image-20210511202627444" style="zoom:67%;" />

不重复的子串，就说明是需要连续的。

能够看出来可以用滑动窗口来做：

## 滑动窗口

主要思想就是：双指针，left、right，right用来遍历，如果right指向的结点在已经出现过了，那么需要调整left指针，而left的调整主要看：该结点的前一个结点的为止和当前left的位置，eg：left=0，prev=2，那么选择prev；如果left = 3，prev=2，那么仍旧是3（表示当前结点的prev结点实际上不在left~right范围内）

为了查找方便，在存储的时候，key：结点内容，value：结点对应的下标+1，**表示从该位置后一个才开始不重复**

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        HashMap<Character, Integer>store = new HashMap<>();
        int maxLen = 0;
        int left = 0;
        for(int i = 0; i < s.length(); i++){
            if(store.containsKey(s.charAt(i))){
                left = Math.max(left, store.get(s.charAt(i)));
            }
            maxLen = Math.max(maxLen, i - left + 1);
            store.put(s.charAt(i), i + 1);
        }
        return maxLen;
    }
}
```

## dp

主要是：利用dp和哈希表，哈希表来判断结点最近出现的为止

dp的思想主要是从头开始遍历，

- 如果该结点不存在（-1），那么就将该结点加入到哈希表中，dp[i] = dp[i - 1]
- 如果该结点存在（非-1），那么看之前的结点的index，如果index不在dp[i-1]的范围内，说明没有将其计算在内，那么还是+1
- 如果index在dp[i-1]范围内，那么说明在index和i上已经出现了重复的结点，那么更新dp到i-index
- 最后，更新最新的i结点的哈希内容

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        HashMap<Character, Integer>store = new HashMap<>();
        int maxLen = 0;
        int[] dp = new int[s.length() + 1];
        dp[0] = 0;
        for(int i = 0; i < s.length(); i++){
            int index = -1;         // 默认没有出现过
            if(store.containsKey(s.charAt(i))) index = store.get(s.charAt(i));      // 如果该值已经出现过，那么找到其索引

            if(i - index > dp[i]) dp[i + 1] = dp[i] + 1;       // 如果结点没有出现过/出现过但是没有在前一个dp范围内
            else dp[i + 1] = i - index;
            store.put(s.charAt(i), i);
            maxLen = Math.max(maxLen, dp[i + 1]);
        }
        return maxLen;
    }
}
```

——两个本质上思想是一样的