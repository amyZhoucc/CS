# JUC下的原子类

原子类是非阻塞的并发包。它能够实现在synchronized使用下的互斥执行，但是性能比synchronized好。

一共有13个原子类，有4类：

- 基本数据类型的原子类
- 数组类型的原子类
- 引用类型的原子类
- 更新属性（字段）的原子类

## Unsafe类

最本质的就3个方法，其他都是调用这3个方法的：

```java
public final native boolean compareAndSwapObject(Object var1, long var2, Object var4, Object var5);
// var2是指要修改的变量在var1的偏移，var4=expect，var5=value

public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);

public final native boolean compareAndSwapLong(Object var1, long var2, long var4, long var6);
```

## 1. 基本数据类型

有AtomicBoolean、AtomicInteger、AtomicLong3个

### 1.1 实例变量

```java
private volatile int value;			// 存放值——保证内存可见性，所以用volatile修饰

private static final long valueOffset;		// 内存中的位移，主要是因为要调用native方法，所以采用直接操作内存中的内容
static {
    try {
        valueOffset = unsafe.objectFieldOffset
            (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}
```

### 1.2 构造方法

```java
public AtomicInteger() {
}

public AtomicInteger(int initialValue) {
    value = initialValue;
}
```

### 1.3 实例方法

普通方法

```java
public final int get() {
    return value;
}
public final void set(int newValue) {
    value = newValue;
}
```

延迟刷新到内存，所以其他线程在一段时间内还是能够读到旧值

```java
public final void lazySet(int newValue) {
    unsafe.putOrderedInt(this, valueOffset, newValue);
}
```

获取值并且设置——这个是如果一直失败都会尝试，直到成功

```java
public final int getAndSet(int newValue) {
    return unsafe.getAndSetInt(this, valueOffset, newValue);
}

// Unsafe中的实现
public final int getAndSetInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);			// 获得最新的值
    } while(!this.compareAndSwapInt(var1, var2, var5, var4));	// 一致尝试修改值，直到没有出现并发修改，才将var4的值设置过去

    return var5;
}
```

经典的：CAS，设置值，如果失败就会返回false

```java
public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```

自加操作

```java
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}

// 尝试自加，直到成功才返回，返回的是成功那次修改前的值
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

增加指定大小：

```java
public final int getAndAdd(int delta) {
    return unsafe.getAndAddInt(this, valueOffset, delta);
}
```

toString方法，就是将当前的值转换成字符串形式：

```java
public String toString() {
    return Integer.toString(get());
}
```

——总结：本质上就是调用CAS实现的，所以能够非阻塞形式进行值的操作。

常用的是：`compareAndSet(int expect, int update);`，`getAndIncrement();`

## 2. 数组类型

可以原子性的修改数组里面的值

具体也有3类：同上面的分类。

使用方法：

```java
public class AtomicIntegerArrayTest {
    static int[] value = new int[] { 1， 2 };
    static AtomicIntegerArray ai = new AtomicIntegerArray(value);		// 传递的是一个数组
    public static void main(String[] args) {
        ai.getAndSet(0， 3);
        System.out.println(ai.get(0));
        System.out.println(value[0]);
    }
}
```

构造方法中传递的是数组，注意会将该数组赋值一份，所以不会有影响。

## 3. 引用类型

上面的AtomicInteger修改的只能是一个变量的值，并且该变量还必须是基本数据类型。

有3类API：

- AtomicReference：原子更新引用类型；
- AtomicReferenceFieldUpdater：原子更新引用类型里的字段。
- AtomicMarkableReference：原子更新带有标记位的引用类

本质上就是去调用`compareAndSwapObject`

## 4. 原子更新字段

更新类里面的某个字段

- AtomicIntegerFieldUpdater：原子更新整型的字段的更新器。
- AtomicLongFieldUpdater：原子更新长整型字段的更新器。
- **AtomicStampedReference：原子更新带有版本号的引用类**——用来解决CAS的ABA问题。

需要先创建一个更新器：并且需要更新的字段必须设置为`public volatile`

```java
public class AtomicIntegerFieldUpdaterTest {
    // 创建原子更新器，并设置需要更新的对象类和对象的属性
    private static AtomicIntegerFieldUpdater<User> a = AtomicIntegerFieldUpdater.
        newUpdater(User.class， "old");			// 指定原子更新old字段
    public static void main(String[] args) {
        // 设置柯南的年龄是10岁
        User conan = new User("conan"， 10);
        // 柯南长了一岁，但是仍然会输出旧的年龄
        System.out.println(a.getAndIncrement(conan));
        // 输出柯南现在的年龄
        System.out.println(a.get(conan));
    }
    public static class User {
        private String name;
        public volatile int old;				// 设置为volatile
        public User(String name， int old) {
            this.name = name;
            this.old = old;
        }
        public String getName() {
            return name;
        }
        public int getOld() {
            return old;
        }
    }
}
```

