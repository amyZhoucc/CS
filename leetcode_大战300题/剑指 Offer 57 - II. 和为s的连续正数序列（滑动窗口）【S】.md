# [剑指 Offer 57 - II. 和为s的连续正数序列](https://leetcode-cn.com/problems/he-wei-sde-lian-xu-zheng-shu-xu-lie-lcof/)

<img src="pic\image-20210512160210501.png" alt="image-20210512160210501" style="zoom:67%;" />

由于它是需要连续的的序列，所以很适合用滑动窗口

## 滑动窗口

**如果当前窗口内的sum>target，那么左边窗口右移；如果sum<target，那么右边窗口右移；如果sum==target，那么就是目标，将其获取出来，并且左边窗口右移。**

虽然整个思想能够理解，但是要写出来还是很难的。

这边设定的是[left, right)

```java
class Solution {
    public int[][] findContinuousSequence(int target) {
        if(target < 2) return new int[0][0];
        List<int[]>res = new ArrayList<>();
        int i = 1;
        int j = 1;		// 此时区间为空[1, 1)
        int sum = 0;
        while(i <= target / 2){		// i超过一半就没有意义了
            if(sum < target){		// 如果未达到目标，那么加上当前的j（本来不算在内的）
                sum += j;
                j++;
            }
            else if(sum > target){	// 如果超过了，那么减去当前的i（现在不算在内了，那么i就要向后移动一个）
                sum -= i;
                i++;
            }
            else{
                int[] temp = new int[j - i];
                int l = 0;
                for(int k = i; k < j; k++)temp[l++] = k;
                res.add(temp);
                sum -= i;
                i++;
            }
        }
        return res.toArray(new int[res.size()][]);
    }
}
```

正确性证明：

1.  1+2+⋯+8 < target，再加上一个 9 之后， 发现 1+2+⋯+8+9> target 了，那么说明以1起始的序列已经找不到正解了
2. 1移动到2，那么必然有2+⋯+8 < target，所以一定要2+...+9，如果2+...9>target，那么说明2起始的序列也找不到正解；需要再次移动做窗口向右；如果2+....+10<target，那么右窗口还需要右移动

——所以，左右窗口均只需要向右移动，且不会丢失需要的解。