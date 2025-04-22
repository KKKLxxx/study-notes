# CentOS7.6安装并使用Nginx

## 一、gcc 安装
安装 nginx 需要先将官网下载的源码进行编译，编译依赖 gcc 环境，如果没有 gcc 环境，则需要安装：

```sql
yum install -y gcc-c++
```

## 二、PCRE pcre-devel 安装
PCRE(Perl Compatible Regular Expressions) 是一个Perl库，包括 perl 兼容的正则表达式库。nginx 的 http 模块使用 pcre 来解析正则表达式，所以需要在 linux 上安装 pcre 库，pcre-devel 是使用 pcre 开发的一个二次开发库。nginx也需要此库。命令：

```
yum install -y pcre pcre-devel
```

## 三、zlib 安装
zlib 库提供了很多种压缩和解压缩的方式， nginx 使用 zlib 对 http 包的内容进行 gzip ，所以需要在 CentOS 上安装 zlib 库。

```
yum install -y zlib zlib-devel
```

## 四、OpenSSL 安装
OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议，并提供丰富的应用程序供测试或其它目的使用。
nginx 不仅支持 http 协议，还支持 https（即在ssl协议上传输http），所以需要在 CentOS 安装 OpenSSL 库。

```
yum install -y openssl openssl-devel
```

## 五、下载Nginx

在官网下载：http://nginx.org/en/download.html

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202202011729293.png" alt="image-20220201172904257" style="zoom: 67%;" />

选择稳定版本即可，Linux环境下点击中间的`nginx-1.20.2`下载

## 六、解压

将压缩包移入Linux的任意目录下，通过以下指令解压并进入解压后的文件夹

```
tar -zxvf nginx-1.20.2.tar.gz
cd nginx-1.20.2
```

## 七、配置

通过以下指令，使用默认配置即可

```
./configure
```

## 八、编译安装

```
make
make install
```

## 九、配置代理

首先通过以下指令找到nginx的安装目录

```
whereis nginx
```

![image-20220201191021947](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202202011910975.png)

则代理配置文件在`/usr/local/nginx/conf/nginx.conf`中

在里面添加

```
server {
    listen 8002;
    server_name 11.2.33.43;

    location / {
        root /k-blog/dist;
        index index.html index.htm;
    }
}
```

意思是当访问`http://11.2.33.43:8002`这个地址时，会被代理访问到`/k-blog/dist`目录下的`index.html`主页

如果需要配置域名访问，则将`server_name`修改为自己的域名即可

```
server {
    listen 80;
    server_name www.xxx.com;

    location / {
        root /k-blog/dist;
        index index.html index.htm;
    }
}
```

但是需要自行配置域名解析

需要注意的是，因为会默认访问80端口，所以在浏览器输入`www.xxx.com`时相当于访问`www.xxx.com:80`。如果想要配置其他端口，且不加端口号，比如输入`www.xxx.com`但实际访问的是`www.xxx.com:8002`，则需要进行一次隐形URL解析。但是这会导致网页的标题和logo无法显示，暂时还未找到解决方案

## 十、启动、停止、重启Nginx

需要在安装目录下的`sbin`目录下执行指令

`/usr/local/nginx/sbin`

```
./nginx 
./nginx -s quit
./nginx -s stop
./nginx -s reload
```

这4条指令的作用分别是启动、停止、强制停止（查询相应进程id并kill）、重启

