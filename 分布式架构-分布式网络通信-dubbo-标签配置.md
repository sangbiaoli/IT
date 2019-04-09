## Dubbo常用配置

1. 标签之间的关系

    ![](dubbo/dubbo-config.jpg)

    * 应用共享

        标签|用途|介绍
        --|--|--
        \<dubbo:application/>|应用配置|适用于提供者和使用者
        \<dubbo:registry/>|注册中心|注册信息：地址，协议等
        \<dubbo:monitor/>|监控中心|监控信息：地址等，可选

    * 服务提供方

        标签|用途|介绍
        --|--|--
        \<dubbo:protocol/>|协议配置|在提供方配置服务的协议，消费方一样
        \<dubbo:provider/>|为提供方的默认配置|为提供方的默认配置，可选
        \<dubbo:service/>|服务导出|用于导出服务、定义服务元数据、使用多个协议导出服务、将服务注册到多个注册中心，即dubbo接口，ref属性引用该接口的实现类

    * 服务消费方

        标签|用途|介绍
        --|--|--
        \<dubbo:reference/>|服务引用|用于创建一个远程代理，发布到多个注册中心，即dubbo接口
        \<dubbo:consumer/>|为消费方的默认配置|为消费方的默认配置，可选

    * 子配置标签

        下面的标签可以作为\<dubbo:service/>和\<dubbo:reference/>的子标签

        标签|用途|介绍
        --|--|--
        \<dubbo:method/>|方法级别配置|为服务配置和引用配置的方法级别配置
        \<dubbo:argument/>|参数配置|用于声明方法参数的配置

   

2. 提供者例子

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo" xsi:schemaLocation="http://www.springframework.org/schema/beanshttp://www.springframework.org/schema/beans/spring-beans.xsdhttp://code.alibabatech.com/schema/dubbohttp://code.alibabatech.com/schema/dubbo/dubbo.xsd">
    <!--提供方信息，用于计算依赖关系-->
    <dubbo:application name="dubbo-server" owner="mic"/>
    
    <!--注册中心 暴露服务地址-->
    <dubbo:registry address="zookeeper://192.168.126.129:2181"/>
    
    <!--用dubbo协议在20880 端口暴露服务-->
    <dubbo:protocol port="20880" name="dubbo"/>
    
    <!--声明需要暴露的服务接口，指定协议为dubbo，设置版本号1.1.1-->
    <dubbo:service interface="com.gupaoedu.dubbo.IGpHello"  ref="gpHelloService" protocol="dubbo" version="1.1.1"/>
    
    <!--和本地服务一样实现服务-->
    <bean id="gpHelloService" class="com.gupaoedu.dubbo.GpHelloImpl"/>

    </beans>
    ```

3. 消费者

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beanshttp://www.springframework.org/schema/beans/spring-beans.xsdhttp://code.alibabatech.com/schema/dubbohttp://code.alibabatech.com/schema/dubbo/dubbo.xsd">
    <!--提供方信息-->
    <dubbo:application name="dubbo-client" owner="mic"/>
    
    <!--注册中心-->
    <dubbo:registry id="zookeeper" address="zookeeper://192.168.126.129:2181?register=false" file="d:/dubbo-server"/>
    
    <!--声明需要暴露的服务接口，指定版本号-->
    <dubbo:reference id="gpHelloService" interface="com.gupaoedu.dubbo.IGpHello" registry="zookeeper" version="1.1.1"/>
    </beans>
    ```

4. 重写和优先级

    以超时为例，这是一个优先级别从高到低(重试，负载均衡，活跃也遵循这个规则)：

    * 方法级别，接口级别，默认/全局级别。
    * 相同的级别下, 消费者比提供者优先级更高

    ![](dubbo/dubbo-config-override.jpg)

原文：http://dubbo.apache.org/en-us/docs/user/configuration/xml.html