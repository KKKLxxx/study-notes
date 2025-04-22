# 为什么不能直接用run()启动线程

通过代码测试一下

```
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

