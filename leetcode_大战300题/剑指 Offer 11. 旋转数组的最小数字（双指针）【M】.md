# [剑指 Offer 11. 旋转数组的最小数字](https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/)

同题154， 是题[153. 寻找旋转排序数组中的最小值](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/)的进阶版

<img src="pic\image-20210302162219759.png" alt="image-20210302162219759" style="zoom:67%;" />



评价：本人觉得这个题难度不小，主要是要考虑特殊情况，如果没有考虑完全，有点像**面向测试的编程**。

心路历程：第一遍写的时候，少考虑了很多问题，一直在出bug，所以放弃没有做完。这次静下心来看这个题，发现抓住几个关键点就能较为正确的编写出代码（但是写完之后，运行发现还是出了问题:weary: ，于是看了[题解](https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/solution/mian-shi-ti-11-xuan-zhuan-shu-zu-de-zui-xiao-shu-3/)，发现前面流程都和大佬想的差不多，后面还是有思维不缜密的地方）

整体方法把握：

- 首先，笨蛋方法：遍历一遍数组，找到最小值，时间复杂度为O(N)——可行，但是没有体现思维价值，所以时间复杂度肯定会优于O(N)
- 概括整个题目是查询数组中的某个值，很自然想到**二分查找**，所以用双指针：left、right

遇到的问题：常规的二分查找都是给定一个target，然后在数组中查找一个符合要求的位置等。但是我们的限制是用二分查找，找最小数字，而最小的数字是一个相对量，并没有一个可以比较的数。 => 将mid指向的值和left/right指向的值比较

1. **那么选择left 还是 right呢？**

   ——需要把握住，由于旋转，所以**正确的解一定出现在后半部分**，且**前半部分的值都 >= 后半部分的值**
   
   而前半部分的值不一定有参考性，即使对于最常规的旋转数组，也有点反思维
   
   如果[1,2,3,4,5]和[3,4,5,1,2]，可以发现mid>left，但是并不能正确区分出最小值的位置
   
   且，由于**存在未旋转的特殊情况**
   
   所以，**选择right指向的值作为判断**
   
   所以，存在情况：
   
   - 当mid的值小于right时，说明最小值还在前面半部分，所以right=mid，**注意，不能right=mid+1，因为显然的，mid可能就是最小值**
   
   - 当mid的值大于right时，说明最小值出现在后半部分，所以left=mid+1，**注意，要left=mid+1，因为显然的，mid肯定不是最小值**
   
   - 当mid的值==right时，我们不能简单的将其划分，因此存在这样的情况：[3,1,3,3,3,3]，如mid==right时，划到后面，那么最小值错过了；如果mid==right时，划到前面[2,2,2,2,1,2]那么最小值还是错过了
   
     ——**所以，当相等时，无法判断最小值在前面还是后面**，所以只能将right排除掉，right-=1，因为既然mid和right相等，如果mid是最小值的话

2. right--的正确性

   x为旋转点，y是right的位置，

   如果x<y，那么y--，旋转点还在，还能找到

   如果x==y，那么y--，旋转点就消失了。但是由于mid的值=y，那么从y到尾巴，从0到mid肯定都是一样的值，那么在[0, y - 1]的范围内是一个单调增的数组，所以一定能找到最小值，该最小值肯定是等于y的。eg：[1,1,1,1,2,3,1]

   [题解](https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/solution/mian-shi-ti-11-xuan-zhuan-shu-zu-de-zui-xiao-shu-3/)分析了这个的可行性，我是通过一个较为具体的例子来理解的

```java
class Solution {
    public int minArray(int[] numbers) {
        int left = 0, right = numbers.length - 1;
        while(left < right){
            int mid = left + (right - left) / 2;
            if(numbers[mid] < numbers[right]) right = mid;	// mid的值小于右边，那么是[left,right]范围太大，导致后半部分有些出现在前面了（且可能mid就是最小值，所以只能right=mid，而不是right=mid-1）
            else if(numbers[mid] > numbers[right]) left = mid + 1;	// mid值大于右边，那么是前半部分有些出现在后面了，且mid肯定不是最小值，所以left=mid+1
            else right--;		// 如果相等，并不能
        }
        return numbers[left];
    }
}
```

参考：https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/solution/mian-shi-ti-11-xuan-zhuan-shu-zu-de-zui-xiao-shu-3/

——感觉，这个题就是不断地举出反例，然后选择另一条路走：

先确定是可以用二分的，接下来就是选择比较的基准right，然后比较的时候，会出现三种情况：大于、小于、等于，问题就出现在等于，然后继续举反例，最后发现right--是可以的。