# LIKE前后置%+复合索引的索引使用情况

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

* * *

执行以下5条语句，查看结果

```
EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 LIKE 'a2%' AND c3 = 'a3' AND c4= 'a4';

EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 LIKE '%a2' AND c3 = 'a3' AND c4= 'a4';

EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 LIKE '%a2%' AND c3 = 'a3' AND c4= 'a4';

EXPLAIN SELECT * FROM test03 WHERE c1 = 'a1' AND c2 LIKE 'a%2%' AND c3 = 'a3' AND c4= 'a4';
```

**第一条**：使用了索引c1, c2, c3, c4。说明**后置%不影响最左匹配**

**第二条**：使用了索引c1。说明**前置%会导致自己及后边的索引失效**

**第三条**：使用了索引c1。分析同第二条

**第四条**：使用了索引c1, c2, c3, c4。说明**只要没有前置%，就不会影响最左匹配**