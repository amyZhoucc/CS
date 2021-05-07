# [剑指 Offer 61. 扑克牌中的顺子](https://leetcode-cn.com/problems/bu-ke-pai-zhong-de-shun-zi-lcof/)

<img src="pic\image-20210505205853184.png" alt="image-20210505205853184" style="zoom:67%;" />

首先要吐槽：不会打牌我连题目都不会做。

总结来说：大小王=0，就是用来作弊的，当出现不连续的时候，就可以用0补上，当连续的差距很大时0不够时，就表明不是连续的

且牌不能重复。

## 排序+模拟

我的解法：将数组先排序（因为数组长度固定，所以时间复杂度不会差很大），然后统计0的个数，然后从非0的开始计算他们之间的差距，如果不连续则尝试用0补，如果补不上了，即0不够用了，就返回false。

并且在遍历的过程中，看到重复的就直接返回false

```java
class Solution {
    public boolean isStraight(int[] nums) {
        Arrays.sort(nums);
        int i = 0;
        while(i < 5 && nums[i] == 0) i++;
        int zero = i;           // 0的个数
        if(i == 5) return true;
        for(; i < 4; i++){
            if(nums[i + 1] == nums[i]) return false;
            if(nums[i + 1] - nums[i] > 1) {
                zero -= (nums[i + 1] - nums[i] - 1);
                if(zero < 0) return false;
            }
        }
        return true;
    }
}
```

## 技巧

- 如果没有大小王，那么要求都要是连续的，所以最大值和最小值差要小于等于4
- 如果有大小王：1个——最大值和最小值（除了0），小于等于4；2——小于等于4....

——所以不论如何，都是小于等于4

而又有另一个限制条件：不能有重复（除了大小王），那么还需要遍历时判断是否重复

```java
class Solution {
    public boolean isStraight(int[] nums) {
        Set<Integer> repeat = new HashSet<>();
        int max = 0, min = 14;
        for(int num : nums) {
            if(num == 0) continue; // 跳过大小王
            max = Math.max(max, num); // 最大牌
            min = Math.min(min, num); // 最小牌
            if(repeat.contains(num)) return false; // 若有重复，提前返回 false
            repeat.add(num); // 添加此牌至 Set
        }
        return max - min < 5; // 最大牌 - 最小牌 < 5 则可构成顺子
    }
}
```

参考：

https://leetcode-cn.com/problems/bu-ke-pai-zhong-de-shun-zi-lcof/solution/mian-shi-ti-61-bu-ke-pai-zhong-de-shun-zi-ji-he-se/

——记住结论