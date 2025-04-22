# 等待、通知机制（wait, notify）

有这样一个场景，线程A只有当满足一个条件时才能继续进行，而这个条件要由线程B去改变，比如

```
// 线程A
while (flag == 0) {
    continue; // 循环等待直到条件满足
}
doSomething();


// 线程B
flag = 1;
```

只有当线程B执行flag = 1后，线程A才可以跳出循环。这里涉及到的一个问题就是如果线程A一直循环，会持续无效地消耗CPU资源

有一种解决办法，可以让线程A不满足条件时先睡眠一段时间，比如

```
// 线程A
while (flag == 0) {
    Thread.sleep(1000); // 每次检验条件不满足就睡1s
}
```

这样便能够避免浪费CPU资源，但是这个方法存在2个问题

1、**及时性差**：如果睡眠时间过久，会导致不能及时发现条件改变

2、**难以降低开销**：如果为了保证及时性而将睡眠时间缩到很短，线程切换的代价也会变大，整体并不一定可以降低开销

* * *

以上问题可以通过等待、通知机制解决

等待、通知相关的方法是任意Java对象都具备的，因为这些方法被定义在所有对象的超类java.lang.Object上，类似于synchronized

首先介绍几个函数

| **函数名**                        | **描述**                                                     |
| --------------------------------- | ------------------------------------------------------------ |
| **wait()**                        | 调用该方法的线程进入WAITING状态，只有等待另外线程的通知或被中断才会返回。**调用wait()方法后，会释放对象的锁** |
| **wait(long timeout)**            | 超时等待，单位是毫秒。如果超过timeout时间还没有通知则自动返回 |
| **wait(long timeout, int nanos)** | 将超时时间控制到纳秒                                         |
| **notify()**                      | 通知一个在对象上等待的线程，使其从wait()方法返回，**返回的前提是该线程获取到了对象的锁** |
| **notifyAll()**                   | 通知所有等待该对象的线程，**本次没有获取锁的线程会留在阻塞队列中等待下一次锁释放时竞争锁** |

用一段代码来演示用法

```
public class Main {
    static boolean flag = true;
    static Object lock = new Object();

    public static void main(String[] args) {
        Thread waitThread = new Thread(new Wait(), "WaitThread");
        waitThread.start();
        Thread notifyThread = new Thread(new Notify(), "NotifyThread");
        notifyThread.start();
    }

    static class Wait implements Runnable {
        @Override
        public void run() {
            synchronized (lock) {
                // 当条件不满足时，保持wait状态，同时释放锁
                while (flag) {
                    try {
                        System.out.println(Thread.currentThread() + " flag is true. wait");
                        lock.wait();
                    } catch (InterruptedException e) {
                    }
                }
                // 条件满足后，完成剩余工作
                System.out.println(Thread.currentThread() + " flag is false. running");
            }
        }
    }

    static class Notify implements Runnable {
        @Override
        public void run() {
            synchronized (lock) {
                // 获取锁并进行通知，通知时不会释放锁
                // 直到当前线程释放锁后，waitThread才能从wait方法中返回
                System.out.println(Thread.currentThread() + " hold lock. notify");
                flag = false;
                lock.notifyAll();
            }
            synchronized (lock) {
                // 此语句先于waitThread的"running"语句执行，说明该线程没有立即释放锁
                System.out.println(Thread.currentThread() + " hold lock again.");
            }
        }
    }
}


```

执行结果

```
Thread[WaitThread,5,main] flag is true. wait
Thread[NotifyThread,5,main] hold lock. notify
Thread[NotifyThread,5,main] hold lock again.
Thread[WaitThread,5,main] flag is false. running
```

这段代码演示的内容是：WaitThread首先获取到lock对象的锁（以下简称锁），并输出一条语句，随后将该线程放入等待队列，并释放锁。随后NotifyThread获取锁，输出一条语句后**通过notifyAll()将等待队列中的线程移至阻塞队列**，并修改条件。注意此时NotifyThread并没有释放锁，可以看到结果的第3行，仍然是NotifyThread输出的内容，这说明**通知线程调用notify/notifyAll方法时并不会立即释放锁**，锁释放的时间仍然是由程序本身和系统调用控制的。等到WaitThread获取锁后，发现flag已经为false，便可以跳出循环，继续执行之后的语句

* * *

**辨析两个概念：阻塞与等待**

**阻塞**：每次锁释放时，处于阻塞队列中的线程都会进行一次锁竞争

**等待**：处于等待队列中的线程不会参与锁竞争，只有当其他线程通过notify/notifyAll通知后，才会将等待队列中的线程转移到阻塞队列，再参与锁竞争

**等待队列存在的意义**：上面可以看到，调用notifyAll()方法的时机是将flag修改为false之后。在修改flag之前，线程A完全没有必要参与锁竞争，因为即使获取了锁也会因为不满足条件而无限循环，这样白白浪费了资源。所以**等待队列可以减少不必要的锁竞争**