## kafka的应用

本文使用springboot来搭建应用，实现发送消息到broker及从broker中读取消息。

1. 环境设置

    搭建应用之前，先把zookeeper和kafka启动起来，在机器192.168.18.131上部署了，配置如下
    
    应用|ip|端口|
    --|--|--|
    zookeeper|192.168.18.131|2181
    kafka|192.168.18.131|9092

    并且在broker上创建了名为test的topic。
    


2. 发送消息应用搭建

    整个应用的运行目的：每5s生成一行随机字符串，并发送消息test主题消息到broker。
    
    1. 新建maven项目kafka-producer

    2. pom.xml文件添加配置

        ```xml
        <parent>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>1.5.9.RELEASE</version>
        </parent>
        <properties>
            <java.version>1.8</java.version>
            <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        </properties>
        <dependencies>
            <dependency>
                <groupId>org.springframework.kafka</groupId>
                <artifactId>spring-kafka</artifactId>
            </dependency>          
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
            </dependency>

            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <scope>test</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
            </dependency>
        </dependencies>
        ```

    3. application.properties

        ```
        server.port=8090
        spring.kafka.bootstrap-servers=192.168.18.131:9092
        ```

    4. java文件

        * **KafkaProducer.java**
        
            ```java
            @Component
            @EnableScheduling
            public class KafkaProducer {
                private final String TOPIC_NAME = "test";

                @Autowired
                private KafkaTemplate<?, String> kafkaTemplate;

                public void send() {
                    String message = LangUtils.randomWords();
                    ListenableFuture<?> future = kafkaTemplate.send(TOPIC_NAME, message);
                    future.addCallback(
                        o -> System.out.println("send-消息发送成功：" + message),
                        throwable -> System.out.println("消息发送失败：" + message)
                    );
                }

            }
            ```

        * **LangUtils.java**

            ```java
            public class LangUtils {
                private final static String LETTERS = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";

                public static String randomWords() {
                    StringBuilder words = new StringBuilder();
                    int length = 1 + (int) (Math.random() * 5);
                    for (int i = 0; i < length; i++) {
                        words.append(randomWord()+" ");
                    }
                    return words.toString();
                }
                
                /**
                * 随机获取长度为1~10的大小写字母混杂的“单词”
                */
                private static String randomWord() {		
                    StringBuilder word = new StringBuilder();
                    int length = 1 + (int) (Math.random() * 10);
                    for (int i = 0; i < length; i++) {
                        int index = (int) (Math.random() * LETTERS.length());
                        word.append(LETTERS.charAt(index));
                    }
                    return word.toString();
                }
                
                public static void main(String[] args) {
                    for(;;){
                        System.out.println(randomWords());
                    }
                }
            }
            ```

        * **SpringUtil.java**

            ```java
            import org.springframework.beans.BeansException;
            import org.springframework.context.ApplicationContext;
            import org.springframework.context.ApplicationContextAware;
            import org.springframework.stereotype.Component;

            @Component
            public class SpringUtil implements ApplicationContextAware {

                private static ApplicationContext applicationContext;

                @Override
                public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
                    if(SpringUtil.applicationContext == null) {
                        SpringUtil.applicationContext = applicationContext;
                    }
                }

                //获取applicationContext
                public static ApplicationContext getApplicationContext() {
                    return applicationContext;
                }

                //通过name获取 Bean.
                public static Object getBean(String name){
                    return getApplicationContext().getBean(name);
                }

                //通过class获取Bean.
                public static <T> T getBean(Class<T> clazz){
                    return getApplicationContext().getBean(clazz);
                }

                //通过name,以及Clazz返回指定的Bean
                public static <T> T getBean(String name,Class<T> clazz){
                    return getApplicationContext().getBean(name, clazz);
                }

            }
            ```

        * **Application.java**

            ```java
            import java.util.concurrent.TimeUnit;

            import org.springframework.boot.SpringApplication;
            import org.springframework.boot.autoconfigure.SpringBootApplication;

            import com.sangbill.kafka.producer.service.KafkaProducer;
            import com.sangbill.kafka.util.SpringUtil;

            @SpringBootApplication
            public class Application {

                public static void main(String[] args) {

                    SpringApplication.run(Application.class, args);
                    System.out.println("启动kafka-consumer成功");
                    sendMsg();
                    
                }

                private static void sendMsg() {
                    KafkaProducer producer = SpringUtil.getBean(KafkaProducer.class);
                    for (;;) {
                        try {
                            producer.send();
                            TimeUnit.SECONDS.sleep(5);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    
                }
            }
            ```

3. 接收消息应用搭建

    整个应用的运行目的：监听broker上test主题消息。

    1. 新建maven项目kafka-consumer

    2. pom.xml文件添加配置(跟上面一样)

    3. application.properties

        ```
        server.port=8091
        spring.kafka.bootstrap-servers=192.168.18.131:9092
        spring.kafka.consumer.group-id=etl
        spring.kafka.consumer.auto-offset-reset=earliest
        spring.kafka.consumer.auto-commit-interval=1000
        spring.kafka.consumer.enable-auto-commit=true
        ```

    4. java文件

        * **KafkaConsumer**

            ```java
            @Component
            public class KafkaConsumer {
            
                /**
                * 实时获取kafka数据(生产一条，监听生产topic自动消费一条)
                * @param record
                * @throws IOException
                */
                @KafkaListener(topics = "test")
                public void receive(String content){
                    System.err.println("Receive:" + content);
                }

            }
            ```

        * **Application**

            ```java
            import org.springframework.boot.SpringApplication;
            import org.springframework.boot.autoconfigure.SpringBootApplication;

            @SpringBootApplication
            public class Application {

                public static void main(String[] args) {
                    SpringApplication.run(Application.class, args);
                    System.out.println("启动kafka-consumer成功");
                }
            }
            ```

