# 一、概述

## （一）Android特点概览

### 1. android的四层架构

四层架构：==Linux内核层、系统运行库层、应用框架层、应用层==

1. Linux内核层——最底层

   为Android设备的各种硬件提供驱动，包括显示驱动、照相机驱动等（就是去调用设备的一些硬件来干活——这些都是通过Linux内核层实现的）

   那么，Android实际上是搭建在Linux操作系统上的应用？

2. 系统运行库层

   库主要是：c/c++（对于Java来说就是native）

   SQLite库提供了数据库的支持；OpenGL|ES库提供了3D绘图的支持；Webkit库提供了浏览器内核的支持等。

   还有Android运行时库，主要是一些核心库，能够允许开发者用Java来编写Android应用；还包含了虚拟机（Dalvik虚拟机，5.0系统之后变成了ART运行环境），它能够让每个Android应用都能运行在独立的进程当中（1个进程1个应用），并且拥有一个自己的虚拟机实例。

3. 应用框架层

   主要就是提供构建应用程序需要用到的各种API

4. 应用层：应用程序

ps：Android 5.0系统后，其中使用**ART运行环境**替代了Dalvik虚拟机，大大提升了应用的运行速度，还提出了**Material Design**的概念来优化应用的界面设计——Material Design用在页面设计的

<img src="pic/android_framework.jpg" style="zoom:67%;" >

### 2. Android应用开发的特色

#### （1）四大组件

- **活动：activity**，应用程序的门面，就是可见的内容都是存放在activity中的
- **服务**：后台运行
- **广播接收器**：应用能够接收来自各处的广播消息，eg：电话、短信等
- **内容提供器**：使得应用程序之间能够共享数据，eg：读取电话簿中的联系人信息

#### （2）丰富的系统控件

主要是跟界面有关，也可以自定义控件

#### （3）SQLite数据库

Android自带的轻量级的、运算速度极快的嵌入式**关系型数据库**。

支持标准的SQL语法，还能通过Android封装好的API进行操作

#### （4）强大的多媒体

系统本身提供了多媒体调用的服务，直接使用即可

#### （5）地理位置定位

GPS

### 3. Android开发需要的工具

- JDK：Java开发包
- Android SDK：谷歌提供的Android的开发工具包，通过导入该包使用Android相关的API
- Android Studio

### 4. 项目内容熟悉

Android Studio会默认使用Android模式的项目结构，并不是项目真实的目录结构，而是被Android Studio转换过的。——主要是针对快速开发使用的

<img src="pic\image-20200811212908388.png" alt="image-20200811212908388"  />

稍微了解一下：在app下面有3个文件夹：

- `manifest`：是app清单；
- `java`：是java源代码文件；
- `res`：是资源文件

res文件下有4类资源：

- `drawable`：图片；
- `layout`：布局文件，一个layout就是一个页面，默认有一个`activity_main.xml`文件；
- `mipmap`：图标文件夹；
- `value`：常量文件，包含有`color.xml`、`strings.xml`：所有字符串常量放在这里、`styles.xml`：样式

![image-20200811213149256](pic\image-20200811213149256.png)

言归正传：

——将从Android模式切换回项目结构模式：这就是项目真实的目录结构：

<img src="pic\image-20210128141638589.png" alt="image-20210128141638589" style="zoom: 50%;" />

分析：

1. `.gradle`和`.idea`：放置的都是Android Studio自动生成的一些文件，我们无须关心，也不要去手动编辑。（不需修改）

2. **`app`：项目中的代码、资源等内容几乎都是放置在这个目录下的，我们后面的开发工作也基本都是在这个目录下进行的（修改的主要对象）**

   <img src="pic\image-20210128141728855.png" alt="image-20210128141728855" style="zoom:50%;" />

   - `build`：包含了一些在编译时自动生成的文件（不关注）

   - `libs`：如果项目中使用到了**第三方jar包，就需要把这些jar包都放在libs目录下**，放在这个目录下的jar包都会被自动添加到构建路径里去

   - `src`：源代码都放在这边

     - `androidTest`：用来编写Android Test测试用例

     - `main`：源代码主要存放位置

       <img src="pic\image-20210128141812510.png" alt="image-20210128141812510" style="zoom:50%;" />

       - **`java`：放置我们所有Java代码的地方（最核心的地方）**

       - **`res`：存放项目中使用到的所有图片、布局、字符串等资源。存在很多子目录**

         **eg：`drawable`（存放图片）、`layout`（存放布局文件）、`values`（存放字符串、样式、颜色等配置值）**

       - **`AndroidManifest.xml`**：整个项目的配置文件，定义的所有四大组件都需要在这个文件里注册，还可以在该文件中给应用程序添加权限声明

     - `test`：来编写Unit Test测试用例的，单元测试脚本

   - `.gitignore`：将**app模块**内的指定的目录或文件排除在版本控制之外，作用和外层的.gitignore文件类似

   - `build.gradle`：app模块的gradle构建脚本，这个文件中会指定很多项目构建相关的配置（而外部的build.gradle是一个全局构建脚本）

   - `proguard-rules.pro`：用于指定项目代码的混淆规则，当代码开发完成后打成安装包文件，如果不希望代码被别人破解，通常会将代码进行混淆，从而让破解者难以阅读——源码保护

3. `gradle`：gradle wrapper的配置文件。使用gradle wrapper的方式不需要提前将gradle下载好，而是会自动根据本地的缓存情况决定是否需要联网下载gradle

   Android Studio默认没有启用gradle wrapper的方式

4. `.gitignore`：是用来将指定的目录或文件排除在版本控制之外的（涉及到版本控制的问题）——整个项目范围的

5. `build.gradle`：项目全局的gradle构建脚本（不需修改）

6. `gradle.properties`：全局的gradle配置文件，在这里配置的属性将会影响到项目中所有的gradle编译脚本。

7. `gradlew`和`gradlew.bat`：用来在命令行界面中执行gradle命令的，其中gradlew是在Linux或Mac系统中使用的，gradlew.bat是在Windows系统中使用的。（不需修改）

8. `HelloWorld.iml`：.iml文件是所有IntelliJ IDEA项目都会自动生成的一个文件（Android Studio是基于IntelliJ IDEA开发的），用于标识这是一个IntelliJ IDEA项目（不需修改）

9. `local.properties`：指定本机中的Android SDK路径，通常内容都是自动生成。除非SDK位置发生变化，那么需要修改。（不需修改）

10. `settings.gradle`：指定项目中所有引入的模块，通常情况下模块的引入都是自动完成的（不需修改）.

    当前状态下，只有一个app模块，所以只引入了这一个模块

    <img src="pic\image-20201220202318071.png" alt="image-20201220202318071"  />

### 5. 代码初步分析

#### （1）`AndroidManifest.xml`

先来看整个项目的配置文件：`AndroidManifest.xml`：

这边关注的是：一个`.MainActivity`的活动在该文件中注册。**没有在该文件中注册的活动是不能使用的**

且声明了`MainActivity`是**整个项目的主活动**——APP最先启动的时候就是该活动

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.myapplication">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"	// 图标引用
        android:label="@string/app_name"	// 通过引用string里面的字符串app_name里面的内容
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <!---------------------activity的注册------------------------>
        <activity android:name=".MainActivity">
            <intent-filter>
                <!--下面两句话是表明，MainActivity是整个项目的主活动——APP最先启动的时候就是该活动-->
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <!----------------------------------------------------------->
    </application>

</manifest>
```

#### （2）`MainActivity.java`和`activity.xml`、res文件夹

那么看一下`MainActivity`实现了什么？\

在src/main/java/com.example.myapplication的包下面

```Java
public class MainActivity extends AppCompatActivity {	// 继承AppCompatActivity父类

    @Override
    protected void onCreate(Bundle savedInstanceState) {	// 提供了一个方法
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);	// 给当前活动引入一个activity_main布局
    }
}
```

分析：

1. 该`MainActivity`对应的就是一个活动，而该活动对应一个类，该类继承了`AppCompatActivity`类，追溯该类，发现：`MainActivity` -> `AppCompatActivity` -> `FragmentActivity` -> `ComponentActivity` -> `androidx.core.app.ComponentActivity` -> `Activity`

   `AppCompatActivity` ：是向下兼容的Activity，可以将Activity在各个系统版本中增加的特性和功能**最低兼容到Android 2.1系统**

   而最底层的**`Activity`：是Android提供的一个活动基类，所有活动必须要继承它或者它的子类才能拥有活动特性**。

2. `onCreate()`方法，是一个活动被创建后必须要执行的方法（即一个活动一定包含的方法）

3. 需要了解：Android程序的设计是讲究**逻辑和视图分离的**，即在**布局文件（res/layout）中写界面，然后将其引入到活动**中，使用`setContentView()`方法

4. 所以，布局文件统一存放在`res/layout`文件中，`activity.xml`文件就放在这里面。

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
       xmlns:app="http://schemas.android.com/apk/res-auto"
       xmlns:tools="http://schemas.android.com/tools"
       android:layout_width="match_parent"
       android:layout_height="match_parent"
       tools:context=".MainActivity">
   	
       <!--可以发现文本内容写在这个里面-->
       <TextView
           android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="Hello World!"			
           app:layout_constraintBottom_toBottomOf="parent"
           app:layout_constraintLeft_toLeftOf="parent"
           app:layout_constraintRight_toRightOf="parent"
           app:layout_constraintTop_toTopOf="parent" />
   
   </androidx.constraintlayout.widget.ConstraintLayout>
   ```

5. 再来详细了解一下res文件夹（resources）

   <img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20201220213755123.png" alt="image-20201220213755123"  />

   **`drawable`：存放图片、`layout`：存放布局文件、`values`：存放字符串、样式、颜色等配置值、mipmap：存放应用图标，出现很多个文件夹，主要是用来更好的兼容各种设备**

   ——实际上我们需要和mipmap一样，为了兼容不同的设备，在drawable文件夹里面也要创建多个文件，来适应不同的设备

   在制作图片的时候，应该要给同一个图片提供几个不同分辨率的版本，放在对应的文件夹下。在程序运行时，会自动根据当前运行设备的分辨率**自动选择**加载的文件夹图片

   那如何使用res里面的资源呢

   `res/values/strings.xml`

   ```xml
   <resources>
       <string name="app_name">Hello World</string>
   </resources>
   ```

   在这边定义：

   然后可以在

   - 代码中通过：`R.string.app_name`引用该字符串的引用；
   - 在XML中通过`@string/app_name`获得该字符串的引用

   string可以替换为其他文件名，eg：替换成drawable、mipmap、layout等，那么就可以引用其他文件夹里面的内容了

#### （3）2个`build.gradle`文件

Android Studio是采用Gradle来构建项目的。

Gradle是一个非常先进的项目构建工具，它使用了一种基于Groovy的领域特定语言（DSL）来声明项目设置，摒弃了传统基于XML（如Ant和Maven）的各种烦琐配置。

最外层的`build.gradle`文件

```groovy
buildscript {
    repositories {
        google()
        jcenter()		// 代码托管仓库
    }
    dependencies {
        classpath "com.android.tools.build:gradle:4.0.1" // 用classpath声明了一个Gradle插件
    }
}

allprojects {
    repositories {
        google()
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

分析：

1. `jcenter()`：代码托管仓库，很多Android开源项目都会选择将代码托管到jcenter上，声明了这行配置之后，就可以在项目中轻松**引用任何jcenter上的开源项目了**
2. 声明插件的主要原因是：Gradle并不是专门为构建Android项目而开发的，Java、C++等很多种项目都可以使用Gradle来构建。如果我们要想使用它来构建Android项目，则需要声明`com.android.tools.build:gradle:4.0.1`这个插件，当前版本号是4.0.1

——一般情况，不需要修改该文件中的内容，除非想添加一些全局的项目构建配置

app下的`build.gradle`

```groovy
apply plugin: 'com.android.application'	// 应用了一个插件，可选择application（应用程序模块）或者library（库模块）

android {
    compileSdkVersion 30	// 指定项目的编译版本
    buildToolsVersion "30.0.1"	// 指定项目构建工具的版本

    defaultConfig {			// 项目的细节配置
        applicationId "com.example.myapplication"		// 指定项目的包名
        minSdkVersion 19	// 最低兼容的Android系统版本
        targetSdkVersion 30	// 目标版本，表明已经在该版本上做了充分的测试，可以启用该版本相对于之前版本的新功能和特性（如果版本降低，那么新的功能就不会启动）
        versionCode 1		// 项目版本号
        versionName "1.0"	// 项目版本名

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {		// 生成安装文件的相关配置
        release {		// 生成正式版安装文件的配置（与之相对的是debug，测试版安装文件的配置，它可以忽略不写）
            minifyEnabled false		// 指定是否对项目的代码进行混淆
            // 指定混淆时使用的规则文件：分别是所有项目通用的混淆规则文件和自定义的当前项目特有的混淆文件
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {		// 指定当前项目的所有依赖关系：一般是本地依赖、库依赖、远程依赖
    implementation fileTree(dir: "libs", include: ["*.jar"])	// 本地依赖，lib下面的所有jar包都添加到项目的构建中——对应了lib文件夹的作用：将用到的第三方jar全部放在该文件才能被构建和使用
    implementation 'androidx.appcompat:appcompat:1.2.0'	// 远程依赖
    implementation 'androidx.constraintlayout:constraintlayout:1.1.3'
    testImplementation 'junit:junit:4.12'		// 这些都是测试用到的
    androidTestImplementation 'androidx.test.ext:junit:1.1.1'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.2.0'

}
```

分析：

1. 第一行应用了一个插件，该插件的选项是`com.android.application`（应用程序模块）/`com.android.library`（库模块），两者的区别在于：前者可以直接运行；后者只能作为代码库依附于别的应用程序模块来运行

2. Android闭包主要是：配置项目构建的各种属性

   - 指定项目的编译版本
   - 指定项目构建工具的版本
   - defaultConfig闭包是：项目的细节配置
   - buildTypes闭包是：生成安装文件的相关配置，一般默认是给生成正式版安装文件的配置，即release

3. dependencies闭包：指定当前项目的所有依赖关系

   有三类依赖：

   - 本地依赖：可对本地的Jar包或目录添加依赖关系
   - 库依赖：对项目中的库模块添加依赖关系
   - 远程依赖：对jcenter库上的开源项目添加依赖关系

4. 分析`androidx.constraintlayout:constraintlayout:1.1.3`

   前面是域名部分`androidx.constraintlayout`，用来和其他公司做区分；

   `constraintlayout`：组名部分，用来和公司中的其他库做区分

   `1.1.3`：版本号，用来和同一库的其他版本做区分

   Gradle在构建项目时会首先检查一下本地是否已经有这个库的缓存，如果没有的话则会去自动联网下载，然后再添加到项目的构建路径当中。

#### ps: 加快开发工具：Log

日志工具类是Log（android.util.Log）

提供了如下5个方法来供我们打印日志：

- Log.v()：用于打印那些最为琐碎的、意义最小的日志信息。对应级别verbose，是Android日志里面级别最低的一种。
- Log.d()：用于打印一些调试信息，对应级别debug，比verbose高一级
- Log.i()：打印一些比较重要的数据，主要是用户行为数据，对应级别info，比debug高一级
- Log.w()：打印一些警告信息，提示程序在这个地方可能会有潜在的风险，最好去修复一下这些出现警告的地方。对应级别warn，比info高一级
- Log.e()：打印程序中的错误信息，比如程序进入到了catch语句当中。程序有错误，必须尽快修复。对应级别error，比warn高一级。

eg：

```Java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Log.d("MainActivity", "test log fun");		// 添加一句日志打印（tag, msg）
    }
}

//在logcat中查看： >> 2020-12-20 22:48:23.526 8970-8970/com.example.myapplication D/MainActivity: test log fun
```

分析：发现打印出了：打印时间、进程号、包名、tag（标签）、具体内容

与日志相对的，**极其不推荐用println输出**，除了方便，存在很多问题：日志打印不可控制、打印时间无法确定、不能添加过滤器、日志没有级别区分等

log+logcat很方便：logcat中还能很轻松地添加过滤器，现在默认有3个：

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20201220225826033.png" alt="image-20201220225826033"  />

- Show only selectedapplication：表示只显示当前选中程序的日志

- Firebase是谷歌提供的一个分析工具

- No Filters相当于没有过滤器，会把所有的日志都显示出来

可以自定义过滤器：
可以对tag进行过滤，也可以对message进行过滤，包名进行过滤等

并且logcat提供了日志级别的选择，如果使用debug，那么debug及以上级别方法打印的日志才会显示出来

还可以选择关键词进行进一步的过滤，并且该关键词是支持正则表达式的，那么能快速定位到你想要的那条日志

ps：可以使用快捷键，输入logd，然后按下Tab键，就会帮你自动补全一条完整的打印语句（其他级别都可以类似这样），eg：`Log.d(TAG, "onCreate: ");`

而在该方法外面，使用快捷键：logt，会自动生成常量`private static final String TAG = "MainActivity";`

