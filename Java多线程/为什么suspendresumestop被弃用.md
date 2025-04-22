# 为什么suspend/resume/stop被弃用

**一、原因**
--------

以suspend()方法为例，在调用后，**线程不会释放已经占有的资源（比如锁）**，而是占有着资源进入睡眠状态，这样**容易引发死锁问题**。同样，stop()方法在终结一个线程时不会保证线程的资源正常释放，通常是没有给予线程完成资源释放工作的机会，因此会导致程序可能工作在不确定的状态下

**二、替代方法**
----------

对于suspend与resume，可以用等待、通知机制

[等待、通知机制（wait, notify）](https://gitee.com/KKKLxxx/study-notes/blob/master/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B/%E7%AD%89%E5%BE%85%E3%80%81%E9%80%9A%E7%9F%A5%E6%9C%BA%E5%88%B6%EF%BC%88wait,%20notify%EF%BC%89.md "等待、通知机制（wait, notify）")

对于stop，可以用interrupt方法，还可以通过自定义一个volatile类型的标志位来检验是否停止某线程

[interrupt()与interrupted()](https://gitee.com/KKKLxxx/study-notes/blob/master/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B/interrupt()%E4%B8%8Einterrupted().md "interrupt()与interrupted()")

