0. [Java指令](#0)
1. [Java的私有构造函数](#1)
2. [protected关键词的理解](#2)
3. [abstract关键字注意](#3)
4. [Java程序的编译和运行](#4)
5. [异常链](#5)
6. [equals()和hashCode()](#6)
7. [getClass()和.class](#7)
8. [fail-fast](#8)
9. [嵌套接口](#9)
10. [CAS原理分析](#10)
11. [AQS的底层实现](#11)
12. [多线程下的挂起区别](#10)

# 0. Java指令（未完，待补充）

<a name = "0"></a>

因为IDEA太好用了，直接`ctrl+shift+f10`就能编译和运行，而忽略了整个的运行流程。如果没有这种工具我们该如何用命令行运行呢——也有助于了解Java程序的运行原理

## 1. 编译

```shell
javac xxx.java			// 最简单的指令
```

关键字：javac（java compiler），会将Java文件进行编译，编译生成字节流文件：同名.class文件

如果该Java文件中使用了其他文件中的类or使用了jar包需要一起编译

```
java xxx.java xxxx.java		// 关联多少就要添加多少
```



## 2. 运行

```shell
java xxx.class		// 最简单模式
```

关键字：java，会启动JVM来运行

注意：如果编译通过，但是运行时提示：*找不到或无法加载主类*

是因为收到package包的影响



# 4. Java程序的编译和运行（未完待续）

<a name="4"></a>

简化来说分为：编译（编译器执行的） + 运行（JVM执行的）
编译：就是编译器将.java文件编译成.class的字节码
运行：生成的字节码由JVM解释运行
——可以发现：Java既需要编译又需要解释，所以被称为**半解释语言**
以如下的代码举例：
```java
// 构建两个不同的类，存放在同文件夹下
//MainApp.java  
public class MainApp {  // 类1
    public static void main(String[] args) {  
        Animal animal = new Animal("Puppy"); // 调用Animal类 
        animal.printName();  
    }  
}  

//Animal.java  
public class Animal {   // 类2
    public String name;  
    public Animal(String name) {    // 构造方法 
        this.name = name;  
    }  
    public void printName() {  
        System.out.println("Animal ["+name+"]");  
    }  
}  
```
## Java的编译

Java编译一个类的时候，如果该类依赖的类还没有被编译，那么会**先编译该依赖类**，然后引用；如果该依赖类已经编译过了，那么就直接引用（有点像make？？？make是啥用在哪的？？）。如果在指定目录下，找不到该类的依赖类，那么编译器会报错`cannot find symbol`

编译后的字节码文件（表现形式为：字节数组byte[]）主要分为两部分：常量池、方法字节码。

常量池：存放代码出现的**所有token**：类名、成员变量名；**符号引用**：方法引用、成员变量引用

方法字节码：存放的是类中各个方法的字节码

## Java的运行

主要包括：类加载、类执行

Java在程序第一次**主动使用类**的时候，才会去加载该类。就是说，JVM不会在一开始就将程序需要的所有类都加载到内存中（内存大小的限制考虑），而是**需要使用的时候才会加载**，且**只加载一次**

如下图就是程序运行的整个流程：
<img src="https://img-blog.csdn.net/20180309094804673" alt="img" style="zoom:80%;" />

1. 编译之后得到.class文件——MainApp.class，再次在命令行输入：`java MainApp`，系统就会启动一个**JVM进程**，进程会从**classpath路径**中找到对应的`MainApp.class`二进制文件，将`MainApp`的类信息加载到**运行时数据区的方法区**内——MainApp**类的加载**（注意，只加载了MainApp）

2. JVM会找到`MainApp`的`main`函数入口，开始执行（在栈中执行的）

3. 由于main的第一行就是创建一个对象`Animal animal = new Animal("Puppy");` 。而此时，方法区中没有Animal类的信息，此时JVM才会去加载Animal类——`Animal.class`，把该类信息加入到方法区中。

4. 加载完成后，开始执行代码，JVM首先在**堆**中为该Animal类的实例分配内存，然后调用构造函数**初始化Animal实例**，该实例**持有指向方法区的Animal类的信息的引用**——包括方法表和Java动态绑定的底层实现

5. 当使用`animal.printName()`时，JVM通过animal引用找到新建的Animal对象，然后通过Animal对象持有的指向方法区的Animal类信息的引用，找到方法区中Animal类的类信息方法表，从而获得`printName()`字节码的地址

6. 开始运行`printName()`方法 

ps：java类中所有public和protected的实例方法都采用动态绑定机制，所有私有方法、静态方法、构造器及初始化方法<clinit>都是采用静态绑定机制。而使用**动态绑定机制的时候会用到方法表，静态绑定时并不会用到**。



# 7. getClass()

前提1：可以发现**数组也有自己class对象， [ 表示维度， Lxxx表示数组的元素类型**

```java
String[] strings = new String[5];
Objects[] objects = new Objects[5];
System.out.println(strings.getClass());	// >> class [Ljava.lang.String;
System.out.println(objects.getClass());	// >> class [Ljava.util.Objects;
System.out.println(String.class);	// >> class java.lang.String
```

前提2：子类重写父类方法的时候，在不修改返回值类型的前提下，子类返回了什么类型，具体得到的是子类的返回值类型，而不会上转成父类的返回值类型

```Java
public class Father {
    public Object fun(){
        return new Object();
    }
}

class Son extends Father{
    @Override
    public Object fun(){
        return 1;
    }
}

new Father().fun().getClass();	// 返回的是返回值的类型 >> class java.lang.Object
new Son().fun().getClass();		// 因为返回值是1，直接将其打包变成Integer类的对象 >> class java.lang.Integer
```

——返回值的类型就是其原本的类型，而不会强转成其父类类型

找问题所在：在ArrayList源码中的第三个构造函数存在一点问题：

```java
transient Object[] elementData;		// 实例变量

public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();		// 将该包装类转换成Object[]数组
    if ((size = elementData.length) != 0) {		
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)	// 判断elementData的类型和Object[]类型是否一致——不一致才进行复制
            elementData = Arrays.copyOf(elementData, size, Object[].class);	// 将不是Object[]的数组内容拷贝到Object[]数组中
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}

Object[] toArray();		// 是包装类Collection的接口定义的方法——实现该需要重写该方法的

public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
    @SuppressWarnings("unchecked")
    T[] copy = ((Object)newType == (Object)Object[].class)		// 新建一个T[]类型的数组
        ? (T[]) new Object[newLength]
        : (T[]) Array.newInstance(newType.getComponentType(), newLength);
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));	// 将U[]数组拷贝到了T[]数组中
    return copy;
}
```

分析：`elementData.getClass() != Object[].class`为啥要进行如此判断呢？

（注释中有提示说`c.toArray`的返回值可能不是`Object[]`，那么问题来了，*为啥默认返回值都是Object[]？何时不会是Object[]，而是其他例外呢？*）

1. 首先elementData是包装类对象参数c去调用自己的`toArray`方法，希望得到的是Object[]返回值

   然后追溯`toArray`方法，发现是在`Collection`接口中定义的一个方法，那么实现该接口的类必须要重写该方法，且返回值要求是`Object[]`类型——所以，可以发现只需要实现了`Collection`接口的类都一定有一个`toArray`方法

   eg：看一下ArrayList的实现：

   ```Java
   public Object[] toArray() {
       return Arrays.copyOf(elementData, size);
   }
   public static <T> T[] copyOf(T[] original, int newLength) {
       return (T[]) copyOf(original, newLength, original.getClass());	// ——返回值就是Object[]，因为传递的original就是Object[]类型
   }
   
   ArrayList<String> arrayList = new ArrayList<String>();
   System.out.println(arrayList.toArray().getClass());	// >> class [Ljava.lang.Object; 可以发现参数就是Obejct[]
   ```

   ——可以发现，大部分toArray执行之后，得到的返回值都是Object[]，符合要求

   ——那么肯定有不满足要求的，所以才需要进行判断和额外操作

2. 符合`elementData.getClass() != Object[].class`的情况

   网上有人追溯了这个bug的位置：**Arrays.asList(x).toArray().getClass() should be Object[].class**

   ```java
   public static <T> List<T> asList(T... a) {
       return new ArrayList<>(a);	// 是Arrays的一个内部类——得到了ArrayList类的对象，但是里面的数组是E[]类型的
   }
   
   // 继承自AbstractList，而又是继承自AbstractCollection，而又是继承自Collection，所以必然实现了toArray方法
   private static class ArrayList<E> extends AbstractList<E>
           implements RandomAccess, java.io.Serializable
   {
       private static final long serialVersionUID = -2764017481108945198L;
       private final E[] a;		// 泛型
   
       ArrayList(E[] array) {		// 直接赋值，保持array的原始类型
           a = Objects.requireNonNull(array);
       }
       ....
       @Override
       public Object[] toArray() {
           return a.clone();		// 直接拷贝，那么里面的任何数据没有变化
       }
   }
   ```

   ——那么通过`Arrays.asList(x)`得到的ArrayList类的对象，但是里面的数组是E[]类型的

   ——接下来如果调用`Arrays.asList(x).toArray()`方法，那么里面的数组还是E[]类型

   ```Java
   String[] strings = new String[]{"hello", "world"};
   System.out.println(strings.getClass());		// >> class [Ljava.lang.String;
   
   List<String> temp = Arrays.asList(strings);	// E = String
   System.out.println(strings.getClass());		// >> class java.util.Arrays$ArrayList
   
   Object[] objects = temp.toArray();		
   System.out.println(objects.getClass());	// >> class [Ljava.lang.String;——和最开始的数据类型一致
   ```

   ——而不是符合要求的Object[]

3. 那么如果不进行判断，放任不同的数组类型存在呢？

   ```Java
   List<Object> list = new ArrayList<Object>(Arrays.asList(new String[]{"hello", "world"}));	
   // 如果不在构造方法中判断并操作，那么l中elementData的数据类型就是String[]
   l.set(0, new Object());	// 会抛出异常，不允许将Object对象存放在String[]中
   
   
   ArrayList<String> arr = new ArrayList<>();
   arr.add("hello");		// 编译通过
   arr.add(new Object());	// 编译错误——因为ArrayList泛型类限定了E=String，这个是在传参的时候就已经确定了的
   // ——这样操作也不符合逻辑需要，String改成Object就又可以了
   ```

   ——而现在操作了之后，l里面的数组就是Object[]类型，那么允许add（而上面的情况是符合逻辑需要的，将String的列表存储到Obejct列表中，而Object列表还能添加其他类型的数据）

总结来说为啥需要这样的判断才行呢，通常来说，在调用该构造方法的时候，需要将该对象通过`toArray()`转换成Object[]数组类型，正常的实现Collection接口的对象都能正确转换成Object[]数组。但是存在一个例外：通过`Arrays.asList`也能创建得到一个`ArrayList`对象，但是和常见不大一样（是Arrays内部实现的私有类），区别在于`elementData`是泛型数组，类型参数与传入的参对应，而普通的ArrayList的`elementData`是Object[]数组；而在该ArrayList对象去调用真正的ArrayList的构造函数时，同样触发了自己的`toArray()`，该方法还是返回的泛型数组，那么`elementData`实际上得到的是一个泛型数组，而不是常见的Object[]数组。

那么存在的问题是，如果我想将一个Object对象存储到该新的ArrayList中时，如果不进行处理会抛出类型不匹配异常

——所以需要强行将数组类型转换成统一的Object[]

——主要还是泛型类型参数引发的问题，主要处于可靠性考虑

ps：ArrayList有2个`toArray()`方法

```java
public Object[] toArray() {
    return Arrays.copyOf(elementData, size);	// 返回的是Object[]的数组——且不能强转为其他类型[]
}

public <T> T[] toArray(T[] a) {		// 返回的是泛型数组，即传入的是什么类型的数组，得到的就是什么类型的数组
    if (a.length < size)
        // Make a new array of a's runtime type, but my contents:
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}

String[] s = arr.toArray(new String[0]);	// 返回的就是传入的String[]类型	
```

pps：why在ArrayList中是用Obejct[] elementData，而不能是E[] elementData

本质上，不允许定义泛型数组，即`E[] data = new E[](5);`——会报错的；但是能够引用

即只能通过`E[] data = (E[])new Object[....];`——强制类型转换

但是又会触发另一个问题，如果此时E=String或者其他，那么该强制类型转换就会出错

如果想要解决这个问题，就要创建每一个可能调用该类的类型数组（不可能做到）

——所以不能如此

=> 所以就选择声明为`Object[]`统一存储在超类Object对象数组中——然后通过传参直接限定类型

参考内容：

https://blog.csdn.net/weixin_39452731/article/details/100189934

https://juejin.cn/post/6844903624062009357

# 9. 嵌套接口

<a name="9"></a>

类似于内部类的，现在将定义再确认一下：

规定：

- 接口可以嵌套在类/其他接口中，即类中含有一个接口（内部接口），接口中再包含一个接口

- 类中嵌套接口时，嵌套接口可以是常规的4种访问形式

  ——被定义为private的嵌套接口只能在所在的类被实现，可以被实现为public的类也可以被实现为private的类

- 接口中嵌套接口时，**该嵌套接口必须是public的类型**（与接口都是public类型决定的）

  ——在实现外部接口时，不必实现该嵌套接口（可以单独创建一个类去实现该嵌套接口，形如：`class TestClass implements Test.innerInterface`）

  ——能够提供给外部接口的实现类创建内部类使用

  <img src="map.png" style="zoom:80%;" >

  

why使用嵌套接口：

- 嵌套接口只会在该接口中使用
- 有利于封装
- 有更好的可读性和维护性

在 Java 类库中一个典型的嵌套接口的例子是 `java.util.Map` 以及 `Java.util.Map.Entry`——就是我学习嵌套接口的来源。

**嵌套接口默认强制是** `static`——所以嵌套接口都是静态嵌套接口

why 强制是static类型的？

static 关键字用于方法、域与作用于接口和类有着不同的含义。**当 static 作用于内部类时，用于强调内部类的实现细节相对于外部类独立**，比如说想要创建嵌套类对象并不需要外部类的对象。

**而嵌套接口也是类似，接口本身就是抽象的集合，不会依赖于外部接口也不与特定类相关，所以只要类实现了该接口的方法，那么就是实现了接口**，所以为了表达这个思想，就默认嵌套接口为static类型

所以，**可以认为嵌套接口和外部接口区别并不大，嵌套接口主要提供了一层内外的逻辑关系**：内作为外的一个功能组成，且并不希望直接暴露给外部。但是这种封装是不彻底的，因为嵌套接口默认且只能使用 public 修饰。

参考内容

https://cloud.tencent.com/developer/article/1585264





# 12. 多线程下的挂起区分

Thread.sleep()、Object.wait()、Condition.await()、LockSupport.park()

### Thread.sleep()和Object.wait()的区别：

- Thread.sleep()不会释放占有的锁，Object.wait()会释放占有的锁；

- Thread.sleep()必须传入时间，Object.wait()可传可不传，不传表示一直阻塞下去；

- Thread.sleep()到时间了会自动唤醒，然后继续执行；

- Object.wait()不带时间的，需要另一个线程使用Object.notify()唤醒；Object.wait()带时间的，假如没有被notify，到时间了会自动唤醒，因为前面释放了锁，所以如果有锁，那么需要去尝试获取锁：如果立即获取到了锁，线程自然会继续执行；二是没有立即获取锁，线程进入同步队列等待获取锁；

  ——这时被挂在了锁的等待队列上；而前面wait是挂在了条件队列上（wait不到条件时，就被挂起在条件队列上）

——**它们的最大的区别就是Thread.sleep()不会释放锁资源，Object.wait()会释放锁资源。**

### Thread.sleep()和Condition.await()的区别

Object.wait()和Condition.await()的原理比较类似

回答思路跟Object.wait()是基本一致的，不同的是Condition.await()底层是调用LockSupport.park()来实现阻塞当前线程的

并且，Condition.await()在阻塞当前线程之前还干了两件事，一是把当前线程添加到条件队列中，二是“完全”释放锁，也就是让state状态变量变为0，然后才是调用LockSupport.park()阻塞当前线程

### Thread.sleep()和LockSupport.park()的区别

- 从功能上来说，Thread.sleep()和LockSupport.park()方法类似，都是阻塞当前线程的执行，且**都不会释放当前线程占有的锁资源**；
- Thread.sleep()没法从外部唤醒，只能自己醒过来；
- LockSupport.park()方法可以被另一个线程调用LockSupport.unpark()方法唤醒；
- Thread.sleep()方法声明上抛出了InterruptedException中断异常，所以调用者需要捕获这个异常或者再抛出；
- LockSupport.park()方法不需要捕获中断异常；
- Thread.sleep()本身就是一个native方法；
- LockSupport.park()底层是调用的Unsafe的native方法；

### Object.wait()和LockSupport.park()的区别

- Object.wait()方法会释放锁；LockSupport.park()不会释放锁
- Object.wait()方法需要在synchronized块中执行；LockSupport.park()可以在任意地方执行；
- Object.wait()方法声明抛出了中断异常，调用者需要捕获或者再抛出；
- LockSupport.park()不需要捕获中断异常;
- Object.wait()不带超时的，需要另一个线程执行notify()来唤醒，但不一定继续执行后续内容；
- LockSupport.park()不带超时的，需要另一个线程执行unpark()来唤醒，一定会继续执行后续内容；
- **如果在wait()之前执行了notify()会怎样？抛出IllegalMonitorStateException异常**；
- **如果在park()之前执行了unpark()会怎样？线程不会被阻塞，直接跳过park()，继续执行后续内容；**

参考内容：

https://segmentfault.com/a/1190000020864747?utm_source=sf-related



