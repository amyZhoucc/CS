

# JavaScript基础语法

主要内容来自于《head first JavaScript》（最后一章和Ajax相关的过了一遍，但是不是很理解，所以没有写上去:sweat_smile:打算后面看《JavaScript DOM编程艺术》和《JavaScript高级程序设计》有更详细的了解之后进行补充，并且后面看这两本书的时候，对简单的必要的知识补充会在这边，如果比较复杂，会直接新开内容）

## JavaScript的性质

JavaScript是脚本解释语言

JavaScript的优势：让原本静态的网页具备**交互性**，根据用户的反馈进行页面变化

**借由响应用户的输入数据，JavaScript给网页带来生命，JavaScript负责响应用户要求网页执行任务**

eg：判断用户的输入是否合法，去查询服务器上面的数据，进行计算

JavaScript、HTML、CSS并称为网页三大元素，HTML是车的骨架、CSS是车漆、JavaScript是轮胎，**只有JavaScript存在，车（网页）才能动起来**

JavaScript一般也直接放在网页中。

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

注意，script标签表示是后面是脚本代码，浏览器可以支持多种脚本语言的代码，但是**`type="text/javascript"`**才是指定脚本语言是JavaScript的

如果JavaScript是用外部文件，则需要导入文件

```html
<script type="text/javascript" src="cookie.js"></script>
```

## JavaScript的语法

JavaScript区分大小写

### 基本数据类型

有3种基本数据类型：

1. **number**，可以是整数也可以是浮点数
2. **boolean**，true/false
3. **string**
4. **undefined**：未初始化
5. **null**

还有其他类型：

1. 描述“没有值”这个状态的值：**`undefined`**
2. 非数字：`NaN`，无法计算出该数字的值，是用来表示数字数据类型有误时的指标，一般是在计算需要数字，但是收到非数值的数据时才会看到NaN

eg：alert里面只能传入string类型，所以如果要输出数字，实际上内部是将它转换为文本之后呈现。特别的如果代码是一个表达式，那么会先计算，eg：`alert(1+3);` 就会输出4

1. **变量`var`**，在使用之前初始化即可

   JS解释器会自行推断变量类型，且赋值修改之后可以改变数据类型

2. **常量`const`**，声明时就需要赋值，且只能赋值一次。有些浏览器不支持（不知道现在支不支持）

重新载入网页，脚本的数据会重置，变到尚未执行时的状态

变量命名：

1. 至少需要有一个字符
2. 第一个字符是字母或者`$`或者`_`
3. 后面的字符是字母、数字、`_`、`$`

默认的命名方式：小写驼峰式，常量全大写

将变量写在方法外面，就是全局变量，它的存活时间和脚本的生命相同

如果在方法里面声明一个变量，但是没有带上关键字var，那么会变成全局变量——不推荐的写法

### 注释

单行注释`//`

多行注释：`/*`开头，`*/`结尾

```javascript
/**
 *			这里面的星是为了提高注释的阅读性
 *
 */
```

### 对象

是复杂数据类型

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

`if...else`和`if.. if else...else`

`switch(...)`

`case "a": ... break;`

### 循环语句

`for(var i = 0; i < arr.length; i++){}`

`while(xxx){}`

### 数组

**`var arr = new Array();`**创建数组

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

注意：外面有双引号，里面就只能是单引号（当然可以添加转义字符\来进行替换）

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

