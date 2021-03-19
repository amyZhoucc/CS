# Collections源码阅读

是工具类，用来对**集合对象**进行辅助的。地位等同于Arrays。所有方法均是静态方法，可以直接通过`Collections.方法名(xxx)`进行调用。

由于源码比较多，所以只关注一些常见的方法：

## 1. 构造方法

```java
private Collections() {
}
```

——和Arrays一样，都是私有构造方法，用来避免实例化。

## 2. sort

对集合进行排序，默认按照升序排序，列表中所有元素**必须实现Comparable接口**（通过比较进行排序的，所以需要有比较功能）

ps：comparable接口只需要实现**`compareTo`方法**

### 1. 原理

```JAVA
public static <T> void sort(List<T> list, Comparator<? super T> c) {
    list.sort(c);
}
```

本质上还是去调用了list的sort方法，List有一个默认实现，每个List的子类也可以重写该方法：

```java
default void sort(Comparator<? super E> c) {
    Object[] a = this.toArray();
    Arrays.sort(a, (Comparator) c);
    ListIterator<E> i = this.listIterator();
    for (Object e : a) {
        i.next();
        i.set((E) e);
    }
}
```

——sort方法本质上还是去对内部的数组进行排序，并且可以自定义比较器

### 2. 使用

```java
class MyComparator implements Comparator{			// 自定义一个比较器
    @Overrride
    public int comapre(Object o1, Object o2){
        return o1.getValue() - o2.getValue();			// 升序排列
    }
}
Collections.sort(list, new MyComparator());			// 创建比较器实例对象
```

## 3. binarySearch

查找，默认是二分查找，要求数组是有序的

```java
public static <T> int binarySearch(List<? extends Comparable<? super T>> list, T key) {
    if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)	// list要是能够随机查找的（数组等），或者长度没有超过阈值的
        return Collections.indexedBinarySearch(list, key);
    else
        return Collections.iteratorBinarySearch(list, key);
}
```

