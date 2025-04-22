# MySQL索引失效案例

通过一张表和不同的查询语句，来实际验证一下索引失效（环境：Windows，MySQL 8.0.23）

建表SQL

```
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

  

```
EXPLAIN SELECT * FROM staffs WHERE name = "1"; -- 1

EXPLAIN SELECT * FROM staffs WHERE name = "1" AND age = 1; -- 2

EXPLAIN SELECT * FROM staffs WHERE name = "1" AND age = 1 AND pos = "1"; -- 3
```

解释一下，**每行语句后面的注释代表实际上用了多少个索引，索引使用数量可以通过key_len字段得知**

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/20211009232508798.png)![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/20211009232521896.png) ![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/20211009232530149.png) 

本例中使用1、2、3个索引的情况下的key_len分别74、78、140

* * *

一、不符合最左匹配原则
-----------

这个其实很经典了，就是如果不使用name字段，就会导致失效，类似下面两条

```
EXPLAIN SELECT * FROM staffs WHERE age = 1 AND pos = "1"; -- 0

EXPLAIN SELECT * FROM staffs WHERE pos = "1"; -- 0
```

**但是需要注意，如果是下面这种情况，索引是可以完全生效的**

```
EXPLAIN SELECT * FROM staffs WHERE pos = "1" AND age = 1 AND name = "1"; -- 3
```

因为MySQL优化器可以自动将AND左右的条件调换顺序，使之符合最左匹配原则

二、复合索引中间字段未使用
-------------

如果不使用中间的age字段，会导致age和pos都失效，即**本身和后面的索引都失效**

```
EXPLAIN SELECT * FROM staffs WHERE name = "1" AND pos = "1"; -- 1
```

三、复合索引前面的字段使用了（范围查询/不等于查询）
--------------------------

复合索引前面的字段使用了（范围查询/不等于查询）**导致后面的索引失效，但它本身会有效**

```
EXPLAIN SELECT * FROM staffs WHERE name > "1" AND age = 1 AND pos = "1"; -- 1

EXPLAIN SELECT * FROM staffs WHERE name = "1" AND age > 1 AND pos = "1"; -- 2

EXPLAIN SELECT * FROM staffs WHERE name != "1" AND age = 1 AND pos = "1"; -- 1
```

四、使用了is null / is not null
--------------------------

这会导致**本身和后面的索引都失效**，比如

```
EXPLAIN SELECT * FROM staffs WHERE name is not null; -- 0

EXPLAIN SELECT * FROM staffs WHERE name is not null AND age = 1 AND pos = "1"; -- 0

EXPLAIN SELECT * FROM staffs WHERE name = "1" AND age is not null AND pos = "1"; -- 1
```

**这里注意到我上面说建表时有没有规定NOT NULL会有区别，如果没有规定NOT NULL，该字段也可以走索引，并且不影响后面的字段走索引**

五、LIKE前置%
---------

也很经典了

```
EXPLAIN SELECT * FROM staffs WHERE name LIKE "%1"; -- 0
```

**但是需要注意也可以通过覆盖索引使得前置%也走索引**

[如何让LIKE前置%也走索引_KKKL的博客-CSDN博客](https://blog.csdn.net/qq_45404693/article/details/120694101 "如何让LIKE前置%也走索引_KKKL的博客-CSDN博客")

六、varchar使用数字查询
---------------

因为name是varchar类型的，查询的时候需要将条件用引号括起来，如果像下面一样，就会索引失效，而且查询结果也会很特别

```
EXPLAIN SELECT * FROM staffs WHERE name = 1; -- 0
```

可以参考这篇文章

[MySQL中varchar字段使用数字查询的特殊现象_KKKL的博客-CSDN博客](https://blog.csdn.net/qq_45404693/article/details/120630862 "MySQL中varchar字段使用数字查询的特殊现象_KKKL的博客-CSDN博客")

七、OR前后字段未均有索引
-------------

虽然name有索引，但是add_time没有索引，如果将这两个字段OR在一起，会导致**完全不走索引**

```
EXPLAIN SELECT * FROM staffs WHERE name = "1" OR name = "2"; -- 1

EXPLAIN SELECT * FROM staffs WHERE name = "1" OR add_time = "2021-10-09 23:12:56"; -- 0
```

**八、使用了函数**
-----------

```
EXPLAIN SELECT * FROM staffs WHERE name = "1"; -- 1

EXPLAIN SELECT * FROM staffs WHERE left(name, 4) = "1"; -- 0
```