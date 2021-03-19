# 包装类源码

## 1. 类头

首先看一下包装类的类头（瞎编的名字）

```java
// 可以看到包装类都是final的（不可继承），继承的是抽象父类Number，并且实现了Comparable接口
public final class Byte extends Number implements Comparable<Byte> {}
```

final：不可被其他类给继承

extends：继承自Number，是一个抽象父类

implements：实现了Comparable接口，有Comparable功能。

### extends：abstract class Number

除了Boolean、Character没有继承，其余的6个包装类均继承了

总结：Number类**定义了包装类对象如何转换成7种基本数据类型的抽象方法**，eg：**当前对象**转换成int值如何表示、转换成double如何表示....

```java
// 除了Boolean没有继承，其余均继承了
// 总结：Number类定义了基本数据类型相互转换的抽象方法，而作为子类每个包装类都需要实现，那么实现了基本数据类型之间的值相互转换
// Number是基本数据类型的超类，可转换为byte、double、float、int、long、short
public abstract class Number implements java.io.Serializable {
    
    // 将当前对象的值转换为int值表示
    public abstract int intValue();

    // 将当前对象的值用long表示
    public abstract long longValue();

    // 用float表示
    public abstract float floatValue();

    // 用double表示
    public abstract double doubleValue();

    // 用byte表示，可能需要舍入或者截断
    public byte byteValue() {
        return (byte)intValue();
    }

    // 用short表示，可能需要舍入or截断
    public short shortValue() {
        return (short)intValue();
    }

    /** 使用JDK 1.0.2中的serialVersionUID进行相互操作 */
    private static final long serialVersionUID = -8742448824652078965L;
}
```

### implements：Comparable

Comparable有且只有一个方法待实现，表示实现该接口的类都有**比较功能**

```java
public interface Comparable<T> {    
    public int compareTo(T o);	
}
```

**特殊的**：

没有继承Number，但是增加实现了java.io.Serializable接口

```java
// Boolean的类头
// 没有继承Number，但是增加实现了java.io.Serializable接口
public final class Boolean implements java.io.Serializable, Comparable<Boolean>{}

// Character的类头
public final class Character implements java.io.Serializable, Comparable<Character> {}
```

## 2. 静态final变量

没有列全，大部分常见的都包括了

```java
// Boolean的常量
public static final Boolean TRUE = new Boolean(true);	// 静态final对象
public static final Boolean FALSE = new Boolean(false);
```

```java
// Byte的常量
public static final int SIZE = 8;		// 最大的二进制位数
public static final int BYTES = SIZE / Byte.SIZE;	// 字节数——1字节
public static final byte   MIN_VALUE = -128;
public static final byte   MAX_VALUE = 127;
```

```java
// Short的常量
public static final int SIZE = 16;	// 最大的二进制位数
public static final int BYTES = SIZE / Byte.SIZE;	// 16/8 字节数——2字节
public static final short   MIN_VALUE = -32768;	// 最小表示范围
public static final short   MAX_VALUE = 32767;	// 最大表示范围
```

```java
// Integer的常量
@Native public static final int SIZE = 32;	// 二进制最大的位数
public static final int BYTES = SIZE / Byte.SIZE;	// 字节数——4字节
@Native public static final int   MIN_VALUE = 0x80000000;	// 最小值（带符号数，补码表示的）
@Native public static final int   MAX_VALUE = 0x7fffffff;	// 最大值（带符号数，最高位表示正负）
```

```java
// Character的常量（还有其他字符表示的相关）
public static final char MIN_VALUE = '\u0000';	// 最小值（0）
public static final char MAX_VALUE = '\uFFFF';	// 最大值（65535），按Unicode字符形式
// 表示的最小和最大进制范围，2~36进制
public static final int MAX_RADIX = 36;
public static final int MIN_RADIX = 2;
```

```java
// Long的常量
@Native public static final int SIZE = 64;
public static final int BYTES = SIZE / Byte.SIZE;	// 字节数——8字节
@Native public static final long MIN_VALUE = 0x8000000000000000L;
@Native public static final long MAX_VALUE = 0x7fffffffffffffffL;
```

float/double稍微多的：包含正无穷、负无穷、NaN数

正无穷之间是相等的——一个正数/0.0；

负无穷之间是相等的——一个负数/0.0;

NaN之间不相等——0.0/0.0，而有一个特别的`NaN != NaN  -- true`，即判断两个NaN不能做任何比较判断，返回值均为false；只有判断不等的时候，才会使true

0x7f800000：0 11111111 000....——正好表示的是符号位为0，表示正数；指数位为8，0xff，而尾数位全为0

0xff800000：1 11111111 000....——同上的含义

当参数为`0x7f800001 ~ 0x7fffffff`或者`0xff800001 ~ 0xffffffff`之间时，结果为`NaN`

<img src="../pic/float.jpg">

```java
// Float（double类似不关注啦）
public static final int SIZE = 32;
public static final int BYTES = SIZE / Byte.SIZE;
// 正无穷大 = Float.intBitsToFloat(0x7f800000)
public static final float POSITIVE_INFINITY = 1.0f / 0.0f;	
// 负无穷大 = Float.intBitsToFloat(0xff800000)
public static final float NEGATIVE_INFINITY = -1.0f / 0.0f;	
// NaN数 = Float.intBitsToFloat(0x7fc00000)——not a number
public static final float NaN = 0.0f / 0.0f;
public static final float MAX_VALUE = 0x1.fffffeP+127f; // 3.4028235e+38f
public static final float MIN_NORMAL = 0x1.0p-126f; // 1.17549435E-38f
public static final float MIN_VALUE = 0x0.000002P-126f; // 1.4e-45f
public static final int MAX_EXPONENT = 127;	// e的最大值
public static final int MIN_EXPONENT = -126;// e的最小值
```

## 3. 方法归纳

### (1) toString()：来自于Object超类

该方法存在同名的静态方法和实例方法：做区分（静态方法需要传参，实例方法不需要传参）

- 两者的参数不同：静态方法是传入的基本数据类型，实例方法是this.value
- 两者目的不同：静态方法是获取基本数据类型数据的string表示，实例方法是获取包装类对象的string表示

该方法的主要目的是：**对该对象直观的文本表示**，所以需要简要的表达出该对象的信息

**返回的是字符串格式**，建议所有的类都重写该方法

```java
// Object类
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

分析：对于超类，默认的方法就是获得该实例对象的类名 + “@” + 实例对象的地址十六进制表示的哈希值

#### Boolean的：

静态方法：

```java
public static String toString(boolean b) {	// 通过Boolean.toString()调用
	return b ? "true" : "false";
}
```

实例方法：

```java
public String toString() {			// 通过实例变量调用
	return value ? "true" : "false";
}
```

分析：对于boolean，只有两种结果：true/false，那么只需要根据值直接转换即可

#### Integer的：⭐

为加快计算，设置的常量：——辅助数组

```java
// 下面第4个方法使用到了
final static char[] digits = {
    '0' , '1' , '2' , '3' , '4' , '5' ,
    '6' , '7' , '8' , '9' , 'a' , 'b' ,
    'c' , 'd' , 'e' , 'f' , 'g' , 'h' ,
    'i' , 'j' , 'k' , 'l' , 'm' , 'n' ,
    'o' , 'p' , 'q' , 'r' , 's' , 't' ,
    'u' , 'v' , 'w' , 'x' , 'y' , 'z'
};

// 4使用
// 10*10，获得二位数的十位，eg：65-6，62-6，所以每一行都一样
final static char [] DigitTens = {
    '0', '0', '0', '0', '0', '0', '0', '0', '0', '0',
    '1', '1', '1', '1', '1', '1', '1', '1', '1', '1',
    '2', '2', '2', '2', '2', '2', '2', '2', '2', '2',
    '3', '3', '3', '3', '3', '3', '3', '3', '3', '3',
    '4', '4', '4', '4', '4', '4', '4', '4', '4', '4',
    '5', '5', '5', '5', '5', '5', '5', '5', '5', '5',
    '6', '6', '6', '6', '6', '6', '6', '6', '6', '6',
    '7', '7', '7', '7', '7', '7', '7', '7', '7', '7',
    '8', '8', '8', '8', '8', '8', '8', '8', '8', '8',
    '9', '9', '9', '9', '9', '9', '9', '9', '9', '9',
} ;
// 10*10，获得二位数的个位：eg：65-5，45-5，所以每一列都一样
final static char [] DigitOnes = {
    '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
    '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
    '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
    '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
    '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
    '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
    '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
    '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
    '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
    '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
} ;

// 判断数字有几位的方法：直接与对应位数的最大值比较，对应的索引就是位数——3使用
final static int [] sizeTable = { 9, 99, 999, 9999, 99999, 999999, 9999999,
                                      99999999, 999999999, Integer.MAX_VALUE };
```

函数实现：

1. `toString()`：实例方法，转而去调用的就是静态方法

   ```java
   // 最上层的调用：
   public String toString() {
   	return toString(value);		// 调用2
   }
   ```

2. `toString(int i)`：静态方法

   默认转换为10进制

   ```java
   // 本质实现：
   public static String toString(int i) {
   	if (i == Integer.MIN_VALUE)	// 如果和最小值一样：0x80000000 = -2147483648
   		return "-2147483648";		// 直接返回
       // 如果该数是负数，转成正数再判断位数，且最后要加1个长度存放符号
   	int size = (i < 0) ? stringSize(-i) + 1 : stringSize(i);	// 判断位数，调用3
   	char[] buf = new char[size];	// 新建一个确定大小的字符数组
   	getChars(i, size, buf);		// 调用4
   	return new String(buf, true);		// 创建一个string对象，将数组传入；true表示是share允许
   }
   ```

3. `stringSize(int x)`

   ```java
   static int stringSize(int x) {
       for (int i=0; ; i++)	// 上面数组进行匹配即可
           if (x <= sizeTable[i])
               return i+1;
   }
   ```

4. `getChars(int i, int index, char[] buf)`

   概括：当数字>=65536时，从最低位开始，每2位进行提取，利用移位关系，提取出最低2位：64+32+4

   ​		当数字 <65536时，从最低位提取每一位，利用52429和>>19的关系，实现i/10；并且利用移位，提取出最低位：8+2

   ps：移位的效率 > 乘法的效率 > 除法的效率

   ```java
   // 将int数字的每一位存储到字符数组中，从最低位计算并存储（所以是倒着来的）
   // i：数字；index：数字长度；buf：存放的字符数组
   static void getChars(int i, int index, char[] buf) {
       int q, r;
       int charPos = index;
       char sign = 0;
   
       if (i < 0) {		// 如果数字是负数，需要添加负号，且将其转换成正数
           sign = '-';
           i = -i;
       }
   
       // 如果i超过65536，每次计算获得两位的char值
       while (i >= 65536) {
           q = i / 100;		// 去掉当前最低的2位
           // really: r = i - (q * 100);
           // 用移位操作快速得到q*100：q*2^6+q*2^5+q*2^2 = q*64+q*32+q*4=q*100
           // r获得是最低两位
           r = i - ((q << 6) + (q << 5) + (q << 2));	// 获得最低的两位
           i = q;
           buf [--charPos] = DigitOnes[r];	// 当前最低位
           buf [--charPos] = DigitTens[r];	// 当前倒数第二位
       }
   	
       //why要先将大小降到65536以内呢？
       // 为了防止65536以上的数*52429太大溢出
       // Fall thru to fast mode for smaller numbers
       // assert(i <= 65536, i);
       // 通过位运算加快运行速度
       for (;;) {
           //  (double)52429/(1<<19)=0.10000038146972656，实际上就是i/10
           q = (i * 52429) >>> (16+3); 
           // q<<3+q<<1=q*8+q*2=q*10,r = i-(q*10) 
           r = i - ((q << 3) + (q << 1));  // r得到的就是当前的最低位
           buf [--charPos] = digits [r];
           i = q;
           if (i == 0) break;
       }
       if (sign != 0) {
           buf [--charPos] = sign;
       }
   }
   ```

   => 妙啊:clap:

5. `toString(int i, int radix)`

   另一个toString方法：（加上进制要求，效率要比10进制的转换效率要低）

   注意的要点：统一转换成负数然后计算，主要是：负数比正数能多表示一个数

   ```java
   // 需要设定转换的进制
   public static String toString(int i, int radix) {
       // 限定进制在2~36范围内（能表示的最大数字26+10），如果超出，默认为10进制
       if (radix < Character.MIN_RADIX || radix > Character.MAX_RADIX)
           radix = 10;
   
       /* 如果选择10进制，就用更快的模式 */
       if (radix == 10) {
           return toString(i);
       }
   
       char buf[] = new char[33];// 对于int类型，最大32位（二进制），不会有比这更大的了其他进制位数更少，再+符号位
       boolean negative = (i < 0);		// 正负的标志记录：负数——true；正数——false
       int charPos = 32;			// 从字符数组的最后一个开始存放
   
       if (!negative) {
           i = -i;		// 保证数字都是负数——why：为了防止溢出-128->128，就会溢出（随便设置的），而上面方法转为正数，是因为提前做过判断，处理了最小负数问题，所以不会溢出——总结就是：负数比正数能多表示一个数
       }
   
       while (i <= -radix) {
           // 从低位开始获取每一位，加入到字符数组中
           buf[charPos--] = digits[-(i % radix)];
           i = i / radix;
       }
       buf[charPos] = digits[-i];	// 将最高位获取
   
       if (negative) {
           buf[--charPos] = '-';	// 负数添负号
       }
   	// 从存放数开始算起，前面的都不算
       return new String(buf, charPos, (33 - charPos));
   }
   ```

#### Byte、Short的：

实际上是调用了Integer的实现方法：

```java
// Byte包装类——本质上去调用Integer实现的方法
public static String toString(byte b) {
	return Integer.toString((int)b, 10);		// radix=10
}
public String toString() {			// 实例方法
	return Integer.toString((int)value);	
}

// Short包装类——本质上去调用Integer实现的方法
public static String toString(short s) {		// 静态方法
	return Integer.toString((int)s, 10);	// radix=10进制
}
public String toString() {			// 实例方法
	return Integer.toString((int)value);
}
```

#### Character的:

是利用String的构造方法直接调用String.valueOf

```java
// Character包装类
public static String toString(char c) {
	return String.valueOf(c);		// 主要利用String类将字符转换为字符串，静态方法调静态方法
}
public String toString() {
	char buf[] = {value};		
	return String.valueOf(buf);
}
public final String toString() {	// 主要是给subset使用的（未求证？？？）
	return name;		
}

// String类的valueOf方法之一：
public static String valueOf(char c) {			// 静态方法
	char data[] = {c};		// 将一个字符存放到字符数组中
	return new String(data, true);	// 将字符存放到实例变量中，返回this就是String类型的
}
// String类的valueOf方法之二：
public static String valueOf(char data[]) {
	return new String(data);	// 相比上面省去第一步
}
```

——未研究两个toString方法为啥不调用同一个String.valueOf？？？？

#### Long的：

```java
public static String toString(long i) {	// 默认转换为10进制
    if (i == Long.MIN_VALUE)		// 处理最小值情况（防止后面的取正数溢出）
        return "-9223372036854775808";
    int size = (i < 0) ? stringSize(-i) + 1 : stringSize(i);	// 获得字符串长度
    char[] buf = new char[size];
    getChars(i, size, buf);
    return new String(buf, true);
}

static void getChars(long i, int index, char[] buf) {
    long q;
    int r;
    int charPos = index;
    char sign = 0;

    if (i < 0) {
        sign = '-';
        i = -i;
    }

    // 在超过integer的最大值范围内，每次获取最低两位的值
    while (i > Integer.MAX_VALUE) {
        q = i / 100;
        // really: r = i - (q * 100);
        // 需要强转为int类型，取低位值，所以不影响
        r = (int)(i - ((q << 6) + (q << 5) + (q << 2)));
        i = q;
        buf[--charPos] = Integer.DigitOnes[r];
        buf[--charPos] = Integer.DigitTens[r];
    }

    // 在integer表示范围后，和integer的实现方式一样
    int q2;
    int i2 = (int)i;
    while (i2 >= 65536) {
        q2 = i2 / 100;
        // really: r = i2 - (q * 100);
        r = i2 - ((q2 << 6) + (q2 << 5) + (q2 << 2));
        i2 = q2;
        buf[--charPos] = Integer.DigitOnes[r];
        buf[--charPos] = Integer.DigitTens[r];
    }

    // Fall thru to fast mode for smaller numbers
    // assert(i2 <= 65536, i2);
    for (;;) {
        q2 = (i2 * 52429) >>> (16+3);
        r = i2 - ((q2 << 3) + (q2 << 1));  // r = i2-(q2*10) ...
        buf[--charPos] = Integer.digits[r];
        i2 = q2;
        if (i2 == 0) break;
    }
    if (sign != 0) {
        buf[--charPos] = sign;
    }
}

// 本质上和Integer的实现一样
public static String toString(long i, int radix) {	// 带进制选择
    if (radix < Character.MIN_RADIX || radix > Character.MAX_RADIX)
        radix = 10;
    if (radix == 10)
        return toString(i);
    char[] buf = new char[65];
    int charPos = 64;
    boolean negative = (i < 0);

    if (!negative) {
        i = -i;
    }

    while (i <= -radix) {
        buf[charPos--] = Integer.digits[(int)(-(i % radix))];
        i = i / radix;
    }
    buf[charPos] = Integer.digits[(int)(-i)];

    if (negative) {
        buf[--charPos] = '-';
    }

    return new String(buf, charPos, (65 - charPos));
}
```

#### Float和Double的：

=> 太过于复杂了（没这本事）

```java
public static String toString(float f) {
    return FloatingDecimal.toJavaFormatString(f);
}
```

### (2) equals()和hashCode()

equals和hashcode关系密切，需要搭配重写：

- Object的equals方法默认是比较地址，只有指向的地址一样才返回true，等同于`==`
- hashCode的默认实现：将对象的地址经过哈希运算返回int值
- **hashCode和equals方法密切**：两个对象，如果**equals返回true（表示同一个对象），那么hashCode也必须返回一样**；如果equal返回false，那么hashCode尽量不一样

#### equals重写：

#### Integer的：（其余包装类同理）

**equals的参数是是Object类型**

思路：先判断是否类型一致 -> 强转类型之后 -> 通过`xxValue()`取出value的值，进行比较

```java
public boolean equals(Object obj) {
	if (obj instanceof Integer) {		// 首先，判断传入的对象是否是Integer类型
		return value == ((Integer)obj).intValue();	// 然后转换成基本类型值再进行比较
	}
	return false;
}
// =>其他几个包装类同理:Boolean、Byte、Short、Character、Long均同样的实现
```

#### Float/Double的：

思路：先判断是否类型一致 -> 强转类型之后 -> 转换成二进制int类型，进行比较

```java
// Float/Double有不同：将Float转换成float，float转换成二进制int成然后进行比较，只有两个值完全一样时才算相等（因为小数计算不精确，数学计算相等但是计算机运行结果不同）
public boolean equals(Object obj) {
        return (obj instanceof Float)
               && (floatToIntBits(((Float)obj).value) == floatToIntBits(value));
}
// Double和Float类似，将Double转换成double，double转换成二进制long，然后按long比较
public boolean equals(Object obj) {
	return (obj instanceof Double)
			&& (doubleToLongBits(((Double)obj).value) == doubleToLongBits(value));
}
```

#### hashCode重写:

要求：

- **一个对象的哈希值不能变**

- **相同对象的哈希值必须一样，不同对象的哈希值一般应该不同（不强制不同）**

  eg：那出生日期作为一个班上学生的hashCode，存在不同对象的哈希值一样的情况，但是问题不大

#### Boolean的：

```java
// boolean
public static int hashCode(boolean value) {
	return value ? 1231 : 1237;	// 返回两个质数之一——使用哈希时，不容易冲突（数学原因）
}
```

why用1231、1237，见下面

#### Integer的：（Byte、Short同）

直接获得对应的value即可

```java
// byte/char/short/int同样的实现方式：——直接返回其值（那么如果它们值一样，那么hashCode也是一样的），符合逻辑
public static int hashCode(byte value) {
	return (int)value;
}
```

#### Long的：

```java
// long：高32位和低32位做异或——因为要求返回的是int类型
public static int hashCode(long value) {
	return (int)(value ^ (value >>> 32));		// 逻辑右移（空位补0）
}
```

#### Float和Double的：

```java
// float：直接转换成二进制int表示（就是equals的操作）
public static int hashCode(float value) {
	return floatToIntBits(value);
}

// double：先转换成二进制long表示，然后逻辑右移高低位异或操作
public static int hashCode(double value) {
	long bits = doubleToLongBits(value);
	return (int)(bits ^ (bits >>> 32));
}
```

### (3) compareTo()——实现比较功能

返回值是：int类型的 ，>0表示当前对象的值大；<表示传递的对象值大；=0，表示值一样大

#### Boolean的：

```java
// Boolean的实现
public int compareTo(Boolean b) {		// 本质上去调用compare方法
	return compare(this.value, b.value);
}

public static int compare(boolean x, boolean y) {
    return (x == y) ? 0 : (x ? 1 : -1);	// 如果相等返回0；不相等，则判断如果前一个为true，返回1；false，返回-1
}
```

#### Byte/Short/Character的：

做减法运算得到

```java
// Byte的实现：获取其值，然后减法运算
public int compareTo(Byte anotherByte) {
    return compare(this.value, anotherByte.value);
}

public static int compare(byte x, byte y) {
    return x - y;
}
=> Short、Character同理
```

#### Integer/Long的:

比较运算得到

```java
// Integer的实现
public static int compare(int x, int y) {
    return (x < y) ? -1 : ((x == y) ? 0 : 1);	// 直接通过运算符号得到，而不是-
}
=> Long同理
```

#### 浮点数的：

原理：先进行比较操作（排除掉NaN），后将浮点数转换成二进制的整型数，然后进行比较

```java
public int compareTo(Float anotherFloat) {
    return Float.compare(value, anotherFloat.value);
}

// 多操作，主要是为了避免两个NaN或者两个0（-0，0）进行比较
public static int compare(float f1, float f2) {
    if (f1 < f2)
        return -1;           // Neither val is NaN, thisVal is smaller
    if (f1 > f2)
        return 1;            // Neither val is NaN, thisVal is larger

    // Cannot use floatToRawIntBits because of possibility of NaNs.
    int thisBits    = Float.floatToIntBits(f1);
    int anotherBits = Float.floatToIntBits(f2);

    return (thisBits == anotherBits ?  0 : // Values are equal
            (thisBits < anotherBits ? -1 : // (-0.0, 0.0) or (!NaN, NaN)
             1));                          // (0.0, -0.0) or (NaN, !NaN)
}

// double同理操作
public static int compare(double d1, double d2) {
    if (d1 < d2)
        return -1;           // Neither val is NaN, thisVal is smaller
    if (d1 > d2)
        return 1;            // Neither val is NaN, thisVal is larger

    // Cannot use doubleToRawLongBits because of possibility of NaNs.
    long thisBits    = Double.doubleToLongBits(d1);
    long anotherBits = Double.doubleToLongBits(d2);

    return (thisBits == anotherBits ?  0 : // Values are equal
            (thisBits < anotherBits ? -1 : // (-0.0, 0.0) or (!NaN, NaN)
             1));                          // (0.0, -0.0) or (NaN, !NaN)
}
```

### (4) 静态方法valueOf() & 实例方法xxxValue()

两个方法对应的是：装箱（将基本数据类型转换为包装类对象，返回对象）、拆箱（将当前对象转换为基本数据类型）

#### valueOf()

valueOf()存在多个重载方法，可以接受传参是String：

原理是：将s先用parseInt转换成int基本数据类型，然后再进行转换

```java
public static Integer valueOf(String s) throws NumberFormatException {
    return Integer.valueOf(parseInt(s, 10));
}
```

由于：`parseInt()`，可能会抛出异常（输入的string里面存在非数字的内容，就转换失败了；

常规的valueOf方法：将int类型的参数，转换成Integer对象：

##### Integer的：⭐

思路：如果调用valueOf创建对象时，**数字在-128~127之间的，就直接返回cache池中创建好的默认的对象（所以同值的这些引用对象的地址都一样）**；超出范围的才进行创建

通过共享常用对象来节省空间，并且由于Integer的不可变性，所以缓存的对象是安全的

```java
// 将int值转换成Integer包装类：如果值在-128~127范围内，那么就在cache池中找到该引用；如果不存在那么就新建一个包装类对象
public static Integer valueOf(int i) {
	if (i >= IntegerCache.low && i <= IntegerCache.high)
		return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}

// 静态私有类，是Integer的缓存
// 在内部的静态初始化代码块中被初始化，默认存储了-128~127范围内的整数对象
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}
```

ps：可以通过`-XX:AutoBoxCacheMax=<size>`选项**自行配置缓存最大值**，但是必须要大于等于127

##### Byte/Short/Character的：

实现和Integer，都存在缓存，缓存范围都是-128~127

那么Byte所有的对象都缓存在cache里面了，除非是用new，就会新建一个对象，其余通过valueOf得到的对象都在cache里面拿的

Character是缓存了0~127，没有负数

##### Float/Double的：

它们不存在缓存，都是直接创建Float/Double对象即可

```java
public static Float valueOf(float f) {
    return new Float(f);
}
```

Double同理

#### xxxValue()：

可以根据xxx，返回指定的拆箱后的类型：

##### intValue()：（其余整型类似）

```java
// Integer中的，其余类中可能需要强转
public int intValue() {
    return value;
}

public byte byteValue() {
    return (byte)value;		// 强制类型转换，可能会溢出（循环计算，即取余）
}
```

### (5) 实例方法parseXXX()

`parseXXX()`（xxx就是对应的包装类名字）：主要就是将**字符串表示的内容转换成对应的包装类**

注意，parseXXX()，返回的是基本数据类型，而不是对应的包装类

#### Boolean的：

parseBoolean，输入的是true

```java
// Boolean
public static boolean parseBoolean(String s) {
    // 判断是否是null
    // 非null的情况下，判断当前字符串和"true"是否一样——true是忽略大小写的，即True、TRUE也都是可以得到true，否则是false
    return ((s != null) && s.equalsIgnoreCase("true"));
}

// String.java——不区分大小写的比较大小
public boolean equalsIgnoreCase(String anotherString) {
    return (this == anotherString) ? true			// 是否指向同一个对象
        : (anotherString != null)			// 看另一个字符串是否为空
    && (anotherString.value.length == value.length)	// 看两个字符串的长度
        && regionMatches(true, 0, anotherString, 0, value.length);// 从头开始进行匹配
}

// 通用的字符串匹配方法——可设置判断时是否需要忽略大小写，如果忽略大小写，也不是全部转换成大/小写后比较，而是通过判断两次
public boolean regionMatches(boolean ignoreCase, int toffset,
            String other, int ooffset, int len) {
    char ta[] = value;
    int to = toffset;
    char pa[] = other.value;
    int po = ooffset;
    // Note: toffset, ooffset, or len might be near -1>>>1.
    if ((ooffset < 0) || (toffset < 0)	// 存在一个字符串为空，或者要比较的长度不一样，直接false
        || (toffset > (long)value.length - len)
        || (ooffset > (long)other.value.length - len)) {
        return false;
    }
    while (len-- > 0) {
        char c1 = ta[to++];
        char c2 = pa[po++];
        if (c1 == c2) {		// 相等就继续
            continue;
        }
        if (ignoreCase) {	// 如果不相等，且设置了忽略大小写，那么将该字符转换成大写，然后再次比较
            char u1 = Character.toUpperCase(c1);
            char u2 = Character.toUpperCase(c2);
            if (u1 == u2) {
                continue;
            }
            // 这个是针对Georgian alpha，再次转换成小写字母比较
            if (Character.toLowerCase(u1) == Character.toLowerCase(u2)) {
                continue;
            }
        }
        return false;
    }
    return true;
}
```

#### Integer/Long的：

总体：代码逻辑上通过计算负数，然后结果判断符号位的逻辑进行运算，这样子可以避免正负逻辑分开处理

```java
// 本质上还是调用下面带指定进制的方法
public static int parseInt(String s) throws NumberFormatException {
    return parseInt(s,10);
}

// 可以设定转换的进制
public static int parseInt(String s, int radix)
                throws NumberFormatException
{
    /**这个方法在VM初始化时可能会早于 IntegerCache 的初始化过程，所以在这里不能使用valueOf，而在valuOf里面可以用parseInt**/
    if (s == null) {	// 如果字符串为空，那么抛出异常
        throw new NumberFormatException("null");
    }

    if (radix < Character.MIN_RADIX) {	// 如果指定的进制比2还小，抛出异常
        throw new NumberFormatException("radix " + radix +
                                        " less than Character.MIN_RADIX");
    }

    if (radix > Character.MAX_RADIX) {	// 指定进制超过最大的进制36，抛出异常
        throw new NumberFormatException("radix " + radix +
                                        " greater than Character.MAX_RADIX");
    }

    int result = 0;
    boolean negative = false;
    int i = 0, len = s.length();
    // 是一个负值，如果输入的是正数，那么limit=最大值的负数；如果输入的是负数，limit=最小值
    int limit = -Integer.MAX_VALUE;		
    int multmin;
    int digit;

    if (len > 0) {
        char firstChar = s.charAt(0);
        if (firstChar < '0') { // 如果字符串第一个不是数字，可能是'+'/'-'（ASCII中比0小）
            if (firstChar == '-') {		// 如果首字符是-，设置标志位
                negative = true;
                limit = Integer.MIN_VALUE;		// 并且设置下界范围
            } else if (firstChar != '+')	// 如果首字符是+，抛出异常
                throw NumberFormatException.forInputString(s);

            if (len == 1) // 如果字符串第一个不是数字，且只有一个符号的长度，抛出异常
                throw NumberFormatException.forInputString(s);
            i++;
        }
        multmin = limit / radix;		// 最值/10，某时刻的值不能超过最值/10，否则res*10就溢出了(假设radix=10)
        while (i < len) {
            // 返回对应字符对应进制的数字值，如果输入进制不在范围内或者字符无效，返回-1
            digit = Character.digit(s.charAt(i++),radix);
            if (digit < 0) {			// -1表示输入有问题，抛出异常
                throw NumberFormatException.forInputString(s);
            }
            if (result < multmin) {		// 判断未增加之前的结果值是否溢出
                throw NumberFormatException.forInputString(s);
            }
            result *= radix;		//已经判断：res>= limit/10，那么res*10，不会溢出
            // 判断加上数字后是否溢出，eg：下界是-2147483647，/10=-214748364，而如果之前的res = -214748364，那么加上digit之后，也有可能越界也可能不越界的，所以还需要判断
            if (result < limit + digit) {		// 本质上是res-digit < limit，看是否会溢出，但是如果溢出了无法判断，所以需要移项，limit是负数，负数+digit，是不会越界的
                throw NumberFormatException.forInputString(s);
            }
            result -= digit;			// 存放的是负数形式
        }
    } else {	// 字符串长度=0，抛出异常
        throw NumberFormatException.forInputString(s);
    }
    return negative ? result : -result;		// 根据标志位看是否需要转换
}
```

Long本质上解法一致。

#### Short/Byte的：

总结：本质上还是调用了Integer的parseInt方法

```java
// String是直接转换成Intger（十进制），然后在强制类型转换得到的
// 可以限定转换的进制
// 由于调用的方法会抛出异常，所以这边需要声明该异常
public static short parseShort(String s) throws NumberFormatException {
    return parseShort(s, 10);
}

public static short parseShort(String s, int radix)
    throws NumberFormatException {		
    int i = Integer.parseInt(s, radix);
    if (i < MIN_VALUE || i > MAX_VALUE)
        throw new NumberFormatException(	// 如果表示的数超过short表示的范围，就抛出异常
        "Value out of range. Value:\"" + s + "\" Radix:" + radix);
    return (short)i;
}
```

## 4. Integer中的二进制相关

### 1. 位翻转

Integer提供两个静态方法，用来进行位翻转

（1）reverse：按位进行互换

本质上使用的是：**归并的思想**。先两位之间交换相邻的位数（两位之间的相对顺序已经确定了），然后以两位一组，交换相邻的位数（4位内部的交换），然后以四位一组，交换相邻的位数（8位内部的交换），然后以8位一组，交换相邻的位数（16位内部的交换），最后以16位一组，交换相邻的位数

eg：拿十进制举例：12 34 56 78 -> 21 43 65 78（2位内部的相对顺序已经确定了）

-> 43 21 78 65（4位内部的相对顺序已经确定）-> 78 65 43 21 

```java
public static int reverse(int i) {
    // HD, Figure 7-1——引用《高效算法的奥秘》书中的代码
    
    i = (i & 0x55555555) << 1 | (i >>> 1) & 0x55555555;// 相邻位交换（2位内有序）
    i = (i & 0x33333333) << 2 | (i >>> 2) & 0x33333333;// 2位一组交换（4位内有序）
    i = (i & 0x0f0f0f0f) << 4 | (i >>> 4) & 0x0f0f0f0f;// 4位一组交换（8位内有序）
    i = (i << 24) | ((i & 0xff00) << 8) |// 8位一组交换，那么就是字节翻转（和下面的函数实现基本一致）
        ((i >>> 8) & 0xff00) | (i >>> 24);
    return i;
}
```

分析：

1. 首先了解：`0x55555555`的二进制表示是：`0101 0101 0101 0101 0101 0101 0101 0101`，`x&0x55555555`取的是所有奇数位的值；`0xAAAAAAAA`的二进制表示：`1010 1010 1010 1010 1010 1010 1010 1010`，`x&0xAAAAAAAA`取得是所有偶数位的值

   那么：**`(x & 0x55555555) << 1 | (x & 0xAAAAAAAA) >>> 1;`**实现了相邻位交换：就是取出奇数位并向左移动1位（到了偶数位上）；取出偶数位并向右移动1位（到了奇数位上），然后或起来——那么相邻位就已经互换了

   稍微优化一下可以只用一个常量：`(i & 0x55555555) << 1 | (i >>> 1) & 0x55555555;`——奇数位不变还是左移动一位（到了偶数位上），偶数位先右移后取数，那么可以用同一个常量——或上，同样效果

2. 同理：**`i = (i & 0x33333333) << 2 | (i & 0xCCCCCCCC)>>>2;`**实现2位为一组的交换：

   `0x33333333`的二进制是：`00110011001100110011001100110011`：低2位取出，左移2位

   `0xCCCCCCCC`的二进制是：`11001100110011001100110011001100`：高2位取出，右移2位（逻辑右移，前面是补0的）

   同理也可以优化成只用一个常量

3. 同理：`i = (i & 0x0f0f0f0f) << 4 | (i >>> 4) & 0x0f0f0f0f;`实现4位为一组的交换，那么移位也需要4位移动

why要写的如此晦涩，不能最高位和最低位交换，第二个和倒数第二个交换这样循环，即按照数组交换的方式来吗？

$\because$ CPU不能高效的操作二进制中的单一位，它的基本单位就是一个字长（32位机器就是32位字长）。并且**移位和逻辑运算（与或非异或等）是高效的，而算术运算（加减乘除等）比较慢**，所以该方法是充分利用CPU的特性实现的。通过简单的移位和逻辑运算得到。

普通的交换用在上层，比如数组逆序等地方是好的，但是对于二进制来说这样的交换效率低。

（2）reverseBytes：按字节进行互换——这个可以在算法题中应用

这边举例：0x12 34 56 78（每两个是一个字节），会变成0x78 56 34 12

```java
public static int reverseBytes(int i) {
    return ((i >>> 24)           ) |	// 最高字节移到最低字节，最低24位全部移除,0x00 00 00 12
        ((i >>   8) &   0xFF00) |		// 固定右边第二个字节：0x00 12 34 56，0x00 00 34 00
        ((i <<   8) & 0xFF0000) |	// 固定右边第三个字节：0x34 56 78 00, 0x00 56 00 00
        ((i << 24));		// 最低字节移动到最高位，高24位全部移除，0x78 00 00 00
}
```

### 2. 循环移位

普通的移位：如果是左移`<<`：那么最后自动补0；如果是右移`>>/>>>`：前者是自动补0，后者是补符号位（即根据正负补0/1）

而循环移位就是一个循环，左移前面的出去了就补到最后面；右移同理

```java
// 循环左移
public static int rotateLeft(int i, int distance) {
    return (i << distance) | (i >>> -distance);
}
```

```java
// 循环右移：
public static int rotateRight(int i, int distance) {
    return (i >>> distance) | (i << -distance);
}
```

分析：

1. 首先需要明白，对于int类型移位来说，最多只能移动31位，超过31位都没有意义了。所以distance实际上只关注了**最低5位的值**（二进制）

2. eg：distance=8，`i << 8 |(i >> -8)`，而-8的二进制表示：`111111111111111111111111111 11000`

   那么只关注后五位，所以实际上是右移24位（所以只剩下最高的8位），所以实现了循环移位

3. 同理循环右移，也是一样

### ps：其他位运算的奇技淫巧

均来自HD这本书

1. highestOneBit

   传入一个数字，得到比该数字小的最大2次幂数，eg：15->8

   我的想法是：遍历每个2次幂和传入的值比大小/得到该数字的最高的1的位置

   ```java
   public static int highestOneBit(int i) {
       // HD, Figure 3-1
       i |= (i >>  1);	// 保证最高1位和其相邻右侧1位为1，即..001100...
       i |= (i >>  2);	// 保证最高位和其后面3位为1，即..00111100...
       i |= (i >>  4);	// 保证最高位和其后面7位为1，即..001111111100...
       i |= (i >>  8);	// 保证最高位在内的16位为1（如果有16位的话）
       i |= (i >> 16);	// 这边的操作是想让最高位的1和其之后的所有位数，全部变成1，然后用减法即可
       return i - (i >>> 1);	// 那么就是001111 - 000111 = 001000，就能得到最大的2次幂数字了
   }
   ```

   eg：18：`0001 0010`

   - 先将`0001 0010`右移1位， 得到`0000 1001`，再与自身进行或运算： 得到`0001 1011`。
   - 再将`0001 1011`右移2位， 得到`0000 0110`，再与自身进行或运算： 得到`0001 1111`。
   - 再将`0001 1111`右移4位， 得到`0000 0001`，再与自身进行或运算： 得到`0001 1111`。
   - 再将`0001 1111`右移8位， 得到`0000 0000`，再与自身进行或运算： 得到`0001 1111`。
   - 再将`0001 1111`右移16位， 得到`0000 0000`，再与自身进行或运算： 得到`0001 1111`。
   - 再将`0001 1111`无符号右移1位， 得到`0000 1111`。

   然后抽象出执行就是：

    现在有一个二进制数据，`0001****`，我们不关心低位的取值情况，我们对其进行右移并且进行或运算。

   先将`0001****`右移1位， 得到`00001***`，再与自身进行或运算： 得到`00011***`。

   再将`00011***`右移2位， 得到`0000011*`，再与自身进行或运算： 得到`0001111*`。

   再将`0001111*`右移4位， 得到`00000001`，再与自身进行或运算： 得到`00011111`。

   最后，00011111-00001111 = 00010000.

   参考[博客](https://juejin.cn/post/6844903927402479629)

2. lowestOneBit

   返回最低的2次幂的值

   ```java
   public static int lowestOneBit(int i) {
       // HD, Section 2-1
       return i & -i;
   }
   ```

   分析：这个跟补码的获得有关系，**取反后1变成了0，后+1，出现了进位，那么本来最低位的1又由于进位出现了**，所以与一下就只剩下本来最低位的1出现了

3. numberOfLeadingZeros

   返回最高1位之前0的个数，eg：10->28（前面有28个0）

   实际上是二分法

   ```java
   public static int numberOfLeadingZeros(int i) {
       // HD, Figure 5-6
       if (i == 0)		// 如果值是0，那么直接返回32，不判断了
           return 32;
       int n = 1;
       // 如果不满足，说明最高位1出现在前16位中；满足，说明前16位全为0，个数+16；左移，移除高16位0
       if (i >>> 16 == 0) { n += 16; i <<= 16; }	
       // 不满足，说明最高位1出现在前8位中；满足高8位为0，个数+8，移除高8位
       if (i >>> 24 == 0) { n +=  8; i <<=  8; }
       // 不满足，最高位1出现在前4位中，本质同上
       if (i >>> 28 == 0) { n +=  4; i <<=  4; }
       // 不满足，最高1出现在前2位中，同上
       if (i >>> 30 == 0) { n +=  2; i <<=  2; }
       n -= i >>> 31;	// 右移31位，只剩下最高位未判断过了，如果i=1，需把多余计算的删除
       return n;
   }
   ```

4. numberOfTrailingZeros

   最低位1之后0的数目

   ```java
   public static int numberOfTrailingZeros(int i) {
       // HD, Figure 5-14
       int y;
       if (i == 0) return 32;
       int n = 31;			// 如果不是0，那么必定存在至少一个1
       // 获取低16位，如果值不为1——最低位的1一定出现在低16位中，那么只保留低16位（此时在高16位中）
       y = i <<16; if (y != 0) { n = n -16; i = y; }	
       y = i << 8; if (y != 0) { n = n - 8; i = y; }
       y = i << 4; if (y != 0) { n = n - 4; i = y; }
       y = i << 2; if (y != 0) { n = n - 2; i = y; }
       return n - ((i << 1) >>> 31);	// 是对原数字进行操作，先左移一位是移走最高位的1，那么结果就是31；而最低位是1的话，那么就是n-1——这一句话直接将最大值和最小值都考虑在内了
   }
   ```

   实现同3

   思想一样，但是对边界条件的处理更为巧妙:clap:

5. bitCount

   常规解法，是数字每次右移后都和1与一下，如果是1就++；直到该数字变成0 => 改进一点就是，每次都是`n&(n-1)`，能抹除最高位的1，直到值为0——我能想到这个就笑死了

   二分法的思想：每两个一组统计一次1，每4个一组统计一次；8个一组统计一次；16个一组统计

   ```java
   public static int bitCount(int i) {
       // HD, Figure 5-2
       i = i - ((i >>> 1) & 0x55555555);
       i = (i & 0x33333333) + ((i >>> 2) & 0x33333333);
       i = (i + (i >>> 4)) & 0x0f0f0f0f;	// 保留4位的数不管，最后统一清除
       i = i + (i >>> 8);			// 保留16组中的高8位不管，统一清掉（因为不会溢出到前面的）
       i = i + (i >>> 16);	// 同上
       return i & 0x3f;		// 最后只保留最后的6位
   }
   ```

   该算法的原型是：

   ```java
   public static int bitCount(int i) {
       // 5=1001，获取奇数位+获取偶数位（移动到奇数位）——每两位内计算出1的个数：eg：01+01=10，存在两个1；00+01，存在一个1
       i = (i & 0x55555555) + ((i >>> 1) & 0x55555555);
       // 3=0011每4位内计算出1的个数，eg：10+10=0100，存在4个1（前面的运算已经计算出2位内1的个数）
       i = (i & 0x33333333) + ((i >>> 2) & 0x33333333);
       // 每8位内1的个数，0100+0100=00001000
       i = (i & 0x0f0f0f0f) + ((i >>> 4) & 0x0f0f0f0f);
       // 每16位内1的个数
       i = (i & 0x00ff00ff) + ((i >>> 8) & 0x00ff00ff);
       // 32位个数计算
       i = (i & 0x0000ffff) + ((i >>> 16) & 0x0000ffff);
       return i;
   }
   ```

   过程如同这样：

   <img src="C:/Users/surface/Desktop/计算机知识整理/java/pic/image-20201218141747139.png" alt="image-20201218141747139" style="zoom:80%;" />

   然后进行优化：

   1. 对于第一步：发现2bit计算1的数量：如果两位都是1，即`0b11 => 0b01 + 0b01 = 0b10 = 2`;如果两位存在一个1，`0b10/0b01 =>0b00 + 0b01 = 0b01 = 1`。而研究发现：`2=0b11-0b01，1=0b10-0b01,1=0b01-0b00`，直接通过减就可以了，可以减少一次与计算：`i = i - ((i >>> 1) & 0x55555555)`
   2. 对于第二步：无优化
   3. 对于第三步：实际是计算每个byte中的1的数量，最多8（0b1000）个，占4bit，可以最后进行位与运算消位，减少一次&运算：`i = (i + (i >>> 4)) & 0x0f0f0f0f`
   4. 第四,五步：同上理由，可以最后消位。
   5. 但是由于int最多32（0b100000）个1，最后一步不需要的bit位抹除就可以了：`i & 0x3f`（只保留低6位）

   参考[文章](https://juejin.cn/post/6844903760045539341)

6. signum

   返回该数字的正负情况：-1：表示该值是负数；0：表示该值是0；1表示该值是正数

   ```java
   public static int signum(int i) {
   	// HD, Section 2-7
   	return (i >> 31) | (-i >>> 31);
   }
   ```

   分析：如果该数字是负数，那么右移31位之后，全部是符号位1；而其相反数，右移之后全为0，或操作之后那么返回值就是-1；

   如果该数字是正数，右移31位之后，全部是0；而其相反数，右移之后最低位为负数符号位1，其余全为0（逻辑右移），或操作之后返回值就是1；

   如果该数字是0，那么0|0=0

## 5. Character一些常用的方法

这边不考虑原理，仅做一个归纳，方便之后查询使用：

### isXXX方法：

```java
isLowerCase(char ch);		// 判断是否是小写字符
isUpperCase(char ch);		// 判断是否是大写字符
isDigit(char ch);			// 是否是数字
isLetter(char ch);			// 是否是字母：包含大写字符、小写字母、标题字母、变换字母、其他字母
isLetterOrDigit(char ch);	// 是否是字母 or 数字
```

### toXXX方法：

```java
toLowerCase(char ch);
toUpperCase(char ch);
```

# 包装类的使用

## 1. 基本数据类型和类对象的转换

**`valueOf()`** =>每个包装类都实现了该函数（上面已经解析过了）

**`xxxValue()`**，eg：`aa.intValue()`，`cc.charValue()`

```java
int a = 1;
Integer aa = Integer.valueOf(a);		// 将基本数据类型转换为对应的包装类
int aaa = aa.intValue();		// 将包装类转换为基本数据类型
```

Java5之后引入了**自动装箱和拆箱**，即直接赋值即可，而不用调用方法（是编译器自动会替换为对应的方法调用）

```java
Integer a = 100;	// 自动装箱，编译器会自动替换：Integer a = Integer.valueOf(100);
int b = a;		// 自动拆箱，实际上编译器会将其替换为 int b = a.intValue();
```

=> 这个是Java编译器提供的能力

# ps：面试常考的点

## 1. 包装类赋值的本质

```java
// Integer类、short类
private final int value;
```

——**发现value是不可更改的**

为什么我们初始化一个Integer对象后还可以改变它的值呢？

——本质上是新创建了一个对象，将原对象指向新对象

```java
Integer i = new Integer(1);
System.out.println(i);//1
i = 5;//底层==>i = Integer.valueOf(5);
System.out.println(i);//5
```

why 设置为不可变类呢？

程序更为简单且安全，不用担心数据会被意外修改，可以**安全的共享数据**，尤其是在多线程的情况下。——一旦创建只能读不能写

来自：https://catwing.blog.csdn.net/article/details/99845472

## 2. 为什么Boolean的hashCode()方法返回值是1231或1237

`hashCode`的主要目的就是**为了提高诸如java.util.HashMap这样的散列表(hash table)的性能**。

用的是较大的素数，且不会太大，也不会太小，能够保证避开常见`hashCode`的取值范围，eg：Integer的256个缓存数字，所以综合多方考虑，就是这两个数据

参考：https://blog.csdn.net/qq_21251983/article/details/52164403

——没有多大价值，考虑为啥散列基础值为31更有意义。

## 3. double和long的原子性

对于没有`volatile`修饰的`long`或者`double`的单个写操作，会被分成两次写操作，每次对32位操作。因此可能会导致线程会读到来自不同线程写入的32位数据组合的错误结果。对于`volatile`修饰的`long`和`double`而言，写和读操作都是原子的。

不过对于现在绝大部分的64位的机器以及使用64位的JVM时，这个问题一般是忽略的，因为底层JVM会保证操作的原子性。而只是从上层代码的层面来说，可能存在非原子性的问题。

## 4. 包装类对象的创建方式

存在两种：

1. 调用new：包装类也是提供了构造方法的，可以通过new来创建对象
2. 调用装箱方法（默认会调用）**valueOf()**——更建议用此种方法，因为new创建，每次都会创建一个新对象。而包装类在**底层实现的过程种都会引入缓存技术**，减少需要创建的对象次数，节省空间提高性能（除了Float和Double）

## 5. equals和hashCode的关系

### (1) equals和==的比较

`equals`是超类Object提供的一个实例方法（每个类的父类都是Object，所以每个类都有`equals`方法）

主要目的是检测当前对象是否和另一个对象相等

而再Object类中提供的默认方法等同于`==`

```java
public boolean equals(Object obj) {		// 参数是对象类型
	return (this == obj);
}
```

而`==`是去判断两个对象是否有**相同的引用**——即是否**指向同一个地址**，如果是，那么一定相等；

$\therefore$如果是默认继承了该Object类，并且没有重写`equals`方法，那么就是进行内存地址比较。`equals`和`==`效果一样。	

但是如果都是进行地址比较，那么equals就没有存在的必要了（直接用==替换即可）

equals在功能上更注重于**两个对象逻辑上是否相等，直观就是内容/值相等**。实际上是提供了一个方法（接口），用户可以根据实际需求实现比较方式。

例如：包装类都重写了equals方法，从地址比较转换成值比较

```java
// Integer类的重写
public boolean equals(Object obj) {
	if (obj instanceof Integer) {		// 首先，判断传入的对象是否是Integer类型
		return value == ((Integer)obj).intValue();	// 然后转换成基本类型值再进行比较
	}
	return false;
}
```

当然也存在一些场景，是不用重写equals的

- **类的实例本质上是唯一的**——是需要比较地址的唯一性，eg：Thread，关注的就是实例本身的唯一性

- **不关心类是否实现了equals方法**，eg：Random，不关注两个随机数是否相等

- **该类的超类已经重写了equals方法，而只需要继承即可**，eg： AbstractMap已经重写了，其子类只需要继承

- 类是私有的（内部私有类）/包级私有（默认default），那么不需要用到equals方法，反而还需要禁止

  ` @Override public boolean equals(Object obj) { throw new AssertionError();} `——禁止使用。

$\therefore$总结： 默认情况下是从超类Object继承而来的，equals方法与`==`是完全等价的，比较的都是对象的内存地址，但我们可以重写equals方法，使其按照我们的需求的方式进行比较，如Integer类重写了equals方法，使其比较的是里面的值，而不是内存地址。 

### (2) equals重写的规则

通用规则（和数学规则很像）

1. **自反性**： 对于任何非空引用值 x，x.equals(x) 都应返回 true（自己和自己比应该一样）
2. **对称性**：对于任何非空引用值 x 和 y，当且仅当 y.equals(x) 返回 true 时，x.equals(y) 才应返回 true。（x和y相互比较应该结果一样） 
3. **传递性**：对于任何非null的引用值x、y与z，如果y.equals(x)返回true，y.equals(z)返回true，那么x.equals(z)也应返回true。 （x和y一样，y和z一样，那么x和z应该一样）
4. **一致性**： 对于任何非null的引用值x与y，假设对象上equals比较中的信息没有被修改，则多次调用x.equals(y)始终返回true或者始终返回false。（结果保持稳定性）
5. 非空性：对于任何非空引用值x，x.equal(null)应返回false。

重写问题容易发生在：父类和子类的混合比较中

eg：破坏了对称性：

```java
public class Car {			// 父类
	private int batch;		// 批次变量
	public Car(int batch) {
		this.batch = batch;
	}
    @Override
    public boolean equals(Object obj) {		// 比较如果类型一样，并且属于同批次，那么就认为一样
        if (obj instanceof Car) {
            Car c = (Car) obj;
            return batch == c.batch;
        }
        return false;
    }
}

public class BigCar extends Car {		// 继承父类Car
	int count;
	public BigCar(int batch, int count) {
		super(batch);
		this.count = count;
	}
	@Override
	public boolean equals(Object obj) {		// 重写父类的方法
		if (obj instanceof BigCar) {		// 要求类型一样
			BigCar bc = (BigCar) obj;
			return super.equals(bc) && count == bc.count;	// 批次一样，且计数一样
		}
		return false;
	}
	public static void main(String[] args) {
		Car c = new Car(1);
		BigCar bc = new BigCar(1, 20);
		System.out.println(c.equals(bc));		// >>true 调用父类的方法
		System.out.println(bc.equals(c));// >> false 调用子类的方法，由于类型不一样，直接false
	}
}
// >> 违反对称性

// 修改如下：
 	@Override
	public boolean equals(Object obj) {
		if (obj instanceof BigCar) {
			BigCar bc = (BigCar) obj;
			return super.equals(bc) && count == bc.count;
		}
		return super.equals(obj);	// 如果类型不是匹配，那么去调用父类的方法
	}

// 符合了对称性，未符合传递性
public static void main(String[] args) {
	Car c = new Car(1);
	BigCar bc = new BigCar(1, 20);
	BigCar bc2 = new BigCar(1, 22);
	System.out.println(bc.equals(c));		// bc=c（这边的等于是表示返回值true）
	System.out.println(c.equals(bc2));		// c=bc2
	System.out.println(bc.equals(bc2));		// bc!=bc2（因为count不一样）
}
```

分析：

1. 最后出现不符合传递性的原因：
   - 父类和子类进行混合比较（纯子类、纯父类比较不存在问题）
   - 子类声明了新变量，且在equals方法中增加了对该变量的判断条件（如果不增加变量的判断条件也不会存在问题）
2. 无法直接解决该问题，可以选择用组合方式替换继承，那么原父类的对象和原子类的对象肯定不一样

最后举一个反例来看不遵守重写规则带来的问题：

```java
public class AbnormalResult {
	public static void main(String[] args) {
		List<A> list = new ArrayList<A>();// 由于B类是继承A的，所以B的对象也能存放在A中
		A a = new A();
		B b = new B();			
		list.add(a);
		System.out.println("list.contains(a)->" + list.contains(a));//>>true
		System.out.println("list.contains(b)->" + list.contains(b));//>>false
		list.clear();
		list.add(b);
		System.out.println("list.contains(a)->" + list.contains(a));//>>true
		System.out.println("list.contains(b)->" + list.contains(b));//>>true
	}
	static class A {			// 静态内部类A，重写了方法equals
		@Override
		public boolean equals(Object obj) {
			return obj instanceof A;	// 如果obj是类A的实例就被认为是一样的
		}
	}
	static class B extends A {		// 静态内部类B，继承了A，且重写方法
		@Override
		public boolean equals(Object obj) {
			return obj instanceof B;		// 如果obj是类B的实例就被认为是一样的
		}
	}
}
```

根据输出结果发现违反了对称性，即a.equals(b) = true，而b.equals(a) = false

下面看`list.contains()`是如何实现的：

```java
@Override       
public boolean contains(Object o) { 	// 看对象是否在list存在，就看是否能找到其索引值	
	return indexOf(o) != -1; 		
}

public int indexOf(Object o) {	// 找对象o在数组中的索引
	E[] a = this.a;
	if (o == null) {			// 如果是null，那么只能去找未赋值的对象
        for (int i = 0; i < a.length; i++)
            if (a[i] == null)
                return i;
	} else {				// 如果是非null，那么遍历数组通过equals匹配
		for (int i = 0; i < a.length; i++)
			if (o.equals(a[i]))	// 最后问题变成了：a.equals(a[i])和b.equals(a[i])
				return i;
	}
	return -1;
}
```

所以将问题具体定位变成了：

`a.equals(a)`：满足自反性；`b.equals(a)`：B是A的子类，也是满足的，true

`a.equals(b)`：类型不匹配，false；`b.equals(b)`：满足自反性

所以将B的方法修改即可：

```java
public boolean equals(Object obj) {
	if(obj instanceof B){
		return true;
	}
	return super.equals(obj);
}
```

可以发现重写equals还是有很多坑的

有人总结出的重写技巧：

1. 首先，**使用==操作符检查“参数是否为这个对象的引用”**：如果是对象本身，则直接返回，拦截了对本身调用的情况，算是一种性能优化；

2. 判空操作， 如果为null,返回false；

3. 接着再**使用instanceof操作符检查“参数是否是正确的类型”**：如果不是，就返回false，正如对称性和传递性举例子中说得，不要想着兼容别的类型，很容易出错。在实践中检查的类型多半是equals所在类的类型，或者是该类实现的接口的类型，比如Set、List、Map这些集合接口。

   - 如果equals的语义在每个子类中有所改变，就使用getClass检测——严格判断类型是否匹配
   - 如果所有的子类都拥有统一的语义，就使用instanceof检测 ：（eg：父类car与子类bigCar混合比，我们统一了批次相同即相等）

4. 在类型一致的情况下，**把参数转化为正确的类型**

5. **对于该类中的“关键域”，检查参数中的域是否与对象中的对应域相等**：

   - 基本类型的域就用`==`比较，

   - Float用Float.compare方法，Double域用Double.compare方法，

   - 对象域，一般递归调用它们的equals方法比较。

     对象域判断的最简单的版本：加上判空检查和对自身引用的检查，一般会写成这样：`(field == o.field || (field != null && field.equals(o.field)))`

6. **编写完成后思考是否满足上面提到的对称性，传递性，一致性等等**

下面是String类中对equals的重写：

```java
 public boolean equals(Object anObject) {
	if (this == anObject) {	// 看是否是引用同一个对象，如果是就避免了后面的比较，对应条1
		return true;
	}
	if (anObject instanceof String) {	// 判断参数类型是否为String类型，对应条3
		String anotherString = (String)anObject;	// 类型转换，对应条4
		int n = value.length;				
		if (n == anotherString.value.length) {	// 判断两个字符串长度是否一样
			char v1[] = value;			// 类中存在一个变量存放的是字符数组
			char v2[] = anotherString.value;
			int i = 0;
			while (n-- != 0) {			// 每个字符一一拿出来比较
				if (v1[i] != v2[i])
					return false;
				i++;
			}
			return true;
		}
	}
	return false;
}
```

注意的是：

- equals函数里面一定要是Object类型作为参数，且最好显式将参数命名为otherObject，稍后将其强转成另一个叫做other的变量 
- equals方法本身不要过于智能，只要判断一些值相等即可

ps：可以看到在对象类型判断中用到了`instanceof`，该关键字能比较左边是否是右边类的对象，如果左边是子类的实例对象，也会返回true

一般都是推荐使用`getClass()`类型判断——严格进行类型判断（除非所有的子类都有统一的语义才使用instanceof，即所有子类中，我们认为某些参数相等那就是相等的，而不考虑其具体类型）

### (3) equals和hashCode

如果重写了equals，那么就要重写hashCode

hashCode主要是针对映射相关的操作（Map接口）。Map接口的类会使用到键对象的哈希码，当我们调用put方法或者get方法对Map容器进行操作时，都是根据键对象的哈希码来计算存储位置的

hashCode在Object类中声明，默认是返回该对象的内存地址的hash值，好的哈希方法应该将实例均匀分布在所有散列值上

hashCode()可以重写，所以hashCode()不一定能代表对象在内存的地址，如果想比对内存地址的不同，需要使用`System.identityHashCode(Object)`方法。

规定了：

- **equals返回true（表示同一个对象），那么hashCode也必须返回一样**；
- equal返回false，那么hashCode尽量不一样

那么既然重写了equals方法，那么hashCode的逻辑也需要修改

ps：Java7之后提供了更安全的调用获得散列码的方式：

```java
// java.util.Objects里面调用：——处理了null的情况，直接返回0
public static int hashCode(Object o) {
	return o != null ? o.hashCode() : 0;
}

// 还提供了多个参数传递获得更加均匀的散列值的方式：
public static int hash(Object... values) {
	return Arrays.hashCode(values);
}
```

ps：如何写一个好的散列函数

- 把某个非零的常数值，比如17，保存在一个int型的result中；（seed）

- 对于每个关键域f（**equals方法中设计到的每个域**），作以下操作：

  为该域计算int类型的散列码；

  ```
   i.如果该域是boolean类型，则计算(f?1:0),
   ii.如果该域是byte,char,short或者int类型,计算(int)f,
   iii.如果是long类型，计算(int)(f^(f>>>32)).
   iv.如果是float类型，计算Float.floatToIntBits(f).
   v.如果是double类型，计算Double.doubleToLongBits(f),然后再计算long型的hash值
   vi.如果是对象引用，则递归的调用域的hashCode，如果是更复杂的比较，则需要为这个域计算一个范式，然后针对范式调用hashCode，如果为null，返回0
   vii. 如果是一个数组，则把每一个元素当成一个单独的域来处理。
  ```

  result = 31 * result + c;

- 返回result

- 编写单元测试验证有没有实现所有相等的实例都有相等的散列码。

（为什么采用`31*result + c`，乘法使hash值依赖于域的顺序，如果没有乘法那么所有顺序不同的字符串String对象都会有一样的hash值，而31是一个奇素数，如果是偶数，并且乘法溢出的话，信息会丢失，31有个很好的特性是`31*i ==(i<<5)-i`,即2的5次方减1，虚拟机会优化乘法操作为移位操作的。）

=> 一个小小的公式有这么多弯弯绕绕:dog:

[这篇博客讲了：为什么 String hashCode 方法选择数字31作为乘子](https://segmentfault.com/a/1190000010799123)
总结，使用31这个参数，主要有2个目的：

1. 31可以被JVM自动优化，通过移位操作完成，`31 ==(i<<5)-1`

2. 31是一个不大不小的质数，是作为 hashCode乘子的优选质数之一

   - 质数：是根据数学上得出的，用质数计算得到的数能够减少冲突（数学问题，记住就行）

   - 而比31小的质数，比如说2等，其计算得到的哈希值过于小，所有哈希值分布在一个小的范围内，冲突率会上升

   - 那如果选择一个较大的数，eg：101，根据`result = 31 * result + c`的迭代调用，用int表示的哈希值容易超出范围，导致丢失信息

   - 而如果选择一个偶数计算的话，如果溢出会导致丢失数据，因为*2相当于移位操作，对于二进制来说乘上偶数就是移位，直接丢失数据了

=>和它相近的数，都没有办法达到条件1，eg：29、37、41、43等；
  比它大或小的数，要么容易溢出，要么值太小容易冲突。而通过大量的实验，发现31、33、37、39、41都很优，所以选择31最优


参考内容：

1. https://blog.csdn.net/javazejian/article/details/51348320
2. https://juejin.cn/post/6844903542440853518

