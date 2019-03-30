### 安装配置

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

### 常用命令使用

1. zookeeper server启动及停用

    ```bash
    [root@localhost /]# cd /usr/local/src/zookeeper-3.4.10/conf

    [root@localhost zookeeper-3.4.10]# ./bin/zkServer.sh start
    [root@localhost zookeeper-3.4.10]# ./bin/zkServer.sh stop
    ```

2. zookeeper client启动及命令使用

    1. 连接和退出客户端

        ```bash
        [root@localhost /]# cd /usr/local/src/zookeeper-3.4.10/conf
        [root@localhost zookeeper-3.4.10]# ./bin/zkCli.sh 
        [zookeeper]
        [zk: localhost:2181(CONNECTED) 3] quit #退出
        ```

    2. 创建

        * create [-s] [-e] path data acl
      
        创建一个Zookeeper节点，其中，-s或-e分别指节点特性：顺序或临时节点。

        ```bash
        [root@localhost /]create /zk-book 123
        ```

    3. 读取

        * ls path [watch]

            列出Zookeeper指定节点下的所有子节点。

            ```bash
            [root@localhost /]ls /
            ```

        * get path [watch]

            获取Zookeeper指点节点的数据内容和属性信息

            ```bash
            [root@localhost /]get /zk-book
            ```

    4. 更新

        * set path data [version]

        可以更新指定节点的数据内容

        ```bash
        [root@localhost /]set /zk-book 456
        ```

    5. 删除

        * delete path [version]

        删除Zookeeper上指定的节点

        ```bash
        [root@localhost /]delete /zk-book
        ```

### 伪集群搭建

假设zookeeper目录为/usr/local/src/zookeeper-3.4.10，下面说明用$ZOOKEEPER_HOME代表

1. 集群规划如表

    myid|data文件夹|监听Client端请求的port号|监听同ZooKeeper集群内其它ZooKeeper Server进程通信请求的port号|监听ZooKeeper集群内首选请求的port号|配置文件名
    --|--|--|--|--|--|
    1|$ZOOKEEPER_HOME/deploy/z1/data|2181|2222|2223|z1.cfg
    2|$ZOOKEEPER_HOME/deploy/z2/data|2182|3333|3334|z2.cfg
    3|$ZOOKEEPER_HOME/deploy/z3/data|2183|4444|4445|z3.cfg

2. 按规划创建目录和文件

    ```bash
    cd /usr/local/src/zookeeper-3.4.10
    mkdir deploy
    cd deploy
    mkdir z1 z2 z3
    mkdir z1/data z2/data z3/data

    echo 1 > z1/data/myid
    echo 2 > z2/data/myid
    echo 3 > z3/data/myid

    touch z1/z1.cfg
    touch z2/z2.cfg
    touch z3/z3.cfg

    ```

3. 配置文件内容设置

    z1.cfg，z2.cfg，z3.cfg这3个文件的文件内容分别例如以下所看到的

    ```bash
    # z1.cfg
    tickTime=2000
    initLimit=10
    syncLimit=5

    dataDir=/usr/local/src/zookeeper-3.4.10/deploy/z1/data  #这里一定要用绝对路径，防止启动找不到myid
    clientPort=2181
    # server.x中的“x”表示ZooKeeper Server进程的标识
    # 同一个ZooKeeper集群内的ZooKeeper Server进程间的通信不仅能够使用详细的点IP地址。也能够使用组播地址
    server.1=127.0.0.1:2222:2223
    server.2=127.0.0.1:3333:3334
    server.3=127.0.0.1:4444:4445
    ```

    ```bash
    # z2.cfg
    tickTime=2000
    initLimit=10
    syncLimit=5

    dataDir=/usr/local/src/zookeeper-3.4.10/deploy/z2/data  #这里一定要用绝对路径，防止启动找不到myid
    clientPort=2182
    # server.x中的“x”表示ZooKeeper Server进程的标识
    # 同一个ZooKeeper集群内的ZooKeeper Server进程间的通信不仅能够使用详细的点IP地址，也能够使用组播地址
    server.1=127.0.0.1:2222:2223
    server.2=127.0.0.1:3333:3334
    server.3=127.0.0.1:4444:4445
    ```

    ```bash
    # z3.cfg
    tickTime=2000
    initLimit=10
    syncLimit=5

    dataDir=/usr/local/src/zookeeper-3.4.10/deploy/z3/data  #这里一定要用绝对路径，防止启动找不到myid
    clientPort=2183
    # server.x中的“x”表示ZooKeeper Server进程的标识
    # 同一个ZooKeeper集群内的ZooKeeper Server进程间的通信不仅能够使用详细的点IP地址。也能够使用组播地址
    server.1=127.0.0.1:2222:2223
    server.2=127.0.0.1:3333:3334
    server.3=127.0.0.1:4444:4445
    ```

4. 执行ZooKeeper Server

    ```bash
    cd /usr/local/src/zookeeper-3.4.10
    ./bin/zkServer.sh start deploy/z1/z1.cfg
    ./bin/zkServer.sh start deploy/z2/z2.cfg
    ./bin/zkServer.sh start deploy/z3/z3.cfg
    ```
    启动三个zookeeper进程

5. 执行ZooKeeper命令行client

    ```bash
    cd /usr/local/src/zookeeper-3.4.10
    ./bin/zkCli.sh -server 127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183
    ```
    建立ZooKeeper Client端到ZooKeeper集群的连接会话。

原文：从Paxos到Zookeeper++分布式一致性原理与实践.pdf
https://www.cnblogs.com/duan2/p/9011762.html