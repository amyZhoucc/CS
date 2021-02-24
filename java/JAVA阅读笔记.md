# JAVA阅读笔记

## 一、通用知识

数据在计算机中实际上都是**二进制形式表示的**，但是人难以阅读编程。于是，**高级语言引入了数据类型、变量的概念**——用来区分不同的数据，通过编译器来将数据转换成对应的二进制表示

程序的流程控制有两种：1. 条件执行；2. 循环

### （一）数的二进制表示

#### 1. 整数的二进制表示

正整数：通过原码进行表示，且正整数的**原码 = 补码**

负整数：通过补码进行表示：流程就是：只取其整数部分，忽略前面的负号，然后**将其原码表示出来，然后全部取反，+1**

eg： -1 => 0000 0001 => 1111 1110 => 1111 1111; -127 => 0111 1111 => 1000 0000 => 1000 0001

所以，可以直接通过最高位1/0，判断出该数是正数还是负数

那么。如果一个数是有符号的，那么其表示的数的绝对值会变小，即：8位二进制，如果是表示无符号数，那么能表示0~255；如果是有符号数，那么能表示-127~0~128（因为**最高位是符号位**）

表示的原因：**计算机中减法本质上就是加法，+相反数**，eg：5 - 3=> 5 + (-3)

<img src="pic\image-20201108194515909.png" alt="image-20201108194515909"  />

所以能理解加法上溢会变成负数：

<img src="pic\image-20201108194619414.png" alt="image-20201108194619414"  />

#### 2. 小数的二进制表示

## 二、Java基础知识

### （一）数据类型

数据类型：主要用于对数据进行归类

变量：数据放到内存中的某个位置，为了方便找到和操作数据，**对这个位置起个名字**，那么Java能够通过该名字来找到该变量的位置，并对其进行操作。

#### 1. 基本数据类型

java数据类型：

| 数据类型 | 种类                                                    |
| -------- | ------------------------------------------------------- |
| 整型     | byte（1字节）/short（2字节）/int（4字节）/long（8字节） |
| 浮点型   | float（4字节）/double（8字节，表示范围、精度更高）      |
| 字符型   | char（单个字符，2字节）                                 |
| 布尔型   | boolean                                                 |

（在给long型赋值时，需要额外加L/l，即 `long a = 233333333333333L`）

（在给float型赋值是，需要额外加F/f，即 `float b = 333.33F`，**因为java中默认小数类型为double**。并且，如果左边定义是float，而右边又没有显式注明是float类型，即仅一个数字，那么**会报错**，eg：float f = 0.1;❌）

（整数也可以直接赋值给float/double）

（字符型是能存放一个字符的，**可以是中文字符/英文字符**，赋值时用**单引号**，eg：char c = '大';）

对于常量数字：可以用最常见的**十进制表示**，也可以**用十六进制表示（在前面加0x/0X即可）**，eg: 0x16 =22；可以**用二进制表示（在前面加0b/0B）**（Java7开始允许）

#### 2. 对象类型

**java除了以上的基本数据以外，其他都是对象类型**，这是由于java的面向对象特性决定的。（有一点点类型于c的结构体）

*eg：学生的信息*

- *姓名：char的数组*
- *学号：int的数组*
- *年龄：整型*
- *性别：char*
- *分数：浮点型/整型*

##### （1）数组类型

有**3种定义数组**的方式，如下：

需要注意的是：java的`[]`紧跟基础数据类型之后的，eg：**`int[]`**；创建数组需要`new`，eg：`new int[]` ,数组的表示用`{}`括上。

```java
int[] arr1 = {1, 2, 3};				// 定义时赋值，数组用{}限定
int[] arr2 = new int[]{1, 2, 3, 4};		// 定义时赋值，并且右侧用new创建新的数组
int[] arr = new int[5];				// 仅定义，定义之后默认值为0/false/'\0'，之后需要逐个赋值
```

数组不能一边给定长度一边赋值，即使赋值数目和长度一样

eg： `int[] arr = new int[5]{1,2,3,4,5}`❌

数组长度可以动态确定，但是确定后就不能改变

**数组属性**：

```java
int len = arr.length;			// 注意后面没有括号!!!
```

**数组内存的形式**：

基本数据变量，内存只分配一个内存空间

数据类型，内存分配两个内存空间，一个**存放 数据存放的空间的首地址**，数据存放的空间是连续的一个内存块

eg：

<img src="pic\image-20201105213732231.png" alt="image-20201105213732231" style="zoom:67%;" />

=> 目的：**为了数组之间可以直接赋值**

eg：`arr1 = arr2`，**实际上就是arr2的内存地址赋值给了arr1**，eg：arr2指向的地址为2000，2000存放的数据是3000，而3000起始的地址就是存放实际的内存数据；经过赋值操作之后，arr1指向的地址变为2000——**实际上，数组内部没有变化**

### （二）基本运算
#### 1. 算数运算
和C语言一致，+、-、*、/、%、++、--（加减乘除取模自加自减）

需要注意的是：取模运算（%），主要适用于 **整数、字符类型**（小数就不行）。其他运算对于所有数值类型和字符类型均适合。

-还能放在变量前面，表示 **改变变量值的符号**。eg：-a

**一些注意事项**
1. 注意表示范围

   eg：两个大数的乘法，很有可能上溢出

   ```java
   int a = 2147483647 * 2; // 会溢出的，2^32-1
   long a = 2147483647 * 2;	// 先按照int计算，溢出后再赋值
   long a = 2147483647 * 2L;		// 将其中一个数表示为long，那么在运算时，会强制类型转换为long和long的乘法
   ```

   两个整数的乘法，默认是舍去小数部分的，除非将其中一个数强制类型转换为float/double类型

   ```java
   double a = 10 / 4; 	// a=2.0，是double类型的
   double a = 10 / (double)4;		// a = 2.5
   ```

#### 2. 比较运算

和C语言的一致。

注意的是，对于普通变量来说，`==`是比较两个的值是否一样；即`a = 2; b = 2; a==b`的答案就是1。而**数组如果进行`==`比较，那么就是判断两个数组名是不是指向同一个数组，而不是判断两个数组的内容是否一样。**

```java
int[] arr4 = new int[]{1,2,3,4};		// arr4指向的是一个地址
int[] arr5 = new int[]{1,2,3,4};		// arr5指向的是另一个地址
System.out.println(arr4 == arr5);		// false，由于指向的地址不同
arr5 = arr4;			// 数组赋值
System.out.println(arr4 == arr5);		// true，由于指向的是同一个地址
```

如果实在需要比较数组中的元素是否一致，需要逐个比较

#### 3. 逻辑运算

&、|、!、^（与、或、非、异或）

&&、||（短路与、短路或）

短路与、短路或和与、或的区别在于：当&&/||前面的部分已经能够判断出结果，那么后面的部分就不计算了；但是，&/|都会计算后面的结果

eg:

```java
boolean a = true;
int b = 0;
boolean flag = a || b++>0;		// flag得到结果为true，而b++不会运算，则b=0
boolean flag = a | b++>0;		// flag得到的结果为true，而b++会运算，则b=1
```

### （三）条件执行

#### 1. 基础知识

if...else... ,和C语言一致

1. 需要注意的是如果关键字后面没有括号，只会执行紧跟关键字后面的一句指令，而后面的指令默认是条件外的指令，什么情况下均会执行.

   建议所有if后面都有大括号

2. if/else if.../else, else if是**前面的条件不满足之后的继续进行判断**

   ```java
   if(score > 60){			
   	return "及格";
   }else if(score > 80){		// else if前提是< 60,所以这个写法是错误的
       return "良好";
   }else{
   	return "优秀";
   }							// 将优秀\良好\及格判断顺序对换就可以了
   ```

switch 和C语言一致,使用的前提,是当if...else的判断很多时,可以使用switch来优化代码

表达式的值和case的value进行匹配, 然后选择哪个case中的代码块执行

```java
switch(表达式){
	case value1: codeblock1;break;	// 后面可以紧跟一组代码块，并且可以不加大括号
	case value2: codeblock2;break;
	...
	case valuen: codeblockn;break;
	default: codeblock;
}
```

**表达式的数据可以是: byte/short/int/char/String/enum**

（byte/short/int/char/enum本质上就是整数，**String也会被转换为整数（通过hashCode的方法转换为整数，那么可能存在不同的string但是有一样的hashcode，所以跳转之后会根据string的内容再次进行比较判断）**，但是不能支持long，因为跳转表的存储空间为32位，容不下long）

且value1~valuen不要求是排序的，编辑器会自动排序。

#### 2. 实现原理

if...else会转换成跳转指令. 有条件跳转 + 无条件跳转

switch看具体系统实现. 如果分支比较少,那么就和if...else一样, 转换为跳转指令. 如果分支较多,使用跳转指令会进行很多次比较运算,效率较低, 使用**跳转表**

<img src="pic\image-20201106220554976.png" alt="image-20201106220554976" style="zoom:80%;" />

有点类似于python字典的每一条数据：条件值——key，跳转地址就是——value

并且**该跳转表是有序的，按照条件值进行排序**，那么可以使用二分查找。如果值是连续的，还会进行特殊的优化——**变成一个数组**，O(1)的时间复杂度就可以找到。（如果不是连续的，但是比较密集，差的不多，那么也会优化为数组型的跳转表）

### （四）循环

有四种：while、do/while、for、foreach，前面三个和C语言一致，foreach是java区别于C有的

foreach不是关键字——即不存在foreach该指令，而是对for用特殊的格式，从而称呼为foreach

格式： for( : )，冒号前面是循环中的每一个元素，需要指定该元素的类型，和名称；冒号后面是需要遍历的数组或者集合——适用场景：**遍历**数组/集合的情况——foreach的语法更为简洁

```java
int[] arr = {1, 2, 3, 4};
for(int element: arr){
	codeblock
}
```

### （五）函数

#### 1. 基础知识

整体来说，和C没有啥差别

函数可以减少重复的代码和分解复杂操作。eg：将一个二分排序写成单独的函数，每次要排序只需要调用即可，而不需要自己重复写，同时也将排序问题简化为调用的问题，去处理其他的问题。

函数格式：需要注意的是，**修饰符**（C也有修饰符，但学的时候没注意，所以掌握不好，且Java和C有不同地方）

```
修饰符 返回值类型 函数名字(参数类型 参数名字, ...){
    代码段;
    return 返回值;
}
eg:
public static int main(String[] args){			// public static 均是函数修饰符
    ....
    return 0;
}
```

函数定义：不会执行代码；函数定义时声明的参数，类似于定义变量

函数调用：函数会被执行；函数调用时传递的参数，类似于给变量赋值

**Java中函数需要放在一个类中**（和C的差别所在），类中可以有多个函数——一般称为方法

如果该类中有一个`main`方法，那么在运行程序时，**编译器会寻找含有main函数的类，然后从该main函数开始运行**

函数可以调用同一个类中的其他函数（方法），也可以调用其他类中的函数（方法）

#### 2. 注意的点

1. 数组传参（和C一致）

   基本类型的传参是值传递，修改不改变原值；传递数组是传递的数组的存储位置，修改是影响数组实际内容的。

2. **可变长度的参数**（和C不一样的）

   就是传参的个数不定

   语法：**在数据类型后面加三个点...**

   限制：**可变长度的参数最多只能有一个，且必须位于参数列表的最后**

   可变长度的参数可以看成是数组。

   ```java
   // 本函数主要实现了求最大值的
   public static int max(int a, int ... b){	// 数据类型 ... name，只能有一个可变参数，且要放在参数列表的最后
       int max = a;
       for(int i = 0; i < b.length; i++){		// 可变参数可以看成一个数组（不定长的数组），有数组的特性——arr.length——求数组的长度
           if(max < b[i]) max = a[i];
       }
       return max;
   }
   ```

   实际上，Java处理时就是将该可变长参数转换为数组参数，即如果max(0, 1, 2, 3, 4) => max(0, new int[]{1, 2. 3, 4})

3. 返回值个数

   返回值最多只能有一个（可以没有，即return;（在函数返回值类型为void的时候））

   如果需要多个返回值，可以用一个数组进行返回

   如果需要返回多个类型的值，可以定义一个对象，然后返回该对象

4. 函数的重名（和C有些区别）

   不同的类（即不同的文件之间），函数名可以重复

   相同的类，函数名可以重复，但是**参数不能完全一样**——要么参数个数不同，要么参数个数相同但是至少一个参数数据类型不同

   ——称为函数重载，主要用于实现的意义一样，但是使用的场景/方向不同

   eg：

   ```java
   public static double max(double a, double b);
   public static float max(float a, float b);
   public static int max(int a, int b);
   public static long max(long a, long b);
   ```

5. 调用匹配（C不知有没有）

   就是在调用函数时，如果需要传参，**需要对传递的参数和声明的函数类型进行匹配，但是不要求完全一样**，可以进行自动类型转换，并寻找最匹配的函数——只要可以进行类型转换，就能进行匹配调用（除非无法转换）

   eg：

   ```java
   System.out.println(max('a', 'b'));		// 会将两个字符转换成int类型，所以去调用上面的第3个函数
   ```

#### 3. 函数调用的基本原理

计算机系统一般使用栈来存放函数调用过程中用到的数据：参数、返回地址、函数内定义的局部变量

（栈先进后出，一般都从高位向低位扩展）

eg：

<img src="pic\image-20201107215904383.png" alt="image-20201107215904383" style="zoom: 80%;" />

在还没有调用sum函数之前，只有两个参数需要存放，如下图：

<img src="pic\image-20201107215940575.png" alt="image-20201107215940575" style="zoom:67%;" />

调用该函数之后：需要将调用的参数入栈，将返回地址入栈（就是sum函数执行完毕，返回main函数的地址需要保存），然后跳转到sum函数，需要保存一个局部变量c。**返回值是专门保存到了返回值存储器中**。该函数调用完毕后，纷纷出栈，然后返回到上图状态。

<img src="pic\image-20201107220043036.png" alt="image-20201107220043036" style="zoom:67%;" />

针对数组操作有点不同：

<img src="pic\image-20201107220402338.png" alt="image-20201107220402338"  />

数组存放的位置是单独开辟的一个堆空间，在main函数的栈中是存放了数组的起始位置

（可以直观的发现，main函数和max函数使用的数组都是指向同一个数组起始地址，所以对其进行操作，数组的内容是会发生变化的）

<img src="pic\image-20201107220430885.png" alt="image-20201107220430885" style="zoom:67%;" />

堆的空间内容是不受栈的变化而变化的（值可能会改变），但是如果栈对应的函数结束，而没有其他的函数在使用该数组了，那么Java的垃圾回收机制会回收该堆，从而释放该空间

针对递归函数，栈的形态又发生了变化

<img src="pic\image-20201107221146337.png" alt="image-20201107221146337"  />

可以看到存放了前面递归函数的返回地址，直到最后一个函数获得了返回值之后，开始一级一级的返回

<img src="pic\image-20201107221209847.png" alt="image-20201107221209847" style="zoom:67%;" />

总结：所以可以看出，每次调用都需要分配栈空间来存放参数、返回地址、局部变量，并且在调用和返回时需要入栈和出栈，这些都是消耗。

且，如果递归深度过深，那么会抛出异常：**java.lang.StackOverflowError——栈溢出**


## 二、面向对象

主要方面：类的基础、类的继承、类的扩展、异常、常用基础类

## （一）类的基础

引出：java定义了8种基本数据类型。那么其他的数据类型如何表示呢——**用类表示**

### 1. 类的基本概念

类可以是**函数的容器**，也可以表示**自定义的数据类型**。

#### (1) 类是函数容器

某个类**只是函数的容器**，即该类中没有定义任何新的数据类型

eg：Java API类中的Math类，就是包含了一系列的数学函数。如果要使用这些函数，可以使用**`Math.`**（不需要导包）<a name="math"></a>

| 函数名           | 参数                           | 返回值   | 作用                                                         |
| ---------------- | ------------------------------ | -------- | ------------------------------------------------------------ |
| `Math.round()`;  | float a/double a               | int/long | 将浮点数四舍五入为整数                                       |
| `Math.ceil();`   | double a                       | double   | 向上取整（返回值仍是浮点数）                                 |
| `Math.floor();`  | double a                       | double   | 向下取整（返回值仍是浮点数）                                 |
| `Math.abs();`    | int/long/float/double a        | 对应类型 | 取绝对值                                                     |
| `Math.pow();`    | double a, double b             | double   | a^b，无论输入是何类型，返回值都是double                      |
| `Math.sqrt();`   | double a                       | double   | 对a开平方，返回值一直是double                                |
| `Math.max();`    | int a, int b/long/float/double | 对应类型 | 求a、b两数的较大值，返回值具体看输入的情况（输入两个的类型不一样就会选择强转） |
| `Math.log();`    | double a                       | double   | 求的自然数对数（e）                                          |
| `Math.random();` |                                | double   | 获得[0,1)范围内的随机数                                      |

这些函数都有相同的修饰符：`public static`

`public`，其他类可以使用，对其他类可见

相对的是`private`，表示私有，**只能在同一个类内部才能被调用**，而在该类外面是不能使用的，对其他类屏蔽

`static`：静态方法，也是类方法，**类方法可以直接通过类名进行调用，不需要创建实例**

相对的是实例方法，**实例方法必须通过实例或对象调用**，即使用之前，必须创建实例，new xxx

```java
public static int abs(int a) {			// 注意修饰符，表明是类方法，且是公开使用的
	return (a < 0) ? -a : a;
}
```

why要区分public和private呢？

private可以防止一些方法被外部类误用，而调用者只需要关注public就可以，而不需要关注其如何实现的。

一些私有变量可以通过`getter/setter`来获得值/设置值，这个也是封装的思想之一

**通过private封装和隐藏内部实现的细节，避免被误操作——计算机程序的一种基本思维**

下面是Arrays类的函数定义，在使用的之前需要导包`import java.util.Arrays`<a name="arrays"></a>

| 函数名                   | 参数                                                        | 返回值                                            | 作用                                                         |
| ------------------------ | ----------------------------------------------------------- | ------------------------------------------------- | ------------------------------------------------------------ |
| `Arrays.sort();`         | int[] a(int fromIndex, int toIndex)（不同数据类型都实现了） | void                                              | 在原数组位置上排序，默认是升序排序<br/>（参数：还可以自定义比较方法Comparator<? super T> c） |
| `Arrays.binarySearch();` | int[] a, int key                                            | int（找不到就返回一个负数，即=-(能插入的位置)-1） | 二分查找（前提是数组已经升序排列）                           |
| `Arrays.fill();`         | int[] a, int key                                            | void                                              | 给所有元素都赋值同一个值                                     |
| `Arrays.copyOf();`       | int[] a, int length                                         | int[]                                             | 将a数组的前length内容赋值给一个新数组，并返回该数组（是两个不同的数组，并且如果length>a.length，那后面的值自动补0） |
| `Arrays.equals();`       | int[] a, int[] b                                            | true/false                                        | 判断两个数组是否一样（是数组里面值的比较）                   |

#### (2) 类是自定义的数据

类作为自定义的数据类型，就是除了那8种以外的其他类型。数据类型需要包括：**属性、该类型可以进行的操作**。属性可以分为：该类型本身有的属性和具体实例有的属性；操作可以分为：该类型本身有的操作和具体实例有的操作。

那么类作为自定义的数据类型有如下的4部分组成：

- 类型本身有的属性：**类变量**，又叫做静态变量/静态成员变量（成员变量1）
- 类型本身可以进行的操作：**类方法**，又叫做静态方法（成员方法1）
- 类型实例有的属性：**实例变量**（成员变量2）
- 类型实例可以进行的操作：**实例方法**（成员方法2）

*（对于具体类型，不一定有全部的4部分，eg：Arrays就只有类方法）*

##### 1. 类变量（静态变量/静态成员变量）

表示的是类型本身有的属性，**经常用来表示一个类型中的常量**（有点类似于C的宏定义、const）

eg：Math类中的类变量：

必须要包含的修饰符是`static`：表示类变量

public代表其他类可见

**final：表示常量，即变量赋值之后就不能再修改了，可以避免误操作**（Java编译器会对final变量进行一些特别的优化）

```java
// 对外部类可以直接用Math.E/Math.PI，但是对其值不能修改
// 			——Cannot assign a value to final variable 'PI'
public static final double E = 2.7182818284590452354;	
public static final double PI = 3.14159265358979323846;
```

##### 2. 实例变量、实例方法

表示的是具体实例可以有的属性和操作，eg：int a;	=> int就是类型，而a就是实例（实际例子）

#### (3) 类的举例（定义、使用）

```java
// 需要新建一个java文件，然后才能写该class代码——类似于，一个类就是一个文件
public class Point {		// public表明该类是外部可以访问到的，但是类一定是public的，不然没有创建的意义
    public int x;		
    public int y;			// 定义了两个实例变量，且是可以被外部类访问的，实例变量不能用static修饰（这样就变成类变量了）
    public double distanceToZero(){			// 定义了一个实例方法
        return Math.sqrt(x * x + y * y);
    }
}
```

可以发现，**类的修饰符都是`public`，而不能是`private`**，因为类一定都能被其他类访问的，不然没有定义的意义。**类的修饰符可以没有——表示一种包级别的可见性，有的话也是public——全局可见性**。

**实例方法可以直接访问实例变量，而不需要传参了**：实例方法是带有一个隐含的参数，该参数就能用来操作实例自己，从而操作实例变量——本质上也还是需要参数的

**类方法只能访问类变量，不能访问实例变量，可以调其他的类方法，但是不能调用实例方法**（因为实例方法需要创建对象，而类方法在使用的时候不需要创建对象，所以没有对象就不能调用实例方法，也不能访问实例变量）

**实例方法既能访问实例变量，也能访问类变量，可以调用实例方法，也能调用类方法**（因为类方法不需要创建对象就可以直接调用，那么创建对象了就更加能调用类方法和类变量了）

```java
// 在其他一个类里面
Point p = new Point();	// 创建一个对象（实例）
p.x = 2;
p.y = 3;
System.out.println(p.distanceToZero());
```

**解释：**（需要重点关注的）

```java
Point p = new Point();	// 创建一个对象（实例）
// 等价于如下：
Point p; 	// 先声明一个变量，是Point类型，只会分配存放位置的内存空间，但是还没有指向任何实际内容——称为引用类型变量——类似于C中的指针声明
p = new Point();	// 创建了一个实例，然后赋值给了变量p
// => 这句话有两个步骤
// ⭐1. 分配内存，用来存储新对象的数据，对象数据包括这个对象的属性，即x、y⭐
// ⭐2. 给实例变量设置默认值，int默认为0（如果有构造方法，还需要去调用构造方法给实例变量赋值）⭐
```

所以p就是引用类型变量

**在创建对象时，所有实例变量均会设置一个默认值**（和局部变量不一样，需要手动赋值），**类似于创建数组时，数组元素都有默认值**（默认值实际上是可以修改的）

- 数值类型变量的默认值为0；
- Boolean是false
- char是'\u0000'
- 引用类型变量的默认值为null，表示不指向任何对象

实例变量的使用：`<对象变量名>.<成员名>`；类变量的使用：`<类名>.<成员名>`

实例方法的使用：`<对象变量名>.<方法名>`；类方法的使用：`<类名>.<方法名>`

所以整个步骤和基本数据类型一致：定义数据（类数据类型特有的设置默认值） ->赋值 ->操作 

ps：实际上不能将实例变量声明为public，而**应该通过调用实例方法来对实例变量进行操作**——因为实例方法可以对参数值进行检查和控制，防止误操作

——可以发现实例变量和实例方法都是通过创建的对象进行的，**通过对象来访问和操作其内部数据是一种基本的面向对象的思想**

#### (4) 变量的默认值

之前说了，实例在创建的时候，会给实例变量创建存储空间，并且设置默认值，那么该默认值是可以修改的，有如下两种修改方式

```java
int x = 1;
int y;
{					// {} 整个被称为代码块
	y = 2;
}

static int one = 1;		// 类变量也可以自定义初始值
static int two;
static{					// static{}整个被称为静态代码块，在类加载的时候执行，且只执行一次
    two = 2;
}
```

在创建对象的时候，首先会调用实例变量初始化，然后才会执行构造方法

而对于静态变量（类变量），是在类加载的时候执行，在创建对象之前就已经执行一遍了（只会执行一遍）

#### (5) private变量的含义、定义和使用方法

将实例变量变成private，在另外一个类里面的`p.x`和`p.y`均开始报错，因为现在的实例变量对其他类不可见了，所以不能直接调用

```java
public class Point {
    private int x;
    private int y;
    public void set(int x, int y){		// add
        this.x = x;			// 当前实例的实例对象x（左侧），被赋值为形参x的值（右侧），this是来消除两边的歧义的
        this.y = y;			// 否则的话，根据就近原则，就是形参赋值给形参，而实例变量值仍旧是0，0
    }
    public int getX(){	// add
        return this.x;
    }
    public int getY(){	// add
        return this.y;
    }
    public double distanceToZero(){
        return Math.sqrt(x * x + y * y);
    }
}
```

**this关键字，表示当前实例**（在此例子中，实际上可以将形参的名字修改为a、b或者其他，然后直接赋值x = a; y =b; 也不会产生歧义）

#### (6) 构造方法

前面(4)说了如何对变量修改默认值（其实就是赋初值），如何通过函数来修改变量的值（不是赋初值了）

那么更为规范的赋初值的方式——**构造方法**

```java
public Point(){			// 构造方法1
    this(1, 1);			// 其实就是去调用“构造方法2”——但是⭐这句话必须放在该构造函数的第一行⭐（也是避免误操作，防止自己定义的操作被覆盖了）
}
public Point(int x, int y){		// 构造方法2——更为常见和通俗的构造方法
    this.x = x;
    this.y = y;
}

// 使用构造方法
Point p = new Point(2, 3);
```

构造方法可以有多个，但是都需要满足如下条件：

- **名称固定，和类名一样**
- **没有返回值，也不能有返回值**，其实隐藏的返回值就是实例本身

——构造方法可以重载

两个特殊的构造方法：

1. 默认的构造方法

   **每个类都需要一个构造方法**，用在new的时候，但是如果构造方法没有什么操作用户可以不定义，而是**Java编译会自动生成一个默认构造方法**，而这个默认的构造方法**本身没有具体操作——就是一个空函数**。但是用户一旦**自定义了一个构造方法（哪怕为空操作），Java编译器将不会提供默认构造方法**。因为编译器会认为用户需要自行构造方法，而不会再提供空的

2. 私有构造方法——修饰符是：private

   有三种原因：不希望给类创建实例；单例模式；只是给同类的其他构造方法使用，用来减少重复代码量，具体见[文档：Java的额外知识.md](http://note.youdao.com/noteshare?id=9f8ab6eaf048c06b8c823ebe1c7f0b8c&sub=38FCCF96A0C74660A8B19E1140BA449E)

#### （7）类和对象的生命周期

就是类和对象在内存中的存活时长

对于类来说，**当第一次new类的对象时/通过类名访问类变量或类方法时**，Java就会将类加载进内存，为这个类分配一块空间，主要是存储：**类的定义、类的变量、方法信息、类的静态变量（并做初始化）**。类一旦加载进内存后不会被释放，**生命周期直到程序结束**，**类只会加载一次**，那么静态变量在内存中只有一份

——类只加载一次，一旦加载就活到程序结束。

每次操作new对象，就产生一个对象。内存中会**存储该对象的实例变量值**。——实例变量看new的次数（即对象个数）

对象**在内存中存储实例变量值，还存储对应类的地址**（就是前面加载类到内存的地址），那么该对象能够直接访问到类的变量和方法代码

**对象的释放是通过Java的垃圾回收机制**（其实不完全是）。具体如下：

- 对象有两块内存，保存地址的部分**分配在栈中**，而保存实际内容的部分**分配在堆中**。
- 栈中的内存是入栈就会分配内存（栈顶指针-1），出栈就自动释放所占用的内存（栈顶指针+1）
- 堆中的内存是使用垃圾回收机制，即当没有活跃变量指向对象时，堆空间可能会被释放（具体释放时间看Java虚拟机）

——总结：

1. 关键字

<img src="pic\image-20201120213330489.png" alt="image-20201120213330489" style="zoom:80%;" />

2. 一种思维方式：通过类实现自定义的数据类型，封装该类型所具有的属性（变量）和操作（方法），隐藏实现细节（只提供外部接口），从而在高层次（类、对象层次，而不是基本数据类型和函数层次）考虑——是解决复杂问题的思维方式（复杂问题一层层化简成简单问题）

### 2. 类的组合

——还是类似于C语言的结构体，结构体里面还是可以嵌套结构体的

类的组合，就是类里面的属性（变量）可以是类

```java
// Line类的定义
public class Line {
    private Point start;		// 实例变量是两个类对象
    private Point end;
    public Line(Point start, Point end){		// 构造方法
        this.start = start;		// 因为这两个变量在这之前应该已经实例化了，所以可以直接赋值
        this.end = end;
    }
    public double length(){		// 本质上是调用point的实例方法
        return this.start.distanceToAnother(this.end);
    }
}

// Point类里面的实例方法
public double distanceToAnother(Point p){
    return Math.sqrt(Math.pow((x - p.getX()), 2) + Math.pow((y - p.getY()), 2));
}

// main函数的使用
public static void main(String[] args) {
    Point start = new Point(2,3);		// 实例化两个点
    Point end = new Point(3,4);
    Line line = new Line(start, end);	// 实例化一条线
    System.out.println(line.length());
}
```

看一下这几个实例对象在内存中的存储情况：

<img src="pic\image-20201120222823380.png" alt="image-20201120222823380" style="zoom: 67%;" />

（由于都是对象，所以有两个地址，一个**保存实际内容的地址在栈中**，**保存实际内容的在堆中**）

——我们在对现实问题进行建模时，对概念分析有哪些属性、哪些行为，以及概念之间的关系，然后就针对概念定义一个类，并定义其方法、属性和类之间的关系（这个就涉及到了类的组合）

**类定义中，还能引用自己**。类似于递归调用

而关键点在于：**实例变量不需要一开始就有值**，即变量可以为null，直到需要的时候再通过`setXXX`进行赋值

```java
// Person类
public class Person {
    private String name;
    private int age;
    private Person father;		// 自己引用自己
    private Person mother;
    private Person[] children;
    public Person(String name, int age){     // 构造方法
        this.name = name;
        this.age = age;
    }
}

// main函数里面
Person p1 = new Person("xiaowang", 24);
Person p2 = new Person("laowang", 56);
p1.setFather(p2);
System.out.println(p1.getFather().getName());
```

<img src="pic\image-20201121102817237.png" alt="image-20201121102817237" style="zoom: 50%;" />

**类定义之间还能互相引用**，本质上是由于某些属性不用一开始就设置值，也不是必须设置（需要才设置，可以一直为null）

ps：如果类似于C语言要求在使用之前对每个变量均初始化，那么由于互相引用，就是a要b的值，而b也要a的值，那么永远也等不到了

```java
// 文件类
public class File {
    private String name;
    private Date createTime;
    private double size;
    private Folder parent;	// 调用了文件夹类
}

// 文件夹类
public class Folder {
    private String name;
    private Date createTime;
    private Folder parent;
    private File[] files;		// 调用了文件类
    private Folder[] subFolders;	// 调用了文件夹类
    
    public double totalSize(){
        double totalSize = 0;
        if(files != null){
            for(File f: files){
                totalSize += f.getSize();
            }
        }
        if(subFolders != null){
            for(Folder f: subFolders){
                totalSize += f.totalSize();	// 涉及到了递归调用
            }
        }
        return totalSize;
    }
}
```

类之间的组合关系在Java里面表现就是（互相）引用。但是逻辑上，有两种引用关系：包含（某个类从属于某个类，而不能用于其他，上下级关系）和单纯引用（类之间因为某些存在联系，但是实际上是独立存在的）

——总结：

类之间可以互相调用，本质上是由于类里的某些属性不必一开始就设置值，也不是必须设置

思想：通过类实现某个概念，而类的组合能够表达更为复杂的概念以及概念之间的关系

### 3. 代码的组织机制

问题引出：这么多代码，我们要如何组织才能避免命名冲突？如何使用第三方库？如何将各种代码和调用的库连接起来编译链接成一个完整的程序？

#### 3.1 包的概念——解决命名冲突

**Java中管理类和接口的方式——包**，类似于文件夹，类和接口都放在包内，包代表一个层次结构。

包有包名，以点号 . 来分隔上下级的层次关系，eg：String类，所在的包就是`java.lang.String`（java是上层包，lang是下层包）。带完整包名的类名：**完全限定名**，eg：java.lang.String

要注意：**同一个项目中，不可以有相同的包名（包括引用的库的包名也算）**，所以命名尽量不要和库一样——因为能够正常编译，但是运行时会有包冲突并且抛出异常：` java.lang.SecurityException: Prohibited package name: java.lang `

ps：**Java API中的所有类和接口都存放在java/javax下的，Java是标准包，javax是扩展包**

- 声明类所在的包

  （没有声明，就存放默认包下——但是不建议）

  在文件的最前面（声明之前不能有除注释外的其他语句），先声明包

  注意：**包名必须和文件目录结构相匹配**，如果不匹配就会编译错误

  ```java
  package com.amyZhoucc;		// 声明了包，com/amyZhoucc
  ```

  Java中包名的命名惯例是：使用域名作为前缀，且是前缀的反序来定义（因为域名是唯一的，不会存在包名的冲突）

  在包的机制下：**同一个项目的所有代码都有一个相同的包前缀，包前缀是唯一的，也不会和其他代码重合**，而项目内部继续细分子包，然后下面继续分——可以**方便模块化开发**

- 通过包使用类

  同一个包下的类可以互相引用而不需要包名，如果是不同的包下，必须要通过包找到类从而引用。有两种方式：**通过类的完全限定名（地址全称）**；**将用到的类引入当前类**。

  ps：java.lang下的包不需要导入可以直接使用，eg：String类、System类

  ```java
  // 使用完全限定名——就是在调用该类的时候，前面要加上它的全部包名
  int[] arr= new int[]{3, 5, 1, 6, 2};
  java.util.Arrays.sort(arr);
  System.out.println(java.util.Arrays.toString(arr));
  
  // 类引入——就是在package之后improt需要的包
  // import需要放在package之后类定义之前
  import java.util.Arrays;
  public class Main {
      public static void main(String[] args) {
          int[] arr= new int[]{3, 5, 1, 6, 2};
          Arrays.sort(arr);			// 那么可以直接通过类名进行调用
          System.out.println(Arrays.toString(arr));
      }
  }
  ```

  ps：import操作，还可以一次性引入某个包的所有类，语法是用(.* ) ，eg：`import java.util.*;`但是这个包不能递归，**只导入了该包下的直接类，而不包含其嵌套包下的类**，例如`java.util.zip`包下面的类不会被导入。也不能嵌套引入：例如`java.util.*.*;`

  注意：在类中对其他类的引用必须是唯一的，不能引入重名的类，如果有，import只能引入一个，而其他的必须要通过完全限定名使用

  还有，有个静态导入，使用static关键字，可以导入指定类的public的类方法和类变量

  ——但是静态导入不能过多使用，因为不知道其所属的类

  ```java
  // public final static PrintStream out = null;
  import static java.lang.System.out;		// 导入的是个类变量
  public class Main{
      public static void main(String[] args){
          out.println("hello world");		// 可以直接使用out变量（还是需要些具体的类变量、类方法，但是不用写前面的包了）
      }
  }
  ```

- 包范围可见性

  **private（类内部可见） < 默认（包范围可见）<protected（子类可见+包范围可见）<public**

  注意这边的包范围是指，同一个直接包，而子包下面的类并不能访问

#### 3.2 jar包

**打包**，我们可以将自己写的代码分享给别人，也可以使用别人分享的代码——就是通过jar包

**打包的是编译后的代码**，而不是源代码。打包可以**将多个编译后的文件打包为一个文件**。

Java中，就是将一个或多个.class文件打包成一个.jar文件——这个就是jar包（.java文件编译之后就会产生一个.class文件）

具体打包指令：

```
jar -cvf <希望输出的包名>.jar 待打包的文件

// 举例
jar cvf test.jar points.class line.class  // 就会将两个文件打包成一个jar文件
jar cvf test.jar *			// 将当前目录下的所有文件都打包
jar cvf test.jar */.		// 打包多级目录

// -cvfm 是需要指定jar的清单文件的MANIFEST.MF
```

Java的类库和第三方库都是通过jar包形式提供的，如果要使用，将其加入类路径（classpath）即可

首先需要了解java文件是如何运行的

#### 3.3 程序的编译和链接

源代码 -> 运行的程序，主要有两步：编译（编译器完成）+ 运行（JVM完成）（具体看[《Java的额外知识》chap2](http://note.youdao.com/noteshare?id=9f8ab6eaf048c06b8c823ebe1c7f0b8c&sub=38FCCF96A0C74660A8B19E1140BA449E)）

编译是通过编译器将源代码.java文件变成.class的字节码——通过运行**javac指令**完成

运行是JVM（Java虚拟机）——通过运行**java指令**完成，它通过解析.class文件，然后将其转换为二进制代码，然后运行——所以是，边解释边运行的

而**链接：就是在执行的时候，根据引用到的类加载相应的类对应的字节码，并且执行**。（因为除了开始要用到的那个类，就是指令`java MainApp`中指定的类，其他类都是需要用到的时候才会进行**链接**）

Java在编译和运行的时候，需要以**参数指定`classpath`，就是类路径**，它是用来**指示JVM如何搜索class**，就是知道要加载哪个类，那从哪里去加载该类呢？——这个就需要`classpath`了。

类路径可以有多个。Windows系统下，多个路径可以用`;`分隔；其他系统用`:`分隔

对于直接的class文件，路径就是class文件的根目录；对于jar包，路径就是jar包的完整名称（路径+jar包名）

```assembly
// 如下的路径下都存放了同名的类，会从前向后搜索匹配，找不到就下一个，全部找不到就报错
C:\work\project1\bin;C:\shared;"D:\My Documents\project1\bin"	// windows下面
/usr/shared:/usr/local/bin:/home/project/bin			// Linux下面
```

编译时：编译器会确定每个引用类的**完全限定名**，是根据`import+classpath`来确定的。所以存在：如果导入的是完全限定名，那么直接确定；如果是模糊导入，eg: import xxx.*，那么就需要找到对应的父包，然后在父包下面看是否存在需要的类。如果**多个模糊导入都存在相同的类名文件，会出现编译错误**。——import就是在此时确定完全限定名的

运行时：根据类的完全限定名进行查找并加载，寻找的方式就是在类路径中找。如果是**.class文件就在根目录下面找**；如果是**jar包，先解压后找**。——运行时，就只是根据完全限定名进行查找

## （二）类的继承

类之间可以组合，还可以**分类**——从某个根开始不断向下细分，形成一个层次分类体系

分类的形象举例：生物界能分为植物界、动物界，而动物又能继续细分下去，就是一个层次分类体系。

而计算机中对象的分类关系表示为**继承**（更细的类继承自大类）。继承中，有**父类、子类**之分（又称，基类和派生类），父类和子类是相对的（不是绝对的）

继承：子类继承了父类的属性和方法，**父类有的属性和方法子类都有**（类似于老虎继承自动物，那么就有动物的特性了），并且**子类可以增加其特有的**属性和方法（类似于老虎有斑纹，是动物没有的属性）

### 1. 基本概念

#### （1）根父类Object

平时在写的时候，没有涉及到父类子类继承，实际上，**每个类都有一个父类Object**，该类没有定义属性，但是定义了一系列方法，如下：这些方法都不要额外导入包，就可以直接使用的，并可以对其进行重写

<img src="pic\image-20201125195445079.png" alt="image-20201125195445079" style="zoom: 50%;" />

例如：toString方法：

```java
// Object
public String toString() {
	// 获得当前对象的类名，后面是返回对象的hash值——默认都是对象的内存地址值，并且是按16进制返回的
	return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

子类可以将该方法**重写**：就是定义和父类一样的方法（同方法名、同传参），但是具体的实现是不同的，这用来反映不同的实现。

```java
// 在Point类中重写toString方法，体现其特色
@Override
public String toString(){
    return "(" + x + ", " + y + ")";
}
```

举例：图形类，下面有子类圆形类、直线类，直线类下面还有带箭头的直线类

```java
// Shap类
public class Shape {
    private static final String DEFAULT_COLOR = "black";	// 注意必须是static（类变量）
    private String color;
    public Shape(){
        this(DEFAULT_COLOR);	// 直接调用下面的构造函数
    }
    public Shape(String color){
        this.color = color;
    }
    public String getColor() {
        return color;
    }
    public void draw(){
        System.out.println("draw shape");
    }
}

// Circle类
public class Circle extends Shape{		// 注意extends
    private Point center;		// Circle特有的属性
    private double r;
    public Circle(Point center, double r){	// 注意构造方法只初始化了circle特有的属性
        this.center = center;
        this.r = r;
    }
    @Override
    public void draw(){
        System.out.println("draw circle at " + center.toString() +
                           " whit r = " + r + 
                           ", using color: "+ getColor());
    }
    public double getArea(){		// 子类特有的方法
        return Math.PI * r * r;
    }
}

// Line类
public class Line extends Shape{
    private Point start, end;		// Line特有的属性
    public Line(Point start, Point end, String color){
        super(color);			// 注意super
        this.start = start;
        this.end = end;
    }
    public Point getStart(){
        return start;
    }
    public Point getEnd(){
        return end;
    }
    @Override
    public void draw(){
        System.out.println("draw line from " + start.toString() +
                           " to " + end.toString() + 
                           ", using color: "+ super.getColor());	// 注意super
    }
    public double length(){
        return start.distanceToAnother(end);
    }
}

// ArrowLine类
public class ArrowLine extends Line{
    private boolean startArrow;
    private boolean endArrow;
    // 注意构造函数
    public ArrowLine(Point start, Point end,String color, 
                     boolean startArrow, boolean endArrow)
    {
        super(start, end, color);
        this.startArrow = startArrow;
        this.endArrow = endArrow;
    }
    @Override
    public void draw(){
        super.draw();
        if(startArrow) System.out.println("draw start arrow");
        if(endArrow) System.out.println("draw end arrow");
    }
}
```

在代码中的知识点：

1. 子类继承，使用了`extends`（注意有s！！！）关键字，一个类最多只能有一个父类

2. 子类不能直接访问父类的**私有属性和方法**，但是能通过public的方法间接访问该内容。eg：circle不能直接访问父类的color，但是可以通过`getColor()`访问

3. 但是，子类继承了父类的非私有属性和方法，比如子类能够使用`getColor()`

4. 并且，在执行子类的构造方法时，会**先执行父类的默认构造方法（就是不带参数的那个）**，后执行子类自己的构造方法

5. super关键字，是指代父类的，可以调用父类的构造方法，访问父类的属性和方法——特定调用父类的方法（而不是隐式的）

   注意的是：

   - 在子类的构造方法中，如果可以用`super(xxx)`调用父类的构造方法，要放在第一行（和this一样）
   - `super.xxx`可以调用父类的方法，如果子类不存在对该方法的重写，可以直接使用；如果存在重写（就是冲突），而又想去调用父类的方法，必须用super
   - super只能引用父类非私有的属性和方法

   和this类似，但是本质上又有区别：

   this引用对象，而该对象是实际存在的，**this可以做参数、做返回值等**；但是**super只能是关键字，不能做参数or返回值**，它只是告诉编译器要调用父类的方法或属性

6. 注意：因为在使用子类的构造函数时，会默认调用父类的构造函数，如果父类不存在空参数的构造函数，那么子类必须要显式去调用父类的构造函数，即`super(xxx)`；如果存在，那么就让父类默认去调用空参数的构造函数即可

#### （2）继承的优势之一：统一处理不同子类对象

可以将每个对象都当作父类对象，然后**调用方法就会默认去调用子类的重写方法**（如果有重写的话，没有就调用父默认父类的对象）

```java
// 图形管理类
public class ShapeManager {
    private static final int MAX_NUM = 10;
    private Shape[] s = new Shape[MAX_NUM];		// 构建一个Shape对象数组
    private int shapeNum = 0;
    public void addShape(Shape shape){			// 向数组里面添加shape对象
        if(shapeNum < MAX_NUM){
            s[shapeNum++] = shape;
        }
    }
    public void draw(){					// 调用对象的draw方法
        for(int i = 0; i < shapeNum; i++){
            s[i].draw();		// 会根据对象的具体类，来调用特定的方法（可能是重写的，或者默认的）
        }
    }
}


ShapeManager sm = new ShapeManager();// 建对象

Point center = new Point(2, 3);
double r = 4.0;
Circle circle = new Circle(center, r);
sm.addShape(circle);

Point start = new Point(1,2);
Point end = new Point(2, 4);
Line l = new Line(start, end, "red");
sm.addShape(l);

Point alStart = new Point(3,4);
Point alEnd = new Point(6,8);
ArrowLine al = new ArrowLine(alStart, alEnd, "green", false, true);
sm.addShape(al);

sm.draw();
```

一些概念：

1. 可以看到，我们将**子类对象赋值给父类引用变量**——**向上转型**（转换为父类型），比如`circle、l`等对象赋值给shape变量

2. **父类型变量shape可以引用任何子类型对象**——**多态**，就是**一种类型的变量能够引用多种实际类型的对象**。

   变量shape有两种类型：

   - 静态类型：类型Shape，本来定义的类型

   - 动态类型：实际引用的对象类型

     那么动态类型下调用动态类型对应的方法，就是动态绑定

   why出现多态和动态绑定？

   $\because$构建对象的代码（对象的定义，例如Circle对象定义的地方）和操作对象的代码（对象的使用，例如ShapeManager）不在同一个文件中实现，而操作代码只知道操作的对象的父类型，而不需要知道具体有哪些子类型

——**多态和动态绑定也是计算机的一种重要思维方式，使得操作对象的程序不需要关注对象的实际类型，而统一处理不同的对象，但是又能实现每个对象的特有行为**

——总结：

<img src="pic\image-20201126105016057.png" alt="image-20201126105016057" style="zoom: 50%;" />

### 2. 继承的细节

#### （1）构造函数

前面已经注意到了，调用子类去创建子类实例的时候，会去调用子类的构造函数，而**子类的构造函数会先去调用父类默认的构造函数**

那么如果父类不存在默认构造函数呢（已经自定义了一个带参数的构造函数）？——需要**显式的调用父类某个构造函数**，用`super(xxx)`，否则就会编译错误

特殊情况——这个只会出现在很恶心的细节考试里面：

```java
public class Base {			// 基类
    public Base(){			// 构造函数里面调用了一个可被重写的函数
        test();
    }
    public void test(){
        System.out.println("base");
    }
}

public class Child extends Base{		// 子类
    private int a = 123;
    public Child(){			// 构造函数首先会去调用父类的构造函数
    }
    public void test(){		// 重写了父类的public方法
        System.out.println(a);
    }
}

// main函数里面：
Child child = new Child();
child.test();

// 输出：
0
123
```

概括就是：父类的构造函数里面调用了可被重写的方法，而该方法又被子类给重写了，那么在子类调用构造函数时，会去调用父类默认的构造函数，而父类的构造函数要调用**被子类重写的方法**，**此时子类的实例变量的赋值语句和构造方法还未被执行**（那么实例变量的赋值是啥时候呢？？?），所以输出为0

——这不是一个好方法！**父类构造函数调用的函数应该private方法**。

#### （2）重名与动态绑定

复习：如果子类重写父类的方法（非private），那么在子类对象调用该方法时，会**动态绑定**——就是执行该重写的方法而不是父类的方法。（其实重写就是重名，类之间存在相同名字的变量、方法等，但是实现不同）

——实例方法都是动态绑定，子类不管静态类型如何，都是去调用子类的重写方法

重名的情况：——以下针对**实例变量、静态方法、静态变量也是可以实现重名的情况**

1. 如果是private变量和方法，那么不存在问题——因为只能在类里面访问private变量和方法
2. 如果是public变量和方法
   - 如果是在**类里面访问**，比如类里面的一个函数去访问该方法、该变量，**那么就是当前类的**。但是子类可以super显式的访问父类的同名
   
   - 如果是在类外面访问，就需要看调用的静态类型是啥
   
     - 如果静态类型是父类，那么访问父类的方法和变量（这边的静态类型我的理解是：引用变量的类型）
     - 如果静态类型是子类，那么访问子类的方法和变量
   
     ——这个就是**静态绑定**（就是访问到的是变量的静态类型，在赋值操作的左边类型）
   
     ```java
     // 基类
     public class Base {
         public static String baseS = "static base"; // 静态变量
         public String s = "base";       // 实例变量
         public static void BaseFun(){
             System.out.println("base static fun");
         }
         public void BaseFun2(){
             System.out.println("base fun2");
         }
     }
     
     // 子类
     public class Child extends Base{
         public static String baseS = "static child";	// 重名（类似于重写）
         public String s = "child";		// 重名
         private int a = 12;
         public static void BaseFun(){		// 重名
             System.out.println("child static fun");
         }
         public void BaseFun2(){			// 重写（实例方法）——动态绑定的
             System.out.println("child fun2");
         }
     }
     
     // main
     Child c = new Child();	// 静态类型为Child类
     Base b = c;		// 静态类型为Base类——通过b访问的就是父类的对象、方法了（除了实例方法）
     
     System.out.println(c.baseS);	// 调用静态变量：static child
     System.out.println(c.s);	// 调用实例变量：child
     c.BaseFun();		// 调用静态方法：child static fun
     c.BaseFun2();		// 调用实例方法：child fun2
     
     System.out.println(b.baseS);	// static base
     System.out.println(b.s);	// base
     b.BaseFun();		// base static fun
     b.BaseFun2();		// 调用实例方法：child fun2（动态绑定）
     
     Base base = new Base();
     System.out.println(base.baseS);		// static base
     System.out.println(base.s);	// base
     base.BaseFun();		// base static fun
     base.BaseFun2();	// base fun2——因为就是父类对象，所以不可能去调用子类的方法or变量
     ```
   
     ps：静态变量or方法都可以直接通过类名来调用，也可以通过实例对象访问
   
     **静态绑定在编译阶段完成；动态绑定在运行时才完成**
   
     **实例变量、静态变量、静态方法都是通过静态绑定的**

#### （3）重载和重写的区别

重写：之前接触到的实例方法就是重写操作

​			特点是：子类和父类的方法完全一样：方法名、方法参数、方法返回值都是一样的；就是**实现不一样（内部不一样）**。且在父子两个类中存在的

**重载：方法名称一样，但是参数不一样**——可能参数个数、类型、顺序不同。可以只在一个类中出现重载情况。

那么出现的问题是：对于同一个方法名有不同的实现，那么在参数调用的时候是如何选择（执行哪个方法）的呢？

当出现多个重载方法时，首先按照参数类型进行匹配——**寻找所有重载版本中最匹配的，然后再选择进行动态绑定的**

举例：

```java
// 基类
public class Base {
    public void sum(int a, int b){
        System.out.println("base fun");
    }
}
// 子类
public class Child extends Base{
    public void sum(long a, long b){
        System.out.println("child fun");
    }
}
// main方法调用
Child c = new Child();
int a = 2, b = 3;
c.sum(a, b);		// base fun——因为int类型和基类方法更匹配，所以选择基类的

long d = 3, e = 4;
c.sum(d, e);		// child fun——因为long类型和子类方法更匹配，所以选子类的
```

#### （4）父子类型转换

前提：子类的对象可以赋值给父类的引用变量——向上转型

那么也存在向下转型：但是有条件的——**父类变量去引用的对象必须是子类对象or子类对象的子类**——那么能将该父类的引用变量赋值给子类的变量。

举例：

```java
Base b = new Child();
Child c = (Child)b;		// 强转成功

Base d = new Base();
Child e = (Child)d;		// 强转失败——因为d的动态类型就是Base

// 关键字instanceof，可以判断b是否属于Child类型，返回值是true（是）/false（不是）
b instanceof Child;	
```

#### （5）继承访问权限（关键字：protected）

前提：

- public表示该变量/方法可被外部类访问（任何地方均能直接访问，如果是静态的话那就是类名+静态名；如果是成员变量or方法，那么创建对象之后再通过对象名+成员变量名/方法访问）
- private表示该变量/方法只能被类内部访问（即使创建了对象，也不能通过对象名+变量or方法名进行访问——因为不在类的内部）
- default（默认）表示包内可见，就是同一个包内能通过类名（针对静态的变量or方法）/对象名访问

现在增加一个关键字**protected**——表示**不能被外部类访问，但是能被子类（即使子类在其他包内）访问**。并且变量和方法都能**包内可见**。

具体来说：

- 基类的protected成员是包内可见的，也包括在同一个包内的子类
- 如果子类和基类不在同一个包内，那么子类可以**访问从基类那边继承得到的protected成员，但是不能访问基类的protected成员**

——关键是要确定：该protected成员出自哪个地方，然后其可见性范围，然后再判断其可行性。

[发现宝藏，可以看看有醍醐灌顶之效，尤其时面对clone场景](https://www.cnblogs.com/liuleicode/p/4946248.html)，[Java的额外知识chap2](http://note.youdao.com/noteshare?id=9f8ab6eaf048c06b8c823ebe1c7f0b8c&sub=38FCCF96A0C74660A8B19E1140BA449E)

```java
public class Base {
    protected int currentStep;
    protected void step1(){}
    protected void step2(){}
    public void action(){
        this.currentStep = 1;
        step1();
        this.currentStep = 2;
        step2();
    }
}

public class Child extends Base {
    protected void step1(){
    	System.out.println("child step " + this.currentStep);
    }
    protected void step2(){
    	System.out.println("child step " + this.currentStep);
    }
}
```

这个是protected常用的一个场景。

**设计模式之一：模板方法——父类只提供了一个模板方法，子类来具体实现，而protected就是在里面常见。**

#### （6）可见重写性

容易忽视的一点：重写时，**子类重写的方法的可见性>父类方法的可见性**，即：如果父类方法是public，那么子类的方法必须是public；父类的方法是默认的，那么子类的方法至少是default/public；以此类推——子类方法的权限可以升级，但是不能低于

原因：父类和子类的关系，是从属关系，父类表示更广泛，而子类细节更多，类-似于”动物-老虎“的关系，**子类属于父类，所以子类要能支持父类所有的对外行为**，所以就是**` is a`**的关系，eg：老虎 is a 动物。

可见性变小了，那么支持的对外行为变少了——逻辑上不允许

#### （7）防止继承（关键字：final）

这边指的是，**父类整个不让其他类继承**——通过关键字：final实现，加了关键字final的类就无法被继承

用法：——当final的限定在于类上

```java
public final class Father {	// 整个类不能被继承
}
```

ps：final也能用在变量、方法上

- 修饰方法

  不允许子类在继承的时候被修改（即不能重写等），类的**private方法会被隐式地指定为final方法**

- 修饰变量

  用在此的最多，**如果修饰的是基本数据类型变量，那么一旦初始化就不能更改；如果修饰的是引用类型变量，那么初始化之后不能指向其他对象**

   <img src="https://images0.cnblogs.com/i/288799/201407/091114508325258.jpg" alt="img" style="zoom: 80%;" />

   

### 3. 继承实现的基本原理（未完）

首先明确：

一个类的信息主要包括如下：

1. 静态变量：static修饰的变量
2. 类初始化代码
   - 静态初始化代码块，eg: `static{ System.out.println(xxx); xxx}`
   - 定义静态变量时的赋值语句，eg: `public static int a = 2;`
3. 静态方法：static修饰的方法
4. 实例变量
5. 实例初始化代码
   - 实例初始化代码块，eg: `{System.out.println(xxx); xxx}`
   - 定义实例变量时的赋值语句，eg: `int a = 2;`
   - **构造方法**
6. 实例方法
7. **父类信息引用**

类加载过程：分配内存保存类的信息 -> **给静态变量赋默认值** -> 加载父类 -> 设置父子关系 -> 执行类初始化代码



### 4. 继承是把双刃剑

继承的优点：Java的类库、API中使用继承，我们能够使用基类和基础公共代码，并且对其进行继承、重写等以满足特殊的需要。

#### （1）继承会破坏封装

封装就是隐藏实现的细节，只提供接口——就是说我只需要知道如何用，至于细节不需要知道——**程序设计的第一原则**，函数是封装、类是封装。

但是继承不只是使用而且可能会重写该封装的函数，就会影响其封装的效果。$\because$父子类之间可能存在着实现细节的依赖，导致不得不在子类重写方法的时候关注其内部实现；而父类修改代码的时候也需要考虑到子类的实现效果是否会发生变化。

对于子类而言，如果父类修改实现细节，那么子类的功能实现可能会被破坏（封装失败）；对父类而言，子类继承并重写了，对于父类本身无影响，但是失去了修改父类内部实现的自由性。

```java
public class Base {
    private static final int MAX_NUM = 1000;
    private int[] arr = new int[MAX_NUM];	// 定义一个长度为1000的数组
    private int count;			// 当前数组使用计数
    public void add(int number){		// 数组尾部增加一个数
        if(count<MAX_NUM){
        	arr[count++] = number;
        }
    }
    public void addAll(int[] numbers){		// 将另外一个数组中的所有数都加入到该数组中
        for(int num : numbers){		
        	add(num);			// 调用add封装
        }
    }
}

public class Child extends Base {
    private long sum;		// 计算数组元素和
    @Override
    public void add(int number) {		// 重写add方法
        super.add(number);				// 使用父类的add单元素方法
        sum+=number;			// 并重新计算和
    }
    @Override
    public void addAll(int[] numbers) {		// 重写addAll方法
        super.addAll(numbers);			// 使用父类的addAll方法——注意会去调用add方法
        for(int i=0;i<numbers.length;i++){
        	sum+=numbers[i];			// 计和
        }
    }
    public long getSum() {
    	return sum;
    }
    public static void main(String[] args) {
        Child c = new Child();
        c.addAll(new int[]{1,2,3});			// 12- error
        System.out.println(c.getSum());
	}
}
```

结果是12，因为子类在使用addAll的时候去调用了父类的addAll，而父类addAll调用了add方法，而子类的add被重写了，就去动态绑定了重写的方法，所以就是加了两遍1、2、3

抽象来说：子类重写中调了父类的方法（super），父类调用过程中发现子类重写了函数中调用到的方法，动态绑定了子类另一个重写的方法（去执行子类的方法，而不是原来父类实现的逻辑了），然后**父类的封装就被破坏了**。

——所以，子类要知道父类的实现方法才能正确的进行重写 

而如果父类对`addAll`和`add`进行实现修改的话——不改变其效果，eg：addAll不使用add方法，子类的实现效果就又会变化。

——父子之间存在着**实现细节的依赖**，所以子类不能随意重写父类方法、父类方法也不能随意修改实现。

——**子类需要知道父类的可重写方法之间的依赖关系，并且父类不能随意改变**，eg：`addAll`和`add`之间的关系

——并且**父类不能随意增加非final方法**，因为父类增加，子类也都会继承，而子类存在不同的逻辑，**需要重写该方法才能保证逻辑正确性**。

#### （2）继承没有体现“is-a”原则

原因是：父类有的属性不一定都适用于子类（应该是在设计是，没有兼顾到所有情况）；子类重写的方法不一定符合父类的预期（不受父类控制）

eg：鸟类有一个属性`fly()`，而企鹅也是鸟类，但是没有`fly()`属性；企鹅这个类可能针对`fly()`这个属性实现了游泳等行为

——所以，如果想统一处理不同子对象，即都用同一个父类引用对象表示，那么会引起混乱（有父类的方法，但是行为可能不符合要求）

#### （3）如何处理

1. **使用final**避免继承方法/类，那么父类就能随意修改方法实现/类实现（那么不用担心一个父类引用变量实际指向了完全不符合行为预期的子类对象）

2. 优先**使用组合**而非继承，在类里面使用其他类的对象，那么无细节依赖。存在的问题是：子类对象不能当作基类对象统一处理。——使用接口解决

3. 正确使用继承

   - 有父类，自己来使用子类

     => 重写的方法不能改变本来的预期行为，特别是方法之间的依赖

     => 阅读修改说明，再来修改子类

   - 写父类给别人用

     =>需要抽象出真正公共的属性作为父类

     =>将一些不能重写的方法设置为final（可以调用但是不能继承）

     => 写说明文档，说明可重写方法的实现机制和方法的依赖性；如父类要修改会影响到子类的地方需要指明

   - 父类、子类全包

## （三）类的扩展

前情：Java有8种基本数据类型；其他的数据类型都是通过**类、类之间的组合、类的继承**来实现了引用数据类型，从而表达各种对象。

本章主要讲：接口、抽象类、内部类、枚举（类C语言的enum）

### 1. 接口

之前都将类看作数据类型，然后在该数据类型下还实现了一些操作。但是还存在情况：**并不在乎类型，而在乎是否实现了某些功能**——有某种能力

$\therefore$那些不在乎类型，而在乎能力的（实现某个具体方法）场景，可以用接口

#### （1）定义接口

概念定义：**接口就是声明了一组能力（方法），但是自己并没有实现**——只是一个约定/协议

接口涉及两方对象：一方实现该接口，另一方使用该接口，**双方对象不直接依赖，而通过接口间接交互**

<img src="pic\image-20201207000753812.png" alt="image-20201207000753812" style="zoom: 50%;" />

eg：USB接口，它规定了USB设备都需要实现的能力，而计算机是通过USB接口来使用这些USB设备的，两者不直接依赖。

Java定义：关键字：**interface**来声明接口，后面紧跟接口名

```java
public interface Mycomparable{
    int compareTo(Object other);	// 参数是Object对象，返回值是-1表<；0表=；1表>
}
```

分析：

1. interface来声明接口（和class对应），前面的修饰符一般都是**public**（如果是private表示其他都不可见，那么该接口并无意义）

   所以格式是：`public interface 接口名{}`

2. 接口里面**声明了一个方法**，但是没有方法体（Java8之前不给实现），接口不需要添加修饰符，默认就是`public abstract`

   所以，声明的方法格式为：`(public abstract) 返回值 函数名（参数列表）;`

=> 接口本身的定义并没有实现什么，而是要看实现者和调用者

#### （2）实现者、调用者

类去实现接口——表示该对象有该接口所表示的能力

```java
public class Point implements Mycomparable{		// 表示实现接口
    private int x;
    private int y;
    public Point(int x, int y){
        this.x = x;
        this.y = y;
    }
    public double distanceToZero(){
        return Math.sqrt(x*x+y*y);
    }
    public int compareTo(Object other){		// 实现接口中声明的方法
        // 如果类型不匹配，就抛出异常
        if(!(other instanceof Point)) 
            throw new IllegalArgumentException();     
        Point otherPoint = (Point) other;       // 将Object对象强转为Point对象
        double distance = distanceToZero();
        double other_distance = otherPoint.distanceToZero();
        if(distance - other_distance < 0) return -1;
        else if(distance - other_distance == 0) return 0;
        else return 1;
    }
}
```

分析：

1. 类可以实现某个接口，表示该类对象有该接口所要求的能力，实现就需要紧跟类名之后添加`implements 接口名`，类可以实现多个接口（表示类对象有多种能力），接口之间通过`,`分隔

   所以格式如下：`public class 类名 implements 接口名1, 接口名2{}`

2. 既然定义了类中实现了某个接口，那么一定要在类内部实现接口中声明的方法。注意**修饰符（因为接口默认是public abstract，所以实现一定要是public）、返回值、函数名、参数都需要一致**



接口使用：**接口不能直接new一个接口对象**，对象只能通过类来创建。但是可以声明接口类型的变量，来**引用实现接口类的对象**

如果一个类实现了多个接口，那么它能被多个接口变量引用。

但是该**接口变量只能调用接口方法**（而不是实际类对象有的方法）

```java
Mycomparable a = new Point(2, 3);// 可以引用实现接口功能的对象
Mycomparable b = new Point(4, 5);
a.compareTo(b);			// 通过接口去调用
```

但是，上面的接口调用并不能很好的反映接口的作用。

接口有用的地方：某些方法实现并不知道具体的对象类型，而是统一对Object对象（实现了某些接口功能）进行操作。 如下：

```java
public class CompUtil {
    // 求出数组中的最大值——可以针对任何Mycomparable类型的对象处理
    public static Object max(Mycomparable[] objects){	// 对实现了接口的对象数组操作
        if(objects == null || objects.length == 0) return null;
        Object max_value = objects[0];	// 对于所有元素都统一当对象处理（而无指定实际的类对象）
        for(int i = 1; i < objects.length; i++){
            if(objects[i].compareTo(max_value) > 0)	// 调用接口方法
                max_value = objects[i];
        }
        return max_value;
    }
    public static void main(String[] args){
        Point[] points = new Point[]{
                new Point(1, 2), new Point(2, 3), new Point(3, 4),
                new Point(4, 5), new Point(5, 6)};
        System.out.println(CompUtil.max(points));
    }
}
```

——该类就是针对**接口的编程**——对接口的使用，不关心具体的类型，统一当作Mycomparable对象处理。实现了对**任意Mycomparable接口类型的某个操作**

——**针对接口而非具体的对象类型进行编程，是计算机程序的重要思维**

优点：**代码复用**，不同类型的对象可以使用同一套代码；

​	  	 **降低耦合，提高灵活性**。使用和实现通过接口间接交互。使用者只在乎接口本身，那么实现者可以修改接口实现而不会影响使用者；

#### （3）接口细节

一些接口小概念

1. 接口变量

   接口中可以定义变量，默认是`public static final`类型（公开静态常量变量），可以不写明。可以通过类名直接使用：

   ```java
   public interface Test{
       public static final int a = 0;	// 修饰符可以不写，默认就是这个
   }
   ```

2. 接口继承

   **接口可以继承其他接口，并且可以继承多个接口**，格式如下：

   ```java
   public interface interface1 extends interface2, interface3{		// 可继承多个接口
       
   } 
   ```

3. 类继承和接口可以同时存在

   即类可以extends继承一个父类，之后又implements多个接口，格式如下：

   ——需要注意：继承只能有一个，实现可以有多个。且extends要在implements之前。格式如下：

   ```java
   public class Son extends Father implements interface1, interface2{
      
   }
   ```

4. 使用instanceof来判断一个对象是否实现了某个接口

   ```java
   Point p = new Point(2, 3);
   if(p instanceof Mycomparable){
       System.out.println("comparable");
   }
   ```

#### （4）接口代替继承

继承的优势：代码复用+利用动态绑定和多态来统一处理多种子类对象

组合：复用代码，但是不能统一处理

接口：能统一处理对象，但是接口本身没有实现代码所以无法代码复用

那么组合+接口就能实现继承的功能。

eg：——不用担心破环封装，Base和Child里面两个函数是独立的两个实现，细节无交叉

```java
public interface AddFun{
    void add(int number);
    void addAll(int[] numbers);
}

public class Base implements AddFun{	// 实现了接口
    
}

public class Child implements AddFun{	// 实现了接口，能够统一处理
    Base base;		// 组合使用Base对象——使用Base方法（代码复用）
    xxx
}
```

#### （5）Java8和Java9的接口增强

Java8之前接口中所有方法只能声明不能实现，Java8允许实现：两类方法——**静态方法和默认方法**，两种方法均能在接口中有实现

1. 静态方法：修饰符static就是静态方法（普通方法默认是`public abstract`）

   ```java
   public interface Mycomparable {
       public static void test(){
           System.out.println("static interface function");
       }
   }
   ```

   **可以直接通过`接口名 . 方法名`调用**——这样静态方法和普通方法就能写在同一个接口中，eg：Comparator接口就多了静态方法；而Collection接口就单独实现了一个类Collections来存放静态方法

2. 默认方法：显式写明`default`，因为默认是`public abstract`

   ```java
   public interface Mycomparable {
       default void test2(){
           System.out.println("default function");
       }
   }
   ```

   默认方法自带实现，而实现类对这些方法也可以**重新实现，也可以不改变**。地位和普通方法一样，不能通过接口直接使用。

   目的：函数式数据处理需求，主要是**为了方便给接口增加功能**

   存在一种场景：该接口增加一个方法，那么实现该接口的所有类都需要添加关于该方法的实现——工作量太大；而现在存在默认方法，接口增加了方法，而类不一定需要去实现（存在于函数式数据处理中，需要给接口增加新方法）

   Java8中默认方法、静态方法都需要public；但是Java9允许private——主要是为了在接口中给多个静态/默认方法提供代码复用（但是又不希望给外部使用）

=> 总结：接口针对的场景是：关心是否有某个功能，而不是类型；针对接口编程是重要的程序思维，它能够代码复用；减少代码耦合，提高灵活性，分解复杂问题的工具（将复杂问题分解为小问题，而小问题之间的耦合性尽量少，独立性强，那么更有助于解决问题）。

### 2. 抽象类

就是抽象的类，和具体类相对比得到的。**具体类能够有直接对应的对象，而抽象类没有具体对应的对象**，只是表达抽象的概念——所以抽象类一般是具体类的父类，eg：Shape类：抽象，表达的是图形这一类的概念；Circle类：具体的，表达圆这一对象。

#### （1）抽象方法和具体使用

定义：**本身不知道如何实现**（太过于抽象，只知道有这么一个功能，而无实际的对象可供操作），**只有子类才知道如何实现**，eg：Shape类中的perimeter()方法：图形有计算周长的功能，但是需要对应具体的图形；Circle类就能够具体计算了。

抽象方法只有声明没有实现，具体方法有实现

```java
public abstract class AbstractMycomparable {		// 抽象类
    public abstract int compareTo(Object other);		// 抽象方法
}
```

分析：

1. 定义抽象类/抽象方法的关键词是：**abstract**（可以发现，接口的普通方法默认就是public abstract

2. **定义了抽象方法的类必须被定义为抽象类**；定义了抽象类，内部可以没有抽象方法，且抽象类内存可以有具体方法、实例变量（能够被继承使用）等

3. 抽象类和具体类的区别在于：**抽象类不能创建对象**，即不能new一个抽象对象。但是可以创建抽象变量，来引用具体类的对象（继承了即可使用）——这点和接口也类似（不能new，但是能引用）

4. 一个类在继承抽象类后，**必须要实现所有的抽象方法**（除非自己也声明为抽象类），注意实现时：返回值、函数名、参数列表必须一致——这点和接口也类似（必须要将所有的普通接口全部实现）

   ```java
   public class Point extends AbstractMycomparable{		// 继承抽象类
       private int x;
       private int y;
       public Point(int x, int y){
           this.x = x;
           this.y = y;
       }
       public double distanceToZero(){
           return Math.sqrt(x*x+y*y);
       }
       public int compareTo(Object other){		// 实现抽象方法
           if(!(other instanceof Point)) throw new IllegalArgumentException();     // 如果类型不匹配，就抛出异常
           Point otherPoint = (Point) other;       // 将Object对象强转为Point对象
           double distance = distanceToZero();
           double other_distance = otherPoint.distanceToZero();
           if(distance - other_distance < 0) return -1;
           else if(distance - other_distance == 0) return 0;
           else return 1;
       }
   }
   
   AbstractMycomparable a = new Point(2, 3);		// 抽象类变量引用具体类对象
   AbstractMycomparable b = new Point(4, 5);
   System.out.println(a.compareTo(b));
   ```

#### （2）why要用抽象类

为啥定义一个普通的类，而该类中如果不知道如何实现某个方法就直接不写实现体即可，为啥还需要单独拎出来？

$\because$虽然从语法上抽象类不是必须的。但这是Java提供的语法工具。需要引导使用者正确的使用一些类和方法，减少误用。

如果将该方法定义为抽象方法，那么*子类在继承的时候就知道必须要实现它*（否则，编译器会报错）；而如果是普通类定义了空方法体，那么从编译角度不会出现，只有在运行时调试才能发现（问题隐蔽，不易发现）

如果使用抽象类，由于不能创建对象，限定使用者只能用实现的子类创建对象，而不会对实现不完整的抽象类创建（编译器会提示错误）；而如果是普通类，那就能直接创建了

=>$\therefore$抽象类和抽象方法，是Java提供的防错机制，防止使用不完整的方法、类

#### （3）抽象类和接口

相同之处在于：内部都可能存在抽象方法（等待子类实现，也可能不存在）；在子类继承or实现时，必须将所有的抽象方法均实现了；均不能创建对象

不同：抽象类**能定义实例变量**（接口和抽象类均能定义静态变量、静态方法、实例方法），本质上还是一个类；一个类能实现多个接口，但是只能继承一个类

抽象类和接口一般是互相配合使用的：**接口声明能力，抽象类提供默认实现，实现部分or全部的能力**。所以一个接口一般对应有一个抽象类：

<img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20201209100842749.png" alt="image-20201209100842749" style="zoom: 50%;" />

所以对于具体类来说，既可以实现接口的所有方法——工程量较大，但是当类已经有其他父类继承的话，只能用实现接口的方法；也可以继承抽象类，然后根据需要重写方法——代码可以复用

抽象类可以”实现“接口——选择全部实现、部分实现、一个都不实现（在抽象类里面甚至不用写明未实现的抽象方法，直接交给子类去实现抽象方法即可），子类继承的话，本质上就是`public class Son extends AbstractFather implements interface1`——实现父类未实现的接口；重写父类实现的方法

所以整个逻辑是：接口定义功能，抽象类提供默认实现，从而方便子类实现接口（本质上还是子类实现某些功能需要）

[关于abstract关键字的辨析——有助于理解接口和抽象类](http://note.youdao.com/noteshare?id=9f8ab6eaf048c06b8c823ebe1c7f0b8c&sub=38FCCF96A0C74660A8B19E1140BA449E)

### 3. 内部类

之前涉及到的（目前在写的）类都是单独一个源文件的，即单独建一个.java文件，并且文件名和类名一致——一个类就是一个文件——称为**外部类**

一个类还能放在另一个类的内部——称为**内部类**

内部类的使用场景：一个类只与某个类关系密切，而与其他类关系不大，那么可以定义在该类内部

=>从而实现**对外的完全隐藏**（将其设置为private），可以有**更好的封装性**，代码实现上也更为简洁。并且内部类可以**直接访问外部类的私有变量**

内部类是Java编译器的概念，JVM都会将内部类处理成一个单独的类，生成一个独立的.class文件——所以，一个**内部类最终都可以被替换为一个独立的类**

#### （1）静态内部类

和外部类的静态方法、静态变量地位一样，且带有关键字**static**

```java
public class Outer {       // 外部类
    private static int a = 2;
    /*************************/
    static class Inner{			 // 静态内部类，关键字static
        private static int aa = 4;
        private int b = 5;
        public static void inStaticFun(){
            System.out.println("outer_a:" + a);		// 直接访问外部类的静态变量/方法
        }
        public void inFun(){
        }
    }
    /************************/
    public void test(){
        Inner inner = new Inner();		// 直接创建静态内部类的实例，然后访问
    }
}

// 其他类中使用静态内部类
Outer.Inner inner = new Outer.Inner();
```

静态内部类和普通的类没有区别，除了放置的位置在一个类内部以外。

静态内部类，可以有自己的静态变量、静态方法、实例变量、实例方法、构造方法（**普通类有的，全部都有**）

静态内部类和其外部类联系并不是很大。

- 内部类可以**直接访问外部类的静态变量、方法**（即使是private的，这个直接是和普通的类变量被访问做比较的），不需要带上外部类名（当然带上也不会报错），但是**不能直接访问实例变量和方法**（可以在内部类里面创建外部类对象，然后进行访问——就是普通的访问方式）

- 在外部类中**直接**通过创建内部类的实例对象，然后被使用，不能直接使用静态内部类的变量or方法（这个直接是和下面的间接做比较的）；外部类可以使用`内部类名.静态变量/方法名`访问内部类的静态变量or方法（和普通的类一样）

- 在外部使用静态内部类（内部类没有设置private），那么需要带上外部类名，即`外部类.静态内部类`

  eg：`Outer.Inner inner = new Outer.Inner();`

静态内部类的实现：

（大概是如此的逻辑，可能不是完全一致，我在IDEA中运行发现class文件中内部类还是在外部类里面；但是如果在命令行中运行，就会生成一个单独的class文件）

```java
// Outer.class文件
public class Outer {
	private static int shared = 100;
	public void test(){
		Outer$StaticInner si = new Outer$StaticInner();		// 创建静态内部类对象
		si.innerMethod();
	}
    // 但在我这里看不到这个方法（可能被隐藏了吧）
	static int access$0(){		// 自动生成一个非private静态方法来访问其private静态变量
		return shared;
	}
    ...
    ...（还存在内部类的代码）
}

// Outer$StaticInner.class文件
public class Outer$StaticInner {	// 内部类名被修改了：外部类$内部类，表示和外部类的关系
	public void innerMethod() {
		System.out.println("inner " + Outer.access$0());	// 通过非private方法访问
	}
}
```

分析：

1. 内部类在编译的时候，会被单独当作一个类，单独创建一个类文件，但是类名会变化`外部类名$内部类名`——从而表明类之间的关系
2. 虽然说，外部类可以直接通过`new 内部类名()`来创建对象，但是编译之后发现，还是需要`new 外部类$内部类()`——因为分为两个文件了，不过这个步骤是编译器帮助我们完成的
3. 因为静态内部类可以直接使用外部类的静态变量和方法（即使是private的），而编译之后在两个文件中，那么不符合**私有变量不能被外部类访问**条件，那么自动在外部类中生成一个非private的方法供内部类调用（对其他类应该是不可见的？？？未求证）

使用场景：与外部类关系密切，但是不依赖于其他类，可以定义为静态内部类

eg：

<img src="pic\image-20201209154445326.png" alt="image-20201209154445326" style="zoom:67%;" />

#### （2）成员内部类

在前面的基础上，**少了`static`修饰**。

和静态内部类的区别：

1. 成员内部类可以访问外部类的**静态变量、静态方法、实例变量、实例方法**

   如果外部类变量和方法（和内部类变量和方法）重名的话，需要用到`外部类.this.xxx`指定，否则根据就近原则，选择内部类的方法or变量

2. 成员内部类中，**不能创建静态变量、静态方法**（不是一个完整的类了），但是如果是**final修饰的变量可以允许是static，因为被final修饰的变量被当成是常量**（但是final static修饰的方法还是不被允许的——表示不能被继承）

   => why不能创建静态变量/方法呢？

   *$\because$因为对外来说，成员内部类必须和外部类的实例绑定才能使用，而不能独立于外部类使用。但是静态变量/方法一般是独立使用的，所以成员内部类不适合有静态变量/方法。如果有需要的话，可以在外部类中创建静态变量/方法*

3. 成员内部类被外部类使用时，同静态内部类——直接创建对象即可

4. 成员内部类使用外部类对象时，同静态内部类——直接创建对象即可

5. 成员内部类被其他类使用时：首先要创建外部类对象，然后利用该实例对象，创建内部类对象

   ```java
   Outer outer = new Outer();
   Outer.Inner inner = outer.new Inner();	// 引用对象还是Outer.Inner，只不过new的方式变化了
   ```

   => 这点，感觉成员内部类更像是外部类的一个*实例变量*，只有外部类对象创建后才能存在实例变量的使用

   ​	 静态内部类更像是，*静态变量*角色，不用创建外部类的对象就能直接通过外部类名使用

使用举例：

```java
public class Outer {
    private static int a = 2;
    private int b = 3;
    class Inner{				// 成员内部类
        final static int c = 6;		// 可以允许final修饰的变量是static的——当作常量
        public void inFun(){		// 只能有实例方法/变量
            System.out.println(a);
            System.out.println(b);
            Outer.this.fun();		// 显式调用外部类方法——因为出现重名
        }
        public void fun(){		// 和外部类中的方法重名
            System.out.println("inner fun");
        }
    }
    public void fun(){
    }
    public void test(){
        Inner inner = new Inner();		// 可以直接创建内部类实例对象
        inner.inFun();
    }
}

// 其他类使用内部类
Outer outer = new Outer();
Outer.Inner inner = outer.new Inner();		// 关联创建
```

编译之后：

```java
// Outer.class
public class Outer {			
	private int a = 100;
    private void action() {
    	System.out.println("action");
    }
    public void test() {
        Outer$Inner inner = new Outer$Inner(this);
        inner.innerMethod();
    }
    static int access$0(Outer outer) {// 会创建两个静态方法，给内部类使用外部类的private变量/方法
    	return outer.a;
    }
    static void access$1(Outer outer) {	// 是要带参数的——因为调用的是实例方法（调用静态方法无）
    	outer.action();
    }
}

// Outer$Inner.class
public class Outer$Inner {
    final Outer outer;		// 我的编译结果没有这个
    public Outer$Inner(Outer outer){		// 构造方法会传入一个外部类对象
    	ths.outer = outer;		// 会去创建一个外部类实例，我的编译结果：this.this$0 = var1;
    }
    public void innerMethod() {
        System.out.println("outer a " + Outer.access$0(outer));//传参：this.this$0
        Outer.access$1(outer);
    }
}
```

可以发现：Inner类在构造函数中会去创建一个Outer对象，然后通过该对象去访问Outer的实例变量、方法

使用场景：内部类和外部类关系密切，内部类需要访问外部类的**实例变量、实例方法**——那么就适合

eg：一些方法的返回值可能是某个接口，为了返回该接口，该类就用成员内部类实现该接口，该内部类可以设置为private，对外完全隐藏。

<img src="pic\image-20201211103330983.png" alt="image-20201211103330983" style="zoom:67%;" />

=> 

```java
// LinkedList的内部类汇总：
private class ListItr implements ListIterator<E>{	//LinkedList里面定义了一个成员内部类
    
}

private class DescendingIterator implements Iterator<E> {		// 成员内部类，且实现接口
    
}

private static class Node<E> {		// 定义了一个私有静态内部类
    
}

static final class LLSpliterator<E> implements Spliterator<E> {	// 静态内部类
    
}
```

#### （3）方法内部类

字面意思：在方法中定义了一个类——那么**只能在定义的方法中使用该类**。

- 方法内部类能够直接访问：静态变量、静态方法——不需加任何的参数or附加条件

  如果它所在的方法是实例方法，那么还能访问外部外部的实例变量、实例方法。

  如果是所在的是静态方法，那么只能访问静态变量、方法。

- 方法内部类能够定义：实例变量、实例方法

  不能定义：静态变量、静态方法

- 方法内部类中，还能直接访问：方法中的局部变量 和 方法参数（这些变量**不用显式定义为final**，书中是这样的，但是实践之后并不需要，实际上**编译器会将方法内部类用到的局部变量or方法参数设置为final**）


使用如下：

```java
public class Outer {
    private static int a = 1;
    private int b = 2;
    private static void staticFun(){
        System.out.println("static fun");
    }
    public void fun(int param){
        final String str = "hello";	// 显式定义为final，就被处理成常量，直接替换变量名
        /************方法内部类的定义**************/
        class Inner{
            int c = 3;			// 只能定义实例变量or方法
            void test(){
                staticFun();
                System.out.println(param);		// 可以使用方法的变量or参数
                System.out.println(str);
            }
        }
        /***************************************/
        /************方法内部类的使用*************/
        Inner inner = new Inner();		// 方法内部类只能在方法内部使用
        inner.test();
    }
}
```

编译器编译之后的结果：

```java
// Outer.class
public class Outer {
    private static int a = 1;
    private int b = 2;
    public Outer() {		// 添加默认的构造方法
    }
	private static void staticFun(){
        System.out.println("static fun");
    }
    public void fun(final int var1) {		// 由于fun的参数被方法内部类使用，所以设置为final
        class Inner {
            int c = 3;
            Inner() {
            }
            void test() {
                System.out.println(var1);
                System.out.println("hello");	// 直接替换
            }
        }
        Inner var3 = new Inner();
        var3.test();
    }
}

// 方法内部类——单独生成一个文件
class Outer$1Inner {
    int c;
    // 不再是一个默认构造函数，而是传了一个Outer对象、方法内部的参数和局部变量=>用到才传，不用到默认不传
    Outer$1Inner(Outer var1, int var2, String var3) {
        this.this$0 = var1;
        this.val$param = var2;	// 是val$参数名
        this.val$str = var3;
        this.c = 3;
    }

    void test() {
        Outer.access$000();		// Outer应该生成了一个public方法给Inner去调用私有的变量or方法
        System.out.println(this.val$param);
        System.out.println("hello");
    }
}
```

可以发现：

- 和成员内部类一样，都是在Inner类中去实例化一个Outer对象，然后通过使用该对象来访问Outer的实例方法、变量；

- 而方法内部的局部变量和参数，都是在构造函数中传参的（用到才传，不用到就不用传），并且会将该变量设置为final；

- 如果显式定义了final的变量（不论是方法内部，还是成员变量or静态变量）会被当成常量，在编译的时候就会直接给实际值（而不是变量名）

- why方法中的变量和参数使用时，会被定义为final

  可以发现，在Inner构造方法中，会对其进行传参，所以实际上**Inner使用的是自己的局部变量**，而不是外部参数——只是值传递（浅拷贝），将其定义为final表示不可修改（实际上修改也没有用）——类似于C的函数参数传递

  =>而由于是值传递，所以在**方法内部类对这些变量or参数修改不影响其原本的值**——如果要修改就要传递引用类型变量，不修改地址，就修改里面的值

——实际上，方法内部类可以用成员内部类实现，然后方法中实例化该成员内部类对象，如果需要使用方法参数/局部变量等可以通过值传递

——但是方法内部类限定了本方法使用，有更好的封装性能。

#### （4）匿名内部类

比前面的类定义更为简单，匿名内部类：没有单独的类定义，在**创建对象的同时定义类**。

格式如下：

```java
new 父类(参数列表){	
    // 实现部分
}

new 父接口(参数列表){
    // 实现部分
}
```

分析：

1. 匿名内部类定义（创建），都是**new**开头，表示创建对象
2. new后面紧跟父类/接口，并且对其进行传参到构造方法（匿名内部类本身不含构造方法，都是调用父类的构造方法）
3. 匿名内部类只能**被使用一次**，用来创建一个对象，没有名字、没有构造方法（但是可以调用父类的构造方法）。可以定义实例变量、方法，可以有初始化代码块（起到构造方法的作用，但是只能有一个，而构造函数可以有多个）
4. 匿名内部类可以：访问外部类的所有变量、方法，也可以访问方法的局部变量、参数

=> 所以匿名内部类实际上是继承/实现某个类/接口，在这个里面重写了方法（如果可以直接使用该父类，那么不需要构建类了，直接声明即可）

举例：

```java
// 抽象类
abstract class Son{
    int x, y;
    public Son(int x, int y){		// 构造方法
        this.x = x;
        this.y = y;
    }
    public abstract void eat();
}

public class Outer {
    public void test(int x2, int y2) {
        System.out.println("test");
        Son son = new Son(x2, y2) {			// 调用抽象类，实现了抽象方法
            @Override
            public void eat() {		// 重写方法
                System.out.println(this.x + ", "+ this.y);
                System.out.println("no name eating");
            }
        };
        son.eat();		// 创建对象后调用对象的实例方法
    }
}
```

编译之后：（外部类被忽略了）

```java
class Outer$1 extends Son {		
    Outer$1(Outer var1, int var2, int var3) {		// 用到的方法中的局部变量、参数
        super(var2, var3);
        this.this$0 = var1;
    }
    public void eat() {
        System.out.println(this.x + ", " + this.y);
        System.out.println("no name eating");
    }
}
```

分析：

1. 匿名内部类也会生成一个独立的class类文件，由于没有名字，所以就是`外部类名$数字编号`，eg：`Outer$1`
2. 会有一个构造方法：里面包含外部类对象（用来调用外部类的方法、变量），传递用到的参数（所有方法的局部变量or参数）
3. 匿名内部类能做的，方法内部类都能做，只不过如果对象只创建一次，并且不需要构造函数来传参（这边传参传给的是父类），可以使用匿名内部类

用处：调用方法时，需要一个接口参数，此时可以使用一个匿名内部类，来实现一个自定义的方法

eg：方法中传递接口参数——实现接口的方法

```java
String[] strs = new String[]{"i", "Hello", "I"};
// sort方法的传参：可以接受一个数组、一个comparator接口参数
// public static <T> void sort(T[] a, Comparator<? super T> c) 
Arrays.sort(strs, new Comparator<String>(){	// comparator接口，
	@Override
	public int compare(String o1, String o2) { // 有一个方法compare用来比较两个对象，自定义实现
		return o1.compareToIgnoreCase(o2);
    }
});
```

——compare是回调函数

eg：还能用于事件处理程序中，用来响应某个事件：

```java
Button bt = new Button();
// 该button的实例方法：addActionListener需要的传参是一个接口ActionListener，而我们需要去实现里面的方法
bt.addActionListener(new ActionListener(){
	@Override
	public void actionPerformed(ActionEvent e) {
    	System.out.println("implements");
    }
});
```

——actionPerformed是回调函数

=>**将程序分为保持不变的框架，和针对具体情况可变的逻辑，通过回调函数（通过实现接口功能）来协作，是计算机程序的常见操作**——匿名内部类就是实现回调接口的一种简便方式

### 4. 枚举

概念类似于C的枚举数据类型，还是有些不同。

枚举：顾名思义，它的取值有限，可以全部列出来（类可以实现枚举效果，但是单独一个枚举类型更为直观、安全、简洁）

#### （1）基础

定义：

关键字：**enum**

```java
public enum Size {				// 将类的class修改为enum，代表枚举类型
    SMALL, MEDIUM, LARGE// 枚举的值一般都是大写字母（代表常量），值之间通过','分隔，如果只有这些可以无分号
}
```

——枚举类型可以单独建一个文件，也可以实现在其他类内部（注意是类内部，而不能在class外面定义）

使用：

```java
Size size = Size.LARGE;		// 声明一个Size类型的变量size，右边类似于类的类变量使用方法

size.toString();	//>> LARGE toString()返回的是字面值，String类型
size.name();		//>> LARGE name()返回的也是字面值，String类型
size.toString() == size.name();		//>> true

size == Size.LARGE;			// 枚举变量可以通过==/equals比较
size.equals(Size.LARGE);

size.ordinal();		//>> 2 枚举值，返回的是int类型（表明声明时的顺序），从0开始计算
size.compareTo(Size.MEDIUM);		//>> 1 可以和其他值进行比较，返回值是-1/0/1

Size.valueOf("LARGE");		//>> LARGE 根据输入的字符串，得出对应的枚举值，如果不存在会报错

for(Size size1: Size.values()){		// values()返回的是枚举类型的数组——包含所有的枚举值
	System.out.println(size1);		//>>SMALL MEDIUM LARGE
}
```

所以枚举可以使用的函数有：

| 函数名         | 返回值       | 作用                                                         |
| -------------- | ------------ | ------------------------------------------------------------ |
| toString()     | String       | 枚举值的字面值                                               |
| name()         | String       | 枚举值的字面值                                               |
| equal(xxx)     | boolean      | 枚举值之间比较                                               |
| ordinal()      | int          | 枚举值声明顺序值（从0开始计算）                              |
| compareTo(xxx) | int(-1/0/1)  | 和其他枚举值比较大小，比较ordinal()值大小                    |
| valueOf(xxx)   | 枚举类型     | 根据输入的String类型，返回对应的枚举类型<br />（不存在会报错）=> 静态方法 |
| values()       | 枚举类型数组 | 返回的是枚举类型的所有枚举值                                 |

枚举变量可以和其他变量一样用在任何位置：方法参数、类变量、实例变量等。还能用于`switch`语句中

eg：

```java
switch(size){
    case SMALL: break;	// 注意这边不能带类型前缀，直接是枚举值本身
    case MEDIUM: break;
    case LARGE: break;
}
```

Java5之前是不支持枚举的，那么就定义一个类，然后设置静态整型变量来代表枚举类型，类似于`public static final int SMALL = 0;`

=> 但是枚举类型：

- 更为简洁：只需要写有哪些枚举值，而不需要定义`public static final`这样
- 更为安全：一个枚举变量**只会有null和枚举值**，而不可能有其他值，但是如果用整型变量，值就无法控制
- 且自带很多的方法

枚举类型实际上会被转换为一个类，继承了`java.lang.Enum`类：

```java
private final String name;		// name实例变量——string
public final String name() {	// name/toString两个方法一样
	return name;
}
public String toString() {
	return name;
}
private final int ordinal;		// ordinal实例变量——int
public final int ordinal() {
	return ordinal;
}
// 构造方法是protected类型：同包和子类可见
protected Enum(String name, int ordinal) {	// 构造方法：需要传递name和ordinal两个实例变量
	this.name = name;
	this.ordinal = ordinal;
}

public final boolean equals(Object other) {		// 比较两个枚举对象是否一样
	return this==other;
}
public final int compareTo(E o) {		// 比较枚举值大小
	Enum<?> other = (Enum<?>)o;
	Enum<E> self = this;
	if (self.getClass() != other.getClass() && // optimization
		self.getDeclaringClass() != other.getDeclaringClass())
		throw new ClassCastException();
	return self.ordinal - other.ordinal;
}
public static <T extends Enum<T>> T valueOf(Class<T> enumType,
                                                String name) {
	T result = enumType.enumConstantDirectory().get(name);
	if (result != null)
        return result;
	if (name == null)
		throw new NullPointerException("Name is null");
    throw new IllegalArgumentException(
            "No enum constant " + enumType.getCanonicalName() + "." + name);
}
```

以上的枚举类型转换成普通类的结果（表示一个大致意思）

```java
public final class Size extends Enum<Size> {		// 继承类Enum
	public static final Size SMALL = new Size("SMALL",0);	// 定义静态对象
	public static final Size MEDIUM = new Size("MEDIUM",1);
	public static final Size LARGE = new Size("LARGE",2);
	private static Size[] VALUES = new Size[]{SMALL,MEDIUM,LARGE};	// 保存所有对象
    private Size(String name, int ordinal){		// 私有构造方法——给上面的使用（本质上调用父类的构造函数）
    	super(name, ordinal);
    }
    public static Size[] values(){		// values函数，——返回一个新数组，存放size的所有数
        Size[] values = new Size[VALUES.length];
        System.arraycopy(VALUES, 0, values, 0, VALUES.length);
        return values;
    }
    public static Size valueOf(String name){
    	return Enum.valueOf(Size.class, name);	// 去调用父类的方法，还要传类的类型信息
    }
}
```

分析：

1. Size是final类，表示该类不能被继承。Enum<Size>是父类，<Size>表示泛型
2. 存在一个私有的构造方法，不公开，只是内部的静态变量使用，用来创建枚举类对象。且该**静态变量是final类型**，不能被修改
3. `values()`方法，是编译器添加的；且有一个数组`VALUES[]`，维护所有的枚举值——values()就是拷贝该数组内容
4. `valueOf()`方法，本质上是去调用父类的方法，注意还需要传递`Size.class`，表示类的类型信息
5. 一般在使用中，枚举变量会被转换成对应的**类变量值**，而在switch中，枚举值会被转换成`ordinal`值（int类型）

=>所以，枚举本质上还是类，只不过编译器附带做了很多工作（所以说，Java的基本数据单位就是类）

#### （2）应用场景

枚举类型中，还会带有关联的实例变量、方法等

```java
public enum Size {		// 枚举
    SMALL("s", "xiaohao"), MEDIUM("M","zhonghao"), LARGE("L", "dahao");
    private String abbr;		// 定义了两个实例变量
    private String chinese;
    private Size(String abbr, String chinese){		// 定义私有的构造方法——给上面的对象用
        this.abbr = abbr;
        this.chinese = chinese;
    }
    
    public String getAbbr() { return abbr;}
	public String getChinese() { return chinese;}
    
    public static Size fromAbbr(String abbr){
        for(Size size: Size.values()){
            if(size.getAbbr().equals(abbr))
                return size;
        }
        return null;
    }
}

```

分析：

1. 注意：枚举值需要放在最上面定义，且之间用`,`间隔，最后用`;`和其他分隔

2. 发现，枚举的构造方法内**不能初始化静态变量**，**只能初始化实例变量**（普通类可以初始化静态变量）

   （因为，枚举值是在类加载初始化阶段最早被调用的——因为静态类型变量是按照文本顺序初始化的，而枚举类型位于类的最前面，那么会去调用构造方法。此时在枚举值下面的静态变量还未初始化，所以不能在构造方法中使用）

3. 可以定义实例变量、方法，类变量（不能在构造方法中）、方法

转换成普通类定义如下：

```java
public final class Size extends Enum<Size> {
    public static final Size SMALL = new Size("SMALL",0, "S", "小号");
    public static final Size MEDIUM = new Size("MEDIUM",1,"M","中号");
    public static final Size LARGE = new Size("LARGE",2,"L","大号");
    private String abbr;
    private String title;
    // 构造方法变化
    private Size(String name, int ordinal, String abbr, String title){
        super(name, ordinal);	// 调用父类构造方法
        this.abbr = abbr;		// 子类特有的方法
        this.title = title;
    }
// other
}
```

ps：枚举值通常会关联一个标识符`id`，唯一表示一个枚举值，一般用int表示。不推荐用默认的ordinal的值——因为其编号是按照定义的顺序来的，那么如果要在枚举中增加一个枚举值，所有的值都有可能发生变化。所以要保证**id和枚举值保持不变**，那么就是增加一个实例变量`id`，然后构造函数定义，创建的时候传参即可	

## （四）异常

在没有异常的情况下，唯一退出的机制就是return，判断是否出现异常就通过返回值（类似C语言中的，对不同情况进行判断，根据不同情况设定返回值，eg：`memory_alloc()`）。然后调用者根据不同的返回值做出不同的判断。那么每层都需要对调用的程序的不同返回值进行判断和处理。**程序正常逻辑和异常逻辑混在一起，代码难以阅读和维护**。并且很多时候都会忽略某些异常的处理，降低了程序的可靠性。

异常机制下，**正常逻辑和异常逻辑可以分离，异常情况可以集中处理，并且异常可以自动上传，不需要每一层都处理，并且异常不会被忽略**。处理异常的代码量减少（不需要对不同返回值进行判断处理，并且异常会封装一些方法，直接调用复用即可），且代码的可读性、可靠性、可维护性提高

### 1. 异常的概念

常见的两个异常：`NullPointerException`（空指针异常）和`NumberFormatException`（数字格式异常）

举例：

```java
String s = null;
System.out.println(s.indexOf("a"));		// 在字符串s中找到'a'所对应的下标
=>会抛出异常：NullPointerException（s未初始化就调用实例方法了）
```

<img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20201212221339003.png" alt="image-20201212221339003" style="zoom:67%;" />

因为JVM在执行到下一句的时候，发现s是一个null变量，那就无法继续操作了——启用异常处理机制：（如果s初始化了，最多找不到就返回-1）

- 首先创建一个异常对象，这边是类NullPointerException的对象

- 然后查找谁能处理该异常

- 发现没有程序能处理，那么启用**默认处理机制：打印出异常栈信息到屏幕**

  异常栈信息，就包括从异常发生点到最上层调用者的轨迹，还包括行号（类似于递归的深度）——这个是分析异常的重要信息

- 退出程序，不再执行下去

举例2：

```java
String s = "abc";
int num = Integer.parseInt(s);		// 将字符串转换为数字——前提是，字符串的内容是数字
System.out.println(num);
=> 会抛出异常：NumberFormatException（数字格式异常）
```

<img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20201212222841111.png" alt="image-20201212222841111" style="zoom:67%;" />

因为parseInt期待的是数字，而s内容是字母，所以无法执行下去——启动异常处理机制

可以发现报错的位置是从最底层到最上层的：

```java
// Main类的main方法中——最上层
int num = Integer.parseInt(s);

// Integer类的parseInt方法，是上层调用的
public static int parseInt(String s) throws NumberFormatException {
	return parseInt(s,10);
}

// Integer类的parseInt方法，是上层调用的
public static int parseInt(String s, int radix) throws NumberFormatException{
    ...
	// Accumulating negatively avoids surprises near MAX_VALUE
	digit = Character.digit(s.charAt(i++),radix);
	if (digit < 0) {
        // 这句话本质上就是创建了一个NumberFormatException对象，然后组合成需要输出的字符串
		throw NumberFormatException.forInputString(s);// 抛出异常，会触发Java的异常处理机制
	}
    ...
}

// NumberFormatException类中的forInputString方法，是上层调用的
static NumberFormatException forInputString(String s) {
    // 创建一个异常类的对象——传递一个字符串，就是最上面打印出的具体异常内容
	return new NumberFormatException("For input string: \"" + s + "\"");
}
```

分析：

1. 这个可以看到明显的异常栈：从上到下从由深变浅，最下面就是最上层的报错位置

2. `throw`和`return`同等级进行类比：

   - return代表正常退出，throw代表异常退出；

   - return的返回位置是确定的，就是上级调用者，而throw后执行的代码是不确定的，由异常处理机制动态决定

     ——异常处理机制会从当前函数开始**查找谁”捕获“了异常**，找不到就找上一层，直到主函数，如果主函数也没有，那么就**采用默认的：输出异常栈信息并退出**

为了保证程序的友好性，需要程序自行”捕获“异常，”捕获“之后做特殊处理，并且程序自行捕获，那么不会异常退出程序（默认是打印异常栈并且退出程序），后面的程序仍会执行下去

关键字`try/catch`

eg:

```java
String s = "abc";
try{									// 尝试执行
    int number = Integer.parseInt(s);
}catch(NumberFormatException e){			// 捕获指定的异常
    System.out.println("not a number: " + s);
}
```

分析：

1. try后面的代码块：包括可能抛出异常的执行；catch后面的括号部分：是异常信息：**异常类型+变量名**（是需要指定具体异常的类型的）；catch后面的代码块包括异常的处理代码：一般是更为友好的提示信息

   如果出现异常，则try异常点之后的代码是不会被执行的

2. 捕获异常之后，程序不会异常退出。而是继续执行catch代码块之后的程序

3. 如果抛出的异常，没有被捕获（没有匹配任何异常），那么也会用默认的异常处理程序并退出

### 2. 异常类

#### （1）父类：Throwable

异常本质上也是一个类对象，**所有异常类都有一个共同的父类：Throwable**

有5个构造方法：

```java
public class Throwable implements Serializable {
    private String detailMessage;			// 私有实例变量——表示异常信息
    private Throwable cause = this;			// Throwable类对象——表示触发该异常的其他异常

    public Throwable() {
        fillInStackTrace();
    }

    public Throwable(String message) {
        fillInStackTrace();
        detailMessage = message;
    }

    public Throwable(String message, Throwable cause) {
        fillInStackTrace();
        detailMessage = message;
        this.cause = cause;
    }

    public Throwable(Throwable cause) {
        fillInStackTrace();
        detailMessage = (cause==null ? null : cause.toString());
        this.cause = cause;
    }

    // 针对子类和同包内类可见
    protected Throwable(String message, Throwable cause, boolean enableSuppression,
                        boolean writableStackTrace) {
        if (writableStackTrace) {
            fillInStackTrace();
        } else {
            stackTrace = null;
        }
        detailMessage = message;
        this.cause = cause;
        if (!enableSuppression)
            suppressedExceptions = null;
    }
}
```

分析：

1. Throwable类有两个主要参数：

   - `message`：表示异常信息，String类
   - `cause`：表示触发该异常的其他异常，是Throwable类型

2. 异常一般能够形成一个**异常链**，上层的异常一般由底层的异常触发，而cause一般表示底层的异常

   还提供了一个方法来设置cause

   ```java
   public synchronized Throwable initCause(Throwable cause) {
   	if (this.cause != this)		// 已经被初始化过了，不能重复设置
   		throw new IllegalStateException("Can't overwrite cause with " +
                                               Objects.toString(cause, "a null"), this);
   	if (cause == this)		// 传递的参数是本身——自己导致自己是不允许的？？？？
   		throw new IllegalArgumentException("Self-causation not permitted", this);
       this.cause = cause;
   	return this;
   }
   ```

   该方法**最多被调用一次**，用在某些子类没有带cause的构造方法，那么就可以通过`initCause`方法设置cause（如果构造函数已经调用过初始化cause过了，该函数就不能在此被调用）

3. 发现构造函数都存在一个函数调用`fillInStackTrace();`

   ```java
   public synchronized Throwable fillInStackTrace() {
   	if (stackTrace != null || backtrace != null /* Out of protocol state */ ) {
   		fillInStackTrace(0);
   		stackTrace = UNASSIGNED_STACK;
   	}
   	return this;
   }
   ```

   它会将异常栈的信息保存下来

   还有很多方法能够得到异常信息：

   ```java
   void printStackTrace() // 打印异常栈的信息到标准错误输出流
   // 打印栈信息到指定的流
   void printStackTrace(PrintStream s)
   void printStackTrace(PrintWriter s)
   String getMessage() // 获取异常的message
   Throwable getCause() // 获取异常的cause
   // 获取异常栈的每一层信息，每个stackTraceElement包括文件名、类名、方法名、行号等信息
   StackTraceElement[] getStackTrace()
   ```

#### （2）异常类体系

<img src="pic\image-20201213120034202.png" alt="image-20201213120034202" style="zoom:67%;" />

**Throwable为根**，下面分为两大类异常：

1. `Error`：**系统自行调用的，表示系统错误or资源耗尽**——未受检异常

   应用程序不应抛出和处理

   下面有：虚拟机错误、内存溢出异常、栈溢出异常等（都是内部的，应用程序不可控制）.....

2. `Exception`：**应用程序错误**。这下面的异常很多，可以**使用，也可以继承`Exception`或者其他异常后自定义异常**

   Exception常见的子类有：输入/输出异常——受检异常、数据库SQL异常——受检异常、**运行时异常——未受检异常**（Exception和它的其他子类都是受检异常）

   （受检异常：编译器会进行检查，看代码会不会存在异常，如果会存在异常会**强行要求你做出处理**，否则编译不通过；——就是有些代码一定要加上try/catch代码，这些就是受检异常）

   （未受检异常：可能存在错误，但是编译的时候不检查，没必要也没办法——上面举例的就是未受检异常）

3. 大部分的异常类和父类并没有很多差别，大部分都是增加了构造方法，且构造方法也是调用父类的构造方法的

   why 还需要定义这么多不同的异常类？

   $\because$ **主要是为了名字不同**，名字本身也是异常的关键信息，通过名字能获得很多信息，捕获异常/抛出异常要选择合适名字的异常类，能够方便代码修改和异常定位

   ——所以异常类的名字要直观方便阅读（发现，异常类的名字都很长，都是一个短语用来形象的）

#### （3）自定义异常

一般是继承`Exception`或者其子类，而自定义的异常，如果继承的父类是`RuntimeException`异常，那么自定义的异常也是非受检异常；如果是Exception和它的其他子类，那么是受检异常

（不能继承error的异常，因为是java内部使用的）

```java
public class AppException extends Exception {		// 类内部，并没有实现什么（纯粹是继承）
    public AppException() {
    	super();
    }
    public AppException(String message, Throwable cause) {
    	super(message, cause);
    }
    public AppException(String message) {
    	super(message);
    }
    public AppException(Throwable cause) {
    	super(cause);
    }
}
```

### 3. 异常的处理

#### （1）catch匹配

之前写的`try/catch`，是一个try对应一个catch，实际上**try后面可以跟多个catch**——**每个catch对应一种异常类型**

```java
try{
    // 需要执行的代码
}catch(NumberFormatException e){
	System.out.println("not valid number");
}catch(RuntimeException e){
	System.out.println("runtime exception "+e.getMessage());	// 获取异常信息
}catch(Exception e){
	e.printStackTrace();		// 打印出异常栈到标准错误输出流
}

=> 简化版本：Java7之后支持，通过 | 将不同的异常分隔	
try {
	int num = s.indexOf(s);
}catch (NumberFormatException | NullPointerException e){
	System.out.println("not initialize");
}    
```

异常处理机制会根据抛出的异常类型找到**第一个匹配的catch块**，一旦找到了适合的，后面的catch将不会被执行。而如果都没有找到，那么会去上层的方法中继续查找——一直查找到最上层。

匹配中，如果是该类型的子类也算是匹配的，所以要将最具体的子类放在最上面，eg：如果将基类`Exception`放在最上面，那么如果出现异常就直接匹配，而后面的都得不到执行了。

——一般会将异常输出到专门的日志中。

#### （2）重新抛出异常

在**catch中**，自己的异常处理完成后，还可以**重新抛出异常**，异常可以是原来的，也可以是新建的

```java
try {
	int num = s.indexOf(s);
}catch (NullPointerException e){
	System.out.println("not a string: "+s);
	throw new MyException("输入内容不正确", e);	// 重新抛出了MyException类异常，而当前异常被传递给了MyException的cause，通过getCause()能够得到现在的异常
}catch (Exception e){
	System.out.println("not initialize");
	throw e;			// 重新抛出异常 Exception类型
}
```

分析：

1. 重新抛出异常，从而形成了一个异常链，我的理解就是**后一个异常保存当前异常的信息来实现链表的效果**

   ——注意，要构成异常链：

   - 首先要触发多个异常，那么是在catch里面加入抛出异常操作throw；
   - 并且链需要连接起来：捕获一个异常抛出另一个异常，且需要保存原始异常信息，将该信息传递到上层的异常catch中，用构造函数传递message（解释）、cause（错误类型）

2. 第一个异常，引起第二个异常，可以让用户知道异常之间的关系，更方便调试

   eg：`throw new MyException("输入内容不正确", e);`就是在处理完成之后又抛出一个新的异常，给后面的异常。而将本次异常的信息记录到新的异常类中：通过message和cause——能够保存本次的异常信息和异常原因

3. why要重新抛出？

   $\because$ 当前的代码不能完全处理异常，需要进一步处理

4. why 要抛出一个新的异常

   $\because$ 当前的异常不是很合适，可能是信息不够，需要一些新的信息；可能是过于细节，不便调用者理解和使用

#### （3）finally

catch后面可以跟随`finally`关键词（注意区分：final）

```java
try{
    // 可能触发异常的操作
}catch(xxx){		// 预测可能发生的异常
    // 异常处理
}finally{
    // 不管是否出现异常都执行
}
```

分析：

1. 就是在之前的`try/catch`后面增加了一段`finally`，它不管是否出现异常都会去执行，具体如下：

   - 如果没有异常，那么执行完try之后就去执行finally
   - 如果有异常，且捕获了，那么catch执行完成后执行finally
   - 如果有异常，但是没有捕获（匹配失败），那么**异常被抛给上层之前去执行**

   所以finally一般是用来释放资源，eg：数据库连接、文件流关闭等

2. catch不是必须出现的，eg：**`try/finally`也是合法的**，那么异常在该层不被捕获，而是向上传递，但是**finally执行顺序还是：异常发生之后，被抛给上层之前去执行**。

3. 还要注意一些迷惑操作：

   - 如果try/catch中存在return语句，那么**return是在finally执行完成之后**，才执行去返回上层方法的。但是**finally不能修改return的内容**

     eg：
     ```java
     int ret = 1;
     try{
     	return ret;
     }finally{
     	ret = 2;		// ret值被修改，但是并没有效果
     }
     >> 1
     ```

     实际上，在进入finally之前会将ret保存在一个临时变量中，执行完成finally之后，反过来执行return，找的是那个临时变量

   - finally中存在return语句：那么**前面的return会被丢失**——以finally为准；存在return**也会掩盖前面的异常**——以finally中的异常为准（如果没有，就当作没有发生异常）

     eg：

     ```java
     public static int test(){
         int ret = 0;
         try{
             int a = 5 / 0;	// 这边会触发异常:ArithmeticException
             return ret;
         }finally{
             return 1;
         }
     }
     >> 无异常，返回值为1
         
     public static int test2(){
         try{
             int a = 5 / 0;
         }finally{
             throw IOException();
         }
     }
     >> 抛出IO异常（忽略了算术异常）
     ```

   ——所以需要避免在finally有return、throw语句


#### （4）try-with-resources

前面说到，finally多用在关闭资源操作中，Java7支持自动关闭资源，被称为`try-with-resources`

语法如下：

```java
public static void test() throws Exception{
    try(AutoCloseable r = new FileInputStream("hello world"))	// 创建资源
    {
        	// 使用资源
    }
}
```

针对的实现了`java.lang.AutoCloseable`接口的对象——AutoCloseable对象是核心

当执行完try之后会默认调用`close()`方法关闭资源

资源可以定义多个，**以分号间隔**（Java9之前资源声明和初始化必须要在try中，但是Java9之后允许可以在try之前，但是资源必须是final或形式上的final，即只被赋值一次）

#### （5）throws关键字：声明异常

注意和throw区分：throw是抛出异常，是一个语句；throws是方法中的声明

`throws`用来**声明一个方法**可能抛出的异常

```java
public static void main(String[] args) throws NullPointerException, NumberFormatException
{
    
}
```

分析：

1. 一个方法可以声明多个异常，**以逗号分隔**。

   该声明的含义是：方法内可能抛出这些异常，且**对这些异常没有处理/没有处理完**，调用者必须进行处理——所以，调用者必须要catch或者继续throws，即：调用者内部要去实现该异常的处理

   ```java
   void b() throws xxx{
       // 实现代码
   }
   
   void a(){
   	try{
   		b();
   	}catch(xxx){		// b可能会抛出异常xxx，所以a要在调用b后try/catch
   		// 操作
   	}
   }
   
   // 或者
   void a() throws xxx{	// a继续上抛异常处理
       b();
       // 其他代码
   }
   ```

2. 对于未受检异常，不要求用throws进行声明的；对于受检异常，一定要用throws进行声明，eg：IOException——没有声明，就不能抛出

   受检异常，不可以不声明就抛出，可以声明而不抛出。这个情况出现在父类子类中：父类不需要抛出某个异常，但是子类需要抛出，而**子类不能抛出父类没有声明的异常**

未受检异常和受检异常：

未受检异常：通常表示的是编程的逻辑错误，编程的时候应该尽量避免这些错误——所以应该避免而不是处理

受检异常：通常是程序本身没有问题，由于IO、网络、数据库等出现不可预测的问题而导致的异常——无法避免，所以要选择处理

——其实，实际上没有很明显的区分，不论是未受检异常or受检异常，是否有throws声明，所有异常都应该得到处理

### 4. 异常的使用

异常应该仅用于异常的情况即：

- **异常不能替代正常的条件判断**

  eg：在处理数组是，应该先判断索引值是否有效来进行循环，而不是靠抛出异常来结束循环

  ​		对于引用对象，在使用前应该先判断是否为null，而不是靠异常来判断

- 一旦出现真正的异常，应该抛出异常，而不是返回特殊值

  eg：

  ```java
  public String substring(int beginIndex) {
      if(beginIndex < 0) {	// 可能会考虑返回一个null，表示参数有误，但是这还是一个异常情况，抛出异常为好
      	throw new StringIndexOutOfBoundsException(beginIndex);
      }
      int subLen = value.length - beginIndex;
      if(subLen < 0) {
      	throw new StringIndexOutOfBoundsException(subLen);
      }
      return(beginIndex == 0) ? this : new String(value, beginIndex, subLen);
  }
  ```

异常处理的来源有：不同的来源应该不同处理方式

- 用户，用户的输入有问题
- 程序员，程序逻辑有问题
- 第三方，IO错误，网络、数据库、第三方存在问题

处理的目标是：恢复和报告

对于用户，如果是输入问题：应该提示用户输入哪里不对，要如何输入等；如果是程序逻辑有问题：应该要提示联系客服、系统错误等；如果第三方问题：应该提示稍后再试

对于程序员，关注程序错误和第三方错误，那么应该要报告尽量完整的细节，包括异常栈和异常链，以便快速定位错误

异常处理一般：如果自己能处理，就在这一层处理了；如果不能完全解决，应该上抛；——一定要有一层处理异常

## （五）常用基础类

主要介绍常用的几个基础类：包装类、String类、StringBuilder类、Arrays类。它们的常用用法（要记）、实现逻辑

### 1. 包装类

之前也用到过，Integer、Character等。

Java的8种基本数据类型，都有分别对应一个包装类

（只需要特别记忆：int-Integer，char-Character，其余都是首字母大写即可）

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
	this.value = parseInt(s, 10);
}
```

why要定义包装类？

$\because$ Java很多方法只能操作对象，为了能够操作基本数据类型，于是就将基本数据类型包装成包装类，然后能够使用这些方法。并且包装类还有很多方法可以来操作数据。

#### （1）基本用法

<a name="valueof_link"></a>

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

将基本数据类型转换为包装类的过程——装箱

将包装类转换为基本数据类型的过程——拆箱

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

#### （2）包装类共同点

它们都重写了Object方法，都实现了Comparable接口

```java
public final class Integer extends Number implements Comparable<Integer> {
```

1. 重写Object方法

   - 重写`equals(Object obj)`方法

     用来判断：当前对象和传入的对象是否相同

     Object默认是比较地址，只有指向的地址一样才返回true，等同于`==`

     而equals应该反映的是对象之间的逻辑相等关系（即值相等），对于包装类（本质上是代表基本数据类型的，基本数据类型应该是值比较）这样的默认实现是不合理的

     ——**所有包装类全部重写，比较的是包装类对应的基本类型值**

     具体见：[Java库源码阅读](http://note.youdao.com/noteshare?id=fc7f62615236fbe3814aefabdd5c2241&sub=E1449B87AC3240ADA389BAF1006C6459)

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

     [详见Java的额外知识5](http://note.youdao.com/noteshare?id=9f8ab6eaf048c06b8c823ebe1c7f0b8c&sub=38FCCF96A0C74660A8B19E1140BA449E)

     **因为包装类重写了equals方法，将其逻辑修改了（虽然不是同一对象，但是值一样就认为是一样的），那么我们也需要重写hashCode方法**：

     包装类们是如何实现的：详见：[Java库源码阅读](http://note.youdao.com/noteshare?id=fc7f62615236fbe3814aefabdd5c2241&sub=E1449B87AC3240ADA389BAF1006C6459)

   - 重写`toString()`方法——注意这个是实例方法（下面的是静态方法），是用来对该对象的直观文本表示，简要的表达出该对象的信息（一般就是输出对象的某些值等，也会带一定的格式）

     Object类下的方法实现：

     ```java
     // 获得该实例对象的类名 @ 该对象的hashCode的十六进制表示
     public String toString() {
         return getClass().getName() + "@" + Integer.toHexString(hashCode());
     }
     ```

     其余的见：[Java库源码阅读](http://note.youdao.com/noteshare?id=fc7f62615236fbe3814aefabdd5c2241&sub=E1449B87AC3240ADA389BAF1006C6459)

   - String和包装类的互相转换

     - 根据字符串表示返回基本类型值：parse

       具体的见：[Java库源码阅读](http://note.youdao.com/noteshare?id=fc7f62615236fbe3814aefabdd5c2241&sub=E1449B87AC3240ADA389BAF1006C6459)

     - 根据基本类型值返回字符串表示：toString——注意这个和上面的实例方法`toString`做区分（实例方法的，是为了表达出该对象的一些信息，而本方法只是为了将值转换成字符串输出）

       具体见：[Java库源码阅读](http://note.youdao.com/noteshare?id=fc7f62615236fbe3814aefabdd5c2241&sub=E1449B87AC3240ADA389BAF1006C6459)

     并且对于整型，还提供了其他进制的转换：

     字符串按照其他进制转换成整型；整型转换成指定进制表示的字符串（默认进制是10进制）

   - 常用常量

     具体见：[Java库源码阅读](http://note.youdao.com/noteshare?id=fc7f62615236fbe3814aefabdd5c2241&sub=E1449B87AC3240ADA389BAF1006C6459)

     => 总结：列出了该包装类的二进制下的最大位数`SIZE`和对应的字节数`BYTES`

     ​	还列出了该包装类的最大最小值`MIN_VALUE`、`MAX_VALUE`，当数字比较小的时候用十进制表示；当数字比较大的时候，用十六进制表示。Float/Double特殊还列出了正无穷大、负无穷大，NaN数（都是通过计算得到的，而不能直接表达出数字），还列出了e的最大值（因为浮点数是用科学计数法表示的）

   - Number

     具体见：[Java库源码阅读](http://note.youdao.com/noteshare?id=fc7f62615236fbe3814aefabdd5c2241&sub=E1449B87AC3240ADA389BAF1006C6459)

     包装类都继承自该父类（除了Boolean），而Number定义了6个抽象方法——将当前的对象值输出为6种基本数据类型格式，要求每个包装类均实现，那么**包装类实例之间可以返回任意的基本数据类型**

2. 实现Comparable接口

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

   具体见：[Java库源码阅读](http://note.youdao.com/noteshare?id=fc7f62615236fbe3814aefabdd5c2241&sub=E1449B87AC3240ADA389BAF1006C6459)

   浮点数同样存在计算误差问题，导致0.01和0.1*0.1的结果并不相等（和equals一样）

3. **不可变性**

   <a name="immutability"></a>

   实例对象一旦创建，里面的值不可变了。

   具体实现在：

   - 所有包装类都是final，表示该类不可被继承
   - 内部的基本数据类型是private的，且声明为final——即只能赋值一次，且不能通过直接访问的方式修改，eg：a.value（不可行）
   - 没有定义setter方法，不能通过调用方法的方式修改值

   why 设置为不可变类呢？

   程序更为简单且安全，不用担心数据会被意外修改，可以安全的共享数据，尤其是在多线程的情况下。——一旦创建只能读不能写

#### （3）Integer

主要是关注二进制操作

1. 位翻转

   Integer提供了两个移位方法：第一是按位翻转；第二个是按字节翻转

   ```java
   public static int reverse(int i);
   public static int reverseBytes(int i);
   ```

   具体见：[Java库源码阅读](http://note.youdao.com/noteshare?id=fc7f62615236fbe3814aefabdd5c2241&sub=E1449B87AC3240ADA389BAF1006C6459)

2. 循环移位

   提供两个方法：循环左移和循环右移

   ```java
   public static int rotateLeft(int i, int distance);
   public static int rotateRight(int i, int distance);
   ```

   具体见：[Java库源码阅读](http://note.youdao.com/noteshare?id=fc7f62615236fbe3814aefabdd5c2241&sub=E1449B87AC3240ADA389BAF1006C6459)

3. valueOf的实现

   在[(1)基本用法](#valueof_link)中有提到，就是Integer会提前缓存-128~127范围内的Integer对象，如果通过valueOf创建对象值是在这个范围内的，那么就直接返回该对象，而不会创建新对象；如果超出范围，才会通过new进行创建

   ——**所以创建包装类对象时，推荐使用`valueOf`方法**

   => Boolean、Byte、Short、Long、Character都有类似的实现

   通过共享常用的对象，可以节省内存空间，且由于设定的包装类是[不可变的](#immutability)，该对象可以被安全的使用

   => 这个也是常见的设计思路：**享元模式**（Flyweight）

#### （4）Character

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

   <img src="pic\image-20201218155946343.png" alt="image-20201218155946343" style="zoom:67%;" />

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

### 2. 文本处理类String和StringBuilder

最常见的操作之一。Java中用来处理字符串的主要类就是`String`和`StringBuilder`。

#### （1）String的基本用法

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

#### （2）String的实现原理

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

<img src="pic\image-20201218183915434.png" alt="image-20201218183915434" style="zoom: 60%;" />

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

### 3. 数组操作类Arrays

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

### 4. 时间和日期的处理

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

<img src="pic\image-20201219131415284.png" alt="image-20201219131415284" style="zoom: 50%;" />

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

### 5. 随机函数

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

## 二、泛型与容器

## （一）泛型

容器类是基于泛型的。

### 1. 泛型的基本概念和原理

复习：Java有8种基本数据类型，而类就是引用数据类型，类之间可以组合、继承，类还有抽象类，子类通过继承来实现抽象类的。接口关注的是是否有该能力，和类平行的概念。——针对接口和能力的编程，可以复用代码，降低耦合，提高灵活性。

而泛型：字面意思就是广泛的类型，**同一套代码可以应用多种数据类型，代码与可操作的数据类型不再绑定在一起**，代码复用性继续提高，降低耦合，提高代码的安全性和可读性。

#### （1）泛型类

```java
// 泛型类
public class Pair<T> {
    T first;		// 实例变量的数据类型也是泛型
    T second;
    public Pair(T first, T second){
        this.first = first;
        this.second = second;
    }
    public T getFirst(){
        return first;
    }
    public T getSecond(){
        return second;
    }
}
```

分析：

1. 泛型的标志：类名后面跟着`<T>`， T代表类型参数，泛型就是**类型参数化，表明它可处理的数据类型不是固定的，而是作为参数传入的**。（就是在使用泛型类的时候，需要将数据类型作为参数传入）
2. 实例变量的数据类型可以是泛型，也可以是正常的数据类型
3. 注意，**在类头设定的类型参数为T，那么在类种如果需要用到泛型，必须用T替代**（不能用其他的）——在多参数的情况下需要注意

```java
Pair<Integer> pair = new Pair<Integer>(1, 100);	// 将数据类型作为参数传入（和LinkedList类似）
Integer first = pair.getFirst();
Integer second = pair.getSecond();
```

注意：传入的数据类型必须是类，而不能是基本数据类型

泛型类，类型可以是多个，在定义的时候，多个类型用逗号分隔

```Java
public class Pair<U, V> {		// 有点类似于HashMap<KEY, VALUE>
    U first;		// U代表一种数据类型
    V second;		// V代表另一种数据类型
    public Pair(U first, V second){		// 一旦传入需要一一对应
        this.first = first;
        this.second = second;
    }
    public U getFirst(){
        return first;
    }
    public V getSecond(){
        return second;
    }
}

// 使用：U=String，V=Integer
Pair<String, Integer> pair = new Pair<String, Integer>("hello", 100);
String first = pair.getFirst();
Integer second = pair.getSecond();
```

=>Java7之后，在创建泛型对象的时候，右边的new不用必须跟指定的参数了，即`Pair<String, Integer> pair = new Pair<>("hello", 100);`

原理：

Java编译器会将泛型代码转换成普通代码，即将**类型T擦除，替换为Object，然后插入必要的强制类型转换**，在JVM执行的时候，只得到了普通数据类型的代码，并不了解泛型的事情。

即上面的代码变成：

```java
public class Pair{
	Object first;
	Object second;
	public pair(Object first, Object second){
		...
	}
    public Object getFirst(){}
    public Object getSecond(){}
}
```

那why不直接用Object替换，而需要引入泛型这个概念呢？

提高安全性和可读性。

只用Object的时候，由于大家的参数都是一样的Object，所以开发环境和编译器无法检查出来：

eg：

```java
Pair pair = new Pair("hello",1);
Integer id = (Integer)pair.getFirst();		// 运行时会出错
String name = (String)pair.getSecond();		// 运行时会出错
```

——编译不会出错，在运行时会抛出异常：ClassCastException（非法强制类型转换异常）

```java
Pair<String,Integer> pair = new Pair<>("hello",1);
Integer id = pair.getFirst(); // 编译出错
String name = pair.getSecond(); // 编译出错
```

——为验证**类型安全**，使用泛型，编译器和开发环境会保证不会使用错误类型

并且，使用泛型不需要进行强制类型转换

#### （2）容器类

容器类是泛型最常用的地方。

容器类就是，容纳并管理多项数据的类。数组就是，但是限制多

容器类适用于各种数据类型，且类型安全（能保证类型不会用错）

容器类的具体类型还可以是泛型类

```java
ArrayList<Pair<String, Integer>> arr = new ArrayList<>();
```

#### （3）泛型方法

方法也可以是泛型，且泛型方法可以出现在普通类中。

```java
public static <T> int indexOf(T[] arr, T target){
    for(int i = 0; i < arr.length; i++){
        if(target.equals(arr[i])) return i;
    }
    return -1;
}

// 使用：
String[] arr2 = new String[]{"hello", "world", "milk"};
System.out.println(Base.indexOf(arr2, "milk"));
```

分析：

1. 类型参数为<T>**放在返回值前**（同样可以有多个类型参数）
2. **方法直接调用**即可，而不用像泛型类一样在new的时候指定数据类型，编译器会自动判断

#### （4）泛型接口

接口也可以是泛型的，那么普通类在实现该接口的时候，**应该指定具体的参数类型**

```java
public interface Comparable<T> {	
	public int compareTo(T o);
}
public interface Comparator<T> {
    int compare(T o1, T o2);
    boolean equals(Object obj);
}

// 实现举例：——在该类中，只能Integer之间比较
public final class Integer extends Number implements Comparable<Integer>{...}
```

#### （5）类型参数的限定

**Java允许给类型参数设定一个上界，即类型参数可以设置为必须是某个上界类型或者其子类**（缩小参数的范围），**限定是通过extends表示的**——这边就不是表示继承了，而是限定

上界可以是：具体的类or具体的接口，也可以是其他类型参数

- 上界是具体的类

  ```java
  // 限定了类型参数必须是Number的子类或Number本身
  public class NumberPair<U extends Number, V extends Number> {
      public double sum(){		// 那么可以使用Number的方法了
          return first.doubleValue() + second.doubleValue();
      }
  }
  ```

  分析：

  1. 指定边界之后，类型擦除不会替换为Object，而是替换为指定的上界，这边就是Number
  2. 因为替换的是Number，那么可以使用Number的方法（抽象方法也可以，其子类会去实现，会去调用对应的子类的实现即可，那么不可能使用的是父类对象（抽象类不能创建实例），而子类一定会去实现）

- 上界是某个接口

  限定是接口，表明该类型参数必须实现了该接口功能，常出现在泛型方法中

  <a name="interface_upper"></a>

  ```java
  // 限定了该类型参数必须要实现了Comparable接口——因为方法里面需要调用该方法
  public static <T extends Comparable> T myMax(T[] arr){
      T max = arr[0];
      for(int i = 1; i < arr.length; i++){
          if(arr[i].compareTo(max) == 1) max = arr[i];	// 因为限定了Comparable接口，所以可以使用该方法
      }
      return max;
  }
  
  // 实际上完整应该如此写：public static <T extends Comparable<T>> T myMax(T[] arr){}
  // 因为Comparable本身也是泛型接口，所以它也需要一个参数
  ```

  分析：对于上界是接口，而接口本身也是泛型接口，那么限定的格式为`<T extends xxx<T>>`——**递归类型限制**，含义是：T是一种数据类型，该数据类型必须实现了Comparable接口，且可以与相同类型的元素进行比较

- 上界是其他类型参数

  eg：

  ```java
  // 实现一个动态数组的功能——底层还是数组，就是当数组不够行的时候，重建一个数组将值赋值过去的操作
  public class DynamicArray<E> {	
      private static final int DEFAULT_CAPACITY = 10;
      private int size;
      private Object[] elementData;
      public DynamicArray() {
      	this.elementData = new Object[DEFAULT_CAPACITY];
      }
      private void ensureCapacity(int minCapacity) {	// 确保添加的时候数组够长
          int oldCapacity = elementData.length;
          if(oldCapacity >= minCapacity){
          	return;
          }
          int newCapacity = oldCapacity * 2;
          if(newCapacity < minCapacity)
          	newCapacity = minCapacity;
          elementData = Arrays.copyOf(elementData, newCapacity);
      }
      public void add(E e) {
          ensureCapacity(size + 1);
          elementData[size++] = e;
      }
      public E get(int index) {
      	return (E)elementData[index];
      }
      public int size() {
      	return size;
      }
      public E set(int index, E element) {
          E oldValue = get(index);
          elementData[index] = element;
          return oldValue;
      }
      public void addAll(DynamicArray<E> c) {	// 将其他数组的值复制到本组中
          for(int i=0; i<c.size; i++){
          	add(c.get(i));
          }
      }
  }
  ```

  分析：这个`addAll`方法存在局限性：

  使用：

  ```Java
  DynamicArray<Number> numbers = new DynamicArray<>();
  DynamicArray<Integer> ints = new DynamicArray<>();
  ints.add(100);
  ints.add(34);
  numbers.addAll(ints); // 编译错误，将Integer数组复制到Number数组中——请求时合理的
  ```

  ——从逻辑上：Integer是Number的子类，为啥不能复制呢？

  $\because$ 在判断的时候：当前是`DynamicArray<Number>`对象，而传递的ints是`DynamicArray<Integer>`对象，编译器认为不是同一个类，也不存在继承关系

  eg：举反例

  ```Java
  DynamicArray<Integer> ints = new DynamicArray<>();
  DynamicArray<Number> numbers = ints; //如果因为Integer是Number而允许次操作
  numbers.add(new Double(12.34));	// 那么该操作也是合法的——不合法，因为在Integer数组中出现了Double类型的值，那么破坏了类型安全的限定
  ```

  => 实际上：虽然Integer是Number的子类，但是**`DynamicArray<Number>`不是`DynamicArray<Integer>`的父类**，那么`DynamicArray<Integer>`的对象也不能赋值过去

  修改：

  ```java
  // 该类的类型参数是E，那么这边设置新的类型参数是T，T的上界是E
  // 在使用的时候就是E=Numbers，T=Integer，那么就能成功复制了
  // 当前是`DynamicArray<Number>`对象，传递的c是`DynamicArray<Integer>`对象，Integer是Number的子类，允许操作
  public <T extends E>void addAll(DynamicArray<T> c) {	// 将其他数组的值复制到本组中
      for(int i=0; i<c.size; i++){
          add(c.get(i));
      }
  }
  ```

总结：**泛型是计算机程序的一种重要思维方式，使得数据结构、算法和数据类型相分离，那么同一套数据结构和算法能够用于很多数据类型中，提高其安全性和可读性**。

Java编译中，通过擦除替换实现泛型到普通类型的转换，而JVM感知不到泛型的存在

### 2. 通配符

#### （1）更为简洁的参数类型限定

上界是其他类型参数，可以用更为简洁的定义方式：

<a name="addall"></a>

```Java
public void addAll(DynamicArray<? extends E> c) {	// 将其他数组的值复制到本组中
    for(int i=0; i<c.size; i++){
        add(c.get(i));
    }
}
```

`<? extends E>`：**有限定通配符，? 就是通配符**，表示要去匹配E或是E的某个子类型，但是没有限定具体是什么类型

比较`<T extends E>`和`<? extends E>`

- `<T extends E>`是用于**定义了一个类型参数，声明了一个类型参数是T**，它可以放在泛型类的后面，泛型方法的返回值前面
- `<? extends E>`是用来**实例化类型参数**，只是这个具体类型是未知的，只知道是E或者是E的子类

——一般能达成同样的目标

#### （2）了解通配符

`<?>`这是无限定通配符，等同于`<T>`

但是无论是无限定通配符or有限定通配符，都限制了**只能读，不能写**

eg：

```java
DynamicArray<Integer> ints = new DynamicArray<>();
DynamicArray<? extends Number> numbers = ints;
Integer a = 200;
numbers.add(a); // 编译错误
numbers.add((Number)a);	// 编译错误
numbers.add((Object)a); 	// 编译错误
```

——?就是表示**类型安全无知**，无论是Number或者其子类，都不能知道其具体的类型，不知道具体类型时，如果允许写入就无法确保其安全性，所以直接禁止

eg：举反例：

```java
DynamicArray<Integer> ints = new DynamicArray<>();
DynamicArray<? extends Number> numbers = ints;		
Number n = new Double(23.0);
Object o = new String("hello world");
numbers.add(n);			// 如果允许，那么Integer的数组存在Double类型和String类型的数据
numbers.add(o);
```

——但是只能读不能写，就使得本应该被允许的操作无法完成

eg：

```Java
// 同数组的两个元素交换
public static void swap(DynamicArray<?> arr, int i, int j){
    Object tmp = arr.get(i);		// 编译通过
    arr.set(i, arr.get(j));		// 编译错误
    arr.set(j, tmp);		// 编译错误
}

// 修改之后：增加一个私有类，实现交换操作，而类型参数是T，是新定义了一个类型参数
private static <T> void swapInternal(DynamicArray<T> arr, int i, int j){
    T tmp = arr.get(i);
    arr.set(i, arr.get(j));
    arr.set(j, tmp);
}
public static void swap(DynamicArray<?> arr, int i, int j){
	swapInternal(arr, i, j);
}
```

——这是一个常用的解决方法（解决通配符带来的，对于合理情况不能写的情况）

public API是通配符形式，本质上去调用的API是类型参数方法

如果类型参数存在依赖，只能用类型参数

eg：

```java
public static <D,S extends D> void copy(DynamicArray<D> dest, DynamicArray<S> src){
    for(int i=0; i<src.size(); i++){
    	dest.add(src.get(i));
    }
}

//可以稍微优化：
public static <D> void copy(DynamicArray<D> dest, DynamicArray<? extends D> src){
    for(int i=0; i<src.size(); i++){
    	dest.add(src.get(i));
    }
}
```

如果返回值依赖类型参数，也不能用通配符

总结，何时能用通配符，何时只能用类型参数

- 通配符都可以用类型参数替换，通配符能实现的，类型参数都能实现
- 通配符的优势是：减少类型参数的数目，形式上简单，可读性好
- 如果**类型参数之间存在依赖/返回值需要类型参数/需要进行写操作**，只能用类型参数
- 一般会将通配符和类型参数配合使用，定义必要的类型参数，其余用通配符替换，eg：copy方法

#### （3）超类型通配符

`<? super E>`：超类型通配符，表示E的某个父类——为了更灵活的写入

eg：在上面的`DynamicArray`增加一个方法：

```java
public void copyTo(DynamicArray<E> dest){	// 就是将当前的数组复制到另一个dest数组中
    for(int i=0; i<size; i++){
    	dest.add(get(i));
    }
}

DynamicArray<Integer> ints = new DynamicArray<Integer>();
ints.add(100);
ints.add(34);
DynamicArray<Number> numbers = new DynamicArray<Number>();
ints.copyTo(numbers);	// 编译错误
```

分析：从逻辑上，将Integer对象的数组复制给Number对象是合理的，但是由于期望的参数类型是`Dynamic-Array<Integer>`, 而`DynamicArray<Number>`并不匹配。是[addAll](#addall)的相对操作（addAll是将其他数组的值复制到本数组中；copyTo是将本数组的值复制到其他数组中）

——使用超类型通配符

```java
public void copyTo(DynamicArray<? super E> dest){
    ...
}
// ——那么匹配的就是E的父类，是允许复制过去的
```

另一个使用场景：是Comparable/Comparator接口，遇到继承的情况

就是父类实现了Comparable接口，而子类就使用父类的实现即可

而现在：希望能用[myMax](#interface_upper)

```java
DynamicArray<Child> childs = new DynamicArray<Child>();
childs.add(new Child(20));
childs.add(new Child(80));
Child maxChild = myMax(childs);		// 希望能使用DynamicArray的方法myMax——编译错误

public static <T extends Comparable<T>> T myMax(T[] arr){}
//——在编译的时候匹配的是Comparable<Child>，而child实现的是Comparable<Base>不匹配

// 修改为，那么就能使用了
public static <T extends Comparable<? super T>> T myMax(T[] arr){}
```

——注意：前面<? extends E>可以用<T extends E>完全替换，而<T super E>是语法错误

#### （4）通配符比较

三种通配符形式<? >（无限定通配符）、<? super E>（超类型通配符）和<? extends E>（有限定通配符）总结：

- 它们的目的都是为了使方法接口更为灵活，可以接受更为广泛的类型；
- **<? super E>用于灵活写入或比较**，使得**对象可以写入（copyTo）父类型的容器，使得父类型的比较方法（compareTo）可以应用于子类对象，它不能被类型参数形式替代**;
- **<? >和<? extends E>用于灵活读取**，使得方法可**读取E或E的任意子类型的容器对象，它们可以用类型参数的形式替代**，但通配符形式更为简洁。

### 3. 局限性

#### （1）使用泛型类、泛型方法、泛型接口

注意的是：

- 基本类型不能用于实例化参数——只能用引用类型

  ```java
  Pair<int> minmax = new Pair<int>(1,100);	// 非法，用包装类替换
  ```

- 运行时类型信息不能用于泛型，即泛型.class是非法的

  内存中的**每个类都有一份类型信息，每个对象也会保存对应类型的信息**

  在Java中，该类型信息也是一个对象，类型为Class，而Class本身也是泛型类。**该类型对象可以通过`<类名.class>`引用，而实例对象可以通过getClass()方法获得**

  ```java
  public final native Class<?> getClass();	// 返回值是一个泛型类
  ```

  该类型对象只有一份，不能支持泛型获取：

  eg:

  ```java
  Pair<Integer>.class		// 非法
      
  if(p1 instanceof Pair<Integer>)		// 非法
  
  if(p1 instanceof Pair<?>)		// 合法的，无限定通配符
  ```

  一个泛型对象的getClass方法的返回值和原始类型对象是相同的

  ```java
  Pair<Integer> p1 = new Pair<Integer>(1,100);
  Pair<String> p2 = new Pair<String>("hello","world");
  System.out.println(Pair.class==p1.getClass()); //true，均为Pair
  System.out.println(Pair.class==p2.getClass()); //true，均为Pair
  ```

- 类型擦除可能会引发冲突

  如果父类：

  ```java
  class Base implements Comparable<Base>{}
  // 子类如果想自己实现comparable方法，这样写是非法的
  class Child extends Base implements Comparable<Child>{}
  ——对于Child来说，Comparable实现了两次
  ```

  ——父子类不能重复实现接口，类型擦除后被认为是同一个

  类型擦除后，只能留一个

  ```Java
  public static void test(DynamicArray<Integer> intArr);
  public static void test(DynamicArray<String> strArr);
  ```

  ——类型擦除后，本质上都是同一个类Object

  同一个泛型作为方法的参数，那么就会被认为是同一个，即使特定了类型参数

#### （2）定义泛型类、泛型方法、泛型接口

- 不能通过类型参数创建对象：

  ```java
  T elm = new T();		// T是类型参数，不能这样创建
  T[] arr = new T[10];	// 编译错误
  ```

  ——因为类型擦除之后，变成`Obejct elem = new Object()`，本质上还是Obejct类型的对象，而无法创建T类型的对象，所以直接限定不允许

  如果要这么做：设计API接受类型对象，即Class对象，并使用Java中的反射机制（后面讲）

- 泛型类类型参数不能用于静态变量和方法

  泛型类型声明的类型参数，只能用在实例变量和方法中

  ```java
  public class Singleton<T> {
      private static T instance;
      public synchronized static T getInstance(){	// ——对于这样的传参/返回值都是不合法的
      if(instance==null){
          // 这里创建实例
      }
      return instance;
      }
  }
  ```

  ——因为如果是合法的，那么对每一种实例化类型，都需要对应生成特定的静态变量和方法，而实际上由于类型擦除，**该类只有一份，静态方法和静态变量都是类型的属性，和类型参数无关**

  但是，对于静态方法，还是可以是泛型方法的，声明自己的类型参数，而与泛型类的类型参数无关——**泛型类和静态泛型方法的类型参数分离**，这个是合法的

- 了解多个类型限定的语法

  类型参数限定还可以限定多个，即支持多个上界，上界之间用&分隔

  如果上界是类，那么类应该放在第一个，类型擦除时，会用第一个上界替换

#### （3）泛型和数组

首先：不能创建泛型数组，主要是为了类型安全

```java
Pair<Object,Integer>[] options = new Pair<Object,Integer>[]{
new Pair("1rmb",7), new Pair("2rmb", 2), new Pair("10rmb", 1)
};			// 提示：非法
```

——**不能创建泛型数组**

因为数组来说，Java知道数组元素实际的类型

```java
Integer[] ints = new Integer[10];	
Object[] objs = ints;		// 可以赋值，因为Object是Integer的父类
objs[0] = "hello";		// 能编译，但是运行时抛出异常
```

——Java知道objs数组的实际类型是Integer，而现在输入的是String类型，所以写入String会抛出异常：`ArrayStoreException`

eg：反例：

```Java
Pair<Object,Integer>[] options = new Pair<Object,Integer>[3];	// 如果允许创建数组
Object[] objs = options;		// 那么能进行赋值
objs[0] = new Pair<Double,String>(12.34,"hello");	// 编译能通过，运行时也不会抛出异常
```

——$\because$ 数组实际类型是Pair，而obj[0]也是Pair，会被认为是可以的，但是它们的类型是不匹配的，因为类型参数的不一样，而泛型是会去执行类型擦除，大家都变成Object，所以被认为是一样的，所以不被认为是错误，且在运行时也不会被认为是错误的，而会在其他地方出错——触发异常

但是如果要存放一组泛型对象**可以使用原始类型的数组**——即不指定具体的类型参数，就当普通类来看

```Java
Pair[] options = new Pair[]{
					new Pair<String,Integer>("1rmb",7),
					new Pair<String,Integer>("2rmb", 2),
					new Pair<String,Integer>("10rmb", 1)
};
```

——**如果要存放一组泛型对象更好的选择是泛型容器**

eg：

```Java
DynamicArray<Integer> ints = new DynamicArray<Integer>();	// 内部是Object数组
ints.add(100);
ints.add(34);
```

 但是，有时候希望能够有一个容器转换成数组的方法，类似如下的：

```Java
Integer[] arr = ints.toArray();	// 先存放在容器中，后将容器中的内容转换到数组中去
```

那么该如何实现呢：利用运行时类型信息和反射机制

```Java
E[] arr = new E[size];		// 不能创建泛型数组

public E[] toArray(){		// 也不能先放在Object类的数组中，后面再转换成E类型
    Object[] copy = new Object[size];
    System.arraycopy(elementData, 0, copy, 0, size);
    return (E[])copy;
}

——会出现强制类型转换异常：Object类型的数据不能转换成Integer
——java需要在运行时知道要转换的数组类型，而这个类型可以作为参数传递
eg：
public E[] toArray(Class<E> type){		// 传递类型参数
    Object copy = Array.newInstance(type, size);	// 产生一个指定类型的数组复制给Object实例——Java知道了copy的真正数据类型
    System.arraycopy(elementData, 0, copy, 0, size);	// 数组复制：elementData -> copy
    return (E[])copy;		// 强制类型转换，因为Java知道了copy的真实数据类型
}

// 使用
Integer[] arr = ints.toArray(Integer.class);
```

总结：

- Java不支持创建泛型数组；
- 如果要存放泛型对象，可以使用原始类型的数组，或者使用泛型容器；
- 泛型容器内部使用Object数组，如果要转换泛型容器为对应类型的数组，需要使用反射

## （二）列表&队列

具体见：[容器]()

## （三）Map&Set

具体见：[容器]()

## （四）堆&优先级队列

具体见：[容器]()

## （五）通用容器类&总结



## 三、动态和函数式编程

## （一）反射

## （二）正则表达式

## （三）函数式编程



## 四、并发

之前都是假设程序中只有一个线程一条执行流，即从main函数开始一条一条执行下去——效率不高

为了提高效率，在程序中**创建多条线程来启动多条执行流**——加快执行速度，但是程序执行变得复杂

## （一）并发的基础知识

### 1. Java线程

这边的线程和OS中的线程还是存在一些差异，但是概念逻辑上类似

在Java中一个线程就是一条执行流，有自己的PC，虚拟机栈（JVM的知识）

#### （1）线程的创建

如何创建线程呢？有两个方法：**继承Thread/实现Runnable接口**

1. 继承Thread类

   `java.lang.Thread`表示线程，**类可以继承并重写`run`方法**，就可以实现一个新线程

   （Thread类本质上也是去实现了Runnable接口）

   ```java
   public Main extends Thread{
       @Override
       public void run(){
           System.out.println("new thread");
       }
   }
   ```

   `run`方法：public修饰符、无返回值、不能抛出受检异常——run方法就是线程的main函数（类似于os中线程实际执行的主函数）

   ——继承并重写方法只是定义了新的线程，但是不会执行，**线程启动过程**：创建对象+调用`start`方法

   ```java
   public static void main(String[] args){
       Thread thread = new Main();		// 引用对象可以是父类类型
       thread.start();		// Thread类中提供的实现方法——启动线程
   }
   ```

   `start`方法：启动该线程，那么它就成为一条单独的执行流，会分配线程相关的资源、pc、栈等，会分配时间片让它执行，而执行的就是`run`方法

   如果，不调用start，而是直接调用run方法，那么**run就是一个普通的方法，而不会创建一个新的线程**，run方法是在main线程中执行

   ——通过调用方法`currentThread()`（静态方法），可以获得当前正在执行的是哪个线程

   ```java
   public static native Thread currentThread();// 获得当前正在执行的线程（是静态native方法）
   public long getId();			// 实例方法，可以用currentThread获得该线程，然后调用实例方法
   public final String getName();
   ```

   如果开启多个线程后，它们并发执行，由操作系统负责调度；如果是单核的就是并发执行；如果多核可以有并行执行——但是这个是底层os实现的，上层不关注

   ps：如果不重写run方法，那么还是能正常执行的，就是该新建的线程无任何操作（创建之后立即销毁？？？）

2. 实现Runnable接口

   使用场景：当某个类已经继承一个父类，而不能再继承一个父类Thread时，只能通过实现`java.lang.Runnable`接口来实现线程

   ```java
   public interface Runnable {
   	public abstract void run();
   }
   ```

   ——只需要实现一个方法`run`即实现了该接口（thread继承是重写了）

   ```java
   public Main implements Runnable{
       @Override
       public void run(){
           System.out.println("son thread");
       }
   }
   
   // 还是要去创建thread的方法，只不过会创建一个Runnable对象传参进去
   public static void main(String[] args){
       Thread thread = new Thread(new Main());	// 创建thread对象，并且传参，使用带参的构造方法
       thread.start();
   }
   
   
   // Thread其中一个构造方法，和上面的匹配
   public Thread(Runnable target) {
       init(null, target, "Thread-" + nextThreadNum(), 0);
   }
   ```

——可以发现，继承Thread或者实现Runnable，都需要通过new Thread来创建线程，并且需要重写/实现run方法（线程的主函数），通过start来启动线程

#### （2）线程的基本属性和方法

- id/name：线程都有一个编号（long类型）和名字（String类型）

  id是一个递增的整数（类似os中的线程创建后给`tid`）

  name默认是：`Thread-编号`（特殊：主线程，名字是main），name可以在创建Thread对象时传参`String name`进去，也可以通过`setName`方法设置

  ——name的主要作用是：方便调试

- priority：**java的线程优先级为[1, 10]，默认为5**

  可以使用的方法有：

  ```java
  public final int getPriority();
  public final void setPriority(int newPriority)
  ```

  ——这个优先级会被映射到所在实际os的实际优先级线程中，由于实际的os不同，所以映射也不同，所以存在**Java不同的优先级可能会映射到相同的优先级中**

  ——不同于os执行很关注thread的优先级，Java中不要太过于依赖优先级

- state：线程的状态

  Java中式一个枚举类型

  ```java
  public enum State {
      NEW,			// 新建状态（还未被调用执行过一次，new Thread后，到thread.start被执行前）
      RUNNABLE,		// 可执行状态（可能是等待分配时间片/正在执行）
      BLOCKED,		// 被阻塞状态
      WAITING,		// 等待状态
      TIMED_WAITING,	// 超时等待状态
      TERMINATED;		// 终止状态
  }
  
  public State getState();		// 获取线程的当前状态
  
  public final native boolean isAlive();	// 判断线程是否存活（从启动开始到终止状态之前都是true，注意是启动start之后，NEW状态下是false）
  ```

- daemon线程：**辅助线程**

  是其他线程的辅助线程，Java可能会为了实现某些功能（打印等功能，而创建实现该特定功能的线程）

  典型的daemon线程是：垃圾回收线程

  概念理解，辅助线程在对应的主线程退出的时候，就没有存在的意义。

  ——所以**程序中只存在daemon线程时，程序就会退出**。

- sleep方法：同os

  ```java
  // 休眠方法，单位是毫秒（可能有一定的偏差）
  public static native void sleep(long millis) throws InterruptException
  ```

  ——睡眠期间，线程可以被中断，被中断后会抛出异常`InterruptException`

- yield方法：同os

  ```java
  // 让方法
  public static native void yield();
  ```

- **join方法**

  ```java
  public final void join() throws InterruptException
  ```

  ——让调用该join的线程，等待指定的线程结束后，再执行下去（即使那个线程被阻塞也还是等待）

  即在当前线程（thread-m）执行过程中，执行到了另一个线程（thread-n）的join方法，就是thread-m去等待thread-n执行完成

  使用方法：

  ```java
  public class Main{			
      public static void main(String[] args){
          Thread thread = new Thread();
          thread.start();
          thread.join();		// 就是main线程去等待thread去执行完成
  	}
  }
  ——如果是在thread_1中去调用thread2.join()，那么就是thread_1去等待thread_2
  ```

  join还能限定等待时间，单位是毫秒`thread.join(long millis)`——在这个时间范围内等待，到期就不再等待继续执行下去

- 过时方法——不应该再使用

  ```java
  public final void stop();
  public final void suspend();
  public final void resume();
  ```

#### （3）共享内存和可能存在的问题——同os中的并发问题

线程有一部分是独有的内存：pc、栈，但是线程之间也存在共享内存，都可以访问和操作的对象——堆

1. 竞态条件：多线程竞争同一个资源，如果**对资源的访问顺序敏感**，就存在竞态条件

   ```java
   public class CounterThread extends Thread {
       private static int counter = 0;			// 静态变量——共享内存
       @Override
       public void run() {
           for(int i = 0; i < 1000; i++) {	// thread的main函数执行的就是循环加操作
           	counter++;
           }
       }
       public static void main(String[] args) throws InterruptedException {
           int num = 1000;
           Thread[] threads = new Thread[num];		// 创建1000个线程
           for(int i = 0; i < num; i++) {			// 启动
               threads[i] = new CounterThread();
               threads[i].start();
           }
           for(int i = 0; i < num; i++) {		// 等待完成
           	threads[i].join();
           }
           System.out.println(counter);
       }
   }
   ```

   ——按照期待来说，最后counter的值为1,000,000，但是一般不能到这个值。主要是这1000个线程是**交替执行的**，所以thread-1在获得count=100之后被切换到thread-2，此时count值没有发生变化，而thread-2也获得了count=100，那么thread-1执行完一个循环=101，thread-2执行完一个循环=101，所以两个循环只增加了1

   但是，线程的循环执行是必须的（时间片轮转），我们需要从其他地方限制‘

   ——**count++ => count = count + 1**该操作不是原子操作，这个语句主要的流程是：去共享内存中获得count的值-->进行加1操作-->然后将count的值写回内存，在这个读取和写回的过程中，共享变量的count是没有发生变化的，如果此时线程切换就会出现竞态

   ——如果将该操作，变成原子操作，那么在执行该步骤的时候不可被切换，那么该操作就是有效的

   如何变成原子操作呢？

   - 使用`synchronized`关键字
   - 使用显式锁
   - 使用原子变量

2. 内存可见性：读操作和写操作发生在不同的线程中，**无法确保读操作能够适时的看到写入的值**，有时甚至一致看不到

   ```java
   public class VisibilityDemo {
       private static boolean shutdown = false;
       static class HelloThread extends Thread {
           @Override
           public void run() {	// thread的main函数——死循环等待shutdown变成true
               while(!shutdown){		
               // do nothing
               }
               System.out.println("exit hello");
           }
       }
       public static void main(String[] args) throws InterruptedException {
           new HelloThread().start();
           Thread.sleep(1000);		// main线程休眠1s后将标志位赋值为true
           shutdown = true;
           System.out.println("exit main");
       }
   }
   ```

   ——在实际运行中，发现程序不会终止，说明子线程一直没有等到shutdown变成true，而实际上在main线程休眠结束后已经修改了，说明**子线程对共享变量值的修改不可见**，主要原因是内存模型的特点造成的：

   <img src="https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMzE1MTAyODU5MDI2?x-oss-process=image/format,png" style="zoom: 80%;" >

   实际上对每个变量（私有变量、共享变量）的操作都只是发生在本地内存区中，本地内存区是不可共享的，共享变量修改完成后，会更新到共享内存区，而其他读该共享变量后进行对应的修改——问题就在于：写线程何时push值，读线程何时pull值，这两个地方造成了不可见性

   解决方法：

   - 使用`volatile`关键字——轻量级
   - 使用`synchronized`关键字——性能问题，效率低易阻塞
   - 使用显式锁同步——性能问题，效率低易阻塞

#### （4）线程的优点及成本

优点是：

- 充分利用多cpu的计算能力——在多cpu环境下
- 充分利用各种资源：cpu、硬盘、网络这些是可以并行工作的，即在等待网络的时候，可以让其他线程去使用cpu
- 在GUI，保持程序的响应性，界面和后台任务是不同的线程，即后台在执行的时候，界面不会完全停止响应，而是时间片轮转时而响应。而如果只是单线程就会出现顺序执行
- 简化建模及IO处理，对每个用户请求使用一个单独的线程进行处理

成本在于：

- 上下文切换是有代价的：时间上 和 cpu缓存上

所以需要权衡优点和成本：比如是多任务下，那么切换为多线程是合理的，而如果只是简单的操作，例如上面的count++，那么单线程就足够了；如果执行的任务是cpu密集型的（就是一直在使用cpu进行计算，而不会去等待资源啥的，那么创建多个线程反而会拖慢执行）

### 2. 关键词synchronized：同步锁

#### （1）用法

可以用来**修饰实例方法、静态方法、代码块**，不针对变量。

举例：

```java
public class Counter {			// 计数类
    private int count;
    public synchronized void incr(){		// ++方法是通过synchronized修饰的
    	count ++;
    }
    public synchronized int getCount() {		// 获取值的方法也是通过synchronized修饰的
    	return count;
    }
}
```

```java
public class CounterThread extends Thread {
    Counter counter;			// 线程的共享对象counter
    public CounterThread(Counter counter) {
    	this.counter = counter;
    }
    @Override
    public void run() {			// 线程的main方法
        for(int i = 0; i < 1000; i++) {		
        counter.incr();
        }
    }
    public static void main(String[] args) throws InterruptedException {
        int num = 1000;
        Counter counter = new Counter();
        Thread[] threads = new Thread[num];
        for(int i = 0; i < num; i++) {			// 创建1000个线程
            threads[i] = new CounterThread(counter);
            threads[i].start();
        }
        for (int i = 0; i < num; i++) {		// 运行该1000个线程
        	threads[i].join();
        }
        System.out.println(counter.getCount());
    }
}
```

——此时的count输出就是预期的1,000,000

#### （2）原理

**synchronized保护的是对象，而不是方法**，即可以同时调用同一个方法，只要它们的对象不同就可以。而对于多线程对同一个对象的操作——**synchronized保证只能有一个线程去执行该对象的方法操作**，该线程执行完成该方法后，才可能让其他线程去执行

synchronized实例方法保护的实际上是**this**，this含有一个锁和等待队列，尝试去获得锁，得不到该线程就会挂在等待队列上，当前线程的状态变成`BLOCKED`——操作和os一致，不再详细解释

——synchronized保护的是对象而不是代码，**只要访问的是，同一个对象的synchronized方法，那么不同的代码也会要求顺序执行**，eg：两个线程一个执行`incr`，另一个执行`getCount`，那么还是不能同步执行，需要一个执行完，才能调度另一个执行。

ps：synchronized方法不能阻止非synchronized方法被同时执行，即**synchronized只能保证有这修饰符的方法们之间的顺序执行，而对非修饰符的方法无效**，eg：count--的decr方法是非synchronized，那么会和incr方法同时执行

——**需要将所有被保护的变量相关的方法上均加上synchronized**

#### （3）静态方法

前面讲的是，synchronized修饰实例方法，而它也能够修饰静态方法

修饰静态方法和修饰实例方法的对象主体不一样：**实例方法保护的是当前实例对象this**，**静态方法保护的是类对象，即xxx.class**——实例对象、类对象都有一个锁和等待队列

所以，如果两个线程一个执行synchronized修饰的静态方法，另一个执行synchronized修饰的实例方法，是可以同时执行的

#### （4）代码块

synchronized可以修饰代码块，形式如下：

```java
public class Counter {
    private int count;
    public void incr(){
        synchronized(this){
        	count ++;
        }
    }
}
```

`synchronized(xxx){....}`——括号里面的就是保护的对象，实例方法就是this，静态方法就是`Counter.class`，大括号里面的就是同步执行的代码。

——所以，synchronized修饰的静态方法和实例方法都可以转换为对应的代码块

**synchronized的对象可以是任意对象，因为所有对象都有一个锁和对应的一个队列**

#### （5）进一步理解

- 可重入性：针对锁

  synchronized是可重入的，即一个线程在获得锁之后，如果执行过程中又需要该锁，那么还是可以获得该锁的——只针对拥有该锁的线程（同zephyr里面的锁机制）

  实现机制：会记录该锁拥有的线程和进入的次数。

  所以流程是：当执行到受保护的方法时，先尝试获取锁，如果没有其他线程持有锁，那么就获得了该锁，锁会记录持有的线程+进入的次数（1）；如果有线程持有该锁，如果该线程就是当前线程，那么更新进入次数（++），如果该线程不是当前线程，那么将当前线程挂到等待队列上。

  ——注意：每次释放都会减少进入次数，**只有次数减少到0，才会释放该锁**

- 内存可见性

  前面描述的都是竞态的情况。而对于内存可见性也是synchronized能保证的

  **释放锁时，会将所有的写入写回内存；获得锁后，会从内存中读最新数据**

  但是synchronized主要就是来避免竞态情况的，而对于**实现内存可见性代价太大**

  ——轻量级的方法是：**给变量添加修饰符volatile**

  ```java
  public class Switcher {
      private volatile boolean on;		// volatile是添加在变量上的，而不是方法上
      public boolean isOn() {
      	return on;
      }
      public void setOn(boolean on) {
      	this.on = on;
      }
  }
  ```

  ——Java对于volatile修饰的变量，会在操作该变量的时候插入特殊的指令，**保证读取到内存最新值**

- 死锁

  锁的通病，死锁问题，即两个及以上的线程持有锁，且都在等待对方的锁

  如何解决：死锁预防、死锁避免、死锁检测

  java提供了：显式锁接口Lock，它能够尝试获取锁（tryLock）和带超时等待的获取锁的方法——获取不到锁的时候会释放持有的锁，然后再次申请该锁，或者直接放弃申请该锁

  Java不会处理死锁，但是又工具会报告死锁**jstack**

#### （6）同步锁修饰的容器类

`Collection`存在子类，通过对每个方法都修饰了synchronized，那么就是**线程安全的同步容器**——这边的安全是针对同一个容器对象

```java
public static <T> Collection<T> synchronizedCollection(Collection<T> c);
public static <T> List<T> synchronizedList(List<T> list);
public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m);
```

但是还是要注意如下问题：

- 复合操作

  可能每一步都是线程安全的，但是合起来确不是

- 伪同步

  注意要同步谁，就指定谁，如果是修饰方法，那么默认的就是指定this对象；如果是修饰代码块，可以指定对象。

- 迭代

  同步容器并没有实现遍历和修改同时发生，即如果遍历的时候进行修改，会抛出异常，需要在遍历的时候给整个容器对象加锁，限制只能遍历结束之后才能修改

- 并发容器

  同步容器的性能较差，当并发访问量大的时候性能不好。

  这时候可以选择用并发的容器类：CopyOnWriteArrayList；ConcurrentHashMap；ConcurrentLinkedQueue；ConcurrentSkipListSet

  ——性能高，没有使用synchronized关键字，没有迭代等问题

### 3. 线程的基本协作机制

线程之间有竞争，也有协作，对应的关键字是：`wait/notify`

多线程的协作本质上就是：条件不满足时**主动等待**；条件满足时**被唤醒**继续执行

#### （1）协作的场景

- 生产者/消费者模式：生产者和消费者通过**共享队列**协作，生产者将产生的数据或任务放到队列上，消费者从队列上获取数据或任务。

  那么队列满时，生产者需要等待消费者消费；队列为空时，消费者需要等待生产者生产

- 同时开始：多线程同时开始，多在模拟仿真程序中

- 等待结束/主从模式：主线程将任务分解为多个子任务，给每个子任务分配线程，那么主线程需要等待所有的子线程执行完成后才能继续执行

- 异步调用：主线程去调用子线程后，让子线程去执行，且直接返回到主线程继续执行，返回值为**future对象**，它在未来某个时间点能够得到子线程的解

- 集合点：多个线程分别去执行，执行到某个点就等待其他线程执行到该集合点，直到所有线程均到了，就开始执行下面的内容，eg：并行迭代计算中，每个线程负责一部分计算，然后在集合点等待其他线程完成，所有线程到齐后，交换数据和计算结果，再进行下一次迭代（类似于GPU的计算）

#### （2）wait/notify

wait和notify是根父类Object提供的两个native方法

```java
public final native void wait(long timeout) throws InterruptedException;// 设置超时时间，超时了就不等待了，如果参数=0，那么就无限期等待；等待期间可以被中断，如果被中断会抛出异常

public final native void notify();		// 唤醒其中一个，看底层的os，可能存在随机性
public final native void notifyAll();	// 唤醒所有在条件队列上的线程
```

ps：wait的抛出中断异常，也不算是出现bug，而是表示该wait方法是可中断的，那么该方法会对中断异常进行处理（而不同于中断程序自己抛出中断异常）

wait/notify的工作原理：

前缀知识：每个对象都有锁和锁对应的等待队列，此外**还存在一个队列——条件队列，主要用来协程间的协作**

调用wait之后：会将当前线程会**释放当前所拥有的锁**，并且**挂到该对象（this）的条件队列**上并阻塞，表示要等待一个条件才能继续执行——**该条件自身已经无法改变了，需要其他线程改变**（注意这边是存在共享变量的）

其他线程改变条件后，会在该线程中**调用该对象的notify方法**，从而唤醒对象的条件队列上的某一个线程

注意：wait/notify必然涉及到对共享变量的访问和修改操作，是会存在竞态问题，所以相关的代码都需要synchronized修饰——规定了**wait/notify方法必须在synchronized代码块中被被调用**，即先获取锁才能执行wait/notify，否则会抛出异常`java.lang.IllegalMonitor-StateException`

下面分析wait的整个过程：

1. 看某个资源是否存在，如果存在就执行下去，不在就进入处理程序中

   一般都是形如：`while(xxx){ this.wait();}`——while中存放的都是不满足的情况

   ——一般不用if语句，因为wait返回之后直接执行到了下面，可能又不满足上面的语句了，所以设置为while最为安全

2. 将**当前线程**加入条件等待队列，**释放持有的锁**，变成阻塞状态`WAITING`/`TIMED_WAITING`

   可能存在多个线程都会去调用该对象的该方法，导致被挂起，从而条件队列上存在多个线程

3. 其他线程开始执行，直到执行到某个变量条件修改后，**使得步骤1条件变得满足**（如果还是不满足，那么释放没有意义，还是会被阻塞），然后该线程中显式调用**该对象(this).notifyAll()**

   这边多选择用`notifyAll`而不是`notify`，唤醒所有线程更为安全，而如果只唤醒一个线程不确定性太多

   ps：**notify/notifyAll只会移除条件队列中等待的线程，而不会释放锁**，即notify会将synchronized代码块中的执行完成后，才开放抢占

4. 如果等待的线程有设置超时等待，超时了还未被唤醒，那么该线程主动清醒，继续执行下去不再等待；如果在等待过程中，其他线程调用notify将它们唤醒，它们会从条件队列中被移除，**重新竞争对象锁，因为synchronized机制**

   - 如果能够获得锁，那么状态就会变成`RUUABLE`，并且从wait下面开始返回
   - 如果无法获得锁，那么就会从条件队列中转移到锁的等待队列，线程状态变成`BLOCKED`

总结：wait/notify是**围绕一个共享的条件变量进行协作的**。该条件变量是**程序自行维护的**，当条件不成立，那么当前线程主动将自己挂起（而前面的锁是被动挂起的，实际上就是内置了一个block函数；主动就是如果没有写wait，实际上并没有后面这些事了）；另一个线程在后面执行过程中修改了条件变量，主动去调用了notify；然后**前一个线程返回后需要重新检查条件变量**（while循环去检查）

存在的问题：wait是让线程休眠，挂在等待队列上，而只有这一个队列，不存在其他的等待队列。

典型的情况：下面的生产者消费者问题

改进：使用ReentrantLock的`condition`，有一个condition就对应产生一个队列，然后将要挂起的生产者加入到一个condition的等待队列上；将要挂起的消费者加入到另一个condition的等待队列上。

#### （3）举例

- 生产者消费者问题

  ```java
  static class MyBlockingQueue<E> {			// 长度有限的队列，可以在上面取数据or放数据
      private Queue<E> queue = null;
      private int limit;
      public MyBlockingQueue(int limit) {
          this.limit = limit;
          queue = new ArrayDeque<>(limit);
      }
      public synchronized void put(E e) throws InterruptedException {	// 给生产者调用的
          while(queue.size() == limit) {		// 队列满了，就等待
          	wait();
          }
          queue.add(e);
          notifyAll();
      }
      public synchronized E take() throws InterruptedException {	// 给消费者调用的
          while(queue.isEmpty()) {		// 队列空了就等待
              wait();
          }
          E e = queue.poll();
          notifyAll();
          return e;
      }
  }
  ```

  ——可以发现，生产者线程被挂起是队列满了，而消费者被挂起是队列空了。但都是挂在MyBlockingQueue实例对象上的——它们的等待条件不同，但是都挂在同一个队列上，所以如果等待条件不同，但是又挂在同一个条件队列上，那么需要使用**notifyAll**去唤醒所有线程，防止使用notify唤醒不该唤醒的。

  所以也说明了wait/notify的局限性：**只能有一个条件等待队列**（显式锁和条件可以解决这个问题）

- 同时开始：让多个线程去等待同一个变量的值变化，而让一个线程去改变该值，改变之后将所有在等待的线程notifyAll，那么就是同时开始

- 等待结束

  ```java
  // thread.join()的原理——
  while (isAlive()) {		// 判断要等待的线程（this）是否还是存活状态——带超时等待的；如果存活那么就挂起当前线程（就是调用该方法的线程）
      long delay = millis - now;
      if (delay <= 0) {
          break;
      }
      wait(delay);		// 如果没有到期就挂起
      now = System.currentTimeMillis() - base;
  }
  ```

  而这个是由该线程在执行完成之后，会主动调用notify去唤醒线程的

  如果需要等待多个线程，那么可以换成一个计数共享变量来记录要等待的线程数，当值>0时就挂起等待；当一个线程执行完成后去--，然后唤醒该线程，该线程会去判断该值是否==0，否则就继续挂起；是就表示等待结束，该线程可以执行下去了

  **Java中有一个专门的同步类，CountDownLatch——就是该原理**

- 异步结果：没看懂，可能是异步这个概念还没了解

  表示异步结果的接口Future和实现类FutureTask；用于执行异步任务的接口Executor，以及有更多功能的子接口ExecutorService； 用于创建Executor和ExecutorService的工厂方法类Executors。

- 集合点：分头行动，到某个特定的点后需要到齐所有线程，交换数据后才能再次分头行动

  ```java
  public class AssemblePoint {
      private int n;
      public AssemblePoint(int n) {
      	this.n = n;
      }
      public synchronized void await() throws InterruptedException {
          if(n > 0) {		// 说明有线程未到齐
          	n--;
              if(n == 0) {		// 如果减去自己后为0，那么已经到齐，将所有线程唤醒
                  notifyAll();
              } 
              else {	// 如果减去后不为0，那么未到齐，挂起等待，直到为0后被唤醒——唤醒后还是会去判断是否等于0——再次验证
                  while(n != 0) {
                      wait();
                  }
              }
         	}
      }
  }
  ```

  Java中有一个专门的同步工具类CyclicBarrier可以替代它

### 4. 线程的中断

#### （1）why需要线程取消/关闭

一般流程下：线程通过`thread.start();`的方法启动线程，然后时间片轮转后开始执行`run()`方法，然后该方法运行结束后**线程正常退出**。

但是还是有其他场景需要手动结束线程

- 某些线程的执行是死循环，当达到某个条件后需要正常关闭线程

  eg：生产者/消费者模式下，两个线程都是死循环，需要我们来自行关闭线程

- 在GUI用户界面程序，线程是用户启动的，eg：下载线程。但是下载线程可能会被用户提前取消该任务，即取消该线程，我们需要能响应该请求

- 我们需要在限定时间内得到某个结果，如果得不到，就取消该任务，也即取消该任务

- 启动多个线程去做同一件事，当事件完成，那么需要取消其他线程

  eg：多线程抢票

#### （2）how取消/关闭

可以使用`stop()`关闭，但是标记为过时，不能使用。

Java中停止一个线程的机制就是**中断**，但是**中断并不是强制终止一个线程，而是一种协作机制，是给线程一个取消信号，然后由线程来决定如何以及何时退出**（所以在run中我们需要在实现的时候，考虑意外终止中断线程的判断操作）

Thread类中

```java
public boolean isInterrupted();		// 实例方法：判断该线程是否在中断状态中：true——在中断状态

public void interrupt();		// 实例方法：设置中断
// ——中断对应的线程：thread.interrupt()

public static boolean interrupted();	// 静态方法：设置中断——就是通知该线程中断了，需要提前结束程序
// ——判断当前线程是否处于中断当中（true），并且清空中断标志位，即原来如果是true，会被赋值为false，如果是false那么还是false
```

——**每个线程都有一个中断标志位**，true：表示中断；false：表示未中断

`interrupt()`的操作相关的受线程状态和IO操作有关（IO本身也是外部中断，IO操作的影响和具体的IO及操作系统相关，这边不关注）。

线程的状态有：

- RUNNABLE：线程可执行状态，等待cpu调度 or 已经在运行状态
- WAITING/TIMED_WAITING：线程在等待某个条件或者超时等待
- BLOCKED：线程在等待锁，进入临界区
- NEW/TERMINATED：线程未启动 or 线程已结束

1. 对于RUNNABLE状态的线程

   如果在该状态下，且没有在执行IO操作，那么`interrupt()`调用之后只是会去设置中断标志位

   线程在运行过程中，需要在合适的位置去检查中断标志位，看是否需要提前终止线程的run方法

   eg：run的主要代码是一个循环，那么可以在循环开始处进行检查

   ```java
   public class InterruptRunnableDemo extends Thread {
       @Override
       public void run() {
           while(!Thread.currentThread().isInterrupted()) {
               // 循环内代码
           }
           System.out.println("done ");
       }
   }
   ```

2. 对于waiting/timed_waiting状态的线程

   线程用join/wait/sleep会使线程进入waiting/timed_waiting状态。如果在此状态下，**对线程调用interrupt()使得该线程抛出InterruptedException异常**（在适时的时候，需要接受该异常并且做出相应处理）

   但是：**抛出异常后，中断标志位会被清空，即被设置为false**

   ```java
   Thread t = new Thread (){
       @Override
       public void run() {
           try {
           	Thread.sleep(1000);		// 线程休眠——在休眠的时候，被main线程提示取消，那么抛出异常
           } catch (InterruptedException e) {
           	System.out.println(isInterrupted());
           }
       }
   };
   t.start();		// 启动线程t
   try {
   	Thread.sleep(100);		// 当前线程main休眠
   } catch (InterruptedException e) {}
   t.interrupt();		// 由于main线程休眠时间短，那么就会去给线程t设置中断标志位，提示线程取消
   ```

   ```java
   public static void sleep(long millis) throws InterruptedException{....}
   // ——该方法可能会抛出异常
   ```

   **InterruptException是一个受检异常，线程必须处理**，所以不能不管

   而处理异常的基本思路是：**如果知道该如何处理，那么就处理；如果不知道如何处理，那么向上传递**——如果catch异常后需要进行处理，而不是忽略。

   如果catch到了InterruptException，一般就是通知该线程要提前结束了，该线程通常有两种处理方式：

   - 继续向上传递该异常，那么该方法也变成了一个可中断的方法，需要调用者进行处理

   - 不能再向上传递，eg：Thread的run方法，由于它方法是固定的，不能再向上抛出异常了，那么就只能catch该异常进行处理

     ——该操作之后，一般还是需要调用interrupt方法再去设置中断标志位，从而让其他代码知道已经发生了中断

     eg：

     ```java
     public class InterruptWaitingDemo extends Thread {
         @Override
         public void run() {
             while(!Thread.currentThread().isInterrupted()) {	// 如果当前线程在中断状态
                 try {
                 	Thread.sleep(2000);		// 尝试执行sleep
                 } catch(InterruptedException e) {		// 如果在执行sleep的时候被传递了终止信息
                     // ... 针对终止的相关操作
                 	Thread.currentThread().interrupt();	// 重新设置中断标志位——让其他代码知道该线程发生中断
                 }
             }
             System.out.println(isInterrupted());
         }
         // 其他代码
     }
     ```

3. 对于blocked状态的线程

   说明该线程是在等待锁，调用`interrupt()`**只是会去设置线程的中断标志位**——而没有任何操作，线程还是处于blocked状态

   ——**`interrupt()`不会使在一个等待锁的线程真正中断**

   ```java
   public class InterruptSynchronizedDemo {
       private static Object lock = new Object();
       private static class A extends Thread {
           @Override
           public void run() {
               synchronized (lock) {			// 获取锁的情况下，去执行
                   while (!Thread.currentThread().isInterrupted()) {
                   }	// 如果该线程（当前线程）没有在中断状态，那么就执行循环；如果在中断状态，那么就退出
               }
               System.out.println("exit");		// 线程即将退出
           }
       }
       public static void test() throws InterruptedException {
           synchronized (lock) {			// 获取锁的情况下，去执行
           A a = new A();		// 创建一个线程
           a.start();		// 启动
           Thread.sleep(1000);	// main线程休眠
           a.interrupt();		// 休眠结束后，设置线程a为中断状态
           a.join();		// 等待线程a执行完成
           }
       }
       public static void main(String[] args) throws InterruptedException {
       	test();
       }
   }	
   ```

   执行过程中，main线程先去获取锁，然后启动线程a，尝试获取该锁，但是该锁被main线程占有，那么会加入锁的等待状态，main线程继续执行，main线程通知线程a的取消信号——只是设置线程的中断状态，但是线程a依旧在锁的等待状态中。

   ——**`synchronized`关键字中，线程去等待锁的时候，不去响应中断请求**——synchronized的局限性，如果需要解决该局限性问题，应该使用显式锁

4. 对于new/terminated状态的线程

   在该状态下，调用`interrupt()`是没有作用的

——所以，`interrupt`不会真正中断线程（和os不一样），**只是一种协作机制，如果不了解线程在做什么，不应该去调用interrupt**

对于那些对线程提供服务的程序来说，应该封装取消线程操作，给调用者封装好的方法，而不是直接让它调用interrupt

所以对于线程的实现者，应该封装好线程的终止方法，并文档描述行为；

作为调用者，应该使用封装好的方法，而不是直接使用interrupt，导致不符合预期

eg：

Future接口：`boolean cancel(boolean mayInterruptIfRunning);`

ExecutorService接口：`void shutdown();List<Runnable> shutdownNow();`

## （二）Java的并发工具包

Java中存在一套并发工具包，位于`java.util.concurrent`，包括易用且高性能的并发开发工具

### 1. 原子变量及其原理

前提知识：synchronized关键字，能够保证修饰的方法/代码块是原子性操作。但是对于简单的操作，用synchronized关键字成本太高，需要获取锁，最后释放锁；如果获取不到还需要等待锁，并且涉及到了上下文切换。

——这个可以使用：**原子变量代替**（即该变量能够保证该变量的操作是原子性的，不会被打断）。Java并发包中提供了基本原子变量类型：

- AtomicBoolean：原子Boolean类型，常用来在程序中表示一个标志位；

- AtomicInteger：原子Integer类型；

- AtomicLong：原子Long类型，常用来在程序中生成唯一序列号；

- AtomicReference：**原子引用类型**，用来以原子方式更新复杂类型

  ——从而解决了之前只能够对一个共享变量进行原子操作，将多个共享变量组合成一个对象，从而调用`AtomicReference`来实现原子性

还有一些其他类

- 如针对数组类型的类，AtomicLongArray、AtomicReferenceArray，
- 以原子方式更新对象中的字段的类，如AtomicIntegerFieldUpdater、AtomicReferenceFieldUpdater等。

Java 8增加了几个类，在高并发统计汇总的场景中更为适合，包括LongAdder、LongAccumulator、Double-Adder和DoubleAccumulator，

#### （1）AtomicInteger

基本原子变量拿`AtomicInteger`举例

基本用法：

```java
public AtomicInteger(int initialValue);	// 构造方法：设置初始化值
public AtomicInteger();		// 构造方法：无参数，默认初始化值为0
```

```java
public final int get();		// 获得原子变量当前的值
public final void set(int newValue);		// 设置原子变量的值
```

——这些都是普通的方法。

原子变量，体现在它包含了一些**以原子方式实现组合操作**的方法：（即多个操作，都能保证原子性）

```java
//以原子方式获取旧值并设置新值
public final int getAndSet(int newValue);
//以原子方式获取旧值并给当前值加1
public final int getAndIncrement();	// 同i++
//以原子方式获取旧值并给当前值减1
public final int getAndDecrement();	// 同i--
//以原子方式获取旧值并给当前值加delta
public final int getAndAdd(int delta);
//以原子方式给当前值加1并获取新值
public final int incrementAndGet();		// 同++i
//以原子方式给当前值减1并获取新值
public final int decrementAndGet();		// 同--i
//以原子方式给当前值加delta并获取新值
public final int addAndGet(int delta);
```

——本质上都是去依赖了一个public方法：**CAS方法**

```java
public final boolean compareAndSet(int expect, int update);		// CAS方法
```

如果当前值等于`expect`，那么就将该值设置为`update`，并且返回true；如果值不等于`expect`，那么不设置，直接返回false

——乐观锁（认为发生冲突的可能性比较小），尝试去修改变量，修改不了就返回

AtomicInteger可以在程序中作为一个计数器，多个线程并发去更新，并且能够保证值的正确性

```java
public class AtomicIntegerDemo {
    private static AtomicInteger counter = new AtomicInteger(0);
    static class Visitor extends Thread {			// 静态内部类
        @Override
        public void run() {
            for(int i = 0; i < 1000; i++) {
            	counter.incrementAndGet();		// 原子++i
            }
        }
    }
    public static void main(String[] args) throws InterruptedException {
        int num = 1000;
        Thread[] threads = new Thread[num];
        for(int i = 0; i < num; i++) {			// 创建1000个线程
            threads[i] = new Visitor();
            threads[i].start();
        }
        for(int i = 0; i < num; i++) {			// main线程等待所有1000个线程执行完成再执行下去
        	threads[i].join();
        }
        System.out.println(counter.get());
    }
}
```

基本原理：

AtomicInteger的实现：

```java
private volatile int value;		// 主要内部成员
```

——声明中带有：volatile，能够保证内存可见性

模拟其中一个原子方法：

```java
public final int incrementAndGet() {
    for(;;) {
        int current = get();		// 获得当前的值
        int next = current + 1;		// 预计++之后的值
        if(compareAndSet(current, next))	// 尝试去修改该值，如果修改成功那么就返回成功的值；如果修改失败，继续循环继续尝试++
        	return next;
    }
}
```

——代码主体是个死循环，先获取当前值current，计算期望的值next，然后调用CAS方法进行更新，如果更新没有成功，说明value被别的线程改了，则再去取最新值并尝试更新直到成功为止。

CAS的实现原理：

```java
public final boolean compareAndSet(int expect, int update) {
	return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}

private static final Unsafe unsafe = Unsafe.getUnsafe();
```

——调用了unsafe的`compareAndSwapInt`方法，而unsafe是sun.misc.Unsafe类的实例对象

unsafe从字面意思表示不安全，一般应用程序不能直接使用，但是大部分的计算机**底层硬件已经能够直接实现CAS**，具体原理可以参考[Java额外知识](http://note.youdao.com/noteshare?id=9f8ab6eaf048c06b8c823ebe1c7f0b8c&sub=38FCCF96A0C74660A8B19E1140BA449E)。对于上层来说，将CAS作为基本操作直接使用即可

synchronized锁是一个**悲观锁**：认为会发生并发冲突，所以要去先去获取锁，得到锁后才能更新；是**阻塞式算法**，如果得不到锁，那么就加入等待队列——有获取锁、释放锁，上下文切换的代价，阻塞的代价

CAS是**乐观锁**：认为冲突比较小，尝试更新，进行冲突检测，如果的确发生冲突，那么继续尝试更新；是**非阻塞式算法**，冲突时就重试——无阻塞、无上下文切换，**对于简单的操作，在高低并发的情况下，性能都会高很多**。

基本数据类型对应的原子变量的原理比较简单，但是对于复杂的数据结构，非阻塞的方式比较难实现和理解，但是Java已经提供了内置函数：

- ConcurrentLinkedQueue和ConcurrentLinkedDeque：非阻塞并发队列；
- ConcurrentSkipListMap和ConcurrentSkipListSet：非阻塞并发Map和Set；

实现锁：

**CAS可以用来实现非阻塞的乐观锁，eg：AtomicInteger，也可以用来实现阻塞式的悲观锁。**实际上，Java的并发包中的乐观锁是用CAS实现的，阻塞式工具、容器、算法也是基于CAS的，eg：Lock。

自行实现的显式锁：（用原子变量实现的）

```java
public class MyLock {
    private AtomicInteger status = new AtomicInteger(0);
    public void lock() {
        while(! status.compareAndSet(0, 1)) {		// 尝试获取锁，要求当前值为0，那么能得到锁，并且设置为1——否则，就挂起
            Thread.yield();
        }
    }
    public void unlock() {			// 最后执行完成释放锁，将1变成0，表示释放
        status.compareAndSet(1, 0);
    }
}
```

——和普通的锁还是存在区别的，无等待队列，也没有在释放锁的时候唤醒线程，这边是处于一个忙等待的状态（如果是按照优先级调度，容易出现死锁情况，低优先级的线程获得锁，但是`thread.yield()`一直不让它执行，那么高优先级一直等不到锁，而低优先级一直持有锁却得不到执行）

实际开发中应该使用Java并发包中的类，如ReentrantLock。

CAS的问题：

- ABA问题：即当前值为A，并发执行中，另一个线程将A修改成B，之后又修改为A。而对当前线程来说**无法分辨该值中间发生过变化**。

  ——如果这个执行顺序是存在问题的，可以加上时间戳，用另一个原子类：AtomicStampedReference。只有时间戳和值都相同时才能进行修改。

  ```java
  public boolean compareAndSet(
      V expectedReference, V newReference, int expectedStamp, int newStamp)
  ```

  ——需要修改两个变量，在原理上，内部AtomicStampedReference会将两个值**组合为一个对象**，修改的是一个值

  ```java
  public boolean compareAndSet(V expectedReference, V newReference,
                               int expectedStamp, int newStamp) {
      Pair<V> current = pair;
      return
          expectedReference == current.reference &&
          expectedStamp == current.stamp &&
          ((newReference == current.reference &&
            newStamp == current.stamp) ||
           casPair(current, Pair.of(newReference, newStamp)));
  }
  
  private static class Pair<T> {
      final T reference;
      final int stamp;
      private Pair(T reference, int stamp) {
          this.reference = reference;
          this.stamp = stamp;
      }
      static <T> Pair<T> of(T reference, int stamp) {
          return new Pair<T>(reference, stamp);
      }
  }
  ```

  ——通过将两个变量组合成一个新的实例变量，然后通过比较该实例变量的值，然后组合修改该实例变量

总结：原子变量背后的原理是CAS，对于并发计数和产生序列号这种比较简单的操作，应该使用原子变量。**CAS是Java并发包的基础**，可以实现高效、乐观、非阻塞式算法，也是并发包中锁、同步工具和各种容器的基础（即CAS也可以用来实现阻塞的、悲观的算法）

### 2. 显式锁

由于：synchronized存在一些局限性，比如不能响应中断等。所以在这种情况下可以用显式锁进行替换

Java并发包中的显式锁接口和类位于包`java.util.concurrent.locks`下，主要接口和类有：

- 锁接口Lock，主要实现类是`ReentrantLock`：可重入锁
- 读写锁接口ReadWriteLock，主要实现类是`ReentrantReadWriteLock`

#### （1）接口Lock

接口定义：

```java
public interface Lock {
    void lock();
    void unlock();		// ——普通的锁获取、释放的方法，获取不到锁会阻塞；释放锁后会通知所有等待的线程
    
    void lockInterruptibly() throws InterruptedException;	// 加上了可响应中断的功能——如果被其他线程中断，会抛出异常
    boolean tryLock();		// 尝试获取锁，立即返回不阻塞，获取到就返回true；获取不到就返回false
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;	// 尝试获取锁，在限定时间内阻塞等待；在等待的过程中能够响应中断，如果在等待中发生中断就抛出异常
    Condition newCondition();		// 新建一个条件，一个lock可以关联多个条件
}
```

——所以，Lock相较于synchronized，能够支持基本的锁，也能够支持可中断锁、限定时间阻塞锁、非阻塞方式获取锁，更为灵活

#### （2）可重入锁ReentrantLock

实现了上面的lock接口。它的基本用法lock/unlock和synchronized一样：可重入；可以解决竞态条件问题；可以保证内存可见性。

```java
public ReentrantLock();		// 默认构造方法
public ReentrantLock(boolean fair);		// 带参构造方法
```

ps：对于下面的带参构造方法，参数fair表示是否保证公平：等待时间最长的线程优先获得锁。保证公平会影响锁的性能，所以默认为false，不保证（synchronized也是不保证公平的）

**使用显式锁，要lock/unlock成对使用**，一般而言，应该将lock之后的代码包装到try语句内，在finally语句内释放锁。

eg：

```java
public class Counter {
    private final Lock lock = new ReentrantLock();
    private volatile int count;
    public void incr() {
        lock.lock();		// 获得锁
        try {			// 获得锁后的具体操作
            count++;
        } finally {
            lock.unlock();		// 最后释放锁，放在finally中保证会执行，即使抛出异常
        }
    }
    public int getCount() {
        return count;
    }
}
```

tryLock能够避免死锁：

**持有一个锁，然后尝试去获取另一个锁而得不到时，会释放已有的锁，然后自己再去尝试获取所有锁**

ReentrantLock和synchronized的区别：

|              | ReentrantLock                        | synchronized             |
| ------------ | ------------------------------------ | ------------------------ |
| 锁的实现机制 | 基于CAS和AQS                         | 监视器模式               |
| 灵活性       | 支持中断响应、尝试获取锁、超时等待锁 | 获取不到就阻塞，直到等到 |
| 释放         | **显式调用unlock释放锁**             | 代码块执行完成自动释放   |
| 锁类型       | 可选择：公平锁、非公平锁             | 非公平锁                 |
| 条件队列     | 可关联多个条件队列                   | 关联一个条件队列         |
| 可重入性     | 可重入                               | 可重入                   |



#### （3）ReentrantLock的实现原理

是利用了CAS+LockSupport的方法

### 3. 替代wait/notify的显式条件

























































## ps：Java的一些包/库函数等

### 1. String类提供的方法汇总

<a name="string_fun"></a>

#### （1）定义：

```java
String s = "hello world";		// 可以直接定义使用（类似于数组，也可以直接跟数组内容）
String s = new String("hello");	// 可以用new
```

可以用+直接连接两个字符串

```Java
String s1 = "hello";
s1 += "world";
```

#### （2）使用

```java
// 判断字符串是否为空 s.isEmpty()，true——表示为空；false——表示不为空
public boolean isEmpty();

// 获取字符串长度 s.length()，返回字符串长度
public int length();

// 取子字符串 s.substring()，从beginIndex~结尾，获取其子串
public String substring(int beginIndex); 
//取子字符串,从beginIndex~endIndex,获取其子串
public String substring(int beginIndex, int endIndex);

//查找字符，s.indexOf()，返回第一个找到的索引位置，没找到返回-1
public int indexOf(int ch);
//查找子串，返回第一个找到的索引起始位置，没找到返回-1
public int indexOf(String str);
//从后面查找字符
public int lastIndexOf(int ch);
//从后面查找子字符串
public int lastIndexOf(String str);

//判断字符串中是否包含指定的字符序列
public boolean contains(CharSequence s);
//判断字符串是否以给定子字符串开头——判断开头是否匹配
public boolean startsWith(String prefix);
//判断字符串是否以给定子字符串结尾——判断结尾是否匹配
public boolean endsWith(String suffix);
//与其他字符串比较，看内容是否相同——注意是看内容（和包装类的equals类似，重写了Object类方法）
public boolean equals(Object anObject);
//忽略大小写比较是否相同
public boolean equalsIgnoreCase(String anotherString);
//比较字符串大小，就看第一个不一样的字符对应的数值大小，大了那就是大了；小了就是整个小了
public int compareTo(String anotherString);
//忽略大小写比较
public int compareToIgnoreCase(String str);

//所有字符转换为大写字符，返回新字符串，原字符串不变    
public String toUpperCase();
//所有字符转换为小写字符，返回新字符串，原字符串不变
public String toLowerCase();

//字符串连接，返回当前字符串和参数字符串合并结果
public String concat(String str);
//字符串替换，替换单个字符
public String replace(char oldChar, char newChar);
//字符串替换，替换字符序列，返回新字符串，原字符串不变
public String replace(CharSequence target, CharSequence replacement);

//删掉开头和结尾的空格，返回新字符串，原字符串不变
public String trim();
//分隔字符串，返回分割后的子字符串数组
public String[] split(String regex); 
```





[数学包Math](#math)

[数组包Arrays（记得有s！！！）](#arrays)

补充：

| 函数名              | 参数                                            | 返回值 | 作用                         |
| ------------------- | ----------------------------------------------- | ------ | ---------------------------- |
| `Arrays.toString()` | int[] arr<br />（可以是其他基本数据类型的数组） | String | 将数组转化为字符串输出[....] |
|                     |                                                 |        |                              |
|                     |                                                 |        |                              |

[日期包Date]()

[字符串类String（记得是大写的S！！！）]()

1. 求某个十进制数，变成二进制之后包含的1的个数/将某个十进制数变成二进制且按照String格式输出

   ```java
   int num = Integer.bitCount(5);		// num = 2
   System.out.println(Integer.toBinaryString(5));		// 0101
   ```

2. java的输入

   ```java
   // 需要导包，Scanner类
   import java.util.Scanner;
   
   // 使用之前先需要新建一个对象
   Scanner reader = new Scanner(System.in);	// 定义一个scanner对象，system.in表示输入
   int num = reader.nextInt();					// 按照整型的格式去获取输入的值
   reader.close();								// 将scanner关闭，不再接收输入
   ```

   

3. 

## pps：Java的常见异常

### 1. RuntimeException

| 异常                              | 说明                                                         |      |
| --------------------------------- | ------------------------------------------------------------ | ---- |
| `NullPointerException`            | 空指针异常                                                   |      |
| `IllegalStateException`           | 非法状态异常                                                 |      |
| `ClassCastException`              | 非法强制类型转换异常                                         |      |
| `IllegalArgumentException`        | 参数错误                                                     |      |
| `NumberFormatException`           | 数字格式异常                                                 |      |
| `IndexOutOfBoundsException`       | 索引越界异常                                                 |      |
| `ArrayIndexOutOfBoundsException`  | 数组索引越界异常                                             |      |
| `StringIndexOutOfBoundsException` | 字符串索引越界异常                                           |      |
| `ArithmeticException`             | 算数错误                                                     |      |
| `ArrayStoreException`             | 数组存储异常<br />（发生在数组增加一个元素，而该元素类型不匹配上） |      |
| `UnsupportedOperationException`   | 未支持操作异常<br />该操作是不被该类允许的（首见于`Iterator`的`remove`） |      |

