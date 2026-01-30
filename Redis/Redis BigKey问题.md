# Redis BigKey问题

## 一、什么是BigKey

BigKey（大key）是由于一个key的value数据过大或者数据量过多导致的，比如：

1、用String存储二进制

2、一个Hash中存储了过多键值对，或者Set中存储了过多元素

## 二、BigKey带来的问题

1、**Redis阻塞**：因为Redis是单线程操作，所以BigKey会阻塞对其他所有key的操作

2、**网络阻塞**：当读取一个BigKey时，少量的QPS就可能导致网络带宽被占满

3、**数据分片不均**：在集群模式下，不同key会被分配到不同插槽上，如果有BigKey存在，就会导致Redis实例内存占用不均匀的问题

## 三、查询BigKey

### 1. 手动查询

手动查询指的是通过Redis命令从Redis中获取key信息：

- redis-cli --bigkeys：能够分别获取每种数据类型最大的key
- scan：scan命令可以一次性获取一些key，然后可以通过编程的方式获取key的内存占用等信息

### 2. 三方工具

如云平台提供的工具，或者一些开源工具，可以通过对RDB文件分析得到BigKey

## 四、处理BigKey

查询到BigKey后，就要对BigKey进行拆分和删除

### 1. 拆分

拆分就是要选择恰当的数据结构，将大hash拆分为多个小hash等，并且从业务逻辑上进行优化

### 2. 删除

删除一个BigKey也是一个比较耗时的工作，如果直接通过`DEL`命令删除也会造成阻塞，所以Redis提供了异步删除指令`UNLINK`

**Q：UNLINK是否会造成并发问题？**

UNLINK的异步操作原理是，先在Redis的字典结构中同步删除对应的key，再异步释放key对应的内存空间

UNLINK 的内存释放不是由主线程完成的，而是由 Redis 的后台线程（BIO 线程）异步完成的。主线程只负责“解绑 key 和 value”，不负责真正 free 内存

对于以下场景：

```
T1: UNLINK order:123
T2: SET order:123 newValue
```

T2时刻新添加的`order:123`并不会被T1执行的UNLINK异步释放，因为新的`order:123`申请的是新的内存，而非旧`order:123`对应的内存
