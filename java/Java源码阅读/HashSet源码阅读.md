# HashSet源码阅读

## 概述

## 1. 类头

```java
public class HashSet<E> extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
```

继承了`AbstractSet`

实现了`Set`，Set继承了Collection，但是没有对其进行扩展，只是概念上限制了里面的元素不能出现重复