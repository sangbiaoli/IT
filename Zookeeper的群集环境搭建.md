### quorum模式(法定人数)
1. 使用quorum模式有两种形式：
    * 使用多台机器，在每台机器上执行一个ZooKeeper Server进程，一般用在生产环境中。
    * 使用一台机器，在该台机器上执行多个ZooKeeper Server进程，一般用在练习环境中。

2. 參数配置
在quorum模式下，要使一个ZooKeeper Server进程能够正常执行，须要配置一些參数。下面是常见的一些參数。
    * data文件夹
        用于存放进程执行数据
    * data文件夹下的myid文件
        用于存储一个数值，该数值用来作为该ZooKeeper Server进程的标识
    * 监听Client端请求的port号
        该port号用来监听Client端请求
    * 监听同ZooKeeper集群内其它ZooKeeper Server进程通信请求的port号
        该port号用来监听同ZooKeeper集群内其它ZooKeeper Server进程的通信请求
    * 监听ZooKeeper集群内首选请求的port号
        该port号用来监听ZooKeeper集群内首选的请求。注意这个是ZooKeeper集群内“leader”的选举，跟分布式应用程序无关

3. 參数配置注意事项
    * 同一个ZooKeeper集群内，不同ZooKeeper Server进程的标识须要不一样。即myid文件内的值须要不一样 
    * 采用上述第2种形式构建ZooKeeper集群，须要注意“文件夹，port号”等资源的不可共享性，假设共享会导致ZooKeeper Server进程不能正常执行。比方“data文件夹。几个监听port号”都不能被共享

### 练习环境举例
采用上述第2种形式构建一个使用quorum模式的ZooKeeper集群。
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
3. 执行ZooKeeper Server
```bash
cd /usr/local/src/zookeeper-3.4.10
./bin/zkServer.sh start deploy/z1/z1.cfg
./bin/zkServer.sh start deploy/z2/z2.cfg
./bin/zkServer.sh start deploy/z3/z3.cfg
```
启动三个zookeeper进程

4. 执行ZooKeeper命令行client
```bash
cd /usr/local/src/zookeeper-3.4.10
./bin/zkCli.sh -server 127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183
```
建立ZooKeeper Client端到ZooKeeper集群的连接会话。