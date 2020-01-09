---
title: spring cloud stream
author: ziyi
tags:
  - spring cloud stream
  - mq
index_img: /img/mybg.jpg
banner_img: /img/mybg.jpg
categories:
  - spring cloud
  - spring cloud stream
comments: true
---

# spring cloud stream
###### 一个构建消息驱动微服务的框架

### springcloud stream架构

![](https://images2018.cnblogs.com/blog/1202638/201805/1202638-20180528204400011-1996551371.png)

应用程序通过通道(channels): inputs(相当于消费者consumer，它是从队列中接收消息的) 或者 outputs(相当于生产者producer，它是从队列中发送消息的) 来与 Spring Cloud Stream 中binder 交互，binder 负责与消息中间件交互。通道通过指定中间件的Binder实现与外部代理连接。业务开发者不再关注具体消息中间件，只需关注Binder对应用程序提供的抽象概念来使用消息中间件实现业务即可。

通过使用Spring Integration来连接消息代理中间件以实现消息事件驱动。Spring Cloud Stream 为一些供应商的消息中间件产品提供了个性化的自动化配置实现，引用了发布-订阅、消费组、分区的三个核心概念。

> notes: 消息的发布（Publish）和订阅（Subscribe）是事件驱动的经典模式。Spring Cloud Stream 的数据交互也是基于这个思想。生产者把消息通过某个 destination 广播出去。其他的微服务，通过订阅特定 destination 来获取广播的消息来触发业务的进行。这种模式，极大的降低了生产者与消费者之间的耦合。即使有新的应用的引入，也不需要破坏当前系统的整体结构。

## SpringCloud Stream消息驱动优缺点

### 解耦合
消息中间件的架构不同。比方说RabbitMQ和Kafka，RabbitMQ有exchange，kafka有Topic，partitions分区，这些中间件的差异性导致我们实际项目开发给我们造成了一定的困扰，我们如果用了两个消息队列的其中一种，后面的业务需求，要往另外一种消息队列进行迁移，这时候很多东西改，因为它跟我们的系统耦合了。这时候springcloud stream给我们提供了一种解耦合的方式。

### 缺点
集成的mq较少 目前仅有rabbitmq和kafka rocketmq未完全集成

## 实例

### 一个简单的配置 以阿里云控制台rocketmq为例

- pom依赖 使用最新的release 低版本的配置没最新的全

```
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rocketmq</artifactId>
            <version>0.2.2.RELEASE</version>
        </dependency>
```

- properties

> notes: aliyun控制台的rocketmq tcp公网连接配置

```
mq.instance-id=MQ_INST_1734053594575120_BbwSzZdg
spring.cloud.stream.default-binder=rocketmq
spring.cloud.stream.rocketmq.binder.name-server=MQ_INST_1734053594575120_BbwSzZdg.mq-internet-access.mq-internet.aliyuncs.com:80
spring.cloud.stream.rocketmq.binder.access-key=xxx
spring.cloud.stream.rocketmq.binder.secret-key=xxx

#普通消息生产者
#相当于topic 需要先在控制台创建account这个topic
spring.cloud.stream.bindings.output.destination=${mq.instance-id}%account

#消费者
spring.cloud.stream.bindings.input.destination=${mq.instance-id}%account
#分组 需要现在控制台创建GID-ACCOUNT这个group
spring.cloud.stream.bindings.input.group=${mq.instance-id}%GID-ACCOUNT
```

- 消费者

```
@EnableBinding(AccountInput.class)
@Slf4j
/**
 * 实例消费者
 * 可在service上使用@EnableBinding绑定一个input channels接口
 * 再在方法上使用@StreamListener监听一个input channel即可接受消息
 * 至于input channel订阅的topic tag在properties文件配置
 */
public class AccountConsumer {

    /**
     * 消费者消费消息
     */
    @StreamListener(AccountInput.INPUT)
    public void receive(Message<String> message) {
        log.info("接收到消费信息");
        log.info(message.getPayload());
        log.info(message.getHeaders().toString());
        log.info("【SpringCloud Stream】StreamListener.stream={}", JSON.toJSONString(message));
    }
}
```

- 生产者

```
@EnableBinding(AccountOutput.class)
public class AccountProducer {

    @Autowired
    private AccountOutput accountOutput;

    /**
     * 生产者生产消息
     */
    public boolean send(Object obj) {
        return accountOutput.output().send(buildMessage(obj));
    }

    private Message buildMessage(Object obj) {
        return MessageBuilder.withPayload(JSON.toJSONString(obj))
                .build();
    }
}
```

- 生产者 消费者channels就不列出惹 使用@Input或@Output绑定channels即可

## 概念解析

### binder
通过定义binder作为中间层，实现了应用程序与消息中间件(Middleware)细节之间的隔离。通过向应用程序暴露统一的通道，使得应用程序不需要再考虑各种不同的消息中间件的实现。当需要更换其他消息中间件产品时，需要做的就是修改binder的配置而不需要修改任何应用逻辑。甚至可以任意的改变中间件的类型而不需要修改一行代码。

为构造Binding提供了 2 个方法，分别是 bindConsumer 和 bindProducer ，它们分别用于构造生产者和消费者。Binder使Spring Cloud Stream应用程序可以灵活地连接到中间件。

### binding
Binding 是连接应用程序跟消息中间件的桥梁，用于消息的消费和生产，由binder创建。

### @Input

#### 实例

```
public interface Consumer {
    @Input("input")
    SubscribableChannel input();
}
```

#### 作用

- 用于接收消息
- 为每个binding生成channel实例
- 指定channel名称
- 在spring容器中生成一个名为input，类型为SubscribableChannel的bean
- 在spring容器中生成一个类，实现Consumer接口。

### @output

类似@input 用于生产消息

### @InboundChannelAdapter

让定义的方法生产消息

```
@Bean
@InboundChannelAdapter(value = Source.OUTPUT,
        poller = @Poller(fixedDelay = "10", maxMessagesPerPoll = "1"))
public MessageSource<String> test() {
    return () -> new GenericMessage<>("Hello Spring Cloud Stream");
}
```

> 用 InboundChannelAdapter 注解的方法上即使有参数也没用。即下面test方法不要有参数。

- fixedDelay：多少毫秒发送1次
- maxMessagesPerPoll：一次发送几条消息。

### @ServiceActivator

表示方法能够处理消息或消息有效内容，监听input消息，用方法体的代码处理，然后输出到output中。

#### 例如消息异常处理

```
#这里set input channel订阅的destination及所在的group
@ServiceActivator(inputChannel = "input-destination.input-group.errors", outputChannel = "xxx")
    public Object handleError(ErrorMessage message) {
        Throwable throwable = message.getPayload();
        log.error("截获异常message:{}", throwable.getMessage());
        Message<?> originalMessage = message.getOriginalMessage();
        if (null != originalMessage) {
            log.info("原始消息体 = {}", new String((byte[]) originalMessage.getPayload()));
        }
        return "xxx";
    }
```

### @Transformer 

和 ServiceActivator 类似，表示方法能够转换消息，消息头，或消息有效内容

### Processor 消息中转站

spring-cloud-stream给我们提供了一个Processor接口，用于将消息处理后再发送出去，相当于一个消息中转站

从源码可看出Processor扩展了Source(生产者channels)和Sink(消费者Channels)。

```
public interface Processor extends Source, Sink {
}
```

#### 仍是rocketmq为例（rocketmq里只要是消息接收者都需要group）

- 生产者 properties

`spring.cloud.stream.bindings.output.destination=${mq.instance-id}%account`

- 中转者 properties

```
spring.cloud.stream.bindings.output.destination=${mq.instance-id}%account-trans
spring.cloud.stream.bindings.input.destination=${mq.instance-id}%account
```
```
@Slf4j
@EnableBinding(Processor.class)
public class TransFormService {
    @ServiceActivator(inputChannel = Processor.INPUT, outputChannel = Processor.OUTPUT)
    public Object transform(Object payload){
        log.info("消息中转站：{}", payload);
        return payload;
    }
}
```

- 接收者

`spring.cloud.stream.bindings.input.destination=${mq.instance-id}%account-trans`


### Consumer Groups

防止同一个事件被重复消费，只要把这些应用放置于同一个 “group” 中，就能够保证消息只会被一个group其中的一个应用消费一次。

> rocketmq里消费者是一定有分组的 如`spring.cloud.stream.bindings.input.group=${mq.instance-id}%GID-ACCOUNT`

### 消息分区 partition

> Spring Cloud Stream对给定应用的多个实例之间分隔数据予以支持。一个或者多个生产者应用实例给多个消费者应用实例发送消息并确保相同特征的数据被同一消费者实例处理。

也就是说Spring Cloud Stream可以根据分区分配算法将相同特征的消息（比如id  % 2 = 1）分配给同一个实例消费。

> notes: Spring Cloud Stream对分割的进程实例实现进行了抽象。使得Spring Cloud Stream 为不具备分区功能的消息中间件（RabbitMQ）也增加了分区功能扩展。

- 消费者

```
spring.cloud.stream.binginds.input.consumer.partitioned=true
#分区数量
spring.cloud.stream.instance-count=2
```
- 生产者
```
#分区的主键，例如payload.id只是一个对象的id用于做为Key
spring.cloud.stream.bindings.output.producer.partitionKeyExpression=payload.id
#Key和分区数量进行取模去分配消息，这里分区数量配置为2
spring.cloud.stream.bindings.output.producer.partitionCount=2
```

### 返回确认ack 
使用@SendTo注解 或使用消息中转的方式亦可
```
@StreamListener("input-channel")
@SendTo("output-channel")
public Object receiveFromInput(Object payload){
        return "ack";
    }
```

### stream 消息过滤

- 使用condition 使用所有mq

```
#producer
MessageBuilder.setHeader("service-header", "mycondition").build();
#consumer
@StreamListener(value = Sink.INPUT, condition = "headers['service-header']=='mycondition'")
```

- 使用tag 只适用rocketmq

```
#producer
MessageBuilder.setHeader(RocketMQHeaders.TAGS, "deduction").build();
MessageBuilder.setHeader(RocketMQHeaders.TAGS, "withdraw").build();
#consumer properties
spring.cloud.stream.rocketmq.bindings.input.consumer.tags= deduction||withdraw
```

- 使用sql过滤 rocketmq

> 首先要确认开启了rocketmq的sql过滤

```
#producer
MessageBuilder.setHeader("amount", "100").build();
#consumer properties
spring.cloud.stream.rocketmq.bindings.input.consumer.sql="amount>=100"
```

###

官网: https://spring.io/projects/spring-cloud-stream