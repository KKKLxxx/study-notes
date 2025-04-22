# ES脑裂问题与选举流程

## 一、概述

- 在Elasticsearch当中，ES分为四种角色：master、data（数据节点）、Coordinating（协调节点）、Ingest（预处理节点）

- master、data、Coordinating三种角色由elasticsearch.yml配置文件中的node.master、node.data来控制；Ingest角色由node.ingest来控制

- 如果不修改elasticsearch的节点角色信息，那么默认就是node.master: true、node.data: true、node.ingest: true

- 默认情况下，一个节点可以充当一个或多个角色，默认四个角色都有。都有成为主节点的资格，也都存储数据，还可以提供查询服务，负载均衡以及数据合并等服务。在高并发的场景下集群容易出现负载过高问题

## 二、角色划分

### 1、master

- master节点具备主节点的选举权，有资格成为主节点，**主节点控制整个集群的元数据(metadata)，比如索引的新增、删除、分片分配等**
- 该节点不和应用创建连接，master节点不占用IO和CPU，内存使用量一般
- 角色配置方式：node.master: true

### 2、data

- 该节点和应用创建连接、接收索引请求，会存储分配在该node上的shard的数据并负责这些shard的写入、查询等，ES集群的性能取决于该节点的个数（每个节点最优配置的情况下），data节点会占用大量的CPU、io和内存
- data节点的分片执行查询语句、获得查询结果后将结果反馈给Coordinating，此过程较消耗硬件资源
- 角色配置方式：node.data: true

### 3、Coordinating

- 该节点和检索应用创建连接、接受检索请求，但其本身不负责存储数据，可当负责均衡节点，Coordinating节点不占用io、cpu和内存
- Coordinating节点接受搜索请求后将请求转发到与查询条件相关的多个data节点的分片上，然后多个data节点的分片执行查询语句或者查询结果再返回给Coordinating节点，Coordinating来把各个data节点的返回结果进行整合、排序等一系列操作后再将最终结果返回给用户请求
- 角色配置方式：node.master: false，node.data: false

### 4、Ingest Node节点

- 可以在任何节点上启用ingest，甚至使用专门的ingest nodes
- Ingest node 专门对索引的文档做预处理，发生在对真实文档建立索引之前。在建立索引对文档预处理之前，先定义一个管道（pipeline），管道里指定了一系列的处理器。每个处理器能够把文档按照某种特定的方式转换。比如在管道里定义一个从某个文档中移除字段的处理器，紧接着一个重命名字段的处理器。集群的状态也会被存储到配置的管道内。定义一个管道，简单的在索引或者bulk request（一种批量请求方法）操作上定义 pipeline 参数,这样 ingest node 就会知道哪个管道在使用。这个节点在使用过程中用的也不多，所以大概了解一下就行
- 角色配置方式：node.ingest: true

## 三、脑裂问题

### 1、问题描述

正常情况下，一个ES集群中只存在一个master节点。如果有多个node当选为master，则集群会出现脑裂，脑裂会**破坏数据的一致性**，导致集群行为不可控，产生各种非预期的影响

### 2、产生原因

- 网络问题：集群间的网络延迟导致一些节点访问不到master，认为master挂掉了从而选举出新的master，并对master上的分片和副本标红，分配新的分片

- 节点负载过大：主节点的角色既为master又为data，访问量较大时可能会导致ES停止响应造成大面积延迟，此时其他节点得不到主节点的响应认为主节点挂掉了，会重新选取主节点

- 内存回收：data节点上的ES进程占用的内存较大，引发JVM的大规模内存回收，造成ES进程失去响应

### 3、解决方法

- 减少误判：discovery.zen.ping_timeout

  节点状态的响应时间，默认为3s，可以适当调大，如果master在该响应时间的范围内没有做出响应应答，判断该节点已经挂掉了。调大该参数可适当减少误判

- 选举触发：discovery.zen.minimum_master_nodes

  该参数是用于控制选举行为发生的最小集群主节点数量

  当备选主节点的个数大于等于该参数的值，且备选主节点中有节点认为主节点挂了，进行选举。官方建议为(n/2)+1，n为有资格成为主节点的节点个数

  增大该参数，当该值为2时，我们可以设置有资格成为master节点的数量为3。这样，挂掉一台，其他两台都认为主节点挂掉了，才进行主节点选举

- 角色分离：即master节点与data节点分离，限制角色

  主节点配置为：

  node.master: true	node.data: false

  从节点配置为：

  node.master: false	node.data: true

## 四、选举机制

### 1、选举触发时机

当一个节点发现包括自己在内的多数候选master节点认为集群没有master时，就可以发起master选举

### 2、确定选举对象

首先是选举谁的问题，如下面源码所示，选举的是排序后的第一个MasterCandidate(即master-eligible node)

```
public MasterCandidate electMaster(Collection<MasterCandidate> candidates) {
        assert hasEnoughCandidates(candidates);
        List<MasterCandidate> sortedCandidates = new ArrayList<>(candidates);
        sortedCandidates.sort(MasterCandidate::compare);
        return sortedCandidates.get(0);
}
```

那么是按照什么排序的？

```java
public static int compare(MasterCandidate c1, MasterCandidate c2) {
    // we explicitly swap c1 and c2 here. the code expects "better" is lower in a sorted
    // list, so if c2 has a higher cluster state version, it needs to come first.
    int ret = Long.compare(c2.clusterStateVersion, c1.clusterStateVersion);
    if (ret == 0) {
        ret = compareNodes(c1.getNode(), c2.getNode());
    }
    return ret;
}
```

 如上面源码所示，先根据节点的clusterStateVersion比较，clusterStateVersion越大，优先级越高；clusterStateVersion相同时，进入compareNodes，其内部按照节点的Id比较(Id为节点第一次启动时随机生成)

总结一下：

1. 当clusterStateVersion越大，优先级越高。这是为了保证新Master拥有最新的clusterState(即集群的meta)，避免已经commit的meta变更丢失。因为Master当选后，就会以这个版本的clusterState为基础进行更新。(一个例外是集群全部重启，所有节点都没有meta，需要先选出一个master，然后master再通过持久化的数据进行meta恢复，再进行meta同步)
2. 当clusterStateVersion相同时，节点的Id越小，优先级越高。即总是倾向于选择Id小的Node，这个Id是节点第一次启动时生成的一个随机字符串。之所以这么设计，应该是为了让选举结果尽可能稳定，不要出现都想当master而选不出来的情况

### 3、什么时候选举成功？

当一个master-eligible node(我们假设为Node_A)发起一次选举时，它会按照上述排序策略选出一个它认为的master

- 假设Node_A选Node_B当Master，Node_A会向Node_B发送join请求，那么此时：

  (1) 如果Node_B已经成为Master，Node_B就会把Node_A加入到集群中，然后发布最新的cluster_state, 最新的cluster_state就会包含Node_A的信息。相当于一次正常情况的新节点加入。对于Node_A，等新的cluster_state发布到Node_A的时候，Node_A也就完成join了

  (2) 如果Node_B在竞选Master，那么Node_B会把这次join当作一张选票。对于这种情况，Node_A会等待一段时间，看Node_B是否能成为真正的Master，直到超时或者有别的Master选成功

  (3) 如果Node_B认为自己不是Master(现在不是，将来也选不上)，那么Node_B会拒绝这次join。对于这种情况，Node_A会开启下一轮选举

- 假设Node_A选自己当Master：

  此时NodeA会等别的node来join，即等待别的node的选票，当收集到超过半数的选票时，认为自己成为master，然后变更cluster_state中的master node为自己，并向集群发布这一消息

### 4、节点失效监测

这里的错误检测可以理解为类似心跳的机制，有两类错误检测：

- Master节点启动NodesFaultDetection定期监测加入集群的节点是否活跃；
- 非Master节点启动MasterFaultDetection定期监测Master节点是否活跃；

节点监测都是通过定期（1s）发送ping探测节点是否正常，如果超过3次，则开始处理节点离开事件；

**NodesFaultDetection事件处理：**

　　如果Master检测到某个Node连不上了，会执行removeNode的操作，将节点从cluste_state中移除，并发布新的cluster_state。当各个模块apply新的cluster_state时，就会执行一些恢复操作，比如选择新的primaryShard或者replica，执行数据复制等

**MasterFaultDetection事件处理：**

　　如果某个Node发现Master连不上了，会清空pending在内存中还未commit的new cluster_state，然后发起rejoin，重新加入集群。(如果达到选举条件则触发新master选举)

### 5、分片与副本的角色切换

当分片不可用时，es就会重新进行选举，把最新的副本提高到分片的地位，这里es的master节点实现了主副本选举的逻辑

