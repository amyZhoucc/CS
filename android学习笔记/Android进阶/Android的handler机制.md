# Android的handler机制

面试超高频（去年秋招的时候连听都没听过，真是罪过:sweat:）

学完我觉得handler机制最妙的在于：整个过程没有涉及到显式的线程切换，但是确实通过几个对象和threadLocal机制等，实现了线程消息传递，以及对应线程处理对应任务

前提知识：

1. **更新UI的操作只能在主线程中执行**

   （更准确的来说，只有创建视图层次结构的线程才能够访问视图——哪个线程创建界面，哪个线程更新——一般就是主线程）

   why不能在子线程中更新UI：存在多线程并发问题；而如果加锁，就会影响性能，出现界面卡顿

2. **主线程中不能进行耗时的操作**

   主线程耗时操作，那么界面不能操作，随之出现**ANR问题**

——如果要执行耗时操作、如果子线程中要更新ui——这些都可以用handler机制解决

## 定义：

> **handler机制是Android中基于单线消息队列模式的一套线程消息机制**
>
> 1. 消息机制：handler机制的本质是负责消息的分发和处理
> 2. 单线消息队列：类似于流水线，**每个线程都有一个流水线**，可以往流水线上放消息，流水线末端会有人处理消息，流水线是单线的，所以是**按照先来后到的顺序依次处理**

handler是**事务驱动型设计**：所有线程都是通过等待message来驱动线程执行的，而线程之间的交流是通过message来传递的。所以通过不断地分发事物来实现整个程序的运行（事务的产生主要来自于外部中断——用户的点击等，内部中断——定时器等）

messageQueue是**生产者-消费者模型**，handler是生产者，会将message放在messageQueue上，looper是消费者，会将message从messageQueue上取出，交给handler执行。

潜在的死锁问题？

eg：looper获取锁之后，但是messageQueue没有消息，就会陷入阻塞（此时仍然持有锁）；而handler需要将message放进入的时候，需要获取锁，所以无法将message放入，所以陷入死锁

——解决方法：循环加锁（阻塞操作在加锁之外）具体见[messageQueue的next方法](#next)

## handler存在的必要性

1. **实现线程切换**

   handler能够根据消息隐式的切换线程

2. **能够按照顺序处理消息，避免并发**

   单消息队列，能够保证一个线程处理事件的先后顺序

   eg：多个子线程需要修改UI，都必须要交给主线程执行，所以需要通过handler机制发送，而因为是单线消息队列，所以按照顺序处理，避免并发修改UI

   更多的，在ActivityThread中有一个H类，他其实就是个Handler。在ActivityThread中定义了一百多中消息类型以及对应的处理逻辑

3. **阻塞线程，避免让线程结束**

   handler机制中，如果线程没有任务就会被阻塞，从而让其他线程获取时间片去执行，当该线程又有任务的时候，会被唤醒继续从消息队列上获取任务去执行

4. **能够延迟处理消息**

   handler机制能够处理需要延迟处理的任务，eg：一个任务需要等待5s去执行，那么只需要handler只需要带上延迟的时间，对应的线程会休眠对应的时间后才开始处理消息

## handler在Android中的地位

Android的main函数：

```java
public static void main(String[] args) {
    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ActivityThreadMain");
    AndroidOs.install();
    CloseGuard.setEnabled(false);
    Environment.initForCurrentUser();
    final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());
    TrustedCertificateStore.setDefaultUserDirectory(configDir);
    Process.setArgV0("<pre-initialized>");
    Looper.prepareMainLooper();			// 主线程的looper会在main中默认启动
    long startSeq = 0;
    if (args != null) {
        for (int i = args.length - 1; i >= 0; --i) {
            if (args[i] != null && args[i].startsWith(PROC_START_SEQ_IDENT)) {
                startSeq = Long.parseLong(
                        args[i].substring(PROC_START_SEQ_IDENT.length()));
            }
        }
    }
    ActivityThread thread = new ActivityThread();		// 再创建主线程
    thread.attach(false, startSeq);
    if (sMainThreadHandler == null) {
        sMainThreadHandler = thread.getHandler();			// 创建handler
    }
    if (false) {
        Looper.myLooper().setMessageLogging(new
                LogPrinter(Log.DEBUG, "ActivityThread"));
    }
    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);		// 启动looper
    Looper.loop();
    throw new RuntimeException("Main thread loop unexpectedly exited");
}
```

可以从main函数看出来，先创建**主线程的looper**，再创建主线程，然后启动线程

——main中会创建主线程和两个binder线程，AMS（Android Manager Service）通过binder机制和程序进行联系，而binder线程和主线程通过handler来发送message，从而驱动整个Android运行

图是整个程序的启动过程：

<img src="..\pic\image-20220429183316979.png" alt="image-20220429183316979" style="zoom:50%;" />

**当应用程序被创建的时候，一开始只有主线程（ActivityThread）和2个Binder线程，主线程创建了Looper和handler。**

**之后AMS和binder线程进行通信，通过给主线程发送message，从而让程序跑起来发送activity**

优势：handler这样的设计，不需要写死循环——来避免退出等待，也不需要等待用户输入，程序跑起来就不会停止。启动程序之后，还可以开启其他线程来接收用户输入，然后通过message发送到主线程更新UI。

ps：在看文章的时候，一直不了解“内部类H”

翻源码的时候找到了：

```java
// ActivityThread.java中
final H mH = new H();
final Handler getHandler() {
    return mH;
}
public static void main(String[] args) {
    ...
    if (sMainThreadHandler == null) {
        sMainThreadHandler = thread.getHandler();			// 获取handler
    }
    ...
}
class H extends Handler {			// 继承自Handler的内部类——ActivityThread内部类
    ....
}
```

## handler的使用过程

> 1. 创建Looper
> 2. 使用Looper创建handler
> 3. 启动Looper
> 4. 使用handler发送消息

更具体的可以看：https://www.jianshu.com/p/e172a2d58905(对理解整个handler机制很有帮助)

### 1. 创建Looper

首先：一个线程只有一个Looper

1. **主线程已经默认创建好了**：在main方法中的==**`Looper.prepareMainLooper();`**==方法在主线程创建的时候产生
2. 子线程需要自行创建：==**`Looper.prepare();`**==

### 2. 创建handler

有2种方法创建：

1. 使用callback创建——适合简单的逻辑

   ==**`Handler(Looper.myLooper(), new Callback(){...})`**==

   ```java
   public class MainActivity extends AppComposeActivity{
       public void onCreate(Bundle saveInstanceState){
           super.onCreate(saveInstanceState);
           ...
           Handler handler = Handler(Looper.myLooper(), new Callback(){
               public boolean handleMessage(Message msg){
                   ....
               }
           });    
       }
   }
   ```

2. 继承handler创建——适合填写复杂的调用逻辑

   ==**`static MyHandler extends Handler{....}`**==

   ```java
   public class MainActivity extends AppComposeActivity{
       ...
       
       // 静态内部类
       static MyHandler extends Handler{
           public MyHandler(Looper looper){
               super(looper);
           }
           public void handleMessage(Message msg){
               super.handleMessage(msg);
               ...		// 重写该方法
           }
       }
   }
   ```

   注意：该继承必须要创建的是静态内部类，否则会造成**内存泄漏**

   本质因为：

   1. **非静态内部类会持有外部类的引用**。
   2. 而**handler发出的message会持有该handler的引用**
   3. 如果message是延迟处理的消息，它还在流水线上；而此时对应的activity退出了
   4. **message持有handler，handler持有activity**，那么对应的activity无法退出——造成内存泄漏

### 3. 启动looper

==**`Looper.loop();`**==

### 4. 使用

首先需要创建handler对象——在对应的线程中创建handler对象：==**`Handler handler = new Handler(Looper.myLooper())`**==

需要指定对应的looper——也就是指定对应的线程（本人觉得这句话最关键，不然就无法实现线程间的消息传递）

有两类方法：

1. ==**`handler.sendMessage(msg)`**== / `handler.sendMessageDelayed(msg, delayTime)`
2. ==**`handler.post(runnable)`**== / `handler.postDelayed(runnable, delayTime)`

更全面的：

```java
public final boolean post(@NonNull Runnable r);
public final boolean postDelayed(@NonNull Runnable r, long delayMillis);
public final boolean postAtTime(@NonNull Runnable r, long uptimeMillis);
public final boolean postAtFrontOfQueue(@NonNull Runnable r);

public final boolean sendMessage(@NonNull Message msg);
public final boolean sendMessageDelayed(@NonNull Message msg, long delayMillis);
public boolean sendMessageAtTime(@NonNull Message msg, long uptimeMillis);
public final boolean sendMessageAtFrontOfQueue(@NonNull Message msg)
```

获得对应线程的handler对象，然后调用上面的方法向该线程发消息，线程收到消息之后，按照流水线上顺序一次处理

## handler机制的内部模式

handler机制由几个方面组成的：

1. handler：是**整个消息机制的发起者和处理者**。真正处理事情的地方，可以有很多个
2. message / runnable：事件
3. looper：**一个线程只有一个**，循环执行消息的处理。
4. messageQueue：消息队列，是Looper的内部对象，同样一个线程只有一个

<img src="..\pic\image-20220427103344663.png" alt="image-20220427103344663" style="zoom:55%;" />

通过handler将message/runnable发送到对应的线程，message/runnable会被加入到messageQueue，looper算是一个死循环，不断从messageQueue中按照顺序取出，交给对应的handler（就是最初发送过来的handler）去处理——用handleMessage方法

### 基本的类图

<img src="..\pic\image-20220429161714118.png" alt="image-20220429161714118" style="zoom:67%;" />



## Handler机制的关键内部类

对应的就是上一部分说到的4个方面

### Message

存放消息的类，主要关键的是它的属性

```java
public int what;			// 用户自定义，用来区分message的类型
public int arg1;
public int arg2;			// 可以用来传递数据
public Object obj;			// 可以用来存放可序列化的对象类型
Bundle data;				// 可以用来存储Bundle数据类型

public long when;			// 事件处理的事件，是相对1970.1.1开始的，对用户不可见
Handler target;				// 处理事件的handler对象，对用户不可见
Runnable callback;			// 存放runnable对象（post方法调用时会将其封装为Message对象），对用户不可见
Message next;				// MessageQueue是一个链表，这个就是用来连接message的，对用户不可见
```

Message在实现的时候，也会存在一个类似于**消息池**（类比于线程池）——Message对象是循环使用的

官方建议：使用==**`Message.obtain()`**==去获取message对象，当使用完毕之后用==**`message.recycle()`**==进行回收

```java
public static Message obtain() {
    synchronized (sPoolSync) {		// 线程安全——线程之间是共用的
        if (sPool != null) {
            Message m = sPool;		// 获取消息队列的第一个message对象——就是要被使用的
            sPool = m.next;			// 更新消息池的队首
            m.next = null;			// 断开连接
            m.flags = 0; 
            sPoolSize--;
            return m;
        }
    }
    return new Message();		// 如果没有获取到，则创建一个新的message对象
}
```

理解：Message维护了一个静态链表，这就是消息池。通过message.next来进行链表连接

调用Message的recycle进行回收，而如果message正在被使用就会抛出异常；否则就调用`recyclerUnchecked`进行回收：把message里面的属性内容清空，然后尝试加入链表，如果消息池的链表长度超过50，就直接回收，否则就加入到链表中

——由于有消息池机制，所以官方建议用提供的方法去获取消息池中的message

### MessageQueue

本质上是链表结构，利用message提供好的属性来方便实现链表。

关键方法：入队、出队

难点：线程休眠，当MessageQueue为空，或者里面的message都延迟处理的时候，就会让线程休眠，让出cpu。当MessageQueue有任务的时候，又会被唤醒并且执行。

#### 消息出队<a name="next"></a>

```java
Message next() {
    final long ptr = mPtr;
    if (ptr == 0) {				// looper已经退出，则直接返回null
        return null;
    }
    ...
    int nextPollTimeoutMillis = 0;		// 记录阻塞时间
    for (;;) {				// 是一个死循环
        if (nextPollTimeoutMillis != 0) {
            Binder.flushPendingCommands();
        }
        nativePollOnce(ptr, nextPollTimeoutMillis);
        synchronized (this) {			// 保证线程安全
            final long now = SystemClock.uptimeMillis();
            Message prevMsg = null;
            Message msg = mMessages;
            ...
            if (msg != null) {		// 获取message
                if (now < msg.when) {	// 如果当前时间还没到该消息的处理时间——则开始阻塞
                    nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                } else {		// 到处理的时间了 / 立即处理
                    mBlocked = false;		// 标记messageQueue为非阻塞
                    if (prevMsg != null) {
                        prevMsg.next = msg.next;
                    } else {
                        mMessages = msg.next;
                    }
                    msg.next = null;
                    msg.markInUse();		// 标记message正在被执行
                    return msg;
                }
            } else {		// 表示messageQueue为空，进入无限的休眠状态，直到有新的message过来
                nextPollTimeoutMillis = -1;	// -1是无限循环的标记，只有通过唤醒才能打破阻塞
            }
           ...
    }
}
```

总体逻辑：

1. Looper已经退出，就直接返回null
2. Looper还在，就进入死循环，直到获取到了message或者Looper退出了
   1. 先判断是否需要继续阻塞，如果需要，就继续；不需要就进行加锁操作
   2. 如果message存在，判断message是否需要延迟处理
      1. message需要在指定时间内执行，那么线程休眠指定的时间
      2. message可以立即执行，进行链表操作之后，将message返回
   3. 如果message不存在，则陷入无限循环，直到有新的message来

#### 消息入队

```java
boolean enqueueMessage(Message msg, long when) {
    if (msg.target == null) {		// 要指定handler，没有就返回异常
        throw new IllegalArgumentException("Message must have a target.");
    }
    if (msg.isInUse()) {	// 如果message正在被处理，那么也要返回异常
        throw new IllegalStateException(msg + " This message is already in use.");
    }

    synchronized (this) {		// 加锁，保证并发操作安全
        if (mQuitting) {		// 判断thread是否已经死亡
            IllegalStateException e = new IllegalStateException(
                    msg.target + " sending message to a Handler on a dead thread");
            Log.w(TAG, e.getMessage(), e);
            msg.recycle();			// message不处理，直接回收
            return false;
        }
        
        msg.markInUse();		// 正常情况下，message标记为正在被执行，以及期待执行的时间
        msg.when = when;

        Message p = mMessages;		// 获取链表头
        boolean needWake;
		
        // 链表为空/message立即执行/message的执行时间早于链表头message执行时间（链表头是最早要执行的message）
        if (p == null || when == 0 || when < p.when) {
            msg.next = p;			// 当前message作为链表头
            mMessages = msg;
            needWake = mBlocked;
        } else {	
            //...
            // 是否被环形的条件：是阻塞状态且链表头是同步屏障，要插入的message是异步消息
            needWake = mBlocked && p.target == null && msg.isAsynchronous();
            Message prev;
            for (;;) {			// 遍历，找到合适的执行时间的位置插入
                prev = p;
                p = p.next;
                if (p == null || when < p.when) {		// 找到不早于when的时间的message之前插入
                    break;
                }
                // 如果message之前有一个异步消息，说明前面有延迟的异步消息（它都没有执行，则不需要被唤醒）
                if (needWake && p.isAsynchronous()) {
                    needWake = false;
                }
                ...
            }
            msg.next = p; 
            prev.next = msg;
        }

        if (needWake) {		// 如果当前是阻塞状态且存在同步屏障且插入的异步message位于所有异步message之前，就唤醒队列
            nativeWake(mPtr);
        }
    }
    return true;
}
```

总体逻辑：

1. 如果message对应的handler为空，或者正在处理中，就抛出对应的异常；否则就进行下一步
2. 上锁，判断线程是否还存活，如果死亡，就抛出异常，并将message回收，返回false表示操作失败
3. 线程存活后，将message标记为inuse——（等待）处理中，以及记录要延迟执行的时间（方法传参获得的）；接着获取链表头节点。判断链表是否为空（头结点为null）/ message是立即执行的 / 需要延迟执行，但是message的执行时间早于头结点的执行时间（头结点是链表中最早执行的message）
   1. 判断为true，那么将message作为链表头节点
   2. 判断为false，遍历链表，找到不早于该message执行时间的节点前插入
4. 根据标志位，判断是否要唤醒messageQueue。有两种情况：message成为新的链表头的状态下是：messageQueue本来是空的，或者是等待原本的链表头的延迟时间到期，那么就要唤醒messageQueue，才能再返回 

### Looper

Looper是handler机制的核心，就跟传送带一样，只有它动起来了，messageQueue才有意义

Looper可以看成handler机制的引擎。

Looper的任务：它负责从messageQueue中取出message，交给对应的handler处理。如果messageQueue中没有任务，则next()方法会阻塞当前线程，直到有新的message出现 / 延迟处理的message到期

**Looper通过ThreadLocal来保证每个线程只有一个相同的副本**

```java
Looper.class

static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();	// 它是Looper类的静态变量，不同的线程，里面的值不一样

public static void prepare() {
    prepare(true);		// 默认表示可以退出
}

// 最终调用到了这个方法，参数表示是否Looper可以退出（主线程不允许，且主线程的Looper是默认创建的）
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {		// 如果已经存在，就抛出异常
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}

// Looper的构造方法
private Looper(boolean quitAllowed) {
    mQueue = new MessageQueue(quitAllowed);		// 创建对应的messageQueue
    mThread = Thread.currentThread();			// 获取当前的thread，作为looper的thread对象
}
```

理解：都需要通过`Looper.prepare()`来初始化Looper

获取当前线程的looper：

```java
public static @Nullable Looper myLooper() {
    return sThreadLocal.get();		// 直接调用thread对应的threadlocal即可
}
```

启动循环：`Looper.loop()`

```java
public static void loop() {
    final Looper me = myLooper();		// 获取当前thread对应的looper
    if (me == null) {			// 尚未创建就调用，抛出异常
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    final MessageQueue queue = me.mQueue;		// 获取looper对应的messageQueue
    ...
    for (;;) {			// 无限循环
        Message msg = queue.next(); // 获取queue的下一个message，可能会出现线程阻塞（但是程序不结束）
        if (msg == null) {		// 返回null，说明messageQueue退出了
            return;			// 直接返回，不再执行无穷循环
        }
        ...
        try {
            msg.target.dispatchMessage(msg);		// 将message丢给对应的handler去处理
            if (observer != null) {
                observer.messageDispatched(token, msg);
            }
            dispatchEnd = needEndTime ? SystemClock.uptimeMillis() : 0;
        }
        ...
        msg.recycleUnchecked();		// 回收对应的message（因为已经执行完成了）
    }
}
```

理解：

1. 尝试获取当前线程的looper，如果没有获得，表示尚未创建looper，则抛出异常；获得，则进入下一步
2. 获取looper对应的messageQueue，则进入死循环
3. 循环中，尝试获取下一个message，如果message为null，表示messageQueue已经退出，则looper也退出——**调用quit，将对应的退出标志位标记——在获取next的时候，进行判断，发现looper已经退出了，直接返回null——looper死循环中，得到返回值为null，知道我已经被调用quit退出了，那么也跟着退出死循环，返回**；message不为null
4. 将message交给对应的handler进行处理，调用handler的dispatchMessage方法（**不同的handler在不同的线程中，所以也实现了线程的切换**）
5. 执行完成后，将message对象回收

looper退出：

```java
public void quit() {
    mQueue.quit(false);
}
// 最终都是调用到了这个方法
void quit(boolean safe) {
    // 如果不能退出则抛出异常
    if (!mQuitAllowed) {
        throw new IllegalStateException("Main thread not allowed to quit.");
    }

    synchronized (this) {		// 线程安全
        if (mQuitting) {		// 表示已经执行过之后了，直接返回
            return;
        }
        mQuitting = true;		// 标记已经退出过了
        if (safe) {
            removeAllFutureMessagesLocked();	// 将不需要等待的消息/已经到期的消息处理完之后再退出
        } else {
            removeAllMessagesLocked();		// 直接将所有的message全部移除，直接退出
        }
        nativeWake(mPtr);		// 唤醒MessageQueue
    }
}
```

注意：looper一旦退出，对应的线程也结束了——因为没有任务给线程了，线程也没存在的必要了

### Handler

post方法（其他都是殊途同归）

```java
public final boolean post(@NonNull Runnable r) {
    return  sendMessageDelayed(getPostMessage(r), 0);
}
// 对runnable对象进行封装
private static Message getPostMessage(Runnable r) {
    Message m = Message.obtain();		// 从对象池中获取一个message对象
    m.callback = r;			// 将runnable对象封装到callback中（因为最后还是callback处理）
    return m;
}

// 大部分post、send方法都会调用到该方法——主要是判断延迟时间是否有误，并且计算出延迟到几点的时间（传递的参数是延迟一段多长的时间）
public final boolean sendMessageDelayed(@NonNull Message msg, long delayMillis) {
    if (delayMillis < 0) {
        delayMillis = 0;
    }
    return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
}
// 最终调用到这边
public boolean sendMessageAtTime(@NonNull Message msg, long uptimeMillis) {
    MessageQueue queue = mQueue;			// 获取当前线程的messageQueue
    if (queue == null) {			// messageQueue尚未初始化就抛出异常
        RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    return enqueueMessage(queue, msg, uptimeMillis);// 都正常，将message加入到对应线程的messageQueue队列中
}

```

handler的`dispatchMessage`方法（在looper中，成功获取message之后，执行message时会调用的方法）——**`msg.target.dispatchMessage(msg);`**

```java
public void dispatchMessage(@NonNull Message msg) {
    if (msg.callback != null) {		// 判断message是否是被包裹的runnable对象
        handleCallback(msg);		// 那么直接交给callback（runnable对象）去执行
    } else {
        if (mCallback != null) {		// 判断handler是否有callback
            if (mCallback.handleMessage(msg)) {		// 有就，直接执行
                return;
            }
        }
        handleMessage(msg);	// 自身不是runnable对象，或者没有callback，就调用handler的handlemessage执行
    }
}

private static void handleCallback(Message message) {
    message.callback.run();
}
```



## ThreadLocal

是handler机制生效的重要组成部分

### 定义

> ThreadLocal是用于**线程内部存储数据**的工具类
>
> 1. 线程内部：每个线程对于其他线程来说，该变量的数据值是不同的
> 2. 存储数据

它主要的使用场景：**相同的数据类型、但是不同线程需要有不同备份的情况**

而Looper作为线程独有的对象，就适合用ThreadLocal来进行存储

### ThreadLocal使用过程

```java
ThreadLocal<String> stringLocal = new ThreadLocal();	// 需要指定存储的数据类型
stringLocal.set("test");
String s = stringLocal.get();
```

并且可以验证，不同的线程之间得到的数据是不一样的

```java
public static void main(String[] args){
    ThreadLocal<String> stringLocal = new ThreadLocal();
    stringLocal.set("test");
    String s = stringLocal.get();			// 得到的是"test"的字符串
    
    new Thread(
        () -> System.out.println(stringThreadLocal.get())		// 得到的是null
    ).start();		
}
```

### ThreadLocal的内部机制

<img src="..\pic\image-20220427133158295.png" alt="image-20220427133158295" style="zoom:50%;" />

每个线程都有一个**ThreadLocalMap**，类似于HashMap——由于ThreadLocalMap是跟着Thread对象走的，所以只有获得Thread对象才能获取里面的ThreadLocal值——这就实现了线程隔离

里面按照键值对进行存储，即Entry：ThreadLocal - value，它是一种**弱引用关系**

```java
static class Entry extends WeakReference<ThreadLocal<?>> {		// 弱引用
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

内存泄漏的问题：ThreadLocal是一种弱引用，value是一种强引用。外部没有对ThreadLocal的引用后，ThreadLocal会被回收，那么value也就没有意义——但是因为是强引用，所以没法回收

——内部解决：**ThreadLocal当不需要使用的时候，记得调用remove方法将Entry移除，从而防止内存泄漏**

#### ThreadLocalMap的set方法

```java
public void set(T value) {
	Thread t = Thread.currentThread();		// 获取当前线程对象
    ThreadLocalMap map = getMap(t);		// 根据线程获取线程特有的ThreadLocalMap
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

private void set(ThreadLocal<?> key, Object value) {
    Entry[] tab = table;		// ThreadLocalMap内部用数组存储，基本元素是键值对对象Entry
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);	// 获取键的hash值，并用当前的数组长度，得到存储的索引下标
    for (Entry e = tab[i];
            e != null;
            e = tab[i = nextIndex(i, len)]) {		// 开放地址法，占了就找下一个
        ThreadLocal<?> k = e.get();
        if (k == key) {			// 如果key一样，就直接覆盖
            e.value = value;
            return;
        }
        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }
    tab[i] = new Entry(key, value);
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();		// 扩容
}
```

注意：ThreadLocalMap和HashMap机制类型，但是它用的是**开放地址法来处理哈希冲突**（HashMap是用链表法）

#### ThreadLocalMap的get方法

```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();		// 如果ThreadLocalMap不存在，就会初始化
}

private Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);	// 定位开始的hash值
    Entry e = table[i];
    if (e != null && e.get() == key)		// 找到就直接返回
        return e;
    else
        return getEntryAfterMiss(key, i, e);		// 没有找到就找下一个index的
}

```

## HandlerThread

背景：有时需要创建一个线程去执行耗时的任务，可以在线程的run方法中，再创建并初始化looper，然后可以切换线程执行任务了

```java
class MyThread extends Thread{
    public Handler myHandler;
    @Override
    public void run(){				// 子线程中执行
        super.run();
        Looper.prepare();
        myHandler = new Handler(Looper.myLooper());
        Looper.loop();
        myHandler.sendMessage(Message.obtain());
    }
};
MyThread myThread = new MyThread();
myThread.start();
myThread.myHandler.sendMessage(Message.obtain());		// 主线程中
```

会存在问题，因为子线程启动需要时间，所以在run执行时，可能主线程会去调用handler的sendmessage方法，而此时handler尚未初始化，所以会出现报错——尚未初始化，就被调用

——所以可以用内置的HandlerThread

```java
public class HandlerThread extends Thread {			// 本质上还是继承自Thread
    int mPriority;		// 优先级
    int mTid = -1;		// 该线程的tid
    Looper mLooper;			// 该线程包含的looper对象
    private @Nullable Handler mHandler;			// 内置的handler

    // 构造方法中，要给定线程名字，此外还能设置优先级
    public HandlerThread(String name) {
        super(name);
        mPriority = Process.THREAD_PRIORITY_DEFAULT;
    }
    public HandlerThread(String name, int priority) {
        super(name);
        mPriority = priority;
    }
    
    // 在Looper开始运行前的方法
    protected void onLooperPrepared() {
    }

    // 初始化并启动Looper
    @Override
    public void run() {
        mTid = Process.myTid();
        Looper.prepare();		// 初始化looper
        synchronized (this) {				// 加锁
            mLooper = Looper.myLooper();	
            notifyAll();		// 初始化完成后，唤醒正在上面等待的线程		
        }
        Process.setThreadPriority(mPriority);
        onLooperPrepared();
        Looper.loop();			// 启动looper
        mTid = -1;
    }
    
    // 获取当前线程的Looper
    public Looper getLooper() {
        if (!isAlive()) {
            return null;
        }
        // 如果尚未初始化则会一直阻塞直到初始化完成
        synchronized (this) {
            while (isAlive() && mLooper == null) {
                try {
                    wait();			
                } catch (InterruptedException e) {
                }
            }
        }
        return mLooper;
    }

    // 获取handler，该方法被标记为hide，用户无法获取
    @NonNull
    public Handler getThreadHandler() {
        if (mHandler == null) {
            mHandler = new Handler(getLooper());
        }
        return mHandler;
    }

    // 两种不同类型的退出
    public boolean quit() {
        Looper looper = getLooper();
        if (looper != null) {
            looper.quit();
            return true;
        }
        return false;
    }
    public boolean quitSafely() {
        Looper looper = getLooper();
        if (looper != null) {
            looper.quitSafely();
            return true;
        }
        return false;
    }

    public int getThreadId() {
        return mTid;
    }
}
```

### HandlerThread使用

```java
HandlerThread thread = new HandlerThread("handlerThread");
thread.start();
Handler handler = new Handler(thread.getLooper());	// 能够保证looper初始化了再返回
handler.sendMessage(Message.obtain());
```

## handler同步屏障

主要是用来加急处理message。

背景：例如主线程更新UI的操作，但是messageQueue上待处理的message很多，所以该操作需要等很久才能执行，所以会造成界面卡顿——可以使用同步屏障

同步屏障的含义：让同步的消息被阻碍不执行，而让**异步的消息先执行**——所以，加急的message就是异步的message

```java
public int postSyncBarrier() {			// 插入同步屏障
    return postSyncBarrier(SystemClock.uptimeMillis());
}

private int postSyncBarrier(long when) {
    synchronized (this) {
        final int token = mNextBarrierToken++;
        final Message msg = Message.obtain();		// 去创建一个message
        msg.markInUse();
        msg.when = when;
        msg.arg1 = token;		// 注意没有给message的target赋值，所以target=null

        Message prev = null;
        Message p = mMessages;
        // 把当前需要执行的Message都放在新message之前
        if (when != 0) {
            while (p != null && p.when <= when) {		// 如果同步屏障需要延期执行，则找到合适的插入位置
                prev = p;
                p = p.next;
            }
        }
        // 在合适的位置插入同步屏障（就是在不迟于when的message后插入）
        if (prev != null) { // invariant: p == prev.next
            msg.next = p;
            prev.next = msg;
        } else {
            msg.next = p;
            mMessages = msg;		// 否则，内存屏障就作为队首
        }
        return token;
    }
}
```

同步屏障本质上就是message，特殊的是是target=null

```java
Message next() {
    ...

    // 阻塞时间
    int nextPollTimeoutMillis = 0;
    for (;;) {
        ...
        // 阻塞对应时间 
        nativePollOnce(ptr, nextPollTimeoutMillis);
        synchronized (this) {
            final long now = SystemClock.uptimeMillis();
            Message prevMsg = null;
            Message msg = mMessages;
            /**
            *  1：针对target==null——就是同步屏障（队首为同步屏障）
            */
            if (msg != null && msg.target == null) {
                // 循环找同步屏障的下一个异步消息（即同步屏障）
                do {
                    prevMsg = msg;
                    msg = msg.next;
                } while (msg != null && !msg.isAsynchronous());
            }
            if (msg != null) {		// 如果存在下一个同步屏障
                if (now < msg.when) {	// 根据时间，判断是否开始。还没开始，等待两者的时间差
                    nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                } else {	// 存在异步消息，现在要执行，标记MessageQueue为非阻塞
                    mBlocked = false;
                    /**
                    *  2：异步消息需要立即执行的，从队列中删除该异步消息
                    */
                    if (prevMsg != null) {
                        prevMsg.next = msg.next;
                    } else {
                        mMessages = msg.next;
                    }
                    msg.next = null;
                    msg.markInUse();
                    return msg;			// 取出该得到的消息
                }
            } else {
                // 没有消息，进入阻塞状态
                nextPollTimeoutMillis = -1;
            }

            // 当调用Looper.quitSafely()时候执行完所有的消息后就会退出
            if (mQuitting) {
                dispose();
                return null;
            }
            ...
        }
        ...
    }
}

```

理解：

1. 判断出有同步屏障了（队首message的target为null），那么遍历链表，将所有到期的异步message全部执行完毕——那么异步message就比同步message（就是实现了优先执行）

当然同步屏障的有效，需要有加急的message调用了==**`msg.setAsynchronous(true)`**==，即异步消息，那么设置了同步屏障后，到期了就会比其他的同步屏障优先执行

注意：

同步屏障不会自动移除，需要手动调用==**`removeSyncBarrier()`**==去执行，否则同步message一直无法被执行

## IdleHandler

背景：当messageQueue为空，或者没有需要执行的message时会回调的接口对象

idleHandler本质上是一个接口，和Handler没啥关系

```java
public static interface IdleHandler {
    boolean queueIdle();
}
```

messageQueue会有一个list存储一个idleHandler对象，当messageQueue没有需要执行的任务就会遍历回调所有idleHandler——主要就是为了在消息队列空闲的时候处理轻量级的工作

```java
Message next() {
    // 如果looper已经退出了，这里就返回null
    final long ptr = mPtr;
    if (ptr == 0) {
        return null;
    }
    int pendingIdleHandlerCount = -1; 		// 计算idelHandler的数量
    int nextPollTimeoutMillis = 0;
    for (;;) {
        	...

            // 当队列中的消息用完了或者都在等待时间延迟执行同时给pendingIdleHandlerCount<0
            // 给pendingIdleHandlerCount赋值MessageQueue中IdleHandler的数量
            if (pendingIdleHandlerCount < 0
                    && (mMessages == null || now < mMessages.when)) {
                pendingIdleHandlerCount = mIdleHandlers.size();
            }
            // 如果没有需要执行的IdleHanlder直接continue
            if (pendingIdleHandlerCount <= 0) {
                // 标记MessageQueue进入阻塞状态
                mBlocked = true;
                continue;
            }

            // 把List转化成数组类型
            if (mPendingIdleHandlers == null) {
                mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
            }
            mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
        }

        // 执行IdleHandler
        for (int i = 0; i < pendingIdleHandlerCount; i++) {
            final IdleHandler idler = mPendingIdleHandlers[i];
            mPendingIdleHandlers[i] = null; // 释放IdleHandler的引用
            boolean keep = false;
            try {
                keep = idler.queueIdle();		// 具体执行的操作
            } catch (Throwable t) {
                Log.wtf(TAG, "IdleHandler threw exception", t);
            }
            // 如果返回false，则把IdleHanlder移除
            if (!keep) {
                synchronized (this) {
                    mIdleHandlers.remove(idler);
                }
            }
        }

        // 最后设置pendingIdleHandlerCount为0，防止再执行一次
        pendingIdleHandlerCount = 0;

        // 当在执行IdleHandler的时候，可能有新的消息已经进来了
        // 所以这个时候不能阻塞，要回去循环一次看一下
        nextPollTimeoutMillis = 0;
    }
}
```

主要逻辑：

1. 刚进入next的时候`pendingIdleHandlerCount = -1`
2. 当messageQueue没有需要处理的message时，判断`pendingIdleHandlerCount < 0`是否成立。如果成立，就将当前的idelhandler数组的长度赋值给`pendingIdleHandlerCount`
3. 如果没有需要执行的idelHandler（size=0），那么标记进入阻塞状态，然后结束下面的任务，进入下一次循环
4. 将存储idelHandler的list转换为数组类型
5. 开始执行每个idlehandler，并且将对应数组的元素清空（防止内存泄漏），如果执行之后返回false，那么将该idleHandler从队列中移除（并发安全，所以要加锁）
6. 将计数`pendingIdleHandlerCount `置0，表示都执行完成了，且不能进入阻塞状态，而重新进入for循环的开头再判断一次

主要参考：

1. https://juejin.cn/post/6887918066043191304
2. https://juejin.cn/post/6918272111152726024
2. https://juejin.cn/post/6887927905473527815
2. https://juejin.cn/post/6887930091991007240
2. https://juejin.cn/post/6887931460024696839
2. https://juejin.cn/post/6887933281686421518
2. https://juejin.cn/post/6844903910113705998
3. 《深入理解Android内核设计思想》

