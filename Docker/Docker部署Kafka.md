# Docker部署Kafka

参考官网：https://kafka.apache.org/quickstart

## 一、拉取镜像

```
docker pull bitnami/kafka:4.0.0
```

官方镜像应为apache/kafka，但实际使用发现apache/kafka难以配置和运行，遂改用bitnami/kafka

经搜索，bitnami应是一个比apache版本更加便于部署运行的版本，虽然难以深度优化，但更适合初学者

## 二、运行镜像

```
docker run \
  --name kafka \
  -p 9092:9092 \
  -p 9093:9093 \
  -e KAFKA_CFG_PROCESS_ROLES=broker,controller \
  -e KAFKA_CFG_NODE_ID=1 \
  -e KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=1@115.190.8.109:9093 \
  -e KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093 \
  -e KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://115.190.8.109:9092 \
  -e KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT \
  -e KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER \
  -e ALLOW_PLAINTEXT_LISTENER=yes \
  bitnami/kafka:4.0.0
```

通过-d选项，使镜像在后台运行即可

`115.190.8.109`为远程服务器的IP，需要设置为公网IP，使得SpringBoot程序能够正常连接到kafka

### 关键参数详解

1. **`KAFKA_CFG_PROCESS_ROLES=broker,controller`**
   - 在 KRaft 模式中定义节点角色：
     - `broker`：处理生产/消费请求
     - `controller`：管理集群元数据和领导选举
   - 单节点部署时可同时担任两种角色
2. **`KAFKA_CFG_NODE_ID=1`**
   - 节点唯一标识符（整数）
   - 集群中每个节点必须有唯一的ID
   - 单节点设置为1即可
3. **`KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=1@115.190.8.109:9093`**
   - 定义控制器仲裁节点列表
   - 格式：`节点ID@主机:端口`
   - 示例表示：ID为1的控制器位于115.190.8.109:9093
   - 多节点格式：`1@host1:9093,2@host2:9093,3@host3:9093`
4. **`KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093`**
   - 配置两个监听器：
     - `PLAINTEXT://:9092`：客户端通信端口（9092），使用明文协议
     - `CONTROLLER://:9093`：控制器内部通信端口（9093）
   - `://:` 表示绑定到所有网络接口（0.0.0.0）
5. **`KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://115.190.8.109:9092`**
   - 告知客户端实际连接地址
   - 必须设置为客户端可访问的地址（此处是公网IP）
   - 客户端连接时返回此地址
6. **`KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT`**
   - 定义监听器名称到安全协议的映射：
     - `CONTROLLER` → `PLAINTEXT`
     - `PLAINTEXT` → `PLAINTEXT`
   - 生产环境应使用`SSL`替代`PLAINTEXT`
7. **`KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER`**
   - 指定用于控制器通信的监听器名称
   - 必须与`LISTENERS`中定义的控制器监听器名称一致
8. **`ALLOW_PLAINTEXT_LISTENER=yes`**
   - Bitnami 镜像特有参数
   - 允许使用非加密的明文连接
   - 生产环境应设置为`no`并使用SSL加密

## 三、创建主题

首先需要进入镜像，找到kafka运行文件的位置

```
docker exec -it kafka /bin/bash
cd /opt/bitnami/kafka
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