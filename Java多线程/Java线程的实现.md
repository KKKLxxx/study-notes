# Java线程的实现

一、线程实现的主要方式
--------

线程的实现主要有三种方式：使用内核线程实现（1：1实现），使用用户线程实现（1：N实现），使用用户线程加轻量级进程混合实现（N：M实现）

其中的比值是**内核线程：用户线程**，即一个内核线程对应多少个用户线程

### 1. 内核线程实现

使用内核线程实现的方式也被称为1：1实现。内核线程（Kernel-Level Thread，KLT）就是直接由操作系统内核（Kernel，下称内核）支持的线程，这种线程由内核来完成线程切换，内核通过操纵调度器（Scheduler）对线程进行调度，并负责将线程的任务映射到各个处理器上。每个内核线程可以视为内核的一个分身，这样操作系统就有能力同时处理多件事情，支持多线程的内核就称为多线程内核（Multi-Threads Kernel）

程序一般不会直接使用内核线程，而是使用内核线程的一种高级接口——轻量级进程（Light Weight Process，LWP），轻量级进程就是我们通常意义上所讲的线程，由于每个轻量级进程都由一个内核线程支持，因此只有先支持内核线程，才能有轻量级进程。这种轻量级进程与内核线程之间1：1的关系称为一对一的线程模型，如图所示

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/2021101219240913.png" style="zoom:50%;" />

由于内核线程的支持，每个轻量级进程都成为一个独立的调度单元，即使其中某一个轻量级进程在系统调用中被阻塞了，也不会影响整个进程继续工作。**轻量级进程也具有它的局限性：首先，由于是基于内核线程实现的，所以各种线程操作，如创建、析构及同步，都需要进行系统调用。而系统调用的代价相对较高，需要在用户态（User Mode）和内核态（Kernel Mode）中来回切换。其次，每个轻量级进程都需要有一个内核线程的支持，因此轻量级进程要消耗一定的内核资源（如内核线程的栈空间），因此一个系统支持轻量级进程的数量是有限的**

### 2. 用户线程实现

使用用户线程实现的方式被称为1：N实现。广义上来讲，一个线程只要不是内核线程，都可以认为是用户线程（User Thread，UT）的一种，因此从这个定义上看，轻量级进程也属于用户线程，但轻量级进程的实现始终是建立在内核之上的，许多操作都要进行系统调用，因此效率会受到限制，并不具备通常意义上的用户线程的优点

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/20211012195647581.png" style="zoom:50%;" />

而狭义上的用户线程指的是完全建立在用户空间的线程库上，系统内核不能感知到用户线程的存在及如何实现的。**用户线程的建立、同步、销毁和调度完全在用户态中完成，不需要内核的帮助。如果程序实现得当，这种线程不需要切换到内核态，因此操作可以是非常快速且低消耗的，也能够支持规模更大的线程数量**，部分高性能数据库中的多线程就是由用户线程实现的。这种进程与用户线程之间1：N的关系称为一对多的线程模型

用户线程的优势在于不需要系统内核支援，劣势也在于没有系统内核的支援，**所有的线程操作都需要由用户程序自己去处理。线程的创建、销毁、切换和调度都是用户必须考虑的问题，而且由于操作系统只把处理器资源分配到进程，那诸如“阻塞如何处理”“多处理器系统中如何将线程映射到其他处理器上”这类问题解决起来将会异常困难，甚至有些是不可能实现的**。因为使用用户线程实现的程序通常都比较复杂，除了有明确的需求外（譬如以前在不支持多线程的操作系统中的多线程程序、需要支持大规模线程数量的应用），一般的应用程序都不倾向使用用户线程。Java、Ruby等语言都曾经使用过用户线程，最终又都放弃了使用它。但是近年来许多新的、以高并发为卖点的编程语言又普遍支持了用户线程，譬如Golang、Erlang等，使得用户线程的使用率有所回升

### 3. 用户线程加轻量级进程混合实现

线程除了依赖内核线程实现和完全由用户程序自己实现之外，还有一种将内核线程与用户线程一起使用的实现方式，被称为N：M实现。在这种混合实现下，既存在用户线程，也存在轻量级进程。用户线程还是完全建立在用户空间中，因此用户线程的创建、切换、析构等操作依然廉价，并且可以支持大规模的用户线程并发。而操作系统支持的轻量级进程则作为用户线程和内核线程之间的桥梁，这样可以使用内核提供的线程调度功能及处理器映射，并且用户线程的系统调用要通过轻量级进程来完成，这大大降低了整个进程被完全阻塞的风险。在这种混合模式中，用户线程与轻量级进程的数量比是不定的，是N：M的关系，如图所示，这种就是多对多的线程模型

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/20211012200009499.png" style="zoom:50%;" />

许多UNIX系列的操作系统，如Solaris、HP-UX等都提供了M：N的线程模型实现。在这些操作系统上的应用也相对更容易应用M：N的线程模型

二、Java线程的实现
-----------

从JDK 1.3起，“主流”平台上的“主流”商用Java虚拟机的线程模型普遍都被替换为基于操作系统原生线程模型来实现，即采用**1：1的线程模型**

同时要了解**主内存和工作内存**的概念

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202602041801159.png" alt="img" style="zoom: 67%;" />

为了提高每个线程的读写速度，每个线程会有各自的工作内存，这个工作内存相当于一个“Redis缓存”，而主内存就相当于一个“MySQL数据库”

为了保证主内存和工作内存的一致性，需要通过volatile或者其他锁机制来实现

## 三、Java线程状态转换

Java语言定义了6种线程状态，在任意一个时间点中，一个线程只能有且只有其中的一种状态，并且可以通过特定的方法在不同状态之间转换。这6种状态分别是：

### 1. 新建（New）

创建后尚未启动的线程处于这种状态

### 2. 运行（Runnable）

包括操作系统线程状态中的Running和Ready，也就是处于此状态的线程**有可能正在执行，也有可能正在等待着操作系统为它分配执行时间**

### 3. 无限期等待（Waiting）

处于这种状态的线程不会被分配处理器执行时间，它们**要等待被其他线程显式唤醒**。以下方法会让线程陷入无限期的等待状态：

* 没有设置Timeout参数的Object::wait()方法；

* 没有设置Timeout参数的Thread::join()方法；

* LockSupport::park()方法

### 4. 限期等待（Timed Waiting）

处于这种状态的线程也不会被分配处理器执行时间，不过**无须等待被其他线程显式唤醒，在一定时间之后它们会由系统自动唤醒**。以下方法会让线程进入限期等待状态：

* Thread::sleep()方法；

* 设置了Timeout参数的Object::wait()方法；

* 设置了Timeout参数的Thread::join()方法；

* LockSupport::parkNanos()方法；

* LockSupport::parkUntil()方法

### 5. 阻塞（Blocked）

线程被阻塞了，**“阻塞状态”与“等待状态”的区别是“阻塞状态”在等待着获取到一个排它锁，这个事件将在另外一个线程放弃这个锁的时候发生**；而“等待状态”则是在等待一段时间，或者唤醒动作的发生。在程序等待进入同步区域的时候，线程将进入这种状态

### 6. 结束（Terminated）

已终止线程的线程状态，线程已经结束执行

上述6种状态在遇到特定事件发生的时候将会互相转换，它们的转换关系如图所示

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111121932599.png" style="zoom:50%;" />

---

**Q：为什么将Running与Ready合并为一个Runnable**

A：因为线程在Running与Ready两个状态之间的转换是非常快的，具体到某一个状态的意义不大。并且Java不太需要关于底层的线程调度，完全交给OS处理即可

## 四、线程的创建方式

### 1. 继承Thread类

```java
class MyThread extends Thread {
    @Override
    public void run() {
        // 线程执行的代码
    }
}

public static void main(String[] args) {
    MyThread t = new MyThread();
    t.start();
}
```

缺点在于由于已经继承了Thread类，所以不能再继承其他的父类

### 2. 实现Runnable接口

```java
class MyRunnable implements Runnable {
    @Override
    public void run() {
        // 线程执行的代码
    }
}

public static void main(String[] args) {
    Thread t = new Thread(new MyRunnable());
    // Thread t1 = new Thread(() -> System.out.println("xxx"));		// 可用lambda简写
    t.start();
}
```

### 3. 实现Callable接口与FutureTask

```java
class MyCallable implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        // 线程执行的代码，这里返回一个整型结果
        return 1;
    }
}

public static void main(String[] args) {
    MyCallable task = new MyCallable();
    FutureTask<Integer> futureTask = new FutureTask<>(task);
    Thread t = new Thread(futureTask);
    t.start();
    
    try {
        Integer result = futureTask.get();  // 获取线程执行结果
        System.out.println("Result: " + result);
    } catch (InterruptedException | ExecutionException e) {
        e.printStackTrace();
    }
}
```

#### 3.1 Runnable与Callable的区别

|            | Runnable     | Callable                                                     |
| ---------- | ------------ | ------------------------------------------------------------ |
| **返回值** | 没有返回值   | 有返回值，和Future、FutureTask配合可以用来获取异步执行的结果 |
| **异常**   | 无法抛出异常 | 可以抛出异常                                                 |

### 4. 使用线程池（Executor框架）

```java
class Task implements Runnable {
    @Override
    public void run() {
        // 线程执行的代码
    }
}

public static void main(String[] args) {
    ExecutorService executor = Executors.newFixedThreadPool(10);  // 创建固定大小的线程池
    for (int i = 0; i < 100; i++) {
        executor.submit(new Task());  // 提交任务到线程池执行
    }
    executor.shutdown();  // 关闭线程池
}
```

线程池的细节可见另一篇文章

## 五、线程的启动方式

### 1. start()

单个线程可通过`Thread`类的`start()`方法启动

```java
Thread t = new Thread(new MyRunnable());
t.start();
```

#### 1.1 为什么不能直接用run()启动线程

通过代码测试一下

```java
public class Main {
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            System.out.println(Thread.currentThread());
            System.out.println("t1");
        });
        t1.run();
        t1.start();
    }
}
```

```
Thread[main,5,main]
t1
Thread[Thread-0,5,main]
t1
```

可以看到，直接调用run()方法后，**并没有新开启一个线程，而是在调用线程里执行**，没有达到多线程的效果

而start()方法才能真正开启一个线程，并在新线程内执行实现的Runnable接口

### 2. execute()或submit()

线程池则通过`execute()`或`submit()`启动

## 六、线程的终止方式

### 1. interrupt()

#### 1.1 interrupt()的作用

interrupt()方法用于**向一个线程发送中断请求，请求结果是将被请求线程的中断标志位设置为true**。需要注意的是，**interrupt()并不会直接中断线程，而是仅仅向一个线程发送中断请求，至于是否会被中断，要看被请求线程是否检查了中断标志位并对其做出响应**

#### 1.2 interrupt()的简单使用

##### 1.2.1 没有检查中断标志位

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            while (true) {
                System.out.println(1);
            }
        });
        thread.start();
        Thread.sleep(10);
        thread.interrupt();
    }
}
```

这种情况下，线程thread会一直进行输出，并没有响应主线程的中断请求

##### 1.2.2 检查中断标志位并做出响应

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            while (!Thread.currentThread().isInterrupted()) {
                System.out.println(1);
            }
        });
        thread.start();
        Thread.sleep(10);
        thread.interrupt();
    }
}
```

通过`Thread.currentThread().isInterrupted()`来检查中断标志位的状态，如果被设置为true，说明收到了中断请求，则退出循环。这段程序的运行结果是在输出10毫秒后程序终止

#### 1.3 sleep/join/wait状态下被中断导致唤醒

**如果被中断线程处于这3种状态时被请求中断，就会抛出一个InterruptedException中断异常，捕获中断异常后中断标志位会被恢复为false**，这就会导致无法通过中断标志位正确中断线程，若要正确响应则需要额外的代码

**思考：为什么发生中断异常要恢复中断标志位？**

因为中断标志位为true代表发生了一次中断，而`catch (InterruptedException e)`代表处理了一次中断，如果不将标志位重置为false，则一次中断会被重复处理（第二次处理发生在`while (!Thread.currentThread().isInterrupted())`）

下面用代码来体验一下

##### 1.3.1 sleep状态下被中断，会恢复中断标志位

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        MyThread t1 = new MyThread();
        t1.start();
        Thread.sleep(10);
        t1.interrupt();
    }
}

class MyThread extends Thread {
    @Override
    public void run() {
        while (!Thread.currentThread().isInterrupted()) {
            try {
                sleep(1000);
            } catch (InterruptedException e) {
                System.out.println("InterruptedException happened");
            }
            System.out.println(Thread.currentThread().isInterrupted());
        }

        System.out.println("t1 stopped");
    }
}
```

这里通过`Thread.currentThread().isInterrupted()`来检查中断标志位的状态，如果被设置为true，说明收到了中断请求，则退出循环。理论上讲可以退出循环，但实际结果是没有退出循环，输出如下

```
InterruptedException happened
false
false
false
false
...
```

 所以在这里就验证了中断标志位会被恢复这个情况

##### 1.3.2 如何正确处理中断请求

仍然用上面那个例子，只在catch语句中增加一行代码

```java
class MyThread extends Thread {
    @Override
    public void run() {
        while (!Thread.currentThread().isInterrupted()) {
            try {
                sleep(1000);
            } catch (InterruptedException e) {
                System.out.println("InterruptedException happened");
                Thread.currentThread().interrupt();
            }
            System.out.println(Thread.currentThread().isInterrupted());
        }

        System.out.println("t1 stopped");
    }
}
```

这里让线程捕获中断异常后再重新自己把中断标志位置为true，最终程序可以正常结束，输出如下

```
InterruptedException happened
true
t1 stopped
```

##### 1.3.3 用interrupt()唤醒等待中的线程

其实**interrupt()有时不仅不会中断线程，还能够唤醒等待中的线程**，上面用sleep举例，这里就用join举例

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("t1 stopped");
        });
        t1.start();
        
        Thread t2 = new Thread(() -> {
            try {
                t1.join();
            } catch (InterruptedException e) {
                System.out.println("InterruptedException happened");
            }
            System.out.println("t2 stopped");
        });
        t2.start();
        t2.interrupt();
    }
}
```

```
InterruptedException happened
t2 stopped
t1 stopped
```

可以看到，t2原本要等待t1睡眠3s之后再结束，但是却因为主线程对t2调用interrupt()导致了t2发生中断异常并提前结束，而t1仍然是在休眠3s后才输出"t1 stopped"。说明在对在join状态中的线程请求中断，可以使其停止等待，达到唤醒的效果

##### 1.3.4 wait与interrupt()

这里单纯为了演示一下wait状态下如何使用interrupt()

```java
public class Main {
    static Object object = new Object();
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            synchronized (object) {
                try {
                    object.wait();
                    System.out.println("t1 running");
                } catch (InterruptedException e) {
                    System.out.println("InterruptedException happened");
                }
            }
            System.out.println("t1 stopped");
        });
        t1.start();
        t1.interrupt();
    }
}
```

```
InterruptedException happened
t1 stopped
```

可以看到，虽然没有其他线程通过notify/notifyAll方法通知线程t1，但是t1恢复运行了

#### 1.4 死锁状态下被中断无响应

```java
public class Main {
    static Object lock1 = new Object();
    static Object lock2 = new Object();

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            deadLock(lock1, lock2);
        }, "t1");
        
        Thread t2 = new Thread(() -> {
            deadLock(lock2, lock1);
        }, "t2");
        
        t1.start();
        t2.start();
        
        // 睡眠时间要保证大于等于deadLock中的时间，不能让deadLock函数停留在sleep中
        // 而要让其运行到synchronized (lock2)，使其产生锁竞争导致死锁
        Thread.sleep(100);
        
        t1.interrupt();
        t2.interrupt();
    }

    public static void deadLock(Object lock1, Object lock2) {
        try {
            synchronized (lock1) {
                Thread.sleep(100); // 睡眠0.1s保证t2拿到另一个锁
                synchronized (lock2) {
                    System.out.println(Thread.currentThread());
                }
            }
        } catch (InterruptedException e) {
            System.out.println("InterruptedException happened");
        }
    }
}
```

在这段代码中的细节已经通过注释提示了，可以自己修改参数体会下

这里形成了一个死锁状态，再对其请求中断是没有相应的，这段程序没有任何输出也不会停止

#### 1.5 interrupted()方法

```java
public static boolean interrupted() {
    return currentThread().isInterrupted(true);
}
```

interrupted()是一个静态方法，它**返回调用线程的中断标志位的状态，并将中断标志位置为false**

比如一个线程此时中断标志位为true，调用interrupted()后返回true，并将中断标志位恢复为false；

如果一个线程此时中断标志位为false，调用interrupted()后仅返回false

| 特性         | interrupted() | isInterrupted() |
| ------------ | ------------- | --------------- |
| 是否 static  | ✅ 是          | ❌ 否            |
| **检测对象** | **当前线程**  | **任意线程**    |
| 是否清除标志 | ✅ 会清除      | ❌ 不清除        |
| 是否有副作用 | ✅ 有          | ❌ 无            |
| 常见用途     | 消费中断信号  | 轮询检查        |

（感觉interrupted()几乎是没什么用处的）

### 2. **shutdown()**或shutdownNow()

线程池可以调用shutdown()或者shutdownNow()方法关闭线程池，它们的原理是**遍历线程池中的工作线程，然后调用线程的interrupt()方法来中断线程**，所以无法响应中断的任务可能永远无法终止

## 七、协程与多线程

### 1. 多线程的缺陷

现代B/S系统中一次对外部业务请求的响应，往往需要分布在不同机器上的大量服务共同协作来实现，这种服务细分的架构在减少单个服务复杂度、增加复用性的同时，也不可避免地增加了服务的数量，缩短了留给每个服务的响应时间。这要求每一个服务都必须在极短的时间内完成计算，这样组合多个服务的总耗时才不会太长；也要求每一个服务提供者都要能同时处理数量更庞大的请求，这样才不会出现请求由于某个服务被阻塞而出现等待

Java目前的并发编程机制就与上述架构趋势产生了一些矛盾，1：1的内核线程模型是如今Java虚拟机线程实现的主流选择，但是这种映射到操作系统上的线程天然的**缺陷是切换、调度成本高昂，系统能容纳的线程数量也很有限**。以前处理一个请求可以允许花费很长时间在单体应用中，具有这种线程切换的成本也是无伤大雅的，但现在**在每个请求本身的执行时间变得很短、数量变得很多的前提下，用户线程切换的开销甚至可能会接近用于计算本身的开销，这就会造成严重的浪费**

### 2. 什么是协程

正如上面所说，多线程的缺陷是切换成本高，且线程数量也受内存限制，而协程可以弥补这两个问题。协程运行在线程之上，当一个协程执行完成后，可以选择主动让出，让另一个协程运行在当前线程之上。**协程并没有增加线程数量，只是在线程的基础之上通过时分复用的方式运行多个协程**，而且协程的切换在用户态完成，切换的代价比线程从用户态到内核态的代价小很多

### 3. 协程的注意事项

实际上协程并不是什么银弹，协程只有在等待IO的过程中才能重复利用线程，而线程在等待IO的过程中会陷入阻塞状态。假设协程运行在线程之上，并且协程调用了一个阻塞IO操作，但实际上操作系统并不知道协程的存在，它只知道线程。**因此在协程调用阻塞IO操作的时候，操作系统会让线程进入阻塞状态，当前的协程和其它绑定在该线程之上的协程都会陷入阻塞而得不到调度，这往往是不能接受的**

因此在协程中不能调用导致线程阻塞的操作，也就是说，**协程只有和异步IO结合起来，才能发挥最大的威力**

### 4. 协程的使用场景

即IO密集型业务

