# Java基本数据类型有哪些，各占多大字节

Java中一共有8种基本类型，可以具体分为4类：

1、整数：byte，short，int，long

2、浮点数：float，double

3、字符：char

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