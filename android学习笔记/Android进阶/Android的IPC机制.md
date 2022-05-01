# Android的IPC机制

看过Android的IPC，有初步的了解，但是这次做一个系统的了解。（从笼统的技术点罗列，到简单的代码分析，较为深入）

资料来源：《Android开发艺术探索》、《Android内核设计思想》（这两本书都很赞，感谢mentor:cherry_blossom:）

## Android中IPC的来源

IPC的来源：操作系统中一定存在多进程的，多进程之间需要通信（传递数据、信号），需要通过一定的机制。Android也不例外

Android中一个应用一个进程，一个进程存在多个线程，Android中的**主线程又称UI线程**，在主线程里面才能**操作界面元素**。如果只有一个线程，应用中的耗时任务都交给主线程去做，那么必然会产生阻塞，导致界面响应延迟（无法响应），严重影响用户体验。

——Android中特定的名字 **ANR**(application not responding，应用无法响应)

所以，将一些耗时的任务都放在子线程中去执行，避免ANR问题

Android特色的IPC机制：

1. binder
2. socket：还能支持两个终端之间的通信

IPC的应用场景：

1. 一个应用需要开多进程模式，例如模块执行耗时，为了不阻塞；大应用需要的内存量大，多进程可以获得多份内存
2. 多个应用之间

## Android的多进程模式

### 开启多进程

如何开启多进程：为四大组件在`AndroidMenifest.xml`中定义的时候：指定属性`android:process="xxxx"`

此外还能：通过JNI在native层去fork一个新的进程

```xml
<activity											// 启动页面
          android:name="com.chap2.MainActivity"
          android:label="@string/app_name"
          android:launchMode="standard">	// 默认进程：com.chap2
    <intent-filter>
    	<action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>

<activity
          android:name="com.chap2.SecondActivity"
          android:label="@string/app_name"
          android:process=":remote">			// 启动多进程，进程名字为：com.chap2:remote
</activity>
<activity
          android:name="com.chap2.ThirdActivity"
          android:label="@string/app_name"
          android:process="com.chap2.remote">	// 启动多进程，进程名字为：com.chap2.remote 
</activity>
```

注意：

1. 在指定时：`:`就会自动带上包名；可以直接写全称

   `:`创建的进程是当前应用的私有进程，其他应用无法共用

   不写，属于全局进程，其他应用通过**ShareUID**方法，从而能和其他应用共用

2. 不指定，就是默认进程，名字为包名

3. **Android系统会每个应用分配一个UID**，相同的UID应用才能够共享数据；而如果两个应用需要共用同一个进程，则需要相同的ShareUID和签名——它们即使不在同一个进程中，也能共享私有数据：data目录、组件信息等；在，还能共享内存数据

### Android的多进程机制

**Android为每个进程（应用）都创建了独立的虚拟机**，它们在内存分配上有不同的地址空间，会导致：**不同虚拟机访问同一类的对象会产生多个副本**

eg：在上面的app中，如果同时修改一个静态变量，那么它们的修改值之间是独立的

多进程的问题：

1. 静态成员和单例模式会失效——有多个副本，所以无法实现
2. 线程同步机制会失效——锁机制锁的不是同一个对象
3. SharedPreference的可靠性下降——不支持多进程并发写
4. Application会多次创建，不同进程的组件会创建不同的虚拟机、application和内存空间

## IPC基础知识

Android相关的IPC技术：Serializable接口（Java提供的对象完成序列化）、Parcelable接口（Android特有的对象完成序列化）、Binder

流程是：对象需要先序列化，然后通过Intent或者Binder传递数据到其他进程

序列化也可以：把对象持久化到存储设备或者通过网络传递

### 1. Serializable

Java提供的，是一个空接口，只需要声明实现，就能为对象提供标准的序列化和反序列化操作

```java
public class User implements Serialaizable{
    private static final long serialVersionUID = 4365675647473463L;		// 标识
    //....
}
```

1. `serialVersionUID`不是必须的，主要是用来辅助序列化和反序列化的，序列化之后的serialVersionUID和当前的类的值对应才能正常被序列化
2. 

Serializable主要就是声明该对象可以被序列化和反序列化。

ObjectOutputStream和ObjectInputStream会先序列化（反序列化）将对象数据输出（输入）

<img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20220414171328383.png" alt="image-20220414171328383" style="zoom: 50%;" />

### 2. Parcelable
