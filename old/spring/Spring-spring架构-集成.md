## spring架构集成

1. 使用Spring的远程处理和Web服务

    1. 通过使用RMI公开服务
    2. 使用Hessian通过HTTP远程调用服务
    3. 使用HTTP调用程序公开服务
    4. Web服务
    5. 通过JMS公开服务
    6. AMQP
    7. 选择技术时的考虑因素
    8. REST端点

2. 企业javabean (EJB)集成

    1. 访问ejb

3. JMS (Java消息服务)

    1. 使用Spring JMS
    2. 发送消息
    3. 接收消息
    4. 支持JCA消息端点
    5. 注解驱动的端点侦听器
    6. JMS名称空间支持

4. JMX

    1. 将bean导出到JMX
    2. 控制bean的管理接口
    3. 控制bean的ObjectName实例
    4. 使用jsr - 160连接器
    5. 通过代理访问mbean
    6. 通知
    7. 进一步的资源

5. JCA CCI

    1. 配置CCI
    2. 使用Spring的CCI访问支持
    3. 将CCI访问建模为操作对象
    4. 交易

6. 电子邮件

    1. 使用
    2. 使用JavaMail MimeMessageHelper

7. 任务执行和调度

    1. Spring TaskExecutor抽象
    2. Spring任务调度程序抽象
    3. 注释支持调度和异步执行
    4. 任务名称空间
    5. 使用Quartz调度程序

8. 缓存的抽象

    1. 理解缓存抽象
    2. 声明基于注解的缓存
    3. JCache (jsr - 107)注释
    4. 声明性xml缓存
    5. 配置缓存存储
    6. 插入不同的后端缓存
    7. 如何设置TTL/TTI/驱逐令策略/XXX特性?
