# MySQL中varchar字段使用数字查询的特殊现象

在一次偶然中发现对varchar类型的字段查询时，没有严格将查询条件用引号括起来，导致出现了一些意外的结果

比如这么一张表，其中col为varchar类型

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/20211006224554131.png)

* * *

执行

```
SELECT * FROM tab1 WHERE col = 0
```

结果

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/20211006224633858.png)

* * *

执行

```
SELECT * FROM tab1 WHERE col = 1
```

结果

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/20211006224709413.png)

* * *

执行

```
SELECT * FROM tab1 WHERE col = 2
```

结果 

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/20211006224735730.png)

* * *

**总结如下**

对于col = 0的条件，可以匹配到所有（非数字开头）或（仅以0开头）或（单独一个数字0）的数据

对于col = x（其中x为数字，包括(正/负)(整/小)数）的条件，会匹配以x开头的所有数据，但如果x之后紧接着其他数字则不会被匹配。比如当x=11时，11a匹配，111不匹配，111a也不匹配

* * *

当然这只是一个偶然的发现，实际上肯定不会这么用的。所以**对varchar字段查询的时候一定要记得加引号**