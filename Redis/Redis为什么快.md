# Redis为什么快

1、纯**内存**数据库，对于内存的存取操作快于硬盘

2、底层的**数据结构**比较高效，如压缩列表、跳表、快速列表

3、通过**IO多路复用**提升监听读写事件的效率

4、通过**单线程**避免执行指令时的上下文切换

5、通过**多线程**提高网络IO的效率

---

**Q：为什么执行指令时只用单线程？**

A：因为Redis执行指令时的性能瓶颈不在于执行速度，而在于网络IO。多线程会带来上下文切换的性能损耗，引入线程安全问题，因此需要加解锁，会进一步降低性能，提高实现的复杂度

---

**Q：Redis的多线程体现在哪？**

A：因为Redis的性能瓶颈在于网络IO，所以可以通过多线程提高网络IO相关操作的执行效率。Redis将**客户端命令的解析**和**向客户端返回响应结果**两个操作通过多线程执行，而指令执行和IO多路复用仍然在主线程中执行

![image-20231123214606608](https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20231123214606608.png)

