# Spring AOP与AspectJ对比

## 一、织入方式

Weaving织入：链接切面和目标对象来创建一个通知对象的过程

Spring AOP采用运行时织入

AspectJ采用编译时织入

所以**AspectJ运行更快**

## 二、连接点

Joinpoint连接点：这是程序执行中的特定点，如方法执行，调用构造函数或字段赋值等

Spring AOP通过cglib代理来实现AOP，所以对于final类，由于不能被继承，所以无法实现。同样对于static和final方法，由于不能覆写，所以也不支持这些连接点

AspectJ在运行前将横切关注点直接织入实际的代码中，因此它不需要继承目标对象

所以**Spring AOP仅支持普通方法执行的连接点，AspectJ支持所有连接点**

## 三、框架限制

Spring AOP仅在Spring框架中生效

AspectJ没有框架限制

所以**AspectJ更通用**