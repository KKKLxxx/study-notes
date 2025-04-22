# 如何让LIKE前置%也走索引

其实就是利用覆盖索引

[覆盖索引与回表_KKKL的博客-CSDN博客](https://blog.csdn.net/qq_45404693/article/details/120690427 "覆盖索引与回表_KKKL的博客-CSDN博客")

举个例子

有这么一张表![202110101943431905](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202110101943431905.png)

有一个复合索引(col1, col2)

如果我们进行SELECT *查询

```
EXPLAIN SELECT * FROM tab1 WHERE col1 LIKE "%1";
```

 会是一个全表查询，但是如果将*改为col1和col2（也可以加上id），或者它们的任意组合，如

```
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

```
EXPLAIN SELECT col1, col2 FROM tab1 WHERE col2 LIKE "%1";
```

**所以应该充分利用覆盖索引，查询时避免使用，而列出来具体需要的所有字段**