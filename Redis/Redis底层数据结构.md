# Redis底层数据结构

## 零、RedisObject

Redis中任意数据类型的键和值都会被封装为一个RedisObject（robj），其底层结构如下：

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20231122220203820.png" alt="image-20231122220203820" style="zoom:67%;" />

- **type**：表示对象的类型，分别对应各种数据类型
- **encoding**：表示数据结构的底层编码方式，各种数据类型各自有不同的编码方式，会在下文介绍
- **lru**：记录最后访问的时间
- **refcount**：对象引用计数器
- **ptr**：指向实际存放数据的空间

## 一、简单动态字符串SDS

SDS（Simple Dynamic String，简单动态字符串）是Redis底层所使用的字符串表示，几乎所有的Redis模块中都用了SDS

因为 `char*` 类型的功能单一，不能高效地支持一些 Redis 常用的操作（比如拼接操作和长度计算操作），所以在 Redis 程序内部，绝大部分情况下都会使用SDS而不是 `char*` 来表示字符串

### 1. SDS底层结构

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20231121170450244.png" alt="image-20231121170450244" style="zoom:67%;" />

SDS分为头部和实际存储字符串内容的两部分，除buf字段外，其他字段均属于SDS的头部

- **len**：buf的实际长度，不包括空字符
- **alloc**：buf申请的空间大小，不包括空字符
- **flags**：包括sdshdr5/8/16/32/64几种类型，表示len和alloc的数据类型
- **buf**：实际存储字符串内容，会在结尾添加空字符（出于兼容性考虑）

一个包含字符串"name"的SDS结构如下：

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20231121171251634.png" alt="image-20231121171251634" style="zoom: 50%;" />

### 2. SDS编码方式

出于节省空间的角度考虑，SDS有三种编码方式

| 条件                           | 编码方式 |
| ------------------------------ | -------- |
| value 是整数，且能用 long 表示 | `int`    |
| 短字符串（≤ 44 字节左右）      | `embstr` |
| 长字符串                       | `raw`    |

### 3. SDS扩容

SDS的扩容机制与新字符串的大小有关，SDS定义了宏`#define SDS_MAX_PREALLOC (1024*1024)`，以1MB为分界点，对新buf的大小采取不同的扩容机制：

- 若新字符串小于1MB，则`alloc = 2 * len`
- 若新字符串大于等于1MB，则`alloc = len + 1MB`

可以通过`APPEND`指令，对字符串类型的key进行拼接操作，例如：

```
127.0.0.1:6379> set k 1
OK
127.0.0.1:6379> append k 2
(integer) 2
127.0.0.1:6379> get k
"12"
```

### 4. SDS相比char*的优势

- **长度计算更高效**：可以直接从头部获取字符串长度len，时间复杂度为O(1)
- **可以保存二进制数据**：SDS通过len判断实际长度，字符串中间可以存储空字符'\0'，不会被空字符截断，从而保存二进制数据
- **支持动态扩容、防止缓冲区溢出**：由SDS自行管理内存，并通过预分配减少内存分配次数；执行字符串拼接等操作时，会校验是否会使缓冲区溢出，并自动扩容

## 二、数字集合intset

intset是一个由整数组成的有序集合，从而便于在上面进行二分查找，用于快速地判断一个元素是否属于这个集合

### 1. intset底层结构

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20231121175955872.png" alt="image-20231121175955872" style="zoom: 67%;" />

各字段含义如下：

- **encoding**：表示intset中的每个数据元素用几个字节来存储，取值范围有2/4/8字节三种
- **length**：表示intset中元素的个数
- **contents**：contents只是一个指针，其指向内存的大小等于`encoding * length`

一个encoding为`INTSET_ENC_INT16`（2字节），包含5、10、15三个元素的intset的结构如下：

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20231121180447404.png" alt="image-20231121180447404" style="zoom: 50%;" />

每个元素都占用了2个`int8_t`的大小，所以实际上每个元素的类型都为`int16_t`

### 2. intset升级

如果新插入的元素大于encoding的限制，那么intset就会进行升级操作。升级过程可以总结为以下4步：

1、根据需要的元素大小，申请contents的内存

2、倒序依次将数组中的元素拷贝到正确位置

3、将新元素加入到数组末尾（因为是升级操作，所以新元素一定比已有元素都要大）

4、修改encoding和length

### 3. intset特征总结

- 元素按升序有序排列
- 查找方式为二分查找
- 插入元素时，需要将新元素之后的元素整体后移，所以时间复杂度为O(n)

## 三、哈希表HashTable

### 1. HashTable底层结构

Redis的哈希表也可称为字典dict。字典由两部分组成，分别是dict与dictEntry

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20231121235909436.png" alt="image-20231121235909436" style="zoom:67%;" />

- **type**：每个dictType结构保存了一套用于操作特定类型键值对的函数
- **ht_table**：对应哈希表的数组，第一个`dictEntry**`用于正常的存储数据，另一个用于rehash
- **ht_used**：表示ht_table使用的数组容量
- **rehashidx**：表示rehash的进度
- **pauserehash**：表示rehash是否暂停
- **ht_size_exp**：存储ht_table的大小的指数
- **metadata**：存储一些特殊的数据

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20231122000015044.png" alt="image-20231122000015044" style="zoom:67%;" />

- **key**：存储键信息
- **v**：存储值信息
- **next**：指向下一个dictEntry
- **metadata**：存储一些特殊的数据

### 2. 渐进式rehash

当哈希表中的元素越来越多，超过负载因子（负载因子=已保存节点数量/哈希表大小）时，需要通过rehash进行扩容，将元素重新分配到一个更大的哈希表上，防止哈希冲突导致存取效率降低

Redis的rehash并不是一次性完成的，因为整个Redis数据库的核心就是一个大的dict结构，如果dict中包含非常多的entry，那么一次完整的rehash将会造成主线程的阻塞

Redis的rehash过程与HashMap的rehash过程类似，而区别在于，**Redis的rehash会在每次执行增删改查操作时，将`ht_table[0]`中对应的数据rehash到`ht_table[1]`，而不是一次性rehash所有数据**。随着操作的执行，最终所有数据都会转移到`ht_table[1]`中，完成rehash

在rehash的过程中，新增操作直接对`ht_table[1]`操作，而删改查操作需要分别对`ht_table[0]`和`ht_table[1]`查找并执行

## 四、压缩列表ZipList（已废弃）

压缩列表可以为不同的元素分配尽可能少的内存空间以节省内存，但是也因此无法进行随机读取，只能通过遍历寻找某一元素，因此压缩列表**读取操作的时间复杂度为O(n)，仅适合用于数据量较少的场景**

### 1. ziplist底层结构

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20231122164920463.png" alt="image-20231122164920463" style="zoom: 50%;" />

- **zlbytes**：记录整个ziplist占用的字节数
- **zltail**：记录压缩列表最后一个entry相对压缩列表起始地址的偏移量
- **zllen**：记录压缩列表的节点数量
- **entry**：存储压缩列表的各节点
- **zlend**：固定为0xFF，用于标识压缩列表的结束

`entry`的结构如下：

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20231122170624346.png" alt="image-20231122170624346" style="zoom: 50%;" />

- **previous_entry_length**：记录前一个节点的长度，若前一节点长度小于254字节，则该字段占用1字节，否则占用5字节（实现向前遍历）
- **encoding**：记录content的数据类型（字符串/数字），以及content的长度
- **content**：存储数据

### 2. 连锁更新问题

由于`previous_entry_length`占用的空间不是固定的，所以当进行节点的插入、修改操作时，可能导致`previous_entry_length`占用的空间发生变化，从而引发一片连续节点的`previous_entry_length`的变化。这个过程中需要对多个节点的内存进行重新分配，可能造成性能问题

但是因为这种连锁更新发生的概率较低，所以实际使用时不必过分担心

## 五、紧凑列表listpack

listpack还是用一块连续的内存空间来紧凑地保存数据

listpack 头包含两个属性，分别记录了 listpack 总字节数和元素数量，末尾也有个结尾标识

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202601281632417.png" alt="img" style="zoom: 67%;" />

listpack相比压缩链表的改变主要在于，**listpack不再记录前一个节点的长度，只记录当前节点的长度**。当向 listpack 加入一个新元素时，不会影响其他节点的长度字段的变化，从而避免了压缩列表的连锁更新问题

但是listpack仍然可以通过特殊的算法得到前一项的长度，所以也可以实现向前遍历

## 六、快速列表QuickList

新版Redis中，快速列表是由多个listpack（而非压缩列表）组成的双向链表，这样做的理由是

- 双向链表便于在表的两端进行push和pop操作，但是它的**内存开销比较大**。首先，它在每个节点上除了要保存数据之外，还要额外保存两个指针；其次，双向链表的各个节点是单独的内存块，地址不连续，**节点多了容易产生内存碎片**
- listpack是一整块连续内存，所以存储效率很高。但是它**不利于修改操作**，每次数据变动都会引发一次内存的realloc。特别是当listpack长度很长的时候，一次realloc可能会导致大批量的数据拷贝，进一步降低性能

所以quickList相当于达到了一种折中的效果

### 1. quickList底层结构

**（本图中显示的是旧版Redis的结构，将ziplist替换为listpack即可）**

![img](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111171931580.png)

## 七、跳表SkipList

### 1. skipList底层结构

对于一个单链表来讲，即便链表中存储的数据是有序的，如果想在其中查找某个数据，也只能从头到尾遍历链表。这样查找效率就会很低，时间复杂度会很高，是O(n)

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111171931184.png" alt="单链表" style="zoom:50%;" />

如果想要提高其查找效率，可以考虑在链表上建立索引。每两个结点提取一个结点到上一级，我们把抽出来的那一级叫作索引

![一层跳跃表](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111171931544.png)

这个时候，我们假设要查找节点8，可以先在索引层遍历，当遍历到索引层中值为7的结点时，发现下一个节点是9，那么要查找的节点8肯定就在这两个节点之间。我们下降到链表层继续遍历就找到了8这个节点。原先我们在单链表中找到8这个节点要遍历8个节点，而现在有了一级索引后只需要遍历五个节点

从这个例子里，我们看出，加来一层索引之后，查找一个结点需要遍的结点个数减少了，也就是说查找效率提高了，同理再加一级索引

![二层跳跃表](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111171931858.png)

从图中我们可以看出，查找效率又有提升。在例子中我们的数据很少，当有大量的数据时，我们可以**增加多级索引，其查找效率可以得到明显提升。若相邻两级的节点数量为1:2，则可以达到O(logn)的时间复杂度**

![跳跃表](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111171931637.png)

**像这种链表加多级索引的结构，就是跳表**

### 2. 跳表在Redis中的应用

跳表是**有序集合ZSet**的底层实现之一，如果一个有序集合包含的**元素数量比较多**，又或者有序集合中元素的**成员是比较长的字符串**时，Redis就会使用跳表来作为有序集合的底层实现

这里我们需要思考一个问题——为什么元素数量比较多或者成员是比较长的字符串的时候Redis要使用跳表来实现？

从上面我们可以知道，跳表在链表的基础上增加了多级索引以提升查找的效率，但其实是一个空间换时间的方案，必然会带来一个问题——索引是占内存的。原始链表中存储的有可能是很大的对象，而索引结点只需要存储关键值值和几个指针，并不需要存储对象，因此**当节点本身比较大或者元素数量比较多的时候，其优势必然会被放大，而缺点则可以忽略**

### 3. 为什么不用红黑树

红黑树也可以保证有序，但由于以下几点，跳表更加有优势

1、**有序集合有很多范围操作，而跳表更适合范围操作**（红黑树需要中序遍历）

2、跳表的实现相对简单，避免了红黑树的旋转操作

### 4. 为什么不用B+树

两者都可以进行范围操作，但是B+树更适合磁盘数据库（比如MySQL），因为它为减少磁盘I/O做了一些优化（比如非叶子节点不存储数据）

而Redis是内存数据库，并不需要这些复杂操作，所以可以使用实现更简单的跳表
