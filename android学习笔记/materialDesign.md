# materialDesign

本质上是Android官方提供的一个库，用来统一ui的风格，使ui更美观和好用。向ios对标

现在：AS里面新建工程，默认就是导入了material design——**不需要再导库了**

## 1. toolBar

用来替换原生的actionbar

问题：ActionBar由于其设计的原因，**被限定只能位于Activity的顶部**，从而不能实现一些MaterialDesign的效果，因此官方现在已经不再建议使用ActionBar。

toolbar的优势：**继承了ActionBar的所有功能**，而且**灵活性很高**，可以配合其他控件完成一些Material Design的效果

现在已经默认用material组件库中的东西了，但不是toolbar

```xml
<style name="Theme.MaterialDesign" parent="Theme.MaterialComponents.DayNight.DarkActionBar">
```

DarkActionBar是一个深色的ActionBar主题——即，带了actionbar的深色主题，就是整个app的主题为深色，那么背景色为浅色。

要使用Toolbar来替代ActionBar，因此需要指定一个不带ActionBar的主题：

- Theme.AppCompat.NoActionBar：没有actionbar，且是整个深色主题的
- Theme.AppCompat.Light.NoActionBar：没有actionbar，且整个是浅色主题的

——在themes中对主题进行替换

```xml
<style name="Theme.MaterialDesign" parent="Theme.AppCompat.Light.NoActionBar">
```

然后，在布局文件中添加toolbar

```xml
<?xml version="1.0" encoding="utf-8"?>
// 默认左上角布局的framelayout
<FrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    // 使用xmlns:app指定了一个新的命名空间，为了能够兼容老系统
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">
		
  // appcompat库提供
    <androidx.appcompat.widget.Toolbar
        android:id="@+id/toolBar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:background="@color/design_default_color_primary"// 设置背景色
         // 将整个toolbar的主题设置为深色——里面的字体变成浅色了                              
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
         // 上面主题是深色，则下拉框为也变成深色，和上面字体冲突，所以手动设置为浅色                              
        app:popupTheme="@style/ThemeOverlay.AppCompat.Light"/>

</FrameLayout>
```

可以在androidmanifest.xml中修改标题内容：

```
<activity android:name=".MainActivity"
            android:label="Fruit">
```

给activity增加了一个**android:label属性**，用于指定在Toolbar中显示的文字内容，如果没有指定的话，会默认使用application中指定的label内容

在mainactivity中：调用setSupportActionBar()方法并将Toolbar的实例传入，那么就能够使用了

```
val toolBar : Toolbar = findViewById(R.id.toolBar)
setSupportActionBar(toolBar)
```



### 增加菜单：

创建一个菜单文件夹，并且创建一个菜单文件。

<item>标签来定义action按钮

**app:showAsAction**来指定按钮的显示位置

- always表示永远显示在Toolbar中，如果屏幕空间不够则不显示；
- ifRoom表示屏幕空间足够的情况下显示在Toolbar中，不够的话就显示在菜单当中；
- never则表示永远显示在菜单当中

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/backUp"
        android:icon="@drawable/ic_backup"
        android:title="Backup"
        app:showAsAction="always" />
    <item
        android:id="@+id/delete"
        android:icon="@drawable/ic_delete"
        android:title="Delete"
        app:showAsAction="ifRoom" />
    <item
        android:id="@+id/settings"
        android:icon="@drawable/ic_settings"
        android:title="Settings"
        app:showAsAction="never" />

</menu>
```

然后在mainactivity中添加点击事件：

```kotlin
override fun onCreateOptionsMenu(menu: Menu?): Boolean {
  menuInflater.inflate(R.menu.toolbar, menu)          // 将布局导入
  return true
}

// 注意要选择的方法是：onOptionsItemSelected
override fun onOptionsItemSelected(item: MenuItem): Boolean {
  when(item.itemId){
    R.id.backUp -> Toast.makeText(this, "you clicked the backup", Toast.LENGTH_LONG).show()
    R.id.delete -> Toast.makeText(this, "you clicked the delete", Toast.LENGTH_LONG).show()
    R.id.settings -> Toast.makeText(this, "you clicked the settings", Toast.LENGTH_LONG).show()
  }
  return true
}
```

## 2. 滑动菜单——drawerlayout

滑动菜单：将一些菜单选项隐藏起来，而不是放置在主屏幕上，然后可以通过滑动的方式将菜单显示出来。例如一些app将一些设置等都隐藏在界面的左边，然后从左向右滑动才能出现该内容

——使用drawerlayout：是一个布局

它是一个布局，在布局中允许放入**两个直接子控件**：

- 第一个子控件是**主屏幕中显示的内容**

- 第二个子控件是**滑动菜单中显示的内容**

  需要注意，第二个子控件需要注意：**layout_gravity这个属性是必须指定的**

  （系统不一定提示），因为我们需要告诉DrawerLayout滑动菜单是在屏幕的左边还是右边。

  left表示菜单在左边，right表示在右边，

  start，表示会**根据系统语言进行判断**，如果系**统语言是从左往右的**，比如英语、汉语，**滑动菜单就在左边**，如果系统语言是从右往左的，比如阿拉伯语，滑动菜单就在右边

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.drawerlayout.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
  // 第一个子控件，同上面的toolbar，里面就放了一个toolbar
    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <androidx.appcompat.widget.Toolbar
            android:id="@+id/toolBar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="@color/design_default_color_primary"
            android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
            app:popupTheme="@style/ThemeOverlay.AppCompat.Light"/>
    </FrameLayout>
  // 第二个子控件，默认是隐藏的
    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_gravity="start"		// 需要指定滑动窗口的位置
        android:background="#fff"
        android:text="this is menu"
        android:textSize="30sp" />
</androidx.drawerlayout.widget.DrawerLayout>
```

<img src="/Users/bytedance/Library/Application Support/typora-user-images/image-20210620114130774.png" alt="image-20210620114130774" style="zoom:80%;" />

并且为了提示用户我们存在滑动窗口，那么需要在页面中添加一个home按钮，点击按钮也能打开窗口

- ——toolbar的最左侧是有一个按钮的，被称为**home**，默认图标是“**返回箭头**”，这个按钮是默认隐藏的——**调用setDisplayHomeAsUpEnabled()**可以显示

- 并且可以设置图标——**调用setHomeAsUpIndicator()**方法来设置一个导航按钮图标

- 然后它就成为menu的一部分，**在onOptionsItemSelected**中重写该跳转逻辑即可:

  该home按钮的id是：**android.R.id.home**

  调用DrawerLayout的**openDrawer()**，要求传入一个**Gravity参数**，为了保证这里的行为和XML中定义的一致，我们传入了**GravityCompat.START**

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
				...
				// getSupportActionBar()方法得到了ActionBar的实例
        val toolBar : Toolbar = findViewById(R.id.toolBar)
        setSupportActionBar(toolBar)
        supportActionBar?.let {
            it.setDisplayHomeAsUpEnabled(true)          // 显示home按钮
            it.setHomeAsUpIndicator(R.drawable.ic_menu)         // 设置图标
        }
    }
		
  	...
  
    override fun onOptionsItemSelected(item: MenuItem): Boolean {
        val drawerLayout : DrawerLayout = findViewById(R.id.drawerLayout)
        when(item.itemId){
						// Home按钮的id永远都是android.R.id.home
            android.R.id.home -> drawerLayout.openDrawer(GravityCompat.START)
            R.id.backUp -> Toast.makeText(this, "you clicked the backup", Toast.LENGTH_LONG).show()
            R.id.delete -> Toast.makeText(this, "you clicked the delete", Toast.LENGTH_LONG).show()
            R.id.settings -> Toast.makeText(this, "you clicked the settings", Toast.LENGTH_LONG).show()
        }
        return true
    }
}
```

<img src="/Users/bytedance/Library/Application Support/typora-user-images/image-20210620120404298.png" alt="image-20210620120404298" style="zoom:67%;" />

## 3. NavigationView

导航窗口展示：类似风格

<img src="/Users/bytedance/Library/Application Support/typora-user-images/image-20210620134754385.png" alt="image-20210620134754385" style="zoom:50%;" />

可以使用navigationview实现。

navigationview需要有两个部分组成：menu和headerLayout

- menu：显示具体的菜单项的
- headerLayout：显示头部布局的

menu：在menu文件夹中定义

嵌套了一个<group>标签，表示一个组

**checkableBehavior**指定为single表示组中的所有菜单项只能单选

里面定义了5个<item>标签

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <group android:checkableBehavior="single">
        <item
            android:id="@+id/navCall"
            android:icon="@drawable/nav_call"
            android:title="call"/>
        <item
            android:id="@+id/navFriends"
            android:icon="@drawable/nav_friends"
            android:title="friend" />
        <item
            android:id="@+id/navLocation"
            android:icon="@drawable/nav_location"
            android:title="location" />
        <item
            android:id="@+id/mail"
            android:icon="@drawable/nav_mail"
            android:title="mail" />
        <item
            android:id="@+id/task"
            android:icon="@drawable/nav_task"
            android:title="task" />
    </group>
</menu>
```

headerlayout：在layout文件夹中定义

使用relativelayout布局方式，

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="180dp"
    android:padding="10dp"
    android:background="@color/design_default_color_primary">
  	// 圆形图片
    <de.hdodenhof.circleimageview.CircleImageView
        android:id="@+id/iconImage"
        android:layout_width="70dp"
        android:layout_height="70dp"
        android:src="@drawable/nav_icon"
        android:layout_centerInParent="true" />		// 相对于父布局居中显示
    <TextView
        android:id="@+id/mailText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"		// 相对于父布局底部显示
        android:text="fsdsgsegse@gmail.com"
        android:textColor="#fff"
        android:textSize="14sp" />
    <TextView
        android:id="@+id/nameText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_above="@id/mailText"		// 在指定的布局上方显示
        android:text="fsergfseg"
        android:textColor="#fff"
        android:textSize="14sp" />

</RelativeLayout>
```

创建navigationview：

将刚才drawerlayout的textview替换掉：

主要设定的是：**app:menu/app:headerLayout**，就是指定两部分，就是刚才写好的menu文件和headerlayout文件

```xml
<com.google.android.material.navigation.NavigationView
        android:id="@+id/navView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_gravity="start"			// layout_gravity还是要指定成start
        app:menu="@menu/nav_menu"
        app:headerLayout="@layout/nav_header"/>
```

menu里面的item都是按钮，我们可以编写逻辑：

 需要先获得navigationview对象，然后**setCheckedItem**指定默认选中的组件。**setNavigationItemSelectedListener**，设置选中某个事件的监听器，点击了任意菜单项时，就会**回调到传入的Lambda表达式**当中——注意这个是要有返回值的，默认是true，表示该事件已经处理了。

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
  ...
  val navigateView : NavigationView = findViewById(R.id.navView)
  navigateView.setCheckedItem(R.id.navCall)
  navigateView.setNavigationItemSelectedListener {
    val drawerLayout : DrawerLayout = findViewById(R.id.drawerLayout)
    drawerLayout.closeDrawers()
    true
  }

}
```

### ps：CircleImageView

开源项目CircleImageView，它可以用来轻松实现图片圆形化的功能——需要导入依赖：

```
// 图片圆化
implementation 'de.hdodenhof:circleimageview:3.0.1'
```

使用：

```xml
<de.hdodenhof.circleimageview.CircleImageView
        ...
        android:layout_width="70dp"
        android:layout_height="70dp"
        android:src="@drawable/nav_icon"
        ... />
```

## 4. FloatingActionButton

colorAccent作为按钮的颜色——强调色

使用：

```xml
<com.google.android.material.floatingactionbutton.FloatingActionButton
            android:id="@+id/floatButton"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="bottom|end"			//默认位于下右角
            android:layout_margin="160dp"
            android:src="@drawable/ic_done"
            app:elevation="8dp"/>			// 悬浮高度：高度值越大，投影范围也越大，但是投影效果越淡
```

因为是在framelayout中的，所以需要指定位置，否则默认就是堆在左上角

悬浮按钮本质上还是按钮，所以和普通的button没任何区别

## 5. snackbar：提示工具

Snackbar并不是Toast的替代品，它们有着不同的应用场景。

- Toast的作用是告诉用户现在发生了什么事情，但无交互
- Snackbar它允许在提示中加入一个**可交互按钮**，当用户点击按钮的时候，可以执行一些额外的逻辑操作

在floatbutton中添加逻辑：

基本操作和toast类似，需要传递3个参数：

- view对象：只要是当前界面布局的任意一个View都可以

  使用这个View自动查找最外层的布局，用于展示提示信息

  ——和toast存在不同

- 显示内容

- 显示时长

之后可以调用一个**setAction方法**，传入一个名字，就是附带的按钮

从而让Snackbar不仅仅是一个提示，而是可以和用户进行交互的——里面写的是按钮的逻辑

注意，**最后需要调用.show()**，将其显示出来

——如果不管他，那么时长之后就会自动消失

如果点击了按钮，那么窗口就会消失，会去执行对应的逻辑了

```
floatButton.setOnClickListener{ view ->
            Snackbar.make(view, "data will delete", Snackbar.LENGTH_LONG).setAction("Undo"){
                Toast.makeText(this, "data will restored", Toast.LENGTH_LONG).show()
            }.show()
        }
```

<img src="/Users/bytedance/Library/Application Support/typora-user-images/image-20210620144643017.png" alt="image-20210620144643017" style="zoom:80%;" />

## 6. coordinatorlayout

coordinatorlayout是加强版的framelayout

CoordinatorLayout可以**监听其所有子控件的各种事件**，并自动帮助我们做出最为合理的响应

eg：上面的弹框遮挡，那么会自动将按钮向上移动

——只需要将framelayout进行简单的替换即可。

但是需要注意，该snackbar必须是在corrdinatorlayout中的

——逻辑是：coordinatorlayout-floatbutton-snackbar，那么coordinatorlayout可以监听到

而如果snackbar指定的是更外层的drawablelayout，那么就无法监听到了

## 7. material card

能够实现卡片效果。

MaterialCardView**本质上也是一个FrameLayout**，只是额外提供了**圆角和阴影等**效果。

可以实现一个小demo

### ps：glide库

可以加载：本地图片，还可以加载网络图片、GIF图片甚至是本地视频

首先需要导包：——注意版本，如果版本不对，会发生冲突

```
// glide库
implementation 'com.github.bumptech.glide:glide:4.11.0'
```

使用：

- Glide.with()方法并传入一个Context、Activity或Fragment参数
- 调用load()方法加载图片，可以是一个URL地址，也可以是一个本地路径，或者是一个资源id
- 调用into()方法将图片设置到具体某一个ImageView中。

eg：

```
Glide.with(context).load(fruit.src).into(holder.fruitView)
```



使用recyclerview，

对应的要写：fruit类，fruitadapter类，fruit_item.xml，将recyclerview引入layout中

而material card就是在fruit_item中引入的

```xml
<?xml version="1.0" encoding="utf-8"?>
<com.google.android.material.card.MaterialCardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="5dp"
    app:cardCornerRadius="4dp">			// 圆角
    <LinearLayout
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
        <ImageView
            android:id="@+id/fruitImage"
            android:layout_width="match_parent"
            android:layout_height="100dp"
            android:scaleType="centerCrop" />
        <TextView
            android:id="@+id/fruitText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_gravity="center_horizontal"
            android:layout_margin="5dp"
            android:textSize="16sp"/>
    </LinearLayout>
</com.google.android.material.card.MaterialCardView>
```

（通过app:elevation属性指定卡片的高度，和floatbutton一致）

就是在onbindviewholder中使用了**`Glide.with(context).load(fruit.src).into(holder.fruitView)`**绑定了图片

然后在mainactivity中，将recyclerview导入：layoutmanager、adapter

<img src="/Users/bytedance/Library/Application Support/typora-user-images/image-20210620161255450.png" alt="image-20210620161255450" style="zoom:67%;" />

## 8. AppBarLayout

AppBarLayout实际上是一个**垂直方向的LinearLayout**，它在内部做了很多滚动事件的封装。

只需要两步就可以了：

- 第一步将Toolbar嵌套到AppBarLayout中

- 第二步给RecyclerView指定一个布局行为：

  ```
  app:layout_behavior="@string/appbar_scrolling_view_behavior"
  ```

当RecyclerView滚动的时候就已经将滚动事件通知给AppBarLayout，可以对该操作进行处理。当AppBarLayout接收到滚动事件的时候，它**内部的子控件**其实是可以**指定如何去响应这些事件的**，eg：是否要跟着上去、是否要显示等。

```xml
<com.google.android.material.appbar.AppBarLayout
            ...>
            <androidx.appcompat.widget.Toolbar
                app:layout_scrollFlags="scroll|enterAlways|snap"/>
        </com.google.android.material.appbar.AppBarLayout>
```

 toolbar去对应着响应里面的事件：`scroll|enterAlways|snap`

- scroll表示当RecyclerView向上滚动的时候，Toolbar会跟着一起向上滚动并实现隐藏；
- enterAlways表示当RecyclerView向下滚动的时候，Toolbar会跟着一起向下滚动并重新显示；
- snap表示当Toolbar还没有完全隐藏或显示的时候，会根据当前滚动的距离，自动选择是隐藏还是显示

——感觉这一块只能是记忆，因为是普遍操作，具体逻辑不是很懂。

## 9. swiperefreshLayout下拉刷新

本质上是一种布局

方法：把想要实现下拉刷新功能的**控件放置到SwipeRefreshLayout中**，就可以迅速让这个控件支持下拉刷新。

首先添加依赖：

```kotlin
// 下拉刷新效果
implementation 'androidx.swiperefreshlayout:swiperefreshlayout:1.0.0'
```

然后将上面的recyclerview添加到swiperefresh中，那么recyclerview就能实现刷新了。

注意：前面的appbar对应的布局行为，需要移动到swiperefreshlayout中。

处理具体逻辑：

1. 先获取该对象

   可以设置背景颜色：setColorSchemeResources

   可以设置刷新时对应调用的逻辑代码：setOnRefreshListener，里面添加lambda代码

```kotlin
val swiperefreshLayout : SwipeRefreshLayout = findViewById(R.id.swiperfreshLayout)
// 设置刷新控件的颜色
swiperefreshLayout.setColorSchemeResources(R.color.design_default_color_primary)
// 每次刷新时对应的逻辑
swiperefreshLayout.setOnRefreshListener {
  refreshFruits(adapter, swiperefreshLayout)
}
```

这边模拟的是发起网络请求，创建一个线程去发起请求，模拟请求的时间：`Thread.sleep(xxx)`，然后`runOnUiThread`方法，从子线程切换到主线程。

```kotlin
private fun refreshFruits(adapter: FruitAdapter, swipeRefreshLayout: SwipeRefreshLayout){
        thread {
            Thread.sleep(2000)		// 这个只是模拟网络请求
            runOnUiThread{		// 模拟将请求到的数据进行处理后更新到recyclerview中
                initFruitArray()
                adapter.notifyDataSetChanged()		// 通知adapter
                swipeRefreshLayout.isRefreshing = false		// 表示刷新结束，可以隐藏进度条

            }
        }
    }
```

2.  swipeRefreshLayout.isRefreshing，当刷新完毕后，需要设置为false，表示刷新结束，会将进度条

## 9. collapsingToolbarLayout可折叠式标题栏

这个是在actionbar的基础上，toolbar的基础上，一个究极改进版。

先明确逻辑关系：

1. collapsingToolbarLayout被限定只能作为AppBarLayout的直接子布局来使用
2. AppBarLayout又限定在coordinatorlayout

所以，如果要使用的话必须：coordinatorlayout -- AppBarLayout -- collapsingToolbarLayout

```xml
<?xml version="1.0" encoding="utf-8"?>
// 最外层布局，是一个改进版的framelayout，能够监听子组件的事件，并作出合理响应
<androidx.coordinatorlayout.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
		
		// 第一层：appbarlayout，垂直方向的LinearLayout，能够监听其他组件的，然后其appbarlayout的子组件能够去响应
    <com.google.android.material.appbar.AppBarLayout
        android:id="@+id/appBar"
        android:layout_width="match_parent"
        android:layout_height="250dp">
				
				// 第三层：可折叠式标题栏
				// contentScrim指出折叠之后的颜色
				// 折叠的效果：下面滚动一起跟着滚；折叠之后不再移出
        <com.google.android.material.appbar.CollapsingToolbarLayout
            android:id="@+id/collapsingToolBar"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:theme="@style/ThemeOverlay.AppCompat.Light"
            app:contentScrim="@color/design_default_color_primary"
            app:layout_scrollFlags="scroll|exitUntilCollapsed">

            <ImageView
                android:id="@+id/imageFruitView"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:scaleType="centerCrop"			// 缩放
                app:layout_collapseMode="parallax"/>	// 指定折叠模式——错位

            <androidx.appcompat.widget.Toolbar
                android:id="@+id/fruitToolBar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_collapseMode="pin"/>		// 折叠是保持不变
        </com.google.android.material.appbar.CollapsingToolbarLayout>
    </com.google.android.material.appbar.AppBarLayout>
		
		// 第一层：NestedScrollView，是在scrollview的基础上增加了嵌套响应滚动事件的功能
		// ScrollView还是NestedScrollView：只允许存在一个直接子布局
    <androidx.core.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        // 滚动时，上面的appbar会跟着响应
        app:layout_behavior="@string/appbar_scrolling_view_behavior">
				
				// 第二层：主要是nestedscrollview只允许一个直接子布局
        <LinearLayout
            android:orientation="vertical"
            android:layout_width="match_parent"
            android:layout_height="wrap_content">
						
						// 第三层：卡片
            <com.google.android.material.card.MaterialCardView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_margin="20dp"
                app:cardCornerRadius="4dp">
								
								// 第四层：卡片里套了具体文本
                <TextView
                    android:id="@+id/fruitContentText"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_margin="10dp"
                    android:textSize="14dp"/>
            </com.google.android.material.card.MaterialCardView>
        </LinearLayout>
    </androidx.core.widget.NestedScrollView>
    
    // 浮动按钮
    <com.google.android.material.floatingactionbutton.FloatingActionButton
        android:id="@+id/fruitFloatButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        android:src="@drawable/ic_comment"
        app:layout_anchor="@id/appBar"				//指定锚点
        app:layout_anchorGravity="end|bottom"/>			// 先对锚点的位置
</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

技巧：

ScrollView还是NestedScrollView，它们的内部都**只允许存在一个直接子布局**。

如果我们想要在里面放入很多东西的话，通常会先嵌套一个LinearLayout，然后再在LinearLayout中放入具体的内容就可以了。

逻辑：

```kotlin
class FruitActivity : AppCompatActivity() {
    companion object{
        const val FRUIT_NAME = "fruit_name"
        const val FRUIT_IMAGE_ID = "fruit_src"
    }
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_fruit)

        val toolBar : Toolbar = findViewById(R.id.fruitToolBar)
        val collapsingToolbarLayout : CollapsingToolbarLayout = findViewById(R.id.collapsingToolBar)
        val imageView : ImageView = findViewById(R.id.imageFruitView)
        val contentText : TextView = findViewById(R.id.fruitContentText)

        val fruitName = intent.getStringExtra(FRUIT_NAME) ?: ""
        val fruitSrc = intent.getIntExtra(FRUIT_IMAGE_ID, 0)

        setSupportActionBar(toolBar)            // 使用toolbar
        supportActionBar?.setDisplayHomeAsUpEnabled(true)   // 显示返回按钮
        collapsingToolbarLayout.title = fruitName
        Glide.with(this).load(fruitSrc).into(imageView)	// 用glide处理图片
        contentText.text = generateContent(fruitName)
    }
		
		// toolbar中的返回按钮监听器
  	// ——具体见toolbar
    override fun onOptionsItemSelected(item: MenuItem): Boolean {
        when(item.itemId){
            android.R.id.home ->{
                finish()			// 点击返回按钮，就消灭该activity
                return true		// 需要有返回值
            }
        }
        return super.onOptionsItemSelected(item)
    }
    private fun generateContent(fruitName : String) = fruitName.repeat(500)
}
```

——是从前面的mainactivity中跳转过来的，所以mainactivity中需要编写跳转逻辑

——在recyclerview中，添加跳转逻辑

```kotlin
holder.fruitView.setOnClickListener{
  val position = holder.adapterPosition
  val fruit = fruits[position]
  val intent = Intent(context, FruitActivity::class.java).apply {
    putExtra(FruitActivity.FRUIT_NAME, fruit.name)
    putExtra(FruitActivity.FRUIT_IMAGE_ID, fruit.src)
  }
  context.startActivity(intent)
}
```

## ps: