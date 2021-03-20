# Java并发知识

根本：Java中所使用的并发机制依赖于**JVM的实现和CPU的指令**

# 1. Java并发机制的底层实现原理

## 1.1 volatile

和synchronized不同，volatile不是锁，而是**在多处理器开发中保证了共享变量的“可见性“**。

可见性：当一个线程去修改该变量时，另外一个线程能够读到该变化（并发存在的一个问题是，某个结点读取值之后被抢占了，而再次调度回来的时候，实际上值已经改变，而线程仍旧用原来的值，这个就是并发的问题）

### volatile的实现原理：

volatile的特性：如果一个字段被声明成volatile，Java线程内存模型确保所有线程看到这个变量的值是一致的。

CPU的术语：（在技术博客中遇到过）：

<img src="../pic/cpu_word.jpg" style="zoom: 80%;" >

eg：

```java
volatile instance;
instance = new Singleton();
```

—— volatile修饰的变量被赋值，就是写操作。

转换为汇编：

```
0x01a3de1d: movb $0×0,0×1104800(%esi);0x01a3de24: 
lock addl $0×0,(%esp);
```

下面一行`lock addl $0×0,(%esp);`有volatile变量修饰的共享变量进行写操作的时候会多出。

Lock前缀的指令在多核处理器下会引发了两件事：

- 将当前处理器缓存行的数据**写回到系统内存**（而不是暂时写在缓存行里，待适合的时机写回内存）
- 这个写回内存的操作会使**在其他CPU里缓存了该内存地址的数据无效**。

解释：

- 对于第一条：常规来说处理器不直接和内存进行通信，而是先将系统内存的数据读到内部缓存（L1，L2或其他）后再进行操作，但操作完不知道何时会写到内存，而**volatile的变量进行写操作，JVM就会向处理器发送一条Lock前缀的指令，将这个变量所在缓存行的数据写回到系统内存**
- 对于第二条：如何确保缓存的同步性——为了保证各个处理器的缓存是一致的，就**会实现缓存一致性协议**，每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。

volatile的两条实现原则：

1. **Lock前缀指令会引起处理器缓存回写到内存**（内存独占式的）

   - 对于旧的CPU，Lock会**锁总线**，导致其他CPU不能访问总线，那么就不能访问内存，那么该CPU就独占内存——但是锁总线开销大
   - 在最近的处理器里，LOCK信号一般不锁总线，而是**锁缓存**。如果访问的内存区域已经缓存在处理器内部，则不会声言LOCK#信号。相反，它会**锁定这块内存区域的缓存并回写到内存**，并使用**缓存一致性机制来确保修改的原子性**，此操作被称为**“缓存锁定”**，缓存一致性机制会阻止同时修改由两个以上处理器缓存的内存区域数据。——只锁部分区域，所以开销少

2. **一个处理器的缓存回写到内存会导致其他处理器的缓存无效**

   处理器使用**嗅探技术**保证它的内部缓存、系统内存和其他处理器的缓存的数据在总线上保持一致。

   如果通过嗅探一个处理器来检测其他处理器打算写内存地址，而这个地址当前处于共享状态，那么正在嗅探的处理器将使它的缓存行无效，在下次访问相同内存地址时，强制执行缓存行填充。

eg：volatile的使用技巧：

JDK7中：LinkedTransferQueue，在使用volatile变量时，用一种追加字节的方式来优化队列出队和入队的性能

```java
/＊＊ 队列中的头部节点 ＊/
private transient final PaddedAtomicReference<QNode> head;
/＊＊ 队列中的尾部节点 ＊/
private transient final PaddedAtomicReference<QNode> tail;
static final class PaddedAtomicReference <T> extends AtomicReference <T> {
  //使用很多4个字节的引用追加到64个字节
  Object p0, p1, p2, p3, p4, p5, p6, p7, p8, p9, pa, pb, pc, pd, pe;
  PaddedAtomicReference(T r) {
    super(r);
  }
}
public class AtomicReference <V> implements java.io.Serializable {
  private volatile V value;			// 4字节
  //省略其他代码
}
```

所以，PaddedAtomicReference对象，有64B。

英特尔酷睿i7等处理器，**缓存行是64个字节宽，不支持部分填充缓存行**。如果头尾都不足64字节的话，处理器会将它们都读到同一个高速缓存行中。所以如果需要修改，那么会将整个缓存行锁定，导致头尾结点均被锁定，而导致其他处理器不能访问自己高速缓存中的头尾结点了——降低了并发性。**追加到64字节的方式来填满高速缓冲区的缓存行，避免头节点和尾节点加载到同一个缓存行，使头、尾节点在修改时不会互相锁定。**

当然，这种字节填充也是有适用场景的，某些场景下是不适用的：

- 缓存行非64字节宽的处理器，某些是32位宽的等
- 共享变量不会被频繁地写，锁的几率也非常小，就没必要通过追加字节的方式来避免相互锁定。

## 1.2 synchronized的实现原理与应用

synchronized进行了各种优化之后，有些情况下它就并不那么重了

synchronized实现同步的基础：**Java中的每一个对象都可以作为锁**

有如下3种形式：

1. 对于普通同步方法，锁是**当前实例对象**
2. 对于静态同步方法，锁是当前类的**Class对象**
3. 对于同步方法块，锁是Synchonized**括号里配置的对象**

synchronized存在4种状态：无锁 -- 偏向锁 -- 轻量级锁 -- 重量级锁，会随着竞争情况进行升级，注意**只能升级不能降级**，即偏向锁升级为轻量级锁后不会变回偏向锁——目的是为了提高获得锁和释放锁的效率

### Java对象头

synchronized是悲观锁，在操作同步资源之前需要给同步资源先加锁，这把**锁就是存在Java对象头**里的

对象是数组类型，则虚拟机用**3个字宽**（Word，1W=4B）**存储对象头，**如果对象是非数组类型，则用**2字宽存储对象头**。

<img src="../pic/java_head.jpg">

理解：

1. 第一个Word存放**标记字段**

   默认存储对象的HashCode、分代年龄和锁标记位

2. 第二个Word存放**类型指针**，向它的类元数据的指针，主要是用来告诉虚拟机**该对象是哪个类的实例**

3. 第个Word是数组特有的，记录数组长度，这就是为什么数组能够直接获取长度，arr.length——放在对象头里了

所以锁是存在于Mark-word的

默认的Mark-word存放如下内容：

<img src="../pic/default-head.jpg">

在运行期间，Mark Word里存储的数据会**随着锁标志位的变化而变化**。

<img src="../pic/state-head.jpg" style="zoom:80%;" >

<img src="../pic/state-head_64.jpg">

### Monitor

理解为一个同步工具或一种同步机制，通常被描述为一个对象。

因为：每一个Java对象就有一把看不见的锁，称为**内部锁或者Monitor锁**。——即对象带锁，如果存在并发操作该对象的情况，需要获取该锁。

而线程可能会访问很多共享对象，所以线程需要记录获得的这些对象和锁的信息——使用Monitor对象来存储

**Monitor是线程私有的数据结构**：monitor中有一个Owner字段存放拥有该锁的线程的唯一标识，表示该锁被这个线程占用。

线程种monitor相关的：

- 每一个线程都有一个可用monitor record列表，存放Monitor对象，一个上锁的对象对应一个monitor对象
- 同时还有一个全局的可用列表

所以：synchronized通过Monitor来实现线程同步，**Monitor是依赖于底层的操作系统的互斥锁（是重量级锁）**来实现的线程同步。

### 锁的状态转换

4个状态会随着竞争情况逐渐升级。**锁可以升级但不能降级**，目的是为了提高获得锁和释放锁的效率

#### 1. 无锁

没有对资源进行锁定，所有的线程都能访问并修改同一个资源，但同时只有一个线程能修改成功。

**CAS就是用无锁的思想实现的**（注意，不是无锁是CAS实现的，而是反过来的）

步骤：修改操作在循环内进行，线程会不断的尝试修改共享资源。多个线程同时修改该值，只有一个线程才能修改成功，而其他线程会不断尝试直到成功

无锁无法全面代替有锁，但无锁在某些场合下的性能是非常高的。

#### 2. 偏向锁

大部分情况下：锁不仅不存在多线程竞争，而且总是由**同一线程多次获得**。（但是还是存在并发的情况，所以还是需要锁的），所以为了**减少获取锁的代价**，提出偏向锁。

线程会自动获取锁，降低获取锁的代价。

步骤：当一个线程访问同步代码块并获取锁时，会在Mark Word里存储**锁偏向的线程ID**。在线程进入和退出同步块时不再通过CAS操作来加锁和解锁，而是**检测Mark Word里是否存储着指向当前线程的偏向锁**。

1. 如果标志字段存储当前线程ID，说明该线程已经获得过锁，那么可以再次进入（即，可能之前已经进入过一次临界区，然后执行完毕出去了，但是偏向锁不会主动放弃，还是指向它，那么需要再次进入时直接看MarkWord即可）
2. 如果检测失败，那么还需要看MarkWord的的偏向锁的标志：
   - 如果设置为1，表示是偏向锁，那么失败是由于其他线程已经占用该锁了，尝试使用**CAS将偏向锁指向当前线程**
   - 如果设置为0，表示是无锁，那么需要用CAS去竞争锁

——主要是由于在访问临界区时只需要简单的判断即可，而轻量级锁的获取和释放需要多次CAS原子操作。偏向锁只需要在**置换ThreadID的时候依赖一次CAS原子指令**即可。

##### 偏向锁的释放：

触发条件：偏向锁只有遇到其他线程**尝试竞争偏向锁**（即上面的2.1状态）时，持有偏向锁的线程才会释放锁，**线程不会主动释放偏向锁**（一旦持有，只有出现竞争才释放）

操作前提：需要等待全局安全点（在这个时间点上没有正在执行的字节码）

具体操作：

- 暂停拥有偏向锁的线程

- 检查持有偏向锁的线程是否活着：

  - 如果没有，将对象头设置成无锁状态，然后竞争的线程通过CAS竞争获得

  - 如果存活，看当前线程是否需要竞争锁（即仍然需要获取该锁）：

    - 如果需要竞争，那么升级为轻量级锁
    - 如果不需要竞争，那么另外一个线程获得偏向锁

    （Epoch表示该对象的偏向锁的撤销次数，如果次数>40，表示竞争次数过多，那么此对象不再适合用偏向锁，而下次再次撤销锁时，直接升级为偏向锁，而不进行判断是否需要竞争）

  如下图：

<img src="../pic/lock_race.png">

偏向锁在Java 6和Java 7里是默认启用的，可以手动关闭

#### 3. 轻量级锁

上面的，如果多个线程竞争获取偏向锁，就会自动升级为轻量级锁。其他线程会**通过自旋的形式尝试获取锁**，不会阻塞，从而提高性能。

##### 轻量级锁加锁

1. JVM会先在**当前线程的栈桢**中创建用于存储**锁记录**的空间，并将对象头中的Mark Word复制到锁记录中——官方称为Displaced MarkWord；

2. 线程尝试使用**CAS**将对象头中的Mark Word替换为指向锁记录的指针，并且将锁记录中的owner指向该对象的markword

   即互相指示：对象头的标志字段指向锁记录，而锁记录指向该标志字段

3. 如果成功，当前线程获得锁；

   如果失败，表示其他线程竞争锁，当前线程便尝试使用**自旋来获取锁。**

当自旋超过一定的次数，或者一个线程在持有锁，一个在自旋，又有第三个来访时，轻量级锁升级为重量级锁。

##### 轻量级锁解锁

会使用原子的CAS操作将Displaced Mark Word替换回到对象头。

如果替换成功：表示没有竞争；

如果替换失败：存在竞争（另一个线程已经将轻量级锁升级到重量级锁，所以标志位变化了），那么说明存在竞争，释放锁，并且唤醒所有在等待的线程，重新开始竞争。

#### 4. 重量级锁

由于轻量级锁下的自旋时间过长（自旋会造成CPU的浪费），多个线程竞争锁，导致锁升级。

锁标志的状态值变为“10”，等待锁的线程都会进入**阻塞状态**

#### 5. 总结：

偏向锁通过对比Mark Word解决加锁问题，从而避免使用CAS，只有在变换偏向对象时需要用到CAS;

轻量级锁是通过用CAS操作（指向MarkWord的指针是用CAS获得的）和自旋（获得锁失败，就自旋等待）来解决加锁问题

重量级锁是将除了拥有锁的线程以外的线程都阻塞。

<img src="../pic/lock_advdis.jpg">

## 1.3 原子操作实现的基本原理

原子操作：不可被中断的一个或一系列操作。即即使出现并发情况也能保证结果的正确性。

<img src="../pic/atomic_base.jpg" style="zoom: 80%;" >

### 原子操作的实现

常用：基于对缓存加锁或总线加锁的方式来实现多处理器之间的原子操作

而处理器会自动保证基本的内存操作的原子性，即读取或者写入一个字节就是原子的，即该cpu在操作该字节，而其他cpu是不能访问该字节所在的内存地址。

- 使用总线锁保证原子性：

  如果多个处理器同时对共享变量进行读改写操作（i++就是经典的读改写操作，读取，改变值，写回），那么需要保证原子性就需要：锁定总线——

  使用处理器提供的一个LOCK #信号，当一个处理器在总线上输出此信号时，其他处理器的请求将被阻塞住，那么该处理器可以**独占共享内存。**——即锁定总线，导致整个CPU和整个内存的通信都锁定了

- 使用缓存锁保证原子性：

  总线锁定的开销比较大。

  频繁使用的内存会缓存在处理器的L1、L2和L3高速缓存里，那么原子操作就可以直接在处理器内部缓存中进行

  使用缓存锁定来保证原子性：内存区域如果被缓存在处理器的缓存行中，并且在Lock操作期间被锁定。当它执行锁操作回写到内存时，处理器不在总线上声言LOCK #信号，而是修改内部的内存地址，并利用它的缓存一致性机制来保证操作的原子性。

  缓存一致性机制，某个CPU将缓存行写回后，会通知其他CPU对该内存地址的缓存无效，需要重新读取。

# 2. Java并发编程的基础

## 2.1 线程

### 2.1.1 线程概念 & Java中的线程

现代操作系统在运行一个程序时，会为其创建一个进程。例如启动一个Java进程，就会为其创建一个进程。

现代操作系统调度的最小单元是线程，一个进程里可以有多个线程，线程有独立的程序计数器、堆栈和局部变量，能共享进程的内存变量。

Java程序天生就是多线程程序，执行main()方法的是一个名称为main的线程。

可以通过代码查看普通的Java程序包含哪些线程：

<img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20210319115440198.png" alt="image-20210319115440198" style="zoom:80%;" />

1. Monitor Ctrl-Break：在idea中特有的线程。且debug启动的不会出现，只有run启动的会出现

   一般是埋坑的，有些调试的时候会发现线程个数不符合预期，那要考虑是否是在idea中特有的线程引起的

2. Attach Listener：监听各种请求的socket连接,把执行的操作扔给VM Thread

3. Signal Dispatcher：接受各种信号，交给JVM去处理发出这些信号的线程

4. Finalizer：调用对象的finalize方法的线程

5. Reference Handler：清除reference的线程

6. **main：用户程序的入口**

（`VM Thread`：**线程母体,最原始的线程**，单例，里面有个队列，存放各种的操作，它负责loop处理队列中的操作.（包括对其他线程的创建，分配和对象的清理，cms-mark等工作））

### 2.1.2 why引入多线程

1. **多核处理器——并行方面**

   现代处理器都配备了多核，所以为了充分利用资源

   将计算逻辑分配到多个处理器核心上，就会显著减少程序的处理时间

2. **更快的响应时间——并发方面**

   对于一些IO密集的程序，大部分时间都用在等待IO响应了，而其他任务得不到执行，所以引入多线程，可以在等待的期间去执行其他的任务

3. 更好的编程模型

   Java提供了一系列的模型，而程序员只需要关注如何解决问题，而不用考虑其他的问题。

### 2.1.3 线程优先级

Java线程中，通过一个整型成员变量priority来控制优先级，优先级的范围从1~10，默认是5，可以通过setPriority(int)方法来修改优先级。

**优先级高的线程分配时间片的数量要多于优先级低的线程**。

- 针对频繁阻塞（休眠或者I/O操作）的线程需要设置较高优先级
- 偏重计算（需要较多CPU时间或者偏运算）的线程则设置较低的优先级，确保处理器不会被独占。

但是注意：

线程优先级不能作为程序正确性的依赖，因为**操作系统可以完全不用理会Java线程对于优先级的设定**。

### 2.1.4 线程状态

归总是5个状态

| 状态                          | 描述                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| new态                         | 线程刚被创建，还未调用start                                  |
| runnable状态（就绪态）        | 随时可以开始执行                                             |
| running（运行态）             | 获得CPU资源，可以开始运行（Java将runnable和running统称为一类） |
| blocked（阻塞状态，同步阻塞） | 线程阻塞于锁                                                 |
| waiting（等待状态）           | 需要等待其他线程做出一定动作，由其他线程notify才能唤醒       |
| time_waiting（超时等待）      | 如果在限定时间内没有被其他线程唤醒，可以超时后自行唤醒，到runnable状态 |
| terminated                    | 终止状态                                                     |

<img src="../pic/thread_state.jpg" alt="image-20210319130003866" style="zoom: 70%;" />

<img src="../pic/thread_state.png" style="zoom:80%;" >

ps：阻塞在java.concurrent包中Lock接口的线程状态却是等待状态，因为java.concurrent包中Lock接口对于阻塞的实现均使用了LockSupport类中的相关方法。

### 2.1.5 Daemon线程——特殊线程

Daemon线程是一种支持性线程，它主要被用作**程序中后台调度以及支持性工作**。当一个Java虚拟机中**不存在非Daemon线程的时候，Java虚拟机将会退出**。

在线程调用start变成就绪态之前可以调用**Thread.setDaemon(true)**将线程设置为Daemon线程。

——只能在线程启动之前进行设置，而不能在启动之后。且，Daemon线程可以用户设置。

```java
public static void main(String[] args) {
    Thread thread = new Thread(new DaemonRunner(), "DaemonRunner");
    thread.setDaemon(true);			// setDaemon在start之前
    thread.start();
}
```

#### why需要设置Daemon线程

因为当**所有线程都运行结束时，JVM需要退出**，进程结束。那么如果**有一个线程没有退出，JVM就不会退出。**——所以，所有线程需要能够及时结束。

但是存在一种线程目的就是无限循环，例如，一个定时触发任务的线程。所以需要另外一个线程来强制该线程终止，这个就是Daemon线程。

而jvm中只存在Daemon线程时，jvm就会终止。jvm退出的时候，所有的Daemon线程必须要立即终止，所以Daemon的finally中的语句不一定会被执行到。

但是需要注意：守护线程**不能持有任何需要关闭的资源**，例如打开文件等，因为虚拟机退出时，守护线程没有任何机会来关闭文件，这会导致数据丢失。

### 2.1.6 线程的创建

如何构造一个线程：

调用Thread的构造方法：都会默认去调用init的方法，init就是对线程初始化（初始化有点类似于OS中的）

```java
public Thread(ThreadGroup group, Runnable target, String name, long stackSize) {
    init(group, target, name, stackSize);
}
```

线程初始化操作：

主要进行一系列的赋值，而都是由其parent线程来进行空间分配的，而child线程继承了parent是否为Daemon、优先级和加载资源的contextClassLoader以及可继承的ThreadLocal

最后给该线程一个tid

```java
private void init(ThreadGroup g, Runnable target, String name, long stackSize) {
    init(g, target, name, stackSize, null, true);
}

private void init(ThreadGroup g, Runnable target, String name,
                  long stackSize, AccessControlContext acc,
                  boolean inheritThreadLocals) {
    if (name == null) {				// 必须要传递线程的名字，否则会抛出异常
        throw new NullPointerException("name cannot be null");			
    }

    this.name = name;

    Thread parent = currentThread();			// 新建线程的父线程是当前线程
    SecurityManager security = System.getSecurityManager();
    if (g == null) {				// 设置所属的线程组
        /* Determine if it's an applet or not */

        /* If there is a security manager, ask the security manager
               what to do. */
        if (security != null) {
            g = security.getThreadGroup();
        }

        /* If the security doesn't have a strong opinion of the matter
               use the parent thread group. */
        if (g == null) {
            g = parent.getThreadGroup();
        }
    }

    /* checkAccess regardless of whether or not threadgroup is
           explicitly passed in. */
    g.checkAccess();

    /*
         * Do we have the required permissions?
         */
    if (security != null) {
        if (isCCLOverridden(getClass())) {
            security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
        }
    }

    g.addUnstarted();

    this.group = g;					// 线程组设置
    this.daemon = parent.isDaemon();			// 将daemon、priority属性设置为父线程的对应属性
    this.priority = parent.getPriority();
    if (security == null || isCCLOverridden(parent.getClass()))
        this.contextClassLoader = parent.getContextClassLoader();
    else
        this.contextClassLoader = parent.contextClassLoader;
    this.inheritedAccessControlContext =
        acc != null ? acc : AccessController.getContext();
    this.target = target;
    setPriority(priority);
    if (inheritThreadLocals && parent.inheritableThreadLocals != null)
        this.inheritableThreadLocals =
        ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);		// ——主要都是赋值父线程的
    /* Stash the specified stack size in case the VM cares */
    this.stackSize = stackSize;

    /* Set thread ID */
    tid = nextThreadID();			// 给新建的线程创建一个编号
}
```

——初始化线程完毕，在**堆内存**中等待着运行。

### 2.1.7 启动线程

当前线程（即parent线程）**同步告知Java虚拟机**，只要线程规划器空闲，应立即启动调用start()方法的线程。

### 2.1.8 中断（和OS不一样）

该中断和OS不一样，eg：场景：例如下载一个软件，期间不想下载了，选择取消，这个取消就是中断，那么该线程就需要终止。

概念：其他线程给该线程发一个信号，该线程收到信号后结束执行`run()`方法，使得该线程能立刻结束运行。即，提前结束线程

而信号通过**其他线程调用该线程的interrupt方法**，本质上就是设置中断标志位，而目标线程会去检测自身是否interrupted状态，如果是，就立刻结束运行。

注意：

1. interrupt方法只是向线程发送中断请求，但是线程何时响应看代码

2. 如果线程处于等待状态，然后又对其调用interrupt，会抛出**异常`InterruptedException`**

   所以，目标线程只要捕获到`join()`方法抛出的`InterruptedException`，就说明有其他线程对其调用了`interrupt()`方法，通常情况下该线程应该立刻结束运行。

相关方法：

- `thread.interrupt()`：其他线程希望指定的线程终止——就是这是对应的中断标志位

- `thread.isInterrupted()`：判断该线程是否被中断了

  而如果该线程已经处于terminated状态，那么不论是否发生中断，那么返回值都是false，就是非中断状态。

- `Thread.interrupt(xxx)`：对指定线程进行中断标志位复位

- 中断异常`InterruptedException`，当阻塞方法（同步阻塞、等待阻塞、其他阻塞）收到中断请求的时候就会抛出InterruptedException异常

  但是在抛出异常之前，Java虚拟机会先将该线程的中断标识位清除，然后抛出异常，那么再调用isInterrupted()方法将会返回false

  ——但是抛出该异常，说明有线程期待该线程被中断

### 2.1.9 挂起、恢复、停止

分别为suspend、resume、stop方法，非常“人性化”。但是这些API是过期的，不建议使用。

原因：

- suspend：将线程挂起，但是不释放线程所占有的资源，而是持有资源进行挂起，容易出现死锁的情况；
- stop：结束线程，但是不能保证线程占用的资源正常释放

——存在副作用，所以不建议使用，可以用wait/notify进行替换。

### 2.1.10 安全终止线程

即能够让程序正常走到最后，然后终止；也可以用中断安全终止。反正不能用stop终止。

## 2.2 线程间通信

多个线程能够相互配合完成工作，才能发挥最大的功能。

### 2.2.1 volatile 和 synchronized 关键字

**volatile能保证变量的可见性。**

对象：**针对共享变量**，但是每个变量都会对共享变量进行拷贝（目的是加速程序的执行，这是现代多核处理器的一个显著特性）。所以程序在执行过程中，一个线程**看到的变量并不一定是最新的**。

volatile可以用来修饰字段（成员变量），就是告知程序任何对该变量的访问均需要从共享内存中获取——保证每次查看结果都是最新的。但是，过多地使volatile是不必要的，因为它会降低程序执行的效率。

**synchronized保证互斥性和可见性**：线程在加锁时，先清空工作内存→在主内存中拷贝最新变量的副本到工作内存→执行完代码→将更改后的共享变量的值刷新到主内存中→释放互斥锁。

——synchronized的范围比volatile广

synchronized的基本原理：**使用了monitorenter和monitorexit指令**。同步方法则是依靠方法修饰符上的ACC_SYNCHRONIZED来完成的。本质就是尝试去获取monitor，而这个获取过程是排他的。

任意一个对象都拥有自己的监视器，如果需要同步调用，那么执行方法的**线程必须先获取到该对象的监视器**才能进入同步块或者同步方法。如果获取失败，那么将被阻塞在入口，进入阻塞状态，被挂到同步队列上，直到另外线程释放了该monitor后会将队列上的线程唤醒，然后重新尝试申请（？？？？是唤醒整个队列 or 队头？）

整个流程如下：

<img src="../pic/monitor.jpg" style="zoom: 67%;" >

### 2.2.2 wait/notify机制

synchronzied解决了多线程的竞争问题，但是无法解决多线程的协调问题，配合工作。

使用该机制的原因：线程之间有数据进行传递，例如生产者-消费者，生产者改变某个值之后，消费者才能继续执行下去。但是，如何感知到该值发生变化呢？——用while循环，每隔一段时间检测一遍。但是存在问题：

- 间隔时间过长：**难以确保及时性**，不能及时判断资源已经来了，导致不必要的等待
- 间隔时间过短：**难以降低开销**，不断进行上下文切换，导致资源浪费

——Java通过内置的等待/通知机制，**是任意Java对象都具备**，因为是在Object类中实现的方法：是native方法

| 方法名            | 描述                                                         |      |
| ----------------- | ------------------------------------------------------------ | ---- |
| `wait()`          | 调用该方法的线程会进入等待状态，只有被其他线程通知 或者 被中断 才会返回，注意wait之后会**释放占用的资源** |      |
| `wait(long)`      | 超时等待，参数是ms为单位的，如果在限定时间内等到了通知或者中断，那么会提前返回；否则超时后也会返回 |      |
| `wait(long, int)` | 超时等待的粒度更细，单位为ns                                 |      |
| `notify()`        | 通知**一个**在对象上等待的线程，让其从wait方法中返回，但是只有获取了锁才能返回，如果获取失败了那么无法返回<br />唤醒哪一个，看OS，结果不一样 |      |
| `notifyAll()`     | 通知****在对象上等待的线程                                   |      |

#### 使用方法：

线程A调用了**对象O的wait()方法**进入等待状态，而另一个线程B调用了对象O的notify()或者notifyAll()方法

```java
public class WaitNotify {
    static boolean flag = true;
    static Object lock = new Object();
    public static void main(String[] args) throws Exception {
        Thread waitThread = new Thread(new Wait(), "WaitThread");
        waitThread.start();			// 先启动wait线程
        TimeUnit.SECONDS.sleep(1);
        Thread notifyThread = new Thread(new Notify(), "NotifyThread");		
        notifyThread.start();		// 后启动notify线程
    }
    static class Wait implements Runnable {
        public void run() {
            synchronized (lock) {		// 加锁，拥有lock的Monitor——wait线程一定会先持有该lock对象
                // 当条件不满足时，继续wait，同时释放了lock的锁
                while (flag) {
                    try {
                        System.out.println(Thread.currentThread() + " flag is true. wait");
                        lock.wait();			// 释放lock资源，然后进行等待
                    } catch (InterruptedException e) { }
                }
                // flag=false，条件满足时，完成工作
                System.out.println(Thread.currentThread() + " flag is false. running");
            }
        }
    }
    
    static class Notify implements Runnable {
        public void run() {
            synchronized (lock) {		// 加锁，拥有lock的Monitor
                // 获取lock的锁，然后进行通知，通知时不会释放lock的锁，
                // 直到当前线程释放了lock后，WaitThread才能从wait方法中返回
                System.out.println(Thread.currentThread() + " hold lock. notify");
                lock.notifyAll();
                flag = false;
                SleepUtils.second(5);
            }
            // 再次加锁
            synchronized (lock) {
                System.out.println(Thread.currentThread() + " hold lock again. sleep");
                SleepUtils.second(5);
            }
        }
    }
}
```

输出：

```
Thread[WaitThread,5,main] flag is true. wait
Thread[NotifyThread,5,main] hold lock. notify
Thread[NotifyThread,5,main] hold lock again. sleep		// 和下面的执行顺序可能会变化，与wait和notify线程的抢占结果相关
Thread[WaitThread,5,main] flag is false. running
```

理解：整个流程：

1. 首先wait线程是先执行的，此时只有main线程和wait线程，所以wait能够得到lock，并且默认flag是true，所以能够执行while里面的循环；执行输出，然后执行**`lock.wait()`**，然后**wait线程变成等待状态WAITING，挂在lock的等待队列上，**它会释放占有的对象，lock会被释放

2. 然后notify线程执行，由于wait线程在等待前，将资源释放了，所以lock又能获取成功。执行输出；然后，notify线程唤醒所有挂在lock上的线程**`lock.notifyAll()`**，那么**会将挂在等待队列上的线程放到同步队列上，变成BLOCKED状态**，但是因为无法获得lock资源，只有notify线程释放资源才能从wait中返回。

3. 然后notify释放了资源，wait线程能从wait返回，那么出现了抢占（wait和notify线程均抢占资源lock），然后输出就不确定了

   <img src="../pic/wait_notify_cfg.jpg" style="zoom: 80%;" >

#### **要求：**

1. 在`synchronized`内部可以调用`wait()`使线程进入等待状态
2. 在`synchronized`内部可以调用`notify()`或`notifyAll()`唤醒其他等待线程
3. **必须在已获得的锁对象上调用`wait()`方法；**只有获得该对象的锁才能挂在该线程上，才能释放该对象（否则不存在这个概念）
4. 必须在已获得的锁对象上调用`notify()`或`notifyAll()`方法
5. **已唤醒的线程还需要重新获得锁后才能继续执行**

所以，消费者和生产者的wait/notify的逻辑：

```java
// 消费者
synchronized(对象){
    while(条件不满足){			//  即使被唤醒并调度回来，还会先进行判断
        对象.wait();
    }
    满足条件后的操作
}

// 生产者
synchronized(对象){
    改变条件
    对象.notifyAll();
}
```

### 2.2.3 管道输入和输出

管道输入/输出流，主要用于**线程之间的数据传输**，而传输的媒介为**内存**。

主要包括了如下4种具体实现：PipedOutputStream、PipedInputStream、PipedReader和PipedWriter（前两种面向字节，而后两种面向字符）

用法：

```java
public class Piped {
    public static void main(String[] args) throws Exception {
        PipedWriter out = new PipedWriter();			// 向管道写的对象
        PipedReader in = new PipedReader();				// 向管道读的对象
        // 将输出流和输入流进行连接，否则在使用时会抛出IOException
        out.connect(in);
        Thread printThread = new Thread(new Print(in), "PrintThread");
        printThread.start();
        int receive = 0;
        try {
            while ((receive = System.in.read()) != -1) {
                out.write(receive);
            }
        } finally {
            out.close();
        }
    }
    static class Print implements Runnable {
        private PipedReader in;
        public Print(PipedReader in) {
            this.in = in;
        }
        public void run() {
            int receive = 0;
            try {
                while ((receive = in.read()) != -1) {
                    System.out.print((char) receive);
                }
            } catch (IOException ex) {
            }
        }
    }
}
```

——在使用前必须先绑定`out.connect(in);`

### 2.2.4 thread.join()：默认带锁

线程A执行了`threadB.join()`语句，其含义是：当前线程A等待**threadB线程终止之后**才从threadB.join()返回

此外，还提供了`join(long millis)`和`join(long millis,int nanos)`两个具备超时特性的方法.

```java
public final synchronized void join(long millis) throws InterruptedException {	// 本质上就是调用带有超时等待的join方法
    long base = System.currentTimeMillis();
    long now = 0;

    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (millis == 0) {			// 说明无限等待
        while (isAlive()) {		// 如果该线程存活，那么一直挂起
            wait(0);		// 0表示无限wait
        }
    } else {			// 表示超时等待，那么只有更新倒计时时间
        while (isAlive()) {	
            long delay = millis - now;		
            if (delay <= 0) {
                break;
            }
            wait(delay);			// 否则就等待一段时间
            now = System.currentTimeMillis() - base;
        }
    }
}
```

理解：

1. 如果某个线程调用`threadB.join()`之后，开始等待线程B，如果期间按其他线程调用该线程的interrupt方法，那么会抛出`InterruptException`异常——因为处于wait中
2. 这边的wait方法，本质上就是**`this.wait()`**，对象就是线程对象，而join方法本身就是synchronized的，所以能在里面执行，它必定已经获取到了`this`锁，所以wait之后，会释放this锁
3. join本质上，还是wait/notify实现的，当线程终止时，会调用线程自身的notifyAll()方法，会通知所有等待在该线程对象上的线程

### 2.2.5 ThreadLocal

#### 原理：

线程本地变量，是一个以**ThreadLocal对象为键、任意对象为值**的存储结构。就是将一个值和线程本身进行绑定。

概念：**每个线程都有同一个变量的独有拷贝**

实现上来说：**每个线程都有一个Map，类型为ThreadLocalMap**，所以调用下面的`set()`方法，实际上是在线程自己的Map里设置了一个条目，**键为当前的ThreadLocal对象，值为value**。

set()方法：

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);		// 获得线程的map
    if (map != null)
        map.set(this, value);		// 将值设置好
    else
        createMap(t, value);		// 如果map还未创建，就创建一个
}
```

get()方法：

```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);		// 根据key获得value
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();			// 如果获取值失败，那么就去调用initialValue方法
}
```

总结：每个线程都有一个Map，对于每个ThreadLocal对象，调用其get/set实际上就是**以ThreadLocal对象为键读写当前线程的Map**

#### 使用：

有两个方法：

```java
public T get() {}			// 获取变量值
public void set(T value) {}		// 设置变量值
```

使用：

```java
ThreadLocal<Integer> local = new ThreadLocal<>();
local.set(200);
System.out.println(local.get());
```

主要是在多线程中，它们对同一个变量设置值，但是调用的时候值都是不一样的——**访问的虽然是同一个变量local，但每个线程都有自己的独立的值**。

还有两个常用的方法：

```java
protected T initialValue() {}		// 这个方法经常用来重写，那么
```

当调用get方法时，如果之前没有设置过，会调用该方法获取初始值，默认实现是返回null；如果set设置之后，又被remove删除掉了，再次调用get，会再调用initialValue获取初始值

```java
public void remove() {}				// 删除该值：
```

eg1：

```java
public class Profiler {
    // 第一次get()方法调用时会进行初始化（如果set方法没有调用），每个线程会调用一次
    private static final ThreadLocal<Long> TIME_THREADLOCAL = new ThreadLocal<Long>() {
        protected Long initialValue() {			// 初始值方法
            return System.currentTimeMillis();
        }
    };
    public static final void begin() {
        TIME_THREADLOCAL.set(System.currentTimeMillis());		// 设置初始值
    }
    public static final long end() {
        return System.currentTimeMillis() - TIME_THREADLOCAL.get();
    }
    public static void main(String[] args) throws Exception {
        Profiler.begin();			// 设置初始值
        TimeUnit.SECONDS.sleep(1);
        System.out.println("Cost: " + Profiler.end() + " mills");
    }
}
```

理解：主要用来计算经过的时间。即使中间其他线程调用过该ThreadLocal，该线程对应的值是不会变的

eg2：

提供上下文信息，eg：一个线程执行用户的请求，在执行过程中，很多代码都会访问一些共同的信息，比如请求信息、用户身份信息、数据库连接、当前事务等，如果需要将其作为参数进行传递就很麻烦，但是可以用ThreadLocal保存

```java
public class RequestContext {
    public static class Request { //...
    };
    private static ThreadLocal<String> localUserId = new ThreadLocal<>();
    private static ThreadLocal<Request> localRequest = new ThreadLocal<>();
    public static String getCurrentUserId() {
        return localUserId.get();
    }
    public static void setCurrentUserId(String userId) {
        localUserId.set(userId);
    }
    public static Request getCurrentRequest() {
        return localRequest.get();
    }
    public static void setCurrentRequest(Request request) {
        localRequest.set(request);
    }
}
```

## 2.3 线程池技术

why需要线程池：如果采用一个任务一个线程的方式，将会创建数以万记的线程，会使操作系统频繁的进行线程上下文切换、而线程创建和销毁都是需要资源的。

概念：线程池会预先创建若干个线程，且不能由用户直接对线程的创建进行控制。然后就能用这些固定的线程来完成无数的任务。

线程池的基本实现：P112/P212

工作线程获得`jobs`（任务列表）对象的锁，取出队首的任务开始执行；当jobs为空时，工作者线程`jobs.wait()`进入等待；

添加一个job到jobs后（jobs是共享变量，所以需要互斥的去获取，所以需要获得jobs对象的锁），然后`job.notify()`唤醒一个工作线程（只唤醒一个，不用唤醒全部，减少开销），当然只有在该释放jobs锁之后，工作线程才能变成执行态。

常用的Java Web服务器，如Tomcat、Jetty，在其处理请求的过程中都使用到了线程池技术。

# 3. 锁

绍Java并发包中与锁相关的API和组件

这些API和组件的使用方式和实现细节

注意**锁和synchronized是不同的**

- 功能上，都是用来实现同步的

- 但是使用上，**lock需要显式的获取和释放**；拥有了锁获取与释放的可操作性；可中断的获取锁以及超时获取锁等synchronized无法实现的功能

  ——lock是显式调用的，但是功能多样，扩展性好

在Java SE5之前，一直都是用synchronized来实现锁功能的，即实现互斥性。但是Java SE5之后，并发包中新增了Lock接口（以及相关实现类）用来实现锁功能。                          

## 3.1 Lock接口

lock更为灵活多样。

使用：

```java
Lock lock = new ReentrantLock();		// 创建锁对象
lock.lock();		// 上锁，然后下面一直到释放锁的部分都是互斥的
try{
    ....
}
finally{
    lock.unlock();
}
```

理解：

1. finally中释放锁，能够保证获取锁之后一定能够去释放，不论try中出现任何问题。
2. try中不能去获取锁，因为如果在获取锁（自定义锁的实现）时发生了异常，异常抛出的同时，也会导致锁无故释放。

Lock中提供了synchronized没有的功能：

| 特性               | 描述                                                       |
| ------------------ | ---------------------------------------------------------- |
| 尝试非阻塞的获取锁 | 如果尝试获取锁，如果成功获得了就持有锁；如果失败了也不阻塞 |
| 被中断地获取锁     | 获取到锁的线程能够响应中断，会抛出中断异常同时会释放锁     |
| 超时获取锁         | 在指定的截止时间获取锁，获取不到就直接返回了               |

## 3.2 







# ps：一些混淆的知识点

## 1. synchronized不可中断 & lock可被中断

不可中断的含义：**等待获取锁的时候不可被中断**，即没有获取到锁之前是不会响应中断的，**拿到锁之后可以被中断**。

本质上：`thread.interrupt()`，就只是给中断标志位赋值

所以存在如下情况：

- 该线程处于非阻塞状态

  只是将该线程的中断状态将被设为true，没有其他操作，需要线程代码中自行判断

- 该线程处于阻塞状态：调用了`wait`,`sleep`,`join`方法

  然后会从阻塞状态退出，然后抛出中断异常，并且将中断标志位恢复。

而ReentrantLock可被中断，即在等待锁过程中也可以被中断，但是必须是通过`ReentrantLock.lockInterruptibly()`来获取的锁（普通的lock是无效的）

```java
private void doAcquireInterruptibly(int arg)
    throws InterruptedException {
    // 把线程放进等待队列
    final Node node = addWaiter(Node.EXCLUSIVE); 
    boolean failed = true;
    try {
        // 自旋
        for (;;) {
            // 获取前置节点
            final Node p = node.predecessor();
            // 前置节点为头节点 && 当前节点获取到锁
            if (p == head && tryAcquire(arg)) {
                // 当前节点设为头节点
                setHead(node);
                p.next = null;  // 应用置null,便于GC
                failed = false;
                // 结束自旋
                return;
            }
            // 检查是否阻塞线程 && 检查中断状态
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                // 显式抛中断异常
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}     
```

——主要就是Interruptibly是在自旋等待时候还是会去判断是否设置中断标志位了，如果是，那么抛出异常

而如果获取到锁了，就跟synchronized一样由线程代码决定是否中断。