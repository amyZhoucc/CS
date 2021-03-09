# 4. Arrays类

算是数组的工具类，提供了一系列的静态方法给数组使用，使用方式是`Arrays.xxx`。

**toString()**：在Arrays类中存在9个方法，是8种基本数据类型+1个Object对象——给对象元素使用的

```java
public static String toString(int[] a) {
    if (a == null)		// 如果数组未初始化，返回null
        return "null";
    int iMax = a.length - 1;
    if (iMax == -1)		// 就是len=0，表示数组未空，返回一个空的格式
        return "[]";

    StringBuilder b = new StringBuilder();	// 创建StringBuilder对象
    b.append('[');
    for (int i = 0; ; i++) {
        b.append(a[i]);			// 依次把数组中的元素加入
        if (i == iMax)
            return b.append(']').toString();		// 指向数组的最后一个元素
        b.append(", ");
    }
}
// Arrays.java
public static String toString(Object[] a) {
    if (a == null)
        return "null";

    int iMax = a.length - 1;
    if (iMax == -1)
        return "[]";

    StringBuilder b = new StringBuilder();
    b.append('[');
    for (int i = 0; ; i++) {
        b.append(String.valueOf(a[i]));	 // 注意这边是去获取该对象的值
        if (i == iMax)
            return b.append(']').toString();
        b.append(", ");
    }
}
// String.java
public static String valueOf(Object obj) {
    return (obj == null) ? "null" : obj.toString();	// 返回的是该对象的toString方法
}
```

### **sort()**⭐

排序方法：一共有18个方法，（7种基本数据类型+1个Object对象 + 1个泛型对象（可以支持所有对象）且支持自定义排序）*（1基本排序方法+1指定排序范围方法）

ps：boolean不需要排序

Java对排序的实现十分复杂：

对于基本数据类型，主要就是**双枢轴快速排序**（DualPivotQuicksort），Java7之前就是普通的快排

——因为快排快，但是是不稳定的

对于对象类型，采用**Tim排序**，Java7之前是归并排序

——对象类型排序需要稳定

```java
public static void sort(int[] a) {
    DualPivotQuicksort.sort(a, 0, a.length - 1, null, 0, 0);
}

// 可以指定排序的范围
public static void sort(int[] a, int fromIndex, int toIndex) {
    rangeCheck(a.length, fromIndex, toIndex);		// 需要进行边界检查
    DualPivotQuicksort.sort(a, fromIndex, toIndex - 1, null, 0, 0);
}

public static void sort(Object[] a) {		// 要求对象是实现了Comparable接口
    if (LegacyMergeSort.userRequested)
        legacyMergeSort(a);		// 归并排序
    else
        ComparableTimSort.sort(a, 0, a.length, null, 0, 0);	// Tim排序
}

// 可以自己写比较器作为其中一个参数，自定义比较方法
// 在两数进行比较的时候，就调用比较器中的compare方法
public static <T> void sort(T[] a, Comparator<? super T> c) {
    if (c == null) {
        sort(a);		// 如果没有定义比较器，就使用上面的Object对象方法
    } else {
        if (LegacyMergeSort.userRequested)
            legacyMergeSort(a, c);
        else
            TimSort.sort(a, 0, a.length, c, null, 0, 0);
    }
}

// 比较器，是一个接口，Comparator.java
public interface Comparator<T> {	
    int compare(T o1, T o2);		// 只有方法头，规定返回值为-1 o1<o2/0 o1=o2/1 o1>o2
    boolean equals(Object obj);
}
```

分析：sort方法默认是从小到大排序的，如果需要自己指定排序方法可以用**匿名内部类**来实现。

Java中排序是通过比较实现的（还有计数排序——不进行比较）

在排序中，当需要进行对象比较时，会去调用比较器comparator的compare方法



String中的CaseInsensitiveComparator比较器的实现：

```java
// String.java 内部实现了一个忽略大小写的比较器，可以直接调用
public static final Comparator<String> CASE_INSENSITIVE_ORDER
                                         = new CaseInsensitiveComparator();

// 实现在String.java内部——是一个私有静态内部类，实现了Comparator接口功能
private static class CaseInsensitiveComparator
            implements Comparator<String>, java.io.Serializable {
    // use serialVersionUID from JDK 1.2.2 for interoperability
    private static final long serialVersionUID = 8575799808933029326L;

    public int compare(String s1, String s2) {
        int n1 = s1.length();
        int n2 = s2.length();
        int min = Math.min(n1, n2);
        for (int i = 0; i < min; i++) {
            char c1 = s1.charAt(i);
            char c2 = s2.charAt(i);
            if (c1 != c2) {		// 如果两者相等就比较下一对；不等，就都转成大写再比较
                c1 = Character.toUpperCase(c1);
                c2 = Character.toUpperCase(c2);
                if (c1 != c2) {		// 转成大写还是不相等，就转成小写（有些字符只能转成小写比较）
                    c1 = Character.toLowerCase(c1);
                    c2 = Character.toLowerCase(c2);	
                    if (c1 != c2) {		// 如果都不相等，那么就是不相等，就看两者的小写字母的差值
                        // No overflow because of numeric promotion
                        return c1 - c2;
                    }
                }
            }
        }
        return n1 - n2;	// 如果都相等，说明前min个都一样，就看字符串是否一样长
    }

    /** 替换反序列化对象 */
    private Object readResolve() { 
        return CASE_INSENSITIVE_ORDER; 
    }
}

// 使用，直接调用String的静态comparator对象
String[] arr = {"hello","world", "Break","abc"};
Arrays.sort(arr, String.CASE_INSENSITIVE_ORDER);
```

**binarySearch**：二分查找方法，同sort一样，也是有18种方法，同样的情况

```java
public static int binarySearch(int[] a, int key) {
    return binarySearch0(a, 0, a.length, key);
}
// 具体实现：就是常见的二分查找方式，时间复杂度为O(NlogN)
private static int binarySearch0(int[] a, int fromIndex, int toIndex, int key) {
    int low = fromIndex;
    int high = toIndex - 1;

    while (low <= high) {
        int mid = (low + high) >>> 1;
        int midVal = a[mid];

        if (midVal < key)
            low = mid + 1;
        else if (midVal > key)
            high = mid - 1;
        else
            return mid; // key found
    }
    return -(low + 1);  // 找不到返回的值是-(插入点+1)
}

// 可以自定义比较方式
public static <T> int binarySearch(T[] a, T key, Comparator<? super T> c) {
    return binarySearch0(a, 0, a.length, key, c);
}

private static <T> int binarySearch0(T[] a, int fromIndex, int toIndex,
                                         T key, Comparator<? super T> c) {
    if (c == null) {
        return binarySearch0(a, fromIndex, toIndex, key);
    }
    int low = fromIndex;
    int high = toIndex - 1;

    while (low <= high) {
        int mid = (low + high) >>> 1;
        T midVal = a[mid];
        int cmp = c.compare(midVal, key);		// 调用自定义比较方法
        if (cmp < 0)
            low = mid + 1;
        else if (cmp > 0)
            high = mid - 1;
        else
            return mid; // key found
    }
    return -(low + 1);  // key not found.
}
```

分析：因为查找本质上还是在比较，比较目标数和当前数的大小，所以对于对象来说还是可以自定义比较内容的

注意：

1. 查找的数组**必须是有序的**
2. 如果该对象在数组中查找失败了，那么返回值是**-(插入点+1)**，eg：-4，表明该对象适合插入到数组的index=3的位置处
3. 如果数组中存在多个匹配的元素，返回值是不确定的——二分查找的特性决定的

**equals**：比较两个数组是否一样，9个方法：8个基本数据类型+1个Object对象

```java
// int很常规，不关注
// 关注一下Object对象的
public static boolean equals(Object[] a, Object[] a2) {
    if (a==a2)		// 先判断是否是引用同一个对象
        return true;
    if (a==null || a2==null)	// 排除特殊情况
        return false;

    int length = a.length;
    if (a2.length != length)		// 长度不一样也排除
        return false;

    for (int i=0; i<length; i++) {	
        Object o1 = a[i];		
        Object o2 = a2[i];
        if (!(o1==null ? o2==null : o1.equals(o2)))	// 本质上是去调用对象的equals方法
            return false;
    }

    return true;
}
```

**copyOf**：基于原数组，复制生成一个新数组——浅拷贝，只是新建了数组，而数组中元素对象，本质上是复制了对象的引用，没有为每个元素创建新对象。

```java
public static int[] copyOf(int[] original, int newLength) {
    int[] copy = new int[newLength];		// 数组长度是根据newLength来的
    System.arraycopy(original, 0, copy, 0,		// 调用的是native方法（c++实现）
                     Math.min(original.length, newLength));	// 选择最小值——可能数组只复制一部分（防止出现越界异常）
    return copy;
}

// 泛型对象
public static <T> T[] copyOf(T[] original, int newLength) {
    return (T[]) copyOf(original, newLength, original.getClass());
}

public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
    @SuppressWarnings("unchecked")
    T[] copy = ((Object)newType == (Object)Object[].class)	// 判断数组元素的对象类型和传入的类型是否一致
        ? (T[]) new Object[newLength]
        : (T[]) Array.newInstance(newType.getComponentType(), newLength);
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));	// 内核一样
    return copy;
}
```

**fill**：为数组的每个元素赋相同的值，有18个方法=（8种基本数据类型 + 1个object类对象）*（1个基本的+1个指定赋的范围）

```java
// 简单，没啥好说的
public static void fill(int[] a, int val) {
    for (int i = 0, len = a.length; i < len; i++)
        a[i] = val;
}

public static void fill(int[] a, int fromIndex, int toIndex, int val) {
    rangeCheck(a.length, fromIndex, toIndex);	// 范围检查
    for (int i = fromIndex; i < toIndex; i++)
        a[i] = val;
}
```

**hashCode**：为数组计算哈希值——本质上和String对象一样，有9种方法（8个基础数据类型+1个Object对象）

```java
public static int hashCode(int a[]) {
    if (a == null)
        return 0;

    int result = 1;
    for (int element : a)
        result = 31 * result + element;	// 数组元素的位置不同，对hash值的影响也不同（前面影响大）

    return result;
}

// boolean的计算：
result = 31 * result + (element ? 1231 : 1237);
// float的计算——转换成int的二进制表示
result = 31 * result + Float.floatToIntBits(element);
// double的计算——和计算包装类的hashCode一样
long bits = Double.doubleToLongBits(element);
result = 31 * result + (int)(bits ^ (bits >>> 32));
// Object对象——就是去调用每个元素对象的hashCode方法
result = 31 * result + (element == null ? 0 : element.hashCode());
```

**n维数组的方法**——数组元素还是数组，就会去递归调用，直到得到基本数据类型/具体对象

```java
// 递归获得数组hash值
public static int deepHashCode(Object a[]) {
    if (a == null)
        return 0;

    int result = 1;

    for (Object element : a) {
        int elementHash = 0;
        if (element instanceof Object[])
            elementHash = deepHashCode((Object[]) element);
        else if (element instanceof byte[])
            elementHash = hashCode((byte[]) element);
        else if (element instanceof short[])
            elementHash = hashCode((short[]) element);
        else if (element instanceof int[])
            elementHash = hashCode((int[]) element);
        else if (element instanceof long[])
            elementHash = hashCode((long[]) element);
        else if (element instanceof char[])
            elementHash = hashCode((char[]) element);
        else if (element instanceof float[])
            elementHash = hashCode((float[]) element);
        else if (element instanceof double[])
            elementHash = hashCode((double[]) element);
        else if (element instanceof boolean[])
            elementHash = hashCode((boolean[]) element);
        else if (element != null)
            elementHash = element.hashCode();

        result = 31 * result + elementHash;
    }

    return result;
}

// 递归判断相等
public static boolean deepEquals(Object[] a1, Object[] a2) {
    if (a1 == a2)
        return true;
    if (a1 == null || a2==null)
        return false;
    int length = a1.length;
    if (a2.length != length)
        return false;

    for (int i = 0; i < length; i++) {
        Object e1 = a1[i];
        Object e2 = a2[i];

        if (e1 == e2)
            continue;
        if (e1 == null)
            return false;

        // Figure out whether the two elements are equal
        boolean eq = deepEquals0(e1, e2);

        if (!eq)
            return false;
    }
    return true;
}

static boolean deepEquals0(Object e1, Object e2) {
    assert e1 != null;
    boolean eq;
    if (e1 instanceof Object[] && e2 instanceof Object[])
        eq = deepEquals ((Object[]) e1, (Object[]) e2);
    else if (e1 instanceof byte[] && e2 instanceof byte[])
        eq = equals((byte[]) e1, (byte[]) e2);
    else if (e1 instanceof short[] && e2 instanceof short[])
        eq = equals((short[]) e1, (short[]) e2);
    else if (e1 instanceof int[] && e2 instanceof int[])
        eq = equals((int[]) e1, (int[]) e2);
    else if (e1 instanceof long[] && e2 instanceof long[])
        eq = equals((long[]) e1, (long[]) e2);
    else if (e1 instanceof char[] && e2 instanceof char[])
        eq = equals((char[]) e1, (char[]) e2);
    else if (e1 instanceof float[] && e2 instanceof float[])
        eq = equals((float[]) e1, (float[]) e2);
    else if (e1 instanceof double[] && e2 instanceof double[])
        eq = equals((double[]) e1, (double[]) e2);
    else if (e1 instanceof boolean[] && e2 instanceof boolean[])
        eq = equals((boolean[]) e1, (boolean[]) e2);
    else
        eq = e1.equals(e2);
    return eq;
}

// 递归获得数组的内容
public static String deepToString(Object[] a) {
    if (a == null)
        return "null";

    int bufLen = 20 * a.length;
    if (a.length != 0 && bufLen <= 0)
        bufLen = Integer.MAX_VALUE;
    StringBuilder buf = new StringBuilder(bufLen);
    deepToString(a, buf, new HashSet<Object[]>());
    return buf.toString();
}
private static void deepToString(Object[] a, StringBuilder buf,
                                     Set<Object[]> dejaVu) {
    if (a == null) {
        buf.append("null");
        return;
    }
    int iMax = a.length - 1;
    if (iMax == -1) {
        buf.append("[]");
        return;
    }

    dejaVu.add(a);
    buf.append('[');
    for (int i = 0; ; i++) {

        Object element = a[i];
        if (element == null) {
            buf.append("null");
        } else {
            Class<?> eClass = element.getClass();

            if (eClass.isArray()) {
                if (eClass == byte[].class)
                    buf.append(toString((byte[]) element));
                else if (eClass == short[].class)
                    buf.append(toString((short[]) element));
                else if (eClass == int[].class)
                    buf.append(toString((int[]) element));
                else if (eClass == long[].class)
                    buf.append(toString((long[]) element));
                else if (eClass == char[].class)
                    buf.append(toString((char[]) element));
                else if (eClass == float[].class)
                    buf.append(toString((float[]) element));
                else if (eClass == double[].class)
                    buf.append(toString((double[]) element));
                else if (eClass == boolean[].class)
                    buf.append(toString((boolean[]) element));
                else { // element is an array of object references
                    if (dejaVu.contains(element))
                        buf.append("[...]");
                    else
                        deepToString((Object[])element, buf, dejaVu);
                }
            } else {  // element is non-null and not an array
                buf.append(element.toString());
            }
        }
        if (i == iMax)
            break;
        buf.append(", ");
    }
    buf.append(']');
    dejaVu.remove(a);
}
```





Collections存在两个静态方法，可以返回逆序的Comparator

```java
public static <T> Comparator<T> reverseOrder()
public static <T> Comparator<T> reverseOrder(Comparator<T> cmp)
```

