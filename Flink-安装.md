## Flink安装

1. 下载并启动Flink

    1. 安装环境

        Flink可以安装在Linux，Mac OS X和Windows，当前所用系统为CentOS7(64bit)。
        
        为了能够启动Flink，主要安装Java 8.x以上版本的jdk，可以通过命令查看jdk版本。
        
        ```bash
        java -version
        ```
        
        如果已经安装了Java 8，则输出可能如下

        ```bash
        java version "1.8.0_111"
        Java(TM) SE Runtime Environment (build 1.8.0_111-b14)
        Java HotSpot(TM) 64-Bit Server VM (build 25.111-b14, mixed mode)
        ```

    2. 下载并编译

        ```bash
        git clone https://github.com/apache/flink.git
        $ cd flink
        $ mvn clean package -DskipTests # 这个执行将要花费10min
        $ cd build-target               # Flink所安装的位置
        ```

    3. 启动一个本地的Flink Cluster

        ```bash
        $ ./bin/start-cluster.sh  # 启动Flink
        ```

        访问地址：http://localhost:8081/，正常的话可以看到界面，如图

        ![](flink/flink-install-visit.png)

        通过查看log目录下的日志验证系统正在运行

        ```
        $ tail log/flink-*-standalonesession-*.log
        INFO ... - Rest endpoint listening at localhost:8081
        INFO ... - http://localhost:8081 was granted leadership ...
        INFO ... - Web frontend listening at http://localhost:8081.
        INFO ... - Starting RPC endpoint for StandaloneResourceManager at akka://flink/user/resourcemanager .
        INFO ... - Starting RPC endpoint for StandaloneDispatcher at akka://flink/user/dispatcher .
        INFO ... - ResourceManager akka.tcp://flink@localhost:6123/user/resourcemanager was granted leadership ...
        INFO ... - Starting the SlotManager.
        INFO ... - Dispatcher akka.tcp://flink@localhost:6123/user/dispatcher was granted leadership ...
        INFO ... - Recovering all persisted jobs.
        INFO ... - Registering TaskManager ... under ... at the SlotManager.
        ```

2. 应用程序

    Flink支持Scala和Java。以Java为例构建一个Maven项目。

    1. 用IDE，比如eclipse新建一个maven项目

    2. 添加flink的依赖配置，打开pom.xml，添加依赖

        ```xml
        <properties>
            <flink.version>1.7.2</flink.version>
        </properties>

        <dependencies>
            <dependency>
                <groupId>org.apache.flink</groupId>
                <artifactId>flink-java</artifactId>
                <version>${flink.version}</version>
            </dependency>
            <dependency>
                <groupId>org.apache.flink</groupId>
                <artifactId>flink-streaming-java_2.11</artifactId>
                <version>${flink.version}</version>
            </dependency>
            <dependency>
                <groupId>org.apache.flink</groupId>
                <artifactId>flink-clients_2.11</artifactId>
                <version>${flink.version}</version>
            </dependency>
            <dependency>
                <groupId>org.apache.flink</groupId>
                <artifactId>flink-connector-wikiedits_2.11</artifactId>
                <version>${flink.version}</version>
            </dependency>
        </dependencies>
        ```

    3. 新建SocketWindowWordCount类

        ```java   
        import org.apache.flink.api.common.functions.FlatMapFunction;
        import org.apache.flink.api.common.functions.ReduceFunction;
        import org.apache.flink.api.java.utils.ParameterTool;
        import org.apache.flink.streaming.api.datastream.DataStream;
        import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
        import org.apache.flink.streaming.api.windowing.time.Time;
        import org.apache.flink.util.Collector;

        public class SocketWindowWordCount {

            public static void main(String[] args) throws Exception {

                //连接的端口
                final int port;
                try {
                    final ParameterTool params = ParameterTool.fromArgs(args);
                    port = params.getInt("port");
                } catch (Exception e) {
                    System.err.println("No port specified. Please run 'SocketWindowWordCount --port <port>'");
                    return;
                }

                // 获取执行环境
                final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

                // 通过连接到socket获取输入数据
                DataStream<String> text = env.socketTextStream("localhost", port, "\n");

                //解析数据，对数据进行分组、打开窗口并聚合计数
                DataStream<WordWithCount> windowCounts = text.flatMap(new FlatMapFunction<String, WordWithCount>() {
                    @Override
                    public void flatMap(String value, Collector<WordWithCount> out) throws Exception {
                        for (String word : value.split("\\s")) {
                            out.collect(new WordWithCount(word, 1L));
                        }
                    }
                }).keyBy("word").timeWindow(Time.seconds(5), Time.seconds(1)).reduce(new ReduceFunction<WordWithCount>() {
                    @Override
                    public WordWithCount reduce(WordWithCount a, WordWithCount b) {
                        return new WordWithCount(a.word, a.count + b.count);
                    }
                });

                //使用单个线程而不是并行打印结果
                windowCounts.print().setParallelism(1);

                env.execute("Socket Window WordCount");
            }

            //WordWithCount的数据类型
            public static class WordWithCount {

                public String word;
                public long count;

                public WordWithCount() {}

                public WordWithCount(String word, long count) {
                    this.word = word;
                    this.count = count;
                }

                @Override
                public String toString() {
                    return word + " : " + count;
                }
            }
        }
        ```
    


原文：https://ci.apache.org/projects/flink/flink-docs-master/tutorials/local_setup.html