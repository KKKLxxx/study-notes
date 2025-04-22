# CAS的缺陷及解决方案

一、CAS机制
-------

CAS（Compare-And-Swap）即比较并交换，是基于CPU的CAS指令来实现的，提供原子化的读写能力

**CAS的思想**：三个参数，一个当前内存值V、旧的预期值A、即将更新的值B，当且仅当预期值A和内存值V相同时，将内存值修改为B并返回true，否则什么都不做，并返回false

**底层原理**：Java的CAS操作会调用一个native方法，最终依靠CPU的CAS操作完成。因为CPU的CAS操作一种原子操作，所以可以保证Java的CAS操作也是一个原子操作

```java
public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}

public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
```

## 二、缺陷及解决方案

### 1、ABA问题

ABA问题是指：并发环境下，假设初始条件是A，去修改数据时，发现是A就会执行修改。但是看到的虽然是A，中间可能发生了A变B，B又变回A的情况。此A已经非彼A，数据即使成功修改，也可能有问题

**解决方案**：通过AtomicStampedReference添加版本号，即使值相同也可以通过版本号来识别

```java
public class Main {
    public static void main(String[] args) {
        AtomicStampedReference<String> atomicStampedReference = new AtomicStampedReference("tom", 10);
        int oldStamp = atomicStampedReference.getStamp();
        String oldName = atomicStampedReference.getReference();

        System.out.println("初始化之后的版本：" + oldStamp);
        System.out.println("初始化之后的值：" + oldName);

        String newName = "jerry";

        if (!atomicStampedReference.compareAndSet(oldName, newName, 1, oldStamp + 1)) {
            System.out.println("版本不一致，无法修改Reference的值");
        }
        if (atomicStampedReference.compareAndSet(oldName, newName, oldStamp, oldStamp + 1)) {
            System.out.println("版本一致，修改reference的值为jerry");
        }
        System.out.println("修改成功之后的版本：" + atomicStampedReference.getStamp());
        System.out.println("修改成功之后的值：" + atomicStampedReference.getReference());
    }
}
```

输出：

```
初始化之后的版本：10
初始化之后的值：tom
版本不一致，无法修改Reference的值
版本一致，修改reference的值为jerry
修改成功之后的版本：11
修改成功之后的值：jerry
```

### 2、自旋时间过长

如果条件一直不满足，CAS会持续自旋，浪费CPU资源

**解决方案**：限制自旋次数，当失败次数过多时返回失败

补充：自适应自旋

> 互斥同步对性能最大的影响是阻塞的实现，挂起线程和恢复线程的操作都需要转入内核态中完成，这些操作给Java虚拟机的并发性能带来了很大的压力。同时，虚拟机的开发团队也注意到**在许多应用上，共享数据的锁定状态只会持续很短的一段时间，为了这段时间去挂起和恢复线程并不值得**。现在绝大多数的个人电脑和服务器都是多路（核）处理器系统，如果物理机器有一个以上的处理器或者处理器核心，能让两个或以上的线程同时并行执行，我们就**可以让后面请求锁的那个线程“稍等一会”，但不放弃处理器的执行时间，看看持有锁的线程是否很快就会释放锁**。**为了让线程等待，我们只须让线程执行一个忙循环（自旋），这项技术就是所谓的自旋锁**
>
> 自旋锁在JDK 1.4.2中就已经引入，只不过默认是关闭的，可以使用-XX：+UseSpinning参数来开启，在JDK 6中就已经改为默认开启了。**自旋等待不能代替阻塞，且先不说对处理器数量的要求，自旋等待本身虽然避免了线程切换的开销，但它是要占用处理器时间的，所以如果锁被占用的时间很短，自旋等待的效果就会非常好，反之如果锁被占用的时间很长，那么自旋的线程只会白白消耗处理器资源，而不会做任何有价值的工作，这就会带来性能的浪费。因此自旋等待的时间必须有一定的限度**，如果自旋超过了限定的次数仍然没有成功获得锁，就应当使用传统的方式去挂起线程。自旋次数的默认值是十次，用户也可以使用参数-XX：PreBlockSpin来自行更改
>
> 不过无论是默认值还是用户指定的自旋次数，对整个Java虚拟机中所有的锁来说都是相同的。在JDK 6中对自旋锁的优化，引入了自适应的自旋。**自适应意味着自旋的时间不再是固定的了，而是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定的**。如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行中，那么虚拟机就会认为这次自旋也很有可能再次成功，进而允许自旋等待持续相对更长的时间，比如持续100次忙循环。另一方面，如果对于某个锁，自旋很少成功获得过锁，那在以后要获取这个锁时将有可能直接省略掉自旋过程，以避免浪费处理器资源。有了自适应自旋，随着程序运行时间的增长及性能监控信息的不断完善，虚拟机对程序锁的状况预测就会越来越精准，虚拟机就会变得越来越“聪明”了
>

### 3、只能保证能一个变量操作的原子性

普通的CAS只能比较一个值，如果要原子地修改一个对象，而这个对象中有多个属性，就无法保证原子性

**解决方案**：通过AtomicReference实现

```java
public class Main {
    public static void main(String[] args) {
        AtomicReference<User> atomicUserRef = new AtomicReference<>();
        User user = new User("name1", 1);
        atomicUserRef.set(user);
        User updateUser = new User("name2", 2);
        atomicUserRef.compareAndSet(user, updateUser);
        System.out.println(atomicUserRef.get().name);
        System.out.println(atomicUserRef.get().old);
    }

    static class User {
        public String name;
        public int old;

        public User(String name, int old) {
            this.name = name;
            this.old = old;
        }
    }
}
```

输出：

```
name2
2
```