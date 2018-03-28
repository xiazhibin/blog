### 核心概念
- broker:接收和分发消息的应用，RabbitMQ Server就是Message Broker。
- exchange:接受生产者的消息并把它转到合适的队列
- queue:存储Exchange发来的消息，并将消息主动发送给Consumer或者 Consumer 主动来获取消息 
- binding:确定exchange和queue之间的关系
- channel和connection:生产者和消费者需要和 RabbitMQ 建立 TCP 连接。一些应用需要多个connection，为了节省TCP 连接，可以使用 Channel，它可以被认为是一种轻型的共享 TCP 连接的连接。连接需要显式关闭。

![概念](https://github.com/xiazhibin/blog/blob/master/pic/mq2.jpg)
![架构图](https://github.com/xiazhibin/blog/blob/master/pic/messagequeue.jpg)

生产者的publish
```python
import pika

params = pika.URLParameters('amqp://username:password@host/')
params.socket_timeout = 5
connection = pika.BlockingConnection(params)
channel = connection.channel()
channel.exchange_declare(exchange='hello_exchanger', exchange_type='direct')
channel.queue_declare(queue='hello')
channel.queue_bind(exchange='hello_exchanger', queue='hello', routing_key='info')
channel.basic_publish(exchange='hello_exchanger',routing_key='info',body='Hello World!2222')
connection.close()
```

消费者的subscribe
```python
import pika

params = pika.URLParameters('amqp://username:password@host/')
connection = pika.BlockingConnection(params)
channel = connection.channel()  # start a channel
channel.queue_declare(queue='hello')  # Declare a queue


def callback(ch, method, properties, body):
    print(body)

# set up subscription on the queue
channel.basic_consume(callback, queue='hello', no_ack=True)

# start consuming (blocks)
try:
    channel.start_consuming()
except:
    pass
finally:
    connection.close()
```

### exchange三种类型：`fanout` `topics` `direct`
- direct:任何发送到Direct Exchange的消息都会被转发到名为`routing_key`的Queue。
  - 一般情况可以使用rabbitMQ自带的Exchange:""(该Exchange的名字为空字符串) `channel.basic_publish(exchange='',routing_key='hello',body='Hello World!xxxxxxx')`
  - 这种模式下不需要将Exchange进行任何绑定(binding)操作.
  - 消息传递时需要一个`routing_key`，可以简单的理解为要发送到的队列名字。
  - 如果`vhost`不存在名为`routing_key`的队列，就会舍弃该消息
 
 - fanout:任何发送到Fanout Exchange的消息都会被转发到与该Exchange绑定(Binding)的所有Queue上。
   - 可以理解为路由表的模式
   - 这种模式不需要`routing_key`
   - 这种模式需要提前将Exchange与Queue进行绑定，一个Exchange可以绑定多个Queue，一个Queue可以同多个Exchange进行绑定。
   - 如果接受到消息的Exchange没有与任何Queue绑定，则消息会被抛弃。
   
 - topics:任何发送到Topic Exchange的消息都会被转发到所有关心`routing_key`中指定话题的Queue上
   - 可理解为跟正则匹配差不多
   - `#`表示0个或若干个关键字，`*`表示某位置一个关键字。如“log.*”能与“log.warn”匹配，无法与“log.warn.timeout”匹配；但是“log.#”能与上述两者匹配。
   - 这种模式需要`routing_key`，也许要提前绑定Exchange与Queue。`routing_key`一般用`.`进行分割，例如`log.app1`
   
### 消息的`ACK`
`consumer`可以选择发送`acknowledgment`给`broker`确定是否成功`comsume`
  
`ACK`告诉broker这条信息成功投递了并且正确处理了。del这条message.如果没有收到`ACK`，那么这个message就会类似
挂起来（不会被其他`consumer`处理），直到当前这个`consumer`断开连接
 
 - 当处理消息遇到不可恢复的错误是
   - 把当前消费者从RabbitMQ服务器断开连接。这会导致RabbitMQ自动重新把消息入队并发送到另外一个消费者。（频繁断开可能对RabbitMQ造成负荷）
   - 可以使用`basic.reject`，当`requeue`为`true`的时候才重新放入队列投递给下一个消费者，否则就移除出队列
   
 ### vhost
 vhost（虚拟消息服务器）类似于namespace，目的是为了将exchange和queue隔离，一个RabbitMQ可以服务更多的应用，而不需要担心exchange重名了。
 
 ### 消息持久化
 - 投递模式选项设置为2
 - 发送到持久化的exchange
 - 到达持久化的queue
持久化会影响消息的吞吐量

### 消息分发机制
- Round-robin dispatching 循环分发:RabbitMQ将第n个Message分发给第n个Consumer
- Fair dispatch 公平分发：`prefetch=1`这样RabbitMQ就会使得每个Consumer在同一个时间点最多处理一个Message。换句话说，在接收到该Consumer的ack前，他它不会将新的Message分发给它

### publisher confirm机制
确认自己的消息到达了broker
