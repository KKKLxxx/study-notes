# interrupt()与interrupted()

一、interrupt()的作用
----------------

interrupt()方法用于**向一个线程发送中断请求，请求结果是将被请求线程的中断标志位设置为true**。需要注意的是，**interrupt()并不会直接中断线程，而是仅仅向一个线程发送中断请求，至于是否会被中断，要看被请求线程是否检查了中断标志位并对其做出响应**

二、interrupt()的简单使用
------------------

### （一）没有检查中断标志位

```
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

### （二）检查中断标志位并做出响应

```
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

通过Thread.currentThread().isInterrupted()来检查中断标志位的状态，如果被设置为true，说明收到了中断请求，则退出循环。这段程序的运行结果是在输出10毫秒后程序终止

三、sleep/join/wait状态下被中断导致唤醒
---------------------------

我认为关于interrupt()最难理解的地方应该是这一点，**如果被中断线程处于这3种状态时被请求中断，就会抛出一个InterruptedException中断异常，抛出中断异常后中断标志位会被恢复为false**，这就会导致无法正确响应中断请求，若要正确响应则需要额外的代码

**思考：为什么发生中断异常要恢复中断标志位？**

其实我也搞不懂，可能人家就是这么设计的

下面用代码来体验一下

### （一）sleep状态下被中断，会恢复中断标志位

```
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

这里通过Thread.currentThread().isInterrupted()来检查中断标志位的状态，如果被设置为true，说明收到了中断请求，则退出循环。理论上讲可以退出循环，但实际结果是没有退出循环，输出如下

```
InterruptedException happened
false
false
false
false
...
```

 所以在这里就验证了中断标志位会被恢复这个情况

### （二）如何正确处理中断请求

仍然用上面那个例子，只在catch语句中增加一行代码

```
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

### （三）用interrupt()唤醒等待中的线程

其实**interrupt()有时不仅不会中断线程，还能够唤醒等待中的线程**，上面用sleep举例，这里就用join举例

```
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

### （四）wait与interrupt()

这里单纯为了演示一下wait状态下如何使用interrupt()

```
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

四、死锁状态下被中断无响应
-------------

```
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

五、interrupted()方法
-----------------

```
public static boolean interrupted() {
    return currentThread().isInterrupted(true);
}
```

interrupted()是一个静态方法，它**返回调用线程的中断标志位的状态，并将中断标志位置为false**

比如一个线程此时中断标志位为true，调用interrupted()后返回true，并将中断标志位恢复为false；

如果一个线程此时中断标志位为false，调用interrupted()后仅返回false