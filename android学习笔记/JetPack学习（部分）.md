# JetPack学习

Google推出了一个全新的开发组件工具集**Jetpack**

## 1. 介绍

jetpack是一个**开发组件工具集**。主要是简化代码，方便开发流程。

Jetpack中的组件有一个特点，它们大部分不依赖于任何Android系统版本，这意味着这些组件通常是**定义在AndroidX库**当中的，并且拥有非常好的**向下兼容性**。

jetpack包含了很多内容：通知、权限、Fragment都属于Jetpack

## 2. ViewModel

### 2.1 简单使用

ViewModel的一个重要作用就是可以**帮助Activity分担一部分工作**

作用：专门用于存放与界面相关的数据的

特性：在页面进行旋转的时候，activity就会重新加载，而viewmodel不会，只有当activity被销毁的时候，viewmodel才会跟随销毁。

——将与界面相关的变量存放在ViewModel当中，这样即使旋转手机屏幕，界面上显示的数据也不会丢失

要使用ViewModel组件，还需要在app/build.gradle文件中添加如下依赖：

```
implementation "androidx.lifecycle:lifecycle-extensions:2.2.0"
implementation "androidx.lifecycle:lifecycle-livedata-ktx:2.2.0"
```

比较好的编程规范是给每一个Activity和Fragment都创建一个对应的ViewModel

绝对不可以直接去创建ViewModel的实例，而是一定要通过ViewModelProvider来获取ViewModel的实例：

（如果直接获取实例，那么生命周期和activity一致了，那么无法保存数据）

格式：

```
ViewModelProvider(activity/fragment实例).get(<创建的viewmodel文件>::class.java)
```

eg：

```kotlin
// 对mainactivity创建一个viewmodel
class MainViewModel :ViewModel() {
    var counter = 0
}

// 在mainactivity中实现逻辑
class MainActivity : AppCompatActivity() {
    lateinit var viewModel: MainViewModel
    lateinit var binding: ActivityMainBinding		// 绑定ui和逻辑
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        viewModel = ViewModelProvider(this).get(MainViewModel::class.java)

        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        binding.button.setOnClickListener{
            viewModel.counter++
            refreshCounter()
        }
        refreshCounter()
    }
    private fun refreshCounter(){
        binding.info.text = viewModel.counter.toString()
    }
}
```

### 2.2 向ViewModel传递参数

背景：需要保存数据，即使程序退出该值还是存在的。

——需要在值更新的时候保存，并且**在程序退出的时候将值保存在某个地方，重新打开的时候能加载到该值**

向ViewModel传递参数用：**ViewModelProvider.Factory**

所以需要在构造函数中传递值：

```kotlin
class MainViewModel(counterReserved: Int) :ViewModel() {		//  主构造函数需要能接收值
    var counter = counterReserved				
}
```

修改逻辑：

```kotlin
class MainActivity : AppCompatActivity() {
    lateinit var viewModel: MainViewModel
    lateinit var binding: ActivityMainBinding
    // add
    lateinit var sp : SharedPreferences				// 轻量级的存储类，能够用来存储数据，能保存简单类型的数据，例如，String、int等
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        // add，操作模式，默认为私有的，仅本应用可用
        sp = getPreferences(Context.MODE_PRIVATE)
        // 尝试获取之前保存的sp里面的值，如果没有获取默认为0
        val countReserved = sp.getInt("count_reserved", 0)

        // 这边将保存的值传递过去，只有使用factory，计数值最终传递给MainViewModel的构造函数
        viewModel = ViewModelProvider(this, MainViewModelFactory(countReserved)).get(MainViewModel::class.java)

        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        binding.button.setOnClickListener{
            viewModel.counter++
            refreshCounter()
        }
        refreshCounter()
        binding.clear.setOnClickListener{
            viewModel.counter = 0
            refreshCounter()
        }
        refreshCounter()
    }
    private fun refreshCounter(){
        binding.info.text = viewModel.counter.toString()
    }

    override fun onPause() {		// 进入到暂停状态时，非活动状态那么就将内容保存，下一次载入可以获得
        super.onPause()
        sp.edit { putInt("count_reserved", viewModel.counter) }
    }
}
```

## 3. LifeCycles

概念：感知activity的生命周期，且是在非activity类中去感知activity的生命周期

lifecycles它可以让任何一个类都能轻松感知到Activity的生命周期，同时又不需要在Activity中编写大量的逻辑处理。

eg：在一个类中我需要能够感知到mainactivity的状态变化，作出不同的响应

1. 新建一个MyObserver类，并让它实现LifecycleObserver接口

   LifecycleObserver接口是一个空接口，没有待实现的方法

   带上注解功能就能实现感知

   **@OnLifecycleEvent(Lifecycle.Event.xxx)**

   ```kotlin
   class MyObserver :LifecycleObserver {
       @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)		// 在onStart()中触发执行
       fun activityStart(){
           Log.d("myObserver", "activity start")
       }
       @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
       fun activityStop(){
           Log.d("myObserver", "activity stop")
       }
   }
   ```

   生命周期事件的类型一共有7种：ON_CREATE、ON_START、ON_RESUME、ON_PAUSE、ON_STOP和ON_DESTROY分别匹配。——对应activity的7个生命状态

   ON_ANY类型，表示可以匹配Activity的任何生命周期回调

2. 需要通知MyObserver发生改变

   在mainactivity中：添加

   ```
   lifecycle.addObserver(MyObserver())
   ```

   用Lifecycle对象，然后调用它的addObserver()方法来观察LifecycleOwner的生命周期
   只要你的Activity是继承自AppCompatActivity的，或者你的Fragment是继承自androidx.fragment.app.Fragment的，那么它们本身就是一个LifecycleOwner的实例

   ——这样MyObserver就能感知到变化

3. 但是，如果在MyObserver需要获得当前的生命周期状态，可以将该lifecycle对象传入到MyObserver中

   就可以在任何地方调用lifecycle.currentState来主动获知当前的生命周期状态。lifecycle.currentState返回的生命周期状态是一个枚举类型，一共

   - INITIALIZED：尚未调用on_create
   - DESTROYED:
   - CREATED：onCreate()方法已经执行了，但是onStart()方法还没有执行
   - STARTED：onStart()方法已经执行了，但是onResume()方法还没有执行
   - RESUMED这5种状态类型

## 4. LiveData

概念：**响应式编程组件**，它可以**包含任何类型的数据**，并在**数据发生变化的时候通知给观察者**

### 4.1 基本使用

LiveData特别适合与ViewModel结合在一起使用

背景：前面的viewmodel中的数据，都是activity主动去获取viewmodel中的数据，而没有办法**实现viewmodel中的数据发生变化主动推给activity**（但是不能把Activity的实例传给ViewModel，那么viewmodel就能主动将数据变化传递给activity，但是就很有可能会因为**Activity无法释放而造成内存泄漏**）
——livedata可以实现

`MutableLiveData<Int>`可变的livedata，包含了整型数据。

3种读写数据的方法，分别是**getValue()、setValue()（只能在主线程中调用）和postValue()（非主线程中进行调用）**方法

```kotlin
class MainViewModel(counterReserved: Int) :ViewModel() {
    val counter = MutableLiveData<Int>()			// 可变的livedata
    init {
        counter.value = counterReserved				// 初始化值
    }

    fun add(){
        val count = counter.value ?: 0			// 如果是null的，那么初值为0，否则就是value值
        counter.value = count + 1
    }
    fun clear(){
        counter.value = 0
    }
}
```

在mainactivity中的实现：

livedata有一个**observe**方法，在哪个地方调用该方法，就可以观察该数据的变化，有两个参数：

1. 是一个LifecycleOwner对象，直接传递this即可
2. Observer接口，里面有一个方法`onChanged`，里面实现变化时的逻辑即可，传递的参数就是该对象的类型

```kotlin
class MainActivity : AppCompatActivity() {
    ...
    override fun onCreate(savedInstanceState: Bundle?) {
       ...
        binding.button.setOnClickListener{
            viewModel.add()
        }
        binding.clear.setOnClickListener{
            viewModel.clear()
        }
        viewModel.counter.observe(this, Observer { count -> binding.info.text = count.toString() })
    }
    override fun onPause() {
        super.onPause()
        sp.edit { putInt("count_reserved", viewModel.counter.value ?: 0) }
    }
}
```

但是这样还是不是很规范：因为直接将counter暴露出来了，即使是在ViewModel的外面也是可以给counter设置数据的，从而破坏了ViewModel数据的封装性，同时也可能带来一定的风险。——永远只暴露不可变的LiveData给外部

```kotlin
class MainViewModel(counterReserved: Int) :ViewModel() {
  // counter是可以对外公开的，且不可修改的，而_counter是私有的，只能通过内部方法修改
    val counter : LiveData<Int> get() = _counter
    private val _counter = MutableLiveData<Int>()
    init {
        _counter.value = counterReserved
    }

    fun add(){
        val count = _counter.value ?: 0
        _counter.value = count + 1
    }
    fun clear(){
        _counter.value = 0
    }
}
```

### 4.2 map和switchMap

LiveData为了能够应对各种不同的需求场景，提供了两种转换方法：**map()**和**switchMap()**方法。

背景：如果值需要关注对象中的某个参数变化，而不关注整个对象。

#### 4.2.1 map

调用了**Transformations.map()**方法来对LiveData的数据类型进行转换：

map()方法接收两个参数：

- 第一个参数是原始的LiveData对象；
- 第二个参数是一个转换函数

```kotlin
class MainViewModel(counterReserved: Int) :ViewModel() {
  //  私有的变量，可变的类型
    private val userData = MutableLiveData<User>()
  // 公开的，不可变的类型，直接通过可变的userData来进行转换
    val userName : LiveData<String> = Transformations.map(userData){
        user -> "${user.firstName} ${user.lastName}"
    }
}
```

#### 4.2.2 switchMap

背景：ViewModel中的某个LiveData对象是调用另外的方法获取的，存在某个livedata每次调用时都会返回一个新的对象，从而无法观察数据的变化。

功能：如果ViewModel中的**某个LiveData对象是调用另外的方法获取的，那么我们就可以借助switchMap()方法**，将这个LiveData对象转换成另外一个可观察的LiveData对象。

1. 定义了一个User数据类

   ```kotlin
   data class User(var firstName : String, var lastName : String, var age : Int)
   ```

2. 定义了一个单例类

   只有一个方法：根据传递来的userId来创建一个新的对象，该对象是livedata类型的，每次调用该方法都会新建一个livedata对象

   ```kotlin
   object Repository {
       fun getUser(userId : String) : LiveData<User>{          // 每次调用getUser()方法都会返回一个新的LiveData实例
           val liveData = MutableLiveData<User>()
           liveData.value = User(userId, userId, 0)
           return liveData
       }
   }
   ```

3. 修改mainviewmodel类

   userName是livedata类型的——对外activity来获得的

   userIdData是不可变的

   ```kotlin
   class MainViewModel(counterReserved: Int) : ViewModel() {
     // 对内数据
       private val userIdData = MutableLiveData<String>()
     
     // 对外数据，根据userIdData的变化会触发，去获取真正的用户数据——调用Repository.getUser
     // 并且switchMap会将返回的LiveData对象转换成一个可观察的LiveData对象
       val userName : LiveData<User> = Transformations.switchMap(userIdData){
           userId -> Repository.getUser(userId)
       }
     // 外部调用该方法，传递来的值使userIdData发生变化，就会触发上面的方法
       fun getUser(userId : String){
           userIdData.value = userId
       }
   }
   ```

4. 