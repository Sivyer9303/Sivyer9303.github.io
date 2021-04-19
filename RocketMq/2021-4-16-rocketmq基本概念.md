## RocketMq基本概念

### RocketMq组成部分

#### 生产者

生产者是指RocketMq中的消息生产的部分，是消息的起点。

###### 支持的发送类型

生产者发送消息支持同步发送、异步发送、顺序发送、单向发送。

同步发送:生产者将消息发送至Broker，必须得到Broker的确认消息才能确认发送成功。

异步发送：生产者发送消息后，通过future来判定是否发送成功。通过在异步方法中添加SendCallback，来监听				消息发送情况，根据是否成功来决定是否回滚等。

顺序发送：RocketMq的顺序消息只能保证单个message queue内的顺序消费，因此，为了保证能够实现顺序消息，在生产者发送消息时，通过选择message queue，将同一批的消息发送到同一个messag queue，即可实现顺序消费。通过在发送消息时，添加MessageQueueSelector 参数，即可自定义messge queue的选择。同时，消费者在消费时，也需要保证顺序消费，通过选择MessageListenerConcurrently监听器即可实现。

单向发送：单向发送是最不靠谱的发送方式，使用时请注意使用场景。单向发送在发送完成以后，不会接受Broker返回来的信息，发送完成以后就不管了。

###### 生产者发送策略

生产者的发送策略决定了当前生产者发送消息到具体哪个queue中，默认为轮询。

轮询：根据queue数量，轮流进行消息发送。

最小延迟：因为每个queue的消息积压情况都有不同，那么在发送时，会记录上一次发送的延迟，通过选择最小延迟的队列来发送消息，在发送失败时，会退化为轮询。

随机：由MessageQueueSelector接口的子类SelectMessageQueueByRandom来实现，简单来说就是生成一个小于队列数量的随机数，然后根据随机数获取对应队列。

hash：由MessageQueueSelector接口的子类SelectMessageQueueByHash来实现，通过对附加参数进行取模，然后对队列数取余，然后根据结果获取对应队列。

机房：根据机房来决定发送到哪个队列，开源项目中并没有提供默认的实现方式，需要自己去实现，大概思路是，通过中间件或者数据库保存生产者和机房的关联关系、队列和机房的关联关系，如果对于消息的延迟接受度稍微高一点，可以考虑把机房和地区的关系也维护起来，将可选的队列扩展到地区级别。



#### 消费者

消费者是指RocketMq中实际进行消息消费的部分，可以是集群，也可以是单机。

###### 消息消费模式

消费者支持两种消费模式，一种是广播模式，一种是集群模式。

广播模式：消息会被所有消费者获取到，不需要消费者返回ack标识。

集群模式：同一个consumerGroupId下的消费者，同一个消息只会被一个消费者获取到，需要返回ack标识给Broker以确认消费成功。

###### 消息拉取模式

消费者支持两种消息拉取模式，pull模式和push模式，本质上push模式是RocketMq对于pull模式的一种封装，仍然是消费者主动去获取，而不是Broker去推送，避免将压力全放到Broker上。

pull模式：通过consumer调用fetch方法（可以阻塞或者不阻塞）获取未消费的消息。

push模式：添加消息监听器即可。

###### 顺序消费

在生产者发送顺序消息后，消费者只需添加顺序消息监听器即可，因为消费者在启动完成后，会根据现有消费者数量和message queue数量，进行message queue和消费者的匹配，每个message queue会对应一个消费者，一个消费者可以对应多个message queue。



#### 主题(topic)

每条消息都必须跟一个topic进行绑定，topic是消息订阅的基本单位。

topic创建分为两种方式，一种是预先创建，一种是自动创建，可以通过修改配置来决定选用哪种方式。

topic保存在namesrv中，生产者和消费者会在本机保存一份缓存的数据。

有一些topic是在正常创建以后自带的，比如重试队列，重试队列的topic为%RETRY% + 消费组名。比如死信队列，死信队列的topic为%DLQ% + 消费组名，两者均与消费组进行绑定。

###### 特殊的topic-TBW102 

当broker开启了自动创建topic，且topic在发送时仍未创建，就会使用TBW102作为topic的父类，让新创建的topic使用TBW102的配置。

###### topic分片

RocketMq中，topic是支持分片的，可以将topic下的queue均匀的分布到每个Broker,例如，现有4个broker，16个queue，那么就可以分给每个broker4个queue。



#### Broker Server

消息中转角色，负责存储消息、转发消息。Broker在RocketMQ系统中负责接收从生产者发送来的消息并存储、同时为消费者的拉取请求作准备。Broker也存储消息相关的元数据，包括消费者组、消费进度偏移和主题和队列消息等。 



#### NameServer

名称服务充当路由消息的提供者。生产者或消费者能够通过名字服务查找各主题相应的Broker IP列表。多个Namesrv实例组成集群，但相互独立，没有信息交换。

当Broker启动时，会将当前Broker所管理的topic信息发送至NameServer。

##### 心跳

NameServer与生产者、消费者、Broker通过心跳进行关联，当Broker出现问题，NameServer中会更新topic和Broker的关系，避免将消息发送至失效的Broker，但NameServer不会通知生产者和消费者，生产者和消费者会通过接口获取topic信息，若发送到某个Broker失败时，会尝试其他Broker或者从NameServer重新拉取topic信息。

每个Broker会与每个NameServer都采用长连接方式保持心跳。

生产者和消费者会随机选取一个NaemServer保持心跳。

同时，生产者会与每个主Broker保持心跳，消费者会与主Broker和从Broker保持心跳。



### 标签（tag）

每个消息都可以绑定若干个tag，tag通过||分隔开。消费者消费topic时，可以通过tag来将topic分类，例如，存在一个交通工具的topic，可以使用tag来进行区分汽车、火车这些。tag不是必须要使用的。

参考资料

https://github.com/apache/rocketmq/blob/master/docs/cn/concept.md
