# MySQL中JOIN的用法

JOIN用于连表查询，主要有5种用法。下面分别演示这5种用法

随便建2张表，结构如下

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/20211006222734636.png)              ![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/20211006222746529.png) 

字段col1用来使两张表有一个同名字段的（但其实没什么用，因为查询条件都需要用ON来指定，这里只是说明一下如果有相同的字段名也没什么影响）

## 一、笛卡尔积：CROSS JOIN

CROSS JOIN**使两张表的所有字段直接进行笛卡尔积**，假设表1有m条数据，表2有n条数据，则结果数量为m*n条

```
SELECT * FROM tab1 CROSS JOIN tab2
```

结果

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/20211006222821447.png)

二、内连接：INNER JOIN
----------------

内连接需要用ON来指定两张表需要比较的字段，最终结果只显示满足条件的数据

```
SELECT * FROM tab1 INNER JOIN tab2 ON tab1.id1 = tab2.id2
```

 结果

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/20211006222851479.png)

注意到**内连接只把满足ON条件的数据相连接**，与笛卡尔积不同

三、左连接：LEFT JOIN
---------------

左连接可以看做在内连接的基础上，把左表中不满足ON条件的数据也显示出来，但结果中的右表部分中的数据为NULL

```
SELECT * FROM tab1 LEFT JOIN tab2 ON tab1.id1 = tab2.id2
```

结果

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/20211006223153183.png)  

四、右连接：RIGHT JOIN
----------------

右连接就是与左连接完全相反

```
SELECT * FROM tab1 RIGHT JOIN tab2 ON tab1.id1 = tab2.id2
```

结果

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/20211006223235999.png)  

五、全连接：OUTER JOIN
----------------

全连接就是左连接和右连接的并集，但是MySQL中并不支持全连接的写法

```
SELECT * FROM tab1 OUTER JOIN tab2 ON tab1.id1 = tab2.id2
```

不过可以用UNION联合左连接和右连接的结果来代替

```
SELECT * FROM tab1 LEFT JOIN tab2 ON tab1.id1 = tab2.id2
UNION
SELECT * FROM tab1 RIGHT JOIN tab2 ON tab1.id1 = tab2.id2
```

结果

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/20211006223436638.png)