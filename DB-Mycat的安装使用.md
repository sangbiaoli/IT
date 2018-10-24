## CentOS6.6安装使用MyCat
1. 安装Java环境

    MyCAT 是使用 JAVA 语言进行编写开发，使用前需要先安装 JAVA 运行环境(JRE),由于 MyCAT 中使用了JDK7 中的一些特性，所以要求必须在 JDK7 以上的版本上运行。

    1. 查看是否已经安装jdk
        ```bash
        [root@localhost lmy]# rpm -qa | grep java
        ```
        显示没有安装jdk，若是存在centOS自带的openjdk，需要卸载。

    2. 安装jdk参考文档：linux-jdk.md

2. 安装MySQL

    1. 卸载原有的mysql

        首先查看系统中是否已经安装mysql
        ```bash
        [root@localhost java]# rpm -qa | grep mysql
        ```
        此时显示系统中并没有安装mysql,如果已经安装了，使用该命令卸载删除。
        ```bash
        [root@localhost java]# rpm -e mysql     //普通删除方式
        [root@localhost java]# rpm -e --nodeps mysql   // 强力删除模式，如果使用上面命令删除时，提示有依赖的其它文件，则用该命令可以对其进行强力删除
        ```
    2. 安装mysql参考文档：linux-mysql.md
   

3. 安装MyCat

    MyCat的安装比较简单，只要下载下来解压，然后就是编写配置文件了。

    1. 下载解压MyCat

        此时我下载的版本为1.6，你也可以根据自己的需要到下载地址选择需要的版本下载。
        ```bash
        [root@localhost java]# cd /usr/local/src/    //下载到该目录
        [root@localhost java]# wget http://dl.mycat.io/1.6-RELEASE/Mycat-server-1.6-RELEASE-20161028204710-linux.tar.gz
        [root@localhost java]# tar -zxvf Mycat-server-1.6-RELEASE-20161028204710-linux.tar.gz   //解压
        ```
    2. 测试启动mycat
        
        Linux环境下启动mycat的方式为：进入bin目录下，使用 ./mycat start命令启动。
        ```bash
        [root@localhost bin]# cd /usr/local/src/mycat/bin
        [root@localhost bin]# ./mycat start
        Starting Mycat-server...
        ```
    3. 测试MyCat

        整体的结构是这样的：有两个表，一个user表，一个goods表。
        * user表放在db01数据库中
        * goods表使用分片的方法放在db02和db03数据库中

        虽然goods表被分片存储在不同的数据库中，但是通过MyCat访问goods表时仍然像一张表一样。我们先在mysql中建立这三张表，然后通过MyCat将分片规则配置，再通过MyCat进行增删改查操作。 

        ![](db/db-mycat-db.png)

        1. 建立数据库和表

            首先我们先在mysql中新建三个数据库和这三张表。脚本分别如下：
            ```sql
            create database db01; 
            use db01;
            CREATE TABLE user (  
                id INT NOT NULL AUTO_INCREMENT,  
                name varchar(50) NOT NULL default '',  
                indate DATETIME NOT NULL default '0000-00-00 00:00:00',  
                PRIMARY KEY (id)  
            )AUTO_INCREMENT= 1 ENGINE=InnoDB DEFAULT CHARSET=utf8;  


            create database db02;  
            use db02;
            CREATE TABLE goods(  
                id INT NOT NULL AUTO_INCREMENT,  
                value INT NOT NULL default 0,  
                indate DATETIME NOT NULL default '0000-00-00 00:00:00',  
                PRIMARY KEY (id)  
            )AUTO_INCREMENT= 1 ENGINE=InnoDB DEFAULT CHARSET=utf8;

            create database db03;  
            use db03;
            CREATE TABLE goods(  
                id INT NOT NULL AUTO_INCREMENT,  
                value INT NOT NULL default 0,  
                indate DATETIME NOT NULL default '0000-00-00 00:00:00',  
                PRIMARY KEY (id)  
            )AUTO_INCREMENT= 1 ENGINE=InnoDB DEFAULT CHARSET=utf8; 
            ```
        2. 配置MyCat.
            
            下面我们要配置三个文件，在mycat/conf目录里面。
            * conf/server.xml：定义用户以及系统相关变量，如端口等. 
            * conf/schema.xml：定义逻辑库，表、分片节点等内容. 
            * conf/rule.xml：中定义分片规则.
            先备份，在按照下面的内容替换
            server.xml
            ```xml
            <?xml version="1.0" encoding="UTF-8"?>
            <!DOCTYPE mycat:server SYSTEM "server.dtd">
            <mycat:server xmlns:mycat="http://io.mycat/">

                <system>
                    <!-- 新增这一项 -->
                    <property name="defaultSqlParser">druidparser</property>
                    <!-- 开启这两项 -->
                    <property name="serverPort">8066</property>
                    <property name="managerPort">9066</property>
                </system>

                <!-- 新增一个user -->
                <user name="mycat">
                    <property name="password">mycat</property>
                    <property name="schemas">TESTDB</property>
                </user>

            </mycat:server>
            ```

            schema.xml
            ```xml
            <?xml version="1.0" encoding="UTF8"?>
            <!DOCTYPE mycat:schema SYSTEM "schema.dtd">
            <mycat:schema xmlns:mycat="http://io.mycat/">
                <!-- 设置表的存储方式.schema name="TESTDB" 与 server.xml中的 TESTDB 设置一致  -->  
                <schema name="TESTDB" checkSQLschema="false" sqlMaxLimit="100">  
                    <table name="user" primaryKey="id"  dataNode="node_db01" />  
                    <table name="goods" primaryKey="id" dataNode="node_db02,node_db03" rule="rule1" />  

                </schema>  

                <!-- 设置dataNode 对应的数据库,及 mycat 连接的地址dataHost -->  
                <dataNode name="node_db01" dataHost="dataHost01" database="db01" />  
                <dataNode name="node_db02" dataHost="dataHost01" database="db02" />  
                <dataNode name="node_db03" dataHost="dataHost01" database="db03" />  

                <!-- mycat 逻辑主机dataHost对应的物理主机.其中也设置对应的mysql登陆信息 -->  
                <dataHost name="dataHost01" maxCon="1000" minCon="10" balance="0" writeType="0" dbType="mysql" dbDriver="native">  
                        <heartbeat>select user()</heartbeat>  
                        <writeHost host="server1" url="127.0.0.1:3306" user="root" password="123456"/>  
                </dataHost>  
            </mycat:schema>
            ```

            rule.xml
            ```xml
            <?xml version="1.0" encoding="UTF8"?>
            <!DOCTYPE mycat:rule SYSTEM "rule.dtd">
            <mycat:rule xmlns:mycat="http://io.mycat/">
                <tableRule name="rule1">
                    <rule>
                        <columns>id</columns>
                        <algorithm>mod-long</algorithm>
                    </rule>
                </tableRule>
                <!-- 分片规则为模2运算 -->  
                <function name="mod-long" class="io.mycat.route.function.PartitionByMod">
                    <!-- how many data nodes -->
                    <property name="count">2</property>
                </function>
            </mycat:rule>
            ```
            三个配置文件配置完之后，重启mycat服务。
        3. 重启mycat服务
            ```bash
            [root@localhost bin]# cd /usr/local/src/mycat/bin/
            [root@localhost bin]# ./mycat restart
            ```

        4. 测试结果

            我们先访问mycat逻辑数据库，此时的视角是看不出分库分表的信息，这些表就好像在一个数据库中。
            ```bash
            [root@localhost bin]# mysql -umycat -pmycat -h127.0.0.1 -P8066 -DTESTDB 
            ```
            补充：后来我在其它机器上实验时，设置了两个datahost,启动mycat时一直报错

            ```bash
            [root@localhost bin]# mysql -umycat -pmycat -h127.0.0.1 -P8066 -DTESTDB 
            ERROR 2003 (HY000): Can’t connect to MySQL server on '127.0.0.1' (111) 
            ```
            我查找了server.xml schema.xml发现用户名，密码都没有配置错，端口也开放了，最后发现有一个datahost的没有写，希望以后对碰到此类问题的朋友有帮助。*

            ![](db/db-mycat-login.png)

            ok，现在我们可以往这两个表中插入数据，看看mycat能否按照我们之前的规则，将数据插入到相应的数据库表中。
            ```sql
            insert into user(name,indate) values('hello',now());
            insert into user(name,indate) values('world',now());
            insert into goods(id,value,indate) values(1,100,now());
            insert into goods(id,value,indate) values(2,100,now());
            ```

            ![](db/db-mycat-table.png)
            可以看到，对于mycat的逻辑表来说，数据已经插入成功了，但是实际上数据是否按照规则存在那三个数据库中呢？我们登录实际的数据库来查看。

            ![](db/db-mycat-query.png)
            ![](db/db-mycat-db01.png)
            ![](db/db-mycat-db02.png)
            我们看到，插入到user表中的数据全部都在db01的user表中，插入到goods表中的数据分别放在了db02和db03数据库中的goods表中。完美的完成了分库分表的操作！

参考：https://blog.csdn.net/u012948302/article/details/78902092