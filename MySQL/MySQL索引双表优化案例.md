# MySQL索引双表优化案例

首先通过如下SQL语句建两张表

```
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

```
EXPLAIN SELECT * FROM class LEFT JOIN book ON class.card = book.card;
```

因为没有建立过索引，所以结果都是全表查询

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111301339591.png)

下面要考虑的是：对于左连接，在左表建立索引更高效还是在右表建立索引更高效

可以直接尝试一下，首先尝试在左表class上建立索引

```
ALTER TABLE class ADD INDEX X(card);

EXPLAIN SELECT * FROM class LEFT JOIN book ON class.card = book.card;
```

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111301339228.png)

发现type已经转为index，有了一定的优化，然后删除该索引，在右表建立索引再查询

```
DROP INDEX X ON class;

ALTER TABLE book ADD INDEX Y(card);

EXPLAIN SELECT * FROM class LEFT JOIN book ON class.card = book.card;
```

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111301339137.png)

发现type变为了ref，比index效果更好

如果两个索引同时存在，会出现以下结果

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111301339804.png)

所以可以得到结论：**左连接时，在右表建立索引效果更好；反之，右连接时，在左表建立索引效果更好。但在内连接时，并没有什么区别**