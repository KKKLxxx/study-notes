

# Docker安装并运行MySQL镜像

## 一、拉取镜像

```
docker pull mysql
```

拉取最新版本镜像

## 二、查看镜像

```
docker images
```

![image-20220129113500628](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202201291135711.png)

## 三、运行镜像

```
docker run -itd --name mysql-test -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root mysql
```

`mysql-test`为容器名

`3306:3306`为端口映射

`MYSQL_ROOT_PASSWORD=root`在设置数据库连接密码

## 四、修改时区

在Navicat中，开启“根据当前时间戳更新”时，会根据当前时间来更新时间，但是这个时间与MySQL时区有关

可以通过

```
show variables like "%time_zone%";
```

查看当前的时区，默认跟随系统时区，可能是UTC时区

可以通过修改MySQL配置文件修改（mysql容器中需要先自己安装vim）

```
apt-get update -y
apt-get install vim -y
vim /etc/mysql/my.cnf
```

在[mysqld]下新增一行

```
default-time_zone = '+8:00'
```

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20221007212643213.png" alt="image-20221007212643213" style="zoom: 80%;" />

然后重启容器即可
