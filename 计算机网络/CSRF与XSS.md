# CSRF与XSS

## 一、CSRF跨站请求伪造(Cross-Site Request Forgery)

### 1、原理

具体场景：

用户C登录网站A后，浏览器保存了网站A的cookie

随后用户C登录网站B，网站B返回一些攻击性代码

这段代码通过用户C的cookie，冒用了用户C的身份，访问了网站A，进行一些操作

### 2、防范

1、验证HTTP Referer字段

缺点：用户如果自己设置不提供Referer，则会误拦截

2、在请求中添加token

CSRF攻击成功是因为用户验证信息都存在cookie中，所以可以在请求参数中添加一个随机token，并在服务端验证

为防止被站外链接获取token，可以在发送请求时判断请求的链接是否为站内链接，如果是则加token，反之不加

## 二、XSS跨站脚本攻击(Cross Site Scripting)

### 1、原理

攻击者在网页的链接中插入恶意的代码，用于盗取用户信息、修改用户设置

### 2、防范

1、输入验证

禁止输入特殊字符，过滤关键字，如`<script>`等

2、输出转义

在HTML中插入不可信数据时，要提前对数据进行转义，类似

```
s = str.replace(/&/g,"&amp;");
s = s.replace(/</g,"&lt;");
s = s.replace(/>/g,"&gt;");
s = s.replace(/ /g,"&nbsp;");
s = s.replace(/\'/g,"&#39;");
s = s.replace(/\"/g,"&quot;");
```

3、使用httponly cookie

将重要的cookie标记为httponly，这样的话当浏览器向Web服务器发起请求的时就会带上cookie字段，但是**在js脚本中却不能访问这个cookie**，这样就避免了XSS攻击利用JavaScript的document.cookie获取cookie