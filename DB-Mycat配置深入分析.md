## MyCAT 配置解析


* server.xml	Mycat的配置文件，设置账号、参数等
* schema.xml	Mycat对应的物理数据库和数据库表的配置
* rule.xml	Mycat分片（分库分表）规则

### 一、server.xml
1. user标签
    ```xml 
    <user name="root">
        <property name="password"></property>
        <property name="schemas">TESTDB</property>
    </user>
    ```
    user	用户配置节点
    * --name	登录的用户名，也就是连接Mycat的用户名
    * --password	登录的密码，也就是连接Mycat的密码
    * --schemas	数据库名，这里会和schema.xml中的配置关联，多个用逗号分开，例如需要这个用户需要管理两个数据库db1,db2，则配置db1,dbs	

2. privileges标签 

    对用户的 schema以及表进行精细化的DML权限控制
    ```xml 
    <privileges check="false"></privileges>
    ```
    * --check	表示是否开启DML权限检查。默认是关闭。server.dtd文件中 <!ELEMENT privileges (schema)*> 说明可以有多个schema的配置。
    * --dml	顺序说明：insert,update,select,delete
    ```xml 
    <schema name="db1" dml="0110" >
        <table name="tb01" dml="0000"></table>
        <table name="tb02" dml="1111"></table>
    </schema>
    ``` 
    * db1的权限是update,select。 
    * tb01的权限是啥都不能干。 
    * tb02的权限是insert,update,select,delete。 
    *其他表默认是udpate,select。
3. system标签

    这个标签内嵌套的所有 property 标签都与系统配置有关。
    ```xml 
    <!-- 字符集 -->
    <property name="charset">utf8</property> 

    <!-- 处理线程数量，默认是cpu数量 -->
    <property name="processors">1</property> 
    
    <!-- 每次读取留的数量，默认4096 -->
    <property name="processorBufferChunk">4096</property> 

    <!-- 创建共享buffer需要占用的总空间大小,processorBufferChunk*processors*100-->
    <property name="processorBufferPool">409600</property>

    <!-- 默认为0。0表示DirectByteBufferPool，1表示ByteBufferArena。-->
    <property name="processorBufferPoolType">0</property>
    
    <!-- 二级共享buffer是processorBufferPool的百分比，这里设置的是百分比。-->
    <property name="processorBufferLocalPercent">100</property>
   
    <!-- 全局ID生成方式。(0:为本地文件方式，1:为数据库方式；2:为时间戳序列方式；3:为ZK生成ID；4:为ZK递增ID生成。-->
    <property name="sequnceHandlerType">100</property>
    
    <!-- 是否开启mysql压缩协议。1为开启，0为关闭，默认关闭。-->
    <property name="useCompression">1</property>
    
    <!-- 指定 Mysql 协议中的报文头长度。默认 4。-->
    <property name="packetHeaderSize">4</property> 
    
    <!-- 指定 Mysql 协议可以携带的数据最大长度。默认 16M。-->
    <property name="maxPacketSize">16M</property>
    
    <!-- 指定连接的空闲超时时间。某连接在发起空闲检查下，发现距离上次使用超过了空闲时间，那么这个连接会被回收，就是被直接的关闭掉。默认 30 分钟，单位毫秒。-->
    <property name="idleTimeout">1800000</property>
    
    <!-- 前端连接的初始化事务隔离级别，只在初始化的时候使用，后续会根据客户端传递过来的属性对后端数据库连接进行同步。默认为 REPEATED_READ，设置值为数字默认 3。 -->
    <!-- READ_UNCOMMITTED = 1; 
         READ_COMMITTED = 2; 
         REPEATED_READ = 3;
         SERIALIZABLE = 4; --> 
    <property name="txIsolation">3</property>
    
    <!-- SQL 执行超时的时间，Mycat 会检查连接上最后一次执行 SQL 的时间，若超过这个时间则会直接关闭这连接。默认时间为 300 秒，单位秒。 --> 
    <property name="sqlExecuteTimeout">300</property>
    
    <!-- 清理 NIOProcessor 上前后端空闲、超时和关闭连接的间隔时间。默认是 1 秒，单 
    位毫秒。 --> 
    <property name="processorCheckPeriod">1000</property>
    
    <!-- 对后端连接进行空闲、超时检查的时间间隔，默认是 300 秒，单位毫秒。 --> 
    <property name="dataNodeIdleCheckPeriod">300000</property> 
    
    <!-- 对后端所有读、写库发起心跳的间隔时间，默认是 10 秒，单位毫秒。 --> 
    <property name="dataNodeHeartbeatPeriod">10000</property>
    
    <!-- mycat 服务监听的 IP 地址，默认值为 0.0.0.0。 --> 
    <property name="bindIp">0.0.0.0</property>
    
    <!-- 定义 mycat 的使用端口，默认值为 8066。 --> 
    <property name="serverPort">8066</property>
    
    <!-- 定义 mycat 的管理端口，默认值为 9066。 --> 
    <property name="managerPort">9066</property>
    
    <!-- mycat 模拟的 mysql 版本号，默认值为 5.6 版本，如非特需，不要修改这个值，目前支持设置 5.5,5.6,5.7 版本，其他版本可能会有问题。 --> 
    <property name="fakeMySQLVersion">5.6</property>
    
    <!-- 是否开启实时统计。1为开启；0为关闭 。 -->
    <property name="useSqlStat">0</property> 
    
    <!-- 是否开启全局表一致性检测。1为开启；0为关闭 。 -->
    <property name="useGlobleTableCheck">0</property> 
    
    <!-- 分布式事务开关。0为不过滤分布式事务；1为过滤分布式事务；2 为不过滤分布式事务,但是记录分布式事务日志。 -->
    <property name="handleDistributedTransactions">0</property>
    
    <!--  默认是65535。 64K 用于sql解析时最大文本长度 
    以上举例的属性仅仅是一部分，可以配置的变量很多，具体可以查看SystemConfig这个类的属性内容。 
    System标签下的属性，一般是上线后，需要根据实际运行的情况，分析后调优的时候进行修改。 -->
    <property name="maxStringLiteralLength">65535</property>
    ```xml 

4. Firewall标签

    顾名思义，这个就是关于防火墙的设置，也就是在网络层对请求的地址进行限制，主要是从安全角度来保证Mycat不被匿名IP进行访问
    ```xml
    <firewall> 
        <whitehost>
            <host host="127.0.0.1" user="mycat"/>
            <host host="127.0.0.2" user="mycat"/>
        </whitehost>
        <blacklist check="false">
        </blacklist>
    </firewall>
    ```
    设置很简单，很容易理解，只要设置了白名单，表示开启了防火墙，只有白名单的连接才可以进行连接。

    
### 二、schema.xml
    
* --schema	数据库设置，此数据库为逻辑数据库，name与server.xml中schema对应
* --dataNode	分片信息，也就是分库相关配置
* --dataHost	物理数据库，真正存储数据的数据库

1. schema 标签
    ```xml
    <schema name="TESTDB" checkSQLschema="false" sqlMaxLimit="10">
    </schema>
    ```
    schema标签用来定义mycat实例中的逻辑库，mycat可以有多个逻辑库，每个逻辑库都有自己的相关配置。可以使用schema标签来划分这些不同的逻辑库
    如果不配置schema标签，所有表的配置会属于同一个默认的逻辑库。逻辑库的概念和MySql的database的概念一样，我们在查询两个不同逻辑库中的表的时候，需要切换到该逻辑库下进行查询。

    * --name	逻辑数据库名，与server.xml中的schema对应
    * --checkSQLschema	数据库前缀相关设置，当该值为true时，例如我们执行语句select * from TESTDB.company 。mycat会把语句修改为 select * from company 去掉TESTDB。
    * --sqlMaxLimit	当该值设置为某个数值时，每条执行的sql语句，如果没有加上limit语句，Mycat会自动加上对应的值。不写的话，默认返回所有的值。
    需要注意的是，如果运行的schema为非拆分库的，那么该属性不会生效。需要自己sql语句加limit。
2. table 标签
    ```xml
    <table name="travelrecord" dataNode="dn1,dn2,dn3" rule="auto-sharding-long" />
    <table name="company" primaryKey="ID" type="global" dataNode="dn1,dn2,dn3" />
    ```
    * --name	表名，物理数据库中表名
    * --dataNode	表存储到哪些节点，多个节点用逗号分隔。节点为下文dataNode设置的name
    * --primaryKey	主键字段名，自动生成主键时需要设置
    * --autoIncrement	是否自增
    * --rule	分片规则名，具体规则下文rule详细介绍
    * --type 该属性定义了逻辑表的类型，目前逻辑表只有全局表和普通表。全局表： global 普通表：无 
    注：全局表查询任意节点，普通表查询所有节点效率低
    * --autoIncrement mysql对非自增长主键，使用last_insert_id() 是不会返回结果的，只会返回0.所以，只有定义了自增长主键的表，才可以用last_insert_id()返回主键值。
    mycat提供了自增长主键功能，但是对应的mysql节点上数据表，没有auto_increment,那么在mycat层调用last_insert_id()也是不会返回结果的。
    * --needAddLimit 指定表是否需要自动的在每个语句后面加上limit限制，由于使用了分库分表，数据量有时候会特别庞大，这时候执行查询语句，
    忘记加上limt就会等好久，所以mycat自动为我们加上了limit 100，这个属性默认为true，可以自己设置为false禁用。如果使用这个功能，最好配合使用数据库模式的全局序列。
    * --subTables	分表，分表目前不支持Join。

    1. childTable标签
        ```xml
        <table name="customer" primaryKey="ID" dataNode="dn1,dn2" rule="sharding-by-intfile"> 
        <childTable name="c_a" primaryKey="ID" joinKey="customer_id" parentKey="id" />
        </table>
        ```
        * --childTable 标签用于定义 E-R 分片的子表。通过标签上的属性与父表进行关联。 
        * --name	子表的名称
        * --joinKey	子表中字段的名称
        * --parentKey	父表中字段名称
        * --primaryKey	同Table
        * --needAddLimit	同Table

3. dataNode标签
    ```xml
    <dataNode name="dn1" dataHost="localhost1" database="db1" />
    ```
    datanode标签定义了mycat中的数据节点，也就是我们所说的数据分片。一个datanode标签就是一个独立的数据分片。
    例子中的表述的意思为，使用名字为localhost1数据库实例上的db1物理数据库，这就组成一个数据分片，最后我们用dn1来标示这个分片。
    * --name	定义数据节点的名字，这个名字需要唯一。我们在table标签上用这个名字来建立表与分片对应的关系
    * --dataHost	用于定义该分片属于哪个数据库实例，属性与datahost标签上定义的name对应
    * --database	用于定义该分片属于数据库实例上 的具体库。


4. dataHost标签

    这个标签直接定义了具体数据库实例，读写分离配置和心跳语句。
    ```xml
    <dataHost name="localhost1" maxCon="1000" minCon="10" balance="0" writeType="0" dbType="mysql" dbDriver="native" switchType="1" slaveThreshold="100">
        <heartbeat>select user()</heartbeat>
        <writeHost host="hostM1" url="192.168.1.100:3306" user="root" password="123456">
            <readHost host="hostS1" url="192.168.1.101:3306" user="root" password="123456" />
        </writeHost>
    </dataHost>
    ```
    * --name	唯一标示dataHost标签，供上层使用
    * --maxCon	指定每个读写实例连接池的最大连接。
    * --minCon	指定每个读写实例连接池的最小连接，初始化连接池的大小
    * --balance	负载均称类型
        * 0 不开启读写分离机制，所有读操作都发送到当前可用的writeHost上
        * 1 全部的readHost与stand by writeHost参与select语句的负载均衡，简单的说，当双主双从模式（M1-S1，M2-S2 并且M1 M2互为主备），正常情况下，M2,S1,S2都参与select语句的负载均衡。
        * 2 所有读操作都随机的在writeHost、readHost上分发
        * 3 所有读请求随机的分发到writeHst对应的readHost执行，writeHost不负担读写压力。（1.4之后版本有）
    * --writeType	负载均衡类型。
        * 0 所有写操作发送到配置的第一个 writeHost，第一个挂了切到还生存的第二个writeHost，重新启动后已切换后的为准，切换记录在配置文件中:dnindex.properties .
        * 1 所有写操作都随机的发送到配置的 writeHost。1.5以后版本废弃不推荐。
    * --switchType	
        * -1 不自动切换
        * 1 默认值 自动切换
        * 2 基于MySql主从同步的状态决定是否切换心跳语句为 show slave status
        * 3 基于mysql galary cluster 的切换机制（适合集群）1.4.1 心跳语句为 show status like 'wsrep%'
    * --dbType	指定后端链接的数据库类型目前支持二进制的mysql协议，还有其他使用jdbc链接的数据库，例如：mongodb，oracle，spark等
    * --dbDriver	指定连接后段数据库使用的driver，目前可选的值有native和JDBC。使用native的话，因为这个值执行的是二进制的mysql协议，所以可以使用mysql和maridb，其他类型的则需要使用JDBC驱动来支持。
    如果使用JDBC的话需要符合JDBC4标准的驱动jar 放到mycat\lib目录下，并检查驱动jar包中包括如下目录结构文件 META-INF\services\java.sql.Driver。 在这个文件写上具体的driver类名，例如com.mysql.jdbc.Driver
    writeHost readHost指定后端数据库的相关配置给mycat，用于实例化后端连接池。
    * --tempReadHostAvailable	
    如果配置了这个属性 writeHost 下面的 readHost 仍旧可用，默认 0 可配置（0、1）。

    1. heartbeat标签 

        这个标签内指明用于和后端数据库进行心跳检查的语句。 
        例如：MYSQL 可以使用 select user()，Oracle 可以使用 select 1 from dual 等。

    2. writeHost /readHost 标签 

        这两个标签都指定后端数据库的相关配置，用于实例化后端连接池。唯一不同的是，writeHost 指定写实例、readHost 指定读实例。 
        在一个 dataHost 内可以定义多个 writeHost 和 readHost。但是，如果 writeHost 指定的后端数据库宕机，那么这个 writeHost 绑定的所有 readHost 都将不可用。
        另一方面，由于这个 writeHost 宕机，系统会自动的检测到，并切换到备用的 writeHost 上去。这两个标签的属性相同，这里就一起介绍。

        * --host	用于标识不同实例，一般 writeHost 我们使用*M1，readHost 我们用*S1。
        * --url	后端实例连接地址。Native：地址：端口 JDBC：jdbc的url
        * --password	后端存储实例需要的密码
        * --user	后端存储实例需要的用户名字
        * --weight	权重 配置在 readhost 中作为读节点的权重
        * --usingDecrypt	是否对密码加密，默认0。具体加密方法看官方文档。

### 三、Rule.xml

    rule.xml 里面就定义了我们对表进行拆分所涉及到的规则定义。我们可以灵活的对表使用不同的分片算法，或者对表使用相同的算法但具体的参数不同。 包含的标签 tableRule 和 function。

1. tableRule 标签

    这个标签定义表规则。 
    定义的表规则，在 schema.xml：
    ```xml
    <tableRule name="rule1">
        <rule>
            <columns>id</columns>
            <algorithm>func1</algorithm>
        </rule>
    </tableRule>
    ```
    * --name 属性指定唯一的名字，用于标识不同的表规则。 内嵌的 rule 标签则指定对物理表中的哪一列进行拆分和使用什么路由算法。 
    * --columns 内指定要拆分的列名字。 
    * --algorithm 使用 function 标签中的 name 属性。连接表规则和具体路由算法。当然，多个表规则可以连接到 同一个路由算法上。table 标签内使用。让逻辑表使用这个规则进行分片。	
2. function 标签
    ```xml
    <function name="hash-int" class="org.opencloudb.route.function.PartitionByFileMap">
        <property name="mapFile">partition-hash-int.txt</property>
    </function>
    ```
    * --name 指定算法的名字。 
    * --class 制定路由算法具体的类名字。 
    * --property 为具体算法需要用到的一些属性。


参考：https://www.cnblogs.com/fxwl/p/7990906.html