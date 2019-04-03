## kafka命令

kafka的bin目录下有常用的脚本，下面对这些脚本解释和使用

1. kafka-topics.sh

    * --alter : 更改主题的分区数、复制分配
    * --bootstrap-server <String: server to connect to> : 要连接的broker
    * --command-config <String: command config property file> : 包含配置属性的文件传递给Admin客户端
    * --config <String: name=value> : 正在创建或更改的主题的配置覆盖
    * --create  :创建新主题              
    * --delete  :删除主题
    * --delete-config <String: name> : 为已存在的主题删除的主题配置覆盖
    * --describe : 根据给定的主题列出详细
    * --force : 抑制控制台提示             
    * --help  : 打印用法           
    * --if-exists : 如果在更改、删除或描述主题时设置，则仅当主题存在时才执行操作。                      
    * --if-not-exists : 如果在更改、删除或描述主题时设置，则仅当主题不存在时才执行操作。                       
    * --list : 列出所有可用主题  
    * --partitions <Integer: # of partitions> : 正在创建或更改的主题的分区数          
    * --replica-assignment <String:broker_id_for_part1_replica1 : broker_id_for_part1_replica2 ,broker_id_for_part2_replica1 : broker_id_for_part2_replica2 , ...> : 正在创建或更改的主题手动分区到代理的分配列表
    * --replication-factor <Integer:replication factor> : 正在创建的主题中的每个分区的复制因子       
    * --topic <String: topic> | 要创建、更改、描述或删除的主题

    1. 创建Topic

        创建一个名为“test”的主题，只有一个分区和一个副本

        ```bash
        bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test
        ```

    2. 查看Topic

        查看主题列表，只显示了名称

        ```bash
        bin/kafka-topics.sh --list --bootstrap-server localhost:9092

        abc
        test
        ```
        
        查看主题列表详情

        ```bash
        bin/kafka-topics.sh --describe --bootstrap-server localhost:9092

        Topic:test      PartitionCount:1        ReplicationFactor:1     Configs:segment.bytes=1073741824
        Topic: test     Partition: 0    Leader: 0       Replicas: 0     Isr: 0
        Topic:abc       PartitionCount:1        ReplicationFactor:1     Configs:segment.bytes=1073741824
        Topic: abc      Partition: 0    Leader: 0       Replicas: 0     Isr: 0
        ```

        * PartitionCount：partition个数 
        * ReplicationFactor：副本个数 
        * Partition：partition编号，从0开始递增 
        * Leader：当前partition起作用的breaker.id 
        * Replicas: 当前副本数据所在的breaker.id，是一个列表，排在最前面的其作用 
        * Isr：当前kakfa集群中可用的breaker.id列表

    3. 删除Topic

        删除名为test的主题

        ```bash
        bin/kafka-topics.sh --delete --bootstrap-server localhost:9092 --topic test
        ```

    4. 修改Topic

2. kafka-console-producer.sh

    * --batch-size <Integer: size> ：在单个批处理中一次发送消息的数量，默认200      
    * --broker-list <String: broker-list> : 必填项，Broker列表，格式如 HOST1:PORT1,HOST2:PORT2.    
    * --compression-codec [String: compression-codec]:压缩编码，可以是'none', 'gzip', 'snappy', 'lz4', 或 'zstd'，如果没设定，默认是'gzip'                
    * --help : 打印用法      
    * --socket-buffer-size <Integer: size> ：接收缓存的大小，(default: 102400)          
    * --sync :如果向代理发送的设置message请求是同步的，则在它们到达时一次发送一个请求。           
    * --timeout <Integer: timeout_ms> :如果设置了且producer在异步下运行，这将为消息排队等待足够的批大小提供最大时间量，(default: 1000) 
    * --topic <String: topic> ：必填，主题id        

    1. 发送消息

        从标准输入中输入文本，发送主题为test的消息

        ```bash
        bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
        This is a message
        This is another message
        ^C
        ```

3. kafka-console-consumer.sh       

    * --bootstrap-server <String: server toconnect to>: 要连接的broker                                                                      
    * --consumer-property <String: consumer_prop>：传递用户定义的属性，格式为key=value                    
    * --formatter <String: class> ：用来格式化kafka消息的类名，(default: kafka.tools.DefaultMessageFormatter)             
    * --from-beginning : 如果使用者还没有建立要使用的偏移量，则从日志中出现的最早消息开始，而不是从最新消息开始。                         
    * --group <String: consumer group id> :消费者的group id。
    * --help  ：打印帮助文档       
    * --isolation-level <String>：隔离级别，设置read_committed则过滤掉还没提交的事务性消息，设置read_uncommittedto则读取所有消息(default:read_uncommitted)
    * --key-deserializer <String: deserializer for key> :反序列化key                                                           
    * --max-messages <Integer: num_messages> ：消费退出前的最大消息数，如果没设置，则消费会一直继续，等待。                  
    * --partition <Integer: partition> :从哪个partition开始消费    
    * --timeout-ms <Integer: timeout_ms> ：如果设置了，在固定的周期之内没有可消费的消息，则退出                
    * --topic <String: topic> ：消费的主题                       
    * --whitelist <String: whitelist> ：正则表达式，指定要包含供使用主题的白名单。   

    1. 接收消息

        从开始读取主题为test的消息，并标准输出，如果没有按下退出键，则一直等待消息

        ```bash
        bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
        This is a message
        This is another message
        ^C
        ```   

原文：http://kafka.apache.org/quickstart
https://blog.csdn.net/qq_29116427/article/details/80202392
