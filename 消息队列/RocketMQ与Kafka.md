# RocketMQ与Kafka

## 一、RocketMQ

### 1. 底层结构

![领域模型](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202603311701122.png)

工作流程：生产者生成消息并将其发送到RocketMQ Broker，消息存储在Broker上的主题中，消费者订阅主题以消费消息

#### 1.1 Producer

消息的生产者

#### 1.2 Broker / Topic / Queue

- Broker 是 RocketMQ 的消息存储和服务节点，负责消息的接收、持久化和分发
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

#### 3.1 生产者顺序生产

也就是保证只有一个生产者，并且消息是串行发送到MQ（不能用线程池并行发送）

#### 3.2 MQ单Queue / 引入消息组

##### 3.2.1 单Queue

由于一个Queue内的消息天然保证有序，所以如果一个Topic只有一个Queue，那么在MQ端就可以保证有序

单Queue能够实现全局有序，但会使并发性严重降低，通常是不可接受的，并且大部分业务场景也不需要全局有序

##### 3.2.2 引入消息组

在发送消息时，可以给每个消息指定一个消息组（Message Group），同一组的消息会被路由到同一个Queue，从而保证局部有序，这也是大部分场景需要的有序性

消息组的名称可以设置为订单ID、用户ID等

#### 3.3 消费者顺序消费

要使用默认的推消费者去消费消息，保证顺序消费

#### 3.4 如果一条消息消费失败了（且重试后也失败），那么后面的消息会怎样

为了保证有序，失败的消息不会进入死信队列，而是会将其所在的Queue阻塞。这种情况通常需要人工跳过该消息组的消息，并进行记录，使后面的消息先消费，后续再进行补偿

### 4. 死信队列

死信队列（Dead Letter Queue，DLQ）是用于存放多次消费失败的消息的队列，避免这些消息一直阻塞系统

RocketMQ 内置了死信队列机制，在并发消费下，消息超过最大重试次数会自动进入 DLQ，但顺序消费默认不会进入死信队列

Kafka 本身没有死信队列，但在业务上可以通过额外的 Topic 来实现。因为Kafka的设计理念是一个日志系统，而不是一个消息队列，所以它不负责处理消费失败的语义（但Kafka也有重试机制，只是没有死信队列）

### 5. 事务消息

#### 5.1 使用场景

假设生产者的一个操作中，包括修改数据库与发送消息到MQ两个步骤，无论先执行哪个，都有可能造成数据不一致。先改数据库，有可能消息没有发送；先发消息，有可能数据库修改失败

所以需要一个方案去保证这两个步骤的一致性

#### 5.2 基于XA的强一致事务

XA 是一种分布式事务标准，它的思路是把“数据库 + MQ”当成一个整体事务，并引入协调者来保证所有参与者（也就是数据库和MQ）全部执行成功或者失败

执行流程分为两个阶段（2PC）：

- Prepare：DB执行并持久化，但不提交；MQ准备好消息并持久化，但不发送
- Commit / Rollback：协调者根据DB和MQ在Prepare阶段的执行结果，决定两边一起提交或一起回滚

**协调者如何保证DB提交与MQ发送同时成功或同时失败？**

由于DB与MQ都完成了持久化，并且协调者会记录事务将Commit还是Rollback，所以即使它们宕机，也是可以恢复的。DB与MQ恢复后，可以询问协调者应如何操作

**XA方案的缺陷是什么**

- 对协调者强依赖，如果协调者宕机，会导致所有参与者阻塞
- 通过长事务保证一致性，会导致DB与MQ加锁，性能很低

#### 5.3 事务消息的原理

事务消息保证的是最终一致性，它通过半消息+回查的方式避免加锁，从而提高了处理事务的性能

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202604010857305.svg" alt="v-11" style="zoom: 80%;" />

整个流程分为三个阶段：

- 阶段一：生产者发送半消息，MQ回复ACK。此时消息还不会投递给消费者
- 阶段二：生产者执行本地事务，根据事务的执行结果
  - 成功：向MQ发送Commit消息，MQ将半消息标记为可投递，并投递给消费者
  - 失败：向MQ发送Rollback消息，MQ将取消半消息的投递
  - 未知：如果因为网络问题或生产者重启，MQ没有收到消息，则半消息处于Unknown状态。MQ会在等待一段时间后向生产者发送请求，回查半消息的状态，进入阶段三
- 阶段三：MQ向生产者回查，根据最终状态进行消息的投递或取消投递

#### 5.4 注意事项

- 事务消息只是保证生产者到MQ这段链路的一致性，MQ到消费者仍由消息重试机制实现
- 应避免触发消息回查机制，否则会导致性能下降

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

RocketMQ相比Kafka额外支持延迟消费、消息组顺序消费（Kafka只能通过单Paritition）、事务消息、死信队列等功能，更能够满足业务的需要；Kafka通过基于顺序写+零拷贝的高效持久化，以及基于offset的消息确认机制，能够提供更高的吞吐量，适合处理日志等可以批量处理的大流量场景

（基于offset的消息确认机制：Partition中有offset的概念，消费者一次可以批量消费多条消息，并回复一个offset表示消费进度。而RocketMQ只能一个一个消费，所以RocketMQ的消息确认是对单条消息确认）

- 底层结构：Queue与Partition，NameServer与Zookeeper / KRaft
- 消费者类型：推和拉