# LinkedList源码分析

本篇，是在分析了ArrayList的基础上，所以可能有些对比思考，有些一致的操作可能就忽略了

## 1. 类头

```java
public class LinkedList<E> extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```

## 2. 静态变量

将静态内部类算在了静态变量里面，因为效果差不多

```java
private static final long serialVersionUID = 876323262645176354L;

// 静态内部类——主要是定义了Node这个数据结构（类似于C的结构体）——该类就单纯的表示一个数据类型，不带任何的方法
private static class Node<E> {
    E item;				// 节点值
    Node<E> next;		// 后继节点
    Node<E> prev;		// 前驱节点

    Node(Node<E> prev, E element, Node<E> next) {	// 构造方法——赋值即可
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

## 3. 实例变量

```java
transient int size = 0;			// 记录链表的长度，从1开始记（和数组的区别）
transient Node<E> first;		// 指向头结点，初始值为null
transient Node<E> last;			// 指向尾节点，初始值是null

protected transient int modCount = 0;		// AbstractList.java定义的，功能同ArrayList
```

ps：记录`modCount`使用到的地方：`addAll`、`clear`、`linkFirst`、`linkLast`、`linkBefore`、`unlinkFirst`、`unlinkLast`、`unlink`、

## 4. 构造方法

```java
public LinkedList() {		// 默认构造方法，啥也么有——和Arraylist不一样，那个至少会给一个空数组
}

public LinkedList(Collection<? extends E> c) {	// 带参数的构造方法
    this();		// 调用上面的默认构造方法
    addAll(c);		// 将c中的元素全部复制过来
}
```

## 5. 静态方法

无

## 6. 实例方法

### （1）增

```java
// 链表尾插入节点
public boolean add(E e) {
    linkLast(e);
    return true;
}
// 在指定索引的前面插入节点
public void add(int index, E element) {
    checkPositionIndex(index);		// 边界判断[0, size]

    if (index == size)		// 如果是size，说明就是插入到链表尾
        linkLast(element);
    else
        linkBefore(element, node(index));
}

public boolean addAll(Collection<? extends E> c) {		// 将集合c的元素全部加入到当前对象的链表尾
    return addAll(size, c);		// 就是去调用下面的方法
}

// 将集合c的元素全部加入到当前对象的链表指定index对应的节点的前面
// false，表示c数据为空，那么add失败；true：add成功
public boolean addAll(int index, Collection<? extends E> c) {	
    checkPositionIndex(index);		// 判断参数是否合法0~size范围内？

    Object[] a = c.toArray();	// 转换成数组后再操作
    int numNew = a.length;
    if (numNew == 0)		// 数组长度为0——数据为空，直接返回失败
        return false;

    Node<E> pred, succ;
    if (index == size) {		// 如果index指向就是链表尾，那么后继就为空，前驱就是链表尾
        succ = null;
        pred = last;
    } else {
        succ = node(index);		// 找index对应的节点——是后继
        pred = succ.prev;		// 该index的节点的前驱——就是插入节点的前驱
    }

    for (Object o : a) {		// 遍历数组
        @SuppressWarnings("unchecked") E e = (E) o;		// 强转一下，当前是Object类型，强转只能是E本身or其子类
        Node<E> newNode = new Node<>(pred, e, null);
        // 出现在index是第一个节点，其前驱为null；且插入第一个节点，那么此时的pred=null/此时为空链表，那么pred=last=null
        if (pred == null)		
            first = newNode;	// 更新链表头
        else
            pred.next = newNode;	// 前驱的next指向新节点
        pred = newNode;		// 前驱更新到新节点
    }

    if (succ == null) {		// 如果插入的地方是链表尾，那么更新last节点
        last = pred;
    } else {			// 如果插入的不是链表尾，那么只需要更新index的前驱和object数组的最后一个的后继
        pred.next = succ;
        succ.prev = pred;
    }

    size += numNew;		// 更新链表元素个数
    modCount++;		// 表示链表结构发生变化
    return true;
}
```

### （2）删

```java
public E remove(int index) {
    checkElementIndex(index);		// 边界检查[0, size)
    return unlink(node(index));		// 先根据index找到该节点，然后执行该节点的移除操作，最后返回该节点的值
}

public boolean remove(Object o) {		// 删除指定对象——提供的是item的值，需要先找到对应的节点，再删除
    if (o == null) {			// 值为null特殊处理
        for (Node<E> x = first; x != null; x = x.next) {	// 从头开始遍历找
            if (x.item == null) {		// 找到第一个值为null的，将其从中删除
                unlink(x);
                return true;
            }
        }
    } else {			
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {			// 值为非null，需要进行equals判断
                unlink(x);
                return true;
            }
        }
    }
    return false;
}

public void clear() {
    // Clearing all of the links between nodes is "unnecessary", but:
    // - helps a generational GC if the discarded nodes inhabit
    //   more than one generation
    // - is sure to free memory even if there is a reachable Iterator——与JVM的垃圾回收相关
    for (Node<E> x = first; x != null; ) {	// 从链表头开始遍历到第一个null之前（注意node=null和node.item=null不等价）
        Node<E> next = x.next;
        x.item = null;		// 将其全部置为null，然后让GC去处理它们——虽然没有必要清除链接，但是清除了能加快回收
        x.next = null;
        x.prev = null;
        x = next;
    }
    first = last = null;
    size = 0;
    modCount++;
}

// 删除第一个遇到值为o的节点
public boolean removeFirstOccurrence(Object o) {
    return remove(o);		// 从头开始找并删除
}
// 删除最后一个遇到的值为o的节点
public boolean removeLastOccurrence(Object o) {		// 从尾部开始找并删除
    if (o == null) {
        for (Node<E> x = last; x != null; x = x.prev) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = last; x != null; x = x.prev) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
```

### （3）改

```java
public E set(int index, E element) {		// 将对应索引的节点值进行修改
    checkElementIndex(index);		// 判断传参是否正确[0,size)
    Node<E> x = node(index);		// 找到该节点
    E oldVal = x.item;
    x.item = element;
    return oldVal;		// 返回旧的值
}
```

### （4）查

```java
// 获得对应索引的元素值——注意获得是值
public E get(int index) {
    checkElementIndex(index);		
    return node(index).item;		// 从头or尾开始遍历查找
}

// 在链表中查找指定索引的元素——返回该节点，注意返回的是节点，而不是节点的值
Node<E> node(int index) {		
    // assert isElementIndex(index);

    if (index < (size >> 1)) {		// 如果index未到链表长度的一半，那么从头开始找——找后继
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {				// 如果index>=链表长度的一半，那么从尾巴开始找——找前驱
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}

// 获得该元素”值“所在的链表的位置——主要是为了统一（其实链表不应该计算index值的），index还是从0开始计算的（为了容器的统一）
public int indexOf(Object o) {
    int index = 0;
    if (o == null) {			// 值为null就特殊处理
        for (Node<E> x = first; x != null; x = x.next) {	// 从头到尾找到第一个值为null的
            if (x.item == null)
                return index;
            index++;
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item))
                return index;
            index++;
        }
    }
    return -1;
}

public int lastIndexOf(Object o) {		// 同上，尾巴开始找，也是从0开始计算的，所以需要先减后判断
    int index = size;
    if (o == null) {
        for (Node<E> x = last; x != null; x = x.prev) {
            index--;
            if (x.item == null)
                return index;
        }
    } else {
        for (Node<E> x = last; x != null; x = x.prev) {
            index--;		// 先--
            if (o.equals(x.item))
                return index;
        }
    }
    return -1;
}

// 判断该节点是否存在于链表中
public boolean contains(Object o) {
    return indexOf(o) != -1;		// 本质上就是去调用indexOf去找索引值，返回非负值就说明存在
}
```

ps：查找时间复杂度为O(N/2)=O(N)

```java
public int size() {
    return size;
}
```

### （5）栈Stack特点的操作

后进先出

需要记忆方法名字：`push()`|`pop()`|`peek()`

```java
// 查看栈顶元素的值——注意不是返回节点，而是返回节点的值，栈为空直接返回null，不会抛出异常（与getFirst不同）
public E peek() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}

// 压栈，元素放到栈顶——其实就是将元素放入头部
public void push(E e) {
    addFirst(e);
}

// 出栈，栈顶元素出——其实就是将头节点移出
// 注意，如果栈为空，下面会抛出异常（没有catch，会上抛）
public E pop() {
    return removeFirst();	// 将链表头节点从链表中删除，释放所占用的资源并更新链表，返回该节点的内容
}
```

### （6）队列Queue特点的操作

先进先出

需要记忆的方法：`peek()`|`element()`|`poll()`|`remove()`|`offer()`

```java
// 查看队列头的值——注意不是返回节点，而是返回节点的值，队列为空直接返回null，不会抛出异常（与getFirst不同）
public E peek() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}

// 同上，队列为空抛出异常
public E element() {
    return getFirst();	
}

// 删除队头，并返回节点值，队列为空直接返回null，不会抛出异常（与removeFirst不同）
public E poll() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}
// 同上，队列为空，会抛出异常
public E remove() {
    return removeFirst();
}

// 在队尾加入节点
public boolean offer(E e) {
    return add(e);
}
```

### （7）双端操作Deque

由于LinkedList是双向链表，所以在两端进行操作的时间复杂度是一样的。所以提供了两端操作的API

总结这些方法：`getxxx`|`peekxxx`|`addxxx`|`offerxxx`|`removexxx`|`pollxxx`——一共12个方法，对应对首、尾的操作，可以划分为6组，而6组又可以分为3组，get/peek, add/offer,remove/poll，分别代表查，增，删，组里的前者会因为链表为空而抛出异常，会因为满而抛出抛出异常；后者不会

ps：add/offer正常都不会抛出异常，如果要抛出异常，是因为有些deque的实现限制了列表的长度

```java
// 获得链表头节点指向的节点的值——注意是内容（同stack的peek，peek当栈为空时，不会抛出异常）
public E getFirst() {		
    final Node<E> f = first;
    if (f == null)		// 链表为空，会抛出异常
        throw new NoSuchElementException();
    return f.item;
}
// 获得链表尾节点指向的节点的值——注意是内容
public E getLast() {
    final Node<E> l = last;
    if (l == null)		// 链表为空，会抛出异常
        throw new NoSuchElementException();
    return l.item;
}

public E peekFirst() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}
public E peekLast() {
    final Node<E> l = last;
    return (l == null) ? null : l.item;
}

// 在链表头插入新节点（同stack的push）
public void addFirst(E e) {
    linkFirst(e);
}
// 在链表尾插入新节点
public void addLast(E e) {
    linkLast(e);
}
// 本质上同addFirst，只不过返回的true，而addFirst无返回
public boolean offerFirst(E e) {
    addFirst(e);
    return true;
}
// 同上
public boolean offerLast(E e) {
    addLast(e);
    return true;
}

// 移除链表头节点，返回节点内容
public E removeFirst() {
    final Node<E> f = first;
    if (f == null)		// 出现的情况——链表为空——会抛出异常
        throw new NoSuchElementException();
    return unlinkFirst(f);
}
// 移除链表尾节点，返回节点内容
public E removeLast() {
    final Node<E> l = last;
    if (l == null)		// 链表为空——会抛出异常
        throw new NoSuchElementException();
    return unlinkLast(l);
}

public E pollFirst() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}
public E pollLast() {
    final Node<E> l = last;
    return (l == null) ? null : unlinkLast(l);
}
```

### （8）内部私有操作——辅助方法

```java
private void linkFirst(E e) {	// 将该节点插入到链表头前，作为新链表头节点
    final Node<E> f = first;
    final Node<E> newNode = new Node<>(null, e, f);
    first = newNode;
    if (f == null)		// 本来链表为空的，那么需要更新last也指向该新节点
        last = newNode;
    else
        f.prev = newNode;
    size++;
    modCount++;
}

void linkLast(E e) {		// 将该节点插入到链表尾后，作为新链表尾节点
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)		// 本来链表为空的，那么需要更新first也指向该新节点
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}

void linkBefore(E e, Node<E> succ) {	// 节点插入到后继succ之前——前提是succ是存在的，而不是null
    // assert succ != null;
    final Node<E> pred = succ.prev;
    final Node<E> newNode = new Node<>(pred, e, succ);
    succ.prev = newNode;
    if (pred == null)			// 如果succ不存在前驱，那么succ本来是头结点，现在newNode变成头节点
        first = newNode;
    else
        pred.next = newNode;
    size++;
    modCount++;
}

// 链表头节点取出操作
// 将头结点的内容和指向全部赋值为null（方便GC回收），更新first和last（看需要），更新计数和修改数
private E unlinkFirst(Node<E> f) {		
    // assert f == first && f != null;
    final E element = f.item;
    final Node<E> next = f.next;
    f.item = null;
    f.next = null; // help GC
    first = next;
    if (next == null)// 如果头节点取出后链表为空，那么链表尾也应该为null（否则，last还指向原来的f，无法正确回收节点f）
        last = null;
    else
        next.prev = null;
    size--;
    modCount++;
    return element;
}
// 链表尾节点取出操作——操作同上，只不过在尾部操作
private E unlinkLast(Node<E> l) {
    // assert l == last && l != null;
    final E element = l.item;
    final Node<E> prev = l.prev;
    l.item = null;
    l.prev = null; // help GC
    last = prev;
    if (prev == null)	// 如果尾节点取出后链表为空，那么链表头也应该为null（否则，first还指向原来的f，无法正确回收节点f）
        first = null;
    else
        prev.next = null;
    size--;
    modCount++;
    return element;
}

// 删除链表中间节点x，让x的前驱和x的后继相连，然后删除x的prev、next、item的引用，更新计数和修改计数并看必要更新first、last
E unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;

    if (prev == null) {		// x不存在前驱，说明是链表头
        first = next;
    } else {
        prev.next = next;
        x.prev = null;		// 取消引用
    }

    if (next == null) {		// x不存在后继，说明是链表尾（上面的判断同时满足，说明删除之后，链表为空，last=first=null）
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;		// 取消引用
    }

    x.item = null;		// 取消引用
    size--;
    modCount++;
    return element;
}

// 边界检查——主要是针对add操作的
private void checkPositionIndex(int index) {
    if (!isPositionIndex(index))		// 如果不在合适范围内，就抛出越界异常
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
private boolean isPositionIndex(int index) {		// 判断当前索引是否在合适的范围内0~size内
    return index >= 0 && index <= size;
}

// 边界检查——主要是针对非add操作，且传参index的，需要对该参数进行判断
private void checkElementIndex(int index) {
    if (!isElementIndex(index))			// 不能超过范围
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

private boolean isElementIndex(int index) {
    return index >= 0 && index < size;		// 和isPositionIndex的区别是index<=size是可以取到的（用在add上，可以添加到链表尾）
}
```

### （9）toArray

将链表转换成Object[]数组形式（和ArrayList类似）

```java
public Object[] toArray() {
    Object[] result = new Object[size];
    int i = 0;
    for (Node<E> x = first; x != null; x = x.next)	// 从头开始遍历，将值加入到数组中
        result[i++] = x.item;
    return result;
}

// 将链表元素加入到指定的数组a中，T是和E不一样的类型参数
public <T> T[] toArray(T[] a) {
    if (a.length < size)		// 如果传进来的数组不够长，那么需要创建一个指定长度的数组
        a = (T[])java.lang.reflect.Array.newInstance(
        a.getClass().getComponentType(), size);
    int i = 0;
    Object[] result = a;
    for (Node<E> x = first; x != null; x = x.next)
        result[i++] = x.item;

    if (a.length > size)		// 如果数组过长，那么在结束之后添上null，表明值已经结束了，不需要再看后面的值了（为了遍历时候节省时间，遇到null就可以停止了，那么如果值本来就为null？——还是存在问题的）
        a[size] = null;

    return a;
}
```

- [ ] `java.lang.reflect.Array.newInstance();` 有待研究
- [ ] 如果数组过长，那么在结束之后添上null，表明值已经结束了，那么如果本来在数组中存在值为null呢？ 所以如果要遍历该数组，还是要看size的记录，而不是判断是否出现了null——目前的理解是这样

### （10）流操作（暂不研究）

发现是private方法，而内部又没有去调用它的方法，不知道作用在哪里。待研究其：

- [ ] Stream操作
- [ ] 定义的意义和使用场景

```java
// 将链表写入流
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException {
    // Write out any hidden serialization magic
    s.defaultWriteObject();

    // Write out size
    s.writeInt(size);

    // Write out all elements in the proper order.
    for (Node<E> x = first; x != null; x = x.next)
        s.writeObject(x.item);
}

/**
     * Reconstitutes this {@code LinkedList} instance from a stream
     * (that is, deserializes it).
     */
@SuppressWarnings("unchecked")
// 从流中取出链表，加入到链表尾
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    // Read in any hidden serialization magic
    s.defaultReadObject();

    // Read in size
    int size = s.readInt();

    // Read in all elements in the proper order.
    for (int i = 0; i < size; i++)
        linkLast((E)s.readObject());
}
```

### （11）clone——浅拷贝

```java
public Object clone() {
    LinkedList<E> clone = superClone();

    // Put clone into "virgin" state
    clone.first = clone.last = null;		// 初始化一下状态值
    clone.size = 0;
    clone.modCount = 0;

    // Initialize clone with our elements
    for (Node<E> x = first; x != null; x = x.next)	// 遍历每个节点，将值加入到新链表中，item只是地址复制
        clone.add(x.item);

    return clone;
}

private LinkedList<E> superClone() {
    try {
        return (LinkedList<E>) super.clone();
    } catch (CloneNotSupportedException e) {
        throw new InternalError(e);
    }
}

protected native Object clone() throws CloneNotSupportedException;	// 调用native方法
```

ps：本质上还是浅拷贝，虽然有新创建一个链表，但是里面的item值还是对该对象的一个引用，即两个链表的item指向的地址都是同一个

### （12）接口实现ListIterator

```
public ListIterator<E> listIterator(int index) {
    checkPositionIndex(index);
    return new ListItr(index);
}

```

和ArrayList类似，但是只实现了一个ListItr，而没有实现普通的迭代器

```java
// 方法内部类
private class ListItr implements ListIterator<E> {
    private Node<E> lastReturned;	// 保存上一次返回的值
    private Node<E> next;		// 指向下一个节点（下次循环就是cur节点）
    private int nextIndex;		// 下一个节点对应的索引值（下次循环就是cur索引）
    private int expectedModCount = modCount;

    ListItr(int index) {			// 构造方法
        // assert isPositionIndex(index);
        // 如果传参过来的指向了尾节点后面，那么next就只能指向null，否则就找到index对应的节点
        next = (index == size) ? null : node(index);
        nextIndex = index;
    }

    public boolean hasNext() {		// 判断是否遍历到了链表尾
        return nextIndex < size;
    }

    public E next() {		// lastreturn=next-1
        checkForComodification();		// 检查并发
        if (!hasNext())		// 如果已经遍历到链表尾，那么抛出异常
            throw new NoSuchElementException();

        lastReturned = next;	// 更新上一次返回的值，变成当前节点
        next = next.next;		// 更新next指向下一个节点
        nextIndex++;
        return lastReturned.item;
    }

    public boolean hasPrevious() {	// 判断是否存在前驱节点（除了链表头无前驱）
        return nextIndex > 0;
    }

    public E previous() {		// next的相反操作——lastreturn=next
        checkForComodification();
        if (!hasPrevious())
            throw new NoSuchElementException();
		// 更新上一次返回的节点，为当前节点
        lastReturned = next = (next == null) ? last : next.prev;	// 如果cur为null，那么就是链表尾节点的后面一个，那么赋值链表尾节点——因为null是找不到前驱和后继的；非null，就返回前驱节点，这个操作主要针对构造方法里面的初始化设置为null，那么这边也要对应设置
        nextIndex--;
        return lastReturned.item;
    }

    public int nextIndex() {
        return nextIndex;		// 针对next方向，那么next即下一个
    }

    public int previousIndex() {
        return nextIndex - 1;	// 针对previous方向，那么需要-1，表示前一个
    }

    public void remove() {		// 删除上一个返回的节点——考虑了不同方向的删除方法
        checkForComodification();
        if (lastReturned == null)
            throw new IllegalStateException();

        Node<E> lastNext = lastReturned.next;
        unlink(lastReturned);		// 将该节点移除
        // 保证next和nextIndex指向同一个节点，next()方向进行删除——那么刚刚删除的就是next的前驱节点，那么next = lastreturned.next（next不需变化）,而当前的next节点在链表中中的相对位置发生变化，那么需要将nextIndex--，以统一；previous()方向删除——那么刚刚删除的是next本身，那么需要指向其后一个节点（算是遍历过的节点，逆序遍历的next就是指向当前节点），但是nextIndex不能变化
        if (next == lastReturned)	
            next = lastNext;
        else
            nextIndex--;
        lastReturned = null;// 置为null，标记为此次循环已经修改过，不能再进行操作，包括set、remove，但是可以add
        expectedModCount++;// 为了和modCount统一
    }

    public void set(E e) {		// 修改上一次返回的值，可以在调用这个之后再调用add/remove
        if (lastReturned == null)// 如果已经修改过（add/remove），会抛出异常，一定要在next/previous操作之后才行
            throw new IllegalStateException();
        checkForComodification();
        lastReturned.item = e;
    }

    public void add(E e) {
        checkForComodification();
        lastReturned = null; // 置为null，标记为此次循环已经修改过，不能再进行操作，包括set和remove（但是还是可以add）
        if (next == null)		// 当前节点在链表尾后一个节点了，那么将该节点加入到链表尾
            linkLast(e);
        else
            linkBefore(e, next);	// 插入在当前节点之前，next不更新——next在next()\previous()更新
        nextIndex++;
        expectedModCount++;		// 为了和modCount统一
    }

    public void forEachRemaining(Consumer<? super E> action) {		// 将剩下的元素全部进行同样的操作——也顺带着遍历
        Objects.requireNonNull(action);
        while (modCount == expectedModCount && nextIndex < size) {
            action.accept(next.item);
            lastReturned = next;
            next = next.next;
            nextIndex++;
        }
        checkForComodification();
    }

    final void checkForComodification() {			// 检查是否出现并发操作（和ArrayList一致）
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```

ps：同样，checkForComodification在任何操作前，都会进行调用，所以在迭代器中需要修改链表数据结构就一定要通过迭代器提供的方法，否则马上就会抛出异常；而且这个也会对并发进行判断，例如两个线程同时通过迭代器进行remove操作等，那么也会抛出异常（但不能发现所有的并发异常，主要是因为性能考虑）

pps：这个双向的链表操作思路，尤其是`remove()`操作实现很优秀，通过一个判断就能处理两个方向的循环——可以学习学习

```Java
public Iterator<E> descendingIterator() {
    return new DescendingIterator();
}

// 实现逆序遍历，实现Iterator方法：本质上是去调用了ListItr的方法
private class DescendingIterator implements Iterator<E> {
    private final ListItr itr = new ListItr(size());	// 本质上调用了ListItr的实现
    public boolean hasNext() {
        return itr.hasPrevious();	// 实际上是去获取previous
    }
    public E next() {
        return itr.previous();
    }
    public void remove() {
        itr.remove();
    }
}
```

### （13）spliterator接口实现（暂放）

```java
@Override
public Spliterator<E> spliterator() {
    return new LLSpliterator<E>(this, -1, 0);
}

/** A customized variant of Spliterators.IteratorSpliterator */
static final class LLSpliterator<E> implements Spliterator<E> {
    static final int BATCH_UNIT = 1 << 10;  // batch array size increment
    static final int MAX_BATCH = 1 << 25;  // max batch array size;
    final LinkedList<E> list; // null OK unless traversed
    Node<E> current;      // current node; null until initialized
    int est;              // size estimate; -1 until first needed
    int expectedModCount; // initialized when est set
    int batch;            // batch size for splits

    LLSpliterator(LinkedList<E> list, int est, int expectedModCount) {
        this.list = list;
        this.est = est;
        this.expectedModCount = expectedModCount;
    }

    final int getEst() {
        int s; // force initialization
        final LinkedList<E> lst;
        if ((s = est) < 0) {
            if ((lst = list) == null)
                s = est = 0;
            else {
                expectedModCount = lst.modCount;
                current = lst.first;
                s = est = lst.size;
            }
        }
        return s;
    }

    public long estimateSize() { return (long) getEst(); }

    public Spliterator<E> trySplit() {
        Node<E> p;
        int s = getEst();
        if (s > 1 && (p = current) != null) {
            int n = batch + BATCH_UNIT;
            if (n > s)
                n = s;
            if (n > MAX_BATCH)
                n = MAX_BATCH;
            Object[] a = new Object[n];
            int j = 0;
            do { a[j++] = p.item; } while ((p = p.next) != null && j < n);
            current = p;
            batch = j;
            est = s - j;
            return Spliterators.spliterator(a, 0, j, Spliterator.ORDERED);
        }
        return null;
    }

    public void forEachRemaining(Consumer<? super E> action) {
        Node<E> p; int n;
        if (action == null) throw new NullPointerException();
        if ((n = getEst()) > 0 && (p = current) != null) {
            current = null;
            est = 0;
            do {
                E e = p.item;
                p = p.next;
                action.accept(e);
            } while (p != null && --n > 0);
        }
        if (list.modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }

    public boolean tryAdvance(Consumer<? super E> action) {
        Node<E> p;
        if (action == null) throw new NullPointerException();
        if (getEst() > 0 && (p = current) != null) {
            --est;
            E e = p.item;
            current = p.next;
            action.accept(e);
            if (list.modCount != expectedModCount)
                throw new ConcurrentModificationException();
            return true;
        }
        return false;
    }

    public int characteristics() {
        return Spliterator.ORDERED | Spliterator.SIZED | Spliterator.SUBSIZED;
    }
}
```

## 总结：

读过ArrayList再读这个就感觉简单很多，主要是链表的一些操作本身就比较简单直观

ps：LinkedList也是支持元素值为null，会对null进行特殊处理