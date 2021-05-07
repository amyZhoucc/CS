# [剑指 Offer 45. 把数组排成最小的数](https://leetcode-cn.com/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/)

<img src="pic\image-20210506163301558.png" alt="image-20210506163301558" style="zoom:67%;" />

分析该题目，可以知道要求最小，则最高位要是所有数字里面最小的，即有0的尽量用0，有1的尽量用1。其实本质上就是一个排序问题，但是不能用内置的排序算法，因为对于数组来说，位数多的自然就比位数少的要大，而我们其实是不关注位数，只关注每个位上的大小值，所以能够自然想到**数字全部转化成字符串**

但是**排序我们需要自定义**：因为存在"3" < "32"，所以会组合成"332"，但是正确的是"323"。

所以，我们可以按照s1 + s2 和s2+s1的排序进行比较

```java
class Solution {
    public String minNumber(int[] nums) {
        String[] s = new String[nums.length];
        for(int i = 0; i < nums.length; i++){
            s[i] = Integer.toString(nums[i]);
        }
        Arrays.sort(s, new Comparator<String>(){		// 自定义比较器
            public int compare(String o1, String o2){
                return (o1 + o2).compareTo((o2 + o1));
            }
        });
        StringBuilder sb = new StringBuilder();
        for(int i = 0; i < s.length; i++){
            sb.append(s[i]);
        }
        return sb.toString();
    }
}
```

——然后我们只需要自定义比较器即可。

当然，我们可以自己写快排：——会更优一点。

```java
private boolean judge(String x, String y){          // < 返回true，>=返回false
    StringBuilder sb1 = new StringBuilder();
    StringBuilder sb2 = new StringBuilder();
    sb1.append(x);
    sb1.append(y);
    sb2.append(y);
    sb2.append(x);
    for(int i = 0; i < sb1.length(); i++){		// 手动比较（StringBuilder没有compareTo方法，要转换成String）
        if(sb1.charAt(i) < sb2.charAt(i)) return true;
        else if(sb1.charAt(i) > sb2.charAt(i)) return false;	// 一旦出现不一样就直接返回了
    }
    return true;
}
private void sort(String[]s, int l, int r){
    if(l >= r) return;       // 递归终止条件
    int left = l, right = r;
    String pivot = s[left];        // 比较结点
    while(left < right){
        while(left < right && judge(pivot, s[right])) right--;
        if(left < right) s[left++] = s[right];
        while(left < right && judge(s[left], pivot)) left++;
        if(left < right) s[right--] = s[left];
    }
    s[left] = pivot;
    sort(s, l, left - 1);
    sort(s, left + 1, r);
}
```

