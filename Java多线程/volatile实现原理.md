# volatile实现原理

一、volatile的作用
-------------

1、**保证此变量对所有线程的可见性**：指当一条线程修改了这个变量的值，新值对于其他线程来说是可以立即得知的

需要注意的是，**保证了线程间的可见性不等于保证了线程安全**，即一个变量仅用volatile修饰，对这个变量的操作并不一定是线程安全的，比如

```java
public class Main {
    public static volatile int race = 0;

    public static void increase() {
        race++;
    }

    private static final int THREADS_COUNT = 10;

    public static void main(String[] args) {
        Thread[] threads = new Thread[THREADS_COUNT];
        for (int i = 0; i < THREADS_COUNT; i++) {
            threads[i] = new Thread(() -> {
                for (int i1 = 0; i1 < 10000; i1++) {
                    increase();
                }
            });
            threads[i].start();
        }
        // 等待所有累加线程都结束
        while (Thread.activeCount() > 1) {
            Thread.yield();
        }
        System.out.println(race);
    }
}
```

在IDEA下如果运行以上程序无法停止可以看这里

[Monitor Ctrl-Break线程_KKKL的博客-CSDN博客](https://blog.csdn.net/qq_45404693/article/details/120789949 "Monitor Ctrl-Break线程_KKKL的博客-CSDN博客")

代码很简单，就是开启10个线程，每个线程将race增加10000次，如果没有线程安全问题，最终结果应该为100000，但是实际结果为32056 

问题就出在自增运算“race++”之中，我们用Javap反编译这段代码后会得到代码清单12-2所示，发现只有一行代码的increase()方法在Class文件中是由4条字节码指令构成（return指令不是由race++产生的，这条指令可以不计算），从字节码层面上已经很容易分析出并发失败的原因了：**当getstatic指令把race的值取到操作栈顶时，volatile关键字保证了race的值在此时是正确的，但是在执行iconst_1、iadd这些指令的时候，其他线程可能已经把race的值改变了，而操作栈顶的值就变成了过期的数据，所以putstatic指令执行后就可能把较小的race值同步回主内存之中**

```java
代码清单12-2 VolatileTest的字节码
public static void increase();
     Code:
     Stack=2, Locals=0, Args_size=0
     0: getstatic #13; //Field race:I
     3: iconst_1
     4: iadd
     5: putstatic #13; //Field race:I
     8: return
```

2、**禁止指令重排序优化**：在单线程环境下，编译器和处理器为了提高运行速度，会在不影响运行结果的前提下对指令进行重排序。而在多线程环境下，这种重排序就会造成错误的结果。JMM通过**在修改volatile变量之后插入一个内存屏障，来禁止重排序时内存屏障之后的指令被重排序到内存屏障之前**

举个例子

```java
Map configOptions;
char[] configText;

// 此变量必须定义为volatile
volatile boolean initialized = false;

// 假设以下代码在线程A中执行
// 模拟读取配置信息，当读取完成后
// 将initialized设置为true,通知其他线程配置可用
configOptions = new HashMap();
configText = readConfigFile(fileName);
processConfigOptions(configText, configOptions);
initialized = true;

// 假设以下代码在线程B中执行
// 等待initialized为true，代表线程A已经把配置信息初始化完成
while (!initialized) {
	sleep();
}

// 使用线程A中初始化好的配置信息
doSomethingWithConfig();
```

如果定义initialized变量时没有使用volatile修饰，就可能会由于指令重排序的优化，导致位于线程A中最后一条代码“initialized=true”被提前执行（这里虽然使用Java作为伪代码，但所指的重排序优化是机器级的优化操作，提前执行是指这条语句对应的汇编代码被提前执行），这样在线程B中使用配置信息的代码就可能出现错误，而volatile关键字则可以避免此类情况的发生 

二、volatile的实现原理
---------------

对于用volatile修饰的变量，对其写操作时会**额外执行一个Lock指令**，这个Lock指令的作用如下

1、**将该处理器的工作内存写回到主内存**：即保证主内存中的值是最新的，为其他处理器读取到最新的值提供基础

2、**使其他处理器的工作内存失效**：如果其他处理器的工作内容仍然有效，则不会从主内存中重新取值，所以需要先使其他处理器的工作内存失效，然后让其从主内存中重新读取

三、volatile与synchronized的对比
--------------------------

1、volatile轻量级，**只能修饰变量**；synchronized重量级，**可以修饰代码块和方法**

2、volatile只能保证数据的可见性，不能用来同步，**因为多个线程并发访问volatile修饰的变量不会阻塞**；synchronized不仅保证可见性，而且还保证原子性，因为只有获得了锁的线程才能进入临界区，从而保证临界区中的所有语句都全部执行。**多个线程争抢synchronized锁对象时会出现阻塞**

四、volatile与synchronized的选择
--------------------------

由于volatile变量只能保证可见性，在不符合以下两条规则的运算场景中，我们仍然要通过加锁（使用synchronized、java.util.concurrent中的锁或原子类）来保证原子性：

1、**运算结果并不依赖变量的当前值，或者能够确保只有单一的线程修改变量的值**

正如上文的race++的例子，自增运算依赖于当前值，所需不符合该条件。又或者如果以某种方法实现只通过10条线程中的1条来使race自增，那么就使用volatile

2、**变量不需要与其他的状态变量共同参与不变约束**

通俗的讲，比如判断条件

```java
volatile boolean a = true;
boolean b = true;
if (a && b) {
}
```

a与b此时就叫做共同参与不变约束，而且应该（这里没搞透）不论b是否为volatile变量都不可以

而在像如下所示的这类场景中就很适合**使用volatile变量来控制并发**，当shutdown()方法被调用时，能保证所有线程中执行的doWork()方法都立即停下来

```java
volatile boolean shutdownRequested;
public void shutdown() {
    shutdownRequested = true;
}
public void doWork() {
    while (!shutdownRequested) {
        // 代码的业务逻辑
    }
}
```

## 五、volatile在Java和C中的区别

### 1. Java

volatile在java中的作用是

1、保证变量在其他线程中的可见性

2、防止指令重排序

### 2. C

但是在Linux C中，volatile表示一个变量是“易变的”，最常见的场景就是多线程

普通变量通常会被编译器优化到寄存器中提高读取效率，volatile使得程序每次读取这个变量都直接从内存中读取，以此保证数据一致性

**坑**：对于这段代码，本意是计算*ptr指向值的平方

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