# 系统学习recyclerView

 相较于listView，更为好用。

## 基本使用

类似于ListView的自定义实现，并且逻辑更为清楚：

1. 首先实现一个item.xml布局文件

2. 然后实现一个adapter适配器文件

   主要：继承**`RecyclerView.Adapter`**，并且要指定范型类型为即将定义的：**`FruitAdapterRecycle.ViewHolder`**

   定义该内部类，并且内部类继承了`RecyclerView.ViewHolder`，也是recyclerview的内部类——主要是要传入一个view，就是item的布局情况，从而获得布局中的控件对象

   需要重写3个方法：

   - onCreateViewHolder：创建viewholder对象
   - onBindViewHolder：绑定viewholder对象
   - getItemCount：统计item个数

   ```kotlin
   class FruitAdapterRecycler(val fruitList :List<Fruit>) : RecyclerView.Adapter<FruitAdapterRecycler.ViewHolder>() {
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

## 原理

recyclerview是**基于适配器显示的视图**，可以实现：普通列表 ( ListView ) , 网格列表 ( GridView ) , 瀑布流 , 以及各种自定义形式的多容器布局 ;

优势：recyclerview的样式和适配器解耦：

adapter和样式可以区分设置：

1. LayoutManager：布局样式
   - 创建LinearLayoutManager：线性布局
   - 创建GridLayoutManager：网格布局
   - 创建StaggeredGridLayoutManager：瀑布流样式
2. ItemDecoration：间隔样式
3. ItemAnimator：删除动画、添加动画

特点：

1. 功能强大，例如网格、瀑布流只需要进行配置即可直接使用
2. **垃圾回收**：远超listview
3. viewholder规范：有给出viewholder的规范

RecyclerView 使用必须有的关键类：**RecyclerView.ViewHolder , RecyclerView.Adapter , LayoutManager**

可选的类：ItemDecoration, ItemAnimator