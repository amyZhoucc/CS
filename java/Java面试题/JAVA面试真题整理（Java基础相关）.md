# JAVA面试真题整理

## 1. 面向对象和面向过程

面向过程：

- 性能更高。对于性能关键的应用来说，单片机、嵌⼊式开发、Linux/Unix 等⼀般采用面向过程开发；
- 但是**没有面向对象易维护、易复用、易扩展**

面向对象：

- **易维护、易复用、易扩展**。因为面向对象有封装、继承、多态性的特性， 所以可以设计出**低耦合**的系统，使系统更加灵活、更加易于维护。
- 但是性能低，主要是因为类调用时需要实例化，开销比较大，资源消耗比较多

但是，Java性能差的原因，并不是面向对象的问题：

面向过程也需要分配内存，计算内存偏移量，Java性能差的主要原因是是**Java是半编译语言**（边解释边运行），最终的执行代码并不是可以直接被CPU执行的二进制机械码（虽然最终都需要是二进制代码才能被计算机所识别和执行）。——主要就是编程语言的**运行机制**决定的

——JAVA为了**一次代码，处处运行**（也就是可移植性），有了一层中间码也就是**字节码**，字节码的作用就是为了可移植性，字节码转换成机器码的工作就交给了java虚拟机，由java虚拟机解释成机器能够运行的二进制机器码。而C语言是直接编译为机器码，机器只需要直接运行二进制机器码即可，而无需和java一样在运行时解释为机器码。从这个角度看，性能与面向过程或对象是无关的。（JAVA只是为了移植性舍弃了一部分性能。）

## 3. == 与 equals

- ==，它的作⽤是判断两个对象的**地址是不是相等**，即判断两个变量是否指向同一个变量：对于基本数据类型就是值比较；对于引用类型就是指向的地址比较
- equals，需要**分类讨论**，它主要就是针对**类对象**，用来判断对象是否相等
  1. 如果类的equals未被重写，则通过 equals() ⽐较该类的两个对象时，等价于==
  2. 如果类的equals被重写了，那么看equals()的实现，而equals一般都是用来对对象的内容（全部 or 部分）进行比较的。如果内容相等就认为相等

```java
public class test1 {
    public static void main(String[] args) {
        String a = new String("ab"); // a为对象引用，堆里面开辟空间
        String b = new String("ab"); // b为另一个对象引用
        String aa = "ab"; // "ab"存放在常量池中的
        String bb = "ab"; // 直接从常量池中找，它们找到的是同一个，所以两个对象引用指向的对象是同一个
        if (aa == bb) // true
            System.out.println("aa==bb");
        if (a == b) // false，因为不是同一个对象
            System.out.println("a==b");
        if (a.equals(b)) // true
            System.out.println("aEQb");
        if (42 == 42.0) { // true
            System.out.println("true");
        }
    }
}
```

## 4. Java中的String为什么是不可变的

明确：**Java中的String类对象是不可变的**，即在它创建完成之后，不能再改变它的状态，即不能改变对象内的成员变量，包括基本数据类型的值不能变、引用类型的变量不能指向其他对象，而引用类型指向的对象的状态也不能变化。

首先，需要明白对象和对象引用的区别：

创建对象时，会在内存中分配一块给该对象，而对象引用就是指向该内存块的首地址的。所以，String对象不可以变化，String对象的引用是可以变化的。

看源码可以知道：**在Java中String类其实就是对字符数组的封装**：

```java
private final char value[];
```

这是一个字符数组，并且是 private final 类型，用于存储字符串内容。从 fianl 关键字可以看出，String 的内容一旦被初始化后，其不能被修改的。

——所以，String对象不能变化。

但是value是引用数据类型，而final修饰的是该value类型，说明该value只能指向该数组而不能指向其他数组了，但是我们还可以改变数组中的值？

——但是，由于是private类型的，且String类没有提供修改value数组的方法，所以根本不能够访问到这个value引用，更不能通过这个引用去修改数组。

但是，**可以用反射**， 可以反射出String对象中的value属性， 进而改变通过获得的value引用改变数组的结构。但是，一般不这么做

ps：用反射可以改变一个原来设定为不可变的对象：如果一个对象，他组合的其他对象的状态是可以改变的，那么这个对象很可能不是不可变对象。例如一个Car对象，它组合了一个Wheel对象，虽然这个Wheel对象声明成了private final 的，但是这个Wheel对象内部的状态可以改变， 那么就不能很好的保证Car对象不可变。

## 5. Java中只有值传递

```java
public class Main {
    public String name;
    public Main(String name){
        this.name = name;
    }
    private static void swap(Main m1, Main m2){
        Main temp = m1;
        m1 = m2;
        m2 = temp;
        System.out.println(m1.name);		// "li"
        System.out.println(m2.name);		// "wang"
    }
    public static void main(String[] args) {
        Main m1 = new Main("wang");
        Main m2 = new Main("li");
        swap(m1, m2);
        System.out.println(m1.name);		// "wang"
        System.out.println(m2.name);		// "li"
    }
}
```

——可以发现，即使传递的是引用类型的变量，还只是值传递，当函数返回的时候拷贝就不存在了。

- 一个方法不能修改一个基本数据类型的参数（即数值型或布尔型）。
- 一个方法可以改变一个对象参数的状态。
- 一个方法**不能让对象参数引用一个新的对象**。

```java
private static void swap(Main m1, Main m2){
    m1.name = "li";
    m2.name = "wang";
}
——这样就能改变，但是是改变了对象内部的数据，而对象本身没有变化
```



## 7. Java基本数据类型和包装类的计算

### 1. int和Integer：

先讲区别：

| 区别   | int              | Integer                        |
| ------ | ---------------- | ------------------------------ |
| 定义   | 基本数据类型     | 包装类（引用数据类型）         |
|        | 直接使用         | 需要实例化后，才能使用，即new  |
| 变量   | 变量存储的就是值 | 存储的是引用，指向对象的首地址 |
| 默认值 | 0                | null                           |

对于两个非 new 生成的 Integer 对象，进行比较时，如果两个变量的值在区间 -128 到 127 之间，则比较结果为 true，如果两个变量的值不在此区间，则比较结果为 false。通过打印出来的地址可以看出来，当不在指定区间范围时，实际上是两个不同的对象。 具体原因： Java 在编译 Integer i = 100 ;时，会翻译成为 Integer i = Integer.valueOf(100)。而 Java API 中对 Integer 类型的valueOf 的定义如下，对于-128 到 127 之间的数，会存储在缓存中，Integer i = 127 时，会直接从缓存中获取，下次再 Integer j = 127时，同样从缓存中取，而不会 new 个新对象。

基本数据类型 int 和它的包装类 Integer 比较时，Java 会自动拆包装为 int（将复合数据类型转化为基本数据类型），然后进行比较，实际上就变为两个 int 变量的比较。即使打印出来的地址不同，但是比较结果仍为 true，主要原因是因为不是通过比较内存地址进行判断的。

**非 new 生成的 Integer 变量指向的是 Java 常量池中的对象**，而 **new Integer()生成的变量指向堆中新建的对象**，两者在内存中的地址不同。

**`Java 的数学计算是在内存栈里操作的`**，Java 会对加数进行拆箱操作，然后比较的是基本类型

```java
Integer i1 = 125;
Integer i2 = 125;
Integer i3 = 0;
Integer i4 = new Integer(127);
Integer i5 = new Integer(127);
Integer i6 = new Integer(0);
System.out.println("i1==i2:\t" + (i1 == i2));			// true, 常量缓存原因，>127就false了
System.out.println("i1==i2+i3:\t" + (i1 == i2 + i3));	// true，拆箱的原因
System.out.println("i4==i5:\t" + (i4 == i5));			// false，对象创建原因
System.out.println("i4==i5+i6:\t" + (i4 == i5 + i6));	// true，拆箱原因

i3 = 5;
Integer i7 = 130;
System.out.println("i7==i2+i3:\t" + (i7 == i2 + i3));	// true：拆箱的原因
```

String类型：

String 类中的 intern()方法会从常量池中查找是否存在这样的值，如果存在则直接返回，不存在则往常量池中插入一个新的这样的值，然后返回

```java
String s2 = new String("abc");
String s3 = s2.intern();		// 会在常量池中创建一个常量"abc"，且s3指向的就是该常量池中的数据
String s1 = "abc";	//由于此时常量池中已经有”abc“常量，所以s1直接指向”abc“
System.out.println(s1 == s3);		// true
```

string拼接：

```java
String s = "abc";
//第一种情况
String s2 = "ab";
String s4 = s2 + "c";
String s6 = new String("ab");
String s7 = s6 + "c";
System.out.println("s == s4:"+(s == s4));		// false
System.out.println("s == s7:"+(s == s7));		// false

//第二种
final String s1 = "ab";
String s3 = s1 + "c";
String s5 = "ab" + "c";
System.out.println("s == s3:"+(s == s3));			// true
System.out.println("s == s5:"+(s == s5));			// true

//第三种
final String s8 = getData();
String s9 = s8 + "c";
System.out.println("s == s9:"+(s == s9));		// false
```

情况 1，JVM 对于字符串引用，由于在字符串的"+"连接中，有字符串引用存在，而**引用的值在程序编译期是无法确定的**，即 `s2+"c" 或 s6+"c"`无法被编译器优化，只有在程序运行期来动态分配并将连接后的新地址赋给 s4 和 s7。所以上面程序的结果也就为 false。

——所以，如果将`s4 = s2+"c"`变成`s4 = "ab" + "c"`，那么就是true

情况 2，和 1 唯一不同的是 s1 字符串加了 final 修饰，对于 final 修饰的变量，它在**编译时被解析为常量值**的一个本地拷贝存储到自己的常量池中或嵌入到它的字节码流中。所以此时 s1+"c" 和 "ab"+"c" 效果是一样的，故上面的结果为 true。

情况 3，JVM 对于字符串引用 s8，它的值在编译期无法确定，只有在程序运行期调用方法后，将方法的返回值和”c“来动态连接并分配地址为 s9，故上面程序的结果为 false。

```java
public class AAA {

    public static void main(String[] args) {
        // TODO Auto-generated method stub

        String hello = "Hello", lo = "lo";
        System.out.print((hello == "Hello") + " ");		// true，常量对常量
        System.out.print((Other.hello == hello) + " ");	// true，同上
        System.out.print((other.Other.hello == hello) + " ");	// true，同上
        System.out.print((hello == ("Hel" + "lo")) + " ");		// true，常量拼接
        System.out.print((hello == ("Hel" + lo)) + " ");		// false，包含了引用对象
        System.out.println(hello == ("Hel" + lo).intern());		//true，获得是指向常量池的对象
    }
}

class Other {
    static String hello = "Hello";
}

package other;

public class Other { 
    static String hello = "Hello";
}
```

在给 s1 赋值的时候去常量池中查找，第一次初始化的常量池为空的，所以是没有的，则在字符串常量池中开辟一块空间，存放 String 常量"abc"，并把引用返回给 s1,当 s2 也是这样的过程，在常量池中找到了，所以 s1 和 s2 指向相同的引用，即 s1==s2 和 s1.equals(s2)都为 true。