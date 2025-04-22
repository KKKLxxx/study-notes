# Docker常用命令

## 一、镜像相关命令

### 1、docker images

**列出所有镜像**

![image-20211121210930740](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111212109806.png)

[-q]选项可以只列出镜像ID

### 2、docker search

**从远程仓库查询镜像**

![image-20211121211136666](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111212111698.png)

### 3、docker pull

**从远程仓库拉取镜像**

![image-20211121211743437](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111212117468.png)

如果不加标签，默认标签为latest。如果要拉取指定版本的镜像，可以先在https://hub.docker.com/中查询对应镜像，获取需要版本的标签，然后加上指定标签

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111212122681.png" alt="image-20211121212227645" style="zoom:80%;" />

可以看到多了一个指定版本的tomcat

### 4、docker rmi

**删除镜像**

![image-20211121212738577](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111212127606.png)

如果一个镜像存在对应的容器，则必须通过[-f]强制删除。或者可以先删除所有对应容器，再删除

**删除镜像后会保留容器**

可以通过指定镜像名+标签删除（不加标签默认latest），也可以指定镜像ID删除

删除多个镜像：`docker rmi -f 镜像名1:标签 镜像名2:标签`

删除所有镜像：`docker rmi -f $(docker images -q)`

## 二、容器相关命令

### 1、docker ps

**列出所有正在运行的容器**

[-a]选项可以查看已经停止的容器

[-q]选项可以只查看容器ID

![image-20211121211048881](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111212110913.png)

### 2、docker run

**创建并运行容器**

![image-20211121215157428](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111212151463.png)

[-it]选项表示交互式运行容器，即运行后进入

[--name]选项表示给容器制定一个名称，可以通过`docker ps`查看到容器的名称

[-d]选项表示守护式运行容器，即运行后不进入容器。但是对于centos镜像，会出现容器立即停止的现象。这是因为容器中没有运行中的前台程序（比如top），容器就会自动停止

[-p]端口映射，主机端口:容器端口。因为我们访问的时候都是通过主机IP+主机端口访问的，这里假设将主机的8888端口与tomcat容器的8080做映射，则在外部访问(主机IP:8888)就可以看到tomcat的主界面（这里的tomcat版本为`tomcat:8.0.52-alpine`，最新版tomcat可能不会显示界面）

![image-20211121230345583](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111212303614.png)

[--privileged]特权模式运行，相当于获取容器的root权限。如果不加该选项，即使在容器内是root身份，也会无法运行部分命令

[-e]传递环境变量

[-v]挂载

```
docker run -itd -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" -v /docker/es/config:/usr/share/elasticsearch/config elasticsearch:6.8.23
```

主要看`-v /docker/es/config:/usr/share/elasticsearch/config`这部分。意思是，让主机目录`/docker/es/config`中的内容挂载到docker容器中`/usr/share/elasticsearch/config`的这个目录上。这样就可以更方面的修改配置文件、复用配置文件

如果在指令后加上`:ro`，`-v /docker/es/config:/usr/share/elasticsearch/config:ro`，意思为docker容器中该目录为只读模式，不可修改，只能在主机中修改。默认情况下为可读写模式，主机与docker中的修改可互相影响

### 3、exit / CTRL + P + Q

**退出容器**

在交互式运行容器中，退出该容器有2种方式

结束并退出：`exit`

不结束退出：`CTRL + P + Q`

### 4、docker attach

**进入正在运行的容器**

在不结束退出容器后如果想要重入容器，可以通过`docker attach 容器名/容器ID`重入

![image-20211121221847782](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111212218811.png)

### 5、docker exec

**不进入容器执行命令**

exec与attach的区别在于，如果要在某个容器内执行容器中的命令，attach只能先进入容器，然后再执行；而exec可以直接在容器外执行容器中的命令

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111212239831.png" alt="image-20211121223921795" style="zoom:80%;" />

可以看到，这里直接在centos容器外执行了centos容器中的`ls`命令并显示结果，而没有进入容器

如果想通过exec进入容器，可以用命令`docker exec -it 容器ID /bin/bash`

### 6、docker start

**启动容器**，命令后接容器名/容器ID

### 7、docker restart

**重启容器**，命令后接容器名/容器ID

### 8、docker stop

**结束容器**，命令后接容器名/容器ID

### 9、docker kill

**强制结束容器**，命令后接容器名/容器ID

### 10、docker rm

**删除已停止的容器**

命令后接容器名/容器ID

[-f]选项可以强制删除运行中的容器

删除多个容器：`docker rm -f 容器ID1/容器名1 容器ID2/容器名2`

删除所有容器：`docker rm -f $(docker ps -qa)`

### 11、docker cp

在主机与docker容器中复制文件/目录，可双向复制

```
docker cp fc1613200882:/usr/share/elasticsearch/config /docker/es/config
```

上面这条指令的意思是，将fc1613200882容器中，`/usr/share/elasticsearch/config`这个目录复制到主机`/docker/es/config`这个目录下

```
docker cp /docker/es/config fc1613200882:/usr/share/elasticsearch/config
```

若要把主机中的内容复制到docker容器中，只用将后面两个路径调换即可，如上