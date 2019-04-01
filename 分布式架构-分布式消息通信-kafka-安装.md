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
    advertised.listeners=PLAINTEXT://192.168.18.131:9092
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
            listeners=PLAINTEXT://:9093
            log.dirs=/tmp/kafka-logs-1
        
        config/server-2.properties:
            broker.id=2
            listeners=PLAINTEXT://:9094
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
        bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 3 --partitions 1 --topic my-replicated-topic
        ```

    5. 查看主题的描述



原文：http://kafka.apache.org/quickstart