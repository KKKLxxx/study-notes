# Monitor Ctrl-Break线程

一段《深入理解Java虚拟机》里经典代码

```
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

运行之后发现死活停止不了，后来百度发现在**IDEA的Run模式**下，会多出来一个Monitor Ctrl-Break线程，导致最终存活的线程数量为2

检验方法为

```
Thread.currentThread().getThreadGroup().list();
```

就可以打印出来当前的所有线程

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/3216b2ef787d43258b8f57236a6c0ef9.png)

不过在Debug模式中不会有这个问题，为了程序的正常运行，可以将存活的线程数暂时改为2