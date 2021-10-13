#  UI

如果你需要在XML中**引用一个id**，就使用**@id/id_name这种语法**，而如果你需要在XML中**定义一个id**，则要使用**@+id/id_name这种语法**

大部分的xml都和Java版本一致，就是逻辑实现上存在不同。下面会对这些不同做补充。

## 1. button

button要为点击事件注册一个监听器，所以需要注册监听器：`setOnClickListener`，该监听器需要传递一个参数`View.OnClickListener`对象，是一个接口，所以需要对它进行实现，它有唯一一个待实现的方法`onClick`，然后对它进行实现即可

简化版：

```kotlin
var button : Button = findViewById(R.id.button_1)
button.setOnClickListener{
  val string = "second"
  SecondActivity.actionStart(this, "data1", "data2")
}
```

完整版：

```kotlin
class SecondActivity : BaseActivity(), View.OnClickListener{
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_second)
    
    val button : Button = findViewById(R.id.button_2)
    button.setOnClickListener(this)
  }

  override fun onClick(v: View?) {            // 这边的view是指button自己
    when(v?.id){            // 判断view的id
      R.id.button_2 ->{
        ActivityCollector.removeAllActivity()
      }
    }
  }
  
}
```

## 2. ImageView

ImageView的属性补充：

1. scaleType

   主要负责对照片的显示进行处理：包括是否进行缩放、等比缩放、缩放后展示位置

   可以在代码中/xml进行设置

   ```
   android:scaleType="centerInside"          //XML中
   imageView.setScaleType(ImageView.ScaleType.CENTER_INSIDE);    //代码中
   ```

   1. 以`FIT_`开头的4种，它们的共同点是都会对图片进行缩放；

      以`CENTER_`开头的3种，它们的共同点是居中显示，图片的中心点会与`ImageView`的中心点重叠；

      `ScaleType.MATRIX`，这种就直接翻到最后看内容吧；

      

2. 

xml中的实现一致，而逻辑部分实现存在不同：

```kotlin
class SecondActivity : BaseActivity(), View.OnClickListener{
  var image : ImageView? = null				// 不知道有没有更简单的方法，创建一个全局变量
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_second)

    val button : Button = findViewById(R.id.button_2)
    image = findViewById(R.id.image_view)		// 对变量赋值
    button.setOnClickListener(this)
  }

  override fun onClick(v: View?) {            // 这边的view是指button自己
    when(v?.id){            // 判断view的id
      R.id.button_2 ->{
        image?.setImageResource(R.drawable.ic_heavy_snow)		// 避免值为null，重写设置图片的内容
      }
    }
  }
}
```

## 3. AlertDialog

这个UI是直接写在程序中的，通过创建对象来显示

```kotlin
// button触发
override fun onClick(v: View?) {            // 这边的view是指button自己
  when(v?.id){            // 判断view的id
    R.id.button_2 ->{
      AlertDialog.Builder(this).apply {		// 利用apply标准函数
        setTitle("alert")
        setMessage("import!")
        setCancelable(false)
        setPositiveButton("ok"){
          dialog, which ->}
        setNegativeButton("cancel"){
          dialog, which -> }
        show()
      }
    }
  }
}
```

## 4. ListView

普通的使用很简单，麻烦的是自定义的ListView页面

步骤：

1. 自定义item的内容——fruit_item.xml

   同Java版本的

2. 自定义ArrayAdapter——fruitAdapter.kt，需要继承arrayAdapter，然后重写getView方法

   存在不同

3. 最后引入使用

   在xml中的使用和listView一致，但是逻辑中稍微有些变化，要将原来的ArrayAdapter修改成FruitAdapter，其余没啥变化

```kotlin
// 传递的变量和普通的arrayAdapter一致，就是 activity， item的布局文件（之前是默认的），数据源
// resourceId是新增加的私有变量，因为它还需要在非构造方法中使用，所以需要保存下来
class FruitAdapter(activity: Activity, val resourceId : Int, data : List<Fruit>) : ArrayAdapter<Fruit>(activity, resourceId, data){
    override fun getView(position: Int, convertView: View?, parent: ViewGroup): View {
        val view = LayoutInflater.from(context).inflate(resourceId, parent, false)
        val fruitImg : ImageView = view.findViewById(R.id.fruit_img)
        val fruitText : TextView = view.findViewById(R.id.fruit_name)
        val fruit = getItem(position)
        if(fruit != null){
            fruitImg.setImageResource(fruit.imgId)
            fruitText.setText(fruit.name)
        }
        return view
    }
}

// 定义数据类
data class Fruit(val name : String, val imgId : Int)
```

使用：

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
  super.onCreate(savedInstanceState)
  setContentView(R.layout.activity_second)

  val fruit = ArrayList<Fruit>()
  initFruits()			// 初始化列表
  val fruitAdapter = FruitAdapter(this, R.layout.fruit_items, fruit)	// 调用自定义的adapter
  val listView : ListView = findViewById(R.id.list_view)
  listView.adapter = fruitAdapter			// 赋值（和普通的流程一致）
  // 还定义了点击效果——传递的参数是一个接口OnItemClickListener，需要实现里面的方法onItemClick，用lambda实现了，可以对比Java来看
  listView.setOnItemClickListener{ parent, view, position, id ->
                                  val fruit_item = fruit[position]
                                  Toast.makeText(this, fruit_item.name, Toast.LENGTH_LONG).show()
                                 }
}
```

## 5. RecyclerView

更为好用。

类似于ListView的自定义实现，并且逻辑更为清楚：

1. 首先实现一个item.xml布局文件

   这边可以服用上面item的布局文件

2. 然后实现一个adapter适配器文件

   主要：继承`RecyclerView.Adapter`，并且要指定范型类型为即将定义的：`FruitAdapterRecycle.ViewHolder`

   定义该内部类，并且内部类继承了`RecyclerView.ViewHolder`，也是recyclerview的内部类——主要是要传入一个view，就是item的布局情况，从而获得布局中的控件对象

   需要重写3个方法：

   - onCreateViewHolder：创建viewholder对象
   - onBindViewHolder：绑定viewholder对象
   - getItemCount：统计item个数

   ```kotlin
   class FruitAdapterRecycle(val fruitList :List<Fruit>) : RecyclerView.Adapter<FruitAdapterRecycle.ViewHolder>() {
       inner class ViewHolder(view : View) : RecyclerView.ViewHolder(view){	// 就是获得每个item的布局
           val imgeView : ImageView = view.findViewById(R.id.fruit_img)
           val textView : TextView = view.findViewById(R.id.fruit_name)
       }
   		
     	// 创建viewholder实例：先获取view，然后将view传入到viewholder的构造函数中创建viewholder对象，最后返回该对象
       override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
           val view = LayoutInflater.from(parent.context).inflate(R.layout.fruit_items, parent, false)
           val holder = ViewHolder(view)
           return holder
       }
   
     	// 数据赋值，item进入屏幕的时候会被赋值，通过传入的viewholder获取item的里面组件对象，然后通过position，即列表中的位置，将具体的值赋值过去
       override fun onBindViewHolder(holder: ViewHolder, position: Int) {
           val fruit = fruitList[position]
           holder.imgeView.setImageResource(fruit.imgId)
           holder.textView.setText(fruit.name)
       }
   		
     	// 统计列表个数
       override fun getItemCount() = fruitList.size
   }
   ```

3. 使用

   - 在布局文件中使用recyclerView

      当然还需要导包。

     ```kotlin
     <androidx.recyclerview.widget.RecyclerView
     android:id="@+id/recycle_view"
     android:layout_width="match_parent"
     android:layout_height="match_parent" />
     ```

   - 逻辑中使用，类似于listView

     主要绑定2个东西：

     layoutManager：主要指定recyclerview的布局方式，这边指定的是线性布局；

     adapter：主要指定适配器，调用刚才重写的适配器类

     ```kotlin
     override fun onCreate(savedInstanceState: Bundle?) {
       super.onCreate(savedInstanceState)
       setContentView(R.layout.activity_second)
     
       val fruit = ArrayList<Fruit>()
     
       val recyclerView : RecyclerView = findViewById(R.id.recycle_view)
       val layoutManager = LinearLayoutManager(this)		// 布局方式对象
       val adapter = FruitAdapterRecycle(fruit)		// 适配器对象
       recyclerView.layoutManager = layoutManager
       recyclerView.adapter = adapter
     }
     ```

RecyclerView的优势之一在于：有很好的扩展性，即横向、纵向滚动可以很方便的切换

变成横向列表：只需要修改：item.xml的布局方向、在layoutManager里面进行修改即可

而如果想实现点击效果，注意recyclerview没有默认点击效果的实现，那么可以每部分分开作为点击事件——提高灵活性。

```kotlin
override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
  val view = LayoutInflater.from(parent.context).inflate(R.layout.fruit_items, parent, false)
  val holder = ViewHolder(view)
  
  /**********************************************/
  holder.imgeView.setOnClickListener{			// 每个组件都添加一个点击事件
    val position = holder.absoluteAdapterPosition		// 获得在列表的位置，为了获取对象的内容
    val fruit = fruitList[position]
    Toast.makeText(view.context, "you click ${fruit.imgId}", Toast.LENGTH_LONG).show()
  }
  holder.textView.setOnClickListener{
    val position = holder.absoluteAdapterPosition
    val fruit = fruitList[position]
    Toast.makeText(view.context, "you click ${fruit.name}", Toast.LENGTH_LONG).show()
  }
  /**********************************************/
  
  return holder
}
```

## 6. ScrollView

ScrollView控件，我们就可以以滚动的形式查看屏幕外的内容。

主要是对里面的内容，可以滚动显示

## 实践

首先全局的布局文件：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#d8e0e8">
    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recycle_view"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"/>
  // 最下面的布局
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
        <EditText
            android:id="@+id/inputText"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:hint="type something here"
            android:maxLines="2"/>
        <Button
            android:id="@+id/send"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="send" />
    </LinearLayout>

</LinearLayout>
```

对于每个item，设定布局文件，分为leftitem和rightitem

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"		// 注意这个，不能是跟随父类，那么一个item就会填满整个页面
    android:padding="10dp">		// item之间的间隙
    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="right"
        android:background="@drawable/message_right">
        <TextView
            android:id="@+id/rightMsg"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:layout_margin="5dp"
            android:textColor="#000" />
    </LinearLayout>
</FrameLayout>

<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:padding="10dp">
    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="right"			// 布局在右边
        android:background="@drawable/message_right">		// 背景图变了
        <TextView
            android:id="@+id/rightMsg"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:layout_margin="5dp"
            android:textColor="#000" />
    </LinearLayout>

</FrameLayout>
```

逻辑文件：

创建一个类，用来表示对话内容，并且为了区分收到的和发送的，所以需要有type类型

```kotlin
class Msg(val content: String, val type: Int) {
    companion object{
        const val RECEIVED = 0
        const val SEND = 1
    }
}
```

实现适配器：

```kotlin
class MsgAdapter(val viewList : List<Msg>) : RecyclerView.Adapter<RecyclerView.ViewHolder>(){
    override fun getItemViewType(position: Int): Int {
        return viewList[position].type
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
        val holder = if(viewType == Msg.RECEIVED) {
            val view = LayoutInflater.from(parent.context).inflate(R.layout.msg_left_item, parent, false)
            LeftViewHolder(view)

        }
        else{
            val view = LayoutInflater.from(parent.context).inflate(R.layout.msg_right_item, parent, false)
            RightViewHolder(view)
        }
        return holder
    }

    override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
        val msg = viewList[position]
        when(holder){
            is LeftViewHolder -> holder.leftMsg.text = msg.content
            is RightViewHolder -> holder.rightMsg.text = msg.content
        }
    }

    override fun getItemCount() = viewList.size
}
```

密封类：

```kotlin
sealed class MsgViewHolder(view : View) : RecyclerView.ViewHolder(view){
}
class LeftViewHolder(view: View) : MsgViewHolder(view){
    val leftMsg : TextView = view.findViewById(R.id.leftMsg)
}
class RightViewHolder(view: View) : MsgViewHolder(view){
    val rightMsg : TextView = view.findViewById(R.id.rightMsg)

}
```

使用类：

```kotlin
class SecondActivity : BaseActivity(), View.OnClickListener{
    private val msgList = ArrayList<Msg>()
    private var adapter: MsgAdapter ?= null
    private lateinit var button: Button
    private lateinit var input : EditText
    private lateinit var recyclerView: RecyclerView
    companion object {
        fun actionStart(context: Context, param1: String, param2: String) {
            val intent = Intent(context, SecondActivity::class.java)
            intent.putExtra("data1", param1)
            intent.putExtra("data2", param2)
            context.startActivity(intent)
        }
    }
    override fun onCreate(savedInstanceState: Bundle?){
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_second)
        initMsg()

        button = findViewById(R.id.send)
        input = findViewById(R.id.inputText)
        recyclerView = findViewById(R.id.recycle_view)

        val layoutManager = LinearLayoutManager(this)
        val adapter = MsgAdapter(msgList)
        recyclerView.layoutManager = layoutManager
        recyclerView.adapter = adapter

        button.setOnClickListener(this)
    }

    override fun onClick(v: View?) {
        when(v){
             button -> {
                 val content = input.text.toString()
                 if (content.isNotEmpty()){
                     val msg = Msg(content, Msg.SEND)
                     msgList.add(msg)
                     adapter?.notifyItemInserted(msgList.size - 1)		// 刷新页面
                     recyclerView.scrollToPosition(msgList.size - 1)
                     input.setText("")
                 }
            }
        }
    }
    private fun initMsg(){
        val msg1 = Msg("hello", Msg.RECEIVED)
        msgList.add(msg1)
        val msg2 = Msg("hello, how are you ? ", Msg.SEND)
        msgList.add(msg2)
        val msg3 = Msg("this is tom, and you?", Msg.RECEIVED)
        msgList.add(msg3)

    }
}
```

