

## 四、并发

之前都是假设程序中只有一个线程一条执行流，即从main函数开始一条一条执行下去——效率不高

为了提高效率，在程序中**创建多条线程来启动多条执行流**——加快执行速度，但是程序执行变得复杂

## （一）并发的基础知识

### 1. Java线程

这边的线程和OS中的线程还是存在一些差异，但是概念逻辑上类似

在Java中一个线程就是一条执行流，有自己的PC，虚拟机栈（JVM的知识）

#### （1）线程的创建

如何创建线程呢？有两个方法：**继承Thread/实现Runnable接口**

1. 继承Thread类

   `java.lang.Thread`表示线程，**类可以继承并重写`run`方法**，就可以实现一个新线程

   （Thread类本质上也是去实现了Runnable接口）

   ```java
   public Main extends Thread{
       @Override
       public void run(){
           System.out.println("new thread");
       }
   }
   ```

   `run`方法：public修饰符、无返回值、不能抛出受检异常——run方法就是线程的main函数（类似于os中线程实际执行的主函数）

   ——继承并重写方法只是定义了新的线程，但是不会执行，**线程启动过程**：创建对象+调用`start`方法

   ```java
   public static void main(String[] args){
       Thread thread = new Main();		// 引用对象可以是父类类型
       thread.start();		// Thread类中提供的实现方法——启动线程
   }
   ```

   `start`方法：启动该线程，那么它就成为一条单独的执行流，会分配线程相关的资源、pc、栈等，会分配时间片让它执行，而执行的就是`run`方法

   如果，不调用start，而是直接调用run方法，那么**run就是一个普通的方法，而不会创建一个新的线程**，run方法是在main线程中执行

   ——通过调用方法`currentThread()`（静态方法），可以获得当前正在执行的是哪个线程

   ```java
   public static native Thread currentThread();// 获得当前正在执行的线程（是静态native方法）
   public long getId();			// 实例方法，可以用currentThread获得该线程，然后调用实例方法
   public final String getName();
   ```

   如果开启多个线程后，它们并发执行，由操作系统负责调度；如果是单核的就是并发执行；如果多核可以有并行执行——但是这个是底层os实现的，上层不关注

   ps：如果不重写run方法，那么还是能正常执行的，就是该新建的线程无任何操作（创建之后立即销毁？？？）

2. 实现Runnable接口

   使用场景：当某个类已经继承一个父类，而不能再继承一个父类Thread时，只能通过实现`java.lang.Runnable`接口来实现线程

   ```java
   public interface Runnable {
   	public abstract void run();
   }
   ```

   ——只需要实现一个方法`run`即实现了该接口（thread继承是重写了）

   ```java
   public Main implements Runnable{
       @Override
       public void run(){
           System.out.println("son thread");
       }
   }
   
   // 还是要去创建thread的方法，只不过会创建一个Runnable对象传参进去
   public static void main(String[] args){
       Thread thread = new Thread(new Main());	// 创建thread对象，并且传参，使用带参的构造方法
       thread.start();
   }
   
   
   // Thread其中一个构造方法，和上面的匹配
   public Thread(Runnable target) {
       init(null, target, "Thread-" + nextThreadNum(), 0);
   }
   ```

——可以发现，继承Thread或者实现Runnable，都需要通过new Thread来创建线程，并且需要重写/实现run方法（线程的主函数），通过start来启动线程

#### （2）线程的基本属性和方法

- id/name：线程都有一个编号（long类型）和名字（String类型）

  id是一个递增的整数（类似os中的线程创建后给`tid`）

  name默认是：`Thread-编号`（特殊：主线程，名字是main），name可以在创建Thread对象时传参`String name`进去，也可以通过`setName`方法设置

  ——name的主要作用是：方便调试

- priority：**java的线程优先级为[1, 10]，默认为5**

  可以使用的方法有：

  ```java
  public final int getPriority();
  public final void setPriority(int newPriority)
  ```

  ——这个优先级会被映射到所在实际os的实际优先级线程中，由于实际的os不同，所以映射也不同，所以存在**Java不同的优先级可能会映射到相同的优先级中**

  ——不同于os执行很关注thread的优先级，Java中不要太过于依赖优先级

- state：线程的状态

  Java中式一个枚举类型

  ```java
  public enum State {
      NEW,			// 新建状态（还未被调用执行过一次，new Thread后，到thread.start被执行前）
      RUNNABLE,		// 可执行状态（可能是等待分配时间片/正在执行）
      BLOCKED,		// 被阻塞状态
      WAITING,		// 等待状态
      TIMED_WAITING,	// 超时等待状态
      TERMINATED;		// 终止状态
  }
  
  public State getState();		// 获取线程的当前状态
  
  public final native boolean isAlive();	// 判断线程是否存活（从启动开始到终止状态之前都是true，注意是启动start之后，NEW状态下是false）
  ```

- daemon线程：**辅助线程**

  是其他线程的辅助线程，Java可能会为了实现某些功能（打印等功能，而创建实现该特定功能的线程）

  典型的daemon线程是：垃圾回收线程

  概念理解，辅助线程在对应的主线程退出的时候，就没有存在的意义。

  ——所以**程序中只存在daemon线程时，程序就会退出**。

- sleep方法：同os

  ```java
  // 休眠方法，单位是毫秒（可能有一定的偏差）
  public static native void sleep(long millis) throws InterruptException
  ```

  ——睡眠期间，线程可以被中断，被中断后会抛出异常`InterruptException`

- yield方法：同os

  ```java
  // 让方法
  public static native void yield();
  ```

- **join方法**

  ```java
  public final void join() throws InterruptException
  ```

  ——让调用该join的线程，等待指定的线程结束后，再执行下去（即使那个线程被阻塞也还是等待）

  即在当前线程（thread-m）执行过程中，执行到了另一个线程（thread-n）的join方法，就是thread-m去等待thread-n执行完成

  使用方法：

  ```java
  public class Main{			
      public static void main(String[] args){
          Thread thread = new Thread();
          thread.start();
          thread.join();		// 就是main线程去等待thread去执行完成
  	}
  }
  ——如果是在thread_1中去调用thread2.join()，那么就是thread_1去等待thread_2
  ```

  join还能限定等待时间，单位是毫秒`thread.join(long millis)`——在这个时间范围内等待，到期就不再等待继续执行下去

- 过时方法——不应该再使用

  ```java
  public final void stop();
  public final void suspend();
  public final void resume();
  ```

#### （3）共享内存和可能存在的问题——同os中的并发问题

线程有一部分是独有的内存：pc、栈，但是线程之间也存在共享内存，都可以访问和操作的对象——堆

1. 竞态条件：多线程竞争同一个资源，如果**对资源的访问顺序敏感**，就存在竞态条件

   ```java
   public class CounterThread extends Thread {
       private static int counter = 0;			// 静态变量——共享内存
       @Override
       public void run() {
           for(int i = 0; i < 1000; i++) {	// thread的main函数执行的就是循环加操作
           	counter++;
           }
       }
       public static void main(String[] args) throws InterruptedException {
           int num = 1000;
           Thread[] threads = new Thread[num];		// 创建1000个线程
           for(int i = 0; i < num; i++) {			// 启动
               threads[i] = new CounterThread();
               threads[i].start();
           }
           for(int i = 0; i < num; i++) {		// 等待完成
           	threads[i].join();
           }
           System.out.println(counter);
       }
   }
   ```

   ——按照期待来说，最后counter的值为1,000,000，但是一般不能到这个值。主要是这1000个线程是**交替执行的**，所以thread-1在获得count=100之后被切换到thread-2，此时count值没有发生变化，而thread-2也获得了count=100，那么thread-1执行完一个循环=101，thread-2执行完一个循环=101，所以两个循环只增加了1

   但是，线程的循环执行是必须的（时间片轮转），我们需要从其他地方限制‘

   ——**count++ => count = count + 1**该操作不是原子操作，这个语句主要的流程是：去共享内存中获得count的值-->进行加1操作-->然后将count的值写回内存，在这个读取和写回的过程中，共享变量的count是没有发生变化的，如果此时线程切换就会出现竞态

   ——如果将该操作，变成原子操作，那么在执行该步骤的时候不可被切换，那么该操作就是有效的

   如何变成原子操作呢？

   - 使用`synchronized`关键字
   - 使用显式锁
   - 使用原子变量

2. 内存可见性：读操作和写操作发生在不同的线程中，**无法确保读操作能够适时的看到写入的值**，有时甚至一致看不到

   ```java
   public class VisibilityDemo {
       private static boolean shutdown = false;
       static class HelloThread extends Thread {
           @Override
           public void run() {	// thread的main函数——死循环等待shutdown变成true
               while(!shutdown){		
               // do nothing
               }
               System.out.println("exit hello");
           }
       }
       public static void main(String[] args) throws InterruptedException {
           new HelloThread().start();
           Thread.sleep(1000);		// main线程休眠1s后将标志位赋值为true
           shutdown = true;
           System.out.println("exit main");
       }
   }
   ```

   ——在实际运行中，发现程序不会终止，说明子线程一直没有等到shutdown变成true，而实际上在main线程休眠结束后已经修改了，说明**子线程对共享变量值的修改不可见**，主要原因是内存模型的特点造成的：

   <img src="https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMzE1MTAyODU5MDI2?x-oss-process=image/format,png" style="zoom: 80%;" >

   实际上对每个变量（私有变量、共享变量）的操作都只是发生在本地内存区中，本地内存区是不可共享的，共享变量修改完成后，会更新到共享内存区，而其他读该共享变量后进行对应的修改——问题就在于：写线程何时push值，读线程何时pull值，这两个地方造成了不可见性

   解决方法：

   - 使用`volatile`关键字——轻量级
   - 使用`synchronized`关键字——性能问题，效率低易阻塞
   - 使用显式锁同步——性能问题，效率低易阻塞

#### （4）线程的优点及成本

优点是：

- 充分利用多cpu的计算能力——在多cpu环境下
- 充分利用各种资源：cpu、硬盘、网络这些是可以并行工作的，即在等待网络的时候，可以让其他线程去使用cpu
- 在GUI，保持程序的响应性，界面和后台任务是不同的线程，即后台在执行的时候，界面不会完全停止响应，而是时间片轮转时而响应。而如果只是单线程就会出现顺序执行
- 简化建模及IO处理，对每个用户请求使用一个单独的线程进行处理

成本在于：

- 上下文切换是有代价的：时间上 和 cpu缓存上

所以需要权衡优点和成本：比如是多任务下，那么切换为多线程是合理的，而如果只是简单的操作，例如上面的count++，那么单线程就足够了；如果执行的任务是cpu密集型的（就是一直在使用cpu进行计算，而不会去等待资源啥的，那么创建多个线程反而会拖慢执行）

### 2. 关键词synchronized：同步锁

#### （1）用法

可以用来**修饰实例方法、静态方法、代码块**，不针对变量。

举例：

```java
public class Counter {			// 计数类
    private int count;
    public synchronized void incr(){		// ++方法是通过synchronized修饰的
    	count ++;
    }
    public synchronized int getCount() {		// 获取值的方法也是通过synchronized修饰的
    	return count;
    }
}
```

```java
public class CounterThread extends Thread {
    Counter counter;			// 线程的共享对象counter
    public CounterThread(Counter counter) {
    	this.counter = counter;
    }
    @Override
    public void run() {			// 线程的main方法
        for(int i = 0; i < 1000; i++) {		
        counter.incr();
        }
    }
    public static void main(String[] args) throws InterruptedException {
        int num = 1000;
        Counter counter = new Counter();
        Thread[] threads = new Thread[num];
        for(int i = 0; i < num; i++) {			// 创建1000个线程
            threads[i] = new CounterThread(counter);
            threads[i].start();
        }
        for (int i = 0; i < num; i++) {		// 运行该1000个线程
        	threads[i].join();
        }
        System.out.println(counter.getCount());
    }
}
```

——此时的count输出就是预期的1,000,000

#### （2）原理

**synchronized保护的是对象，而不是方法**，即可以同时调用同一个方法，只要它们的对象不同就可以。而对于多线程对同一个对象的操作——**synchronized保证只能有一个线程去执行该对象的方法操作**，该线程执行完成该方法后，才可能让其他线程去执行

synchronized实例方法保护的实际上是**this**，this含有一个锁和等待队列，尝试去获得锁，得不到该线程就会挂在等待队列上，当前线程的状态变成`BLOCKED`——操作和os一致，不再详细解释

——synchronized保护的是对象而不是代码，**只要访问的是，同一个对象的synchronized方法，那么不同的代码也会要求顺序执行**，eg：两个线程一个执行`incr`，另一个执行`getCount`，那么还是不能同步执行，需要一个执行完，才能调度另一个执行。

ps：synchronized方法不能阻止非synchronized方法被同时执行，即**synchronized只能保证有这修饰符的方法们之间的顺序执行，而对非修饰符的方法无效**，eg：count--的decr方法是非synchronized，那么会和incr方法同时执行

——**需要将所有被保护的变量相关的方法上均加上synchronized**

#### （3）静态方法

前面讲的是，synchronized修饰实例方法，而它也能够修饰静态方法

修饰静态方法和修饰实例方法的对象主体不一样：**实例方法保护的是当前实例对象this**，**静态方法保护的是类对象，即xxx.class**——实例对象、类对象都有一个锁和等待队列

所以，如果两个线程一个执行synchronized修饰的静态方法，另一个执行synchronized修饰的实例方法，是可以同时执行的

#### （4）代码块

synchronized可以修饰代码块，形式如下：

```java
public class Counter {
    private int count;
    public void incr(){
        synchronized(this){
        	count ++;
        }
    }
}
```

`synchronized(xxx){....}`——括号里面的就是保护的对象，实例方法就是this，静态方法就是`Counter.class`，大括号里面的就是同步执行的代码。

——所以，synchronized修饰的静态方法和实例方法都可以转换为对应的代码块

**synchronized的对象可以是任意对象，因为所有对象都有一个锁和对应的一个队列**

#### （5）进一步理解

- 可重入性：针对锁

  synchronized是可重入的，即一个线程在获得锁之后，如果执行过程中又需要该锁，那么还是可以获得该锁的——只针对拥有该锁的线程（同zephyr里面的锁机制）

  实现机制：会记录该锁拥有的线程和进入的次数。

  所以流程是：当执行到受保护的方法时，先尝试获取锁，如果没有其他线程持有锁，那么就获得了该锁，锁会记录持有的线程+进入的次数（1）；如果有线程持有该锁，如果该线程就是当前线程，那么更新进入次数（++），如果该线程不是当前线程，那么将当前线程挂到等待队列上。

  ——注意：每次释放都会减少进入次数，**只有次数减少到0，才会释放该锁**

- 内存可见性

  前面描述的都是竞态的情况。而对于内存可见性也是synchronized能保证的

  **释放锁时，会将所有的写入写回内存；获得锁后，会从内存中读最新数据**

  但是synchronized主要就是来避免竞态情况的，而对于**实现内存可见性代价太大**

  ——轻量级的方法是：**给变量添加修饰符volatile**

  ```java
  public class Switcher {
      private volatile boolean on;		// volatile是添加在变量上的，而不是方法上
      public boolean isOn() {
      	return on;
      }
      public void setOn(boolean on) {
      	this.on = on;
      }
  }
  ```

  ——Java对于volatile修饰的变量，会在操作该变量的时候插入特殊的指令，**保证读取到内存最新值**

- 死锁

  锁的通病，死锁问题，即两个及以上的线程持有锁，且都在等待对方的锁

  如何解决：死锁预防、死锁避免、死锁检测

  java提供了：显式锁接口Lock，它能够尝试获取锁（tryLock）和带超时等待的获取锁的方法——获取不到锁的时候会释放持有的锁，然后再次申请该锁，或者直接放弃申请该锁

  Java不会处理死锁，但是又工具会报告死锁**jstack**

#### （6）同步锁修饰的容器类

`Collection`存在子类，通过对每个方法都修饰了synchronized，那么就是**线程安全的同步容器**——这边的安全是针对同一个容器对象

```java
public static <T> Collection<T> synchronizedCollection(Collection<T> c);
public static <T> List<T> synchronizedList(List<T> list);
public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m);
```

但是还是要注意如下问题：

- 复合操作

  可能每一步都是线程安全的，但是合起来确不是

- 伪同步

  注意要同步谁，就指定谁，如果是修饰方法，那么默认的就是指定this对象；如果是修饰代码块，可以指定对象。

- 迭代

  同步容器并没有实现遍历和修改同时发生，即如果遍历的时候进行修改，会抛出异常，需要在遍历的时候给整个容器对象加锁，限制只能遍历结束之后才能修改

- 并发容器

  同步容器的性能较差，当并发访问量大的时候性能不好。

  这时候可以选择用并发的容器类：CopyOnWriteArrayList；ConcurrentHashMap；ConcurrentLinkedQueue；ConcurrentSkipListSet

  ——性能高，没有使用synchronized关键字，没有迭代等问题

### 3. 线程的基本协作机制

线程之间有竞争，也有协作，对应的关键字是：`wait/notify`

多线程的协作本质上就是：条件不满足时**主动等待**；条件满足时**被唤醒**继续执行

#### （1）协作的场景

- 生产者/消费者模式：生产者和消费者通过**共享队列**协作，生产者将产生的数据或任务放到队列上，消费者从队列上获取数据或任务。

  那么队列满时，生产者需要等待消费者消费；队列为空时，消费者需要等待生产者生产

- 同时开始：多线程同时开始，多在模拟仿真程序中

- 等待结束/主从模式：主线程将任务分解为多个子任务，给每个子任务分配线程，那么主线程需要等待所有的子线程执行完成后才能继续执行

- 异步调用：主线程去调用子线程后，让子线程去执行，且直接返回到主线程继续执行，返回值为**future对象**，它在未来某个时间点能够得到子线程的解

- 集合点：多个线程分别去执行，执行到某个点就等待其他线程执行到该集合点，直到所有线程均到了，就开始执行下面的内容，eg：并行迭代计算中，每个线程负责一部分计算，然后在集合点等待其他线程完成，所有线程到齐后，交换数据和计算结果，再进行下一次迭代（类似于GPU的计算）

#### （2）wait/notify

wait和notify是根父类Object提供的两个native方法

```java
public final native void wait(long timeout) throws InterruptedException;// 设置超时时间，超时了就不等待了，如果参数=0，那么就无限期等待；等待期间可以被中断，如果被中断会抛出异常

public final native void notify();		// 唤醒其中一个，看底层的os，可能存在随机性
public final native void notifyAll();	// 唤醒所有在条件队列上的线程
```

ps：wait的抛出中断异常，也不算是出现bug，而是表示该wait方法是可中断的，那么该方法会对中断异常进行处理（而不同于中断程序自己抛出中断异常）

wait/notify的工作原理：

前缀知识：每个对象都有锁和锁对应的等待队列，此外**还存在一个队列——条件队列，主要用来协程间的协作**

调用wait之后：会将当前线程会**释放当前所拥有的锁**，并且**挂到该对象（this）的条件队列**上并阻塞，表示要等待一个条件才能继续执行——**该条件自身已经无法改变了，需要其他线程改变**（注意这边是存在共享变量的）

其他线程改变条件后，会在该线程中**调用该对象的notify方法**，从而唤醒对象的条件队列上的某一个线程

注意：wait/notify必然涉及到对共享变量的访问和修改操作，是会存在竞态问题，所以相关的代码都需要synchronized修饰——规定了**wait/notify方法必须在synchronized代码块中被被调用**，即先获取锁才能执行wait/notify，否则会抛出异常`java.lang.IllegalMonitor-StateException`

下面分析wait的整个过程：

1. 看某个资源是否存在，如果存在就执行下去，不在就进入处理程序中

   一般都是形如：`while(xxx){ this.wait();}`——while中存放的都是不满足的情况

   ——一般不用if语句，因为wait返回之后直接执行到了下面，可能又不满足上面的语句了，所以设置为while最为安全

2. 将**当前线程**加入条件等待队列，**释放持有的锁**，变成阻塞状态`WAITING`/`TIMED_WAITING`

   可能存在多个线程都会去调用该对象的该方法，导致被挂起，从而条件队列上存在多个线程

3. 其他线程开始执行，直到执行到某个变量条件修改后，**使得步骤1条件变得满足**（如果还是不满足，那么释放没有意义，还是会被阻塞），然后该线程中显式调用**该对象(this).notifyAll()**

   这边多选择用`notifyAll`而不是`notify`，唤醒所有线程更为安全，而如果只唤醒一个线程不确定性太多

   ps：**notify/notifyAll只会移除条件队列中等待的线程，而不会释放锁**，即notify会将synchronized代码块中的执行完成后，才开放抢占

4. 如果等待的线程有设置超时等待，超时了还未被唤醒，那么该线程主动清醒，继续执行下去不再等待；如果在等待过程中，其他线程调用notify将它们唤醒，它们会从条件队列中被移除，**重新竞争对象锁，因为synchronized机制**

   - 如果能够获得锁，那么状态就会变成`RUUABLE`，并且从wait下面开始返回
   - 如果无法获得锁，那么就会从条件队列中转移到锁的等待队列，线程状态变成`BLOCKED`

总结：wait/notify是**围绕一个共享的条件变量进行协作的**。该条件变量是**程序自行维护的**，当条件不成立，那么当前线程主动将自己挂起（而前面的锁是被动挂起的，实际上就是内置了一个block函数；主动就是如果没有写wait，实际上并没有后面这些事了）；另一个线程在后面执行过程中修改了条件变量，主动去调用了notify；然后**前一个线程返回后需要重新检查条件变量**（while循环去检查）

存在的问题：wait是让线程休眠，挂在等待队列上，而只有这一个队列，不存在其他的等待队列。

典型的情况：下面的生产者消费者问题

改进：使用ReentrantLock的`condition`，有一个condition就对应产生一个队列，然后将要挂起的生产者加入到一个condition的等待队列上；将要挂起的消费者加入到另一个condition的等待队列上。

#### （3）举例

- 生产者消费者问题

  ```java
  static class MyBlockingQueue<E> {			// 长度有限的队列，可以在上面取数据or放数据
      private Queue<E> queue = null;
      private int limit;
      public MyBlockingQueue(int limit) {
          this.limit = limit;
          queue = new ArrayDeque<>(limit);
      }
      public synchronized void put(E e) throws InterruptedException {	// 给生产者调用的
          while(queue.size() == limit) {		// 队列满了，就等待
          	wait();
          }
          queue.add(e);
          notifyAll();
      }
      public synchronized E take() throws InterruptedException {	// 给消费者调用的
          while(queue.isEmpty()) {		// 队列空了就等待
              wait();
          }
          E e = queue.poll();
          notifyAll();
          return e;
      }
  }
  ```

  ——可以发现，生产者线程被挂起是队列满了，而消费者被挂起是队列空了。但都是挂在MyBlockingQueue实例对象上的——它们的等待条件不同，但是都挂在同一个队列上，所以如果等待条件不同，但是又挂在同一个条件队列上，那么需要使用**notifyAll**去唤醒所有线程，防止使用notify唤醒不该唤醒的。

  所以也说明了wait/notify的局限性：**只能有一个条件等待队列**（显式锁和条件可以解决这个问题）

- 同时开始：让多个线程去等待同一个变量的值变化，而让一个线程去改变该值，改变之后将所有在等待的线程notifyAll，那么就是同时开始

- 等待结束

  ```java
  // thread.join()的原理——
  while (isAlive()) {		// 判断要等待的线程（this）是否还是存活状态——带超时等待的；如果存活那么就挂起当前线程（就是调用该方法的线程）
      long delay = millis - now;
      if (delay <= 0) {
          break;
      }
      wait(delay);		// 如果没有到期就挂起
      now = System.currentTimeMillis() - base;
  }
  ```

  而这个是由该线程在执行完成之后，会主动调用notify去唤醒线程的

  如果需要等待多个线程，那么可以换成一个计数共享变量来记录要等待的线程数，当值>0时就挂起等待；当一个线程执行完成后去--，然后唤醒该线程，该线程会去判断该值是否==0，否则就继续挂起；是就表示等待结束，该线程可以执行下去了

  **Java中有一个专门的同步类，CountDownLatch——就是该原理**

- 异步结果：没看懂，可能是异步这个概念还没了解

  表示异步结果的接口Future和实现类FutureTask；用于执行异步任务的接口Executor，以及有更多功能的子接口ExecutorService； 用于创建Executor和ExecutorService的工厂方法类Executors。

- 集合点：分头行动，到某个特定的点后需要到齐所有线程，交换数据后才能再次分头行动

  ```java
  public class AssemblePoint {
      private int n;
      public AssemblePoint(int n) {
      	this.n = n;
      }
      public synchronized void await() throws InterruptedException {
          if(n > 0) {		// 说明有线程未到齐
          	n--;
              if(n == 0) {		// 如果减去自己后为0，那么已经到齐，将所有线程唤醒
                  notifyAll();
              } 
              else {	// 如果减去后不为0，那么未到齐，挂起等待，直到为0后被唤醒——唤醒后还是会去判断是否等于0——再次验证
                  while(n != 0) {
                      wait();
                  }
              }
         	}
      }
  }
  ```

  Java中有一个专门的同步工具类CyclicBarrier可以替代它

### 4. 线程的中断

#### （1）why需要线程取消/关闭

一般流程下：线程通过`thread.start();`的方法启动线程，然后时间片轮转后开始执行`run()`方法，然后该方法运行结束后**线程正常退出**。

但是还是有其他场景需要手动结束线程

- 某些线程的执行是死循环，当达到某个条件后需要正常关闭线程

  eg：生产者/消费者模式下，两个线程都是死循环，需要我们来自行关闭线程

- 在GUI用户界面程序，线程是用户启动的，eg：下载线程。但是下载线程可能会被用户提前取消该任务，即取消该线程，我们需要能响应该请求

- 我们需要在限定时间内得到某个结果，如果得不到，就取消该任务，也即取消该任务

- 启动多个线程去做同一件事，当事件完成，那么需要取消其他线程

  eg：多线程抢票

#### （2）how取消/关闭

可以使用`stop()`关闭，但是标记为过时，不能使用。

Java中停止一个线程的机制就是**中断**，但是**中断并不是强制终止一个线程，而是一种协作机制，是给线程一个取消信号，然后由线程来决定如何以及何时退出**（所以在run中我们需要在实现的时候，考虑意外终止中断线程的判断操作）

Thread类中

```java
public boolean isInterrupted();		// 实例方法：判断该线程是否在中断状态中：true——在中断状态

public void interrupt();		// 实例方法：设置中断
// ——中断对应的线程：thread.interrupt()

public static boolean interrupted();	// 静态方法：设置中断——就是通知该线程中断了，需要提前结束程序
// ——判断当前线程是否处于中断当中（true），并且清空中断标志位，即原来如果是true，会被赋值为false，如果是false那么还是false
```

——**每个线程都有一个中断标志位**，true：表示中断；false：表示未中断

`interrupt()`的操作相关的受线程状态和IO操作有关（IO本身也是外部中断，IO操作的影响和具体的IO及操作系统相关，这边不关注）。

线程的状态有：

- RUNNABLE：线程可执行状态，等待cpu调度 or 已经在运行状态
- WAITING/TIMED_WAITING：线程在等待某个条件或者超时等待
- BLOCKED：线程在等待锁，进入临界区
- NEW/TERMINATED：线程未启动 or 线程已结束

1. 对于RUNNABLE状态的线程

   如果在该状态下，且没有在执行IO操作，那么`interrupt()`调用之后只是会去设置中断标志位

   线程在运行过程中，需要在合适的位置去检查中断标志位，看是否需要提前终止线程的run方法

   eg：run的主要代码是一个循环，那么可以在循环开始处进行检查

   ```java
   public class InterruptRunnableDemo extends Thread {
       @Override
       public void run() {
           while(!Thread.currentThread().isInterrupted()) {
               // 循环内代码
           }
           System.out.println("done ");
       }
   }
   ```

2. 对于waiting/timed_waiting状态的线程

   线程用join/wait/sleep会使线程进入waiting/timed_waiting状态。如果在此状态下，**对线程调用interrupt()使得该线程抛出InterruptedException异常**（在适时的时候，需要接受该异常并且做出相应处理）

   但是：**抛出异常后，中断标志位会被清空，即被设置为false**

   ```java
   Thread t = new Thread (){
       @Override
       public void run() {
           try {
           	Thread.sleep(1000);		// 线程休眠——在休眠的时候，被main线程提示取消，那么抛出异常
           } catch (InterruptedException e) {
           	System.out.println(isInterrupted());
           }
       }
   };
   t.start();		// 启动线程t
   try {
   	Thread.sleep(100);		// 当前线程main休眠
   } catch (InterruptedException e) {}
   t.interrupt();		// 由于main线程休眠时间短，那么就会去给线程t设置中断标志位，提示线程取消
   ```

   ```java
   public static void sleep(long millis) throws InterruptedException{....}
   // ——该方法可能会抛出异常
   ```

   **InterruptException是一个受检异常，线程必须处理**，所以不能不管

   而处理异常的基本思路是：**如果知道该如何处理，那么就处理；如果不知道如何处理，那么向上传递**——如果catch异常后需要进行处理，而不是忽略。

   如果catch到了InterruptException，一般就是通知该线程要提前结束了，该线程通常有两种处理方式：

   - 继续向上传递该异常，那么该方法也变成了一个可中断的方法，需要调用者进行处理

   - 不能再向上传递，eg：Thread的run方法，由于它方法是固定的，不能再向上抛出异常了，那么就只能catch该异常进行处理

     ——该操作之后，一般还是需要调用interrupt方法再去设置中断标志位，从而让其他代码知道已经发生了中断

     eg：

     ```java
     public class InterruptWaitingDemo extends Thread {
         @Override
         public void run() {
             while(!Thread.currentThread().isInterrupted()) {	// 如果当前线程在中断状态
                 try {
                 	Thread.sleep(2000);		// 尝试执行sleep
                 } catch(InterruptedException e) {		// 如果在执行sleep的时候被传递了终止信息
                     // ... 针对终止的相关操作
                 	Thread.currentThread().interrupt();	// 重新设置中断标志位——让其他代码知道该线程发生中断
                 }
             }
             System.out.println(isInterrupted());
         }
         // 其他代码
     }
     ```

3. 对于blocked状态的线程

   说明该线程是在等待锁，调用`interrupt()`**只是会去设置线程的中断标志位**——而没有任何操作，线程还是处于blocked状态

   ——**`interrupt()`不会使在一个等待锁的线程真正中断**

   ```java
   public class InterruptSynchronizedDemo {
       private static Object lock = new Object();
       private static class A extends Thread {
           @Override
           public void run() {
               synchronized (lock) {			// 获取锁的情况下，去执行
                   while (!Thread.currentThread().isInterrupted()) {
                   }	// 如果该线程（当前线程）没有在中断状态，那么就执行循环；如果在中断状态，那么就退出
               }
               System.out.println("exit");		// 线程即将退出
           }
       }
       public static void test() throws InterruptedException {
           synchronized (lock) {			// 获取锁的情况下，去执行
           A a = new A();		// 创建一个线程
           a.start();		// 启动
           Thread.sleep(1000);	// main线程休眠
           a.interrupt();		// 休眠结束后，设置线程a为中断状态
           a.join();		// 等待线程a执行完成
           }
       }
       public static void main(String[] args) throws InterruptedException {
       	test();
       }
   }	
   ```

   执行过程中，main线程先去获取锁，然后启动线程a，尝试获取该锁，但是该锁被main线程占有，那么会加入锁的等待状态，main线程继续执行，main线程通知线程a的取消信号——只是设置线程的中断状态，但是线程a依旧在锁的等待状态中。

   ——**`synchronized`关键字中，线程去等待锁的时候，不去响应中断请求**——synchronized的局限性，如果需要解决该局限性问题，应该使用显式锁

4. 对于new/terminated状态的线程

   在该状态下，调用`interrupt()`是没有作用的

——所以，`interrupt`不会真正中断线程（和os不一样），**只是一种协作机制，如果不了解线程在做什么，不应该去调用interrupt**

对于那些对线程提供服务的程序来说，应该封装取消线程操作，给调用者封装好的方法，而不是直接让它调用interrupt

所以对于线程的实现者，应该封装好线程的终止方法，并文档描述行为；

作为调用者，应该使用封装好的方法，而不是直接使用interrupt，导致不符合预期

eg：

Future接口：`boolean cancel(boolean mayInterruptIfRunning);`

ExecutorService接口：`void shutdown();List<Runnable> shutdownNow();`

## （二）Java的并发工具包

Java中存在一套并发工具包，位于`java.util.concurrent`，包括易用且高性能的并发开发工具

### 1. 原子变量及其原理

前提知识：synchronized关键字，能够保证修饰的方法/代码块是原子性操作。但是对于简单的操作，用synchronized关键字成本太高，需要获取锁，最后释放锁；如果获取不到还需要等待锁，并且涉及到了上下文切换。

——这个可以使用：**原子变量代替**（即该变量能够保证该变量的操作是原子性的，不会被打断）。Java并发包中提供了基本原子变量类型：

- AtomicBoolean：原子Boolean类型，常用来在程序中表示一个标志位；

- AtomicInteger：原子Integer类型；

- AtomicLong：原子Long类型，常用来在程序中生成唯一序列号；

- AtomicReference：**原子引用类型**，用来以原子方式更新复杂类型

  ——从而解决了之前只能够对一个共享变量进行原子操作，将多个共享变量组合成一个对象，从而调用`AtomicReference`来实现原子性

还有一些其他类

- 如针对数组类型的类，AtomicLongArray、AtomicReferenceArray，
- 以原子方式更新对象中的字段的类，如AtomicIntegerFieldUpdater、AtomicReferenceFieldUpdater等。

Java 8增加了几个类，在高并发统计汇总的场景中更为适合，包括LongAdder、LongAccumulator、Double-Adder和DoubleAccumulator，

#### （1）AtomicInteger

基本原子变量拿`AtomicInteger`举例

基本用法：

```java
public AtomicInteger(int initialValue);	// 构造方法：设置初始化值
public AtomicInteger();		// 构造方法：无参数，默认初始化值为0
```

```java
public final int get();		// 获得原子变量当前的值
public final void set(int newValue);		// 设置原子变量的值
```

——这些都是普通的方法。

原子变量，体现在它包含了一些**以原子方式实现组合操作**的方法：（即多个操作，都能保证原子性）

```java
//以原子方式获取旧值并设置新值
public final int getAndSet(int newValue);
//以原子方式获取旧值并给当前值加1
public final int getAndIncrement();	// 同i++
//以原子方式获取旧值并给当前值减1
public final int getAndDecrement();	// 同i--
//以原子方式获取旧值并给当前值加delta
public final int getAndAdd(int delta);
//以原子方式给当前值加1并获取新值
public final int incrementAndGet();		// 同++i
//以原子方式给当前值减1并获取新值
public final int decrementAndGet();		// 同--i
//以原子方式给当前值加delta并获取新值
public final int addAndGet(int delta);
```

——本质上都是去依赖了一个public方法：**CAS方法**

```java
public final boolean compareAndSet(int expect, int update);		// CAS方法
```

如果当前值等于`expect`，那么就将该值设置为`update`，并且返回true；如果值不等于`expect`，那么不设置，直接返回false

——乐观锁（认为发生冲突的可能性比较小），尝试去修改变量，修改不了就返回

AtomicInteger可以在程序中作为一个计数器，多个线程并发去更新，并且能够保证值的正确性

```java
public class AtomicIntegerDemo {
    private static AtomicInteger counter = new AtomicInteger(0);
    static class Visitor extends Thread {			// 静态内部类
        @Override
        public void run() {
            for(int i = 0; i < 1000; i++) {
            	counter.incrementAndGet();		// 原子++i
            }
        }
    }
    public static void main(String[] args) throws InterruptedException {
        int num = 1000;
        Thread[] threads = new Thread[num];
        for(int i = 0; i < num; i++) {			// 创建1000个线程
            threads[i] = new Visitor();
            threads[i].start();
        }
        for(int i = 0; i < num; i++) {			// main线程等待所有1000个线程执行完成再执行下去
        	threads[i].join();
        }
        System.out.println(counter.get());
    }
}
```

基本原理：

AtomicInteger的实现：

```java
private volatile int value;		// 主要内部成员
```

——声明中带有：volatile，能够保证内存可见性

模拟其中一个原子方法：

```java
public final int incrementAndGet() {
    for(;;) {
        int current = get();		// 获得当前的值
        int next = current + 1;		// 预计++之后的值
        if(compareAndSet(current, next))	// 尝试去修改该值，如果修改成功那么就返回成功的值；如果修改失败，继续循环继续尝试++
        	return next;
    }
}
```

——代码主体是个死循环，先获取当前值current，计算期望的值next，然后调用CAS方法进行更新，如果更新没有成功，说明value被别的线程改了，则再去取最新值并尝试更新直到成功为止。

CAS的实现原理：

```java
public final boolean compareAndSet(int expect, int update) {
	return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}

private static final Unsafe unsafe = Unsafe.getUnsafe();
```

——调用了unsafe的`compareAndSwapInt`方法，而unsafe是sun.misc.Unsafe类的实例对象

unsafe从字面意思表示不安全，一般应用程序不能直接使用，但是大部分的计算机**底层硬件已经能够直接实现CAS**，具体原理可以参考[Java额外知识](http://note.youdao.com/noteshare?id=9f8ab6eaf048c06b8c823ebe1c7f0b8c&sub=38FCCF96A0C74660A8B19E1140BA449E)。对于上层来说，将CAS作为基本操作直接使用即可

synchronized锁是一个**悲观锁**：认为会发生并发冲突，所以要去先去获取锁，得到锁后才能更新；是**阻塞式算法**，如果得不到锁，那么就加入等待队列——有获取锁、释放锁，上下文切换的代价，阻塞的代价

CAS是**乐观锁**：认为冲突比较小，尝试更新，进行冲突检测，如果的确发生冲突，那么继续尝试更新；是**非阻塞式算法**，冲突时就重试——无阻塞、无上下文切换，**对于简单的操作，在高低并发的情况下，性能都会高很多**。

基本数据类型对应的原子变量的原理比较简单，但是对于复杂的数据结构，非阻塞的方式比较难实现和理解，但是Java已经提供了内置函数：

- ConcurrentLinkedQueue和ConcurrentLinkedDeque：非阻塞并发队列；
- ConcurrentSkipListMap和ConcurrentSkipListSet：非阻塞并发Map和Set；

实现锁：

**CAS可以用来实现非阻塞的乐观锁，eg：AtomicInteger，也可以用来实现阻塞式的悲观锁。**实际上，Java的并发包中的乐观锁是用CAS实现的，阻塞式工具、容器、算法也是基于CAS的，eg：Lock。

自行实现的显式锁：（用原子变量实现的）

```java
public class MyLock {
    private AtomicInteger status = new AtomicInteger(0);
    public void lock() {
        while(! status.compareAndSet(0, 1)) {		// 尝试获取锁，要求当前值为0，那么能得到锁，并且设置为1——否则，就挂起
            Thread.yield();
        }
    }
    public void unlock() {			// 最后执行完成释放锁，将1变成0，表示释放
        status.compareAndSet(1, 0);
    }
}
```

——和普通的锁还是存在区别的，无等待队列，也没有在释放锁的时候唤醒线程，这边是处于一个忙等待的状态（如果是按照优先级调度，容易出现死锁情况，低优先级的线程获得锁，但是`thread.yield()`一直不让它执行，那么高优先级一直等不到锁，而低优先级一直持有锁却得不到执行）

实际开发中应该使用Java并发包中的类，如ReentrantLock。

CAS的问题：

- ABA问题：即当前值为A，并发执行中，另一个线程将A修改成B，之后又修改为A。而对当前线程来说**无法分辨该值中间发生过变化**。

  ——如果这个执行顺序是存在问题的，可以加上时间戳，用另一个原子类：AtomicStampedReference。只有时间戳和值都相同时才能进行修改。

  ```java
  public boolean compareAndSet(
      V expectedReference, V newReference, int expectedStamp, int newStamp)
  ```

  ——需要修改两个变量，在原理上，内部AtomicStampedReference会将两个值**组合为一个对象**，修改的是一个值

  ```java
  public boolean compareAndSet(V expectedReference, V newReference,
                               int expectedStamp, int newStamp) {
      Pair<V> current = pair;
      return
          expectedReference == current.reference &&
          expectedStamp == current.stamp &&
          ((newReference == current.reference &&
            newStamp == current.stamp) ||
           casPair(current, Pair.of(newReference, newStamp)));
  }
  
  private static class Pair<T> {
      final T reference;
      final int stamp;
      private Pair(T reference, int stamp) {
          this.reference = reference;
          this.stamp = stamp;
      }
      static <T> Pair<T> of(T reference, int stamp) {
          return new Pair<T>(reference, stamp);
      }
  }
  ```

  ——通过将两个变量组合成一个新的实例变量，然后通过比较该实例变量的值，然后组合修改该实例变量

总结：原子变量背后的原理是CAS，对于并发计数和产生序列号这种比较简单的操作，应该使用原子变量。**CAS是Java并发包的基础**，可以实现高效、乐观、非阻塞式算法，也是并发包中锁、同步工具和各种容器的基础（即CAS也可以用来实现阻塞的、悲观的算法）

### 2. 显式锁

由于：synchronized存在一些局限性，比如不能响应中断等。所以在这种情况下可以用显式锁进行替换

Java并发包中的显式锁接口和类位于包`java.util.concurrent.locks`下，主要接口和类有：

- 锁接口Lock，主要实现类是`ReentrantLock`：可重入锁
- 读写锁接口ReadWriteLock，主要实现类是`ReentrantReadWriteLock`

#### （1）接口Lock

接口定义：

```java
public interface Lock {
    void lock();
    void unlock();		// ——普通的锁获取、释放的方法，获取不到锁会阻塞；释放锁后会通知所有等待的线程
    
    void lockInterruptibly() throws InterruptedException;	// 加上了可响应中断的功能——如果被其他线程中断，会抛出异常
    boolean tryLock();		// 尝试获取锁，立即返回不阻塞，获取到就返回true；获取不到就返回false
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;	// 尝试获取锁，在限定时间内阻塞等待；在等待的过程中能够响应中断，如果在等待中发生中断就抛出异常
    Condition newCondition();		// 新建一个条件，一个lock可以关联多个条件
}
```

——所以，Lock相较于synchronized，能够支持基本的锁，也能够支持可中断锁、限定时间阻塞锁、非阻塞方式获取锁，更为灵活

#### （2）可重入锁ReentrantLock

实现了上面的lock接口。它的基本用法lock/unlock和synchronized一样：可重入；可以解决竞态条件问题；可以保证内存可见性。

```java
public ReentrantLock();		// 默认构造方法
public ReentrantLock(boolean fair);		// 带参构造方法
```

ps：对于下面的带参构造方法，参数fair表示是否保证公平：等待时间最长的线程优先获得锁。保证公平会影响锁的性能，所以默认为false，不保证（synchronized也是不保证公平的）

**使用显式锁，要lock/unlock成对使用**，一般而言，应该将lock之后的代码包装到try语句内，在finally语句内释放锁。

eg：

```java
public class Counter {
    private final Lock lock = new ReentrantLock();
    private volatile int count;
    public void incr() {
        lock.lock();		// 获得锁
        try {			// 获得锁后的具体操作
            count++;
        } finally {
            lock.unlock();		// 最后释放锁，放在finally中保证会执行，即使抛出异常
        }
    }
    public int getCount() {
        return count;
    }
}
```

tryLock能够避免死锁：

**持有一个锁，然后尝试去获取另一个锁而得不到时，会释放已有的锁，然后自己再去尝试获取所有锁**

ReentrantLock和synchronized的区别：

|              | ReentrantLock                        | synchronized             |
| ------------ | ------------------------------------ | ------------------------ |
| 锁的实现机制 | 基于CAS和AQS                         | 监视器模式               |
| 灵活性       | 支持中断响应、尝试获取锁、超时等待锁 | 获取不到就阻塞，直到等到 |
| 释放         | **显式调用unlock释放锁**             | 代码块执行完成自动释放   |
| 锁类型       | 可选择：公平锁、非公平锁             | 非公平锁                 |
| 条件队列     | 可关联多个条件队列                   | 关联一个条件队列         |
| 可重入性     | 可重入                               | 可重                     |



