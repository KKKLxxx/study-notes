# DNS

## 一、作用

将域名解析为IP地址，用于建立连接

## 二、报文结构

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/XxjzuCiKblQNDUv.gif)

## 三、解析过程

![img](https://raw.githubusercontent.com/KKKLxxx/img-host/master/EZDYxOqFNWJgKyt.webp)

### 1、本地缓存

本地解析主要查询3个地方是否有缓存：浏览器缓存、操作系统缓存、本地host文件

### 2、本地DNS服务器

包括缓存服务器和递归服务器。如果本地缓存中无法解析，则需要请求本地DNS服务器进行解析

**缓存服务器**：是一些本地运营商提供的DNS服务器

**递归服务器**：如果缓存服务器无法完成解析，则需要通过递归服务器请求根域名服务器

### 3、根域名服务器

递归服务器以此请求根域名服务器、顶级域名服务器、权威域名服务器进行域名解析

![img](https://raw.githubusercontent.com/KKKLxxx/img-host/master/yYBgRac1AnEVLeF.webp)

**根域名服务器**：根据顶级域名(如.com/.cn)分配给不同的顶级域名服务器解析

**顶级域名服务器**：分配给权威域名服务器解析

**权威域名服务器**：解析并返回给本地DNS服务器

## 四、常见资源记录类型

| 记录名  | 中文名   | 作用                                                         |
| ------- | -------- | ------------------------------------------------------------ |
| A、AAAA | 主机记录 | 记录域名和IP地址的对应关系。IPv4使用的是A记录，IPv6使用的是AAAA记录 |
| CNAME   | 别名记录 | 记录该域名与另一域名的对应关系                               |

## 五、DNS劫持

DNS劫持即通过修改DNS缓存、重定向本地DNS服务器或权威域名服务器的方式，返回错误的解析结果

解决方法：手动指定DNS服务器地址