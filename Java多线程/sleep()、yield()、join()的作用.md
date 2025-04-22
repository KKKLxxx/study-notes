# sleep()、yield()、join()的作用

## 一、sleep()

```
public static native void sleep(long millis) throws InterruptedException;
```

通过调用sleep()方法可以**使当前线程休眠，进入等待状态**。**如果线程在等待状态被中断，将会抛出InterruptedException中断异常**

sleep()是一个静态方法，**只能够让当前线程休眠，而不能休眠其他线程**

二、yield()
---------

```
public static native void yield();
```

yield()是一个静态方法，作用是**让出当前线程对CPU的占有权，触发一次线程间的竞争**。在这次竞争中，**获得CPU的线程可能还是它自己，也可能是别的线程**

**使用场景**：

**场景1**：如执行某项复杂的任务时，如果担心占用资源过多，可以在完成某个重要的工作后使用yield()方法让掉当前CPU的调度权，等下次获取到再继续执行，这样不但能完成自己的重要工作，也能给其他线程一些运行的机会，**避免一个线程长时间占有CPU资源**

**场景2**：当A、B两线程为合作线程时，B需要等待A数据处理完成后才可以进行，当B获取到了CPU，却检测到A数据未处理完成时可立即触发重新竞争，以免浪费CPU资源

三、join()
--------

```
public final void join() throws InterruptedException {
    join(0);
}
```

通过调用join()方法可以**让当前线程挂起，进入等待状态，等待某个线程执行完成**。**如果线程在join状态被中断，将会抛出InterruptedException中断异常**

通过一个例子演示

```
public class Main {
    public static void main(String[] args) {

        Thread t1 = new Thread(() -> System.out.println("t1 stopped"));

        Thread t2 = new Thread(() -> {
            try {
                t1.join();
                Thread.sleep(1000); // 即使t2休眠1s，t3也会等待它执行完成
                System.out.println("t2 stopped");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        Thread t3 = new Thread(() -> {
            try {
                t2.join();
                System.out.println("t3 stopped");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        t1.start();
        t2.start();
        t3.start();
    }
}
```

输出结果

```
t1 stopped
t2 stopped
t3 stopped
```

这里实现了一个t2等t1，t3等t2的效果，并且让t2休眠了1s，t3也仍然继续等待t2执行结束

四、sleep()与yield()的区别
--------------------

可以将sleep分为2种情况

### （一）sleep(0)

这个语句与yield()的作用是一样的，都是让出当前线程对CPU的占有权

### （二）sleep(x)，其中x > 0

这种用法与yield()就是有区别的，区别在于

1、sleep期间可以响应中断，而yield()执行之后就再中断这个线程就没有效果

2、sleep可以指定该线程在x时间内不参与CPU竞争，而yield之后的线程可以立即参与CPU竞争

五、sleep/yield/join/wait对于锁和CPU的释放
---------------------------------

| **方法**    | **锁** | **CPU** |
| ----------- | ------ | ------- |
| **sleep()** | 不释放 | 释放    |
| **yield()** | 不释放 | 释放    |
| **join()**  | 释放   | 释放    |
| **wait()**  | 释放   | 释放    |