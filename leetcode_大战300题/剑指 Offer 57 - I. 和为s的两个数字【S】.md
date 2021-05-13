# [剑指 Offer 57. 和为s的两个数字](https://leetcode-cn.com/problems/he-wei-sde-liang-ge-shu-zi-lcof/)

<img src="pic\image-20210506210122253.png" alt="image-20210506210122253" style="zoom:67%;" />

这是一个有序数组，所以肯定考虑使用二分法或者双指针。

这个是使用双指针（从三数之和上的思路），初始指向数组的首和尾，如果和>0，那么说明值太大，right--；如果和<0，那么说明值太小，left++。

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        int left = 0, right = nums.length - 1;
        while(left < right){
            if(nums[left] + nums[right] == target) return new int[]{nums[left], nums[right]};
            else if(nums[left] +  nums[right] > target) right--;
            else left++;
        }
        return new int[2];
    }
}
```

正确性证明：

初始left=0,right=num.length-1，如果和<target，那么left++，因为right已经最大，所以旧left下，从right=[left+1，right]都不可能找到正确的解。

right的推理，如果和>target，那么right--，因为left已经最小，所以在旧right下，从left=[left, right-1]都不可能找到正确的解

——所以删除不会丢失正确值。