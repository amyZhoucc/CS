# chap1 介绍

kotlin有 if表达式的简化写法：

**if会默认有一个返回值，默认是最后一个表达式的值**

——注意**如果要写一行，if和else都要有**；如果是普通的if格式，那么else不是必须的

```kotlin
if(x > y) x else y			 
```

Kotlin交互式shell，又称为REPL，能够快速地在主函数之外尝试代码片段。点击边上的按钮，能够看到运行结果

<img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20210801133800928.png" alt="image-20210801133800928" style="zoom:67%;" />

基本语法：（和Java不同的地方）

1. 使用fun来定义函数

2. 每一个应用程序都需要一个名为main的主函数

   ```kotlin
   fun main(args: Array<String>){
       ...
   }
   ```

3. String类型是表示一串字符

4. **使用var来定义一个可变变量。val来定义一个固定值。**

5. 条件分支语句放在if和else之间。else语句是非必要的。**使用if表达式来返回一个值。在这种情况下，else语句是必需的。**

# chap2 基本数据结构

Byte：8位，-128~127

Short：16位

Int：32位

Long：64位

**Kotlin不支持八进制数，只支持二进制0b，十六进制0x，十进制**

Float：32位

Double：64位

显式定义变量类型：

```kotlin
var x : Short				// 声明变量（如果是val的话，只能在后面被赋值一次）
x = 6				// 那么就是short类型的数据

var y: Short = 7		// 可以声明的时候赋值
```

和Java的区别：

1. Kotlin编译器只能成功编译所赋值与变量类型**兼容**的情况。当所赋变量值超出变量类型所能存储的数据大小或是**变量类型与变量值不兼容时，该代码将不能编译。**

   ——基本数据类型之间不能互相赋值，也不能通过强制类型转换，只能通过自带的`xxx.toXxx()`方法，可以实现数字之间的类型转换而如果转换过程中超出了容纳范围，会舍弃超出部分。

2. Java中的基本数据类型是原生的，所以这些变量存储的是数据本身。而在kotlin中，基本数据类型也还是对象，所以变量存储的是数据对象的引用
3. Java中Char本质上是数字，**Char在Kotlin中是字符而不是数字**

数组：

```kotlin
val arr = arrayOf(1, 2, 3)
arr[0]
arr.size()

val arr1: Array<Int> = arrayOf(1,2,3)
```

理解：

1. 同Java，数组只能存放存放某一特定类型的变量，要么让编译器推测出固定的变量类型，要么使用Array<Type>来显式指定
2. 对于val是不可变的变量，var是可变变量，而如果是数组格式，val只能指向同一个数组，但是能够**修改数组里面的内容**

随机数：

```kotlin
Math.random()			// [0,1)的随机数
```

String模板：

```kotlin
val x = 2
val s = "x is $x"
val ss = "x is ${x.toInt()}"		// 引用对象的属性或是调用其函数，需要用花括号，甚至还能在里面写复杂的表达式
```

基本语法：

1. 创建变量时，编译器需要知道**变量名、变量类型以及该变量是否可被重用**

   ——变量名a/arr，变量类型：编译器可以推测/显式给定，val/var

2. 变量存储了**对象的引用**

3. 一个对象有状态和行为，其行为可以从它的函数中得知

4. Kotlin有以下几种基本类型：Byte、Short、Int、Long、Float、Double、Boolean、Char和String

5. 换为另一个数值类型，然而如果新类型较小时，该操作将会导致精度损失

6. 使用arrayOf函数来创建数组，可以通过数组下标来访问数组元素，例如myArray[0]。使用myArray.size来获取数组大小

7. 可以更新val定义的数组中的元素

8. String模板——`$/${}`提供了从String内部引用或是计算表达式的快速简便的方法。

9. 判断某个元素是否存在于数组中，`item in array`——返回true/false

# chap3 方法

无返回值：可以省略不写，也可以写**Unit为返回类型**。将函数的返回值类型声明为Unit意味着该函数不返回任何值。

如果函数只有单个表达式，那么可以继续简写：

```kotlin
fun foo(args: Int) = if(args == 1) true else false
```

注意：

1. kotlin一个函数只能有一个返回值，如果要返回多个相同类型的值，那么可以返回一个Int数组（同Java）
2. **kotlin的形参默认是val声明的**，所以不能再赋值

while循环：

```kotlin
var i = 0
while(i < 10){
    print("${i++} ")
}
```

for循环：

不能按照经典的类似于Java、c的格式

```kotlin
var x = 0
for(x in 1..10){			// x会自动遍历[1, 10]
	....
}

for(x in 1 util 100){		// x会自动遍历[1, 100)，util不包括最后一个值
	...
}

for(x in 10 downTo 1){		// x会自动遍历[10, 1]，逆序
    ...
}

for(x in 1..10 step 2){			// step步长
    ...
}
```

也可以遍历数组：

```kotlin
val arr: Array<Int> = arrayOf(1,2,3,4,5)
for(item in arr){			
    print("$item ")
}

for (index in arr.indices){			// 遍历下标
    print("${arr[index]} ")
}

for ((index, item) in arr.withIndex()){		// 内容 + 下标
    print("${index} is ${item}")
}
```

基本语法：

1. 如果方法没有返回值，可以不写，也可以传入Unit
2. 对于已知循环次数的循环操作，优先选择for，而不是while
3. readLine()：可以读取标准的一行，以回车作为结尾，返回值是String类型
4. 如果输入流被重定向到一个文件，并且readLine()读到了文件的结尾，那么readLine()将会返回null值——可以用null来判断是否读到了文档尾

# chap4 类

类的概念已经明确，只需要知道语法。

```kotlin
class Dog(					// 构造方法+类的实例变量定义
	val name: String,
	val sex: Boolean,
	var weight: Double){
    ...
}

// 复杂写法
class Dog(				// 构造方法，但是没有定义类的实例
    name: String,		
    sex: Boolean,
    weight: Double
) {
    private val name = name			// 变量命名可以重复（构造方法的形参本质上还是局部变量）
    private val sex = sex
    private var weight = weight/2.2
```

理解：

1. 不同于Java，**kotlin支持直接在类名后面加构造方法——被称为“主构造方法”**，构造方法里面的也是实例变量，如果要创建类对象的话，必须要传递的对应的参数，类型、个数都一样

2. 实例变量的定义：一定要加上val/var——如果不加，那么实例方法还是要在类里面定义的

3. 如果不需要构造方法，则可以不要带，那么编译器会给一个默认空构造方法

   ```kotlin
   val dog1 = Dog("xiaobai", true, 20)		// 如上的类构造
   val dog = Dog()				// 如果类名后面没有带参数——本质上调用的是默认构造方法
   ```

4. 系统为创建的变量创建空间，构造方法负责初始化内容

如果要在创建的时候进行更复杂的操作，可以使用若干个**init代码块**，在**构造方法执行完成后紧接着执行**，**init代码块之间的顺序按照从上到下，且和实例属性的初始化同时**——即如果代码块之间有实例属性的定义和初始化就按照顺序执行

初始化：

1. 实例变量在使用之前必须要初始化

2. 实例变量不需要在创建对象的时候初始化：**`private latinit var name: String`**——编译器会认为你之后会初始化，但是如果发现了情况1，还是会报错的——这是一个保护机制

   注意：

   - 对应的基本数据类型（虽然是类类型），**必须要初始化**——JVM的底层决定的
   - **延迟初始化的变量，只能是var**

getter和setter：

```kotlin
private val weight: Double				// 在主构造方法中有传递参数weight
	get() = field / 2.2

private var weight: Double = 0.0		// 注意区别，一定要加field，不然要报错
	get() = weight / 2.2

// 只有var才有setter方法
private var weight = 0.0
	set(value) {
    	if (value > 0) field = value	// 通过field来设置值，否则是不起效的
    }
```

### 难点：kotlin的类属性

类属性的关键字：

1. `@Transient`：该属性不参与序列化
2. `@Volatile`：多线程避免重排序
3. `@JvmField`：将kotlin类的属性直接暴露——属性就变成`public final String name`类似

对于kotlin的属性转换成Java语言之后变成：

```kotlin
// kotlin语言
class Lion(val name: String, var color: String){
    
}

// Java语言
class Lion{
    private final String name;		// 私有不可变的变量
    public final String getName(){
        return name;
    }
}


// kotlin语言
val lion = Lion("lion", "yellow")
println(lion.name)			// 本质上还是去调用了getName方法
```

private类

1. private类，会默认不生成getter和setter方法——因为不需要

backing field：

**kotlin只会为符合要求的类属性加上JVM意义上的类属性**

```kotlin
class Lion(a: String) {
    val aa: String = a		// 语法报错——aa属性没有backing field，所以不能赋值
        get() = aa		// 这个出现了循环调用
}

class Lion(a: String){
    var aa: String = a		// 不报错——因为aa是可变的
    	get() = aa			// 出现了循环调用
}
```

——只有添加`backing field`的对象，才会在JVM上有类属性

1. 使用默认的getter/setter的属性，默认有filed字段

2. 自定义的getter/setter的属性中，使用了field字段

   ——filed是关键字

```kotlin
class Lion(a: Double) {
    val aa = a
        get() = field / 2.2		// 通过对field来赋值
}
```

field的使用：

```kotlin
class Lion(a: Double) {
    private var _size: Int = 0		// 只有内部才能改变——就是backing field
    val size get() = _size		// 外部访问
}
```

# chap5 继承

父类写的是公共的属性和方法；

子类可以写子类特有的属性和方法，且能重写父类的方法。

父类下面还可以有子类，子类下面还可以有子子类。。。——从而构成层级结构

`X is a Y`：X可以做Y任何能做的事情，X甚至可以做的更多，且是**单向的**

kotlin的类为了安全默认是不可以继承的，**如果要继承，父类需要添加关键字open，且需要覆盖的方法也需要声明为open**，因为方法也是默认不可继承的

```kotlin
open class Animal{
    ....
}
```

子类继承：

```kotlin
class Lion: Animal(){			// 本质上是去调用父类的构造函数
    
}
```

理解：

1. **继承的时候，强制调用父类的构造函数**——如果父类在定义的时候有显式的给出主构造函数，那么需要调用主构造函数——传递对应的参数；如果没有编译器会自动给一个空构造函数

对于实例变量：

（Java不同的），如果父类的有些属性是val类型，且在类中是写死的（不能通过构造函数赋值），那么如果需要重新赋值，必须要**将属性添加`open`修饰符，且子类中`override val name`**

eg：

```kotlin
open class Animal{
	open val name = ""			// open
    open val color = ""
}

// 添加override修饰符
class Lion(override val name: String, override val color: String): Animal(){
    ....
}
```

而如果定义的变量是var类型的，那么不需要override，重新赋值即可

```kotlin
open class Animal{
	var name = ""
	var color = ""
}

class Lion: Animal(){
    init{
        name = "Lion"			// 直接赋值
        color = "yellow"
    }
    ....
}

// 或者：——注意主构造函数里面不能定义重名的变量，即使是var，也需要用override
class Lion(name: String, color: String): Animal(){
    init{
        this.name = name			// 用this进行区分
        this.color = color
    }
}
```

注意：

1. 如果在主构造方法中，添加了val/var，那么就意味着是该类的属性，那么不能和父类重名，如果需要重名，需要添加override
2. 如果想重新赋值，只能写在init代码块中，如果变量和属性重名，可以用this进行区分
3. 子类中，如果需要对父类的某个属性的getter和setter进行重写，可以覆盖父类的属性，且添加新的getter和setter
4. **子类中，可以用var覆盖父类的val，但是不能用val覆盖var**
5. 如果要覆盖，那么属性的类型是不能变的

对于方法，同上。

但是要注意，如果子类重写了父类的方法，而子类不想让子类的子类继续重写，就需要添加**final关键字**：

```kotlin
final override fun eat(){
    println("last can override this fun")
}
```

why：val可以被重写为var的变量？而var的变量不能重写为val？

因为：var覆盖时，需要在原来的getter的基础上添加一个setter，并不破坏规则；

​			规则来说：**定义类层次结构时，你需要确保可以对子类执行与父类相同的操作**，而父类的var是希望该变量值可以被修改，而子类的val是希望只能被修改一次，所以破坏了父类和子类的公共协议

基本语法：

1. 在子类中通过**添加override前缀覆盖属性和方法**。覆盖属性时，其类型必须与父类属性的类型兼容。覆盖方法时，其参数列表必须保持原样，**其返回类型必须与父类的兼容**——返回相同的类型、其子类型
2. 被覆盖的方法和属性保持open状态，直到它们被声明为final

# chap6 抽象类、接口

类声明为抽象类，从而避免实例化。

父类被标记为抽象类，那么必定要被继承的，所以不需要前面加open——即默认是open的

```kotlin
abstract class Animal{
	abstract val name: String
    abstract val color: String
}
```

类可以包含抽象和非抽象的属性和方法，甚至可以没有抽象的属性和方法，但是如果存在抽象的属性和方法，那么该类就一定要被定义成抽象类。

**抽象属性、抽象方法不能初始化，也没有getter、setter方法。**

**抽象的属性、方法、类均不需要添加open**，默认都是open的，需要被继承

——抽象类不能被实例化

继承父类：

```kotlin
class Lion: Animal(){		// 还是需要调用父类的构造方法的
    override val name = "lion"
    override val color = "yellow"
    override fun eat(){
        println("eating meat")
    }
}
```

总结：

继承和继承普通的父类一样：需要调用父类的构造方法；重写父类的属性，必须有和父类相同的属性名，属性类型和父类兼容——相同类型或者是其子类型。

eg:

```kotlin
// 抽象类
abstract class Animal {
    abstract val name: String		// 抽象属性没有赋值
    abstract val color: String
    open var volume: Int = 0		// 普通的实例变量，只有加了open，才能重写，否则只能使用

    abstract fun eat()			// 抽象方法没有方法体
    fun roam(){					// 没有加open，并不能重写
        println("hou~~~ with $volume")
    }
}

// 实现类
class Lion(volume: Int): Animal(){
    override val name = "LION"				// 重写了
    override val color = "yellow"
    override var volume: Int = volume		// 赋初值
    	get() = field * 10			
    	set(value) {			// 可以不写，默认就是这样的
        	field = value
    	}
    override fun eat() {
        println("eating meat")
    }
}

// 创建对象
val lion = Lion(3)
lion.eat()
lion.roam()
```

接口：表示一种能力。

1. 对于里面定义的方法，不需要添加abstract，且也可以有默认实现，**且即使实现了这个方法还默认是open**，可以被重写的——和abstract的区别
2. 接口没有构造方法
3. **接口的属性不能初始化**，**属性没有backing field字段**，也不需要添加open。但是可以添加getter方法，但是不能通过setter方法设置field（如果setter中没有涉及field，是可以使用的）
4. **接口的所有属性均需要实现，所有未有默认的方法均需要实现**

```kotlin
interface Animal {
    val name: String
        get() = "animal"

    fun eat() {
        println("eating everything")
    }
}

class Lion: Animal{
    override val name: String
        get() = "lion"
}
```

特殊情况，如果类实现A接口、B接口，都有同一个方法，而该类需要实现该接口：

```kotlin
overrride fun myFunction(){
    super<A>.myFunction()		// 调用接口A的接口默认实现
    super<B>.myFunction()
    ....
}
```

由于抽象类和接口也是继承，所以存在多态

——涉及到一个问题：类型判断的问题

## is/as

可以用is判断对象的潜在对象类型——等同于Java的`instanceof`

即：判断该变量实际指向的对象的数据类型，是否是某个类型、该类型的子类

```kotlin
val dog: Animal = Dog()
if(dog is Dog){
    dog.bark()
}
```

一般与when匹配使用：

```kotlin
when(animal){					// 可以不需要else
    is Dog -> animal.bark()
    is Cat, is Lion -> animal.meow()
    is Wolf -> animal.wo()
}
```

一般调用is之后，如果满足要求，会自动将该对象转换成判断的类型

但是**编译器不会将var类型的变量进行自动类型转换**——因为可能存在其他地方将该变量的引用给修改了

——使用as进行显式转换

```kotlin
val lion = animal as Lion		// lion变量的类型就是限定的Lion，而不是animal的Animal类型——保存有同一个对象的引用
```

——但是as能转换，是需要确定animal实际上存放的对象是Lion类型或者Lion类型的子类

# chap7 数据类

一种特殊的类——在Android中，最常用在model中，用来构建数据模型。

## 1. Any超类

类似于Java的Object超类。

有3个方法：

```kotlin
public open class Any {
    public open operator fun equals(other: Any?): Boolean
    public open fun hashCode(): Int
    public open fun toString(): String
}

```

所有对象的父类，最根本的是Any，所以不同的对象可以放在同一个数组中，数组的元素类型就是Any：

```kotlin
val array = arrayOf(Car(), Lion(), Tree())		// Array<Any>
```

**==实际上是去调用了该对象的equals方法。**

和Java一样，默认是比较这两个引用变量是否指向同一个对象，即使两个对象有相同的属性，也还返回false。

很多时候：我们需要比较对象之间的属性值。——如果某些属性值相等，那么认为两个对象相等。

——提出了**数据类**

一般：**数据类中定义的属性只建val——数据对象一旦被创建就不可改变**

要求：数据类定义时，必须要包含至少一个属性，属性需要包含val/var

```kotlin
data class Lion(val name: String, val sex: Boolean, val area: String){
  ....
}			// 数据类
```

数据类的优势：**会自动覆盖equals方法**，通过将主构造方法中的属性内容进行比较，来判断对象是否相等

——调用==，就是调用equals方法，同时也会修改hashCode、toString方法

——**调用`===`，还是会去比较指向的对象的地址**

方法：

1. copy：复制对象，对于要修改的属性进行指定，原始的对象不变

   ——**复制时仅包含主构造方法中的属性**

   一般：数据类对象的属性都是val类型，表示不可更改，所以如果想改变属性，就用copy来新建对象

   ```kotlin
   val person = Person("lilei", true, "China")
   val person2 = person.copy(name="hanmeimei", sex=false)
   ```

2. 获得属性

   除了用**对象名.属性名**：`person.name`

   还能用**对象名.component1()**：`person.component1()`——后面带的就是编号，是主构造方法中的定义顺序（**非主构造函数中的属性，不能如此调用，只能用上面的方法**）

   componentN可以用来解构对象：

   ```kotlin
   val (name, sex, area) = person
   ```

3. **data class不能是abstract和open修饰的——不是抽象类、也不可被继承，但是可以实现接口，可以去继承**

4. equals

   **数据类会默认覆盖equals方法，而该方法只会比较主构造方法的属性，其他属性不一样会忽略**

## 2. 默认参数值

```kotlin
data class Person(val name: String, val sex: Boolean = true, val area: String = "China")
```

创建对象时：

1. 可以按照指定顺序进行传参，对于包含默认参数的部分，可以不传入
2. 可以显式指定：`val person = Person(name="lilei")`——更为灵活，不想要赋值的可以不赋值
3. 必须要为每个没有默认值的参数赋值，否则会报错

方法中，也可以加入默认构造方法

```kotlin
fun getFunction(title: String = "",
               difficulty: Int = 0,
               ingredient: Array<String>){
  ...
}
```

## 3. 辅助构造方法

关键词：constructor

```kotlin
data class Person(val name: String, val sex: Boolean, val area: String = "China"){
  val height: Double = 183.0			//不在主构造方法中的属性——默认的equals、toString、hashCode都不包含它
  
  constructor(sex_param: Boolean): this("null", sex_param){	// 辅助构造方法
    println("using secondary constructor")
  }
}

// 使用
val person = Person(true)		——调用了辅助构造方法
```

要点：

1. 如果包含主构造方法，那么每个辅助构造方法必须要调用它，用this关键字

## 4. 方法重载

1. 返回类型可以不同，也可以相同
2. 不能只更改返回类型，必须要去更改参数列表——增删参数，修改参数类型等

## 5. Java使用kotlin

kotlin的构造方法可以包含给定默认值的，而Java需要调用其中一个的构造方法来构建对象

——Java调用kotlin：

1. Java调用的时候可以将所有的值全部一一赋值

2. **关键字`@JvmOverloads`——自动会创建包含默认值的重载构造方法**

   ```kotlin
   @JvmOverloads fun getFun(param1: String = "", param2: Int) // 会自动构建另一个重载的方法getFun()
   
   // 等价于：
   fun getFun(param1: String = "", param2: Int){  
       println("fun 1")
   }
   fun getFun(param: Int)
   {  
       getFun(param1 = "", param2 = param)  
       println("fun 2")
   }
   ```

3. 辅助构造方法

   注意，是**主构造方法，也必须带`constructor`**

   ``` kotlin
   class Person @JvmOverloads constructor(val name: String, val sex: Boolean, val area: String = "China"){  ....}
   ```

## ps: 其他特殊的类

### 1. 枚举类

一组值，是变量的唯一有效值，只能从中选择确定的值

定义和使用：

```kotlin
enum class Student{ 
    lilei, 
    wangxiaoming, 
    hanmeimei, 
    zhangxiaohong 
}

val student: Studentstudent = Student.lilei				// 只能从确定的几个变量中选择一个，而不能自定义一个
```

本质上：**枚举类有一个构造方法，会初始化每个枚举值——每个枚举值都是该枚举类的一个实例**

所以，我们可以给每个实例一些属性，还能添加方法：

```kotlin
enum class Student(val score: Int){				// 可以增加属性  
    lilei(89),  wangxiaoming(90),  hanmeimei(87),  zhangxiaohong(92);				// 注意这边有个分号
    fun graduate(){						// 可以添加方法    
        println("get score")  
    }
}
```

——所以能看出来枚举值本质上就是枚举类的实例

实例对于每个open的属性和方法都是可以重写的

```kotlin
zhangxiaohong(87){	
    override fun graduate(){
        println("crying")	
    }
}
```

### 2. 密封类

类似于加强版的枚举类

背景：枚举类限制很严格，只能有定义的那几个类型，且都是写死的——在创建类的时候就写死了，如果修改了，则其他地方会全部修改；每个值都有相同的属性和方法——有些情况，某个枚举值想要有点新增的属性or 方法

eg：

某个常见的回调方法：——表示执行是否成功：只有两种类型：success、failure

```kotlin
enmu class Message(var msg: String){  			// 枚举类
    Success("yes"),  
    Failure("no")
}
```

——密封类有多个层次：密封类是上层类型，子类型可以定义自己的方法和属性，而每个子类型可以创建多个实例

```kotlin
sealed class Messageclass Success(val msg: String): Message()
class Failure(val msg: String, val exception: Exception): Message()
```

**密封类只有有限子类，默认是private类型**，所以子类只能定义在同一个文件中

eg:

```kotlin
val success1 = Success("yes")
val success2 = Success("ya")
val failure = Failure("no", Exception("bad gate"))
val message: Message = failure				// 需要限定message的类型
val myMessgae = when(message){  
    is Success -> "suceess get the message: "+ message.msg  
    is Failure -> "failure get the message: "+ message.msg + ", exception: "+ message.exception.message
}
println(myMessgae)
```

——优势：由于是密封类的子类型，所以只有有限类，在when的时候不需要写else了

——该类在Android的响应回调函数中经常用到

### 3. 嵌套类和内部类

嵌套类和内部类都是写在一个类中的。

嵌套类类似于Java的静态内部类

无法访问外部类属性和方法

定义：按照普通的类来定义即可

使用：

```kotlin
val inn = Outer.InnerClass()			// 然后就能访问内部类的方法和属性了
```

内部类：能够访问内部类，但是需要先创建外部类的实例，才能创建内部类的实例/直接在外部类中创建内部类的实例，然后就不需要在创建了，可以直接创建外部类实例，然后直接使用

```kotlin
// 创建
class Outer{  
    inner class Inner{    
        val name: String = "inner class"    
        fun innerFun(){      
            println("inner class has a fun")    
        }  
    }
}

// 使用：
val inner = Outer().Inner()
```

### 4. 单例类/伴生对象/匿名类

kotlin提供了编译器自动提供的单例模式：——不能带构造方法

可以有属性、方法、可以继承（如果需要传参要写死）、实现接口

```kotlin
Object Instance{  
    var param = ""  
    fun instanceFun(){    
        ...  
    }
} //使用——直接用类名即可Instance.param
```

**对象声明——在类中声明，那么该对象就属于该类，该类的所有实例都可以共享**

eg： 

```kotlin
class Person{  
    Object PersonFactory{				// 对象声明——该类就拥有PersonFactory这个实例对象了    
        fun create() = Person()  
    }
}

// 使用
val person = Person.PersonFactory.create()
```

伴生对象：类似Java中的类属性、类方法。

——伴生对象是属于类的，所以每个实例对象都拥有，可以通过类名来直接进行调用

```kotlin
class Person{  
    companion object inner{				// 可以有对象名，可以没有    
        var x = 0    
        var y = 0    
        
        fun innerFun(){      
            println("location is ($x, $y)")    
        }  
    }
}

// 使用：
Person.innerFun()
Person.inner.innerFun()			// 等价
// 如果伴生对象是匿名的，可以用
Person.Companion
// 如果伴生对象有名字，不能用上面表达式，只能用伴生名字
```

匿名内部类 ，就是kotlin中的对象表达式：在表达式中创建一个没有名字的没有预先定义类型的类的对象

——常见的是，在类中实现某个接口/抽象类，直接在表达式中写了

```kotlin
window.addMouseListener(object: MouseAdapter(){	// 该方法需要传递一个MouseAdapter对象，而该类是抽象类，需要实现  
    override fun mouseClick(e: MouseEvent){    
        ...  
    }  
    override fun mouseRelease(e: MouseEvent){    
        ...  
    }
})
```

# chap8 kotlin的空值和异常

背景：kotlin能够自动推断变量的类型，然后将该变量的类型指定——那么该引用变量只能引用该类型，或者其子类

——如果变量赋值之后（该变量指向对应的对象），如何删除对变量的引用（不需该引用）

## 1. 可空数据类型

```kotlin
val person: Person? = Person(...)
person = null				// person该对象不指向任何一个对象
```

理解：

1. **默认的类型不能接受空值，除非指定该类型能够接受为空**
2. kotlin的优势：如果指定某个变量为可空数据类型，**kotlin会去追踪可以为空的变量，防止它进行非法操作NullPointException**
3. ?表示数据可为空

注意：

1. **方法的参数如果是可空数据类型，它还是要传递参数的**，除非给了默认值（可以是指定的对象，也可以是null，但是不能不传）

2. 方法的返回值是可空数据类型，可以返回指定的对象（或其子类），也可以是null

3. 可空数据类型的数组：

   ```kotlin
   val arr: Array<String?> = arrayOf("hello", null, "world")	// 数组可以是null类型——当然可以不事先指定数据类型
   
   for (item in arr)
   {  
       println(item)				// null值的输出也是null
   }
   ```

4. 如果变量在创建的时候`var temp = null`，那么编译器就认为**temp只能存放null的数据类型**

### 2. 访问可空数据类型

如果变量是可空数据类型，那么**需要在调用它的方法/属性之前需要判断该变量是否为null**

```kotlin
val person: Person? = Person("lilei", true)
if(person != null){			// 需要判空，否则会报错  
    person.eating()
}
```

注意：

1. 上面的写法也存在问题，如果在第二句和第三句中间，还是可能存在赋空操作的

——更安全的方法：

**`person?.eating()`：只有person非null才会执行后面的方法**

链式安全调用：即可以链接起来，只有前面一个为非null，才能执行下一步....

eg：

```kotlin
person?.name?.getFun()		// 只有person非null，之后判断name为非null，才会去调用getFun()
```

可空数据类型的赋值：

```kotlin
var height = person?.height		// 编译器会推断出：height的数据类型是Double?
```

```kotlin
person?.height = 183.0		// 只有person非空，才会将183.0赋值给该属性，否则就不操作，同理链式调用同样
```

### 3. let关键字

前面的安全访问`person?.height`，如果是要person非空的情况下进行某些person相关的操作：

1. 在执行这些操作的时候，person不能被其他线程赋空
2. 需要执行一系列的操作——多个表达式

——可以用关键字let配合?使用

```kotlin
person?.let{  
    println(it.name)
}
```

必须用{}包围起代码块，否则会编译错误。

### 4. Elvis操作符

**`x?.xxx ?: -1`**： x为非null的时候，返回它的某个属性值；x为null的时候，返回-1

eg：

```
person?.name ?: "lilei"
```

理解：

1. `?:`就是安全版的if表达式，如果左侧不为null，则返回左侧的值；否则返回右侧的值

### 5. !!关键词

正常情况下，kotlin的代码是不会抛出NullPointerException异常。

实际上，我们是**不用处理所有的空值去保证代码的安全——有些代码是能保证不会出现null值**。

!! 可以用来避免处理空值的问题

eg：`person!!.name`如果person为null，那么抛出异常

### 6. try...catch..finally

捕获异常：

```kotlin
try{			// 可能发生异常的代码块  
    ...
}catch(e: NumberFormatException){		// 捕获异常的时候，进行处理的代码块  
    ....
}finally{			// 可选，保证无论如何都会执行  
    ...
}
```

流程：

1. try代码块，一旦出现异常，就立即跳转到catch中执行代码
2. catch完成后会去执行finally
3. finally执行完成后，会执行剩下的代码块
4. 如果try/catch中有return指令，那么finally执行完成后会去返回执行return代码

每个异常都是Exception类型的一个对象，而Throwable是Exception的父类。

常见的异常：

1. NullPointerException：对空值对象进行操作
2. ClassCastException：对象转换时出现异常
3. IllegalArgumentException：非法参数异常
4. IllegalStateException：对象状态无效时异常

——可以自定义异常

### 7. 类型的安全转换

关键字is进行类型判断之后，会智能进行类型转换

eg：

```kotlin
val person: Animal = Person()
if(person is Person){  
    person.eat()
}
```

理解：

1. 并不能通过编译，因为编译器不能保证在类型判断之后，不会被其他线程重新赋值引用

——使用as关键词

```kotlin
if(person is Person){  
    val temp = person as Person  
    temp.eat()
}
```

理解：

1. 能够编译通过，但是如果在类型判断之后，被其他线程重新赋值，会抛出异常ClassCasrException

——**`person as? Person`如果person类型为Person才会被类型转换，否则返回null**

显式抛出异常：

**关键词 throw**

```kotlin
throw IllegalArgumentException("...")		// 参数异常
```

——但是，这个还需要包含try...catch...finally

```kotlin
try{  ...  
    throw IllegalArgumentException("...")  
    ...				// 不会被执行
}catch(e: IllegalArgumentException){
    ...
}finally{			// 可选
    ...
}
```

try是存在返回值的，默认是try中的最后一个表达式，或者是catch的最后一个表达式

对于throw的表达式：

1. throw的返回类型为Nothing，是一个没有值的特殊类型——该变量只能存放空值

   ```kotlin
   val x = throw(IllegalArgumentException("xxx"))
   ```

   x的变量类型为Nothing，所以赋值的话，只能赋值为null

2. 可以用Nothing代表不会被访问到的代码地址，eg：用作永不返回的函数的返回类型——代码执行到这边之后就会停止

# chap9 集合

## 1.  数组的常见操作

类似：Array<String?>

```kotlin
val arr = arrayOf(1,2,3)			// 创建数组
val nullArr: Array<Int?> = arrayOfNulls(2)			// 初始化时赋值为null，需要指定数组的类型
val size = arr.size			// 获取数组的长度
arr.reverse()				// 数组翻转
if(arr.contains(xx))			// 判断数组是否包含这个元素
val sum = arr.sum()			// 数组元素求和——必须要是数组，不能是Int?类型
val average = arr.average()				// 数组求平均——必须要是数组

// 已经废弃了，需要替换
val min = arr.min()
val max = arr.max()
val min = arr.minOrNull()			// 如果数组元素个数为0，那么返回的是null
val max = arr.maxOrNull()			// 同上

arr.sort()				// 数组升序排列
val arr2 = arr.plus(4)				// 复制数组，并且在数组后面加上新元素(有且仅有一个)
```

局限性：

1. 不能改变数组大小
2. 数组本身不能改变，而其中的**内容可以改变**`arr[1] = 8`

## 2. Collection类

只会写与Java的差别

简单的List、Map、Set类，是不可变的，**不能添加、删除、修改元素**

需要修改，MutableList、MutableMap、MutableSet可变集合类型

### 2.1 List类

list允许重复的元素，且元素中有先后顺序

```kotlin
val list = listOf(1,2,3)			// 编译器会自动推断数据类型，也可以显式指定
```

方法：

1. get

   如果输入的超出list的范围，那么会报错

   ```kotlin
   list.get(2)			// 获取index=2的元素内容
   list[2]				// 和上面等价
   list[2] = -1			// 编译失败——listOf创建的数组不能变化
   ```

2. indexOf

   ```kotlin
   list.indexOf(xxx)			// 如果元素存在就返回对应的index，如果不存在返回-1
   ```

3. 同数组一样，可以遍历

4. 同数组一样，可以用size判断数组长度

5. 同数组一样，可以判断list中是否存在某个元素contains

mutableList：可变的list，是list的子类型，所以list的方法全部都能用，并且还能**添加、删除、重排、更新已有的元素**

```kotlin
val mutableList = mutableList(1,2,3)
```

方法：

1. add

   ```kotlin
   mutableList.add(-1)				// 将元素-1添加到list尾部
   mutableList.add(1, 0)				// 指定插入的位置在index=1——会导致之前list中的1及之后的元素全部移动
   ```

2. removeAt

   ```kotlin
   mutableList.removeAt(1)				// 删除指定index的元素
   ```

3. set

   ```kotlin
   mutableList.set(0, 4)				// 修改指定index的元素
   ```

4. sort——修改list中的相对顺序

5. reverse——修改list中的相对顺序

6. shuffle——相对顺序的修改，会

7. addAll

   ```kotlin
   val list = listOf(1,2,3)
   val mutableList = mutableListOf(1,2,3)
   mutableList.addAll(list)			
   ```

8. removeAll——删除提到的所有数组

9. retainAll——只留下这部分的数据

10. clear——清空list

11. toList/toMutableList——list拷贝

### 2.2 Set类

唯一，无序的，所以没有get方法，同样不可变

存储同Java，就是放在哈希桶中，相同的哈希码的对象都放在同一个桶中

查重步骤：

1. 获取要插入对象的哈希码，与set中已有的元素进行比较

   如果没有重复，直接插入

   如果重复了，继续

2. 哈希码出现重复，就用`===`，比较指向的是否是同一个对象

   如果是，不插入返回

   如果不是，继续

3. 哈希码出现重复，且对象不一样，用`==`，比较属性是否一样——调用equals

   如果是，不插入返回

   如果不是，就插入set

同上面的list提供的方法。

并且，list和set能够互相之间转换，如果list中出现重复，转换为set的时候会将重复的删除

set之间可以比较是否相等，mutableSet和Set之间也能进行比较是否相等

list/set可以通过`toTypedArray()`转换成指定的Array

### 2.3 Map类

key-value，键值对

key不能重复，value可以重复。

创建map：

```kotlin
val map = mapOf("1" to 'a', "2" to 'b', "3" to 'c')
```

使用：

1. containsKey/containsValue

   判断是否包含key/value，如果不包含返回false，包含返回true

2. 访问

   map["1"]——访问value，但是如果key不存在就返回null

   map.get(key)——访问value，如果不存在返回null，不会报错

   Map.getValue(key)——访问value，不存在就报错

3. 遍历

   ```kotlin
   for((key, value) in map){	....}
   ```

4. 可变map

   ```kotlin
   val mutableMap = mutableMapOf("1" to 'a', "2" to 'b', "3" to 'c')
   mutableMap.put("-1", 'd')
   ```

   也可以使用putAll

   删除元素

   ```kotlin
   mutableMap.remove("1")			// 删除指定键
   mutableMap.remove("1", 'a')			// 只有在键值对完全匹配后，才能删除
   ```

5. toList

   可以将map转换成list类型

   ```kotlin
   val list = map.toList()				// [(1, a), (2, b), (3, c), (4, d)]
   ```

6. map.entries 可以直接访问键值对——返回的是set类型（mutableMap返回的是mutableSet类型）

7. map.keys/map.values——返回的是key/value的集合

   可以将其转换为set类型：`map.values.toSet()`——那么会删除重复的元素

# chap10 范型

类似Java的范型。

定义范型时，默认命名规范：

1. 普通的范型，用T作为占位符
2. 编写集合类、接口，用E作为占位符
3. 编写map时，用k、v作为占位符

可以限定T传入的必须是某个类型or其子类型

```kotlin
class Contest<T: Pet>{			// 传入的T必须是Pet或者Pet的子类型  
    ...
}
```

编译器能够自动推断出范型类型：

1. 指定了一个变量的范型类型

   那么在赋值的时候，就会默认知道赋值的变量类型

   eg：

   ```kotlin
   val contest: Contest<Dog>contest = Contest()
   ```

2. 编译器能够从构造函数中推断出范型的类型

   eg：

   ```kotlin
   class Contest<T: Pet>(t: T){  
       ....
   }
   
   // 创建对象
   val contest = Contest(Dog("wangcai"))		// ——因为参数就是对应的范型类型，所以能够直接推断
   ```

函数可以定义范型类型：

1. 需要在函数定义时，声明范型类型
2. **放在函数名之前，且用`<>`进行包围**
3. 调用该函数时，需要指定该函数处理的对象类型，如果编译器能够推断，则可以省略

```kotlin
fun <T: Pet>getPet(t: T): MutableSet<T>{  
    ....
}

val set = getPet<Dog>(Dog("wangcai"))
val set = getPet(Dog("wangcai"))				// 等效
```

如果希望返回的范型类型可以为空，则可以`T?`——要么是指定的范型t，要么是null

类型转换：

eg：

```kotlin
interface Retailer<T: Pet>{					// 包含范型的接口    
    fun sell(): T
}

class DogRetailer: Retailer<Dog>{				// 实现接口，需要指定具体的T的类型    
    override fun sell() = Dog("wangcai")
}
class CatRetailer: Retailer<Rabbit>{    
    override fun sell() = Rabbit("xiaobai")
}
```

```kotlin
val retailer: Retailer<Dog> = DogRetailer()				// 能编译通过
val retailer2: Retailer<Pet> = DogRetailer()	// 编译失败：实际类型是Retailer<Dog>,指定的类型是Retailer<Pet>
```

——从理论上可以，但是语法不通过

——**用范型的子类型对象代替范型的父类型对象，用out**

表示该范型类型是**协变的**：表示**子类型可以替换为父类型**

但是：

1. 类或者接口需要泛型类型作为方法的**返回类型**或者val属性的类型时，可以为该泛型类型加上out前缀
2. 但是如果类的**方法参数**或者var属性使用泛型类型作为类型，则不能使用out

eg:

```kotlin
val param: T
var param2: T					// 会报错的——逆变，不能用在var上
fun myFunction(): T{  
    ....
}
fun myFunction2(t: T){			// 会报错——逆变不能用在形参指定上
    ...
}
```

**逆变：父类型对象替代子类型，用in**

背景：

```kotlin
class Contest<T: Pet>(var vet: Vet<T>){    
    ....
}

class Vet<T>{				// Vet是范型类型    
    fun treat(t: T){        
        println("treat pet $t")    
    }
}

// 使用
val vetDog = Vet<Dog>()			// 创建能够看狗病的兽医
val vetPet = Vet<Pet>()			// 创建能够看任何宠物的兽医
val dogContest = Contest<Dog>(vetDog)			// 狗狗比赛，指定狗兽医
val dogContest2 = Contest<Pet>(vetPet)			// 狗狗比赛，指定能够看任何宠物的兽医——会编译错误
```

理解：从理论上讲，能够医治任何宠物的兽医，能够看狗病，但是编译失败

——**要用父类型来替换子类型**

 ——`class Vet<in T: Pet>`能够正确转换

——说明该范型类型可以逆变

局部逆变：——只有在Contest的创建传参过程中，才能进行逆变转换

```kotlin
class Contest<T: Pet>(var vet: Vet<in T>){			// Vet  
    ...
}
```

使用：

```kotlin
val dogContest = Contest<Dog>(vetPet)				// 编译通过
val dogVet: Vet<Dog> = Vet<Pet>()				// 编译失败
```

# chap11 lambda

Lambda表达式，就是包含代码块的对象，可以把它按照普通的对象进行处理。

## 1. 格式

1. `{}`包围，不能省略
2. 里面可以有0，1，多个参数
3. 参数定义之后紧跟->，用来**分隔参数和代码**
4. 最后一个表达式的结果就是lambda的返回值

eg：`{x: Int -> x + 5}`

**如果没有参数，那么可以省略前面部分**

## 2. 使用

赋值：`val addInt = {x: Int -> x + 5}`

——这个只是赋值，还需要显式调用代码块，才会去执行lambda中的代码，最后将结果赋值给变量

```kotlin
val addInt = {x: Int -> x + 5}
val addResult = addInt(5)					// 返回值是10
val addResult = addInt.invoke(5)
```

本质上：

1. 先创建一个lambda对象，然后用addInt对象去引用该变量
2. 执行lambda，并传入对应的参数x=5
3. lambda执行完成后，会创建一个10的对象，并返回该对象的引用给result

**类型：参数类型，返回值类型**

**(parameters) -> return_type**

eg：上面就是：`(Int) -> Int`，

1. 如果有多个参数，那么参数列表中有多个变量，eg：`(Int, String) -> Int`

2. 如果没有参数，那么可以是，eg：`() -> String`

3. 如果没有返回值，需要用Unit进行替换，eg：`val temp: () -> Unit = {println("hello")}`

   并且，如果你不想让lambda的结果可得，那么可以将返回值定位Unit，那么该表达式将没有结果。

   eg：`val calculate: (Int, Int) -> Unit = {x, y -> x + y}`

同普通变量一样：lambda对象也可以显式声明类型，并且对其进行重写：

```kotlin
val greeting : (Int) -> String
greeting = {x: Int -> "my name is $x"}
// 等价于如下：
val greeting : (Int) -> String = { x: Int -> "i am $x years old"}
```

如果表达式只有一个参数，且编译器能够推断出它的类型，那么可以将该参数省略，并且在表达式中用it进行替换

eg：

```kotlin
val addInt: (Int) -> Int = {x -> x + 5}			// 原始表达式
val addInt: (Int) -> Int = {it + 5}
```

**Any变量能够接受任何类型对象的引用，包括lambda**

## 3. 高阶函数

lambda还能作为函数参数。

需要指定：lambda的名字和参数类型

——参数列表中包含lambda / 返回值是lambda的函数——高阶函数

eg：

```kotlin
fun convert(param1: Double, param2: Int, converter: (Double) -> Double){	
    val result = converter(param1)				// 调用lambda  
    ....
}

// 传入参数：
convert(3.14, 4, {it * 2 + 0.1})
```

注意：

1. 如果lambda作为方法的最后一个参数，那么在调用该方法的时候，可以将lambda放在括号外面

   eg：`convert(3.14, 4) {it * 2 + 0.1}`

2. 如果该函数有且仅有一个参数，且参数为lambda，那么可以将整个括号删除

   eg：`convert{it * 2 + 0.1}`

3. lambda作为参数也可以有默认值

lambda作为函数返回值

eg：

```kotlin
fun convert(s: String): (Double) -> Double {    
    when (s) {        
        "yuanToDollar" -> return { it / 10.0 }        
        "yuanToJan" -> return {it * 50 }        
        "yuanTomeiyuan" -> return {it / 7.0}        
        else -> return {it}    
    }
}

// 使用
val result = convert("yuanToDollar")(5.0)			// 前面一个括号是调用方法，后面一个括号是调用lambda
```

ps：为类型取别名

背景：lambda可能有多个参数需要写，而每次都需要重新写一遍该类型，容易出错且浪费时间

——**可以用typealias来进行替换**

```kotlin
typealias doubleConversion = (Double) -> Double			// 所有出现该类型的地方可以用别名进行替换
```

——注意这个要**作为全局变量**，不然不能定义

不仅仅是lambda类型可以使用，**所有类型均能取别名**，eg：`typealias typeString = Array<String>`

# chap12 kotlin内置高阶方法

kotlin大多内置函数都是配合集合工作的。

## 1. 求最值/求和

之前集合中学到了，用min、max求集合的最大值——限定在元素类型都是基本数据类型才能直接进行比较

因为基本数据类型是自然有序的，即能够进行排序，但是

**min、max无法用于非自然顺序的类型**

——**minBy、maxBy函数可以对所有类型的元素进行排序，只需要传入指定的条件即可**

传入的条件是用lambda写的

eg：

```kotlin
// 数据类，构成的一个listdata 
class Good(val name: String,                   
           val goodClass: String,                   
           val unit: String,                   
           val unitPrice: Double,                   
           val quantity: Int)
fun main(args: Array<String>){    
    val grocery = listOf(        
        Good("tomatato", "vegetable", "kg", 4.5, 5),        
        Good("potato", "vegetable", "kg", 2.0, 10),        
        Good("ice cream", "frozen", "pack", 6.5, 5),        
        Good("oil", "cook", "bottle", 35.99, 4)    
    )
}
```

求最值 :

```kotlin
val highestPrice = grocery.maxBy{it.unitPrice * it.quantity}
```

理解：

1. `maxByOrNull`传递的参数是lambda

   ```kotlin
   // R是比较器，T是list的每个元素类型
   public inline fun <T, R : Comparable<R>> Iterable<T>.maxByOrNull(selector: (T) -> R): T? {  
       ...
   }
   ```

   所以格式：要传递元素类型，返回值是比较器，由于lambda的参数类型是已知的，所以不需要声明，直接写比较器即可：——即比较的条件

   ```
   it.unitPrice
   ```

2. 它的返回值是：元素类型——最大值/最小值元素

3. why：max/min只能比较基本数据类型，而maxByOrNull/minByOrNull能比较所有类型

   基本数据类型，内置实现了comparable接口的compare方法，所以知道如何排序

   而对于自定义的对象类型，需要自行实现comparable接口——如果实现了该接口（类似于Java），那么还是可以用maxOrNull/minOrNull来求最值的——太麻烦，直接用lambda会更方便一点

4. maxByOrNull/minByOrNull，如果list中没有元素，返回值为null

类似于maxByOrNull/minByOrNull，sumBy/sumByDouble的写法类似：

```kotlin
val totalPrice = grocery.sumByDouble { it.unitPrice * it.quantity }
```

理解：

1. sumBy只能对Int求值，sumByDouble只能对Double求值

2. 且两个方法不能直接对Map求值，因为map有key和value，遍历的时候就有两个参数，但是可以在value/key上对其进行求值

   ```
   map.values.sumBy(it)
   ```

## 2. 过滤器

过滤器，用在list上将返回一个新的list，用在map上将返回一个新的map

使用：

```kotlin
// 使用在list上
val newList = grocery.filter{it.goodClass == "vegetable"}			// 只有该条件为true的元素才能进入该list

// 使用在map上
val map = mapOf(  
    "tomtato" to tomtato,  
    "potato" to potato,  
    "ice cream" to ice_cream,  
    "oil" to oil)
val vegetableMap = map.filter { it.value.goodClass == "vegetable"}
println(vegetableMap.keys)
```

理解：

1. lambda同上面一样，因为元素类型是确定的，可以省略参数声明

   ```kotlin
   public inline fun <T> Iterable<T>.filter(predicate: (T) -> Boolean): List<T> {  ....}
   ```

   **表达式需要返回的是boolean类型**

filter方法的变种：

1. filterTo：将过滤后的元素加入到指定的list中——注意该list必须是MutableList类型，MutableMap也是一样

   ```kotlin
   val list: MutableList<Good> = mutableListOf()			
   
   // 初始化list
   grocery.filterTo(list){it.quantity > 3}				// 多传递一个参数
   ```

2. filterIsInstance：只有满足指定类型，或者该类型的子类才能加入到新的list中

   ```kotlin
   val newList = grocery.filterIsInstance<Good>()			// 不需要传参，只需要指定类型即可——不能不指定
   ```

3. filterNot：与filter相反

   不满足条件的进入新的list

**map函数：作用于list，按照指定公式对旧list进行运算，然后构建一个新的list存放**

——作用：作为旧list的映射/转换，取出里面元素的某些部分重新构建一个list

eg：

```kotlin
val newList = grocery.map{ it.name }
```

变种：

mapValues作用于map类型：——返回值是一个map，key值是不变的，变的是value进行了重新的映射

```kotlin
val newMap = map.mapValues { it.value.name }
val newMap = map.mapValues { it.key.unitPrice }		// 将原来key的其中一个元素作为新的value
```

### 3. foreach

Java中的foreach实际上就是for的简化版本——就是kotlin默认的for循环语法

在kotlin还有一个更为简单的遍历方法：

```kotlin
grocery.foreach{ println(it.name) }
```

更为简洁，且由于是函数所以可以在函数调用链中使用——放在最后，因为它**没有返回值**

foreach可以对**数组、list、set和map的entry、keys、values使用foreach**

由于foreach没有返回值，而如果要进行某些操作时，可以传递一个参数进去

——**lambda可以访问变量**

eg：

```kotlin
var totalItemName = ""grocery.foreach{ totalItemName += "${it.name} "}
```

官方一点的说法：lambda的闭包捕获了该变量

### 4. groupBy

按照某个条件进行分组，返回一个map，里面的每个元素都是一个list，存放满足条件的具体元素

注意：不能直接对map进行调用，可以对values、keys、entry进行分组

eg：

```kotlin
val groupMap = grocery.groupBy{ it.goodClass }

// 返回值：map：key-value：it.goodClass, list
{  
    vegetable=[Good(name=tomtato, goodClass=vegetable, unit=kg, unitPrice=4.5, quantity=5),            
               Good(name=potato, goodClass=vegetable, unit=kg, unitPrice=2.0, quantity=10)],  	
    frozen=[Good(name=ice cream, goodClass=frozen, unit=pack, unitPrice=6.5, quantity=5)],  	
    cook=[Good(name=oil, goodClass=cook, unit=bottle, unitPrice=35.99, quantity=4)]
}

// 而如果是一个表达式：
val groupMap = grocery.groupBy{ it.unitPrice > 3}
// 返回值：key-value：true表示表达式为真，里面存放的是满足表达式的元素，false相反
{  
    true=[Good(name=tomtato, goodClass=vegetable, unit=kg, unitPrice=4.5, quantity=5),         
          Good(name=ice cream, goodClass=frozen, unit=pack, unitPrice=6.5, quantity=5),         
          Good(name=oil, goodClass=cook, unit=bottle, unitPrice=35.99, quantity=4)],   
    false=[Good(name=potato, goodClass=vegetable, unit=kg, unitPrice=2.0, quantity=10)]
}
```

由于是可以链式调用的，所以可以在这个之后去遍历该map：

```kotlin
val group = grocery.groupBy { it.goodClass }.forEach {		// 遍历map  
    println(it.key)  
    it.value.forEach { 
        println("\t ${it.name}")
    }			// 对value的list里面的每个元素再次遍历
}

// 结果
vegetable	 tomtato	 potatofrozen	 ice creamcook	 oil
```

## 5. fold

最灵活的函数，主要的作用：给定一个变量（一般是基本数据类型/string类型），并给予一个初始化的值，然后遍历list/map的values、keys、entries对该变量进行操作

语法：两个参数

1. 初始值：可以为任意值，一般是基本数据类型 or String类型——不能省略，如果不想给初始值，可以用reduce方法，和fold类似，但是不能指定初始值
2. lambda：集合元素对初始值对应的变量进行操作

eg：

```kotlin
val totalPrice = grocery.fold(0.0){
    sumPrice, item -> sumPrice + item.unitPrice * item.quantity
}
```

理解：

1. 0.0就是初始值
2. sumPrice就是初始值对应的变量，item就是遍历的元素的名字
3. 后面的表达式就是具体的操作——这个的结果将赋值给sumPrice
4. 注意：fold、reduce都是从左向右遍历的，如果需要逆序遍历可以用：foldRight、reduceRight，但是只能用于数组和list，不能用于set和map——因为他们是无序的

Why有些函数map不能使用：

因为诸如`fold`、`forEach`和`groupBy`、`sumBy`、`maxByOrNull`等函数是**基于Iterable设计的**。由于Map不是Iterable类型，如果尝试对Map使用这些函数将获得编译错误，而map的keys、values、entry都是list类型，可以调用

# 其他细节

## 1. package、import

语法和Java一致。

代码都在包里。

优势：

1. 更好的组织代码
2. 避免命名冲突

完全限定名：包名 + 类名

在引用该类的时候可以用完全限定名，也可以通过import导入，那么可以直接使用类名进行调用了。

如果存在多个相同的类名（来自不同的包），那么可以用as重新命名（类似于python）

```kotlin
import kotlin.math.max
```

——如果类没有指名包，即在创建的过程中没有在确定的包中创建，那么会存放到无名包中

## 2. 修饰符

有4种修饰符：public、protected、private、internal

对于顶级代码：

1. public：所有地方可见，默认

2. private：只在该源文件可见

3. **internal：同模块代码可见，其余不可见**

   模块：是编译在一起的kotlin文件集合

对于类、接口、变量，可以用：

public、private、protected、internal

eg：

类的构造方法默认是public，如果想要限制其可见性，可以用：

```kotlin
class Student private constructor(val x: Int)
```

## 3. 扩展

可以在已有的类上扩展某个方法、属性，而不需要继承创建子类型

```kotlin
fun Double.toDollar(): String = "$$this"					// 直接在Double上增加了新的方法
val Double.floor							// 增加一个新的属性	
	get() = this.toInt()			

// 使用
println(3.14.toDollar())
println(3.14.floor)
```

## 4. 带标签的break/continue/return

嵌套循环等，可以直接跳出大循环

```kotlin
loop@ while (x < 20){  
    while(y < 10){		
        ...    
        if(y == 5) 
        break@loop				// 跳出指定循环  
    }	
}
```

continue类似，跳出后，会继续执行所在的循环的下一轮

带标签的return：主要是为了当return在循环中满足条件就返回时，还能执行最后的表达式

——注意，**只能用在foreach这种遍历的lambda中**

```kotlin
fun test(){  
    grocery.forEach loop@{
        if(it.name == "oil") 
        return@loop  				// 跳出循环，执行最后的表达式之后返回
    }  
    println("out loop")
}
```

## 5. 其他的一些关键字

 `vararg`：表示参数个数不确定，需要放在方法的最后，可以传递多个参数，本质上就是一个数组传递过去，所以可以遍历该数组

`*`：扩展运算符，在数组名前加上*来表示将数组中的值全部传入到这个vararg的变量中

```kotlin
val arr = arrayOf(1,2,3,4)

valuesToList(0, *arr, 5, 6)
```

`infix`：为函数添加前缀，让函数能够不使用点就可以被调用

```kotlin
class Dog{	
    infix fun bark(length: Int){    
        ...  
    }
}

// 使用
Dog() bark 5			// 甚至传参的括号也可以省略——需要只有单个参数，且没有vararg参数，且没有默认值
```

`inline`：为函数设置inline可以用来提高性能

——主要是在编译的时候，将该函数的内容替换到该函数调用中，从而减少了函数调用带来的栈等资源的消耗——代码量变大，但是耗时减少

# ps：库方法

## 1. 字符串相关

1. toLowerCase/toUpperCase/capitalize：全部转换为小写、全部转换为大写、首字母大写