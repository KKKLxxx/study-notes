# Redis批处理

## 一、批处理的概念

Redis的批处理指的是将多个命令组合在一起，一次性发送给Redis服务器后，由Redis一次性处理完成，并一次性返回结果给客户端

众所周知，Redis的性能瓶颈不在于命令的执行，而在于网络IO

如果单独发送N条命令，那么总耗时就等于”N次网络IO往返耗时+N条命令执行耗时“

如果将N条命令组合一次性发送，那么总耗时就等于”1次网络IO往返耗时+N条命令执行耗时“

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20231126230510410.png" alt="image-20231126230510410" style="zoom:50%;" />

这样能大大减少耗时

## 二、Redis提供的批处理命令

Redis提供了一些原生的指令支持批处理，但是可操作的数据类型非常有限

对于String类型，提供了`MGET`和`MSET`指令，用于批量获取或设置key

对于Hash类型，提供了`HMGET`、`HMSET`和`HDEL`指令，用于批量对某个key的多个字段进行操作（注意不是多个key）

还有通用的`DEL`指令，用于批量删除key

除此之外，基本没有其他的命令

## 三、pipeline

为了提供通用的批处理功能，Redis提供了pipeline机制。pipeline中可以存放多条任意的Redis命令，并一次性发送给Redis服务器

pipeline的具体使用方法与Redis客户端有关

## 四、Redis的事务

pipeline不保证事务，但是Redis提供了`MULTI`和`EXEC`指令用于构建并执行事务，确保一系列命令要么全部执行，要么全部回滚

所以如果要使pipeline也具有事务的特性，可以在pipeline的首尾两处分别添加这两条指令

## 五、集群模式的批处理

### 1、问题

如果批处理指令在集群环境下对多个key进行操作，并且这些key不在同一个slot内，那么Redis就会返回错误。因为一个批处理中的多个slot的key可能会涉及到多个节点，这会引入性能问题和复杂性，所以Redis会报错，以提醒开发者避免跨节点的批处理操作。这样的设计是为了确保集群能够高效地处理请求，同时保持简单和可靠

### 2、解决方法

为了实现集群环境下的批处理操作，需要客户端在发送批处理命令前，计算所有key对应的slot，并根据slot将指令分组，生成多个新的批处理指令，再分别发送到Redis服务器执行

因为指令数量随slot数量变多了，所以可以通过异步执行以降低网络IO带来的时间消耗