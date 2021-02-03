# ReentrantLock部分源码分析

ReentrantLock是显式锁、互斥锁、可配置是否公平锁、可重入锁、独占锁

下面分析一些关键代码（不会进行dfs进行查询，而是BFS式的）

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

