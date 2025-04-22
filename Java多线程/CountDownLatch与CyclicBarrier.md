# CountDownLatch与CyclicBarrier

一、CountDownLatch
----------------

### 1、作用

`CountDownLatch`用于等待一个或多个线程完成操作，类似于`join()`。**`CountDownLatch`使用起来没有`join()`方便，但是可以提供比`join()`更细粒度的控制**（下面会举例子）

### 2、使用示例

```java
public class Main {
    static CountDownLatch countDownLatch = new CountDownLatch(2);

    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            System.out.println(1);
            countDownLatch.countDown();
            System.out.println(2);
            countDownLatch.countDown();
        }).start();
        countDownLatch.await();
        System.out.println(3);
    }
}
```

首先将计数器初始化为2，在子线程中，每输出一个数字，通过`countDown()`方法将`countDownLatch`的计数减一。主线程中通过`await()`方法等待，直到计数器为0才会返回，当然也提供了超时等待方法

### 3、实现原理

`CountDownLatch`是共享锁的一种实现，它默认构造AQS的`state`值为`count`。当线程使用`countDown()`方法时，其实使用了`tryReleaseShared()`方法以CAS的操作来减少`state`，直至`state`为0。当调用`await()`方法的时候，如果`state`不为0，那就证明任务还没有执行完毕，`await()`方法就会一直阻塞，也就是说`await()`方法之后的语句不会被执行。然后，`CountDownLatch`会自旋CAS判断`state == 0`，如果`state == 0`的话，就会释放所有等待的线程，`await()`方法之后的语句得到执行

```java
// CountDownLatch同步器部分的源码
private static final class Sync extends AbstractQueuedSynchronizer {
    private static final long serialVersionUID = 4982264981922014374L;

    Sync(int count) {
        setState(count);
    }

    int getCount() {
        return getState();
    }

    protected int tryAcquireShared(int acquires) {
        return (getState() == 0) ? 1 : -1;
    }

    protected boolean tryReleaseShared(int releases) {
        // Decrement count; signal when transition to zero
        for (;;) {
            int c = getState();
            if (c == 0)
                return false;
            int nextc = c-1;
            if (compareAndSetState(c, nextc))
                return nextc == 0;
        }
    }
}
```

### 4、更细粒度的控制

如果上面的示例用`join()`实现，会有一个问题：如果我们想实现子线程输出1后即可让主线程停止等待，然后让主线程子线程并行执行，那么将计数器初始化为1即可；但如果使用`join()`，那么就只能等子线程中的全部语句执行完才能返回

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            System.out.println(1);
            System.out.println(2);
        });
        t1.start();
        t1.join();
        System.out.println(3);
    }
}
```

二、CyclicBarrier
---------------

### 1、作用

`CyclicBarrier`与`CountDownLatch`大同小异，也是通过计数来等待线程。**计数初始化为0，到达指定值时会自动重置为0**

### 2、与CountDownLatch的区别

1、`CyclicBarrier`是增加计数，`CountDownLatch`是减少计数

2、`CyclicBarrier`是可通过`reset()`循环使用，`CountDownLatch`是一次性的

3、`CyclicBarrier`可以提供一个方法在计数完成后指定执行一个任务，`CountDownLatch`没有

### 3、使用示例

```java
public class CyclicBarrierTest implements Runnable {
    CyclicBarrier cyclicBarrier = new CyclicBarrier(4, this);
    AtomicInteger atomicInteger = new AtomicInteger(0);

    public void test() {
        for (int i = 0; i < 4; ++i) {
            int finalI = i;
            new Thread(() -> {
                atomicInteger.incrementAndGet();
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }

    @Override
    public void run() {
        System.out.println(atomicInteger);
    }

    public static void main(String[] args) throws InterruptedException {
        CyclicBarrierTest cyclicBarrierTest = new CyclicBarrierTest();
        cyclicBarrierTest.test();
    }
}
```