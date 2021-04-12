# Java集合相关

## 1. List & Set & Map的区别

List：顺序

Set：注重独一无二性

Map：键值对

## 2. ArrayList & LinkedList的异同

1. 都非线程安全
2. ArrayList底层是数组、LinkedList底层是双向链表（不是循环双向链表）
3. ArrayList实现了RandomAcess接口，里面为空，其实就是表示了该类含有的特性，本质上是数组特性实现的随机访问

ArrayList在数组长度不够时，需要扩容，**扩容是1.5倍**，创建一个新的大容量的数组，然后利用`Arrays.copyOf()`将原数组的内容复制过去。

## 3. HashMap

扩容：2倍，初始大小为16，容量一定是2的幂次。

负载因子：0.75，当容量超过大小 * 负载因子时，就会触发扩容

如果遇到重复的key插入，会覆盖之前的，重复的判断：首先看hash值是否一样，一样，然后看是否指向同一个对象；如果不是指向同一个对象，那么调用equals，看是否一样，如果一样，就表示是同一个对象（这边就体现了equals和hashCode匹配重写的必要性）

哈希冲突：拉链法（还有散地址法）

## 4. comparable & comparator区别

如果需要自定义排序的方法，那么需要重写比较方法。

- comparable是java.lang包下面的类都有的一个方法——**compareTo(Object obj)**，如果类将该方法重写了，那么可以调用`compareTo`方法进行比较，在数组排序中，会默认使用该compareTo方法进行比较

- 对于集合，已经不能重写该compareTo方法，那么需要自行定义一个比较器，`comparator`是来自于`java.util`的一个比较器，需要重写里面的`compare(Object obj1, object obj2);`方法——最常见的方法

  `comparator`接口的`compare()`方法：然后调用`Collections.sort(arrayList, comparator)`

  ```java
  Collections.sort(arrayList, new Comparator<Integer>(){
      @Override
      public int compare(Integer obj1, Integer obj2){
          return obj1.compareTo(obj2);
      }
  });
  ```

