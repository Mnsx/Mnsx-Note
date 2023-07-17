# 客户端开发向导

## 连接RabbitMQ

如果在使用Channel的时候其已经处于关闭状态，那么程序将会抛出`ShutdownSignalException`异常，或者通过捕获IOException或者SocketException来防止Connection意外关闭

## 使用交换机和队列

```java
Channel channel = connection.createChannel();
channel.exchangeDeclare("ex1", "direct", true);
String queueName = channel.queueDeclare().getQueue();
channel.queueBind(queueName, "ex1", "r1");
```

这里创建了**一个持久化的，非自动删除的，绑定类型为direct的交换机**

同时创建了**一个非持久化、排他的、自动删除的队列**

**同一个Connection的不同Channel可公用，并且会在应用连接断开时自动删除**

```java
Channel channel = connection.createChannel();
channel.exchangeDeclare("ex1", "direct", true);
channel.queueDeclare("queue1", true, false, false, null);
channel.queueBind("queue1", "ex1", "r1");
```

这个队列是**持久化的、非排他的、非自动删除的**

### exchangeDeclare方法详解

```java
Exchange.DeclareOk exchangeDeclare(String exchange,
                                   String type,
                                   boolean durable,
                                   boolean autoDelete,
                                   boolean internal,
                                   Map<String, Object> arguments) throws IOException;
```

返回值是**Exchange.DeclareOK**，用来**标识成功声明了一个交换机**

参数介绍——

* exchange：交换机名称

* type：交换机类型

* durable：设置是否持久化

  > 持久化可以将交换机存盘，**在服务器重启的时候**不会丢失相关信息

* autoDelete：设置是否自动删除

  > 自动删除的前提是**至少有一个队列或者交换机**于这个交换机绑定，如果这些队列和交换机取消绑定后，**自动删除这个交换机**

* internal：设置是否是内置的

  > 内置的交换机，客户端程序**无法直接发消息到这个交换机**，只能通过**交换机路由到交换机**这种方式

* argument：其他一些结构化参数

```java
void exchangeDeclareNoWait(String exchange,
                           String type,
                           boolean durable,
                           boolean autoDelete,
                           boolean internal,
                           Map<String, Object> arguments) throws IOException;
```

这个exchangeDeclareNoWait多设置一个nowait参数，意思是**不需要服务器返回，这个方法返回的是void**

因为没有返回值，如果用户声明完一个交换机之后（**实际服务器还并没有完成交换机的创建**），那么必然会发生异常

```java
Exchange.DeleteOk exchangeDelete(String exchange, boolean ifUnused) throws IOException;

void exchangeDeleteNoWait(String exchange, boolean ifUnused) throws IOException;
```

参数ifUnused表示**是否在交换机没有被使用时删除**

```java
Exchange.DeclareOk exchangeDeclarePassive(String name) throws IOException;
```

### queueDeclare方法详解

```java
Queue.DeclareOk queueDeclare() throws IOException;

Queue.DeclareOk queueDeclare(String queue, boolean durable, boolean exclusive, boolean autoDelete,
                             Map<String, Object> arguments) throws IOException;
```

不带参数的queueDeclare方法**默认**创建一个**由RabbitMQ命名的、排他的、自动删除的、非持久化的队列**

参数介绍——

* queue：队列名称

* durable：设置是否持久化

* exclusive：设置是否排他

  > 如果一个队列被声明为排他队列，该队列**仅对首次声明它的连接可见**，并在**连接断开时自动删除**
  >
  > 首次指如果一个连接已经声明了一个排他队列，**其他连接不允许建立同名的排他队列**，这与普通队列
  >
  > 即使该队列是持久化的，**一旦连接关闭或者客户端退出，该排他队列都会被自动删除**

* autoDelete：设置是否自动删除
* arguments：设置队列其他参数

```java
void queueDeclareNoWait(String queue, boolean durable, boolean exclusive, boolean autoDelete,
                        Map<String, Object> arguments) throws IOException;
```

```java
Queue.DeclareOk queueDeclarePassive(String queue) throws IOException;
```

```java
Queue.DeleteOk queueDelete(String queue, boolean ifUnused, boolean ifEmpty) throws IOException;

void queueDeleteNoWait(String queue, boolean ifUnused, boolean ifEmpty) throws IOException;
```

**queuePurge可以清空队列中的所有信息但是不删除这个队列**

```java
Queue.PurgeOk queuePurge(String queue) throws IOException;
```

### queueBind方法详解

```java
Queue.BindOk queueBind(String queue, String exchange, String routingKey, Map<String, Object> arguments) throws IOException;

void queueBindNoWait(String queue, String exchange, String routingKey, Map<String, Object> arguments) throws IOException;
```

参数介绍——

* queue：队列名称
* exchange：交换机的名称
* routingKey：用来绑定队列和交换机的路由键
* argument：定义绑定的一些参数

```java
Queue.UnbindOk queueUnbind(String queue, String exchange, String routingKey, Map<String, Object> arguments) throws IOException;
```

### exchangeBind方法详解

```java
Exchange.BindOk exchangeBind(String destination, String source, String routingKey, Map<String, Object> arguments) throws IOException;

void exchangeBindNoWait(String destination, String source, String routingKey, Map<String, Object> arguments) throws IOException;
```

destination——>source，将两个交换机绑定

## 何时创建

RabbitMQ的消息存储队列中，交换机的使用**并不真正的消耗服务器的性能**，而**队列会**

衡量RabbitMQ**当前的QPS只需要看队列即可**

## 发送消息

```java
byte[] bytes = "Hello world!".getBytes(StandardCharsets.UTF_8);
channel.basicPublish("ex1", "r1", null, bytes);
```

```java
channel.basicPublish("ex1", "r1", false, MessageProperties.PERSISTENT_TEXT_PLAIN, bytes);
```

```java
channel.basicPublish("ex1", "r1", new AMQP.BasicProperties().builder()
                     .contentType("text/plain")
                     .deliveryMode(2)
                     .priority(1)
                     .userId("hidden")
                     .build(), bytes);
```

```java
Map<String, Object> headers = new HashMap<>();
        headers.put("location", "here");
        headers.put("time", "now");
        channel.basicPublish("ex1", "r1", new AMQP.BasicProperties().builder()
                .headers(headers).build(), bytes);
```

```java
channel.basicPublish("ex1", "r1", new AMQP.BasicProperties().builder().expiration("60000").build(), bytes );
```

参数介绍——

* exchange：交换机的名称

* routingKey：路由键

* props：消息的基本属性集

  > contentType\contentEncoding\headers\deliveryMode\priority\correlationId\replyTo\expiration\messageId\timestamp\type\userId\appId\clusterId

* body：消息体
* mandatory和immediate

## 消费消息

RabbitMQ的消费模式分两种：推（push）模式和拉（pull）模式

### 推模式

接收消息一般通过**实现Consumer接口或者集成DefualtConsumer类**来实现

```java
boolean autoAck = false;
channel.basicQos(64);
channel.basicConsume("queue1", autoAck, "ct1", 
                     new DefaultConsumer(channel) {
                         @Override
                         public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                             String routingKey = envelope.getRoutingKey();
                             String contentType = properties.getContentType();
                             long deliveryTag = envelope.getDeliveryTag();
                             channel.basicAck(deliveryTag, false);
                         }
                     });
```

参数介绍——

* queue：队列名称
* autoAck：设置是否自动确认
* comsumerTag：消费者标签，**用来区分多个消费者**
* noLocal：设置为true则表示**不能将同一个Connection中生产者发送的信息传送给这个Connection中的消费者**
* exclusive：设置是否排他
* arguments：设置消费者的其他参数
* callback：设置消费者的回调函数，用来处理RabbitMQ推送过来的消息

### 拉模式

通过`channel.basicGet`可以单条获取消息

```java
GetResponse basicGet(String queue, boolean autoAck) throws IOException;
```

返回值是GetResponse

![image-20221023124632387](D:\WorkSpace\Note\Picture\image-20221023124632387.png)

如果想要从队列中获取单条消息，那么必须通过Basic.Get方法来获取，**但是不能通过循环批量获取，这样严重损耗RabbitMQ性能**

## 消费端的确认与拒绝

当autoAck参数设置为false，对于RabbitMQ服务端而言，队列中的消息分成了两个部分，**一部分是等待投递给消费者的消息**，**一部分是已经投递给消费者，但是没有收到消费者确认信号的消息**

如果RabbitMQ**一直收不到消费者的确认信号**，并且**消费此消息的消费者已经断开连接**，则RabbitMQ会**安排此消息重新进入队列**

```java
void basicReject(long deliveryTag, boolean requeue) throws IOException;
```

参数介绍——

deliveryTag：消息编号64位长整型

requeue：设置是否重新入队

```java
void basicNack(long deliveryTag, boolean multiple, boolean requeue)
    throws IOException;
```

批量拒绝消息

# RabbitMQ进阶

## 消息何去何从

### mandatory参数

当mandatory参数设置为true时，**交换机无法根据自身的类型和路由键找到一个符合条件的队列**，那么RabbitMQ就会调用**Basic.Return**命令**将消息返回给生产者**

```java
public class MandatoryDemo {
    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setUsername("root");
        connectionFactory.setPassword("123123");
        connectionFactory.setVirtualHost("center");
        connectionFactory.setHost("mnsx.top");
        connectionFactory.setPort(5672);
        Connection connection = connectionFactory.newConnection();
        Channel channel = connection.createChannel();

        channel.exchangeDeclare("ex1", "direct", false, false, false, null);
        channel.basicPublish("ex1", "", true, MessageProperties.PERSISTENT_TEXT_PLAIN, "mandatory test".getBytes(StandardCharsets.UTF_8));
        channel.addReturnListener(new ReturnListener() {
            @Override
            public void handleReturn(int replyCode, String replyText, String exchange, String routingKey, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String message = new String(body);
                System.out.println("Basic.Return返回的结果是：" + message);
            }
        });
    }
}
```

### immediate参数

当immediate参数被设为true时，如果**交换机在将消息路由到队列时**发现队列上**不存在任何消费者**，那么这条消息将不会存入队列中

RabbitMQ3.0开始去掉了immediate参数的支持，对此RabbitMQ官方解释，**immediate参数会影响镜像队列的性能，增加代码复杂性，建议采用TTL和DLX的方法替代**

### 备份交换机

备份交换机也被称为AE，它会将**未被路由的消息存储在RabbitMQ中**，再在需要的时候去处理这些信息

可以通过声明交换机的时候，添加`alternate-excahgne`参数来实现，也可以通过策略（跳过）的方式实现，如果两者同时使用，**前者优先级更高**

```java
public class AlternateExchange {
    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setUsername("root");
        connectionFactory.setPassword("123123");
        connectionFactory.setVirtualHost("center");
        connectionFactory.setHost("mnsx.top");
        connectionFactory.setPort(5672);
        Connection connection = connectionFactory.newConnection();
        Channel channel = connection.createChannel();

        Map<String, Object> arg = new HashMap<>();
        arg.put("alternate-exchange", "ae1");
        channel.exchangeDeclare("ex1", "direct", true, false, arg);
        channel.exchangeDeclare("ae1", "fanout", false, false, null);
        channel.queueDeclare("queue1", true, false, false, null);
        channel.queueBind("queue1", "ex1", "key1");
        channel.queueDeclare("aeQueue", true, false, false, null);
        channel.queueBind("aeQueue", "ae1", "");

        channel.basicPublish("ex1", "key2", MessageProperties.PERSISTENT_TEXT_PLAIN, "hello world".getBytes(StandardCharsets.UTF_8));
    }
}
```

* 如果设置的备份交换机不存在，客户端和RabbitMQ服务端都不会由异常出现，此时消息会丢失
* 如果备份交换机没有绑定任何队列，客户端和RabbitMQ服务端都不会有异常出现，此时消息会丢失

* 如果备份交换机没有任何匹配的队列，客户端和RabbitMQ服务端都不会有异常小狐仙此时消息会丢失
* 如果备份交换机和mandatory一起使用，那么mandatory参数无效

## 过期时间

### 设置消息的TTL

两种方法可以设置消息的TTL

* 通过队列属性设置，队列中所有的消息都有相同的过期时间
* 对消息本身进行单独设置，每条消息的TTL可以不同

如果两个都设置，那么以两个中数值最小的为准

通过队列属性设置消息TTL的方法在queueuDeclare中，通过x-message-ttl参数实现，单位是ms

```java
public class TTLQueueDemo {
    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = MqInit.getChannel();
        Map<String, Object> arg = new HashMap<>();
        arg.put("x-message-ttl", 6000);
        channel.queueDeclare("queue1", false, false, false, arg);
    }
}
```

消息过期后就会成为死信队列

针对每条消息设置TTL的方法是在basicPublish中加入expiration属性参数，单位ms

```java
public class TtlMessageDemo {
    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = MqInit.getChannel();
        AMQP.BasicProperties build = new AMQP.BasicProperties.Builder().deliveryMode(2).expiration("6000").build();
//        builder.deliveryMode(2); // 持久化消息
//        builder.expiration("6000");
//        AMQP.BasicProperties build = builder.build();
        channel.basicPublish("ex1", "key1", false, build, "ttl".getBytes(StandardCharsets.UTF_8));
    }
}
```

第一种方法如果过期，那么队列中的消息将会被删除，第二种方法如果过期，不会立即删除，会在被消费者消费之前才进行删除

### 设置队列的TTL

通过queueDeclare方法中x-expires参数可以空置队列被自动删除前未使用状态的时间

用于表示过期时间的x-expires参数以ms为单位，不能设置为0

```java
public class QueueTtlDemo {
    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = MqInit.getChannel();
        Map<String, Object> arg = new HashMap<>();
        arg.put("x-expires", 1800000);
        channel.queueDeclare("queue1", false, false, false, arg);
    }
}
```

## 死信队列

DLX，Dead-Letter-Exchange，可以称之为私信交换机

当一个消息在一个队列中变成死信之后，他能被重新发送到另一个交换机中，这个交换机就是DLX，绑定DLX的队列就是死信队列

* 消息被拒绝，并设置了requeue参数为false
* 消息过期
* 队列长度达到最大值

通过在queueDeclare方法中设置x-dead-letter-exchange参数来为这个队列添加DLX

```java
public class DlxDemo {
    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = MqInit.getChannel();
        channel.exchangeDeclare("de", "direct");
        HashMap<String, Object> arg = new HashMap<>();
        arg.put("x-dead-letter-exchange", "dx");
        channel.queueDeclare("queue1", false, false, false, arg);
    }
}
```

也可以为这个DLX指定路由键，如果没有特殊指定，则使用原队列的路由键

```java
args.put("x-dead-letter-routing-key", "drk")
```

```java
public class DlxTtlDemo {
    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = MqInit.getChannel();
        channel.exchangeDeclare("dlx", "direct", true);
        channel.exchangeDeclare("ex1", "fanout", true);
        HashMap<String, Object> arg = new HashMap<>();
        arg.put("x-message-ttl", 1000);
        arg.put("x-dead-letter-exchange", "dlx");
        arg.put("x-dead-letter-routing-key", "key1");
        channel.queueDeclare("queue1", true, false, false, arg);
        channel.queueDeclare("dlq", true, false, false, null);
        channel.queueBind("queue1", "ex1", "");
        channel.queueBind("dlq", "dlx", "key1");
        channel.basicPublish("ex1", "xx", MessageProperties.PERSISTENT_TEXT_PLAIN, "test".getBytes());
    }
}
```

## 优先级队列

具有高优先级的队列具有高的优先权，优先级高的消息具备优先被消费的特权

可以通过**设置x-max-pririty参数来实现**

```java
public class PriorityDemo {
    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = MqInit.getChannel();
        HashMap<String, Object> arg = new HashMap<>();
        arg.put("x-max-priority", 10);
        channel.queueDeclare("queue1", true, false, false, arg);
    }
}
```

还可以通过参数设置消息优先级

```java
public class PriorityDemo {
    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = MqInit.getChannel();
        AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder().priority(5).build();
        channel.basicPublish("ex1", "key1", properties, "xxx".getBytes(StandardCharsets.UTF_8));
    }
}
```

## 持久化

* 交换机持久化

  声明交换机时，将durable参数设置为true

* 队列持久化

  声明队列将durable参数设置为true

* 消息持久化

  通过将消息的投递模式改为2即可实现消息持久化，PERSISITENT_TEXT_PLAIN实际上时封装了这个属性

## 生产者确认

### 事务机制

```java
public class Transaction {
    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = null;
       try {
           channel = MqInit.getChannel();
           channel.txSelect();
           channel.basicPublish("ex1", "key1", MessageProperties.PERSISTENT_TEXT_PLAIN, "test".getBytes(StandardCharsets.UTF_8));
           channel.txCommit();
       } catch (Exception e) {
           e.printStackTrace();
           channel.txRollback();
       }
    }
}
```

**使用事务会耗尽RabbitMQ的性能**

