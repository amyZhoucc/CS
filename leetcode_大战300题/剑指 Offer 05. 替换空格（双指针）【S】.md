# [剑指 Offer 05. 替换空格](https://leetcode-cn.com/problems/ti-huan-kong-ge-lcof/)

（简单题目，还是有深度的）

<img src="pic\image-20210502100125107.png" alt="image-20210502100125107" style="zoom:67%;" />

本题对于Java来说比较简单，因为Java限制了字符串是final类型，即不可变的，所以如果要修改字符串本质上就是要创建一个新的字符串。所以很显然用`StringBuilder`类去创建一个可变的字符串，然后从头到尾将旧字符串复制过去，并且将空格替换即可。

但是，对于c++来说，String是可变类型的，所以：

- 我们需要知道新的字符串的长度——一次遍历，求空格数

- 不能从头开始修改字符串，那么每次修改后面的内容全部需要移动，时间复杂度为

  $O(N^2)$——从尾巴开始替换，双指针——p1指向新数组（实际上是同一个）的尾部，p2指向旧数组（实际上是同一个）的尾部，然后p2遍历

  - 如果是普通字符，那么p1内容添加，p1--，p2--
  - 如果是空格，那么p1内容添加，p1-=3，p2--

     ——这样O(N)时间复杂度即可完成替换

```java
class Solution {
    public String replaceSpace(String s) {
        StringBuilder sb = new StringBuilder();
        for(int i = 0; i < s.length(); i++){
            if(s.charAt(i) == ' '){
                sb.append("%20");
            }
            else{
                sb.append(s.charAt(i));
            }
        }
        return sb.toString();
    }
}
```

运用双指针的解法：

```java
class Solution {
    public String replaceSpace(String s) {
        int len = s.length();
        int count = 0;
        for(int i = len - 1; i >= 0; i--){      // 遍历统计空格出现的次数
            if(s.charAt(i) == ' ') count++;
        }
        char[] arr = new char[len + count * 2];
        int j = len - 1;
        for(int i = arr.length - 1; i >= 0; i--){
            if(s.charAt(j) == ' '){
                arr[i--] = '0';
                arr[i--] = '2';
                arr[i] = '%';
                j--;
            }
            else{
                arr[i] = s.charAt(j--);
            }
        }
        return new String(arr);
    }
}
```

参考：

https://leetcode-cn.com/problems/ti-huan-kong-ge-lcof/solution/mian-shi-ti-05-ti-huan-kong-ge-ji-jian-qing-xi-tu-/

c++的解法：——思想的参考

```c++
class Solution {
public:
    string replaceSpace(string s) {
        int count = 0, len = s.size();
        // 统计空格数量
        for (char c : s) {
            if (c == ' ') count++;
        }
        // 修改 s 长度
        s.resize(len + 2 * count);
        // 倒序遍历修改
        for(int i = len - 1, j = s.size() - 1; i < j; i--, j--) {
            if (s[i] != ' ')
                s[j] = s[i];
            else {
                s[j - 2] = '%';
                s[j - 1] = '2';
                s[j] = '0';
                j -= 2;
            }
        }
        return s;
    }
};
```

# 扩展：

```
两个数组A1、A2有序，而A1后面有足够的空间容纳A2，现在需要将A2插入到A1中，且保持A1的整个有序状态
```

我们之前碰到的是，有序的A1、A2复制到一个新的数组里面，那么用双指针，从头开始比较，将较小的数插入到新数组中，i++。

但是现在的问题是，原地修改数组，所以不能从头开始插入，会涉及到数组后面的频繁移动

——**从尾巴开始遍历**，更大的数放在最后面，需要用到3个指针（A1尾巴指针p1，A1旧数组尾巴指针p2，A2旧数组尾巴指针p3）

<img src="pic\image-20210515135259194.png" alt="image-20210515135259194" style="zoom:67%;" />

```java
public class Solution {
    public void merge(int A[], int m, int B[], int n) {
        int pA = m - 1, pB = n - 1, p = m + n - 1;
        while(pA >= 0 && pB >= 0){
            if(A[pA] >= B[pB]) A[p--] = A[pA--];
            else A[p--] = B[pB--];
        }
        while(pA >= 0) A[p--] = A[pA--];
        while(pB >= 0) A[p--] = B[pB--];
    }
}
```

