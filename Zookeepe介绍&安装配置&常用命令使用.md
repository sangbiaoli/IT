
### 一、Zookeepe介绍
1. 特点及应用场景
    * ZooKeeper的设计更专注于任务协作， 并不提供任何锁的接
口或通用存储数据接口。 同时， ZooKeeper没有给开发人员强加任何特
殊的同步原语， 使用起来非常灵活。
    * 构建分布式系统
2. 使用实例
    * Apache HBase
    * Apache Kafka
    * Apache Solr
    * Yahoo！ Fetching Service
    * Facebook Messages
3. ZooKeeper不适用的场景
    整个ZooKeeper的服务器集群管理着应用协作的关键数据，ZooKeeper不适合用作海量数据存储。

### 二、安装配置
1. 下载
    可以去zookeeper官网http://zookeeper.apache.org下载，本例子用3.4.10.版本，链接地址为
    http://mirrors.hust.edu.cn/apache/zookeeper/zookeeper-3.4.10/zookeeper-3.4.10.tar.gz

2. 安装及配置
```bash
[root@localhost /]# cd /usr/local/src   #我的所有源码包习惯放在该目录下
[root@localhost src]# wget http://mirrors.hust.edu.cn/apache/zookeeper/zookeeper-3.4.10/zookeeper-3.4.10.tar.gz
[root@localhost src]# tar -xvzf zookeeper-3.4.10.tar.gz
[root@localhost src]# cd zookeeper-3.4.10/conf
[root@localhost conf]# cp zoo_sample.cfg zoo.cfg #复制默认的配置
[root@localhost conf]# vi zoo.cfg
dataDir=/usr/local/zk/data      #修改data目录
dataLogDir=/usr/local/zk/logs   #修改log目录
```

```bash
vi /etc/profile
#追加下面内容
export ZOOKEEPER_HOME=/usr/local/src/zookeeper-3.4.10
export PATH=$PATH:$ZOOKEEPER_HOME/bin:$ZOOKEEPER_HOME/conf

#保存后，令配置生效
source /etc/profile
````
### 三、常用命令使用
1. zookeeper server启动及停用
```bash
[root@localhost /]# cd /usr/local/src/zookeeper-3.4.10/conf

[root@localhost zookeeper-3.4.10]# ./bin/zkServer.sh start
[root@localhost zookeeper-3.4.10]# ./bin/zkServer.sh stop
```
2. zookeeper client启动及命令使用
```bash
[root@localhost /]# cd /usr/local/src/zookeeper-3.4.10/conf
[root@localhost zookeeper-3.4.10]# ./bin/zkCli.sh 
[zk: localhost:2181(CONNECTED) 0]  ls /     #查看已存在的节点
[zookeeper]
[zk: localhost:2181(CONNECTED) 1]  create /workers ""     #创建新节点workers，""串是为了不在znode中保存数据
Created /workers
[zk: localhost:2181(CONNECTED) 2] delete /workers   #删除节点
[zk: localhost:2181(CONNECTED) 3] quit #退出
```