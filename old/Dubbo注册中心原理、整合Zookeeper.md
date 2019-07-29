### **Dubbo注册中心原理**
Zookeeper是apache hadoop的自工程，因为它提供像树一样的服务和支持变更通知，因此适合用作dubbbo的注册中心。它是验证过的产品，因此被推荐用于生产环境。

如图

![](dubbo/dubbo-zookeeper.jpg)

注册过程的描述:

* 当提供方服务启动: 把服务URL地址写入到目录 /dubbo/com.foo.BarService/providers
* 当消费方服务启动: 从目录 /dubbo/com.foo.BarService/providers下订阅提供方服务URL地址，同时，把消费者URL地址写入到目录 /dubbo/com.foo.BarService/providers
* 当监控中心启动: 发布目录/dubbo/com.foo.BarService给所有的提供方和消费者的URL地址

支持下面的特性:
1. 当提供方发生意外停止了，注册中心能自动移除它的信息
2. 当注册中心重启后，所有注册数据和订阅请求能自动恢复
3. 当回话超时时，所有注册数据和订阅请求能自动恢复
4. 当配置<dubbo:registry check="false" />，失败的订阅或注册或被记录并且在后台保持重试
5. 配置<dubbo:registry username="admin" password="1234" />用于zookeeper登录
6. 配置<dubbo:registry group="dubbo" />用于zookeeper上的dubbo根节点。如果没有特别说明，或用默认的根节点
7. 支持使用通配符*在标签<dubbo:reference group="*" version="*" />，为了订阅引用服务的所有组和版本。


### **整合Zookeeper**
1. 使用zkclient
    * 添加依赖
        ```xml
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.3.3</version>
        </dependency>
        <dependency>
        <!-- 添加zkclient依赖-->
        <groupId>com.github.sgroschupf</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.1</version>
        </dependency>
        ```
    * 配置zkclient

        默认配置:
        ```xml
        <dubbo:registry ... client="zkclient" />
        ```
        Or:
        ```
        dubbo.registry.client=zkclient
        ```
        Or:
        ```
        zookeeper://10.20.153.10:2181?client=zkclient
        ```
2. 使用curator
    * 添加依赖
        ```xml
        <dependency>
            <groupId>com.netflix.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>1.1.10</version>
        </dependency>
        ```
    * 配置
        ```xml
        <dubbo:registry ... client="curator" />
        ```
        Or:
        ```
        dubbo.registry.client=curator
        ```
        Or:
        ```
        zookeeper://10.20.153.10:2181?client=curator
        ```
