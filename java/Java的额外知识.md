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



# 1. Java的私有构造函数

<a name="1"></a>

我们接触到的构造函数的权限修饰符都是public，比如，
```Java
public class Test {
    public Test() {
    ...
    }
}
```
构造函数的用途是创建一个类的实例。比如，
```Java
Test instance = new Test();
```

既然有public修饰符，那么private修饰符也能够作为构造函数的修饰符——那么构造函数就变成私有构造函数了。
而priavte的作用就是：只能在定义的类内部使用，而无法在其他类中使用，对于构造函数来说，那么**该类就无法在其他类中实例化了，对应的就无法在其他类中使用实例变量和实例方法了**。
存在即合理，那么私有构造函数的作用是什么呢？

## 1. 防止实例化

虽然某个类使用了私有构造函数（那么Java就不会自动生成默认构造函数），一定不能在其他类实例化该类，而**有些类就不能让其他类实例化自己**，比如Java的工具类，不希望被实例化，被用户滥用。而是希望静态访问其类变量和类函数，所以其构造函数就是私有的
```Java
public final class Math {

    /**
     * Don't let anyone instantiate this class.——不让实例化该类
     */
    private Math() {}
}
```

## 2. 单例模式

即类的对象有且只能有一个——单个实例
（单例模式，这是最初级的设计模式之一）
```Java
public class Singleton {

    private static Singleton instance;  // 一个私有的类变量，外部类不能访问
    
    private int x;      // 创建两个私有的实例变量
    private int y;
    
    private Singleton() {
        this.x = 1;
        this.y = 2;
    }

    public static Singleton getInstance() {
        if (instance == null)       // 如果已经调用过了，那么就直接返回
            instance = new Singleton(); // 调用私有方法
        return instance;
    }
}
```
单例模式主要有3个特点，
1. 类的内部有一个类的实例，并且为static类型（类变量，但是在未调用方法之前是未初始化的）
2. 构造函数为私有——外部类不能调用
3. 通过提供一个获取实例的方法，比如getInstance，该方法也为static类型（类方法）
调用的时候，我们可以通过
```Java
Singleton instance = Singleton.getInstance();
```
来得到其实例化，而且每次调用，都是使用同一个实例

为什么要用单例模式呢？ 因为很多时候，我们只需要一个对象就可以了，不希望用户来构造对象，比如线程池，驱动，显示器等。如果把构造函数私有，那么所有程序在调用时，只会有同一个实例变量，不容易带来混乱。

## 3. 被其他构造函数调用

它只是作为中间函数，用来减少重复代码，用来被其他构造函数调用的，而不希望用户直接调用该构造函数（可能该构造函数还不是很完善等等）



# 2. protected关键词的理解

<a name="2"></a>

书上，将关于**protected**很简单：它的访问权限比default大一些，不仅能对包内类可见，还对子类可见。这边做一个详细的解释

1. 当在和protected关键字修饰的成员同一个包内，通过实例化，然后对象名.成员名就能够进行访问（同defalut，同包内均可见）

2. 当不在同一个包内，只有继承了该类（类中包含protected成员）称为其子类，才能使用**子类继承到的protected成员，且只能在子类中使用**，但是**不能访问其父类的成员**，eg: super.xxx(编译不通过)；在子类中建一个父类对象，然后调用protected成员（编译不通过） 

   （原因是，从父类继承，子类就可以**获得了父类方法的地址信息并把这些信息保存到自己的方法区**，这样就可以通过子类对象访问自己的方法区从而间接的访问父类的方法， 继承产生了自己能访问的方法表包括父类的保护区域（实例方法），并**无权限访问父类对象的方法表中保护区域**，就是只能通过自己的对象去访问自己的方法表中保护区域来调用已继承的方法）

场景1：

```java
// 同包内
// 前提clone是java.lang.Object的protected方法——该方法能在同包的java.lang和子类访问
class MyObject{}    // 默认继承了java.lang.Object

public class Test{  // 默认继承了java.lang.Object
    public static void main(String[] args){
        MyObject obj = new MyObject();
        obj.clone();    // compile error
    }
}
```
编译错误的原因：虽然MyObject和Test都是继承自同一个类的子类，但是不能在一个子类中访问另一个子类的protected方法。
$\because$ Object和Test不在同一个类中（不符合条件1），而Test里面调用的是MyObject的clone方法（不符合条件2，如果Test是调用自己继承的方法，那就是可行的）

场景2：

```java
// 同包内：
public class MyObject {
    protected Object clone() throws CloneNotSupportedException{	// 重写了父类的方法
        return super.clone();
    }
}

public class Test{		// 和上面的非继承关系，都是默认继承java.lang.Object
    public static void main(String[] args) throws CloneNotSupportedException{
        MyObject obj = new MyObject();
        obj.clone();		// compile OK
    }
}
```

编译通过的原因：**MyObject重写了clone的方法——原方法被覆盖了**，那么Test和MyObject在同包内（符合条件1），那么可以Test可以通过创建对象然后调用

场景3：

```java
// 异包内
public class MyObject {			// 父类
    protected Object clone() throws CloneNotSupportedException{	// 重写了父类的方法
        return super.clone();
    }
}

public class Test extends MyObject{		// 子类
    public static void main(String[] args) throws CloneNotSupportedException{
        MyObject obj = new MyObject();
        obj.clone();			// compile Error
        Test test = new Test();
        Test.clone();		// compile OK
    }
}
```

两个文件是异包，所以不符合条件1；但是存在继承关系，第一个clone错误的原因是：子类不能在异包的条件去访问父类的clone函数；但是子类可以访问自己继承到的clone方法，所以第二个通过

ps：clone用来复制对象，是分配一个和源对象一样的大小空间，然后在该空间内**创建一个新的对象**。（和单纯的赋值是不一样的，只是创建了一个新的引用对象，然后引用的内容都是同地址的）——**clone是浅拷贝**：创建一个新对象，如果属性是基本类型，那么拷贝基础类型的值；如果是引用类型，那么**拷贝的就是内存地址**

clone是在java.lang.Object中实现的，是**protected**修饰的，在其他类对其进行重写的时候能够将其覆盖为**public**

eg： vector覆盖了clone，所以vector能够直接使用clone

重写clone()方法时，通常都有一行代码**super.clone();**——缺省行为，因为首先要把父类中的成员复制到位，然后才是复制自己的成员。

pps：深拷贝会拷贝所有的属性，包括对象和其所引用的对象一起拷贝（不再是拷贝内存地址，而是拷贝其内容）



# 3. Abstract关键字注意

<a name="3"></a>

分析发现：abstract和private、static、final是不能同时存在的

abstract只能和public、protected一起使用

1. abstract和private

   private是私有的，即只能在类内部使用。而被abstract修饰的方法是需要子类去实现的——所以，修饰为private是无意义的。private类不能被外部；而private不会去修饰类（一个类不能被外部使用，那定义该类还有什么意义呢）

2. abstract和static

   static下的静态方法，静态方法意味着可以直接通过`类名.方法名`使用，而abstract修饰的方法是没有方法体，等待子类去实现的，那么直接调用也没有意义（是个空方法）

3. abstract和final

   final表达的是，该方法不能被重写or继承，而abstract方法就是要被继承和重写才存在实际意义的，所以不符合

4. abstract和public、protected

   后两个都是可见范围修饰符，和abstract没有逻辑上的冲突，所以可以使用

抽象类和接口的区别：

1. 关键字不同，抽象类本质上还是类，所以是`public abstract class XXX`; 接口是`public interface XXX`
2. 一个类只能继承一个抽象类（准确说是只能继承一个类），但是能实现多个接口
3. **抽象类可以有构造方法**（用来继承时调用，因为存在实例变量），但是接口中不能有
4. **抽象类中的抽象方法的访问类型可以是：public、protected**；而接口的抽象方法只能是：public abstract
5. 抽象类中的静态变量的访问类型可以随意，但是接口中的变量只能是`public static final`（默认即这样）
6. 抽象类中可以有实例变量，而接口中的变量只能是静态变量

但是都能有：静态变量、静态方法、实例方法（接口中的实例方法是要用关键字`default`修饰的，可以在子类中重写也可以不）







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

# 5. 异常链

<a name="5"></a>

**定义**：**捕获一个异常后抛出另一个异常**（发生在catch里面），并且把**原始异常信息保存下来**（通过创建新的异常对象，并且传参message和cause），这被称为异常链。

```java
public class Main {
    public static void a() throws MyException {
        try {
            if(true) throw new IOException();		// 直接抛出异常
        }catch (IOException e){			// 抛出的异常被接收
            throw new MyException("IO exception", e);	// 又抛出新的异常，构造异常链
        }		// => 因为捕获异常而产生新的异常
    }
    public static void main(String[] args){
        try {
            a();		// 去调用函数a
        }catch (MyException e){		// 接收了a中抛出的异常
            e.printStackTrace();	// 输出异常栈
        }
    }
}
```

输出：

<img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20201213164038980.png" alt="image-20201213164038980" style="zoom:67%;" />

"IO错误"是产生`MyException`的原因，而`MyException`产生的原因是触发`IOException`

```java
public class Main {
    public static void a() throws MyException {
        try {
            if(true) throw new IOException();
        }catch (IOException e){
            throw new MyException("IO错误", e);
        }
    }
    public static void main(String[] args){
        try {
            a();
        }catch (MyException e){
            Throwable cause = e.getCause();	// 获得是e的cause——异常信息
            cause.printStackTrace();		// 索引打印的是IOException的异常栈
        }
    }
}
```

<img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20201213165100701.png" alt="image-20201213165100701" style="zoom:67%;" />

多个异常链：

```java
public class Main {
    public static void b()throws ClassNotFoundException{
        try{
            if(true) throw new MyException();	// 抛出异常
        }catch (MyException e){					// 捕获异常
            // cause = MyException
            throw new ClassNotFoundException("b方法产生的异常", e);//重新抛出新的异常
        }
    }
    public static void a() throws IOException {
        try {
            b();
        }catch (ClassNotFoundException e){		// 捕获b产生的新异常
            // cause = IOException
            throw new IOException("a方法产生的异常", e);	// 重新抛出新异常
        }
    }
    public static void main(String[] args){
        try {
            a();
        }catch (IOException e){		// 捕获异常
            e.printStackTrace();	// 打印出异常栈
        }
    }
}
```

<img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20201213165615610.png" alt="image-20201213165615610" style="zoom:67%;" />

先弹出：最上层的异常：由a产生的IO异常

接着IO异常是由b产生的类异常

而b产生的类异常是最底层的：MyException产生

——由此构成了一条链

# 6. equals和hashCode

<a name="6"></a>

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

  ` @Override public boolean equals(Object obj) { throw new AssertionError();} `

$\therefore$总结： 默认情况下是从超类Object继承而来的，equals方法与`==`是完全等价的，比较的都是对象的内存地址，但我们可以重写equals方法，使其按照我们的需求的方式进行比较，如Integer类重写了equals方法，使其比较的是里面的值，而不是内存地址。 

### (2) equals重写的规则

通用规则（和数学规则很像）

1. **自反性**： 对于任何非空引用值 x，x.equals(x) 都应返回 true（自己和自己比应该一样）
2. **对称性**：对于任何非空引用值 x 和 y，当且仅当 y.equals(x) 返回 true 时，x.equals(y) 才应返回 true。（x和y相互比较应该结果一样） 
3. **传递性**：对于任何非null的引用值x、y与z，如果y.equals(x)返回true，y.equals(z)返回true，那么x.equals(z)也应返回true。 （x和y一样，y和z一样，那么x和z应该一样）
4. **一致性**： 对于任何非null的引用值x与y，假设对象上equals比较中的信息没有被修改，则多次调用x.equals(y)始终返回true或者始终返回false。（结果保持稳定性）
5.  非空性：对于任何非空引用值x，x.equal(null)应返回false。

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

可以发现重写equal还是有很多坑的

有人总结出的重写技巧：

- 首先，**使用==操作符检查“参数是否为这个对象的引用”**：如果是对象本身，则直接返回，拦截了对本身调用的情况，算是一种性能优化；

- 判空操作， 如果为null,返回false；

- 接着再**使用instanceof操作符检查“参数是否是正确的类型”**：如果不是，就返回false，正如对称性和传递性举例子中说得，不要想着兼容别的类型，很容易出错。在实践中检查的类型多半是equals所在类的类型，或者是该类实现的接口的类型，比如Set、List、Map这些集合接口。

  - 如果equals的语义在每个子类中有所改变，就使用getClass检测——严格判断类型是否匹配
  - 如果所有的子类都拥有统一的语义，就使用instanceof检测 ：（eg：父类car与子类bigCar混合比，我们统一了批次相同即相等）

- 在类型一致的情况下，**把参数转化为正确的类型**

- **对于该类中的“关键域”，检查参数中的域是否与对象中的对应域相等**：

  - 基本类型的域就用`==`比较，

  - Float用Float.compare方法，Double域用Double.compare方法，

  - 对象域，一般递归调用它们的equals方法比较。

    对象域判断的最简单的版本：加上判空检查和对自身引用的检查，一般会写成这样：`(field == o.field || (field != null && field.equals(o.field)))`

- **编写完成后思考是否满足上面提到的对称性，传递性，一致性等等**

下面是String类中对equals的重写：

```java
 public boolean equals(Object anObject) {
	if (this == anObject) {	// 看是否是引用同一个对象，如果是就避免了后面的比较，对应条1
		return true;
	}
	if (anObject instanceof String) {	// 判断参数类型是否为String类型，对应条2
		String anotherString = (String)anObject;	// 类型转换，对应条3
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

hashCode在Object类中声明，默认是返回该对象的内存地址，好的哈希方法应该将实例均匀分布在所有散列值上

规定了：

- **equals返回true（表示同一个对象），那么hashCode也必须返回一样**；
- 如果equal返回false，那么hashCode尽量不一样

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

1. 31可以被JVM自动优化，通过移位操作完成，`31*i ==(i<<5)-i`

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



# 8. fail-fast 

<a name="8"></a>

### （1）fail-fast解释

在Java的注释经常可以碰到`fail-fast`，这是一种机制

wiki的解释：在系统设计中，快速失效系统一种可以立即报告任何可能表明故障的情况的系统。常用来停止当前正常的操作，而不是试图继续执行可能存在缺陷的故障，让系统自行触发异常而返回

——即**系统设计的时候优先考虑异常情况，一旦探测到异常（但是当前还未触发异常），直接停止操作并上报**（有点像，写leetcode的时候处理特殊情况，因为前面不处理，继续执行下去可能会发生越界等异常）

通常说的**`fail-fast`就是Java的Collection中的错误检测机制**

即多个线程同时对集合的结构进行修改，就会触发fail-fast机制，这时候就会抛出`ConcurrentModificationException`（并发修改异常）

这边存在一个陷阱：设计集合的fail-fast的初心就是为了防止多线程同时去修改集合的结构。但是由于实现机制导致：**在使用迭代器进行遍历操作的时候，如果使用外部的remove/add等操作，那么就会触发CME**——具体原因看源码（主要是由于modCount和exceptModCount不匹配引起的），所以并没有出现并发，但是引起了并发异常——解决方法：调用迭代器提供的remove方法（ListIterator还提供add等方法）

ps：解释一下这边的并发操作：一个或者多个线程在遍历操作该集合，操作可以是只读、也可以是读写都存在。而抛出的异常CME，是并发修改，即在一个或多个线程迭代过程中，存在2个及以上的线程去修改该集合的结构

### （2）对应的：fail-safe

java实现了`fail-safe`机制的集合类，这些集合在遍历时，不是直接在其本身上操作，而是在副本上面操作

`java.util.concurrent`包下的容器都是fail-safe的，可以在多线程下并发使用，并发修改。同时也可以在foreach中进行add/remove ——但是由于是副本，那么之后对内容的修改，迭代器无法访问到

eg：ArrayList对应的线程安全的类是：`CopyOnWriteArrayList`

（博客后面还有关于`copyonwrite`的知识，后面学了并发知识之后再来补充）

参考内容：

https://juejin.cn/post/6844903824709287943

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



# 10. CAS原理分析

why有CAS呢？

因为单纯用synchronized，是重量级锁，很大程度的影响性能；而volatile不能保证原子性。而Lock一定需要和unlock配套使用——CAS+AQS的底层实现。

ps：在JDK1.6之后做了优化，如果并发量不大，那么相较于Lock更优，并且会自动释放锁

所以可以使用CAS的硬件支持的原子变量来实现比较简单的并发操作，例如i++，所以可以在原子类中的实现过程中看到CAS的影子，所以是CAS是核心，需要仔细分析（也是Java面试高频考点）

pps：LongAdder 原子类在 JDK1.8 中新增的类， 也是在 java.util.concurrent.atomic 并发包下。LongAdder 适合于高并发场景下，特别是写大于读的场景，相较于 AtomicInteger、AtomicLong 性能更好，代价是消耗更多的空间，以空间换时间。

### （1）背景知识

背景介绍：在Java的原子类中，接触到了CAS，它是原子类实现的核心。书中说，CAS是底层硬件的支持，可以将CAS函数看成单个指令。

那么还是想了解一下：底层如何实现CAS的，why是硬件支持的

函数介绍：CAS 全称是 compare and swap，字面意思就是比较并且交换（值）。是一种在多线程下的实现同步功能的机制，即能够做到并发修改。

实现逻辑：CAS 的实现逻辑是将内存位置处的数值与预期数值expect想比较，若相等，则将内存位置处的值替换为新值newValue。若不相等，则不做任何操作。如果修改成功返回——true；未修改返回——false。整个流程是满足原子性的

在 Java 中，Java 并没有直接实现 CAS，**CAS 相关的实现是通过 C++ 内联汇编的形式实现的**。Java 代码需通过 JNI 才能调用。

背景知识：Intel 处理器可以保证单次访问内存对齐的指令以原子的方式执行。——即一次读/写指令都是原子性的。但是如果两次访存指令，比如说`i++/ i += 1`，都是涉及到了一次读指令、一次写指令，那么会存在多线程的并发修改，导致结果不符合预期。

而底层的硬件保证：在多处理器环境下，**LOCK# 信号可以确保处理器独占使用某些共享内存**。lock 可以被添加在下面的指令前：ADD, ADC, AND, BTC, BTR, BTS, CMPXCHG, CMPXCH8B, CMPXCHG16B, DEC, INC, NEG, NOT, OR, SBB, SUB, XOR, XADD, and XCHG。通过**在 inc 指令前添加 lock 前缀，即可让该指令具备原子性**。多个核心同时执行同一条 inc 指令时，**会以串行的方式进行**，也就避免了上面所说的那种情况。

那么底层的硬件如何保证lock下的原子性呢：

有两种方式保证处理器的某个核心独占某片内存区域：

- 锁定总线，使用Lock信号，让某个核心独占使用总线，但这样代价太大——锁定后，其他核心就不能访问内存了

- 锁定缓存，某处内存数据被缓存在处理器缓存中。处理器发出的 LOCK# 信号不会锁定总线，而是**锁定缓存行对应的内存区域**。其他处理器在这片内存区域锁定期间，无法对这片内存区域进行相关操作。具体来说，就是某个处理器对缓存中的共享变量进行了操作，其他处理器会有个**嗅探机制，将其他处理器的该共享变量的缓存失效，待其他线程读取时会重新从主内存中读取最新的数据**，它是基于 MESI 缓存一致性协议来实现的。

  ——代价变小，只是无法访问对应的内存区域，而其他内存区域还是可以访问的

  （如果处理器不支持“锁定缓存”，那么只能使用总线锁定；）

### （2）源码分析

原子类 AtomicInteger 中的 compareAndSet 方法进行分析

```java
// AtomicInteger部分相关源码

// 静态/实例变量，代码块等
private static final Unsafe unsafe = Unsafe.getUnsafe();	// 创建对象——像是一个单例模式
private static final long valueOffset;	// valueOffset，是记录value的偏移量的

/* 这个是发生在类加载过程中的，是最先初始化的，是调用了unsafe实例对象的objectFieldOffset实例方法，获得AtomicInteger类文件中value的偏移量 */
static {		
    try {
        // 保存变量 value 在类对象unsafe中的偏移
        valueOffset = unsafe.objectFieldOffset
            (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}

private volatile int value;		// 保证内存可见性，实际的值是存放在value中

public final class Unsafe {
    // compareAndSwapInt 是 native 类型的方法
    public final native boolean compareAndSwapInt(Object o, long offset,
                                                  int expected,int x);
}

public final boolean compareAndSet(int expect, int update) {
    /*
   	 * compareAndSet 实际上只是一个壳子，传递的是value的偏移量——可以间接获取value的值；期待的值；需要更新的值
     */
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}

public final int incrementAndGet() {		// 等同于++i
    for(;;) {		// 死循环，直到修改成功就跳出，否则一直尝试++
        int current = get();		// 原子操作，去获取当前的值
        int next = current + 1;		// 新的值
        if(compareAndSet(current, next))		// 期待的值为原来的值，如果是原来的值（未发生修改），那么修改为新的值
        	return next;		// 如果CAS返回true，表示修改成功，那么返回当前的新值
    }
}
```

可见：incrementAndGet的原子性的核心就是compareAndSet，而其实现本质上是native方法`compareAndSwapInt`，下面分析unsafe及CAS的实现：

```c++
// unsafe.cpp
/*
 * 这个看起来好像不像一个函数，不过不用担心，不是重点。UNSAFE_ENTRY 和 UNSAFE_END 都是宏，
 * 在预编译期间会被替换成真正的代码。下面的 jboolean、jlong 和 jint 等是一些类型定义（typedef）：
 * 
 * jni.h
 *     typedef unsigned char   jboolean;
 *     typedef unsigned short  jchar;
 *     typedef short           jshort;
 *     typedef float           jfloat;
 *     typedef double          jdouble;
 * 
 * jni_md.h
 *     typedef int jint;
 *     #ifdef _LP64 // 64-bit
 *     typedef long jlong;
 *     #else
 *     typedef long long jlong;
 *     #endif
 *     typedef signed char jbyte;
 */
UNSAFE_ENTRY(jboolean, Unsafe_CompareAndSwapInt(JNIEnv *env, jobject unsafe, jobject obj, jlong offset, jint e, jint x))
  UnsafeWrapper("Unsafe_CompareAndSwapInt");
  oop p = JNIHandles::resolve(obj);		// p是取出的对象，就是前面的this
  // 根据偏移量，计算 value 的地址。这里的 offset 就是 AtomaicInteger 中的 valueOffset
  jint* addr = (jint *) index_oop_from_field_offset_long(p, offset);
  // 调用 Atomic 中的函数 cmpxchg，该函数声明于 Atomic.hpp 中
  return (jint)(Atomic::cmpxchg(x, addr, e)) == e;
UNSAFE_END

// atomic.cpp
unsigned Atomic::cmpxchg(unsigned int exchange_value,
                         volatile unsigned int* dest, unsigned int compare_value) {
  assert(sizeof(unsigned int) == sizeof(jint), "more work to do");
  /*
   * 根据操作系统类型调用不同平台下的重载函数，这个在预编译期间编译器会决定调用哪个平台下的重载
   * 函数。相关的预编译逻辑如下：
   * 
   * atomic.inline.hpp：
   *    #include "runtime/atomic.hpp"
   *    
   *    // Linux
   *    #ifdef TARGET_OS_ARCH_linux_x86
   *    # include "atomic_linux_x86.inline.hpp"
   *    #endif
   *   
   *    // 省略部分代码
   *    
   *    // Windows
   *    #ifdef TARGET_OS_ARCH_windows_x86
   *    # include "atomic_windows_x86.inline.hpp"
   *    #endif
   *    
   *    // BSD
   *    #ifdef TARGET_OS_ARCH_bsd_x86
   *    # include "atomic_bsd_x86.inline.hpp"
   *    #endif
   * 
   * 接下来分析 atomic_windows_x86.inline.hpp 中的 cmpxchg 函数实现
   */
  return (unsigned int)Atomic::cmpxchg((jint)exchange_value, (volatile jint*)dest,
                                       (jint)compare_value);
}
```

```c++
// atomic_windows_x86.inline.hpp
#define LOCK_IF_MP(mp) __asm cmp mp, 0  \   // 如果mp == 0，那么是单核cpu，那么会跳到L0开始的地方，即cmpxchg处
                       __asm je L0      \	
                       __asm _emit 0xF0 \	// 如果mp!=0,不跳转，0XF0是lock前缀的机器码，没有使用lock而是直接用字节码
                       __asm L0:
              
inline jint Atomic::cmpxchg (jint exchange_value, volatile jint* dest, jint compare_value) {
  // 判断是否多核cpu，如果是多核cpu，那么就给总线加锁，那么其他cpu不能通过总线访问内存（性能会受到影响），如果是单核系统，那么不需要添加
  int mp = os::is_MP();			
  __asm {
    mov edx, dest		// value地址的偏移量
    mov ecx, exchange_value	// newValue的值
    mov eax, compare_value	// expect的值
    LOCK_IF_MP(mp)
    // dword: 全称是 double word,word = 2 byte，dword = 4 byte = 32 bit,双字节
    // ptr: 全称是 pointer，与前面的 dword 连起来使用，表明访问的内存单元是一个双字单元
    cmpxchg dword ptr [edx], ecx	// 比较并交换，如果eax == [edx]，那么将ecx ->[edx]；如果不相等，那么就不赋值
  }
}
```

——可以发现，关键的指令就是`cmpxchg dword ptr [edx], ecx`，一个CPU指令能够执行完成，所以能保证原子性。

确保对内存的读-改-写操作原子执行

ps：

Intel手册对lock前缀的说明如下：

1. 确保对内存的读-改-写操作原子执行。在Pentium及Pentium之前的处理器中，带有lock前缀的指令在执行期间会锁住总线，使得其他处理器暂时无法通过总线访问内存。很显然，这会带来昂贵的开销。

   从Pentium 4，Intel Xeon及P6处理器开始，intel在原有总线锁的基础上做了一个很有意义的优化：如果要访问的内存区域（area of memory）在lock前缀指令执行期间已经在处理器内部的缓存中被锁定（即包含该内存区域的缓存行当前处于独占或以修改状态），并且该内存区域被完全包含在单个缓存行（cache line）中，那么处理器将直接执行该指令。由于在指令执行期间该缓存行会一直被锁定，其它处理器无法读/写该指令要访问的内存区域，因此能保证指令执行的原子性。这个操作过程叫做缓存锁定（cache locking），缓存锁定将大大降低lock前缀指令的执行开销，但是当多处理器之间的竞争程度很高或者指令访问的内存地址未对齐时，仍然会锁住总线。

2. 禁止该指令与之前和之后的读和写指令重排序。

3. 把写缓冲区中的所有数据刷新到内存中。

参考文章：

https://segmentfault.com/a/1190000014858404

https://juejin.cn/post/6844903558937051144

https://zhuanlan.zhihu.com/p/94762520



# 11. AQS的底层实现

<a name="11"></a>

背景引入：在学习ReentrantLock的时候，了解到底层实现是基于CAS+AQS，而CAS的底层实现上面已经了解了，而AQS是很多类的实现，那么AQS的底层实现还需要分析（因为也是面试的高频考点）。下面将从ReentrantLock的顶层开始分析到底层AQS的实现

AQS：全称为AbstractQueuedSynchronizer：抽象队列同步器。AQS是提供了一种原子式同步状态、阻塞、唤醒线程功能及队列模型的简单框架。

和ReentrantLock（最常用的显式锁）相对应的是 synchronized（隐式锁，依赖的是监视器模式）

### （1）AQS的原理

- 如果被请求的共享资源空闲，那么就将当前请求资源的线程设置为有效的工作线程，将共享资源设置为锁定状态；
- 如果共享资源被占用，就需要一定的阻塞等待唤醒机制来保证锁分配。这个机制主要用的是**CLH队列的变体实现的**，将暂时获取不到锁的线程加入到队列中。

CLH：Craig、Landin and Hagersten队列，是单向链表，AQS中的队列是CLH变体的虚拟双向队列（FIFO）。

AQS是通过将每条请求共享资源的**线程封装成一个节点**来实现锁的分配。AQS使用一个**Volatile**的int类型的**成员变量**来表示同步状态，通过内置的**FIFO队列**来完成资源获取的排队工作，通过**CAS**完成对State值的修改。

下面介绍一下队列中每个节点的内容：

| 方法和属性值 | 含义                                                  |
| ------------ | ----------------------------------------------------- |
| waitStatus   | 等待状态标志位，表示当前节点在队列中的状态            |
| thread       | 表示处于该节点的线程                                  |
| prev         | 前驱指针                                              |
| next         | 后继指针                                              |
| predecessor  | 返回前驱节点，没有的话抛出npe                         |
| nextWaiter   | 指向下一个处于CONDITION状态的节点（这个指针不多介绍） |

ps：变量waitStatus：共有4种取值

- 0：初始化默认的值
- CANCELLED：1，线程获取锁的请求已经取消了，在同步队列中等待的线程等待超时或被中断
- SIGNAL：-1，表示线程已经准备好了，就等资源释放了
- CONDITION：-2，表示节点在等待队列中，节点线程等待唤醒（与Condition相关，该标识的结点处于等待队列中，结点的线程等待在Condition上，当其他线程调用了Condition的signal()方法后，CONDITION状态的结点将从等待队列转移到同步队列中）
- PROPAGATE：-3，当前线程处在SHARED情况下，该字段才会使用，与共享模式相关，在共享模式中，该状态标识结点的线程处于可运行状态。

参考文章：

https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html

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