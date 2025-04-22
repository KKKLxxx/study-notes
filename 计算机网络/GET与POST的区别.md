# GET与POST的区别

## 区别

1、**参数传递方式**：get参数通过URL传递，post的参数放在request body中

2、**参数长度限制**：get有限制，post无限制

3、**幂等性**：get是幂等的，多次请求的结果相同，post会重复提交

4、**缓存**：get可以被缓存，post不可缓存

## 关于GET/POST请求产生的TCP包个数

网上有很多地方说，get产生一个TCP包，post产生两个。因为get一次性把header和data发送出去，服务器响应200；post先发送header，服务器响应100 continue，再发送data，服务器响应200

但是自己实际测试的时候发现post也是只发一个包

## GET能否带请求体

可以，但一般不用，有的工具不支持GET+请求体

