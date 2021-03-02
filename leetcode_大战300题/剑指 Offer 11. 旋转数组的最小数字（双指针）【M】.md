# [剑指 Offer 11. 旋转数组的最小数字](https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/)

同题154， 是题[153. 寻找旋转排序数组中的最小值](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/)的进阶版

<img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20210302162219759.png" alt="image-20210302162219759" style="zoom:67%;" />



评价：本人觉得这个题难度不小，主要是要考虑特殊情况，如果没有考虑完全，有点像**面向测试的编程**。

心路历程：第一遍写的时候，少考虑了很多问题，一直在出bug，所以放弃没有做完。这次静下心来看这个题，发现抓住几个关键点就能较为正确的编写出代码（但是写完之后，运行发现还是出了问题:weary: ，于是看了[题解](https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/solution/mian-shi-ti-11-xuan-zhuan-shu-zu-de-zui-xiao-shu-3/)，发现前面流程都和大佬想的差不多，后面还是有思维不缜密的地方）

整体方法把握：

- 首先，笨蛋方法：遍历一遍数组，找到最小值，时间复杂度为O(N)——可行，但是没有体现思维价值
- 概括整个题目是查询数组中的某个值，很自然想到**二分查找**，所以用双指针：left、right

遇到的问题：常规的二分查找都是给定一个target，然后在数组中查找一个符合要求的位置等。但是我们的限制是用二分查找，找最小数字，而最小的数字是一个相对量，并没有一个可以比较的数。 => 将mid指向的值和left/right指向的值比较

**那么选择left 还是 right呢？**

——需要把握住，由于旋转，所以**正确的解一定出现在后半部分**，且**前半部分的值都 >= 后半部分的值**

而前半部分的值不一定有参考性。如果mid的值>left的值，并不能说明解一定在[mid, left]范围内；还是[mid, right]范围内，eg：[3,4,5,1,2]，同理mid的值<left的值，也无法做出判断。

所以，**选择right指向的值作为判断**



参考：https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/solution/mian-shi-ti-11-xuan-zhuan-shu-zu-de-zui-xiao-shu-3/