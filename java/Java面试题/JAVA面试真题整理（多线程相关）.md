# Java并发相关

## 1. Java的内存区域



## 2. 为何要引入多线程

- 提高并行

  为了利用多处理器，所以需要用多线程，使得多核CPU都能够运行

- 提高并发

  对于IO密集操作，在等待期间可以进行其他操作。

但是，存在的问题：内存可见性、上下文切换、死锁、进程间同步的问题。

## 3. 死锁

产生死锁的条件：

- 互斥条件
- 请求和保持条件
- 不剥夺条件
- 循环等待条件

破坏其中之一个条件就可以了。

## 4. sleep和wait的区别

两者：都可以暂停线程的执行

最主要区别：**sleep是不会释放锁的；wait是会释放锁的**

- sleep用来暂停执行；wait用来进程间交互和通信（和notify配合使用）
- wait不会主动唤醒，要notify才能唤醒该线程；而sleep会自动苏醒，超时等待

## 5. synchronized关键字⭐

无锁 -> 自旋锁 -> 轻量级锁 -> 重量级锁



## 6. 单例模式

```java
public Singleton{
    private volatile static Singleton instance;
    private singleton(){}
    public static Singleton getInstance(){
        if(instance == null){			// DCL
            synchronized(Singleton.class){
                if(instance == null){
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

volatile能够禁止指令重排，那么多线程下也能正确执行

## 7. volatile关键字

保证修饰的变量可见性和禁止指令重排。

能够保证：该线程修改该变量后，其他进程的缓存都无效，需要重新加载该变量

volatile并不能保证原子性：eg：i++。

## 8. ThreadLocal

线程能够绑定自己的值，对该值进行修改不会引起并发竞争。