# Java数据类型

## 一、基本数据类型

Java中一共有8种基本类型，可以具体分为4类：

1、整数：byte，short，int，long

2、浮点数：float，double

3、字符：char（无符号）

4、布尔值：boolean

| 基本类型  | 作为局部变量（栈） | 作为数组元素（堆） | 备注                           |
| :-------- | :----------------- | :----------------- | :----------------------------- |
| `byte`    | 4 字节             | 1 字节             | 局部变量被提升为 `int` 处理    |
| `short`   | 4 字节             | 2 字节             | 同上                           |
| `int`     | 4 字节             | 4 字节             | —                              |
| `long`    | 8 字节             | 8 字节             | 局部变量占用连续两个槽位       |
| `float`   | 4 字节             | 4 字节             | —                              |
| `double`  | 8 字节             | 8 字节             | 局部变量占用连续两个槽位       |
| `char`    | 4 字节             | 2 字节             | 局部变量被提升为 `int` 处理    |
| `boolean` | 4 字节             | 1 字节             | 局部变量被当作 `int`（值 0/1） |

作为单个局部变量时，因为栈的每个槽位是4字节，所以为了复用int的指令集，并提高执行效率，小于等于int的数据类型会被当作int处理，占用4个字节

作为数组时，为了节省空间，会按照其本身所应占的大小进行分配

虽然boolean理论上只需要1bit的存储空间，但是计算机的最小寻址单元是字节，如果用bit存储的话，可能无法直接访问和修改一个单独的位

## 二、自动装箱、拆箱

自动装箱、拆箱指的是8种基本类型在**编译期**与其对应的包装类发生自动地转换

**自动装箱**：如int自动转为Integer

**自动拆箱**：如Integer自动转为int

**好处**：简化代码

**坏处**：降低代码执行效率

### 1. 包装类的作用

Java中泛型方法都是用来处理引用类型的，而无法接收基本数据类型，比如`List<Integer>`

### 2. 为什么还要保留基本数据类型

包装类因为封装了各种方法，所以会占用更大的空间，并且操作速度也更慢。在一般情况下，还是应该使用基本数据类型

### 3. Integer缓存池

当通过`Integer.valueOf(num)`方法创建一个`Integer`对象时，如果num的值在[-128,127]之间，则会从一个缓存池中获取一个对象并直接返回，而不用申请空间创建新的对象

## 三、数据类型转换方式

- 数字之间的转换
  - 自动类型转换（隐式）：当一个`int`与一个`long`做运算时，`int`会自动升为`long`再做运算
  - 强制类型转换（显式）：比如`int a = (int) b`，其中`b`为`long`类型。大类型转小类型可能发生溢出或者精度丢失问题
- 数字与字符串之间的转换：如`Integer.parseInt()`和`String.valueOf()`
- 父子类之间的转换：子类转父类可以自动转换，比如`Parent parent = child`；父类转子类需要显式转换，并且有风险，运行时可能抛出`ClassCastException`，比如`Child child = (Child) parent`，因为`parent`实际可能指向另一个子类`Child2`，可能与`Child`不兼容

**Integer.parseInt()与Integer.valueOf()的区别**

查看源码

```java
public static int parseInt(String s) throws NumberFormatException {
    return parseInt(s,10);
}
```

````java
public static Integer valueOf(String s) throws NumberFormatException {
    return Integer.valueOf(parseInt(s, 10));
}
````

可以看到，valueOf实际上是调用了parseInt，不过返回值的类型有一些区别，这就涉及到了自动拆箱与自动装箱

众所周知，自动拆箱与自动装箱会对性能有一些影响。如果我们最终需要的类型就是int，那么最好直接使用parseInt，否则可以选择valueOf

## 四、BigDecimal的作用

BigDecimal用于精确的数字计算，比如金钱交易等场景。double等浮点数类型是基于二进制存储的，不能准确地表示部分十进制小数，比如0.1

`BigDecimal` 通过将十进制数字表示为**未缩放的整数值（unscaled value）**和**小数位数（scale）**的组合，并结合整数运算与舍入规则，实现了精确的数字计算

