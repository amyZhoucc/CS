# 并发AQS理解

Java中的大部分同步类（Lock、Semaphore、ReentrantLock等）都是基于AQS

eg：**ReentrantLock的底层实现是基于CAS+AQS**

AQS：AbstractQueuedSynchronizer：抽象队列同步器。AQS是提供了一种原子式同步状态、阻塞、唤醒线程功能及队列模型的简单框架。

ps：ReentrantLock：显式锁，底层AQS + CAS；synchronized：隐式锁，监视器模式

### （1）AQS的原理

- 如果被请求的共享资源空闲，那么就将当前请求资源的线程设置为有效的工作线程，将共享资源设置为锁定状态；
- 如果共享资源被占用，就需要一定的阻塞等待唤醒机制来保证锁分配。这个机制主要用的是**CLH队列的变体实现的**，将暂时获取不到锁的线程加入到队列中。

CLH：Craig、Landin and Hagersten队列，是单向链表，AQS中的队列是CLH变体的虚拟双向队列（FIFO）。

AQS是通过将每条请求共享资源的**线程封装成一个节点**来实现锁的分配。AQS使用一个**Volatile**的int类型的**成员变量**来表示同步状态，通过内置的**FIFO队列**来完成资源获取的排队工作，通过**CAS**完成对State值的修改。

下面介绍一下队列中每个节点的内容：

| 方法和属性值 | 含义                                                  |
| ------------ | ----------------------------------------------------- |
| waitStatus   | 等待状态标志位，表示当前节点在队列中的状态            |
| thread       | 表示处于该节点的线程                                  |
| prev         | 前驱指针                                              |
| next         | 后继指针                                              |
| predecessor  | 返回前驱节点，没有的话抛出npe                         |
| nextWaiter   | 指向下一个处于CONDITION状态的节点（这个指针不多介绍） |

ps：变量waitStatus：共有4种取值

- 0：初始化默认的值
- CANCELLED：1，线程获取锁的请求已经取消了，在同步队列中等待的线程等待超时或被中断
- SIGNAL：-1，表示线程已经准备好了，就等资源释放了
- CONDITION：-2，表示节点在等待队列中，节点线程等待唤醒（与Condition相关，该标识的结点处于等待队列中，结点的线程等待在Condition上，当其他线程调用了Condition的signal()方法后，CONDITION状态的结点将从等待队列转移到同步队列中）
- PROPAGATE：-3，当前线程处在SHARED情况下，该字段才会使用，与共享模式相关，在共享模式中，该状态标识结点的线程处于可运行状态。

参考文章：

https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html

