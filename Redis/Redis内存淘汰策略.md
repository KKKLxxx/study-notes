# Redis内存淘汰策略

## 一、Redis最大运行内存

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

Redis并没有根据最后访问时间对所有key进行排序，否则维护成本过高，Redis采用了一种利用候选池的**近似LRU算法**

候选池中的key是随机采样的，池中的数据根据访问时间进行排序，第一次随机选取的key都会放入池中，随后每次随机选取的key只有在访问时间小于池中最小的时间才会放入池中。当放满后，如果有新的key需要放入，则将池中最近被访问的移除

当需要淘汰的时候，则直接从池中选取最久没被访问的key淘汰掉

而LFU算法就是记录池中每个key在过去一段时间内的访问次数
