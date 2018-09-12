---
title: RabbitMQ工作模式与示例(二)
date: 2012-12-01 17:00:26
tags: [amqp]
---
![](https://dn-xiwei.qbox.me/wp-content/uploads/2012/12/F24833E55C4DABB1F06536F6D7F7D34270E33C05049E_524_174.jpeg)
示例2中，交换机被配置为direct路由方式，它将根据bind路由信息将特定的路由键信息送至对应的队列中。
生产者只负责将信息发送到交换机，而不关心交换机是如何配置的。从代码中可以看出这一点。
<!--more--->
#### php:
```bash
<?php

// Create a connection
$connection = new AMQPConnection();
$connection->connect();
if (!$connection->isConnected()) {
    die('Not connected :(' . PHP_EOL);
}

// Create and open a channel
$channel = new AMQPChannel($connection);

// Declare exchange
$exchange = new AMQPExchange($channel);
$exchange->setName('exchange2');
$exchange->setType('direct');
$exchange->declare();

// Create Queue
// we will use temp queue, so no queue should be created here

$message = $exchange->publish('key1 - Custom Message (ts): '.time(), 'key1');
$message = $exchange->publish('key2 - Custom Message (ts): '.time(), 'key2');
$message = $exchange->publish('key3 - Custom Message (ts): '.time(), 'key3');
if (!$message) {
    echo 'Message not sent', PHP_EOL;
} else {
    echo 'Message sent!', PHP_EOL;
}
?>
```
#### python:
```bash
import pika
def mqClient_rcv():
    credentials = pika.PlainCredentials("guest", "guest")
    conn_params = pika.ConnectionParameters("172.16.2.38", credentials = credentials)
    connection = pika.BlockingConnection(conn_params)

    channel = connection.channel()
    channel.exchange_declare(exchange='exchange2', type='direct')

    #here we create 2 temp queues
    result_a = channel.queue_declare()
    queue_name_a=result_a.method.queue
    result_b = channel.queue_declare()
    queue_name_b=result_b.method.queue

    #bind
    channel.queue_bind(exchange='exchange2',
                       queue=queue_name_a,
                       routing_key='key1')
    channel.queue_bind(exchange='exchange2',
                       queue=queue_name_a,
                       routing_key='key2')
    channel.queue_bind(exchange='exchange2',
                       queue=queue_name_b,
                       routing_key='key3')

    channel.basic_consume(callback_a,
                          queue=queue_name_a,
                          no_ack=True)
    channel.basic_consume(callback_b,
                          queue=queue_name_b,
                          no_ack=True)
    channel.start_consuming()

def callback_a(ch, method, properties, body):
    print "Queue A Received %r" % (body,)

def callback_b(ch, method, properties, body):
    print "Queue B Received %r" % (body,)

mqClient_rcv()
```
