# ReentrantLock源码阅读

实现是在JUC：java.util.concurrent并发包里面

ReentrantLock是

- 显式锁
- 互斥锁（独占锁）
- 可配置是否公平锁
- 可重入锁

实际上底层是另一个类实现的AQS

## 静态变量：

这些都是`AbstractQueuedSynchronizer`类中的实现：通过继承得到

```java
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long stateOffset;			// 记录偏移量
private static final long headOffset;
private static final long tailOffset;
private static final long waitStatusOffset;
private static final long nextOffset;
```

```java
static {			// 在类加载的时候就已经获得 state、head、tail、waitStatus、next 在ReentrantLock类的实例中的偏移量
    try {
        stateOffset = unsafe.objectFieldOffset
            (AbstractQueuedSynchronizer.class.getDeclaredField("state"));	
        headOffset = unsafe.objectFieldOffset
            (AbstractQueuedSynchronizer.class.getDeclaredField("head"));
        tailOffset = unsafe.objectFieldOffset
            (AbstractQueuedSynchronizer.class.getDeclaredField("tail"));
        waitStatusOffset = unsafe.objectFieldOffset
            (Node.class.getDeclaredField("waitStatus"));
        nextOffset = unsafe.objectFieldOffset
            (Node.class.getDeclaredField("next"));

    } catch (Exception ex) { throw new Error(ex); }
}
```



## 实例变量：

```java

```



```java

```



## 静态方法：



## 实例方法：

**尝试获取锁**：（非公平状态下）

——尝试获取锁，是看是否能获取锁，能获取就获取，获取不到不发生阻塞，返回失败

```java
public boolean tryLock() {
    return sync.nonfairTryAcquire(1);
}
```

```java
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();		// 获取当前线程
    int c = getState();			// 获取对应的状态，记录是否有线程持有该锁，并且记录进入的次数
    if (c == 0) {			// 表明没有线程持有该锁
        if (compareAndSetState(0, acquires)) {		// 那么尝试去修改共享变量state——用CAS
            setExclusiveOwnerThread(current);	// 记录当前锁的持有者——因为是独占锁，所以最多只有一个持有者	
            return true;
        }
    }		// 如果该锁已经被占有了
    else if (current == getExclusiveOwnerThread()) {	// 如果该锁占有的线程就是当前线程，那么可重入，只需要增加state的次数
        int nextc = c + acquires;
        if (nextc < 0) // <0，表示进入次数已经超过2^32-1，发生越界，则抛出异常
            throw new Error("Maximum lock count exceeded");
        setState(nextc);			// 更新最新的状态
        return true;
    }
    return false;		// 如果该锁已经被占用，且占有的线程不是当前线程，那么无法获得锁，返回失败
}
```

```java
// AbstractQueuedSynchronizer类中实现：
protected final boolean compareAndSetState(int expect, int update) {
    // See below for intrinsics setup to support this
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);	// 就是调用底层CAS的实现
}

// AbstractOwnableSynchronizer类中实现：
private transient Thread exclusiveOwnerThread;
protected final void setExclusiveOwnerThread(Thread thread) {
    exclusiveOwnerThread = thread;			// 更新独占锁的持有者
}

// AbstractOwnableSynchronizer类中实现：
protected final Thread getExclusiveOwnerThread() {
    return exclusiveOwnerThread;
}

// AbstractQueuedSynchronizer类中实现：
protected final void setState(int newState) {
    state = newState;
}
```

**尝试释放锁**：（非公平下）



**（普通的）获取锁**：（非公平下）

```java
final void lock() {
    if (compareAndSetState(0, 1))			// 如果CAS去修改锁的状态值，从0->1，如果成功了，那么就能得到该锁
        setExclusiveOwnerThread(Thread.currentThread());		// 获得该值之后，将当前线程设置为锁的持有者
    else			// 如果CAS去修改该状态值，返回false，那么无法得到该值
        acquire(1);
}
```

acquire方法是FairSync和UnfairSync的父类AQS中的核心方法。

```java
// AbstractQueuedSynchronizer类中实现：
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

抽象方法——具体实现分为NonfairSync/fairSync

```java
abstract static class Sync extends AbstractQueuedSynchronizer {
    private static final long serialVersionUID = -5179523762034025860L;

    /**
         * Performs {@link Lock#lock}. The main reason for subclassing
         * is to allow fast path for nonfair version.
         */
    abstract void lock();

    /**
         * Performs non-fair tryLock.  tryAcquire is implemented in
         * subclasses, but both need nonfair try for trylock method.
         */
    final boolean nonfairTryAcquire(int acquires) {				// 非公平锁的尝试获取，参数=1
        final Thread current = Thread.currentThread();		// 获得当前线程（就是尝试获取锁的线程）
        int c = getState();		// 获得锁的状态
        if (c == 0) {			// 没有线程占用，就CAS去设置（同lock中一样）
            if (compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }			// 有线程占用，看锁的独占线程是否就是当前线程（表示可重入）
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;			// c+1
            if (nextc < 0) // overflow			// 重入次数太多，溢出
                throw new Error("Maximum lock count exceeded");
            setState(nextc);			// 更新次数
            return true;
        }
        return false;			// 如果锁未被占用但尝试CAS获取锁失败 or 锁占用且占用的不是当前线程没法重入，那么返回获取锁失败 
    }

    protected final boolean tryRelease(int releases) {
        int c = getState() - releases;
        if (Thread.currentThread() != getExclusiveOwnerThread())
            throw new IllegalMonitorStateException();
        boolean free = false;
        if (c == 0) {
            free = true;
            setExclusiveOwnerThread(null);
        }
        setState(c);
        return free;
    }

    protected final boolean isHeldExclusively() {
        // While we must in general read state before owner,
        // we don't need to do so to check if current thread is owner
        return getExclusiveOwnerThread() == Thread.currentThread();
    }

    final ConditionObject newCondition() {
        return new ConditionObject();
    }

    // Methods relayed from outer class

    final Thread getOwner() {
        return getState() == 0 ? null : getExclusiveOwnerThread();
    }

    final int getHoldCount() {
        return isHeldExclusively() ? getState() : 0;
    }

    final boolean isLocked() {
        return getState() != 0;
    }

    /**
         * Reconstitutes the instance from a stream (that is, deserializes it).
         */
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        s.defaultReadObject();
        setState(0); // reset to unlocked state
    }
}
```

### 公平锁的实现：

主要流程：

1. 首先去判断锁的状态，如果锁==0，**且等待队列为空 or 我是第一个等待的线程**，那么尝试用CAS去获取锁
   1. 尝试去获取锁，获取锁成功，设置独占线程，返回成功；
   2. 尝试获取失败，说明存在并发情况，返回失败；
2. 如果锁 != 0，那么看当前占有锁的线程是否是自己
   1. 如果是自己，那么增加重入次数；
   2. 如果不是自己，返回失败；

##### 第一层：FairSync-tryAcquire

```java
static final class FairSync extends Sync {
    private static final long serialVersionUID = -3000897897090466540L;

    final void lock() {
        acquire(1);				// 调用acquire
    }
	
    // 公平锁的调用：锁可用下，如果队列为空，那么能够直接获得锁；队列不为空，只有等待队列中的第一个线程才能获得锁，其余均按照来到的时间插到队尾
    protected final boolean tryAcquire(int acquires) {		// acquire(1)实际会调用这个函数——AQS的核心
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {				// 如果当前线程要获得锁，且锁可用
            if (!hasQueuedPredecessors() &&			// 返回false，表示可以去获取锁了
                compareAndSetState(0, acquires)) {		// 尝试去获取锁，且成功了，那么更新独占线程内容，返回成功
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {		// 表现可重入性
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;		// 锁空闲，但是当前线程不是第一个等待的线程；尝试CAS失败了（存在并发操作）；锁非空闲，且占有锁的不是自己人
    }
}
```

##### 第二层：hasQueuedPredecessors

判断等待队列中是否存在等待线程：

false：表示不存在等待线程/第一个等待线程就是自己——表明能去获取锁了

true：表示存在等待线程，且该线程不是自己——表明自己没资格，要插入队尾等待

```java
public final boolean hasQueuedPredecessors() {

    Node t = tail; // Read fields in reverse initialization order
    Node h = head;
    Node s;
    return h != t &&			
        ((s = h.next) == null || s.thread != Thread.currentThread());
}
```

理解：

1. 队列为空，即`h == t`，返回false

2. 队列不为空，h.next表示真实的第一个等待结点，如果它为null，那么说明有线程要进行等待，只不过刚在执行队列的初始化操作（出现并发操作的问题）

   因为设置head结点是CAS操作，所以不会出现错误。

   而队列初始化，只发生在第一个线程要插入的时候，先进行队列初始化，后插入，所以h.next==null，就一定有线程要等待了，队列中一定有线程了，返回true

3. 队列不为空，且h.next正确指向第一个线程的结点，看该线程是否就是当前线程

   1. 如果是，说明该锁可以获得了，返回false
   2. 如果不是，申请锁失败，需要等待了

### 非公平锁的实现：

主要流程：

1. 尝试用CAS去获取锁（这边就体现了非公平性，不排队直接尝试获取，获取失败再说）
2. 如果获取成功，那么就将当前线程设置为该锁的独占线程——表示该锁已经有线程占用了
3. 如果获取失败，用acquire去继续操作（不会去放弃申请锁，这样并发性太差了；而是进行等待，保留获取锁的可能，还在获取锁的过程中）——下面就是AQS框架的处理流程

##### 第一层：NonfairSync-tryAcquire

```java
static final class NonfairSync extends Sync {
    private static final long serialVersionUID = 7316153563782823691L;

    /**
         * Performs lock.  Try immediate barge, backing up to normal
         * acquire on failure.
         */
    final void lock() {
        if (compareAndSetState(0, 1))	// 用CAS尝试获得锁，如果成功，那么设置当前线程为独占的线程
            setExclusiveOwnerThread(Thread.currentThread());
        else				// 获得锁失败，调用acquire方法继续
            acquire(1);	
    }

    protected final boolean tryAcquire(int acquires) {		// acquire(1)实际会调用这个函数——AQS的核心
        return nonfairTryAcquire(acquires);			// 返回获取锁成功 or 失败
    }
}
```

总结：实际上FairSync和NonfairSync底层都是同一套AQS，具体体现在lock中要调用的`acquire()`方法（如下）

下面是获取锁的整个流程：

<img src="../pic/lock_cfg.png"  >

1. 尝试CAS获取锁——就是`lock()`的前面一部分。获取成功，就设置独占线程
2. 如果获取失败，就会去调用`acquire(1)`的方法，转而去调用`tryAcquire(1)`方法，本质就是`nonfairTryAcquire(1)`方法（在sync的抽象类中有默认实现）

## 线程加入等待队列⭐

### 加入的时机：

当执行Acquire(1)时，会通过tryAcquire获取锁。在这种情况下，如果获取锁失败，就会调用addWaiter加入到等待队列中去。

##### 最上层：acquire：

执行acquire的契机：

1. 公平锁，调用lock，本质上就是调用`acquire(1)`
2. 非公平锁，调用lock，如果CAS尝试获取锁失败，那么会去调用`acquire(1)`

```java
public final void acquire(int arg) {		// 传递的参数为1
    if (!tryAcquire(arg) &&				// 得到的是获取锁成功 or 失败的表示，false表示获取失败
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))			// 该方法返回的是interrupt的状态，如果true，表示要中断
        selfInterrupt();			// 那么将当前节点中断
}
```

理解：

1. `tryAcquire(1)`根据多态，实际上会根据锁的类型去调用对应的具体的`tryAcquire(1)`方法，如果获取成功，那么占有了锁，没有下面的麻烦事了

2. 如果`tryAcquire()`返回false，表示获取失败，那么调用`addWaiter(Node.EXCLUSIVE)`**加入到等待队列中**

3. 加入到等待队列之后，会调用`acquireQueued(node, arg);`，在队列中尝试获得锁，如果失败多次，就被挂起，最后被唤醒，然后再次尝试获得锁

   最后acquireQueued返回，那肯定是带着锁的，然后返回的是interrupt的状态——true，表示在此期间，该线程是被设置为中断使能的；false就没有，程序结束

4. 进入if，然后执行自我中断——其实还是设置中断标志位，本质上就是前面已经设置过中断标志位了，但是这边是获得锁之后重复设置一次

```java
static void selfInterrupt() {
    Thread.currentThread().interrupt();
}

public void interrupt() {
    if (this != Thread.currentThread())
        checkAccess();
    synchronized (blockerLock) {
        Interruptible b = blocker;
        if (b != null) {
            interrupt0();           // Just to set the interrupt flag
            b.interrupt(this);
            return;
        }
    }
    interrupt0();
}
```

why在获取锁的情况下还要去执行线程中断？

1. 当中断线程被唤醒时，并不知道被唤醒的原因，可能是当前线程在等待中被中断，也可能是释放了锁以后被唤醒。因此我们通过Thread.interrupted()方法检查中断标记（该方法返回了当前线程的中断状态，并将当前线程的中断标识设置为False），并记录下来，如果发现该线程被中断过，就再中断一次。
2. 线程在等待资源的过程中被唤醒，唤醒后还是会不断地去尝试获取锁，直到抢到锁为止。也就是说，**在整个流程中，并不响应中断，只是记录中断记录**。最后抢到锁返回了，那么**如果被中断过的话，就需要补充一次中断。**

##### 第二层：addWaiter：

加入等待队列的代码

```java
// AbstractQueuedSynchronizer.java中实现
private Node addWaiter(Node mode) {		// 传递的是EXCLUSIVE，表示独占形式获取锁
    Node node = new Node(Thread.currentThread(), mode);		// 创建一个新的结点
    // Try the fast path of enq; backup to full enq on failure
    Node pred = tail;			
    if (pred != null) {				// pred就是当前node结点的前驱，就是之前的等待队列的尾，如果不存在，说明等待队列为空
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {		// 尝试修改tail的位置，指向最新的结点——如果更新失败，说明出现了并发的情况
            pred.next = node;
            return node;
        }
    }
    enq(node);
    return node;
}
```

就是将获取锁失败的线程加入到等待队列：

1. 需要包装成Node结点
2. 将结点插入到队列尾部（所以在队列中按照的是FIFO执行的）
   1. 如果pred为null，说明此时**等待队列为空**，那么插入的操作不同——enq
   2. 如果pred不为null，但是尝试更新tail结点的时候失败了，即**pred和tail指向的位置不同，说明别的线程在并发修改**，那么插入操作不同——enq
   3. 如果pred不为null，且尝试更新tail结点成功了，普通的双向链表的插入操作，之后直接返回最新的结点

针对上面的2.1、2.2问题，还需要调用特别的方法

##### 第三层：enq：

一个自旋，直到更新tail结点成功

```java
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) { // ——说明之前等待队列为空
            if (compareAndSetHead(new Node()))		// 设置头结点（为一个dummy结点）
                tail = head;			// tail初始时就是指向head节点的——然后下一次循环，就能执行下面的else了
        } else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {			// 再次尝试更新tail
                t.next = node;
                return t;
            }
        }
    }
}
```

可以发现：设置head结点是CAS，但是tail结点的设置不是原子性的，所以存在一段时间`head != tail`，即if里面的语句没有来得及执行。而下一次循环去插入第一个节点也是非原子操作，所以存在一段时间head的next为null，所以存在上面的FairSync的`hasQueuedPredecessors`的判断

实际内部的等待队列结构：

<img src="../pic/waitQueue.png">

可以发现，head结点是空结点，只是为了统一操作。

##### 第二层：acquireQueued

```java
acquireQueued(addWaiter(Node.EXCLUSIVE), arg)		// 将线程加入到队尾之后
```

`acquireQueued`：**对在队列中的线程尝试获取锁操作**，直到获取锁成功 or 不再需要获取锁，是一个自旋的状态：

存在如下情况：

1. 如果当前node是第一个等待节点，那么去尝试获取锁——不断嗅探，看是否能够获取到锁
   1. 如果获取锁成功，那么更新head节点，就是将当前node节点的内容清掉，head指向当前node（还是一个dummy节点）
   2. 如果获取锁失败了（非公平锁抢占了，）
2. 如果当前node是头结点但是获取锁失败了 or 当前node不是头节点，那么看我是否需要自行阻塞

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;		// 标记是否成功拿到资源
    try {
        boolean interrupted = false;		// 标记是否被中断
        for (;;) {
            final Node p = node.predecessor();		
            if (p == head && tryAcquire(arg)) {		// 表示node是第一个节点，那么尝试去获取锁
                setHead(node);				// 获取成功了，将node内容清掉，head向后移动，把原来的head节点给清掉
                p.next = null; // help GC
                failed = false;				// 标记为成功
                return interrupted;			// 返回是否中断标记——如果true，表示要进行中断；如果false表示不中断
            }
            // 说明p为头节点且当前没有获取到锁（可能是非公平锁被抢占了）或者是p不为头结点，这个时候就要判断当前node是否要被阻塞（被阻塞条件：前驱节点的waitStatus为-1），防止无限循环浪费资源
            if (shouldParkAfterFailedAcquire(p, node) &&			// 前驱节点设置为signal
                parkAndCheckInterrupt())		// 返回的是interrupt的状态
                interrupted = true;			// 进入这个，表示interrupt为true，表示要将其中断了
        }
    } finally {			// tryAcquire出现异常，循环结束，并且failed=true，那么下面的方法会被执行
        if (failed)		
            cancelAcquire(node);
    }
}
```

注意：`setHead`没有清waitStatus状态，主要是还有用，所以head节点是有状态的，并且它参与到下面的状态设置中的

##### 第三层：shouldParkAfterFailedAcquire

根据前驱节点去判断是否要将当前节点阻塞

返回值：true：表示要挂起自己，等待被唤醒；false：不要挂起，还能抢救一下

存在如下情况：

1. 前驱节点是：SIGNAL状态，说明当前节点可以挂起，之后唤醒，所以返回true
2. 如果前驱是：CANCELED状态，说明都是要被取消的，所以找到第一个非取消的节点连接，为pre节点，返回false
3. 如果前驱是：其他状态，将前驱状态设置为SIGNAL，返回false

结合上面的自旋来看：如果当前线程在等待队列中尝试获取锁，但是获取锁失败了，于是我需要看前驱节点：如果前驱节点是取消状态，那么我要找到非取消状态的前驱节点，并且本轮不挂起；第二次循环尝试，还是获取锁失败了，如果我的前驱节点是非signal状态，那么我将其设置为signal状态，然后本轮不挂起，再次去尝试；第三次循环尝试，还是获取失败了，那么此时就是signal状态了，我需要将自己挂起——所以，经过多轮尝试，我依旧得不到锁，所以认为开销太大，所以挂起了

```Java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;		// 获得前驱节点的wait状态
    if (ws == Node.SIGNAL)			// 表示前驱节点预计马上就能得到资源了，那么当前节点可以挂起，等待被唤醒
        return true;
    if (ws > 0) {			// 表示前驱接节点马上就要被取消了
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);	// 向前开始遍历，将所有canceled节点删除，结束之后node节点之前的是非canceled的节点
        pred.next = node;
    } else {
        // 当前驱节点的状态不是canceled时，设置前任节点等待状态为SIGNAL，但是本次循环是不阻塞，再次尝试获取，如果还是失败了，那么就要阻塞了
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```

##### 第三层：parkAndCheckInterrupt

```java
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);				// 将当前线程阻塞掉，是挂到当前的对象的阻塞队列上
    return Thread.interrupted();		// 返回当前线程的阻塞状态
}

public static void park(Object blocker) {
    Thread t = Thread.currentThread();
    setBlocker(t, blocker);		
    UNSAFE.park(false, 0L);		
    setBlocker(t, null);
}
```

——此时，**线程会进入waiting状态**（不是blocking状态），那么不会得到调度，线程阻塞在当前位置——注意，它还是在CLH队列中的（之前，线程虽然在队列中，但是**它还是running状态**）

当当前节点unlock释放锁，会唤醒队首的存在waiting的节点，而该节点的线程得到cpu后，会回到这边。最后返回当前线程（该节点线程）的状态

配合第二层的acquireQueued看，

```java
try {
        boolean interrupted = false;		// 标记是否被中断
        for (;;) {
            final Node p = node.predecessor();		
            if (p == head && tryAcquire(arg)) {		// 表示node是第一个节点，那么尝试去获取锁
                setHead(node);				// 获取成功了，将node内容清掉，head向后移动，把原来的head节点给清掉
                p.next = null; // help GC
                failed = false;				// 标记为成功
                return interrupted;			// 返回是否中断标记——如果true，表示要进行中断；如果false表示不中断
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())		// 从这边返回
                interrupted = true;			// 因为返回true，需要更新interrupt为true；返回false，那么不更新；都会再次去执行循环
        }
    }
```



##### 第三层：cancelAcquire

——在这边，这个只是用来处理异常的特殊情况（正常情况下并不会被执行）

设置cancel，更多出现在超时等待中。——超时等待，才有可能会

该方法的主要作用：获取当前节点的前驱节点，如果前驱节点的状态是CANCELLED，那就一直往前遍历，找到第一个waitStatus <= 0的节点，将找到的Pred节点和当前Node关联，将当前Node设置为CANCELLED；并且将prev的next指向node的next

存在3种情况：

1. 当前节点是尾节点：第一个非canceled节点就是新的tail（next连线清掉，prev还是连着的）
2. 当前节点是头节点：那么唤醒node的后继节点——这边的后继是指第一个非虚、非cancelled 节点
3. 当前节点是中间节点：将pred的next指向node.next

```java
private void cancelAcquire(Node node) {
    // 无效节点，直接返回
    if (node == null)
        return;

    node.thread = null;		// 需要清空node里面的内容——该节点是虚节点

    // 跳过那些已经canceled的节点
    Node pred = node.prev;
    while (pred.waitStatus > 0)
        node.prev = pred = pred.prev;

    // pred是最近的非canceled的节点，然后predNext就是其后继
    Node predNext = pred.next;

    // 将当前节点的状态设置为CANCELLED
    node.waitStatus = Node.CANCELLED;

    // 如果当前节点是尾节点，那么将pred设置为尾节点——第一个非canceled节点为尾节点，并且将其的next设置为null
    if (node == tail && compareAndSetTail(node, pred)) {
        compareAndSetNext(pred, predNext, null);
    } else {
        // 如果当前节点不是尾节点 or 设置为节点失败了
        int ws;
        // 存在如下情况：
        // 1. 当前节点是第一个节点（head.next），那么执行else
       // 2. 当前节点是中间节点，那么看其前驱节点是否是signal状态；如果是进入if；如果不是，尝试将prev状态修改为signal，如果修改失败，进入else；如果修改成功，进入if，在进入if之前还需要确认pred是否是虚节点——并发操作有关
        if (pred != head &&
            ((ws = pred.waitStatus) == Node.SIGNAL ||
             (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
            pred.thread != null) {		// if里面的操作：将当前节点的前驱节点的next指向当前节点的后驱节点
            Node next = node.next;
            if (next != null && next.waitStatus <= 0)
                compareAndSetNext(pred, predNext, next);
        } else {
            unparkSuccessor(node);	// else里面的操作：唤醒当前节点的后继节点
        }

        node.next = node; // help GC
    }
}
```

可以发现，cancelAcquire是对next指针进行操作，但是并没有操作pred指针。why？

在这个之前，当前节点的前继节点有可能已经执行完并移除队列了，此时prev是不确定的，例如把prev指向一个已经移除队列的node。所以修改prev指针不大安全。

而对pred操作在这边的代码：

而在shouldParkAfterFailedAcquire执行，执行到这边，说明已经有其他线程获取锁了，所以状态确定，可以用来更新prev

```java
do {
    node.prev = pred = pred.prev;
} while (pred.waitStatus > 0);
```

##### 第三层：unparkSuccessor

```java
private void unparkSuccessor(Node node) {
    int ws = node.waitStatus;
    if (ws < 0)				// 根据上面的传参，是不会进入的
        compareAndSetWaitStatus(node, ws, 0);
    Node s = node.next;
    if (s == null || s.waitStatus > 0) {			// 找到第一个后继节点
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);
}
```

why需要从后向前找：

节点入队不是原子性操作，更新tail节点是CAS，但是更新之后就有可能去执行unpark，所以可能存在pred.next还没来得及更新。所以这样不能从前向后找，队列不完整，所以需要从前向后，看pred找。

并且在cancelled都是先断开next，并没有截断pred，所以需要pred才能遍历完所有的节点。

```java
if (pred != null) {
    node.prev = pred;
    if (compareAndSetTail(pred, node)) {
        pred.next = node;			// 这句执行前被打断了
        return node;
    }
}
```

## 解锁：存在唤醒

ReentrantLock在解锁的时候，并不区分公平锁和非公平锁。

##### 第一层：unlock/release

```java
// ReentrantLock.java实现
public void unlock() {
    sync.release(1);
}

// AQS.java实现
public final boolean release(int arg) {			// 传的参数：1
    if (tryRelease(arg)) {				// 说明能彻底释放该锁
        Node h = head;
        if (h != null && h.waitStatus != 0)	// h!=null，表示有等待的线程；h.waitStatus == 0，那么后继线程还是处于运行中，不需要唤醒；!=0, 那么唤醒第一个等待的线程——head的后继
            unparkSuccessor(h);
        return true;
    }
    return false;			// 存在重入的情况，还不能彻底释放（当前线程还占有锁）
}
```

可以发现：用的是框架，里面会回调子类实现的方法

##### 第二层：tryRelease

```java
// ReentrantLock里面的sync实现的：对于FairSyn/NonfairSyn都是一样的
protected final boolean tryRelease(int releases) {			// 默认给的是1
    int c = getState() - releases;			// 记录更新的state的值（主要存在重入的情况）
    if (Thread.currentThread() != getExclusiveOwnerThread())			// 当前线程不是持有锁的线程，不存在解锁的情况，抛出异常
        throw new IllegalMonitorStateException();
    boolean free = false;			// 标志是否能彻底释放该锁
    if (c == 0) {			// 说明能彻底释放该锁
        free = true;
        setExclusiveOwnerThread(null);		// 清空独占线程
    }
    setState(c);
    return free;
}
```



## AQS基本数据结构

队列中结点的数据结构定义：

创建了一个静态内部Node类：

```java
static final class Node {
    static final Node SHARED = new Node();		// Node.SHARED表示共享模式，那么给一个空的结点表示共享
    static final Node EXCLUSIVE = null;			// Node.EXCLUSIVE表示独占模式，就是一个空对象
    
    // waitStatus的枚举值
    static final int CANCELLED =  1;
    static final int SIGNAL    = -1;
    static final int CONDITION = -2;
    static final int PROPAGATE = -3;
    
    volatile int waitStatus;		// 值就是对应上面4个状态 + 1个默认状态
    volatile Node prev;			// 前一个结点
    volatile Node next;		
    volatile Thread thread;			// 存储的就是等待的线程——这个才是结点的内心
    Node nextWaiter;
	
    // 构造方法
    Node() {    // 主要用来初始化队列和给shared标志位的
    }

    Node(Thread thread, Node mode) {     // 用在增加等待结点
        this.nextWaiter = mode;
        this.thread = thread;
    }

    Node(Thread thread, int waitStatus) { // Used by Condition
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
    // 方法：
    final boolean isShared() {				// 判断结点的mode
        return nextWaiter == SHARED;
    }
    final Node predecessor() throws NullPointerException {		// 获取前一个结点，如果prev不存在抛出异常——说明即使是第一个结点也一定有prev（说明存在dummy结点）
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }
}
```

AQS的静态变量 or 实例变量：

都用了volatile修饰，保证内存可见性，并发修改用CAS，但是这样无法保证修改的可见性，所以用volatile修饰。

```java
private transient volatile Node head;
private transient volatile Node tail;
private volatile int state;
```

事先会获取这些实例变量在内存中的偏移量，直接根据偏移量找到属性进行修改

```java
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long stateOffset;
private static final long headOffset;
private static final long tailOffset;
private static final long waitStatusOffset;
private static final long nextOffset;

static {
    try {
        stateOffset = unsafe.objectFieldOffset
            (AbstractQueuedSynchronizer.class.getDeclaredField("state"));
        headOffset = unsafe.objectFieldOffset
            (AbstractQueuedSynchronizer.class.getDeclaredField("head"));
        tailOffset = unsafe.objectFieldOffset
            (AbstractQueuedSynchronizer.class.getDeclaredField("tail"));
        waitStatusOffset = unsafe.objectFieldOffset
            (Node.class.getDeclaredField("waitStatus"));
        nextOffset = unsafe.objectFieldOffset
            (Node.class.getDeclaredField("next"));

    } catch (Exception ex) { throw new Error(ex); }
}
```

——why要设置偏移量，因为CAS是用了unsafe的方法，而该方法是native的，所以直接对内存进行修改。

一些设置操作：

```java
private void setHead(Node node) {			// 设置头结点
    head = node;
    node.thread = null;			// 头节点是空节点，所以要把里面的内容给清掉
    node.prev = null;
}
```



参考：

1. https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html
2. https://www.cnblogs.com/fsmly/p/11274572.html