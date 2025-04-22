#  TCP与HTTP中KeepAlive的区别

## 一、TCP中的KeepAlive

TCP中的KeepAlive用于检测双方连接是否断开。每隔一段时间，双方会通过心跳包来检测连接是否断开

**作用**：及时清理失效的连接，减少资源占用

## 二、HTTP中的keep-alive

在请求头中会有这么一行`Connection: keep-alive`

说明这是一个长连接，多个请求可以复用同一个TCP连接

**作用**：复用连接，减少连接重复建立、断开的消耗

