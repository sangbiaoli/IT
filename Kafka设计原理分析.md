## **Kafka简介**
转载请注明出处：http://www.cnblogs.com/BYRans/


Apache Kafka发源于LinkedIn，于2011年成为Apache的孵化项目，随后于2012年成为Apache的主要项目之一。Kafka使用Scala和Java进行编写。Apache Kafka是一个快速、可扩展的、高吞吐、可容错的分布式发布订阅消息系统。Kafka具有高吞吐量、内置分区、支持数据副本和容错的特性，适合在大规模消息处理场景中使用。

接下来先介绍下消息系统的基本理念，然后再介绍Kafka。

### **消息系统介绍**
一个消息系统负责将数据从一个应用传递到另外一个应用，应用只需关注于数据，无需关注数据在两个或多个应用间是如何传递的。分布式消息传递基于可靠的消息队列，在客户端应用和消息系统之间异步传递消息。有两种主要的消息传递模式：点对点传递模式、发布-订阅模式。大部分的消息系统选用发布-订阅模式。

### **点对点消息系统**
在点对点消息系统中，消息持久化到一个队列中。此时，将有一个或多个消费者消费队列中的数据。但是一条消息只能被消费一次。当一个消费者消费了队列中的某条数据之后，该条数据则从消息队列中删除。该模式即使有多个消费者同时消费数据，也能保证数据处理的顺序。这种架构描述示意图如下：

![](kafka/kafka_p2p-MsgQueue.png)

发布-订阅消息系统
在发布-订阅消息系统中，消息被持久化到一个topic中。与点对点消息系统不同的是，消费者可以订阅一个或多个topic，消费者可以消费该topic中所有的数据，同一条数据可以被多个消费者消费，数据被消费后不会立马删除。在发布-订阅消息系统中，消息的生产者称为发布者，消费者称为订阅者。该模式的示例图如下：

![](kafka/kafka_pub-subMsgQueue.png)


## **Kafka工作流程**
Kafka将某topic的数据存储到一个或多个partition中。一个partition内数据是有序的，每条数据都有一个唯一的index，这个index叫做offset。新来的数据追加到partition的尾部。每条数据可以在不同的broker上做备份，从而保证了Kafka使用的可靠性。

生产者将消息发送到topic中，消费者可以选择多种消费方式消费Kafka中的数据。下面介绍两种消费方式的流程。

### **一个消费者订阅数据：**
* 生产者将数据发送到指定topic中
* Kafka将数据以partition的方式存储到broker上。Kafka支持数据均衡，例如生产者生成了两条消息，topic有两个partition，那么Kafka将在两个partition上分别存储一条消息
* 消费者订阅指定topic的数据
* 当消费者订阅topic中消息时，Kafka将当前的offset发给消费者，同时将offset存储到Zookeeper中
* 消费者以特定的间隔（如100ms）向Kafka请求数据
* 当Kafka接收到生产者发送的数据时，Kafka将这些数据推送给消费者
消费者受到Kafka推送的数据，并进行处理
当消费者处理完该条消息后，消费者向Kafka broker发送一个该消息已被消费的反馈
* 当Kafka接到消费者的反馈后，Kafka更新offset包括Zookeeper中的offset。
* 以上过程一直重复，直到消费者停止请求数据
* 消费者可以重置offset，从而可以灵活消费存储在Kafka上的数据

### **消费者组数据消费流程**
Kafka支持消费者组内的多个消费者同时消费一个topic，一个消费者组由具有同一个Group ID的多个消费者组成。具体流程如下：

* 生产者发送数据到指定的topic
* Kafka将数据存储到broker上的partition中
* 假设现在有一个消费者订阅了一个topic，topic名字为“test”，消费者的Group ID为“Group1”
* 此时Kafka的处理方式与只有一个消费者的情况一样
* 当Kafka接收到一个同样Group ID为“Group1”、消费的topic同样为“test"的消费者的请求时，Kafka把数据操作模式切换为分享模式，此时数据将在两个消费者上共享。
* 当消费者的数目超过topic的partition数目时，后来的消费者将消费不到Kafka中的数据。因为在Kafka给每一个消费者消费者至少分配一个partition，一旦partition都被指派给消费者了，新来的消费者将不会再分配partition。即一个partition只能分配给一个消费者，一个消费者可以消费多个partition。

## **Kafka自带工具**

Kafka tool包在org.apache.Kafka.tools.*下，分为系统工具和复制工具两类，重点介绍几个系统工具：

* Kafka Migration Tool：该工具用于将broker的版本从一个版本更新或还原为另一版本。
* Mirror Maker：该工具用于将源Kafka集群的数据镜像到目的集群。
* Consumer Offset Checker：该工具用于显示指定topic和消费者组的信息，信息包括：消费者组名、topic名、partition、offset、logSize、owner等。