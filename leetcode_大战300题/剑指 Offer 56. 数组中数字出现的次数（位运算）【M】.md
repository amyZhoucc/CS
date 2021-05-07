# [剑指 Offer 56 - I. 数组中数字出现的次数](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/)

<img src="pic\image-20210506203520943.png" alt="image-20210506203520943" style="zoom:67%;" />

印象最深的是，找出有且仅有一个的出现1次的数字，直接异或一轮就完事了。

但是本题，是限定有两个仅出现一次的数字，其他出现两次，那么简单异或就只能得到这两个数字的混合物。我能想到要将数组划分，划分成两组：一组包含重复数字+不重复数字1，一组包含重复数字+不重复数字2，则它们内部进行异或，就能分别得到不重复数字，但是问题是**如何将数组划分**——**利用异或得到值，两个数字值不同，那么肯定存在位不同，找到该不同的位，再次进行遍历异或，异或时先和该位进行与运算**

- 如果结果为1，则与a进行异或
- 如果结果为0，则与b进行异或

出现两次的数字，会两次进入同一个判断，异或之后就是0；出现一次的数字只会进入其中一个

```java
class Solution {
    public int[] singleNumbers(int[] nums) {
        int mix = 0;
        for(int num : nums){
            mix ^= num;
        }                       // 得到两个只出现一次的数值的异或值
        int radix = 1;
        while((mix & radix) == 0){      // 找到第一个不是0的位——用来区分两个数
            radix <<= 1;
        }
        int a = 0, b = 0;
        for(int num : nums){
            if((num & radix) == 0){    // 出现2次的数，如果成立，那么一定会进入两次；出现1次的数，只会进入一次
                a ^= num;
            }
            else{
                b ^= num;
            }
        }
        int[]res = new int[2];
        res[0] = a;
        res[1] = b;
        return res;   
    }
}
```

——最关键是：两个数字不同，则异或后一定存在一个位为1，找到该1的位置就能区分两个数



# [剑指 Offer 56 - II. 数组中数字出现的次数 II](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-ii-lcof/)

<img src="pic\image-20210506205227705.png" alt="image-20210506205227705" style="zoom:67%;" />

这个又是另外一个解法，对于其他数字均出现相同次数，而只有一个数字只出现一次，那么可以用位运算

就是将所有数字的位提取出来，统计对应的每个位的出现次数，构成一个32位长度的数组。如果一个数字出现3次，那么对应的位一定出现了3次，所以如果剔除只出现一次的数字，那么该数组的每位出现次数都是3的倍数，如果对应的位不是3的倍数，那么就是待求的数字对应的位为1。

```java
class Solution {
    public int singleNumber(int[] nums) {
        int[]bit = new int[32];
        for(int num : nums){
            int temp = num;
            int i = 0;
            while(temp != 0){           // 提取每个数字的每一位，如果为1，对应的bit++
                bit[i++] += (temp & 1);
                temp >>= 1;
            }
        }
        int res = 0, radix = 1;
        for(int i = 0; i < 32; i++){
            if(bit[i] % 3 != 0) res += radix;
            radix <<= 1;
        }
        return res;
    }
}
```

——关键：发现每个位出现的次数都是3的倍数