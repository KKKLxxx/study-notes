# Redis过期键实现定时任务

项目源码：https://gitee.com/KKKLxxx/schedule-task



Redis过期键实现定时任务的思路就是

1、提供一个controller接收请求，设置键的内容及其过期时间，其中键的内容就是键过期后需要执行的任务内容

2、提供一个service监听redis键过期事件，并根据键的内容执行不同的业务

由于Redis在定时任务这方面的功能比较有限，所以可以充分利用键来规定任务的内容、过期时间等信息



服务启动后的测试：

在postman或网页中输入以下url

```
http://localhost:8080/putExKey?key=b
```

控制台会输入类似如下内容

```
2022-02-14 18:31:29.706  INFO 15844 --- [nio-8080-exec-2]                                          : 业务开始时间：Mon Feb 14 18:31:29 CST 2022
2022-02-14 18:31:39.811  INFO 15844 --- [    container-2]                                          : 十秒钟时的时间：Mon Feb 14 18:31:39 CST 2022
```

说明键b在10s后过期的事件成功被监听到

