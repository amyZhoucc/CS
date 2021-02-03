# HashMap和HashSet源码分析

由于HashSet内部实现上，是去创建了HashMap的实例对象，所以合在一起分析。

## HashMap

介绍：hashMap是基于Map的实现，实现了所有Map提供的方法；**允许存储的key是null的，value也可以是null**（hashMap和hashSet基本等效，除了：HashMap是不同步的，且允许为null）

**HashMap不能保证映射的顺序，并且随着时间的推移顺序可能变化**

基本操作：**put/get（通过key的查找）的时间复杂度为O(1)**，前提是，方法能够将元素分散在桶中。

迭代遍历的时间复杂度与桶的大小和key-value对成比例，如果关注迭代性能，那么不要将初始的桶容量设置太大/factor太低（会出现不断扩容的现象）

HashMap对象有两个参数影响性能：初始容量`DEFAULT_INITIAL_CAPACITY`和负载系数`loadFactor`，负载系数，就是当前容量超过：桶容量*负载系数时，就会扩容，哈希表就会被重新映射——内部数据将会重构（本来的顺序可能变化了），一般哈希表的大小是桶容量的2倍——所以设置初始化的容量时，需要考虑预期的条数，用来最小限度减少rehash操作的次数（如果初始时设定的容量 > 最大容量 /负载因子，那么不会发生rehash操作，因为无法再扩容了）。如果需要存储很多键值对，那么提前设定容量，比自动扩容来的效率高

注意，如果许多key有相同的hash值，这会降低性能，那么当key对象实现了Comparable，可以让它们进行比较从而排序

同样实现了迭代器Iterator，采用`fail-fast`机制，即在迭代过程中如果使用外部的remove等方法修改了map的结构，那么就会触发CME异常，除非使用迭代器自带的remove方法（算是陷阱，逻辑上不一样触发异常）。而对并发修改操作，迭代器会直接抛出异常，不会继续执行下去（主要目的）。

——但是，迭代器并不能保证检测出全部的：非同步的并发修改情况，只是尽可能抛出存在的CME异常，因此这个只用来检测bug，而不能用来编写程序通过依赖该抛出的异常来保证程度的正确性

普通的hashmap被称为：桶哈希表（bucketed），当**桶太大的时候会被转换成树节点的哈希表，类似于TreeMap**，方法接口均无变化。对于上层接口感知不到下层数据结构的变化，并且在数量较多的情况下查找会加快。但是树节点的开销是普通节点的2倍（拿空间换时间），所以除非数量超过某个阈值（`TREEIFY_THRESHOLD`）才会触发结构的修改，当树结构下哈希表结构缩小到某个阈值（`UNTREEIFY_THRESHOLD`），又会退回到桶结构。在良好分布的用户hashCode下，树结构很少被使用，

ps：阅读源码过程中会跳过对树结构实现的分析

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
```

HashMap => AbstractMap => Map

### 1. 构造方法

4个构造方法

```java
public HashMap() {			// 默认构造方法
    this.loadFactor = DEFAULT_LOAD_FACTOR; // 所有值都是默认值
}

public HashMap(int initialCapacity, float loadFactor) {		// 初始容量和负载系数都是自定义的
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)		// 超过上限就给上限
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))		// 负载系数要为非负小数，且不能为NaN
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;	
    this.threshold = tableSizeFor(initialCapacity);
}

public HashMap(int initialCapacity) {			// 有设定初始容量，负载系数为默认
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

public HashMap(Map<? extends K, ? extends V> m) {		// 传入一个Map
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
```

### 2. 静态变量

```java
private static final long serialVersionUID = 362498820763181265L;		// 序列化使用

static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // 默认的大小
static final int MAXIMUM_CAPACITY = 1 << 30;		// 最大容量

static final float DEFAULT_LOAD_FACTOR = 0.75f;		// 默认的负载系数，是否扩容的关键

static final int TREEIFY_THRESHOLD = 8;		// 树化的阈值
static final int UNTREEIFY_THRESHOLD = 6;	// 非树化的阈值

static final int MIN_TREEIFY_CAPACITY = 64;		// 树结构下的最小容量

```

ps：负载系数：0.75是一个在时间和空间上的权衡，较高的值会减少空间开销，但是增加查找的时间开销（同一个hash值上挂了较长的链表），较低的值增加空间开销，且查找时间也不会少到哪里去

阈值的计算：capacity * load_factor，当table被占用的数目超过该阈值，就要扩容

静态内部类：

```java
static class Node<K,V> implements Map.Entry<K,V> {	// 主要就是用内部类去实现Map的嵌套接口——方法是：外部接口.内部接口
    final int hash;		// 对应了key的hash值 ——主要是为了加快计算，所以选择直接存储
    final K key;
    V value;
    Node<K,V> next;		// 指向下一个节点（类似于LinkedList的node节点）

    Node(int hash, K key, V value, Node<K,V> next) {		// 构造方法，就是一系列值的初始化
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }	// 输出结果是“key=value"

    public final int hashCode() {			// 获得该节点的哈希值——key的hash值和value的hash值的异或
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {		// 判断两个节点是否一样：类型一样为Map.Entry，且key一样、value一样
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

ps：定义了一个Node类，对应的就是table中的元素（类似于C的结构体，只不过增加了方法），Node类中

### 3. 实例变量

```java
transient Node<K,V>[] table;		// 表，存放的元素是Entry类型
transient Set<Map.Entry<K,V>> entrySet;
transient int size;			// 记录key-value对的个数
transient int modCount;
int threshold;
final float loadFactor;		// 负载系数0~1范围内
```

ps：table：被称为hash表/hash桶

### 4. 静态方法

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

### 5. 实例方法

```java
// 将一个键值对加入到hash表中
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

// 将key的哈希值，
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}


final Node<K,V>[] resize() {		// 调整存储键值对的数组的大小
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;	// 获得当前的数组长度
    int oldThr = threshold;		// 获得当前的阈值
    int newCap, newThr = 0;
    if (oldCap > 0) {			// 存在数组，且存在元素
        if (oldCap >= MAXIMUM_CAPACITY) {		// 如果当前的长度已经超过1<<30（到达了不能再扩容的时候）
            threshold = Integer.MAX_VALUE;		// 阈值更新为int的最大值
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // 扩容为原来的2倍
    }
    else if (oldThr > 0) // 如果数组为空，那么看有无设定阈值，如果有就直接给阈值大小
        newCap = oldThr;
    else {               // 如果数组为空，且没有设定阈值，那么全部给默认的
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```



```java
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();				// 
    if (s > 0) {
        if (table == null) { // pre-size
            float ft = ((float)s / loadFactor) + 1.0F;
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                     (int)ft : MAXIMUM_CAPACITY);
            if (t > threshold)
                threshold = tableSizeFor(t);
        }
        else if (s > threshold)
            resize();
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```



## ps：

### 1. Map接口：

Map在Java中存在，是一个接口，有定义如下方法：

```java
public interface Map<K,V> {		// 类型参数有两个，对应key的类型和value的类型
    int size();			// 查看key-value对的个数
    boolean isEmpty();	// 判断表是否为空
    boolean containsKey(Object key);		// 查看是否包含该键，最多一个
    boolean containsValue(Object value);	// 查看是否包含该值，可能有多个
    V get(Object key);		// 根据key去获取对应的value值
    V put(K key, V value);	// 保存key-value对，如果key存在的话就覆盖
	V remove(Object key);	// 溢出key-value对，按照key移除，并返回原来的key值
    void putAll(Map<? extends K, ? extends V> m);	// 保存所有key-value到当前对象
    void clear();			// 清空key-value对
    boolean equals(Object o);		// 判断这两个表是否一样
    int hashCode();		// 获得该表的hash值
    
    Set<K> keySet();		// 获得表中key的集合
    Collection<V> values();		// 获取表中value的集合
    Set<Map.Entry<K, V>> entrySet();		// 获取表中的key-value对
    
    interface Entry<K,V> {			// 嵌套接口——内置了一个接口（类似于内部类），默认是public类
        K getKey();
        V getValue();
        V setValue(V value);
        boolean equals(Object o);
       	int hashCode();
        public static <K extends Comparable<? super K>, V> 		                                Comparator<Map.Entry<K,V>> comparingByKey() {
                return (Comparator<Map.Entry<K, V>> & Serializable)
                    (c1, c2) -> c1.getKey().compareTo(c2.getKey());
        }
        public static <K, V extends Comparable<? super V>> 				                        Comparator<Map.Entry<K,V>> comparingByValue() {
                return (Comparator<Map.Entry<K, V>> & Serializable)
                   (c1, c2) -> c1.getValue().compareTo(c2.getValue());
        }
    }
    
    // 下面是默认方法：
    default V getOrDefault(Object key, V defaultValue){
        ....
    }
    default void forEach(BiConsumer<? super K, ? super V> action) {
        Objects.requireNonNull(action);
        for (Map.Entry<K, V> entry : entrySet()) {
            K k;
            V v;
            try {
                k = entry.getKey();
                v = entry.getValue();
            } catch(IllegalStateException ise) {
                // this usually means the entry is no longer in the map.
                throw new ConcurrentModificationException(ise);
            }
            action.accept(k, v);
        }
    }
    default boolean remove(Object key, Object value) {
        Object curValue = get(key);
        if (!Objects.equals(curValue, value) ||
            (curValue == null && !containsKey(key))) {
            return false;
        }
        remove(key);
        return true;
    }
    default void replaceAll(BiFunction<? super K, ? super V, ? extends V> function){
        .... 
    }
    default V putIfAbsent(K key, V value){
        ....
    }
    default boolean replace(K key, V oldValue, V newValue) {	
        Object curValue = get(key);
        if (!Objects.equals(curValue, oldValue) ||
            (curValue == null && !containsKey(key))) {
            return false;
        }
        put(key, newValue);
        return true;
    }
    default V replace(K key, V value) {		// 将key对应的值替换为value
        V curValue;
        if (((curValue = get(key)) != null) || containsKey(key)) {
            curValue = put(key, value);
        }
        return curValue;
    }
    default V computeIfAbsent(K key, Function<? super K, ? extends V> mappingFunction) {
        ....
    }
    default V computeIfPresent(K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction){
        ....
    }
    default V compute(K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction){
        ....
    }
    default V merge(K key, V value, BiFunction<? super V, ? super V, ? extends V> remappingFunction){
        ....
    }
}
```

### 2. Set接口

继承自`Collection`，方法都是来自于Collection，本身没有添加方法。

但是**有限定语义：所有实现者，不能存在重复元素**