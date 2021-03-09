# Java面向对象PART5

# ——常用基础类

主要介绍常用的几个基础类：包装类、String类、StringBuilder类、Arrays类。它们的常用用法（要记）、实现逻辑

## 1. 包装类

ps：关于源码的分析，都在另一个文档中了

Java的8种基本数据类型，都有**分别对应一个包装类**

| 基本数据类型 | 包装类      | 基本数据类型 | 包装类        |
| ------------ | ----------- | ------------ | ------------- |
| byte         | Byte        | char         | **Character** |
| boolean      | Boolean     | short        | Short         |
| int          | **Integer** | long         | Long          |
| float        | Float       | double       | Double        |

包装类内部有：一个实例变量，保存对应的基本类型的值。还有很多静态方法、静态变量、实例方法，能够方便堆数据进行操作。

eg：Integer：

```java
private final int value;			// 私有实例变量，只能赋值一次
public Integer(int value) {			// 两个构造方法
	this.value = value;			// ——传递int值
}
public Integer(String s) throws NumberFormatException {// ——传递String值，然后转换成int赋值
	this.value = parseInt(s, 10);		// 可能会抛出异常
}
```

why要定义包装类？

$\because$ Java很多方法只能操作对象，为了能够操作基本数据类型，于是就将基本数据类型包装成包装类，然后能够使用这些方法。并且包装类还有很多方法可以来操作数据。

### （1）基本用法

包装类可以和对应的基本数据类型互相转换

基本数据类型 -> 包装类：**`valueOf()`** =>每个包装类都实现了该函数

包装类 -> 基本数据类型：**`xxxValue()`**，eg：`aa.intValue()`，`cc.charValue()`

eg:

```java
int a = 1;
Integer aa = Integer.valueOf(a);		// 将基本数据类型转换为对应的包装类
int aaa = aa.intValue();		// 将包装类转换为基本数据类型

// 将int值转换成Integer包装类：如果值在-128~127范围内，那么就在cache池种找到该引用；如果不存在那么就新建一个包装类对象
public static Integer valueOf(int i) {
	if (i >= IntegerCache.low && i <= IntegerCache.high)
		return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
    
// => 那么意味着：通过aa = a = 40;然后调用Integer.valueOf(aa),Integer.valueOf(a)，得到的都是同一个对象；如果aa = a = 250; 那么就是两个不同的对象
int a = 2, aa = 2;
System.out.println(Integer.valueOf(a) == Integer.valueOf(aa));
int b = 250, bb = 250;
System.out.println(Integer.valueOf(b) == Integer.valueOf(bb));
>> true
>> false
```

**将基本数据类型转换为包装类的过程——装箱**

**将包装类转换为基本数据类型的过程——拆箱**

Java5之后引入了自动装箱和拆箱，即直接赋值即可，而不用调用方法（是编译器自动会替换为对应的方法调用）

```java
Integer a = 100;	// 自动装箱，编译器会自动替换：Integer a = Integer.valueOf(100);
int b = a;		// 自动拆箱，实际上编译器会将其替换为 int b = a.intValue();
```

=> 这个是Java编译器提供的能力

每个包装类也会提供构造方法：new来创建（和普通类一样）

那么现在有两种方式创建包装类对象：

- new：普通类对象创建方式
- 调用装箱方法（默认会调用）**valueOf()**——更建议用此种方法，因为new创建，每次都会创建一个新对象。而包装类在**底层实现的过程种都会引入缓存技术**，减少需要创建的对象次数，节省空间提高性能（除了Float和Double）

### （2）包装类共同点

它们都重写了Object方法，都实现了Comparable接口

```java
public final class Integer extends Number implements Comparable<Integer> {
```

#### 重写Object超类的方法

- 重写`equals(Object obj)`方法

  用来判断：当前对象和传入的对象是否相同

  Object默认是比较地址，只有指向的地址一样才返回true，等同于`==`

  而equals应该反映的是对象之间的逻辑相等关系（即值相等），对于包装类（本质上是代表基本数据类型的，基本数据类型应该是值比较）这样的默认实现是不合理的

  ——**所有包装类全部重写，比较的是包装类对应的基本类型值**

  具体见：[包装类源码](http://note.youdao.com/noteshare?id=fc7f62615236fbe3814aefabdd5c2241&sub=E1449B87AC3240ADA389BAF1006C6459)

  使用：

  ```java
  Integer b = 400, a = 400;
  System.out.println(b.equals(a));	// >>true。因为值一样
  Float f = 0.01f, ff = 0.1f*0.1f;
  System.out.println(f.equals(ff));	// >>false。因为小数计算存在误差，所以值不相等
  ```

- 重写`hashCode()`方法

  hashCode返回的是一个**对象的哈希值**，int类型，主要是由对象中一般不变的属性映射得到。

  用处：快速对对象进行区分、分组

  要求：**一个对象的哈希值不能变，相同对象的哈希值必须一样，不同对象的哈希值一般应该不同（不强制不同**，eg：那出生日期作为一个班上学生的hashCode，存在不同对象的哈希值一样的情况，但是问题不大）

  **hashCode和equals方法密切**，两个对象，如果**equals返回true（表示同一个对象），那么hashCode也必须返回一样**；如果equal返回false，那么hashCode尽量不一样

  hashCode的默认实现：将对象的地址转换为整数返回。**子类如果重写了equals方法，那么也必须重写	hashCode**（编译器并不会这么要求，但是从逻辑上是需要的）

  [详见包装类源码ps5](http://note.youdao.com/noteshare?id=9f8ab6eaf048c06b8c823ebe1c7f0b8c&sub=38FCCF96A0C74660A8B19E1140BA449E)

  **因为包装类重写了equals方法，将其逻辑修改了（虽然不是同一对象，但是值一样就认为是一样的），那么我们也需要重写hashCode方法**：

  包装类们是如何实现的：详见：[包装类源码](http://note.youdao.com/noteshare?id=fc7f62615236fbe3814aefabdd5c2241&sub=E1449B87AC3240ADA389BAF1006C6459)

- 重写`toString()`方法——注意这个是实例方法（下面的是静态方法），是用来对该对象的直观文本表示，简要的表达出该对象的信息（一般就是输出对象的某些值等，也会带一定的格式）

  Object类下的方法实现：

  ```java
  // 获得该实例对象的类名 @ 该对象的hashCode的十六进制表示
  public String toString() {
      return getClass().getName() + "@" + Integer.toHexString(hashCode());
  }
  ```

  其余的见：[包装类源码](http://note.youdao.com/noteshare?id=fc7f62615236fbe3814aefabdd5c2241&sub=E1449B87AC3240ADA389BAF1006C6459)

- String和包装类的互相转换

  - 根据字符串表示返回基本类型值：parse

    具体的见：[包装类源码](http://note.youdao.com/noteshare?id=fc7f62615236fbe3814aefabdd5c2241&sub=E1449B87AC3240ADA389BAF1006C6459)

  - 根据基本类型值返回字符串表示：toString——注意这个和上面的实例方法`toString`做区分（实例方法的，是为了表达出该对象的一些信息，而本方法只是为了将值转换成字符串输出）

    具体见：[包装类源码](http://note.youdao.com/noteshare?id=fc7f62615236fbe3814aefabdd5c2241&sub=E1449B87AC3240ADA389BAF1006C6459)

  并且对于整型，还提供了其他进制的转换：

  字符串按照其他进制转换成整型；整型转换成指定进制表示的字符串（默认进制是10进制）

- 常用常量

  具体见：[包装类源码](http://note.youdao.com/noteshare?id=fc7f62615236fbe3814aefabdd5c2241&sub=E1449B87AC3240ADA389BAF1006C6459)

  => 总结：列出了该包装类的二进制下的最大位数`SIZE`和对应的字节数`BYTES`

  ​	还列出了该包装类的最大最小值`MIN_VALUE`、`MAX_VALUE`，当数字比较小的时候用十进制表示；当数字比较大的时候，用十六进制表示。Float/Double特殊还列出了正无穷大、负无穷大，NaN数（都是通过计算得到的，而不能直接表达出数字），还列出了e的最大值（因为浮点数是用科学计数法表示的）

- Number

  具体见：[包装类源码](http://note.youdao.com/noteshare?id=fc7f62615236fbe3814aefabdd5c2241&sub=E1449B87AC3240ADA389BAF1006C6459)

  包装类都继承自该父类（除了Boolean），而Number定义了6个抽象方法——将当前的对象值输出为6种基本数据类型格式，要求每个包装类均实现，那么**包装类实例之间可以返回任意的基本数据类型**

#### 实现Comparable接口

所有包装类均实现了接口Comparable的功能：

原始的Comparable接口的定义：——只有一个`compareTo`方法

限定返回值是：-1：当前值 < 其他值；0：当前值 = 其他值；1：当前值 > 其他值

```java
// Comparable接口：主要就是一个方法
public interface Comparable<T> {
	public int compareTo(T o);
}
```

<T>是泛型语法，T表示比较的类型，由实现的接口传入指定类型

包装类的`compareTo`的实现：

具体见：[包装类源码](http://note.youdao.com/noteshare?id=fc7f62615236fbe3814aefabdd5c2241&sub=E1449B87AC3240ADA389BAF1006C6459)

浮点数同样存在计算误差问题，导致0.01和0.1*0.1的结果并不相等（和equals一样）

#### 包装类对象有不可变性

<a name="immutability"></a>

**实例对象一旦创建，里面的值不可变了。**

具体实现在：

- 所有包装类都是final，表示该类不可被继承
- 内部的基本数据类型是private的，且声明为final——即只能赋值一次，且不能通过直接访问的方式修改，eg：a.value（不可行）
- 没有定义setter方法，不能通过调用方法的方式修改值

why 设置为不可变类呢？

程序更为简单且安全，不用担心数据会被意外修改，可以**安全的共享数据**，尤其是在多线程的情况下。——一旦创建只能读不能写

### （3）Integer

主要是关注二进制操作

1. 位翻转

   Integer提供了两个移位方法：第一是按位翻转；第二个是按字节翻转

   ```java
   public static int reverse(int i);
   public static int reverseBytes(int i);
   ```

   具体见：[包装类源码](http://note.youdao.com/noteshare?id=fc7f62615236fbe3814aefabdd5c2241&sub=E1449B87AC3240ADA389BAF1006C6459)

2. 循环移位

   提供两个方法：循环左移和循环右移

   ```java
   public static int rotateLeft(int i, int distance);
   public static int rotateRight(int i, int distance);
   ```

   具体见：[包装类源码](http://note.youdao.com/noteshare?id=fc7f62615236fbe3814aefabdd5c2241&sub=E1449B87AC3240ADA389BAF1006C6459)

3. valueOf的实现

   在[(1)基本用法](#valueof_link)中有提到，就是Integer会提前缓存-128~127范围内的Integer对象，如果通过valueOf创建对象值是在这个范围内的，那么就直接返回该对象，而不会创建新对象；如果超出范围，才会通过new进行创建

   ——**所以创建包装类对象时，推荐使用`valueOf`方法**

   => Boolean、Byte、Short、Long、Character都有类似的实现

   通过共享常用的对象，可以节省内存空间，且由于设定的包装类是[不可变的](#immutability)，该对象可以被安全的使用

   => 这个也是常见的设计思路：**享元模式**（Flyweight）

### （4）Character

1. Unicode基础

   见[计算机常见概念](#4)

2. 检查codePoint和char

   ```java
   //判断一个int是不是一个有效的代码点，小于等于0x10FFFF的为有效，大于的为无效
   public static boolean isValidCodePoint(int codePoint);
   //判断一个int是不是BMP字符，小于等于0xFFFF的为BMP字符，大于的不是
   public static boolean isBmpCodePoint(int codePoint);
   //判断一个int是不是增补字符，0x010000～0X10FFFF为增补字符
   public static boolean isSupplementaryCodePoint(int codePoint);
   //判断char是否是高代理项，0xD800～0xDBFF为高代理项
   public static boolean isHighSurrogate(char ch);
   //判断char是否为低代理项，0xDC00～0xDFFF为低代理项
   public static boolean isLowSurrogate(char ch);
   //判断char是否为代理项， char为低代理项或高代理项，则返回true
   public static boolean isSurrogate(char ch);
   //判断两个字符high和low是否分别为高代理项和低代理项
   public static boolean isSurrogatePair(char high, char low);
   //判断一个代码点由几个char组成，增补字符返回2, BMP字符返回1
   public static int charCount(int codePoint);
   ```

3. codePoint和char的转换

   实现整型表示的编号和char之间的转换

   ```java
   //根据高代理项high和低代理项low生成代码点，这个转换有个公式，这个方法封装了这个公式
   public static int toCodePoint(char high, char low);
   //根据代码点生成char数组，即UTF-16表示，如果code point为BMP字符，则返回的char
   //数组长度为1，如果为增补字符，长度为2, char[0]为高代理项，char[1]为低代理项
   public static char[] toChars(int codePoint);
   //将代码点转换为char数组，与上面方法类似，只是结果存入指定数组dst的指定位置index
   public static int toChars(int codePoint, char[] dst, int dstIndex);
   //对增补字符code point，生成低代理项
   public static char lowSurrogate(int codePoint);
   //对增补字符code point，生成高代理项
   public static char highSurrogate(int codePoint);
   ```

4. 按codePoint处理char数组或序列

   ```java
   // 返回char数组从offset开始到offset+count个char中包含code_point的个数
   public static int codePointCount(char[] a, int offset, int count);
   
   eg：
   char[] chs = new char[3];
   chs[0] = '马';
   Character.toChars(0x1FFFF, chs, 1);
   System.out.println(Character.codePointCount(chs, 0, 3)); >> 2（马包含2个字符）
   ```

   具体的看书吧，用到再说：p183~184

5. 字符属性

   Unicode给每个字符分配编号，也提供了一些属性，而Character类封装了对Unicode字符属性的检查和操作

   获取字符类型：

   ```java
   public static int getType(int codePoint);	// 可以接受int类型的编号，可以处理所有字符
   public static int getType(char ch);	// 可以接受char类型的，但就只能处理BMP
   ```

   ——该类型很重要，很多其他检查和操作都是基于该类型的

   举例：

   <img src="C:/Users/surface/Desktop/计算机知识整理/java/pic/image-20201218155946343.png" alt="image-20201218155946343" style="zoom:67%;" />

   ```java
   public static boolean isDefined(int codePoint);	// 检查该编号字符是否在Unicode被定义
   ——返回0表示未定义；返回非0，表示其编号
   public static boolean isDigit(int codePoint);	// 检查是否是数字（包括中英文下的）
   ——主要就是根据类型进行判断的
   ```

   具体看书，p185~188

6. 字符转换

   有大小写的字符规定了其对应的大小写；如果字符有数值含义，也给定了数值

   ```java
   public static int toLowerCase(int codePoint);	// 主要针对英文字符A~Z获得对应的小写字符
   public static int toUpperCase(int codePoint);	// a~z，获得对应的大写字符
   ```

   ```java
   public static int getNumericValue(int codePoint);	// 返回字符对应的数值
   // 0~9,a~z(A~Z):10~35
   public static int digit(int codePoint, int radix);	// 返回按给定进制表示的数值，数值需要小于radix
   //eg:digit('F', 16) >> 15;digit('G', 16)>>-1
   
   public static char forDigit(int digit, int radix);	// 给定数值和进制，返回字符
   //eg:forDigit(15, 16)>>'F'
   
   public static char reverseBytes(char ch);	// 按字节翻转
   ```



## 2. 文本处理类String和StringBuilder

最常见的操作之一。Java中用来处理字符串的主要类就是`String`和`StringBuilder`。

### （1）String的基本用法

定义使用：

```java
String s = "hello world";		// 可以直接定义使用（类似于数组，也可以直接跟数组内容）
String s = new String("hello");	// 可以用new
```

字符串可以直接连接，用+连接

```java
String s1 = "hello";
s1 += "world";
```

[其他的方法](#string_fun)

### （2）String的实现原理

常量：

```java
private final char value[];	// 存在一个字符数组来表示字符串，注意是final——不可修改的
```

构造方法：（还有其他的构造方法）

```java
public String(char value[]);
public String(char value[], int offset, int count);	// 会重新创建一个数组存放，而不是直接使用

// 可以根据给定的编码方式创建String对象
public String(byte bytes[], int offset, int length, String charsetName);
public String(byte bytes[], Charset charset);
```

String常用的方法：

```java
public int length();
public String subString(int beginIndex, int endIndex);
// 指定字符，返回第一次出现该字符的索引值
public int indexOf(char ch);
//返回指定索引位置的char——指定下标返回对应的值（用在字符串遍历中）s.charAt(i)
public char charAt(int index); 
//返回字符串对应的char数组， 注意，返回的是一个复制后的数组，而不是原数组
public char[] toCharArray();
//将char数组中指定范围的字符复制入目标数组指定位置
public void getChars(int srcBegin, int srcEnd, char dst[], int dstBegin);
```

编码转换：

存在不同的编码方式，那如何处理不同编码呢？

使用`Charset`类表示各种编码，两个静态方法

```java
public static Charset defaultCharset();	// 返回系统默认的编码方式，UTF-8本机
public static Charset forName(String charsetName);	// 指定编码方式——编码名字
// 可以有：US-ASCII、ISO-8859-1、windows-1252、GB2312、GBK、GB18030、Big5、UTF-8....
// eg: Charset charset = Charset.forName("GB18030");

public byte[] getBytes();		// 按照默认的编码方式，返回字符串
public byte[] getBytes(String charsetName);	// 给定编码名称
public byte[] getBytes(Charset charset);	// 给定编码对象
```

hashCode

```java
private int hash; // 默认是0，是内部实例变量，用来缓存，第一调用hashCode()方法后会将值保存在该变量中，以后再次调用就直接返回该值即可

public int hashCode() {		// 生成hash值的方法
    int h = hash;
    if (h == 0 && value.length > 0) {	// 如果h！=0，说明已经计算过了，可以直接返回
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];	// 从字符串头开始，*31+当前字符串对应的值
        }
        hash = h;
    }
    return h;
}
// 实际上是如此：s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]
```

使用31的原因，见[Java的额外知识](http://note.youdao.com/noteshare?id=9f8ab6eaf048c06b8c823ebe1c7f0b8c&sub=38FCCF96A0C74660A8B19E1140BA449E)

——Java中大部分都是用循环相乘多个参数来实现hashCode的计算的，至于如何生成一个合理的hash值，见[Java的额外知识](http://note.youdao.com/noteshare?id=9f8ab6eaf048c06b8c823ebe1c7f0b8c&sub=38FCCF96A0C74660A8B19E1140BA449E)

#### （3）String的不可变性

对象一旦创建就不能更改了：String类不能被继承；内部`char[] value`数组是final，可以且只可以被赋值一次

——很多看似是修改了字符串内容的方法，实际上是创建了新对象，而旧对象不可被修改。

eg：

```java
public String concat(String str) {	// 实现字符串连接
    int otherLen = str.length();
    if(otherLen == 0) {
        return this;
    }
    int len = value.length;
    char buf[] = Arrays.copyOf(value, len + otherLen);// 创建数组，原来的数组+添加的数组合并
    str.getChars(buf, len);
    return new String(buf, true);
}
```

String设置为不可变，那么更简洁更安全。但是如果需要频繁修改字符串，每次修改实际上都是创建新的对象，那么性能太低，应该考虑其他类：StringBuilder和StringBuffer

#### （4）常量字符串

常量字符串可以直接调用String的各种方法，它们本质上也是String类型的对象。

eg：

```java
System.out.println("hello world".length());	// 这个就是常量字符串
```

在内存中，常量字符串会被放在一个共享的地方——**字符串常量池**，保存所有的常量字符串，且只保存一份，可以让所有使用者读，不能写。常量形式使用字符串，使用的就是常量池中的对象

——如果通过new得到的字符串，就不是常量字符串了

eg：

```java
String s1 = "hello world";
String s2 = "hellow world";
// >> s1 == s2， 得到true，都是使用常量池中的对象

String ss1 = new String("hello world");	// ss1和ss2的char[]value是指向同一个，
String ss2 = new String("hello world");	// 但是分配的对象内存不同
// >> ss1 != ss2 equals相等
```

<img src="C:/Users/surface/Desktop/计算机知识整理/java/pic/image-20201218183915434.png" alt="image-20201218183915434" style="zoom: 60%;" />

#### （5）正则表达式

String中有些方法接受的实际上不是字符串，而是正则表达式——**本质上是字符串，但是表达的是一个规则，用来文本匹配、查找、替换等**。

Java中存在专门的正则表达式的类，eg：Pattern、Matcher。但是String会处理一些简单的情况

```java
public String[] split(String regex);  //分隔字符串——传递的就是正则表达式
public boolean matches(String regex); //检查是否匹配
public String replaceFirst(String regex, String replacement); //字符串替换
public String replaceAll(String regex, String replacement); //字符串替换
```

ps：Java 9对String的实现进行了优化，它的内部不是char数组，而是**byte数组**，如果字符都是ASCII字符，它就可以**使用一个字节表示一个字符，而不用UTF-16BE编码，节省内存**。

#### （6）StringBuilder的基本用法

对于需要频繁修改字符串的场景，不适合用String（会不断创建新的对象，而浪费内存），而应该采用：`StringBuffer`和`StringBuilder`。它们的实现几乎一样，方法也一样，但是`StringBuffer`是线程安全的（会影响性能，有程本代价的），`StringBuilder`不是

创建对象：

```java
StringBuilder sb = new StringBuilder();
```

方法：

```java
sb.append("hello");	// 用实例方法append，添加字符串
System.out.println(sb.toString());		// toString() 方法输出字符串
```

#### （7）StringBuilder的实现原理

StringBuilder里面也封装了一个字符数组，但是是可以修改内容的

```Java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    char[] value;	// 存储字符数组（和String类似），只不过不是final，还可以修改
    int count;		// 存储数组中已经使用的字符个数（就是当前字符串长度）
    ....
}
```

构造方法：

```java
public StringBuilder() {
    super(16);		// 默认调用父类的方法——创建长度为16的字符数组
}

AbstractStringBuilder(int capacity) {
    value = new char[capacity];
}

// 如果预先知道字符串的长度，可以直接传递数组长度
public StringBuilder(int capacity) {
    super(capacity);
}
```

append方法：

```java
@Override
public StringBuilder append(String str) {
    super.append(str);
    return this;
}

public AbstractStringBuilder append(String str) {
    if (str == null)
        return appendNull();
    int len = str.length();	// 求要加入的字符串长度
    ensureCapacityInternal(count + len);	// 如果当前字符数组不够长，会扩容，指数级扩容
    str.getChars(0, len, value, count);	// 会将新的字符串加入到原来的字符串后面
    count += len;		// 更新最新实际的字符串长度
    return this;
}
```

具体见：[Java库源码阅读](http://note.youdao.com/noteshare?id=fc7f62615236fbe3814aefabdd5c2241&sub=E1449B87AC3240ADA389BAF1006C6459)

扩容操作：

- 为了减少内存浪费，所以不申请很大的数组长度
- 指数扩容也能减少内存分配次数

=> 在不知道最终字符串有多长的情况下，**指数扩展是一种常见的策略**，广泛应用于各种内存分配相关的计算机程序中

insert方法：见：[Java库源码阅读](http://note.youdao.com/noteshare?id=fc7f62615236fbe3814aefabdd5c2241&sub=E1449B87AC3240ADA389BAF1006C6459)

#### （8）String和+/+=运算符

Java编译器提供的功能：String字符串添加操作可以直接通过+/+=实现，但是一般编译中会将其**转换为StringBuilder，+/+=操作会转换成append**

```java
String hello = "hello";
hello+=",world";

// 编译之后：
StringBuilder hello = new StringBuilder("hello");
hello.append(",world");
```

——一般情况，编译器都会自动进行转换，而稍微复杂的情况下，Java编译器可能不会进行转换，例如：循环的情况：

```java
String hello = "hello";
for(int i=0;i<3;i++){
	hello+=",world";
}
System.out.println(hello);

// 编译之后：
String hello = "hello";
for(int i=0;i<3;i++){
    StringBuilder sb = new StringBuilder(hello);	// 本质上没有优化，应该在循环进入前变成StringBuffer，所以，需要直接使用StringBuilder
    sb.append(",world");
    hello = sb.toString();
}
System.out.println(hello);
```

## 3. 数组操作类Arrays

对标C的数组，相比较后面的容器，效率会高很多——因为是随机存取的，所以时间复杂度就是O(1)

Java中存在一个类Arrays，里面包含了**对数组操作的静态方法**

```java
int[] arr = {9,8,3,4};
System.out.println(Arrays.toString(arr));
String[] strArr = {"hello", "world"};			// 对象元素
System.out.println(Arrays.toString(strArr));// 本质上去调用针对Object对象的方法——打印出数组的内容
System.out.println(strArr);		// 输出的是数组的内存首地址
```

#### （1）用法

排序：

```java
int[] arr = {4, 9, 3, 6, 10};
Arrays.sort(arr);			// 排序，默认升序

// 自定义排序方法——只能接受对象，而不是基本数据类型
Integer[] arr = {4, 9, 3, 6, 10};
Arrays.sort(arr, new Comparato<Integer>(){
   @Override
    public int compare(Integer o1, Integer o2){	// 数组中的元素，o1前一个元素；o2后一个元素
        return o1 < o2 ? 1:(o1 == o2 ? 0 : -1);
    }
});

String[] arr = {"hello","world", "Break","abc"};	// 忽略大小的逆序输出
Arrays.sort(arr, new Comparator<String>() {
    @Override
    public int compare(String o1, String o2) {
    	return o2.compareToIgnoreCase(o1);
    }
});
```

=> 注意：自定义排序就是去实现比较器接口Comparator，实现一个方法即可compare

注意，**该方法只能接受对象参数**，所以普通的基本类型数组要转换成对应的包装类才行（这边就体现了包装类的功能）

——这边可以发现：**排序的基本步骤是不变的，但是按什么排序是变化的，sort将不变的算法设计为主体逻辑，而将变化的排序方式设计为参数，允许调用者动态指定——策略模式**，将变化和不变相分离。

查找：

```java
int[] arr = {3,5,7,13,21};
System.out.println(Arrays.binarySearch(arr, 13));	//>>3
System.out.println(Arrays.binarySearch(arr, 11));	//>>-4，表示适合插入在index=3的地方
```

比较：比较两个数组的元素是否完全一样——长度一样，每个元素都一样，才算是相等（本质上就是去调用对象的equals方法）

copyOf：复制一个数组到新数组中去——**浅克隆**，只是新建了数组，而数组中元素对象，本质上是复制了对象的引用，没有为每个元素创建新对象。

#### （2）多维数组

创建多维数组时，**只需要指定第一维的长度即可**，其他维的不需要，第一维的每个元素数组的长度可以不一样（和C差别很大，C的每个维度必须一样，因为原理不一样）

```java
int[][] arr = new int[2][];
arr[0] = new int[3];		// 长度不一样
arr[1] = new int[5];
```

——因为从本质上来讲，**只有一维数组这个概念，而二维数组实际上就是一维数组的元素都是数组类型**，n维类推即可。

针对多维数组的方法：如果发现数组的元素还是数组类型，那么就去递归调用，直到数组元素是基本数据类型

```java
public static String deepToString(Object[] a);
public static boolean deepEquals(Object[] a1, Object[] a2);
public static int deepHashCode(Object a[]);
```

#### （3）实现原理

具体见：[Java库源码阅读](http://note.youdao.com/noteshare?id=fc7f62615236fbe3814aefabdd5c2241&sub=E1449B87AC3240ADA389BAF1006C6459)

## 4. 时间和日期的处理

介绍的是Java8之前的时间API，Java8开始重新设计了日期和时间的API，新增了Java包`java.time`（使用了Lambda，之后再看）

#### （1）时间的基本概念

计算机统一**用一个整数表示时刻**，是距离格林尼治标准时间1970.1.1.00:00:00开始的**毫秒数**。而该开始时间被称为纪元时（Epoch Time）。1970年之前的用负数表示。

时刻，世界上各个地方都是一样的，只不过各个地区对该时刻的解读不一样——生成了时区和年历。

时区：同一时刻，具体时间是不同的，看所在的不同时区。全球共有24个时区。0时区时间用GMT+0表示；北京时间用GMT+8:00表示

年历还分公历和农历（我国特有），Java API没有支持农历，所以只关注公历。

#### （2）Date类的API

Date是最早引入的关于日期的类，但是很多方法都已经过时，被标记为`@Deprecated`（不推荐使用）

**Date现在主要用来表示时刻**

变量：

```java
private transient long fastTime;	// 记录距离纪元时的毫秒数
```

构造方法：

```java
public Date() {			// 默认构造方法
    this(System.currentTimeMillis());		// 返回当前时刻距离纪元时的毫秒数
}

public Date(long date) {
    fastTime = date;		// 根据输入的毫秒数进行初始化
}
```

还可以用的方法：<a name="date_fun"></a>

```java
public long getTime(); //返回距离纪元时的毫秒数
public boolean equals(Object obj); //主要就是比较内部的毫秒数是否相同
//与其他Date进行比较，如果当前Date的毫秒数小于参数中的返回-1，相同返回0，否则返回1——进行时刻比较
public int compareTo(Date anotherDate);
public boolean before(Date when); //判断当前时刻是否在给定日期之前
public boolean after(Date when); //判断当前时刻是否在给定日期之后
public int hashCode(); //哈希值算法与Long类似
```

#### （3）TimeZone类的API

类头：发现是抽象类

```java
abstract public class TimeZone implements Serializable, Cloneable {}
```

存在静态方法获得当前默认的时区：`getDefault()`；静态方法获得指定时区的对象`getTimeZone()`

通过实例方法`getID()`得到该对象的ID

```Java
public static TimeZone getDefault() {		// 返回的是TimeZone对象
    return (TimeZone) getDefaultRef().clone();
}

// 使用
TimeZone tz = TimeZone.getDefault();		// 调用静态方法得到TimeZone对象
System.out.println(tz.getID());		// 调用实例方法 >> Asia/Shanghai

// 系统属性，保存了本机的默认时区
System.out.println(System.getProperty("user.timezone"));
// Java启动的时候可以进行修改
java -Duser.timezone=US/Eastern
    
// 还可以获得任意时区的实例对象
TimeZone tz = TimeZone.getTimeZone("US/Eastern");	// 也可以是GMT形式："GMT+08:00"
```

#### （4）Locale类的API

表示国家/地区和语言，

有两个主要参数：每个参数都有代码

- 国家/地区，eg：中国CN，中国台湾TW，美国US
- 语言，eg：英文en，中文zh

<img src="C:/Users/surface/Desktop/计算机知识整理/java/pic/image-20201219131415284.png" alt="image-20201219131415284" style="zoom: 50%;" />

```java
Locale locale = Locale.getDefault();		// 获得默认的国家/语言属性
System.out.println(locale.toString());
```

#### （5）Calendar类的API

是日期和时间操作的主要类

类头：Calendar也是抽象类

```java
public abstract class Calendar implements Serializable, Cloneable, Comparable<Calendar> 
```

常量：

```java
protected long time;	// 记录毫秒数
protected int fields[];		// 存储日历中各个字段的值，长度为17

// 主要包括了如下字段的值：
Calendar.YEAR：表示年
Calendar.MONTH：表示月，1月是0，还定义了月份的静态变量，eg：Calendar.JULY
Calendar.DAY_OF_MONTH：表示日，每月的第一天是1
Calendar.HOUR_OF_DAY：表示小时，为0～23
Calendar.MINUTE：表示分钟，为0～59
Calendar.SECOND：表示秒，为0～59
Calendar.MILLISECOND：表示毫秒，为0～999
Calendar.DAY_OF_WEEK：表示星期几，周日是1，周一是2，还定义对应的静态变量，eg：Calendar.SUNDAY
```

由于是抽象类不能new对象，提供了静态方法可以获得Calendar对象实例

本质上就是去调用：Locale和TimeZone的方法，如果没有传递这两个对象会使用默认值`getDefault`

```java
public static Calendar getInstance()
{
	return createCalendar(TimeZone.getDefault(),                                      Locale.getDefault(Locale.Category.FORMAT));
}

public static Calendar getInstance(TimeZone zone, Locale aLocale)
{
    return createCalendar(zone, aLocale);
}
```

分析：

1. `getInstance()`会**根据TimeZone和Locale对象来创建对应的Calendar子类对象**（不同对象创建不同的子类）
2. `getInstance()`方法封装了Calendar对象创建的细节

——**隐藏对象创建的细节，根据传参不同创建的具体子类也不同，也是计算机中常见的设计模式：工厂方法**，getInstance就是一个工厂方法，作用就是生产对象。

获取不同字段的信息：

```java
// 获得当前时间的各种信息：年、月、日等
Calendar calendar = Calendar.getInstance();
System.out.println("year: "+calendar.get(Calendar.YEAR));
System.out.println("month: "+calendar.get(Calendar.MONTH));
System.out.println("day: "+calendar.get(Calendar.DAY_OF_MONTH));
System.out.println("hour: "+calendar.get(Calendar.HOUR_OF_DAY));
System.out.println("minute: "+calendar.get(Calendar.MINUTE));
System.out.println("second: "+calendar.get(Calendar.SECOND));
System.out.println("millisecond: " +calendar.get(Calendar.MILLISECOND));
System.out.println("day_of_week: " + calendar.get(Calendar.DAY_OF_WEEK));
```

分析：实现上来说：Calendar会将时刻转换成对应的年历，存放在`fields`中，然后`calendar.get()`就是根据其索引获得对应字段的值

设置时间：

```java
public final void setTime(Date date);		// 根据时间设置
public void setTimeInMillis(long millis);	// 根据毫秒数设置
//根据各个字段设置
public final void set(int year, int month, int date);
public final void set(int year, int month, int date,
                      int hourOfDay, int minute, int second);
public void set(int field, int value);

// 还可以根据amount修改时间，带进位的
public void add(int field, int amount);	// eg:13:59 + 2m = 14:01
// 修改时间，不带进位的，如果溢出就从本字段从头开始
public void roll(Calendar.MINUTE, 3);	// eg: 13:59 +2m = 13:01
```

在内部实现上，Calendar会更新fields数组对应字段的值，但一般不会立即更新其他相关字段或内部的毫秒数的值，不过在获取时间或字段值的时候， Calendar会重新计算并更新相关字段。——延迟更新，到需要获得标准时间的时候更新

还能进行Calendar对象之间的比较`compareTo`等，同[Date](#date_fun)

#### （6）DateFormat类和子类SimpleDateFormat

DateFormat类主要就是在Date和字符串表示之间进行转换

```java
public final String format(Date date);	// 日历转换为string格式
public Date parse(String source);		// string格式转换为日历对象
```

Date的字符串表示与TimeZone和Locale都是相关的，除此之外，还与两个格式化风格有关，一个是日期的格式化风格，另一个是时间的格式化风格。

DateFormat定义了4个静态变量，表示4种风格：`SHORT`、`MEDIUM`、`LONG`和`FULL`；还定义了一个静态变量DEFAULT，表示默认风格，值为MEDIUM，不同风格输出的信息详细程度不同。

DateFormat也是抽象类，也用**工厂方法创建对象**，提供了多个静态方法创建DateFormat对象，有三类方法：

```java
public final static DateFormat getDateTimeInstance();	// 获得时间和日期
public final static DateFormat getDateInstance();	// 只获得日期
public final static DateFormat getTimeInstance();	// 只获得时间

// 每个都还有2个重载方法，用来接受风格设置
DateFormat getDateTimeInstance(int dateStyle, int timeStyle);
DateFormat getDateTimeInstance(int dateStyle, int timeStyle, Locale aLocale);
// eg: 输出中文格式
System.out.println(DateFormat.getDateTimeInstance(DateFormat.LONG,
DateFormat.SHORT,Locale.CHINESE).format(calendar.getTime()));

// 支持修改时区的setter方法
public void setTimeZone(TimeZone zone)
```

SimpleDateFormat可以接受一个自定义的模式（pattern）作为参数，规定了Date字符串的形式：

```java
Calendar calendar = Calendar.getInstance();
calendar.set(2020, 12, 19, 14, 31, 05);// 2020-12-19 14:31:05
SimpleDateFormat sdf = new SimpleDateFormat(
				"yyyy年MM月dd日 E HH时mm分ss秒");		// 这个就是自定义的模式	
System.out.println(sdf.format(calendar.getTime()));

// >> 2020年12月19日 星期六 14时31分05秒
```

注意：pattern中英文字符a~z/A~Z表示特殊含义，其他字符都按原样输出

HH：表示24小时制；hh：表示12小时制，a能表示上下午

字符串和Date的相互转换

```java
String str = "2016-08-15 14:15:20.456";		// 字符串
// 自定义处理模式
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.SSS");
try {
    Date date = sdf.parse(str);		// parse将字符串转换为Date
    SimpleDateFormat sdf2 = new SimpleDateFormat("yyyy年M月d h:m:s.S a");
    System.out.println(sdf2.format(date));		// format将Date转换为字符串模式
    } catch (ParseException e) {
    	e.printStackTrace();
}
```

分析：这边会抛出一个受检异常，**受检异常必须要处理**，所以要try/catch格式，异常为`parseException`

#### （7）局限性

总结前面的：

- Date主要就是表示时刻（其他日历方法过时，不使用）
- Calendar表示日历，主要和TimeZone、Locale相关，并且可以进行运算
- DateFormat和子类SimpleDateFormat，可以使字符串和Date对象相互转换，并且后者可以自定义模式

但是存在局限：

1. Date很多方法和常识不符合

   ```java
   Date date = new Date(2016,8,15);	// 实际上是3916-9-15（Date计算的是和1900年的差值）
   ```

2. Calendar操作繁琐

   简单操作需要多次进行调用，且难以进行复杂的日期操作，eg：计算两个日期之间间隔的月份等

3. DateFormat的线程安全性问题

   不是线程安全的，多个线程同时使用一个实例对象会存在问题。

## 5. 随机函数

#### （1）基本随机：Math.random()

`Math.random()`**生成一个[0, 1)的double类型的随机数**

```java
public static double random() {
    return RandomNumberGeneratorHolder.randomNumberGenerator.nextDouble();// 下一个随机double类型的数
}

// 私有静态内部类
private static final class RandomNumberGeneratorHolder {
    static final Random randomNumberGenerator = new Random();	// 存放一个Random对象
}

// Random.java
public double nextDouble() {
    return (((long)(next(26)) << 27) + next(27)) * DOUBLE_UNIT;
}
```

分析发现：本质上就是去调用Random类，生成一个random对象去生成一个随机数

#### （2）进阶随机：Random类

Random提供了更多的随机数类型

但是需要先创建Random对象，才能开始使用

```java
Random rnd = new Random();		// 创建实例对象
System.out.println(rnd.nextInt());	// 获得在int范围内的随机数，正负均可能
System.out.println(rnd.nextInt(100));	// 在[0, 100)内的随机数

public long nextLong(); //随机生成一个long
public boolean nextBoolean(); //随机生成一个boolean
public void nextBytes(byte[] bytes); //产生随机字节，字节个数就是bytes的长度
public float nextFloat(); //随机浮点数，[0, 1)
public double nextDouble(); //随机浮点数，[0, 1)
```

有一个构造方法：

```java
public Random(long seed);		// 接受种子参数

synchronized public void setSeed(long seed);	// 同样功能
```

在其他语言的随机数我们也能看到seed参数，seed决定了随机产生的序列，种子相同生成的随机数序列就相同——随机数就是以seed为基础进行的一系列计算，而该计算是固定的，所以值一定一样

why要指定seed，是**为了实现可重复的随机**

#### （3）基本原理

Random产生的都是**伪随机数**，真正的随机数比较难产生——计算机程序中的随机数一般都是伪随机数

基本原理是：基于一个seed，每需要一个随机数，都是对当前的seed进行一系列的数学计算，得到一个数，然后这个数就是随机数，并且得到一个新的seed

Random的默认构造方法会提供一个**真随机数**

```java
public Random() {
    // nanoTime，得到的是高精度的当前时间（ns）
    this(seedUniquifier() ^ System.nanoTime());	// 两个long值按位异或
}

private static long seedUniquifier() {
    // L'Ecuyer, "Tables of Linear Congruential Generators of
    // Different Sizes and Good Lattice Structure", 1999
    for (;;) {
        long current = seedUniquifier.get();
        long next = current * 181783497276652981L;
        // 将next值更新为current值，确保值不会重复——保证随机性
        if (seedUniquifier.compareAndSet(current, next))	
            return next;
    }
}
```

生成随机数的函数：

```java
// 这些本质上都去调用了next()方法
public int nextInt() {
	return next(32);
}
public long nextLong() {
	return ((long)(next(32)) << 32) + next(32);
}
public float nextFloat() {
	return next(24) / ((float)(1 << 24));
}
public boolean nextBoolean() {
	return next(1) != 0;
}

private static final long multiplier = 0x5DEECE66DL;
private static final long addend = 0xBL;
private static final long mask = (1L << 48) - 1;

// 生成指定bit位的随机数
protected int next(int bits) {
    long oldseed, nextseed;
    AtomicLong seed = this.seed;
    do {
        oldseed = seed.get();
        // 旧种子*一个固定的数，+一个固定的数，取低48位作为结果
        nextseed = (oldseed * multiplier + addend) & mask;
    } while (!seed.compareAndSet(oldseed, nextseed));
    return (int)(nextseed >>> (48 - bits));
}
```

原理：**线性同余随机数生成器**（linear congruential pseudorandom number generator）

#### （4）使用场景

生成随机密码：随机数字/随机大小字母、数字、特殊符号等/限定必须出现

```java
// 生成6位随机数
public static String randomPassword(){
    char[] chars = new char[6];
    Random rnd = new Random();
    for(int i=0; i<6; i++){
    chars[i] = (char)('0'+rnd.nextInt(10));		// 数字在0~9范围内，直接转换成char类型
    }
    return new String(chars);
}

// 生成带大小写、数字、特殊符号的8位随机数
private static final String SPECIAL_CHARS = "!@#$%^&*_=+-/";
private static char nextChar(Random rnd){
    switch(rnd.nextInt(4)){		// 随机生成0~3的整数，选择该字符的模式：大、小写、数字、特殊符号
    case 0:
    return (char)('a'+rnd.nextInt(26));
    case 1:
    return (char)('A'+rnd.nextInt(26));
    case 2:
    return (char)('0'+rnd.nextInt(10));
    default:
    return SPECIAL_CHARS.charAt(rnd.nextInt(SPECIAL_CHARS.length()));
    }
}
public static String randomPassword(){
    char[] chars = new char[8];
    Random rnd = new Random();
    for(int i=0; i<8; i++){
    	chars[i] = nextChar(rnd);
    }
    return new String(chars);
}

// 在上面的基础上，要求八位数字中一定包含大写、小写、数字、特殊符号
private static int nextIndex(char[] chars, Random rnd){
    int index = rnd.nextInt(chars.length);	// 生成随机索引值
    while(chars[index]!=0){
        index = rnd.nextInt(chars.length);	
	}
	return index;
}
private static char nextSpecialChar(Random rnd){
	return SPECIAL_CHARS.charAt(rnd.nextInt(SPECIAL_CHARS.length()));
}
private static char nextUpperlLetter(Random rnd){
	return (char)('A'+rnd.nextInt(26));
}
private static char nextLowerLetter(Random rnd){
	return (char)('a'+rnd.nextInt(26));
}
private static char nextNumLetter(Random rnd){
	return (char)('0'+rnd.nextInt(10));
}
public static String randomPassword(){
    char[] chars = new char[8];
    Random rnd = new Random();
    chars[nextIndex(chars, rnd)] = nextSpecialChar(rnd);	// 先满足必须出现这个要求
    chars[nextIndex(chars, rnd)] = nextUpperlLetter(rnd);
    chars[nextIndex(chars, rnd)] = nextLowerLetter(rnd);
    chars[nextIndex(chars, rnd)] = nextNumLetter(rnd);
    for(int i=0; i<8; i++){		// 然后补全其他位
        if(chars[i]==0){
        chars[i] = nextChar(rnd);
    	}
    }
    return new String(chars);
}
```

随机重排：给定一组有序的数字，将其随机重排（洗牌）

```java
private static void swap(int[] arr, int i, int j){
    int tmp = arr[i];
    arr[i] = arr[j];
    arr[j] = tmp;
}
public static void shuffle(int[] arr){
    Random rnd = new Random();
    for(int i=arr.length; i>1; i--) {		// 从后向前开始洗牌，
    	swap(arr, i-1, rnd.nextInt(i));	// 当前数字和前面数字随机交换
	}
}
```

**带权重的随机选择：（用在抽奖操作中）**

计算每个选项的累计概率，构成一个数组，然后生成一个[0, 1)的随机数，然后使用二分查找，在数组中找到该随机数应该插入的位置——就是应用二分查找中找不到就返回-(应该插入的索引位置+1)特性——这个就是对应的选项

=>很妙:clap:

```java
public class WeightRandom {
    private Pair[] options;
    private double[] cumulativeProbabilities;
    private Random rnd;
    public WeightRandom(Pair[] options){
        this.options = options;
        this.rnd = new Random();
        prepare();
    }
    private void prepare(){		// 准备工作
        int weights = 0;
        for(Pair pair : options){
            weights += pair.getWeight();		// 获得总权重值
        }
        cumulativeProbabilities = new double[options.length];
        int sum = 0;
        for(int i = 0; i<options.length; i++) {
            sum += options[i].getWeight();
            cumulativeProbabilities[i] = sum / (double)weights;	// 获得每个选项的累计概率
        }
    }
    public Object nextItem(){
        double randomValue = rnd.nextDouble();	// 生成随机数
        // 进行二分查找
        int index = Arrays.binarySearch(cumulativeProbabilities, randomValue);
        if(index < 0) {		// 表示没找到
            index = -index-1;	// 得到索引值，就是对应的选项
        }
        return options[index].getItem();
    }
}
```

总结：Random类是线程安全的，多线程可以同时使用同一个Random对象。但是如果并发性很高，会产生竞争。

这时，可以考虑使用多线程库中的`ThreadLocalRandom`类。另外，Java类库中还有一个随机类`SecureRandom`，可以产生安全性更高、随机性更强的随机数，用于安全加密等领域。