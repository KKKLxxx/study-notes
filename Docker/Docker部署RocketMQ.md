# Docker部署RocketMQ

参考官网：https://rocketmq.apache.ac.cn/docs/quickStart/02quickstartWithDocker/

## 一、拉取镜像

```
docker pull apache/rocketmq:5.3.2
```

## 二、为容器创建共享网络

RocketMQ 涉及多个服务，需要多个容器。创建 Docker 网络有助于容器间通信

```
docker network create rocketmq
```

## 三、启动 NameServer

```
# Start NameServer
docker run -d --name rmqnamesrv -p 9876:9876 --network rocketmq apache/rocketmq:5.3.2 sh mqnamesrv

# Verify if NameServer started successfully
docker logs -f rmqnamesrv
```

当我们从 `namesrv.log` 中看到 **The Name Server boot success..** 时，表示 NameServer 已经成功启动（按Ctrl+C退出）

## 四、启动 Broker 和 Proxy

```
# Configure the broker's IP address
echo "brokerIP1=127.0.0.1" > broker.conf
# 注意，若需要将MQ部署到远程服务器，并使本地程序可以访问的话，需要将127.0.0.1改为服务器对应的公网IP
# 如echo "brokerIP1=115.190.8.109" > broker.conf


# Start the Broker and Proxy
docker run -d \
--name rmqbroker \
--network rocketmq \
-p 10912:10912 -p 10911:10911 -p 10909:10909 \
-p 8080:8080 -p 8081:8081 \
-e "NAMESRV_ADDR=rmqnamesrv:9876" \
-v ./broker.conf:/home/rocketmq/rocketmq-5.3.2/conf/broker.conf \
apache/rocketmq:5.3.2 sh mqbroker --enable-proxy \
-c /home/rocketmq/rocketmq-5.3.2/conf/broker.conf


# Verify if Broker started successfully
docker exec -it rmqbroker bash -c "tail -n 10 /home/rocketmq/logs/rocketmqlogs/proxy.log"
```

当我们从 `proxy.log` 中看到 **The broker[brokerName,ip:port] boot success..** 时，表示 Broker 已经成功启动

