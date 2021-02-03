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









































# 四种布局



# 自定义控件



# ListView详解



# 滚动控件RecyclerView