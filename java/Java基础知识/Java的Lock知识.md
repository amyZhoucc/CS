# Java的Lock知识

这边只是对锁有个大致的了解，具体如何实现 or JDK的源码去看Java的并发知识和对应的源码分析

## 1. 锁的分类

主要就是根据如下的图来的

<img src="../pic/lock.png" style="zoom:50%;" >

## 2. 悲观锁 & 乐观锁

### 2.1 概念

按照线程是否需要锁住资源进行分类。

悲观锁：认为使用数据的时候一定有其他的线程也来修改，所以**先上锁再使用数据**

eg：synchronized 和 Lock 都是悲观锁

乐观锁：认为使用数据的时候是不会发生并发修改问题的，所以不上锁，**只是在更新的时候去判断其他线程是否也修改了该线程，即该结果是否是修改前的结果**，如果没有发生修改，那么就可以成功写入；如果发生修改了，就重试 or 报错。

eg：乐观锁都是用**无锁编程**来实现的，最常见的是CAS算法。Java原子类中的递增操作就通过**CAS自旋**实现的。

特性：

- 悲观锁适合**写操作多**的场景，先加锁可以保证写操作时数据正确。
- 乐观锁适合**读操作多**的场景，不加锁的特点能够使其读操作的**性能大幅提升**。

### 2.2 使用

```java
// ------------------------- 悲观锁的调用方式 -------------------------
// synchronized
public synchronized void testMethod() {		
    // 操作同步资源
}
// ReentrantLock
private ReentrantLock lock = new ReentrantLock(); // 需要保证多个线程使用的是同一个锁
public void modifyPublicResources() {
    lock.lock();
    // 操作同步资源
    lock.unlock();
}
```

```java
// ------------------------- 乐观锁的调用方式 -------------------------
private AtomicInteger atomicInteger = new AtomicInteger();  // 需要保证多个线程使用的是同一个AtomicInteger
atomicInteger.incrementAndGet(); //执行自增1
```

## 3. 自旋锁 & 适应性自旋锁

针对是获取同步资源失败，要不要发生阻塞，如果不发生阻塞，就能选择：自旋 or 自适应阻塞

### 自旋的意义

阻塞和唤醒线程是需要上下文切换等具体操作的，耗费CPU资源。

在许多场景中，**同步资源的锁定时间很短**。而如果等待时间很短，而为了这个很短的时间进行频繁的上下文切换，代价太大

所以，采用不放弃CPU，忙等待资源的到来——前提是**多处理器的设备，线程能够并行执行**（如果是单处理器，那么一定要阻塞，否则该线程会一直不放弃CPU，又没有时间片轮转，导致无限循环）

——主要就是为了减少上下文切换带来的代价，但是要在多处理器中才最有效

### 缺陷

它不能代替阻塞。自旋等待虽然避免了线程切换的开销，但它**要占用处理器时间**，等待时间过长，处理器资源浪费过多。

自旋等待的时间必须要有一定的限度，如果**自旋超过了限定次数**（默认是10次，可以使用-XX:PreBlockSpin来更改）**没有成功获得锁，就应当挂起线程。**

### 举例

实际上自旋锁就是一个概念，具体实现很多都是一个for循环

```java
for(;;) {		// 死循环，直到修改成功就跳出，否则一直尝试++
    int current = get();		// 原子操作，去获取当前的值
    int next = current + 1;		// 新的值
    if(compareAndSet(current, next))		// 期待的值为原来的值，如果是原来的值（未发生修改），那么修改为新的值
        return next;		// 如果CAS返回true，表示修改成功，那么返回当前的新值
}
```

——这就是一个自旋锁

### 自适应自旋锁

自适应意味着自旋的时间（次数）不再固定（前面的自旋锁是固定的，eg：10次后就阻塞），而是由**前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定**。如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行中，那么虚拟机就会认为这次自旋也是很有可能再次成功，进而它将允许自旋等待持续相对更长的时间。如果对于某个锁，自旋很少成功获得过，那在以后尝试获取这个锁时将可能省略掉自旋过程，直接阻塞线程，避免浪费处理器资源。

## 4. 无锁 & 偏向锁 & 轻量级锁 & 重量级锁

专门针对synchronized的，代表了synchronized的4种状态。主要是通过Java对象头的标记字段来进行改变的。

## 5. 公平锁 & 非公平锁

公平锁：指多个线程按照申请锁的顺序来获取锁，线程直接进入队列中排队，队列中的第一个线程才能获得锁。

优点：能够保证每个线程都能获得锁，不会出现饥饿现象；

缺点：整体吞吐效率相对非公平锁要低；CPU唤醒阻塞线程的开销比非公平锁大

非公平锁：多个线程加锁时**直接尝试获取锁**，获取不到才会到等待队列的队尾等待。但如果此时锁刚好可用，那么这个线程可以无需阻塞直接获取到锁，所以非公平锁有**可能出现后申请锁的线程先获取锁的场景**。——即，可以存在插队的情况

优点：可以减少唤起线程的开销，整体的吞吐效率高，因为线程有几率不阻塞直接获得锁，CPU不必唤醒所有线程。缺点是处于等待队列中的线程可能会饿死，或者等很久才会获得锁。

ReentrantLock可以选择公平锁还是非公平锁。

公平锁在获取同步状态时多了一个限制条件：hasQueuedPredecessors()，通过同步队列来实现多个线程按照申请锁的顺序来获取锁，从而实现公平的特性。

## 6. 可重入锁 & 非可重入锁

### 概念

可重入锁又名递归锁，是指在同一个线程在**外层方法获取锁的时候，再进入该线程的内层方法会自动获取锁**（前提锁对象得是同一个对象或者class）

eg：**Java中ReentrantLock和synchronized都是可重入锁**，可重入锁的一个优点是可一定程度避免死锁

```java
public class Widget {
    public synchronized void doSomething() {
        System.out.println("方法1执行...");
        doOthers();
    }

    public synchronized void doOthers() {
        System.out.println("方法2执行...");
    }
}
```

这两个方法可以正常执行，因为synchronized是可重入锁。

如果是不可重入的，就导致doOthers方法必须要等待doSomething执行完成后才能执行，而doSomething是调用了doOther，所以出现了死锁。

### 实现

重入锁ReentrantLock，非可重入锁NonReentrantLock的源码分析

ReentrantLock和NonReentrantLock都继承父类AQS，其**父类AQS中维护了一个同步状态status来计数重入次数**，status初始值为0。

- 当线程尝试获取锁时，可重入锁先尝试获取并更新status值，如果status == 0表示没有其他线程在执行同步代码，则把status置为1，当前线程开始执行。如果status != 0，则判断当前线程是否是获取到这个锁的线程，如果是的话执行status+1，且当前线程可以再次获取锁

  释放锁时，可重入锁同样先获取当前status的值，在当前线程是持有锁的线程的前提下。如果status-1 == 0，则表示当前线程所有重复获取锁的操作都已经执行完毕，然后该线程才会真正释放锁。

- 非可重入锁是直接去获取并尝试更新当前status的值，如果status != 0的话会导致其获取锁失败，当前线程阻塞。

  非可重入锁则是在确定当前线程是持有锁的线程之后，直接将status置为0，将锁释放

  ——status只可以是0/1

## 7. 独享锁 & 共享锁

即看他们能不能多个线程同时持有一把锁，注意和上面的可重入锁区分，可重入锁是指**同一个线程在执行一段代码时内层代码可以直接获得锁**。而这个分类是针对多线程的。

### 概念

独享锁也叫排他锁，是指该锁一次只能被一个线程所持有。如果线程T对数据A加上排它锁后，则其他线程不能再对A加任何类型的锁。**获得排它锁的线程即能读数据又能修改数据**

共享锁是指该锁可被多个线程所持有。如果线程T对数据A加上**共享锁后，则其他线程只能对A再加共享锁，不能加排它锁**。获得共享锁的线程只能读数据，不能修改数据。

### 实现

独享锁与共享锁也是通过AQS来实现的，通过实现不同的方法，来实现独享或者共享。

ReentrantLock：是独享锁，即不区分读还是写，都是统一只能有一个线程占用，其他线程均不能使用只能等待。

**ReentrantReadWriteLock**：有两把锁：ReadLock和WriteLock。

（ReadLock和WriteLock是靠内部类Sync实现的锁。Sync是AQS的一个子类，这种结构在CountDownLatch、ReentrantLock、Semaphore里面也都存在）

- 读锁：共享锁，读锁的共享锁可保证并发读非常高效	
- 写锁：独享锁

那么只有读读的时候，才能不断加共享锁；而读写、写读、写写的时候，都是需要涉及到独享锁，所以都不能多线程进行，符合逻辑。

——所以整体来说，比一般的并发锁性能有了很大的提高。

具体加锁：还是用到了AQS的state字段（32位），它可以描述线程持有锁的类型和个数

<img src="../pic/read&write_state.png">

将state变量“按位切割”切分成了两个部分，**高16位表示读锁状态（读锁个数**），**低16位表示写锁状态（写锁个数）**。

写锁的加锁源码：

```java
protected final boolean tryAcquire(int acquires) {
    Thread current = Thread.currentThread();
    int c = getState(); // 取到当前锁的状态
    int w = exclusiveCount(c); // 取写锁的个数w
    if (c != 0) { // 如果已经有线程持有了锁(c!=0)，可能是写锁、可能是读锁
        if (w == 0 || current != getExclusiveOwnerThread()) // 存在读锁 or 写锁但是持有锁的线程不是当前线程就返回失败
            return false;
        // 到这一步，说明是写锁，且当前线程持有写锁（可重入锁）
        if (w + exclusiveCount(acquires) > MAX_COUNT)  // 如果写入锁的数量大于最大数（65535，2的16次方-1）就抛出一个Error。
            throw new Error("Maximum lock count exceeded");
        // Reentrant acquire
        setState(c + acquires);
        return true;
    }
    // 到这边，说明c==0，没有线程获得锁
    if (writerShouldBlock() || !compareAndSetState(c, c + acquires)) // 如果当前线程需要阻塞那么就返回失败 or 不需要休眠，但是尝试通过CAS增加写线程数失败也返回失败。
        return false;
    // 到这边说明线程已经成功获得锁
    setExclusiveOwnerThread(current); // 如果c=0，w=0或者c>0，w>0（重入），则设置当前线程或锁的拥有者
    return true;
}
```

注意：

1. 写锁和读锁是互斥的，即读锁占有时，写锁不能占有，必须等读锁释放才能占用。原因在于：**必须确保写锁的操作对读锁可见**。如果允许读锁在已被获取的情况下对写锁的获取，那么正在运行的其他读线程就无法感知到当前写线程的操作。
2. 写锁是独占锁，但是写锁是可重入锁。且只有写锁全部释放完，读锁才有可能再次使用
3. 写锁的state的计数，是指同一个线程重入的个数；读锁的state计数，是指不同线程加锁的个数

```java
protected final int tryAcquireShared(int unused) {
    Thread current = Thread.currentThread();
    int c = getState();
    if (exclusiveCount(c) != 0 &&
        getExclusiveOwnerThread() != current)
        return -1;                                 // 如果其他线程已经获取了写锁，则当前线程获取读锁失败，进入等待状态
    // 到这边，说明没有写锁，最多只有读锁
    int r = sharedCount(c);			// 获得读锁
    if (!readerShouldBlock() &&			// 当前线程不需要阻塞 且 读锁个数没有超上限，且CAS获取锁成功
        r < MAX_COUNT &&
        compareAndSetState(c, c + SHARED_UNIT)) {
        if (r == 0) {					// 如果当前线程是第一个获取锁的
            firstReader = current;		// 记录一下第一个占有锁的人，且记录占用的个数
            firstReaderHoldCount = 1;
        } else if (firstReader == current) {		// 还是第一个线程重复加锁
            firstReaderHoldCount++;
        } else {				// 如果不是第一个获得锁，且也不是第一个线程重复获得锁
            HoldCounter rh = cachedHoldCounter;
            if (rh == null || rh.tid != getThreadId(current))
                cachedHoldCounter = rh = readHolds.get();
            else if (rh.count == 0)
                readHolds.set(rh);
            rh.count++;
        }
        return 1;
    }
    return fullTryAcquireShared(current);
}
```

参考：

1. https://tech.meituan.com/2018/11/15/java-lock.html（赞！）
2. 





# PS：额外知识

## 1. CAS原理分析

CAS是一种无锁算法，能够不使用锁而实现并发操作

why有CAS呢？

锁会引起阻塞，代价太大，如果在读操作多的场景下，如果每次访问共享资源都需要上锁，那么降低并发性能。所以可以选择用无锁来实现

所以可以使用CAS的硬件支持的原子变量来实现比较简单的并发操作，eg：原子类中的i++；

ps：

1. Lock一定需要和unlock配套使用——本质上是CAS+AQS的底层实现。
2. LongAdder 原子类在 JDK1.8 中新增的类， 也是在 java.util.concurrent.atomic 并发包下。LongAdder 适合于高并发场景下，特别是写大于读的场景，相较于 AtomicInteger、AtomicLong 性能更好，代价是消耗更多的空间，以空间换时间。

### （1）背景知识

在Java的原子类中，有CAS，它是原子类实现的核心。——**CAS是底层硬件的支持，可以将CAS函数看成单个指令**。

#### 概念：

函数介绍：CAS 全称是 compare and swap，字面意思就是比较并且交换（值）。是一种在多线程下的实现同步功能的机制，即能够做到并发修改。

涉及到三个操作数：

- 需要读写的内存值 V。
- 进行比较的值 A。
- 要写入的新值 B。

实现逻辑：CAS 的实现逻辑是将内存位置处的数值V与预期数值A比较，若相等，则将内存位置处的值替换为新值B。若不相等，则不做任何操作。如果修改成功返回——true；未修改返回——false。整个流程是满足原子性的。一般来说该操作是不断重试的过程，直到成功。

（在 Java 中，Java 并没有直接实现 CAS，**CAS 相关的实现是通过 C++ 内联汇编的形式实现的**。Java 代码需通过 JNI 才能调用）

#### 硬件保证：

背景知识：Intel 处理器可以保证**单次访问内存对齐的指令以原子的方式执行**——即一次读/写指令都是原子性的。但是如果两次访存指令，比如说`i++/ i += 1`，都是涉及到了一次读指令、一次写指令，那么会存在多线程的并发修改，导致结果不符合预期。

而底层的硬件保证：在多处理器环境下，**LOCK# 信号可以确保处理器独占使用某些共享内存**。lock 可以被添加在下面的指令前：ADD, ADC, AND.. 。通过**在 inc 指令前添加 lock 前缀，即可让该指令具备原子性**。多个核心同时执行同一条 inc 指令时，**会以串行的方式进行**，也就避免了上面所说的那种情况。

那么底层的硬件如何保证lock下的原子性呢：

有两种方式保证处理器的某个核心独占某片内存区域：

- **锁定总线**，使用Lock信号，让某个核心独占使用总线，但这样代价太大——锁定后，其他核心就不能访问内存了

- **锁定缓存**，某处内存数据被缓存在处理器缓存中。处理器发出的 LOCK# 信号不会锁定总线，而是**锁定缓存行对应的内存区域**。其他处理器在这片内存区域锁定期间，无法对这片内存区域进行相关操作。具体来说，就是某个处理器对缓存中的共享变量进行了操作，其他处理器会有个**嗅探机制，将其他处理器的该共享变量的缓存失效，待其他线程读取时会重新从主内存中读取最新的数据**，它是基于 **MESI 缓存一致性协议**来实现的。

  ——代价变小，只是无法访问对应的内存区域，而其他内存区域还是可以访问的

  （如果处理器不支持“锁定缓存”，那么只能使用总线锁定）

### （2）源码分析

原子类 AtomicInteger 中的 compareAndSet 方法进行分析

```java
// AtomicInteger部分相关源码

// 静态/实例变量，代码块等
private static final Unsafe unsafe = Unsafe.getUnsafe();	// 获取并操作内存的数据
private static final long valueOffset;	// valueOffset，是记录value的在该对象内存中的偏移量
private volatile int value;			// 存储实际的值，volatile修饰，能够保证可见性

/* 这个是发生在类加载过程中的，是最先初始化的，是调用了unsafe实例对象的objectFieldOffset实例方法，获得AtomicInteger类文件中value的偏移量 */
static {		
    try {
        // 保存变量 value 在类对象unsafe中的偏移
        valueOffset = unsafe.objectFieldOffset
            (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}

public final class Unsafe {
    // compareAndSwapInt 是 native 类型的方法
    public final native boolean compareAndSwapInt(Object o, long offset,
                                                  int expected,int x);
}

public final boolean compareAndSet(int expect, int update) {
    /*
   	 * compareAndSet 实际上只是一个壳子，传递的是value的偏移量——可以间接获取value的值；期待的值；需要更新的值
     */
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}

public final int incrementAndGet() {		// 等同于++i
    for(;;) {		// 死循环，直到修改成功就跳出，否则一直尝试++
        int current = get();		// 原子操作，去获取当前的值
        int next = current + 1;		// 新的值
        if(compareAndSet(current, next))		// 期待的值为原来的值，如果是原来的值（未发生修改），那么修改为新的值
        	return next;		// 如果CAS返回true，表示修改成功，那么返回当前的新值
    }
}
```

可见：incrementAndGet的原子性的核心就是compareAndSet，而其实现本质上是native方法**`compareAndSwapInt`**，下面分析unsafe及CAS的实现：

```c++
// unsafe.cpp
/*
 * UNSAFE_ENTRY 和 UNSAFE_END 都是宏，
 * 在预编译期间会被替换成真正的代码。下面的 jboolean、jlong 和 jint 等是一些类型定义（typedef）：
 * 
 * jni.h
 *     typedef unsigned char   jboolean;
 *     typedef unsigned short  jchar;
 *     typedef short           jshort;
 *     typedef float           jfloat;
 *     typedef double          jdouble;
 * 
 * jni_md.h
 *     typedef int jint;
 *     #ifdef _LP64 // 64-bit
 *     typedef long jlong;
 *     #else
 *     typedef long long jlong;
 *     #endif
 *     typedef signed char jbyte;
 */
UNSAFE_ENTRY(jboolean, Unsafe_CompareAndSwapInt(JNIEnv *env, jobject unsafe, jobject obj, jlong offset, jint e, jint x))
  UnsafeWrapper("Unsafe_CompareAndSwapInt");
  oop p = JNIHandles::resolve(obj);		// p是取出的对象，就是前面的this
  // 根据偏移量，计算 value 的地址。这里的 offset 就是 AtomaicInteger 中的 valueOffset
  jint* addr = (jint *) index_oop_from_field_offset_long(p, offset);
  // 调用 Atomic 中的函数 cmpxchg，该函数声明于 Atomic.hpp 中
  return (jint)(Atomic::cmpxchg(x, addr, e)) == e;
UNSAFE_END

// atomic.cpp
unsigned Atomic::cmpxchg(unsigned int exchange_value,
                         volatile unsigned int* dest, unsigned int compare_value) {
  assert(sizeof(unsigned int) == sizeof(jint), "more work to do");
  /*
   * 根据操作系统类型调用不同平台下的重载函数，这个在预编译期间编译器会决定调用哪个平台下的重载
   * 函数。相关的预编译逻辑如下：
   * 
   * atomic.inline.hpp：
   *    #include "runtime/atomic.hpp"
   *    
   *    // Linux
   *    #ifdef TARGET_OS_ARCH_linux_x86
   *    # include "atomic_linux_x86.inline.hpp"
   *    #endif
   *   
   *    // 省略部分代码
   *    
   *    // Windows
   *    #ifdef TARGET_OS_ARCH_windows_x86
   *    # include "atomic_windows_x86.inline.hpp"
   *    #endif
   *    
   *    // BSD
   *    #ifdef TARGET_OS_ARCH_bsd_x86
   *    # include "atomic_bsd_x86.inline.hpp"
   *    #endif
   * 
   * 接下来分析 atomic_windows_x86.inline.hpp 中的 cmpxchg 函数实现
   */
  return (unsigned int)Atomic::cmpxchg((jint)exchange_value, (volatile jint*)dest,
                                       (jint)compare_value);
}
```

```c++
// atomic_windows_x86.inline.hpp
#define LOCK_IF_MP(mp) __asm cmp mp, 0  \   // 如果mp == 0，那么是单核cpu，那么会跳到L0开始的地方，即cmpxchg处
                       __asm je L0      \	
                       __asm _emit 0xF0 \	// 如果mp!=0,不跳转，0XF0是lock前缀的机器码，没有使用lock而是直接用字节码
                       __asm L0:
              
inline jint Atomic::cmpxchg (jint exchange_value, volatile jint* dest, jint compare_value) {
  // 判断是否多核cpu，如果是多核cpu，那么就给总线加锁，那么其他cpu不能通过总线访问内存（性能会受到影响），如果是单核系统，那么不需要添加
  int mp = os::is_MP();			
  __asm {
    mov edx, dest		// value地址的偏移量
    mov ecx, exchange_value	// newValue的值
    mov eax, compare_value	// expect的值
    LOCK_IF_MP(mp)
    // dword: 全称是 double word,word = 2 byte，dword = 4 byte = 32 bit,双字节
    // ptr: 全称是 pointer，与前面的 dword 连起来使用，表明访问的内存单元是一个双字单元
    cmpxchg dword ptr [edx], ecx	// 比较并交换，如果eax == [edx]，那么将ecx ->[edx]；如果不相等，那么就不赋值
  }
}
```

——可以发现，关键的指令就是`cmpxchg dword ptr [edx], ecx`，该指令实际上不能保证原子性，还是存在被打断的可能。实际上，保证该指令原子性的语句是`LOCK_IF_MP(mp)`，即如果是多核，那么就上锁；如果是单核，那么该指令一定不会被抢占（单核肯定是串行执行的），就不用上锁。从而，保证该指令的原子性

确保对内存的读-改-写操作原子执行

而该Lock指令，和OS的互斥锁等存在不同

ps：

Intel手册对lock前缀的说明如下：

1. 确保对内存的读-改-写操作原子执行。在Pentium及Pentium之前的处理器中，带有lock前缀的指令在执行期间会**锁住总线**，使得其他处理器暂时无法通过总线访问内存。很显然，这会带来昂贵的开销。

   从Pentium 4，Intel Xeon及P6处理器开始，intel在原有总线锁的基础上做了一个很有意义的优化：如果要访问的内存区域（area of memory）在lock前缀指令执行期间已经在处理器内部的缓存中被锁定（即包含该内存区域的缓存行当前处于独占或以修改状态），并且该内存区域被完全包含在单个缓存行（cache line）中，那么处理器将直接执行该指令。由于在指令执行期间该缓存行会一直被锁定，其它处理器无法读/写该指令要访问的内存区域，因此能保证指令执行的原子性。这个操作过程叫做**缓存锁定**（cache locking），缓存锁定将大大降低lock前缀指令的执行开销，但是当多处理器之间的竞争程度很高或者指令访问的内存地址未对齐时，仍然会锁住总线。

2. **禁止该指令与之前和之后的读和写指令重排序**。

3. **把写缓冲区中的所有数据刷新到内存中**。（volatile的功能之一，所以lock是包含了volatile的可见性的）

### （3）CAS的问题

1. **ABA问题**

   CAS会将内存中的值V和期待的值A进行比较，如果一样就进行交换；但是存在一个问题：在这个过程中A变成B，又变成A，那么CAS无法检测，但是实际上是发生了变化的

   eg：在银行汇款中，我从提款机取50，但是有A和B两个线程同时操作，约束一个线程能够执行成功即可。A、B两个线程同时修改100->50，A成功修改了，变成50，B被阻塞了，阻塞期间C汇款了50，此时又变成100，而B阻塞回来发现就是100，CAS满足要求，就又修改成50，正确应该是100，但是现在是50。

   解决方法：在变量前面**添加版本号**，每次变量更新的时候都把版本号加一，这样变化过程就从“A－B－A”变成了“1A－2B－3A”。

   - JDK从1.5开始提供了**AtomicStampedReference**类来解决ABA问题，具体操作封装在compareAndSet()中。compareAndSet()首先检查当前引用和当前标志与预期引用和预期标志是否相等，如果都相等，则以原子方式将引用值和标志的值设置为给定的更新值。

2. **循环时间长开销大**

   CAS操作如果长时间不成功，会导致其一直自旋，给CPU带来非常大的开销。

3. **只能保证一个共享变量的原子操作**

   对多个共享变量操作时，CAS是无法保证操作的原子性的。Java从1.5开始JDK提供了**AtomicReference**类来保证引用对象之间的原子性，可以把多个变量放在一个对象里来进行CAS操作

参考文章：

1. https://segmentfault.com/a/1190000014858404
2. https://juejin.cn/post/6844903558937051144
3. https://zhuanlan.zhihu.com/p/94762520

## 2. ReentrantLock & synchronized比较

|              | ReentrantLock                                                | synchronized                 |
| ------------ | ------------------------------------------------------------ | ---------------------------- |
| 锁的实现机制 | 依赖AQS + CAS                                                | 监视器模式                   |
| 灵活性       | 支持响应中断、超时、尝试获取锁<br/>（是否在等待中进行响应，需要配置可响应的锁） | 不灵活                       |
| 释放形式     | **显式调用**unlock()才能释放锁                               | 该部分结束后会自动释放监视器 |
| 锁类型       | 公平锁 or 非公平锁（可配置）                                 | **非公平锁**                 |
| 条件队列     | 可关联多个条件队列                                           | 只能关联一个条件队列         |
| 可重入性     | 可重入                                                       | 可重入                       |
|              |                                                              |                              |

```java
// **************************Synchronized的使用方式**************************
// 1.同步代码块
synchronized (this) {}			// 锁住的是实例对象，但是粒度较小（同步代码块上下的代码还是能够不获取锁去执行的）
// 2.用于对象
synchronized (object) {}		// 锁住的是实例对象
// 3.同步方法
public synchronized void test () {}		// 锁住的是实例对象
// 4.静态方法
public static synchronized void test(){}		// 锁住的是类对象
// 4.可重入
for (int i = 0; i < 100; i++) {
    synchronized (this) {}
}
```

```java
// **************************ReentrantLock的使用方式**************************
public void test () throw Exception {
	// 1.初始化选择公平锁、非公平锁
	ReentrantLock lock = new ReentrantLock(true);
	// 2.可用于代码块
	lock.lock();
	try {
		try {
			// 3.支持多种加锁方式，比较灵活; 具有可重入特性
			if(lock.tryLock(100, TimeUnit.MILLISECONDS)){ }
		} finally {
			// 4.手动释放锁
			lock.unlock()
		}
	} finally {
		lock.unlock();
	}
}
```

参考：

1. https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html