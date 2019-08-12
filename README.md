# itcast-rabbitmq

来自黑马的rabbitmq的小demo
 
 rabbitmq说是5种模式
其实 消息队列一共就分两类 点对点 和 订阅模式

### 点对点 ：
1.基本消息类型：就是一对一  对应的包simple

其中消息队列的消息确认机制有自动确认和手动确认

自动确认就是消息一旦被接收，消费者自动发送ACK

手动确认就是消息接收后，不会发送ACK，需要手动调用``` channel.basicAck(envelope.getDeliveryTag(), false);```进行手动确认
代码再消息消费完后写 如果中间出现异常 会进入未确认队列

 ```channel.basicConsume(QUEUE_NAME, false, consumer);```来设置消息需要手动确认
 
 设置完手动确认 如果不进行确认  会进入未确认的队列  停掉消费者后未确认的消息会进入ready队列 消费者再启动后再次接收这个消息

2.work消息类型：一条消息被多个消费者任务分发消费 
我们可以使用basicQos方法和prefetchCount = 1设置  谁先执行完谁从队列中拿消息消费 性能高的消费消息多

### 订阅模式

解读：
1、1个生产者，多个消费者

2、每一个消费者都有自己的一个队列

3、生产者没有将消息直接发送到队列，而是发送到了交换机

4、每个队列都要绑定到交换机

5、生产者发送的消息，经过交换机到达队列，实现一个消息被多个消费者获取的目的

X（Exchanges）：交换机一方面：接收生产者发送的消息。另一方面：知道如何处理消息，例如递交给某个特别队列、递交给所有队列、或是将消息丢弃。到底如何操作，取决于Exchange的类型。

订阅模式就是根据交换机类型分的

Exchange类型有以下几种：

         Fanout：广播，将消息交给所有绑定到交换机的队列

         Direct：定向，把消息交给符合指定routing key 的队列

         Topic：通配符，把消息交给符合routing pattern（路由模式） 的队列

Exchange（交换机）只负责转发消息，不具备存储消息的能力，因此如果没有任何队列与Exchange绑定，或者没有符合路由规则的队列，那么消息会丢失！

1.Fanout，也称为广播。
 
 交换机把消息发送给绑定过交换机的所有消费者
 
2.Direct 

根据routing key 来分给匹配的消费者

P：生产者，向Exchange发送消息，发送消息时，会指定一个routing key。

X：Exchange（交换机），接收生产者的消息，然后把消息递交给 与routing key完全匹配的队列

C1：消费者，其所在队列指定了需要routing key 为 error 的消息

C2：消费者，其所在队列指定了需要routing key 为 info、error、warning 的消息

3.topic 

Topic类型的Exchange与Direct相比，都是可以根据RoutingKey把消息路由到不同的队列。只不过Topic类型Exchange可以让队列在绑定Routing key 的时候使用通配符！

### 消息队列的持久化
如何避免消息丢失？

1）  消费者的ACK机制。可以防止消费者丢失消息。

2）  但是，如果在消费者消费之前，MQ就宕机了，消息就没了。


是可以将消息进行持久化呢？

 要将消息持久化，前提是：队列、Exchange都持久化
