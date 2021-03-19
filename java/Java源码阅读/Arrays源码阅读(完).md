# Arrays源码阅读

面试不大考里面的实现，但是在刷题的时候遇到的比较多关于数组的操作，稍微看一下源码有助于记忆。

所以，只关注常见的好用的方法，生僻的方法不做考虑。

总体介绍：Arrays是Java中用来操作数组的类

## 1. 类头：

```java
public class Arrays{...}
```

构造方法

```java
 private Arrays() {...}
```

私有的构造方法，表示不可被实例化

其内部能够使用的都是静态方法，所以没有实例化的必要

## 2. sort

### 2.1 实现原理

Java利用的是改进后的快排，基本思想是双轴快排，思路可以见[博客](https://blog.csdn.net/Holmofy/article/details/71168530?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522161603636716780269869141%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=161603636716780269869141&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_v29-1-71168530.pc_v2_rank_blog_default&utm_term=%E5%8F%8C%E8%BD%B4	)，双轴快排是不稳定的

基本思路：

1.  首先检查数组的长度，比一个阈值小的时候直接使用双轴快排

   双轴快排：从传统的单轴快排，然后进行改进，变化到3-way快排演化过来的，主要思想就是设置两个pivot，然后将整个数组划分成：$x<pivot1， pivot1\le x\le pivot2，未遍历到的，>pivot$，用i、k、j三个变量来划分区域，其中k是用来遍历的，最后遍历完成后，会只剩下三部分。

   具体见[博客](https://blog.csdn.net/Holmofy/article/details/71168530)，后半部分写的很详细

2. 再检查数组中数据的顺序连续性，如果顺序连续性好，直接使用TimSort算法

3. 顺序连续性不好的数组直接使用了 双轴快排 + 成对插入排序

```java
public static void sort(int[] a) {
    DualPivotQuicksort.sort(a, 0, a.length - 1, null, 0, 0);
}
```

对于泛型数据类型，它需要指定一个比较器。

1. 如果没有比较器，那么用上面的排序
2. 如果指定了，那么用归并排序实现，或者采用了TimSort算法（看配置，默认是TimSort），TimSort是稳定的

```java
public static <T> void sort(T[] a, Comparator<? super T> c) {
    if (c == null) {
        sort(a);
    } else {
        if (LegacyMergeSort.userRequested)
            legacyMergeSort(a, c);
        else
            TimSort.sort(a, 0, a.length, c, null, 0, 0);
    }
}
```

### 2.2 使用

Arrays为7个基本数据类型均设置了对应的sort方法，

使用：

```java
Arrays.sort(a);
```

并且还能指定sort范围：

```java
Arrays.sort(a, 0, 5);
```

针对对象类型的数据：**它要求传进来的数组对象必须实现`Comparable`接口**，使用方法一样

```java
public static void sort(Object[] a);
```

针对泛型类型的数据：我们需要自定义比较器，可以实现Comparator接口的compare方法

```java
public class Test {
    public static void main(String[] args) {
        /*
         * 注意，要想改变默认的排列顺序，不能使用基本类型（int,double,char）而要使用它们对应的类
         */
        Integer[] a = { 9, 8, 7, 2, 3, 4, 1, 0, 6, 5 };
        // 定义一个自定义类MyComparator的对象
        Comparator cmp = new MyComparator();
        Arrays.sort(a, cmp);
        for (int arr : a) {
            System.out.print(arr + " ");
        }
    }
}

// 实现Comparator接口
class MyComparator implements Comparator<Integer> {
    @Override
    public int compare(Integer o1, Integer o2) {
        /*
         * 如果o1小于o2，我们就返回正值，如果o1大于o2我们就返回负值， 这样颠倒一下，就可以实现降序排序了,反之即可自定义升序排序了
         */
        return o2 - o1;
    }
}
```

参考：

1. https://www.jianshu.com/p/6d26d525bb96

## 3. binarySearch

### 3.1 原理

Arrays提供的只有二分查找。而二分查找的前提是：**数组有序**

```java
public static int binarySearch(int[] a, int key) {
    return binarySearch0(a, 0, a.length, key);
}
```

并且也可以指定查找的范围

```java
public static int binarySearch(int[] a, int fromIndex, int toIndex, int key) 
```

当然也有对对象类型数组的查找，**同样对象必须实现`Comparable`接口**，能够比较才能进行查找

也有泛型类型的数组查找，可以自定义比较器

如果查找失败了：返回值是**-(插入点+1)**，eg：-4，表明该对象适合插入到数组的index=3的位置处；如果数组中存在多个匹配的元素，返回值是不确定的——二分查找的特性决定的

### 3.2 使用

```java
Arrays.binarySearch(nums, 4);
Arrays.binarySearch(nums, 1, 7, 4);			// 可以限制查找范围
Arrays.bingarySearch(arr, obj, cmp);		// 泛型类型数组，自定义比较器
```

## 4. equals

判断两个数组是否一样

1. 判断是否是同一个对象，如果是同一个对象，那么直接返回
2. 如果不是同一个对象，再次判断是否存在一个为null，如果存在那么返回false
3. 如果均为非null，再次判断数组的长度，如果长度不等，那么返回false
4. 如果数组长度一样，循环判断数组的每一个元素，只有每个元素一样才算是一样

```java
public static boolean equals(int[] a, int[] a2) {
    if (a==a2)
        return true;
    if (a==null || a2==null)
        return false;

    int length = a.length;
    if (a2.length != length)
        return false;

    for (int i=0; i<length; i++)
        if (a[i] != a2[i])
            return false;

    return true;
}
```

当然存在对object类型的数组的判断，当然需要通过obj1.equals(obj2)对每个元素进行判断

还有一个高维数组的判断，通过递归实现：

```java
public static boolean deepEquals(Object[] a1, Object[] a2) {
    if (a1 == a2)
        return true;
    if (a1 == null || a2==null)
        return false;
    int length = a1.length;
    if (a2.length != length)
        return false;				// 同前面一样，进行粗略的判断

    for (int i = 0; i < length; i++) {		// 循环比较每个结点
        Object e1 = a1[i];
        Object e2 = a2[i];

        if (e1 == e2)
            continue;
        if (e1 == null)
            return false;

        // Figure out whether the two elements are equal
        boolean eq = deepEquals0(e1, e2);	// e1,e2可能本身还是一个数组or其他引用对象，所以需要去调用方法去判断

        if (!eq)
            return false;
    }
    return true;
}

static boolean deepEquals0(Object e1, Object e2) {
    assert e1 != null;
    boolean eq;
    if (e1 instanceof Object[] && e2 instanceof Object[])// 如果e1和e2是对象类型的数组，那么还是要递归调用前面的方法
        eq = deepEquals ((Object[]) e1, (Object[]) e2);			
    else if (e1 instanceof byte[] && e2 instanceof byte[])	// 如果是基本数据类型的数组，那么调用对应的方法即可
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
        eq = e1.equals(e2);				// 如果对象不是一个数组，那么直接调用对象的equals方法
    return eq;
}
```

——如果两个如组同时存在循环引用的情况下可能出现死循环的风险。

## 5. fill

数组填充

```java
public static void fill(int[] a, int val) {
    for (int i = 0, len = a.length; i < len; i++)
        a[i] = val;
}

public static void fill(Object[] a, Object val) {		// 对象循环赋值——它们都指向同一个对象
    for (int i = 0, len = a.length; i < len; i++)
        a[i] = val;
}
```

### 5.2 使用

```java
Arrays.fill(nums, 2);
```

## 6. copyOf⭐

一个很常见的方法，是**浅拷贝**，新建了数组，数组中的每个对象都是直接引用，而不会创建一组新的对象

会创建一个新的数组，然后将原始数组的 [0, newLength) 的数据拷贝过去，返回值是数组

```java
public static int[] copyOf(int[] original, int newLength) {
    int[] copy = new int[newLength];
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));
    return copy;
}
```

——本质上还是去调用了System.arrayCopy的方法，这个方法是native

但是这个方法存在的问题：只能从头开始复制，从头开始的某一段进行复制，而无法复制中间的某一段。

如果要用，可以选择：

```java
public static int[] copyOfRange(int[] original, int from, int to) {
    int newLength = to - from;
    if (newLength < 0)
        throw new IllegalArgumentException(from + " > " + to);
    int[] copy = new int[newLength];
    System.arraycopy(original, from, copy, 0,
                     Math.min(original.length - from, newLength));
    return copy;
}
```

因此，常见的都是直接用System.arraycopy()

### 6.2 使用

```java
int[] newArr = Arrays.copyOf(arr, 5);		// 复制的是[0,5)范围的数组
int[] newArr2 = Arrays.copyOfRange(arr, 1, 5);		// 复制的是[1, 6)范围的数组
```

```java
int[] newArr3 = new int[20];
System.arraycopy(arr, 2, newArr3, 0, 5);		// 复制的是从[2, 7)范围的数组到newArr3的0起始的开始的位置
```

### ps: System.arraycopy() 和 Arrays.copyOf()

Arrays.copyOf底层也是调用System.arraycopy的方法，可以说是**受限的System.arraycopy方法**。

主要区别在于：

1. System.arraycopy需要自己先新建一个数组，然后将新数组和旧数组均传递进去，而直接将旧数组复制到提供的数组中

   Arrays.copyOf会在内部创建一个数组，然后复制完成之后将数组的首地址返回给调用者

2. System.arraycopy需要传递**旧数组的起始地址，和新数组的存放起始地址，复制长度范围**

   copyOf只能指定复制的长度范围，且只能从头开始复制

## 7. asList

也是较为常见的方法，主要是将传递来的参数数组，**转换成ArrayList**

注意，传递的参数是对象数据类型，普通数组类型会自动装箱的

实际上传递的是一个个参数，但是编译器会自动将其组成一个数组

```java
public static <T> List<T> asList(T... a) {
    return new ArrayList<>(a);
}
```

注意，**这边返回的ArrayList对象是Arrays内部实现的一个ArrayList**，而不是我们常用的java.util.ArrayList

该ArrayList只有简单的功能，查询、排序、遍历、设置值等，但是无法添加和删除元素，同时也无法扩容等

### 7.2 使用

```java
ArrayList<String> stooges = Arrays.asList("Larry", "Moe", "Curly");
```

## 8. hashCode

计算数组的hashCode

和String的计算方法一致

```java
public static int hashCode(int a[]) {
    if (a == null)			// 如果数组为空，直接返回0
        return 0;

    int result = 1;
    for (int element : a)
        result = 31 * result + element;

    return result;
}
```

也存在高维数组的hash值的计算：

——递归调用，如果出现自己引用自己的情况，就会出现死循环。

```java
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
```

## 9. toString

主要是在调试的时候，可以直接看到数组的内容：

```java
public static String toString(int[] a) {
    if (a == null)
        return "null";
    int iMax = a.length - 1;
    if (iMax == -1)
        return "[]";

    StringBuilder b = new StringBuilder();		// 用sb来实现动态string的
    b.append('[');
    for (int i = 0; ; i++) {
        b.append(a[i]);
        if (i == iMax)
            return b.append(']').toString();
        b.append(", ");
    }
}
```

输出的格式是："[1,2,3,4,5]"

当然也有针对高维数组的：——这边对循环引用进行了处理。

```java
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

    dejaVu.add(a);			// 将该对象加入
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
                    if (dejaVu.contains(element))	// 如果已经出现过该对象，表示出现循环引用，那么不再进入该递归了
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

