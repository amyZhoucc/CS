# Ajax

主要用来**异步加载页面**内容的技术，那么服务器在处理请求的时候，页面还能够正常被浏览和交互

并且使用Ajax就可以做到**只更新页面中的一小部分**。其他内容——标志、导航、头部、脚部，都不用重新加载

Ajax依赖JavaScript

Ajax技术的核心就是**XMLHttpRequest对象**

XMLHttpRequest对象：是中间人角色，用来沟通客户端和服务器端的交互。之前的请求都是由浏览器发出，有了该对象之后，JavaScript能够发出请求，并且自己处理响应。

存在的问题：不同浏览器实现XMLHttpRequest对象的方式不太一样。为了保证跨浏览器，你不得不为同样的事情写不同的代码分支。

## 1. 获取XMLHTTPObject对象

微软最早在IE5中以ActiveX对象的形式实现了一个名叫XMLHTTP的对象。

为了适应不同的IE版本，所以要进行适配。

Chorme可以用XMLHTTPObject方法去获取对象

```javascript
function getHTTPObject(){
    if(typeof XMLHttpRequest == "undefined"){
        XMLHttpRequest = function(){
            try{
                return new ActiveXObject("Msxml2.XMLHTTP.6.0");
            }catch(e){}
            try{
                return new ActiveXObject("Msxml2.XMLHTTP.3.0");
            }catch(e){}
            try{
                return new ActiveXObject("Msxml2.XMLHTTP");
            }catch(e){}
            return false;
        }
    }
    return new XMLHttpRequest();
}
```

## 2. 使用XMLHTTPObject对象

XMLHttpRequest对象有许多的方法。其中最有用的是open方法，它用来指定服务器上将要访问的文件：

1. 参数1：指定请求类型：GET、POST或SEND
2. 参数2：访问的地址
3. 参数3：true：以异步方式发送和处理；false：同步方式，则会出现阻塞

eg:

```javascript
function getNewContent(){
    let request = getHTTPObject();
    if(request){
        request.open("GET", "example.txt", true);	// 调用open方法
        request.onreadystatechange = function(){		// 收到响应之后的处理函数
            if(request.readyState == 4){
                let para = document.createElement("p");
                let content = document.createTextNode(request.responseText);
                para.appendChild(content);
                document.getElementById("new").appendChild(para);
            }
        };
        request.send(null);		// 发送请求
    }else{
        alert("your browser don't support XMLHTTPRequest");
    }
}
addLoadEvent(getNewContent);
```

理解：

1. `onreadystatechange`是一个事件处理函数，它在服务器给对象返回响应的时候被触发执行。可以根据服务器的具体响应做出对应的处理

2. 浏览器会在不同阶段更新readyState属性的值，它有5个可能的值：

   - 0：未初始化
   - 1：正在加载
   - 2：加载完毕
   - 3：正在交互
   - 4：完成

   ——只要readyState属性的值变成了4，就可以访问服务器发送回来的数据了。

3. 访问服务器发送回来的数据要通过两个属性完成：

   1. responseText属性，这个属性用于保存文本字符串形式的数据。
   2. responseXML属性，用于保存Content-Type头部中指定为"text/xml"的数据，其实是一个DocumentFragment对象

注意：

1. 在使用Ajax时，**千万要注意同源策略**。使用XMLHttpRequest对象发送的请求只能访问与其所在的HTML处于同一个域中的数据，不能向其他域发送请求。有些浏览器还会限制Ajax请求使用的协议

   eg：chrome就会限制file://协议从自己的硬盘里加载example.txt文件，所以必须开启服务器，才能正确执行上面的代码，否则会报错：**Cross origin requests are only supported for HTTP”（跨域请求只支持HTTP协议）**——经常遇到的错误

## Ajax的平稳退化

首先构建不使用ajax的代码——例如直接利用浏览器发送表单

再构建使用ajax的代码

那么，即使浏览器不支持Ajax，也能够实现基本功能