# JAVA面试真题整理



但是，**可以用反射**， 可以反射出String对象中的value属性， 进而改变通过获得的value引用改变数组的结构。但是，一般不这么做

ps：用反射可以改变一个原来设定为不可变的对象：如果一个对象，他组合的其他对象的状态是可以改变的，那么这个对象很可能不是不可变对象。例如一个Car对象，它组合了一个Wheel对象，虽然这个Wheel对象声明成了private final 的，但是这个Wheel对象内部的状态可以改变， 那么就不能很好的保证Car对象不可变。

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