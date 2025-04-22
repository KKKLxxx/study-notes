# MySQL如何解决死锁

## 一、死锁产生场景

### 1、**先 update 再 insert 的并发死锁问题**

表结构如下：

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/640" alt="图片" style="zoom:67%;" />

测试用例如下：

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/641.png" alt="641" style="zoom:67%;" />

死锁分析：可以看到两个事务 update 不存在的记录，先后获得间隙锁，间隙锁之间是兼容的所以在update环节不会阻塞。两者都持有间隙锁，然后去竞争插入意向锁。当存在其他会事务持有间隙锁的时候，当前事务申请不了插入意向锁，导致死锁

## 二、解决方案

1、**合理设计索引并充分使用索引**，避免索引失效升级为表锁

2、**避免大事务**，尽量将大事务拆成多个小事务来处理，小事务发生锁冲突的几率也更小。

3、优化表设计，**减少表连接**

参考链接：https://mp.weixin.qq.com/s?__biz=MzUxODAzNDg4NQ==&mid=2247518397&idx=1&sn=c75e71eca9127474d60d9ea9bdcb546a&chksm=f98dcc17cefa450181debb590e79958d247ef9ffbbb94a36f5726b19829eec598acce24b31dd&mpshare=1&scene=23&srcid=08048R2YHYaiCBsyvYaPzbMs&sharer_sharetime=1659605941721&sharer_shareid=19016e55d7718b51a304b063133a75f7#rd