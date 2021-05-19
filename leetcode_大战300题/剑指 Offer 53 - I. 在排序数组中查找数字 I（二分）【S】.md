# [剑指 Offer 53 - I. 在排序数组中查找数字 I](https://leetcode-cn.com/problems/zai-pai-xu-shu-zu-zhong-cha-zhao-shu-zi-lcof/)

<img src="pic\image-20210508104501935.png" alt="image-20210508104501935" style="zoom: 67%;" />

本题的暴力解法：O(N)，遍历一遍数组，找到第一个匹配的结点，然后找到最后一个匹配的结点，差值就是出现次数。

所以比这个更优的肯定是时间复杂度要比O(N)优的，而这是一个排序数组，所以可以想到用二分（双指针不行）

所以：关键点是如何对普通的二分查找进行改良，并且处理重复的数据。

我们可以看到，普通的二分：对于找到target后，立即返回；而如果数组中没有找到该结点，那么返回时**left指向的就是该target的合适插入位置**

1. 如果是重复的数据，我们肯定要选择最后一个结点，所以`mid=target时，left=mid+1`——那么能够找到指定目标的最后一个的下一个，即第一个不是该数据的位置
2. 由于前面找的是数据的最后一个，那么我们可以找target-1，如果该数据存在，那么找到的就是target-1的最后一个的下一个，就是target的第一个；如果该数据不存在，那么找到的就是它适合插入的位置，就是target结点的第一个

```java
class Solution {
    private int findIndex(int[] nums, int target){
        int left = 0, right = nums.length - 1;
        while(left <= right){
            int mid = left + (right - left) / 2;
            if(nums[mid] <= target) left = mid + 1;
            else right = mid - 1;
        }
        return left;		// 找到的是target的最后一个结点的后一个
    }
    public int search(int[] nums, int target) {
        // 找到target结点之后的第一个结点 - 找到target的第一个结点=长度
        return findIndex(nums, target) - findIndex(nums, target - 1);
    }
}
```

附上：

常见的二分解法：

```java
int[] res = new int[]{1, 4, 6, 7, 8, 9, 10, 12, 14};
int target = 5;
int left = 0, right = res.length - 1;
while(left <= right){				// 注意边界条件是：left<=right，即left>right才会跳出循环
    int mid = left + (right - left) / 2;
    if(res[mid] == target){
        System.out.println(mid);
        break;
    }
    else if(res[mid] > target) right = mid - 1;
    else left = mid + 1;
}
if(left > right) System.out.println(left);		// 如果没有找到，left指向就是target适合插入的位置
```

# 牛客网上的题

[二分查找II](https://www.nowcoder.com/practice/4f470d1d3b734f8aaf2afb014185b395?tpId=188&tags=&title=&diffculty=0&judgeStatus=0&rp=1&tab=answerKey)

<img src="pic\image-20210514203735064.png" alt="image-20210514203735064" style="zoom: 50%;" />

<img src="pic\image-20210514203757335.png" alt="image-20210514203757335" style="zoom: 50%;" />

主要用到了上面的思想，主要就是查找target-1的数的最后一次出现位置的下一个，那么就是target结点第一次出现的位置，如果未找到，**left就是第一个适合插入的位置**

所以在得到left之后我们需要判断，如果left对应的值是和target一样的，那么返回target；如果不一样，那么说明该结点不存在，返回-1；但是需要注意**left结果可能是num.length，即适合插入在数组最后，所以我们需要获取下标之前进行判断**。

```java
import java.util.*;

public class Solution {
    /**
     * 代码中的类名、方法名、参数名已经指定，请勿修改，直接返回方法规定的值即可
     *
     * 如果目标值存在返回下标，否则返回 -1
     * @param nums int整型一维数组 
     * @param target int整型 
     * @return int整型
     */
    private int binarySearch(int[] nums, int target){
        int left = 0, right = nums.length - 1;
        while(left <= right){
            int mid = left + (right - left) / 2;
            if(nums[mid] > target) right = mid - 1;
            else if(nums[mid] < target) left = mid + 1;
            else left = mid + 1;		// 相等，则left向右移动指向下一个
        }
        return left;
    }
    public int search (int[] nums, int target) {
        // write code here
        int res = binarySearch(nums, target - 1);
        if(res == nums.length || target != nums[res]) return -1;
        else return res;
    }
}
```

