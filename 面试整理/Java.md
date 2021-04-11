# Java

## 1. 美团

https://www.nowcoder.com/discuss/633017?source_id=discuss_experience_nctrack&channel=-1

1. 刚刚提到volatile，讲一下你了解的相关知识 

   volatile是怎么保证可见性和有序性？我以双重校验锁的单例模式中用到的volatile举例，解释volatile保证有序性，然后面试官就问为什么指令执行顺序会是随机的（重[排序]()）？ 

2. 介绍一下ThreadLocal？ThreadLLocal实现原理是什么？Map中Key和值分别什么？
3. 熟悉哪些设计模式？随机挑一个让你介绍
4. 了解过OOM、ANR、内存泄露吗？**都分别介绍一下**

https://www.nowcoder.com/discuss/635640?type=2&channel=-1&source_id=discuss_terminal_discuss_hot_nctrack

1. HashMap底层结构 
2. HashMap 1.7 1.8分别会出现什么问题？怎么解决？1.7的头插法导致环形[链表]()具体怎么回事？ 
3. ConcurrentHashMap 
4. Int Integer包装类的比较问题
5. [项目]()里用多线程了吗？线程的几个状态说一下？  
6. CAS是什么，用到CAS的类有哪些？为什么要用到CAS，有什么好处？ 
7. AQS的底层实现？ 
8. 说一下ReentrantLock？ 
9. 说一下公平锁，非公平锁？ 
10. 说一下ReentrantLock怎么用AQS实现公平锁，非公平锁？
11. JVM

https://www.nowcoder.com/discuss/635772?type=2&channel=-1&source_id=discuss_terminal_discuss_hot_nctrack

1. 介绍线程池的核心参数；介绍任务提交过程和线程池的一个运作过程
2. sychronized和reentrantlock的区别；reentrantlock实现了哪些高级功能？
3. sychronized在1.6的优化是什么？(这里从偏向锁开始说) 
4. 对象头中有什么内容？ 
5. Java对象创建的过程 
6. JVM类加载的过程以及讲解一下双亲委派机制；双亲委派机制的作用有什么？ 
7. 讲解一下CMS垃圾回收器的具体回收过程以及特点

https://www.nowcoder.com/discuss/635772?type=2&channel=-1&source_id=discuss_terminal_discuss_hot_nctrack

1. 你了解虚拟内存吗？讲一下
2. hashmap的底层数据结构说一下。它是线程安全的吗？它在什么情况下会发生线程不安全的情况？(这里头插法和尾插法的线程不安全是不同的，可以分类说一下) 
3. hashmap为什么用的是[红黑树]()而不是[二叉树]()和二叉平衡树？

## 2. 搜狐

1. hashmap，简单
2. 反射

## 3. 携程

1. java基本数据类型
2. 大概是内存模型，内存溢出
3. string类是否可以继承，为什么用final修饰
4. 线程池里面的一些，问我懂吗，没记住

## 4. 网易

https://www.nowcoder.com/discuss/627203?type=2&order=3&pos=1&page=1&channel=-1&source_id=discuss_tag_nctrack

1. 说一下设计模式
2. String、StringBuffer、StringBuilder的区别
3. HashMap
   - 插入扩容
   - 判断何时扩容
4. 接口、抽象类
5. 面向对象三个特性和举例
6. HashMap
   - 插入扩容
   - 判断何时扩容
7. 接口、抽象类
8. 面向对象三个特性和举例

https://www.nowcoder.com/discuss/613274?type=2&order=3&pos=2&page=1&channel=-1&source_id=discuss_tag_nctrack

1. API 设计规范
2. 手写单例

https://www.nowcoder.com/discuss/569165?type=2&order=3&pos=3&page=1&channel=-1&source_id=discuss_tag_nctrack

1. 缓存雪崩 缓存穿透
2. 讲一下CorrentHashMap（底层、线程安全，分段锁有什么缺点）
3. 了解过压力测试吗

https://www.nowcoder.com/discuss/636176?type=2&channel=-1&source_id=discuss_terminal_discuss_hot_nctrack

1. oom怎么排查
2. 知不知道jvm排查问题的相关工具
3. 一个死循环导致cpu占满后，怎么排查到死循环在哪里
4. 双亲委派模型，有什么好处
5. HashMap线程安全吗
6. 注解原理
7. 乐观锁的实现原理
8. 排它锁的实现原理
9. 意向锁呢

## 5. 科大讯飞

1. java面向对象的三个特征
2. 介绍一下多态
3. 重写和重载的区别
4. 构造器什么时候被运行
5. 线程池
6. Java里的集合类
7. map下有哪些接口，线程安全是指。。。
8. 抽象类和接口的区别
9. java里的元注解