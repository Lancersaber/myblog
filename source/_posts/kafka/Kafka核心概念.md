---
title: kafka核心概念
time: 2019-11-17
categories: Kafka
tags: Kafka
---

# Kafka核心概念
---

## 1、消息
消息是Kafka中最基本的数据单元。消息由一串字节构成，其中主要由key和value构成，key和value也都是byte数组。key的主要作用是根据一定的策略，将次消息路由到指定的分区中，这样就可以保证包含同一key的消息全部写入同一分区中，key可以是null。消息的真正有效负载是value部分的数据。为了提高网络和存储的利用率，生产者会批量发送消息到Kafka，并在发送之前对消息进行压缩。

## 2、Topic&分区&Log
&emsp;&emsp;Topic是用于存储消息的逻辑概念，可以看作一个消息集合。每个Topic可以有多个生产者向其中推送(push)消息，也可以有任意多个消费者消费其中的消息，入下图
![2019011811321224.png](https://i.loli.net/2019/11/17/75lfDVbnQOTWq26.png)

&emsp;&emsp;每个Topic可以划分成多个分区(每个Topic都至少有一个分区)，同一Topic下的不同分区包含的消息是不同的。每个消息在被添加到分区时，都会被分配一个offset，它是消息在此分区中的唯一编号，Kafka通过offset保证消息在分区内的顺序，offset的顺序性不跨分区，即Kafka只保证在同一个分区内的消息是有序的；同一Topic的多个分区内的消息，Kafka并不保证其顺序性
![2019011811343340.png](https://i.loli.net/2019/11/17/imfGPgnpz1NlwOL.png)

&emsp;&emsp;同一Topic的不同分区会分配到不同的Broke。分区是Kafka水平扩展性的基础，我们可以通过增加服务器并在其上分配Partition的方式来增加Kafka的并行处理能力。
&emsp;&emsp;分区在逻辑上对应着一个Log，当生产者将消息写入分区时，实际上是写入到了分区对应的Log中。Log是一个逻辑概念，可以对应到磁盘上的一个文件夹。Log由多个Segment组成，每个Segment对应一个日志文件和索引文件。在面对海量数据时，为避免出现超大文件，每个日志文件的大小是有限制的，当超出限制后则会创建新的Segment，继续对外提供服务。这里需要注意，因为Kafka采用顺序I/O，所以只向最新的Segment追加数据。

### 保留策略(Rentention Policy) & 日志压缩(Log Compaction)
&emsp;&emsp;无论消费者是否已经消费了消息，Kafka都会一直保存这些消息，但不会像数据库那样长期保存。为了避免磁盘被占满，Kafka会配置相应的“保留策略(rentention policy”),以长期周期性地删除陈旧的消息。

&emsp;&emsp;两种保留策略
* 1.根据消息保留的时间，当消息在Kafka中保存的时间超过了指定的时间，就可以被删除
* 2.根据Topic存储的数据大小，当Topic所占的日志文件大小大于一个阈值，则可以开始删除最旧的消息。
Kafka会启动一个后台线程，定期检查是否存在可以删除的消息。保留策略的配置非常灵活，可以有全局的配置，也可以针对Topic进行配置来覆盖全局配置。

&emsp;&emsp;除此之外，Kafka还会进行“日志压缩”(Log Compaction).在很多场景中，消息的key与value的值之间的对应关系是不断变化的，就像数据库中的数据会不断修改一样，消费者只关心key对应的最新的value值。此时，可以开启Kafka的日志压缩功能，Kafka会在后台启动一个线程，定期将相同key的消息进行合并吗，只保留最新的value值。如下图：
![2019-11-17 16_35_26-未命名表单.drawio - draw.io.png](https://i.loli.net/2019/11/17/ZRqDsXc7YhokajF.png)



## 3、Broker
一个单独的Kafka Server就是一个Broker。Broker的主要工作就是接收生产者发过来的消息，分配offset，之后保存到磁盘中，同时，接收消费者、其他Broker的请求，根据请求类型进行相应处理并返回响应。在一般的生产环境中，一个Broker独占一台物理服务器。

## 副本
Kafka对消息进行了冗余备份，每个Partition可以有多个副本，每个副本中包含的消息是一样的。每个分区至少有一个副本，当分区中只有一个副本时，就只有Leader副本，没有Follower副本

每个分区的副本集合中，都会选举出一个副本作为Leader副本，Kafka在不同的场景下会采用不同的选举策略。所有的读写请求都由选举出的Leader副本处理，其他都作为Follower副本，Follower副本仅仅是从Leader副本处把数据拉取到本地之后，同步更新到自己的Log中。下图展示了出来。
![2019-11-17 16_52_08-未命名表单.drawio - draw.io.png](https://i.loli.net/2019/11/17/GVoNhRkyfvSs3Z9.png)

一般情况下，同一分区的多个分区会被分配到不同的Broker上，这样，当Leader所在的Broker宕机后，可以重新选举新的Leader，继续对外提供服务。

## ISR集合


## HW&LEO

## Cluster&Controller

## 生产者

## 消费者

## Consumer Group
在Kafka中，多个Consumer可以组成一个Consumer Group，一个Consumer只能属于一个Consumer Group。Consumer Group保证其订阅的Topic的每个分区只能被分配给此Consumer Group中的一个消费者处理。如果不同的Consumer Group订阅了同一个Topic，Consumer Group彼此之间不会干扰。 