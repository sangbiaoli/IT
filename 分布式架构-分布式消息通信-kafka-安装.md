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

2. 启动服务端

    Kafka使用zookeeper，因此在启动Kafka之前，再启动
    ```bash
    ./bin/zookeeper-server-start.sh config/zookeeper.properties
    ```

原文：http://kafka.apache.org/quickstart