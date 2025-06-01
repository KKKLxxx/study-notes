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
docker run -d --name mysql-test -p 33061:3306 -e MYSQL_ROOT_PASSWORD=root mysql --default-time-zone=Asia/Shanghai
```

`-d`表示在后台运行容器

`--name mysql-test`用于指定容器名，表示将`mysql-test`设为容器名

`-p 33061:3306`用于端口映射，表示将`宿主机33061端口`映射到`容器3306端口`

`-e MYSQL_ROOT_PASSWORD=root`在设置数据库连接密码

`mysql`表示默认使用`mysql:latest`镜像，可通过`mysql:tag`具体指定某一个版本

`--default-time-zone=Asia/Shanghai`可以设置MySQL的时区，在Navicat中，开启“根据当前时间戳更新”时，会根据当前时区的时间来更新记录更新的时间。可以通过

```
show variables like "%time_zone%";
```

查看当前的时区，默认跟随系统时区（设置成功后应显示为time_zone=Asia/Shanghai）

---

由于设定为后台运行，所以指令执行成功后应仍在宿主机的界面

## 四、访问容器

执行如下指令后，再输入一次密码（root），即可进入docker容器

```
docker exec -it mysql-test mysql -u root -p
```

界面如下：

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20250601151544282.png" alt="image-20250601151544282" style="zoom:67%;" />

## 五、Navicat连接MySQL

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20250601151933877.png" alt="image-20250601151933877" style="zoom: 67%;" />

注意端口号要根据创建容器时的端口映射输入，用户名是root而非mysql，密码仍为root
