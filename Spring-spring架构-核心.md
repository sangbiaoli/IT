## spring架构核心

参考文档的这一部分涵盖了Spring框架中不可或缺的所有技术。

最重要的是Spring框架对的控制反转(IoC)容器。对Spring框架的IoC容器进行彻底的处理之后，紧接着是对Spring面向方面编程(AOP)技术的全面覆盖。它在概念上很容易理解，并且成功地解决了Java企业编程中AOP需求的80%的最佳点。

还提供了Spring与AspectJ集成的内容(就特性而言，AspectJ目前是最丰富的，当然也是Java企业空间中最成熟的AOP实现)。

**因此掌握重点：Ioc，AOP，Aspect集成。**

1. Ioc容器

    1. Spring Ioc容器和Bean介绍

        **IoC也称为依赖注入(dependency injection, DI)**。对象仅通过构造函数参数、工厂方法的参数或对象实例构造或从工厂方法返回后在该对象实例上设置的属性来定义它们的依赖关系(即与它们一起工作的其他对象)。容器在创建bean时注入这些依赖项。这个过程本质上与bean本身相反(因此称为控制反转)，bean本身通过直接构造类或一种机制(如服务定位器模式)来控制依赖项的实例化或位置。

        *org.springframework.beans*和*org.springframework.context*包是Spring框架Ioc容器的基础，*BeanFactory*接口提供了能够管理任何类型对象的高级配置机制。*ApplicationContext*是BeanFactory的子接口，添加：

        * 与Spring的AOP特性比较容易集成
        * 消息资源处理(用于国际化)
        * 事件发布
        * 应用程序层特定的上下文，如web应用程序中使用的WebApplicationContext

        **在Spring中，构成应用程序主干并由Spring IoC容器管理的对象称为bean。** bean是由Spring IoC容器实例化、组装和管理的对象。否则，bean只是应用程序中的许多对象之一。bean及其之间的依赖关系反映在容器使用的配置元数据中。

    2. 容器概述

        *org.springframework.context.ApplicationContext*接口表示Spring IoC容器，负责实例化、配置和组装bean。

        Srping提供了几种ApplicationContext的实现，在独立应用程序中，通常创建ClassPathXmlApplicationContext或FileSystemXmlApplicationContext的实例。

        在大多数应用程序场景中，不需要显式用户代码来实例化Spring IoC容器的一个或多个实例。

        下图显示了Spring如何工作的高级视图。

        ![](spring/spring-core-ioc-container-magic.png)

        1. 配置元数据

            配置元数据通常以简单直观的XML格式提供，本章的大部分内容都使用这种格式来传达Spring IoC容器的关键概念和特性。

            Spring容器的元数据配置的其他方式，可以参考：

            * [Annotation-based configuration](https://docs.spring.io/spring/docs/5.1.6.RELEASE/spring-framework-reference/core.html#beans-annotation-config):Spring 2.5引入了对基于注释的配置元数据的支持。

            * [Java-based configuration](https://docs.spring.io/spring/docs/5.1.6.RELEASE/spring-framework-reference/core.html#beans-java)，从Spring 3.0开始，Spring JavaConfig项目提供的许多特性成为Spring核心框架的一部分，比如@Configuration, @Bean, @Import, and @DependsOn等。

            基于xml的配置元数据的基本结构:

            ```xml
            <?xml version="1.0" encoding="UTF-8"?>
            <beans xmlns="http://www.springframework.org/schema/beans"
                xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                xsi:schemaLocation="http://www.springframework.org/schema/beans
                    https://www.springframework.org/schema/beans/spring-beans.xsd">

                <bean id="..." class="...">   
                    <!-- collaborators and configuration for this bean go here -->
                </bean>

                <bean id="..." class="...">
                    <!-- collaborators and configuration for this bean go here -->
                </bean>

                <!-- more bean definitions go here -->

            </beans>
            ```

            1. id属性是标识单个bean定义的字符串。
            2. class属性定义bean的类型并使用完全限定的类名。

        2. 初始化容器

            提供给ApplicationContext容器的本地路径是资源字符串，允许容器从各种外部资源(如本地文件系统、Java类路径等)加载配置元数据。

            ```java
            ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
            ```

            service.xml

            ```xml
            <?xml version="1.0" encoding="UTF-8"?>
            <beans xmlns="http://www.springframework.org/schema/beans"
                xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                xsi:schemaLocation="http://www.springframework.org/schema/beans
                    https://www.springframework.org/schema/beans/spring-beans.xsd">

                <!-- services -->

                <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
                    <property name="accountDao" ref="accountDao"/>
                    <property name="itemDao" ref="itemDao"/>
                    <!-- additional collaborators and configuration for this bean go here -->
                </bean>

                <!-- more bean definitions for services go here -->

            </beans>
            ```

            daos.xml

            ```xml
            <?xml version="1.0" encoding="UTF-8"?>
            <beans xmlns="http://www.springframework.org/schema/beans"
                xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                xsi:schemaLocation="http://www.springframework.org/schema/beans
                    https://www.springframework.org/schema/beans/spring-beans.xsd">

                <bean id="accountDao"
                    class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
                    <!-- additional collaborators and configuration for this bean go here -->
                </bean>

                <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
                    <!-- additional collaborators and configuration for this bean go here -->
                </bean>

                <!-- more bean definitions for data access objects go here -->

            </beans>
            ```

            service.xml中PetStoreServiceImpl定义的两个属性accountDao，itemDao分别ref了daos.xml中的两个bean(id=accountDao和id=itemDao)，id和ref元素之间的这种链接表示协作对象之间的依赖关系。

            *组合基于xml的配置元数据*

            在一个xml中使用一个或多个\<import/>标签来引入其他的配置文件，其Resource属性指向配置文件路径

            ```xml
            <beans>
                <import resource="services.xml"/>
                <import resource="resources/messageSource.xml"/>
                <import resource="/resources/themeSource.xml"/>

                <bean id="bean1" class="..."/>
                <bean id="bean2" class="..."/>
            </beans>
            ```

        3. 使用容器

            ApplicationContext容器初始化后，可以通过getBean获取bean类，ApplicationContext有好几个getBean的方法，以T getBean(String name, Class<T> requiredType)为例

            ```java
            // create and configure beans
            ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

            // retrieve configured instance
            PetStoreService service = context.getBean("petStore", PetStoreService.class);

            // use configured instance
            List<String> userList = service.getUsernameList();
            ```

    3. Bean概述

        Spring IoC容器管理着一个或多个bean，在容器内部，这些bean定义被表示为BeanDefinition对象，其中包含(其他信息)以下元数据:

        * 包限定的类名:通常，定义**bean的实际实现类**。
        * Bean行为配置元素，它声明**Bean在容器中的行为**(范围、生命周期回调，等等)。
        * 对bean执行其工作所需的其他bean的引用。这些引用也称为**协作者或依赖项**。
        * 要在新创建的对象中**设置的其他配置设置**—例如，池的大小限制或在管理连接池的bean中使用的连接数。




    4. 依赖
    5. Bean作用范围
    6. 自定义Bean的性质
    7. Bean定义继承
    8. 容器扩展点
    9. 基于注解的容器配置
    10. ClassPath扫描和管理组件
    11. 使用JSR330标准注解
    12. 基于Java的容器配置
    13. 环境抽象
    14. 注册一个LoadTimeWeaver
    15. ApplicationContext附加功能
    16. BeanFactory

2. 资源

    1. 介绍
    2. Resource接口
    3. 内置的Resource接口实现
    4. ResourceLoader
    5. ResourceLoaderWare接口
    6. 资源依赖关系
    7. 应用上下文和资源路径

3. 校验，数据绑定和类型转换

    1. 使用Spring的Validator接口进行验证
    2. 将代码解析为错误消息
    3. Bean操作和BeanWrapper
    4. Spring类型转换
    5. Spring字段格式化
    6. 配置全局的Date和Time格式
    7. Spring校验

4. Spring表达式语言

    1. 评估
    2. Bean定义中的表达式
    3. 语言参考
    4. 类使用举例

5. Spring切面编程

    1. AOP概念
    2. Spring AOP功能和目标
    3. AOP代理
    4. 基于模式的AOP支持
    5. 选择要使用哪种AOP声明风格
    6. 混合Aspect类型
    7. 代理机制
    8. @AspectJ代理的编程式创建
    9. Spring应用使用AspectJ
    10. 更多资源

6. Spring AOP APIS

    1. PointCut API
    2. Advice API
    3. Advisor API
    4. 使用ProxyFactoryBean创建AOP代理
    5. 简化代理定义
    6. 使用ProxyFactory以编程方式创建AOP代理
    7. 操纵Advised对象
    8. 使用“自动代理”功能
    9. 使用TargetSource实现
    10. 定义新的Advice类型
    11. 用例
    12. JSR-305元注解

7. 数据缓冲区和编解码器

    1. DataBufferFactory
    2. DataBuffer
    3. PooledDataBuffer
    4. DataBufferUtils
    5. Codecs
    6. 使用DataBuffer



原文：https://docs.spring.io/spring/docs/5.1.6.RELEASE/spring-framework-reference/core.html#spring-core