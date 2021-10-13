

# Service

概念：Service是Android中实现**程序后台运行**的解决方案。Service的运行不依赖于任何用户界面

但是：Service并不是运行在一个独立的进程当中的，而是依赖于创建Service时所在的应用程序进程。该进程被杀死了，那么service也会停止运行。

所以必要的：我们需要显式的在service内部手动创建子线程，去执行任务，不能在主线程中去执行，容易发生阻塞。

## 1. Android中的线程

同Java中一样，有两种方法：

```kotlin
class MyThread : Thread(){
  override fun run(){
    ....
  }
}

// 使用：
MyThread().start()
```

```kotlin
class MyThread : Runnable{
	override fun fun(){
    ....
  }
}

// 使用
val myThread = MyThread()
Thread(myThread).start()
```

可以简化写法：

```kotlin
Thread{
	...
}.start()
```

但是需要注意的是：**子线程中不能进行UI操作**

## 2. 在子线程中更新ui

基本规定：在子线程中不能**直接**进行UI更新

但是存在这样的需求：在子线程里执行一些耗时任务，然后根据任务的执行结果来更新相应的UI控件。——可以在子线程中执行完成后，发信息给主线程，然后主线程去更新对应的ui

```kotlin
class MainActivity : AppCompatActivity() {
    val updateText = 1
  	// 创建handler对象，并且重写父类的handlemessage方法——实现具体的修改UI逻辑
    val handler = object : Handler(Looper.getMainLooper()){
        override fun handleMessage(msg: Message) {
            when(msg.what){
                updateText -> {
                    val textView : TextView = findViewById(R.id.text)
                    textView.text = (0..50).random().toString()
                }
            }
        }
    }
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        val button : Button = findViewById(R.id.change)
        button.setOnClickListener {			// 监听器
            thread {
                val msg = Message()				// 创建message对象
                msg.what = updateText			// what是int类型的数据
                handler.sendMessage(msg)		// 通过handler将数据传递出去
            }
        }
    }
}
```

## 3. 异步处理消息的机制

Message、Handler、MessageQueue、Looper：

1. message：消息，能够在线程之间传递消息，能够带少量的信息——消息本身

   what：int类型的、arg1、arg2：整型、obj：object对象

2. handler：用来发送和处理消息的——发 and 处理

   发送：handler.sendMessage、post

   接收：一般是重写父类的handleMessage方法，写入需要处理的逻辑

3. messageQueue——存储收到的数据

   消息队列，它主要用于存放所有通过Handler发送的消息

   一个线程只有一个消息队列

4. looper——来处理消息队列的

   MessageQueue的管家，调用Looper的loop()方法后，就会进入一个**无限循环**当中，然后每当发现MessageQueue中存在一条消息时，就会将它取出，并传递到Handler的handleMessage()方法中。每个线程中只会有一个Looper对象。

所以异步消息处理：

1. 在主线程创建handler对象，然后重写handleMessage方法

2. 创建message对象，将需要传递的数据带上，eg：msg.what

3. 使用handle的sendMessage方法，将message方法发送出去

4. 被添加到MessageQueue的队列中等待被处理

5. Looper则会一直尝试从MessageQueue中取出待处理消息

   Looper.getMainLooper()——所以是在主线程中运行的

## 4. AsyncTask

算是封装好的异步处理消息机制，实现的效果同上。

需要实现`AsyncTask`接口，

当然需要传递的参数：

1. Params。在执行AsyncTask时需要传入的参数

   eg： Unit 表示不需要传入参数

2. Progress。在后台任务执行时，如果需要在界面上显示当前的进度

   eg：int，表示进度用int（百分比来表示）

3. Result。当任务执行完毕后，如果需要对结果进行返回

   eg：boolean，表示返回true/false表示任务是否正确执行

需要重写的方法：onPreExecute / doInBackground / onPostExecute / onProgressUpdate（是在doInBackground中调用了publishProgress的方法，而该方法会去调用onProgressUpdate）

使用：DownloadTask().execute()

## 5. Service

### 5.1 定义

com.example.servicetest→New→Service→Service。

需要选择：

1. Exported属性表示是否将这个Service**暴露给外部其他程序访问**
2. Enabled属性表示**是否启用**这个Service

创建好之后，as会自动将service进行注册（**四大组件都需要注册**）

需要注意：MyService是继承自系统的Service类的，而该类中有一个方法必须要实现：**onBind()方法**。

具体的逻辑可以重写里面的方法：

1. onCreate()：在Service创建的时候调用

   所以在该service第一次调用的时候，会去调用oncreate方法创建，后面就不会再调用了

   如果之前调用了ondestroy，那么再次调用的时候还是会执行onCreate方法的

2. onStartCommand()：每次Service启动的时候调用

3. onDestroy()：在Service销毁的时候调用——一般用来回收资源

```kotlin
override fun onCreate() {
        super.onCreate()
        Log.d("myService", "oncreate")
    }

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        Log.d("myService", "onstart")
        return super.onStartCommand(intent, flags, startId)
    }

    override fun onDestroy() {
        super.onDestroy()
        Log.d("myService", "ondestory")
    }
```

### 5.3 启动service

**借助Intent来实现的**

```kotlin
val start : Button = findViewById(R.id.start)
val end : Button = findViewById(R.id.end)
start.setOnClickListener {
  val intent = Intent(this, MyService::class.java)
  startService(intent)			// 启动一个service
}
end.setOnClickListener {
  val intent = Intent(this, MyService::class.java)
  stopService(intent)			// 停止一个service
}
```

Service也可以自我停止运行，只需要在Service内部调用**stopSelf()**方法即可。

Android8之后：只有当应用保持在前台可见状态的情况下，Service才能保证稳定运行，一旦应用进入后台之后，Service随时都有可能被系统回收。——防止手机变卡

如果需要长期在后台执行任务——可以使用前台Service或者WorkManager

### 5.4 activity和service之间的通信

让Activity和Service的关系更紧密一些呢？例如在Activity中指挥Service去干什么，Service就去干什么——**onBind()方法**

1. 需要在service类中实现一个binder类，继承自Binder类

   然后提供内部的逻辑：

   ```kotlin
   class DownloadBinder : Binder(){
     fun startDownload(){			// 内部方法1，开始下载方法
       Log.d("myService", "start downloading")
     }
     fun getProgress() : Int{			// 内部方法2，获取下载进度方法
       Log.d("myService", "downloading")
       return 0
     }
   }
   ```

2. 实现onBind方法

   ```kotlin
   private val myBinder = DownloadBinder()
   override fun onBind(intent: Intent): IBinder {
     return myBinder
   }
   ```

3. 在主activity中创建了一个ServiceConnection的匿名类实现，并在里面重写了onServiceConnected()方法和onServiceDisconnected()方法

   ```kotlin
   private lateinit var myBinder : MyService.DownloadBinder
   // 创建匿名类
   private val connection = object : ServiceConnection{
     
     override fun onServiceConnected(name: ComponentName?, service: IBinder?) {
       // 向下转型得到了DownloadBinder的实例
       myBinder = service as MyService.DownloadBinder
       myBinder.startDownload()		// 那么能够调用任何自定的bingder方法了
       myBinder.getProgress()
     }
   
     override fun onServiceDisconnected(name: ComponentName?) {
     }
   }
   ```

   1. onServiceConnected()方法方法会在Activity与Service成功绑定的时候调用，
   2. onServiceDisconnected()方法在Service的创建进程崩溃或者被杀掉的时候才会调用

4. Activity和Service进行绑定

   调用**bindService()**方法：3个参数

   1. 刚刚构建出的Intent对象

   2. ServiceConnection的实例

   3. 标志位：**BIND_AUTO_CREATE**，绑定后自动创建Service

      所以onCreate会执行，但是onStartCommand()方法不会执行

   解除绑定：**unbindService()**——会去调用上面的onDestory方法

   ```kotlin
   bindBtn.setOnClickListener {
     val intent = Intent(this, MyService::class.java)
     bindService(intent, connection, BIND_AUTO_CREATE)
   }
   unbindBtn.setOnClickListener {
     unbindService(connection)
   }
   ```

注意：

任何一个Service在**整个应用程序范围内都是通用的**，即MyService不仅可以和MainActivity绑定，还可以和**任何一个其他的Activity进行绑定**，而且在绑定完成后，它们都可以**获取相同的DownloadBinder实例**

## 6. service的生命周期

1. startService

   相应的Service就会启动，并回调onStartCommand()方法

   而如果此时service没有创建过，那么onCreate()方法会先执行

   Service启动了之后会**一直保持运行状态**，直到stopService()或stopSelf()方法被调用，或者被系统回收

   调用一次startService()方法，onStartCommand()就会执行一次，但实际上每个Service只会存在一个实例

2. stopService

   onDestroy()方法就会执行，表示Service已经销毁了

3. bindService

   回调Service中的onBind()方法

   这个Service之前还没有创建过，onCreate()方法会先于onBind()方法执行

4. unbindService

   onDestroy()方法也会执行

注意：如果一个service既调用了startService()方法，又调用了bindService()方法

——**要同时调用stopService()和unbindService()方法，onDestroy()方法才会执行。**

## 7. service的高级技巧

### 7.1 前台service

希望Service能够一直保持运行状态，就可以考虑使用前台Service——不会被系统回收。

展示：前台service一直会有一个正在运行的图标在系统的状态栏显示——系统会认为该service有前台可见状态，那么不会将其回收。

修改myService的onstartcommand方法：

```kotlin
override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
  Log.d("myService", "onstart")
  
  /*************************/
  val intent = Intent(this, MainActivity::class.java)
  val pi = PendingIntent.getActivity(application, 0, intent, 0)
  // 创建提示框
  val notification = NotificationCompat.Builder(this, "my_service")
  .setContentTitle("This is content title")
  .setContentText("This is content text")
  .setSmallIcon(R.drawable.ic_launcher_background)
  .setLargeIcon(BitmapFactory.decodeResource(resources, R.drawable.ic_launcher_foreground))
  .setContentIntent(pi)			// 对该提示框作出的响应
  .build()
  startForeground(1, notification)	// 在activity中写会报错——是service中的方法
  /****************************/
  
  return super.onStartCommand(intent, flags, startId)
}
```

修改mainActivity的oncreate方法：

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
  super.onCreate(savedInstanceState)
  setContentView(R.layout.activity_main)
  /*******************************/
  val manager = getSystemService(Context.NOTIFICATION_SERVICE) as
  NotificationManager
  if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {			//8.0以上的版本需要创建通道
    val channel = NotificationChannel("my_service", "前台Service通知",
                                      NotificationManager.IMPORTANCE_DEFAULT)
    manager.createNotificationChannel(channel)
   /***************************/
    ....
  }
```

### 7.2 IntentService

直接在Service里处理一些耗时的逻辑，就很容易出现**ANR（Application Not Responding）**的情况
——使用多线程技术。

所以在service中的每个具体的方法里**开启一个子线程**

eg：在方法里面开启子线程，并且里面的逻辑执行完成后会自己停止自己。

```kotlin
override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
  thread{
    // 具体逻辑
    stopSelf()
  }
  return super.onStartCommand(intent, flags, startId)
}
```

——Android提供的**IntentService类**，异步的能停止的service

（目前已经被废弃了）

```kotlin
// 必须先调用父类的构造函数，并传入一个字符串，这个字符串可以随意指定，只在调试的时候有用
class MyIntentService : IntentService("myIntentService") {
  override fun onHandleIntent(intent: Intent?) {		// 是在子线程中运行的
    Log.d("myIntentService", "current thread is ${Thread.currentThread()}.name")
  }

  override fun onDestroy() {
    super.onDestroy()
    Log.d("myIntentService", "be destroyed")
  }
}
```

在mainActivity中，启动service的方法一样：创建intent，然后调用startService去启动service

能够发现，点击startIntent按钮之后，就能启动该service，并且执行完onhandleintent之后，**会自动调用ondestroy方法停止**——本质上就是去调用了stopself去停止自己。