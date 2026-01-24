# MySQL一行记录的存储结构

InnoDB提供了4种行格式，分别是Redundant、Compact、Dynamic和Compressed

其中Redundant比较古老，Dynamic和Compressed都是基于Compact的改进，从 MySQL5.7 版本之后，默认使用 Dynamic 行格式

这里仅介绍Compact，再顺带说明Dynamic与Compressed的不同

## 一、COMPACT行格式

![img](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202601231727097.png)

### 1. 额外信息

#### 1.1 变长字段长度列表

对于varchar, text, blob等类型的变长字段，会在此处存储变长字段真实数据的长度，从而读取数据

假设有以下结构的表t_user：

```sql
CREATE TABLE `t_user` (
    `id` int(11) NOT NULL,
    `name` VARCHAR(20) DEFAULT NULL,
    `phone` VARCHAR(20) DEFAULT NULL,
    `age` int(11) DEFAULT NULL,
    PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB DEFAULT CHARACTER SET = ascii ROW_FORMAT = COMPACT;
```

表中数据如下：

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202601231737435.png" alt="img" style="zoom: 80%;" />

**第一条记录**：

- name 列的值为 a，真实数据占用的字节数是 1 字节，十六进制 0x01；
- phone 列的值为 123，真实数据占用的字节数是 3 字节，十六进制 0x03；
- age 列和 id 列不是变长字段，所以这里不用管。

这些变长字段的真实数据占用的字节数会按照列的顺序**逆序存放**（等下会说为什么要这么设计），所以「变长字段长度列表」里的内容是「 03 01」，而不是 「01 03」。

![img](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202601231738962.png)

同样的道理，我们也可以得出**第二条记录**的行格式中，「变长字段长度列表」里的内容是「 04 02」，如下图：

![img](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202601231739980.png)

**第三条记录**中 phone 列的值是 NULL，**NULL 是不会存放在行格式中记录的真实数据部分里的**，所以「变长字段长度列表」里不需要保存值为 NULL 的变长字段的长度。

![img](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202601231739313.png)

**为什么「变长字段长度列表」的信息要按照逆序存放？**

主要是因为「记录头信息」中指向下一个记录的指针，指向的是下一条记录的「记录头信息」和「真实数据」之间的位置，这样的好处是向左读就是记录头信息，向右读就是真实数据，比较方便

同理， NULL 值列表的信息也需要逆序存放

**每个数据库表的行格式都有「变长字段字节数列表」吗？**

不是

当数据表没有变长字段的时候，比如全部都是 int 类型的字段，这时候表里的行格式就不会有「变长字段长度列表」了，因为没必要，不如去掉以节省空间

#### 1.2 NULL值列表

为了节省空间，如果存在允许 NULL 值的列，则每个列对应一个二进制位（bit），二进制位按照列的顺序逆序排列

- 二进制位的值为`1`时，代表该列的值为NULL
- 二进制位的值为`0`时，代表该列的值不为NULL

另外，NULL 值列表必须用整数个字节的位表示（1字节8位），如果使用的二进制位个数不足整数个字节，则在字节的高位补 `0`

同样根据以上的三条记录展示NULL值列表的情况

**第一条记录**不存在 NULL 值，补位后NULL 值列表用十六进制表示是 0x00：

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202601231749230.png" alt="img" style="zoom:50%;" />

**第二条记录**，NULL值列表用十六进制表示是 0x04：

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202601231750565.png" alt="img" style="zoom:50%;" />

**第三条记录**，NULL 值列表用十六进制表示是 0x06：

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202601231751028.png" alt="img" style="zoom:50%;" />

三条记录的 NULL 值列表都填充完毕后，它们的行格式是这样的：

![img](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202601231751576.png)

**每个数据库表的行格式都有「NULL 值列表」吗？**

不是

当数据表的字段都定义成 NOT NULL 的时候，这时候表里的行格式就不会有 NULL 值列表了

所以在设计数据库表的时候，通常都是建议将字段设置为 NOT NULL，这样可以至少节省 1 字节的空间（NULL 值列表至少占用 1 字节空间）

#### 1.3 记录头信息

一些比较重要的头信息：

- **delete_mask**：标识此条数据是否被删除。从这里可以知道，我们执行 detele 删除记录的时候，并不会真正的删除记录，只是将这个记录的 delete_mask 标记为 1
- **next_record**：下一条记录的位置。从这里可以知道，记录与记录之间是通过链表组织的。在前面我也提到了，指向的是下一条记录的「记录头信息」和「真实数据」之间的位置，这样的好处是向左读就是记录头信息，向右读就是真实数据，比较方便。

### 2. 真实数据

#### 2.1 三个隐藏字段

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202601231754887.png" alt="img" style="zoom: 50%;" />

- **row_id**：在讨论聚簇索引时，我们知道，聚簇索引默认是主键，如果表中没有定义主键，InnoDB会选择一个唯一的非空索引代替。如果没有这样的索引，InnoDB会隐式定义一个主键来作为聚簇索引。这个隐式定义的主键就是row_id，所以如果存在主键或者唯一的非空索引，就不存在row_id，也就是说row_id不是一定存在的
- **trx_id**：事务id，表示这个数据是由哪个事务生成的。trx_id是必需的
- **roll_pointer**：记录上一个版本的指针，与MVCC相关。roll_pointer是必需的

## 二、常见问题

### 1. varchar(n) 中 n 最大取值为多少？

MySQL 规定除了 TEXT、BLOBs 这种大对象类型之外，其他所有的列（不包括额外信息中的记录头信息和真实数据中的三个隐藏字段）占用的字节长度加起来**不能超过 65535 个字节**

varchar(n)字段类型的 n 代表的是最多存储的**字符数量**，并不是字节大小。要算 varchar(n) 最大能允许存储的字符数，还要看数据库表的字符集

- ASCII：一个字符占用1字节，那么varchar括号内最大为65535(65535/1)
- UTF-8：一个字符占用3字节，所以varchar括号内最大为21845(65535/3)
- utf8mb4：意思是most bytes 4，相比utf-8，兼容了emoji表情等特殊符号。这种情况下一个字符占用4字节，所以最大为16383(65535/4)

**以上讨论的情况并未额外考虑变长字段长度列表和NULL值列表，所以真实情况会略小于上述计算值**

### 2. 行溢出后，MySQL 是怎么处理的？

对于大对象字段，如text和blob，明显会超过一行65535字节的限制

这些字段的存储方式是，在真实数据中留出20字节，指向**溢出页**的地址，并在溢出页中存储溢出部分

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202601231821955.png" alt="img" style="zoom: 50%;" />

Dynamic和Compressed采用完全的行溢出方式，记录的真实数据处不会存储该列的一部分数据，只存储 20 个字节的指针来指向溢出页。而实际的数据都存储在溢出页中，看起来就像下面这样：

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202601231917383.png" alt="img" style="zoom:50%;" />

### 3. NULL值是如何存放的？

如上所述，每行的额外信息中会利用一个NULL值列表逆向存储每个字段是否为NULL，从而快速判断是否为空并节省空间

如果所有字段均为NOT NULL，则不会存在NULL值列表，更加节省了空间

### 4. 执行DELETE的数据实际上删除了吗？

首先说结论：**没有**

MySQL执行DELETE操作后，存储在硬盘上的数据没有被删除，只不过在记录行上做了**逻辑删除**，即通过删除标志位实现（也就是额外信息的记录头信息中的delete_mask标志位）

因为如果真的物理删除掉了数据，往往**会导致索引结构的改变**，如果每次删除数据都要改变索引结构，那么会造成大量的磁盘I/O，降低性能

所有被删除的记录都会组成一个垃圾链表，这个链表中记录占用的空间叫做可重用空间，之后**有新记录插入的时候可能把已删除记录占用的空间覆盖掉**

### 5. 