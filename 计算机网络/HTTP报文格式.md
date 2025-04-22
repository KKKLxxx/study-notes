# HTTP报文格式

## 请求报文

**请求行**：请求方法(GET/POST)、请求地址、http版本

**请求头**：浏览器类型(User-Agent)、cookie、连接方式(Connection: keep-alive)

**空行**：发送回车符和换行符，通知服务器以下不再有请求头

**请求体**：请求数据不在GET方法中使用，而是在POST方法中使用

## 响应报文

**状态行**：即http状态码

**响应头**：content-type（json/html/图片/纯文本）、content-length、content-encoding

**空行**：发送回车符和换行符，通知客户端以下不再有响应头

**响应体**

### 短连接与长连接

**短连接**：一次请求结束后就关闭连接

**优点**：便于管理

**缺点**：需要重复建立TCP连接

**长连接**：连接可以复用（Keep-Alive），一段时间内没有HTTP请求则断开