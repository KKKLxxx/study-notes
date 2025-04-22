# MySQL索引单表优化案例

首先建一张表

```
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

```
EXPLAIN SELECT id, author_id FROM article WHERE category_id = 1 AND comments > 0 ORDER BY views DESC LIMIT 1;
```

结果如下

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111301312766.png)

发现是一个全表查询，所以需要添加索引。直观上是需要建立一个(category_id, comments, views)的复合索引，所以尝试建立后再查询

```
CREATE INDEX idx_ccv ON article(category_id, comments, views);
EXPLAIN SELECT id, author_id FROM article WHERE category_id = 1 AND comments > 0 ORDER BY views DESC LIMIT 1;
```

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111301312475.png)

发现虽然用到了索引，但是在Extra中有Using filesort，说明排序的时候没有按照索引进行排序，这会导致性能有一定下降

**索引失效的原因是：虽然复合索引的3个字段按顺序出现在查询语句中了，但是comments字段是一个范围查询，导致后面的索引失效**

可以尝试运行以下查询语句，其中将comments的范围查询改为了等值查询

```
EXPLAIN SELECT id, author_id FROM article WHERE category_id = 1 AND comments = 0 ORDER BY views DESC LIMIT 1;
```

结果变为了

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111301312838.png)

type已经达到ref，Extra中也没有Using filesort，说明已经优化的不错了。但是这里是改变了查询语句，没有达到原来的目的。我们仍然需要优化索引

```
DROP INDEX idx_ccv ON article;

CREATE INDEX idx_cv ON article(category_id, views);

EXPLAIN SELECT id, author_id FROM article WHERE category_id = 1 AND comments > 0 ORDER BY views DESC LIMIT 1;
```

对索引做出如上修改，结果如下

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111301313360.png)

通过type和Extra的信息可知，该索引优化完成了