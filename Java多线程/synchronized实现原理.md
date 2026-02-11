# synchronized实现原理

## 一、synchronized的作用

synchronized关键字可以修饰**方法**或者**代码块**，**它主要确保多个线程在同一个时刻，只能有一个线程处于方法或者代码块中，它保证了线程对变量访问的可见性和排他性**

Java中的每一个对象都可以作为锁，具体表现为以下3种形式

1、对于**同步代码块**，锁是synchronized括号里配置的对象

2、对于**普通同步方法**，锁是当前实例对象

3、对于**静态同步方法**，锁是当前类的Class对象

## 二、synchronized底层原理

通过具体的代码来说明加锁对象的不同情况

```java
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

```java
javac Main.java
javap -v Main
```

 截取部分

```java
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

- **对于代码块**，synchronized关键字经过Javac编译之后，会**在代码块的前后分别形成monitorenter和monitorexit这两个字节码指令**。monitorenter插入到代码块的开始位置，monitorexit插入到**代码块结束处**和**异常处**。JVM保证每个monitorenter必须有对应的monitorexit与之配对，但可能不止一个monitorexit（正如此例）

  **Java中任何一个对象都有一个monitor与之关联，当一个monitor被持有后，它将处于锁定状态。**在执行monitorenter指令时，首先要去尝试获取对象的锁。如果这个对象没被锁定，或者当前线程已经持有了那个对象的锁，就把锁的计数器的值增加一，而在执行monitorexit指令时会将锁计数器的值减一。一旦计数器的值为零，锁随即就被释放了。**如果获取对象锁失败，那当前线程就应当被阻塞等待，直到请求锁定的对象被持有它的线程释放为止**

- **对于方法**，则是通过ACC_SYNCHRONIZED这个修饰符来完成的。**方法级的同步是隐式的，无须通过字节码指令来控制**，JVM可以从方法常量池的方法表结构中的ACC\_SYNCHRONIZED访问标志得知一个方法是否声明为同步方法。当方法调用的时，调用指令会检查方法的ACC\_SYNCHRONIZED访问标志是否被设置，如果设置了，执行线程就要求先持有monitor对象，然后才能执行方法，最后当方法执行完（无论是正常完成还是非正常完成）时释放monitor对象

**但从本质上来说，对于代码块和方法，都是基于对一个对象的monitor的获取与释放来实现同步，只不过两者的实现细节不同**

## 三、synchronized优化

### 1. 锁消除

锁消除是**指虚拟机即时编译器在运行时，对一些代码要求同步，但是对被检测到不可能存在共享数据竞争的锁进行消除**。锁消除的主要判定依据来源于逃逸分析的数据支持，如果判断到一段代码中，在堆上的所有数据都不会逃逸出去被其他线程访问到，那就可以把它们当作栈上数据对待，认为它们是线程私有的，同步加锁自然就无须再进行

也许读者会有疑问，变量是否逃逸，对于虚拟机来说是需要使用复杂的过程间分析才能确定的，但是程序员自己应该是很清楚的，怎么会在明知道不存在数据争用的情况下还要求同步呢？这个问题的答案是：有许多同步措施并不是程序员自己加入的，同步的代码在Java程序中出现的频繁程度也许超过了大部分读者的想象。我们来看看如下所示的例子，这段非常简单的代码仅仅是输出三个字符串相加的结果，无论是源代码字面上，还是程序语义上都没有进行同步

```java
public String concatString(String s1, String s2, String s3) {
    return s1 + s2 + s3;
}
```

我们也知道，由于String是一个不可变的类，对字符串的连接操作总是通过生成新的String对象来进行的，因此Javac编译器会对String连接做自动优化。在JDK 5之前，字符串加法会转化为StringBuffer对象的连续append()操作，在JDK 5及以后的版本中，会转化为StringBuilder对象的连续append()操作。即以上代码可能会变成以下所示的样子

```java
public String concatString(String s1, String s2, String s3) {
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    sb.append(s3);
    return sb.toString();
}
```

现在大家还认为这段代码没有涉及同步吗？每个StringBuffer.append()方法中都有一个同步块，锁就是sb对象。虚拟机观察变量sb，经过逃逸分析后会发现它的动态作用域被限制在concatString()方法内部。也就是**sb的所有引用都永远不会逃逸到concatString()方法之外，其他线程无法访问到它，所以这里虽然有锁，但是可以被安全地消除掉**。在解释执行时这里仍然会加锁，但在经过服务端编译器的即时编译之后，这段代码就会忽略所有的同步措施而直接执行 

### 2. 锁粗化

**原则上，我们在编写代码的时候，总是推荐将同步块的作用范围限制得尽量小**——只在共享数据的实际作用域中才进行同步，这样是为了使得需要同步的操作数量尽可能变少，即使存在锁竞争，等待锁的线程也能尽可能快地拿到锁

**大多数情况下，上面的原则都是正确的，但是如果一系列的连续操作都对同一个对象反复加锁和解锁，甚至加锁操作是出现在循环体之中的，那即使没有线程竞争，频繁地进行互斥同步操作也会导致不必要的性能损耗**

代码所示连续的append()方法就属于这类情况。**如果虚拟机探测到有这样一串零碎的操作都对同一个对象加锁，将会把加锁同步的范围扩展（粗化）到整个操作序列的外部**，以代码2为例，就是扩展到第一个append()操作之前直至最后一个append()操作之后，这样只需要加锁一次就可以了

### 3. 锁膨胀/锁升级

锁膨胀也称为锁升级，并不同于锁粗化。**锁粗化是指锁同步范围的扩大，而锁膨胀是指锁类型升级为更加重量级的锁**

Java中的锁一共有4种状态，级别从低到高依次是：**无锁、偏向锁、轻量级锁、重量级锁**。这几个状态会随着竞争情况逐渐升级，锁可以升级但不能降级，意味着偏向锁升级为轻量级锁后不能降级为偏向锁。**这种锁升级却不能降级的策略，目的是提高获得锁和释放锁的效率**

锁升级过程可以简单理解为，**每个对象起始状态都是最低级，但随着线程竞争的发生，这个对象的锁等级会一步一步提高，最终到达重量级锁**

HotSpot虚拟机对象头Mark Word的内容：

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/6cd1fa5a09c44479a82e5c73f372205b.png" style="zoom:67%;" />

- **偏向锁通过在对象头记录当前线程的ID从而记录偏向线程**。偏向线程再次进入同步块时，只需比较当前线程的ID与偏向线程的ID，如果相同则无需CAS即可获得锁，适用于单线程重复访问场景
- **轻量级锁则通过在栈中创建Lock Record（锁记录，用于存储锁对象Mark Word的拷贝），并使用CAS将对象头指向Lock Record实现加锁**。如果这个更新动作成功了，即代表该线程拥有了这个对象的锁。通过对象头是否指向自己的栈来确定是否为重入，适用于低竞争环境（即虽然有多个线程，但不同线程不会同时访问的场景）
- 当CAS失败且存在真实竞争时，锁会升级为重量级锁

三种锁的对比：

| **锁**       | **优点**                                         | **缺点**                                       | **适用场景**                           |
| ------------ | ------------------------------------------------ | ---------------------------------------------- | -------------------------------------- |
| **偏向锁**   | 没有加锁、解锁的额外消耗，几乎没有不影响运行速度 | 如果线程间存在竞争，则会带来额外的锁撤销的消耗 | 只有一个线程访问同步块                 |
| **轻量级锁** | 竞争的线程不会阻塞，提高响应速度                 | 如果线程长时间得不到锁会一直自旋消耗CPU资源    | 虽然有多个线程，但不同线程不会同时访问 |
| **重量级锁** | 线程竞争不会自旋，不额外消耗CPU                  | 线程阻塞，响应缓慢                             | 多个线程会产生并发冲突                 |

**更新：偏向锁在JDK18中已被废弃，因为偏向锁的性能收益不明显，且JVM维护成本过高。随着JDK的发展，出现了一些高性能的线程安全的集合（比如ConcurrentHashMap），所以偏向锁被废弃了**

