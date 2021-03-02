# Java常见库方法整理

## 1. Math类

只包含一系列的数学方法，且全都是静态方法，直接通过`Math.`调用

| 函数名           | 参数                           | 返回值   | 作用                                                         |
| ---------------- | ------------------------------ | -------- | ------------------------------------------------------------ |
| `Math.round()`;  | float a/double a               | int/long | 将浮点数四舍五入为整数                                       |
| `Math.ceil();`   | double a                       | double   | 向上取整（返回值仍是浮点数）                                 |
| `Math.floor();`  | double a                       | double   | 向下取整（返回值仍是浮点数）                                 |
| `Math.abs();`    | int/long/float/double a        | 对应类型 | 取绝对值                                                     |
| `Math.pow();`    | double a, double b             | double   | a^b，无论输入是何类型，**返回值都是double**                  |
| `Math.sqrt();`   | double a                       | double   | 对a开平方，返回值一直是double                                |
| `Math.max();`    | int a, int b/long/float/double | 对应类型 | 求a、b两数的较大值，返回值具体看输入的情况（输入两个的类型不一样就会选择强转） |
| `Math.log();`    | double a                       | double   | 求的自然数对数（e）                                          |
| `Math.random();` |                                | double   | 获得[0,1)范围内的随机数                                      |

## 2. Arrays类

针对数组操作的类

| 函数名                   | 参数                                                        | 返回值                                            | 作用                                                         |
| ------------------------ | ----------------------------------------------------------- | ------------------------------------------------- | ------------------------------------------------------------ |
| `Arrays.sort();`         | int[] a(int fromIndex, int toIndex)（不同数据类型都实现了） | void                                              | 在原数组位置上排序，默认是升序排序<br/>（参数：还可以自定义比较方法Comparator<? super T> c） |
| `Arrays.binarySearch();` | int[] a, int key                                            | int（找不到就返回一个负数，即=-(能插入的位置)-1） | 二分查找（前提是数组已经升序排列）                           |
| `Arrays.fill();`         | int[] a, int key                                            | void                                              | 给所有元素都赋值同一个值                                     |
| `Arrays.copyOf();`       | int[] a, int length                                         | int[]                                             | 将a数组的前length内容赋值给一个新数组，并返回该数组（是两个不同的数组，并且如果length>a.length，那后面的值自动补0） |
| `Arrays.equals();`       | int[] a, int[] b                                            | true/false                                        | 判断两个数组是否一样（是数组里面值的比较）                   |
