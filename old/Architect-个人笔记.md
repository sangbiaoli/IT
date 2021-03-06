## 架构师大纲

1. 源码分析
    1. 常用设计模式
        * Proxy 代理模式
        * Factory 工厂模式
        * Singleton 单例模式
        * Delegate 委派模式
        * Strategy 策略模式
        * Template 模板模式
        * Observer 观察者模式
    2. spring5源码
        * Beans
            * 接口实例化
                * 声明实体bean，这个类有无参构造方法
                * 声明静态工厂及创建实体bean的静态方法
                * 声明实例工厂bean及创建实体bean的方法
                * 定义类实现FactoryBean接口，重写getObject方法和getObjectType
            * 代理Bean操作
        * Context
            * IOC容器设计原理及高级特性
                * Spring的核心:关于bean一切
                    * bean如何定义
                    * bean如何读取（加载），加载资源的方式（文件，类路径，url），解析资源得到bean定义
                    * 由bean的行为，bean之间的关系，bean的自动装配定义出bean工厂

                    bean工厂实现加载，加载解析得到bean

                * BeanFactory
                    BeanFactory是顶级接口，其子类接口有
                    * ListableBeanFactory:可列表
                    * HierarchicalBeanFactory：可集成
                    * AutowireCapableBeanFactory：可自动装配
                    这四个接口定义了Bean的集合，Bean之间的关系，Bean的行为。

                * IOC容器
                    * XmlBeanFactory（屌丝级别）:实现基本的容器的功能
                        两个构造方法：
                        ```java
                        public XmlBeanFactory(Resource resource){
                            super(resource,null);
                        }
                        public XmlBeanFactory(Resource resource,XmlBeanFactory parent){
                            super(parent);
                            this.reader = new XmlBeanDefinitionReader(this);
                            this.reader.loadBeanDefinitions(resource);
                        }
                        ```
                        初始化步骤
                        ```java
                        ClassPathResource  resource = new ClassPathResource("application.xml");
                        DefaultListableBeanFactory  factory = new DefaultListableBeanFactory ();
                        XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
                        reader.loadBeanDefinitions(resource);
                        ```

                    * ApplicationContext(高富帅级别)：除了基本容器功能，还实现新特性

                        * 支持信息源，可以实现国际化。（实现MessageResource接口）
                        * 访问资源。（实现ResourcePatternResolver接口）
                        * 支持应用事件。（实现ApplicationEventPublisher接口）

                        两个构造方法：
                        ```java
                        public ApplicationContext(String[] configLocations){
                            super(configLocations,true,null);
                        }
                        public ApplicationContext(String[] configLocations,boolean refresh,ApplicationContext parent){
                            super(parent);
                            setConfigLocations(configLocations);
                            if(refresh){
                                refresh();
                            }
                        }
                        ```
                * 高级特性
                    * 使用lazy-init属性对Bean预初始化
                    * FactoryBean产生或者修饰Bean对象的生成
                    * IoC容器初始化Bean过程中使用BeanPostProcessor后置处理器对Bean声明周期事件管理
                    * IoC容器的autowiring自动装配功能

            * AOP设计原理
            * FactoryBean与BeanFactory
        * Transaction
            * 声明式是事务底层原理
            * Spring事务处理机制
            * 事务的传播与监控
            * 基于SpringJDBC手写ORM架构
        * MVC
            * MVC原理介绍

                ![](dia/SpringMVC.png)

            * 与IOC容器整合原理
            * HandlerMappering实现原理
            * HanlderAdapter实现原理
            * ViewResolver实现原理
            * Controller调用原理
            * 动态参数匹配原理
            * StringMVC与Struts对比分析
            * 手写实现SpringMVC框架
        * Spring5新特性
            * Spring5.x的兼容性
            * 分析自带通用日志架构
            * 多序列化数据格式绑定API
            * 函数是分隔的ApplicationContext
            * Kotlin表达式的支持
            * WebFlux模块介绍
            * Testing改进
    3. mybatis源码
        * 代码自动生成器：Generator
        * MYbatis下1对多，多对多 嵌套结果、嵌套查询
        * 以及缓存、二级缓存使用场景及选择策略
        * Mybatis与spring集成Spring-mybatis.jar分析
        * Spring集成下的SqlSession与Mapper
        * MyBatis的事务
        * 分析MyBatis的动态代理的真正实现
        * 一步一步手写实现MyBatis 1.0到2.0
2. 分布式架构
    1. 漫谈分布式架构
        * 初识分布式架构及意义
        * 如何把应用从单机扩展到分布式
        * 大型分布式架构演进过程
        * 构建分布式架构最重要因素
            * CDN加速静态文件访问
            * 分布式存储
            * 分布式搜索引擎
            * 应用发布与监控
            * 应用容灾及机房规划
            * 系统动态扩容
        * 分布式架构设计
            * 主流架构模型-SOA机构和微服务架构
            * 领域驱动设计及业务驱动划分
            * 分布式建构的基础理论CAP、BASE以及其应用
            * 什么是分布式架构下的高可用设计
            * 分布式架构下的可伸缩设计
            * 构建高性能的分布式架构
    2. 分布式架构策略-分而治之
        * 从简到难，从网络通信探究分布式通信的原理
        * 基于消息方式的系统间通信
        * 理解通信协议创术过程中的序列化和反序列化机制
        * 基于框架的RPC通信技术
            * Webservice/Apache CXF
            * RMI/Spring RMI
            * Hessian
        * 传统RPC技术在大型分布式架构下面临的问题
        * 分布式架构的RPC解决方案
        * 分布式系统的基石
            * 从0开始搭建3个节点的zookeeper集群
            * 深入分析Zookeeper在disconf配置中心的应用
            * 基于Zookeeper的分布式锁解决方案
            * Zookeeper Watcher核心机制深入源码分析
            * Zookeeper集群升级、迁移
            * 基于Zookeeper实现分布式服务器动态上下线感知
            * 深入分析Zookeeper Zab协议及选举机制源码解读
        * 使用Dubbo对单一应用服务化改造        
            * Dubbo管理中心及监控平台安装部署
            * Dubbo分布式服务模块划分（领域驱动）
            * 基于Dubbo负载均衡策略分析
            * Dubbo服务调试之服务只订阅及服务只注册配置
            * Dubbo服务接口的设计原则（实战经验分享）
            * Dubbo设计原理及源码分析
            * 基于Dubbo构建大型分布式电商平台实战雏形
            * Dubbo容错机制及高扩展性分析
    3. 分布式架构-中间件
        * 分布式消息通信      
            * 消息中间件在分布式架构中的应用
            * ActiveMQ
                * ActiveMQ搞可用集群企业级部署方案
                * ActiveMQ P2P及PUB/SUB模型详解
                * ActiveMQ消息确认及重发策略
                * ActiveMQ基于Spring完成分布式消息队列实战
            * kafka
                * Kafka基于Zookeeper搭建高可用集群实战
                * Kafka消息处理过程剖析
                * Java客户端实现Kafka生产者和消费者实例
                * Kafka的副本机制及选举原理剖析
                * 基于Kafka实现应用日志实时上报统计分析
            * RabbitMQ
                * 初步认识RabbitMQ及高可用群集部署
                * 详解RabbitMQ消息分发机制及主题消息分发
                * RabbitMQ消息路由机制分析
                * RabbitMQ消息确认机制
        * 分布式缓存-redis
            * 从入门到精通，Redis的数据结构分析
            * Redis主从复制原理及无磁盘复制分析
            * Redis管道模式详解
            * Redis缓存与数据库已执行问题解决方案
            * 基于Redis实现分布式锁实战
            * 图解Redis中AOF和RDB持久化策略的原理
            * Redis读写分析架构实战
            * Redis哨兵结构及数据丢失问题分析
            * Redis Cluster数据分布算法之Hash slot
            * Redis使用常见问题及性能优化思路
            * Redis高可用及搞伸缩架构实战
            * 缓存击穿、缓存雪崩预防策略
            * Redis批量查询优化
            * Redis高性能集群值twemproxy or codis
        * 数据存储
            * mongoDB
                * NoSQL简介及MongoDB基本概念
                * MongoDB支持的数据类型分析
                * MongoDB可视化客户端及Java API实战
                * 手写基于MongoDB的ORM框架
                * MongoDB企业级集群解决方案
                * MongoDB聚合、索引及基本执行命令
                * MongoDB数据分片、转存及恢复策略
            * Mycat
                * MySQL主从复制及读写分离实战
                * MySQL+keepalived实现双柱高可用方案实战
                * MySQL高性能解决方案之分库分表
                * 数据中间件初识Mycat
                * 基于Mycat实现mysql数据库读写分离
                * 基于Mycat实战值数据库切分策略剖析
                * Mycat全局表、ER表、分片策略分析
        * 后台服务-NGINX
            * 基于OpenResty部署应用层Nginx以及Nginx+lua实战
            * Nginx反向代理服务器及负载均衡服务配置实战
            * 利用keepalived+Nginx实现Nginx高可用方案
            * 基于Nginx实现访问控制、链接限制
            * Nginx动静分离实战
            * Nginx Location、Rewrite等语法配置及原理分析
            * Nginx提供https服务
            * 基于Nginx+lua完成访问流量实时上报kafka的实战
        * 高性能NIO框架-Netty
            * IO的基本概念、NIO、AIO、BIO深入分析
            * NIO的核心设计思想
            * Netty产生的背景及应场景分析
            * 基于Netty实现高性能IM聊天
            * 基于Netty实现Dubbo多协议通信支持
            * Netty无锁化串行设计及高并发机制
            * 手写实现多协议RPC框架        
    4. 分布式解决方案
        * 分布式全局ID生成方案
        * session跨域共享及企业级单点登录解决方案实战
        * 分布式事务解决方案实战
        * 高并发下的服务降级、限流实战
        * 基于分布式架构下分布式锁的解决方案实战
        * 分布式架构下实现分布式定时调度
3. 微服务架构
    1. Spring Boot
        * Spring Boot与微服务之间的关系
        * Spring Boot热部署实战
        * 核心组件之starter、actuator、auto-configuation、cli
        * Spring Boot集成Mybatis实现多数据源路由实战
        * Spring Boot集成Dubbo实战
        * Spring Boot集成Redis缓存实战
        * Swagger与Spring Boot集成构建API管理及测试体系
        * Spring Boot实现多环境配置动态解析
    2. Spring Cloud
        * Eureka注册中心
        * Ribbon集成REST实现负载均衡
        * Feign声明式服务调用
        * Hystrix服务熔断降级方式
        * Zuul实现微服务网关
        * Config分布式统一配置中心
        * Sleuth调用链路跟踪
        * BUS消息总线
        * 基于Hystrix实现接口降级实战
        * Spring Boot集成Spring Cloud实现统一整合方案
    3. docker虚拟化
        * 了解Docker的景象、仓库、容器
        * Dockerfile构建LNMP环境部署个人博客wordpress
        * Docker Compose构建LNMP环境部署个人博客wordpress
        * Docker网络组成、路由互联、openvswitch
        * 基于swarm构建Docker集群实战
        * Kubernets简介
    4. 漫谈微服务架构
        * SOA架构和微服务架构之间的区别和联系
        * 如何设计微服务及其设计原则
        * 解惑Spring Boot流行原因及能够解决什么问题
        * 什么是Spring Cloud，为何要选择Spring Cloud
        * 基于全局分析Spring Cloud各个组件所解决的问题
4. 并发编程
    1. Java内存模型(JMM)
        * 线程通信
        * 消息传递      
    2. 内存模型
        * 重排序
        * 顺序一致性
        * happens-before
        * as-if-serial
    3. synchronized
        * 同步、重量级锁
        * synchronized原理
        * 锁优化
            * 自旋锁
            * 轻量级锁
            * 重量级锁
            * 偏向锁
    4. volatitle
        * volatitle实现机制
        * 内存语义
        * 内存模型
    5. DCL
        * 单例模式
        * DCL
        * 解决方案
    6. 并发基础
        * AQS
            * AbstractQueuedSynchronizer同步器 
            * CLH同步队列
            * 同步状态的获取和释放
            * 线程阻塞和唤醒
        * CAS
            * Compare And Swap
            * 缺陷
    7. 锁
        * ReentrantLock
        * ReentrantReadWriteLock
        * Condition
    8. 并发工具类
        * CyclicBarrier
        * CountDownLatch
        * Semphore
    9. 并发集合
        * ConcurrentHashMap
        * ConcurrentLinkedQueue
    10. 原子操作
        * 基本类型
            * AtomicBoolean
            * AtomitInteger
            * AtomicLong
        * 数组
            * AtomicIntegerArray
            * AtomicLongArray
            * AtomicReferenceArray
        * 引用类型
            * AtomicReference 
            * AtomicReferenceFiledUpdater
    11. 线程池
        * Executor
        * ThreadPoolExecutor
        * Callable和Future
        * ScheduledExecutorService
    12. 其他
        * ThreadLocal
        * Fork/Join
5. 性能优化
    1. 理解性能优化
        * 性能基准
        * 性能优化到底是什么
        * 衡量维度
    2. JVM调优篇
        * 知其然，知其所以然
        * 什么是JVM运行时数据区
        * 什么是JVM内存模型JMM
        * 各垃圾回收器使用场景（Throughput\CMS）
        * 理解GC日志，从日志看端倪
        * 实战MAT分析dump文件
    3. Tomcat调优篇
        * How is works?探查Tomcat的运行机制及框架
        * 分析Tomcat线程模型
        * Tomcat系统参数认识及调优
        * 基准测试
    4. MySQL调优篇
        * 理解MySQL底层B+Tree机制
        * SQL执行计划详解
        * 索引优化详解
        * SQL语句优化
6. DevOps
    1. Maven
        * 生成可执行jar、理解scope生成最精确的jar
        * 解决类冲突、宝依赖、NoClassDefFoundError问题定位及解决
        * 全面理解Maven的LifeCycle\Phase\Goal
        * 架构师必备之Maven生成Archetype
        * Maven流行插件实战、手写自己的插件
        * Nexus使用、上传、配置
        * 对比Gradle
    2. Jenkins
        * 持续集成，一次build解决所有手动工作
    3. sonarqube
        * 减少人为疏忽，静态代码检查，让你的代码更健壮
    4. git
        * 什么是git以及git的工作原理
        * git常用命令best practise(避坑教学)
        * git冲突怎么引起的，如何解决
        * 架构师职责：git flow规范团队git使用规程
        * 团队案例分享（买不到带才是最贵的）
    5. 敏捷开发
        * 敏捷的由来
            * 传统模式的问题
            * 当前行业面临的问题
            * 微服务与敏捷
        * 敏捷开发模式
            * 敏捷组织架构
            * 敏捷最佳实战
                * TDD
                * 绝对编程
                * CI
            * 敏捷与DevOps
        * 敏捷开发实战
            * 用户故事 
            * 看板
            * 每日站会
            * 迭代冲刺
            * 回顾会议
7. 电商项目实战
    1. 用户认证系统(passport)
        * 用户注册
        * 用户登录
            * SSO单点登录
            * 第三方登录
        * 用户权限控制
            * UI页面拦截
            * 业务方法拦截
    2. 搜索模块(大数据)
        * 大数据存储
            * 分布式环境配置
            * Hadoop基本配置介绍
        * 大数据检索
            * ElasticSearch环境配置
            * ElasticSearch的API使用
        * 动静分离
    3. 商品管理系统(item)
        * 店铺管理
            * 创建店铺
            * 店铺主页指定
        * 商品管理
            * 商品录入
            * 商品预览
    4. 订单系统(order)
        * 订单号统一生成规则
        * 下单流程管理
        * 库存管理
        * 购物车
            * 购物车管理
            * 未登录状态下的购物车同步
    5. 支付系统(pay)
        * 优惠券支付
        * 积分支付
        * 金融支付
            * 微信支付
            * 支付宝支付
            * 银联支付
    6. 数据统计分析系统(anal)
        * 用户行为分析
            * 用户兴趣分析
            * 登录异常分析
        * 行业分析
        * 区域分析
    7. 通知推送系统
        * 融云推送
            * 活动推送
            * 交易信息推送
            * 异常提醒
        * 消息中间件
            * 消息同步
            * 消息处理
    8. 聊天系统(im)
        * 用户群聊
        * 点对点聊天
        * 文件断点续传