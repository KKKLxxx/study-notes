# Docker安装并运行Nacos

## 一、拉取镜像

```
docker pull nacos/nacos-server
```

拉取最新版本镜像

## 二、查看镜像

```
docker images
```

![image-20220129121728931](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202201291217961.png)

## 三、运行镜像

```
docker run -e MODE=standalone -e SPRING_DATASOURCE_PLATFORM=mysql -e MYSQL_SERVICE_HOST=111.111.111.111 -e MYSQL_SERVICE_DB_NAME=nacos_config -e MYSQL_SERVICE_USER=root -e MYSQL_SERVICE_PASSWORD=root --name nacos -d -p 8848:8848 nacos/nacos-server
```

`-e`后面均为环境变量