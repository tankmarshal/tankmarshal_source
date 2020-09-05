---
title: kafka的读写机制
date: 2020-09-04 17:34:33
tags: kafka
categories: kafka
---
# kafka的读写机制

## 概念
先了解kafka的几个概念：熟悉的可以略过
**Topic**
是Kafka下消息的类别，类似于RabbitMQ中的Exchange的概念。这是逻辑上的概念，用来区分、隔离不同的消息数据，屏蔽了底层复杂的存储方式。对于大多数人来说，在开发的时候只需要关注数据写入到了哪个topic、从哪个topic取出数据。
**Partition**
是Kafka下数据存储的基本单元，这个是物理上的概念。同一个topic的数据，会被分散的存储到多个partition中，这些partition可以在同一台机器上，也可以是在多台机器上，比如下图所示的topic就有4个partition，分散在两台机器上。这种方式在大多数分布式存储中都可以见到，比如MongoDB、Elasticsearch的分片技术，其优势在于：有利于水平扩展，避免单台机器在磁盘空间和性能上的限制，同时可以通过复制来增加数据冗余性，提高容灾能力。为了做到均匀分布，通常partition的数量通常是Broker Server数量的整数倍。
**Consumer Group**
同样是逻辑上的概念，是Kafka实现单播和广播两种消息模型的手段。同一个topic的数据，会广播给不同的group；同一个group中的worker，只有一个worker能拿到这个数据。换句话说，对于同一个topic，每个group都可以拿到同样的所有数据，但是数据进入group后只能被其中的一个worker消费。group内的worker可以使用多线程或多进程来实现，也可以将进程分散在多台机器上，worker的数量通常不超过partition的数量，且二者最好保持整数倍关系，因为Kafka在设计时假定了一个partition只能被一个worker消费（同一group内）。
**Broker** 
物理概念，指服务于Kafka的一个node。
**topic** 
MQ中的抽象概念，是一个消费标示。用于保证Producer以及Consumer能够通过该标示进行对接。可以理解为一种Naming方式。
**partition** 
是一个物理概念。partition会实际存储在系统的摸个目录。
Topic的一个子概念，一个topic可具有多个partition，但Partition一定属于一个topic。
值得注意的是：
&ensp;&ensp;&ensp;&ensp;在实现上都是以每个Partition为基本实现单元的。
&ensp;&ensp;&ensp;&ensp;消费时，每个消费线程最多只能使用一个partition。
&ensp;&ensp;&ensp;&ensp;一个topic中partition的数量，就是每个user group中消费该topic的最大并行度数量。

**User group** 
为了便于实现MQ中的多播，重复消费等引入的概念。如果ConsumerA以及ConsumerB同在一个UserGroup，那么ConsumerA消费的数据ConsumerB就无法消费了。即：所有usergroup中的consumer使用一套offset
**Offset** 
Offset专指Partition以及User Group而言，记录某个user group在某个partiton中当前已经消费到达的位置。
**Consumer Group**
consumer group是kafka提供的可扩展且具有容错性的消费者机制。既然是一个组，那么组内必然可以有多个消费者或消费者实例(consumer instance)，它们共享一个公共的ID，即group ID。组内的所有消费者协调在一起来消费订阅主题(subscribed topics)的所有分区(partition)。当然，每个分区只能由同一个消费组内的一个consumer来消费。理解consumer group记住下面这三个特性就好了：
&ensp;&ensp;&ensp;&ensp;1.consumer group下可以有一个或多个consumer instance，consumer instance可以是一个进程，也可以是一个线程
&ensp;&ensp;&ensp;&ensp;2.group.id是一个字符串，唯一标识一个consumer group
&ensp;&ensp;&ensp;&ensp;3.consumer group下订阅的topic下的每个分区只能分配给某个group下的一个consumer(当然该分区还可以被分配给其他group)