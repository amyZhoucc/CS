# kotlin的基础语法

## 1. 原理简介

主要就是：Java的编译过程是先编译生成class文件，后面再通过JVM将class文件进行**解释，**然后生成二进制文件。

所以，kotlin虽然语法不同，但是都是**生成相同规格的class文件**，所以对于jvm来说都是一样的内容。

kotlin的优势：兼容java，即程序中可以kotlin和Java代码并存；kotlin安全，例如常见的null异常，kotlin通过编译器可以避免；代码优雅，可以使用面向对象和函数式编程

## 2. 语法

### 2.1 基础数据类型

**所有数据都是对象，无基本数据类型**，所以，任何变量都是有成员变量和函数的。

但是，从定义上来说，还是将其区分为**原生类型值**和**对象类型值**。

基本数据类型/原生类型：

1. 数字：存在多个数字类型，注意**首字母都是大写**

   分别为：Byte, Short, Int, Long, Float, Double

2. 字符

3. 布尔值

4. 数组

5. 字符串

| 类型    | 长度 | 最小值    | 最大值     |
| ------- | ---- | --------- | ---------- |
| Byte    | 8    | -128      | 127        |
| Short   | 16   | -32768    | 32767      |
| Int     | 32   | $-2^{31}$ | $2^{31}-1$ |
| Long    | 64   | $-2^{64}$ | $2^{64}-1$ |
| Float   | 32   |           |            |
| Double  | 64   |           |            |
| Char    |      |           |            |
| Boolean |      |           |            |

对于实际的值：所有超出 `Int` 最大值的整型值初始化的变量都会推断为 `Int` 类型。如果初始值超过了其最大值，那么推断为 `Long` 类型。 如需显式指定 `Long` 型值，请在该值后追加 `L` 后缀。（这点同Java，和Java的区别在于：**非强数据类型，全看编译器自行判断，当然也可以显式定义**）

浮点数默认是Double数据类型，如果要变成Float，那么需要在数字后面添加F/f

```kotlin
val name1 = 1
val name2 = 1L
val name3 = 3000000000
val name4:Byte = 1		// 设定name4的数据就是Byte的数据类型

val pi = 3.14
val e = 2.7182818284f // Float，实际值为 2.7182817，存在精度损失
```

——注意的是：kotlin**无法实现隐式的数据转换**，即如果限制为Double数据类型，那么不能赋值一个Float数据

```kotlin
fun main(){
    fun doublePrint(d : Double){
        print(d)
    }
    val i = 1
    val j = 1.1
    val k = 1.1f
    doublePrint(k.toDouble())
}
```

强制类型转换：关键字：as

`Int`：就是对应Java的int数据类型；

`Unit`：

| 标志 | 含义                                                         | 使用                            |
| ---- | ------------------------------------------------------------ | ------------------------------- |
| ?    | 如果变量为空，就不执行后面的                                 | `mVelocityTracker?.add()`       |
| ?=   | 表示该变量是可空的                                           | `val mVelocityTracker ?= null ` |
| !!   | 该变量不可能为空，编译器跳过该报错，所以在运行的时候，变量如果为空，会抛出异常，程序会终止 | `mVelocityTracker!!.add()`      |
| ?:   | 如果前面的变量为null，那么就给后面的内容                     | `list.size()?:0`                |

?和?:可以有效避免nullPointerException的情况

### 2.2 定义变量

kotlin定义一个变量，只允许用关键字：**val、var**。

- val（value的简写）用来声明一个**不可变的变量**，这种变量在初始赋值之后就再也不能重新赋值

  ——主要是为了实现好的习惯，如果一个变量值不再变化，那么它应该被修饰为不可变类型

- var（variable的简写）用来声明一个**可变的变量**，这种变量在初始赋值之后仍然可以再被重新赋值，对应Java中的非final变量

——永远优先使用val来声明一个变量，而当val没有办法满足你的需求时再使用var

具体的类型，是编译器自行推导的，当然也可以显式指定。

```kotlin
val a: Int = 10
```

### 2.3 定义函数

基本语法规则：

```kotlin
fun methodName(param1: Int, param2: Int): Int{
	return 0
}

fun larger(num1: Int, num2: Int) = max(num1, num2)

```

主函数：

1.2及其之前的版本，必须要写成：

```kotlin
fun main(args: Array<String>){		// 即带参数
  ....
}
```

1.3及其之后的版本可以不带参，但是带参的版本能够兼容低版本。

### 2.4 逻辑控制

if：可以存在返回值，返回值就是最后一行代码

```kotlin
fun larger(num1:Int, num2:Int): Int{
  return if(num1 > num2){
    num1
  }
  else num2
}

fun larger(num1: Int, num2: Int) = if(num1 > num2) num1 else num2
```

When : 也可以有返回值

when中的条件的结构：

```
匹配值 -> {执行逻辑}
```

```kotlin
fun getScore(name : String) = when(name){
  "Tom" -> 86
  "jim" -> 60
  else -> 0
}
```

可以将参数放在了里面，作为表达式。——更为灵活 

```kotlin
fun getScore(name : String) = when{
  name == "Tom" -> 86
  name == "jim" -> 60
  else -> 0
}
```

while循环和Java一致。

可以表示一个区间：

```kotlin
val range = 0..10   // [0, 10]
val range = 0 until 10		// [0, 10)
```

```kotlin
fun main(){
  for(i in 0 until 10 step 2){
    ...
  }
  for(i in 10 downTo 1){		// [10, 1]
    ....
  }
}
```

### 2.5 静态方法

kotlin中很少用到，建议都是用单例类来实现类似于工具类的实现（Java中工具类就是不能实例化的，所有的方法全部是静态方法，直接通过类名进行调用），**单例类里面的方法可以类似于静态方法的调用，Util.fun()**

 但是：单例类会让里面所有的方法都必须按照静态的方法调用了，而如果**只希望个别方法变成类似于静态方法，可以用`companion object`**

本质：kotlin中没有定义静态方法的关键字，而是在**该类内部生成了一个伴生类**，并且保证类内部**只存在一个伴生类对象**，所以调用该方法，实际上就是调用该伴生类的实例方法

——单例、companion object都是从表明的调用格式上模仿了静态方法的调用格式

真正的静态方法：

1. 在**单例类/companion object中方法的基础上，加上了`@JvmStatic`**，那么编译器会编译成真正的静态方法（.class文件中是了）
2. 顶层方法：该方法不属于任何一个类，就默认是静态方法了，如果是顶层方法，就可以直接被调用了，不需要带任何前缀

### 2.6 常量

定义常量的关键字是：**const**，只有在单例类、companion object、顶层方法中才可以使用const关键字

### 2.7 变量延迟初始化

背景：由于kotlin会有判空操作，所以我们在定义一个变量的时候，如果不能直接给赋值，那么我们会写出类似的代码：

```kotlin
var textView : TextView ?= null

// 未来某个地方进行赋值
// 未来某个地方使用，注意用之前需要进行判空操作，即：
textView?.xxx(xxx)
```

所以会存在，在所有用到该变量的地方都需要进行判空操作

解决方法——延迟初始化

**关键字：`lateinit`**，编译器会知道晚些进行初始化，那么不需要给初始值null了，且对于那些不可以为null的类型，在使用的时候不需要进行判空操作了。

```
lateinit var textView : TextView
```

风险：如果该变量在未初始化时就进行了调用，那么程序会崩溃，且这个问题编译器检查不出来的

——所以**如果要用lateinit关键字，就需要保证在调用之前必须完成初始化**

可以通过代码判断一个变量是否已经初始化了：

```
::textView.isInitialized
```

——固定写法

## 3. 面向对象

### 3.1 类的基本概念

同Java的class

类的继承：类默认是不可继承的，所以需要显式打开

```
open class Person{
	...
}
```

```kotlin
class Student(val no :String, val grade: Int):Person(){}
```

——继承：只能有一个类被继承，继承必须要添加Person的构造方法，所以添加在内`Person()`，而如果Person的主构造方法需要传参的，那么继承时也需要传参

kotlin分为：主构造函数和次构造函数。

主构造函数：每个类默认都会有一个不带参数的主构造函数，当然你也可以显式地给它指明参数。**主构造函数的特点是没有函数体，直接定义在类名的后面即可**。

如果对主构造函数需要编写一些逻辑，可以用**init**结构体，都可以写在里面：

```kotlin
class Student(val no :String, val grade: Int):Person(){
  init{
    ....
  }
}
```

**次构造函数**：所有的次构造函数都必须调用主构造函数（包括间接调用）

关键字：通过**constructor**关键字来定义的

存在一个类没有主构造函数，但是有次构造函数的。因为没有主构造函数，所以也父类继承的时候也不需要构造函数。

### 3.2 接口

概念上同Java，但是实现上存在不同。

kotlin中统一使用`:`来分隔类和对应的父类和接口。

```kotlin
class Student(name: String, age: Int) : Person(name, age), Study{
  override fun readBooks(){
    ...
  }
  override fun doHomework(){
    ...
  }
}
```

存在允许对接口进行默认实现，所以同Java：只需要对**接口方法进行默认实现**，而对实现的方法进行选择性实现即可，那么调用的时候就会选择默认的实现逻辑。

——继承、实现，那么当然存在**多态**

### 3.3 函数的可见性

kotlin有4种可见性修饰符：

- public：默认修饰符
- private：类内部才可见
- protected：当前类和子类（和Java有不同）
- internal：在模块化开发中用到，对于模块中可见，而模块外部不可见。

### 3.2 数据类

`data class`：数据类，相对于Java的class所特有的。

`data class`定义时，**只有一些数据字段**，而对于一些常用的方法我们不需要再实现了。

编译器在背后默默给我们生成了如下的东西：

- equals()/hashCode()
- toString()方法
- componentN()方法
- copy()方法

——可以直接使用

需要注意的：

- 主构造函数必须要**至少有一个参数**
- 主构造函数中的**所有参数必须被标记为val或者var**
- 数据类不能有以下修饰符：abstract, inner,open,sealed
- 只能实现接口（Kotlin1.1以前的规则），现在也可以继承其它类

```kotlin
data class Cellphone(val brand: String, val price: Double)
```

一句话就可以将上面的方法、属性全部默认完成了。

当一个类中没有任何代码时，还可以将尾部的大括号省略

### 3.3 单例类

就是设计模式中的单例模式。

**有关键词：Object**，可以直接用来创建单例，编译器将一些固定的、重复的逻辑实现隐藏了起来，只暴露给我们最简单方便的用法。

然后里面写对应的方法即可，按照静态方法调用的形式进行调用即可。

```kotlin
object Singleton{
  fun singeltonFun(){
    ...
  }
}
```

这就是一个单例类，不需要向Java一样自行实现单例DCL模式

调用方法类似于Java的静态方法的调用：`Singleton.singletonFun()`——能够保证全局只有一个Singleton的实例变量。

### 3.4 内部类

使用inner class来定义内部类。

eg：

```kotlin
inner class ViewHolder(val fruitImg : ImageView, val fruitText : TextView)
```

### 3.5 密封类

背景：某个接口，只存在两个实现类，在使用的时候只能选择其中之一，而不存在其他的可能，那么可以尝试使用密封类，更加安全和规范。

eg：

```kotlin
interface Result
class Success(val msg: String) : Result			// 实现接口
class Failure(val error: Exception) : Result		// 实现接口2
```

然后再写一个方法来获得返回值：

```kotlin
fun getResultMsg(result: Result) = when (result){
  is Success -> result.msg
  is Failure -> result.error.message
  else -> throw IllegalArgumentException()
}
```

比较常规的做法：就是通过result的值来选择执行，但是存在一个非必要的else，因为只可能是其中的两个类型，这个是不可达的

所以可以选择用封闭类：

**关键字：`sealed class`**

```kotlin
sealed class Result
class Success(val msg: String): Result()
class Failure(val error: Exception): Result()
```

可以看到，从一个接口变成了一个封闭类，那么子类在继承的时候，需要带上括号（表示调用父类的构造方法）

那么：可以将里面的else去掉了

```kotlin
fun getResultMsg(result: Result) = when (result){
  is Success -> result.msg
  is Failure -> result.error.message
}
```

kotlin会检查密封类有哪些子类，并且将每个字类的所对应的条件全部补全，并且如果增加子类的话，对应的地方也要增加判断条件。

优势：没有冗余的代码，也不会存在条件漏写的情况。

### 3.6 泛型

泛型允许我们在不指定具体类型的情况下进行编程。——扩展性，然后在使用的时候传入具体的类型。

泛型主要有两种定义方式：

1. 定义泛型类：class MyClass<T>

2. 定义泛型方法

   ```kotlin
   fun method(param : T){}				// 在泛型类中，直接使用泛型变量
   // MyClass<Int>().method(123)
   
   fun <T> method(param : T){}		// 只有泛型方法
   // 使用
   MyClass().method<Int>(123)
   ```

使用的语法结构都是<T>

和Java中一样，可以限定类型约束：即上界：`fun <T : Number>`

——指定成数字类型，比如Int、Float、Double等。但是如果你指定成字符串类型，就肯定会报错

在默认情况下，所有的泛型都是可以指定成可空类型的，这是因为在不手动指定上界的时候，泛型的**上界默认是Any?**

#### 泛型的高级特征

kotlin中特有的功能。

泛型实化

Java的泛型功能是通过类型擦除机制来实现的，所有jvm语言就是使用类型擦除来实现的。

Kotlin提供的一个内联函数，会在编译的时候自动被替换到调用它的地方，这样的话也就不存在什么泛型擦除的问题了

——Kotlin中是可以将内联函数中的泛型进行实化的。

方法：

1. 首先，该函数必须是内联函数，也就是要用**inline**关键字来修饰该函数。
2. 在声明泛型的地方必须加上**reified**关键字来表示该泛型要进行实化

使用：

```kotlin
inline fun <reified T> getGenericType() = T::class.java
```

泛型T就是一个被实化的泛型，该方法就能获取指定的泛型的具体类型

## 4. Lambda

### 4.1 集合的了解和简化使用

背景：集合类中，kotlin和Java基本一致，eg：arraylist就是类似于java中的使用方法。但是，原始的代码可能存在很多冗余，而kotlin可以修改。

eg：

```kotlin
val list = listOf("apple", "banana", "orange", "grape")

//打印
for(fruit in list){
  print(...)
}
```

`listOf()`函数创建的是一个**不可变的集合**，只能用于读取，我们无法对集合进行添加、修改或删除操作。

如果需要可变集合，那么可以调用**`mutableListOf`**，那么创建的集合就是可变的。

`Set`操作和list基本一致，只不过将listOf变成了`setOf, mutableSetOf`

当然，set的特性和Java中一致，也需要保证元素的单一性。

而map和python的字典更像一点。

对于键值对，可以类似于Java中先创建哈希表，然后加入元素。当然也可以简化成：

```kotlin
map["apple"] = 1

var value = map["apple"]
```

当然还可以改进：

可以用上面的类似方法：`mapOf, mutableMapOf`

```kotlin
val map = mapOf("apple" to 1, "pear" to 2, "banana" to 3)

// 打印会稍微麻烦一点
for((key, value) in map){
  println(key + ", " + value)
}
```

to并不是关键字，而是一个`infix函数`

### 4.2 Lambda的语法和使用

概念：Lambda就是一小段可以作为参数传递的代码

语法结构：

```
{参数名1 : 参数类型, 参数名2 : 参数类型 -> 函数体}
```

可以看一个函数式API的整个进化流程：

是求：一个list中最长的字符串

```kotlin
val list = listOf("apple", "banana", "pear")
for (fruit in list){
  println(fruit)
}
```

可以调用一个list的方法 maxByOrNull，**本身就是接受一个lambda类型的参数的**，工作原理：根据我们传入的条件来遍历集合，从而找到该条件下的最大值——所以我们传入的条件是字符串的长度。

```kotlin
val lambda = {fruit : String -> fruit.length}
val maxLength = list.maxByOrNull(lambda)
// UP，可以直接将这两句话合并
val maxLength = list.maxByOrNull({fruit : String -> fruit.length})
//UUP，如果表达式是最后一个参数，那么可以移到括号外面
val maxLength = list.maxByOrNull(){fruit : String -> fruit.length}
//UUUP，如果表达式是最后一个参数，那么可以移到括号外面
val maxLength = list.maxByOrNull{fruit : String -> fruit.length}
//UUUUP，可以去掉参数类型声明
val maxLength = list.maxByOrNull{fruit -> fruit.length}
//UUUUUP，当参数列表中只有一个参数时，可以用it关键字代替
val maxLength = list.maxByOrNull{it.length}
```

eg:

```kotlin
val fruit = listOf("apple", "banana", "pear")
val newList = fruit.map { it.toUpperCase() }		// map函数接受表达式，将所有的值都变成大写的，然后生成一个新的集合
for (item in newList){
  println(item)
}
```

```kotlin
val newList = fruit.filter{it.length > 5}  // 将所有长度小于5的字符串全部过滤，构成一个新的list
```

```kotlin
val anyRes = fruit.any{ it.length > 5}
val allRes = fruit.all { it.length > 5 }
```

返回的是Boolean的值，一个是判断是否存在满足条件的值，一个是判断是否所有都满足

### 4.3 kotlin中使用Java的函数式API

可以在调用Java方法的时候使用函数式API，但是限制在：

**该方法只能接收：Java的单抽象方法接口参数（即Java的对象对应的类，只需要实现一个接口的一个方法，而不能需要实现多个方法）**

一个kotlin中未经过修改的创建线程的方法：

```kotlin
Thread(object : Runnable{			// kotlin中没有new，所以用object关键字来代替匿名类的创建中的new
  override fun run() {
    println("another thread")
  }
}).start()
```

精简：

由于只有值需要实现一个方法，所以不需要写出方法头，值需要包含实现，并且也不需要带上object关键字了

```kotlin
Thread(Runable{
  println("another thread")
}).start()

Thread{
  println("another thread")
}.start()
```

——在kotlin中调用Java的API是一件很常见的事，所以需要学会使用。

eg：常用的button的用法：

```kotlin
button.setOnClickListener{
  ....
}
```

## 5. 空指针检查

因为万物皆对象，所以传的参数可能存在null的情况，这时候如果想调用具体的方法，那么会出现异常，导致程序崩溃。

kotlin避免了这个问题：**编译器会进行判空操作**，即如果发现存在null对象却要进行方法调用，那么在编译的时候就会报错；

### 5.1 空类型

但是，还是存在有些变量是允许为null的，所以**可以有为空的类型操作**：

- Int? 表示类型可空

如果我们需要自行去判断是否为空，传统的是用if，但是可以进行简化

```kotlin
if(a != null){
  a.fun()
}

a?.fun()   // 表明如果为null，就不会调用该方法；不为null，才调用
```

还存在`?:`的方法

——它，左右两边都都是表达式，如果左边表达式的结果不为null，那么返回左边表达式的值，如果不成立（即为null），返回右边表达式的值。

eg：

```kotlin
fun getLength(text: String ?): Int{
  if(text != null){
  	return text.length
  }
  else{
  	return 0
  }
}
// 当然可以简化
fun getLength(text: String ?) = if(text != null) text.length else 0

// 可以可以用?:进行简化
fun getLength(text: String ?) = text?.length ?: 0
```

但是，对于全局变量，判空检查会存在失败，eg：如果全局变量已经给变量赋值，但是在方法中调用的时候，编译器还是会提示存在空指针风险。

——可以强行通过编译，**非空断言工具!!**，提示编译器我已经确保它不为空，但是这个不是最优的

### 5.2 标准函数

标准函数，是Standard.kt文件中定义的函数，可以随意调用该函数

#### 5.2.1 let函数

let函数是一个标准函数，主要作用是：辅助`?.`来进行辅助判空处理的

它提供了函数式API编程接口，并且能够将原始的对象作为参数传递到表达式中。

```
obj.let{obj2 ->
	//逻辑
}
```

obj对象会作为参数传递到里面的表达式中

eg：

```kotlin
fun study(stu: Stu?){
	stu?.let{stud ->
           stud.read()			
           stud.dowork()
  }
}

// 由于表达式中只有一个参数，那么可以修改成：
fun study(stu: Stu?){
  stu?.let{
    it.read()
    it.dowork()
  }
}
```

——**let可以处理全局变量的判空问题**

#### 5.2.2 with

参数：

1. 任意类型的对象
2. lambda表达式

作用：连续调用同一个对象的多个方法时，代码更加精简

效果：lambda表达式最后一行代码是返回值，表达式提供第一个参数对象的上下文

```kotlin
val result = with(obj){
  ....				// obj的上下文
  "value"			// 返回值
}
```

eg：将list中的所有内容都存放到StringBuilder进行保存

```kotlin
val fruits = listOf("apple", "pear", "banana", "grape")
val result = with(StringBuilder()){
  append("all fruits")			
  for(fruit in fruits){
    append(fruit + " ")
  }
  append("\n")
  toString()			// 最后一行就是返回值，就是sb.toString()
}
println(result)
```

可以发现里面的所有方法都不需要再找对象名了，因为处于**该对象的上下文中**了

#### 5.2.3 run

类同于with，也是用来简化函数调用的

参数：只有一个参数：Lambda表达式

返回值：最后一行就是对应的返回值

run实际上就是可以被替换成里面的方法

```kotlin
val result = StringBuilder().run{
  append("all fruits")			
  for(fruit in fruits){
    append(fruit + " ")
  }
  append("\n")
  toString()			// 最后一行就是返回值，就是sb.toString()
}
println(result)
```

#### 5.2.4 apply

类似

参数：只有一个参数Lambda

返回值：不是最后一行的，而是自动返回调用对象本身

```kotlin
val result = StringBuilder().apply{
  append("all fruits")			
  for(fruit in fruits){
    append(fruit + " ")
  }
  append("\n")
  toString()			// 最后一行就是返回值，就是sb.toString()
}
println(result.toString())
```

——with、run、apply类似，本质上就是用来简化代码的，可以合理利用他们

#### 5.2.5 repeat

就是将里面的内容重复执行n次

```kotlin
repeat(n){
  ...
}
```

## 6. 协程

协程属于Kotlin中非常有特色的一项技术

协程：可以理解成一种轻量级的线程

**协程却可以仅在编程语言的层面就能实现不同协程之间的切换，而不需要调度器进行调度了**——编程语言来决定如何在多个协程之间进行调度，让谁运行，让谁挂起

协程允许我们在单线程模式下模拟多线程编程的效果

### 使用

添加依赖库：

```
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.0"
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.1.1"
```

开启协程：

**GlobalScope.launch函数**可以创建一个协程的作用域，这样传递给launch函数的代码块。

```kotlin
fun main(){
  GlobalScope.launch{
  	println("xxxx")			// 这里面就是协程的作用域——这就是在协程中会执行的
  }
}
```

每次创建的都是一个顶层协程，**这种协程当应用程序运行结束时也会跟着一起结束**

 由于这边main函数为空，那么运行就结束了，所以这边协程无法执行，所以没有输出。

让程序延迟一段时间再结束，就能有输出了

Thread.sleep()方法让主线程阻塞1秒钟，但是如果协程中的代码块不能在阻塞时间中完成，那么就会被强制中断

`delay(xxx)` 可以在指定时间内进行挂起，只挂起当前协程，而不会影响其他协程。

——delay()函数只能在协程的作用域或其他挂起函数中调用

`Thread.sleep()`方法会阻塞当前线程，包括里面的协程

**runBlocking**函数：可以保证在协程作用域内的所有代码和子协程没有全部执行完之前一直**阻塞当前线程**——尽量在测试中使用，而正式中可能会影响性能。

创建多个协程：

在协程作用域中使用launch函数，传递的也是lambda语句

launch：**在协程的作用域中才能调用**，其次它会在当前协程的作用域下**创建子协程**

——子协程的特点是如果外层作用域的协程结束了，**该作用域下的所有子协程也会一同结束**

GlobalScope.launch函数创建的永远是顶层协程

### suspend关键字

背景：launch是在协程作用域中，开启一个子协程去执行里面的内容，但是如果内容较为复杂，需要单独成一个函数，那么存在的问题是：launch函数中编写的代码是拥有协程作用域的，但是提**取到一个单独的函数中就没有协程作用域**，而如果想使用delay等函数就存在问题

——suspend关键字：可以将任意函数声明成挂起函数，而挂起函数之间都是可以互相调用的，所以可以在里面使用delay挂起函数了

eg：

```kotlin
suspend fun printDot(){
	println(".")
	delay(1000)
}
```

但是：suspend存在的问题是只能声明为挂起函数，而**无法提供协程作用域**，eg：无法调用launch函数

### coroutineScope函数

coroutineScope函数也是一个挂起函数，因此可以在任何其他挂起函数中调用。

特点：会继承外部的协程的作用域并**创建一个子协程**，且保证其作用域内的所有代码和子协程在全部执行完之前，外部的协程会一直被挂起。

```kotlin
suspend fun printDot() = coroutineScope{
	launch{
		println("xxxx")
		delay(1000)
	}
}
```

对比：

coroutineScope和runBlocking的对比：

1. coroutineScope函数只会阻塞当前协程，既不影响其他协程，也不影响任何线程，因此是不会造成任何性能上的问题的。
2. runBlocking函数会挂起外部线程，会存在性能问题

### 批量取消协程

```kotlin
val job = Job()
val scope = CoroutineScope(job)			// 会返回一个实例对象
scope.launch {			// 可以调用launch来创建协程了

}
job.cancel()
```

——这边，launch函数创建的协程都会被关联在Job对象的作用域下面

只需要调用一次cancel()方法，就可以将同一作用域内的所有协程全部取消

### 使用async函数

launch函数返回的是job对象，但是没有办法获取该代码块中的执行结果——使用**async**函数

但是：**必须在协程作用域当中才能调用**。

效果：创建一个新的子协程并返回一个Deferred对象

那么只需要调用**Deferred对象的await()方法**即可

```kotlin
runBlocking {
  val result =  async {
    // 具体操作
  }.await()
  println(result)
}
```

内部逻辑：在调用了async函数之后，代码块中的代码就**会立刻开始执行**。当调用await()方法时，如果代码块中的代码还没执行完，那么**await()方法会将当前协程阻塞住**，直到可以获得async函数的执行结果

所以，如果一个协程中存在多个async，并且后面紧跟await，就会变成串行执行，而为了避免这种情况，可以将先获取deferred对象，之后在需要的地方调用await方法即可。

### withContext()函数

withContext()函数是一个挂起函数，大体可以将它理解成async函数的一种简化版写法

具体表现：**立即执行代**码块中的代码，同时将**外部协程挂起**。当代码块中的代码全部执行完之后，会将最**后一行的执行**结果作为withContext()函数的返回值返回

```kotlin
runBlocking {
  val res = withContext(Dispatchers.Default){		// 等价于=async{}.await()
    // 具体操作
  }
  println(res)
}
```

注意，这个需要传参：指定一个具体运行的线程（因为很多操作必须在线程中执行，而不能在协程中，eg：网络请求，还是要在指定的子线程中，虽然可以在指定的线程的协程中执行，但是不能在主线程的协程中执行）

线程参数主要有以下3种值可选：

1. Dispatchers.Default：默认低并发

2. Dispatchers.IO：高并发

   当你要执行的代码大多数时间是在阻塞和等待中，比如说执行网络请求时，为了能够支持更高的并发数量，此时就可以使用Dispatchers.IO

3. Dispatchers.Main：不开启子线程，只能在Android项目中使用

其他所有的函数都是可以指定这样一个线程参数的，只不过withContext()函数是强制要求指定的，而**其他函数则是可选的。**

### suspendCoroutine函数简化回调函数

背景：在网络技术学习中，可以发现我们需要写一个回调接口，然后用一个匿名类来实现回调函数，但是每次都请求网络的时候都需要去写回调函数——将其简化。

——借助suspendCoroutine函数。

要求：suspendCoroutine函数只能在协程作用域or挂起函数中调用，它接收一个Lambda表达式参数

作用：将**当前协程立即挂起**，然后在一个**普通的线程**中执行Lambda表达式中的代码

Lambda表达式的参数列表上会传入一个**Continuation参数**，调用它的resume()方法或resumeWithException()可以让协程恢复执行。

```kotlin
// 是一个挂起函数，那么能调用suspendCoroutine函数
// 返回值是string类型
suspend fun request(address : String) : String {
    return suspendCoroutine { continuation ->
        // 这个就是普通的httputil的请求调用，具体见11章
        // 需要传递的就是匿名类
        HttpUtil.sendHttpRequest(address, object : HttpCallbackListener{
            override fun onFinish(response: String) {
                continuation.resume(response)		// 协程恢复执行，并且带上返回值
            }

            override fun onError(e: Exception) {
                continuation.resumeWithException(e)		// 协程恢复执行，并且带上异常
            }
        })
    }
}
```

使用：

```kotlin
suspend fun getResponse(){
	try{
    val response = request("https://www.baidu.com")		// 调用之后，当前协程马上被挂起，直到上面的执行结束后继续执行，那么就能得到返回值了
    xxxx(处理返回的response值，string类型)
  }catch(e : Exception){
    xxxx（处理异常情况）
  }
}
```

但是：**该getResponse必须在挂起函数 or 协程作用域中执行**

而对于retrofit来说也很简单：

原来的代码：

```kotlin
// 创建一个retrofit对象
val retrofit = Retrofit.Builder().baseUrl("https://www.baidu.com").addConverterFactory(GsonConverterFactory.create()).build()

// 调用create方法，创建该接口的动态代理对象——能够调用接口中的方法了
val appService = retrofit.create(AppService::class.java)
// 里面有匿名类，用来实现onResponse、onFailure方法
appService.getAppData().enqueue(object : retrofit2.Callback<List<App>>{
  override fun onResponse(
    call: retrofit2.Call<List<App>>,
    response: retrofit2.Response<List<App>>
  ) {
    val list = response.body()
    if(list != null){
      for(app in list){

      }
    }
  }

  override fun onFailure(call: retrofit2.Call<List<App>>, t: Throwable) {
    TODO("Not yet implemented")
  }
})
```

可以简化成：

```kotlin
// 挂起函数await，且作为retrofit2.Call的扩展函数（即，所有retrofit2.Call都有该方法了），传递的参数是泛型，返回值也是泛型，对应的数据类型
suspend fun <T> retrofit2.Call<T>.await() : T{
    return suspendCoroutine { continuation ->
        // 由于本身是call对象，那么可以直接使用enqueue发起网络请求
        enqueue(object : retrofit2.Callback<T>{
            override fun onResponse(call: retrofit2.Call<T>, response: Response<T>) {
                val body = response.body()		// 可能存在为null
                if(body != null){
                    continuation.resume(body)
                }
                else{		
                    continuation.resumeWithException(RuntimeException("response body is null"))
                }
            }

            override fun onFailure(call: retrofit2.Call<T>, t: Throwable) {
                continuation.resumeWithException(t)
            }
        })
    }
}

// 使用：
try {
  val appList = ServiceCreator.create<AppService>().getAppData().await()
  ....
}catch (e : Exception){
	....
}
```



## 小技巧

### 字符串拼接

格式：`${}`，那么在运行的时候，编译器会自动将这个替换成具体的内容

eg：

```
"hello, ${obj.name}, nice to meet you"
```

如果字符串中**只存在一个变量**，那么可以将括号省略

### 参数默认值

就是在函数定义的时候，直接给值，那么该参数的传参不是必须的，但是其他参数的值必须传递过来

eg：

```
fun function(num1: Int, str: String = "hello"){
	...
}
```

并且，kotlin允许通过键值对的方式传参：

eg：

```kotlin
function(str = "hello", num1 = 123)
```

——有了这个构造方法后，只需要给主构造方法默认值，然后按照任何的传参方式都可以组合式的进行选择性实例化对象了

### getter/setter

对于kotlin中调用Java的方法时，如果方法是getter/setter，那么可以直接通过读取对应的实例变量，来调用该方法

eg：

```kotlin
val book = Book()
book.pages = 500		// 实际上默认调用了getter/setter方法
```





## ps：一些默认方法

1. `MainActivity::class.java`：返回class对象
2. `javaClass.simpleName`：javaClass等同于Java中的getClass，可以获取当前实例的class对象，simpleName是语法糖，等同于方法`getSimpleName()`

## pps：常见的报错问题

1. Smart cast to 'VelocityTracker' is impossible, because 'velocityTracker' is a mutable property that could have been changed by this time

   意思是：该变量可能会被其他线程修改，导致在执行的时候它的值有可能为`null`

   解决方案：如果该变量能够确保不会被修改为null，可以强制为!!

   如果不能保证，那么用?

2. 注意强制类型转换，只能出现在父子类中，如果是其他的需要有方法进行转换，这个常见在基本数据类型之间的转换

   `Float -> Int`——有方法可以直接使用：`xx.toInt()`，而不能使用as，不会编译错误，但是程序会崩溃

   并且如果在调用该函数的时候，发现该变量可能为0，那么需要进行兜底：

   `x?.toInt() ?: 0`——?:表示的含义是，如果前面的变量为null，那么返回后面的值0 


