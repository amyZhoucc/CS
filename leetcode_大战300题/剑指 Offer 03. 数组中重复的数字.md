# [剑指 Offer 03. 数组中重复的数字](https://leetcode-cn.com/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof/)

关键词：【数组】【哈希】【二分】

一刷：使用数组

二刷：数组 / 哈希

三刷：数组

## 1. 题目描述

找出数组中重复的数字。


在一个长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。

示例 1：

```
输入：
[2, 3, 1, 0, 2, 5, 3]
输出：2 或 3 
```


限制：

```
2 <= n <= 100000
```

## 2. 题目分析

心路历程：被0~n-1范围吸引，想起了leetcode其中一题，**使用了”一个萝卜一个坑“的方法**，但是这题只是限制数字范围在0~n-1，而重复的数字个数和重复的次数都未定，所以不满足。

- 最复杂的方法就是一次排序，时间复杂度为O(NlogN)，排序然后遍历一遍，找到第一个重复的就可以，

  ——那么时间复杂度为$O(NlogN)+O(N)$，空间复杂度为$O(1)$

  ——这个很显然是暴力算法，所以时间复杂度一定会比这个低

- 所以能想到，用空间换时间，利用哈希表/额外数组来解决

  ——时间复杂度为O(N)，空间复杂度为O(N)

- 但是最优的方法——可能不是马上能想到的，其实也是利用了数组的特性，**快速索引**

  可以发现，我们**没有利用限制条件”值的范围在0~n-1“**（就是不知道该如何用）——那如果我们利用起来，空间复杂度一定还可以降（时间复杂度是不能降了，不可能是O(1)复杂度完成的）

  **重点**：

  其实也是利用了**”一个萝卜一个坑“**的思路，我们可以看一下该题的反面：数组中没有出现重复数字，那么n长度的数组里面的元素一定是0~n-1每个数组有且出现一次，且最终都能整理成：index下对应的就是同值的value，[详见](#corner)

  那么现在出现重复，就是**一个index下对应的同值value出现重复**——其实就是哈希冲突

  所以思路是：重排数组：遍历数组，将当前index下的value和对应index下的值进行交换，那么该index的值就是对应的了，然后不断重复，直到当前index找到对应的value，那如果在交换过程中，出现了冲突，那么就是重复数字了

  eg：[0, 4, 3, 2, 4, 5]，index=1，value=4，发现 index 和 value 不匹配，那么将 value=4 交换到 index=4 处，而发现 index=4，value=4，已经匹配了，说明出现了冲突——存在两个4

  ——时间复杂度为O(N)，空间复杂度为O(1)

## 3. 解法

#### （1）暴力解

```java
class Solution {
    public int findRepeatNumber(int[] nums) {
        Arrays.sort(nums);
        int prev = nums[0];
        for(int i = 1; i < nums.length; i++){
            if(prev == nums[i]) return nums[i];
            prev = nums[i];
        }
        return -1;
    }
}
```

——这个暴力解没啥意义（但是能保证找到最小重复出现的数字，最大重复出现的也可以）

#### （1）哈希表/数组

```java
class Solution {
    public int findRepeatNumber(int[] nums) {
        HashMap<Integer, Integer> map = new HashMap<>();        // 创建哈希表进行存储（set也可以）
        for(int i = 0; i < nums.length; i++){
            if(map.containsKey(nums[i])) return nums[i];
            else map.put(nums[i], 1);
        }
        return -1;
    }
}

class Solution {
    public int findRepeatNumber(int[] nums) {
        HashSet<Integer> set = new HashSet<>();
        for(int i = 0; i < nums.length; i++){
            if(set.contains(nums[i])) return nums[i];
            else set.add(nums[i]);
        }
        return -1;
    }
}
```

数组：

```java
class Solution {
    public int findRepeatNumber(int[] nums) {
        int[] arr = new int[nums.length];           // 额外数组，记录index对应的value出现的次数
        for(int i = 0; i < nums.length; i++){
            if(arr[nums[i]] == 1) return nums[i];       // 标记已经出现过了
            else
                arr[nums[i]] = 1;
        }
        return -1;          // 不存在重复数字
    }
}
```

——最基础，应该马上想到的（但是能保证找到第一个重复数字的下标位置）

#### （2）原地排序

```java
class Solution {
    public int findRepeatNumber(int[] nums) {
        for(int i = 0; i < nums.length; i++){
            int temp;
            while(nums[i] != i){			// 一个萝卜一个坑操作
                if(nums[i] == nums[nums[i]]) return nums[i]; // 出现了重复数字
                temp = nums[i];
                nums[i] = nums[temp];
                nums[temp] = temp;
            }
        }
        return -1;
    }
}
```

——需要绕弯的，最为精巧，实践运用了哈希冲突（有点像Java底层的哈希表的实现思路），和数组的index和value的关系

但是不能找到最小重复值，也不能找到最小重复index值

## 4. 收获

1. **”一个萝卜一个坑“**思路：<a name ="corner"></a>

   n长度的数组，限制数组值的范围在$[0, n-1]$，有且出现一次，如何将其变得有序？

   可以发现，[0, 4, 2, 5, 3, 1] - > [0, 1, 2, 3, 4, 5]——对应的就是其索引下标

   那么只需要遍历一遍数组，将和自己下标不对应的数字都放到对应的索引里面，直到找到适合当前索引的值，然后找下一个.....

   ```java
   int[]nums = {4, 3, 5, 1, 2, 0};
   for(int i = 0; i < nums.length; i++){
       int temp;
       while(nums[i] != i){
           temp = nums[i];
           nums[i] = nums[temp];
           nums[temp] = temp;
       }
   }
   System.out.println(Arrays.toString(nums));
   ```

   ——使用场景：**在限制数组值范围在$[0, n-1]$，可以考虑是否能够使用**

   ps：单纯这样的场景，这样做是没有必要的，直接赋值更为简单

2. 可以尝试考虑该题目的反面情况——不存在重复是何种样子，如果出现重复打破了什么

## 5. 扩展

不修改数组，找到重复的数字

```
在一个长度为n+l的数组里的所有数字都在1~n的范围内，所以数组中至少有一个重复的数字，请找到其中一个重复的数字，但不能修改输入的数组。
例如，如果输入长度为8的数组{2, 3, 5, 4, 3, 2, 6, 7}, 那么对应的输出是重复的数字2或者3。
```

不能修改数组，且空间复杂度为O(1)，

所以不能使用上面的“一个萝卜一个坑”，或者哈希集合。而采用**二分查找**

查找范围：[0, n -1]，每次都是遍历数组，统计前半部分数字出现的次数，如果超过范围的个数，那么说明该部分出现重复数字；如果没有超过，则重复数字出现在后半部分

但是还是存在问题：[0, 0, 1, 2, 4, 5, 6, 7, 8 ,9]，在第一次二分的时候就没法找到正确的解。

所以我稍微优化了一下，每次在统计前半部分出现数字出现次数的同时，也进行加和，那么如果遇到上面的问题，也还是可以发现的

```java
class Solution {
    public int findRepeatNumber(int[] nums) {       // 二分
        int left = 0;           
        int right = nums.length - 1;    
        while(left < right){
            int mid = (right - left) / 2 + left;
            int count = 0;
            int sum = 0;
            for(int i = 0; i < nums.length; i++){
                if(nums[i] <= mid && nums[i] >= left){
                    count++;
                    sum += nums[i];		// 求和
                }
            }
            if(count > (mid - left + 1)) right = mid;
            else if(count == (mid - left + 1)){
                if(sum == (mid + left) * (mid - left + 1) / 2) left = mid + 1;
                else right = mid;
            }
            else left = mid + 1;
        }
        return left;
    }
}
```

# [442. 数组中重复的数据](https://leetcode-cn.com/problems/find-all-duplicates-in-an-array/)

<img src="pic\image-20210509164326403.png" alt="image-20210509164326403" style="zoom:67%;" />

这个题目是要求，找到数组中所有重复的数据。

最优的解法：还是一个萝卜一个坑。

需要注意的是：如果发现该数据已经重复了，那么不能再交换了，需要将该数组清除为-1，而该点的交换应该终止；易错点：从1开始的，那么对应的index应该向前移动。

```java
class Solution {
    public List<Integer> findDuplicates(int[] nums) {
        ArrayList<Integer>res = new ArrayList<>();
        for(int i = 0; i < nums.length; i++){
            while(nums[i] != i + 1 && nums[i] != -1){		// 如果当前点变成-1，那么跳过
                if(nums[i] == nums[nums[i] - 1]){		// 对应位置上已经发现了解，那么一定出现重复了
                    res.add(nums[i]);
                    nums[i] = -1;
                    break;
                }
                int temp = nums[i];
                nums[i] = nums[temp - 1];
                nums[temp - 1] = temp;			// 注意交换的数组下标
            }
        }
        return res;
    }
}
```

