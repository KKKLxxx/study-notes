# Executor多线程框架

一、Executor框架
---------------

### 1. Executor框架的作用

Executor是线程池的调度工具，线程池是Executor的一部分

### 2. Executor框架的结构

Executor框架由三大部分组成

- **线程池**：有两个关键实现类**ThreadPoolExecutor**和**ScheduledThreadPoolExecutor**
- **任务**：即被执行任务需要实现的接口：**Runnable**接口或**Callable**接口

- **异步计算的结果**：**Future**接口及其实现类**FutureTask**

### 3. Executor框架的执行过程

- 通过构造函数或工厂方法创建线程池

- 通过实现`Runnable`接口或`Callable`接口创建任务

- 通过`execute()`或`submit()`方法提交任务

- 通过`FutureTask.get()`获取返回结果（如果有）

二、线程池
----------------------

### 1. 线程池的优势

1、**降低资源消耗**：通过重复利用已创建的线程降低线程创建和销毁造成的消耗

2、**提高响应速度**：当任务到达时，任务可以不需要等到线程创建就能立即执行

3、**提高线程的可管理性**：线程是稀缺资源，如果无限制地创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一分配、调优和监控

### 2. 线程池的基本参数

```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler)
```

#### 2.1 corePoolSize（核心线程数）

当提交一个任务到线程池时，如果已创建线程数小于核心线程数，即使此时有空闲线程，线程池也会创建一个新的线程来执行任务

如果已创建线程数大于等于核心线程数就可以复用空闲线程

可以通过调用`prestartAllCoreThreads()`方法提前创建并启动所有核心线程

（当核心线程数设置为0时，所有任务都通过非核心线程执行）

#### 2.2 maximumPoolSize（最大线程数）

线程池允许创建的最大线程数

如果任务队列满了，并且已创建的线程数小于最大线程数，则线程池会创建新的线程执行任务

注意：**如果使用了无界的任务队列，则这个参数没有效果**

#### 2.3 keepAliveTime（超过corePoolSize的线程空闲后存活的时间）

超过`corePoolSize`的线程空闲后，如果在`keepAliveTime`时间内没有处理任务，会被销毁，而`corePoolSize`之内的线程不会被销毁

如果任务较多，但每个任务的执行时间较短，可以调大时间以提高线程利用率

#### 2.4 unit（keepAliveTime的时间单位）

#### 2.5 workQueue（任务队列）

用于保存等待执行的任务的阻塞队列，主要有4种

- **ArrayBlockQueue**：基于数组结构的有界队列
- **LinkedBlockQueue**：基于链表结构的无界队列
- **SynchronousQueue**：不存储元素的队列，每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态
- **PriorityBlockQueue**：具有优先级的无界队列

#### 2.6 threadFactory（创建线程的工厂）

比如可用自定义的线程工厂给创建出来的线程设置有意义的名字，也可以使用默认的线程工厂

#### 2.7 handler（拒绝策略）

当达到了最大线程数并且任务队列已满时，会对新添加的任务执行拒绝策略，有4种

- **AbortPolicy（默认）**：直接丢弃任务，并抛出异常
- **CallerRunsPolicy**：用提交任务的线程来运行该任务，可以用这个策略来**保证所有任务都被执行**，但可能阻塞主线程
- **DiscardOldestPolicy**：丢掉任务队列里最早的任务并重新添加该任务
- **DiscardPolicy**：直接丢弃

### 3. ThreadPoolExecutor的3种类型

ThreadPoolExecutor是最常用的线程池，通常使用工厂类Executors创建，有3种类型

- **FixedThreadPool**：固定线程数的线程池，适用于为了满足资源管理需要，而需要限制线程数量的场景，即负载较重的服务器

- **SingleThreadPool**：使用单个线程的线程池，保证顺序地执行各个任务

- **CachedThreadPool**：核心线程数为0，最大线程数无限，要搭配SynchronousQueue使用，适用于执行突发大量短期异步任务的场景，有耗尽资源的风险。CachedThreadPool适合处理突发流量，因为它平时不会保活线程，只在流量到达时创建尽可能多的线程，并复用这些线程处理多个短期任务，在流量过去后又会销毁

### 4. ScheduledThreadPoolExecutor的2种类型

ScheduledThreadPoolExecutor用于执行延迟任务或定期任务，有2种类型

- **ScheduledThreadPoolExecutor**：包含多个线程，适用于需要多个后台线程执行延迟或周期任务的场景

- **SingleThreadScheduledExecutor**：仅有一个线程，适用于需要单个后台线程执行延迟或周期任务，并且需要保证任务的顺序执行的场景

### 5. 线程池创建的注意事项

《阿里巴巴Java开发手册》中强制线程池不允许使用`Executors`去创建，而是通过`ThreadPoolExecutor`的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险

比如：

```java
ExecutorService fixedThreadPool = Executors.newFixedThreadPool(3);
```

而`newFixedThreadPool`底层的请求队列是无限长的`LinkedBlockingQueue`，可能导致OOM

```java
public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
    return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>(), threadFactory);
}
```

**推荐：使用构造函数自行指定所有参数**

```java
public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit,
                          BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)
```

### 6. 线程池的工作流程

可概括为以下4步

1、已创建线程数小于核心线程数，则创建新线程

2、线程数等于核心线程数，将新任务放入任务队列

3、任务队列满，创建新线程，上限为最大线程数

4、队列和线程池都满了，执行拒绝策略

**通过一个具体的例子来理解**：

假设核心线程数为5，最大线程数为10，任务队列最大容量为5

对于前5个任务，线程池会为每个任务创建一个新的线程来执行，无论是否有线程处于空闲状态

当5个核心线程都在工作，此时又来了5个线程，这新的5个线程都会在任务队列中等待

假设此时有1个核心线程任务执行完了，处于空闲状态，就会从任务队列中获取一个任务并执行

此时又来了2个任务，导致任务队列装不下了，则创建新线程，上限为最大线程数

如果任务队列满了并且也已经达到最大线程数，则对新添加的线程执行拒绝策略

### 7. 关闭线程池

可以通过对线程池调用**shutdown()**或者**shutdownNow()**方法关闭线程池

- **shutdown()**：已提交但未开始的任务仍会被调度，已经开始的任务会正常执行完毕
- **shutdownNow()**：已提交但未开始的任务会被撤销，已经开始的任务会被发送中断信号（但是否停止要看任务内部是否对中断信号正确响应）

### 8. 线程池配置建议

#### 8.1 核心线程数配置

核心线程数一般设置为(**最大线程数 / 5**)

#### 8.2 最大线程数配置

- **CPU密集型**

  这种情况下最大线程数设置为**n+1**即可，n为CPU核数（**处理器核心数/计算单元的个数**）。比CPU核心数多出来的一个线程是为了防止偶发的缺页中断，或者其它原因导致的任务暂停而带来的影响

  因为更多的线程也无法提高CPU利用率，反而会带来上下文切换的开销

- **IO密集型**

  一般认为可以将最大线程数设置为**2n**

  实际情况是要根据请求的IO时间相对总运行时间的占比来决定的。假如一个请求的总运行时间为2s，IO时间为1s，那么在不考虑上下文切换的情况下，将线程数设置为2n就可以充分利用CPU

#### 8.3 任务队列配置

建议使用有界任务队列

有界队列能够增加系统的稳定性和预警能力。比如当数据库出现慢查询导致连接数暴增时，有界队列可以及时执行拒绝策略以保证系统稳定性。如果是无界队列，那么这个队列会持续增长，可能撑满内存导致系统崩溃

三、任务
----------------------

### 1. Runnable与Callable的区别

|              | Runnable      | Callable                                                     |
| ------------ | ------------- | ------------------------------------------------------------ |
| **返回值**   | 没有返回值    | 有返回值，和Future、FutureTask配合可以用来获取异步执行的结果 |
| **异常**     | 无法抛出异常  | 可以抛出异常                                                 |
| **提交方式** | 通过execute() | 通过submit()                                                 |

**可以通过Executors工厂类将Runnable封装为一个Callable对象**

| 方法                                              | 描述                                 |
| ------------------------------------------------- | ------------------------------------ |
| Callable<Object> callable(Runnable task)          | 在原Runnable的基础上返回null         |
| <T> Callable<T> callable(Runnable task, T result) | 在原Runnable的基础上自定义一个返回值 |

### 2. 任务提交

任务提交的方式有两种：`execute()`与`submit()`

`execute()`执行后**没有返回结果**，只有1种用法

| 方法                           | 描述                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| void execute(Runnable command) | 常规使用方法，用没有返回值的execute执行没有返回值的Runnable任务 |

`submit()`执行后**有返回结果**，有3种用法

| **方法**                                      | **描述**                                                     |
| --------------------------------------------- | ------------------------------------------------------------ |
| <T> Future<T> submit(Callable<T> task)        | 常规使用方法，传入有返回值的callable任务，最终返回task的返回值 |
| <T> Future<T> submit(Runnable task, T result) | 由于Runnable没有返回值，但是可以自定义一个返回值用于返回     |
| Future<?> submit(Runnable task)               | 强行使用submit执行Runnable方法，忽略返回值                   |

四、Executor框架使用示例
----------------

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        // 创建线程池，线程工厂和拒绝策略可省略使用默认值
        ThreadPoolExecutor threadPool = new ThreadPoolExecutor(5, 10, 5,
                TimeUnit.SECONDS, new LinkedBlockingQueue<Runnable>(),
                Executors.defaultThreadFactory(), new ThreadPoolExecutor.AbortPolicy());

        // 通过execute方法用线程池执行一个任务
        threadPool.execute(new Runnable() {
            @Override
            public void run() {
                System.out.println("Task1");
            }
        });

        // 通过lambda化简
        threadPool.execute(() -> System.out.println("Task2"));

        // 通过submit方法可获取线程执行结果(结果即return的内容)
        Future<Object> future = threadPool.submit(() -> {return "Task3";});

        try {
            Object obj = future.get();
            System.out.println(obj);

            // 可设置超时时间
            // future.get(1, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            System.out.println("InterruptedException");
        } catch (ExecutionException e) {
            System.out.println("ExecutionException");
        } finally {
            // 关闭线程池
            threadPool.shutdown();
            // 或者使用shutdownNow()方法
            // threadPool.shutdownNow();

            // 如果调用了shutdown()或者shutdownNow()，那么isShutdown()就为真
            System.out.println(threadPool.isShutdown());

            // 只有当所有任务都关闭后isTerminated()才为真
            System.out.println(threadPool.isTerminated());
            // sleep1秒让线程池有足够时间关闭
            Thread.sleep(1);
            System.out.println(threadPool.isTerminated());
        }
    }
}
```

输出结果

```
Task1
Task2
Task3
true
false
true
```

