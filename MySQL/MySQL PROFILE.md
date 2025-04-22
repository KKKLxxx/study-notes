# MySQL PROFILE

一、查看开启状态
--------

```
SHOW VARIABLES LIKE '%profiling%';

SHOW GLOBAL VARIABLES LIKE '%profiling%';
```

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/3bd8a9638ca2450f8d0d976200743ce8.png)

profiling为ON说明开启，size表示记录的数量

二、开启/关闭PROFILE
--------------

```
SET profiling = ON;

SET profiling = OFF;
```

三、查看PROFILE
-----------

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