# Java中String相关问题

## 一、String字面量创建与new创建的区别

最简单的方式就是通过编写一段代码实际运行检验一下

```java
public class Main {
    public static void main(String[] args) {
        String a = "string";
        String b = "string";
        if (a == b) {
            System.out.println("字面量创建字符串会从字符串常量池中获取，如果没有再创建");
        }

        String c = new String("string");
        if (a != c) {
            System.out.println("new创建不会从常量池中获取，每次都会新开辟一个空间");
        }

        String d = new String("string2");
        String e = "string2";
        if (d != e) {
            System.out.println("new创建的字符串不会加入到字符串常量池中");
        }
    }
}
```

注意这里比较的是字符串的地址，所有直接用`==`作比较，正常情况下字符串比较需要用`equals()`方法

最终输出的结果即结论

```
字面量创建字符串会从字符串常量池中获取，如果没有再创建
new创建不会从常量池中获取，每次都会新开辟一个空间
new创建的字符串不会加入到字符串常量池中
```

**补充：字符串常量池在内存中的分布**

由《深入理解Java虚拟机：JVM高级特性与最佳实践（第3版）》一书中的介绍，在JDK1.7之后字符串常量池分布在堆中

## 二、String, StringBuilder, StringBuffer的区别

### 1. String

在Java中，String被设计为不可变对象，如果要修改字符串，就会重新申请一处空间并赋值。这导致当有大量字符串拼接操作时，如果直接使用String对象相加，效率会极其低下。所以引入了两个类StringBuilder与StringBuffer来解决这个问题

#### 1.1 为什么String不可变？

在JDK8中，看到String的源码

```java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}
```

可以看到

用于存储字符串的value数组是用`private final`修饰的，这保证了value指向的数组不可变（但是仍然能通过`value[0] = 'a'`这样的写法来改变value数组的内容，不过整个String类并没有提供这样的方法）

String类是用final修饰的，所以避免了通过继承String类来修改value数组的可能

#### 1.2 可变与不可变的对比

```java
public class Main {
    public static void main(String[] args) {
        String s1 = "s1";
        String s2 = changeString(s1);
        System.out.println("s1 = " + s1);
        System.out.println("s2 = " + s2);

        StringBuilder sb1 = new StringBuilder("sb1");
        StringBuilder sb2 = changeStringBuilder(sb1);
        System.out.println("sb1 = " + sb1);
        System.out.println("sb2 = " + sb2);
    }

    public static String changeString(String s) {
        s += "s2";
        return s;
    }

    public static StringBuilder changeStringBuilder(StringBuilder sb) {
        sb.append("sb2");
        return sb;
    }
}
```

输出结果

```
s1 = s1
s2 = s1s2
sb1 = sb1sb2
sb2 = sb1sb2
```

可以看到

不可变的String，经过changeString函数的修改后，在main函数中的值没有变化

可变的StringBuilder，在main函数中的值变化了

#### 1.3 不可变的好处

1、满足字符串常量池的需要

2、可缓存哈希值：String类中有一个名为`hash`的字段，用于缓存哈希值，可以提高Hash结构确定下标的速度

3、保证线程安全：因为不可变的String相当于一个只读对象，所以没有线程安全问题

### 2. StringBuilder与StringBuffer

**两者的区别在于**：Builder线程不安全，但是快；Buffer线程安全，但是慢

查看两个类的源码可知（以最简单的append方法为例）

StringBuilder

```java
public StringBuilder append(Object obj) {    
	return append(String.valueOf(obj));
}
```

StringBuffer

```java
public synchronized StringBuffer append(Object obj) {
    toStringCache = null;    
    super.append(String.valueOf(obj));    
    return this;
}
```

StringBuffer通过一个synchronized锁来保证线程安全，自然会导致速度相对较慢

**如何选择**：在单线程环境中，选择StringBuilder，否则选择StringBuffer

### 3. StringBuilder为什么比String直接拼接快

String是一个不可变类，每次字符串拼接后，原String都会被垃圾回收，并申请一块新的内存用于存储新String

实际上，Java对String类型的`+`和`+=`运算符做了重载，在通过`+`对String进行拼接时，Java底层会创建一个StringBuilder来实现拼接

```
String str1 = "he";
String str2 = "llo";
String str3 = "world";
String str4 = str1 + str2 + str3;
```

上面的代码对应的字节码如下：

![img](https://raw.githubusercontent.com/KKKLxxx/img-host/master/68747470733a2f2f67756964652d626c6f672d696d616765732e6f73732d636e2d7368656e7a68656e2e616c6979756e63732e636f6d2f6769746875622f6a61766167756964652f6a6176612f696d6167652d32303232303432323136313633373932392e706e67)

可以看出，String对象通过`+`的字符串拼接方式，实际上是通过 `StringBuilder` 调用 `append()` 方法实现的，拼接完成之后调用`toString()`得到一个`String`对象

不过，在循环内使用`+`进行字符串的拼接的话，存在比较明显的缺陷：**编译器不会创建单个 `StringBuilder` 以复用，会导致创建过多的 `StringBuilder` 对象**

而StringBuilder实际上是在内部维护了一个char数组，扩容原理类似于HashMap，每次都会额外申请一些空间，大大减少了内存申请与销毁的次数，所以效率更高

## 三、String.valueOf(arr)和arr.toString()的区别

在Java中，`String.valueOf(arr)` 和 `arr.toString()` 在处理数组时有**重要区别**，但结果**通常相同**（除了`char[]`数组的特例）。

### 1. 对于非`char[]`类型数组

对于大多数数组类型（`int[]`、`String[]`、`Object[]`等），两者**行为相同**：

```java
int[] intArr = {1, 2, 3};
String[] strArr = {"a", "b", "c"};

// 两者都输出类似 "[I@1b6d3586"
System.out.println(intArr.toString());
System.out.println(String.valueOf(intArr));

// 两者都输出类似 "[Ljava.lang.String;@4554617c"
System.out.println(strArr.toString());
System.out.println(String.valueOf(strArr));
```

**原因：**

- `arr.toString()`：调用数组继承自`Object`的`toString()`方法
- `String.valueOf(arr)`：内部也是调用`arr.toString()`

### 2. 对于`char[]`数组的特例

这是**最重要的区别**：

```java
char[] charArr = {'h', 'e', 'l', 'l', 'o'};

// 输出类似 "[C@1b6d3586"（对象地址表示）
System.out.println(charArr.toString());

// 输出 "hello"（字符连接成字符串）
System.out.println(String.valueOf(charArr));

// 输出类似 "[h, e, l, l, o]"（数组格式）
System.out.println(Arrays.toString(charArr));
```

**原因：**

- `String.valueOf(char[])` 有**专门的重载方法**，会将字符数组转换为字符串
- 这是Java为字符数组提供的特殊处理

### 3. `null` 处理的不同

```java
int[] nullArr = null;

// 抛出 NullPointerException
// System.out.println(nullArr.toString());

// 输出 "null"（不会抛异常）
System.out.println(String.valueOf(nullArr));
```

### 4. 获取数组内容的正确方式

如果想要数组内容的可读表示，**两者都不是正确选择**，应该使用：

```java
int[] arr = {1, 2, 3};

// 错误：输出类似 "[I@1b6d3586"
System.out.println(arr.toString());

// 正确：输出 "[1, 2, 3]"
System.out.println(Arrays.toString(arr));

// 对于多维数组
int[][] multiArr = {{1, 2}, {3, 4}};
System.out.println(Arrays.deepToString(multiArr));  // 输出 "[[1, 2], [3, 4]]"
```

### 5. 总结对比表

| 方法                   | 非`char[]`数组     | `char[]`数组         | `null`处理 | 推荐使用场景                 |
| :--------------------- | :----------------- | :------------------- | :--------- | :--------------------------- |
| `arr.toString()`       | 对象地址字符串     | 对象地址字符串       | 抛出NPE    | 极少使用                     |
| `String.valueOf(arr)`  | 对象地址字符串     | **字符连接成字符串** | 返回"null" | 处理`char[]`或需要null安全时 |
| `Arrays.toString(arr)` | **数组内容字符串** | 数组格式字符串       | 抛出NPE    | 查看数组内容                 |

### 6. 最佳实践

1、**查看数组内容**：使用 `Arrays.toString(arr)` 或 `Arrays.deepToString(arr)`

2、**将`char[]`转字符串**：使用 `String.valueOf(charArr)` 或 `new String(charArr)`

3、**需要null安全**：使用 `String.valueOf(arr)`（但注意对于非`char[]`数组，输出的是对象地址）

4、**避免使用**：直接对数组调用`toString()`（除非你确实需要对象地址）

