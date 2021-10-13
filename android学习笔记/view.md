# view的体系

## view的基础知识

View是Android中所有控件的基类

<img src="https://raw.githubusercontent.com/amyZhoucc/CS/main/android%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/pic/inherited.jpg" alt="img" style="zoom: 50%;" />

他们之间存在包含和迭代关系。

### 1. view的位置参数

View的位置主要由它的四个顶点来决定，分别对应于View的四个属性：**top、left、right、bottom**

left：左上角的横坐标、right的右下角的横坐标；top：左上角的纵坐标；bottom：右下角的纵坐标——能够对应算出width和height

在View的源码中它们对应于mLeft、mRight、mTop和mBottom这四个成员变量

<img src="/Users/bytedance/Library/Application Support/typora-user-images/image-20210622153052575.png" alt="image-20210622153052575" style="zoom:50%;" />

并且view中还有其他参数：x、y、translationX和translationY——相对于父容器的坐标

1. x和y是View左上角的坐标，
2. translationX和translationY是View左上角相对于父容器的偏移量，默认是0,0

x = left + translationX, y = top + translationY

View在平移的过程中，**top和left表示的是原始左上角的位置信息**，其值并不会发生改变，此时发生改变的是x、y、translationX和translationY这四个参数

### 2. MotionEvent和TouchSlop

MotionEvent就是手指接触屏幕：

1. ACTION_DOWN——手指刚接触屏幕；
2. ACTION_MOVE——手指在屏幕上移动；
3. ACTION_UP——手机从屏幕上松开的一瞬间。

点击屏幕后离开松开，事件序列为DOWN -> UP；

点击屏幕滑动一会再松开，事件序列为DOWN -> MOVE -> … > MOVE ->UP。

==getX/getY返回的是相对于当前View左上角的x和y坐标，而getRawX/getRawY返回的是相对于手机屏幕左上角的x和y坐标。==

TouchSlop是系统所能识别出的被认为**是滑动的最小距离**。如果手指移动的距离不超过该touchslop，那么不认为是在滑动

这是一个**常量，和设备有关**，在不同设备上这个值可能是不同的

——`ViewConfiguration.get(getContext()).getScaledTouchSlop()`

```kotlin
// 自定义的控件，该控件能够根据点击
// TextView也是类似的
class MyEditText(context: Context, attributeSet: AttributeSet) :
    AppCompatEditText(context, attributeSet) {
    private var lastX = 0;		// 保存上次的位置
    private var lastY = 0;

    override fun onTouchEvent(event: MotionEvent?): Boolean {
        super.onTouchEvent(event)			// 如果没有调用父类的方法，那么
        val x = event?.getX()			// 点击事件的具体坐标
        val y = event?.getY()	
        when (event?.action) {
            MotionEvent.ACTION_DOWN -> {		// 如果只是按下，那么更新当前坐标
                lastX = x?.toInt() ?: 0
                lastY = y?.toInt() ?: 0
            }
            MotionEvent.ACTION_MOVE -> {		// 如果是移动，那么，计算移动的距离
                val offsetX = (x?.toInt() ?: 0) - lastX
                val offsetY = (y?.toInt() ?: 0) - lastY
                layout(left + offsetX, top + offsetY, right + offsetY, bottom + offsetY)	// 直接更新该控件的坐标
            }
        }
        return true
    }
}
```

——这样就实现了一个自定义控件，只需要直接调用即可

### 3. VelocityTracker、GestureDetector和Scroller

### 3.1 VelocityTracker

VelocityTracker：速度追踪，用于**追踪手指在滑动过程中的速度**，包括水平和竖直方向的速度。

首先要追踪，所以在`onTouchEvent()`重写，并且在里面调用如下代码：

```kotlin
override fun onTouchEvent(event: MotionEvent): Boolean{
  velocityTracker = VelocityTracker.obtain()		// 获得该对象
  velocityTracker?.addMovement(event)				// 然后去关注该点击事件
  velocityTracker?.computeCurrentVelocity(100)			// 先计算当前的速度——采样的时间长度，单位是ms
  xVelocity = velocityTracker?.xVelocity as Int		// 该采样时间中的x移过的长度，单位是像素
  yVelocity = velocityTracker?.yVelocity as Int
  ....

  ...
  velocityTracker?.clear()				// 不需要的时候需要清空，并且回收
  velocityTracker?.recycle()
  return true
}
```

<img src="/Users/bytedance/Library/Application Support/typora-user-images/image-20210622154901041.png" alt="image-20210622154901041" style="zoom: 67%;" />

所以步骤是：

1. 先注册`obtain` -> 添加移动事件`addMovement(event)`
2. 测速，首先需要定义采样的时间长度`computeCurrentVelocity`然后获得该段时间内的x/y移动距离`xVelocity/yVelocity`
3. 回收：`clear`,`recycle`

### 3.2 GestureDetector

手势检测，用于辅助检测用户的单击、滑动、长按、双击等行为。

背景：普通的控件主要是实现在View的onTouchEvent方法中实现所需的监听，而不需要使用GestureDetector。只是监听滑动相关的，建议自己在onTouchEvent中实现，如果要监听双击这种行为的话，那么就使用GestureDetector。

GestureDetector.OnGestureListener 和 GestureDetector.OnDoubleTapListener一共9个方法，如果两个都需要使用则均需实现

```kotlin
// 单独写一个文件，GestureDetector.OnGestureListener实现接口：6个方法
class GestureListener: GestureDetector.OnGestureListener{
    private val TAG = "MY_ACTIVITY"
    override fun onDown(e: MotionEvent?): Boolean {
        Log.d(TAG, "click down")
        return true
    }

    override fun onShowPress(e: MotionEvent?) {
        Log.d(TAG,"click continue")
    }

    override fun onSingleTapUp(e: MotionEvent?): Boolean {
        Log.d(TAG, "click")
        return true
    }

    override fun onScroll(
        e1: MotionEvent?,
        e2: MotionEvent?,
        distanceX: Float,
        distanceY: Float
    ): Boolean {
        return true
    }

    override fun onLongPress(e: MotionEvent?) {
        Log.d(TAG, "press long")
    }

    override fun onFling(
        e1: MotionEvent?,
        e2: MotionEvent?,
        velocityX: Float,
        velocityY: Float
    ): Boolean {
        Log.d(TAG, "click_fling")
        return true
    }
}

// 单独写一个文件
class DoubleTapListener: GestureDetector.OnDoubleTapListener {		// 实现双击接口——需要实现3个方法
    private val TAG ="MY_ACTIVITY_2"
    override fun onSingleTapConfirmed(e: MotionEvent?): Boolean {		// onSingleTapConfirmed和onDoubleTap之间互斥
        Log.d(TAG, "single click")
        return true
    }

    override fun onDoubleTap(e: MotionEvent?): Boolean {
        Log.d(TAG, "double click")
        return true
    }

    override fun onDoubleTapEvent(e: MotionEvent?): Boolean {
        return false
    }
}

class MainActivity : AppCompatActivity(), View.OnTouchListener {			// 实现onTouchListener接口
    private lateinit var mDetector: GestureDetector
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        mDetector = GestureDetector(this as Context, GestureListener())		// 创建mDetector对象
        mDetector.setOnDoubleTapListener(DoubleTapListener())			// 设置双击监听
        val editText: EditText = findViewById(R.id.edit_text)
        editText.setOnTouchListener(this)
    }
    override fun onTouch(v: View?, event: MotionEvent?): Boolean {
        return mDetector.onTouchEvent(event)					// 在重写的方法里面需要返回onTouchEvent方法
    }
}
```

使用GestureDetector的步骤：

1. 实现GestureDetector的onGestureListener、onDoubleTapListener的接口选择性实现
2. 创建GestureDetector的对象

### 3.3 ScroIIer

弹性滑动对象，用于实现View的弹性滑动,使用Scroller来实现**有过渡效果的滑动**

`scrollTo/scrollBy`方法来进行滑动时，其过程是瞬间完成的，这个没有过渡效果的滑动用户体验不好

## view的滑动

滑动方式有3种：

1. 通过View本身提供的scrollTo/scrollBy方法来实现滑动

   主要是通过mScrollX和mScrollY来实现相对位置的改变

   注意：scrollTo和scrollBy**只能改变View内容的位置**而不能改变View在布局中的位置。

   mScrollX和mScrollY的单位为像素：viewX - contentX = mScrollX, viewY - contentY = mScrollY

   初始未移动时，mScrollX和mScrollY=0，当移动时，从左向右移动，那么mScrollX变成负数；从上到下移动，那么mScrollY变成负数

2. 通过**动画**给View施加平移效果来实现滑动

   主要是操作View的translationX和translationY属性。

   可以用原始的view动画，也可以用属性动画。

   1. 原始view动画制作

      <img src="/Users/bytedance/Library/Application Support/typora-user-images/image-20210622161909829.png" alt="image-20210622161909829" style="zoom: 50%;" />

      需要注意：是对View的影像做操作，它并不能真正改变View的位置参数。如果希望动画后的状态得以保留还必须将**fillAfter属性设置为true**，否则**动画完成后其动画结果会消失**

   2. 属性动画

      <img src="/Users/bytedance/Library/Application Support/typora-user-images/image-20210622161948566.png" alt="image-20210622161948566" style="zoom:67%;" />

      为了能够兼容3.0以下的版本，需要采用开源动画库nineoldandroids（但是，对于Android3版本一下的，该库的本质上还是用原始view动画实现的）

      存在的问题：view动画不能改变view本身的位置。那么如果移动之后需要进行交互，那么需要点击移动的原本位置才能进行响应——致命逻辑问题。

      ——**从Android 3.0开始，使用属性动画可以解决上面的问题**，但是对于小于3的设备，是无法解决的，技巧：在新位置预先创建一个和目标Button一模一样的Button，它们不但外观一样连onClick事件也一样。当目标Button完成平移动画后，就把目标Button隐藏，同时把**预先创建的Button显示出来**

3. 通过改变View的LayoutParams使得View**重新布局**从而实现滑动

   eg：比如我们想把一个Button向右平移100px，我们只需要将这个Button的LayoutParams里的**marginLeft参数的值增加100px**即可

   <img src="/Users/bytedance/Library/Application Support/typora-user-images/image-20210622163023684.png" alt="image-20210622163023684" style="zoom:67%;" />

总结3种方式的优劣：

1. scrollTo/scrollBy：操作简单，适合对View内容的滑动；存在的问题是：只能移动view内容，而无法移动view本身

2. 动画：操作简单，主要适用于没有交互的View和实现复杂的动画效果；

   Android3以上效果很好，且可以实现复杂的动画效果。

   Android3以下由于滑动不改变view本身的位置，所以存在严重的bug——所以适合无交互的动画场景

3. 改变布局参数：操作稍微复杂，适用于**有交互的View**——真正改变了view的内容

### 3. 弹性滑动

前面的是生硬的滑动，现在使用的是渐近式滑动，即弹性滑动。

本质思想是：将一次大的滑动分成若干次小的滑动并在一个时间段内完成

方法：

通过Scroller、Handler#postDelayed以及Thread#sleep等

#### 1. scroller

目前只了解基本的工作机制

最上层方法：

1. smoothScrollTo，弹性滑动方法。里面去调用startScroll方法——只是设定了初始值等；实现滑动的是**invalidate()**方法。
2. invalidate()方法会导致view的重绘，它会去调用**computeScroll()**方法
3. computeScroll()该方法需要**自行实现**，本质上就是调用了上面的view动画，eg：scrollTo()，将当前的x/y，移动到新的scrollx/scrolly上，然后又开始重新绘制**postinvalidate()**
4. 而scrollx/scrolly是调用**computeScrollOffset()**方法，主要是根据上一次调用该方法到现在的时间的流逝来计算出当前的scrollX和scrollY的值，并且它的返回值返回true表示滑动还未结束，false则表示滑动已经结束

工作机制：Scroller本身并不能实现View的滑动，它需要配合View的**computeScroll**方法才能完成弹性滑动的效果，它**s**，而每一次**重绘距滑动起始时间会有一个时间间隔**，通过这个时间间隔Scroller就可以得出View当前的滑动位置，知道了滑动位置就可以通过scrollTo方法来完成View的滑动。就这样，View的每一次重绘都会导致View进行小幅度的滑动，而多次的小幅度滑动就组成了弹性滑动

#### 2. 动画

利用动画实现scrollto的弹性移动效果。

<img src="/Users/bytedance/Library/Application Support/typora-user-images/image-20210622170745464.png" alt="image-20210622170745464" style="zoom:67%;" />

#### 3. 延时策略

可以使用：Handler或View的postDelayed方法/线程的sleep方法

## view的事件分发机制

概念：事件是指点击这个动作，即MotionEvent。

关注点：当一个MotionEvent产生了以后，系统需要把这个事件**传递给一个具体的View**——传递过程就是view的分发

有3个核心方法：

1. dispatchTouchEvent：返回值boolean

   用来进行事件的分发。

   事件能够传递给当前View，那么此方法**一定会被调用**

   返回结果受当前View的**onTouchEvent**和下级View的**dispatchTouchEvent**方法的影响，表示**是否消耗当前事件**。

2. onInterceptTouchEvent：返回值boolean

   用来判断是否拦截某个事件（表示我这个view会处理该事件了，不需要再给下层的view了），如果当前View拦截了某个事件，那么在**同一个事件序列当中，此方法不会被再次调用**，返回结果表示是否拦截当前事件。

3. onTouchEvent：返回值boolean

   处理点击事件，返回结果表示是否消耗当前事件，如果不消耗，则在同一个事件序列中，当前View无法再次接收到事件

   是在表示要拦截之后，那么才会去执行该方法

3个方法的关系：对于一个根ViewGroup来说，点击事件产生后，首先会传递给它，这时它的dispatchTouchEvent就会被调用，如果这个ViewGroup的onInterceptTouchEvent方法返回true就表示它要拦截当前事件，接着事件就会交给这个ViewGroup处理，即它的onTouchEvent方法就会被调用；如果这个ViewGroup的onInterceptTouchEvent方法返回false就表示它不拦截当前事件，这时当前事件就会继续传递给它的子元素，接着子元素的dispatchTouchEvent方法就会被调用，如此反复直到事件被最终处理。



# view的工作原理









## ps：

### 1. onTouchEvent()和onTouch()

都是用来处理点击事件——手指碰到屏幕之后会产生的事件

处理的常见事件是：单击、双击、触摸、滑动等

两个方法传递的**参数**：

1. onTouchEvent(MotionEvent event)

   参数event为手机屏幕触摸事件封装类的对象，其中封装了该事件的所有信息，例如触摸的位置、触摸的类型以及触摸的时间等。

   该对象会在用户触摸手机屏幕时被创建。

   因为该方法是写在view内部的（自定义view的时候可以选择是否重写），所以不需要指定view

2. onTouch(View v, MotionEvent event) 

   view是对应的点击的view对象

两个方法的**使用时机**

1. onTouchEvent(MotionEvent event)：**View中定义的方法**，而且是Public类型，所以Activity、ViewGroup、View均可以调用这个方法，最常见的是在**Android事件分发**中，**onTouchEvent()是每个事件处理对象都有的方法**，用于根据下层的onTouchEvent()返回值类型判断是否调用，最常见的用法是自定义View时写入到view中，从而让该View获取用户对手机屏幕的各种操作，并对不同类型的操作实现不同的反馈，属于一个宏观的屏幕触摸监控方法；
2. onTouch((View v, MotionEvent event)：**View.OnTouchListener**接口中实现的唯一方法，接收两个参数，第二个参数是之前提过的event事件对象，第一个参数是一个具体的view类型对象，这就意味着o**nTouch()方法必须和某个控件进行绑定**，即某个控件实现了View.OnTouchListener接口，才能调用onTouch()方法。

两个方法的**返回值**

均是布尔类型。

1. true：**已经完整地处理了该事件且不希望其他回调方法再次处理时返回true**

   那么其他的viewgroup就不会进行处理

2. false：如果view处理不了事件，需要往上级传时，返回值为false，同理ViewGroup和Activity之间事件的控制也是如此。

两个方法的优先级：

1. 如果onTouch()方法返回值是true（事件被消费）时，则onTouchEvent()方法将不会被执行；
2. 当onTouch()方法返回值是false（事件未被消费，向下传递）时，onTouchEvent方法才被执行

——ontouch的优先级更高：view的ontouch返回false，才会触发ontouchevent。

所以：**click事件的实现等等都基于onTouchEvent，假如onTouch返回true，这些事件将不会被触发。**OnClickListener，其优先级最低，即处于事件传递的尾端。

