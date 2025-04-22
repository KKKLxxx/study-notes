# Java中String字面量创建与new创建的区别

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