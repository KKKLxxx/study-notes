# MySQL慢查询分析方法

## 一、慢查询日志

### 1. 检验慢查询是否开启，查看日志位置

```
SHOW VARIABLES LIKE '%slow_query_log%';
```

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/e12843a128854b95b54878fe798ed111.png)

**ON说明开启，OFF说明关闭**

日志位置方面，在**Windows10，MySQL8**的环境下，日志文件在

```
C:\ProgramData\MySQL\MySQL Server 8.0\Data\KKKL-slow.log
```

中

### 2. 开启/关闭慢查询日志

```
SET GLOBAL slow_query_log = 0;

SET GLOBAL slow_query_log = 1;
```

0关闭，1开启

### 3. 查询慢查询时间阈值

```
SHOW VARIABLES LIKE '%long_query_time%';

SHOW GLOBAL VARIABLES LIKE '%long_query_time%';
```

注意有2种，**一种加GLOBAL，可以应用于除当前会话外的所有会话；不加GLOBAL的话，只有当前会话生效**

**但是尽管设置了GLOBAL，当MySQL服务重启后，仍然会按照my.ini的配置重置**

**my.ini文件位置**

```
C:\ProgramData\MySQL\MySQL Server 8.0\my.ini
```

### 4. 设置慢查询时间阈值

```
SET long_query_time = 5;

SET GLOBAL long_query_time = 5;
```

### 5. 查询慢日志文件

假设当前慢查询阈值为3，通过以下命令进行一个慢查询

```
SELECT SLEEP(4);
```

在慢查询日志文件中会出现这么一条记录

```
# Time: 2021-10-31T14:52:45.633715Z
# User@Host: root[root] @ localhost [::1]  Id:    10
# Query_time: 4.000819  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 1
use learning;
SET timestamp=1635691961;
SELECT SLEEP(4);
```

就可以准确定位慢查询语句了

### 6. 查询慢查询语句条数

```
SHOW STATUS LIKE '%Slow_queries%';

SHOW GLOBAL STATUS LIKE '%Slow_queries%';
```

## 二、EXPLAIN

一个例子

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111301313118.png)

下面分别解释这12个字段

### 1. id

select**查询的序列号**，表示查询中执行select子句或操作表的顺序

三种情况：

**1、id相同，执行顺序由上至下**

2、**id不同**，如果是子查询，id的序号会递增，**id值越大优先级越高，越先被执行**

3、id相同、不同同时存在

id如果相同，可以认为是一组，从上往下顺序执行；  
在所有组中，id值越大，优先级越高，越先执行

### 2. select_type

**查询的类型**，主要是用于区别普通查询、联合查询、子查询等的复杂查询

**1、SIMPLE**

简单的 select 查询，查询中不包含子查询或者UNION

**2、PRIMARY**

查询中若包含任何复杂的子部分，最外层查询则被标记为Primary

**3、DERIVED**

在FROM列表中包含的子查询被标记为DERIVED(衍生)，MySQL会递归执行这些子查询，把结果放在临时表里

**4、SUBQUERY**

在SELECT或WHERE列表中包含了子查询

**5、DEPENDENT SUBQUERY**

在SELECT或WHERE列表中包含了子查询，子查询基于外层

dependent subquery 与 subquery 的区别  
依赖子查询 ： 子查询结果为 多值  
子查询：查询结果为 单值

**6、UNCACHEABLE SUBQUREY**

无法被缓存的子查询

**7、UNION**

若第二个SELECT出现在UNION之后，则被标记为UNION；若UNION包含在FROM子句的子查询中，外层SELECT将被标记为DERIVED

UNION RESULT 两个语句执行完后的结果

**8、UNION RESULT**

即UNION后的结果

### 3. table

显示这一行的数据是关于哪张表的

### 4. partitions

使用的哪个分区，需要结合表分区才可以看到

### 5. type

type显示的是访问类型，是较为重要的一个指标，结果值从最好到最坏依次是（其中加粗的比较常见）：   
**system** \> **const** \> **eq_ref** \> **ref** \> fulltext > ref\_or\_null > index\_merge > unique\_subquery > index_subquery > **range** \> **index** \> **ALL**  
一般来说，得保证查询至少达到range级别，最好能达到ref

**1、system**

表只有一行记录（等于系统表），这是const类型的特列，平时不会出现，这个也可以忽略不计

**2、const**

表示通过索引一次就找到了，const用于比较primary key或者unique索引  
如将主键置于where列表中，MySQL就能将该查询转换为一个常量

**3、eq_ref**

唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描

**4、ref**

非唯一性索引扫描，返回匹配某个单独值的所有行

**5、range**

只检索给定范围的行，使用一个索引来选择行，一般出现在where语句中使用了between、<、>、in等的查询  
这种范围扫描索引扫描比全表扫描要好，因为它只需要开始于索引的某一点，结束于另一点，不用扫描全部索引

**6、index**

index与all区别：index可以通过索引树形成的顺序来对数据表进行有序遍历，而非随机遍历

index与range和ref的区别：index虽然可以有序遍历，但是仍然要先以索引-数据-索引-数据...的方式将索引和数据全部遍历一遍，而后两者可以通过索引直接定位具体的一行或多行，不用遍历多余的数据

**7、all**

直接将遍历全表以找到匹配的行

**8、index_merge**

在查询过程中需要多个索引组合使用，通常出现在有 or 的关键字的sql中

**9、 ref\_or\_null**

对于某个字段既需要关联条件，也需要null值得情况下。查询优化器会选择用ref\_or\_null连接查询

**10、index_subquery**

利用索引来关联子查询，不再全表扫描

**11、unique_subquery **

该联接类型类似于index_subquery。 子查询中的唯一索引

### 6. possible_keys

查询涉及到的字段上若存在索引，则该索引将被列出，但**不一定被查询实际使用**

### 7. key

实际使用的索引。如果为NULL，则没有使用索引

### 8. key_len

表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度

key_len应尽量短，越短效率越高

### 9. ref

哪些列或常量被用于查找索引列上的值。如果是列，则显示字段名；如果是常数，则显示const

### 10. rows

显示MySQL认为它执行查询时必须检查的行数，越小越好

### 11. filtered

表示返回结果的行数占需读取行数的百分比，越大越好

### 12. Extra

一些额外信息

因为下面会需要举很多例子，所以假设一张表test(id, a, b, c, d, e)，有一个复合索引(a, b, c, d)

* * *

**注意字段是varchar类型！！如果是char会有不同的结果，在此只讨论varchar**

* * *

**1、Using filesort **

**无法利用索引完成的排序操作称为“文件排序”**

出现filesort的情况：比如

```sql
SELECT * FROM test WHERE a = 1 ORDER BY c;
```

因为跳过了b，所以根据c排序时，MySQL**不能直接根据索引排序，需要自己重新进行文件排序**

如果存在复合索引(a,c)，那么经过a字段筛选后得到的数据就是根据c排序后的数据，无需文件排序

**2、Using temporary **

**使用了临时表保存中间结果**，MySQL在对查询结果排序时使用临时表。常见于分组查询group by

```sql
SELECT * FROM test WHERE a = 1 GROUP BY c;
```

**3、Using index**

表示相应的select操作中**使用了覆盖索引**，避免访问了表的数据行

```sql
SELECT a FROM test WHERE a = 1;
```

**id（主键）永远属于任意索引字段的一部分**

**4、Using where**

**使用了索引，但查询条件不都走索引**

```sql
EXPLAIN SELECT * FROM test WHERE a = 1 AND e = 2
```

**5、Using index condition**

用到了索引，但是**所查询的字段不全都在使用的索引中**，没有实现索引覆盖，需要回表查询。但是这个表经过了索引过滤，比全表查询效率高

**6、Using index + Using where**

表明虽然没有索引覆盖，但是**需要查找的字段都在所使用的索引中**，不用回表查询。效率比Using index condition高

**7、Using join buffer**

使用了连接缓存：

出现在当两个连接时  
驱动表(被连接的表，left join 左边的表。inner join 中数据少的表) 没有索引的情况下。  
给驱动表建立索引可解决此问题。且 type 将改变成 ref

## 三、PROFILE

### 1. 查看开启状态

```
SHOW VARIABLES LIKE '%profiling%';

SHOW GLOBAL VARIABLES LIKE '%profiling%';
```

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/3bd8a9638ca2450f8d0d976200743ce8.png)

profiling为ON说明开启，size表示记录的数量

### 2. 开启/关闭PROFILE

```
SET profiling = ON;

SET profiling = OFF;
```

### 3. 查看PROFILE

随便进行一个查询，然后查看它的query id

```
SELECT * FROM test03;

SHOW PROFILES;
```

得到query id后，便可通过以下语句查询具体耗时

```
SHOW PROFILE FOR QUERY 154;

SHOW PROFILE CPU, BLOCK IO FOR QUERY 154;

SHOW PROFILE ALL FOR QUERY 154;
```

可以指定查询的字段