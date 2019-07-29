### **Dubbo多协议和多注册中心**
本文参考出处：[Dubbo多协议和多注册中心](https://blog.csdn.net/zhoulovelian/article/details/52909820)

**一、Dubbo多协议**
dubbo提供服务支持多种协议，包括但不限于以下：
* dubbo协议（默认）
* rmi协议
* hessian协议
* http协议
* thrift协议

下面就各种协议具体说明

**1. dubbo协议**

Dubbo缺省协议采用单一长连接和NIO异步通讯，适合于小数据量大并发的服务调用，以及服务消费者机器数远大于服务提供者机器数的情况。Dubbo缺省协议不适合传送大数据量的服务，比如传文件，传视频等，除非请求量很低。

* dubbo协议配置
    ```xml
    <dubbo:protocol name="dubbo" port="20880"/>
    ```
    Set default protocol:
    ```xml
    <dubbo:provider protocol="dubbo"/>
    ```
    Set service protocol:
    ```xml
    <dubbo:service protocol="dubbo"/>
    ```
    Multi port:
    ```xml
    <dubbo:protocol id="dubbo1"name="dubbo"port="20880"/>
    <dubbo:protocol id="dubbo2"name="dubbo"port="20881"/>
    ```
    Dubbo protocol options:
    ```xml
    <dubbo:protocol name="dubbo" port="9090" server="netty" client="netty" codec="dubbo" serialization="hessian2" charset="UTF-8" threadpool="fixed" threads="100" queues="0" iothreads="9" buffer="8192" accepts="1000" payload="8388608" />
    ```
    Dubbo协议缺省每服务每提供者每消费者使用单一长连接，如果数据量较大，可以使用多个连接。
    ```xml
    <dubbo:protocol name="dubbo"connections="2"/>
    <dubbo:service connections="0">或<dubbo:reference connections="0">表示该服务使用JVM共享长连接。(缺省)
    <dubbo:service connections="1">或<dubbo:reference connections="1">表示该服务使用独立长连接。
    <dubbo:service connections="2">或<dubbo:reference connections="2">表示该服务使用独立两条长连接。
    ```
    为防止被大量连接撑挂，可在服务提供方限制大接收连接数，以实现服务提供方自我保护。
    ```xml
    <dubbo:protocol name="dubbo"accepts="1000"/>
    ```
* dubbo协议特性

    缺省协议，使用基于mina1.1.7+hessian3.2.1的tbremoting交互。
    * 连接个数：单连接
    * 连接方式：长连接
    * 传输协议：TCP
    * 传输方式：NIO异步传输
    * 序列化：Hessian二进制序列化
    * 适用范围：传入传出参数数据包较小（建议小于100K），消费者比提供者个数多，单一消费者无法压满提供者，尽量不要用dubbo协议传输大文件或超大字符串。
    * 适用场景：常规远程服务方法调用

* dubbo协议疑问

    *1. 为什么要消费者比提供者个数多？*
    ```
    因dubbo协议采用单一长连接，
    假设网络为千兆网卡(1024Mbit=128MByte)，
    根据测试经验数据每条连接最多只能压满7MByte(不同的环境可能不一样，供参考)，
    理论上1个服务提供者需要20个服务消费者才能压满网卡。
    ```

    *2. 为什么不能传大包？*
    ```
    因dubbo协议采用单一长连接，
    如果每次请求的数据包大小为500KByte，假设网络为千兆网卡(1024Mbit=128MByte)，每条连接最大7MByte(不同的环境可能不一样，供参考)，
    单个服务提供者的TPS(每秒处理事务数)最大为：128MByte / 500KByte = 262。
    单个消费者调用单个服务提供者的TPS(每秒处理事务数)最大为：7MByte / 500KByte = 14。
    如果能接受，可以考虑使用，否则网络将成为瓶颈。
    ```

    *3. 为什么采用异步单一长连接？*
    ```
    因为服务的现状大都是服务提供者少，通常只有几台机器，
    而服务的消费者多，可能整个网站都在访问该服务，
    比如Morgan的提供者只有6台提供者，却有上百台消费者，每天有1.5亿次调用，
    如果采用常规的hessian服务，服务提供者很容易就被压跨，
    通过单一连接，保证单一消费者不会压死提供者，
    长连接，减少连接握手验证等，
    并使用异步IO，复用线程池，防止C10K问题。
    ```

* dubbo协议约束
    * 参数及返回值需实现Serializable接口
    * 参数及返回值不能自定义实现List, Map, Number, Date, Calendar等接口，只能用JDK自带的实现，因为hessian会做特殊处理，自定义实现类中的属性值都会丢失。
    * Hessian序列化，只传成员属性值和值的类型，不传方法或静态变量，兼容情况：
        数据通讯	|情况	|结果
        --|--|--
        A->B	|类A多一种属性（或者说类B少一种属性）	|不抛异常，A多的那个属性的值，B没有， 其他正常
        A->B	|枚举A多一种枚举（或者说B少一种枚举），A使用多出来的枚举进行传输	|抛异常
        A->B	|枚举A多一种枚举（或者说B少一种 枚举），A不使用多出来的枚举进行传输	|不抛异常，B正常接收数据
        A->B	|A和B的属性名相同，但类型不相同	|抛异常
        A->B	|serialId不相同	|正常传输

        总结：会抛异常的情况：枚举值一边多一种，一边少一种，正好使用了差别的那种，或者属性名相同，类型不同。

        * 接口增加方法，对客户端无影响，如果该方法不是客户端需要的，客户端不需要重新部署；
        * 输入参数和结果集中增加属性，对客户端无影响，如果客户端并不需要新属性，不用重新部署；
        * 输入参数和结果集属性名变化，对客户端序列化无影响，但是如果客户端不重新部署，不管输入还是输出，属性名变化的属性值是获取不到的。

        总结：服务器端和客户端对领域对象并不需要完全一致，而是按照最大匹配原则。

**2. rmi协议**

RMI协议采用JDK标准的java.rmi.*实现，采用阻塞式短连接和JDK标准序列化方式。

* rmi协议服务接口
    * 如果服务接口继承了java.rmi.Remote接口，可以和原生RMI互操作，即：
        * 提供者用Dubbo的RMI协议暴露服务，消费者直接用标准RMI接口调用。
        * 或者提供方用标准RMI暴露服务，消费方用Dubbo的RMI协议调用。
    * 如果服务接口没有继承java.rmi.Remote接口，
        * 缺省Dubbo将自动生成一个com.xxx.XxxService$Remote的接口，并继承java.rmi.Remote接口，并以此接口暴露服务。
        * 但如果设置了<dubbo:protocol name="rmi" codec="spring" />，将不生成$Remote接口，而使用Spring的RmiInvocationHandler接口暴露服务，和Spring兼容。

* rmi协议配置

    Define rmi protocol:
    ```xml
    <dubbo:protocol name="rmi"port="1099"/>
    ```
    Set default protocol:
    ```xml
    <dubbo:provider protocol="rmi"/>
    ```
    Set service protocol:
    ```xml
    <dubbo:service protocol="rmi"/>
    ```
    Multi port:
    ```xml
    <dubbo:protocol id="rmi1"name="rmi"port="1099"/>
    <dubbo:protocol id="rmi2"name="rmi"port="2099"/>
    <dubbo:service protocol="rmi1"/>
    ```
    spring compatible:
    ```xml
    <dubbo:protocol name="rmi"codec="spring"/>
    ```

* Java标准的远程调用协议特性
    * 连接个数：多连接
    * 连接方式：短连接
    * 传输协议：TCP
    * 传输方式：同步传输
    * 序列化：Java标准二进制序列化
    * 适用范围：传入传出参数数据包大小混合，消费者与提供者个数差不多，可传文件。
    * 适用场景：常规远程服务方法调用，与原生RMI服务互操作

* rmi协议约束
    * 参数及返回值需实现Serializable接口
    * dubbo配置中的超时时间对rmi无效，需使用java启动参数设置：-Dsun.rmi.transport.tcp.responseTimeout=3000

**3. hessian协议**

Hessian协议用于集成Hessian的服务，Hessian底层采用Http通讯，采用Servlet暴露服务，Dubbo缺省内嵌Jetty作为服务器实现。

* Hessian框架

    Hessian是Caucho开源的一个RPC框架http://hessian.caucho.com，其通讯效率高于WebService和Java自带的序列化。
    依赖：
    ```xml
    <dependency>
        <groupId>com.caucho</groupId>
        <artifactId>hessian</artifactId>
        <version>4.0.7</version>
    </dependency>
    ```

    可以和原生Hessian服务互操作，即：
    * 提供者用Dubbo的Hessian协议暴露服务，消费者直接用标准Hessian接口调用。
    * 或者提供方用标准Hessian暴露服务，消费方用Dubbo的Hessian协议调用。
* Hessian协议配置

    Define hessian protocol:
    ```xml
    <dubbo:protocol name="hessian" port="8080" server="jetty"/>
    ```
    Set default protocol:
    ```xml
    <dubbo:provider protocol="hessian"/>
    ```
    Set service protocol:
    ```xml
    <dubbo:service protocol="hessian"/>
    ```
    Multi port:
    ```xml
    <dubbo:protocol id="hessian1"name="hessian"port="8080"/>
    <dubbo:protocol id="hessian2"name="hessian"port="8081"/>
    ```
    Directly provider:
    ```xml
    <dubbo:reference id="helloService"interface="HelloWorld"url="hessian://10.20.153.10:8080/helloWorld"/>
    ```
    h4. Jetty Server: (default)
    ```xml
    <dubbo:protocol... server="jetty"/>
    ```
    h4. Servlet Bridge Server: (recommend)
    ```xml
    <dubbo:protocol... server="servlet"/>
    ```
    web.xml：
    ```xml
    <servlet>
            <servlet-name>dubbo</servlet-name>
            <servlet-class>com.alibaba.dubbo.remoting.http.servlet.DispatcherServlet</servlet-class>
            <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
            <servlet-name>dubbo</servlet-name>
            <url-pattern>/*</url-pattern>
    </servlet-mapping>
    ```
    注意，如果使用servlet派发请求：
    * 协议的端口<dubbo:protocol port="8080" />必须与servlet容器的端口相同，
    * 协议的上下文路径<dubbo:protocol contextpath="foo" />必须与servlet应用的上下文路径相同。
* 基于Hessian的远程调用协议特性
    * 连接个数：多连接
    * 连接方式：短连接
    * 传输协议：HTTP
    * 传输方式：同步传输
    * 序列化：Hessian二进制序列化
    * 适用范围：传入传出参数数据包较大，提供者比消费者个数多，提供者压力较大，可传文件。
    * 适用场景：页面传输，文件传输，或与原生hessian服务互操作

* Hessian协议约束：
    * 参数及返回值需实现Serializable接口
    * 参数及返回值不能自定义实现List, Map, Number, Date, Calendar等接口，只能用JDK自带的实现，因为hessian会做特殊处理，自定义实现类中的属性值都会丢失。

**4. http协议**

此协议采用spring 的HttpInvoker的功能实现

* 配置
    ```xml
    <dubbo:protocol name="http" port="8080" />
    ```
* 基于htt协议特性
  * 连接个数：多个
  * 连接方式：长连接
  * 连接协议：http
  * 传输方式：同步传输
  * 序列化：表单序列化
  * 适用范围：传入传出参数数据包大小混合，提供者比消费者个数多，可用浏览器查看，可用表单或URL传入参数，暂不支持传文件。
  * 适用场景：需同时给应用程序和浏览器JS使用的服务。

**5.thrift协议**

当前dubbo支持的thrift协议是对thrift原生协议的扩展，在原生协议的基础上添加了一些额外的头信息，比如service name，magic number等。使用dubbo thrift协议同样需要使用thrift的idl compiler编译生成相应的java代码，后续版本中会在这方面做一些增强
* thrift原生协议的扩展

    添加依赖
    ```xml
    <dependency>  
        <groupId>org.apache.thrift</groupId>  
        <artifactId>libthrift</artifactId>  
        <version>0.8.0</version>  
    </dependency>  
    ```
    所有服务共用一个端口：(与原生Thrift不兼容)

* 配置
    ```xml
    <dubbo:protocol name="thrift" port="3030" />  
    ```
* Thrift不支持数据类型
    * null值 (不能在协议中传递null值)

 所有配置信息如下：
 ```xml
 <?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans
                        http://www.springframework.org/schema/beans/spring-beans.xsd
                        http://code.alibabatech.com/schema/dubbo
                        http://code.alibabatech.com/schema/dubbo/dubbo.xsd">  
   
    <!-- 提供方应用信息，用于计算依赖关系 -->  
    <dubbo:application name="hello-world-app"  />  
   
    <!-- 使用multicast广播注册中心暴露服务地址 -->  
    <dubbo:registry address="zookeeper://127.0.0.1:2181" />     
   
    <!-- 用dubbo协议在20880端口暴露服务 -->  
    <dubbo:protocol name="dubbo" port="20880" />  
    <dubbo:protocol name="rmi" port="1099" />  
    <dubbo:protocol name="http" port="1199" />  
    <dubbo:protocol name="hessian" port="8080" />  
    <!-- 声明需要暴露的服务接口 ，若选择hessian协议，需要加入hessian包-->    
    <dubbo:service interface="com.alibaba.dubbo.demo.DemoService" ref="demoService"   protocol="hessian"/>     
     
    <!-- 和本地bean一样实现服务 -->    
    <bean id="demoService" class="com.alibaba.dubbo.demo.provider.DemoServiceImpl" />    
      
 <!-- 扫描注解包路径，多个包用逗号分隔，不填pacakge表示扫描当前ApplicationContext中所有的类 -->  
 <dubbo:annotation package="com.alibaba.dubbo.annotation.service" />  
</beans>  
 ```
 同服务使用多协议
 ```xml
 <?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans
                        http://www.springframework.org/schema/beans/spring-beans.xsd
                        http://code.alibabatech.com/schema/dubbo
                        http://code.alibabatech.com/schema/dubbo/dubbo.xsd">  
   
    <dubbo:application name="world"  />  
    <dubbo:registry id="registry" address="10.20.141.150:9090" username="admin" password="hello1234" />  
   
    <!-- 多协议配置 -->  
    <dubbo:protocol name="dubbo" port="20880" />  
    <dubbo:protocol name="hessian" port="8080" />  
   
    <!-- 使用多个协议暴露服务 -->  
    <dubbo:service id="helloService" interface="com.alibaba.hello.api.HelloService" version="1.0.0" protocol="dubbo,hessian" />  
   
</beans>  
 ```

 **二、多注册中心**
> 同时向2个注册中心，注册服务，这样，2个注册中心都拥有此服务

> 同样，不同的服务可以注册到不同的注册中心，比如：CRM有些服务是专门为国际站设计的，有些服务是专门为中文站设计的（官方文档举得例子）。
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans        
                        http://www.springframework.org/schema/beans/spring-beans.xsd
                        http://code.alibabatech.com/schema/dubbo        
                        http://code.alibabatech.com/schema/dubbo/dubbo.xsd">  
   
    <!-- 提供方应用信息，用于计算依赖关系 -->  
    <dubbo:application name="hello-world-app"  />  
   
    <!-- 使用multicast广播注册中心暴露服务地址 -->  
    <dubbo:registry  id="localhost"  address="zookeeper://127.0.0.1:2181" />     
    <dubbo:registry id="localcomputer"  address="192.168.0.12:9010" default="false" />  
    <!-- 用dubbo协议在20880端口暴露服务 -->  
    <dubbo:protocol name="dubbo" port="20880" />  
    <dubbo:protocol name="rmi" port="1099" />  
    <!-- 声明需要暴露的服务接口 -->    
    <dubbo:service interface="com.alibaba.dubbo.demo.DemoService" ref="demoService"   protocol="rmi" registry="localhost,localcomputer" />    
     
    <!-- 和本地bean一样实现服务 -->    
    <bean id="demoService" class="com.alibaba.dubbo.demo.provider.DemoServiceImpl" />    
      
    <!-- 扫描注解包路径，多个包用逗号分隔，不填pacakge表示扫描当前ApplicationContext中所有的类 -->  
    <dubbo:annotation package="com.alibaba.dubbo.annotation.service"  />  
</beans>
```