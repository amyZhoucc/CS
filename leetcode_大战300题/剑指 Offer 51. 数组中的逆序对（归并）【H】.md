# [剑指 Offer 51. 数组中的逆序对](https://leetcode-cn.com/problems/shu-zu-zhong-de-ni-xu-dui-lcof/)

<img src="pic\image-20210512122816041.png" alt="image-20210512122816041" style="zoom:67%;" />

暴力：肯定就是O(N^2)，就是遍历每个结点，计算每个结点和它之后结点的逆序个数。

本质就是用分治思想。利用归并排序的方法来计算逆序对。——**利用「归并排序」计算逆序对，是非常经典的做法**。

可以发现，如果我们将数组分成两半，那么逆序对存在于：左区间的逆序对 + 右区间的逆序对 + 两个区间之间的逆序对

所以如果区间内的结点是有序的，那么只需要在合并区间的时候，计算区间之间的逆序对即可

<img src="pic\offer51.png" style="zoom: 50%;" >

```java
class Solution {
    private int reversePairs(int[] nums, int left, int right){
        if(left == right) return 0;     // 递归终止条件
        int mid = left + (right - left) / 2;		// 分两半
        int leftPairs = reversePairs(nums, left, mid);
        int rightPairs = reversePairs(nums, mid + 1, right);
        if(nums[mid] <= nums[mid + 1]) return leftPairs + rightPairs;// 如果两部分已经是顺序的了，那么不需要归并了
        return leftPairs + rightPairs + mergeAndCount(nums, left, right, mid);
    }
    private int mergeAndCount(int[] nums, int left, int right, int mid){
        int count = 0;
        int[] temp = new int[right - left + 1];		// 创建一个临时数组，存放有序的数字
        int i = left, j = mid + 1, k = 0;
        while(i <= mid && j <= right){
            if(nums[i] <= nums[j]){
                temp[k++] = nums[i++];
            }
            else{							// 只有后半部分小于前半部分才需要计数
                count += (mid - i + 1);
                temp[k++] = nums[j++];		
            }
        }
        while(i <= mid) temp[k++] = nums[i++];
        while(j <= right) temp[k++] = nums[j++];
        k = 0;
        while(left <= right){
            nums[left++] = temp[k++];
        }
        return count;
    }
    public int reversePairs(int[] nums) {
        if(nums.length < 2) return 0;		// 处理特殊情况
        return reversePairs(nums, 0, nums.length - 1);
    }
}
```

——其实只要能够想到归并排序的基础上计算逆序，至于实现就比较简单了