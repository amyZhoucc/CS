# ArrayList源码分析

总结：

1. ArrayList是**非线程安全的**（即多个线程同时修改里面的内容，会发生并发问题），如果要并发修改，需要使用synchronized进行同步。Vector是最早实现的集合类，基本原理和ArrayList类似，内部使用同步锁实现线程安全。但是性能有所降低。所以不需要考虑并发的情况下，ArrayList优先。
2. 可能抛出的异常：**ConcurrentModificationException**：并发修改异常。（在迭代器遍历时，如果另外一个线程修改了数组的结构，就会触发该异常）
3. ArrayList 的**容量会根据列表大小自动调整**，每次扩容大小起码是1.5倍。在添加大量元素之前，可以使用ensureCapacity 方法来保证列表有足够空间存放元素。
4. **ArrayList可以存放null**
5. 由于内部是数组，所以可以实现O(1)时间的查找
6. ArrayList是一个**泛型容器**，新建ArrayList需要实例化泛型参数，eg：`ArrayList<Integer>arr = new ArrayList<>();` 
7. 注意和LinkedList做区分

## 1. 类头：

继承自AbstractList

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable{}
```

继承自：

实现：Serializable：序列化接口；Cloneable：可进行克隆；RandomAccess 也是一个**标记接口**（本身没有方法，只是做标记，**用于声明类的一种属性**），实现 RandomAccess 表示该类支持快速随机访问——实际上什么也没有实现，是底层数据结构决定的，表明可以随机访问

标明了随机访问的优点：可以用于一些**通用的算法代码中，它可以根据这个声明而选择效率更高的实现**。比如，**Collections类中有一个方法binarySearch**，在List中进行二分查找，它的实现代码就**根据list是否实现了RandomAccess而采用不同的实现机制**

## 2. 静态变量

```java
private static final int DEFAULT_CAPACITY = 10;	// 默认的数组长度为10

// 数组长度的上限（有些JVM需要在数组中保存一些标题字，head words，所以需将这些空间预留，再次申请会触发OOM异常） 
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;	
```

默认的数组大小是10，但是创建的时候，**除非指定数组大小（构造方法2）/传递一个实例变量过来（构造方法3），都不会默认分配10大小**，主要是为了节省空间

```java
private static final Object[] EMPTY_ELEMENTDATA = {};	// 空数组实例，和下面的区分
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};	// 默认的共享的空数组实例（和上面区分，主要是为了看添加第一个元素的时候需要将数组扩展到多大）
```

`EMPTY_ELEMENTDATA`和`DEFAULTCAPACITY_EMPTY_ELEMENTDATA`两个，前一个是通过第二个构造函数产生的，即`new ArrayList(0)`；后一个是无参构造方法产生的，即`new ArrayList();`

## 3. 实例变量

```Java
transient Object[] elementData; // 内部私有数组，如果当前大小不够了会重新申请一个数组，那么elementData修改引用即可
```

elementDate就是内部实际存储的数据元素的地方

why设置为object类型，因为在编写ArrayList代码时，并不知道会传入何种类型的对象，所以统一设置为object方便扩展。若只允许特定的类型，会妨碍它成为一个“常规用途”的工具，为用户带来麻烦

```java
private int size;// 记录数组当前元素个数，私有，通过size()访问
```

需要注意的是：size是指示数组实际存储的元素个数，一般会小于数组实际长度，即`size <= arr.length`

```java
protected transient int modCount = 0;		// 记录数组的结构性变化次数——是继承自AbstractList的
```

来**记录修改次数**，主要是在**使用迭代器遍历**的时候，来记录那些发生结构性变化的操作，主要用在**多线程环境下**，防止一个线程在迭代而另一个线程在修改结构。——但是这个只能仅最大可能发现。

modCount出现的函数记录：`trimToSize`，`ensureExplicitCapacity`（所有add操作中用到）、`remove`、`fastRemove`（remove操作中用到）、`removeRange`、`clear`

## 4. 构造方法

### 4.1 无参构造方法

```java
public ArrayList() {		// 无参数构造方法
    // 直接传递空数组实例过去，此时size=0——第二个空数组
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;	
}
```

默认是空数组，没有指定数组的长度和内容

当添加第一个元素时，如果为DEFAULTCAPACITY_EMPTY_ELEMENTDATA的数组，会将扩充到默认大小DEFAULT_CAPACITY（10）。

### 4.2 带参构造方法

指定初始长度：

如果指定的长度>0，那么会创建指定长度的数组；如果长度=0，那么传递的是默认的空数组

```java
public ArrayList(int initialCapacity) {	// 传递了数组的初始长度
    if (initialCapacity > 0) {		// 传递的初始长度>0
        this.elementData = new Object[initialCapacity];	// 创建指定长度的数组
    } else if (initialCapacity == 0) {		// 初始长度为0，那么传递一个空数组实例即可——这便是第一个空数组（而不是第二个）
        this.elementData = EMPTY_ELEMENTDATA;
    } else {			// 抛出异常——非法传参
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}
```

指定传入一个集合类型的对象，eg：ArrayList/LinkedList，本质上就是将集合对象转换为数组，然后直接将数组首地址赋值给elementData

```java
public ArrayList(Collection<? extends E> c) {		// 传递了一个ArrayList对象
    elementData = c.toArray();		// 将传递来的容器转换成数组形式
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

ps：构造方法3,`elementData.getClass() != Object[].class`需要额外进行判断，主要是存在		**`Arrays.asList(xxx).toArray()`**，Arrays.asList()会将一个数组转换成ArrayList对象，而这个ArrayList对象是Arrays内部实现的一个类，和这边的ArrayList有所不同，所以toArray的实现也是不同，是直接返回了一个泛型数组`E[] a`，所以需要将数组复制成`Object[]`类型。

## 5. 实例方法

### （1）trimToSize()：缩小容量

- 容量常常会大于实际元素的数量（length>size）。**内存紧张**时，可以调用该方法删除预留的位置，调整容量为元素实际数量；
- 如果确定不会再有元素添加进来时也可以调用该方法来节约空间

=> 所以使用场景是：**内存紧张时/数组元素个数确定不变时**

```java
public void trimToSize() {
    modCount++;				// 修改次数++，表明该数组结构性发生变化了
    if (size < elementData.length) {	// 说明存在缩容的余地
        elementData = (size == 0)		// 如果此时元素个数为0（前面有过元素，后面都被删除完了）——直接赋值为空数组
            ? EMPTY_ELEMENTDATA			// 如果不为0，还存在元素，那么产生一个新数组，大小正好为元素个数
            : Arrays.copyOf(elementData, size);
    }
}
```

——本质上是创建一个新的数组存储，且数组大小完全按照结点个数进行申请

### （2）ensureCapacityInternal()：扩容相关

因为涉及到较多方法之间的传递，需要明白如下：

- mincapacity：代表增加元素后**实际的总元素个数**
- newcapacity：代表能满足min下的需要扩容的大小
- 在扩容时，**默认的扩容因子是1.5**，每次需要扩容时，会将原数组大小的1.5倍（newcapacity）和实际需要的数组空间（mincapacity）进行比较，从中取最大值作为数组大小

```java
// 用来确认elementData数组是否满足minCapacity大小，如果不满足就会进行扩容
private void ensureCapacityInternal(int minCapacity) {	
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

// 计算需要扩容的大小——主要是为了处理第一次扩容的/非第一次扩容的情况（会员首充便宜很类似）
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {	//如果该数组为第一次使用
        return Math.max(DEFAULT_CAPACITY, minCapacity);		// 10和minCapacity挑大的——首次使用最小扩容10个或以上（需要单独判断的原因是：因为本来长度为0，1.5*0=0，防止扩容扩个寂寞）
    }
    return minCapacity;		
}

// 判断是否需要扩容，如果大小还够就不扩容了，如果不够就去扩容
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;		// 内部的修改次数++

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)// 如果最小需要的长度大于当前的数组长度，那么就去扩容——如果数组空间还够就不扩容了
        grow(minCapacity);
}

// 实际去扩容
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

// 主要为了处理已经超过数组大小上限的情况：最小容量溢出那么抛出异常；如果未溢出，那么就给一个上界值
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow		// 说明该值已经溢出（超过int正数的上界，变成了负数）
        throw new OutOfMemoryError();		// 抛出OOM异常
    // 如果在int上界-8 ~ int上界范围内，那么就给int上界的大小（有种再给一次机会的感觉）；如果不在（应该不会进入）
    return (minCapacity > MAX_ARRAY_SIZE) ? Integer.MAX_VALUE : MAX_ARRAY_SIZE;
}
```

理解：

1. 可以看到，如果ArrayList通过无参构造方法创建，在第一次扩容（第一次加入内容时），最小扩容为10
2. 扩容在这里面是最耗费时间的操作，**不仅仅需要重新分配空间，而且需要重新赋值**。
3. 这边如果触发扩容，就会改变`elementData`，而其他无变化，那么**对应的`arr.length`变化（数组本身提供的），而size是不会变化的**

### （3）查

都比较简单，稍微过一遍即可

查询ArrayList的存储结点个数、查询是否为空

```java
// ——Collection接口定义的需要实现的
public int size() {		
    return size;		// 直接返回实例变量即可：size记录的是当前数组中元素个数
}

// ——Collection接口定义的需要实现的
public boolean isEmpty() {		// 判断数组是否为空——这边不是看数组长度，而是看数组中存在的元素个数是否为0
    return size == 0;
}
```

查找ArrayList是否存在某个对象，或者某个对象的index/lastIndex

```java
// ——Collection接口定义的需要实现的
public boolean contains(Object o) {		// 查找是否存在指定的元素，不存在false；存在true
    return indexOf(o) >= 0;		// 如果返回值为非-1，那么说明存在
}

// 查找第一个符合的就返回对应的索引，O(N)
public int indexOf(Object o) {
    if (o == null) {			// 对存放了null的元素做了额外的处理——主要是因为null不是实例对象，无法调用equals方法
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;		// 找不到就返回-1
}

// 查找最后一个符合要求的就返回对应的索引
public int lastIndexOf(Object o) {
    if (o == null) {
        for (int i = size-1; i >= 0; i--)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = size-1; i >= 0; i--)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```

ps：从`indexOf`可以发现：**ArrayList中可以存放null值**，在比较时对null值都进行了处理，后面的`remove(object obj)`也符合这个情况

根据index去获取对应下标的对象

```java
public E get(int index) {		// 获得数组中下标对应的元素
    // 边界检查——只检查是否超过上界，下界不检查——数组会进行下界越界异常:ArrayIndexOutOfBoundsException
    rangeCheck(index); 

    return elementData(index);		// 返回下标对应的元素
}
// 检查是否访问了数组未存放元素的位置，能返回值，但是是null（容易和本身传入null混淆）——主要是防止返回的内容引起混淆
private void rangeCheck(int index) {
    if (index >= size)		
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

E elementData(int index) {		// 求对应索引位置的元素——不进行边界检查
    return (E) elementData[index];
}
```

### （4）增

针对add的边界检查——需要检查上下界，和上面的rangeCheck有所不同

```java
// 针对add的边界检查——需要检查上下界，如果越界就要抛出特定异常进行提示
// 与之对比的是，（3）中的边界检查只检查上界，下界就让数组本身机制操作
private void rangeCheckForAdd(int index) {
    if (index > size || index < 0)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
private String outOfBoundsMsg(int index) {
    return "Index: "+index+", Size: "+size;
}
```

添加：——均存在扩容的可能

添加到列表尾

```java
// 在数组末尾添加元素——可能存在扩容问题——大部分时间都花在此处
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // 确定是否当前容量是否满足——注意这边modCount++——因为数组结构发生变化
    elementData[size++] = e;	// 添加元素，并且更新最新的size
    return true;
}
```

添加到指定index，那么index后面的数组需要全部移动1位

```java
// ——Collection接口定义的需要实现的
// 在数组的指定位置添加元素element
public void add(int index, E element) {
    rangeCheckForAdd(index);		// 检查该索引位置是否上下越界

    ensureCapacityInternal(size + 1);  // 确定是否当前容量是否满足——注意这边modCount++——因为数组结构发生变化
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);	// index及其之后的元素都需要移动到index+1后面
    elementData[index] = element;	// 空出来的index给element
    size++;		// 计数++
}
```

添加整个集合到当前对象，可以指定起始index，那么index及其之后的数组需要移动n位

```java
// ——Collection接口定义的需要实现的
// 将包装类c的内容加入到本对象的数组中
public boolean addAll(Collection<? extends E> c) {		// c的类型参数必须是E的子类或E本身——这样才能加入到数组中去
    Object[] a = c.toArray();		// 将c转换成数组类型
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // ——modCount++，表示数组结构发生变化
    System.arraycopy(a, 0, elementData, size, numNew);	// 将数组a全部复制到elementData的size后面
    size += numNew;		// 更新size
    return numNew != 0;		// 如果得到的a为空，那么返回false；不为空，那么返回true
}

// 这个主要是加c对象中的内容加入到指定index开始的位置（前面是默认加到数组尾部）
public boolean addAll(int index, Collection<? extends E> c) {
    rangeCheckForAdd(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // ——modCount++

    int numMoved = size - index;	// elementData需要移动的范围——index及其之后的都需要移动到index+numNew开始的位置
    if (numMoved > 0)		// 如果真的需要移动——可能传的参=size，那么不需要移动
        System.arraycopy(elementData, index, elementData, index + numNew,
                         numMoved);		// 先将位置空出来

    System.arraycopy(a, 0, elementData, index, numNew);	// 再将新内容插入进去
    size += numNew;
    return numNew != 0;
}
```

ps：`addAll()`可以发现调用了`toArray()`，但是没有对其结果进行判断`a.getClass() == Object[].class`这边不需要再针对Arrays.asList().toArray()进行处理了，因为这边的操作只是获得数组a，然后将a中的元素后面还是会合并到elementData中的，所以**不会涉及到对a数组内容进行增，所以不会存在类型不匹配的问题**。

### （5）删 

常规的remove

```java
// 删除指定索引的元素，并返回该元素
public E remove(int index) {
    rangeCheck(index);		// 检查index的是否越界（上越界）

    modCount++;		// 因为修改了数组结构，所以需要modCount
    E oldValue = elementData(index);		// 获得该元素——作为返回值返回

    int numMoved = size - index - 1;		// 看需要移动哪些——index之后的元素都需要要移动
    if (numMoved > 0)		// 看是否需要移动——可能删除的是最后一个节点（不需要移动）
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);	// 将index+1~size的移动到index之后
    elementData[--size] = null; // 清除最后一个节点——让GC判断是否需要回收

    return oldValue;
}

// 删除指定对象，true——删除成功；false——删除失败
public boolean remove(Object o) {	// 删除指定对象
    if (o == null) {		// 如果指定对象是null，进行特殊处理——从头开始找第一个为null的元素，然后将其删除
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {	
                fastRemove(index);
                return true;
            }
    } else {		// 普通对象处理
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {		// 如果找到匹配了
                fastRemove(index);		// 调用快速移除操作
                return true;
            }
    }
    return false;		// 如果没有找到就返回false
}
// 快速移除操作——本质和第一个remove一样，主要是写成独立方法后方便直接调用，避免出现代码冗余
private void fastRemove(int index) {
    modCount++;		//——因为结构发生变化，所以需要记录
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // 释放最后一个节点，之后就让GC处理
}

// 删除指定范围的元素[)
protected void removeRange(int fromIndex, int toIndex) {
    modCount++;
    int numMoved = size - toIndex;		// toIndex后面需要移动的数字
    System.arraycopy(elementData, toIndex, elementData, fromIndex,
                     numMoved);		// 直接将后面的元素移动即可，直接覆盖

    // 将最后空出来的位置的元素索引清除——此时可能最后几个元素在数组中存在2个（移动前的，移动后的）——自然的需要将其清除
    int newSize = size - (toIndex-fromIndex);
    for (int i = newSize; i < size; i++) {
        elementData[i] = null;
    }
    size = newSize;		// 更新最新的size
}
```

与集合相关的删除

```java
// ——Collection接口定义的需要实现的
// 删除所有同时出现在集合c中的元素
public boolean removeAll(Collection<?> c) {	
    Objects.requireNonNull(c);		// 判断该对象c是否是null
    return batchRemove(c, false);	
}

// ——Collection接口定义的需要实现的
// 只保留所有出现在C集合中的元素
public boolean retainAll(Collection<?> c) {
    Objects.requireNonNull(c);
    return batchRemove(c, true);
}

// 判断对象是否不存在
public static <T> T requireNonNull(T obj) {
    if (obj == null)
        throw new NullPointerException();	// 如果是null，就抛出异常
    return obj;
}

// 具体的删除操作——是会根据complement进行选择，是删除存在or保留存在的
private boolean batchRemove(Collection<?> c, boolean complement) {
    final Object[] elementData = this.elementData;
    int r = 0, w = 0;		// r：读取的索引值；w：写的索引值
    boolean modified = false;
    try {
        for (; r < size; r++)
            if (c.contains(elementData[r]) == complement)// complement=true——存在即写入；false——不存在即写入
                elementData[w++] = elementData[r];	// 将符合要求的内容写到w对应的索引处，并更新w
    } finally {
        // 这个就是如果触发了异常，那么会导致后面东西无法处理完，那么后面的数据都是乱的，所以需要将r后面的数据全部保存到w后面，使后面的数据仍旧保证正常
        if (r != size) {		// 到这一步，说明可能发生了异常
            System.arraycopy(elementData, r,
                             elementData, w,
                             size - r);
            w += size - r;
        }
        // 到这一步说明数据处理完了or数据即使没有处理完也全部保存完毕了，那么更新哪些最后的数据让其引用减少方便回收
        if (w != size) {		// 说明存在被删除的元素
            // 将最后面的元素全部赋为null，取消对这些元素的引用，让GC去判断是否回收对应的元素
            for (int i = w; i < size; i++)	
                elementData[i] = null;
            modCount += size - w;		// 记录被修改的次数——数组结构发生变化，size-w就是被删除的元素个数
            size = w;		// 更新最新的数组存储元素个数
            modified = true;	// 并标记为修改了
        }
    }
    return modified;
}
```

ps：w和r有点巧妙，虽然也是常规思路，限制了w<=r，所以w如果要写入的话可以直接覆盖掉当前值，因为r已经在w后面了

pps：modCount其他地方都是直接++，而这边是唯一一个地方进行了+(size-w)操作——我的分析可能和并发有关系——留待后续研究

```java
public void clear() {
    modCount++;			// 结构发生改变，就++

    // 将数组中0~size的全部设置为null——表明该arraylist为空
    for (int i = 0; i < size; i++)
        elementData[i] = null;

    size = 0;		// size记录为0——表示此时数组元素个数为0
}
```

### （6）改

```java
// 找到对应的索引，将值设置为element，并返回旧的值——实际上赋值的过程是引用赋值
public E set(int index, E element) {
    rangeCheck(index);		// 参数范围检查

    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
```

#### （7）clone()：浅拷贝

实际上会创建一个新的arrayList对象，然后将数组内容全部复制过去

```java
public Object clone() {
    try {
        ArrayList<?> v = (ArrayList<?>) super.clone();		// 创建一个新的ArrayList变量
        v.elementData = Arrays.copyOf(elementData, size);	// 创建一个新的数组对象，将里面的值全部复制过去
        v.modCount = 0;			// 初始化该修改次数为0
        return v;
    } catch (CloneNotSupportedException e) {
        // ——说是不会发生，因为ArrayList实现了Cloneable接口
        throw new InternalError(e);
    }
}
```

### （8）toArray()：转换成数组——浅拷贝

每个collection对象都需要实现的

```java
// ——Collection接口定义的需要实现的
public Object[] toArray() {
    return Arrays.copyOf(elementData, size);	// 不是直接将内部数组返回——而是创建了一个新数组返回——是浅拷贝
}

// ——Collection接口定义的需要实现的
// 将ArrayList数组数据转存到数组a中，如果a的空间足够则直接存放，否则会新建一个数组进行存储，然后返回a——返回指定数据类型的数组
@SuppressWarnings("unchecked")
public <T> T[] toArray(T[] a) {		// T是和ArrayList的泛型E不同的泛型
    if (a.length < size)
        // a数组太小，就新建一个足够大小的数组复制过去
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());
    System.arraycopy(elementData, 0, a, 0, size);	// 如果a数组大小足够，就直接将elementData内容copy过去
    if (a.length > size)
        a[size] = null;
    return a;
}
```

ps：注意数组类型要求是Object[]类型，因为elementData是Object[]类型

（主要是与之对比的是Arrays内部实现的ArrayList类，就是E[]类型的数组，返回值也是虽然是Obejct[]类型，但是本质还是E[]类型）

### （9）流操作（待研究）

——stream操作还没涉猎到，暂时不管

```Java
// 将列表实例保存到输出流（即序列化）
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();

    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);

    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }

    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}

// 从输入流中读取列表实例
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    elementData = EMPTY_ELEMENTDATA;

    // Read in size, and any hidden stuff
    s.defaultReadObject();

    // Read in capacity
    s.readInt(); // ignored

    if (size > 0) {
        // be like clone(), allocate array based upon size not capacity
        int capacity = calculateCapacity(elementData, size);
        SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
        ensureCapacityInternal(size);

        Object[] a = elementData;
        // Read in all elements in the proper order.
        for (int i=0; i<size; i++) {
            a[i] = s.readObject();
        }
    }
}
```

### （10）sort：排序

对ArrayList进行排序，本质上还是对内部的数组进行排序，并且需要指定比较器

——该方法一般是通过**`Collections.sort`**进行调用的

```java
public void sort(Comparator<? super E> c) {
    final int expectedModCount = modCount;
    Arrays.sort((E[]) elementData, 0, size, c);
    if (modCount != expectedModCount) {				// 防止期间发生数组结构改变的事件
        throw new ConcurrentModificationException();
    }
    modCount++;			//是原地sort，所以改变了数组的结构
}
```



## 7. 接口实现Iterable

ArrayList --继承--> AbstractList --继承--> AbstractCollection --继承--> Collection -- 继承--> Iterable，所以实际上是ArrayList的超类Collection接口继承了iterable——可以不实现，但是在具体类中必须要给实现了

```java
public Iterator<E> iterator() {		// 实现iterator方法，就是实现了Iterable接口
    return new Itr();		// 创建了一个Itr对象——该类就是去实现Iterator接口
}
```

Iterator接口主要是要实现：hasNext()、next()、remove()（有默认方法，可以不实现，如果不实现，表示不支持在遍历的时候进行删除）、forEachRemaining（有默认方法，可以不实现）

两个Iterator接口的default方法

```java
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
```

```java
// ArrayList.java中的接口实现
public Iterator<E> iterator() {
    return new Itr();		// ——创建一个Itr实例对象（是一个私有方法内部类）
}

// 私有方法内部类，实现了Iterator接口
private class Itr implements Iterator<E> {
    int cursor;       // 指向下一个参数——每次都是提前指向next，下一次循环时就是cur
    int lastRet = -1; // 最近一次next调用后返回的索引位置（每次都是最后指向cur，下一次循环就是prev）; 如果没有就是-1——主要是为了防止多次调用remove而出现问题——一次循环中只能删除一个
    int expectedModCount = modCount;	// 初始化时：期望的修改次数 = 默认的修改次数

    Itr() {}		// 构造方法

    public boolean hasNext() {
        return cursor != size;	// size是ArrayList的实例变量之一，cursor=size，就说明已经到头了
    }

    @SuppressWarnings("unchecked")
    public E next() {		// 获得下一个元素
        checkForComodification();	// 是否有结构性变化
        int i = cursor;		// i 指向之前的next节点，那么就是本次的cur
        if (i >= size)			// 如果i已经溢出了——抛出异常	
            throw new NoSuchElementException();
        Object[] elementData = ArrayList.this.elementData;	// 获得ArrayList实际存放元素的数组
        if (i >= elementData.length)		// 如果i已经超过数组的长度——抛出异常（通常来说size<=length），产生这个的原因是进行外部的操作使数组发生结构性变化（例如缩小容量等）——这边再次增加判断的原因是可能会出现并发操作，所以需要再次进行判断
            throw new ConcurrentModificationException();
        cursor = i + 1;		// 更新cursor指向下一个
        return (E) elementData[lastRet = i];		// 返回当前的元素，并且更新lastRet到当前值——那下次循环就是prev
    }
	
    // remove操作一定要在next操作调用过之后才能使用
    public void remove() {			// 移动操作
        if (lastRet < 0)		// -1表明当前节点已经被删除过了，不能再次删除
            throw new IllegalStateException();		// 抛出异常
        checkForComodification();		// 检查是否进行外部修改

        try {
            ArrayList.this.remove(lastRet);		// 还是去调用了外部的remove——将上次返回的节点删除（next中的上次返回节点）
            cursor = lastRet;		// 将当前指针指向lastRet（一般cursor是指向lastRet+1），因为出现了删除，所以最新的lastRet对应索引的元素未被访问过
            lastRet = -1;		// 设置为-1，表明本次循环中已经进行了删除操作，不能重入了
            expectedModCount = modCount;		// 更新expectedModCount使其一致
        } catch (IndexOutOfBoundsException ex) {	// 如果抛出了越界异常，那么向上抛出修改异常——这个越界的原因是由并发修改操作引起的——正常来说lastRet是不会越界的，其本质上就是next处获得的，而next已经进行范围判断了，除非在此过程中进行了删除等操作
            throw new ConcurrentModificationException();
        }
    }

    @Override
    @SuppressWarnings("unchecked")
    // 对剩余的元素都进行指定的consumer操作
    public void forEachRemaining(Consumer<? super E> consumer) {
        Objects.requireNonNull(consumer);		// 判断consumer是否有效
        final int size = ArrayList.this.size;
        int i = cursor;
        if (i >= size) {		// 如果当前指针到头了，那就没有元素要操作了，直接返回
            return;
        }
        final Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length) {		// 并发操作的锅
            throw new ConcurrentModificationException();
        }
        while (i != size && modCount == expectedModCount) {
            consumer.accept((E) elementData[i++]);		// 执行统一操作
        }
        // 迭代结束后才一次性修改，防止频繁进行修改
        cursor = i;
        lastRet = i - 1;
        checkForComodification();		// 再次检查是否再次过程中出现并发操作
    }
	
    // 检查是否进行了外部修改
    final void checkForComodification() {	// 判断是否发生结构性变化——即是否调用外部的add/remove方法
        if (modCount != expectedModCount)	// 主要看当前的修改次数和期待的修改次数是否一致——一致表明无变化，继续
            throw new ConcurrentModificationException();		// 不一致，就抛出异常
    }
}
```

迭代器有如下限制：在迭代的过程中，**如果调用了非迭代器内部的`remove`方法，会抛出`ConcurrentModificationException`异常。想要不抛出这个异常，就必须要使用迭代器内部的`remove`方法**，

ps：前面的`modCount`主要在这边发挥了作用，迭代器每次操作的时候都会去调用——`checkForComodification`：就是检查modCount是否和一致expectedModCount，看是否悄咪咪的调用了外部的add/remove导致数组结构性变化

——可以发现，在代码里面有很多判断操作，并且throw了很多`ConcurrentModificationException`异常，主要是考虑到**并发**的问题——针对一个线程用迭代器进行遍历，而另一个线程修改数组结构，这时候就会触发并发修改异常。因为修改数组结构后，数组遍历将不再正确

pps：但是我觉得这个并发限制做的还是存在问题的，只是粗糙的考虑了一些情况还是存在漏洞的（ArrayList本身也不是线程安全的，尽可能发现即可）

该迭代器实现了`Iterator`接口并实现了相关方法，提供我们对集合的遍历能力。总结：`ArrayList`的迭代器默认是其内部类实现，实现一个自定义迭代器只需要实现`Iterator`接口并实现相关方法即可。而实现`Iterable`接口表示该实现类具有像`for-each loop`迭代遍历的能力。

## 8. 接口实现ListIterator

ListIterator扩展了Iterator接口，增加了一些方法，向前遍历、添加元素、修改元素、返回索引位置等

（iterator只可以删除，从前向后遍历）

```java
public ListIterator<E> listIterator(int index) {
    if (index < 0 || index > size)
        throw new IndexOutOfBoundsException("Index: "+index);
    return new ListItr(index);
}

public ListIterator<E> listIterator() {
    return new ListItr(0);
}
```

是相对于`AbstractList`中优化的版本（注释）：

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

从大方面来看：ListIterator是继承自Itr，它比Itr功能更完善，但是适用范围变窄：

- ListIterator只能用于**List及其子类型**；Iterator可以**应用于所有的集合**，Set、List和Map和这些集合的子类型
- ListIterator可实现顺序向后遍历和顺序向前遍历；Iterator只能实现顺序向后遍历
- ListIterator可以实现remove操作，add操作，set操作；Iterator只能实现remove操作

从小方面来看：通过参数`cursor`、`lastRet`实现了既能顺序遍历又能逆序遍历，且删除和添加之后几乎对遍历无影响，真的很厉害啊:+1:

尤其是`lastRet`的使用，限制删除次数

### why要使用迭代器呢？

（如果要实现遍历可以使用普通的get、size这些方法组合形成）

$\because$ foreach语法更为简洁一些，更重要的是，**迭代器语法更为通用，它适用于各种容器类。**

**并且迭代器体现了一种常见设计模式：关注点分离的思想，将数据的实际组织方式与数据的迭代遍历相分离**

需要访问容器元素只需要对Iterator进行引用，**而不需要关注数据结构，可以使用统一的方式访问**——也体现了封装的思想，只需要调用的简单接口就可以实现复杂的操作

并且Iterator接口实现是了解了数据结构的，所以是**实现了迭代器的高效访问方式**，eg：LinkedList的迭代器性能就比自己写要高的多

## 9. SubList

这个主要是对AbstractList中的SubList的一个重新实现，主要是为了实现`List<E> subList(int fromIndex, int toIndex);`方法的，这个方法在List中定义，然后在AbstractList有过实现

SubList就是将原来的ArrayList截取一部分，返回值还是List类型，仍旧有全套的方法

但是不能将subList类型强转成ArrayList等，因为**subList 返回是一个视图**——并没有重新创建一个list，而是在原来arrayList的基础上进行的修改，只不过修改是限定在指定的范围内了。SubList只是ArrayList的内部类，他们之间并没有集成关系，故无法直接进行强制类型转换。

```java
public List<E> subList(int fromIndex, int toIndex) {
    subListRangeCheck(fromIndex, toIndex, size);		// 范围检查，针对参数的检查
    return new SubList(this, 0, fromIndex, toIndex);	// 然后就去创建SubList实例对象
}

static void subListRangeCheck(int fromIndex, int toIndex, int size) {
    if (fromIndex < 0)
        throw new IndexOutOfBoundsException("fromIndex = " + fromIndex);
    if (toIndex > size)
        throw new IndexOutOfBoundsException("toIndex = " + toIndex);
    if (fromIndex > toIndex)
        throw new IllegalArgumentException("fromIndex(" + fromIndex +
                                           ") > toIndex(" + toIndex + ")");
}
```

```java
// SubList也是从0开始的，而它是截取了ArrayList的一部分，所以需要记录一个相对偏移
// eg:[1,2,3,4,5,6,7]（0~6） =>[3,4,5,6]（0~3），parentOffset=fromIndex=2，那么offset就是2，一般默认是0
private class SubList extends AbstractList<E> implements RandomAccess {
    private final AbstractList<E> parent;
    private final int parentOffset;
    private final int offset;		// 相对于父类ArrayList的偏移量
    int size;			// 自己也有一个size，和ArrayList的size区分：this.size 和ArrayList.this.size

    SubList(AbstractList<E> parent,
            int offset, int fromIndex, int toIndex) {		// 一般默认是0
        this.parent = parent;		// 获得父对象
        this.parentOffset = fromIndex;	// 副本在父对象的起始点
        this.offset = offset + fromIndex;
        this.size = toIndex - fromIndex;
        this.modCount = ArrayList.this.modCount;		// 记录创建该对象时的modCount数
    }
    public E set(int index, E e) {		// 单独定义了set方法
        rangeCheck(index);		// 边界检查
        checkForComodification();		// 并发检查
        E oldValue = ArrayList.this.elementData(offset + index);	// 找到父对象对应的index，然后修改
        ArrayList.this.elementData[offset + index] = e;
        return oldValue;
    }
    public E get(int index) {	// 单独定义了get方法
        rangeCheck(index);
        checkForComodification();
        return ArrayList.this.elementData(offset + index);// 找到父对象对应的index
    }
    public int size() {
        checkForComodification();
        return this.size;
    }
    public void add(int index, E e) {
        rangeCheckForAdd(index);
        checkForComodification();
        parent.add(parentOffset + index, e);
        this.modCount = parent.modCount;	// 更新修改的
        this.size++;
    }
    public E remove(int index) {
        rangeCheck(index);
        checkForComodification();
        E result = parent.remove(parentOffset + index);
        this.modCount = parent.modCount;
        this.size--;
        return result;
    }
    protected void removeRange(int fromIndex, int toIndex) {
        checkForComodification();
        parent.removeRange(parentOffset + fromIndex,
                           parentOffset + toIndex);
        this.modCount = parent.modCount;
        this.size -= toIndex - fromIndex;
    }
    public boolean addAll(Collection<? extends E> c) {
        return addAll(this.size, c);
    }
    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);
        int cSize = c.size();
        if (cSize==0)
            return false;

        checkForComodification();
        parent.addAll(parentOffset + index, c);
        this.modCount = parent.modCount;
        this.size += cSize;
        return true;
    }

    public Iterator<E> iterator() {
        return listIterator();		// 返回的是listIterator
    }

    public ListIterator<E> listIterator(final int index) {		
        checkForComodification();
        rangeCheckForAdd(index);
        final int offset = this.offset;

        return new ListIterator<E>() {	// 内部又去实现了一个匿名内部类，里面的实现和Arraylist的ListIterator基本一样，但是调用的方法是SubList的
            int cursor = index;
            int lastRet = -1;
            int expectedModCount = ArrayList.this.modCount;

            public boolean hasNext() {
                return cursor != SubList.this.size;
            }

            @SuppressWarnings("unchecked")
            public E next() {
                checkForComodification();
                int i = cursor;
                if (i >= SubList.this.size)
                    throw new NoSuchElementException();
                Object[] elementData = ArrayList.this.elementData;
                if (offset + i >= elementData.length)
                    throw new ConcurrentModificationException();
                cursor = i + 1;
                return (E) elementData[offset + (lastRet = i)];
            }

            public boolean hasPrevious() {
                return cursor != 0;
            }

            @SuppressWarnings("unchecked")
            public E previous() {
                checkForComodification();
                int i = cursor - 1;
                if (i < 0)
                    throw new NoSuchElementException();
                Object[] elementData = ArrayList.this.elementData;
                if (offset + i >= elementData.length)
                    throw new ConcurrentModificationException();
                cursor = i;
                return (E) elementData[offset + (lastRet = i)];
            }

            @SuppressWarnings("unchecked")
            public void forEachRemaining(Consumer<? super E> consumer) {
                Objects.requireNonNull(consumer);
                final int size = SubList.this.size;
                int i = cursor;
                if (i >= size) {
                    return;
                }
                final Object[] elementData = ArrayList.this.elementData;
                if (offset + i >= elementData.length) {
                    throw new ConcurrentModificationException();
                }
                while (i != size && modCount == expectedModCount) {
                    consumer.accept((E) elementData[offset + (i++)]);
                }
                // update once at end of iteration to reduce heap write traffic
                lastRet = cursor = i;
                checkForComodification();
            }

            public int nextIndex() {
                return cursor;
            }
            public int previousIndex() {
                return cursor - 1;
            }
            public void remove() {
                if (lastRet < 0)
                    throw new IllegalStateException();
                checkForComodification();

                try {
                    SubList.this.remove(lastRet);
                    cursor = lastRet;
                    lastRet = -1;
                    expectedModCount = ArrayList.this.modCount;
                } catch (IndexOutOfBoundsException ex) {
                    throw new ConcurrentModificationException();
                }
            }
            public void set(E e) {
                if (lastRet < 0)
                    throw new IllegalStateException();
                checkForComodification();

                try {
                    ArrayList.this.set(offset + lastRet, e);
                } catch (IndexOutOfBoundsException ex) {
                    throw new ConcurrentModificationException();
                }
            }
            public void add(E e) {
                checkForComodification();

                try {
                    int i = cursor;
                    SubList.this.add(i, e);
                    cursor = i + 1;
                    lastRet = -1;
                    expectedModCount = ArrayList.this.modCount;
                } catch (IndexOutOfBoundsException ex) {
                    throw new ConcurrentModificationException();
                }
            }
            final void checkForComodification() {
                if (expectedModCount != ArrayList.this.modCount)
                    throw new ConcurrentModificationException();
            }
        };
    }

    public List<E> subList(int fromIndex, int toIndex) {	
        subListRangeCheck(fromIndex, toIndex, size);
        return new SubList(this, offset, fromIndex, toIndex);	// 递归调用SubList
    }

    private void rangeCheck(int index) {		// 上下界均检查
        if (index < 0 || index >= this.size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private void rangeCheckForAdd(int index) {		// 针对add的检查，index==size可以允许
        if (index < 0 || index > this.size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private String outOfBoundsMsg(int index) {		// 抛出异常时的提示信息
        return "Index: "+index+", Size: "+this.size;
    }

    private void checkForComodification() {		// 是否出现并发判断
        if (ArrayList.this.modCount != this.modCount)	// 外部的modCount和内部的modeCount不一致
            throw new ConcurrentModificationException();
    }

    public Spliterator<E> spliterator() {
        checkForComodification();
        return new ArrayListSpliterator<E>(ArrayList.this, offset,
                                           offset + this.size, this.modCount);
    }
}
```

## 10. 接口实现spliterator

```java
public Spliterator<E> spliterator() {		// 是List里面的方法
	return new ArrayListSpliterator<>(this, 0, -1, 0);
}
```

——返回这个对象，主要是想表达该对象可分割（普通的arrayList是不可分割的），如果需要分割可以调用

```java
// 主要实现了将arrayList进行分割
// 注释说：没有牺牲太多性能的情况下，检测尽可能多的干扰（并发产生），主要通过modCount判断——不能保证一定能检测到并发冲突，即可以使用，但是并不能说是线程安全的，但是考虑了其中的部分问题影响
static final class ArrayListSpliterator<E> implements Spliterator<E> {
    private final ArrayList<E> list;
    private int index; // 起始下标
    private int fence; // 分割的点，如果值-1，那么在使用的时候自动填充为size大小——延迟初始化，直到要使用的时候才初始化——getFence，为了提高精度
    private int expectedModCount; // 分割点计算时，就初始化该值——延迟初始化，直到要使用的时候才初始化——getFence，为了提高精度

    /** Create new spliterator covering the given  range */
    ArrayListSpliterator(ArrayList<E> list, int origin, int fence,
                         int expectedModCount) {
        this.list = list; //如果传递的是null，不会报错，除非需要开始遍历
        this.index = origin;
        this.fence = fence;		// 默认的分割点
        this.expectedModCount = expectedModCount;
    }
	
    // 获得分割点——初始时即fence=-1，那么返回list.size，其余时候都是返回0——主要就是初始化使用的
    private int getFence() { // initialize fence to size on first use
        int hi; // (a specialized variant appears in method forEach)
        ArrayList<E> lst;
        if ((hi = fence) < 0) {	// 只有初始化时fence=-1，那么就会被初始化为list，size
            if ((lst = list) == null)		// 数组为null，那么直接返回0
                hi = fence = 0;
            else {						// 数组不为null，那么返回数组的size，即最后一个位置
                expectedModCount = lst.modCount;
                hi = fence = lst.size;
            }
        }
        return hi;
    }

    public ArrayListSpliterator<E> trySplit() {		// 尝试分割
        int hi = getFence(), lo = index, mid = (lo + hi) >>> 1;		// 取中点
        return (lo >= mid) ? null : // 中分，除非数组太小，小到只有一个元素，那么lo和mid重叠了（0+1）/2=0，那么0=0
        new ArrayListSpliterator<E>(list, lo, index = mid, expectedModCount);// 如果重叠，那么返回null，否则就将前面的[lo, mid]产生一个新的对象，返回                          
    }
	
    // 如果存在1个剩余元素，那么就对其进行操作
    public boolean tryAdvance(Consumer<? super E> action) {
        if (action == null)
            throw new NullPointerException();
        int hi = getFence(), i = index;
        if (i < hi) {			
            index = i + 1;			// index更新
            @SuppressWarnings("unchecked") E e = (E)list.elementData[i];	// 操作当前的i
            action.accept(e);
            if (list.modCount != expectedModCount)			// 并发异常，上抛
                throw new ConcurrentModificationException();
            return true;
        }
        return false;
    }
	// 如果存在剩余元素，就对剩下的每个元素执行指定的action操作
    // 阅读源码发现，只有在最后进行一次并发异常判断，主要是为了性能考虑（注释写的）
    public void forEachRemaining(Consumer<? super E> action) {
        int i, hi, mc; // hoist accesses and checks from loop
        ArrayList<E> lst; Object[] a;
        if (action == null)
            throw new NullPointerException();
        if ((lst = list) != null && (a = lst.elementData) != null) {
            if ((hi = fence) < 0) {		// 如果发现hi未初始化过（不是从其他分割过来的，见trySplit），那么就初始化为size
                mc = lst.modCount;
                hi = lst.size;
            }
            else
                mc = expectedModCount;
            // 主要是对index和hi判断，并且进行赋值，index=hi，表示该循环结束之后，数组已经遍历完成
            if ((i = index) >= 0 && (index = hi) <= a.length) {	
                for (; i < hi; ++i) {	// 从i~index遍历并进行指定操作
                    @SuppressWarnings("unchecked") E e = (E) a[i];
                    action.accept(e);
                }
                if (lst.modCount == mc)		// 判断是否出现并发修改操作
                    return;
            }
        }
        // 到这一步没有返回，说明list存在，但是要么index和hi参数出现问题（这边出现问题也是因为并发问题，导致数组长度发生变化），就是循环操作中间出现了并发修改操作——上抛并发异常
        throw new ConcurrentModificationException();
    }

    public long estimateSize() {		// 主要是在forEachRemaining调用之前进行长度预估，如果是无限长你，那么返回Long#MAX_VALUE——让方法根据该返回值进行合理的处理
        return (long) (getFence() - index);
    }

    public int characteristics() {		// 获得该序列的特征值——有序的|元素的个数是可计数的，有界的|所有的子迭代器（直接的或者间接的），都是'SIZED'和SUBSIZED的
        return Spliterator.ORDERED | Spliterator.SIZED | Spliterator.SUBSIZED;
    }
}
```

## 11. 其他方法的重写

Collection：

```java
@Override
// 重写了Collection提供的默认方法——实现过滤器式删除，只有通过过滤的元素才会被删除
public boolean removeIf(Predicate<? super E> filter) {
    Objects.requireNonNull(filter);		// 判断过滤器对象是否存在
    // figure out which elements are to be removed
    // any exception thrown from the filter predicate at this stage
    // will leave the collection unmodified
    int removeCount = 0;
    final BitSet removeSet = new BitSet(size);
    final int expectedModCount = modCount;
    final int size = this.size;
    for (int i=0; modCount == expectedModCount && i < size; i++) {
        @SuppressWarnings("unchecked")
        final E element = (E) elementData[i];
        if (filter.test(element)) {
            removeSet.set(i);
            removeCount++;
        }
    }
    if (modCount != expectedModCount) {			
        throw new ConcurrentModificationException();
    }

    // shift surviving elements left over the spaces left by removed elements
    final boolean anyToRemove = removeCount > 0;
    if (anyToRemove) {
        final int newSize = size - removeCount;
        for (int i=0, j=0; (i < size) && (j < newSize); i++, j++) {
            i = removeSet.nextClearBit(i);
            elementData[j] = elementData[i];
        }
        for (int k=newSize; k < size; k++) {
            elementData[k] = null;  // Let gc do its work
        }
        this.size = newSize;
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        modCount++;
    }

    return anyToRemove;
}
```

List：

```java
@Override
@SuppressWarnings("unchecked")
public void replaceAll(UnaryOperator<E> operator) {
    Objects.requireNonNull(operator);
    final int expectedModCount = modCount;
    final int size = this.size;
    for (int i=0; modCount == expectedModCount && i < size; i++) {
        elementData[i] = operator.apply((E) elementData[i]);
    }
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
    modCount++;
}

@Override
@SuppressWarnings("unchecked")
public void sort(Comparator<? super E> c) {
    final int expectedModCount = modCount;
    Arrays.sort((E[]) elementData, 0, size, c);
    if (modCount != expectedModCount) {				// 处理并发操作
        throw new ConcurrentModificationException();
    }
    modCount++;		// 发现了调用了sort之后，就不能再进行其他的迭代器相关的操作了——都是会抛出异常的
}
```

Iterable：

```java
public void forEach(Consumer<? super E> action) {
    Objects.requireNonNull(action);
    final int expectedModCount = modCount;
    @SuppressWarnings("unchecked")
    final E[] elementData = (E[]) this.elementData;
    final int size = this.size;
    for (int i=0; modCount == expectedModCount && i < size; i++) {
        action.accept(elementData[i]);		// 对elementData的每个元素做一个指定的操作
    }
    if (modCount != expectedModCount) {		// 并发异常判断
        throw new ConcurrentModificationException();
    }
}
```

## 12. ArrayList直接继承的方法

Collection定义了两个方法：`equals`和`hashCode`，只有在`AbstractList`中进行了实现，而`ArrayList`是直接使用的

```java
public boolean equals(Object o) {			// 实现了Collection的方法，ArrayList是直接使用的
    if (o == this)			// 是不是对象本身
        return true;
    if (!(o instanceof List))		// 是不是不是同类：可能是子类or本身类
        return false;

    ListIterator<E> e1 = listIterator();		// 调用迭代器ListIterator
    ListIterator<?> e2 = ((List<?>) o).listIterator();
    while (e1.hasNext() && e2.hasNext()) {
        E o1 = e1.next();
        Object o2 = e2.next();
        if (!(o1==null ? o2==null : o1.equals(o2)))		// 每个元素都一样才算行——如果存在一个不一样，就直接返回
            return false;
    }
    return !(e1.hasNext() || e2.hasNext());		// 最后还有剩余也就是不一样
}
public int hashCode() {					// 实现了Collection的方法，ArrayList是直接使用的
    int hashCode = 1;
    for (E e : this)			// 和String算法类似——每个元素均获得其hash值，然后带权重相加
        hashCode = 31*hashCode + (e==null ? 0 : e.hashCode());
    return hashCode;

```

# ArrayList的使用

## 1. 查

```java
arr.size();			// 结点个数
arr.isEmpty();		// 判空
```

```java
arr.contains(obj);		// 看列表中是否存在该结点
arr.indexOf(obj);			// 看指定结点的index
arr.lastIndexOf(obj);		// 从后向前找
```

```java
arr.get(1);			// 获得指定下标的内容
```

```java
for(Integer num: nums){
	System.out.println(num);
}

// 复杂一点，但是操作更灵活的
Iterator<Integer>it = arr.iterator();
while(it.hasNext()){
    System.out.println(it.next());
}
```

## 2. 增

```java
arr.add(obj);		// 默认将值添到数组尾部
arr.add(1, obj);		// 将值添加到指定的index处，那么这个之后的节点全部需要移动
```

## 3. 删

```
arr.remove(3);		// 删除指定下标的结点
arr.remove(obj);		// 删除具体对象——需要先查找后删除
arr.clear();		// 清空
```

### 4. 改

```java
arr.set(1, obj);		// 指定index修改为obj内容
```

# 总结

1. 里面大量用到了`Arrays.copyOf(elementData, size);`用来创建一个新的数组，并将原来的数据全部复制过去。因为ArrayList的方法操作的都是同一个内部数组，而所有方法都没有加锁，没有同步机制，所以它是线程不安全的。
2. `System.arraycopys(elementData, 0, a, 0, size)`：涉及到的是数组拷贝，参数分别是：（原数组，原数组起始位置，目标数组，目标数组起始位置，原数组被复制的长度
3. 代码量不是很大，但是涉及范围广，并且有多层的继承、实现，还需要去品味里面每个参数的使用原因。认为最为复杂的就是关于并发的判断了（感觉只是在不影响其本身性能的情况下，做了一些判断，所以有些判断的操作还是有点奇怪的 ）

### TODO

- [ ] `batchRemove()`中的`modCount += size - w;`操作的具体原因
- [ ] Iterator并发判断的研究
- [ ] spliterator的使用进行研究

参考：

1. https://cloud.tencent.com/developer/article/1141497
2. https://juejin.cn/post/6844903624062009357
3. https://juejin.cn/post/6844903614704680974
4. https://www.huaweicloud.com/articles/3a7734560df006964bc1b6d66115625e.html

# ArrayList实现的接口/抽象类

## 1. Collection

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

## 2. AbstractCollection

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

# ps：面试的考点

## 1. fail-fast 

### （1）fail-fast解释

在Java的注释经常可以碰到`fail-fast`，这是一种机制

wiki的解释：在系统设计中，快速失效系统一种可以立即报告任何可能表明故障的情况的系统。常用来停止当前正常的操作，而不是试图继续执行可能存在缺陷的故障，让系统自行触发异常而返回

——即**系统设计的时候优先考虑异常情况，一旦探测到异常（但是当前还未触发异常），直接停止操作并上报**（有点像，写leetcode的时候处理特殊情况，因为前面不处理，继续执行下去可能会发生越界等异常）

通常说的**`fail-fast`就是Java的Collection中的错误检测机制**

目的是：多个线程同时对集合的结构进行修改，就会触发fail-fast机制，这时候就会抛出`ConcurrentModificationException`（并发修改异常）

这边存在一个陷阱：设计集合的fail-fast的初心就是为了防止多线程同时去修改集合的结构。但是由于实现机制导致：**在使用迭代器进行遍历操作的时候，如果使用外部的remove/add等操作，那么就会触发CME**——具体原因看源码（主要是由于modCount和exceptModCount不匹配引起的），所以并没有出现并发，但是引起了并发异常——解决方法：调用迭代器提供的remove方法（ListIterator还提供add等方法）

ps：解释一下这边的特定并发操作：一个或者多个线程在遍历操作该集合，操作可以是只读、也可以是读写都存在。而抛出的异常CME，是并发修改，即在一个或多个线程迭代过程中，**存在2个及以上的线程去修改该集合的结构**

### （2）对应的：fail-safe

java实现了`fail-safe`机制的集合类，这些集合在遍历时，不是**直接在其本身上操作，而是在副本上面操作**

`java.util.concurrent`包下的容器都是fail-safe的，可以在多线程下并发使用，并发修改。同时也可以在foreach中进行add/remove ——但是由于是副本，那么之后对内容的修改，迭代器无法访问到

eg：ArrayList对应的线程安全的类是：`CopyOnWriteArrayList`

（博客后面还有关于`copyonwrite`的知识，后面学了并发知识之后再来补充）

参考内容：

https://juejin.cn/post/6844903824709287943

## 2. 遍历时修改ArrayList的正确操作

根据实际经验，如果运行如下代码

```java
ArrayList<Integer>arr = new ArrayList<>(Arrays.asList(1,4,3,2,5,5,7,8));
for(int i = 0; i < arr.size(); i++){
    if(arr.get(i) == 5){
        arr.remove(i);
    }
}

// >> {1, 4, 3, 2, 5, 7, 8}，可以发现少删除了一个5，这是因为arr删除前一个5之后，i就指向了后一个的5，而此时i++，就导致后一个直接被忽略了
```

——一定会使部分元素没有被遍历到（删除结点后面紧跟的元素被忽略了）

主要原因：当前结点被删除，那么i之后的节点会向前移动一位，此时i指向的就是之前i+1的结点，所以它会被忽略。

方法是，**删除后，添加i--**，类似于跳过本轮循环，那么下一次就能遍历到

参考很多解法，都建议从后向前遍历：——那么移动发生在遍历过的地方，不会忽略

```java
for(int i = arr.size() - 1; i >= 0; i--){
	if(arr.get(i) == 5){
        arr.remove(5);
    }
}
```

下边一种是由于fail-fast引起的错误：

```java
for(Integer num : nums){
	if(num == 5){
        nums.remove(num);
    }
}
```

由于foreach会去调用arrayList提供的迭代器，而迭代器不允许在遍历期间，去调用外部的add、remove等方法改变数组的结构状态，会引起fail-fast。所以这个代码是会抛出异常的。

方法是：删了一个就跑，直接break结束循环

还有一种，使用迭代器内部的remove方法：但是这个遍历较为麻烦：

```java
Iterator<Integer>it = arr.iterator();		// 获得arrayList的迭代器
while(it.hasNext()){
    if(it.next() == 5){
        it.remove();			// 这个就只能删除当前结点，而不能指定结点
    }
}
// >> 1,4,3,2,7,8
```

需要注意：一个循环中不能多次调用next方法，因为next方法每次都会将结点向后移动一个

参考：

1. https://juejin.cn/post/6844904038442467336