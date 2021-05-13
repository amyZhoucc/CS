# [剑指 Offer 53 - II. 0～n-1中缺失的数字](https://leetcode-cn.com/problems/que-shi-de-shu-zi-lcof/)

<img src="pic\image-20210508142425600.png" alt="image-20210508142425600" style="zoom:67%;" />

很显然用到了二分。

它实际上也是一个有序的数组，而我们需要找到缺失的那个，所以我们在二分的时候可以将value和index进行比较。

如果value=index，那么说明这个之前是不缺少数字的，那么肯定是这之后，所以left=mid+1;

如果value>index，那么说明这个之前已经发生数字缺少了（也可能是当前数字），那么right=mid-1

```java
class Solution {
    public int missingNumber(int[] nums) {
        int left = 0, right = nums.length - 1;
        while(left <= right){		// 二分查找的标准模板，但是里面的判断条件不一样而已
            int mid = left + (right - left) / 2;
            if(nums[mid] == mid) left = mid + 1;
            else right = mid - 1;
        }
        return left;
    }
}
```

# 扩展

```
数组中数值下标相等的元素：
在单调增的数组中，每个元素都是整数且唯一。请找出里面任意一个下标等于其值的元素。
eg：{-3, -1, 1, 3, 5}，这个3就是下标相等的元素
```

限定是有序的数组，所以肯定考虑用二分，

- 如果mid结点的value和index一致，那么直接返回
- 如果mid结点的value>index，那么说明该之后的结点的value一定都大于index（因为限定的是有序且唯一的），所以需要向前找，right=mid-1
- 如果mid结点的value<index，那么该结点之前的结点的value均小于index，所以需要向后找，left=mid+1

```java
public static void main(String[] args){
    int[]arr = new int[]{-3, 1, 3, 4, 5};
    int left = 0, right = arr.length - 1;

    while(left <= right){
        int mid = left + (right - left) / 2;
        if(arr[mid] == mid){
            System.out.println(mid);
            break;
        }
        else if(arr[mid] < mid) left = mid + 1;
        else right = mid - 1;
    }
}
```

