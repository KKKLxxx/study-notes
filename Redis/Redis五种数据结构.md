# Redis五种数据结构

## 零、RedisObject

Redis中任意数据类型的键和值都会被封装为一个RedisObject（robj），其底层结构如下：

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20231122220203820.png" alt="image-20231122220203820" style="zoom:67%;" />

- **type**：表示对象的类型，分别对应5种数据结构
- **encoding**：表示数据结构的底层编码方式，5种数据结构各自有不同的编码方式，会在下文介绍
- **lru**：记录最后访问的时间
- **refcount**：对象引用计数器
- **ptr**：指向实际存放数据的空间

## 一、字符串String

String的编码方式有三种，分别为RAW、EMBSTR与INT

### 1、RAW

RAW为String最常见的编码方式，其实现基于SDS

在RAW编码下，字符串内容会单独存储在一个SDS空间中，并由robj的ptr指向

![image-20231122221023643](https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20231122221023643.png)

### 2、EMBSTR

当SDS的长度小于等于44字节时，会采用EMBSTR编码方式，这种情况下，SDS的内存空间与robj是连续的，所以只用申请一次内存，并提高读写效率

44字节这个界限的选取，在于robj头部和SDS加起来的总长度能够在64字节以下，而Redis以2的n次方进行内存的分配，64字节恰好是一个内存分片的大小，所以可以减少内存碎片

![image-20231122221050210](https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20231122221050210.png)

### 3、INT

如果存储的字符串是整数，并且范围在LONG_MAX内，则会采用INT编码，将值直接存储在ptr所在的位置（指针大小为8字节，与long相同）

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20231122221137726.png" alt="image-20231122221137726" style="zoom: 50%;" />

## 二、链表List

### 1、quickList

list类似于一个双向链表，在新版Redis中完全采用quickList实现

## 三、集合Set

Set的编码方式有两种，分别为intSet与HT(dict)

### 1、intSet

当元素数量小于一定值，并且所有元素均为整数时，采用intSet编码

### 2、HT

其他情况下，会使用HT编码，也就是dict类型。dictEntry中用key来存储元素，value统一为null

## 四、有序集合Sorted Set/ZSet

ZSet的编码方式有两种，分别为`skipList + HT`与`zipList`

ZSet中的元素有2个关键属性，`score`和`member`。`score`用于排序，`member`需要保证唯一

### 1、skipList + HT

因为ZSet需要满足可排序、保证key唯一、可快速通过`member`查找`score`等特性，而`skipList`可以满足前两个特性，`HT`可以满足后两个特性，所以在一定情况下，ZSet会将元素同时存储于`skipList`和`HT`中，分别对不同的数据结构进行操作，以保证功能和性能的要求

### 2、zipList

因为将元素同时存储在`skipList + HT`的方式在元素数量较少时优势不明显，并且比较耗费内存，所以当元素数量较少并且元素均较小的情况下，可以采用`zipList`的编码方式节省内存

然而`zipList`本身并没有排序功能，也没有键值对的概念，因此需要在业务逻辑上实现排序和键值对功能

排序功能：根据score手动对元素进行排序

键值对功能：将score和member都视为entry，并使其相邻

## 五、哈希表Hash

Hash的编码方式有两种，分别为`zipList`与`HT`

### 1、zipList

类似于ZSet，采用zipList的编码方式仍然是为了节省内存，也需要在业务逻辑上实现键值对功能，同样仅适用于元素数量较少并且元素均较小的情况

### 2、HT

在元素较多或者元素较大的情况下，编码方式会改为HT

## 六、Stream

stream是Redis 5.0版本新增加的数据结构，主要用作消息队列。Redis原本的发布订阅机制虽然有简易的消息队列功能，但是它无法进行消息的持久化，所以无法正确应对宕机等情况

stream不仅提供了消息的持久化功能，并且可以为每个消息自动生成一个递增的ID，记录每个客户端访问的位置，保证消息不丢失

stream还提供了ACK和pending-list功能，ACK用于消费者消费完成后返回确认消息，pengding-list用于存储已接收但未确认的消息

以及提供了消费者组的功能

stream的底层结构是`radix-tree`前缀树以及`listpack`紧凑列表

- **radix-tree**：用于存储消息的key，即ID。因为ID的格式为“时间戳-序号”，有大量重复的前缀，所以适合用radix-tree来存储
- **listpack**：用于存储消息的value，listpack是一种类似于zipList的结构，用于节省内存

`listpack`相比`zipList`的主要区别在于，`listpack`的entry不再记录前一个entry的长度，而是只记录自身entry的长度，所以不会产生连锁更新的问题。而`listpack`更加结构化，导致内存利用率相对较低，但是可以存储更多、类型差异更大的元素