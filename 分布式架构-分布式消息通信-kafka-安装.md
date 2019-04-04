## kafka安装

环境说明

虚拟机：CentOS7 64bit

编写文档时，Kafka的发布版本是2.2.0，因此以这个版本为例。

1. 下载
    
    访问地址：https://www.apache.org/dyn/closer.cgi?path=/kafka/2.2.0/kafka_2.12-2.2.0.tgz，并下载安装包
    
    ```bash
    wget http://mirrors.shu.edu.cn/apache/kafka/2.2.0/kafka_2.12-2.2.0.tgz
    tar -xzf kafka_2.12-2.2.0.tgz
    cd kafka_2.12-2.2.0
    ```

    修改配置
    /config/server.properties
    
    ```
    listeners=PLAINTEXT://:9092     #socket监听端口，就按这样的格式，不要加localhost
    advertised.listeners=PLAINTEXT://192.168.18.131:9092  #启动端口提供给provider和consumer
    ```

    192.168.18.131为本机地址，这样其他主机可以访问Kafka，比如生产消息，消费消息

2. 启动服务端

    Kafka使用zookeeper，因此在启动Kafka之前，要先启动zookeeper

    ```bash
    bin/zookeeper-server-start.sh config/zookeeper.properties &

    netstat -nlp|grep :2181
    ```

    现在启动kafka

    ```bash
    bin/kafka-server-start.sh config/server.properties &

    netstat -nlp|grep :9092
    ```

    **注意**
    kafka的broker，topic节点会注册到zookeeper上的，如果停止原来的zookeeper并启动另一个zookeeper，新起的这个zookeeper是看不到topic的。

3. 创建一个主题

    创建一个名为“test”的主题，只有一个分区和一个副本

    ```bash
    bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test
    ```

    查看主题列表

    ```bash
    bin/kafka-topics.sh --list --bootstrap-server localhost:9092
    ```

4. 发送消息

    Kafka附带一个命令行客户机，它将从文件或标准输入中获取输入，并将其作为消息发送到Kafka集群。默认情况下，每一行都将作为单独的消息发送。

    运行生成器，然后在控制台中键入一些消息发送到服务器。

    ```bash
    bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
    This is a message
    This is another message
    ^C
    ```

5. 启动一个消费者

    Kafka还有一个命令行使用者，它将把消息转储到标准输出。

    ```bash
    bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
    This is a message
    This is another message
    ^C
    ```

6. 设置多代理集群

    对于Kafka，单个代理只是一个大小为one的集群，因此除了启动多个代理实例外，没有什么大的变化。但是为了更好地理解它，让我们将集群扩展到三个节点(仍然都在本地机器上)。

    1. 复制配置

        ```bash
        cp config/server.properties config/server-1.properties
        cp config/server.properties config/server-2.properties
        ```

    2. 修改配置文件

        ```
        config/server-1.properties:
            broker.id=1
            listeners=PLAINTEXT://:9095
            advertised.listeners=PLAINTEXT://192.168.18.131:9095  #启动端口提供给provider和consumer
            log.dirs=/tmp/kafka-logs-1
        
        config/server-2.properties:
            broker.id=2
            listeners=PLAINTEXT://:9096
            advertised.listeners=PLAINTEXT://192.168.18.131:9096  #启动端口提供给provider和consumer
            log.dirs=/tmp/kafka-logs-2
        ```

        在集群中每个节点的broken.id是唯一的且永久的。

    3. 启动新节点

        目前已经启动了zookeeper和单独的一个节点，现在启动两个新节点

        ```bash
        bin/kafka-server-start.sh config/server-1.properties &
        bin/kafka-server-start.sh config/server-2.properties &
        ```

    4. 现在创建一个复制因子为3的新主题

        ```bash
        bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic my
        ```

    5. 查看主题的描述

        ```bash
        bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my
        
        Topic:my       PartitionCount:1        ReplicationFactor:3     Configs:segment.bytes=1073741824
        Topic: my      Partition: 0    Leader: 1       Replicas: 0,2,1 Isr: 0,1,2
        ```

        * leader是负责给定分区的所有读写的节点。每个节点都是分区中随机选择的一部分的领导者。
        * replicas是复制这个分区日志的节点列表，无论它们是主节点还是当前活动节点。
        * isr是一组“同步”副本。这是副本列表的子集，副本列表当前是活动的，并被提交给领导者。

    6. 生产一些消息

        ```bash
        bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my
        my test message 
        my test message 2
        my test message 3
        my test message 4
        ^C
        ```

    7. 测试容错性

        如上第5步，主题my的Leader节点为broker1，现在停止该broker

        ```bash
        netstat -nlp | grep 9095

        tcp6       0      0 :::9092                 :::*                    LISTEN      29195/java

        kill -9 29195
        ```

        此时再来查看，发现主题my的Leader节点更新为broker2。

        ```bash
        bin/kafka-topics.sh --describe --bootstrap-server localhost:9092 --topic my

        Topic:my        PartitionCount:1        ReplicationFactor:3     Configs:segment.bytes=1073741824
        Topic: my       Partition: 0    Leader: 2       Replicas: 1,2,0 Isr: 2,0
        ```

        尝试消费信息

        ```bash
        bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my
        
        my test message 
        my test message 2
        my test message 3
        my test message 4
        ```

7. 脚本

    * broker启动脚本
    
        ```bash
        #bash
        cd /usr/local/src/kafka_2.12-2.2.0/
        bin/kafka-server-start.sh ./config/server.properties &
        ```

    * 停止broker

        ```bash
        bin/kafka-server-stop.sh 9092
        ```

    * zookeeper启动脚

        ```bash
        #bash
        cd /usr/local/src/kafka_2.12-2.2.0/
        bin/zookeeper-server-start.sh ./config/zookeeper.properties &
        ```

    * 停止zookeeper

        ```bash
        bin/zookeeper-server-stop.sh
        ```

    配置脚本可执行
    ```bash
    chmod a+x startBroker.sh
    chmod a+x startZookeeper.sh
    ```

8. server.properties的常用配置项

    名称|描述|类型
    --|--|--
    broker.id|broker的id，必须是全局唯一的。如果没设置，则会自动生成一个唯一id|int
    listeners|监听器列表——将侦听以逗号分隔的uri列表和侦听器名称</br>合法样例：PLAINTEXT://myhost:9092,SSL://:9091|string
    advertised.listeners|代理将向生产者和消费者发布主机名和端口，如果没设置，则用listeners</br>样例:PLAINTEXT://192.168.18.131:9092|string
    log.dirs|存储log数据的目录|string
    zookeeper.connect|zookeeper连接字符串，</br>格式为hostname1:port1,hostname2:port2|string
    zookeeper.connection.timeout.ms|连接zookeeper的超时时间(ms)|int
    
原文：http://kafka.apache.org/quickstart