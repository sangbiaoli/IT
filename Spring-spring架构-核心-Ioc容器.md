## Ioc容器

1. Spring Ioc容器和Bean介绍

    IoC也称为依赖注入(dependency injection, DI)。对象仅通过构造函数参数、工厂方法的参数或对象实例构造或从工厂方法返回后在该对象实例上设置的属性来定义它们的依赖关系(即与它们一起工作的其他对象)。
    
    容器在创建bean时注入这些依赖项。这个过程本质上与bean本身相反(因此称为控制反转)，bean本身通过直接构造类或一种机制(如服务定位器模式)来控制依赖项的实例化或位置。

    org.springframework.beans和org.springframework.context包是Spring框架Ioc容器的基础。

    在Spring中，构成应用程序主干并由Spring IoC容器管理的对象称为bean。bean是由Spring IoC容器实例化、组装和管理的对象。

2. 容器概述

    org.springframework.context.ApplicationContext接口表示Spring IoC容器，负责实例化、配置和组装bean。

    Srping提供了几种ApplicationContext的实现，在独立应用程序中，通常创建ClassPathXmlApplicationContext或FileSystemXmlApplicationContext的实例。

    在大多数应用程序场景中，不需要显式用户代码来实例化Spring IoC容器的一个或多个实例。

    下图显示了Spring如何工作的高级视图。

    ![](spring/spring-core-ioc-container-magic.png)

    1. 配置元数据

        配置元数据通常以简单直观的XML格式提供，本章的大部分内容都使用这种格式来传达Spring IoC容器的关键概念和特性。

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

    属性|详情参考
    --|--
    Class|bean初始化
    Name|bean命名
    Scope|bean范围
    Constructor arguments|依赖注入
    Properties|依赖注入
    Autowiring mode|自动装配的合作者
    Lazy initialization mode|懒加载bean
    Initialization method|初始化回调
    Destruction method|销毁回调

    1. bean初始化

        1. 调用其构造函数直接创建bean

            ```xml
            <bean id="exampleBean" class="examples.ExampleBean"/>
            ```

        2. 通过静态工厂方法调用来创建bean

            ```xml
            <bean id="clientService" class="examples.ClientService" factory-method="createInstance"/>
            ```

            ```java
            public class ClientService {
                private static ClientService clientService = new ClientService();
                private ClientService() {}

                public static ClientService createInstance() {
                    return clientService;
                }
            }
            ```

        3. 使用实例工厂方法实例化

            ```xml
            <!-- the factory bean, which contains a method called createInstance() -->
            <bean id="serviceLocator" class="examples.DefaultServiceLocator">
                <!-- inject any dependencies required by this locator bean -->
            </bean>

            <!-- the bean to be created via the factory bean -->
            <bean id="clientService"
                factory-bean="serviceLocator"
                factory-method="createClientServiceInstance"/>
            ```

            ```java
            public class DefaultServiceLocator {

                private static ClientService clientService = new ClientServiceImpl();

                public ClientService createClientServiceInstance() {
                    return clientService;
                }
            }
            ```
    
    2. bean命名

        bean可以有一个或多个别名。

        1. 多个别名之间可以用逗号(,)，分号(;)，空格( ) 

            ```xml
            <bean name="alias1 alias2;alias3,alias4" id="hello1" class="x.y.Example">
            ```

        2. 用alias标签

            ```xml
            <bean name="exmaple" id="hello1" class="x.y.Example">
            <alias name="exmaple" alias="alias1"/>
            <alias name="exmaple" alias="alias1"/>
            ```


4. 依赖

    1. 依赖注入
        
        1. 基于构造器的依赖注入

            1. 使用<constructor-arg/\>标签

                ```java
                package x.y;

                public class ThingOne {

                    public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
                        // ...
                    }
                }
                ```

                ```xml
                <beans>
                    <bean id="beanOne" class="x.y.ThingOne">
                        <constructor-arg ref="beanTwo"/>
                        <constructor-arg ref="beanThree"/>
                    </bean>

                    <bean id="beanTwo" class="x.y.ThingTwo"/>

                    <bean id="beanThree" class="x.y.ThingThree"/>
                </beans>
                ```

            2. 构造函数参数类型匹配或构造函数参数索引

                ```java
                package examples;

                public class ExampleBean {

                    // Number of years to calculate the Ultimate Answer
                    private int years;

                    // The Answer to Life, the Universe, and Everything
                    private String ultimateAnswer;

                    public ExampleBean(int years, String ultimateAnswer) {
                        this.years = years;
                        this.ultimateAnswer = ultimateAnswer;
                    }
                }
                ```

                ```xml
                <bean id="exampleBean" class="examples.ExampleBean">
                    <constructor-arg type="int" value="7500000"/>
                    <constructor-arg type="java.lang.String" value="42"/>
                </bean>
                ```

                ```xml
                <bean id="exampleBean" class="examples.ExampleBean">
                    <constructor-arg index="0" value="7500000"/>
                    <constructor-arg index="1" value="42"/>
                </bean>
                ```

        2. Setter方式依赖注入

            ```java
            public class ExampleBean {

                private AnotherBean beanOne;

                private YetAnotherBean beanTwo;

                private int i;

                public void setBeanOne(AnotherBean beanOne) {
                    this.beanOne = beanOne;
                }

                public void setBeanTwo(YetAnotherBean beanTwo) {
                    this.beanTwo = beanTwo;
                }

                public void setIntegerProperty(int i) {
                    this.i = i;
                }
            }
            ```

            ```xml
            <bean id="exampleBean" class="examples.ExampleBean">
                <!-- constructor injection using the nested ref element -->
                <constructor-arg>
                    <ref bean="anotherExampleBean"/>
                </constructor-arg>

                <!-- constructor injection using the neater ref attribute -->
                <constructor-arg ref="yetAnotherBean"/>

                <constructor-arg type="int" value="1"/>
            </bean>

            <bean id="anotherExampleBean" class="examples.AnotherBean"/>
            <bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
            ```


    2. 依赖及配置细节

        1. 直接值(原语、字符串等)

            ```xml
            <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
                <!-- results in a setDriverClassName(String) call -->
                <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
                <property name="username" value="root"/>
                <property name="password" value="masterkaoli"/>
            </bean>
            ```

        2. 对其他bean的引用(协作者)

            ```xml
            <!-- in the parent context -->
            <bean id="accountService" class="com.something.SimpleAccountService">
                <!-- insert dependencies as required as here -->
            </bean>
            ```

            ```xml
            <!-- in the child (descendant) context -->
            <bean id="accountService" <!-- bean name is the same as the parent bean -->
                class="org.springframework.aop.framework.ProxyFactoryBean">
                <property name="target">
                    <ref parent="accountService"/> <!-- notice how we refer to the parent bean -->
                </property>
                <!-- insert other configuration and dependencies as required here -->
            </bean>
            ```

        3. 集合

            <list/\>、<set/\>、<map/\>和<props/\>元素分别设置Java集合类型列表、set、map和属性的属性和参数。

            ```xml
            <bean id="moreComplexObject" class="example.ComplexObject">
                <!-- results in a setAdminEmails(java.util.Properties) call -->
                <property name="adminEmails">
                    <props>
                        <prop key="administrator">administrator@example.org</prop>
                        <prop key="support">support@example.org</prop>
                        <prop key="development">development@example.org</prop>
                    </props>
                </property>
                <!-- results in a setSomeList(java.util.List) call -->
                <property name="someList">
                    <list>
                        <value>a list element followed by a reference</value>
                        <ref bean="myDataSource" />
                    </list>
                </property>
                <!-- results in a setSomeMap(java.util.Map) call -->
                <property name="someMap">
                    <map>
                        <entry key="an entry" value="just some string"/>
                        <entry key ="a ref" value-ref="myDataSource"/>
                    </map>
                </property>
                <!-- results in a setSomeSet(java.util.Set) call -->
                <property name="someSet">
                    <set>
                        <value>just some string</value>
                        <ref bean="myDataSource" />
                    </set>
                </property>
            </bean>
            ```

        4. Null和空串

            ```xml
            <bean class="ExampleBean">
                <property name="email" value=""/>
            </bean>

            <bean class="ExampleBeanTwo">
                <property name="email">
                    <null/>
                </property>
            </bean>
            ```

        5. 使用p-namespace的XML快捷方式

            p-namespace允许使用bean元素的属性(而不是嵌套的<property/>元素)来描述协作bean的属性值，或者两者都使用。

            ```xml
            <beans xmlns="http://www.springframework.org/schema/beans"
                xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                xmlns:p="http://www.springframework.org/schema/p"
                xsi:schemaLocation="http://www.springframework.org/schema/beans
                    https://www.springframework.org/schema/beans/spring-beans.xsd">

                <bean name="classic" class="com.example.ExampleBean">
                    <property name="email" value="someone@somewhere.com"/>
                </bean>

                <bean name="p-namespace" class="com.example.ExampleBean"
                    p:email="someone@somewhere.com"/>
            </beans>
            ```

        6. 使用c-namespace的XML快捷方式

            与带有p-namespace的XML快捷方式类似，Spring 3.1中引入的c-namespace允许配置构造函数参数的内联属性，而不是嵌套的构造函数-arg元素。

            ```xml
            <beans xmlns="http://www.springframework.org/schema/beans"
                xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                xmlns:c="http://www.springframework.org/schema/c"
                xsi:schemaLocation="http://www.springframework.org/schema/beans
                    https://www.springframework.org/schema/beans/spring-beans.xsd">

                <bean id="beanTwo" class="x.y.ThingTwo"/>
                <bean id="beanThree" class="x.y.ThingThree"/>

                <!-- traditional declaration with optional argument names -->
                <bean id="beanOne" class="x.y.ThingOne">
                    <constructor-arg name="thingTwo" ref="beanTwo"/>
                    <constructor-arg name="thingThree" ref="beanThree"/>
                    <constructor-arg name="email" value="something@somewhere.com"/>
                </bean>

                <!-- c-namespace declaration with argument names -->
                <bean id="beanOne" class="x.y.ThingOne" c:thingTwo-ref="beanTwo"
                    c:thingThree-ref="beanThree" c:email="something@somewhere.com"/>

            </beans>
            ```
    
    3. depends-on

        有时候bean之间的依赖关系不是那么直接。例如，需要触发类中的静态初始化器，例如数据库驱动程序注册。依赖项属性可以显式强制在使用此元素的bean初始化之前初始化一个或多个bean。

        ```xml
        <bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
            <property name="manager" ref="manager" />
        </bean>

        <bean id="manager" class="ManagerBean" />
        <bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
        ```

    4. 懒加载

        可以通过将bean定义标记为延迟初始化来防止单例bean的预实例化。延迟初始化的bean告诉IoC容器在第一次请求时创建bean实例，而不是在启动时。

        ```xml
        <bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>
        <bean name="not.lazy" class="com.something.AnotherBean"/>
        ```

        也可以在容器级别设置懒加载

        ```xml
        <beans default-lazy-init="true">
            <!-- no beans will be pre-instantiated... -->
        </beans>
        ```

    5. 自动装配的合作者

        Spring容器可以自动连接协作bean之间的关系。通过检查ApplicationContext的内容，您可以让Spring为您的bean自动解析协作者(其他bean)。自动装配具有以下优点:

        1. 自动装配可以显著减少指定属性或构造函数参数的需要。

        2. 自动装配可以随着对象的发展更新配置。

        自动装配模式

        装配模式|解释
        --|--
        no|(默认)没有自动装配。Bean引用必须由ref元素定义。
        byName|通过属性名自动装配，Spring寻找与需要自动装配的具有相同名称的bean。
        byType|如果容器中只有一个属性类型则允许自动装配，否则抛出异常
        constructor|类似于byType，但适用于构造函数参数。如果容器中没有构造函数参数类型的bean，则会引发致命错误。

    6. 方法注入

        当bean的生命周期不同时，我们可能需要用到方法注入

        1. Lookup方法注入

            ```java
            package fiona.apple;

            // no more Spring imports!

            public abstract class CommandManager {

                public Object process(Object commandState) {
                    // grab a new instance of the appropriate Command interface
                    Command command = createCommand();
                    // set the state on the (hopefully brand new) Command instance
                    command.setState(commandState);
                    return command.execute();
                }

                // okay... but where is the implementation of this method?
                protected abstract Command createCommand();
            }
            ```

            ```xml
            <!-- a stateful bean deployed as a prototype (non-singleton) -->
            <bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
                <!-- inject dependencies here as required -->
            </bean>

            <!-- commandProcessor uses statefulCommandHelper -->
            <bean id="commandManager" class="fiona.apple.CommandManager">
                <lookup-method name="createCommand" bean="myCommand"/>
            </bean>
            ```

        2. 任意的方法替换

            ```java
            public class MyValueCalculator {

                public String computeValue(String input) {
                    // some real code...
                }

                // some other methods...
            }
            ```

            实现接口org.springframework.beans.factory.support.MethodReplacer

            ```java
            /**
            * meant to be used to override the existing computeValue(String)
            * implementation in MyValueCalculator
            */
            public class ReplacementComputeValue implements MethodReplacer {

                public Object reimplement(Object o, Method m, Object[] args) throws Throwable {
                    // get the input value, work with it, and return a computed result
                    String input = (String) args[0];
                    ...
                    return ...;
                }
            }
            ```

            ```xml
            <bean id="myValueCalculator" class="x.y.z.MyValueCalculator">
                <!-- arbitrary method replacement -->
                <replaced-method name="computeValue" replacer="replacementComputeValue">
                    <arg-type>String</arg-type>
                </replaced-method>
            </bean>

            <bean id="replacementComputeValue" class="a.b.c.ReplacementComputeValue"/>
            ```
        
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