# Redis主从模式、哨兵模式、集群模式

## 一、主从模式

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111171932404.jpeg" alt="img" style="zoom:50%;" />

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

主从模式即一主(master)一从(slave)或一主多从，从节点定期备份主节点的数据。**主节点可提供读写操作，从节点只提供读操作**

**优点**：

1、分担了主节点的读压力

2、实现了数据备份

**缺点**：

1、如果主节点失效，需要人工干预才可恢复

2、写能力和存储能力都受到单机限制

## 二、哨兵模式

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111212342027.jpeg" alt="img" style="zoom:50%;" />

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

哨兵模式即在主从模式的基础上，添加一个或多个哨兵，实现主节点宕机后**自动化恢复**

哨兵：特殊的redis节点，不存储数据，用于监控各个主从节点的存活状态，当主节点宕机时负责选举新的主节点

哨兵监控原理：心跳包。当有足够多的哨兵发送心跳包却没有收到响应时，该节点会被标记为下线状态。如果是主节点下线，之后会通过选举机制在从节点中选取一个自动升级为主节点，一般会根据预设的优先级和数据同步的offset选取

**优点**：

1、包含主从模式的所有优点

2、实现了自动化恢复

**缺点**：

写能力和存储能力仍受到单机限制

## 三、集群模式

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/44ec8f2e53bc65ddc1f50f4560360467.png" alt="img" style="zoom: 50%;" />

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

上面两种模式都没有解决单机限制，所以集群模式要解决的就是这个问题

### 1、数据分片

Redis会把每一个master节点映射到0-16383一共16384个插槽（slot）上，数据会与插槽绑定而不是与节点绑定。进行数据操作时，Redis会将数据key的有效部分通过CRC16算法得到一个hash值，并将hash值对16384取余，得到key对应的插槽

如果要将某一类型的数据固定在某个实例上，可以通过`{}`指定key的有效部分，例如`{phone}:123`，则算法只会对固定的phone前缀计算hash值，保证所有以phone为前缀的key固定在同一实例上

### 2、故障转移

在Redis集群中，节点之间通过Gossip协议进行通信，通过PING、PONG消息来维持节点之间的心跳连接。当主节点不可用时，集群中的其他节点会通过Gossip协议感知到主节点的失效，并进行一系列的选举过程，选择新的主节点

Gossip过程是由种子节点发起，当一个种子节点有状态需要更新到网络中的其他节点时，它会随机的选择周围几个节点散播消息，收到消息的节点也会重复该过程，直至最终网络中所有的节点都收到了消息。所以在集群模式中，无需哨兵就可以完成故障转移

**优点**：

1、包含哨兵模式的所有优点，并且无需单独的哨兵进程

2、突破单机限制

**缺点**：

1、数据倾斜问题：由于数据的分片可能不均匀，导致某些Redis实例内存占用高，而另一些实例内存占用少

2、性能问题：由于需要对slot和节点进行选择，所以会导致一定的性能下降

3、命令兼容性问题：比如pipeline指令会因为所操作的key不在同一slot上而导致错误
