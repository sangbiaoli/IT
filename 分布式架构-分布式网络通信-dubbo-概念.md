## dubbo概念

1. 背景

    随着Internet的快速发展，web应用程序的规模不断扩大，最终我们发现传统的垂直体系结构(单片)已经无法处理这种情况。分布式服务体系结构和流计算体系结构势在必行，迫切需要一个治理系统来保证体系结构的有序演化。

    ![](dubbo/dubbo-concept-architecture-roadmap.jpg)


    * 整体架构

        当通信量非常低时，只有一个应用程序，所有的特性都部署在一起，以减少部署节点和成本。此时，数据访问框架(ORM)是简化CRUD工作负载的关键。

    * 垂直架构

        当通信量增大时，添加单片应用程序实例不能很好地加快访问速度，提高效率的一种方法是将单片应用程序分割成离散的应用程序。此时，用于加速前端页面开发的Web框架(MVC)是关键。

    * 分布式服务架构

        当演变出越来越多的垂直应用程序,应用程序之间的交互是不可避免的,一些核心业务提取并担任独立的服务,逐渐形成了一个稳定的服务中心,这样前端应用程序可以更快地应对多变的市场需求。此时，用于业务重用和集成的分布式服务框架(RPC)是关键。

    * 流计算架构

        当服务越来越多时，能力评价就变得困难，而且规模较小的服务往往造成资源浪费。为了解决这些问题，需要增加一个调度中心来管理基于流量的集群容量，提高集群的利用率。此时，用于提高机器利用率的资源调度和治理中心(SOA)是关键。

2. 要求

    ![](dubbo/dubbo-concept-service-governance.jpg)

    在出现大型服务之前，应用程序可能只是通过使用RMI或Hessian公开或引用远程服务，调用是通过配置serive URL完成的，负载平衡是通过硬件完成的，比如F5。 如此抛出一下问题

    1. **随着服务的增多，服务URL的配置变得越来越困难，F5硬件负载均衡器的单点压力也越来越大。**
    
        此时，需要一个服务注册中心来动态注册和发现服务，以使服务的位置透明。

    2. **当事情进一步发展时，服务依赖关系变得如此复杂，以至于它甚至不知道在此之前应该启动哪个应用程序，甚至架构师也不能完全描述应用程序体系结构关系。** 
    
        此时，需要自动绘制应用程序的依赖关系图，以帮助架构师明确关系。

    3. **流量变得更大，暴露了服务的容量问题，需要多少台机器来支持此服务?什么时候添加机器?**

        要解决这些问题，首先应将每天的服务调用和响应时间作为容量规划的参考。其次，动态调整权重，增加在线机器的权重，并记录响应时间的变化，直到达到阈值，记录此时的访问次数，然后将这个访问次数乘以机器总数，依次计算容量。

    以上是Dubbo最基本的要求。

3. dubbo架构

    1. 架构
    
        ![](dubbo/dubbo-concept-architecture.jpg)

        节点|角色说明
        --|--
        Provider|提供者公开远程服务
        Consumer|消费者调用远程服务
        Register|注册中心负责服务发现和配置
        Monitor|监视器计算服务调用的次数和时间
        Container|容器管理服务的生命周期

        图中的服务关系

        0. Container负责启动、加载和运行服务提供者
        1. Provider在它启动时注册服务到注册中心
        2. Consumer启动时从注册中心订阅需要的服务
        3. Register返回提供者列表到消费者，当发生改变时，注册中心会通过长连接推送变动数据到消费者
        4. Consumer根据软负载平衡算法选择其中一个提供者并执行调用，如果失败，它将选择另一个提供者。
        5. Consumer和Provider都将计算服务调用的数量和内存中的耗时，并每分钟将统计数据发送给Monitor。

    2. 连通性

        * Register负责注册和搜索服务地址，如目录服务、提供者和使用者仅在启动时与注册中心交互，而注册中心不转发请求，因此压力较小
        * Monitor负责计算服务调用的数量和时间，统计数据将先在提供者和使用者的内存中组装，然后发送给Monitor
        * Provider注册服务到注册中心，并向“Monitor”报告耗时统计数据(不包括网络开销)
        * Consumer从注册中心获取服务提供者地址列表，根据LB算法直接调用提供者，报告需要监控的耗时统计量，包括网络开销
        * Register, Provider及Consumer之间连接都是长连接，Monitor除外
        * Register通过长连接可以感知到提供者的存在，当提供者宕掉，提供者会给消费者推送事件
        * 即便注册中心和监控中心都宕掉，不影响已经运行的提供者和消费者的实例，因为消费者已经获取了提供者的缓存列表
        * 注册中心和监控中心时可选的，消费者可以直连提供者。

    3. 健壮性

        * Monitor的停机时间不会影响使用，只会丢失一些采样数据
        * 当DB服务器宕机时，Register可以通过检查其缓存将服务提供者列表返回给消费者，但是新的提供者不能注册任何服务
        * 即使Register的所有实例都停止了，提供者和消费者仍然可以通过检查本地缓存进行通信
        * 服务提供者是无状态的，一个实例的停机不会影响使用
        * 当一个服务的所有提供者都宕机后，使用者不能使用该服务，并无限地重新连接以等待服务提供者恢复

    4. 扩展性

        * Register是一个可以动态增加实例的对等集群，所有客户端都会自动发现新的实例。
        * Provider是无状态的，它可以动态地增加部署实例，注册中心将把新的服务提供者信息推送给消费者。

    5. 可更新性

        当服务集群进一步扩展和IT治理结构进一步升级时，需要动态部署，当前的分布式服务体系结构不会带来阻力。下面是未来可能的架构:

        ![](dubbo/dubbo-concept-architecture-future.jpg)

        节点|角色说明
        --|--
        Deployer|用于自动服务部署的本地代理
        Repository|用于存储应用包的仓库
        Scheduler|根据访问压力自动增加或减少服务提供者的调度程序
        Admin|统一管理控制台
        Registry|负责服务发现和配置的注册中心
        Monitor|计算服务调用时间和时间的监视器

4. 用法

    1. 本地服务的spring配置

        * local.xml

            ```xml
            <bean id=“xxxService” class=“com.xxx.XxxServiceImpl” />
            <bean id=“xxxAction” class=“com.xxx.XxxAction”>
                <property name=“xxxService” ref=“xxxService” />
            </bean>
            ```

    2. 远程服务的spring配置

        把local.xml分成两部分

        * remote-provider.xml

        ```xml
        <!-- define remote service bean the same way as local service bean -->
        <bean id=“xxxService” class=“com.xxx.XxxServiceImpl” /> 
        <!-- expose the remote service -->
        <dubbo:service interface=“com.xxx.XxxService” ref=“xxxService” /> 
        ```

        * remote-consumer.xml

        ```xml
        <!-- reference the remote service -->
        <dubbo:reference id=“xxxService” interface=“com.xxx.XxxService” />
        <!-- use remote service the same say as local service -->
        <bean id=“xxxAction” class=“com.xxx.XxxAction”> 
            <property name=“xxxService” ref=“xxxService” />
        </bean>
        ```


原文：http://dubbo.apache.org/en-us/docs/user/preface/background.html