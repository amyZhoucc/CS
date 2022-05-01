# DOM

文档对象模型，它提供了一组完整的、全面控制网页结构和内容的对象，可以给JavaScript调用

——单纯的JavaScript只包含语法，不能够对网页进行很好的控制

**DOM本质上是一棵树**，是符合万维网标准的HTML操作方式

## D、O、M分解

1. D：document，当创建了一个网页并把它加载到Web浏览器中时，DOM就在幕后悄然而生。**它把你编写的网页文档转换为一个文档对象**。

2. O：object，包括用户定义的对象、内建对象、**宿主对象**，eg：window对象，它对应浏览器窗口本身

3. M：model，浏览器提供了网页的模型，我们可以通过JavaScript来访问该模型，DOM把文档表现为家谱树，家谱树就是一种模型

   <img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20220417100632241.png" alt="image-20220417100632241" style="zoom:67%;" />
   
   更详细的：
   
   ```html
   <div id="testdiv">
       <p>This is <em>my</em> content.</p>
   </div>
   ```
   
   包含了元素节点、属性节点、文本节点
   
   <img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20220422132208287.png" alt="image-20220422132208287" style="zoom:80%;" />

## 节点

节点树上面挂满了节点，节点之间类型不一样

### 元素节点

DOM的单位是元素节点，标签名字就是元素名字

### 文本节点

就是元素内包含的内容，它存在于元素节点的内部

eg：<p>元素包含着文本“Don't forget to buy thisstuff.”——就是文本节点

### 属性节点

用来对元素做出更具体的描述

eg：title属性——就是一个属性节点

<img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20220417103357921.png" alt="image-20220417103357921" style="zoom:67%;" />

## DOM提供的方法

DOM提供的方法并不专属于JavaScript，而是对于支持DOM的任何语言都可以使用

它的用途也不局限于网页处理，还能用来处理任何标记语言编写出来的文档，eg：XML

DOM不仅可以获取文档内容，还能改变文档内容——但是不改变文档的物理内容

### 获取元素

DOM提供了各种方法来访问和控制里面的元素

最常用的：

1. ==**`getElementById("...")`**==：返回一个对象类型

2. ==**`getElementsByTagName("...")`**==：参数是给定标签的元素，会返回一个数组，里面每个元素都是对象

   并且允许把一个通配符`*`作为它的参数，它就会将所有的元素都放在数组中返回——可以通过arr.length知道页面元素中的总数

3. ==**`getElementsByClassName("...")`**==：通过class属性中的类名来访问元素；使用这个方法还可以查找那些带有多个类名的元素，用空格分隔

   eg: `getElementByClassName("important sale")`

1、2可以配合使用，

```javascript
let item = document.getElementById("shopping");		// 获得id="shopping"对应的元素对象
let childItems = item.getElementByTagName("li");	// 获取所有无序清单里的元素
```

### 获取和设置属性

方法不属于document对象，所以不能通过document对象调用。**它只能通过元素节点对象调用**

1. ==**`object.getAttribute()`**==，得到元素的各个属性

2. ==**`object.setAttribute(属性名, 属性值)`**==，设置对应的属性

   注意：做出的修改不会反映在文档本身的源代码里。这种“表里不一”的现象源自DOM的工作模式：**先加载文档的静态内容，再动态刷新，动态刷新不影响文档的静态内容**

   特别的：`element.title = "xxx"`，这也是可以进行设置属性的，但是前者的可移植性更好，且能修改文档中的任何一个元素的任一属性

   ——**严格遵守“第1级DOM”能够让你避免与兼容性有关的任何问题**

### 动态创建标记方法

这个主要用来改变DOM的结构，通过JavaScript代码来动态修改

——注意，**使用DOM将重要的内容添加到网页上，是一种滥用**，缺乏平稳退化

#### 旧方法：document.write 和 innerHTML（HTML专属方法）

浏览器在呈现这种XHTML文档时根本不会执行document.write方法和innerHTML方法

```html
<body>
	<script>
    	document.write("<p>this is js inserted</p>");		// 动态插入的
    </script>
</body>
```

缺点：

1. **违背行为和表现分离原则**，必须要在<body>中显式调用JS的代码才行
2. 容易导致验证错误，eg：“<p>”很容易被误认为是<p>标签

innerHTML是不关注细节的，如果要关注细节，就需要使用DOM

当你需要将一大段HTML内容插入网页时，innerHTML属性更适用。它既支持读取，又支持写入。

eg：

```javascript
var parent = document.getElementById("testdiv");
alert(parent.innerHTML);
parent.innerHTML = "<p>This is <em>new</em> content</p>"
```

注意：**一旦使用了innerHTML属性，它的全部内容都将被替换。**

#### DOM提供的方法

##### 添加新的元素

1. 创建新的元素：==**var decisionElem = document.createElement("p")**==
2. 增加内容，然后将文本节点挂载在元素节点下面：==**decisionElem.appendChild(document.createTextNode("xxxx"))**==
3. 将新的元素挂到指定的元素下：==**document.getElementById("history").appendChild(decisionElem)**==

##### 在元素前插入一个元素insertBefore()

==**`父节点.insertBefore(新元素节点, 目标元素节点)`**==

注意：**父节点必须是元素节点**，而不能是属性节点 / 文本节点

eg：`placeholder.parentNode.insertBefore(description, placeholder);`

但是没有insertAfter方法，但是可以自行创建一个

```javascript
function insertAfter(newElement, targetElement){
    let parent = targetElement.parentNode;
    // 目标节点是最后一个孩子节点，插入在它之后就是最后一个节点（targetElement.nextSibling不存在，所以要区分）
    if(targetElement == parent.lastChild){	
        parent.appendChild(newElement);
    }else{
        parent.insertBefore(newElement, targetElement.nextSibling);
    }
}
```



### 节点属性

1. ==**nodeValue**==：存储节点的值，只能访问文本和属性节点，不包含元素

   注意：如果尝试获取元素的nodeValue返回的是null，如果是元素就要尝试获取其孩子节点，然后获取其孩子节点，eg：<p>

   element.firstChild.nodeValue

2. ==**nodeType**==：节点类型，例如获取TEXT类型、DOCUMENT类型，但是返回的是代号，

   - 1：元素节点
   - 2：属性节点
   - 3：文本节点

3. ==**nodeName**==：节点的名字，就是标签的名字，是string类型的

4. ==**parentNode**==：节点的父结点

3. ==**childNodes**==：包含该元素的所有子节点的**数组**，以出现在HTML中的顺序排列

   `bodyElement.childNodes.length`就能求出某个元素的子节点个数

4. ==**firstChild**==：节点下的第一个节点

7. ==**lastChild**==：节点下的最后一个节点

5. ==**nextSibling**==：节点的下一个兄弟节点

<img src="pic\image-20220405205905415.png" alt="image-20220405205905415" style="zoom:50%;" />

里面的内容是story元素的第一个孩子（唯一一个）

### 安全改变节点文本

1. 移除该节点下的所有子节点 ==**removeChild()**==的一个for循环删除所有
2. 创建新的**文本**节点 ==**createTextNode()**==
3. 将新创建的本文节点放在节点下 ==**appendChild()**==

```javascript
var node = document.getElementById("story");
while(node.firstChild){			// 存在孩子节点
    node.removeChild(node.firstChild());
}
node.appendChild(createTextNode("ok, come on"));	
```

## HTML-DOM

HTML-DOM还提供了更多的方法，用来管理网页。

注意：**只能用来处理Web文档**。如果你打算用DOM处理其他类型的文档，请千万注意这一点

1. `document.getElementsByTagName("form")` === **`document.forms`**==
2. `element.getAttribute("src")` === **`element.src`**==
3. `element.setAttribute("src", "xxx")` === **`element.src = "xxx"`**==

DOM-CORE更通用，HTML-DOM的代码更为简化



# 实例1

利用JavaScript和DOM创建一个图片库，图片库的图片量较大，如果一次性下载所有的图片量大，会造成页面卡顿，所以图片是按需加载

目标：

1. 把整个图片库的浏览链接集中安排在图片库主页里，只在用户点击了这个主页里的某个图片链接时才把相应的图片传送给他
2. 点击链接，不会跳转到网址页面，而是在当前网页，并且能看到图片和其他的序列

策略：

1. 占位符，用来进行图片展示
2. 点击某个链接，拦截网页的默认行为

```html
<html>
    <head>
        <title>隔离云看花</title>
        <script src="showPic.js"></script>	<!--注意不能写成<script ... /> -->
    </head>
    <body>
        <h1>找春天</h1>
        <ul>
            <li>
                <a href="http:..." title="sunflower" onclick="showPic(this); return false;">向日葵</a>
            </li>
            <li>
                <a href="..." title="starflower" onclick="showPic(this); return false;">星星花</a>
            </li>
            <li>
                <a href="..." title="rose" onclick="showPic(this); return false;">玫瑰花</a>
            </li>
        </ul>
        <img id="placeholder" src= "..." alt="春天的图片库" width="800px"/>
        <p id="description">我的花</p>
    </body>
</html>
```

```javascript
function showPic(element){
    showTitle(element);
    let href = element.getAttribute("href");    // 获取网页链接
    let placeholder = document.getElementById("placeholder");
    placeholder.setAttribute("src", href);
}
function showBodyChildren(){
    let bodyElement = document.getElementsByTagName("body")[0];
    alert(bodyElement.childNodes.length);
}
function showTitle(element){
    let title = element.getAttribute("title");
    let description = document.getElementById("description");
    description.firstChild.nodeValue = title;
}
window.onload = showBodyChildren();
```

需要注意的内容：

1. 事件处理函数，eg：onclick，处理函数所触发的JavaScript代码返回布尔值true或false，所以当点击链接，**如果返回true，表示该点击事件产生了**；**返回false，表示该点击事件没有发生，所以这个链接的默认行为没有被触发，可以防止用户被带到目标链接窗口**

## 改进版

接下来是根据《JavaScript概览》中的“JavaScript的实战小策略”进行的改进

1. 平稳退化：可以发现即使不支持JavaScript、禁用，但是超链接中的`href`带着真实的属性，能够支持图片正常出现，只不过出现了页面跳转

2. 分离JS：JavaScript代码和HTML没有做到分离

   **之前都是将JavaScript代码嵌入到onclick中去执行的**——**变成在外部文件中完成添加onclick事件处理函数，HTML代码无杂质**

   eg：给<a>标签加一个class属性，然后JS代码根据`getElementsByTagName`和`getAttribute("class")`去进行查找

   ——每个a都添加一个，太麻烦

   ——由于<a>里面的表现均一致，就是将链接href里的图片加载到下面的<img>标签上（代码后面放上），且存在在列表中，可以获取列表头，然后在列表节点中查找所有<a>标签节点

   ——这个就是挂钩<ul id="xxx,">

3. 对象检测：代码中用到了`getElementById`等，但是没有对他们进行支持性的检测

```html
<html>
    <head>
        <title>隔离云看花</title>
        <link rel="stylesheet" href="layout.css" media="screen">
    </head>
    <body id="body">
        <h1>找春天</h1>
        <ul id="imageGallery">		// 给ul设置了一个id
            <li>
                <a href="http://xxx" title="sunflower">向日葵</a>	// a里面的onclick事件取消了
            </li>
            <li>
                <a href="https://xxx" title="starflower">星星花</a>
            </li>
            <li>
                <a href="https://xxx" title="rose">玫瑰花</a>
            </li>
        </ul>
        <img id="placeholder" src= "http://xxx" alt="春天的图片库" width="800px"/>
        <p id="description">我的花</p>
        <script src="showPic.js"></script>
    </body>
</html>
```

```javascript
// version2
// 根据分离原则
function prepareGallery(){
    // 对象检测，判断是否支持方法、是否存在元素节点
    if(!document.getElementById) return false;
    if(!document.getElementsByTagName) return false;        // 判断是否支持这两个方法
    if(!document.getElementById("imageGallery")) return false;   // 判断是否有该元素节点（后面需要使用，防止获取不到，缺使用而出错）
    let gallery = document.getElementById("imageGallery");
    let list = gallery.getElementsByTagName("a");		// 找到所有超链接标签
    for(let i = 0; i < list.length; i++){	
        list[i].onclick = function(){		// 给超链接标签注册点击事件处理函数
            return showPic(this);
        }
    }
}

// 这边的逻辑是，只要有一个不存在，就会返回false，那么就会触发链接跳转
function showPic(element){
    if(!document.getElementById("placeholder")) return false;	// 判断元素是否存在
    let href = element.getAttribute("href");    // 获取网页链接
    let placeholder = document.getElementById("placeholder");
    placeholder.setAttribute("src", href);

    if(!document.getElementById("description")) return false;	// 判断元素是否存在
    let title = element.getAttribute("title");
    let description = document.getElementById("description");
    description.firstChild.nodeValue = title;
    return true;		// 最后返回true，表示用JS代码成功的实现了图片的替换
}
// 这边的逻辑是，placeholder不存在，会返回false触发链接跳转；description不存在，返回true，只不过不显示而已
function showPic(element){
    if(!document.getElementById("placeholder")) return false;	// 判断元素是否存在
    let href = element.getAttribute("href");    // 获取网页链接
    let placeholder = document.getElementById("placeholder");
    placeholder.setAttribute("src", href);

    if(document.getElementById("description")){
    	let title = element.getAttribute("title");
        let description = document.getElementById("description");
        description.firstChild.nodeValue = title;
    }
    return true;		// 最后返回true，表示用JS代码成功的实现了图片的替换
}
```

注意：

1. `if(!document.getElementById("imageGallery"))`的必要性，如果后面HTML代码将整个列表均删除了，那么也不用担心该JS代码会出错

   因为后面会直接使用它，如果获取失败，返回的是null，对null的操作必然会导致程序奔溃

   ——**如果想用JavaScript给某个网页添加一些行为，不应该让JavaScript代码对这个网页的结构有任何依赖。**

2. 方法需要执行才能被执行，如果在HTML加载完毕之前，DOM不完整，可能会在某个判断时直接返回false——需要在网页加载完毕之后立即执行——在onload中执行

   问题：如果有多个方法需要在onload中执行

   ——可以包裹一个匿名方法，然后将多个方法都放在匿名方法中，然后匿名方法绑定到onload事件：

   ```javascript
   window.onload = function(){
       showBodyChildren();
       prepareGallery();
   }
   ```

   还有一个弹性最佳的解决方案：

   ```javascript
   // 将方法加载到onload的通用方法
   function addLoadEvent(fun){
       let oldload = window.onload;
       if(typeof oldload != "function"){       // 首次绑定方法
           window.onload = fun;
       }else{
           window.onload = function(){
               oldload();
               fun();
           }
       }
   }
   addLoadEvent(showBodyChildren);
   addLoadEvent(prepareGallery);
   ```

   ——这个方法只需要定义一次，之后只需要调用即可

   尤其是在代码变得越来越复杂的时候。无论打算在页面加载完毕时执行多少个函数，只要多写一条语句就可以安排好一切

——正常情况下，JS代码不用对HTML代码进行过多的假设，抓住关键的进行验证，其他的需要实现设计好

# 实例2

背景：标签很多都带属性，属性节点也是DOM树的节点之一。

大多数属性的内容在浏览器中不会显示，只有极少数会显示。但是**问题是：不同浏览器呈现属性的方式多种多样**

eg：

1. title属性：一些浏览器显示为弹出式的提示框，另一些浏览器则会把它们显示在状态栏里
2. alt属性：一些浏览器显示为弹出式的提示框，这就是对alt属性的滥用

——利用DOM编程，把这种控制权重新掌握在自己的手里

——获取属性内容 -> 封装这些内容信息 -> 将内容标记插入文档

