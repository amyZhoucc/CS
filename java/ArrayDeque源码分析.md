## ArrayDeque源码分析

本质上是一个循环数组

**像是一个ArrayList和LinkedList的集合，底层是ArrayList一样的数组，通过不同的实现方式，上层对外的接口又都是LinkedList一致的**

兼具了数组的查询高效，又有高效的增、删特点

所以，这边应该会和ArrayList和LinkedList都进行对比的基础上看源码的实现

## 1. 类头

```java
public class ArrayDeque<E> extends AbstractCollection<E>
                           implements Deque<E>, Cloneable, Serializable
```

## 2. 静态变量（暂无研究）

```java
private static final long serialVersionUID = 2340985798034038923L;
```

## 3. 实例变量

```java
transient Object[] elements;		// 数组存放元素
transient int head;		// 指示数组头的位置——LinkedList有的——是实现高效删除的关键
transient int tail;		//  指示数组尾的位置，是指向下一个空的位置——LinkedList有的，但是含义稍有不同

private static final int MIN_INITIAL_CAPACITY = 8;		// 数组的最小容量（ArryaList是默认为0）
```

——循环数组必须要时刻留下一个空位，tail指向下一个空位，如果tail==head，表示数组已经存满了，马上要扩容而不能等待下一次再扩容

ps：实例变量中没有size，来存放元素个数，如果要知道元素个数，可以通过head和tail的计算得到；

且arrayDeque限制数组的最小长度为8，默认为16

## 4. 构造方法

和arrayList一样存在3个构造方法，但是实现稍有不同

```java
public ArrayDeque() {		// 默认构造方法——去创建长度为16的数组
    elements = new Object[16];		
}

public ArrayDeque(int numElements) {		// 传入大小（不再是传入多少就给多少，而是要进行处理）
    allocateElements(numElements);
}

public ArrayDeque(Collection<? extends E> c) {
    allocateElements(c.size());		// 根据c的大小扩容
    addAll(c);			// 然后将c中的内容全部复制到本对象中
}
```

## 4. 静态方法（提升幸福品质）

计算容量

这个是计算数组应该的大小：**数组的大小都是大于当前数组长度的2的幂次**。主要是为了后面的位计算方便——关键方法

```java
private static int calculateSize(int numElements) {
    int initialCapacity = MIN_INITIAL_CAPACITY;		// 最小为8
    // Find the best power of two to hold elements.
    // Tests "<=" because arrays aren't kept full.
    if (numElements >= initialCapacity) {			// 如果传参不小于最小阈值
        initialCapacity = numElements;
        initialCapacity |= (initialCapacity >>>  1);// 这边是将numElements的二进制最高位1后面的数全部置为1
        initialCapacity |= (initialCapacity >>>  2);
        initialCapacity |= (initialCapacity >>>  4);
        initialCapacity |= (initialCapacity >>>  8);
        initialCapacity |= (initialCapacity >>> 16);
        initialCapacity++;			// +1之后自然就进位了，那么就获得了>numElements的最小二进制数
        // eg: 8 = 1000 -> 1100 -> 1111 -> 10000=16

        if (initialCapacity < 0)   // 如果值为负数，说明是int数上溢，那么就给一个2^30大小的数组——这是最大上限了（int：2^31-1）
            initialCapacity >>>= 1;
    }
    return initialCapacity;
}
```

## 5. 实例方法

接口同LinkedList，都是有3套方法——deque、stack、queue

### （1）增

add当数组容量超过2^30以上，就会因为int溢出，而触发`IllegalStateException`

如果add的元素是null，会抛出异常：`NullPointerException`

```java
// 数组头部增加一个元素——能进来说明此时数组还未满，出去之前会判断是否满了，满了就扩容，保证下一次进来还是有空间存放的
public void addFirst(E e) {
    if (e == null)			// 如果元素为null，抛出异常
        throw new NullPointerException();
    elements[head = (head - 1) & (elements.length - 1)] = e;
    if (head == tail)		// add完之后发现数组用完，那么就扩容
        doubleCapacity();
}
```

ps：`head = (head - 1) & (elements.length - 1)`：这句是关键，主要就是来确定要插入的索引位置

由于elements.length=2^n，那么elements.length-1 = 2^n-1，那么低位全部为1，高位全部为0

如果head-1>=0，那么就是其本身。而如果head-1 = -1（不可能是其他负数），那么就是elements.length-1，就是数组的最后一个位置——符合要求。

**这边运用了2的幂次的特性和位操作，使计算得到的值限制在0~2^n-1范围内，并且对于边界处理的是正确的**

那么存在一个问题，不用2^n作为数组长度可以吗？

答：不可以，如果作为边界情况，例如-1=11....11&len=len——符合要求；0&len=0——符合要求；len+1&len——不符合要求了（获得是len+1的最高位1出现的位置）；其他数& len $\neq$其他数——不符合要求

——所以只有2^n-1才有如此特性才能满足要求

```java
// 在数组尾部添加一个元素
public void addLast(E e) {
    if (e == null)
        throw new NullPointerException();
    elements[tail] = e;		// tail是存放指向下一个空位的，那么e就可以存在当前空位中
    if ( (tail = (tail + 1) & (elements.length - 1)) == head)	// 计算下一个空位的index
        doubleCapacity();
}
```

ps：`tail = (tail + 1) & (elements.length - 1)`：这句同上

tail+1<length，那么计算得到其本身；tail+1=length，那么计算得到0——符合要求

限制了tail在0~length-1的范围内移动

```java
public boolean offerFirst(E e) {	// 传参为null时抛出异常
    addFirst(e);
    return true;
}
public boolean offerLast(E e) {
    addLast(e);
    return true;
}
// collection的方法实现
public boolean add(E e) {		// 等同于尾巴上添加元素
    addLast(e);
    return true;
}
// queue方法实现
public boolean offer(E e) {
    return offerLast(e);
}
// stack方法实现（不是真实的stack类，而是deque提供的）
public void push(E e) {
    addFirst(e);
}
```

### （2）删

同LinkedList一样，使用`removexxx`，会存在数组为空，而抛出异常：`NoSuchElementException`，`pop`也会抛出异常（本质是调用了remove方法）

而`pollxxx`，数组为空，返回null，不会抛出异常

`removeFirstOccurrence`、`removeLastOccurrence`，数组为空，返回删除失败false，不会抛出异常

```java
public E removeFirst() {
    E x = pollFirst();
    if (x == null)		// 出现在数组为空，那么抛出异常
        throw new NoSuchElementException();
    return x;
}

public E removeLast() {
    E x = pollLast();
    if (x == null)		// 出现在数组为空，那么抛出异常
        throw new NoSuchElementException();
    return x;
}

public E pollFirst() {		// 数组头出，head+1，并控制好范围表示（类同于last+1的感觉）
    int h = head;
    @SuppressWarnings("unchecked")
    E result = (E) elements[h];
    if (result == null)			// 出现在数组为空，没有取到数据，那么返回null
        return null;
    elements[h] = null;     // 将原来的位置元素清空——不清空对于查找等操作会有影响
    head = (h + 1) & (elements.length - 1);
    return result;
}

public E pollLast() {
    int t = (tail - 1) & (elements.length - 1);		// 实际元素存放在tail-1的位置处（tail存放的是下一空位）
    @SuppressWarnings("unchecked")
    E result = (E) elements[t];
    if (result == null)		// 出现在数组为空，没有取到数据，那么返回null
        return null;
    elements[t] = null;		// 将原来的位置元素清空
    tail = t;
    return result;
}

public boolean removeFirstOccurrence(Object o) {
    if (o == null)
        return false;
    int mask = elements.length - 1;		// 掩码
    int i = head;			// 从前向后找
    Object x;
    while ( (x = elements[i]) != null) {	// 循环终止条件是出现null（所以元素不能存在null，将元素删除后要将对应位置赋值为null）	
        if (o.equals(x)) {
            delete(i);		// 调用删除操作——从中间删除
            return true;
        }
        i = (i + 1) & mask;
    }
    return false;
}
public boolean removeLastOccurrence(Object o) {
    if (o == null)
        return false;
    int mask = elements.length - 1;
    int i = (tail - 1) & mask;
    Object x;
    while ( (x = elements[i]) != null) {
        if (o.equals(x)) {
            delete(i);		// 调用删除操作——从中间删除
            return true;
        }
        i = (i - 1) & mask;
    }
    return false;
}

// collection方法实现
public E remove() {		// 删头
    return removeFirst();
}

// queue方法实现
public E poll() {
    return pollFirst();
}
// stack方法实现
public E pop() {
    return removeFirst();
}

public boolean remove(Object o) {		// 删除指定的元素
    return removeFirstOccurrence(o);
}

public void clear() {			// 清空
    int h = head;
    int t = tail;
    if (h != t) { // 清除所有相关变量和数组内容——如果h==t，表示本来就为空数组
        head = tail = 0;
        int i = h;
        int mask = elements.length - 1;
        do {
            elements[i] = null;
            i = (i + 1) & mask;
        } while (i != t);
    }
}
```

——从头尾删除，时间复杂度均为O(1); 从中间删除，需要查找，时间为O(N)，删除需要移动数组，时间为O(N/2)，总结就是O(N)

=>可以发现，

- 时间性能比ArrayList好（除了尾巴删除是O(1)，其余均因要移位和查找都是O(N)），
- 和LinkedList性能基本一样：首尾O(1)，中间，查找时间也是O(N)，删除ArrayDeque耗时多一点，但是总时间复杂度未变为O(N)；区别是，**如果是指定index删除：LinkedList是O(1)，ArrayDeque是O(N/2)**

### （3）改

未实现set，set是List的方法（而arrayDeque未实现）

### （4）查

和LinkedList一样，`getxxx`会因为数组为空，而抛出异常`NoSuchElementException`，`element`也同样（本质上调用了getFirst方法）

而`peekxxx`数组为空，直接返回null，不会抛出异常

```java
public E getFirst() {
    @SuppressWarnings("unchecked")
    E result = (E) elements[head];
    if (result == null)			// 数组为空，会抛出异常
        throw new NoSuchElementException();
    return result;
}

public E getLast() {
    @SuppressWarnings("unchecked")
    E result = (E) elements[(tail - 1) & (elements.length - 1)];	// 获得tail-1，确保范围正确
    if (result == null)		// 数组为空，会抛出异常
        throw new NoSuchElementException();
    return result;
}

public E peekFirst() {
    return (E) elements[head];		// 如果数组为空，那么就是返回null，并不会抛出异常（确保是null，是因为在删除的时候，对所在的位置设定为null）
}

public E peekLast() {
    return (E) elements[(tail - 1) & (elements.length - 1)];	// 如果数组为空，那么就是返回null
}

public E element() {
    return getFirst();
}

public E peek() {
    return peekFirst();
}
```

其他查的方法：

```java
// 查数组元素个数——如果让我写肯定先判断head和tail的大小，然后用+/-运算得到，而这边用了位运算
public int size() {
    return (tail - head) & (elements.length - 1);		// 太妙了，如果是负数，那么得到就是length-（-负数）
}

public boolean isEmpty() {		// 判空方法
    return head == tail;
}

public boolean contains(Object o) {		// 判断数组中是否存在元素o
    if (o == null)		// 肯定不存在null
        return false;
    int mask = elements.length - 1;
    int i = head;
    Object x;
    while ( (x = elements[i]) != null) {		// 从头开始遍历
        if (o.equals(x))
            return true;
        i = (i + 1) & mask;
    }
    return false;
}
```

ps：size()方法可以学学！！！

### （5）辅助方法（高效的关键）

```java
private void allocateElements(int numElements) {	// 根据numElements计算适合的数组长度，并创建新的数组
    elements = new Object[calculateSize(numElements)];
}

// 扩容操作：原来长度*2，并且将原来的内容复制过去
private void doubleCapacity() {		// 两倍的扩容
    assert head == tail;		// 只有当head和tail相遇的时候才能继续执行，不相遇那么数组还未满，会抛出AssertionError
    int p = head;
    int n = elements.length;
    int r = n - p; // 求head右边有多少元素（因为存在tail<head的情况）
    int newCapacity = n << 1;		// 扩容2倍
    if (newCapacity < 0)		// 扩容之后溢出
        throw new IllegalStateException("Sorry, deque too big");
    Object[] a = new Object[newCapacity];		// 创建新数组
    System.arraycopy(elements, p, a, 0, r);		// 先复制head~length-1
    System.arraycopy(elements, 0, a, r, p);		// 后复制0~tail-1的
    elements = a;		// 更新数组指针
    head = 0;		// 更新最新的head和tail的位置
    tail = n;
}

// 将本对象的数组复制到a中
private <T> T[] copyElements(T[] a) {
    if (head < tail) {				// 不存在分布在数组两边的情况——直接从head~tail-1复制过去
        System.arraycopy(elements, head, a, 0, size());
    } else if (head > tail) {		// 存在分布两边的情况——先复制head~length-1，再复制0~tail-1
        int headPortionLen = elements.length - head;
        System.arraycopy(elements, head, a, 0, headPortionLen);
        System.arraycopy(elements, 0, a, headPortionLen, tail);
    }
    return a;
}

// 根据索引位置元素从数组中删除——比较麻烦，看i离head和tail哪个更近——减少移动次数，离head近，那么head向后移动一位；tail近，那么tail向前移动一位
private boolean delete(int i) {	
    checkInvariants();
    final Object[] elements = this.elements;
    final int mask = elements.length - 1;
    final int h = head;
    final int t = tail;
    final int front = (i - h) & mask;	// i位置和head的距离
    final int back  = (t - i) & mask;	// i位置和tail的距离

    // Invariant: head <= i < tail mod circularity
    if (front >= ((t - h) & mask))
        throw new ConcurrentModificationException();

    // Optimize for least element motion
    if (front < back) {		// i和head距离近
        if (h <= i) {		// h和i同方向，那么h~i-1复制到h+1~i处
            System.arraycopy(elements, h, elements, h + 1, front);
        } else { // h和i不同方向（i和t同方向），那么[0,i-1]的移动到[1, i]，len-1的元素复制到0，[head, len-1)的元素复制到[head+1, len-1]
            System.arraycopy(elements, 0, elements, 1, i);
            elements[0] = elements[mask];
            System.arraycopy(elements, h, elements, h + 1, mask - h);
        }
        elements[h] = null;		// 将原来的head赋值为null
        head = (h + 1) & mask;
        return false;
    } else {			// i和tail距离近
        if (i < t) { // i在t前——同方向，那么就将i+1~tail的节点复制到i~tail-1
            System.arraycopy(elements, i + 1, elements, i, back);
            tail = t - 1;
        } else { // i不在t的同方向（i和head在同方向），那么将[i+1, len)的元素复制到[i, len-1)处，0的元素复制到len-1处，[1, tail)的元素复制到[0,tail-1)处
            System.arraycopy(elements, i + 1, elements, i, mask - i);
            elements[mask] = elements[0];
            System.arraycopy(elements, 1, elements, 0, t);
            tail = (t - 1) & mask;
        }
        return true;
    }
}

// 这个主要是在检查tail、head指向的元素和其周围的元素是否正确：
// 数组为空时，head和tail应该相等，且head指向null；数组不为空时，head指向不为空，head-1的指向为空（非满状态），tail应该为空，tail-1应该不为空——符合循环数组的要求
private void checkInvariants() {
    assert elements[tail] == null;		// 证明tail没有出错
    assert head == tail ? elements[head] == null :	// 如果数组为空，那么head指向的值应该为null；不为空，那么head指向的值不为null，tail-1指向的值也不为null
    (elements[head] != null &&
     elements[(tail - 1) & (elements.length - 1)] != null);
    assert elements[(head - 1) & (elements.length - 1)] == null;		// head-1之前的值应该为null，否则就满了，没有及时扩容
}
```

### （6）toArray

```Java
public Object[] toArray() {		// 将本对象转换成Object[]数组
    return copyElements(new Object[size()]);
}

public <T> T[] toArray(T[] a) {		// 将本对象放到指定的数组中
    int size = size();
    if (a.length < size)		// 长度不够建新的数组
        a = (T[])java.lang.reflect.Array.newInstance(
        a.getClass().getComponentType(), size);
    copyElements(a);
    if (a.length > size)		// 后面未填满的全部清空为null
        a[size] = null;
    return a;
}
```

### （7）clone：浅拷贝

```java
public ArrayDeque<E> clone() {
    try {
        @SuppressWarnings("unchecked")
        ArrayDeque<E> result = (ArrayDeque<E>) super.clone();
        result.elements = Arrays.copyOf(elements, elements.length);
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

### （8）流操作（暂无研究）

```java
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException {
    s.defaultWriteObject();

    // Write out size
    s.writeInt(size());

    // Write out elements in order.
    int mask = elements.length - 1;
    for (int i = head; i != tail; i = (i + 1) & mask)
        s.writeObject(elements[i]);
}

/**
     * Reconstitutes this deque from a stream (that is, deserializes it).
     */
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    s.defaultReadObject();

    // Read in size and allocate array
    int size = s.readInt();
    int capacity = calculateSize(size);
    SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
    allocateElements(size);
    head = 0;
    tail = size;

    // Read in all elements in the proper order.
    for (int i = 0; i < size; i++)
        elements[i] = s.readObject();
}
```

### （9）迭代器：Iterator

不同于linkedList，是iterator，而不是listIterator

```java
public Iterator<E> iterator() {		// 实现collection要的方法
    return new DeqIterator();
}
```

```java
private class DeqIterator implements Iterator<E> {
    private int cursor = head;	// 指向下一次需要访问的节点
    private int fence = tail;		// 主要来终止迭代，也能检查并发修改

    private int lastRet = -1;	// 记录上次返回的index，如果上次返回的index已经被删除了，那么设置为-1，表示不可再删除

    public boolean hasNext() {		// 是否还有未访问的节点
        return cursor != fence;
    }

    public E next() {
        if (cursor == fence)		// 遍历到头了
            throw new NoSuchElementException();
        @SuppressWarnings("unchecked")
        E result = (E) elements[cursor];
        // 不检查并发，但是会发现破坏遍历的情况——在遍历的过程中去调用了add导致扩容，或者去调用了外部的remove，导致当前节点在tail外面了——抛出异常
        if (tail != fence || result == null)
            throw new ConcurrentModificationException();
        lastRet = cursor;		// 更新上次返回的——就是本次返回的
        cursor = (cursor + 1) & (elements.length - 1);		// 下一个要访问的更新
        return result;
    }

    public void remove() {
        if (lastRet < 0)			// 说明已经被删除过，或是初始的时候没有去调用next就去删除
            throw new IllegalStateException();
        if (delete(lastRet)) { // if left-shifted, undo increment in next()
            cursor = (cursor - 1) & (elements.length - 1);	// lastret=next-1，而lastret被删除了，那么lastret=本来的next，那么next也需要更新到lastret=本来的next
            fence = tail;		// tail-1，那么需要更新fence
        }
        lastRet = -1;
    }

    public void forEachRemaining(Consumer<? super E> action) {	//当前剩余的元素全部执行action操作，并且也算在操作中
        Objects.requireNonNull(action);
        Object[] a = elements;
        int m = a.length - 1, f = fence, i = cursor;
        cursor = f;		// 该遍历结束后，cursor就到了fence，迭代可以结束了
        while (i != f) {
            @SuppressWarnings("unchecked") E e = (E)a[i];
            i = (i + 1) & m;
            if (e == null)
                throw new ConcurrentModificationException();
            action.accept(e);
        }
    }
}
```

descendingIterator：逆序迭代

```java
public Iterator<E> descendingIterator() {
    return new DescendingIterator();		// deque的方法实现
}

// 和DeqIterator独立实现——但是实现技术类似，只不过是镜像翻转
private class DescendingIterator implements Iterator<E> {
    /*
         * This class is nearly a mirror-image of DeqIterator, using
         * tail instead of head for initial cursor, and head instead of
         * tail for fence.
         */
    private int cursor = tail;
    private int fence = head;
    private int lastRet = -1;

    public boolean hasNext() {
        return cursor != fence;
    }

    public E next() {
        if (cursor == fence)
            throw new NoSuchElementException();
        cursor = (cursor - 1) & (elements.length - 1);
        @SuppressWarnings("unchecked")
        E result = (E) elements[cursor];
        if (head != fence || result == null)
            throw new ConcurrentModificationException();
        lastRet = cursor;
        return result;
    }

    public void remove() {
        if (lastRet < 0)
            throw new IllegalStateException();
        if (!delete(lastRet)) {
            cursor = (cursor + 1) & (elements.length - 1);
            fence = head;
        }
        lastRet = -1;
    }
}
```

### （10）spliterator（暂无研究）

```java
public Spliterator<E> spliterator() {
    return new DeqSpliterator<E>(this, -1, -1);
}

static final class DeqSpliterator<E> implements Spliterator<E> {
    private final ArrayDeque<E> deq;
    private int fence;  // -1 until first use
    private int index;  // current index, modified on traverse/split

    /** Creates new spliterator covering the given array and range */
    DeqSpliterator(ArrayDeque<E> deq, int origin, int fence) {
        this.deq = deq;
        this.index = origin;
        this.fence = fence;
    }

    private int getFence() { // force initialization
        int t;
        if ((t = fence) < 0) {
            t = fence = deq.tail;
            index = deq.head;
        }
        return t;
    }

    public DeqSpliterator<E> trySplit() {
        int t = getFence(), h = index, n = deq.elements.length;
        if (h != t && ((h + 1) & (n - 1)) != t) {
            if (h > t)
                t += n;
            int m = ((h + t) >>> 1) & (n - 1);
            return new DeqSpliterator<>(deq, h, index = m);
        }
        return null;
    }

    public void forEachRemaining(Consumer<? super E> consumer) {
        if (consumer == null)
            throw new NullPointerException();
        Object[] a = deq.elements;
        int m = a.length - 1, f = getFence(), i = index;
        index = f;
        while (i != f) {
            @SuppressWarnings("unchecked") E e = (E)a[i];
            i = (i + 1) & m;
            if (e == null)
                throw new ConcurrentModificationException();
            consumer.accept(e);
        }
    }

    public boolean tryAdvance(Consumer<? super E> consumer) {
        if (consumer == null)
            throw new NullPointerException();
        Object[] a = deq.elements;
        int m = a.length - 1, f = getFence(), i = index;
        if (i != fence) {
            @SuppressWarnings("unchecked") E e = (E)a[i];
            index = (i + 1) & m;
            if (e == null)
                throw new ConcurrentModificationException();
            consumer.accept(e);
            return true;
        }
        return false;
    }

    public long estimateSize() {
        int n = getFence() - index;
        if (n < 0)
            n += deq.elements.length;
        return (long) n;
    }

    @Override
    public int characteristics() {
        return Spliterator.ORDERED | Spliterator.SIZED |
            Spliterator.NONNULL | Spliterator.SUBSIZED;
    }
}
```



## 6. 总结

ArrayList和LinkedList都是允许元素存在null的，但是ArrayDeque不允许