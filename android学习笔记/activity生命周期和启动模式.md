# activity生命周期

一个图来概括：

<img src="/Users/bytedance/Downloads/livehood.jpg" alt="livehood" style="zoom:67%;" />

这边要关注的是：

1. onPause必须先执行完，新Activity的onResume才会执行

   所以，onPause不能执行太耗时的操作，否则会影响新的activity的执行。

   此时可以做一些存储数据、停止动画等工作

2. 如果从当前activity切换到新的activity中，那么一般会回调onPause和onStop，但是如果新Activity采用了透明主题，那么当前Activity不会回调onStop——即，旧的activity还是处于可见状态

前面提到的：

之后前一个activity执行完成onPause，后一个activity的onResume才能继续执行。

原理：

1. activity的启动主要涉及到了：instrumentation、activitythread、activitymanagerservice（AMS）

   activity的启动由instrumentation负责，它通过binder向AMS发起请求，而AMS会维护一个activityStack来同步activity的状态，AMS是会通过activitythread来完成状态同步从而完成上面7个方法的回调。

2. 在ActivityStack中的`resumeTopActivity-InnerLocked`方法里面，限定了需要先pause前一个activity，然后才能执行当前的resume当前activity
   1. 最后在ActivityStackSupervisor中的`realStartActivityLocked`方法会在里面调用`scheduleLaunchActivity`，该方法的调用者是ActivityThread中的ApplicationThread对象，那么`scheduleLaunchActivity`会去执行新Activity的onCreate、onStart、onResume的调用过程

## 异常情况的生命周期

异常，例如：

当资源相关的系统配置发生改变以及系统内存不足时，Activity就可能被杀死

eg：页面翻转之后，配置发生变化，所以需要重新启动该activity。

### 1. 资源相关的系统配置发生改变

eg：突然旋转屏幕，系统配置发生变化，那么activity会被销毁并且重建。——默认是这样的，当然可以配置旋转而不触发销毁重建

周期变成：

<img src="/Users/bytedance/Library/Application Support/typora-user-images/image-20210622112212420.png" alt="image-20210622112212420" style="zoom:50%;" />

那么原来活动的activity会执行：onPause, onStop, onDestroy

但是这是异常终止，系统会调用`onSaveInstanceState()`来保存当前状态

1. 可以重写该方法，从而来额外保存自己需要的数据。该方法会携带一个**Bundle类型的参数**，可以将需要的内容填在里面：`outState.putString("key", tempData)`
2. **调用时机是在onStop之前**，但是没有明确和onPause的时序关系，所以顺序是随机的

重新创建activity之后，系统会调用`onRestoreInstanceState`，并且将保存的Bundle对象作为参数同时传递给onRestoreInstanceState和onCreate方法，**调用时机在onStart之后**

所以`onSaveInstanceState`和`onRestoureInstanceState`是配对使用的，系统自动为我们做了一定的恢复工作：系统会默认为我们保存**当前Activity的视图结构**，且之后能够恢复，eg：用户输入的数据、ListView滚动的位置等，但是临时数据等不会被保存（所以还需要在onSaveInstanceState进行自定义保存）

且每个View都有onSaveInstanceState和onRestoreInstanceState这两个方法。

保存和恢复view的工作流程：

1. Activity会调用onSaveInstanceState去保存数据
2. **Activity**会委托**Window**去保存数据
3. Window再委托它上面的顶级容器去保存数据，顶层容器是一个**ViewGroup**，一般来说它很可能是DecorView
4. 顶层容器再去一一通知它的子元素来保存数据

——委托思想，上层委托下层、父容器委托子元素去处理事情

实际使用：

可以发现：只有在异常终止的时候才会去调用：`onSaveInstanceState`和`onRestoureInstanceState`的。如果是正常销毁，不会去调用。

onCreate和onRestoureInstanceState都能带一个bundle的参数，那么之间存在何种区别呢？

1. 在使用上，两者都可以来恢复因为异常销毁的数据。

2. 但是，具体实现上

   onRestoreInstanceState一旦被调用，其参数Bundle savedInstanceState一定是有值的，我们不用额外地判断是否为空

   onCreate区分是否是正常启动，所以Bundle savedInstanceState需要额外判断是否为null

——两个都可以进行数据恢复，但是更推荐onRestoureInstanceState，因为更专业做恢复数据的事

### 2. 资源内存不足

其数据存储和恢复过程和上面完全一致

activity的优先级是根据是否可见来排的：

1. 前台Activity——正在和用户交互的Activity，优先级最高
2. 可见但非前台Activity——比如Activity中弹出了一个对话框，导致Activity可见但是位于后台无法和用户直接交互
3. 后台Activity——已经被暂停的Activity，比如执行了onStop，优先级最低。——最可能被异常销毁

所以据此，**如果一个进程中没有四大组件在执行，那么这个进程将很快被系统杀死**，因此，一些后台工作不适合脱离四大组件而独自运行在后台中，这样进程很容易被杀死。比较好的方法是将**后台工作放入Service中从而保证进程有一定的优先级**，这样就不会轻易地被系统杀死。

# activity启动模式

## 1. 启动模式

在《第一行代码》中已经学过了，有4种启动模式

1. standard：标准

   每次启动一个Activity都会重新创建一个新的实例，不管这个实例是否已经存在。

   **谁启动了该activity，那么该activity就运行在对应的栈中**

   但是用ApplicationContext去启动standard模式的Activity的时候会报错，这是因为：非Activity类型的Context（如ApplicationContext）并没有所谓的任务栈。

   ——解决方法：待启动Activity指定FLAG_ACTIVITY_NEW_TASK标记位，那么会对应创建一个新的任务栈，然后将该activity加入到该栈中。

2. singletop：栈顶复用

   新Activity已经位于任务栈的栈顶，那么此Activity不会被重新创建，同时它的**onNewIntent方法会被回调**

   如果activity不是在栈顶就会重新创建

3. singletask：栈内复用，系统也会回调其**onNewIntent**

   步骤：系统首先会寻找是否存在A想要的任务栈，如果不存在，就重新创建一个任务栈，然后创建A的实例后把A放到栈中。如果存在A所需的任务栈，这时要看A是否在栈中有实例存在，如果有实例存在，那么系统就会把A调到栈顶并调用它的onNewIntent方法，**且上面的所有activity全部出栈**，如果实例不存在，就创建A的实例并把A压入栈中

4. singleInstance：单实例模式，该activity会单独创建一个栈进行存储，所以之后关于该activity（未销毁的情况下），都不会重新创建该activity

在实现中，一个关键的参数：**TaskAffinity**：标识了一个Activity所需要的任务栈的名字，默认情况下，所有Activity所需的任务栈的名字为**应用的包名**，当然也可以为每个activity单独指定。

TaskAffinity属性主要和**singleTask启动模式**或者**allowTaskReparenting属性配对使用**，在其他情况下没有意义

1. singleTask启动模式配对使用：待启动的Activity会运行在名字和TaskAffinity相同的任务栈中。
2. allowTaskReparenting配对使用：当一个应用A启动了应用B的某个Activity后（此时，该activity会在应用a的任务栈中），如果这个Activity的allowTaskReparenting属性为true的话，那么当应用B被启动后，**此Activity会直接从应用A的任务栈转移到应用B的任务栈中**

如果设定了singletask的activity，那么Activity的确没有重新创建，只是**先调用了onPause**，然后**调用了onNewIntent**，接着调用onResume就又继续了。

任务栈分为**前台任务栈和后台任务栈**，后台任务栈中的Activity**位于暂停状态**，用户可以通过切换将后台任务栈再次调到前台。