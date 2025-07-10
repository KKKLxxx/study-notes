# Docker安装并运行Elasticsearch

## 一、拉取镜像

```
docker pull elasticsearch:8.4.3
```

拉取`8.4.3`版本镜像

## 二、查看镜像

```
docker images
```

![image-20221022232410855](https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20221022232410855.png)

## 三、运行镜像

```
docker run -d -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" --name es-test elasticsearch:8.4.3

docker run -d -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" --name es-test elasticsearch:8.4.3
```

9200是http协议的web客户端RESTful端口

9300是集群节点指点的tcp通讯端口

`discovery.type=single-node` 表示，如果你正在使用单个节点开发，那就要添加这句话避开引导检查

`ES_JAVA_OPTS=-Xms512m -Xmx512m`是为了限制内存，因为如果内存不够会无法启动。内存足够的话可以删除这个配置

## 四、准备工作

通过

```
docker exec -it --user root es-test /bin/bash
```

命令进入容器

`--user root`的作用是需要在容器中安装vim，如果不添加这个选项，会提示没有权限

通过

```
apt update
apt-get install vim -y
```

安装vim（安装不了vi）

## 五、配置跨域

配置跨域主要是让各种连接工具（如Elasticsearch Head）可以连接到ES

在`elasticsearch.yml`文件末尾添加如下内容（可根据需要自行修改）

```
vim /usr/share/elasticsearch/config/elasticsearch.yml

http.cors.enabled: true
http.cors.allow-origin: "*"
```

在ES8版本之后，在配置文件中有很多安全检查的选项，为了方便可以先将安全检查全部关闭（否则SpringBoot无法通过HTTP连接，要设置为HTTPS），即

```
xpack.security.enabled: false
```

`xpack.security`还有很多其他enabled设置项，但只需要把这个总开关改为false即可

## 六、安装ik插件

```
./bin/elasticsearch-plugin install https://get.infini.cloud/elasticsearch/analysis-ik/8.4.3
```

注意要对应版本

## 七、重启容器

`Ctrl+P+Q`退出容器后执行`docker restart es-test`，等待运行成功后即可

