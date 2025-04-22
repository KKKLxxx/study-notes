# RabbitMQ底层结构与工作原理

RabbitMQ的工作流程如下，生产者先发送消息给Exchange，然后Exchange根据一定的路由规则转发给各个Queue，消费者监听Queue来获取消息

```ascii
                                    ┌───────┐
                               ┌───>│Queue-1│
                ┌──────────┐   │    └───────┘
            ┌──>│Exchange-1│───┤
┌────────┐  │   └──────────┘   │    ┌───────┐
│Producer│──┤                  ├───>│Queue-2│
└────────┘  │   ┌──────────┐   │    └───────┘
            └──>│Exchange-2│───┤
                └──────────┘   │    ┌───────┐
                               └───>│Queue-3│
                                    └───────┘
```

## 一、Exchange

交换机，用于转发消息，但是它不会做存储 ，如果没有Queue绑定到Exchange的话，它会直接丢弃掉Producer发送过来的消息

Exchange主要有以下几个属性

### 1、Binding key与Routing key

Exchange与Queue是通过Binding key绑定起来的，消息的转发规则是由Routing key指定的。比如，Exchange1通过Binding key与Queue1与Queue2绑定，表示Queue1与Queue2在某种情况下可以收到Exchange1转发的消息。Routing key就比如，登录成功，Exchange1会给Queue1发送消息，登陆失败Exchange1会给Queue2发送消息等

### 2、Type

类型，虽然有Routing key来指定分发逻辑，但不同的类型也会影响Exchange分发消息给Queue的分发逻辑，有4种Type，性能由高到低

#### ①fanout

只要Exchange与Queue绑定了，就会向所有Queue转发消息，而不受Routing Key的制约

#### ②direct

需要Routing key与设置的完全匹配才会转发到对应的Queue

#### ③topic

与direct的区别在于，direct是完全匹配，topic是模糊匹配。RabbitMQ自定义了一套匹配规则，在这里我假设生产者发送了一个消息，其中的的Routing key为`wiki.imooc.com`，那么交换器为`topic`类型时候，想要获取到这条消息，可以用`*`号作为通配符，来指定Routing key,分别是`*.*.com`、`*.imooc.*`、`*wiki.imooc.*`；同样也可以使用`#`作为通配符来指定路由键，例如`wiki.#`、`#.com`

在上面的通配符列子中，我们需要掌握这几点：

- 路由键以`.`为分隔符，每一个分隔符的代表一个单词
- 通配符`*`匹配一个单词，通配符`#`可以匹配多个单词

#### ④headers

在绑定Exchange与Queue的时候指定一个键值对，当Exchange在分发消息的时候会先解开消息体里的`headers`数据，然后判断里面是否有所设置的键值对，如果发现匹配成功，才将消息分发到队列中。这种类型在性能上相对来说较差，在实际工作中很少会用到

## 二、Queue

存放消息的队列，消费者通过监听各个Queue来获取消息。可以设置是否持久化，当Consumer不在线时，持久化的Queue会暂存消息，非持久化的Queue会丢弃消息

