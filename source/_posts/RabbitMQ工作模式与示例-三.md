---
title: RabbitMQ工作模式与示例(三)
date: 2012-12-01 16:56:35
tags: [amqp, python, rabbitmq]
---
示例3中的交换机使用了topic模式，这种模式下交换机会试图将消息的路由键与规则进行模式匹配。
模式中，符号‘#’匹配一个或多个词，符号‘*’匹配不多于一个词。
如‘#’可匹配到(a.b) (a.b.c)  (b.c) ，即所有路由键
而‘a.*’可匹配到(a.b)
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
$exchange->setName('exchange3');
$exchange->setType('topic');
$exchange->declare();

// Create Queue
// we will use temp queue, so no queue should be created here

$message = $exchange->publish('a.b - Custom Message (ts): '.time(), 'a.b');
$message = $exchange->publish('a.b.c - Custom Message (ts): '.time(), 'a.b.c');
$message = $exchange->publish('b.c - Custom Message (ts): '.time(), 'b.c');
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
    channel.exchange_declare(exchange='exchange3', type='topic')

    #here we create 2 temp queues
    result_a = channel.queue_declare()
    queue_name_a=result_a.method.queue
    result_b = channel.queue_declare()
    queue_name_b=result_b.method.queue


    channel.queue_bind(exchange='exchange3',
                       queue=queue_name_a,
                       routing_key='a.*')
    channel.queue_bind(exchange='exchange3',
                       queue=queue_name_b,
                       routing_key='#')

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
