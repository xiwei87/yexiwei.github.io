---
title: RabbitMQ工作模式与示例(一)
date: 2012-12-01 17:03:07
tags: [amqp]
---
![](https://static.yexiwei.com/wp-content/uploads/2012/12/1AE02B9AEE4C25FDA5A78996DCEFCBCC020ACD05049E_362_154.png)
#### 概念：
*     channel:通道，amqp支持一个tcp连接上启用多个mq通信通道，每个通道都可以被作为通信流。
*     producer：生产者，是消息产生的源头。
*     exchange：交换机，可以理解为具有路由表的路由规则。
*     queues：队列，装载消息的缓存容器。
*     consumer：消费者，连接到队列并取走消息的客户端。
#### 核心思想：
	 在RabbitMQ中，生产者从不直接将消息发送给队列。
    事实上，有些生产者甚至不知道消息是否被送到某个队列中去了。生产者只负责将消息送给交换机，而交换机确切地知道什么消息应该送到哪。
    bind：绑定，实际上可以理解为交换机的路由规则。每个消息都有一个称为路由键的属性(routing key)，就是一个简单的字符串。一个绑定将【交换机，路由键，消息送达队列】三者绑定在一起，形成一条路由规则。
    exchange type：交换机类型：
1.         fanout：不处理路由键，转发到所有绑定的队列上
1.         direct：处理路由键，必须完全匹配，即路由键字符串相同才会转发
1.         topic：路由键模式匹配，此时队列需要绑定要一个模式上。符号“#”匹配一个或多个词，符号“*”匹配不多不少一个词。因此“audit.#”能够匹配到“audit.irs.corporate”，但是“audit.*” 只会匹配到“audit.irs”
#### 示例1：Publish/Subscribe
示例1中生产者是基于php的，而消费者是基于python的，工作模式采用不处理路由键方式转发。
#### PHP:
```bash
<?php
// Create a connection
$connection = new AMQPConnection();
$connection->connect();
if (!$connection->isConnected()) {
    die('Not connected :(' . PHP_EOL);
}
// Create and open a channel
$channel    = new AMQPChannel($connection);
// Declare exchange
$exchange = new AMQPExchange($channel);
$exchange->setName('exchange1');
$exchange->setType('fanout');
$exchange->declare();
// Create Queue
$queue = new AMQPQueue($channel);
$queue->setName('queue1');
$queue->declare();
$message = $exchange->publish('Custom Message (ts): '.time(), 'key1');
if (!$message) {
    echo 'Message not sent', PHP_EOL;
} else {
    echo 'Message sent!', PHP_EOL;
}
?>
```
#### python:
```bash
def mqClient_rcv():
    #login
    credentials = pika.PlainCredentials("guest", "guest")
    conn_params = pika.ConnectionParameters("172.16.2.38", credentials = credentials)
    connection = pika.BlockingConnection(conn_params)
    channel = connection.channel()
    channel.exchange_declare(exchange='exchange1', type='fanout')
    result = channel.queue_declare()
    queue_name=result.method.queue
    channel.queue_bind(exchange='exchange1', queue=queue_name)
    channel.basic_consume(callback,
                          queue=queue_name,
                          no_ack=True)
    channel.start_consuming()

def callback(ch, method, properties, body):
    print type(body)
    print "[x] Received %r" % (body,)

mqClient_rcv()
```

