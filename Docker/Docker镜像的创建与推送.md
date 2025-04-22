# Docker镜像的创建与推送

这里演示了如何创建一个可运行自己应用的CentOS7镜像，并将其推送到远程仓库。在CentOS7容器中需要手动安装MySQL与Redis（因为Docker无法嵌套运行，或者是我不会嵌套运行）

## 1、在Linux服务器中安装Docker

在其他文章中介绍过，略

## 2、获取并运行CentOS

获取即`docker pull centos:centos7.9.2009 `

这里指定了版本号为7.9

运行即`docker run -d --privileged -p 8080:8080 -p 3306:3306 -p 6379:6379 eeb6 /usr/sbin/init`

注意一定要加`--privileged`选项，否则会有一些命令无法运行

指定了3个端口映射，分别是tomcat、mysql、redis的端口

`eeb6`是我这里的镜像ID

`/usr/sbin/init`使得能够在容器内执行`systemctl`命令

由于守护式启动，需要再通过`docker exec -it 容器ID /bin/bash`进入（不可直接-it交互式启动）

## 3、在CentOS容器中安装Docker

同1，略

## 4、在容器中通过Docker安装MySQL

