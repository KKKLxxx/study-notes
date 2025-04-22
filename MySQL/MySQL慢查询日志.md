# MySQL慢查询日志

一、检验慢查询是否开启，查看日志位置
------------------

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

二、开启/关闭慢查询日志
------------

```
SET GLOBAL slow_query_log = 0;

SET GLOBAL slow_query_log = 1;
```

0关闭，1开启

三、查询慢查询时间阈值
-----------

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

四、设置慢查询时间阈值
-----------

```
SET long_query_time = 5;

SET GLOBAL long_query_time = 5;
```

五、查询慢日志文件
---------

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

六、查询慢查询语句条数
-----------

```
SHOW STATUS LIKE '%Slow_queries%';

SHOW GLOBAL STATUS LIKE '%Slow_queries%';
```