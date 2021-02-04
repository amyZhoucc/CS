# UI开发

AS支持使用可视化的方式进行页面设计，即拖拽那些组件从而实现。但是，不利于知道背后的原理，且对于复杂的页面就无法单纯的用拖拽来实现。所以，学习的时候，都采用XML代码来实现页面设计。

# 常用控件的使用

## 1. TextView

Android中最简单的一个控件了，主要就是在页面上显示一段文本信息。

如下图就是一个最简单的布局：

```java
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_height="match_parent"
    android:layout_width="match_parent">
    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/text_view"
        android:text="hello_world" />
</LinearLayout>
```

理解：可以看到里面的一个<TextView>就是一个控件

1. 使用`android:id`给当前控件定义了一个唯一标识符

2. **`android:layout_width`和`android:layout_height`**指定了控件的宽度和高度：**Android中所有的控件都具有这两个属性**

   有3个选项：

   - `match_parent`：当前控件的大小和父布局的大小一样，也就是由父布局来决定当前控件的大小——即**和父布局同步**
   - `fill_parent`：同上，但是官方更推荐`match_parent`
   - `wrap_content`：当前控件的大小能够刚好包含住里面的内容，**控件内容决定当前控件的大小**

   当然也可以指定一个固定的大小，这可能会存在适配的问题

   所以上面控件：宽度和父布局一样，即手机屏幕的宽度一样；高度足够包含住里面的内容就行

   <img src="pic\image-20210203210844675.png" alt="image-20210203210844675" style="zoom: 67%;" />

3. TextView的文字是**默认左上角对齐的**，所以如上图会显示在左侧

4. TextView可以修改**文字的对齐方式：`android:gravity`**

   注意，和**`android:layout_gravity`：是整个TextView控件在屏幕上的布局**（由于前面的宽度设定的是match_parent，所以设置为center，没有效果，因为控件整个已经居中了）

   对齐方式可选的有：

   - top、bottom、left、right、center

   可以**用`|`间隔来指定多个值**，效果等同于`center_vertical|center_horizontal`

   eg：

   ```xml
   android:gravity="center"
   ```

5. TextView可以修改文字的大小和颜色

   **`android:textSize`**属性可以指定文字的大小，字体大小使用sp作为单位

   通过**`android:textColor`**属性可以指定文字的颜色

   ```xml
   android:textSize="24sp"
   android:textColor="#ff0000"
   ```

<img src="pic\image-20210203213921049.png" alt="image-20210203213921049" style="zoom:50%;" />

## 2. Button

基本功能同TextView，能够设置控件的宽度、高度等，并且设置内容的字体大小、颜色等，具体可以看手册

```xml
<Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="20sp"
        android:textColor="#000000"
        android:id="@+id/button_1"
        android:text="button" />
```

可以根据下图发现，button有一些默认的格式，比如：内容自动居中并且内容会自动转换为大写；控件有圆角，控件有默认背景色（都是和TextView有不同的地方）。

<img src="pic\image-20210204111231631.png" alt="image-20210204111231631" style="zoom:50%;" />

**系统默认会对Button中的所有英文字母自动进行大写转换**，可以修改来禁用该默认配置。

button的通常使用：（在activity中已经用了很多了）

为Button的点击事件**注册一个监听器**

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button button = (Button)findViewById(R.id.button_1);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(MainActivity.this, "you clicked me", Toast.LENGTH_SHORT).show();
            }
        });
    }
}
```

理解：

1. 在`onCreate()`里面创建一个监听器

2. 需要先获取该button对象——`findViewById`，记得要强转一下；后注册监听器`button.setOnClickListener()`

3. 由于`setOnClickListener()`需要传递一个参数，该参数是`View.OnclickedListener`，是一个View的内部接口，需要有类去实现该接口（即只需要实现`onClick()`方法），所以常见是传入一个**匿名内部类**，它继承了`View.OnclickedListener`，然后重写了该接口方法即可。当然，如果觉得这个不利于代码阅读，也可以使用实现接口的方式来进行注册

   eg：

   ```java
   public class MainActivity extends AppCompatActivity implements View.OnClickListener{	// 实现该接口
   
       @Override
       protected void onCreate(Bundle savedInstanceState) {
           super.onCreate(savedInstanceState);
           setContentView(R.layout.activity_main);
           Button button = (Button)findViewById(R.id.button_1);
           button.setOnClickListener(this);		// 直接调用自己即可，因为自己实现了该接口，也是该接口的子类
       }
   
       @Override
       public void onClick(View v) {		// 具体实现接口方法
           Toast.makeText(MainActivity.this, "you clicked me", Toast.LENGTH_SHORT).show();
   
       }
   }
   ```

   效果同上

## 3. EditText

EditText是用于和用户进行交互的另一个重要控件

特性：**允许用户在控件里输入和编辑内容**，程序会对该内容读取处理并做出响应。

应用场景：发短信、发微博、聊QQ等操作时——有关于用户输入的，都少不了它

```xml
<EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:maxLines="2"
        android:id="@+id/edit_text"
        android:hint="tell me something"/>
```

理解：其他同上。

只关注特殊的：

1. `android:hint="xxxx"`——指定一段提示性的文本当我们输入任何内容时，这段文本就会自动消失

2. `android:maxLines="2"`——指定输入文本的显示最大不超过2行，如果超过了就上滚，EditText不会再无限扩张

   因为，EditText设置的高度的属性是`wrap_context`，所以如果输入的内容不断增加，这部分会不断扩张，界面会很难看，所以用`maxLines`这个属性可以控制住

<img src="pic\image-20210204114957175.png" alt="image-20210204114957175" style="zoom:67%;" />

EditText通常和button配合使用：点击按钮，来获取EditText里面的内容并做出处理

```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener{
    private EditText editText;			// 实例变量
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button button = (Button)findViewById(R.id.button_1);
        editText = (EditText)findViewById(R.id.edit_text);	// 同button方式一样，去获取editText对象
        button.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {		// 点击按钮就去获取editText的内容
        switch (v.getId()){
            case R.id.button_1:
                String input = editText.getText().toString();	// 获取editText的输入内容，并且转换为String类型
                Toast.makeText(MainActivity.this, input, Toast.LENGTH_SHORT).show();
                break;
            default: break;
        }
    }
}
```

理解：

1. editText对象的获取方式和button类似
2. 获取editText的内容的方式：`editText.getText()`，返回值是`Editable`类型，可以转换为string类型，通过`toString()`

## 4. ImageView

用于在界面上展示图片，图片通常都是放在**以“drawable”开头的目录下的**。一般工程下有一个drawable目录，不过由于这个目录没有指定具体的分辨率，所以一般不使用它来放置图片

将自己找的两张图片放到`mipmap-hdpi`文件下面

（ps：书中说自己新建一个`drawable-xhpi`，实验了一下发现不可以，应该是AS限定了不能取这个名字）

```xml
<ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/image_view"
        android:src="@mipmap/img1" />
```

由于图片的宽度和高度是不确定的，所以设置都是跟随图片大小来确定ImageView的范围——这样就保证了不管图片的尺寸是多少，图片都可以完整地展示出来

如下图：

<img src="pic\image-20210204142004314.png" alt="image-20210204142004314" style="zoom:50%;" />

还可以配合按钮使用，点击按钮就切换图片：

```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener{
    private EditText editText;
    private ImageView imageView;		// 私有实例变量

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button button = (Button)findViewById(R.id.button_1);
        editText = (EditText)findViewById(R.id.edit_text);
        imageView = (ImageView)findViewById(R.id.image_view);	// 同上的获取方法
        button.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()){
            case R.id.button_1:
                imageView.setImageResource(R.mipmap.img2);	// 点击按钮就切换另一张图片
            default: break;
        }
    }
}
```

理解：

1. 获取`ImageView`对象的方法同button、TextView
2. 重新设置展示的图片`imageView.setImageResource(xxxx)`，这回注意到传递的参数都是int类型，即`R.id.xxx`/`R.mipmap.xxx`所有资源都有一个编号，通过传递编号来调用。

ps：根据上面的控件设计可以发现：Android控件的用法基本上都很相似：**给控件定义一个id，再指定控件的宽度和高度**，然后再适当**加入一些控件特有的属性**就差不多了。

## 5. ProcessBar

用于在界面上显示一个进度条，表示程序正在加载一些数据

```xml
<ProgressBar
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/process_bar" />
```

可以看到进度条一直在转（不会停），并且图形默认是在控件中居中的

<img src="pic\image-20210204143437202.png" alt="image-20210204143437202" style="zoom:50%;" />

那么如何控制，进度条在加载完成后消失呢？

首先了解：**Android控件的可见属性**——`android:visiablity`，**所有的Android控件都具有这个属性。**

有3种属性：

- visible：默认值，不设置该属性，都是默认可见的
- invisible：不可见，但是仍然占据所在的空间（可以认为是变透明了）
- gone：消失了，不可见且不占据所在的空间了

但是，xml就是一锤子买卖，在activity创建的时候就加载进去，里面空间的配置就不能变化了——能够通过代码来改变配置的状态**`setVisiablity`——来设置空间的可见状态**，可以传入`View.VISIBLE`、`View.INVISIBLE`和`View.GONE`这3种值

通过按钮来实现进度条是否可见：

```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener{
    private EditText editText;
    private ImageView imageView;
    private ProgressBar progressBar;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button button = (Button)findViewById(R.id.button_1);
        editText = (EditText)findViewById(R.id.edit_text);
        imageView = (ImageView)findViewById(R.id.image_view);
        progressBar = (ProgressBar)findViewById(R.id.process_bar);
        button.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()){
            case R.id.button_1:
                if(progressBar.getVisibility() == View.GONE){	// 如果原来是消失的，那么变得可见
                    progressBar.setVisibility(View.VISIBLE);
                }
                else{							// 如果原来是可见的，那么变得消失
                    progressBar.setVisibility(View.GONE);
                }
            default: break;
        }
    }
}
```

理解：根据前面的命名规则，可以自然推出，获得当前控件的可视状态可以用`xxx.getVisiable()`，如果要设置可是状态可以用`xxx.setVisiable()`，同理，其他属性也可以用类似的方法

进度条还可以有不同的样式：默认是圆圈，还可以将它指定成水平进度条等很多形态

```xml
<ProgressBar
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        style="?android:progressBarStyleHorizontal"
        android:max="100"
        android:id="@+id/process_bar" />
```

理解：

1. 样式可以用`style="xxxx"`设定，注意这个没有android前缀了
2. 通过`android:max="xxxx"`给进度条设置一个最大值——默认进度条状态为0（且不会像圆圈一样滚动）

可以通过代码来改变进度：——通过按钮

```java
public void onClick(View v) {
    switch (v.getId()){
        case R.id.button_1:
            int process = progressBar.getProgress();
            process += 10;
            progressBar.setProgress(process);
        default: break;
    }
}
```

理解：按一下按钮，+10的进度，获得当前进度用`processBar.getProcess()`，设置当前进度用`processBar.setProcess()`

<img src="pic\image-20210204150055700.png" alt="image-20210204150055700" style="zoom:50%;" />

## 6. AlertDialog

在当前的界面弹出一个对话框，这个对话框是置顶于所有界面元素之上的，能够屏蔽掉其他控件的交互能力。**AlertDialog一般都是用于提示一些非常重要的内容或者警告信息**，eg：防止用户误删重要内容，在删除前弹出一个确认对话框

常用方法是写在逻辑代码中。

eg：点击按钮，弹出警告

```java
public void onClick(View v) {
    switch (v.getId()){
        case R.id.button_1:
            AlertDialog.Builder dialog= new AlertDialog.Builder(MainActivity.this);
            dialog.setTitle("ALERT");
            dialog.setMessage("important!!!");
            dialog.setCancelable(false);
            dialog.setPositiveButton("OK", new DialogInterface.OnClickListener() {
                @Override
                public void onClick(DialogInterface dialog, int which) {
                }
            });
            dialog.setNegativeButton("CANCEL", new DialogInterface.OnClickListener() {
                @Override
                public void onClick(DialogInterface dialog, int which) {
                }
            });
            dialog.show();
            break;
        default: break;
    }
}
```

理解：

1. 首先通过`AlertDialog.Builder`的构造方法创建一个`AlertDialog.Builder`实例，需要传递一个context上下文，就用当前activity

2. 设置标题：`dialog.setTitle("xxxx")`

3. 设置内容：`dialog.setMessage("xxxx")`

4. 是否能够通过back键关闭对话框：`dialog.setCancelable()`，传递的参数是true（可以使用back关闭）/false（不可以）

5. 设置确认按钮的内容和对应的逻辑操作：`dialog.setPositiveButton()`

   参数：按钮的内容：string类型；匿名内部类（和button的监视器类似）

   ```java
   dialog.setPositiveButton("OK", new DialogInterface.OnClickListener() {
       @Override
       public void onClick(DialogInterface dialog, int which) {
       }
   });
   ```

6. 设置取消按钮的内容和对应的逻辑操作：`dialog.setNegativeButton()`。具体方法同上

7. 设置警示框是否可见：`dialog.show()`

可以看到该控件位于所有的界面上面，必须先处理该控件，才能继续使用该app。

<img src="pic\image-20210204151952215.png" alt="image-20210204151952215" style="zoom: 50%;" />

## 7. ProgressDialog

和AlertDialog有点类似，都可以在界面上弹出一个对话框，都能够屏蔽掉其他控件的交互能力。

主要不同的是应用场景，**ProgressDialog会在对话框中显示一个进度条**，一般用于表示当前操作比较耗时，让用户耐心地等待

用法和AlertDialog类似：

```java
public void onClick(View v) {
    switch (v.getId()){
        case R.id.button_1:
            ProgressDialog dialog = new ProgressDialog(MainActivity.this);
            dialog.setTitle("WAIT");
            dialog.setMessage("loading...");
            dialog.setCancelable(true);
            dialog.show();
            break;
        default: break;
    }
}
```

理解：

1. 创建ProgressDialog的实例对象：`new ProgressDialog(xxx)`，需要传递一个Context的上下文，就用当前activity即可
2. 设置对话框的开头：同上：`setTitle("xxx")`
3. 设置内容（即进度条旁边的内容，进度条默认是有的）：`setMessage("Xxx")`
4. 设置是否可以用back对话框：`setCancable(xxx)`，true/false，实验发现，想要关闭对话框，可以点页面的其他地方即可，如果配置了可以用back，那么back也可以
5. `show()`显示对话框

注意，该对话框没有按钮，默认有进度条——所以适用场景不同

<img src="pic\image-20210204162231678.png" alt="image-20210204162231678" style="zoom:50%;" />



# 四种布局

界面总是要由很多个控件组成的，需要借助布局来实现各个控件的合理摆放

概念：布局是一种可用于放置很多控件的**容器**，它可以按照一定的规律调整内部控件的位置，从而编写出精美的界面

布局的内部除了放置控件外，也可以放置布局，通过多层布局的嵌套，我们就能够完成一些比较复杂的界面实现

## 1. 线性布局

LinerLayout：线性布局，最常用的布局之一。将它所包含的控件**在线性方向上依次排列**

之前用到的都是`<LinearLayout></LinearLayout>`，并且都是在垂直方向上线性排列。

1. **`android:orientation = "xxxx"`**，选项：`vertical/horizontal`

   主要是配置了：`android:orientation="vertical"`，如果改成`horizontal`，那么就是在水平方向线性排列

   eg：

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
       android:orientation="horizontal"		// vertical
       android:layout_height="match_parent"
       android:layout_width="match_parent">
       <Button
           android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:id="@+id/button_1"
           android:text="button" />
       <Button
           android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:id="@+id/button_2"
           android:text="button_2" />
       <Button
           android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:id="@+id/button_3"
           android:text="button_3" />
   </LinearLayout>
   ```

   <img src="pic\image-20210204181603250.png" alt="image-20210204181603250" style="zoom:50%;" />

   修改成vertical：

   <img src="pic\image-20210204181722008.png" alt="image-20210204181722008" style="zoom:50%;" />

   ps：在实验过程中发现，如果设置为水平方向线性，然后将button的`android:layout_width="match_parent"`，可以发现button都重叠在一起了。——因为单独一个控件就会将整个水平方向占满，其他的控件就没有可放置的位置了

   同样的道理，如果LinearLayout的排列方向是vertical，内部的控件就不能将高度指定为match_parent。

2. `android:layout_gravity="xxx"` 和`android:gravity="xxx"`，选项一样，常用的选项：`center/bottom/top`等

   <img src="pic\image-20210204183652672.png" alt="image-20210204183652672" style="zoom:50%;" />

   需要注意的是：

   - 如果当前排列方向是vertical，那么控件的方向只能修改左右的

     ```xml
     <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
         android:orientation="vertical"
         android:layout_height="match_parent"
         android:layout_width="match_parent">
         <Button
             android:layout_width="wrap_content"
             android:layout_height="wrap_content"
             android:layout_gravity="center_horizontal"	// 水平居中
             android:id="@+id/button_1"
             android:text="button" />
         <Button
             android:layout_width="wrap_content"
             android:layout_height="wrap_content"
             android:id="@+id/button_2"
             android:layout_gravity="left"	// 居左
             android:text="button_2" />
         <Button
             android:layout_width="wrap_content"
             android:layout_height="wrap_content"
             android:id="@+id/button_3"
             android:layout_gravity="right"		// 居右
             android:text="button_3" />
     </LinearLayout>
     ```

     <img src="pic\image-20210204195758587.png" alt="image-20210204195758587" style="zoom:43%;" />

     垂直方向根据前后顺序排列，不可修改；水平方向的位置可以修改，但只能在垂直的范围内修改

   - 如果当前排列方向是horizontal，那么控件的方向只能修改上下的

     ```xml
     <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
         android:orientation="horizontal"
         android:layout_height="match_parent"
         android:layout_width="match_parent">
         <Button
             android:layout_width="wrap_content"
             android:layout_height="wrap_content"
             android:layout_gravity="top"			// 置顶
             android:id="@+id/button_1"
             android:text="button" />
         <Button
             android:layout_width="wrap_content"
             android:layout_height="wrap_content"
             android:id="@+id/button_2"
             android:layout_gravity="center_vertical"		// 垂直居中
             android:text="button_2" />
         <Button
             android:layout_width="wrap_content"
             android:layout_height="wrap_content"
             android:id="@+id/button_3"
             android:layout_gravity="bottom"			// 置底
             android:text="button_3" />
     </LinearLayout>
     ```

     <img src="pic\image-20210204200333712.png" alt="image-20210204200333712" style="zoom:50%;" />

     水平方向根据先后顺序排列；垂直方向可以在限定的水平范围内上下移动

3. 重要属性：**`android:layout_weight="xxx"`**，填入数字即可

   **它允许我们使用比例的方式来指定控件的大小**，它在手机屏幕的适配性方面可以起到非常重要的作用

   ```xml
   <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
       android:orientation="horizontal"			// 水平布局
       android:layout_height="match_parent"
       android:layout_width="match_parent">
       <EditText
           android:layout_width="0dp"			// 宽度设置为0，高度跟随父布局（全部占满）
           android:layout_height="wrap_content"
           android:layout_weight="1"			// 占比为1/2（即水平）	
           android:id="@+id/edit_text"
           android:hint="type something"/>
       <Button
           android:layout_width="0dp"
           android:layout_height="wrap_content"
           android:layout_gravity="top"
           android:layout_weight="1"			// 水平占比为1/2（水平）
           android:id="@+id/button_1"
           android:text="button" />
   </LinearLayout>
   ```

   宽度都设置为0dp，然后使用`android:layout_weight=1`，实验中可以发现，两个控件是平分的。因为weight是按照比例来分配的

   系统会先把LinearLayout下所有控件指定的layout_weight值相加，得到一个总值，然后每个控件所占大小的比例就是用该控件的layout_weight值除以刚才算出的总值。所以总值为2，那么editText占比为1/2，所以就是占比一半

   <img src="pic\image-20210204212223584.png" alt="image-20210204212223584" style="zoom:50%;" />

   这样是对半分。

   修改为`3:1`，那么就是如下的：

   <img src="pic\image-20210204212110581.png" alt="image-20210204212110581" style="zoom:50%;" />

   可以只设定部分的控件的`android:layout_weight`，那么这些控件**会按照比例分配剩下来的空间**

   ```xml
   <EditText
       android:layout_width="0dp"
       android:layout_height="wrap_content"
       android:layout_weight="1"
       android:id="@+id/edit_text"
       android:hint="type something"/>
   <Button
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"
       android:layout_gravity="top"
       android:id="@+id/button_1"
       android:text="button" />
   ```

   理解：只设定了editText按照比例分配；而button是按照内容分配。editText是获得剩下的所有空间

   <img src="pic\image-20210204215215549.png" alt="image-20210204215215549" style="zoom:50%;" />

   ——使用这种方式编写的界面，不仅在各种屏幕的适配方面会非常好，而且看起来也更加舒服

## 2. 相对布局

RelativeLayout：相对布局，是最常用的布局之一。RelativeLayout显得更加随意一些，它可以通过**相对定位的方式**让控件出现在布局的任何位置。但是因此可用的属性多很多。

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_height="match_parent"
    android:layout_width="match_parent">
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/button_1"
        android:layout_alignParentTop="true"
        android:layout_alignParentLeft="true"
        android:text="button_1" />
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentTop="true"
        android:layout_alignParentRight="true"
        android:id="@+id/button_2"
        android:text="button_2" />
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_alignParentLeft="true"
        android:id="@+id/button_3"
        android:text="button_3" />
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_alignParentRight="true"
        android:id="@+id/button_4"
        android:text="button_4" />
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:id="@+id/button_5"
        android:text="button_5" />
</RelativeLayout>
```

理解：关键的是`android:layout_alignParentLeft`、`android:layout_alignParentRight`、`android:layout_alignParentTop`、`android:layout_alignParentBottom`、`android:layout_centerInParent`——这些都是相对父布局的定位方式

也可以相对于某个控件进行布局

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_height="match_parent"
    android:layout_width="match_parent">
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/button_1"
        android:layout_centerInParent="true"
        android:text="button_1" />
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_above="@id/button_1"
        android:layout_toLeftOf="@id/button_1"
        android:id="@+id/button_2"
        android:text="button_2" />
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_above="@id/button_1"
        android:layout_toRightOf="@id/button_1"
        android:id="@+id/button_3"
        android:text="button_3" />
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@id/button_1"
        android:layout_toLeftOf="@id/button_1"
        android:id="@+id/button_4"
        android:text="button_4" />
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@id/button_1"
        android:layout_toRightOf="@id/button_1"
        android:id="@+id/button_5"
        android:text="button_5" />
</RelativeLayout>
```

主要就是4个属性设置的`android:layout_above="xxxx"`，参数是指定的控件——是相对另一个控件的位置，格式`@id/xxx`同样`android:layout_below="xxxx"`，`android:layout_toLeftOf="xxxx"`，`android:layout_toRightOf="xxxx"`，就是用来实现相对其他控件相对位置布局的。

还需要注意：当一个控件去引用另一个控件的id时，该控件一定要定义在引用控件的后面，不然会出现找不到id的情况

<img src="pic\image-20210204222318847.png" alt="image-20210204222318847" style="zoom:50%;" />

## 3. 帧布局

FrameLayout：帧布局，功能少很多，应用场景少很多。

这种布局没有方便的定位方式，所有的控件都会**默认摆放在布局的左上角**——且会堆在一起

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_height="match_parent"
    android:layout_width="match_parent">
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/text_view"
        android:text="tell me something: i love you" />
    <ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/image_view"
        android:src="@mipmap/ic_launcher_round" />
</FrameLayout>
```

会发现所有组件都默认堆在左上角

<img src="pic\image-20210204223133590.png" alt="image-20210204223133590" style="zoom:50%;" />

也可以修改布局，用`android:layout_gravity="xxx"`

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_height="match_parent"
    android:layout_width="match_parent">
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="left"
        android:id="@+id/text_view"
        android:text="tell me something: i love you" />
    <ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="right"
        android:id="@+id/image_view"
        android:src="@mipmap/ic_launcher_round" />
</FrameLayout>
```

<img src="pic\image-20210204223339580.png" alt="image-20210204223339580" style="zoom:50%;" />

FrameLayout由于定位方式的欠缺，导致它的应用场景也比较少，不过在下一章中介绍碎片的时会用到它的。

## 4. 百分比布局

背景：**只有LinearLayout支持使用layout_weight属性来实现按比例指定控件大小的功能**，RelativeLayout和FrameLayout都不支持

为此，Android引入了一种全新的布局方式来解决此问题——**百分比布局——允许直接指定控件在布局中所占的百分比**，这样的话就可以轻松实现平分布局甚至是任意比例分割布局的效果了。

分类：**提供了`PercentFrameLayout`和`PercentRelativeLayout`这两个全新的布局**，根据命名规律可以发现分别是对FrameLayout和RelativeLayout进行了功能扩展（LinearLayout已经能够通过weight来实现相对布局了，不需要扩展了）

提前准备：百分比布局是新布局，所以需要导包——Android团队将百分比布局定义在了support库当中，我们只需要在**项目的build.gradle中添加百分比布局库的依赖**，就能保证百分比布局在Android所有系统版本上的兼容性了。







































# 自定义控件



# ListView详解



# 滚动控件RecyclerView