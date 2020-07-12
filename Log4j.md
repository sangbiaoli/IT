## SLF4J和log4j的使用

1. 概念

    **SLF4J**：即简单日志门面（**Simple Logging Facade for Java**），不是具体的日志解决方案，它只服务于各种各样的日志系统。按照官方的说法，SLF4J是一个用于日志系统的简单Facade，允许最终用户在部署其应用时使用其所希望的日志系统。

    在使用SLF4J的时候，不需要在代码中或配置文件中指定你打算使用那个具体的日志系统，SLF4J提供了统一的记录日志的接口，只要按照其提供的方法记录即可，最终日志的格式、记录级别、输出方式等通过具体日志系统的配置来实现，因此可以在应用中灵活切换日志系统。

    官方网站：http://www.slf4j.org/


    **log4j**：Log For Java，Apache的一个开源项目，可以灵活地记录日志信息，我们可以通过Log4j的配置文件灵活配置日志的记录格式、记录级别、输出格式，而不需要修改已有的日志记录代码。

    官方网站：http://logging.apache.org/log4j/1.2/


2. 使用介绍

    下面是一个log4j配置示例：

    ```properties
    # 日志输出级别（INFO）和输出位置（stdout，R）
    log4j.rootLogger=INFO, stdout , R

    # 日志输出位置为控制台
    log4j.appender.stdout=org.apache.log4j.ConsoleAppender
    log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
    log4j.appender.stdout.layout.ConversionPattern=[QC] %p [%t] %C.%M(%L) | %m%n

    # 日志输出位置为文件
    log4j.appender.R=org.apache.log4j.DailyRollingFileAppender
    log4j.appender.R.File=D:\\Tomcat 5.5\\logs\\qc.log
    log4j.appender.R.layout=org.apache.log4j.PatternLayout
    log4j.appender.R.layout.ConversionPattern=%d-[TS] %p %t %c - %m%n

    # 定义相应包路径下的日志输出级别
    log4j.logger.com.alibaba=DEBUG
    log4j.logger.com.opensymphony.oscache=ERROR
    log4j.logger.org.springframework=DEBUG
    log4j.logger.com.ibatis.db=WARN
    log4j.logger.org.apache.velocity=FATAL
    
    log4j.logger.org.hibernate.ps.PreparedStatementCache=WARN
    log4j.logger.org.hibernate=DEBUG
    log4j.logger.org.logicalcobwebs=WARN
    ```


    说明：

    1. log4j.rootCategory=INFO, stdout , R

        此句为将等级为INFO的日志信息输出到stdout和R这两个目的地，stdout和R的定义在下面的代码，可以任意起名。等级可分为
        * OFF
        * FATAL
        * ERROR
        * WARN
        * INFO
        * DEBUG
        * ALL
        
        如果配置OFF则不打出任何信息，如果配置为INFO这样只显示INFO, WARN, ERROR的log信息，而DEBUG信息不会被显示，具体讲解可参照第三部分定义配置文件中的logger。

    2. log4j.appender.stdout=org.apache.log4j.ConsoleAppender

        此句为定义名为stdout的输出端是哪种类型
        * org.apache.log4j.ConsoleAppender（控制台）
        * org.apache.log4j.FileAppender（文件）
        * org.apache.log4j.DailyRollingFileAppender（每天产生一个日志文件）
        * org.apache.log4j.RollingFileAppender（文件大小到达指定尺寸的时候产生一个新的文
        * org.apache.log4j.WriterAppender（将日志信息以流格式发送到任意指定的地方）

    3. log4j.appender.stdout.layout=org.apache.log4j.PatternLayout

        此句为定义名为stdout的输出端的layout是哪种类型
        * org.apache.log4j.HTMLLayout（以HTML表格形式布局）
        * org.apache.log4j.PatternLayout（可以灵活地指定布局模式）
        * org.apache.log4j.SimpleLayout（包含日志信息的级别和信息字符串）
        * org.apache.log4j.TTCCLayout（包含日志产生的时间、线程、类别等等信息）

    4. log4j.appender.stdout.layout.ConversionPattern= [QC] %p [%t] %C.%M(%L) | %m%n

        如果使用pattern布局就要指定的打印信息的具体格式ConversionPattern，打印参数如下：
        * %m 输出代码中指定的消息；
        * %M 输出打印该条日志的方法名；
        * %p 输出优先级，即DEBUG，INFO，WARN，ERROR，FATAL；
        * %r 输出自应用启动到输出该log信息耗费的毫秒数；
        * %c 输出所属的类目，通常就是所在类的全名；
        * %t 输出产生该日志事件的线程名；
        * %n 输出一个回车换行符，Windows平台为"rn”，Unix平台为"n”；
        * %d 输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，比如：%d{yyyy-MM-dd HH:mm:ss,SSS}，输出类似：2002-10-18 22:10:28,921；
        * %l 输出日志事件的发生位置，及在代码中的行数；
        
        [QC]是log信息的开头，可以为任意字符，一般为项目简称。输出示例[TS] DEBUG [main] AbstractBeanFactory.getBean(189) | Returning cached instance of singleton bean 'MyAutoProxy'

3. 使用步骤

    1. 在JavaWeb项目中使用SLF4J和LOG4J，需要在项目中添加下面三个jar包

        1. log4j-1.2.x.jar

        2. slf4j-api-1.x.x.jar

        3. slf4j-log4j12-1.x.x.jar

        x-具体版本号

    2. 在项目类路径下添加log4j.properties配置文件，具体内容参照上面的示例。

    3. 在项目的web.xml配置文件中添加加载log4j的配置

        ```xml
        <!-- log4j配置文件位置 -->
        <context-param>
            <param-name>log4jConfigLocation</param-name>
            <param-value>classpath:log4j.properties</param-value>
        </context-param>

        <!-- 利用spring来使用log4j -->
        <listener>
            <listener-class>org.springframework.web.util.Log4jConfigListener</listener-class>
        </listener>
        ```

    4. 在需要输出日志的类中添加slf4j的logger实例对象：

        ```java
        // 导入slf4j类
        import org.slf4j.Logger;
        import org.slf4j.LoggerFactory;

        // 添加slf4j日志实例对象
        final static Logger logger = LoggerFactory.getLogger(Test.class);

        // 输出日志
        logger.info("测试：{}", "输出日志");
        ```

    5. 启动并运行项目。

原文：https://www.cnblogs.com/haoqipeng/p/5300376.html