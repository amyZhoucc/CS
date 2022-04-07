# DOM

文档对象模型，它提供了一组完整的、全面控制网页结构和内容的对象，可以给JavaScript调用

——单纯的JavaScript只包含语法，不能够对网页进行很好的控制

DOM本质上是一棵树，是符合万维网标准的HTML操作方式

DOM提供了各种方法来访问和控制里面的元素，eg：`getElementById("...")`和`getElementsByTagName("...")`

DOM还有很多特性：

1. ==**nodeValue**==：存储节点的值，只能访问文本和属性节点，不包含元素
2. ==**nodeType**==：节点类型，例如获取TEXT类型、DOCUMENT类型，但是返回的是代号，TEXT是3，ELEMENT是1
3. ==**childNodes**==：包含所有子节点的**数组**，以出现在HTML中的顺序排列
4. ==**firstChild**==：节点下的第一个节点
5. ==**lastChild**==：节点下的最后一个节点

<img src="pic\image-20220405205905415.png" alt="image-20220405205905415" style="zoom:50%;" />

里面的内容是story元素的第一个孩子（唯一一个）

## 安全改变节点文本

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

## 添加新的元素

1. 创建新的元素：==**var decisionElem = document.createElement("p")**==
2. 增加内容：==**decisionElem.appendChild(document.createTextNode("xxxx"))**==
3. 将新的元素挂到指定的元素下：==**document.getElementById("history").appendChild(decisionElem)**==