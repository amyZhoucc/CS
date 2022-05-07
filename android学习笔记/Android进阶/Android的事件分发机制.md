# Android的事件分发机制

*面试中被问到过，当时只是很浅的回答了一下。实习的时候，也遇到过相关的问题，却发现原理不知道，连为啥实现不了都无法理解。最后还是在mentor的帮助下（感恩:heart:），才稍微搞懂，但也是头痛医头脚痛医脚。这次就想着搜罗比较全的博客，并结合书籍，**全面、详细、深入的了解一下***

## 相关概念

### 1. view树

首先，页面上的布局存在ViewGroup和View混合的模式，而ViewGropu本质上还是继承自View的：

eg：

<img src="..\pic\image-20220501184740443.png" alt="image-20220501184740443" style="zoom:50%;" />

（右侧就是view树）

可以看到view树是有层级关系的，view节点一定都是叶子节点，当然view树可以只有一个view节点

每个view树都有一个根节点——**ViewRootImpl**——**它负责管理view树的绘制、事件分发**

应用可能存在多个view树，**一个activity就包含了一个view树**，**应用的悬浮窗是一个view树**、**dialog界面是一个view树**、用windowManager添加的view也是一个view树

### 2. window

**Android的view管理是以window为单位的**（我的理解：**一个页面就是一个window，对应一个view树**，所以window就通过view树来管理该页面——所以以window为单位进行管理）

window机制主要负责的：view的展示（view的绘制），**view的事件分发**

系统服务WindowManagerService（WMS）管理的单位就是window，也就是view树为单位，而view树是靠ViewRootImpl来管理的

<img src="..\pic\image-20220501185331390.png" alt="image-20220501185331390" style="zoom:60%;" />

通过图可以发现：

1. **WMS是在系统服务进程中的**，负责管理所有的window。**应用程序和WMS之间通过Binder通信进行跨进程通信**（而不是handler）
2. 每个viewRootImpl在WMS都有一个**windowState**与之对应，所以WMS是通过windowState来查找对应的viewRootImpl，从而实现管理

### 3. view内置的相关方法

前面知识已知：ViewGroup本质上是继承View方法，并且对部分方法进行重写。

View类有一个方法用于事件分发：==**`dispatchTouchEvent()`**==（字面意思就是分发触摸事件），子类可重写该分发方法的逻辑，ViewGroup由于继承自View，所以会重写该方法

1. ViewGroup对象中，`dispatchTouchEvent()`主要负责将事件下发
2. View对象中，`dispatchTouchEvent()`主要负责处理事件

==**`onTouchEvent()`**==，事件的消费都在此

一般自定义view会重写`dispatchTouchEvent()`和`onTouchEvent()`两个方法。



### 4. PhoneWindow

PhoneWindow是继承自Window抽象类的，它是一个**窗口辅助类**，而不是一个window。**在android中，PhoneWindow是Window抽象类的唯一实现**（在android.policy.PhoneWindow）

PhoneWindow内部维护着**view树和一些window参数**，而**该view树的根为DecorView**。

**Activity持有该PhoneWindow的实例**，从而实现对view树的管理

<img src="..\pic\image-20220502145801691.png" alt="image-20220502145801691" style="zoom:60%;" />

目的：PhoneWindow作为窗口辅助类，能够帮助类更好的管理控件与页面布局

注意：PhoneWindow不是activity特有的，dialog也如此

### 5. DecorView

DecorView可以看成是一个界面模板，包含了主题颜色、标题栏，这些都是在DecorView中设定的

<img src="..\pic\image-20220502150227958.png" alt="image-20220502150227958" style="zoom:50%;" />

Activity的布局页面都会插入到内容中，所以activity的布局会成为DecorView树的一部分

——而activity持有PhoneWindow对象，通过该对象**间接管理自己的布局**，**把window相关的操作都交给PhoneWindow进行处理，减轻activity的管理压力**

### 6. MotionEvent

触摸，主要分为单点触摸和多点触摸。

**MotionEvent：是触摸事件信息类**，算是事件的源头

#### 1. 单点触摸

触摸的基本事件类型：

1. ==**`ACTION_DOWN`**==：手指刚接触屏幕 0x00000000

2. ==**`ACTION_MOVE`**==：手指在屏幕上滑动，不断滑动就会产生一系列的MOVE事件

3. ==**`ACTION_UP`**==：手指从屏幕上离开的瞬间

4. ==**`ACTION_CANCEL`**==：当出现异常情况，事件序列被中断，就会产生该事件；**上层view拦截了下层正在处理的事件（上层的view回收了处理权）**，则会发送CANCEL到子view，表示事件已经结束，后续不会再传递过来

   eg: RecyclerView收到了DOWN事件，所以它会选择交给对应范围内的item询问是否处理。而此时，又传递过来MOVE，且滑动方向和RecyclerView的可滑动方向一致，那么RecyclerView认为这是一个滑动事件，所以它要回收触摸的处理权，所以对应的item会收到一个CANCEL的通知，且之后也不会收到关于该事件的通知了

5. `ACTION_OUTSIDE`：手指不在控件区域时触发

   eg：Dialog，不占满全屏的，但是能实现点击Dialog外区域关闭的，这个就是靠OUTSIDE实现的。还有悬浮窗等

正常情况下，一个触摸行为包含：

1. `DOWN` ->`UP`：简单的点击事件，没有手指滑动的操作（或者有滑动，但是距离很短，低于阈值）
2. `DOWN` -> `MOVE` -> `MOVE` -> .... -> `UP`

异常情况下：以`CANCEL`结束，而不是`UP`

调用MotionEvent对象的==**`getAction()`**==方法，可以得到某个事件对应的事件类型，返回的是一个整型变量

MotionEvent对象会提供点击事件的具体坐标：x、y值，可以通过方法调用获得，传参：触控点的索引：

1. ==**`getX()`/`getY()`**==：获取的是点击位置相对于当前view的坐标
2. ==**`getRawX()/getRawY()`**==：获取的是点击位置相对于屏幕左上角的坐标，左上角的坐标为(0,0)

#### 2. 多点触摸

多点触摸情况，会增加事件类型：

1. ==**`ACTION_POINTER_DOWN`**==：一个手指按下，另外一个手指也按下，就会产生该事件， 0x00000005
2. ==**`ACTION_POINTER_UP`**==：多个手指同时按下时，有一个手指抬起了，就会产生该事件

那么一个事件序列的边界还是`ACTION_DOWN`和`ACTION_UP`/`ACTION_CANCEL`，`POINTER_DOWN`和`POINTER_UP`都是序列中间可能存在的事件

**一个手指产生的序列边界是`ACTION_DOWN`（第一个手指按下）/`ACTION_POINTER_DOWN` -> ... -> `ACTION_UP`（最后一个手指离开）/`ACTION_POINTER_UP`/`ACTION_CANCEL`**

MotionEvent对象如何记录触摸的坐标呢？

——**维护一个数组**，数组中的每项对应不同的触摸点，通过数组下标进行索引

<img src="..\pic\image-20220502210507592.png" alt="image-20220502210507592" style="zoom:60%;" />

——注意，**触控点的索引是在变化的**，所以**追踪触控点，需要根据其id，而不是索引**

<img src="..\pic\image-20220502211018227.png" alt="image-20220502211018227" style="zoom:60%;" />

（本来有两个触摸点a、b，此时用户将触摸点a抬起了，那么触摸点b就顺移到了索引0的位置）

调用MotionEvent对象的==**`getActionMask()`**==方法，可以得到某个事件对应的触控点，返回的是一个整型变量——多点触摸必须使用这个方法才能获得实际的事件类型。因为为了在一个整型变量中同时存储事件类型和触控点的索引，该值分成了两部分：

1. 低1~8位：对应事件的类型
2. 低9~16位：是触控点的索引

eg: 第一个手指按下`0x00000000` -> 第二个手指按下`0x00000105` -> 第三个手指按下`0x00000205`->...

后面代码涉及到比较多的相关方法：

| 方法                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ==**`getActionMasked()`**==（常用，<br />getAction()不常用了） | 类似于`getAction()`，在多触摸点中一定要用这个方法才能获得事件类型<br />它能够将9~16位的触控点遮盖掉，从而获得真正的事件类型 |
| `getActionIndex()`                                           | 获取该事件由哪个手指产生的——确定触摸点的index                |
| `getPointerCount()`                                          | 获取在屏幕上手指个数                                         |
| `getPointerId(int poniterIndex)`                             | 获取一个指针（手指）的唯一标识符ID，在整个事件序列过程中，是不变的<br />索引是会变化的 |
| `findPointerIndex(int pointerId)`                            | 通过id获取当前的index（是在变化的），之后通过index来获取其他内容 |
| `getX(int pointerIndex)`                                     | 获取一个指针的X坐标（这个就是要通过index获取的）             |
| `getY(int pointerIndex)`                                     | 获取一个指针的Y坐标                                          |

注意：MOVE事件 和 CANCEL事件 没有涉及到触控点索引的变化，只有 DOWN事件 和 UP事件 才会涉及，所以在MOVE状态下无法获取actionID

——**pointID能够追踪事件流，在一个事件流程中不会随着其他手指的抬起或者按下而出现变化**

分发的要点：**一个view消费了某个触点的DOWN事件后，该事件的后续序列就都需要由该view处理**——不会出现相同事件不同view处理的情况，避免响应事件的混乱——也能简化代码，只需要对序列的第一个事件进行特殊判断，谁处理就负责处理到底，除非被上层的view拦截

但是：如果是多触摸点，那么在派发事件时，每个触摸点的信息就要分开发给对应感兴趣的view——**事件分离**

MotionEvent事件到来的时候，会将不同的触摸点的信息组成不同的MotionEvent派发给不同的view

可以通过`FLAG_SPLIT_MOTION_EVENT`判断是否要多触摸点分离

<img src="..\pic\image-20220503101529095.png" alt="image-20220503101529095" style="zoom:67%;" />

ps：还增加了相关鼠标操作的事件，eg：`ACTION_HOVER_ENTER`（没有按下，但是鼠标移动到了view区域），`ACTION_HOVER_MOVE`（没有按下，在view区域移动），`ACTION_HOVER_EXIT`（没有按下，鼠标移出view区域），`ACTION_SCROLL`（滚轮滚动，可以有水平AXIS_HSCROLL和竖直滚动AXIS_VSCROLL）

### 7. TouchSlop

是系统能够识别滑动的最小距离，即小于该距离，系统就不认为你在滑动

值：是一个常量，不同的手机值不同。==**`ViewConfiguration.get(getContext()).getScaledTouchSlop()`**==——可以用该值做一个是否滑动的过滤

ps：在`frameworks/base/core/res/res/values/config.xml`文件的`config_viewConfigurationTouchSlop`就是常量值的设定

### 8. TouchTarget

背景：一个ViewGroup有多个子view，如果不同的view接收了不同的触摸点的down事件，那么ViewGroup需要将后续的内容发送给对应的view——ViewGroup需要记录触摸点和view的对应关系，并且准确发送

——TouchTarget，它本身是一个链表节点，每个节点维护每个子view和对应触摸点的id（可以有多个id），viewGroup中维护的链表头是**`mFirstTouchTarget`**，如果它为null，表示没有子view接收了down事件（那么也不会处理后续的事件序列）

TouchTarget用一个整型数来记录所有的触摸点id，通过0/1来表示是否绑定该触摸点

——说明**能控制的最大触控点为32个**

——优势：位操作即可判断出该TouchTarget对象绑定的id

操作：当来了一个down事件，viewGroup会为down事件寻找感兴趣的view，并且创建一个TouchTarget加入到链表中。当up事件来临，又会将该节点从链表中删除

### 9. AccessibilityFocus辅助功能

在`dispatchTouchEvent()`等方法中，有很多都涉及到了`accessibilityFocus`相关的内容，主要是来帮助有障碍的用户使用Android。

所以如果有一个事件带有**`TargetAccessibilityFocus`标志**，那么说明这是一个特殊的辅助功能事件，需要特殊处理

——在读代码的时候，可以将这部分忽略（有点复杂，没必要）

## 触摸屏幕操作如何到达ViewRootImpl

背景：一个页面的view组成了一棵View树，而view树有一个根节点viewRootImpl，主要负责管理view树的绘制和事件分发。同一级别的就是window。WMS是系统服务，负责管理所有的window，它通过windowState来定位每个window，且WMS通过binder通信来管理viewRootImpl——这是事件的后半程

问题：用户触摸屏幕的操作是如何传递到WMS（前半程的任务）

<img src="..\pic\image-20220501190634091.png" alt="image-20220501190634091" style="zoom:60%;" />

用户触摸之后，由底层硬件感知，然后通知系统底层对应的驱动，交给Android的输入系统：IMS（inputManagerService）。

IMS会处理该触摸信息，通过WMS找到对应的window，然后将该信息发送给viewRootImpl——所以**WMS只提供对应的window信息，而不负责发送触摸信息，IMS负责发送触摸信息给viewRootImpl**

viewRootImpl收到该触摸信息后，事件的分发就开始了

## viewRootImpl分发事件的操作

ViewGroup本质上还是继承自View的，view树的顶层是ViewGroup（顶层不可能是view），所以从外部看，**整棵view树还是一个view对象**——所以**在view树内部，还是能够自行分发事件的**

**viewRootImpl收到事件后，经过处理，将事件封装成MotionEvent发送到顶层的view**，然后就让view自行分发了

顶层的view存在两种可能：

1. View：那么直接交给view进行处理

   eg：整个页面只有一个button按钮。

2. ViewGroup：

   - 如果是**应用布局界面（activity对应的）/ dialog，则它们的顶层为DecorView**，所以会调用DecorView的dispatchTouchEvent，而该方法重写：

     ```java
     // DecorView.java api29
     public boolean dispatchTouchEvent(MotionEvent ev) {
         final Window.Callback cb = mWindow.getCallback();	// 获取window的callback对象
         return cb != null && !mWindow.isDestroyed() && mFeatureId < 0
                 ? cb.dispatchTouchEvent(ev) : super.dispatchTouchEvent(ev);
     }
     ```

     理解：

     1. 尝试去获取window的callback对象，如果存在就调用该方法的dispatchTouchEvent

     2. 如果不存在，就调用父类的ViewGroup的dispatchTouchEvent

     3. Window.Callback本质上是一个接口，包含了window变化的回调方法，里面就包含了`dispatchTouchEvent`、`onTouchEvent`两个方法

        eg：**Activity实现了Window.Callback接口，在创建布局的时候，==activity把自己设置给了DecorView（？？？）==，所以在activity布局中，产生事件后DecorView会将事件分发给activity**。Dialog类似

   - 其他，顶层就是ViewGroup类对象，则调用自身类的dispatchTouchEvent进行分发，如果子类重写了就调用子类的；如果子类未重写，就调用ViewGroup的默认dispatchTouchEvent方法（多态）

   下图就是分发的逻辑：

<img src="..\pic\image-20220502142023066.png" alt="image-20220502142023066" style="zoom: 67%;" />

1. viewRootImpl在将触摸动作封装成`MotionEvent`对象之后，调用view树顶层的view对象的`dispatchTouchEvent()`（这边的view对象是一个泛指），这边就会判断view的具体类型
2. 顶层如果是view类型（这边的view对象是特指），那么**该view对象会直接处理事件**；如果是viewGroup类型，则继续判断
3. 如果是DecorView类型，因为DecorView类重写了`dispatchTouchEvent()`方法，在方法里面会判断是否存在callback对象；如果不是，就调用ViewGroup对象的`dispatchTouchEvent()`方法
4. 如果存在callback对象，则调用callback对象的`dispatchTouchEvent()`方法——**由于activity、dialog等将该对象设置为自己，所以本质上是交给activity、dialog进行分发**；如果不存在，就调用ViewGroup对象的`dispatchTouchEvent()`方法

误区：触摸事件是从activity开始的吗？——✖

activity只是其中的一种情况，其他情况下不会经过activity，更确切地说：**触摸事件是从viewRootImpl开始**

activity之后，根据代码是获取activity对象内部持有的window对象（本质上是phoneWindow对象），该对象持有view树的顶层节点decorView，它会操控decorView去将事件分发下去

——所以才会有书上说：事件分发的顺序是：activity -> window -> view

## DecorView之后的事件分发

由于Activity、Dialog对`dispatchTouchEvent`的操作是不同的，所以需要分类

### 1. Activity

在activity中，Window.Callback调用dispatchTouchEvent时，由于本质上是去调用Activity的`dispatchTouchEvent()`，所以直接定位到Activity的实现代码中

```java
// Activity.java
public boolean dispatchTouchEvent(MotionEvent ev) {
    // 点击屏幕操作，本质上是去调用onUserInteraction()方法
    // onUserInteraction()方法是一个空方法，等待程序员在创建activity子类的时候，重写
    if (ev.getAction() == MotionEvent.ACTION_DOWN) {
        onUserInteraction();
    }
    // 去获取activity持有的PhoneWindow对象，去调用该对象的分发方法
    if (getWindow().superDispatchTouchEvent(ev)) {
        return true;
    }
    // 到这边，说明在事件分发的过程中，没有view对象处理该事件，则activity自己处理该事件
    return onTouchEvent(ev);
}

// 空方法
public void onUserInteraction() {
}

// PhoneWindow.java api29
public boolean superDispatchTouchEvent(MotionEvent event) {
    return mDecor.superDispatchTouchEvent(event);	// mDecor就是PhoneWindow维护的view树的顶部view
}

// DecorView.java api29
public boolean superDispatchTouchEvent(MotionEvent event) {
    return super.dispatchTouchEvent(event);
}
```

理解：window.callback交给activity去分发事件 -> activity调用phoneWindow对象来分发事件 -> phoneWindow调用decorView来分发事件 -> decorView直接调用父类的`dispatchTouchEvent()`，它的父类是FrameLayout，而该类并没有重写该方法，所以**最终调用到ViewGroup类的`dispatchTouchEvent()`方法**

——ViewGroup会按照逻辑分发到树中感兴趣的子控件

ps：`onUserInteraction()`方法是留给用户自定义的，主要作用：在用户开始和activity进行交互的时候被调用。常用地方：屏保：长时间没有操作手机，就会显示一张图片，而当点击时会取消屏保——取消操作就是在`onUserInteraction()`中进行的；自动隐藏工具栏，当用户长时间不动，就隐藏工具栏，那么可以在这个方法中重现工具栏，那么用户再次点击的时候就会出现工具栏

### 2. Dialog

逻辑和Activity类似

```java
// Dialog.java api29
public boolean dispatchTouchEvent(@NonNull MotionEvent ev) {
    if (mWindow.superDispatchTouchEvent(ev)) {
        return true;
    }
    return onTouchEvent(ev);
}
```

### 3. PopupWindow

```java
// PopupWindow.java
private PopupDecorView mDecorView;		// PopupWindow的decorView是PopupDecorView，它是一个内部类
public boolean dispatchTouchEvent(MotionEvent ev) {
    if (mTouchInterceptor != null && mTouchInterceptor.onTouch(this, ev)) {
        return true;
    }
    return super.dispatchTouchEvent(ev);
}

private class PopupDecorView extends FrameLayout{
    ....
}
```

和前面两个不同的是：PopupWindow的view树的根是`PopupDecorView`，它是直接继承于FrameLayout，跟DecorView并无关系

理解：

1. **`mTouchInterceptor`是一个拦截器**，可以自定义拦截器。那么事件会优先交给它处理，而如果拦截器不处理，就交给父类的（FrameLayout）事件分发方法处理——后面步骤就同activity、dialog了

## 	ViewGroup事件分发过程

### 1. 核心方法介绍

主要流程：每个事件来临时，viewGroup都会判断是否拦截，如果是down事件（包含`ACTION_DOWN`和`ACTION_POINTER_DOWN`	），需要遍历子view看是否有子view消费该事件，然后将该事件派发出去

主要涉及到的方法：都是在View类中定义的，所以其子类都是有这三个方法的，可以对他们进行重写

1. `public boolean dispatchTouchEvent(MotionEvent ev)`：分发事件，事件分发的核心方法，分发的逻辑都在这边体现的

   普通的view、ViewGroup、DecorView等以及自定义view一般都会重写该方法

   在viewGroup的该方法中，主要是将事件分发给子view，如果子view没有处理则尝试自己处理；在view的该方法中，主要是消费触摸事件

   **如果事件能够传递到当前的view，那么该方法一定会被调用**。即：当前viewGroup会优先将该事件分发下去，而不是自行处理

   返回值：表示是否消耗该事件，由 下级的view的dispatchTouchEvent的返回值 和 自身的onTouchEvent的返回值（如果下级的view不处理，那么当前的view会判断是否处理）决定

   ps：

   1. 即使是view对象（狭义），也会有该方法的原因？

      $\because$ view可以注册很多事件监听器，eg：OnClickListener、OnTouchListener，View还有`onTouchEvent()`方法，这么多方法该调用哪个、先后顺序等都是交给`dispatchTouchListener()`来管理的

   2. 问题1的回答中，涉及到了多个事件监听器，那么它们的优先级是如何的？

      **`OnTouchListener` > `onTouchEvent()` > `OnLongClickListener` > `OnClickListener`**

      理解：`OnClickListener`需要完整的事件`ACTION_DOWN`和`ACTION_UP`，才能够触发的，如果它优先级最高，那么等它判断完（用户手指已经离开屏幕了），`OnTouchListener`等已经无法再做判断了——所以在最后；

      `OnLongClickListener`类似，它需要等待很长时间，但是它不需要`ACTION_UP`事件的到来——所以排在`OnClickListener`之前；

      `OnTouchListener`和`onTouchEvent()`功能类似，但是后者是view自定义的（view默认带的），前者是用户注册的，所以优先交给用户处理——前者优先级比后者高

      且`OnTouchListener`中的回调方法`onTouch()`就会被回调执行，根据其返回值，**如果返回false，那么`onTouchEvent()`方法还是会被执行**；**如果返回true，则`onTouchEvent()`就不会被执行了**

      <img src="..\pic\image-20220505101847773.png" alt="image-20220505101847773" style="zoom:60%;" />

   

   pps:`OnLongClickListener`和`onClickListener`都会在`onTouchEvent()`中可能被调用

2. `public boolean onInterceptTouchEvent(MotionEvent ev)`：拦截事件

   在dispatchTouchEvent中调用，判断是否拦截该事件。如果当前view拦截了该事件，那么之后该事件的序列都不会再调用该方法——因为该view决定拦截之后，系统会将事件的后续序列都交给他处理，而不需要再调用该方法进行判断了

   viewGroup的默认方法中只对一种事情进行的拦截，其他的情况均不拦截

   返回值：表示是否拦截该事件

   **view对象是没有该方法的，只有viewGroup对象才有**

3. `public boolean onToucnEvent(MotionEvent ev)`：处理事件

   在dispatchTouchEvent中调用，用来处理事件

   viewGroup默认没有重写该方法，还是调用view的原始方法（所以viewGroup需要自己处理的时候，会把自己当成一个普通的view处理——前提是该viewGroup用户没有重写）
   
   返回值：是否消耗该事件，**如果不消耗，则同事件序列的后续序列都不会在传递给当前的view了**，而是**交给它的父元素去处理，即父view的onTouchEvent会被调用**。它拦截了，但是不消耗，那么返回给父类view，父类会尝试消耗，如果它也无法执行就交给父类的父类——意味着：一个事件一旦交给某个view去执行，就必须消耗掉，否则同一个事件后续就不会再交给他处理了（类似于出现信任危机）

注意：Activity对象有`dispatchTouchEvent()`和`onTouchEvent()`方法，但是没有拦截方法，说明activity可以把事件分发下去 和 当事件没人处理的时候，自己处理，但是它不拦截事件，因为activity拦截了，下面的view就无法执行了

三个方法之间的关系：

```java
public boolean dispatchTouchEvent(MotionEvent ev){
    boolean consume = false;
    if(!onInterceptTouchEvent(ev)){		// 表示当前的view不拦截
        consume = child.dispatchTouchEvent(ev);		// 该事件分发给子view进行处理
    }
    if(!consume){			// 根据子view的返回值，如果false，表示子view不处理 或者是 当前的view要拦截
        consume = onTouchEvent();		// 当前view处理，看返回值，从而通知当前view的父view是否已处理
    }
    return consume;
}
```

注意：

1. 如果view不消耗除`ACTION_DOWN`的其他事件，那么该点击事件会消失，但是父view的onTouchEvent不会被调用，且当前的view能够收到后续序列，**最终消失的点击事件会传递给activity处理**

2. **ViewGroup默认不拦截任何事件**，onInterceptTouchEvent默认返回false

3. **View没有onInterceptTouchEvent方法**，所以一旦有事件传递给view，它就会马上处理

   **View的onTouchEvent默认返回true，即都会消耗事件**，除非该view是不可点击的。

   **view的不可点击需要：`clickable`和`longClickable`同时为false**，其中`longclickable`默认为false，`clickable`的默认：Button等默认为true，TextView等默认为false。不可点击和enable/disable并没有关系，哪怕是灰的，也还是能够处理点击事件的

4. onClick会发生的前提是：view可点击，且收到了down和up的完整事件序列

——可以发现：事件的分发是从外向内，一步步深入的。从父view到子view，当然可以通过`requestAllowInterceptTouchEvent()`方法来实现子元素干预父元素的分发过程，但是如果事件是ACTION_DOWN，则无法实现干扰

概括整体分发逻辑：

1. 首先是发生在ViewGroup的`dispatchTouchEvent()`中，它首先调用onInterceptTouchEvent()`，判断当前view是否拦截，即返回true/false
   1. 如果拦截，首先判断`mOnTouchListener`是否设置
      1. 如果设置，则回调`onTouch()`方法，则不再调用`onTouchEvent()`方法
      2. 如果没有设置，则调用`onTouchEvent()`，在执行中，如果设置了`mOnClickListener`，则会调用`onClick()`方法，执行完成之后继续执行`onTouchEvent()`后续的代码
   2. 如果不拦截，则调用子view的`dispatchTouchEvent()`分发给子view，子view再重复上述循环

### 2. 责任链模式

**责任链模式的定义：当有多个对象都可以处理同一请求的时候，可以将这些对象串联成一条链，通过这条链传递请求，直到有对象处理该请求**

根据代码显示出来的逻辑，事件分发机制主要采用的是责任链模式

ps：Java的异常也是责任链模式

特殊来说：事件分发遵循的是：`activity` -> `phoneWindow` -> `decorView` -> `ViewGroup` -> .... -> `View`

如果遍历一次，发现没有view处理，则又会回去：

`view` -> ... ->  `viewGroup` -> `decorView` -> `phoneWindow` -> `activity`，到最后如果activity都不处理，则这个事件才会被丢弃。

在整个过程中，上层的view可以事先拦截自己处理，如果不想自己处理，则交给子view继续去判断是否处理 

1. 事件一路发下去，一直没人处理，直到回到activity处：

   <img src="..\pic\image-20220504212437456.png" alt="image-20220504212437456" style="zoom: 60%;" />

   （这边是为了方便绘图做了一些简化：1. 父view是通过`onInterceptTouchEvent()`的返回值来判断是否自行拦截，还是发给子view，但是不是在该方法中去调用子类的`dispatchTouchEvent()` 2. 子view不处理，`dispatchTouchEvent()`是一个返回值，父view会有对应的标志位接收，而不是直接返回给父view的`onTouchEvent()`，而该方法是根据标志位来判断是否调用的）

2. 事件一路发下去，最底层的view处理了

   <img src="..\pic\image-20220504213045426.png" alt="image-20220504213045426" style="zoom:60%;" />

3. 事件一路发下去，中间有viewGroup拦截处理了

   <img src="..\pic\image-20220504213243334.png" alt="image-20220504213243334" style="zoom:60%;" />

   eg：在list中，滑动操作是交给RecyclerView，而不会交给下面的tiem的view处理

### 3. ViewGroup的分发代码

#### 1. `dispatchTouchEvent()`的完整代码

```java
public boolean dispatchTouchEvent(MotionEvent ev) {
    // 调试用的
    if (mInputEventConsistencyVerifier != null) {
        mInputEventConsistencyVerifier.onTouchEvent(ev, 1);
    }

    // 判断事件是否是针对可访问的焦点视图（屏幕辅助相关，适老化、盲人等）
    if (ev.isTargetAccessibilityFocus() && isAccessibilityFocusedViewOrHost()) {
        ev.setTargetAccessibilityFocus(false);
    }

    boolean handled = false;
    if (onFilterTouchEventForSecurity(ev)) {
        // 获取触控点，低1~8位表示事件类型，高9~16位表示索引值，down/up事件才有索引值
        final int action = ev.getAction();
        final int actionMasked = action & MotionEvent.ACTION_MASK;	// 获取低8位，获得事件类型
        
        // 当接收到的事件序列是down——是事件的起始
        if (actionMasked == MotionEvent.ACTION_DOWN) {
            // 将前面的状态全部清除，因为开启了一个新的动作
            cancelAndClearTouchTargets(ev);
            resetTouchState();			// 会清除之前设置的所有状态，包括FLAG_DISALLOW_INTERCEPT
        }
        // Check for interception.
        final boolean intercepted;			// 拦截的标志位
        // 事件序列起始，需要判断是否拦截；事件序列的后续，且子view已经拦截了，当前view不会再拦截
        if (actionMasked == MotionEvent.ACTION_DOWN || mFirstTouchTarget != null) {		
            // 标志位，判断是否子view禁止父view处理
            final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) ! = 0;	
            if (! disallowIntercept) {		// ViewGroup需要拦截的
                intercepted = onInterceptTouchEvent(ev);// 自己的onInterceptTouchEvent判断一下是否拦截
                ev.setAction(action); // 恢复操作，防止中途被更改
            } else {
                intercepted = false;		// ViewGroup被禁止拦截
            }
        } else {	// 到这边是：事件序列的后续，且没有子view处理的（mFirstTouchTarget == null）
            // ViewGroup默认拦截
            intercepted = true;
        }

        // 如果要拦截，或者有子view要处理该事件——去除辅助功能标志，按照普通事件进行分发
        if (intercepted || mFirstTouchTarget != null) {
            ev.setTargetAccessibilityFocus(false);
        }

        // 检查事件是否被取消
        final boolean canceled = resetCancelNextUpFlag(this)
            || actionMasked == MotionEvent.ACTION_CANCEL;

        // Update list of touch targets for pointer down, if needed.
        final boolean isMouseEvent = ev.getSource() == InputDevice.SOURCE_MOUSE;
        // 判断是否要对事件进行分裂，主要是用于多点触摸的情况
        final boolean split = (mGroupFlags & FLAG_SPLIT_MOTION_EVENTS) != 0
            && !isMouseEvent;
        // 如果来临的事件是down/pointer_down，则需要绑定新的TouchTarget对象
        TouchTarget newTouchTarget = null;		
        
        boolean alreadyDispatchedToNewTouchTarget = false;			// 标志位
        // 如果事件没有被取消，也没有想拦截的，那么就进入子view的分发状态
        if (!canceled && !intercepted) {
            // 如果事件的目标是可访问焦点，那么将该事件只分发给可访问性焦点的view去处理。
            // 如果没有这些view中的之一能处理，那么就清除标志位，将该事件分发给普通的子view
            // (我们正在查找以可访问性为重点的主机以避免保持状态，因为这些事件非常罕见。)
            View childWithAccessibilityFocus = ev.isTargetAccessibilityFocus()
                ? findChildWithAccessibilityFocus() : null;
			
            // 如果事件是序列的开始（down、多触摸点的down）
            if (actionMasked == MotionEvent.ACTION_DOWN
                || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
                || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
                final int actionIndex = ev.getActionIndex(); // always 0 for down
                final int idBitsToAssign = split ? 1 << ev.getPointerId(actionIndex)
                    : TouchTarget.ALL_POINTER_IDS;

                // 清除该触摸点的早期索引TouchTarget信息，防止信息不同步——前提是：当前事件是序列的开始
                removePointersFromTouchTargets(idBitsToAssign);

                final int childrenCount = mChildrenCount;
                if (newTouchTarget == null && childrenCount != 0) {
                    // 获取触摸点的坐标，还区分是鼠标的操作，还是手部操作
                    final float x =
                        isMouseEvent ? ev.getXCursorPosition() : ev.getX(actionIndex);
                    final float y =
                        isMouseEvent ? ev.getYCursorPosition() : ev.getY(actionIndex);
                    // 从后向前，遍历能够处理该事件的子view
                    final ArrayList<View> preorderedList = buildTouchDispatchChildList();
                    final boolean customOrder = preorderedList == null
                        && isChildrenDrawingOrderEnabled();
                    final View[] children = mChildren;
                    for (int i = childrenCount - 1; i >= 0; i--) {		// 从后向前扫描
                        final int childIndex = getAndVerifyPreorderedIndex(
                            childrenCount, i, customOrder);
                        final View child = getAndVerifyPreorderedView(
                            preorderedList, children, childIndex);
                        // 检查view是否能够接收该事件（view可见或者view正在播放动画）——不满足，就找下一个view
                        // 并且检查触摸点是否在view区域内——不满足，就找下一个view
                        if (!child.canReceivePointerEvents()
                            || !isTransformedTouchPointInView(x, y, child, null)) {
                            continue;
                        }
						
                        // 判断该子view是否有正在处理的事件（查链表即可），存在返回对应的target，否则返回null
                        newTouchTarget = getTouchTarget(child);
                        if (newTouchTarget != null) {		// 存在
                            // 该子view已经在它的范围内，接收了一个触摸点，所以增加一个新的触摸点即可
                            // 将对应的位置1即可
                            // 那么找到对应处理的view，跳出循环
                            newTouchTarget.pointerIdBits |= idBitsToAssign;
                            break;
                        }

                        resetCancelNextUpFlag(child);
                        if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                            // Child wants to receive touch within its bounds.
                            mLastTouchDownTime = ev.getDownTime();
                            if (preorderedList != null) {
                                // childIndex points into presorted list, find original index
                                for (int j = 0; j < childrenCount; j++) {
                                    if (children[childIndex] == mChildren[j]) {
                                        mLastTouchDownIndex = j;
                                        break;
                                    }
                                }
                            } else {
                                mLastTouchDownIndex = childIndex;
                            }
                            mLastTouchDownX = ev.getX();
                            mLastTouchDownY = ev.getY();
                            newTouchTarget = addTouchTarget(child, idBitsToAssign);
                            alreadyDispatchedToNewTouchTarget = true;
                            break;
                        }

                        // The accessibility focus didn't handle the event, so clear
                        // the flag and do a normal dispatch to all children.
                        ev.setTargetAccessibilityFocus(false);
                    }
                    if (preorderedList != null) preorderedList.clear();
                }
				// 如果没有找到子view来处理，且有触摸点正在被处理
                // 那么将该点交给最近最少被添加的那个节点，即链表尾巴节点
                if (newTouchTarget == null && mFirstTouchTarget != null) {
                    newTouchTarget = mFirstTouchTarget;
                    while (newTouchTarget.next != null) {		// 找到链表尾巴节点
                        newTouchTarget = newTouchTarget.next;
                    }
                    newTouchTarget.pointerIdBits |= idBitsToAssign;		// 将对应的触摸点置1
                }
            }
        }

        // 如果没有找到子view来处理，且没有子view正在处理触摸点，那么将viewGroup当成普通的view来处理
        if (mFirstTouchTarget == null) {
            handled = dispatchTransformedTouchEvent(ev, canceled, null,
                                                    TouchTarget.ALL_POINTER_IDS);
        } else {
            // Dispatch to touch targets, excluding the new touch target if we already
            // dispatched to it.  Cancel touch targets if necessary.
            TouchTarget predecessor = null;
            TouchTarget target = mFirstTouchTarget;
            while (target != null) {
                final TouchTarget next = target.next;
                if (alreadyDispatchedToNewTouchTarget && target == newTouchTarget) {
                    handled = true;
                } else {
                    final boolean cancelChild = resetCancelNextUpFlag(target.child)
                        || intercepted;
                    if (dispatchTransformedTouchEvent(ev, cancelChild,
                                                      target.child, target.pointerIdBits)) {
                        handled = true;
                    }
                    if (cancelChild) {
                        if (predecessor == null) {
                            mFirstTouchTarget = next;
                        } else {
                            predecessor.next = next;
                        }
                        target.recycle();
                        target = next;
                        continue;
                    }
                }
                predecessor = target;
                target = next;
            }
        }

        // Update list of touch targets for pointer up or cancel, if needed.
        if (canceled
            || actionMasked == MotionEvent.ACTION_UP
            || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
            resetTouchState();
        } else if (split && actionMasked == MotionEvent.ACTION_POINTER_UP) {
            final int actionIndex = ev.getActionIndex();
            final int idBitsToRemove = 1 << ev.getPointerId(actionIndex);
            removePointersFromTouchTargets(idBitsToRemove);
        }
    }

    if (!handled && mInputEventConsistencyVerifier != null) {
        mInputEventConsistencyVerifier.onUnhandledEvent(ev, 1);
    }
    return handled;
}

// 判断view是否允许接收事件：view要可见 或者 view正在播放动画
protected boolean canReceivePointerEvents() {
    return (mViewFlags & VISIBILITY_MASK) == VISIBLE || getAnimation() != null;
}
// 根据view，查找在父view的链表中，是否有正在处理的事件
private TouchTarget getTouchTarget(@NonNull View child) {
    for (TouchTarget target = mFirstTouchTarget; target != null; target = target.next) {
        if (target.child == child) {
            return target;
        }
    }
    return null;
}
```

理解：主要步骤是

1. 判断是否被拦截
1. 如果没有被拦截，且是初始DOWN情况，那么从后向前遍历尝试找可以处理的子view，可见且在合适的点击范围内——就是合适处理的子view，那么就分发给该子view，如果它有正在处理的触摸点，就直接将对应的位置1即可；如果是该view处理的第一个触摸点，让他调用`dispatchTouchEvent()`将事件分发下去，如果返回的是true，表示该view处理了事件，则跳出循环不再遍历（后续的子view不再有机会收到该事件）；如果不处理，就继续寻找
1. 而如果事情是被拦截了，向对应的子view发送CANCEL信息，并且将链表中对应的点清除
1. 如果事情没有被处理，而touchTarget链表不为空（说明有子view在处理其他触摸点的信息），那么交给该链表的尾部节点处理（最近最少使用）；如果链表为空，就将当前的view当成普通的view节点，调用`dispatchTouchEvent()`进行处理

#### 2. `dispatchTouchEvent()`的分块理解

判断是否要判断调用`onInterceptTouchEvent()`来判断是否拦截：

```java
...
// 过滤覆盖状态的情况（安全拦截），下面的是逻辑拦截
if (onFilterTouchEventForSecurity(ev)) {
    // 获取触控点，低1~8位表示事件类型，9~16位表示索引值，down/up事件才有索引值
    final int action = ev.getAction();
    final int actionMasked = action & MotionEvent.ACTION_MASK;	// 获取低8位，获得事件类型

    // 当接收到的事件序列是down——是事件的起始
    if (actionMasked == MotionEvent.ACTION_DOWN) {
        // 将前面的状态全部清除，因为开启了一个新的动作
        cancelAndClearTouchTargets(ev);
        resetTouchState();			// 会清除之前设置的所有状态，包括FLAG_DISALLOW_INTERCEPT
    }
    // Check for interception.
    final boolean intercepted;			// 标志位
    // 事件序列起始，需要判断是否拦截；事件序列的后续，且子view已经拦截了，当前view不会再拦截
    if (actionMasked == MotionEvent.ACTION_DOWN || mFirstTouchTarget != null) {		
        // 标志位，判断是否子view禁止父view处理
        final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) ! = 0;	
        if (! disallowIntercept) {		// ViewGroup需要拦截的
            intercepted = onInterceptTouchEvent(ev);// 调用自己的onInterceptTouchEvent判断一下是否拦截
            ev.setAction(action); // 恢复action的状态，防止在之前的操作中修改了action的值	从上到下、
        } else {				// ViewGroup被禁止拦截
            intercepted = false;		// 标志位置false
        }
    } else {	// 到这边是：事件序列的后续，且没有子view处理的（mFirstTouchTarget == null）——那么当前view就应该继续处理这件事
        intercepted = true;		// ViewGroup默认拦截
    }
    ...
}
...
```

理解：

1. 判断当前的view是否拦截，在调用`onInterceptTouchEvent()`方法前，还需要再判断一下：
   `actionMasked == MotionEvent.ACTION_DOWN || mFirstTouchTarget != null`，只有这个返回true，才会进入

   - `actionMasked == MotionEvent.ACTION_DOWN`，事件序列的开始，所以肯定要判断是否拦截
   - `mFirstTouchTarget != null`，到这边说明`actionMasked == MotionEvent.ACTION_MOVE/MotionEvent.ACTION_UP`，说明已经是事件的后续序列了，此时`mFirstTouchTarget != null`，说明子view有拦截，所以当前的view不会再拦截，而是直接交给子view处理——所以不会进入到else

2. `FLAG_DISALLOW_INTERCEPT`是标志位，子view可以通过方法`requestDisallowInterceptTouchEvent()`来实现设置，从而禁止父类拦截事件，但是注意`ACTION_DOWN`事件将一定被父类view处理

   因为：ViewGroup在遇到`ACTION_DOWN`事件时，会重置`FLAF_DISALLOW_INTERCEPT`，所以子view之前设置的都变得无效

   ——**所以，ACTION_DOWN事件来临，父view一定会先判断自己是否需要拦截，不拦截再交给子view去判断**

```java
....
// 紧接上面
// 检查事件是否被取消——设置对应的标志位
final boolean canceled = resetCancelNextUpFlag(this)
    || actionMasked == MotionEvent.ACTION_CANCEL;

// 标志位——判断事件来源是否是鼠标
final boolean isMouseEvent = ev.getSource() == InputDevice.SOURCE_MOUSE;
// 判断是否要对事件进行分裂，主要是用于多点触摸的情况
final boolean split = (mGroupFlags & FLAG_SPLIT_MOTION_EVENTS) != 0
    && !isMouseEvent;
// 如果来临的事件是down/pointer_down，则需要绑定新的TouchTarget对象——所以先新建一个touchTarget对象
TouchTarget newTouchTarget = null;		

boolean alreadyDispatchedToNewTouchTarget = false;			// 标志位
// 如果事件没有被取消，也没有想拦截的，那么就进入子view的分发状态
if (!canceled && !intercepted) {
    ...
```

理解：

1. 在上面代码判断之后，设置一系列的标志位，主要包含：该事件是否是CANCEL事件（上层view进行拦截，不再给下层的view处理，那么会发送这样一个事件过来通知）；针对多触摸点，所以需要将触摸点进行分离（按照`ACTION_DOWN`去处理`ACTION_POINTER_DOWN`事件）

```java
if (!canceled && !intercepted) {
    // 如果事件的目标是可访问焦点，那么将该事件只分发给可访问性焦点的view去处理。
    // 如果没有这些view中的之一能处理，那么就清除标志位，将该事件分发给普通的子view
    // (我们正在查找以可访问性为重点的主机以避免保持状态，因为这些事件非常罕见。)
    View childWithAccessibilityFocus = ev.isTargetAccessibilityFocus()
        ? findChildWithAccessibilityFocus() : null;

    // 如果事件是序列的开始（down、多触摸点的down）
    if (actionMasked == MotionEvent.ACTION_DOWN
        || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
        || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
        final int actionIndex = ev.getActionIndex(); 	// 获取对应的触摸点的索引值（该值是变化的）——通过这个值去获取pointerID，是不变的
        // 获取触摸点对应的PointerID，一个id表示一个触摸点。如果不分离（split为false），那么就获取全部的触摸点
        final int idBitsToAssign = split ? 1 << ev.getPointerId(actionIndex)
            : TouchTarget.ALL_POINTER_IDS;

        // 清除该触摸点的早期索引TouchTarget信息，防止信息不同步——前提是：当前事件是序列的开始
        removePointersFromTouchTargets(idBitsToAssign);

        final int childrenCount = mChildrenCount;
        if (newTouchTarget == null && childrenCount != 0) {
            // 获取触摸点的坐标，还区分是鼠标的操作，还是手部操作
            final float x =
                isMouseEvent ? ev.getXCursorPosition() : ev.getX(actionIndex);
            final float y =
                isMouseEvent ? ev.getYCursorPosition() : ev.getY(actionIndex);
            // 从后向前，遍历能够处理该事件的子view（后面就是后添加的view，有更高的优先权）
            // 创建待遍历的view列表
            final ArrayList<View> preorderedList = buildTouchDispatchChildList();
            // 是否采用自定义view的顺序——按照优先级来派发事件
            final boolean customOrder = preorderedList == null
                && isChildrenDrawingOrderEnabled();
            final View[] children = mChildren;
            ...
        }
    }
}

// 辅助方法
public ArrayList<View> buildTouchDispatchChildList() {
    return buildOrderedChildList();
}
ArrayList<View> buildOrderedChildList() {
    final int childrenCount = mChildrenCount;
    // child个数为1或者没有子view有z轴，那么直接返回
    if (childrenCount <= 1 || !hasChildWithZ()) return null;
	// 清除之前的状态
    if (mPreSortedChildren == null) {
        mPreSortedChildren = new ArrayList<>(childrenCount);
    } else {
        // callers should clear, so clear shouldn't be necessary, but for safety...
        mPreSortedChildren.clear();
        mPreSortedChildren.ensureCapacity(childrenCount);		// 扩容
    }
	// 判断是否是自定义child列表的顺序
    final boolean customOrder = isChildrenDrawingOrderEnabled();
    for (int i = 0; i < childrenCount; i++) {
        // add next child (in child order) to end of list
        final int childIndex = getAndVerifyPreorderedIndex(childrenCount, i, customOrder);
        final View nextChild = mChildren[childIndex];
        final float currentZ = nextChild.getZ();

        // 如果列表中的最后一个view的z值大于待插入的view，那么将当前的view插入到不超过自身z值的位置
        int insertIndex = i;
        while (insertIndex > 0 && mPreSortedChildren.get(insertIndex - 1).getZ() > currentZ) {
            insertIndex--;
        }
        mPreSortedChildren.add(insertIndex, nextChild);
    }
    return mPreSortedChildren;
}
```

理解：在将事件派发给子view之前的一系列操作，包括特殊操作（可访问行焦点。accessibilityFocus等辅助功能），清除操作（避免之前的赋值信息影响），获取触摸点对应的x、y坐标，建立子view的遍历列表（默认是按照创建顺序的从后向前，也可以自定义）

当前view不拦截，则分发给子view，看子view是否需要拦截

```java
...
// 如果ViewGroup不拦截，则交给子view们去处理
final View[] children = mChildren;
// 遍历所有的子view，从后向前（后面的有更高的优先级，在z轴更上面/最后渲染的）
for (int i = childrenCount - 1; i >= 0; i--) {
    final int childIndex = getAndVerifyPreorderedIndex(
        childrenCount, i, customOrder);
    final View child = getAndVerifyPreorderedView(
        preorderedList, children, childIndex);
    // 子view能够接收触摸事件 且 触摸事件在该view的范围内——那么就找到了对应的view
    // ——这边的判断是两个条件存在一个不满足，就找下一个子view继续判断，而不进行下面的操作
    if (!child.canReceivePointerEvents()
        || !isTransformedTouchPointInView(x, y, child, null)) {
        continue;
    }
	// 到这边 说明已经找到了能接收该事件的子view
    // 判断该child对应的是否存在已经在处理的触摸事件——存在就返回对应的touchTarget，否则返回false
    newTouchTarget = getTouchTarget(child);
    if (newTouchTarget != null) {	// 说明该view已经接收了在它管理范围内的触摸事件（是一个多点触摸的情况）
        // 那么将对应的触摸点置1即可——可以退出当前循环了
        newTouchTarget.pointerIdBits |= idBitsToAssign;
        break;
    }
	
    // 到这边，说明当前的view还没有正在处理的触摸事件
    
    resetCancelNextUpFlag(child);
    // dispatchTransformedTouchEvent本质上还是子类的dispatchTouchEvent()方法
    // 即交给下一层的view进行事件分发，如果返回true，表示事件被子view（或者是子view的孩子view）处理了
    if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
        mLastTouchDownTime = ev.getDownTime();
        if (preorderedList != null) {
            // 保存mLastTouchDownTime、mLastTouchDownIndex、mLastTouchDownX、mLastTouchDownY
            // ——保存最近点击的事件的处理view的相关信息，防止找不到处理的view时，找不到大怨种来顶包
            for (int j = 0; j < childrenCount; j++) {
                if (children[childIndex] == mChildren[j]) {
                    mLastTouchDownIndex = j;
                    break;
                }
            }
        } else {
            mLastTouchDownIndex = childIndex;
        }
        mLastTouchDownX = ev.getX();
        mLastTouchDownY = ev.getY();
        // 添加touchTarget，并且跳出循环
        newTouchTarget = addTouchTarget(child, idBitsToAssign);	
        alreadyDispatchedToNewTouchTarget = true;	// 表示已经处理过新的这个touchTarget了，避免重复处理
        break;
    }

    // The accessibility focus didn't handle the event, so clear
    // the flag and do a normal dispatch to all children.
    ev.setTargetAccessibilityFocus(false);
}
...

/**
	上述方法的一些关键辅助方法
*/
// 将touchTaget加入到队头，添加的属性包含：处理的子view、点击的id
private TouchTarget addTouchTarget(@NonNull View child, int pointerIdBits) {
    final TouchTarget target = TouchTarget.obtain(child, pointerIdBits);
    target.next = mFirstTouchTarget;
    mFirstTouchTarget = target;		// 头插法
    return target;
}

// 检查该view是否可见，是否存在动画等 
private static boolean canViewReceivePointerEvents(@NonNull View child) {
    return (child.mViewFlags & VISIBILITY_MASK) == VISIBLE
            || child.getAnimation() != null;
}
// 将值从屏幕坐标系转换到view的坐标系，检查是否在view的区域内
protected boolean isTransformedTouchPointInView(float x, float y, View child,
            PointF outLocalPoint) {
    final float[] point = getTempLocationF();
    point[0] = x;
    point[1] = y;
    transformPointToViewLocal(point, child);
    final boolean isInView = child.pointInView(point[0], point[1]);
    if (isInView && outLocalPoint != null) {
        outLocalPoint.set(point[0], point[1]);
    }
    return isInView;
}
public void transformPointToViewLocal(float[] point, View child) {
    point[0] += mScrollX - child.mLeft;
    point[1] += mScrollY - child.mTop;

    if (!child.hasIdentityMatrix()) {
        child.getInverseMatrix().mapPoints(point);
    }
}

// 这个方法主要是，父view调用子view的方法，将事件分发下去。主要是将触摸事件设置好偏移之后分发下去。如果没有子view，那么就调用自身view的分发方法（就是将当前的viewGroup当作普通的view进行处理）
// 参数：原来的motionEvent事件，是否取消操作，目标子view，子view感兴趣的pointerID
private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
            View child, int desiredPointerIdBits) {
    final boolean handled;

    // 保存原始action，便于之后恢复状态
    final int oldAction = event.getAction();
    // 如果事件是取消，那么不需要做转换，调用分发的方法——不需要修改内容，只需要通知取消处理即可
    if (cancel || oldAction == MotionEvent.ACTION_CANCEL) {
        event.setAction(MotionEvent.ACTION_CANCEL);
        if (child == null) {
            handled = super.dispatchTouchEvent(event);	// 调用自身的view的分发方法
        } else {
            handled = child.dispatchTouchEvent(event);
        }
        event.setAction(oldAction);		// 恢复原来的action
        return handled;			// 返回是否处理的状态
    }

    // 计算出需要传递的触摸点id
    final int oldPointerIdBits = event.getPointerIdBits();		// 现在屏幕上的所有触摸点
    // 存在一种情况：desiredPointerIdBits可能全是1，所以需要进行一次过滤——得到真正可接受的触摸点信息
    final int newPointerIdBits = oldPointerIdBits & desiredPointerIdBits;

    // 没有一个触摸点id符合的，直接返回
    if (newPointerIdBits == 0) {
        return false;
    }

    // 如果所有点都使用，那么直接使用原来的event，
    // 如果不是，就要创建一个副本进行事件分发，主要是为了保持event的状态
    final MotionEvent transformedEvent;
    if (newPointerIdBits == oldPointerIdBits) {		// 相等，就直接使用原来的event
        if (child == null || child.hasIdentityMatrix()) {
            // 当目标控件不存在通过setScaleX()等方法进行的变换时，
            // 为了效率会将原始事件简单地进行控件位置与滚动量变换之后
            // 发送给目标的dispatchTouchEvent()方法并返回
            if (child == null) {
                handled = super.dispatchTouchEvent(event);
            } else {
                final float offsetX = mScrollX - child.mLeft;
                final float offsetY = mScrollY - child.mTop;
                event.offsetLocation(offsetX, offsetY);		// 计算并设置偏移

                handled = child.dispatchTouchEvent(event);	// 事件分发下去

                event.offsetLocation(-offsetX, -offsetY);		// 恢复event的值
            }
            return handled;
        }
        transformedEvent = MotionEvent.obtain(event);
    } else {
        transformedEvent = event.split(newPointerIdBits);	// 创建一个新的副本，来存放新的触摸点，分离了事件类型和索引点
    }

    // 切换坐标系，转换为目标view的坐标系并进行分发
    if (child == null) {
        handled = super.dispatchTouchEvent(transformedEvent);
    } else {
        final float offsetX = mScrollX - child.mLeft;		// 计算偏移量
        final float offsetY = mScrollY - child.mTop;
        transformedEvent.offsetLocation(offsetX, offsetY);		
        if (! child.hasIdentityMatrix()) {			// 存在scale等变换，则需要进行矩阵转换
            transformedEvent.transform(child.getInverseMatrix());
        }

        handled = child.dispatchTouchEvent(transformedEvent);
    }

    // 回收这个临时变量
    transformedEvent.recycle();
    return handled;
}
```

理解：这部分的前提是整个代码都是一个新的DOWN事件下

这部分主要操作是遍历子view，找到可以接收该事件的view：

1. 代码中从后开始向前遍历（或者是自定义的顺序）

   why从后开始向前遍历呢？

   在判断子view是否可以处理时，首先需要判断触摸点是否在子view的控件范围内（前提是该子view是可点击的，如果不可点击就跳过），如果不再则默认不分发的；而存在多个控件重叠的问题，那么**优先交给最上面的view处理——在最上面的就是最后加载的，就是在childView中的相对后面节点**

2. `if (!child.canReceivePointerEvents() || !isTransformedTouchPointInView(x, y, child, null))`

   根据这个去判断是否找到对应的能处理的view：子view能够接收触摸事件 且 触摸事件在该view的范围内，两个条件均需要满足，如果不满足，就直接找下一个view继续判断

3. 当找到可以处理的子view之后，判断该view是否已经有在处理其他触摸点了（针对多触摸点的情况）：

   1. 如果有其他正在处理的点，那么直接将对应的新的触摸点的id置1即可——即更新TouchTarget对象的内容即可，然后跳出该循环；
   2. 如果没有，说明该view之前没有处理过其他触摸点，那么**交给该子view进行事件的分发（注意这边存在一个递归操作）**，
      1. 如果返回true，表示该子view或者其孩子view能处理该事件，那么更新mLastTouchDownXXX相关信息（留着有用吧），创建一个新的touchTarget对象，以头插法的形式插入到父view的链表中，并且更新mFirstTouchTarget为新的这个TouchTarget对象，且跳出循环；
      2. 如果返回false，表示子view或者其孩子view不处理，进入下一个循环找下一个view

4. ==**存在的问题：在3.1中，针对多触摸点的情况，它不会交给子view去分发给它的孩子view去看谁要处理，而是直接置位。那么是否存在一个情况，前一个触摸点是子view的其中一个view处理的，而现在这个触摸点是子view的另外一个view范围的，两个触摸点都在该子view的范围内，那么该如何通知另外一个view要处理了呢——毕竟连事件都不分发了？**==

下面这部分的代码出现在for循环跳出来之后，和for循环同级别（这部分的前提是整个代码都是一个新的DOWN事件下）

```java
// 如果没有找到子view来处理，且有触摸点正在被处理
// 那么将该点交给最近最少被添加的那个节点，即链表尾巴节点
if (newTouchTarget == null && mFirstTouchTarget != null) {
    newTouchTarget = mFirstTouchTarget;
    while (newTouchTarget.next != null) {		// 找到链表尾巴节点
        newTouchTarget = newTouchTarget.next;
    }
    newTouchTarget.pointerIdBits |= idBitsToAssign;		// 将对应的触摸点置1
}
```

这个之后，跳出DOWN循环，和是否CANCEL循环，即：

```java
if (!canceled && !intercepted) {
	if (actionMasked == MotionEvent.ACTION_DOWN
        || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
        || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
        ....
    }
}
```

去问了一圈的子view，发现均不处理，那么只能父类负责了（或者根本没进上面的循环）

```java
...
// 如果找不到接收的子view，那么就将父view当作普通的View来进行处理
if (mFirstTouchTarget == null) {
    handled = dispatchTransformedTouchEvent(ev, canceled, null,
                                            TouchTarget.ALL_POINTER_IDS);
} else {	// 说明已经有子view处理了这个触摸事件了，那么直接将事件分发给该view，如果是CANCEL，就通知取消事件
    TouchTarget predecessor = null;
    TouchTarget target = mFirstTouchTarget;
    while (target != null) {
        final TouchTarget next = target.next;
        // 说明该touchtarget已经在前面被处理过，那么标记已处理
        if (alreadyDispatchedToNewTouchTarget && target == newTouchTarget) {
            handled = true;
        } else {		
            final boolean cancelChild = resetCancelNextUpFlag(target.child)
                || intercepted;
            // 向下分发取消的信息 或者 正常分发的信息——DOWN之后的后续事件，所以还是要分发的
            if (dispatchTransformedTouchEvent(ev, cancelChild,
                                              target.child, target.pointerIdBits)) {
                handled = true;			// 对应标志位置位
            }
            // 如果是CANCEL操作，那么移除
            if (cancelChild) {
                if (predecessor == null) {
                    mFirstTouchTarget = next;
                } else {
                    predecessor.next = next;
                }
                target.recycle();
                target = next;
                continue;
            }
        }
        predecessor = target;
        target = next;
    }
}
```

理解：

1. why需要遍历链表？

   因为可能存在的是CANCEL消息，那么事件的触摸点可能位于链表的任何位置，所以需要遍历，通过计算找到对应的child的触摸点，然后删除；或者是找了最近最少使用的大怨种来处理，那么需要找到它，将事件派发给它

2. 对于多点触摸来说，viewGroup会将POINTER_DOWN当作DOWN事件进行处理的，分发下去

   1. 如果有子view消耗该事件，和单点触摸一致，创建touchTarget对象
   2. 如果没有view消耗，就交给链表尾尾部（最近最少使用的子view）的子view进行处理

   注意：viewGroup有两个view接收了不同触摸点的事件，那么拦截其中一个view的事件，viewGroup还是不会消费拦截的事件序列的，即**viewGroup和其子view不会同时处理触摸事件，即使是两个触摸点的**

最后，和`if (!canceled && !intercepted){} `同一等级的

```java
// 如果是取消事件，或者是手指抬起，鼠标抬起等，那么就重置点击状态信息（等待下一次点击）
if (canceled
    || actionMasked == MotionEvent.ACTION_UP
    || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
    resetTouchState();
} else if (split && actionMasked == MotionEvent.ACTION_POINTER_UP) {	
    // 如果是需要分离多触摸点的，清除对应所保存的触摸点信息
    final int actionIndex = ev.getActionIndex();
    final int idBitsToRemove = 1 << ev.getPointerId(actionIndex);
    removePointersFromTouchTargets(idBitsToRemove);
}
```

#### 3. 安全拦截

背景：当一个控件a被另一个非全屏的控件b遮挡，就容易出现恶意操作，表面上的按钮b是不可点击的，实际上触摸触发的下面的控件a

eg：

<img src="..\pic\image-20220507150536576.png" alt="image-20220507150536576" style="zoom:50%;" /><img src="..\pic\image-20220507150549655.png" alt="image-20220507150549655" style="zoom:50%;" />、

```java
// 在dispatchTouchEvent中执行之前进行过滤的内容
public boolean onFilterTouchEventForSecurity(MotionEvent event) {
    // 当被覆盖时 且 当前窗口被覆盖，就不处理
    if ((mViewFlags & FILTER_TOUCHES_WHEN_OBSCURED) != 0
        && (event.getFlags() & MotionEvent.FLAG_WINDOW_IS_OBSCURED) != 0) {
        // Window is obscured, drop this touch.
        return false;
    }
    return true;
}
```

理解：

1. `FILTER_TOUCH_WHEN_OBSCURED`，表示被非全屏的控件覆盖时，自动忽略所有触摸事件；`FLAG_WINDOW_IS_OBSCURED`，表示当前窗口被非全屏view覆盖了

#### 4. `onInterceptTouchEvent()`方法

```java
public boolean onInterceptTouchEvent(MotionEvent ev) {
    if (ev.isFromSource(InputDevice.SOURCE_MOUSE)
        && ev.getAction() == MotionEvent.ACTION_DOWN
        && ev.isButtonPressed(MotionEvent.BUTTON_PRIMARY)
        && isOnScrollbarThumb(ev.getX(), ev.getY())) {
        return true;
    }
    return false;
}
```

ViewGroup的默认方法，当只有一种特殊情况的时候：

1. 点击来源是鼠标
2. 当前点的点击事件是DOWN
3. ....

——这个情况下，会拦截，返回true；否则，返回false

## View事件分发过程

注意，view默认代码是不支持多点触摸处理的，如果需要，要用户自行重写`dispatchTouchEvent`

### `dispatchTouchEvent`方法

当一个触摸事件分发到view对象（狭义），或者ViewGroup不再向下分发（本身拦截 或者 没有子view处理），那么view的`dispatchTouchEvent()`会被调用

```java
public boolean dispatchTouchEvent(MotionEvent event) {
    // 辅助功能事件的处理
    if (event.isTargetAccessibilityFocus()) {
        // 控件没有获取到焦点，那么不处理该事件，直接返回false
        if (!isAccessibilityFocusedViewOrHost()) {
            return false;
        }
        // 如果获取到焦点，就按照常规方法处理事件
        event.setTargetAccessibilityFocus(false);
    }
    // 标志位，主要是返回告诉父view，是否已经消费该事件了
    boolean result = false;		
	
    // 一致性检验，检查事件是否有被改变
    if (mInputEventConsistencyVerifier != null) {
        mInputEventConsistencyVerifier.onTouchEvent(event, 0);
    }

    final int actionMasked = event.getActionMasked();
    // 触摸事件的起始——按下DOWN，需要进行预防性的清除——停止嵌套滚动
    if (actionMasked == MotionEvent.ACTION_DOWN) {
        stopNestedScroll();
    }
	
    // 类似于ViewGroup，过滤安全模式（覆盖状态）后的点击情况
    if (onFilterTouchEventForSecurity(event)) {
        // 如果事件是滚动进度条
        if ((mViewFlags & ENABLED_MASK) == ENABLED && handleScrollBarDragging(event)) {
            result = true;
        }
        //noinspection SimplifiableIfStatement
        ListenerInfo li = mListenerInfo;
        // 如果当前view设置了OnTouchListener，且当前的view可点击，那么优先回调onTouch方法处理
        // 并根据其返回值，设置result——返回true，表示事件已处理，设置标识位为true；否则就进入下一个判断
        // ——注意到这一步，onTouch已经被调用过了（除非前面的判断不成立）
        if (li != null && li.mOnTouchListener != null
            && (mViewFlags & ENABLED_MASK) == ENABLED
            && li.mOnTouchListener.onTouch(this, event)) {
            result = true;
        }
		// 到这边，可能是交给onTouch方法，但是处理失败，返回false；
        // 或者是没有设置onTouchListener监听器，所以交给默认的方法处理
        // 并且根据该方法的返回值，判断是否设置标志位（同上）
        if (!result && onTouchEvent(event)) {
            result = true;
        }
    }
	 // 一致性检验，检查事件是否有被改变
    if (!result && mInputEventConsistencyVerifier != null) {
        mInputEventConsistencyVerifier.onUnhandledEvent(event, 0);
    }

    // 如果是UP事件 或者是 CANCEL事件（都是该view收到该事件序列的终点），
    // 或者是DOWN事件，但一系列判断后，并没有消耗该事件（根据机制，不处理down事件，那也不会收到该事件的后续）
    // 就停止滚动状态
    if (actionMasked == MotionEvent.ACTION_UP ||
        actionMasked == MotionEvent.ACTION_CANCEL ||
        (actionMasked == MotionEvent.ACTION_DOWN && !result)) {
        stopNestedScroll();
    }

    return result;		// 返回状态，表示是否消费了该事件
}
```

理解：这边最核心的是，

1. 根据是否初始化了`mListenerInfo`，是否注册了`mOnTouchListener`，那么会调用该监听器的`onTouch()`方法，如果返回true，表示消耗了该事件，那么返回true；如果返回false，则继续处理
2. 然后调用`onTouchEvent()`，根据返回值，判断是否消耗了该事件，如果消耗了，就返回true；否则，继续处理
3. 后面就是一系列的收尾工作

### `onTouchEvent`方法

```java
public boolean onTouchEvent(MotionEvent event) {
    final float x = event.getX();
    final float y = event.getY();		// 获得点击事件对应的x、y坐标
    final int viewFlags = mViewFlags;
    final int action = event.getAction();
	
    // 判断当前的view是否是可点击状态
    final boolean clickable = ((viewFlags & CLICKABLE) == CLICKABLE
                               || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)
        || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE;
	
    // 如果是禁用状态，但是按下状态为true，且当前是事件的尾声了UP，那么取消按下的状态
    // 禁用状态下，直接返回当前view的是否可点击状态
    if ((viewFlags & ENABLED_MASK) == DISABLED) {
        if (action == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
            setPressed(false);
        }
        mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
        // 禁用状态的view仍然会消耗点击事件，只不过没有响应而已
        return clickable;
    }
    // 如果该view设置了触摸事件代理，那么交给代理的onTouchEvent处理——代理方法返回true，就返回true；否则继续
    if (mTouchDelegate != null) {
        if (mTouchDelegate.onTouchEvent(event)) {
            return true;
        }
    }
	
    // 如果view是可点击的——不论进入哪一个，最终都会返回true
    // 两种情况可以进入：view是可点击的；或者view可以悬停或者长按时显示工具提示tooltip
    if (clickable || (viewFlags & TOOLTIP) == TOOLTIP) {
        switch (action) {
            // 手指抬起操作
            case MotionEvent.ACTION_UP:
            ....
			// 手指按下操作
            case MotionEvent.ACTION_DOWN:
			....
            case MotionEvent.ACTION_CANCEL:
            ....    
			// 手指抬起事件
            case MotionEvent.ACTION_MOVE:
            ....
        }

        return true;
    }

    return false;
}
```

理解：

1. **在默认情况下，可点击的view一定会消耗事件**——代码中，无论哪个状态下，返回值都是true（即使收到的是CANCEL，也返回true）
2. 涉及到一个焦点判断。主要是源于，操作模式分为按键模式和触摸模式，按键模式需要用方向光标选中一个按钮（获取一个focus），然后再点击按下按钮，但是在触摸模式下，按钮不需要获得焦点——**所以在view的触摸模式下可以获取焦点，那么它将无法响应点击事件，那么也就无法调用监听器**

#### 1. ACTION_DOWN的处理

```java
case MotionEvent.ACTION_DOWN:
	// 根据屏幕来源，进行标志位的置1——是否是可触摸屏
    if (event.getSource() == InputDevice.SOURCE_TOUCHSCREEN) {	
        mPrivateFlags3 |= PFLAG3_FINGER_DOWN;
    }
    mHasPerformedLongPress = false;
	// 如果该按钮是不可点击的，那么能进入到这边判断的是第二种条件：
	// 表示此视图可以在悬停或长按时显示工具提示
	// 那么检查长按的相关配置，并且跳出switch，不再执行下去
    if (!clickable) {
        checkForLongClick(
            ViewConfiguration.getLongPressTimeout(),
            x,
            y,
            TOUCH_GESTURE_CLASSIFIED__CLASSIFICATION__LONG_PRESS);
        break;
    }
	
	// 检查button点击的特殊情况
    if (performButtonActionOnTouchDown(event)) {
        break;
    }

    // 向上遍历，判断view是否处于一个可滚动的容器中
    boolean isInScrollingContainer = isInScrollingContainer();

    // 如果是在一个滚动容器中，稍微延迟反馈从而判断这是否是一个滚动操作
    if (isInScrollingContainer) {
        mPrivateFlags |= PFLAG_PREPRESSED;
        if (mPendingCheckForTap == null) {
            mPendingCheckForTap = new CheckForTap();		// 新建一个对象用来检测单击事件
        }
        mPendingCheckForTap.x = event.getX();
        mPendingCheckForTap.y = event.getY();
        // 利用消息队列来延迟发送检测单击事件的方法，会有一个延迟执行的事件
        postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());
    } else {		// 不在滚动容器中，那么立即响应触摸事件
        setPressed(true, x, y);
        checkForLongClick(
            ViewConfiguration.getLongPressTimeout(),
            x,
            y,
            TOUCH_GESTURE_CLASSIFIED__CLASSIFICATION__LONG_PRESS);
    }
    break;


// 辅助方法
// 主要是特殊处理了来自鼠标点击的情况
protected boolean performButtonActionOnTouchDown(MotionEvent event) {
    if (event.isFromSource(InputDevice.SOURCE_MOUSE) &&
        (event.getButtonState() & MotionEvent.BUTTON_SECONDARY) != 0) {
        showContextMenu(event.getX(), event.getY());
        mPrivateFlags |= PFLAG_CANCEL_NEXT_UP_EVENT;
        return true;
    }
    return false;
}

// 内部类，是一个延迟单击检测的runnable对象
// 如果超时之后，还在队列中，那么说明这就是一个单击的事件，就进行触摸反馈，并检测长按
// 我的理解：如果该事件是一个滑动事件，那么是交给上层的滑动控件处理，而控件会向下发送CANCEL，
// 那么Android会将对应view的尚未执行的信息给清除掉（不知道是不是这个机制），所以该消息就没了
// 反之，如果还存在就说明该事件确实是一个点击事件，就还在消息队列中，还执行到了run()方法，那么立即执行响应
private final class CheckForTap implements Runnable {
    public float x;
    public float y;

    @Override
    public void run() {
        mPrivateFlags &= ~PFLAG_PREPRESSED;
        setPressed(true, x, y);
        final long delay =
            ViewConfiguration.getLongPressTimeout() - ViewConfiguration.getTapTimeout();
        checkForLongClick(delay, x, y, TOUCH_GESTURE_CLASSIFIED__CLASSIFICATION__LONG_PRESS);
    }
}
// 检测长按，也是利用了延迟的消息
private void checkForLongClick(long delay, float x, float y, int classification) {
    if ((mViewFlags & LONG_CLICKABLE) == LONG_CLICKABLE || (mViewFlags & TOOLTIP) == TOOLTIP) {
        mHasPerformedLongPress = false;

        if (mPendingCheckForLongPress == null) {
            mPendingCheckForLongPress = new CheckForLongPress();
        }
        mPendingCheckForLongPress.setAnchor(x, y);
        mPendingCheckForLongPress.rememberWindowAttachCount();
        mPendingCheckForLongPress.rememberPressedState();
        mPendingCheckForLongPress.setClassification(classification);
        postDelayed(mPendingCheckForLongPress, delay);		// 延迟的消息
    }
}
// 上面方法对应的runnable对象对应的类
private final class CheckForLongPress implements Runnable {
    private int mOriginalWindowAttachCount;
    private float mX;
    private float mY;
    private boolean mOriginalPressedState;
    /**
         * The classification of the long click being checked: one of the
         * FrameworkStatsLog.TOUCH_GESTURE_CLASSIFIED__CLASSIFICATION__* constants.
         */
    private int mClassification;

    @UnsupportedAppUsage
    private CheckForLongPress() {
    }

    @Override
    public void run() {		// 超时后还在消息队列中，则会轮到执行
        if ((mOriginalPressedState == isPressed()) && (mParent != null)
            && mOriginalWindowAttachCount == mWindowAttachCount) {
            recordGestureClassification(mClassification);
            if (performLongClick(mX, mY)) {		// 会执行到这边
                mHasPerformedLongPress = true;
            }
        }
    }

    public void setAnchor(float x, float y) {
        mX = x;
        mY = y;
    }

    public void rememberWindowAttachCount() {
        mOriginalWindowAttachCount = mWindowAttachCount;
    }

    public void rememberPressedState() {
        mOriginalPressedState = isPressed();
    }

    public void setClassification(int classification) {
        mClassification = classification;
    }
}
// 上面run方法调用到的
public boolean performLongClick(float x, float y) {
    mLongClickX = x;
    mLongClickY = y;
    final boolean handled = performLongClick();
    mLongClickX = Float.NaN;
    mLongClickY = Float.NaN;
    return handled;
}
public boolean performLongClick() {
    return performLongClickInternal(mLongClickX, mLongClickY);
}
// 本质上会调用该方法
private boolean performLongClickInternal(float x, float y) {
    sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_LONG_CLICKED);

    boolean handled = false;
    final ListenerInfo li = mListenerInfo;
    if (li != null && li.mOnLongClickListener != null) {	// 判断是否注册了长按事件监听器——是，就执行
        handled = li.mOnLongClickListener.onLongClick(View.this);
    }
    if (!handled) {		// 如果没有处理，则先判断长按是否有锚点，然后根据此显示可能存在的菜单
        final boolean isAnchored = !Float.isNaN(x) && !Float.isNaN(y);
        handled = isAnchored ? showContextMenu(x, y) : showContextMenu();
    }
    if ((mViewFlags & TOOLTIP) == TOOLTIP) {		// 如果是显示工具栏
        if (!handled) {
            handled = showLongClickTooltip((int) x, (int) y);
        }
    }
    if (handled) {		// 如果正确处理，就显示一个触觉反馈
        performHapticFeedback(HapticFeedbackConstants.LONG_PRESS);
    }
    return handled;
}
```

理解：DOWN下的主要操作：

1. 判断当前的view是否处在可滑动的容器中，如果是，则当前的DOWN操作可能是滑动的前兆，所以需要延迟等待一下——就用延迟消息，将消息按照handler机制发送出去，如果超时还在队列中，说明是普通的点击事件；不在，就说明上层的滑动容器已经处理了，发送CANCEL已经取消了操作
2. 如果是简单的点击操作，那么去判断是否是长按操作，也是通过延迟消息判断的——这边会涉及到长按事件监听器`OnLongClickListener`——可以发现，长按不需要DOWN-UP完整序列，只需要DOWN即可

#### 2. ACTION_MOVE的处理

```java
case MotionEvent.ACTION_MOVE:
    if (clickable) {
        drawableHotspotChanged(x, y);		// 通知子view或者drawable触摸点发生了移动
    }

    final int motionClassification = event.getClassification();
    final boolean ambiguousGesture =			// 是否是模棱两可的手势
        motionClassification == MotionEvent.CLASSIFICATION_AMBIGUOUS_GESTURE;
    int touchSlop = mTouchSlop;
    if (ambiguousGesture && hasPendingLongPressCallback()) {、
        // 检测是否移动超出了范围，touchSlop是稍微移出范围还是能够正常判断点击的
        // 系统并不知道用户的意图，所以即使滑出了view的范围，并不会取消长按标志——给一定的误差和时间
        if (!pointInView(x, y, touchSlop)) {
            // 移除长按的事件
            removeLongPressCallback();
            long delay = (long) (ViewConfiguration.getLongPressTimeout()
                                 * mAmbiguousGestureMultiplier);
            // 扣掉已经经过的事件，再次发送延迟消息——感觉是再给一次机会
            delay -= event.getEventTime() - event.getDownTime();
            checkForLongClick(
                delay,
                x,
                y,
                TOUCH_GESTURE_CLASSIFIED__CLASSIFICATION__LONG_PRESS);
        }
        touchSlop *= mAmbiguousGestureMultiplier;
    }

    // 判断触控点的位置是否还在view的范围内
	// touchSlop是一个小范围的误差，超出view位置slop距离依旧判定为在view范围内
    if (!pointInView(x, y, touchSlop)) {
        // 超出范围了，那么立即移除相关的超时等待消息——说明是滑动操作
        removeTapCallback();
        removeLongPressCallback();
        if ((mPrivateFlags & PFLAG_PRESSED) != 0) {
            setPressed(false);			// 取消按下的状态
        }
        mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
    }
	
	// 表示用户在屏幕上用力按下，那么需要加快响应速度
    final boolean deepPress =
        motionClassification == MotionEvent.CLASSIFICATION_DEEP_PRESS;
    if (deepPress && hasPendingLongPressCallback()) {
        // 移除原来的长按标志，直接响应长按事件
        removeLongPressCallback();
        checkForLongClick(
            0 /* send immediately */,
            x,
            y,
            TOUCH_GESTURE_CLASSIFIED__CLASSIFICATION__DEEP_PRESS);
    }

    break;
```

#### 3. ACTION_UP的处理

```java
case MotionEvent.ACTION_UP:
    mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
    if ((viewFlags & TOOLTIP) == TOOLTIP) {		// 如果是长按显示工具提示tooltip，那么调用对应的方法
        handleTooltipUp();		// 工具栏的显示额外处理
    }
    if (!clickable) {			// 如果是不可点击的view
        removeTapCallback();
        removeLongPressCallback();			// 移除长按
        mInContextButtonPress = false;
        mHasPerformedLongPress = false;
        mIgnoreNextUpEvent = false;
        break;
    }
    boolean prepressed = (mPrivateFlags & PFLAG_PREPRESSED) != 0;
	// 如果有pressed标志
    if ((mPrivateFlags & PFLAG_PRESSED) != 0 || prepressed) {
        // 没有获取焦点，就应该去获得，并且进入到touch的状态
        boolean focusTaken = false;
        // 正常的触摸模式下是不需要获取焦点，例如Button
        // 但是如果在按键模式下，需要先移动光标选中按钮，也就是获取focus
        // 再点击确认触摸按钮事件
        if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {
            focusTaken = requestFocus();
        }

        if (prepressed) {
            // 用户手指已经抬起，而此时还没有显示点击的效果。所以要保证马上显示点击状态——在执行点击内容之前
            // ——说明这是一个单击事件，对应的延迟执行的消息队列中的消息还没来得及执行——所以需要立即反馈
            setPressed(true, x, y);
        }
		
        // 长按事件未被响应了且不忽略本次UP事件
        if (!mHasPerformedLongPress && !mIgnoreNextUpEvent) {
            // 长按的延迟消息还没执行点击事件就已经到尾声，所以移除长按的消息——说明是一个单击事件，移除长按表直
            removeLongPressCallback();

            // 只有处于按下状态时才能执行点击操作——当没有获得焦点状态时才能触发点击事件
            // ——说明一个可以获得焦点的view是无法触发点击事件的
            if (!focusTaken) {
                // 使用runnable执行，而不是直接调用该方法，主要是为了在执行之前能够更新view的视觉状态
                if (mPerformClick == null) {
                    mPerformClick = new PerformClick();
                }
                if (!post(mPerformClick)) {			// 如果post没有成功，就直接调用该方法
                    performClickInternal();	
                }
            }
        }
		// runnable方法，用来取消view的prepressed或者pressed状态
        if (mUnsetPressedState == null) {
            mUnsetPressedState = new UnsetPressedState();
        }

        if (prepressed) {
            // 取消已按下状态
            postDelayed(mUnsetPressedState,
                        ViewConfiguration.getPressedStateDuration());
        } else if (!post(mUnsetPressedState)) {	// 如果post没有成功，就直接调用该方法
            mUnsetPressedState.run();
        }
        removeTapCallback();		// 清除单击检测消息
    }
    mIgnoreNextUpEvent = false;
    break;

// 辅助方法
private final class PerformClick implements Runnable {
    @Override
    public void run() {
        recordGestureClassification(TOUCH_GESTURE_CLASSIFIED__CLASSIFICATION__SINGLE_TAP);
        performClickInternal();
    }
}
private boolean performClickInternal() {
    // 必须在执行单击操作之前通知自动填充管理器，避免app存在点击监听器修改相关自动填充器的状态
    notifyAutofillManagerOnClick();

    return performClick();
}
public boolean performClick() {
    // 防止这个方法被单独调用，所以还是要再执行一遍下边的方法
    notifyAutofillManagerOnClick();

    final boolean result;
    final ListenerInfo li = mListenerInfo;
    if (li != null && li.mOnClickListener != null) {
        playSoundEffect(SoundEffectConstants.CLICK);
        li.mOnClickListener.onClick(this);		// 调用点击事件处理器
        result = true;
    } else {
        result = false;
    }

    sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);

    notifyEnterOrExitForAutoFillIfNeeded(true);

    return result;
}
```

理解：UP主要是来处理点击事件的，而如果该view能够获得焦点，则无法处理点击事件

#### 4. ACTION_CANCEL的处理

```java
 case MotionEvent.ACTION_CANCEL:
    if (clickable) {
        setPressed(false);		// 取消显示按下效果
    }
    removeTapCallback();			// 取消一系列触摸动作
    removeLongPressCallback();
    mInContextButtonPress = false;
    mHasPerformedLongPress = false;
    mIgnoreNextUpEvent = false;
    mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
    break;
```

主要参考：

1. https://juejin.cn/post/6918272111152726024
2. https://juejin.cn/post/6920883974952714247
3. https://www.gcssloop.com/customview/dispatch-touchevent-theory.html
4. https://www.gcssloop.com/customview/dispatch-touchevent-source.html
5. https://www.gcssloop.com/customview/motionevent.html
6. https://blog.csdn.net/guolin_blog/article/details/9097463
7. https://blog.csdn.net/guolin_blog/article/details/9153747
8. [《Android开发艺术探索》](https://book.douban.com/subject/26599538/)
9. http://wuxiaolong.me/2015/12/19/MotionEvent/
10. https://www.viseator.com/2017/09/14/android_view_event_1/（偏底层硬件）
11. https://www.viseator.com/2017/09/14/android_view_event_2/（偏底层硬件）
12. https://www.viseator.com/2017/11/02/android_view_event_3/

画图工具：

1. ProcessOn（主要是参考上面的博客）——内容几乎是完全按照上面博客依葫芦画瓢的，作为自己已经理解的证明（还能熟悉processOn工具的使用方法，提高审美）



