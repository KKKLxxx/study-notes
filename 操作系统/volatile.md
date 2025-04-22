# volatile

## 一、Java

volatile在java中的作用是

1、保证变量在其他线程中的可见性

2、防止指令重排序

## 二、C

但是在linux c中，volatile表示一个变量是“易变的”，最常见的场景就是多线程

普通变量通常会被编译器优化到寄存器中提高读取效率，volatile使得程序每次读取这个变量都直接从内存中读取，以此保证数据一致性

### 坑

对于这段代码，本意是计算*ptr指向值的平方

```c
int square(volatile int *ptr) {
    return (*ptr) * (*ptr);
}
```

但由于每次都是从内存中读取变量，所以可能实际执行效果如下

```c
int square(volatile int *ptr) {
    int a = *ptr;
    int b = *ptr;
    return a * b;
}
```

如果两次分别读取`*ptr`的过程中，`*ptr`有变化，那么就会变为两个不同的数相乘，是一个意想不到的结果