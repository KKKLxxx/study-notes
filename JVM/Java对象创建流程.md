# Java对象创建流程

## 一、new

JVM遇到一条new指令时，先对符号引用进行解析，如果找不到对应的符号引用，那么说明这个类还没有被加载过，JVM便会先进行[类加载过程](https://gitee.com/KKKLxxx/study-notes/blob/master/JVM/Java%E7%B1%BB%E5%8A%A0%E8%BD%BD%E8%BF%87%E7%A8%8B.md "类加载过程")

## 二、为对象在堆中分配内存

包括**实例字段**、**对象头**的空间

## 三、初始化实例字段

保证了对象的实例字段在Java代码中可以不赋初始值就直接使用

## 四、初始化对象头

例如**这个对象是哪个类的实例**、如何才能找到类的元数据信息、对象的**哈希码**（实际上对象的哈希码会延后到真正调用Object::hashCode()方法时才计算）、对象的**GC分代年龄**等信息

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/6cd1fa5a09c44479a82e5c73f372205b.png" style="zoom:67%;" />

## 五、执行init方法

将构造函数中的参数赋值给对象的字段

