# [剑指 Offer 10- I. 斐波那契数列](https://leetcode-cn.com/problems/fei-bo-na-qi-shu-lie-lcof/)

<img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20210302145023494.png" alt="image-20210302145023494" style="zoom: 67%;" />

本来这题是，递归求解的经典题目

```java
class Solution {
    public int fib(int n) {
        if(n == 0 || n == 1) return n;
        return (fib(n - 1) + fib(n - 2)) % 1000000007;
    }
}
```

但是，存在大量的重复计算，eg：f(6) = f(5) + f(4) = f(4) + f(3) + f(3) + f(2) ——这边就有重复计算的内容了，所以会造成超时

所以，我们可以保存那些已经做过计算的值，如果该值出现过就直接使用了

```java
class Solution {
    public int fibHelper(int n, HashMap<Integer, Integer> map){
        if(n < 2) return n;
        if(map.containsKey(n)){
            return map.get(n);
        }
        int first = fibHelper(n - 1, map) % 1000000007;
        map.put(n-1, first);
        int second = fibHelper(n - 2, map) % 1000000007;
        map.put(n-2, second);
        int sum = (first + second) % 1000000007;
        map.put(n, sum);
        return sum;
    }
    public int fib(int n) {
        return fibHelper(n, new HashMap());
    }
}
```

还能使用非递归解法，该解法的思想就是动态规划，已经给定了递推方程式和边界情况，只需要使用即可：

```java
class Solution {
    public int fib(int n) {
        if(n == 0 || n == 1) return n;
        int f = 0, s = 1;
        for(int i = 2; i <= n; i++){
            int temp = (f + s) % 1000000007;
            f = s;
            s = temp;
        }
        return s;
    }
}
```

