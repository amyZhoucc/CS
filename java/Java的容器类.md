# Java的容器类汇总

由于容器类在开发过程中太过于常见，所以单独设为一篇进行整理

每个容器类主要按照一下流程进行了解：基本用法、基本原理、迭代操作、实现的接口、特点

目前了解到的容器类有：

- 列表：ArrayList、LinkedList、ArrayDeque
- 集合：HashMap、HashSet、TreeMap、TreeSet、LinkedHashMap、LinkedHashSet、EnumMap、EnumSet

Collection作为容器类的超类，继承它称为它的子类的有：Set、List、Map、SortSet、SortMap、HashSet、TreeSet、ArrayList、LinkedList、Vector、Collections、Arrays、AbstractCollection

## 一、ArrayList

**ArrayList：动态数组容器类**，对标的是普通的数组（普通的数组长度是固定的，不能变化，如果长度不够需要重新建一个数组，将原来的数组复制过去）

首先看一下类头：

```Java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable{...}
```

### （一）使用

首先发现是一个**泛型类**，**新建ArrayList需要实例化类型参数**

```Java
ArrayList<Integer> arr = new ArrayList<Integer>();	// 右边的Integer可以省略

// 由于实现了List接口，所以可以如下定义：
List<Integer> arr = new ArrayList<Integer>();
List<Integer> arr = new ArrayList<Integer>(5);		// 创建一个长度为5的数组
List<Integer> arr = new ArrayList<Integer>(Arrays.asList(new Integer[]{1, 2, 3}));
```

——可以发现ArrayList数组中的元素是引用类型的，不能是基本数据类型（其实在add的时候会默认转换）

主要方法有：

```Java
public int size();		// 获取当前数组长度 arr.size();——注意和普通数组的区分a.length
public boolean isEmpty();	// 判断数组是否为空 arr.isEmpty();

public boolean contains(Object o);		// 判断数组中是否包含某个对象 arr.contains(o);——注意很多时候用到是直接arr.contains(1);实际上是编译器自动将int类型打包成Integer类型
public int indexOf(Object o);		// 获得该对象首次在数组中的索引值 arr.indexOf(o); 如果找不到就返回-1
public int lastIndexOf(Object o);	// 从后先前找，找到对象最后出现的索引值 arr.lastIndexOf(o);

public Object clone();		// 拷贝数组，浅拷贝，元素本身没有被拷贝（只是增加引用而已） arr.clone();
public Object[] toArray();	// 将ArrayList转换成数组，arr.toArray();

public E get(int index);	// 获得该索引下的对象 arr.get(1);
public E set(int index, E element);		// 该索引下的对象赋值，改变引用而已, arr.set(1, 3);
public boolean add(E e);	// 在数组尾部增加元素 arr.add(2);——注意是末尾
public void add(int index, E element);		// 在指定位置插入元素，如果index=arr.size()，那就是插入到数组尾部
public boolean addAll(Collection<? extends E> c);	// 将另一个ArrayList元素全部加入到当前ArrayList尾部，必须是当前元素的同类或子类
public boolean addAll(int index, Collection<? extends E> c);	// 同上，从尾部插入，变成插入到index开始的位置（那么原来index及之后的都顺应后移）
    
public E remove(int index);		// 将对应索引的元素删除，后面的值都会左移一位，返回值是被删除的元素
public boolean remove(Object o);	// 将对象删除，只删除第一个equals的对象，如果为null，就删除null元素
public void clear();		// 清空数组， arr.clear();
```

### （二）基本原理

最最基本的原理，其底层还是数组，数组的操作就是增、删、改、查，就是增加稍微有些不同，其他和普通数组一样

#### 1. ArrayList的实例变量：

```Java
transient Object[] elementData; // 内部私有数组，如果当前大小不够了会重新申请一个数组，那么elementData修改引用即可
private int size;		// 记录数组当前长度，私有，只能通过size()访问——这两个变量就是上面操作的基本对象了

private static final int DEFAULT_CAPACITY = 10;	// 默认的数组长度
private static final Object[] EMPTY_ELEMENTDATA = {};	// 空数组实例，和下面的区分
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};	// 共享的空数组实例（和上面区分，主要是为了看添加第一个元素的时候需要将数组扩展到多大）
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;	// ArrayList里面数组的上限（有些JVM需要在数组中保存一些标题字，head words，所以需要将这些空间预留，再次申请会触发OOM异常） 
```

#### 2. 构造方法：

```Java
public ArrayList() {		// 无参数构造方法
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;	// 直接赋值空数组实例过去
}

public ArrayList(int initialCapacity) {	// 传递了数组的初始长度
    if (initialCapacity > 0) {		// 初始长度>0
        this.elementData = new Object[initialCapacity];	// 创建指定长度的数组
    } else if (initialCapacity == 0) {		// 初始长度为0，那么传递一个空数组实例即可
        this.elementData = EMPTY_ELEMENTDATA;
    } else {			// 抛出异常——非法传参
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}

public ArrayList(Collection<? extends E> c) {		// 传递了一个ArrayList对象
    elementData = c.toArray();		// 将传递来的容器转换成ArrayList
    if ((size = elementData.length) != 0) {		// 如果数组不为空
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)	// 可能c.toArray()不能得到Object[]类的数组，需要额外操作
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;		// 数组为空，就返回空数组
    }
}
```

——第三个构造函数比较复杂，具体见[Java的额外知识](http://note.youdao.com/noteshare?id=9f8ab6eaf048c06b8c823ebe1c7f0b8c&sub=38FCCF96A0C74660A8B19E1140BA449E#7)

#### 3. 源代码阅读：

主要关注增：add：

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // ——确保数组容量是够的，传参是最小容量要求：当前长度+1
    elementData[size++] = e;		// 将该元素插入到数组尾部，size=last_index+1（实际数组的长度）
    return true;
}

// mincapacity：代表增加元素后实际的总元素个数
// newcapacity：代表能满足min下的需要扩容的大小
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {	//如果该数组为第一次使用（刚刚调用无参构造方法创建）
        return Math.max(DEFAULT_CAPACITY, minCapacity);		// 10和minCapacity挑大的——首次使用最小扩容10个
    }
    return minCapacity;		
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;		// 内部的修改次数++

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)		// 如果需要的长度大于当前的数组长度，那么就去扩容
        grow(minCapacity);
}

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);	// 预设的大小为1.5倍旧长度
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;		// 1.5倍和min里面挑大的
    if (newCapacity - MAX_ARRAY_SIZE > 0)		// 如果该值已经超过最大上限
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);	// 将旧数组赋值到新数组那边，并且elementData更新引用地址
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow		// 说明该值已经溢出（超过int正数的上界，变成了负数）
        throw new OutOfMemoryError();		// 抛出OOM异常
    // 如果在int上界-8 ~ int上界范围内，那么就给int上界的大小（有种再给一次机会的感觉）；如果不在（应该不会进入）
    return (minCapacity > MAX_ARRAY_SIZE) ? Integer.MAX_VALUE : MAX_ARRAY_SIZE;
}
```

删：remove：

```Java
public E remove(int index) {
    rangeCheck(index);

    modCount++;		// 修改计数++
    E oldValue = elementData(index);		// 得到要被删除的对象

    int numMoved = size - index - 1;		// 计算要移动的元素个数
    if (numMoved > 0)			// 如果不是最后一个，那么就需要移动
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // 将最后一位设置为null，释放该对象由 GC 完成（看是否需要垃圾回收）

    return oldValue;
}
```

——可以看到这些接口内部是比较复杂的，但是对外的操作很简单——计算机的基本思维之一：**封装复杂操作，提供简化接口**

ps：这边注意一点：每次ArrayList发生结构性变化时，都会让**modCount++**（修改次数增加），包括增加元素（add）和删除元素（remove）

——这个涉及到后面迭代器中的判断：对对象是否发生结构性变化的判断

### （三）迭代

#### 1. 引出

迭代是ArrayList的常见操作，通常用在遍历数组中

**ArrayList是支持foreach语法的，其本质就是迭代器的实现**

先来看一下ArrayList的foreach使用：

```Java
ArrayList<Integer> arr = new ArrayList<>();
arr.add(12); arr.add(23); arr.add(34);
for(Integer a: arr){			// foreach就跟数组类似，元素类型 元素名字：容器变量名
    System.out.println(a);
}
```

虽然这个还是可以和普通的for循环等效替换的，但是**foreach更为简洁，且适合所有的容器，更为通用**。

那么foreach是如何实现的呢？——通过迭代器

编译器会将上面的代码转换为：

```Java
// Main.class文件源码：
ArrayList<Integer> arr = new ArrayList();
arr.add(12); arr.add(23); arr.add(34);

Iterator var2 = arr.iterator();		// 创建一个迭代器
while(var2.hasNext()) {			// 是通过一个while循环实现for循环的遍历——如果存在下一个元素，就继续
    Integer a = (Integer)var2.next();	// 更新指向下一个元素
    System.out.println(a);
}
```

下面来看迭代器是如何实现的：

#### 2. Iterator接口和ListIterator接口

ArrayList实现了Iterable接口，但是可以发现在类头里面没有定义`implements Iterable`，那么在何时定义的呢——追溯到最开始，是**Collection接口（容器类的超类）继承了Iterable**，所以整个体系是：ArrayList --继承--> AbstractList --继承--> AbstractCollection --继承--> Collection -- 继承--> Iterable

——而之前的类/接口只是继承并没有实际实现，在具体类ArrayList中具体实现了

——具体类中必须需要实现Iteable，集合都是可迭代的

```java
// Iterable接口
public interface Iterable<T> {		// Iterable接口主要就是去实现Iterator方法
    Iterator<T> iterator();	// ——返回值是Iterator类对象
}
```

—— 对于Iterable接口，本质上只需要实现一个方法即可

首先需要了解一个关键的点：

在迭代的过程中，**如果调用了非迭代器内部的`remove/add`方法，会抛出`ConcurrentModificationException`异常。想要不抛出这个异常，就必须要使用迭代器内部的`remove`方法**

```Java
// ArrayList.java中的接口实现
public Iterator<E> iterator() {
    return new Itr();		// ——创建一个Itr实例对象（是一个私有方法内部类）
}

// 私有方法内部类，实现了Iterator接口
private class Itr implements Iterator<E> {
    int cursor;       // 指向下一个参数
    int lastRet = -1; // 最近一次返回的索引位置; 如果没有就是-1
    int expectedModCount = modCount;	// 初始化时：期望的修改次数 = 默认的修改次数

    Itr() {}		// 构造方法

    public boolean hasNext() {
        return cursor != size;	// size是ArrayList的实例变量之一，cursor=size，就说明已经到头了
    }

    @SuppressWarnings("unchecked")
    public E next() {		// 获得下一个元素
        checkForComodification();	// 没有结构性变化
        int i = cursor;		// i 指向之前的next节点，那么就是本次的cur
        if (i >= size)			// 如果i已经溢出了——抛出异常	
            throw new NoSuchElementException();
        Object[] elementData = ArrayList.this.elementData;	// 获得ArrayList实际存放元素的数组
        if (i >= elementData.length)		// 如果i已经超过数组的长度——抛出异常，这边说明i<size满足，不一定能使i<arr.length满足
            throw new ConcurrentModificationException();
        cursor = i + 1;		// 更新cursor指向下一个
        return (E) elementData[lastRet = i];		// 返回当前的，并且更新lastRet——表明本次最终返回的索引值
    }
，
    public void remove() {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();

        try {
            ArrayList.this.remove(lastRet);
            cursor = lastRet;
            lastRet = -1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    @Override
    @SuppressWarnings("unchecked")
    public void forEachRemaining(Consumer<? super E> consumer) {
        Objects.requireNonNull(consumer);
        final int size = ArrayList.this.size;
        int i = cursor;
        if (i >= size) {
            return;
        }
        final Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length) {
            throw new ConcurrentModificationException();
        }
        while (i != size && modCount == expectedModCount) {
            consumer.accept((E) elementData[i++]);
        }
        // update once at end of iteration to reduce heap write traffic
        cursor = i;
        lastRet = i - 1;
        checkForComodification();
    }

    final void checkForComodification() {	// 判断是否发生结构性变化——即是否调用外部的add/remove方法
        if (modCount != expectedModCount)	// 主要看当前的修改次数和期待的修改次数是否一致——一致表明无变化，继续
            throw new ConcurrentModificationException();		// 不一致，就抛出异常
    }
}
```

——可以看一下Iterator接口的定义：

```Java
// Iterator接口
// 主要是要实现：hasNext()、next()、remove()（有默认方法，可以不实现）、forEachRemaining（有默认方法，可以不实现）
public interface Iterator<E> {
    // 判断之后是否还会存在其他元素——实现接口需要实现的
    boolean hasNext();

    // 获得下一个元素——实现接口需要实现的
    E next();
    
    /*
     * @throws IllegalStateException 如果next方法没有被调用，或者该remove方法已经被刚刚的next方法调用过了
     */
    default void remove() {		// 提供了默认的方法——默认是不允许通过remove去删除元素的
        throw new UnsupportedOperationException("remove");// 抛出不支持异常，表示该迭代器（具体）不支持该remove操作
    }

    /**
     * 对剩余的每个元素执行给定的操作，直到所有元素都已经操作过了or该操作触发了异常。如果指定了顺序那就按照指定的顺序迭代
     * 操作产生的异常会上抛给给调用者
     *
     * @throws NullPointerException 如果指定的action为null
     */
    default void forEachRemaining(Consumer<? super E> action) {	// 提供了默认的方法
        Objects.requireNonNull(action);	// 检查指定的对象引用是否为null（该方法主要是在方法、构造方法中进行参数验证）
        while (hasNext())
            action.accept(next());
    }
}
```

如果需要进行删除操作的话，必须调用itr的remove方法，那么必须要显式声明itr对象

```java
Iterator itr = arr.iterator();		// 获得迭代器对象
while(itr.hasNext()){		// 调用iterator的三个方法
    Integer integer = (Integer) itr.next();	// 注意next的调用
    if(integer == 23) {
        itr.remove();
    }
}
```

可以发现：Iterator只有顺序遍历，删除功能，如果要使用add等其他功能又会抛出异常。

从而产生了一个新的接口：ListIterator，它继承自Iterator，还增加了add、set、并且能通过previous实现逆向遍历：

```java
// 这边的cursor就是当前节点——主要是针对prev遍历
private class ListItr extends Itr implements ListIterator<E> {	// 继承了上面Itr，并且实现了ListIterator接口
    ListItr(int index) {			// 构造方法
        super();				// 调用Itr的构造方法——其实就是空
        cursor = index;
    }

    public boolean hasPrevious() {
        return cursor != 0;
    }

    public int nextIndex() {
        return cursor;
    }

    public int previousIndex() {
        return cursor - 1;
    }

    @SuppressWarnings("unchecked")
    public E previous() {			// 和next是相对的操作，找前面一个元素
        checkForComodification();
        int i = cursor - 1;
        if (i < 0)		// 数组不存在负数的index
            throw new NoSuchElementException();
        Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length)		// 出现这个说明存在并发操作，本来i应该是<length，后面应该是并发线程去缩小了数组的范围，那么导致该异常
            throw new ConcurrentModificationException();
        cursor = i;		// 更新下一个index
        return (E) elementData[lastRet = i];		// 更新lastRet就是本次返回的index
    }

    public void set(E e) {			// 修改上一次返回的值即从上一次previous/next调用后的index对应元素的值
        if (lastRet < 0)			// 表示不能修改
            throw new IllegalStateException();
        checkForComodification();

        try {
            ArrayList.this.set(lastRet, e);
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }
	
    // 分析之后发现，一个循环内可以多次add，且是头插法，但是add之后不能再对数组进行其他操作，eg：set、remove
    // ——官方注释里面没有写是否会出现一个循环内的多次插入——可能是没考虑到？
    public void add(E e) {		// 添加元素，和remove相对
        checkForComodification();

        try {
            int i = cursor;
            ArrayList.this.add(i, e);		// 插入到i的位置，原来的i及其之后的元素都后移
            cursor = i + 1;		// 并且当前索引更新到下一个——就是原来的i
            lastRet = -1;		// 不能在这之后remove，也不能set但是可以继续add
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }
}
```

那么为啥要使用迭代器呢？（如果要实现遍历可以使用普通的get、size这些方法组合形成）

$\because$ foreach语法更为简洁一些，更重要的是，**迭代器语法更为通用，它适用于各种容器类。**

**并且迭代器体现了一种常见设计模式：关注点分离的思想，将数据的实际组织方式与数据的迭代遍历相分离**

需要访问容器元素只需要对Iterator进行引用，而不需要关注数据结构，可以使用统一的方式访问——也体现了封装的思想，只需要调用的简单接口就可以实现复杂的操作

并且Iterator接口实现是了解了数据结构的，所以是实现了迭代器的高效访问方式，eg：LinkedList的迭代器性能就比自己写要高的多

### （四）总结ArrayList实现了哪些接口/抽象类

#### 1. Collection

```java
public interface Collection<E> extends Iterable<E> {
    int size();
    boolean isEmpty();
    boolean contains(Object o);
    Iterator<E> iterator();
    Object[] toArray();
    <T> T[] toArray(T[] a);
    boolean add(E e);
    boolean remove(Object o);
    boolean containsAll(Collection<?> c);
    boolean addAll(Collection<? extends E> c);
    boolean removeAll(Collection<?> c);
    boolean retainAll(Collection<?> c);
    void clear();
    boolean equals(Object o);		// ArrayList是直接继承了AbstractList的实现
    int hashCode();		// ArrayList是直接继承了AbstractList的实现
    
    // Java8 后面新增的，为了提高对前面版本的兼容性，所以提供了默认的方法
    // 实现过滤器式删除，只有通过过滤的元素才会被删除
    default boolean removeIf(Predicate<? super E> filter) {		// 提供了默认的方法——ArrayList有重写
        Objects.requireNonNull(filter);
        boolean removed = false;
        final Iterator<E> each = iterator();		// 调用迭代器
        while (each.hasNext()) {
            if (filter.test(each.next())) {
                each.remove();
                removed = true;
            }
        }
        return removed;
    }
    default Spliterator<E> spliterator() {			// 提供了默认的方法——有重写
        return Spliterators.spliterator(this, 0);
    }
    default Stream<E> stream() {			// 提供了默认的方法
        return StreamSupport.stream(spliterator(), false);
    }
    default Stream<E> parallelStream() {		// 提供了默认的方法
        return StreamSupport.stream(spliterator(), true);
    }
}
```

#### 2. AbstractCollection

对`Collection`接口均有实现，默认是通过迭代器的实现：

```Java
int size();		// 没有实现
boolean isEmpty();		// 间接调用size
boolean contains(Object o);		// 迭代器实现
Iterator<E> iterator();		// 没有实现
Object[] toArray();			// 迭代器实现
<T> T[] toArray(T[] a);		// 迭代器实现
boolean add(E e);			// 只是抛出异常——表明如果其子类没有重写该方法，就说明不支持此方法——throw new UnsupportedOperationException();
boolean remove(Object o);		// 迭代器实现

boolean containsAll(Collection<?> c){	// ArrayList是直接使用的，并没有单独实现，看c对象中的所有元素是否在e中均存在
    for (Object e : c)			// 本质上还是调用了迭代器			
        if (!contains(e))	
            return false;
    return true;
}

boolean addAll(Collection<? extends E> c);	// 本质上还是调用了迭代器	
boolean removeAll(Collection<?> c);		// 迭代器实现
boolean retainAll(Collection<?> c);	// 迭代器实现
void clear();	// 迭代器实现

public String toString() {		// 特别的，实现了一个toString的方法——ArrayList是直接使用的，并没有单独实现
    Iterator<E> it = iterator();		// 迭代器实现
    if (! it.hasNext())
        return "[]";

    StringBuilder sb = new StringBuilder();		// StringBuilder对象创建——因为一直需要修改内容
    sb.append('[');
    for (;;) {
        E e = it.next();
        sb.append(e == this ? "(this Collection)" : e);
        if (! it.hasNext())
            return sb.append(']').toString();
        sb.append(',').append(' ');		// 输出格式[xxx, xxx, ...]
    }
}
```

#### 3. List

是继承了`Collection`，并且增加了其他的方法

内部均没有实现Collection中的方法，只是声明

下面是新增的方法：（注意区分，有些只是参数变化而已，方法名没有变化）

```java
void add(int index, E element);		// 与add相比多了index参数
boolean addAll(int index, Collection<? extends E> c);	// 与addAll相比多了index参数
E remove(int index);		// 与remove相比，从object对象变成了index

E get(int index);
E set(int index, E element);
int indexOf(Object o);
int lastIndexOf(Object o);
ListIterator<E> listIterator();
ListIterator<E> listIterator(int index);
List<E> subList(int fromIndex, int toIndex);

// 默认实现——将每个元素使用一次operator，将运算结果替换原来的值
default void replaceAll(UnaryOperator<E> operator) {		// ArrayList有重写
    Objects.requireNonNull(operator);
    final ListIterator<E> li = this.listIterator();		// 用ListIterator迭代器实现
    while (li.hasNext()) {
        li.set(operator.apply(li.next()));
    }
}

// 默认实现——排序
default void sort(Comparator<? super E> c) {		// ArrayList有重写
    Object[] a = this.toArray();		// 转换成数组
    Arrays.sort(a, (Comparator) c);		// 调用数组排序，排序方式需要传参Comparator，比较器
    ListIterator<E> i = this.listIterator();
    for (Object e : a) {
        i.next();
        i.set((E) e);
    }
}

// 默认实现——分割器
default Spliterator<E> spliterator() {
    return Spliterators.spliterator(this, Spliterator.ORDERED);
}
```

#### 3. AbstractList

会去重写AbstractCollection的一些方法，有些是默认继承的；并且实现了AbstractCollection没有实现的Collection中的方法；增加了几个方法和内部类——主要是为了实现：`ListIterator`、`RandomAccess`

```java
// 继承AbstractCollection，并重写的
public boolean add(E e);		// 去调用下面的add
public boolean addAll(int index, Collection<? extends E> c);		// 重写，优化了
public void clear();		// 调用了下面的removeRange

public Iterator<E> iterator() {		// 实现了
    return new Itr();
}
private class Itr implements Iterator<E> {}	// 实现了方法内部类Iterator

public boolean equals(Object o);			// 实现了Collection的方法，ArrayList是直接使用的
public int hashCode();				// 实现了Collection的方法，ArrayList是直接使用的
```

```java
// 继承List的：
abstract public E get(int index);		// 未实现
public E set(int index, E element);		// 只是抛出异常——表明如果其子类没有重写该方法，就说明不支持此方法——throw new UnsupportedOperationException();
public void add(int index, E element);// 只是抛出异常——表明如果其子类没有重写该方法，就说明不支持此方法——throw new UnsupportedOperationException();
public E remove(int index);	// 只是抛出异常——表明如果其子类没有重写该方法，就说明不支持此方法——throw new UnsupportedOperationException();

public int indexOf(Object o);		// 迭代器实现
public int lastIndexOf(Object o);	// 迭代器实现

protected void removeRange(int fromIndex, int toIndex);	// 迭代器实现

public ListIterator<E> listIterator() {		// 实现了
    return listIterator(0);
}
private class ListItr extends Itr implements ListIterator<E> {}	// 实现了该方法内部类

public List<E> subList(int fromIndex, int toIndex);		// 实现了

class SubList<E> extends AbstractList<E>{}	// 实现了方法内部类

class RandomAccessSubList<E> extends SubList<E> implements RandomAccess{};		// 实现了方法内部类
```

在这边可以发现：实现了一个接口：RandomAccess

```java
public interface RandomAccess {		// 没有方法的接口
}
```

没有任何定义代码——这种接口被称为**标记接口**，**用于声明类的一种属性**。——eg：实现了该接口（实际上啥也没实现）表明我可以随机访问，这可能是其底层的特有属性决定的，例如ArrayList底层是数组，那么就是带有随机访问这个属性的。

实现了RandomAccess接口的类表示可以随机访问，可随机访问就是具备类似数组那样的特性，数据在内存是连续存放的，根据索引值就可以直接定位到具体的元素，访问效率很高。而LinkedList，它就不能随机访问——那么就不能实现该RandomAccess

标明了随机访问的优点：可以用于一些**通用的算法代码中，它可以根据这个声明而选择效率更高的实现**。比如，Collections类中有一个方法binarySearch，在List中进行二分查找，它的实现代码就根据list是否实现了RandomAccess而采用不同的实现机制

### （五）ArrayList特点分析

对于同一个操作，不同的数据结构可能性能差距很多，不同的数据结构适用的场景也不同

对于ArrayList来说，常见的增、删、改、查的效率分别是：

- 按index查：可以随机访问，按照索引位置进行访问效率很高，O(1)
- 按内容查：除非数组已排序，否则按照内容查找元素效率比较低，O(N)
- 添加元素——主要开销就是在扩容和复制中，O(N)
- 删除元素——主要开销就是在移动数据中

=> 总结来说：和数组的性能类似，相较于数组它是动态的，内部封装了大量的操作，比数组作用更为广泛

ps：**ArrayList不是线程安全的**，在不需要线程安全的情况下可以使用；如果要求线程安全，那么推荐使用：**`Vector`**

## 二、LinkedList

先来个对比：LinkedList增、删效率高，改查效率低；与ArrayList性能正好相反

LinkedList对标的是C的链表，且是双向链表。

首先看一下类头

```java
public class LinkedList<E> extends AbstractSequentialList<E> 	// 继承
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable	// 实现了
```

可以发现LinkedList实现了List和Deque，可以按照队列、栈、双端队列的特性处理

### （一）使用

存在两个构造方法（ArrayList是3个构造方法）

```java
public LinkedList() {		// 默认构造方法，啥也么有——和Arraylist不一样，那个至少会给一个空数组
}

public LinkedList(Collection<? extends E> c) {	// 带参数的构造方法
    this();		// 调用上面的默认构造方法
    addAll(c);		// 将c中的元素全部复制过来
}
```

使用方法：（和ArrayList类似）

```java
LinkedList<Integer> list = new LinkedList<>();
// 因为实现了List可以赋值给该引用对象
List<Integer> list2 = new LinkedList<>(Arrays.asList(new Integer[]{1,2,3}));
```

使用队列：

原因是LinkedList 实现了 -> Deque -> 继承了 Queue，所以自然有了Queue的特性，但是Deque都只是将自己的方法和Queue的方法列出来，并没有实现，所以LinkedList是实现了所有的方法

```java
Queue<Integer> queue = new LinkedList<>();		// 定义
queue.offer(1);		// 入队
queue.peek();		// 看队首元素
queue.poll();		// 出队
```

queue的方法：

```Java
boolean add(E e);		// 每个方法均有两份（前者可能会抛出异常，后者不会，最多返回null/false）
boolean offer(E e);
E remove();
E poll();
E element();
E peek();
```

why要两份方法：

$\because$ 细节、需求不同

- 在队列为空时，element和remove会抛出异常NoSuchElementException，peek和poll返回特殊值null；
- LinkedList的实现中，队列长度没有上限，但别的Queue的实现可能有。所以需要两份方法：在队列为满时，add会抛出异常IllegalStateException，而offer只是返回false。

使用栈：

```java
Deque<Integer> stack = new LinkedList<>();		// 定义——注意不能用Stack（并没有实现）
stack.push(1);		// 入栈
stack.peek();		// 看栈顶元素
stack.pop();		// 出栈
```

注意，LinkedList并没有实现Stack，所以不能使用Stack

ps：**存在一个Stack类，并没有实现Deque，它是Vector的子类，而它是线程安全的，通过`synchronized`实现的。如果需要线程安全，那么需要使用Stack**

deque的方法：（除了queue自己特有的）

```java
E getFirst();
E getLast();
E peekFirst();
E peekLast();

void addFirst(E e);
void addLast(E e);
boolean offerFirst(E e);
boolean offerLast(E e);

E removeFirst();
E removeLast();
E pollFirst();
E pollLast();			// 以上，双端队列的操作方法

boolean removeFirstOccurrence(Object o);
boolean removeLastOccurrence(Object o);	// 移除首次/最后一次出现的元素

void push(E e);
E pop();

Iterator<E> descendingIterator();		// 逆序遍历

// 使用：
Deque<String> deque = new LinkedList<>(Arrays.asList(new String[]{"Hello", "World","Love", "Milk"}));
Iterator<String>itr = deque.descendingIterator();
while(itr.hasNext()){
    System.out.println(itr.next());
}
// ——逆序输出：Milk Love World Hello
```

——所以LinkedList重复实现了很多功能一样，但是方法名不同的方法，它们都可以使用，但是使用针对性的，在概念上更清晰，可读性和开发性也较好

### （二）基本原理

内部是一个双向链表，每个元素都包装成了一个节点，单独存放在一个内存块中，然后通过引用对象将相邻的节点串起来

```java
transient int size = 0;
transient Node<E> first;		// 初始均为null
transient Node<E> last;
```

### （三）特点分析

- 按需分配空间，不需要预先分配很多空间
- 不可以随机访问，按照索引位置访问效率比较低，必须从头或尾顺着链接找，效率为O(N/2)
- 不管列表是否已排序，只要是按照内容查找元素，效率都比较低，必须逐个比较，效率为O(N)
- 在两端添加、删除元素的效率很高，为O(1)
- 在中间插入、删除元素，要先定位，效率比较低，为O(N)，但修改本身的效率很高，效率为O(1)

所以，对于有明确队列、栈特点的数据结构，可以直接使用LinkedList进行实现

## 三、ArrayDeque

使用方法和LinkedList一样，但是底层实现是数组，所以查找高效，且有链表的特性，删除和增加效率也很高——在数组的基础上增加两个标志位head、tail，从而构成了一个**循环数组**，**数组长度为2的幂次方**，使用**高效的位操作**进行各种判断，以及对head和tail进行维护

具体见源码分析：

#### 特点分析：

- 在两端添加、删除元素的效率很高，添加N个元素的效率为O(N)
- 根据元素内容查找和删除的效率比较低，为O(N)
- 与ArrayList和LinkedList不同，**没有索引位置的概念，不能根据索引位置进行操作**，只有相对位置的概念

从两端进行操作，ArrayDeque效率更高一些；如果同时需要根据索引位置进行操作，或者经常需要在中间进行插入和删除，则应该选LinkedList

——由于其功能和LinkedList高度冲突，且性能来说也没有优化很多，所以常用的就是LinkedList，且符合了链表的特性，更符合语义。

## 四、HashMap

HashMap就是hash的映射方式

### （一）概念了解

Map，有键和值概念，即**`key-value`**，它们是对应的，一个键对应一个值

**Map按照键存储和访问，所以要求键不能重复**，如果出现重复会覆盖原来的值。所以在Map中key是唯一的，而value不唯一

应用场景：

- 字典：key：单词，value：单词的发音、含义等
- 配置文件的配置项：key：配置对象，value：配置值
- 人员：key：身份证编号，value：姓名、性别等

——数组、ArrayList、LinkedList其实都是特殊的Map，它们的key就是index，value就是索引存放的值

Map在Java中存在，是一个接口，有定义如下方法：

```java
public interface Map<K,V> {		// 类型参数有两个，对应key的类型和value的类型
    int size();			// 查看key-value对的个数
    boolean isEmpty();	// 判断表是否为空
    boolean containsKey(Object key);		// 查看是否包含该键，最多一个
    boolean containsValue(Object value);	// 查看是否包含该值，可能有多个
    V get(Object key);		// 根据key去获取对应的value值
    V put(K key, V value);	// 保存key-value对，如果key存在的话就覆盖
	V remove(Object key);	// 移除key-value对，按照key移除，并返回原来的key值
    void putAll(Map<? extends K, ? extends V> m);	// 保存所有key-value到当前对象
    void clear();			// 清空key-value对
    boolean equals(Object o);		// 判断这两个表是否一样
    int hashCode();		// 获得该表的hash值
    
    Set<K> keySet();		// 获得表中key的集合，返回一个set（不可重复）
    Collection<V> values();		// 获取表中value的集合
    Set<Map.Entry<K, V>> entrySet();		// 获取表中的key-value对
    
    interface Entry<K,V> {			// 嵌套接口
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
    
}
```

Set是视图，即修改Set的值就是修改Map本身的内容