# 概览

主要知识来自于《JavaScript高级程序设计》（edition 4）

主要是对JavaScript的构成和关键点进行一个梳理，以及如何在前端代码中引入JavaScript，算是一个准备工作（主要对应书中的chap1、chap2）

ps：红宝书真的太详细了，有些知识点只能说作为了解，看过一遍就忘了:sweat: ，用到的时候再行查阅

## 1. JavaScript初始作用

代替Perl等服务器语言提前对表单进行验证，在客户端进行验证，从而避免用户每次非法的输入都需要与服务端的一次往返通信

## 2. JavaScript的实现

<img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20220407165342089.png" alt="image-20220407165342089" style="zoom:50%;" />

JavaScript包含了核心的语法：

1. ECMAScript，它不仅用于浏览器，只是该语言的基准

2. 文档对象模型DOM：本质上是一个API，可以在HTML中使用扩展的XML。DOM将整个页面抽象成一组分层节点（其实就是构建 了一棵HTML树），每个HTML的组成成分都会对应一个节点，开发者可以根据DOM提供的API控制网页的内容和结构，轻松的增、删、改、查

   ```html
   <html>
       <head>
           <title>Sample Page</title>
       </head>
       <body>
           <p> Hello World! </p>
       </body>
   </html>
   ```

   <img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20220407170836037.png" alt="image-20220407170836037" style="zoom:50%;" />

   在历史开发进程中，有不同的DOM Level，可以看成不同的版本迭代，每一代都在原来的基础上增加了功能

3. 浏览器对象模型BOM

   同DOM类似，也是提供了浏览器相关的API，能够支持访问和操作浏览器窗口，能够让开发者操作浏览器展示页面外的部分。

   HTML5能够较为统一的涵盖尽可能多的BOM（不同浏览器有不同的支持，可以使用公共的API）

## 3. 如何使用script

总结：通过<script>标签将JS代码引入到HTML中，已经成为规范，标签有8个属性：


1. async：异步执行，立即下载脚本，且不会阻塞页面的其他动作，**只对外部文件有效**

   标记为async的脚本并不保证能按照它们出现的次序执行——所以，如果有多个该脚本，脚本之间不能存在依赖，且异步脚本不能在加载期间修改DOM

2. charset：设定字符集，浏览器一般忽略设置的值

3. crossorigin

4. defer：脚本延迟到文档完全被解析和显示之后再执行，**只对外部文件有效**，但是脚本还是会下载完的，只不过不执行

   **推迟执行的脚本不一定总会按顺序执行**，所以为了明确顺序，最好只包含一个推迟执行的脚本

5. integrity

6. **src**：指定导入外部文件

3. type：又称MIME类型，用来替换被废弃的language，表示代码中脚本语言的内容类型，默认是`text/javascript`（虽然这种写法也是被废弃了）

JavaScript代码可以：

1. 直接写在HTML代码中，通过标签<script></script>围起来

   注意，代码中不要出现字符串`</script>`，在JS解释器进行分析时，通过找到该字符串，判断JS代码已经结束，从而使后面的代码报错。如果一定要出现可以进行转义：==**`<\/script>`**==

2. 通过src将外部的JS代码导入

   - **在解释外部文件时，页面会阻塞**，如果代码需要通过网络下载，则还包括下载的时间——串行解释执行
   - 导入的代码不一定要.js后缀，浏览器不会检查后缀，它还能接受一些扩展语言的代码，如TypeScript，或React的JSX。但是要保证**服务器返回正确的MIME类型**
   - src还可以接受完整的URL代码，浏览器解析时会向该URL发送一个GET请求，从而获得相应的资源——注意如果需要外部资源，要确保来源可信

——如果已经选择从外部导入文件（2），那么不能再这个标签中加入其他代码（1），即**一个<script>标签的功能只能二选一**，如果两者都提供，那么只会下载外部文件。

**浏览器会按照<script>标签在页面中出现的顺序依次解释**，第一个解释完才能解释第二个（前提是没有使用defer、async）

特别的：<script>标签如果放在<head>中，需要将里面所有的代码全部执行完毕，才能执行到<body>去渲染页面，所以如果<script>内容很多，则会延迟渲染，页面出现长时间的空白——**现代Web应用程序通常将所有JavaScript引用放在<body>元素中的页面内容后面**（这个也是最常见的写法，现在终于知道原因了）

```html
<! DOCTYPE html>
<html>
    <head>
        ...
    </head>
    <body>
        <! -- 这里是页面内容 -->
        <! -- js的代码都写在body里面，html代码之后 -->
        <script src="example1.js"></script>	
        <script src="example2.js"></script>
    </body>
</html>
```

## 4. 动态加载脚本

JS可以使用DOM API修改页面结构，所以**可以向DOM添加script元素实现动态加载**

```javascript
let script = document.createElement("script");
script.src = "example.js";
document.head.appendChild(script);
```

这段代码只有执行到才会发起加载脚本请求，以这种方式创建的脚本是以异步方式加载的，等同于添加async

——问题：DOM API是都支持了，不是所有浏览器都支持async属性——**可以明确将其设置为同步加载**

`script.async=false;`

潜在的问题：以这种方式获取的资源对浏览器预加载器是不可见的。这会严重影响它们在资源获取队列中的优先级，这种方式可能会**严重影响性能**

解决方法：让预加载器知道这些动态请求文件的存在

`<link rel="preload" href="example.js">`

总结：推荐将JavaScript代码单独写在一个文件中，这样有较高的维护性；浏览器会对外部的JavaScript文件进行缓存页面能够加载更快（如果不同页面同时用到一个文件，则只需要下载一次）；能够适应之后的扩展

## 5. 不支持JavaScript时

不支持（更多的是禁止）脚本，出现了<noscript>，在不支持JavaScript的页面上提供替代内容。

当出现：

1. 浏览器不支持脚本
2. 浏览器支持脚本，但功能被关闭

在<noscript>中的内容就会被渲染，没有满足上面的任一条件，就不会渲染

# JavaScript的实战小策略

与HTML语言相比，JavaScript语言的生存环境的要求要苛刻得多。

浏览器在遇到不符合语法规定的的HTML代码时，还会努力将其呈现出来；而，JavaScript代码不符合语言规范时，浏览器的解释器将拒绝继续执行下去，并且在console上面报错

弹出的广告窗口和内容覆盖是一个典型的滥用JavaScript的例子

## 平稳退化⭐

存在一种问题：浏览器不支持JavaScript；浏览器支持但是被用户禁用了JavaScript

——需要能保证某个功能无法使用，但是用户仍能够完成基本操作

eg：用户点击链接时出现一个弹窗（例如支付界面等）

（ps：应该只在绝对必要的情况下才使用弹出窗口，因为这将牵涉到网页的可访问性问题，所以**如果网页上的某个链接将弹出新窗口，最好在这个链接本身的文字中予以说明**）

打开新窗口：`window.open(url, name, features)`

可以传递三个参数，均是非必须的：

1. url：待打开的网页的url，如果省略，那么将会弹出一个空白窗口

2. name：新窗口名字，那么可以用这个名字和新窗口进行通信

3. features：一个字符串，内部用逗号分隔，主要是设置新窗口的各种属性，eg：窗口的大小、新窗口被启用或禁用的各种浏览功能（工具条、菜单条、初始显示位置等）

   ——新窗口的浏览功能要少而精

eg：

```javascript
function popUp(winURL){
    window.open(winURL, "popup", "width=320, height=480");
}
```

调用popUp函数有两种方法：

1. **JavaScript伪协议**：通过一个链接来调用JavaScript函数——类似于http协议的这种

   `<a href="javascript:popUp('http://www.baidu.com');">click</a>`

   存在的问题：

   - 在支持“javascript:”伪协议的浏览器中运行正常
   - 较老的浏览器则会去尝试打开那个链接但失败
   - 支持这种伪协议但禁用了JavaScript功能的浏览器会什么也不做

   ——本来有的功能减少

2. 事件处理函数

   `<a href="#" onclick="popUp(http://www.baidu.com); return false;">click</a>`

   存在的问题：不能平稳退化。如果用户已经禁用了浏览器的JavaScript功能，这样的链接将毫无用处。

3. 让链接地址真实存在

   `<a href="http://www.baidu.com" onclick="popUp(this.href); return false;">click</a>`

   即使JavaScript已被禁用（或遇到了搜索机），这个链接也是可用的，功能还是完善的，虽然效果差一点

## 渐进增强⭐

渐进增强：用一些额外的信息层去包裹原始数据。按照“渐进增强”原则创建出来的网页几乎都符合“平稳退化”原则。

eg：CSS就是控制内容的显示效果，CSS指令就构成了一个表示层。如果CSS失效，页面内容也还能正常显示

## 分离JavaScript

可以不在HTML代码中加入事件处理函数

```javascript
// html中的代码
<a href="http://www.baidu.com" class="popup" />

// javasript中对应处理的代码，给onload调用
window.onload = prepareLinks();
function prepareLinks(){
    var links = document.getElementsByTagName("a");		// 获取所有带链接的元素对象
	for(let i = 0; i < links.length; i++){
        if(links[i].getAttribute("class") == "popup"){		// 判断class是否匹配
            links[i].onclick = function(){				// 创建事件处理函数
                popUp(links[i].getAttribute("href"));
                return false;
            }
        }
    }
}

```

——这样就实现了JavaScript和HTML代码的较为完整分离

## 向后兼容

不同的浏览器对JavaScript的支持程度也不一样，所以有些浏览器支持JavaScript脚本，但是某些脚本不一定能顺利运行

### 对象检测

在使用对象检测时，**一定要删掉方法名后面的圆括号**，如果不删掉，测试的将是方法的结果，无论方法是否存在。

eg:

下面要涉及到这两个方法，如果不支持就直接返回

```javascript
if(!document.getElementById || !document.getElementsByTagName){	
    return false;
}
```

### 浏览器嗅探技术

浏览器嗅探”指通过提取浏览器供应商提供的信息来解决向后兼容问题，理论上，可以通过JavaScript代码检索关于浏览器品牌和版本的信息。但是：

1. 浏览器有时会“撒谎”
2. 为了适用于多种不同的浏览器，浏览器嗅探脚本会变得越来越复杂
3. 许多浏览器嗅探脚本在进行这类测试时要求浏览器的版本号必须得到精确的匹配。因此，每当市场上出现新版本时，就不得不修改这些脚本。

——所以相较之下，还是对象检测更为有效

## 性能考虑

### 1. 尽量少访问DOM和尽量减少标记

访问DOM的脚本是十分耗时的，很大影响了脚本的性能表现。	

——较好的方法是将搜索的结果保存在一个变量中，下一次需要就直接拿来用，而不需要二次访问DOM

### 2. 合并和放置脚本

如果是访问外部脚本，因为可能涉及到网络请求下载脚本。最好将多个脚本并到一个脚本文件中——减少加载页面时发送的请求数量。因为浏览器对JavaScript脚本的下载和解释是串行执行的，前一个没有下载执行完，下一个无法进行

### 3. 压缩脚本

把脚本文件中不必要的字节，如空格和注释，统统删除，从而达到“压缩”文件的目的——这时候分号的重要性就出来了

——有很多工具可以实现，有的精简程序甚至会重写你的部分代码，使用更短的变量名，从而减少整体文件大小。

通常，为了与非精简版本区分开，**最好在精简副本的文件名中加上min字样**

# HTML版本的选择

HTML/XHTML

更优选择是XHTML，它会对标记有更严格的要求，例如：

1. XHTML要求所有标签名和属性名都必须严格小写
2. XHTML要求标签必须严格配对，即使是孤立标签，最后也要加上 `/>`

但是文档开头的设置较为长：

<img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20220425143046379.png" alt="image-20220425143046379" style="zoom:67%;" />

使用HTML5更为简单：`<!DOCTYPE html>`
