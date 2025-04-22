# Kafka底层结构与工作原理

Kafka的工作流程如下，生产者指定消息的Topic并发送，消费者订阅Topic获取消息。

```ascii
            ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
            |Topic              |
            │                   │
            |   ┌───────────┐   |    ┌──────────┐
            │┌─>│Partition-1│──┐│┌──>│Consumer-1│
            |│  └───────────┘  │|│   └──────────┘
┌────────┐  ││  ┌───────────┐  │││   ┌──────────┐
│Producer│───┼─>│Partition-2│──┼─┼──>│Consumer-2│
└────────┘  ││  └───────────┘  │││   └──────────┘
            |│  ┌───────────┐  │|│   ┌──────────┐
            │└─>│Partition-3│──┘│└──>│Consumer-3│
            |   └───────────┘   |    └──────────┘
            └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
```

## Topic

Topic就是消息的主题，没有太多可以介绍的

## Partition

Partition是Kafka的核心内容，一个Topic可包含多个Partition，所以使得Kafka有较高的并发性。Kafka只保证在一个Partition内部，消息是有序的，但是，存在多个Partition的情况下，Producer发送的3个消息会依次发送到Partition-1、Partition-2和Partition-3，Consumer从3个Partition接收的消息并不一定是Producer发送的顺序。因此，多个Partition并不能保证完全按Producer发送的顺序。这一点在使用Kafka作为消息服务器时要特别注意，对发送顺序有严格要求的Topic只能有一个Partition。每个Topic的分区数量可通过命令行修改

## Group

每个消费者不仅要指定它要订阅的Topic，还要指定自己的分组。假设两个消费者A、B订阅了同一个Topic，但它们的Group ID不同，那么生产者给这个Topic发送10条消息，A、B会各收到10条不重复的消息；如果它们的Group ID相同，A、B会共享这10条消息，比如A收到5条，B收到另外5条