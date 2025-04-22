# 复合索引+ORDER/GROUP BY相关的索引使用情况（filesort/temporary）

首先建表

```
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

```
EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1';
EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 = 'a2';
EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 = 'a2' AND c3 = 'a3';
EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 = 'a2' AND c3 = 'a3' AND c4 = 'a4';
```

结果分别为：43，86，129，172

**但是需要注意OEDER/GROUP BY如果使用了索引并不会显示在key_len中（覆盖索引除外），对于OEDER BY的优化主要看Extra字段有无出现Using filesort，如果可以利用索引排序，则不会出现filesort。同样，对于GROUP BY的优化主要看Extra字段有无出现Using temporary，如果可以利用索引分组，则不会出现temporary**

**因为OEDER与GROUP对索引的使用情况是一样的，所以下面只用ORDER举例**

* * *

```
EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 = 'a2' ORDER BY c1;

EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 = 'a2' ORDER BY c2;

EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 = 'a2' ORDER BY c3;

EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 = 'a2' ORDER BY c4;
```

对于这4条语句，除了最后一条，均没有出现filersort。说明**如果符合最左匹配原则，就可以使用索引排序**

* * *

```
EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' ORDER BY c2, c3;

EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 = 'a2' ORDER BY c2, c3;

EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 = 'a2' ORDER BY c3, c2;

EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' ORDER BY c3, c2;
```

上面3条无filesort，下面有filesort 

* * *

```
EXPLAIN SELECT * FROM test03 ORDER BY c1;

EXPLAIN SELECT * FROM test03 WHERE c2 = 'a2' ORDER BY c1;
```

这两条均出现filesort，说明**如果没有WHERE语句或者有WHERE但不符合最左匹配，就会有filesort**

* * *

```
EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' ORDER BY c2, c3;

EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' ORDER BY c2, c3 DESC;
```

上面无filesort，下面有filesort。说明**降序OEDER BY也会导致索引失效**

* * *

```
EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' ORDER BY c2;

EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' ORDER BY c2 DESC;
```

两条均无filesort，但是下面有**Backward index scan**，说明**如果仅对一个索引项进行倒序排序，那么也可以避免filesort，使用反向索引扫描**

* * *

总结一下，**其实最主要的还是最左匹配原则，只要满足最左匹配，几乎就不会出现什么问题。但前提是要有WHERE语句**