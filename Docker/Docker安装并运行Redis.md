# Docker安装并运行Redis

## 一、拉取镜像

```
docker pull redis
```

拉取最新版本镜像

## 二、查看镜像

```
docker images
```

![image-20220129140227826](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202201291402860.png)

## 三、运行镜像

```
docker run -d --name redis-test -p 63791:6379 redis
```

## 四、远程连接Redis

在本地（安装了redis客户端的目录）可通过执行以下命令连接远程服务器上的Redis

```
redis-cli -h 111.111.111.111 -p 63791
```

`-h`指定主机号，`-p`指定端口号

