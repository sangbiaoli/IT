### **Dubbo常用配置**

**1、标签之间的关系**

如图

![](dubbo/dubbo-config.jpg)


**英文原文**

标签| 	作用| 	介绍
--|--|--
\<dubbo:application/>	|Application Config 	|Applies to both provider and consumer.
\<dubbo:registry/> 	|Registry Center 	|Registry info: address, protocol, etc.
\<dubbo:monitor/> 	|Monitor Center 	|Monitor info: address, address, etc. Optional.
\<dubbo:protocol/> 	|Protocol Config 	|Configure the protocol for services on provider side, the consumer side follows.
\<dubbo:provider/> 	|Default Config for Providers 	|Default Config for ServiceConfigs. Optional.
\<dubbo:service/> 	|Service Export 	|Used to export service, define service metadata, export service with mutiple protocols, register service to multiple registries
\<dubbo:consumer/> 	|Default Config for Consumers |	Default Config for ReferenceConfigs. Optional.
\<dubbo:reference/> 	|Service Reference 	|Used to create a remote proxy, subscribe to multiple registries
\<dubbo:module/> 	|Module Config 	|Optional.
\<dubbo:method/> 	|Method level Config 	|Method level Config for ServiceConfig and ReferenceConfig.
\<dubbo:argument/> 	|Argument Config 	|Used to specify the method parameter configuration.


**中文版**

标签 	|作用 	|介绍   |归属包   
--|--|--|--
\<dubbo:application/>	|应用配置	|对于提供者和消费者一样的应用   |应用共享
\<dubbo:registry/> 	|注册中心 	|注册信息: 地址, 协议, 等等.    |应用共享
\<dubbo:monitor/> 	|监控中心 	|监控信息: 地址, 地址, 等等. 可选的.    |应用共享
\<dubbo:protocol/> 	|协议配置	|在提供者这边配置服务的协议，消费者那边一样 |提供方
\<dubbo:provider/> 	|服务端的默认配置 	|服务端的默认配置. 可选的. |提供方
\<dubbo:service/> 	|服务导出 	|用于导出服务，定义服务元数据，导出多协议服务和注册服务到多个注册中心 |提供方
\<dubbo:consumer/> 	|消费者的默认配置 |	动态调用的默认配置. 可选的. |消费方
\<dubbo:reference/> 	|服务引用	|创建远程代理，发布订阅到多个注册中心 |消费方
\<dubbo:module/> 	|模块配置 	|可选的
\<dubbo:method/> 	|方法级别配置 	|服务端和动态调用的方法级别的配置.  |\<dubbo:service/>及\<dubbo:reference/>内置方法
\<dubbo:argument/> 	|参数配置 	|用于声明方法参数配置.  |\<dubbo:method/>内置参数

分别举例子说明

**提供者**

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

**消费者**

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

**2、重写和优先级**

优先级别从高到低(重试，负载均衡，活跃也遵循这个规则)：

* 方法级别，接口级别，默认/全局级别。
* 相同的级别下, 消费者比提供者优先级更高


![](dubbo/dubbo-config-override.jpg)