# Java一些库源码阅读

1. [包装类源码](#source_1)

# 1. 包装类源码

<a name="source_1"></a>

首先看一下包装类的类头（瞎编的名字）

```java
// 可以看到包装类都是final的（不可继承），继承的是抽象父类Nubmber，并且实现了Comparable接口
public final class Byte extends Number implements Comparable<Byte> {}

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

// Boolean的类头（和其余6种包装类不同）
// 没有继承Number，但是增加实现了java.io.Serializable接口
public final class Boolean implements java.io.Serializable, Comparable<Boolean>{}

// Comparable接口：主要就是一个方法
public interface Comparable<T> {
	public int compareTo(T o);
}
```

## 常量

没有列全，大部分常见的都包括了

```java
// Boolean的常量
public static final Boolean TRUE = new Boolean(true);	// 静态常量对象
public static final Boolean FALSE = new Boolean(false);

// Short的常量
public static final int SIZE = 16;	// 最大的二进制位数
public static final int BYTES = SIZE / Byte.SIZE;	// 16/8 字节数——2字节
public static final short   MIN_VALUE = -32768;	// 最小表示范围
public static final short   MAX_VALUE = 32767;	// 最大表示范围

// Character的常量（还有其他字符表示的相关）
public static final char MIN_VALUE = '\u0000';	// 最小值（0）
public static final char MAX_VALUE = '\uFFFF';	// 最大值（65535）
// 表示的最小和最大进制范围
public static final int MAX_RADIX = 36;
public static final int MIN_RADIX = 2;

// Integer的常量
@Native public static final int SIZE = 32;	// 二进制最大的位数
public static final int BYTES = SIZE / Byte.SIZE;	// 字节数——4字节
@Native public static final int   MIN_VALUE = 0x80000000;	// 最小值
@Native public static final int   MAX_VALUE = 0x7fffffff;	// 最大值

// Byte的常量
public static final int SIZE = 8;
public static final int BYTES = SIZE / Byte.SIZE;	// 字节数——1字节
public static final byte   MIN_VALUE = -128;
public static final byte   MAX_VALUE = 127;

// Long的常量
@Native public static final int SIZE = 64;
public static final int BYTES = SIZE / Byte.SIZE;	// 字节数——8字节
@Native public static final long MIN_VALUE = 0x8000000000000000L;
@Native public static final long MAX_VALUE = 0x7fffffffffffffffL;

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

### (1) toString()

该方法是实例方法，注意和同名的静态方法做区分（传参也不一样，目的也不一样）

该方法的主要目的是：对该对象直观的文本表示，所以需要简要的表达出该对象的信息，

**返回的是字符串格式**，建议所有的类都重写该方法

```java
// Object类
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

分析：对于超类，默认的方法就是获得该实例对象的类名+实例对象的十六进制表示的哈希值

```java
// Boolean包装类
public static String toString(boolean b) {	// 通过Boolean.toString()调用
	return b ? "true" : "false";
}

public String toString() {			// 通过实例变量调用
	return value ? "true" : "false";
}
```

分析：对于boolean，只有两种结果：true/false，那么只需要根据值直接转换即可

下面重点分析Integer的toString的实现：

#### Integer的toString：

为加快计算，设置的常量：

```java
// 4使用
final static char[] digits = {
    '0' , '1' , '2' , '3' , '4' , '5' ,
    '6' , '7' , '8' , '9' , 'a' , 'b' ,
    'c' , 'd' , 'e' , 'f' , 'g' , 'h' ,
    'i' , 'j' , 'k' , 'l' , 'm' , 'n' ,
    'o' , 'p' , 'q' , 'r' , 's' , 't' ,
    'u' , 'v' , 'w' , 'x' , 'y' , 'z'
};

// 4使用
// 10*10，获得二位数的十位，65-6，62-6，所以每一行都一样
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
// 10*10，获得二位数的个位：65-5，45-5，所以每一列都一样
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

1. `toString()`

   ```java
   // 最上层的调用：
   public String toString() {
   	return toString(value);		// 调用2
   }
   ```

2. `toString(int i)`

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
   	return new String(buf, true);
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
           q = i / 100;
           // really: r = i - (q * 100);
           // 用移位操作快速得到q*100：q*2^6+q*2^5+q*2^2 = q*64+q*32+q*4=q*100
           // r获得是最低两位
           r = i - ((q << 6) + (q << 5) + (q << 2));	
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

   另一个toString方法：（加上进制要求）

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
   
       char buf[] = new char[33];// 对于int类型，最大32位（二进制），其他进制位数更少，再+符号位
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

Byte、Short是调用了Integer的实现方法：

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

Character是利用String的构造方法直接调用String.valueOf

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

Long的实现方法：

```java
public static String toString(long i) {	// 默认转换为10进制
    if (i == Long.MIN_VALUE)		// 处理最小值情况（防止后面的去正数溢出）
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

Float和Double的实现：

=> 太过于复杂了（没这本事）

```java
public static String toString(float f) {
    return FloatingDecimal.toJavaFormatString(f);
}
```

### (2) equals()和hashCode()

```java
public boolean equals(Object obj) {
	if (obj instanceof Integer) {		// 首先，判断传入的对象是否是Integer类型
		return value == ((Integer)obj).intValue();	// 然后转换成基本类型值再进行比较
	}
	return false;
}
// =>其他几个包装类同理:Boolean、Byte、Short、Character、Long均同样的实现
    
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

```java
// boolean
public static int hashCode(boolean value) {
	return value ? 1231 : 1237;	// 返回两个质数之一——使用哈希时，不容易冲突（数学原因）
}

// byte/char/short/int同样的实现方式：——直接返回其值（那么如果它们值一样，那么hashCode也是一样的），符合逻辑
public static int hashCode(byte value) {
	return (int)value;
}

// long：高32位和低32位做异或——因为要求返回的是int类型
public static int hashCode(long value) {
	return (int)(value ^ (value >>> 32));		// 逻辑右移（空位补0）
}

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

### (3) compareTo()

Boolean的实现：

```java
// Boolean的实现
public int compareTo(Boolean b) {		// 本质上去调用compare方法
	return compare(this.value, b.value);
}

public static int compare(boolean x, boolean y) {
    return (x == y) ? 0 : (x ? 1 : -1);	// 如果相等返回0；不相等，则判断如果前一个为true，返回1；false，返回-1
}
```

整型实现：

```java
// Byte的实现：获取其值，然后减法运算
public int compareTo(Byte anotherByte) {
    return compare(this.value, anotherByte.value);
}

public static int compare(byte x, byte y) {
    return x - y;
}
=> Short、Character同理
  
// Integer的实现
public static int compare(int x, int y) {
    return (x < y) ? -1 : ((x == y) ? 0 : 1);	// 直接通过运算符号得到，而不是-
}
=> Long同理
```

浮点数实现：

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

### (4) 静态方法toString()和parseXXX()——未完待续

`parseXXX()`（xxx就是对应的包装类名字）：主要就是将字符串表示的内容转换成对应的包装类

```java
// Boolean
public static boolean parseBoolean(String s) {
    // 判断是否是null
    // 非null的情况下，判断当前字符串和"true"是否一样——必须和"true"完全一样才返回ture，否则是false
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

```java
// String是直接转换成Intger（十进制），然后在强制类型转换得到的
// 可以限定转换的进制
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

```java
// 可以设定转换的进制
public static int parseInt(String s, int radix)
                throws NumberFormatException
{
    /*
         * WARNING: This method may be invoked early during VM initialization
         * before IntegerCache is initialized. Care must be taken to not use
         * the valueOf method.
         */

    if (s == null) {	// 如果字符串为空，那么抛出异常
        throw new NumberFormatException("null");
    }

    if (radix < Character.MIN_RADIX) {	// 如果 进制超过最小进制2，抛出异常
        throw new NumberFormatException("radix " + radix +
                                        " less than Character.MIN_RADIX");
    }

    if (radix > Character.MAX_RADIX) {	// 超过最大的进制36，抛出异常
        throw new NumberFormatException("radix " + radix +
                                        " greater than Character.MAX_RADIX");
    }

    int result = 0;
    boolean negative = false;
    int i = 0, len = s.length();
    int limit = -Integer.MAX_VALUE;
    int multmin;
    int digit;

    if (len > 0) {
        char firstChar = s.charAt(0);
        if (firstChar < '0') { // 如果字符串第一个不是数字，可能是'+'/'-'（ASCII中比0小）
            if (firstChar == '-') {		// 如果首字符是-，设置标志位
                negative = true;
                limit = Integer.MIN_VALUE;
            } else if (firstChar != '+')	// 如果首字符是+，抛出异常
                throw NumberFormatException.forInputString(s);

            if (len == 1) // Cannot have lone "+" or "-"如果数字带符号，且只有一个符号的长度，抛出异常
                throw NumberFormatException.forInputString(s);
            i++;
        }
        multmin = limit / radix;
        while (i < len) {
            // Accumulating negatively avoids surprises near MAX_VALUE
            digit = Character.digit(s.charAt(i++),radix);
            if (digit < 0) {
                throw NumberFormatException.forInputString(s);
            }
            if (result < multmin) {
                throw NumberFormatException.forInputString(s);
            }
            result *= radix;
            if (result < limit + digit) {
                throw NumberFormatException.forInputString(s);
            }
            result -= digit;
        }
    } else {	// 字符串长度=0，抛出异常
        throw NumberFormatException.forInputString(s);
    }
    return negative ? result : -result;
}
```

### (5) Integer中的二进制相关

#### 1. 位翻转

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

#### 2. 循环移位

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

#### ps：其他位运算的奇技淫巧

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

   参考[博客](https://juejin.cn/post/6844903927402479629)

2. lowestOneBit

   返回最低的2次幂的值

   ```java
   public static int lowestOneBit(int i) {
       // HD, Section 2-1
       return i & -i;
   }
   ```

   分析：这个跟补码的获得有关系，取反后1变成了0，后+1，出现了进位，那么本来最低位的1又由于进位出现了，所以与一下就只剩下本来最低位的1出现了

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

   <img src="pic\image-20201218141747139.png" alt="image-20201218141747139" style="zoom:80%;" />

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

   如果该数字是正数，右移31位之后，全部是0；而其相反数，右移之后全为最低位为负数符号位1，其余全为0（逻辑右移），或操作之后返回值就是1；

   如果该数字是0，那么0|0=0

### （6）valueOf

```java
// 将int值转换成Integer包装类：如果值在-128~127范围内，那么就在cache池种找到该引用；如果不存在那么就新建一个包装类对象
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

如果调用valueOf创建对象时，数字在-128~127之间的，就直接返回默认的对象（所以同值的这些引用对象的地址都一样）；超出范围的才进行创建

通过共享常用对象来节省空间，并且由于Integer的不可变性，所以缓存的对象是安全的

# 2. String类

类头：final禁止继承；没有继承，实现了3个接口

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {}
```

常量：

```java
private final char value[];	// 存在一个字符数组来表示字符串，注意是final——不可修改的
```

构造方法：（还有其他的构造方法）

```java
public String(char value[]);
public String(char value[], int offset, int count);	// 会重新创建一个数组存放，而不是直接使用

// 实现方法：
public String(char value[], int offset, int count) {
    if (offset < 0) {
        throw new StringIndexOutOfBoundsException(offset);
    }
    if (count <= 0) {
        if (count < 0) {
            throw new StringIndexOutOfBoundsException(count);
        }
        if (offset <= value.length) {
            this.value = "".value;
            return;
        }
    }
    // Note: offset or count might be near -1>>>1.
    if (offset > value.length - count) {
        throw new StringIndexOutOfBoundsException(offset + count);
    }
    this.value = Arrays.copyOfRange(value, offset, offset+count);
}
```

# 3. StringBuilder类

看类头：

```java
public final class StringBuilder
    extends AbstractStringBuilder
    implements java.io.Serializable, CharSequence{}

// 继承自抽象父类，里面包含了一些实例变量和方法
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    char[] value;	// 存储字符数组（和String类似），只不过不是final，还可以修改
    int count;		// 存储数组长度（就是字符串长度）
    ....
}
```

构造方法：（还有其他多个构造方法）

```java
public StringBuilder() {
    super(16);		// 默认调用父类的方法——创建长度为16的字符数组
}

AbstractStringBuilder(int capacity) {
    value = new char[capacity];
}
```

分析：如果使用`new StringBuilder();`，那么会默认调用该构造函数，创建一个长度为16的字符串

实例方法：

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
    ensureCapacityInternal(count + len);	// 如果当前字符数组不够长，会扩容
    str.getChars(0, len, value, count);	// 会将新的字符串加入到原来的字符串后面
    count += len;		// 更新最新实际的字符串长度
    return this;
}

// 如果字符数组不够长，那么就新申请一个满足要求的数组，然后将旧数组中的值全部复制到新的数组中，然后让value指向这新的数组中
private void ensureCapacityInternal(int minimumCapacity) {
    // overflow-conscious code
    if (minimumCapacity - value.length > 0) {
        value = Arrays.copyOf(value, newCapacity(minimumCapacity));
    }
}

// 计算新申请的数组需要的长度
private int newCapacity(int minCapacity) {
    // overflow-conscious code
    int newCapacity = (value.length << 1) + 2; // 2 * 当前数组长度 + 2
    if (newCapacity - minCapacity < 0) {	// 如果还是不满足最小需要，那么就给最小需要
        newCapacity = minCapacity;
    }
    return (newCapacity <= 0 || MAX_ARRAY_SIZE - newCapacity < 0)
        ? hugeCapacity(minCapacity)
        : newCapacity;
}

private int hugeCapacity(int minCapacity) {
    if (Integer.MAX_VALUE - minCapacity < 0) { // overflow
        throw new OutOfMemoryError();
    }
    return (minCapacity > MAX_ARRAY_SIZE)
        ? minCapacity : MAX_ARRAY_SIZE;
}
```

分析扩容

1. 是在原来数组长度的基础上*2+2，所以如果都不超过最小需求`minCapacity`，那么是16->34->70->142->....

   为啥要+2，为了当length=0的时候也可以进行操作（防止只*2，而长度一直等于0）

   尝试增加的是当前数组的两倍，而不是仅了满足当前的最小需要——$\because$ 那下次继续append，又需要进行内存分配，效率低下

2. 扩容操作：

   - 为了减少内存浪费，所以不申请很大的数组长度
   - 指数扩容也能减少内存分配次数

   ——指数级扩容，是内存分配中常见的策略

toString方法：

```java
public String toString() {
    // Create a copy, don't share the array
    return new String(value, 0, count);	// 创建了一个新的字符串
}
```

注意：并不会直接使用`StringBuilder`提供的数组，而是新建了一个数组——主要是为了保证**String的不变性**（不受外界的控制）



```java
// 在指定的offset开始的地方插入字符串str
@Override
public StringBuilder insert(int offset, String str) {
    super.insert(offset, str);
    return this;
}

// 
public AbstractStringBuilder insert(int offset, String str) {
    if ((offset < 0) || (offset > length()))		// 处理传参错误——抛出异常
        throw new StringIndexOutOfBoundsException(offset);
    if (str == null)
        str = "null";
    int len = str.length();
    ensureCapacityInternal(count + len);		// 确保数组够长
    // 语义是：将value数组从offset到最后的数组移动到offset+len开始的位置——就是中间空出str长度的空间给str，即将插入使用
    System.arraycopy(value, offset, value, offset + len, count - offset);
    // 然后将str赋值到value从offset开始的位置
    str.getChars(value, offset);
    count += len;
    return this;
}

// 数组src从 srcPos 开始的length个数组移动到数组dest从destPos开始的地方
public static native void arraycopy(Object src, int srcPos,
								Object dest, int destPos, int length){}
```

分析：

1. arraycopy的优点是：即使src和dest是同一个数组，也能正确操作
2. arraycopy是native的——是Java的本地接口实现的——是调用非Java实现的代码，实际上使用C++实现的——因为该功能十分常见，而c++的实现效率很高

# 4. Arrays类

算是数组的工具类，提供了一系列的静态方法给数组使用，使用方式是`Arrays.xxx`。

**toString()**：在Arrays类中存在9个方法，是8种基本数据类型+1个Object对象——给对象元素使用的

```java
public static String toString(int[] a) {
    if (a == null)		// 如果数组未初始化，返回null
        return "null";
    int iMax = a.length - 1;
    if (iMax == -1)		// 就是len=0，表示数组未空，返回一个空的格式
        return "[]";

    StringBuilder b = new StringBuilder();	// 创建StringBuilder对象
    b.append('[');
    for (int i = 0; ; i++) {
        b.append(a[i]);			// 依次把数组中的元素加入
        if (i == iMax)
            return b.append(']').toString();		// 指向数组的最后一个元素
        b.append(", ");
    }
}
// Arrays.java
public static String toString(Object[] a) {
    if (a == null)
        return "null";

    int iMax = a.length - 1;
    if (iMax == -1)
        return "[]";

    StringBuilder b = new StringBuilder();
    b.append('[');
    for (int i = 0; ; i++) {
        b.append(String.valueOf(a[i]));	 // 注意这边是去获取该对象的值
        if (i == iMax)
            return b.append(']').toString();
        b.append(", ");
    }
}
// String.java
public static String valueOf(Object obj) {
    return (obj == null) ? "null" : obj.toString();	// 返回的是该对象的toString方法
}
```

### **sort()**⭐

排序方法：一共有18个方法，（7种基本数据类型+1个Object对象 + 1个泛型对象（可以支持所有对象）且支持自定义排序）*（1基本排序方法+1指定排序范围方法）

ps：boolean不需要排序

Java对排序的实现十分复杂：

对于基本数据类型，主要就是**双枢轴快速排序**（DualPivotQuicksort），Java7之前就是普通的快排

——因为快排快，但是是不稳定的

对于对象类型，采用**Tim排序**，Java7之前是归并排序

——对象类型排序需要稳定

```java
public static void sort(int[] a) {
    DualPivotQuicksort.sort(a, 0, a.length - 1, null, 0, 0);
}

// 可以指定排序的范围
public static void sort(int[] a, int fromIndex, int toIndex) {
    rangeCheck(a.length, fromIndex, toIndex);		// 需要进行边界检查
    DualPivotQuicksort.sort(a, fromIndex, toIndex - 1, null, 0, 0);
}

public static void sort(Object[] a) {		// 要求对象是实现了Comparable接口
    if (LegacyMergeSort.userRequested)
        legacyMergeSort(a);		// 归并排序
    else
        ComparableTimSort.sort(a, 0, a.length, null, 0, 0);	// Tim排序
}

// 可以自己写比较器作为其中一个参数，自定义比较方法
// 在两数进行比较的时候，就调用比较器中的compare方法
public static <T> void sort(T[] a, Comparator<? super T> c) {
    if (c == null) {
        sort(a);		// 如果没有定义比较器，就使用上面的Object对象方法
    } else {
        if (LegacyMergeSort.userRequested)
            legacyMergeSort(a, c);
        else
            TimSort.sort(a, 0, a.length, c, null, 0, 0);
    }
}

// 比较器，是一个接口，Comparator.java
public interface Comparator<T> {	
    int compare(T o1, T o2);		// 只有方法头，规定返回值为-1 o1<o2/0 o1=o2/1 o1>o2
    boolean equals(Object obj);
}
```

分析：sort方法默认是从小到大排序的，如果需要自己指定排序方法可以用**匿名内部类**来实现。

Java中排序是通过比较实现的（还有计数排序——不进行比较）

在排序中，当需要进行对象比较时，会去调用比较器comparator的compare方法



String中的CaseInsensitiveComparator比较器的实现：

```java
// String.java 内部实现了一个忽略大小写的比较器，可以直接调用
public static final Comparator<String> CASE_INSENSITIVE_ORDER
                                         = new CaseInsensitiveComparator();

// 实现在String.java内部——是一个私有静态内部类，实现了Comparator接口功能
private static class CaseInsensitiveComparator
            implements Comparator<String>, java.io.Serializable {
    // use serialVersionUID from JDK 1.2.2 for interoperability
    private static final long serialVersionUID = 8575799808933029326L;

    public int compare(String s1, String s2) {
        int n1 = s1.length();
        int n2 = s2.length();
        int min = Math.min(n1, n2);
        for (int i = 0; i < min; i++) {
            char c1 = s1.charAt(i);
            char c2 = s2.charAt(i);
            if (c1 != c2) {		// 如果两者相等就比较下一对；不等，就都转成大写再比较
                c1 = Character.toUpperCase(c1);
                c2 = Character.toUpperCase(c2);
                if (c1 != c2) {		// 转成大写还是不相等，就转成小写（有些字符只能转成小写比较）
                    c1 = Character.toLowerCase(c1);
                    c2 = Character.toLowerCase(c2);	
                    if (c1 != c2) {		// 如果都不相等，那么就是不相等，就看两者的小写字母的差值
                        // No overflow because of numeric promotion
                        return c1 - c2;
                    }
                }
            }
        }
        return n1 - n2;	// 如果都相等，说明前min个都一样，就看字符串是否一样长
    }

    /** 替换反序列化对象 */
    private Object readResolve() { 
        return CASE_INSENSITIVE_ORDER; 
    }
}

// 使用，直接调用String的静态comparator对象
String[] arr = {"hello","world", "Break","abc"};
Arrays.sort(arr, String.CASE_INSENSITIVE_ORDER);
```

**binarySearch**：二分查找方法，同sort一样，也是有18种方法，同样的情况

```java
public static int binarySearch(int[] a, int key) {
    return binarySearch0(a, 0, a.length, key);
}
// 具体实现：就是常见的二分查找方式，时间复杂度为O(NlogN)
private static int binarySearch0(int[] a, int fromIndex, int toIndex, int key) {
    int low = fromIndex;
    int high = toIndex - 1;

    while (low <= high) {
        int mid = (low + high) >>> 1;
        int midVal = a[mid];

        if (midVal < key)
            low = mid + 1;
        else if (midVal > key)
            high = mid - 1;
        else
            return mid; // key found
    }
    return -(low + 1);  // 找不到返回的值是-(插入点+1)
}

// 可以自定义比较方式
public static <T> int binarySearch(T[] a, T key, Comparator<? super T> c) {
    return binarySearch0(a, 0, a.length, key, c);
}

private static <T> int binarySearch0(T[] a, int fromIndex, int toIndex,
                                         T key, Comparator<? super T> c) {
    if (c == null) {
        return binarySearch0(a, fromIndex, toIndex, key);
    }
    int low = fromIndex;
    int high = toIndex - 1;

    while (low <= high) {
        int mid = (low + high) >>> 1;
        T midVal = a[mid];
        int cmp = c.compare(midVal, key);		// 调用自定义比较方法
        if (cmp < 0)
            low = mid + 1;
        else if (cmp > 0)
            high = mid - 1;
        else
            return mid; // key found
    }
    return -(low + 1);  // key not found.
}
```

分析：因为查找本质上还是在比较，比较目标数和当前数的大小，所以对于对象来说还是可以自定义比较内容的

注意：

1. 查找的数组**必须是有序的**
2. 如果该对象在数组中查找失败了，那么返回值是**-(插入点+1)**，eg：-4，表明该对象适合插入到数组的index=3的位置处
3. 如果数组中存在多个匹配的元素，返回值是不确定的——二分查找的特性决定的

**equals**：比较两个数组是否一样，9个方法：8个基本数据类型+1个Object对象

```java
// int很常规，不关注
// 关注一下Object对象的
public static boolean equals(Object[] a, Object[] a2) {
    if (a==a2)		// 先判断是否是引用同一个对象
        return true;
    if (a==null || a2==null)	// 排除特殊情况
        return false;

    int length = a.length;
    if (a2.length != length)		// 长度不一样也排除
        return false;

    for (int i=0; i<length; i++) {	
        Object o1 = a[i];		
        Object o2 = a2[i];
        if (!(o1==null ? o2==null : o1.equals(o2)))	// 本质上是去调用对象的equals方法
            return false;
    }

    return true;
}
```

**copyOf**：基于原数组，复制生成一个新数组——浅拷贝，只是新建了数组，而数组中元素对象，本质上是复制了对象的引用，没有为每个元素创建新对象。

```java
public static int[] copyOf(int[] original, int newLength) {
    int[] copy = new int[newLength];		// 数组长度是根据newLength来的
    System.arraycopy(original, 0, copy, 0,		// 调用的是native方法（c++实现）
                     Math.min(original.length, newLength));	// 选择最小值——可能数组只复制一部分（防止出现越界异常）
    return copy;
}

// 泛型对象
public static <T> T[] copyOf(T[] original, int newLength) {
    return (T[]) copyOf(original, newLength, original.getClass());
}

public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
    @SuppressWarnings("unchecked")
    T[] copy = ((Object)newType == (Object)Object[].class)	// 判断数组元素的对象类型和传入的类型是否一致
        ? (T[]) new Object[newLength]
        : (T[]) Array.newInstance(newType.getComponentType(), newLength);
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));	// 内核一样
    return copy;
}
```

**fill**：为数组的每个元素赋相同的值，有18个方法=（8种基本数据类型 + 1个object类对象）*（1个基本的+1个指定赋的范围）

```java
// 简单，没啥好说的
public static void fill(int[] a, int val) {
    for (int i = 0, len = a.length; i < len; i++)
        a[i] = val;
}

public static void fill(int[] a, int fromIndex, int toIndex, int val) {
    rangeCheck(a.length, fromIndex, toIndex);	// 范围检查
    for (int i = fromIndex; i < toIndex; i++)
        a[i] = val;
}
```

**hashCode**：为数组计算哈希值——本质上和String对象一样，有9种方法（8个基础数据类型+1个Object对象）

```java
public static int hashCode(int a[]) {
    if (a == null)
        return 0;

    int result = 1;
    for (int element : a)
        result = 31 * result + element;	// 数组元素的位置不同，对hash值的影响也不同（前面影响大）

    return result;
}

// boolean的计算：
result = 31 * result + (element ? 1231 : 1237);
// float的计算——转换成int的二进制表示
result = 31 * result + Float.floatToIntBits(element);
// double的计算——和计算包装类的hashCode一样
long bits = Double.doubleToLongBits(element);
result = 31 * result + (int)(bits ^ (bits >>> 32));
// Object对象——就是去调用每个元素对象的hashCode方法
result = 31 * result + (element == null ? 0 : element.hashCode());
```

**n维数组的方法**——数组元素还是数组，就会去递归调用，直到得到基本数据类型/具体对象

```java
// 递归获得数组hash值
public static int deepHashCode(Object a[]) {
    if (a == null)
        return 0;

    int result = 1;

    for (Object element : a) {
        int elementHash = 0;
        if (element instanceof Object[])
            elementHash = deepHashCode((Object[]) element);
        else if (element instanceof byte[])
            elementHash = hashCode((byte[]) element);
        else if (element instanceof short[])
            elementHash = hashCode((short[]) element);
        else if (element instanceof int[])
            elementHash = hashCode((int[]) element);
        else if (element instanceof long[])
            elementHash = hashCode((long[]) element);
        else if (element instanceof char[])
            elementHash = hashCode((char[]) element);
        else if (element instanceof float[])
            elementHash = hashCode((float[]) element);
        else if (element instanceof double[])
            elementHash = hashCode((double[]) element);
        else if (element instanceof boolean[])
            elementHash = hashCode((boolean[]) element);
        else if (element != null)
            elementHash = element.hashCode();

        result = 31 * result + elementHash;
    }

    return result;
}

// 递归判断相等
public static boolean deepEquals(Object[] a1, Object[] a2) {
    if (a1 == a2)
        return true;
    if (a1 == null || a2==null)
        return false;
    int length = a1.length;
    if (a2.length != length)
        return false;

    for (int i = 0; i < length; i++) {
        Object e1 = a1[i];
        Object e2 = a2[i];

        if (e1 == e2)
            continue;
        if (e1 == null)
            return false;

        // Figure out whether the two elements are equal
        boolean eq = deepEquals0(e1, e2);

        if (!eq)
            return false;
    }
    return true;
}

static boolean deepEquals0(Object e1, Object e2) {
    assert e1 != null;
    boolean eq;
    if (e1 instanceof Object[] && e2 instanceof Object[])
        eq = deepEquals ((Object[]) e1, (Object[]) e2);
    else if (e1 instanceof byte[] && e2 instanceof byte[])
        eq = equals((byte[]) e1, (byte[]) e2);
    else if (e1 instanceof short[] && e2 instanceof short[])
        eq = equals((short[]) e1, (short[]) e2);
    else if (e1 instanceof int[] && e2 instanceof int[])
        eq = equals((int[]) e1, (int[]) e2);
    else if (e1 instanceof long[] && e2 instanceof long[])
        eq = equals((long[]) e1, (long[]) e2);
    else if (e1 instanceof char[] && e2 instanceof char[])
        eq = equals((char[]) e1, (char[]) e2);
    else if (e1 instanceof float[] && e2 instanceof float[])
        eq = equals((float[]) e1, (float[]) e2);
    else if (e1 instanceof double[] && e2 instanceof double[])
        eq = equals((double[]) e1, (double[]) e2);
    else if (e1 instanceof boolean[] && e2 instanceof boolean[])
        eq = equals((boolean[]) e1, (boolean[]) e2);
    else
        eq = e1.equals(e2);
    return eq;
}

// 递归获得数组的内容
public static String deepToString(Object[] a) {
    if (a == null)
        return "null";

    int bufLen = 20 * a.length;
    if (a.length != 0 && bufLen <= 0)
        bufLen = Integer.MAX_VALUE;
    StringBuilder buf = new StringBuilder(bufLen);
    deepToString(a, buf, new HashSet<Object[]>());
    return buf.toString();
}
private static void deepToString(Object[] a, StringBuilder buf,
                                     Set<Object[]> dejaVu) {
    if (a == null) {
        buf.append("null");
        return;
    }
    int iMax = a.length - 1;
    if (iMax == -1) {
        buf.append("[]");
        return;
    }

    dejaVu.add(a);
    buf.append('[');
    for (int i = 0; ; i++) {

        Object element = a[i];
        if (element == null) {
            buf.append("null");
        } else {
            Class<?> eClass = element.getClass();

            if (eClass.isArray()) {
                if (eClass == byte[].class)
                    buf.append(toString((byte[]) element));
                else if (eClass == short[].class)
                    buf.append(toString((short[]) element));
                else if (eClass == int[].class)
                    buf.append(toString((int[]) element));
                else if (eClass == long[].class)
                    buf.append(toString((long[]) element));
                else if (eClass == char[].class)
                    buf.append(toString((char[]) element));
                else if (eClass == float[].class)
                    buf.append(toString((float[]) element));
                else if (eClass == double[].class)
                    buf.append(toString((double[]) element));
                else if (eClass == boolean[].class)
                    buf.append(toString((boolean[]) element));
                else { // element is an array of object references
                    if (dejaVu.contains(element))
                        buf.append("[...]");
                    else
                        deepToString((Object[])element, buf, dejaVu);
                }
            } else {  // element is non-null and not an array
                buf.append(element.toString());
            }
        }
        if (i == iMax)
            break;
        buf.append(", ");
    }
    buf.append(']');
    dejaVu.remove(a);
}
```





Collections存在两个静态方法，可以返回逆序的Comparator

```java
public static <T> Comparator<T> reverseOrder()
public static <T> Comparator<T> reverseOrder(Comparator<T> cmp)
```

