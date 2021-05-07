# [剑指 Offer 39. 数组中出现次数超过一半的数字](https://leetcode-cn.com/problems/shu-zu-zhong-chu-xian-ci-shu-chao-guo-yi-ban-de-shu-zi-lcof/)

<img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20210506233415434.png" alt="image-20210506233415434" style="zoom:67%;" />

有3种方法可以实现：

## 哈希表

将所有数字都存储在哈希表中，然后遍历哈希表，看其值超过一半的就是众数

```java
class Solution {
    public int majorityElement(int[] nums) {
        HashMap<Integer, Integer>store = new HashMap<>();
        for(int num: nums){
            if(store.containsKey(num)){
                store.put(num, store.get(num) + 1);
            }
            else{
                store.put(num, 1);
            }
        }
        int mid = nums.length / 2;
        for(Map.Entry<Integer, Integer>entry: store.entrySet()){
            if(entry.getValue() > mid) return entry.getKey();
        }
        return -1;
    }
}
```

## 排序

将数组排序，然后位于中间的数一定是众数。

```java
class Solution {
    public int majorityElement(int[] nums) {
        Arrays.sort(nums);
        return nums[nums.length / 2];
    }
}
```

## 摩尔投票

本质就是人多势众：一对一抵消，那么最后留下的一定是众数。

思路：默认第一个数是众数，然后开始遍历，如果后面一个数相等（也是众数），则投票++；如果不相等（不是众数），则投票--。如果投票数==0，那么当前数成为众数，且票数++。

```java
class Solution {
    public int majorityElement(int[] nums) {
        int vote = 0, maj = 0;
        for(int num : nums){
            if(vote == 0) maj = num;		// 之前全部抵消了，当前数成为新众数
            vote += num == maj ? 1 : -1;
        }
        return prev;
    }
}
```

