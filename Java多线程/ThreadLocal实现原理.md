# ThreadLocal实现原理

一、ThreadLocal的作用
----------------

ThreadLocal，即线程本地变量。如果创建了一个threadLocal变量，那么多个线程操作这个变量的时候，实际是操作自己本地内存里面的变量，从而**起到线程隔离的作用，避免了线程安全问题**

其次ThreadLocal还可以**减少传参**，比如

```java
public class Main {
    static ThreadLocal<Long> threadLocal = new ThreadLocal<>();

    public void func(Long time) {
    }
    
    // 通过threadLocal减少了参数的传递
    public void func() {
        Long time = threadLocal.get();
    }
}
```

通过ThreadLocal减少了参数的传递，这在参数较多时效果更明显

二、ThreadLocal实现原理
------------------

首先观察Thread这个类的源码，可以看到里面有一个属性是ThreadLocal.ThreadLocalMap threadLocals

```java
public class Thread implements Runnable {
    ThreadLocal.ThreadLocalMap threadLocals = null;
}
```

所以**每个线程都持有一个ThreadLocal.ThreadLocalMap（以下简称ThreadLocalMap）类型的对象，这是线程隔离的基础**

举个使用的例子

```java
public class Main {
    static ThreadLocal<Long> threadLocal = new ThreadLocal<>();

    public static void main(String[] args) throws InterruptedException {
        threadLocal.set(System.currentTimeMillis());
        doWork();
        Long time = System.currentTimeMillis() - threadLocal.get();
        System.out.println(time);
    }

    public static void doWork() throws InterruptedException {
        Thread.sleep(1000);
    }
}
```

这段代码中展示了ThreadLocal中set()与get()方法的使用，这两个方法也是最常用的，查看一下源码

```java
public void set(T value) {
    Thread t = Thread.currentThread(); 	// 获取当前线程
    ThreadLocalMap map = getMap(t); 	// 获取当前线程的ThreadLocalMap
    if (map != null)
        map.set(this, value); 			// 以threadLocal对象为key，添加键值对
    else
        createMap(t, value);
}
```

```java
public T get() {
    Thread t = Thread.currentThread(); 					// 获取当前线程
    ThreadLocalMap map = getMap(t); 					// 获取当前线程的ThreadLocalMap
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this); 	// 以threadLocal对象为key，获取value
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

ThreadLocalMap以调用get()或set()方法的那个ThreadLocal对象为key，添加键值对或获取对应的值

三、ThreadLocalMap底层结构
--------------------

其实类似于HashMap，底层结构是数组+链表（但没有红黑树），查看部分源码

```java
static class ThreadLocalMap {

    static class Entry extends WeakReference<ThreadLocal<?>> {
        /**
         * The value associated with this ThreadLocal.
         */
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }

    private Entry[] table;

    private Entry getEntry(ThreadLocal<?> key) {
        int i = key.threadLocalHashCode & (table.length - 1);
        ThreadLocal.ThreadLocalMap.Entry e = table[i];
        if (e != null && e.get() == key)
            return e;
        else
            return getEntryAfterMiss(key, i, e);
    }
}
```

用一个Entry类型的数组保存键值对，然后通过链地址法解决哈希冲突，确定数组下标的代码是这行

```java
int i = key.threadLocalHashCode & (table.length - 1);
```

即用key的哈希码 与 (数组长度- 1)做一个与运算，如果发现冲突会进行其他处理

四、ThreadLocal内存泄露
-----------------

### 1. 对于key的内存泄露

因为ThreadLocalMap的生命周期与它所在的线程相同，**如果**作为key的threadLocal对象的引用存入了ThreadLocalMap中，并且是一个**强引用**，那么即使threadLocal对象在外部没有了强引用链，也会因为ThreadLocalMap中存在一条与threadLocal对象的强引用链而一直存在这个键值对，且无法访问，所以造成了内存泄露

**ThreadLocalMap通过弱引用来解决这个问题**

可以看到Entry类的源码，它对于threadLocal是一个弱引用

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /**
     * The value associated with this ThreadLocal.
     */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    } 
}
```

**弱引用：在GC时，只要发现弱引用，不管堆空间是否足够，都会将对象进行回收**

这样保证了只要外部的强引用链断了，这个threadLocal对象就只剩一个内部的弱引用链，在下次GC时就可以被回收

**如果threadLocal在使用前就被回收了怎么办？**

其实这是一种不会发生的情况

- 如果threadLocal是一个静态变量，那么它不会被回收
- 如果是局部变量，那么在对应作用域内，threadLocal对象也存在强引用，也不应该被回收

而弱引用就是为了解决threadLocal为局部变量的情况下，threadLocal引用已经出栈，但仍存在“线程-ThreadLocalMap-threadLocal对象”这条引用链的情况（注意区分“threadLocal引用”与“threadLocal对象”，引用在栈中，对象在堆中）

### 2. 对于value的内存泄露

上面讲过ThreadLocalMap通过弱引用解决了key的内存泄露问题，但是value并不会随着key的回收而回收，不过ThreadLocalMap也提供了解决办法

**ThreadLocal中提供了remove()方法，用来断开value对象的引用链。并且每次调用get()和set()方法也会检查部分key为null的entry，也可以及时释放内存，不过最好保证在每次用完之后通过remove()方法释放，因为有些key可能不会扫描到**

**可以使用 `try-finally` 块可以确保即使发生异常，`remove()` 方法也一定会被执行**

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ThreadLocalRemoveDemo {

    private static final ThreadLocal<String> threadLocal = new ThreadLocal<>();

    public static void main(String[] args) {

        ExecutorService pool = Executors.newFixedThreadPool(1);

        for (int i = 1; i <= 3; i++) {
            int taskId = i;

            pool.submit(() -> {
                try {
                    threadLocal.set("任务" + taskId);

                    System.out.println(
                        Thread.currentThread().getName()
                        + " 设置数据: "
                        + threadLocal.get()
                    );
                } finally {
                    // ⭐ 非常重要：任务结束必须清理
                    threadLocal.remove();
                    System.out.println(Thread.currentThread().getName() + " 已 remove ThreadLocal");
                }
            });
        }

        pool.shutdown();
    }
}

```

```
pool-1-thread-1 设置数据: 任务1
pool-1-thread-1 已 remove ThreadLocal
pool-1-thread-1 设置数据: 任务2
pool-1-thread-1 已 remove ThreadLocal
pool-1-thread-1 设置数据: 任务3
pool-1-thread-1 已 remove ThreadLocal
```

## 五、在线程池中使用ThreadLocal

### 1. 脏数据问题

因为线程池中的线程是复用的，如果一个任务执行完后没有调用remove()进行清理，那么下个任务可能读取到上一个任务的脏数据，导致逻辑错误

### 2. 解决方法

- 及时使用remove()清理
- 使用TransmittableThreadLocal：TTL能够自动清空每个任务设置的ThreadLocal对象
