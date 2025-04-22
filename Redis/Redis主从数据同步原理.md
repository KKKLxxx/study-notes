# Redis主从数据同步原理

## 一、全量同步

当两台Redis实例刚建立主从关系时，需要首先进行一次全量同步。对于是否是第一次请求同步的判断，是通过`Replication ID`实现的

`Replication ID`是数据集的ID，简称`replid`，ID一致表明是统一数据集。所以在主从关系中，所有Redis实例应有相同的`replid`

因此全量同步过程可以分为三个阶段：

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20231125222152134.png" alt="image-20231125222152134" style="zoom: 50%;" />

- **第一阶段**：slave请求同步数据，这时slave会发送自己的replid和offset，其中offset用于记录repl_baklog中已同步数据的偏移量，而repl_baklog用于记录在RDB过程中执行的命令。master收到请求后，判断replid是否一致，如果不一致，说明是首次同步，需要进行全量同步，会向slave发送master的replid和offset
- **第二阶段**：master执行bgsave生成快照并发送给slave，slave清空数据并加载快照。同时master会将这期间所有的写命令记录到repl_baklog中
- **第三阶段**：master将repl_baklog发送给slave，slave接收到后继续同步

## 二、增量同步

当slave重启后，可以只进行增量同步

增量同步过程可以分为两个阶段：

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20231125223717655.png" alt="image-20231125223717655" style="zoom:50%;" />

- **第一阶段**：与全量同步相同，但是返回的结果不同
- **第二阶段**：master通过offset从repl_baklog中获取slave需要的数据，并发送给slave

增量同步存在的问题在于，repl_baklog有大小限制，写满后会覆盖最早的数据。如果slave断开时间过久，可能导致数据丢失，此时需要重新执行全量同步

## 三、总结

想要提高主从同步的性能，就是要尽量避免全量同步，或者提高同步的性能

为了避免全量同步，可以提高repl_baklog的大小限制，并在slave宕机后及时重启

为了提高同步的性能，有以下两种方法：

- 开启master中的`repl-diskless-sync yes`配置（默认开启），这样RDB文件不会写入磁盘，而是直接写入与slave的socket连接中
- 建立“主-从-从”结构。当slave过多时，如果所有slave都需要从master同步数据，那么会造成master的压力过大。可以让一部分从节点以另一部分从节点为master，这样master用于同步数据的压力就会小很多