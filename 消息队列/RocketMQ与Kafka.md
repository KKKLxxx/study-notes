# RocketMQ与Kafka

## 一、RocketMQ

### 1. 底层结构

![领域模型](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202603311701122.png)

工作流程：生产者生成消息并将其发送到RocketMQ Broker，消息存储在Broker上的主题中，消费者订阅主题以消费消息

#### 1.1 Producer

消息的生产者

#### 1.2 Broker / Topic / Queue

- Broker 是 RocketMQ 中负责接收、持久化和分发消息的节点
  - Broker支持主从部署，实现数据备份与压力分摊
- Topic 是逻辑上的消息分类，本身不存储数据
- 一个 Topic 会被划分为多个 Queue，这些 Queue 分布在不同的 Broker 上，消息最终是存储在 Broker 的 Queue 中的
  - 一个Queue中的消息是有序的，不同Queue中的消息是无序的

这种设计实现了消息的水平扩展、负载均衡以及高可用

#### 1.3 Consumer Group / Consumer

每个消费者不仅要指定它要订阅的Topic，还要指定自己的分组，同一个分组中的消费者会共同消费一个Topic中的消息，不同分组中的消费者则会分别消费

假设两个消费者A、B订阅了同一个Topic，但它们的Group ID不同，那么生产者给这个Topic发送10条消息，A、B会各收到10条不重复的消息；如果它们的Group ID相同，A、B会共享这10条消息，比如A收到5条，B收到另外5条

#### 1.4 NameServer

NameServer是RocketMQ的管理中心，负责Broker的注册发现与管理，并为生产者/消费者提供Broker的路由信息

- Broker需要向NameServer上报自己的地址、名称、ID、Topic以及Queue的配置等元数据
- NameServer会通过心跳机制检查Broker的存活状态，并自动更新路由信息

NameServer也可以集群部署，但NameServer是去中心化的，每个Broker会分别与每个NameServer建立长连接，所以NameServer之间是不用通信的，这样使得架构更加简单

### 2. 消费者的类型

#### 2.1 PushConsumer（RocketMQ默认）

PushConsumer中，消息的拉取、重试、负载均衡等都由MQ控制，消费者只用编写消费逻辑，使用最简单（一次只消费一条消息）

#### 2.2 PullConsumer（Kafka默认）

PullConsumer中，以上功能都需要由消费者自己实现，最灵活但也最复杂（比如可以一次拉取多条消息）

Kafka选择拉模型的理由：消费节奏必须由消费者控制，避免被压垮

#### 2.3 SimpleConsumer

SimpleConsumer介于两者之间，但基本还是一种拉模型

### 3. 如何保证顺序消费

顺序消费有两种实现方式，一种是全局有序，另一种是局部有序

#### 3.1 全局有序

全局有序就是利用一个queue中的消息是有序的的特点，保证一个Topic只有一个queue，并且只有一个生产者和一个消费者，就能够实现顺序消费。但是全局有序的消费速度会比较慢，并且大部分场景也不需要全局有序

#### 3.2 局部有序

局部有序就是引入了消息组的概念，同一个信息组中的消息会被发送到同一个queue中，从而保证了局部有序。可以有多个生产者和多个消费者，但是需要保证一个消息组中消息是同一个生产者产生，同一个消费者消费的

**如何保证一个消息组中消息是同一个生产者产生的？**

可以将消息组ID对生产者实例数量取模，使得一个组中的消息都是由同一个生产者发送的，并且消息要串行发送，不能使用多线程

**如何保证一个消息组中消息是同一个消费者消费的？**

需要消费者以顺序模式进行消费：

```java
@Component
@RocketMQMessageListener(
    topic = "orderlyTopicBoot",
    consumerGroup = "orderlyConsumerGroup",
    consumeMode = ConsumeMode.ORDERLY  // 关键：开启顺序消费模式
)
public class OrderlyConsumer implements RocketMQListener<String> {
    @Override
    public void onMessage(String message) {
        System.out.printf("Consumer Thread: %s, Message: %s %n", Thread.currentThread().getName(), message);
        // 处理业务逻辑
    }
}
```

这样MQ就会开启队列锁，每个消费者会获取一个队列的锁，从而保证这个队列中的消息只会发送给同一个消费者

- 如果消费者宕机，会通过超时机制释放锁
- 消费者实例数超过队列数时，多余的消费者无法消费

#### 3.3 有序消息如果消费失败会怎样

首先会进行重试，但如果超过一定次数仍然失败，失败的消息也不会进入死信队列，并且会将其所在的Queue阻塞。这种情况通常需要人工跳过该消息组的消息，并进行记录，使后面的消息先消费，后续再进行补偿

### 4. 死信队列

死信队列（Dead Letter Queue，DLQ）是用于存放多次消费失败的消息的队列，避免这些消息一直阻塞系统

RocketMQ 内置了死信队列机制，在并发消费下，消息超过最大重试次数会自动进入 DLQ，但有序消息默认不会进入死信队列

Kafka 本身没有死信队列，但在业务上可以通过额外的 Topic 来实现。因为Kafka的设计理念是一个日志系统，而不是一个消息队列，所以它不负责处理消费失败的语义（但Kafka也有重试机制，只是没有死信队列）

### 5. 事务消息

#### 5.1 使用场景

事务消息的作用就是保证本地事务和MQ发送消息的一致性

假设生产者的一个操作中，包括修改数据库与发送消息到MQ两个步骤，无论先执行哪个，都有可能造成数据不一致

- 先改数据库再发消息，有可能在消息发送后产生异常，DB回滚但MQ无法回滚
- 先发消息再改数据库，有可能数据库修改失败，MQ仍无法回滚

所以需要一个方案去保证这两个步骤的一致性

#### 5.2 事务消息的原理

事务消息通过半消息+回查机制保证最终一致性

整个流程分为三个阶段：

- 阶段一：生产者发送半消息，半消息不会投递给消费者
- 阶段二：生产者执行本地事务，根据事务的执行结果
  - 成功：向MQ发送Commit消息，MQ将半消息标记为可投递，并投递给消费者
  - 失败：向MQ发送Rollback消息，MQ将取消半消息的投递
  - 未知：如果因为网络问题或生产者重启，MQ没有收到消息，则半消息处于Unknown状态，进入阶段三
- 阶段三：MQ会在等待一段时间后向生产者发送请求，回查半消息的状态，根据最终状态进行消息的投递或取消投递

#### 5.3 使用方法

Demo：https://rocketmq.apache.ac.cn/docs/featureBehavior/04transactionmessage/

主要就是要注册一个事务回查器。回查器会从半消息中获取业务相关的字段，然后通过一个指定的方法检查本地事务是否执行成功（比如数据库中是否有相关的数据）

#### 5.4 注意事项

- 事务消息只是保证生产者到MQ这段链路的一致性，MQ到消费者仍由消息重试机制实现
- 应避免触发消息回查机制，否则会导致性能下降

#### 5.5 基于XA的强一致事务

XA 是一种分布式事务标准，它的思路是把“数据库 + MQ”当成一个整体事务，并引入协调者来保证所有参与者（也就是数据库和MQ）全部执行成功或者失败

执行流程分为两个阶段（2PC）：

- Prepare：DB执行并持久化，但不提交；MQ准备好消息并持久化，但不发送
- Commit / Rollback：协调者根据DB和MQ在Prepare阶段的执行结果，决定两边一起提交或一起回滚

**协调者如何保证DB提交与MQ发送同时成功或同时失败？**

由于DB与MQ都完成了持久化，并且协调者会记录事务将Commit还是Rollback，所以即使它们宕机，也是可以恢复的。DB与MQ恢复后，可以询问协调者应如何操作

**XA方案的缺陷是什么**

- 对协调者强依赖，如果协调者宕机，会导致所有参与者阻塞
- 通过长事务保证一致性，会导致DB与MQ加锁，性能很低

## 二、Kafka

### 1. 底层结构

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202603311842849.png" alt="img" style="zoom: 80%;" />

Kafka中的结构与RocketMQ几乎相同，主要在以下三点有些许不同：

#### 1.1 Partition

Kafka中的Partition与RocketMQ中的Queue作用相同

但还要注意一点，Kafka是对Partition做副本，而RocketMQ是对Broker做副本

#### 1.2 Consumer

Kafka只有拉模型，理由：消费节奏必须由消费者控制，避免被压垮

#### 1.3 Zookeeper / KRaft

这两个相当于RocketMQ中的NameServer的作用

早期Kafka依赖Zookeeper实现Broker的管理，需要维护Zookeeper和Kafka两套系统，非常复杂

而新版Kafka中，Kafka自己实现了Raft协议，从而不再依赖ZooKeeper，部署更简单，并且有了更高的性能和更强的一致性

## 三、对比

RocketMQ相比Kafka额外支持延迟消费、消息组顺序消费（Kafka只能通过单Paritition）、死信队列、事务消息等功能，更能够满足业务的需要；Kafka通过基于顺序写+零拷贝的高效持久化，以及基于offset的消息确认机制，能够提供更高的吞吐量，适合处理日志等可以批量处理的大流量场景

（基于offset的消息确认机制：Partition中有offset的概念，消费者一次可以消费多条消息，并回复一个offset表示消费进度。而RocketMQ只能一个一个消费，所以RocketMQ的消息确认是对单条消息确认）

- 底层结构：Queue与Partition，NameServer与Zookeeper / KRaft
- 消费者类型：推和拉
- 可靠性：
  - RocketMQ与Kafka默认都是异步刷盘，所以都可能导致数据丢失。两者也都提供了同步刷盘的机制，但如果Kafka开启同步刷盘，那么就完全背离了Kafka高吞吐量的设计理念，所以官方是不推荐开启的。所以在单节点情况下，RocketMQ更能够保证数据的可靠性
  - Kafka主要是通过集群模式的多副本机制来保证数据可靠性，主节点会将数据同步到从节点的缓存当中，所以虽然没有刷盘，但也能够依靠多点备份来保证即使主节点宕机数据也不会丢失。还可以通过配置`ack=all`来让所有从节点同步完成后才向生产者返回ACK
