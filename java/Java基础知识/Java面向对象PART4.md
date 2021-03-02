# Java面向对象PART4

# ——异常

在没有异常的情况下，唯一退出的机制就是return，判断是否出现异常就通过返回值（类似C语言中的，对不同情况进行判断，根据不同情况设定返回值，eg：`memory_alloc()`）。然后调用者根据不同的返回值做出不同的判断。那么每层都需要对调用的程序的不同返回值进行判断和处理。**程序正常逻辑和异常逻辑混在一起，代码难以阅读和维护**。并且很多时候都会忽略某些异常的处理，降低了程序的可靠性。

异常机制下，**正常逻辑和异常逻辑可以分离，异常情况可以集中处理，并且异常可以自动上传，不需要每一层都处理，并且异常不会被忽略**。处理异常的代码量减少（不需要对不同返回值进行判断处理，并且异常会封装一些方法，直接调用复用即可），且代码的可读性、可靠性、可维护性提高

## 1. 异常的概念

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

## 2. 异常类

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

<img src="C:/Users/surface/Desktop/计算机知识整理/java/pic/image-20201213120034202.png" alt="image-20201213120034202" style="zoom:67%;" />

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

## 3. 异常的处理

### （1）catch匹配

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

### （2）重新抛出异常

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

### （3）finally

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

2. **catch不是必须出现的**，eg：**`try/finally`也是合法的**，那么异常在该层不被捕获，而是向上传递，但是**finally执行顺序还是：异常发生之后，被抛给上层之前去执行**。

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


### （4）try-with-resources

前面说到，finally多用在关闭资源操作中，Java7支持自动关闭资源，被称为`try-with-resources`

语法如下：

```java
public static void test() throws Exception{
    try(AutoCloseable r = new FileInputStream("hello world"))	// 创建资源
    {
        	// 使用资源
    }
}

// >>等价于
public static void useResource() throws Exception {
    AutoCloseable r = new FileInputStream("hello"); //创建资源
    try {
        //使用资源
    } finally {
        r.close();
    }
}
```

**针对的实现了`java.lang.AutoCloseable`接口的对象**——AutoCloseable对象是核心

当执行完try之后会默认调用`close()`方法关闭资源

资源可以定义多个，**以分号间隔**（Java9之前资源声明和初始化必须要在try中，但是Java9之后允许可以在try之前，但是资源必须是final或形式上的final，即只被赋值一次）

### （5）throws关键字：声明异常

注意和throw区分：throw是抛出异常，是一个语句；throws是方法中的声明

`throws`用来**声明一个方法**可能抛出的异常

```java
public static void main(String[] args) throws NullPointerException, NumberFormatException
{
    
}
```

分析：

1. 一个方法可以声明多个异常，**以逗号分隔**。

   该声明的含义是：方法内可能抛出这些异常，且**对这些异常没有处理/没有处理完**，调用者必须进行处理——所以，调用者必须要catch或者继续throws，而如果方法内已经catch了，那么可以不声明

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

2. **对于未受检异常，不要求用throws进行声明的；对于受检异常，一定要用throws进行声明**，eg：IOException——没有声明，就不能抛出

   受检异常，不可以不声明就抛出，可以声明而不抛出。这个情况出现在父类子类中：父类不需要抛出某个异常，但是子类需要抛出，而**子类不能抛出父类没有声明的异常——即子类抛出的异常不能比父类范围大**

未受检异常和受检异常：

未受检异常：通常表示的是编程的逻辑错误，编程的时候应该尽量避免这些错误——所以应该避免而不是处理

受检异常：通常是程序本身没有问题，由于IO、网络、数据库等出现不可预测的问题而导致的异常——无法避免，所以要选择处理

——其实，实际上没有很明显的区分，不论是未受检异常or受检异常，是否有throws声明，所有异常都应该得到处理

**非运行时异常就是受检异常**，是需要throws的；运行时异常是非受检异常，不需要throws

## 4. 异常的使用

异常应该仅用于异常的情况即：

- **异常不能替代正常的条件判断**

  eg：在处理数组是，应该先判断索引值是否有效来进行循环，而不是靠抛出异常来结束循环

  ​		对于引用对象，在使用前应该先判断是否为null，而不是靠异常来判断

- **一旦出现真正的异常，应该抛出异常，而不是返回特殊值**

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