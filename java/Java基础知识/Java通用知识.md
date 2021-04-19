# Java通用知识

数据在计算机中实际上都是**二进制形式表示的**，但是人难以阅读编程。于是，**高级语言引入了数据类型、变量的概念**——用来区分不同的数据，通过编译器来将数据转换成对应的二进制表示

程序的流程控制有两种：1. 条件执行；2. 循环

## （一）数的二进制表示

### 1. 整数的二进制表示

正整数：通过原码进行表示，且正整数的**原码 = 补码**

负整数：通过补码进行表示：流程就是：只取其整数部分，忽略前面的负号，然后**将其原码表示出来，然后全部取反，+1**

eg： -1 => 0000 0001 => 1111 1110 => 1111 1111; -127 => 0111 1111 => 1000 0000 => 1000 0001

所以，可以直接通过最高位1/0，判断出该数是正数还是负数

那么。如果一个数是有符号的，那么其表示的数的绝对值会变小，即：8位二进制，如果是表示无符号数，那么能表示0~255；如果是有符号数，那么能表示-127~0~128（因为**最高位是符号位**）

用补码表示的原因：**计算机中减法本质上就是加法，+相反数**，eg：5 - 3=> 5 + (-3)

<img src="C:/Users/surface/Desktop/计算机知识整理/java/pic/image-20201108194515909.png" alt="image-20201108194515909"  />

所以能理解加法上溢会变成负数：

<img src="C:/Users/surface/Desktop/计算机知识整理/java/pic/image-20201108194619414.png" alt="image-20201108194619414"  />

### 2. 小数的二进制表示

因为计算机是按照二进制计算的，所以在小数方面有天然缺陷，即使在一些非常基本的小数运算中，计算的结果也是不精确的。

eg：

```java
float x1 = 0.1f * 0.1f;
>> x1 = 0.010000001
```

二进制格式不能精确表示0.1，它只能表示一个非常接近0.1但又不等于0.1的一个数

对于小数：2的某次方之和（某次方，是指负数）的数可以精确表示，其他数则不能精确表示

计算不精确的方法：

- 如果不需要那么高的精度，可以四舍五入，或者在输出的时候只保留固定个数的小数位；
- 如果需要，将小数转化为整数进行运算，运算结束后再转化为小数；或者转换成十进制再计算，比较复杂

二进制中表示小数，用的是科学计数法：`m× (2^e)`（m称为尾数，e称为指数），其实，还有一个符号位表示正负

几乎所有的硬件和编程语言表示小数的二进制格式都是一样的，有2种格式：

- 32位的：1位符号 + 23位尾数 + 8位指数
- 64位的：1位符号 + 52位尾数 + 11位指数

除了表示正常的数，标准还规定了一些特殊的二进制形式表示一些特殊的值，比如**负无穷、正无穷、0、NaN**（非数值，比如0乘以无穷大）

## 3. 字符编码

### 1. 非Unicode编码

有很多非Unicode编码，常见的有：**ASCII**、ISO 8859-1、Windows-1252、**GB2312**、**GBK**、GB18030和Big5

西欧国家中流行的是ISO 8859-1和Windows-1252；中国是GB2312、GBK、GB18030和Big5

- ASCII：美国信息互换标准代码，7位，128个字符，在一个字节中，统一将最高位设置为0
- ISO 8859-1、Windows-1252：128~255，即最高位为1，和ASCII兼容，当最高位为0时，代表使用ASCII；当最高位为1时，代表使用自己的码
- GB2312：包括约7000个汉字和一些罕用词和繁体字，**两个字节表示汉字**，同时和ASCII兼容，方法同上，最高位设置为0.
- GBK：建立在GB2312的基础上，向下兼容GB2312，约21 000个汉字，其中包括繁体字。**两个字节**

——其他编码都是兼容ASCII的，最高位使用0/1来进行区分

### 2. Unicode编码

Unicode将所有编码进行统一，给世界上所有字符都分配了一个唯一的数字编号

**Unicode本身只规定了每个字符的数字编号是多少**，但是没有指定对应的二进制表示

有多种方案，主要有UTF-32、UTF-16和UTF-8。

- UTF-32：统一用4字节，浪费空间
- UTF-16：用变长字节表示，至少2个
- UTF-8：使用的字节个数为1～4不等。

只有UTF-8兼容ASCII

### 3. 编码转换

编码转换的具体过程可以是：一个字符从A编码转到B编码，

- 先找到字符的A编码格式，
- 通过A的映射表找到其Unicode编号，
- 然后通过Unicode编号再查B的映射表，找到字符的B编码格式。

### 4. 乱码

1. 解析错误——错误识别

   解析数据的方式错了，换成正确的解析方式即可，即本来是Windows-1252类型的，看成了GB2312，那么就将其当成GB2312的二进制来解析，就错误了。

2. 先解析错误，后进行编码转换了，此时就回不去了——错误转换

   即本来是Windows-1252类型的，看成了GB2312，并且还将其转换成了UTF-8形式，然后转换成GB18030，然后就是乱码了

恢复的方法：尝试进行逆向操作，将原始的数据正确看成其对应的类型，然后点击编码转换，指定正确的转换方向

# ps：补充的知识

## 1. Java指令（未完，待补充）

因为IDEA太好用了，直接`ctrl+shift+f10`就能编译和运行，而忽略了整个的运行流程。如果没有这种工具我们该如何用命令行运行呢——也有助于了解Java程序的运行原理

### 1.1 编译

```shell
javac xxx.java			// 最简单的指令
```

关键字：javac（java compiler），会将Java文件进行编译，编译生成字节流文件：同名.class文件

如果该Java文件中使用了其他文件中的类or使用了jar包需要一起编译

```
javac xxx.java xxxx.java		// 关联多少就要添加多少
```

### 1.2 运行

```shell
java xxx.class		// 最简单模式
```

关键字：java，会启动JVM来运行

注意：如果编译通过，但是运行时提示：*找不到或无法加载主类*

是因为收到package包的影响

## 2. Java程序的编译和运行（未完待续）

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

### Java的编译

Java编译一个类的时候，如果该类依赖的类还没有被编译，那么会**先编译该依赖类**，然后引用；如果该依赖类已经编译过了，那么就直接引用（有点像make？？？make是啥用在哪的？？）。如果在指定目录下，找不到该类的依赖类，那么编译器会报错`cannot find symbol`

编译后的字节码文件（表现形式为：字节数组byte[]）主要分为两部分：常量池、方法字节码。

常量池：存放代码出现的**所有token**：类名、成员变量名；**符号引用**：方法引用、成员变量引用

方法字节码：存放的是类中各个方法的字节码

### Java的运行

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

## 3. getClass()

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

