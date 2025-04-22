# Redis内存满了会怎么样

## 一、Redis占用内存大小

1、通过配置文件配置

通过在Redis安装目录下面的redis.conf配置文件中添加以下配置设置内存大小

```cpp
// 设置Redis最大占用内存大小为100M
maxmemory 100mb
```

redis的配置文件不一定使用的是安装目录下面的redis.conf文件，启动redis服务时可以传一个参数指定redis的配置文件的路径

2、通过命令修改

Redis支持运行时通过命令动态修改内存大小

```csharp
// 设置Redis最大占用内存大小为100M
127.0.0.1:6379> config set maxmemory 100mb
// 获取设置的Redis能使用的最大内存大小
127.0.0.1:6379> config get maxmemory
```

## 二、Redis的内存淘汰

既然可以设置Redis最大占用内存大小，那么配置的内存就有用完的时候。那在内存用完的时候，还继续往Redis里面添加数据不就没内存可用了吗？

实际上Redis定义了几种策略用来处理这种情况：

- noeviction(默认策略)：对于写请求不再提供服务，直接返回错误（DEL请求和部分特殊请求除外）
- allkeys-lru：从所有key中使用LRU算法进行淘汰
- allkeys-random：从所有key中随机淘汰数据
- volatile-lru：从设置了过期时间的key中使用LRU算法进行淘汰
- volatile-random：从设置了过期时间的key中随机淘汰
- volatile-ttl：在设置了过期时间的key中，根据key的过期时间进行淘汰，越早过期的越优先被淘汰

当使用volatile-lru、volatile-random、volatile-ttl这三种策略时，如果没有key可以被淘汰，则和noeviction一样返回错误

## 三、如何获取及设置内存淘汰策略

获取当前内存淘汰策略：

> 127.0.0.1:6379> config get maxmemory-policy

通过配置文件设置淘汰策略（修改redis.conf文件）：

> maxmemory-policy allkeys-lru

通过命令修改淘汰策略：

> 127.0.0.1:6379> config set maxmemory-policy allkeys-lru

## 四、LRU在Redis中的实现

Redis 3.0对近似LRU算法进行了一些优化。新算法会维护一个候选池（大小为16），池中的数据根据访问时间进行排序，第一次随机选取的key都会放入池中，随后每次随机选取的key只有在访问时间小于池中最小的时间才会放入池中，直到候选池被放满。当放满后，如果有新的key需要放入，则将池中最近被访问的移除

当需要淘汰的时候，则直接从池中选取最近访问时间最小（最久没被访问）的key淘汰掉就行

Redis中每个key都会以redisObject的形式存储，其中`lru`字段记录了不同淘汰策略需要的属性

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20231124221020561.png" alt="image-20231124221020561" style="zoom:67%;" />

- LRU策略：记录最后一次访问时间
- LFU策略：记录最后一次访问时间和逻辑访问次数（不是真实的访问次数）

