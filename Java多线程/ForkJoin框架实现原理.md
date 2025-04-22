# Fork/Join框架实现原理

一、Fork/Join的作用
--------------

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/55b9dc9adb60051c0d68df4fa1e4008d.png" style="zoom:50%;" />

对于一个较大的任务，可以将其分成多个小任务并由多个线程执行（即Fork），最后将其结果汇总（即Join），就得到了最终结果，类似于分治。显然一般情况下，多线程处理能够**提升处理速度**

二、工作窃取算法
---------

在fork时，虽然会分成很多小任务，但不一定会为每个任务分配一个线程，毕竟这样开销也很大。实际上，**框架会把小任务放在不同的队列里，为每个队列分配一个线程**。但是每个队列里的任务执行速度是不一样的，有的队列可能先于其他队列完成，这时候**为了充分利用资源，可以让空闲线程去其他队列窃取任务**

**为了减少线程竞争，这里的任务队列采用双端队列**。被窃取任务线程永远从双端队列的头部拿任务执行，窃取任务线程永远从双端队列的尾部拿任务执行

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/8ec87b74dd6ad93db93da58276a41acc.png" style="zoom:50%;" />

三、Fork/Join的使用示例
----------------

首先要**创建一个ForkJoin任务**，创建方式是继承一个ForkJoinTask的子类，有2种：

1、**RecursiveAction**：用于没有返回结果的任务

2、**RecursiveTask**：用于有返回结果的任务

然后要**通过一个ForkJoinPool提交并执行ForkJoinTask**

```
public class CountTask extends RecursiveTask<Integer> {
    private static final int THRESHOLD = 2;
    private int start;
    private int end;

    public CountTask(int start, int end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Integer compute() {
        int sum = 0;

        // 如果任务足够小就计算
        boolean canCompute = (end - start) <= THRESHOLD;
        if (canCompute) {
            for (int i = start; i <= end; ++i) {
                sum += i;
            }
        } else {
            // 如果任务大于阈值，则分裂为2个子任务
            int middle = start + ((end - start) >> 1);
            CountTask leftTask = new CountTask(start, middle);
            CountTask rightTask = new CountTask(middle + 1, end);
            // 执行子任务
            leftTask.fork();
            rightTask.fork();
            // 等待子任务执行完，并得到其结果
            int leftRes = leftTask.join();
            int rightRes = rightTask.join();
            // 合并子任务
            sum = leftRes + rightRes;
        }
        return sum;
    }

    public static void main(String[] args) {
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        // 生成一个计算任务，负责计算1+2+...+10000
        CountTask countTask = new CountTask(1, 10000);
        // 执行一个任务
        Future<Integer> res = forkJoinPool.submit(countTask);
        try {
            System.out.println(res.get());
        } catch (InterruptedException e) {

        } catch (ExecutionException e) {

        }
    }
}
```

这里模拟的是一个计算1+2+...+10000的任务，因为是又返回结果的任务，所以继承的是RecursiveTask