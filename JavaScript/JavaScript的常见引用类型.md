# 集合引用类型

## 1. Object

Object的作用：Object的实例没有多少功能，但很适合**存储和在应用程序间交换数据**。

### 实例化

有两种实例化方法：

1. new

   通过调用构造函数来创建对象，而Object类的构造函数不需要传参

   ```javascript
   var person = new Object();
   person.name = "zcc";
   person.age = 18;
   ```

2. 对象字面量

   对象字面量一般用在给函数传递大量可选属性，比较直观

   **该方法定义时，并不会实际调用Object构造函数**

   ```javascript
   var person = {
       name: "zcc",		// : 代表属性名和对应的值，不同属性之间用逗号分隔
       age: 18,
       5: 19
   };			// 最后语句结束，记得带上分号
   ```

   ——对象字面量表示法通常只在为了让属性一目了然时才使用

   在最后一个属性后面加上逗号在非常老的浏览器中会导致报错，但所有现代浏览器都支持这种写法。

   这边注意概念：

   - **表达式上下文**：期待返回值的上下文，例如赋值操作符后面是需要一个值的，所以上面的例子的`{}`就是表达式上下文
   - **语句上下文**：位于语句后面，例如if后面的`{}`

```javascript
var person = {};			// 按照方式2创建，但是不给定属性
person.name = "zcc";
person.age = 18;		// 可以在
```

**属性名可以是字符串、数值**

选择何种构造方法：**最好的方式是对必选参数使用命名参数，再通过一个对象字面量来封装多个可选参数**

### 存取方式

1. 点语法

   `person.name=...;`

2. 中括号法（类似于python的字典）

   `person["name"]=...;`

中括号的优势是，**可以通过变量来访问属性**：

```javascript
var propertyName = "name";
person[propertyName] = "xxx";
```

如果属性名中包含**可能会导致语法错误的字符，或者包含关键字/保留字时，也可以使用中括号语法**

`person["first name"] = "cc"`，该属性只能够通过中括号去访问

### 宿主对象

和内建对象不同，eg：Array、Object等，这些都是ECMAScript自身定义的对象，还有一些**不是由ECMAScript提供的，而是由浏览器提供，由浏览器提供的预定义对象就是宿主对象**

eg：Form、Image、Element等，通过获取这些对象可以获得、控制网页上的信息

## 2. Array

数组里面可以存放任意数据类型的数据，JavaScript的数组不是密集型的

JS数组是动态大小的，会随着数据的添加而动态变化

### 创建数组

1. new，通过构造函数

   可以省略new操作符

   ```javascript
   var arr = new Array();		// 不给定长度
   
   var arr = new Array(4);			// 给定长度
   var arr = new Array("zcc", 18, true);		// 直接给定元素，元素有几个，数组就有多长
   ```

   存在一个问题：

   如果只传入一个值，**如果值为字符串（或者其他类型）——创建长度为1的数组**，里面只有这一个元素

   ​								**如果值为数值——创建长度为数值的数组**

2. 数组字面量，不会调用数组构造函数

   ```javascript
   var arr = ["red", "green", "blue"];
   var arr = [];		// 空数组
   var arr = [1, 2, ];			// 长度还是为2
   ```

3. 静态方法，类数组结构转换为数组实例：**`Array.from()`** 

   类数组对象，即任何可迭代的结构，或者有一个length属性和可索引元素的结构

   

4. 静态方法，将参数转换为实例： `Array.of()`



`var arr = [12, 14, 15, 18];`边创建边初始化

数组里面可以存放不同类型的数据

数组有一个方法`arr.sort()`，来进行排序，默认是升序排列

可以添加自定义的排序方式，可以传递方法：`arr.sort(compare)`

```javascript
function compare(x, y){
    return y - x;	// 这就是逆序排列
}
arr.sort(compare);

// 其他方法
arr.sort(function compare(blog1, blog2){
    return blog2.date - blog1.date;
})
```

### 数组填充

创建数组，之后填充元素——需要指定下标和对应的元素的值

`arr[0] = "zcc"`

如果下标超过了当前数组的长度，则会扩展到对应的长度，数组中间未填充的则会**填充为`undefined`**

### 关联数组（不推荐，本质上不是数组操作）

```javascript
let arr = new Array();
arr["name"] = "zcc";
arr["age"] = 18;
...
```

传入的索引值不是整数，而是字符串类型，这本质上不是在增加数组元素，而是**数组对象属性集合上的变量**，如上就是添加了至少两个属性name和age

——正常情况下，不应该添加数组的属性，这个功能应该Object来做

## 3. Date

Date对象可以用来存储和检索与特定日期和时间有关的信息

在创建Date对象的新实例时，**JavaScript解释器将自动地使用当前日期和时间对它进行初始化**，当然我们也可以自行指定时间

Date对象提供了getDay()、getHours()、getMonth()等一系列方法，以供人们用来检索与特定日期有关的各种信息