# [剑指 Offer 50. 第一个只出现一次的字符](https://leetcode-cn.com/problems/di-yi-ge-zhi-chu-xian-yi-ci-de-zi-fu-lcof/)

<img src="pic\image-20210505180128164.png" alt="image-20210505180128164" style="zoom:67%;" />

用哈希表或者自制的哈希表就可以完成（没有更优的解了）

```java
class Solution {
    public char firstUniqChar(String s) {
        int[] alp = new int[26];
        for(int i = 0; i < s.length(); i++){
            alp[s.charAt(i) - 'a']++;
        }
        for(int i = 0; i < s.length(); i++){
            if(alp[s.charAt(i) - 'a'] == 1) return s.charAt(i);
        }
        return ' ';
    }
}
```

——需要统计次数，且限定在小写/大写字母等条件时，可以用自制的哈希表进行统计。

ps：

https://leetcode-cn.com/problems/di-yi-ge-zhi-chu-xian-yi-ci-de-zi-fu-lcof/solution/mian-shi-ti-50-di-yi-ge-zhi-chu-xian-yi-ci-de-zi-3/

在哈希表的基础上，有序哈希表中的键值对是 **按照插入顺序排序** 的。基于此，可通过遍历有序哈希表，实现搜索首个 “数量为 1的字符”。

# 扩展

在字符流中找出第一个只出现过一次的字符。

eg：当字符流截止到go时，那么结果是g；当截止到google时，是l

可以用**队列+哈希表**（哈希表可以自制，eg：限定在小写字母上时，可以申请一个26大小的数组即可）。

队列上面维护的是按照流顺序仅出现一次的字符，如果查哈希表发现出现不止一次，则不加入队列。——**注意队列只能保证插入时该字符只出现一次。**

当需要求解的时候，获取队列头结点，查其出现的次数，如果次数>1，则出队，查看新的队头结点，直到队列为空 or 找到第一个出现一次的结点。

```java
public class Main {
    private Queue<Character>queue;		// 按照插入顺序将插入时还只出现一次的字符，但是后面可能会重复，所以后面会出队
    private int[]hash;		// 字符出现的次数
    private ArrayList<Character>arrayList;		// 存放字符流
    public Main(){
        queue = new LinkedList<>();
        hash = new int[26];     // 限定在小写字母上
        arrayList = new ArrayList<>();     
    }
    private void addCharacter(char c){
        arrayList.add(c);
        hash[c - 'a']++;		// 出现次数统计
        if(hash[c - 'a'] == 1) queue.offer(c);	// 如果只出现一次，那么将其加入队列
    }
    private char firstAppear(){
        // 将队首出现不止一次的结点都出队（注意，可能不存在只出现一次的字符，所以需要防止队列为空）
        while(!queue.isEmpty() && hash[queue.peek() - 'a'] > 1){
            queue.poll();
        }
        if(queue.isEmpty()){
            return '\0';
        }
        return queue.peek();
    }
    public static void main(String[] args){
        Main main = new Main();
        for (int i = 0; i < 26; i++) {
            main.addCharacter((char) ('a' + i));
            System.out.println(main.firstAppear());
        }
        for (int i = 0; i < 26; i++) {
            main.addCharacter((char)('a' + i));
            System.out.println(main.firstAppear());
        }
    }
}
```

