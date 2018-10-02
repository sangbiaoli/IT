## ActiveMQ介绍
ActiveMQ 是Apache出品，最流行的，能力强劲的开源消息总线。ActiveMQ 是一个完全支持JMS1.1和J2EE 1.4规范的 JMS Provider实现，尽管JMS规范出台已经是很久的事情了，但是JMS在当今的J2EE应用中间仍然扮演着特殊的地位。 

特性列表： 
1. 多种语言和协议编写客户端。语言: Java,C,C++,C#,Ruby,Perl,Python,PHP。应用协议： OpenWire,Stomp REST,WS Notification,XMPP,AMQP 
2. 完全支持JMS1.1和J2EE 1.4规范 （持久化，XA消息，事务) 
3. 对spring的支持，ActiveMQ可以很容易内嵌到使用Spring的系统里面去，而且也支持Spring2.0的特性 
4. 通过了常见J2EE服务器（如 Geronimo,JBoss 4,GlassFish,WebLogic)的测试，其中通过JCA 1.5 resource adaptors的配置，可以让ActiveMQ可以自动的部署到任何兼容J2EE 1.4 商业服务器上 
5. 支持多种传送协议：in-VM,TCP,SSL,NIO,UDP,JGroups,JXTA 
6. 支持通过JDBC和journal提供高速的消息持久化 
7. 从设计上保证了高性能的集群，客户端-服务器，点对点 
8. 支持Ajax 
9. 支持与Axis的整合 
10. 可以很容易的调用内嵌JMS provider，进行测试

## ActiveMQ安装配置
1. 下载

    到ActiveMQ官网，找到下载点。

    目前，官网为http://activemq.apache.org/。

    截止2018.09.10，最新的Linux版本包下载地址：http://mirrors.hust.edu.cn/apache//activemq/5.15.6/apache-activemq-5.15.6-bin.tar.gz。

 
2. 启动
    ```bash
    # 下载，并解压
    [root@localhost bill]# cd /usr/local/src/
    [root@localhost src]# wget http://mirrors.hust.edu.cn/apache//activemq/5.15.6/apache-activemq-5.15.6-bin.tar.gz
    [root@localhost src]# tar -xvf apache-activemq-5.15.6-bin.tar.gz

    #启动（当然，由于依赖于JAVA，如果你没有安装JAVA，它会提醒你的，哈哈）
    [root@localhost src]# cd apache-activemq-5.15.6
    [root@localhost src]# ./bin/activemq start
    INFO: Loading '/usr/local/src/apache-activemq-5.15.6//bin/env'
    INFO: Using java '/usr/java/jdk1.8.0_152/bin/java'
    INFO: Starting - inspect logfiles specified in logging.properties and log4j.properties to get details
    INFO: pidfile created : '/usr/local/src/apache-activemq-5.15.6//data/activemq.pid' (pid '2272')
    ```
3. 测试启动成功与否

    ActiveMQ默认监听61616端口，查此端口看看是否成功启动，如果一切顺利，会看到如下日志
    ```bash
    [root@localhost src]#  netstat -an | grep 61616
    tcp6       0      0 :::61616                :::*                    LISTEN
    ```

4. 登录管理员页面
    ```
    URL : http://192.168.141.129:8161/admin/
    默认的用户名/密码 : admin/admin
    ```

参考：

https://www.cnblogs.com/hushaojun/p/6016709.html