# JavaScript基础语法

主要内容来自于《head first JavaScript》（最后一章和Ajax相关的过了一遍，但是不是很理解，所以没有写上去:sweat_smile:打算后面看《JavaScript DOM编程艺术》和《JavaScript高级程序设计》有更详细的了解之后进行补充，并且后面看这两本书的时候，对简单的必要的知识补充会在这边，如果比较复杂，会直接新开内容）

## JavaScript的性质

JavaScript是脚本解释语言，弱类型语言

JavaScript的优势：让原本静态的网页具备**交互性**，根据用户的反馈进行页面变化

**借由响应用户的输入数据，JavaScript给网页带来生命，JavaScript负责响应用户要求网页执行任务**

eg：判断用户的输入是否合法，去查询服务器上面的数据，进行计算

JavaScript、HTML、CSS并称为网页三大元素，HTML是车的骨架、CSS是车漆、JavaScript是轮胎，**只有JavaScript存在，车（网页）才能动起来**

JavaScript代码一般也直接放在网页中。

浏览器执行逻辑：

<img src="pic\image-20220330134028003.png" alt="image-20220330134028003" style="zoom:77%;" />

**JavaScript有一个事件机制**，可以在网页上发生某个事件后才触发对应的JavaScript代码

因为互联网存在漏洞（https出现之前），所以要防止JavaScript有太大的权限，例如：**JavaScript不允许读写用户硬盘上的文件**，但是

（浏览器内置有JavaScript的解释器，所以能够运行脚本代码）

JavaScript的代码放在head里面，函数里的代码调用时才被运行，而函数外的在代码载入的时候直接运行了

eg：`var console = new DebugConsole()`不能在标头创建，因为它的构造函数和网页主体内容十分相关，所以要把它放在HTML代码中**`<body onload="console = new DebugConsole();">`**，后面在JavaScript代码中可以控制控制台打印出内容**`console.displayMsg("...")`**

## 标签

**`<script></script>`**，提示浏览器这个标签内的是脚本

HTML的任何地方都能插入脚本，但是最好固定地方，例如文件尾巴（书上说是<head>部分，但是我看实际代码都是将脚本放在尾巴处）

```html
<html>
    <head>
        <script type="text/javascript">		// 脚本相关的内容
        </script>
    </head>
    <body>
        ....（html的正文）
    </body>
</html>
```

注意，script标签表示是后面是脚本代码，浏览器可以支持多种脚本语言的代码，但是**`type="text/javascript"`**才是指定脚本语言是JavaScript的（目前已经被废弃）

如果JavaScript是用外部文件，则需要导入文件

```html
<script type="text/javascript" src="cookie.js"></script>
```

## JavaScript的语法

JavaScript区分大小写

### 基本数据类型

基本数据类型都是按值访问的

有3种基本数据类型：

1. **number**，可以是整数也可以是浮点数

   如果数字本身是整数，就是小数点后面跟着0，它也会被转换为整数（为了减少内存开销）

   number能表示的数值有范围，最小为Number.MIN_VALUE；最大为Number.MAX_VALUE（和Java中的一样）

   当计算得到的数超过表示范围后，这个值会自动转换成一个特殊的值，正数为**`Infinity`**，负数会被转换为**`-Infinity`**

   可以用`isFinite()`函数来测试值是否是有限大

   转型函数，可以用于任何数据类型：`Number()`

   字符串转换为数值：`parseInt()`和`parseFloat()`

   - Number()

     字符串可以转换为合法的数值时（整个字符串都是合法数值的），能够进行转换；空字符串，返回0；其他无法转换的，直接返回NaN

     对于对象，调用`valueOf()`按照规则转换，如果转换结果为NaN，调用`toString()`方法，再按照字符串的规则转换

   - parseInt()

     从第一个非空格字符开始转换。如果第一个字符不是数值字符、加号或减号，parseInt()立即返回NaN。一直转换直到字符串末尾，或碰到非数值字符。所以空字符串会直接返回NaN

     eg：`parseInt(1234blue)=1234`，`parseInt(blue1234)=1234`

2. **boolean**，true/false

   其他类型都有相应的布尔值等价形式，可调用特定的转型函数：`Boolean()`

   eg：`Boolean(message)`

   <img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20220410160842837.png" alt="image-20220410160842837" style="zoom:40%;" />

3. **string**——特别注意，它是基本数据类型，而不是对象，也不是数组

   可以用单引号`''`、双引号`""`、反引号` 标示

   转换为字符串的方法`toString()`，`11.toString()="11"`，`toString()`接受参数，表示以什么底数进行转换`10.toString()="1010"`

4. **undefined**：未初始化，是由null派生而来，他们字面上相等

   `undefined == null`返回的是true

   ——假值

5. **null**：是对空对象的引用

   ——假值

   建议：对于要保存对象的变量是，而当前又无法赋值的，建议直接赋值为null——这样就可以保证变量的语义，并且也能和undefined进行区分

5. symbol

还有其他类型：

1. 描述“没有值”这个状态的值：**`undefined`**，声明了变量但没有初始化就是赋予了undefined值

2. 非数字：**`NaN`**，无法计算出该数字的值，是用来表示数字数据类型有误时的指标，一般是在计算需要数字，但是收到非数值的数据时才会看到NaN

   并且非数值可以转换为数值的话，返回也是false（先转换为数值，后判断）；布尔值也能转换为数值，所以返回false

eg：alert里面只能传入string类型，所以如果要输出数字，实际上内部是将它转换为文本之后呈现。特别的如果代码是一个表达式，那么会先计算，eg：`alert(1+3);` 就会输出4

判断变量当前类型：**`typeof xxx`**，返回的是字符串类型的结果

一些反直觉常见的返回结果：

1. `typeof null = "object"`
2. `typeof message = "undefined"`（message是声明了，但尚未初始化的变量）
3. `typeof message = "undefined"`（message尚未声明）——对于未声明的变量，只能进行这一个操作

### 变量声明的关键字

1. **变量`var`**，在使用之前初始化即可

   - 未初始化，变量会保存一个特殊值undefined

   - JS解释器会自行推断变量类型，且赋值修改之后可以改变数据类型（合法但不推荐）

   - **var声明提升**——**所有变量声明都拉到函数作用域的顶部**

     ```javascript
     function example(){
         console.log(age);
         var age = 23;
     }
     example();			// 不会报错，返回undefined
     
     // 上面的代码等价于
     function example(){
         var age;			// var的声明提升，但是赋值不会提升
         console.log(age);
         age = 23;
     }
     example();
     ```

   - 反复用var声明同一个变量名是不会报错的

   - var在全局声明的变量会成为**window对象的属性**

     eg:

     ```javascript
     var name = "zcc";
     console.log(window.name);		// 正常输出
     ```

2. **let**：

   - let声明的范围是块作用域，离开代码块后就被清除了（var的区别）

   - 用let声明同一个变量名会报错（var的区别2）

   - **let的声明不会提升**（var的区别3）

     ```javascript
     function example(){
         console.log(age);
         let age = 23;
     }
     example();			// 会报错，提示Reference Error，变量尚未定义就使用
     ```

   - 如果let和var都声明一个变量名，会报错——因为let和var只是指出变量作用域的位置

   - let即时全局声明，也不会成为window对象的属性

     eg：

     ```javascript
     let name = "zcc";
     console.log(window.name);		// undefined
     ```

   ——let不能依赖条件声明：let的作用域是代码块内部，所以不会去检查前面是否定义过；且它禁止重复声明，所以声明过了保险起见的重新声明会报错

   ```javascript
   // 条件声明1
   if(typeof name == 'undefined'){
       let name;		// 这样声明的name作用域在if语句内，出去了就无效了
   }
   
   // 条件声明2
   try{
       console.log(name);
   }catch(error){
       let name;			// 这样的条件声明也是无效的
   }
   ```

   ——**对于let这个新的ES6声明关键字，不能依赖条件声明模式**

   let最常用的地方：for循环的迭代变量i，声明let，可以避免渗透到循环体外部

3. **常量`const`**，声明时就需要赋值，且只能赋值一次。

   如果const修饰的是一个对象，那么该对象的引用无法修改，但是对象的属性值可以修改

4. 如果在方法里面声明一个变量，但是没有带上关键字var，那么会变成全局变量——不推荐的写法（这些变量将很难维护，在严格模式下，会抛出ReferenceError）

重新载入网页，脚本的数据会重置，变到尚未执行时的状态

变量命名：

1. 至少需要有一个字符
2. 第一个字符是字母或者`$`或者`_`
3. 后面的字符是字母、数字、`_`、`$`

默认的命名方式：**小写驼峰式**，常量全大写

将变量写在方法外面，就是全局变量，它的存活时间和脚本的生命相同

### 操作符

特别需要注意的点：

1. 指数操作符，`**`，可以用来替换Math.pow()
2. `~`按位非操作，返回数值的补数，最终效果是数值取反，后-1
3. 逻辑非操作（`!`）会操作数转换为布尔值
4. ++/--，字符串，如果是有效的数值形式，则转换为数值再应用改变。**变量类型从字符串变成数值**；如果不是有效的数值形式，则将变量的值设置为NaN；浮点值，加1或减1；如果对象，valueOf()方法取得可以操作的值，然后再应用规则

#### 比较操作符

需要注意的是：

1. 操作数都是字符串，则逐个比较字符串中对应字符的编码

2. 任一操作数是数值，则将另一个操作数转换为数值，执行数值比较

3. 任一操作数是对象，则调用其valueOf()方法，取得结果后再根据前面的规则执行比较。如果没有valueOf()操作符，则调用toString()方法，取得结果后再根据前面的规则执行比较

4. 有任一操作数是布尔值，则将其转换为数值再执行比较

5. 如果一个是字符，一个是数值，字符会进行转换，但是如果无法转换（变成NaN）

   ——任何关系操作符在涉及比较NaN时都返回false

   ——所以就会返回false

#### */%三种操作符

**对于非数值，会增加一个自动类型转换，使用Number()转换成数值**，如果转换失败则返回NaN

*的特殊情况：

1. 如果有任一操作数是NaN，则返回NaN
2. Infinity乘以0，则返回NaN
3. Infinity乘以非0的有限数值，则根据第二个操作数的符号返回Infinity或-Infinity
4. Infinity乘以Infinity，则返回Infinity

/的特殊情况：（如果无法除尽，则会出现小数）

1. 如果有任一操作数是NaN，则返回NaN
2. Infinity除以Infinity，则返回Infinity
3. 0/0，返回NaN
4. 非0的有限值除以0，则根据第一个操作数的符号返回Infinity或-Infinity
5. Infinity除以任何数值，则根据第二个操作数的符号返回Infinity或-Infinity

%的特殊情况：

1. 被除数为infinity，除数是有限值，返回NaN
2. 被除数为有限值，除数是0，返回NaN
3. infinity % infinity，返回NaN
4. 被除数为有限值，除数是无限值，返回被除数
5. 0 % 非0值，返回0

#### 相等操作符

有两组，来比较两个变量是否相等的

1. `==` 和 `!=`，在比较之前如果便两个类型不同，会先进行转换，后比较
2. `===` 和 `!==`，类型不同也不转换

类型转换的规则：

1. 存在一个布尔值，先转换为数值，后比较
2. 存在一个字符串，一个数值，字符串尝试转数值（调用Number()方法转换的）
3. 一个变量是对象，则先调用valueOf()取得值，再比较

比较的规则：

1. null和undefined在`==`上相等，在`===`不相等，因为不是相同的数据类型
2. null和undefined不能转换为其他类型的值再进行比较
3. 有任一操作数是NaN，则相等操作符返回false
4. 如果两个操作数都是对象，则比较它们是不是同一个对象

### 分号

语句以分号做结尾，**虽然不是必须的，也应该加上**

好处：

1. 能够方便删除空行，从而压缩代码
2. 加分号能够提升性能，因为解析器会尝试在合适的位置补上分号以纠正语法错误。

### 注释

单行注释`//`

多行注释：`/*`开头，`*/`结尾

```javascript
/**
 *			这里面的星是为了提高注释的阅读性
 *
 */
```

还可以使用HTML格式的注释，可以作为单行注释（不建议）

`<! --`，注意不需要结束的`-->`

### 严格模式

严格模式是一种不同的JavaScript解析和执行模型。

一些不规范的写法会在这种模式下会被处理，一些不安全的操作也会被抛出

1. 对整个脚本启用严格模式，脚本开头加上：`"use strict";` 这是一个预处理指令，浏览器能够据此开启严格模式
2. 可以单独指定对单个函数开启严格模式，在函数开头即可：`"use strict";`

### 对象

是复杂数据类型，按照引用访问

#### 动态属性

对于引用值来说，可以随时添加、修改和删除属性和方法

```javascript
let person = new Object();
person.name = "zcc";			// 给该对象添加了一个name的动态属性，后面就可以对它操作了
```

辨析：原始值是不能有属性的，虽然添加属性的操作不会报错，但如果想访问就会报错

```javascript
let name = "zcc";
name.age = 27;
console.log(name.age);		// 会打印出undefined
```

如果基本数据类型，但是可以创建一个Object类型的实例，从而能够有动态属性

```javascript
let name = new String("zcc");		// 这就变成了一个对象，但是行为类似原始值
```



#### 

构造函数的写法和普通的方法一致，额外要求：**方法名和类名一致**

成员变量和成员方法都直接写在构造函数中

在JavaScript中，构造方法其实就是类定义的过程

```javascript
// 在js中
function Blog(message, date){		// 构造方法
    this.message = message;
    this.date = date;
    
    this.toString = function(){			// 就将实例方法当作成员变量即可，尾巴要带分号
        ...
    };
    this.toHTML = function(highlight){		// 带参数
        ...
    };
    this.containsText = function(text){
  		...  
    };
}
Blog.showSignature = function(){			// 类方法，通过 类名.方法名 调用
    ...
}
var blog = [
    new Blog("xxx", "..."),
    new Blog("xxx", "..."),
    new Blog("xxx", "...")];

```

每个JavaScript对象都有一个`toString()`方法

创建**类对象**关键词：==**prototype**==，类变量不一定要初始化，可以后面赋值

1. eg：**`Blog.prototype.signature = "...."`**

   ——一个类一份，从而减少创建对象的开销

2. 还能**扩展对象**，对内置对象也能够扩展类特性和类方法

   eg：**`String.prototype.myFun = function(){...}`**

   （类似于kotlin中的语法）

   调用也需要用`xxx.prototype.myFun()`

   ——注意和类方法的创建进行区分

类方法能访问类变量，但是不能访问实例变量

### 内置对象

#### Date

Date最小粒度为毫秒

==**setMonth()/setYear()**==可以用来设置

==**getDate()/getDay()/getFullYear()**==可以用来获取

eg: 

1. 指定一个日期：`var day = new Date("04/06/2022")`
2. 日期相减：date1-date2，结果是以毫秒为单位的

#### String

1. ==**xxx.length()**==
2. ==**xxx.indexOf("...")**==：判断字符串中是否包含指定的子字符串，如果存在返回首字符的index，如果不存在返回-1
3. ==**xxx.charAt()**==：获取指定index的字符
4. ==**xxx.toLowerCase()/xxx.toUpperCase()**==：字符串字符全部换小（大）写

每个字符串都是一个对象

#### Math

Math是一个组织对象，没有变量，只包含数学相关的公用方法和常量的集合

1. ==**Math.round(xxx)**==：四舍五入
2. ==**Math.floor(xxx)**==：舍去小数部分
3. ==**Math.ceil(xxx)**==：直接进位
4. ==**Math.random(xxx)**==：随机数，[0~1)
5. ==**Math.PI**==

### 选择语句

1. `if...else`和`if.. if else...else`

   if中传入的表达式可以是任意的，可以编译器会调用`Boolean()`进行转换成布尔值

2. ```javascript
   switch(...){
          case xxx: break;
          ...
          default: xxx;
   }
   ```

   JS对switch的功能进行了扩展

   - switch语句可以用于所有数据类型，可以传入字符串、对象等
   - case可以常量、变量、表达式

   eg：

   ```javascript
   switch("helloworld"){		// switch是字符串
       case "hello" + "world": break;	// case是表达式
       case "goodbye": break;
       default:
   }
   ```

   ```javascript
   let num = 25;
   switch (true) {
       case num < 0:
           console.log("Less than 0.");
           break;
       case num >= 0 && num <= 10:
           console.log("Between 0 and 10.");
           break;
       case num > 10 && num <= 20:
           console.log("Between 10 and 20.");
           break;
       default:
           console.log("More than 20.");
   }
   ```

   注意：switch语句在比较每个条件的值时会使用**全等操作符，因此不会强制转换数据类型**（比如，字符串"10"不等于数值10）

### 循环语句

1. `for(var i = 0; i < arr.length; i++){}`

   - for-in语句：`for(property in expression){...}`

     它是一个严格的迭代语句，用来遍历对象中的非符号键属性

     ```javascript
     for(const proName in window){
         alert(proName);
     }
     ```

     ——这边的const不是必须的，**主要是为了保证局部变量不被修改**

     如果for-in循环中的变量是null/undefined，就不会开始循环

   - for-of语句：`for(property of statement){...}`

     用来遍历可迭代对象的元素（数组、序列等）

     ```javascript
     for(const proName of [2,3,4,5]){
         alert(proName);
     }
     ```

2. `while(xxx){}/do{}while(xxx)`

#### 标签语句

 给语句添加标签（类似于单片机中的操作），典型应用场景就是嵌套循环。

eg：

```javascript
start: for(let i = 0; i < 5; i++){
    ...
}
```

一般在continue和break中使用——和break配合使用，就能跳出多重循环

eg：

```javascript
outermost: for(let i = 0; i < 10; i++){
    for(let j = 0; j < 10; j++){
        if(i == 5 && j == 5){
            break outermost;
        }
    }
}
// 这个循环会执行55次

outermost: for(let i = 0; i < 10; i++){
    for(let j = 0; j < 10; j++){
        if(i == 5 && j == 5){
            continue outermost;
        }
    }
}
// 这个循环会执行95次，在i= 5 j = 5之后跳出内层循环，重新开始i = 6 j=0之后的循环
```

### with语句

有点像语法糖

with是将代码作用域设置为特定的对象，主要使用场景：针对一个对象反复使用，用with将代码作用域设置为该对象能较少代码量

eg：

```javascript
let qs = location.search.substring(1);
let hostName = location.hostname;
let url = location.href;

// 上面等价于
with(location){
    let qs = search.substring(1);
    let hostname = hostname;
    let url = href;
}
```

解释器在该代码段中，会先将遇到的变量当作局部变量，看是否有声明；如果没有找到，则认为是location对象的属性，在location对象中找，如果找不到就会报错

——严格模式禁止使用

### 函数

`function 方法名(变量名)`

注意：参数只需要给变量名，不需要给变量类型的

函数可以赋值给变量

eg:

```javascript
var showStatus = function(num){		// 函数字面量（匿名函数）
    alert("cccc");
}

var various = showStatus;
various(123);		// 能够通过变量调用函数
```

注意：函数变量本质上地址引用

## JavaScript内置方法

==**onload="..."**==当网页加载完毕事件发生时，JavaScript就可以用事件响应，调用里面的方法——onload就是一个事件处理器

==**window.load=函数名;**== 等价于上面的，这个代码是JS代码，所以可以保持HTML的功能单一性

如果该函数需要传参，则可以创建匿名函数的方式

```javascript
document.getElementById("seat").onclick = function(param){
    initSeat(param);
}
```

==**onclick="...."**==当点击事件发生时，调用——onclick就是一个事件

==**onchange="..."**==当内容不再关注，且出现变化，调用（不适合验证用户输入，eg：初始时的空表单，用户没有输入，就不会触发里面的验证代码）

==**onresize="..."**==当浏览器窗口尺寸发生变化，调用

==**onfocus="..."**==选择了输入域，说明关注点在这里（光标出现）

==**onblur="..."**==移向下一个目标，说明之前的关注点不再关注了（光标移走了）（适合验证用户输入）

==**onmouseover="..."**==鼠标到了元素上方

==**onmouseout="..."**==鼠标移开了

```html
...
<body onload="alert('hello');">
    ...
</body>
...
```

==**alert()**==内置函数，参数是string

注意：**外面有双引号，里面就只能是单引号（当然可以添加转义字符\来进行替换）**

JavaScript代码可以放在<script>标签中，也可以放在事件处理器中

```html
<html>
	<head>
		<title>iRock</title>
		<script type="text/javascript">
			function touchRock(){			// JavaScript的方法
				var name = prompt("what's your name?", "enter your name here");// prompt负责创建弹出窗口
				if(name){
					alert("nice to meet you, " + name + ".");
					document.getElementById("rockImg").src = "smile.jpg";
				}
			}
		</script>
	</head>
	<body onload="alert('hello i am your pet');">
		<div style="margin-top:100px; text-align: center">
			<img id="rockImg" src="rock.jpg" alt="iRock" onclick="touchRock()" />
		</div>
	</body>
<html>
```

注意：

1. ==**variable = prompt(标题名, 提示语言)**==

   会创建弹窗，将输入的内容作为返回值赋值

   <img src="pic\image-20220330192809116.png" alt="image-20220330192809116" style="zoom:40%;" />

2. ==**document.getElementById("xxx")**==通过标签的id获取结构树上的某个HTML域内容（本质上是个对象），从而修改它的内容、格式等

   如果是img类型，可以获得src

   如果是input类型，可以获得value

   可以获得className（className一般用来统一设置CSS样式，注意和JavaScript的class进行区分）

   `document.getElementById("decision").className = "decisionReverse"`

   <img src="pic\image-20220405212706506.png" alt="image-20220405212706506" style="zoom:50%;" />

   比较：style属性后可以修改少量样式

3. `onclick="touchRock()"`，事件处理器里面可以传入JS的方法，注意一定是被包括在双引号内的

==**parseInt()**==和==**parseFloat()**==能够将文本转换为数字，注意在转换时如果后面出现字符串就转换停止了，只将之前的转换成数字

eg：parseInt("3ert")=3

如果传入的非纯数字，则会返回NaN：`parseFloat($1.45) = NaN`

==**isNaN()判断是否是数值**==：是数值返回false；不是，返回true

==**string.indexOf("xxx")**==，获取xxx在字符串中的第一个出现的首字符下标

eg：`“helloworldwworldhello”.indexOf("world") = 5`

==**setTimeOut("到期后执行的内容", 超时时间)**==

注意:

1. 时间是以毫秒为单位
2. 代码设定的是**经过一段时间**，而不是定一个确定的时间，且是一个**单次定时器**
3. 执行的指令接受的是字符串，所以代码用双引号括起来

eg：`setTimeOut("alert('wake up')", 6000)`

==**setInterval("timer_code", timer_delay)**==间隔定时器，每过指定的时间会重复执行代码，格式同上

注意：

1. 该方法会返回一个值，定时器的id
2. 如果想停止间隔定时器，可以调用==**clearInterval(id)**==，id就是在set的时候的返回值

浏览器关闭，JavaScript代码也停止了，包括定时器也停止了

==**location.reload()**==控制浏览器重新载入

**==document.body.clientHeight==**和==**document.body.clientWidth**==获取浏览器**客户页面的高度和宽度**，客户也就是网页展示的页面，不包括上面的标题栏、工具栏和下面的状态栏

==**document.getElementById("id").style.height**==和==**document.getElementById("id").style.width**==

网页上的所有元素都有一个style对象，从而获取它的宽和高，这个值可以get/set；也可以获得visibility，从而设置该元素是否可见：`visibility = "visible"`和`visibility = "hidden"`

如果要改变图片的尺寸，修改一个参数，另外一个会配合图像比例自动缩放

==**var res = confirm("xxx")**== 出现是否确认的弹窗，点击确认，就会返回true；否则，返回false

<img src="pic\image-20220401153150536.png" alt="image-20220401153150536" style="zoom:50%;" />

## 浏览器给JS开放的权限

1. 浏览器的度量单位，能够获得各种尺寸数据，eg：网页大小、可视网页范围、制造商和版本编号等
2. 浏览器历史记录
3. 定时器，指定过一段事件后触发某段JS代码
4. cookie（缓存的数据）

## cookie

能够较为长期的保存数据，只能存储少量数据	

cookie可以独一无二的名称存储一段数据，且可以设置有效日期，到期就销毁cookie；如果没有设置有效日期，浏览器关闭就消失了

cookie以**长字符串形式存储**在用户计算机中，字符串的内容和网址相关，cookie之间通过`;`间隔，而读取cookie就是通过分号区分的

```javascript
function writeCookie(name, value, days){		// 参数不需要写类型	
    var expire = "";		
    if(days){
        var date = new Date();
        date.setTime(date.getTime() + (days * 24 * 60 * 60 * 1000));		// 设置到期时间
        expire = "; expires=" + date.toGMTString();		// name=...; expires=...; path=...
    }
    document.cookie = name + "=" + value + expire + "; path=/";
}
function readCookie(name){
    var searchName = name + "=";
    var cookies = document.cookie.split(";");
    for(var i = 0; i < cookie.length; i++){
        var c = cookies[i];
        while(c.charAt(0) == ' ')		// 去掉前面的空格
            c = c.substring(1, c.length);
        if(c.indexOf(searchName) == 0){		// 遍历找到第一个匹配的首字母索引
            return c.substring(searchName.length, c.length);
        }
    }
    return null;
}
function earseCookie(name){
    writeCookie(name, "", -1);
}
```

注意：如果在本地需要存储cookie，需要开启服务。可能是因为在服务器或者本地的服务器中的时候，才能使用cookie存储（几年前搞过，完全忘了），毕竟官网给的cookie存储和获取方法就是如此的简单粗暴

VSCode里面安装一个Live Server

<img src="pic\image-20220401131813575.png" alt="image-20220401131813575" style="zoom:50%;" />

<img src="pic\image-20220401131842445.png" alt="image-20220401131842445" style="zoom:80%;" />

点击go live按钮，就会开启服务

想要打开本地网页，需要输入网址`http://127.0.0.1:5500/rock.html`

这个时候就会发现能开启cookie服务了（而不是你浏览器的问题）

一些细节：

1. cookie的存储位置，不一定是使用本地硬盘
2. cookie的名字在指定的网页中唯一即可，对于不同网页的cookie，名字可以重复。因为cookie在存储的时候，会带上网址进行间隔
3. 不同的浏览器之间的cookie是无法共享的

判断浏览器是否支持cookie，==**navigator.cookieEnabled**==

## JavaScript表单

HTML的标签<form></form>，里面的元素是按照键值对的形式存储的

### 创建表单

```html
<head>
	<script type="text/javascript">
            function placeOrder(thisForm){
                alert(thisForm["message"].value);
            }
            function validateMessage(inputField, helpMessage){
                if(inputField.value.length == 0){
                    if(helpMessage != null){
                        helpMessage.innerHTML = "please enter your banner";
                    }
                }else{
                    if(helpMessage != null){
                        helpMessage.innerHTML = "";
                        validateLength(inputField, helpMessage);
                    }
                }
            }
            function validateLength(inputField, helpMessage){
                if(inputField.value.length < 4 || inputField.value.length > 32){
                    if(helpMessage != null){
                        helpMessage.innerHTML = "please control banner in 4 - 32 word";
                    }
                }else{
                    if(helpMessage != null){
                        helpMessage.innerHTML = "";
                    }
                }
            }
    </script>
</head>
<body>
        <form>
            <div class="field">
                "Enter the banner message: "
                <input id="message" name="message" type="text" size="32" 
                       onblur="validateMessage(this, document.getElementById('message_help'));"/>
                <span id="message_help" class="help"></span>
            </div>
            <div class="field">
                "Enter ZIP code of the location: "
                <input id="zipcode" name="zipcode" type="text" size="5" />
                <span id="zipcode_help" class="help"></span>
            </div>
            ...
            <input type="button" value="Order Banner" onclick="placeOrder(this.form);" />
        </form>
    </body>
```

<img src="pic\image-20220402203132788.png" alt="image-20220402203132788" style="zoom:57%;" />

类似如上的形式

注意：

1. 标签关键词==**<form></form>**==

2. 里面可以通过<div>进行段划分，div里面可以增加文字和输入==**<input>**==

3. <input>里面的元素，id可以独一无二标记网页元素，name可以独一无二标记表单中的元素

4. 每个表单域都有一个form对象，所以可以通过this.form获得所有表单里面的元素，这个对象本质是一个数组。可以通过name进行索引，eg：`form["zipcode"]`就能获得里面的值

5. this，就表示当前元素对象，是一个对象类型

6. ==**对象.innerHTML**==能够获取元素的内容，包括里面包含的标签，这边就是<span>属性的内容，能够获得也能够赋值

   ——它主要是设置**内容类元素**，例如div、span、p等装载内容的元素

   ——set就会完全改写里面的所有内容，但是可以串联`innerHTML += "..."`

   ——不是一个标准的万维网的方法

## 正则表达式

JavaScript中最常用的内容——在文本使用方面效果很好——只用在对字符串的匹配上

正则表达式是在字符串中找匹配的地方，即找**子字符串**

规则：

1. 所有正则表达式需要用斜线圈起==**/.../**==

元字符：

1. ==**.**==：可以匹配任何字符，除了换行符
2. ==**\s**==：可以匹配空格、tab、换行符、return/enter
3. ==**\d**==：可以匹配任何数字字符（1个）
4. ==**\w**==：可以匹配任何字母、数字（1个）
5. ==**^**==：字符串的起始，字符串前没有其他文字，eg：`/^A/`会匹配 `An e`，但是不会匹配`an A`
6. ==**$**==：字符串必须以指定模式结束，eg：`/t$/`会匹配`eat`，但是不会匹配`eater`

限定符：

1. ==*****==：限定符前的子模式必须出现0次或者多次（可选，且出现任意次数）
2. ==**{n}**==：限定符前的子模式要出现恰好n次
3. ==**+**==：限定符前的子模式要出现1次或者n次（必现，出现任意次数）
4. ==**?**==：限定符前的子模式出现0次或者1次
5. ==**()**==：归类子模式
6. ==**{min, max}**==：子模式至少出现min次，最多出现max次，多用于限制密码长度、用户名长度
7. **==|==**：要么xx要么xx，eg：`/^\d{2}\/\d{2}\/(\d{2}|\d{4})$/`，日期格式 MM/DD/YY 或者是 MM/DD/YYYY
8. **==[]==**：存储字符类，能够在一组单字符中进行单字符匹配，每一个字符都是可匹配项，eg:`/d[iu]g/`，dug和dig都是可匹配的；`/\$\d[\d.]*/`，$5匹配，\$5.23匹配，\$52.23匹配

转义：

1. 如果关键词作为单纯的字符，则需要转义，**`\$`**类似这样

eg：`/^\d{5}-\d{4}$/`含义就是：xxxxx-xxxx 且限定出现的字符是数字

```javascript
function validateZipCodePattern(inputField, helpMessage){
    var regex = /^\d{5}$/				// 本质上不是字符串，它本质上是对字符串的描述
    if(!regex.test(inputField.value)){		// 匹配失败
        ...
    }else{
        ...
    }
}
```

注意：==**regex.test(string)**== 模式尝试去字符串中匹配，如果匹配失败返回false

eg2：`/^\w+@\w+\.\w{2,3}$/`邮件的规则

eg3：<img src="pic\image-20220405162801501.png" alt="image-20220405162801501" style="zoom:45%;" />（更复杂的邮件规则）

`/^[\w\.-\+]+@[\w-]+(\.\w{2,4})+$/`

## 小tip

1. var和let

   for循环中的超时逻辑

   ```javascript
   for(var i = 0; i < 5; i++){
       setTimeout(() => console.log(i), 0);
   }
   // 输出：5,5,5,5,5
   
   for(let i = 0; i < 5; i++){
       setTimeout(() => console.log(i), 0);
   }
   // 输出：0，1，2，3，4
   ```

   第一个迭代变量声明的是var，所以它保存的是退出循环后的值5，所有超时逻辑拿到的值就是5

   第二个是let，**JavaScript引擎在后台会为每个迭代循环声明一个新的迭代变量**，所以console.log就是每次保存的不同变量实例
   
2. typeof和instanceof

   typeof最适合用来判断变量是否是基本数据类型，但是对引用数据类型作用不大，一般都返回object

   ```javascript
   console.log(typeof a);		// 得到的是字符串类型的 "string"
   ```

   instanceof是通过给定变量名和类型，判断是否给定类型的实例，大部分用于引用类型判断上

   ```javascript
   console.log(a instanceof Object);		// true
   ```

   由于所有引用值都是Object的实例，所以，`对象实例 instanceof Object`都是true，但是如果是基本数据类型，就均返回false

   此外：typeof操作符在用于检测函数时也会返回"function"；typeof对正则表达式也返回"function"（在IE和Firefox中，typeof对正则表达式返回"object"）

   

3. 





