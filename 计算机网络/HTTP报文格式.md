# HTTP报文格式

## 1. 请求报文

```http
POST /api/login HTTP/1.1
Host: www.example.com
Content-Type: application/json
Content-Length: 56
User-Agent: MyApp/1.0.0
Accept: application/json
Connection: keep-alive
Cookie: session_id=abc123def456

{"username": "john_doe", "password": "secret123"}
```

**请求行**：请求方法（GET/POST）、请求地址、HTTP版本

**请求头**：Host、User-Agent（浏览器类型）、Connection（连接方式，如keep-alive）、Cookie

- **短连接**：一次请求结束后就关闭连接，便于管理，但需要重复建立TCP连接
- **长连接**：连接可以复用（Keep-Alive），一段时间内没有HTTP请求则断开

**空行**：发送回车符和换行符，通知服务器以下不再有请求头

**请求体**：请求体不在GET方法中使用，而是在POST方法中使用

## 2. 响应报文

```http
HTTP/1.1 200 OK
Date: Mon, 23 May 2022 22:38:34 GMT
Server: nginx/1.18.0
Content-Type: application/json; charset=utf-8
Content-Length: 89
Connection: keep-alive
Cache-Control: no-cache, no-store, must-revalidate
Set-Cookie: session=abc123; Path=/; HttpOnly; Secure
Access-Control-Allow-Origin: *
ETag: "abc123def456"

{
  "status": "success",
  "data": {
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com"
  }
}
```

**状态行**：即HTTP状态码

**响应头**：Content-Type（json/html/图片/纯文本）、Content-Length、Content-Encoding（数据的压缩方法）

**空行**：发送回车符和换行符，通知客户端以下不再有响应头

**响应体**：响应的数据

