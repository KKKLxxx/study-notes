# synchronized实现原理

synchronized关键字可以修饰**方法**或者**代码块**，**它主要确保多个线程在同一个时刻，只能有一个线程处于方法或者代码块中，它保证了线程对变量访问的可见性和排他性**

Java中的每一个对象都可以作为锁，具体表现为以下3种形式

1、对于**同步代码块**，锁是synchronized括号里配置的对象

2、对于**普通同步方法**，锁是当前实例对象

3、对于**静态同步方法**，锁是当前类的Class对象

通过具体的代码来说明加锁对象的不同情况

```
public class Main {
    public static void main(String[] args) {
        Main main = new Main();
        // 对于同步代码块，锁是synchronized括号里配置的对象，即main对象
        synchronized (main) {
        }
    }

    // 对于普通同步方法，锁是当前实例对象，即调用这个方法的对象
    public synchronized void objectFun() {
    }

    // 对于静态同步方法，锁是当前类的Class对象，即Main的Class对象
    public static synchronized void staticFun() {
    }
}
```

在Main.java同级目录下启动命令行，输入以下两条命令，即可查看class文件信息

```
javac Main.java
javap -v Main
```

 截取部分

```
public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=4, args_size=1
         0: new           #2                  // class Main
         3: dup
         4: invokespecial #3                  // Method "<init>":()V
         7: astore_1
         8: aload_1
         9: dup
        10: astore_2
        11: monitorenter
        12: aload_2
        13: monitorexit
        14: goto          22
        17: astore_3
        18: aload_2
        19: monitorexit
        20: aload_3
        21: athrow
        22: return

public synchronized void objectFun();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_SYNCHRONIZED
    Code:
      stack=0, locals=1, args_size=1
         0: return
      LineNumberTable:
        line 9: 0

public static synchronized void staticFun();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_STATIC, ACC_SYNCHRONIZED
    Code:
      stack=0, locals=0, args_size=0
         0: return
      LineNumberTable:
        line 12: 0
```

 可以看到**代码块和方法的同步方式是不同的**

**对于代码块**，synchronized关键字经过Javac编译之后，会**在代码块的前后分别形成monitorenter和monitorexit这两个字节码指令**。monitorenter插入到代码块的开始位置，monitorexit插入到**代码块结束处**和**异常处**。JVM保证每个monitorenter必须有对应的monitorexit与之配对，但可能不止一个monitorexit（正如此例）

**Java中任何一个对象都有一个monitor与之关联，当一个monitor被持有后，它将处于锁定状态。**在执行monitorenter指令时，首先要去尝试获取对象的锁。如果这个对象没被锁定，或者当前线程已经持有了那个对象的锁，就把锁的计数器的值增加一，而在执行monitorexit指令时会将锁计数器的值减一。一旦计数器的值为零，锁随即就被释放了。**如果获取对象锁失败，那当前线程就应当被阻塞等待，直到请求锁定的对象被持有它的线程释放为止**

**对于方法**，则是通过ACC_SYNCHRONIZED这个修饰符来完成的。**方法级的同步是隐式的，无须通过字节码指令来控制**，JVM可以从方法常量池的方法表结构中的ACC\_SYNCHRONIZED访问标志得知一个方法是否声明为同步方法。当方法调用的时，调用指令会检查方法的ACC\_SYNCHRONIZED访问标志是否被设置，如果设置了，执行线程就要求先持有monitor对象，然后才能执行方法，最后当方法执行完（无论是正常完成还是非正常完成）时释放monitor对象

**但从本质上来说，对于代码块和方法，都是基于对一个对象的monitor的获取与释放来实现同步，只不过两者的实现细节不同**

还有一点就是**对象头**，**其作用是保存锁状态和指向重量级锁（monitor）的指针**。但是对象头部分在《轻量级锁与偏向锁》中已经说过了，所以直接放链接，唯一不同的就是锁类型的标志位

[轻量级锁与偏向锁](https://gitee.com/KKKLxxx/study-notes/blob/master/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B/%E8%BD%BB%E9%87%8F%E7%BA%A7%E9%94%81%E4%B8%8E%E5%81%8F%E5%90%91%E9%94%81.md "轻量级锁与偏向锁")

* * *

从功能上看，根据以上《Java虚拟机规范》对monitorenter和monitorexit的行为描述，我们可以得出两个关于synchronized的直接推论，这是使用它时需特别注意的：

1、**被synchronized修饰的同步块对同一条线程来说是可重入的**。这意味着同一线程反复进入同步块也不会出现自己把自己锁死的情况

2、**被synchronized修饰的同步块在持有锁的线程执行完毕并释放锁之前，会无条件地阻塞后面其他线程的进入**。这意味着无法像处理某些数据库中的锁那样，强制已获取锁的线程释放锁；也无法强制正在等待锁的线程中断等待或超时退出

从执行成本的角度看，持有锁是一个重量级的操作。我们知道在主流Java虚拟机实现中，**Java的线程是映射到操作系统的原生内核线程之上的，如果要阻塞或唤醒一条线程，则需要操作系统来帮忙完成，这就不可避免地陷入用户态到核心态的转换中，进行这种状态转换需要耗费很多的处理器时间**。尤其是对于代码特别简单的同步块（譬如被synchronized修饰的getter()或setter()方法），状态转换消耗的时间甚至会比用户代码本身执行的时间还要长。因此才说，synchronized是Java语言中一个重量级的操作，有经验的程序员都只会在确实必要的情况下才使用这种操作。而虚拟机本身也会进行一些优化，譬如在通知操作系统阻塞线程之前加入一段自旋等待过程，以避免频繁地切入核心态之中

关于这点优化，也可以通过上面那个链接了解