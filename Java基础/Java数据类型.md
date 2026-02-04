# Java数据类型

## 一、基本数据类型

Java中一共有8种基本类型，可以具体分为4类：

1、整数：byte，short，int，long

2、浮点数：float，double

3、字符：char（无符号）

4、布尔值：boolean

可以通过基本类型对应的引用类型来查看其大小

```java
public class Main {    
		public static void main(String[] args) {
        System.out.println("byte类型大小：" + Byte.BYTES + " 字节");
        System.out.println("short类型大小：" + Short.BYTES + " 字节");
        System.out.println("int类型大小：" + Integer.BYTES + " 字节");
        System.out.println("long类型大小：" + Long.BYTES + " 字节");
        System.out.println("float类型大小：" + Float.BYTES + " 字节");
        System.out.println("double类型大小：" + Double.BYTES + " 字节");
        System.out.println("char类型大小：" + Character.BYTES + " 字节");
        // System.out.println("boolean类型大小：" + Boolean.BYTES + " 字节");
    }
}
```

点进各个类型的源码中可以看见有两个字段，一个是SIZE（位数），一个是BYTES（字节数）

但是Boolean类型中并没有这两个字段，所以稍后再具体解释boolean

得到结果：

```
byte类型大小：1 字节
short类型大小：2 字节
int类型大小：4 字节
long类型大小：8 字节
float类型大小：4 字节
double类型大小：8 字节
char类型大小：2 字节
```

对于boolean类型，在《Java虚拟机规范》一书中的描述

> 虽然定义了boolean这种数据类型，但是对它提供了非常有限的支持。在Java虚拟机中没有任何供boolean值专用的字节码指令，Java语言表达式所操作的boolean值，在编译之后都使用Java虚拟机中的int数据类型来代替，而boolean数组将会被编码成Java虚拟机的byte数组，每个元素boolean元素占8位

不提供boolean专用字节码指令的原因：可能是为了简化指令

**总结一下**：

整数：byte（1），short（2），int（4），long（8）

浮点数：float（4），double（8）

字符：char（2）

布尔值：boolean（4或1）

对于单个boolean值，其所占大小为4字节

对于boolean数组，其每个boolean值所占大小为1字节

## 二、自动装箱、拆箱

自动装箱、拆箱指的是8种基本类型在**编译期**与其对应的包装类发生自动地转换

**自动装箱**：如int自动转为Integer

**自动拆箱**：如Integer自动转为int

**好处**：简化代码

**坏处**：降低代码执行效率

### 1. 包装类的作用

Java中绝大部分泛型方法都是用来处理引用类型的，而无法接收基本数据类型，比如`List<Integer>`

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

BigDecimal用于精确的数值计算，比如金钱交易等场景。double等浮点数类型不能准确地表示部分十进制小数，比如0.1

