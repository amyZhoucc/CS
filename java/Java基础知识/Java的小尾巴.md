# Java的小尾巴

# 1. 正则表达式

正则表达式是一串字符，它描述了一个文本模式，利用它可以方便地处理文本，包括文本的查找、替换、验证、切分等

正则表达式中的字符有两类：一类是普通字符，就是匹配字符本身；另一类是元字符，这些字符有特殊含义。

## 1.1 语法

### 单个字符：

有一些单个字符使用多个字符表示，这些字符都以斜杠`\`开头：

- 特殊字符：tab、换行
- 八进制\0，十六进制\x
- unicode字符\u
- 斜杠本身：`\\`表示自己
- 元字符本身，除了`\`，正则表达式中还有很多元字符，比如．、*、? 、+等，要匹配这些元字符自身，需要在前面加转义字符'\'，比如`\.`

### 字符组：

点号字符`.`是一个元字符，默认模式下，它匹配除了换行符以外的任意字符，比如`a.f` == abf、adf等

单行匹配模式/点号匹配模式：以(? s)开头，`.`匹配任意字符，包括换行符。eg：(? s)a.f

有一个字符组的概念，匹配组中的任意一个字符，用中括号[]表示：

`[a,b,c,d]`表示匹配其中的任意一个字符。

如果字符是连续的可以用`-`来省略中间的。也可以如此写[0-9a-zA-Z]

而如果要匹配`-`本身，那么需要带上转义字符，或者将`-`放在最前面：`[-0-9]`，表示匹配-、0~9的字符

如果选择排除：`[^abcd]`匹配除abcd以外的字符

### 量词

- `+`：表示前面字符的一次或多次出现，比如正则表达式ab+c，既能匹配abc，也能匹配abbc，或abbbc。
- `*`：表示前面字符的零次或多次出现，比如正则表达式ab*c，既能匹配abc，也能匹配ac，或abbbc
- ? ：表示前面字符可能出现，也可能不出现（最多出现一次）

....（太多了，暂时不看了）

### 边界环视

主要是在匹配过程中可能从其他某个字符串中提取：

eg："邮编123456，电话123456789011"，然后我只需要提取邮编，邮编固定长度为6位`[0-9]{6}`，但是结果是电话也会有前6位被提取出来。

所以需要边界环视，看其左右是否还是存在数字：`?<![0-9]`，右边界`?![0-9]`

## 1.2 Java API

### 1.2.1 split

```java
String[] fields = str.split(", ");
String[] fields = str.split("\\.");		// 按照点分隔
String[] fields = str.split("[\\s.]+");		// 将一个或多个空白字符或点号作为分隔符
```

### 1.2.2 replace

```java
String regex = "\\s+";
String string = "hello world        good";
string.replaceAll(regex, " ");
```

——将中间多个空格替换为一个空格

# 2. 序列化

一个对象可以被表示为一个字节序列，**该字节序列包括该对象的数据、有关对象的类型的信息和存储在对象中数据的类型**。

将序列化对象写入文件之后，可以从文件中读取出来，并且对它进行反序列化，也就是说，对象的类型信息、对象的数据，还有对象中的数据类型可以用来在内存中新建对象。

整个过程都是JVM独立的，在一个平台上序列化的对象可以在另一个完全不同的平台上反序列化该对象。

一个类的对象要想序列化成功，必须满足两个条件：

- 该类必须实现 java.io.Serializable 接口
- 该类的所有属性必须是可序列化的。如果有一个属性不是可序列化的，则该属性必须注明是transient

eg：Employee.java文件

```java
public class Employee implements java.io.Serializable		// 实现了序列化的接口
{
   public String name;
   public String address;
   public transient int SSN;			// transient修饰的
   public int number;
   public void mailCheck()
   {
      System.out.println("Mailing a check to " + name
                           + " " + address);
   }
}
```

序列化一个对象：

ObjectOutputStream 类用来序列化一个对象

当序列化一个对象到文件时， 按照 Java 的标准约定是给文件一个 **.ser 扩展名**。

```java
public class SerializeDemo
{
   public static void main(String [] args)
   {
      Employee e = new Employee();
      e.name = "Reyan Ali";
      e.address = "Phokka Kuan, Ambehta Peer";
      e.SSN = 11122333;
      e.number = 101;
      try
      {
         FileOutputStream fileOut =
         new FileOutputStream("/tmp/employee.ser");
         ObjectOutputStream out = new ObjectOutputStream(fileOut);
         out.writeObject(e);
         out.close();
         fileOut.close();
         System.out.printf("Serialized data is saved in /tmp/employee.ser");
      }catch(IOException i)
      {
          i.printStackTrace();
      }
   }
}
```

反序列化：

```java
public class DeserializeDemo
{
   public static void main(String [] args)
   {
      Employee e = null;
      try
      {
         FileInputStream fileIn = new FileInputStream("/tmp/employee.ser");
         ObjectInputStream in = new ObjectInputStream(fileIn);
         e = (Employee) in.readObject();
         in.close();
         fileIn.close();
      }catch(IOException i)
      {
         i.printStackTrace();
         return;
      }catch(ClassNotFoundException c)
      {
         System.out.println("Employee class not found");
         c.printStackTrace();
         return;
      }
      System.out.println("Deserialized Employee...");
      System.out.println("Name: " + e.name);
      System.out.println("Address: " + e.address);
      System.out.println("SSN: " + e.SSN);			// =0,因为禁止了反序列化，所以不能获得原来的值
      System.out.println("Number: " + e.number);
    }
}
```

