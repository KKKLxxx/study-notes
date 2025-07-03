# Docker部署Kafka

参考官网：https://kafka.apache.org/quickstart

## 一、拉取镜像

```
docker pull apache/kafka:4.0.0
```

## 二、运行镜像

```
docker run -d -p 9092:9092 --name kafka apache/kafka:4.0.0
```

通过-d选项，使镜像在后台运行即可

## 三、创建主题

首先需要进入镜像，找到kafka运行文件的位置

```
docker exec -it kafka /bin/bash
cd /opt/kafka
```

此时可以看到有一个bin目录，执行如下命令创建主题：

```
bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092
```

## 四、测试生产

```
bin/kafka-console-producer.sh --topic quickstart-events --bootstrap-server localhost:9092
>This is my first event
>This is my second event
```

使用 `Ctrl-C` 命令停止生产者客户端

## 五、测试消费

```
bin/kafka-console-consumer.sh --topic quickstart-events --from-beginning --bootstrap-server localhost:9092
This is my first event
This is my second event
```

使用 `Ctrl-C` 命令停止消费产者客户端