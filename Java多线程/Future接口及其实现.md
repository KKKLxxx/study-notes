# Future接口及其实现

## 一、Future

### 1. Future的作用

在 Java 中，`Future`是一个泛型接口，主要用于执行异步任务，其中定义了 5 个方法：

```java
// V 代表了Future执行的任务返回值的类型
public interface Future<V> {
    
    // 取消任务执行。成功取消返回 true，否则返回 false
    boolean cancel(boolean mayInterruptIfRunning);
    
    // 判断任务是否被取消
    boolean isCancelled();
    
    // 判断任务是否已经执行完成
    boolean isDone();
    
    // 获取任务执行结果
    V get() throws InterruptedException, ExecutionException;
    
    // 指定时间内没有返回计算结果就抛出 TimeOutException 异常
    V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
}
```

简单理解就是：我有一个任务，提交给了 `Future` 来处理。任务执行期间我自己可以去做任何想做的事情。并且，在这期间我还可以取消任务以及获取任务的执行状态。一段时间之后，我就可以 `Future` 那里直接取出任务执行结果

### 2. Callable与Future的关系(FutureTask)

我们可以通过 `FutureTask` 来理解 `Callable` 和 `Future` 之间的关系

`FutureTask` 提供了 `Future` 接口的基本实现，常用来封装 `Callable` 和 `Runnable`，具有取消任务、查看任务是否执行完成以及获取任务执行结果的方法。`ExecutorService.submit()` 方法返回的其实就是 `Future` 的实现类 `FutureTask` 

```java
<T> Future<T> submit(Callable<T> task);
Future<?> submit(Runnable task);
```

`FutureTask` 不光实现了 `Future`接口，还实现了`Runnable` 接口，因此可以作为任务直接被线程执行

![img](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202602112005550.jpeg)

`FutureTask` 有两个构造函数，可传入 `Callable` 或者 `Runnable` 对象。实际上，传入 `Runnable` 对象也会在方法内部转换为`Callable` 对象

```java
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;
}

public FutureTask(Runnable runnable, V result) {
    // 通过适配器RunnableAdapter来将Runnable对象runnable转换成Callable对象
    this.callable = Executors.callable(runnable, result);
    this.state = NEW;
}
```

`FutureTask`相当于对`Callable`进行了封装，管理着任务执行的情况，存储了 `Callable` 任务的执行结果

## 二、CompletableFuture

### 1. CompletableFuture的作用

`CompletableFuture`相比`Future`的主要优点在于，支持异步任务的编排、回调式处理与灵活的异常处理

- **异步任务的编排**
  - `CompletableFuture`能够将通过`allOf()`并行运行多个任务，或通过`thenCompose()`方法指定多个任务的执行顺序
  - `Future`只能分别提交每个任务
- **回调式处理**
  - `CompletableFuture`能够通过`thenRun()/thenApply()`等指定一个任务完成后的回调函数，不阻塞当前线程
    - 但通常还是要加个`join()`等待任务执行完再返回结果
    - `thenRun()/thenApply()`的区别在于，前者不能访问异步任务的返回结果，后者可以
    - 这点其实也属于任务编排
  - `Future`只能通过`join()/get()`阻塞当前线程，等待任务执行完成
- **异常处理**
  - `CompletableFuture`能够在异步任务执行的过程中，通过`handle()`等方法由调用者处理异常
  - `Future`只能在异步任务中自行处理异常或在最终通过`get()`获取结果时由调用者处理异常

### 2. 使用示例

**任务提交方法**：

- `runAsync()`：无返回值
- `supplyAsync()`：有返回值

假设一个场景：任务T3需要在T1和T2完成之后再执行

```java
// T1
CompletableFuture<Void> futureT1 = CompletableFuture.runAsync(() -> {
    System.out.println("T1 is executing. Current time：" + DateUtil.now());
    ThreadUtil.sleep(1000);		// 模拟耗时操作
});

// T2
CompletableFuture<Void> futureT2 = CompletableFuture.runAsync(() -> {
    System.out.println("T2 is executing. Current time：" + DateUtil.now());
    ThreadUtil.sleep(1000);
});

// 使用allOf()方法合并T1和T2的CompletableFuture，等待它们都完成
CompletableFuture<Void> bothCompleted = CompletableFuture.allOf(futureT1, futureT2);

// 当T1和T2都完成后，执行T3
bothCompleted.thenRunAsync(() -> System.out.println("T3 is executing after T1 and T2 have completed.Current time：" + DateUtil.now()));

// 等待所有任务完成，验证效果
ThreadUtil.sleep(3000);
```

### 3. 使用CompletableFuture时为什么要自定义线程池？

`CompletableFuture` 默认使用全局共享的线程池，但自定义线程池可以根据任务类型选择不同的线程数、任务队列与拒绝策略，使任务能够高效、可靠地执行

但并不是每个场景都要自建一个线程池，否则会导致占用过多资源去存储线程、上下文切换严重。应该按照任务类型去复用有限个线程池，比如CPU密集型任务与IO密集型任务各一个线程池
