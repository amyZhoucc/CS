# Markdown语法

## 1. 标题

```
# 标题一     表示一级语法，#后面要带空格，才能使能
## 标题二 .....
```

一共支持6级标题

## 2. 字体

**加粗**

```
**文字**   ——表示加粗，要紧跟文字，否则就不会加粗了，当作普通的星号来处理
```

*斜体*

```
*文字*  ——表示斜体，要紧跟文字
```

***加粗加斜体***

```
***文字***   ——表示斜加粗和斜体
```

~~删除~~

```
~~文字~~   ——表示删除线
```

> 引用

```
>
>>			——引用，支持多级
```

## 3. 分割线

---

***

```
***
---				——表示分割线，用三个及以上的*/-的就可以实现
```

## 4. 图片

```
<img src="" alt="name" style="zoom:40%">			——github只能识别这个
![name](src "name")					——注意有!，没有感叹号变成超链接
```

## 5. 超链接

```
[描述](链接)

[name](#xxx)    <a name="xxx"></a>
```

[baidu](https://www.baidu.com)

## 6. 列表

有序列表：

数字加点就可以了，默认有缩进。

1. 

无序列表：

```
- list1   ——横杠加空格，就变成无序列表
+ list
* list			——效果同上
```

- 

列表嵌套

1. sdgfsde
   - aefsd
2. sdgfgds
   - sfsdsg

## 7. 表格

| 姓名  | 性别 | 学号  |
| :---- | ---: | :---: |
| fvfsd | sfds | sfdsf |

```
|姓名|性别|学号|
|:--|:--:|--:|			——左边冒号表示左对齐；右边冒号表示右对齐；左右冒号表示居中对齐
|add|asdfasd|avd|
```

## 8. 代码

单行代码`ccccc`

```
`content`
```

多行代码

```
content
```

```
​```java
	content
​```
```

