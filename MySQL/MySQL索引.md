# MySQL索引

## 一、索引的作用和缺点

**作用**：**提高查询效率**，可以将随机IO变成顺序IO。与查阅图书所用的目录是一个道理：先定位到章，然后定位到该章下的一个小节，然后找到页数

**缺点**：创建和维护索引需要耗费额外的时间和空间，且在表内容较少时直接扫描效率更高

## 二、索引的结构

可以先熟悉一下相关数据结构

[B树（B-树）、B+树、红黑树](https://gitee.com/KKKLxxx/study-notes/blob/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/B%E6%A0%91%EF%BC%88B-%E6%A0%91%EF%BC%89%E3%80%81B+%E6%A0%91%E3%80%81%E7%BA%A2%E9%BB%91%E6%A0%91.md "B树（B-树）、B+树、红黑树")

**索引的数据结构**：主要为B+树（不可能在非叶子结点命中）

**不用哈希表的原因**：

（哈希索引通过对索引列计算哈希值实现快速查找，索引中包含哈希值和行指针，行指针指向数据所在行。查询快，效率为O(1)）

1、仅支持等值查询，不支持范围查询

2、可能有哈希冲突，降低查询效率

3、不支持索引列的部分查找，因为哈希值是对所有索引列的计算

4、无法避免回表，因为索引中只有哈希值没有数据

5、数据量大时，可能无法一次性加载到内存中

**不用B树的原因**：

1、如果进行范围查询的话，B树每次都要从根节点开始查询，效率会有所降低（中序遍历？查询时间不稳定？）

2、B+树的非叶子节点只存储索引，没有数据，所以可以一次加载更多节点到内存中，减少IO次数

**不用红黑树的原因**：红黑树（二叉）的深度比B+树高，会增加磁盘IO次数

**使用B+树的优势**：

1、B+树叶子结点之间有互相连接的指针，所以更适合范围操作

2、B+树的非叶子节点不存储数据，所以在数据量很多时，树的高度也不会增长很快，查询速度比较稳定（高度一般在3-4层）

## 三、索引的使用场景

由于索引创建和维护都需要额外开销，所以以下几种情况都是**不推荐**使用索引的

1、**数据量小**：这时候全表扫描效率更高

2、**频繁更新**：数据的频繁更新会导致索引也频繁更新，有些索引，如聚簇索引，更新的代价可能极大

3、**唯一性差**：唯一性差的字段，如性别，只有两种，这时候索引的作用几乎无法体现

4、**索引失效**：如果索引失效，则应该删除相关索引，以避免索引的开销

所以索引的使用场景应该避免上面几种情况

## 四、索引分类

先按照聚簇/非聚簇来分类，再列举这两大类里的每种索引

### 0. 聚簇索引与非聚簇索引

聚簇索引和非聚簇索引并不是某种单独类型的索引，只是索引在数据存储方式上的一种分类。比如主键索引是聚簇索引，其他自行创建的索引都属于非聚簇索引

#### 0.1 聚簇索引

聚簇索引默认是主键，如果表中没有定义主键，InnoDB会选择一个唯一的非空索引代替。如果没有这样的索引，InnoDB会隐式定义一个主键来作为聚簇索引

- **优点**：
  - 查找速度快，聚簇索引查找数据只用查找一次，即通过主键找到对应的数据；非聚簇索引需要先通过索引找到主键，再根据主键找到数据

- **缺点**：
  - 插入速度严重依赖于插入顺序，按照主键的顺序插入是最快的方式，否则将会出现页分裂，严重影响性能。因此对于InnoDB表，一般都会定义一个自增的ID列为主键
  - 更新主键的代价很高，因为将会导致被更新的行移动。因此对于InnoDB表，一般定义主键为不可更新

#### 0.2 非聚簇索引

也称**辅助索引**、**二级索引**，在聚簇索引之上创建的索引称为辅助索引，辅助索引的存在不影响数据在聚簇索引中的组织

**存在的原因**：当数据库一条记录里包含多个字段时，如果检索的是非主键字段，则主键索引失去作用，又变成顺序查找了，这时应该建立辅助索引

**索引覆盖**：如果查询的数据在辅助索引中存在（比如主键，或复合索引中的其他字段），则无需再根据主键进行第二次查询（即无需**回表**）

#### 0.3 区别

- 聚簇索引只有一个；非聚簇索引可以有多个

- 聚簇索引查询速度快，只需要查询一次即可找到数据；非聚簇索引查询速度慢，需要先查询一次找到主键，再由聚簇索引查询一次找到数据

- 聚簇索引创建和更新代价大；非聚簇索引代价小

- 聚簇索引叶子节点存储的是数据；非聚簇索引叶子节点存储的是主键+索引

### 1. 聚簇索引

#### 1.1 主键索引

顾名思义，是以表中的主键为索引列创建的索引，会在创建表时**自动创建**，且**不能删除**。如果表中没有主键也就没有主键索引

### 2. 非聚簇索引

#### 2.1 普通索引

最基本的索引，没有任何限制

#### 2.2 唯一索引

索引列的值必须唯一，允许有空值

#### 2.3 复合索引

组合索引指在多个字段上创建的索引，只有在查询条件中使用了创建索引时的第一个字段，索引才会被使用。使用组合索引时遵循**最左匹配原则**

#### 2.4 全文索引

全文索引主要用来查找文本中的关键字，而不是直接与索引中的值相比较。fulltext索引跟其它索引大不相同，它更像是一个搜索引擎，而不是简单的where语句的参数匹配

比如要搜索所有标题以“MySQL”开头的文章，也许可以选择在标题列创建一个索引，并使用“LIKE MySQL%”查询，但是LIKE相比全文索引效率低很多

然而全文索引的效率相比ElasticSearch还是低很多的，一般需要全文检索时都会选择ES，所以就不详细介绍全文索引了

## 五、索引相关SQL语句

### 1. 查询

```sql
SHOW INDEX FROM table_name;
```

### 2. 创建

```sql
-- 普通索引
ALTER TABLE table_name ADD INDEX index_name(column_name);

-- 唯一索引
ALTER TABLE table_name ADD UNIQUE INDEX index_name(column_name);

-- 复合索引
ALTER TABLE table_name ADD INDEX index_name(column_name1, column_name2, column_name3);

-- 全文索引
ALTER TABLE table_name ADD FULLTEXT INDEX index_name(column_name);
```

### 3. 删除

```sql
DROP INDEX index_name ON table_name;
```

### 4. 检验是否使用索引 

在语句前加"explain"，如

```sql
EXPLAIN SELECT * FROM table_name WHERE id = 1;
```

其中id为主键，则查询结果中，possible_keys会有一个结果为PRIMARY 

## 六、索引失效场景

通过一张表和不同的查询语句，来实际验证一下索引失效（环境：Windows，MySQL 8.0.23）

建表SQL

```sql
CREATE TABLE staffs (
    id INT PRIMARY KEY AUTO_INCREMENT,
    `name` VARCHAR(24)NOT NULL DEFAULT'' COMMENT'姓名',
    `age` INT NOT NULL DEFAULT 0 COMMENT'年龄',
    `pos` VARCHAR(20) NOT NULL DEFAULT'' COMMENT'职位',
    `add_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT'入职时间'
)CHARSET utf8 COMMENT'员工记录表';

INSERT INTO staffs(`name`,`age`,`pos`,`add_time`) VALUES('z3',22,'manager',NOW());
INSERT INTO staffs(`name`,`age`,`pos`,`add_time`) VALUES('July',23,'dev',NOW());
INSERT INTO staffs(`name`,`age`,`pos`,`add_time`) VALUES('2000',23,'dev',NOW());

ALTER TABLE staffs ADD INDEX index_staffs_nameAgePos(`name`,`age`,`pos`);
```

**注意到这里将所有字段都设置为NOT NULL，如果不要求NOT NULL，结果会稍有不同**

```sql
EXPLAIN SELECT * FROM staffs WHERE name = "1"; -- 1

EXPLAIN SELECT * FROM staffs WHERE name = "1" AND age = 1; -- 2

EXPLAIN SELECT * FROM staffs WHERE name = "1" AND age = 1 AND pos = "1"; -- 3
```

解释一下，**每行语句后面的注释代表实际上用了多少个索引，索引使用数量可以通过key_len字段得知**

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/20211009232508798.png)![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/20211009232521896.png) ![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/20211009232530149.png) 

本例中使用1、2、3个索引的情况下的key_len分别74、78、140

* * *

### 1. 不符合最左匹配原则

这个其实很经典了，就是如果不使用name字段，就会导致失效，类似下面两条

```sql
EXPLAIN SELECT * FROM staffs WHERE age = 1 AND pos = "1"; -- 0

EXPLAIN SELECT * FROM staffs WHERE pos = "1"; -- 0
```

**但是需要注意，如果是下面这种情况，索引是可以完全生效的**

```sql
EXPLAIN SELECT * FROM staffs WHERE pos = "1" AND age = 1 AND name = "1"; -- 3
```

因为MySQL优化器可以自动将AND左右的条件调换顺序，使之符合最左匹配原则

### 2. 复合索引中间字段未使用

如果不使用中间的age字段，会导致age和pos都失效，即**本身和后面的索引都失效**

```sql
EXPLAIN SELECT * FROM staffs WHERE name = "1" AND pos = "1"; -- 1
```

### 3. 复合索引前面的字段使用了（范围查询/不等于查询）

复合索引前面的字段使用了（范围查询/不等于查询）**导致后面的索引失效，但它本身会有效**

```sql
EXPLAIN SELECT * FROM staffs WHERE name > "1" AND age = 1 AND pos = "1"; -- 1

EXPLAIN SELECT * FROM staffs WHERE name = "1" AND age > 1 AND pos = "1"; -- 2

EXPLAIN SELECT * FROM staffs WHERE name != "1" AND age = 1 AND pos = "1"; -- 1
```

可理解为：获取`name > "1"`的部分后，`age=1`可能出现在`name=2/3/4...`中，所以都需要分别查询，也就是全表扫描；如果理解为在`name=2/3/4...`中分别查找`age =1`，这样不符合最左匹配，所以也是没法给`age`利用到索引的

但当查询为`name >= "1" AND age = 1`时，在`name="1"`的范围内查找，可以利用到`age`的索引

### 4. 使用了is null / is not null

这会导致**本身和后面的索引都失效**，比如

```sql
EXPLAIN SELECT * FROM staffs WHERE name is not null; -- 0

EXPLAIN SELECT * FROM staffs WHERE name is not null AND age = 1 AND pos = "1"; -- 0

EXPLAIN SELECT * FROM staffs WHERE name = "1" AND age is not null AND pos = "1"; -- 1
```

**这里注意到我上面说建表时有没有规定NOT NULL会有区别，如果没有规定NOT NULL，该字段也可以走索引，并且不影响后面的字段走索引**

### 5. LIKE前置%

也很经典了

```sql
EXPLAIN SELECT * FROM staffs WHERE name LIKE "%1"; -- 0
```

**但是需要注意也可以通过覆盖索引使得前置%也走索引**

[如何让LIKE前置%也走索引_KKKL的博客-CSDN博客](https://blog.csdn.net/qq_45404693/article/details/120694101 "如何让LIKE前置%也走索引_KKKL的博客-CSDN博客")

### 6. varchar使用数字查询

因为name是varchar类型的，查询的时候需要将条件用引号括起来，如果像下面一样，就会索引失效，而且查询结果也会很特别

```sql
EXPLAIN SELECT * FROM staffs WHERE name = 1; -- 0
```

可以参考这篇文章

[MySQL中varchar字段使用数字查询的特殊现象_KKKL的博客-CSDN博客](https://blog.csdn.net/qq_45404693/article/details/120630862 "MySQL中varchar字段使用数字查询的特殊现象_KKKL的博客-CSDN博客")

### 7. OR前后字段未均有索引

虽然name有索引，但是add_time没有索引，如果将这两个字段OR在一起，会导致**完全不走索引**

```sql
EXPLAIN SELECT * FROM staffs WHERE name = "1" OR name = "2"; -- 1

EXPLAIN SELECT * FROM staffs WHERE name = "1" OR add_time = "2021-10-09 23:12:56"; -- 0
```

### 8. 使用了函数

```sql
EXPLAIN SELECT * FROM staffs WHERE name = "1"; -- 1

EXPLAIN SELECT * FROM staffs WHERE left(name, 4) = "1"; -- 0
```



## 七、索引优化方法

1、**避免索引失效**

2、**避免回表**

3、**单机环境尽量使用自增主键**：防止页分裂等，详细见另一篇文章

4、**索引列设置为非空NOT NULL**：否则会占用1字节空间存储 NULL 值列表，并且查询时也会对空值做额外处理（如count对NULL值忽略），降低效率

## 八、索引优化案例

### 1. 单表优化案例

首先建一张表

```sql
CREATE TABLE IF NOT EXISTS `article`(
    `id` INT(10) UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
    `author_id` INT (10) UNSIGNED NOT NULL,
    `category_id` INT(10) UNSIGNED NOT NULL , 
    `views` INT(10) UNSIGNED NOT NULL , 
    `comments` INT(10) UNSIGNED NOT NULL,
    `title` VARBINARY(255) NOT NULL,
    `content` TEXT NOT NULL
);

INSERT INTO `article`(`author_id`,`category_id` ,`views` ,`comments` ,`title` ,`content` )VALUES
(1,1,1,1,'1','1'),
(2,2,2,2,'2','2'),
(3,3,3,3,'3','3');
```

然后进行如下查询

```sql
EXPLAIN SELECT id, author_id FROM article WHERE category_id = 1 AND comments > 0 ORDER BY views DESC LIMIT 1;
```

结果如下

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111301312766.png)

发现是一个全表查询，所以需要添加索引。直观上是需要建立一个(category_id, comments, views)的复合索引，所以尝试建立后再查询

```sql
CREATE INDEX idx_ccv ON article(category_id, comments, views);

EXPLAIN SELECT id, author_id FROM article WHERE category_id = 1 AND comments > 0 ORDER BY views DESC LIMIT 1;
```

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111301312475.png)

发现虽然用到了索引，但是在Extra中有Using filesort，说明排序的时候没有按照索引进行排序，这会导致性能有一定下降

**索引失效原因**：虽然复合索引的3个字段按顺序出现在查询语句中了，但是comments字段是一个范围查询，导致后面的索引失效

可以尝试运行以下查询语句，其中将comments的范围查询改为了等值查询

```sql
EXPLAIN SELECT id, author_id FROM article WHERE category_id = 1 AND comments = 0 ORDER BY views DESC LIMIT 1;
```

结果变为了

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111301312838.png)

type已经达到ref，Extra中也没有Using filesort，说明已经优化的不错了。但是这里是改变了查询语句，没有达到原来的目的。我们仍然需要优化索引

**优化思路**：既然comments使得索引断开，那么可以不对comments建立索引，仅对(category_id, views)建立索引。这样查询过程变为，先得到所有`category_id=1`的数据，这段区间内的数据已经按 `views` 排好序，回表取`comments`并判断是否大于0

```sql
DROP INDEX idx_ccv ON article;

CREATE INDEX idx_cv ON article(category_id, views);

EXPLAIN SELECT id, author_id FROM article WHERE category_id = 1 AND comments > 0 ORDER BY views DESC LIMIT 1;
```

对索引做出如上修改，结果如下

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111301313360.png)

通过type和Extra的信息可知，该索引有了一定的优化，但comments仍然没有利用到索引，其实还可以进一步优化

**优化思路**：利用索引下推，减少存储引擎层向服务器层传递的数据量，可以建立以下索引

```sql
CREATE INDEX idx_cvc ON article(category_id, views, comments);
```

**这个案例的启示：在有where和order by的范围查询语句中，并不一定要按照语句中字段出现的顺序建立索引**

### 2. 双表优化案例

首先通过如下SQL语句建两张表

```sql
CREATE TABLE IF NOT EXISTS `class`(
    `id` INT(10) UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
    `card` INT (10) UNSIGNED NOT NULL
);
CREATE TABLE IF NOT EXISTS `book`(
    `bookid` INT(10) UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
    `card` INT (10) UNSIGNED NOT NULL
);
INSERT INTO class(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card)VALUES(FLOOR(1+(RAND()*20)));
 
INSERT INTO book(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card)VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card)VALUES(FLOOR(1+(RAND()*20)));
```

 作以下连表查询

```sql
EXPLAIN SELECT * FROM class LEFT JOIN book ON class.card = book.card;
```

因为没有建立过索引，所以结果都是全表查询

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111301339591.png)

下面要考虑的是：对于左连接，在左表建立索引更高效还是在右表建立索引更高效

可以直接尝试一下，首先尝试在左表class上建立索引

```sql
ALTER TABLE class ADD INDEX X(card);

EXPLAIN SELECT * FROM class LEFT JOIN book ON class.card = book.card;
```

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111301339228.png)

发现type已经转为index，有了一定的优化，然后删除该索引，在右表建立索引再查询

```sql
DROP INDEX X ON class;

ALTER TABLE book ADD INDEX Y(card);

EXPLAIN SELECT * FROM class LEFT JOIN book ON class.card = book.card;
```

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111301339137.png)

发现type变为了ref，比index效果更好

如果两个索引同时存在，会出现以下结果

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111301339804.png)

所以可以得到结论：**左连接时，在右表建立索引效果更好；反之，右连接时，在左表建立索引效果更好。但在内连接时，要看哪张表被优化器选择为了驱动表（类似于左连接时的左表，更小、过滤性更强的表作为驱动表的成本更小），对另外一张表建立索引效果更好**

**原理**：对于左连接，可以看作把左表所有数据显示出来，但右表中的数据如果不满足ON条件，则显示为NULL。所以对于左表是一个全表扫描，而对右表中的数据，会按照`card`字段去查询，所以对右表建立索引的效果更好

## 九、索引覆盖与回表

了解了聚簇索引和非聚簇索引后可以知道，如果使用非聚簇索引，都会先定位到聚簇索引上，如果有需要，再定位到表的数据上，**这个由聚簇索引定位到数据表上的查询过程就叫做回表**，所以非聚簇索引会走2次索引

那么有没有哪种情况可以让非聚簇索引也只走1次索引呢？有，那就是使用覆盖索引

我们知道，**索引上的内容也不过是将表上的某些字段以B+树的结构储存起来，如果我们要查询的字段刚好就是索引包括的字段，那就可以在扫描完索引后直接得到结果，不需要回表，这就是覆盖索引**

举个例子

有这么一张表 ![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/20211010194343905.png)

* * *

**注意字段是varchar类型！！如果是char会有不同的结果，在此只讨论varchar**

* * *

有一个复合索引(col1, col2)

如果我们进行SELECT *查询

```sql
EXPLAIN SELECT * FROM tab1 WHERE col1 = "1";
```

结果很正常也很普通

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/20211010194749929.png)

但是假设我们只需要col1和col2两个字段上的数据，就可以查询语句改为

```sql
EXPLAIN SELECT col1, col2 FROM tab1 WHERE col1 = "1";
```

然后会发现结果有变化

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/20211010194826365.png)

 **在Extra字段出现了一个Using index，说明使用了覆盖索引**

正如上面所说，查询的字段与索引中的字段对应，所以不需要回表查询，这样就提高了查询速度

我们甚至可以再添加一个id字段

```sql
EXPLAIN SELECT id, col1, col2 FROM tab1 WHERE col1 = "1";
```

结果与上面相同，也会使用覆盖索引

像下面这样单独查询索引字段也可以用到覆盖索引

```sql
EXPLAIN SELECT id FROM tab1 WHERE col1 = "1";

EXPLAIN SELECT col1 FROM tab1 WHERE col1 = "1";

EXPLAIN SELECT col2 FROM tab1 WHERE col1 = "1";
```

但如果查询col3，结果肯定是用不到覆盖索引的

```sql
EXPLAIN SELECT col3 FROM tab1 WHERE col1 = "1";
```

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/2021101019522246.png)

 **如果随便在col3上加一个索引也是无法使用覆盖索引的**

* * *

还有一点，我们知道复合索引需要符合最左匹配原则，否则会导致索引失效。但是**假如我们查询的字段仅为id或索引字段，索引也并不会完全失效**，比如

```sql
EXPLAIN SELECT id FROM tab1 WHERE col2 = "1";

EXPLAIN SELECT col1 FROM tab1 WHERE col2 = "1";

EXPLAIN SELECT col2 FROM tab1 WHERE col2 = "1";
```

 会出现如下结果

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/20211010195615966.png)

 type是index，而之前的type是ref，说明查找效率有一定下降，但是也用到了索引。**在Extra中同时出现了Using where和Using index，可以理解为MySQL在查询时知道这不符合最左匹配，但是因为查询的字段都在索引中，所以可以在相比整个表数据更小的索引文件中查找，查询速度也比全表查询更快**

## 十、索引下推

### 1. 基本概念

**索引下推（Index Condition Pushdown，ICP）** 是 MySQL 5.6 引入的一项重要优化技术。它允许在**存储引擎层**直接过滤掉不满足查询条件的记录，从而减少不必要的数据读取和回表操作。

### 2. 核心思想

将原本在 **MySQL 服务器层** 执行的部分过滤操作，"下推"到 **存储引擎层**（如 InnoDB）执行，从而减少数据在不同层级间的传输和处理。

### 3. 工作原理对比

#### 3.1 **没有 ICP（MySQL 5.6 之前）**

```sql
查询：SELECT * FROM t WHERE a > 1 AND b = 2
索引：(a,b)

执行流程：
1. 存储引擎：扫描索引，找到所有 a > 1 的记录（仅使用 a 列）
2. 存储引擎 → 服务器层：返回所有 a > 1 的记录（包含主键）
3. 服务器层：对每条记录检查 b = 2 的条件
4. 服务器层：对满足 b = 2 的记录回表获取完整数据
5. 服务器层：返回最终结果

问题：需要回表所有 a > 1 的记录，即使 b != 2
```

#### 3.2 **有 ICP（MySQL 5.6+）**

```sql
查询：SELECT * FROM t WHERE a > 1 AND b = 2
索引：(a,b)

执行流程：
1. 存储引擎：扫描索引，找到所有 a > 1 的记录
2. 存储引擎：在索引层面直接过滤掉 b != 2 的记录
3. 存储引擎 → 服务器层：仅返回满足 b = 2 的记录（包含主键）
4. 服务器层：对满足条件的记录回表获取完整数据
5. 服务器层：返回最终结果

优势：只回表真正满足所有条件的记录，减少 I/O 和 CPU 开销
```

### 4. 使用条件

#### 4.1 ICP 生效的条件

1、查询使用二级索引（非聚簇索引）：因为ICP主要就是减少回表开销，而聚簇索引无需回表，用不到ICP

2、WHERE 条件包含索引列的条件

3、条件不是用于索引查找（range/ref/eq_ref），而是用于过滤

4、表必须是 InnoDB 或 MyISAM 引擎

5、适用于 `SELECT`、`UPDATE`、`DELETE` 语句

#### 4.2 ICP 无法使用的情况

1、查询使用聚簇索引（主键索引）

2、条件引用了不在索引中的列

3、使用了虚拟生成列（Virtual Generated Column）的索引

4、子查询、函数索引等特殊情况

### 5. 实际示例

#### 5.1 示例表结构：

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    salary DECIMAL(10,2),
    hire_date DATE,
    INDEX idx_dept_salary (department_id, salary)
);
```

#### 5.2 示例1：复合索引的范围查询

```sql
-- 查询：部门ID在1-100之间，且工资大于5000的员工
SELECT * FROM employees 
WHERE department_id BETWEEN 1 AND 100 
  AND salary > 5000;

-- 没有ICP：先找到所有部门1-100的员工，再在服务器层过滤工资>5000
-- 有ICP：在存储引擎层就过滤掉工资<=5000的记录
```

#### 5.3 示例2：复合索引的部分使用

```sql
-- 查询：部门ID=5，且姓名包含'John'的员工（姓名不在索引中）
SELECT * FROM employees 
WHERE department_id = 5 
  AND name LIKE '%John%';

-- 没有ICP：先找到所有部门5的员工，再在服务器层过滤姓名
-- 有ICP：在存储引擎层找到部门5的员工时，可以直接过滤（但姓名不在索引中，所以ICP效果有限）
```

### 6. 如何确认使用了 ICP

使用 EXPLAIN 查看：

```sql
EXPLAIN SELECT * FROM employees 
WHERE department_id BETWEEN 1 AND 100 
  AND salary > 5000;
```

结果中的 Extra 列：

- **有 ICP**：`Using index condition`
- **无 ICP**：可能显示 `Using where`

### 7. 性能影响

#### 7.1 优势：

1、**减少回表次数**：只对真正满足条件的记录回表

2、**减少数据传输**：存储引擎到服务器层传输的数据减少

3、**提高缓存效率**：减少不必要的数据页读取

#### 7.2 性能提升场景：

- 索引选择性较差时（大量记录满足部分条件）
- 查询需要回表时
- WHERE 条件包含多个索引列，但只有部分用于查找

### 8. 控制 ICP

#### 8.1 查看 ICP 设置：

```sql
SHOW VARIABLES LIKE 'optimizer_switch';
-- 查看结果中是否有 'index_condition_pushdown=on'
```

#### 8.2 启用/禁用 ICP：

```sql
-- 全局禁用
SET GLOBAL optimizer_switch = 'index_condition_pushdown=off';

-- 会话级禁用
SET SESSION optimizer_switch = 'index_condition_pushdown=off';

-- 全局启用
SET GLOBAL optimizer_switch = 'index_condition_pushdown=on';
```

### 9. 与覆盖索引的区别

- 覆盖索引：避免回表，索引包含所有查询列

- ICP：减少回表，但在索引层面过滤

### 10. 总结

**索引下推（ICP）** 是 MySQL 优化器的一项重要优化，它：

1. 将 WHERE 条件的过滤操作从服务器层下推到存储引擎层
2. 减少不必要的数据传输和回表操作
3. 特别适用于复合索引中只有部分列用于查找的场景
4. 在 MySQL 5.6+ 中默认启用，通常能显著提升查询性能

理解 ICP 有助于我们更好地设计索引和编写高效的 SQL 查询，特别是在处理范围查询和复合索引时。

## 十一、索引相关问题

### 1. LIKE前后置%+复合索引的索引使用情况

首先建表

```sql
create table test03(
id int primary key not null auto_increment,
c1 varchar(10),
c2 varchar(10),
c3 varchar(10),
c4 varchar(10),
c5 varchar(10));
 
insert into test03(c1,c2,c3,c4,c5) values ('a1','a2','a3','a4','a5');
insert into test03(c1,c2,c3,c4,c5) values ('b1','b2','b3','b4','b5');
insert into test03(c1,c2,c3,c4,c5) values ('c1','c2','c3','c4','c5');
insert into test03(c1,c2,c3,c4,c5) values ('d1','d2','d3','d4','d5');
insert into test03(c1,c2,c3,c4,c5) values ('e1','e2','e3','e4','e5');
 
create index idx_test03_c1234 on test03(c1,c2,c3,c4);
```

查看用1-4个索引的key_len情况

```sql
EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1';
EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 = 'a2';
EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 = 'a2' AND c3 = 'a3';
EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 = 'a2' AND c3 = 'a3' AND c4 = 'a4';
```

结果分别为：43，86，129，172

* * *

执行以下5条语句，查看结果

```sql
EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 LIKE 'a2%' AND c3 = 'a3' AND c4= 'a4';

EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 LIKE '%a2' AND c3 = 'a3' AND c4= 'a4';

EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 LIKE '%a2%' AND c3 = 'a3' AND c4= 'a4';

EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 LIKE 'a%2%' AND c3 = 'a3' AND c4= 'a4';
```

**第一条**：使用了索引c1, c2, c3, c4。说明**后置%不影响最左匹配**

**第二条**：使用了索引c1。说明**前置%会导致自己及后边的索引失效**

**第三条**：使用了索引c1。分析同第二条

**第四条**：使用了索引c1, c2, c3, c4。说明**只要没有前置%，就不会影响最左匹配**

### 2. 如何让LIKE前置%也走索引

其实就是**利用覆盖索引**

举个例子

有这么一张表![202110101943431905](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202110101943431905.png)

有一个复合索引(col1, col2)

如果我们进行SELECT *查询

```sql
EXPLAIN SELECT * FROM tab1 WHERE col1 LIKE "%1";
```

 会是一个全表查询，但是如果将*改为col1和col2（也可以加上id），或者它们的任意组合，如

```sql
EXPLAIN SELECT col1, col2 FROM tab1 WHERE col1 LIKE "%1";

EXPLAIN SELECT id, col1, col2 FROM tab1 WHERE col1 LIKE "%1";

EXPLAIN SELECT id FROM tab1 WHERE col1 LIKE "%1";

EXPLAIN SELECT col1 FROM tab1 WHERE col1 LIKE "%1";

EXPLAIN SELECT col2 FROM tab1 WHERE col1 LIKE "%1";
```

结果就会变为

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111301352482.png)

type变为index，说明这个查询使用了索引

而且正如上面那个链接中的情况一样，如果不符合最左匹配原则，也可以走索引，如

```sql
EXPLAIN SELECT col1, col2 FROM tab1 WHERE col2 LIKE "%1";
```

**所以应该充分利用覆盖索引，查询时避免使用，而列出来具体需要的所有字段**

### 3. 复合索引+ORDER/GROUP BY相关的索引使用情况（filesort/temporary）

首先建表

```sql
create table test03(
id int primary key not null auto_increment,
c1 varchar(10),
c2 varchar(10),
c3 varchar(10),
c4 varchar(10),
c5 varchar(10));

insert into test03(c1,c2,c3,c4,c5) values ('a1','a2','a3','a4','a5');
insert into test03(c1,c2,c3,c4,c5) values ('b1','b2','b3','b4','b5');
insert into test03(c1,c2,c3,c4,c5) values ('c1','c2','c3','c4','c5');
insert into test03(c1,c2,c3,c4,c5) values ('d1','d2','d3','d4','d5');
insert into test03(c1,c2,c3,c4,c5) values ('e1','e2','e3','e4','e5');

create index idx_test03_c1234 on test03(c1,c2,c3,c4);
```

查看用1-4个索引的key_len情况

```sql
EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1';
EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 = 'a2';
EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 = 'a2' AND c3 = 'a3';
EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 = 'a2' AND c3 = 'a3' AND c4 = 'a4';
```

结果分别为：43，86，129，172

**但是需要注意OEDER/GROUP BY如果使用了索引并不会显示在key_len中（覆盖索引除外），对于OEDER BY的优化主要看Extra字段有无出现Using filesort，如果可以利用索引排序，则不会出现filesort。同样，对于GROUP BY的优化主要看Extra字段有无出现Using temporary，如果可以利用索引分组，则不会出现temporary**

**因为OEDER与GROUP对索引的使用情况是一样的，所以下面只用ORDER举例**

* * *

```sql
EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 = 'a2' ORDER BY c1;

EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 = 'a2' ORDER BY c2;

EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 = 'a2' ORDER BY c3;

EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 = 'a2' ORDER BY c4;
```

对于这4条语句，除了最后一条，均没有出现filersort。说明**如果符合最左匹配原则，就可以使用索引排序**

* * *

```sql
EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' ORDER BY c2, c3;

EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 = 'a2' ORDER BY c2, c3;

EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 = 'a2' ORDER BY c3, c2;

EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' ORDER BY c3, c2;
```

上面3条无filesort，下面有filesort 

* * *

```sql
EXPLAIN SELECT * FROM test03 ORDER BY c1;

EXPLAIN SELECT * FROM test03 WHERE c2 = 'a2' ORDER BY c1;
```

这两条均出现filesort，说明**如果没有WHERE语句或者有WHERE但不符合最左匹配，就会有filesort**

* * *

```sql
EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' ORDER BY c2, c3;

EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' ORDER BY c2, c3 DESC;
```

上面无filesort，下面有filesort。说明**降序OEDER BY也会导致索引失效**

* * *

```sql
EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' ORDER BY c2;

EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' ORDER BY c2 DESC;
```

两条均无filesort，但是下面有**Backward index scan**，说明**如果仅对一个索引项进行倒序排序，那么也可以避免filesort，使用反向索引扫描**

* * *

总结一下，**其实最主要的还是最左匹配原则，只要满足最左匹配，几乎就不会出现什么问题。但前提是要有WHERE语句**
