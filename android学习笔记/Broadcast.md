# 广播

概念：

1. 标准广播

   完全异步执行的广播。

   在广播发出之后，所有的广播接收器几乎都会在**同一时刻接收到这条广播消息**，因此它们之间没有任何先后顺序可言。这种广播的**效率会比较高，但同时也意味着它是无法被截断的**

   ——符合普通意义上的广播

2. 有序广播

   同步执行的广播，在广播发出之后，**同一时刻只会有一个广播接收器能够收到这条广播消息**，当这个广播接收器中的逻辑执行完毕后，**广播才会继续传递**。所以此时的广播接收器是有先后顺序的，优先级高的广播接收器就可以先收到广播消息，并且前面的广播接收器还可以截断正在传递的广播，这样后面的广播接收器就无法收到广播消息了。

## 1. 系统广播

### 1.1 动态注册

创建BroadcastReceiver：

1. 继承自BroadcastReceiver

2. 重写父类的onReceive()方法

   ——当有广播到来时，onReceive()方法就会得到执行

3. 创建IntentFilter实例对象，就是用来过滤

4. 添加监听内容：filter.addAction(xxx)——里面就是写监听的广播

5. 创建内部类实例对象

6. 注册接收器：registerReceiver

   需要传递两个参数：recevier、intentfilter，即步骤3、5创建的对象

7. 在onDestroy中取消注册

   调用unregisterReceiver()方法

```kotlin
class MainActivity : AppCompatActivity() {
    lateinit var timeChangeReceiver: TimeChangeReceiver
    inner class TimeChangeReceiver : BroadcastReceiver(){
        override fun onReceive(context: Context?, intent: Intent?) {
            Toast.makeText(context, "time changed", Toast.LENGTH_LONG).show()
        }
    }
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        val filter = IntentFilter()
        filter.addAction("android.intent.action.TIME_TICK")
        timeChangeReceiver = TimeChangeReceiver()
        registerReceiver(timeChangeReceiver, filter)
    }

    override fun onDestroy() {
        super.onDestroy()
        unregisterReceiver(timeChangeReceiver)
    }
}
```

### 1.2 静态注册

在Android 8.0系统之后，静态注册的BroadcastReceiver是无法接收隐式广播的，只有部分的隐式广播还能接收

创建一个单独的类，可以用快捷方式创建——类似于activity的创建方式。

然后就会默认有一个继承了BroadcastReceiver的类：

```kotlin
class MyReceiver : BroadcastReceiver() {
  override fun onReceive(context: Context, intent: Intent) {
    // This method is called when the BroadcastReceiver is receiving an Intent broadcast.
    Toast.makeText(context, "reboot complete", Toast.LENGTH_LONG).show()
  }
}
```

静态的BroadcastReceiver一定要在AndroidManifest.xml文件中注册才可以使用

```xml
<receiver
          android:name=".MyReceiver"
          android:enabled="true"
          android:exported="true">
  <intent-filter>
    <action android:name="android.intent.action.BOOT_COMPLETED" />
  </intent-filter>
</receiver>
```

并且添加权限：

```
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
```

需要注意的是，不要在onReceive()方法中添加过多的逻辑或者进行任何的耗时操作，因为**BroadcastReceiver中是不允许开启线程的**，当onReceive()方法运行了较长时间而没有结束时，程序就会出现错误。

## 2. 发送广播

### 2.1标准广播

1. 创建intent对象，需要将发送广播的值传入——就是其他应用在接收的对象

2. **setPackage**——指定发送给哪个应用程序的，从而让它变成一条显式广播

   Android8之后变严格了：静态注册的BroadcastReceiver是无法接收隐式广播的，而默认的自定义广播都是隐式广播。所以需要显式指定发送的对象

3. sendBroadcast(intent)，发送广播出去

```kotlin
val button : Button = findViewById(R.id.button)
button.setOnClickListener {
  
  val intent = Intent("com.example.broadcasttest.MY_BROADCAST")	// 发送的值
  intent.setPackage(packageName)		// 显式指定发送给自己
  sendBroadcast(intent)			// 发送广播出去
}
```

### 2.2 有序广播

就是前面描述的，存在一定的顺序，只有一个节点处理完成后了下一个才能继续处理。

并且存在一定的优先级，优先级高的能够先收到，并且它可以提前终止广播的发送。

只需要将原来的sendBroadcast变成sendOrderedBroadcast即可，需要传递2个参数：

1. intent
2. 与权限相关的字符串，一般传入null

```
sendOrderedBroadcast(intent, null)
```

对于接收该广播的应用，可以设定优先级

```　xml
<receiver
          android:name=".MyBroadcastReceiver"
          android:enabled="true"
          android:exported="true">
  <intent-filter android:priority="10">
    <action android:name="com.example.broadcasttest.MY_BROADCAST"/>
  </intent-filter>
</receiver>
```

对于receiver可以设定是否要终止有序广播：

```kotlin
override fun onReceive(context: Context, intent: Intent) {
  // This method is called when the BroadcastReceiver is receiving an Intent broadcast.
  Toast.makeText(context, "reboot complete", Toast.LENGTH_LONG).show()
  abortBroadcast()			// 终止广播
}
```

