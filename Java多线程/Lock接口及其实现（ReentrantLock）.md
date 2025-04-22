# Lock接口及其实现（ReentrantLock）

一、Lock接口与synchronized
---------------------

`Lock`接口的功能与`synchronized`类似，都是用来控制多个线程访问共享资源的方式。**`synchronized`的优势在于隐式获取释放锁的便捷性**，不用考虑异常等特殊情况；但是**`Lock`也有`synchronized`不具备的特性，能够更灵活地使用**

| **特性**               | **描述**                                                     |
| ---------------------- | ------------------------------------------------------------ |
| **尝试非阻塞地获取锁** | 当前线程尝试获取锁，如果这一时刻锁没有被其他线程获取到，则成功获取并持有锁 |
| **超时获取锁**         | 在指定的截止时间之前获取锁，如果超时仍无法获取锁，则返回     |
| **获取锁时可被中断**   | **`synchronized`在获取锁时无法被中断**，而`Lock`可以通过调用`lockInterupt()`方法可中断地获取锁 |

**Lock的常用API**

| **方法名称**                                                 | **描述**                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **void lock()**                                              | 获取锁，当锁获得后，会从该方法返回                           |
| **boolean tryLock()**                                        | 尝试非阻塞地获取锁，调用后立即返回，如果获取锁成功，则返回true，否则返回false |
| **boolean tryLock(long time, TimeUnit unit) throws InterruptedException** | 超时地获取锁，会在以下3种情况下返回<br />1、在超时时间内获得了锁<br />2、在超时时间内被中断<br />3、超时 |
| **void lockIntettuptibly() throws InterruptedException**     | 可中断地获取锁，即在锁的获取过程中可以中断当前线程           |
| **void unlock()**                                            | 释放锁                                                       |
| **Condition newCondition()**                                 | 获取等待通知组件，该组件和当前的锁绑定，当前线程只有获得了锁，才能调用该组件的wait()方法，调用后，该线程将释放锁 |

二、Lock接口的实现——AQS
----------------

### 1、AQS的作用

抽象队列同步器（**A**bstract**Q**ueued**S**ynchronizer）是`Lock`接口实现的基础，AQS**用一个`volatile int`成员变量表示同步状态，通过内置的FIFO队列来完成资源获取线程的排队工作**

AQS的主要使用方式是**继承**，**子类通过继承AQS并实现它的抽象方法来管理同步状态，在抽象方法的实现过程中免不了要对同步状态进行更改，这时就需要使用AQS提供的3个方法**

| **方法名**                                     | **作用**                                  |
| ---------------------------------------------- | ----------------------------------------- |
| **getState()**                                 | 获取当前同步状态                          |
| **setState(int newState)**                     | 设置当前同步状态                          |
| **compareAndSetState(int expect, int update)** | 使用CAS设置当前状态，保证状态设置的原子性 |

**这些方法保证状态的改变是安全的**

**子类推荐被定义为自定义同步组件的静态内部类**，其中**自定义同步组件就是Lock接口的实现（具体的某一种锁）**，比如`ReentrantLock`。同步器自身没有实现任何同步接口，它仅仅是定义了若干同步状态获取和释放的方法来供锁使用

锁与同步器之间的关系可以这样理解：

**锁是面向使用者的，它定义了使用者与锁交互的接口，隐藏了实现细节**

**同步器是面向锁的实现者的，它简化了锁的实现方式，屏蔽了同步状态管理、线程的排队、等待与唤醒等底层操作**

### 2、锁的实现示例

实现分为2步：

**1、继承同步器(AQS)并重写指定方法，并以静态内部类的形式组合在锁(比如ReentrantLock)中**

**2、实现Lock接口的方法（也可以新增一些方法），这些方法会调用自定义同步器中的方法**

**使用者在使用时仅仅调用Lock接口提供的方法和实现者在锁中新增的方法，这些方法在底层会调用实现者在静态内部类中重写的方法（这里建议看看ReentrantLock的源码去理解，篇幅限制不在此处复制）**

也就是说，使用者调用的接口几乎不变，但是会根据锁的类型（如公平与非公平，独占与共享）的不同达到不同的效果

一个锁的实现示例

```java
class Mutex implements Lock {
    // 继承同步器并重写指定方法，并以静态内部类的形式组合在自定义同步组件中
    private static class Sync extends AbstractQueuedSynchronizer {
        // 当状态为0时获取锁
        protected boolean tryAcquire(int acquires) {
            if (compareAndSetState(0, 1)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        // 返回一个condition，每个condition都包含了一个condition队列
        public Condition newCondition() {
            return new ConditionObject();
        }
    }

    // 将操作代理到sync上
    private final Sync sync = new Sync();

    @Override
    public void lock() {
        sync.acquire(1);
    }

    @Override
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }

    @Override
    public boolean tryLock() {
        return sync.tryAcquire(1);
    }

    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(time));
    }

    @Override
    public void unlock() {
        sync.release(1);
    }

    @Override
    public Condition newCondition() {
        return sync.newCondition();
    }
}
```

三、ReentrantLock
---------------

`ReentrantLock`是`Lock`接口最经典的实现类，它支持可重入以及公平/非公平的选择，下面说说这2个特性的实现方法

### 1、可重入

可重入就是指**支持一个线程对资源重复加锁**，对于不支持可重入的锁，当一个线程占有锁之后再次对这个锁调用`lock()`，会持续阻塞，因为`lock()`调用的自定义同步器的`acquire()`方法并没有考虑重复加锁的情况，获取不到锁就会一直阻塞，比如上面自己实现的`Mutex`

实现可重入需要考虑以下2点：

1、**线程再次获取锁**：锁需要识别获取锁的线程是否为当前占据锁的线程，如果是，则再次成功获取

2、**锁的最终释放**：通过计数表示一个线程对一个锁的获取次数，一个线程重复n次获取了锁，那么计数为n，每释放一次计数减一，当计数等于0时表示锁成功释放

以`ReentrantLock`非公平模式（默认）下获取锁、释放锁相关函数的实现为例

```java
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState(); // 获取同步状态
    if (c == 0) { // 如果没有线程获取锁，则直接加锁
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    } else if (current == getExclusiveOwnerThread()) { // 否则判断当前线程是否是获取锁的线程
        int nextc = c + acquires; // 重复获取锁，则将计数增加
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}

protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) { // 计数为0，则释放锁
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```

可以看到，加锁时判断了当前持有锁的线程是否为调用线程本身，并且通过`nextc`实现计数；解锁时判断了计数是否为0，如果为0才能释放

### 2、公平与非公平

公平即按照线程的排队顺序分配资源的使用权，**越先开始等待的线程越早得到资源**，这种机制能够避免某一线程长时间得不到资源的情况发生。然而**公平锁的效率往往比非公平锁低很多**。因为一个线程释放锁后再次获取锁的可能性非常大，如果连续多次获取一个锁的线程都是同一个线程，就避免了线程切换的开销，但代价就是有的线程可能被“饿死”

非公平锁的实现只体现在获取锁的过程中，以`ReentrantLock`公平模式下获取锁相关函数的实现为例

```java
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (!hasQueuedPredecessors() && compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    } else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

其实与非公平模式下的代码只有一点不同，就是判断条件多了一个`!hasQueuedPredecessors()`。这个函数用来判断当前线程在同步队列中是否有前驱节点，如果是，才可以获得锁。这个同步队列就是AQS提供的FIFO队列，用来完成资源获取线程的排队工作

