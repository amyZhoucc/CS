# Window和WindowManager

在学习Android的事件分发机制的时候，接触到过Window，这个是事件传递的一个环节，但是并没有详细的了解，正好参考书和一篇好的博客有讲，于是再做个较为深入的理解。在实习的时候用到过popUpWindow，也涉及到过出现遮盖的问题，本质上还是因为应用级的window和popUpWindow本质上是两个window级别的，所以popUpWindow因为创建在后面，势必出现遮挡问题

## what is Window

Android允许一个屏幕上出现多个window

> **Window是：负责view的绘制和事件分发（即触摸事件的传递）**
>
> popUpWindow、Toast、Dialog、Menu都是相对于页面的单独window，都是需要通过window创建的

window和view树是一一对应的。

**view树是window的存在形式，window是view树的载体**：

1. view树里面的每个view是我们看得到的，而**window是不可见的**，它只是一个概念，本身并不存在，所以view树是存在形式
2. window是view树的管理者，负责统筹，window负责控制view的如何展示，所以window是view树的载体

eg：班集体，这就是个概念，并不真实存在，它的存在形式是整个班的学生，当学生不存在，那么班集体就不存在，但是概念仍然存在；而班集体可以作为一个整体进行活动，所以它是学生们的载体（ps：这个比喻真的太赞了）

## Window的各种属性

### 1. type属性

字面意思就是window的类型，有3种：应用window、子window、系统window——主要控制创建出来的window的级别，从而控制window在屏幕上的展示

1. 应用window，**一个activity对应一个应用window**
2. 子window，**不能单独存在，需要依附在父window上的**，eg：Dialog
3. 系统window，需要获取系统权限，才能创建的window，eg：Toast，系统状态栏等

window是分层的，多了一个z轴，**z-ordered**，层级大的会覆盖在小的上面

1. 应用window：1~99
2. 子window：1000~1999
3. 系统window：2000~2999

——数字越大，层级越高，如果要window位于所有window的顶层，将层级设置为系统级。系统级也有很多值，常用的来设置的有：

1. `TYPE_SYSTEM_OVERLAY`：建议用这个
2. `TYPE_SYSTEM_ERROR`：`@deprecated`

设置方法：`mLayoutParams.type = LayoutParams.TYPE_SYSTEM_ERROR;` 同时申请权限：`<user-permission android:name="android.permission.SYSTEM_ALERT_WINDOW">`

否则，在创建window的时候，系统会报错：Exception:  Unable  to  add  window  android.view.ViewRootImpl$W@42882fe8  -- **permission denied for this window type**

#### type属性的举例

```java
// WindowManager.java api30
public static final int FIRST_APPLICATION_WINDOW = 1;		// 应用程序window的开始值
public static final int TYPE_BASE_APPLICATION = 1;		// 应用程序window的基础值

public static final int TYPE_APPLICATION = 2;			// 普通应用程序的值

public static final int TYPE_APPLICATION_STARTING = 3;		// 特殊的应用程序窗口，在window可以正常显示之前，用这个window来显示东西

// TYPE_APPLICATION 的变体，在应用程序显示之前，WindowManager 会等待这个 Window 绘制完毕
public static final int TYPE_DRAWN_APPLICATION = 4;

public static final int LAST_APPLICATION_WINDOW = 99;			// 应用程序window的结束值
```

```java
// WindowManager.java api30
public static final int FIRST_SUB_WINDOW = 1000;			// 子window的开始值

// 应用程序 Window 顶部的面板。这些 Window 出现在其附加 Window 的顶部。
public static final int TYPE_APPLICATION_PANEL = FIRST_SUB_WINDOW;

// 用于显示媒体(如视频)的 Window。这些 Window 出现在其附加 Window 的后面。
public static final int TYPE_APPLICATION_MEDIA = FIRST_SUB_WINDOW + 1;

// 应用程序 Window 顶部的子面板。这些 Window 出现在其附加 Window 和任何Window的顶部
public static final int TYPE_APPLICATION_SUB_PANEL = FIRST_SUB_WINDOW + 2;

// 当前Window的布局和顶级Window布局相同时，不能作为子代的容器
public static final int TYPE_APPLICATION_ATTACHED_DIALOG = FIRST_SUB_WINDOW + 3;

// 用显示媒体 Window 覆盖顶部的 Window， 这是系统隐藏的 API
public static final int TYPE_APPLICATION_MEDIA_OVERLAY  = FIRST_SUB_WINDOW + 4;

// 子面板在应用程序Window的顶部，这些Window显示在其附加Window的顶部， 这是系统隐藏的 API
public static final int TYPE_APPLICATION_ABOVE_SUB_PANEL = FIRST_SUB_WINDOW + 5;

public static final int LAST_SUB_WINDOW = 1999;			// 子window类型的结束值
```

```java
// WindowManager.java api30
public static final int FIRST_SYSTEM_WINDOW = 2000;		// 系统Window类型的开始值

// 系统状态栏，只能有一个状态栏，它被放置在屏幕的顶部，所有其他窗口都向下移动
public static final int TYPE_STATUS_BAR = FIRST_SYSTEM_WINDOW;

// 系统搜索窗口，只能有一个搜索栏，它被放置在屏幕的顶部
public static final int TYPE_SEARCH_BAR = FIRST_SYSTEM_WINDOW+1;

// 已经从系统中被移除，可以使用 TYPE_KEYGUARD_DIALOG 代替
public static final int TYPE_KEYGUARD = FIRST_SYSTEM_WINDOW+4;

// 系统对话框窗口
public static final int TYPE_SYSTEM_DIALOG = FIRST_SYSTEM_WINDOW+8;

// 锁屏时显示的对话框
public static final int TYPE_KEYGUARD_DIALOG = FIRST_SYSTEM_WINDOW+9;


// 输入法窗口，位于普通 UI 之上，应用程序可重新布局以免被此窗口覆盖
public static final int TYPE_INPUT_METHOD = FIRST_SYSTEM_WINDOW+11;

// 输入法对话框，显示于当前输入法窗口之上
public static final int TYPE_INPUT_METHOD_DIALOG= FIRST_SYSTEM_WINDOW+12;

// 墙纸
public static final int TYPE_WALLPAPER = FIRST_SYSTEM_WINDOW+13;

// 状态栏的滑动面板
public static final int TYPE_STATUS_BAR_PANEL = FIRST_SYSTEM_WINDOW+14;

// 应用程序叠加窗口显示在所有窗口之上（用来替换 TYPE_SYSTEM_ERROR）
public static final int TYPE_APPLICATION_OVERLAY = FIRST_SYSTEM_WINDOW + 38;

public static final int LAST_SYSTEM_WINDOW = 2999;		// 系统Window类型的结束值
```

### 2. flag属性

flag的主要作用：

1. 各种情景下的显示逻辑，例如游戏状态下、锁屏状态下的显示
2. 触控事件的处理逻辑，交给哪个window处理等

```java
// 当 Window 可见时允许锁屏
public static final int FLAG_ALLOW_LOCK_WHILE_SCREEN_ON = 0x00000001;

// Window 后面的内容都变暗
public static final int FLAG_DIM_BEHIND = 0x00000002;

// Window 不能获得输入焦点，即不接受任何按键或按钮事件，例如该 Window 上 有 EditView，点击 EditView 是 不会弹出软键盘的
// Window 范围外的事件依旧为原窗口处理；例如点击该窗口外的view，依然会有响应。另外只要设置了此Flag，都将会启用FLAG_NOT_TOUCH_MODAL
public static final int FLAG_NOT_FOCUSABLE = 0x00000008;

// 设置了该 Flag,将 Window 之外的按键事件发送给后面的 Window 处理, 而自己只会处理 Window 区域内的触摸事件
// Window 之外的 view 也是可以响应 touch 事件。
public static final int FLAG_NOT_TOUCH_MODAL  = 0x00000020;

// 设置了该Flag，该 Window 将不会接受任何 touch 事件，例如点击该 Window 不会有响应，只会传给下面有聚焦的窗口。
public static final int FLAG_NOT_TOUCHABLE      = 0x00000010;

// 只要 Window 可见时屏幕就会一直亮着
public static final int FLAG_KEEP_SCREEN_ON     = 0x00000080;

// 允许 Window 占满整个屏幕
public static final int FLAG_LAYOUT_IN_SCREEN   = 0x00000100;

// 允许 Window 超过屏幕之外
public static final int FLAG_LAYOUT_NO_LIMITS   = 0x00000200;

// 全屏显示，隐藏所有的 Window 装饰，比如在游戏、播放器中的全屏显示
public static final int FLAG_FULLSCREEN      = 0x00000400;

// 表示比FLAG_FULLSCREEN低一级，会显示状态栏
public static final int FLAG_FORCE_NOT_FULLSCREEN   = 0x00000800;

// 当用户的脸贴近屏幕时（比如打电话），不会去响应此事件
public static final int FLAG_IGNORE_CHEEK_PRESSES    = 0x00008000;

// 则当按键动作发生在 Window 之外时，将接收到一个MotionEvent.ACTION_OUTSIDE事件。
public static final int FLAG_WATCH_OUTSIDE_TOUCH = 0x00040000;

@Deprecated
// 窗口可以在锁屏的 Window 之上显示, 使用 Activity#setShowWhenLocked(boolean) 方法代替
public static final int FLAG_SHOW_WHEN_LOCKED = 0x00080000;

// 表示负责绘制系统栏背景。如果设置，系统栏将以透明背景绘制，
// 此 Window 中的相应区域将填充 Window＃getStatusBarColor()和 Window＃getNavigationBarColor()中指定的颜色。
public static final int FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS = 0x80000000;

// 表示要求系统壁纸显示在该 Window 后面，Window 表面必须是半透明的，才能真正看到它背后的壁纸
public static final int FLAG_SHOW_WALLPAPER = 0x00100000;
```

==`FLAG_NOT_FOCUSABLE`==，Window不需要获取焦点，也不需要接受任何的输入事件，并且默认启动下面的FLAG_NOT_TOUCH_MODAL，**最终事件都会传递给下层的具有焦点的window**（否则下层的都收不到事件了）

——理解：说明这个window就是个展示用的（不点击、不输入）

==`FLAG_NOT_TOUCH_MODAL`==，系统会将当前window区域以外的单击事件传递给底层的window，而在区域内的单击事件自己处理。——一般都需要开启，否则其他window将收不到事件

==`FLAG_SHOW_WHEN_LOCKED`==，该window能够显示在锁屏界面上

### 3. softInputMode属性

主要负责的是：当软键盘弹起时，window的处理逻辑。

常见的：输入框会被顶上去；也可以设置覆盖

```java
// 没有指定状态，系统会选择一个合适的状态或者依赖于主题的配置
public static final int SOFT_INPUT_STATE_UNCHANGED = 1;

// 当用户进入该窗口时，隐藏软键盘
public static final int SOFT_INPUT_STATE_HIDDEN = 2;

// 当窗口获取焦点时，隐藏软键盘
public static final int SOFT_INPUT_STATE_ALWAYS_HIDDEN = 3;

// 当用户进入窗口时，显示软键盘
public static final int SOFT_INPUT_STATE_VISIBLE = 4;

// 当窗口获取焦点时，显示软键盘
public static final int SOFT_INPUT_STATE_ALWAYS_VISIBLE = 5;

// window会调整大小以适应软键盘窗口
public static final int SOFT_INPUT_MASK_ADJUST = 0xf0;

// 没有指定状态,系统会选择一个合适的状态或依赖于主题的设置
public static final int SOFT_INPUT_ADJUST_UNSPECIFIED = 0x00;

// 当软键盘弹出时，窗口会调整大小,例如点击一个EditView，整个layout都将平移可见且处于软件盘的上方
// 同样的该模式不能与SOFT_INPUT_ADJUST_PAN结合使用；
// 如果窗口的布局参数标志包含FLAG_FULLSCREEN，则将忽略这个值，窗口不会调整大小，但会保持全屏。
public static final int SOFT_INPUT_ADJUST_RESIZE = 0x10;

// 当软键盘弹出时，窗口不需要调整大小, 要确保输入焦点是可见的,
// 例如有两个EditView的输入框，一个为Ev1，一个为Ev2，当点击Ev1想要输入数据时，当前的Ev1的输入框会移到软键盘上方
// 该模式不能与SOFT_INPUT_ADJUST_RESIZE结合使用
public static final int SOFT_INPUT_ADJUST_PAN = 0x20;

// 将不会调整大小，直接覆盖在window上
public static final int SOFT_INPUT_ADJUST_NOTHING = 0x30;
```

### 4. 其他属性

常用的属性：

1. x、y属性：确定window在屏幕中的位置
2. alpha：确定window的透明度
3. gravity：确定在屏幕中的位置，使用的是Gravity类中的常量
4. format：window的像素点格式，使用的是PixelFormat类中的常量

### 5. window属性赋值方法

`LayoutParams`是`WindowManager`的一个静态内部类，主要是用来获取常量

1. 创建window后赋值

   ```java
   WindowManager.LayoutParams windowParams = new WindowManager.LayoutParams();
   windowParams.flag = WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE;
   TextView view = new TextView(this);
   getWindowManager.addView(view, windowParams);
   ```

2. 直接给window赋值

   **`getWindow().flag = WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE`**

   ——获取window对象，后直接赋值即可

而`softInputMode`可以直接再`AndroidManifest.xml`中设置——当然也是可以在代码中设置的

<activity android:windowSoftInputMode="adjustNothing" />

## Window的添加过程

创建可以通过WindowManager来实现

WindowManager提供了3个方法：

1. **==`addView`==**：添加view

   首先，需要有view和windowManager

   常见的创建方法：

   ```java
   // 创建view
   Button button = new Button(this);
   // 创建windowManager
   WindowManager.LayoutParams layoutParams = new WindowManager.LayoutParams();
   // layoutParams.flag = ....
   getWindowManager().addView(button, layoutParams);
   ```

2. **==`updateViewLayout`==**：更新view

3. **==`deleteView`==**：删除view

## Window的关键类

Window的实现位于WindowMangagerService中

WindowManager和WindowManagerService是通过IPC进行交互的

给一个结论：

> 和 WindowManager 相关的类：
>
> 1. ViewManager类
> 2. WindowManager类
> 3. WindowManagerImpl类
> 4. WindowManagerGlobal类 
> 5. IWindowSession类
>
> View的相关类：
>
> 1. ViewRootImpl类
>
>    作用：
>
>    - **负责连接view和window**
>    - **负责和WMS联系**
>    - **负责管理和绘制view树**
>    - **是事件的中转站**

### 1. WindowManager

WindowManager是外界访问Window的入口。

WindowManager是实现了ViewManager接口（WindowManager本身也是interface类型），实现了里面的三个方法：

```java
public interface ViewManager{
    public void addView(View view, ViewGroup.LayoutParams params);
    public void updateViewLayout(View view, ViewGroup.LayoutParams params);
    public void removeView(View view);
}
```

——创建一个window，并向里面添加view；更新window中的view；删除里面所有的view，就删除了整个window

所以可以发现：**WindowManager不是管理window的，而是管理window里面的view的**

WindowManager的真正实现是WindowManagerImpl类

```java
// WindowManagerImpl.java api30
private final WindowManagerGlobal mGlobal = WindowManagerGlobal.getInstance();	// 全局单例

public void addView(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
    applyDefaultToken(params);
    mGlobal.addView(view, params, mContext.getDisplayNoVerify(), mParentWindow,
                    mContext.getUserId());
}

public void updateViewLayout(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
    applyDefaultToken(params);
    mGlobal.updateViewLayout(view, params);
}

public void removeView(View view) {
    mGlobal.removeView(view, false);
}
```

在实现上看，本质上去调用了`WindowManagerGlobal`类的对应实现。而`WindowMangerGlobal`是以单例模式的形式给出实例的，而`WindowMangerImpl`这种持有某个对象引用，是典型的**桥接模式**，他将所有工作都交给`WindowManagerGlobal`去实现

### 2. WindowManagerGlobal类

`WindowManagerGlobal`类是具体负责实现`ViewManager`三个方法的类

`WindowManagerGlobal`的较为关注的变量：

```java
// 所有window对应的view
private final ArrayList<View> mViews = new ArrayList<View>();
// 所有window对应的ViewRootImpl（即view树的根）
private final ArrayList<ViewRootImpl> mRoots = new ArrayList<ViewRootImpl>();
// 所有window对应的布局参数
private final ArrayList<WindowManager.LayoutParams> mParams =
    new ArrayList<WindowManager.LayoutParams>();
// 所有正在被删除的view对象，即已经调用removeView，但是还未完成删除的view对象
private final ArraySet<View> mDyingViews = new ArraySet<View>();
```



#### 1. `addView()`源码分析

```java
public void addView(View view, ViewGroup.LayoutParams params,
            Display display, Window parentWindow, int userId) {
    if (view == null) {
        throw new IllegalArgumentException("view must not be null");
    }
    if (display == null) {
        throw new IllegalArgumentException("display must not be null");
    }
    if (!(params instanceof WindowManager.LayoutParams)) {
        throw new IllegalArgumentException("Params must be WindowManager.LayoutParams");
    }
	// 到上面这段是检查参数是否合法
    final WindowManager.LayoutParams wparams = (WindowManager.LayoutParams) params;
    if (parentWindow != null) {		// 如果不是子窗口（父窗口），会对其中的参数进行调整
        parentWindow.adjustLayoutParamsForSubWindow(wparams);
    } else {
        // If there's no parent, then hardware acceleration for this view is
        // set from the application's hardware acceleration setting.
        final Context context = view.getContext();
        if (context != null
            && (context.getApplicationInfo().flags
                & ApplicationInfo.FLAG_HARDWARE_ACCELERATED) != 0) {
            wparams.flags |= WindowManager.LayoutParams.FLAG_HARDWARE_ACCELERATED;
        }
    }			

    ViewRootImpl root;
    View panelParentView = null;

    synchronized (mLock) {			// 加锁，防止并发
        // Start watching for system property changes.
        if (mSystemPropertyUpdater == null) {
            mSystemPropertyUpdater = new Runnable() {
                @Override public void run() {
                    synchronized (mLock) {
                        for (int i = mRoots.size() - 1; i >= 0; --i) {
                            mRoots.get(i).loadSystemProperties();
                        }
                    }
                }
            };
            SystemProperties.addChangeCallback(mSystemPropertyUpdater);
        }

        int index = findViewLocked(view, false);
        if (index >= 0) {
            if (mDyingViews.contains(view)) {
                // Don't wait for MSG_DIE to make it's way through root's queue.
                mRoots.get(index).doDie();
            } else {
                throw new IllegalStateException("View " + view
                                                + " has already been added to the window manager.");
            }
            // The previous removeView() had not completed executing. Now it has.
        }

        // If this is a panel window, then find the window it is being
        // attached to for future reference.
        if (wparams.type >= WindowManager.LayoutParams.FIRST_SUB_WINDOW &&
            wparams.type <= WindowManager.LayoutParams.LAST_SUB_WINDOW) {
            final int count = mViews.size();
            for (int i = 0; i < count; i++) {
                if (mRoots.get(i).mWindow.asBinder() == wparams.token) {
                    panelParentView = mViews.get(i);
                }
            }
        }

        root = new ViewRootImpl(view.getContext(), display);	// 创建viewRootImpl，并设置了参数（可以发现，每次调用addView，都会创建一个viewRootImpl，即也是创建一个Window）

        view.setLayoutParams(wparams);		// 设置布局

        mViews.add(view);				// 添加view
        mRoots.add(root);				// 添加viewRootImpl
        mParams.add(wparams);			// 添加布局

        // 通过新建好的viewRootImpl来添加view
        try {
            root.setView(view, wparams, panelParentView, userId);
        } catch (RuntimeException e) {
            // BadTokenException or InvalidDisplayException, clean up.
            if (index >= 0) {
                removeViewLocked(index, true);
            }
            throw e;
        }
    }
}
```

而viewRootImpl的具体`addView`方法：

```java
public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView,
            int userId) {
    synchronized (this) {
        ...
            try {
                mOrigWindowType = mWindowAttributes.type;
                mAttachInfo.mRecomputeGlobalAttributes = true;
                collectViewAttributes();
                adjustLayoutParamsForCompatibility(mWindowAttributes);
                // 调用了windowSession的方法，而它会去调用WMS的方法，把添加window的逻辑交给WMS去实现
                res = mWindowSession.addToDisplayAsUser(mWindow, mSeq, mWindowAttributes,
                                                        getHostVisibility(), mDisplay.getDisplayId(), userId, mTmpFrame,
                                                        mAttachInfo.mContentInsets, mAttachInfo.mStableInsets,
                                                        mAttachInfo.mDisplayCutout, inputChannel,
                                                        mTempInsets, mTempControls);
                setFrame(mTmpFrame);
            } catch...
			...
    }
}
```

这边主要关注的是：通过获得`mWindowSession`对象来调用WMS方法，而`mWindowSession`对象

```java
// ViewRootImpl.java api30
final IWindowSession mWindowSession;

// 构造方法之一，直接调用WindowManagerGlobal的静态方法获得
public ViewRootImpl(Context context, Display display) {
    this(context, display, WindowManagerGlobal.getWindowSession(),
         false /* useSfChoreographer */);
}

// WindowManagerGlobal.java api30
// 按照单例的懒加载模式去获得windowSession的变量对象
public static IWindowSession getWindowSession() {
    synchronized (WindowManagerGlobal.class) {
        if (sWindowSession == null) {
            try {
                // Emulate the legacy behavior.  The global instance of InputMethodManager
                // was instantiated here.
                // TODO(b/116157766): Remove this hack after cleaning up @UnsupportedAppUsage
                InputMethodManager.ensureDefaultInstanceForDefaultDisplayIfNecessary();
                IWindowManager windowManager = getWindowManagerService();
                sWindowSession = windowManager.openSession(
                    new IWindowSessionCallback.Stub() {
                        @Override
                        public void onAnimatorScaleChanged(float scale) {
                            ValueAnimator.setDurationScale(scale);
                        }
                    });
            } catch (RemoteException e) {
                throw e.rethrowFromSystemServer();
            }
        }
        return sWindowSession;
    }
}
```

理解：

1. 按照[文章](https://juejin.cn/post/6888688477714841608#heading-8)中的介绍，IWindowSession是一个IBinder接口，具体的实现在`WindowManagerService`类中（:sweat:没有找到该类），本地mWindowSession只是一个Binder对象，通过该对象可以直接调用WMS进行跨进程通信——这句话还有待理解

2. 注意这个session是一个单例，整个应用只有这一个对象，主要是由于WMS是对多个应用的，如果每个viewRootImpl都对应一个session，那么需要处理的任务太多了。

   ——WMS的处理单位是应用，那么windowSession对象对应的也应该是应用

   并且每个应用session有一些数据结构，可以用来保存每个window、viewRootImpl数据

总结：addView的过程：

WindowManager（继承自ViewManager）-> WindowManagerImpl（是WindowManger的实现类）-> WindowManagerGlobal（单例）-> 里面创建viewRootImpl对象 -> viewRootImpl.mWindowSession对象（单例）-> 进程间通信，调用WMS去创建window

### 2. WindowManagerService

就是著名的WMS，**所有的window创建都需要通过WMS开始**，在Android的window机制中，WMS是核心

它决定window该如何显示和如何分发点击事件

## Window的实现类

Window本身是一个抽象类，**唯一的实现类是`PhoneWindow`**，是一个窗口辅助类，

PhoneWindow是一个顶级窗口，它被添加到顶级窗口管理器的顶级视图，其他的window都要添加到这个顶层窗口中——**所以，PhoneWindow是window的容器，而不是view的容器**

一个小点：**PhoneWindow和WindowManagerImpl两者是成对存在**（没有找到证据）

> PhoneWindow的存在意义：
>
> 1. 提供DecorView模板
>
>    <img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20220513164642558.png" alt="image-20220513164642558" style="zoom:60%;" />
>
>    Activity通过setContentView将布局加入到DecorView的content中，那么DecorView就成为背景。所以对DecorView的window进行属性设置，也会影响到布局效果的改变
>
>    ——这就是Window提供标准UI策略，例如背景、标题区域、默认键处理等（我的理解是，PhoneWindow提供了DecorView，而DecorView就是一个模板布局）
>
> 2. 抽离Activity中关于window的逻辑
>
>    关于window的相关操作都给了PhoneWindow去做了，eg：当activity需要添加页面布局，只需要调用`setContentView()`后面就是调用PhoneWindow的方法去实现了
>
> 3. 限制组件添加window权限
>
>    PhoneWindow内部有一个**token属性**，用于验证一个PhoneWindow是否允许添加window
>
>    （在Activity创建PhoneWindow的时候，就会把从AMS传过来的token赋值给他，从而他也就有了添加token的权限。而其他的PhoneWindow则没有这个权限，因而也无法添加window）









参考资料

1. [《Android开发艺术探索》](https://book.douban.com/subject/26599538/)
2. https://juejin.cn/post/6888688477714841608