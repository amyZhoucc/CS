# Activity的学习

## 1. 第一次创建activity

创建的工程为no activity，然后里面为空的，且没有配置

- 创建主activity文件：注意不要勾选Generate LayoutFile和Launcher Activity这两个选项——分别为：自动生成布局文件，和默认为主activity

  ——然后就会产生一个默认的代码内容

- 在res下面创建一个layout文件夹，然后在里面右键→New→Layout resource file，默认是LinearLayout作为根元素

  ——如果不是这样创建的，那么是创建了一个`constraintlayout`

  ```xml
  <androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:app="http://schemas.android.com/apk/res-auto"
      xmlns:tools="http://schemas.android.com/tools"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      tools:context=".MainActivity">
  </androidx.constraintlayout.widget.ConstraintLayout>
  ```

那么如何将activity和布局文件**进行关联**呢？

——调用了setContentView()方法来给当前的Activity加载一个布局，传递的参数就是一个id

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
      // 设定该activity的布局文，其实就是一个id
        setContentView(R.layout.my_layout)      
    }
}
```

（项目中添加的任何资源都会在R文件中生成一个相应的资源id）

**注册：所有的Activity都要在AndroidManifest.xml中进行注册才能生效。**

——AS默认会进行注册

```
<activity android:name=".MainActivity"></activity>
```

——使用了android:name来指定具体注册哪一个Activity

——**android:label**指定Activity中标题栏的内容

——给主Activity指定的label不仅会成为标题栏中的内容，还会成为启动器（Launcher）中应用程序显示的名称

但是，由于这个是我们的主activity，所以我们需要在里面设定为主activity：

```xml
<activity android:name=".MainActivity">
  <intent-filter>
    <action android:name="android.intent.action.MAIN"/>		//主activity
    <category android:name="android.intent.category.LAUNCHER" />		//启动activity
  </intent-filter>
</activity>
```

如果应用程序中没有声明任何一个Activity作为主Activity，这个程序仍然是可以正常安装的，只无法打开该程序，一般用为第三方服务给其他应用程序进行调用的。



## 2. Toast的使用

默认版本同Java版

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.my_layout)      // 设定该activity的布局文，其实就是一个id
        var button : Button = findViewById(R.id.button_1)
        button.setOnClickListener{				// 这个实际上是匿名方法，只不过省略了
            Toast.makeText(this, "hello", Toast.LENGTH_SHORT).show()
        }
    }
}
```

### 进阶版：

背景：toast本身也比较麻烦，且容易忘记调用最后的show()方法，导致各种问题，而调试麻烦，所以将其封装成一个方法直接调用，能够解耦合。

新建一个kotlin文件，不需要创建类

```kotlin
// 需要传参context，就是当前toast的上下文
// 其实还有一个参数duration，但是会给默认值，如果可以动态选择传参
fun String.showToast(context: Context, duration : Int = Toast.LENGTH_SHORT){
    Toast.makeText(context, this, duration).show()
}
fun Int.showToast(context: Context, duration: Int = Toast.LENGTH_SHORT){
    Toast.makeText(context, this, duration).show()
}
```

理解：

- 是**给String类和Int类新增了一个showToast()函数**。
- 这边的this，是具体的内容，因为是在对应的数据对象中进行的调用，而上面的this，指代的是context对象

activity中：

```kotlin
button.setOnClickListener{
  "this is hello".showToast(this)			//直接调用上面的方法
}
```

### 进进阶版

在进阶技巧中设置了全局的context的，那么我们可以直接使用这个全局的context，而不需要在传参了：

```kotlin
fun String.showToast(duration : Int = Toast.LENGTH_SHORT){
  // 使用静态变量context，使用方法同java，定义在进阶技巧中
    Toast.makeText(MyApplication.context, this, duration).show()
}
fun Int.showToast(duration: Int = Toast.LENGTH_SHORT){
    Toast.makeText(MyApplication.context, this, duration).show()
}
```

## 3. 使用menu

创建menu的xml文件和原来一样：需要先创建文件夹menu，然后在里面新建menu文件

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android"> 
  <item
        android:id="@+id/add_item"
        android:title="Add"/> 
  <item
        android:id="@+id/remove_item"
        android:title="Remove"/>
</menu>
```

逻辑上面，存在语法的不同：

```kotlin
override fun onCreateOptionsMenu(menu: Menu?): Boolean {
  menuInflater.inflate(R.menu.main, menu)		// 语法糖，调用了方法getMenuInflater方法
  return true		// 表示可见
}

override fun onOptionsItemSelected(item: MenuItem): Boolean {
  when(item.itemId){			// 语法糖，其实也是调用了getItemId的方法
    R.id.add_item -> "you clicked add".showToast()
    R.id.remove_item -> "you clicked remove".showToast()
    else -> " ".showToast()
  }
  return true
}
```

理解：

逻辑上来讲：需要重写

1. onCreateOptionsMenu：将menu导入逻辑中
2. **onOptionsItemSelected**：定义菜单响应事件

从实现上来讲：

1. onCreateOptionsMenu，有语法糖，实际上调用的是方法`getMenuInflater()`，但是可以简化写成menuFlater，编译器会自动进行转换的。该方法返回的是一个menuInflater对象，然后调用该对象的inflate方法
2. onOptionsItemSelected，Java是用switch实现的，而kotlin可以用when进行实现

## 4. 使用Intent

功能：主要是能够启动一个活动、服务、发送广播等

目前学习：启动活动

### 4.1 显式使用

就实现创建一个activity，直接使用默认的实现即可。

然后创建点击事件，需要先创建一个intent对象：

需要传递两个参数：

- 当前上下文，即当前activity
- 意图：目标activity

```kotlin
button.setOnClickListener{
  val intent = Intent(this, SecondActivity::class.java)
  startActivity(intent)
}
```

注意：`SecondActivity::class.java`等同于Java中的`SecondActivity.class`

这样就能启动一个activity了，然后点击back，就会返回，该activity就会被销毁

### 4.2 隐式使用

并不明确指出我们想要启动哪一个活动，而是指定了一系列更为抽象的action和category等信息，交由**系统去分析这个Intent，并帮我们找出合适的活动去启动。**

原则上一个Intent不应该既是显式调用又是隐式调用，**如果二者共存的话以显式调用为主**

如下是过滤信息：

```xml
<activity android:name=".SecondActivity">
  <intent-filter>
    <action android:name="com.example.activity.ACTION_START" />
    <category android:name="android.intent.category.DEFAULT" />
  </intent-filter>
</activity>
```

**只有和两个内容同时能够匹配上Intent中指定的action和category时，这个活动才能响应该Intent。**

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
  super.onCreate(savedInstanceState)
  setContentView(R.layout.my_layout)
  var button : Button = findViewById(R.id.button_1)
  button.setOnClickListener{
    val intent : Intent = Intent("com.example.activity.ACTION_START")
    startActivity(intent)
  }
}
```

1. 直接将刚才配置的action的字符串内容传了进去，表明我们想要启动能够响应`com.example.activitytest.ACTION_START`这个action的活动
2. 而前面设置的category，并没有配置，这是因为android.intent.category.DEFAULT是一种**默认的category**，在调用startActivity()方法的时候会自动将这个category添加到Intent中。

并且，一个Inent，只能设定一个action，而可以有指定多个category，那么1个action和多个category必须全部同时匹配，才能实现跳转。

Intent也能够启动其他程序的活动，那么**多个app之间能够共享功能**。

```kotlin
val intent = Intent(Intent.ACTION_VIEW)
intent.setData(Uri.parse("https://baidu.com"))
startActivity(intent)
```

IntentFilter中的过滤信息有**action、category、data**，可以选择性的实现部分过滤，而一旦增加了，**必须同时匹配这几部分才能算是完全匹配，才能响应该intent**

一个过滤列表中的action、category和data可以有多个，分为3组过滤类别。一个Activity中可以有多个intent-filter，一**个Intent只要能匹配任何一组intent-filter即可成功启动对应的Activity**

1. action的匹配规则：——intent中只能指定一个action

   一个过滤规则中可以有多个action，那么只要Intent中的action能够和过滤规则中的**任何一个action相同**即可匹配成功。

   但是：如果存在action要匹配，那么**intent中的action必须存在**，且**和其中之一匹配**，**且区分大小写**——那么匹配成功

2. category的匹配规则：——intent中可以指定多个category

   intent中的每个category必须和过滤器中的一个进行匹配，才算是完全匹配。

   当然，如果intent没有设定category，系统会为intent默认加上android.intent.category.DEFAULT，而activity的过滤器的category必须要添加该category，那么就会默认匹配

3. data的匹配规则：mimeType（媒体类型）和URI

   Intent中必须含有data数据，并且data数据能够完全匹配过滤规则中的某一个data

### 4.3 利用intent传递数据

#### 4.3.1 向下传递

```kotlin
button.setOnClickListener{
  val string = "second"
  val intent = Intent(this, SecondActivity::class.java)
  intent.putExtra("data", string)		// intent对象中带着数据
  startActivity(intent)
}
```

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
  super.onCreate(savedInstanceState)
  setContentView(R.layout.activity_second)

  val extaData = intent.getStringExtra("data")
  Log.d("secondActivity", "extra data is $extaData")
}
```

这边和Java版本的存在不同，Java中需要先`getIntent()`，用来获得启动当前activity的intent，然后才能调用`getStringExtra()`方法。

而kotlin由于存在语法糖，所以可以直接用intent。所以这一句的实际含义：调用的是父类的getIntent()方法，该方法会获取用于启动 SecondActivity的Intent，然后调用getStringExtra()方法并传入相应的键值，就可以得到传递的数据了。

#### 4.3.2 返回数据

在活动销毁的时候能够返回一个结果给上一个活动

那么需要secondActivity中进行触发，所以选择button来进行触发：

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
  super.onCreate(savedInstanceState)
  setContentView(R.layout.activity_second)

  val extaData = intent.getStringExtra("data")		
  Log.d("secondActivity", "extra data is $extaData")		// 这个是获取first传递过来的数据

  val button : Button = findViewById(R.id.button_2)		// 用来触发返回
  button.setOnClickListener{
    val intent = Intent()				// 需要创建一个intent对象才能返回，注意不能用之前的intent，它是用来启动当前activity的
    intent.putExtra("data_return", extaData)		//intent不带目的，只是用来传递数据
    setResult(RESULT_OK, intent)		// 设置返回值
    finish()
  }
}
```

调用方法：**`setResult()`**，专门用于向上一个活动返回数据的，需要两个参数：

- 第一个参数，表示返回的处理结果，一般只使用**RESULT_OK**或**RESULT_CANCELED**这两个值
- 第二个参数，要返回去的带有数据的Intent

最后调用finish，用来销毁当前activity

而mainActivity需要接收数据：

```kotlin
button.setOnClickListener{
  Toast.makeText(this, "hello", Toast.LENGTH_LONG).show()
  "hello".showToast()
  val string = "second"
  val intent = Intent(this, SecondActivity::class.java)
  intent.putExtra("data", string)
  startActivityForResult(intent, 1)		// 就这边需要设定启动activity，并且接受返回值。并且需要传递requestCode
```

**`startActivityForResult()`**需要两个参数：

- 第一个参数是Intent；
- 第二个参数是**请求码Request Code**，用于在之后的回调中判断数据的来源

**每个activity被销毁之后会回调到上一个活动的`onActivityResult()`方法**，所以还需要重写该方法：

（可能存在多个activity，会回调到该方法，那么需要做区分，用——requestCode）

```kotlin
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
  super.onActivityResult(requestCode, resultCode, data)
  when(requestCode){				// 根据requestCode进行区分，如果存在多个activity回调，则这个是有多个的
    1 -> if(resultCode == RESULT_OK){		// 然后再根据传递过来的resultCode进行判断
      val returnData = data?.getStringExtra("data_return")		// 允许数据为null，否则就会报错
      Log.d("mainActivity", "back data is $returnData")
    }
  }
}
```

当前按back按钮也会销毁当前activty，一般我们也需要它能够返回值

重写方法`onBackPress()`：内容和上面的一致

```kotlin
override fun onBackPressed() {
  super.onBackPressed()
  val intent = Intent()
  intent.putExtra("data_return", "goodbye")
  setResult(RESULT_OK, intent)
  finish()
}
```

## 5. activity的生命周期

同Java，不赘述了

需要注意的是：**如果activity被中途回收走了**，如果再次用到该activity，会重新从`onCreate`开始创建，eg：在进入停止状态，由于内存不够就会被回收，那么那些临时数据就会消失。但是是可以保存的：**`onSaveInstanceState()`**回调方法：可以保证在活动被回收之前，该方法一定被调用，因此我们可以通过这个方法来**解决活动被回收时临时数据得不到保存的问题**。

```kotlin
override fun onSaveInstanceState(outState: Bundle, outPersistentState: PersistableBundle) {
  super.onSaveInstanceState(outState, outPersistentState)
  val string = "save something"
  outState.putString("save", string)			// 将数据保存在bundle中
}
```

而bundle在`onCreate()`是作为参数被调用，该bundle对象默认是null值，只有被回收过且调用`onSaveInstanceState`方法的，才会在再次调用`onCreate`方法时，才是非null

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
  if(savedInstanceState != null){
    val tempData = savedInstanceState.getString("save")
    Log.d("MainActivity", "temp is $tempData")
  }
}
```

并且：可以将数据传递到bundle中`putString()`，然后将bundle传递给intent中，然后通过intent启动另一个activity，然后从intent中取出bundle`getBundleExtra`，然后bundle再取出`getString`

注意：手机屏幕发生旋转，activity也会发生重新onCreate，也可以用上面的方法保存临时数据——但是，不是很优雅

——其实输入等状态都会onSaveInstanceState，然后可以在onRestoreInstanceState恢复——系统自行实现的

## 6. activity的启动模式

> 具体见Java版的文档，存在4类启动方法：

- standard
- singleTop
- singleTask
- singleInstance

```xml
<activity android:name=".SecondActivity"
          android:launchMode="standard">		// 设置启动模式
  ...
</activity>

```

在intent中设定标志位来为activity指定启动模式：

```kotlin
val Intent = Intent(this, SecondActivity::class.java)
intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
startActivity(intent)
```

两者存在一定的区别：

1. 第二种方式的优先级要高于第一种
2. 在限定范围上有所不同：第一种方式无法直接为Activity设定FLAG_ACTIVITY_CLEAR_TOP标识；第二种无法为Activity指定singleInstance模式

activity中的flag有很多种：

常见：

1. FLAG_ACTIVITY_NEW_TASK：设置启动模式为singletask

2. FLAG_ACTIVITY_SINGLE_TOP：设置启动模式为singletop

3. FLAG_ACTIVITY_CLEAR_TOP：当它启动时，在同一个任务栈中所有位于它上面的Activity都要出栈

   如果和FLAG_ACTIVITY_NEW_TASK配合使用：那么实例如果已经存在，那么系统就会调用它的onNewIntent——singletask会默认有cleartop的效果

   而如果被启动的Activity采用standard模式启动，且已经存在，那么**它连同它（旧的）之上的Activity都要出栈**，且会创建一个新的activity放入栈顶

4. FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS：具有这个标记的Activity不会出现在历史Activity的列表中，即一旦出去了就没了

## 7. activity实现的模式

分为3部分，主要和Java版本一致，就是实现语句上稍微存在不同

1. 可以设定一个baseActivity类，它是所有activity的父类

   然后这里面可以自定义一些activity均有的内容，例如打印当前的页面是哪个activity的、统一管理所有activity

2. 可以一键关闭所有的activity，那么需要将所有activity全部放在一个列表中

   这边稍微的区别是：创建一个单例类

   ```kotlin
   package com.example.activity
   import android.app.Activity
   object ActivityCollector {				// 单例类
       private val activities = ArrayList<Activity>()
   
       fun addActivity(activity: Activity){	// 每个activity在onCreate的时候，需要添加到列表中——在baseActivity中写
           activities.add(activity)
       }
       fun removeActivity(activity: Activity){	// 每个activity在onDestory的时候，从列表中删除——在baseActivity中写
           activities.remove(activity)
       }
       fun removeAllActivity(){
           for (activity in activities){
               if(!activity.isFinishing){
                   activity.finish()
               }
           }
           activities.clear()
           android.os.Process.killProcess(android.os.Process.myPid())
       }
   }
   ```

3. 启动活动之前就是直接在前一个activity中写intent，而正确写法应该是在对应的activity中写一个方法（**静态方法**），然后要启动该活动的activity去调用该方法：

   优点：直观、简化启动过程

   ```kotlin
   companion object{				// 可以按照静态方法的形式调用
     fun actionStart(context: Context, param1: String, param2 : String){
       val intent = Intent(context, SecondActivity::class.java)
       intent.putExtra("data1", param1)
       intent.putExtra("data2", param2)
       context.startActivity(intent)
     }
   }
   
   // MainActivity.kt中调用
   SecondActivity.actionStart(this, "data1", "data2")
   ```

## 8. 禁止销毁创建

在生命周期那边提到，如果屏幕发生旋转，那么默认是会销毁当前activity并重建的，而我们可以配置不让他重新创建。

```xml
android:configChanges="orientation"
```

当然还能串其他的，常用的：只有locale（一般指语言发生变化）、orientation（屏幕旋转）和keyboardHidden（键盘变化，是否调出键盘等）

<img src="/Users/bytedance/Library/Application Support/typora-user-images/image-20210622121926107.png" alt="image-20210622121926107" style="zoom:80%;" />

screenSize和smallestScreenSize，它们两个比较特殊，它们的行为和编译选项有关，但和运行环境无关。

即如果设定了指定的编译选项，那么就会

