# Redis批处理

## 一、批处理的概念

Redis的批处理指的是将多个命令组合在一起，一次性发送给Redis服务器后，由Redis一次性处理完成，并一次性返回结果给客户端

众所周知，Redis的性能瓶颈不在于命令的执行，而在于网络IO

如果单独发送N条命令，那么总耗时就等于”N次网络IO往返耗时+N条命令执行耗时“

如果将N条命令组合一次性发送，那么总耗时就等于”1次网络IO往返耗时+N条命令执行耗时“

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20231126230510410.png" alt="image-20231126230510410" style="zoom:50%;" />

这样能大大减少耗时

## 二、批处理指令

### 1. `MGET`和`MSET`等指令

Redis提供了一些原生的指令支持批处理，但是可操作的数据类型非常有限

对于String类型，提供了`MGET`和`MSET`指令，用于批量获取或设置key

对于Hash类型，提供了`HMGET`、`HMSET`和`HDEL`指令，用于批量对某个key的多个字段进行操作（注意不是多个key）

还有通用的`DEL`指令，用于批量删除key

除此之外，基本没有其他的命令

### 2. MULTI与EXEC

Redis提供了`MULTI`和`EXEC`指令用于连续执行一串命令

```
# 基本流程
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> SET user:1 "Alice"
QUEUED
127.0.0.1:6379> INCR counter
QUEUED
127.0.0.1:6379> SADD tags "redis" "database"
QUEUED
127.0.0.1:6379> EXEC
1) OK
2) (integer) 1
3) (integer) 2
```

- `MULTI`：开启事务，后续命令不再立即执行，而是进入队列
- 每输入一条命令，返回 `QUEUED`
- `EXEC`：触发执行，批量提交所有队列中的命令，返回结果数组

`MULTI`和`EXEC`不能保证事务，主要是因为其**不能保证原子性**：MySQL事务的原子性指的是所有指令要么全部成功，要么全部回滚。而`MULTI`和`EXEC`只是在执行`EXEC`后将一串命令连续执行，即使中间有命令执行失败也不会回滚，并且会继续执行后续命令（注意`MULTI`与`EXEC`之间并不是原子的，可能被其他请求打断）。举个经典反例：

```
MULTI
SET a 1
INCR b   # b 是字符串，会报错
SET c 3
EXEC
```

执行结果是：

- `SET a 1` ✅ 成功
- `INCR b` ❌ 报错
- `SET c 3` ✅ 仍然会执行

👉 已经成功的命令不会回滚

### 3. Lua

Lua也能够使一串命令连续执行，且是更常用的连续执行策略，但

- Lua中某条命令执行失败后，后续命令不会继续执行

- Lua同样不能实现回滚，但可以**Lua通过临时key的方法模拟回滚**

思路：

- 修改写到 `tmp:*`
- 全部成功后再 `RENAME` 覆盖正式 key

```
redis.call("SET", "tmp:a", 1)
redis.call("SET", "tmp:b", 2)
redis.call("RENAME", "tmp:a", "a")
redis.call("RENAME", "tmp:b", "b")
```

- `RENAME` 是原子的
- 如果中途报错，正式数据完全没动

### 4. 对比

- `MGET`和`MSET`：仅支持部分数据类型，并且操作方式也有限（不能INCR等）
- MULTI与EXEC：能够执行各种操作
- Lua：在MULTI的基础上，还能够添加条件判断、循环等逻辑

三者都能保证指令执行时不被打断，因为Redis的指令执行是单线程的，批处理指令会被作为一个整体去执行

## 三、集群模式的批处理

### 1. 问题

如果批处理指令在集群环境下对多个key进行操作，并且这些key不在同一个slot内，那么Redis就会返回错误。因为一个批处理中的多个slot的key可能会涉及到多个节点，这就无法保证操作的顺序性，并且存在性能问题

### 2. 解决方法

为了实现集群环境下的批处理操作，需要客户端在发送批处理命令前，计算所有key对应的slot，并根据slot将指令分组，生成多个新的批处理指令，再分别发送到Redis服务器执行

