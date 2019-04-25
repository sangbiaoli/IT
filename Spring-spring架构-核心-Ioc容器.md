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

    Scope可以控制由特定bean定义创建的对象的范围。这种方法功能强大且灵活，因为您可以选择通过配置创建的对象的范围，而不必在Java类级别上考虑对象的范围。可以将bean定义为部署在多个范围中的一个。
    
    Spring框架支持六个作用域，其中四个只有在使用web感知的应用程序上下文时才可用。您还可以创建自定义范围。

    作用范围|描述
    --|--
    singleton|(默认值)为每个Spring IoC容器将单个bean定义范围限定为单个对象实例。
    prototype|将单个bean定义的范围限定为任意数量的对象实例。
    request|将单个bean定义的范围限定为单个HTTP请求的生命周期，只在web感知应用中有效
    session|将单个bean定义的范围限定为HTTP会话的生命周期，只在web感知应用中有效
    application|将单个bean定义的范围限定为ServletContext的生命周期，只在web感知应用中有效
    websocket|将单个bean定义的范围限定为WebSocket的生命，只在web感知应用中有效

    1. singleton作用范围

        Spring IoC容器只创建一次这个bean实例，并存储在此类单例bean的缓存中，对于该命名bean的所有后续请求和引用都返回缓存的对象。

        ![](spring/spring-core-ioc-container-singleton.png)

    2. prototype

        每一次请求时都会创建新的bean。通常，您应该为所有有状态bean使用原型范围，为无状态bean使用单例范围。

        ![](spring/spring-core-ioc-container-prototype.png)

        (数据访问对象(DAO)通常不配置为原型，因为典型的DAO不包含任何会话状态。)
    
    3. 具有原型bean属性的单例bean

        把原型bean注入到单例bean时，一个新的原型bean被实例化，然后依赖注入到单例bean中。

    4. Request, Session, Application和WebSocket

        在web感知应用中，request, session, application和websocket才有效。

        * Web配置初始化

            为了在request, session, application和websocket级别(web范围bean)上支持bean的作用域，在定义bean之前需要进行一些较小的初始配置。

            * 如果使用springMVC，DispatcherServlet在web启动时会初始化
            * 使用Servlet 2.5容器，可以注册org.springframework.web.context.request.RequestContextListener

                ```xml
                <web-app>
                    ...
                    <listener>
                        <listener-class>
                            org.springframework.web.context.request.RequestContextListener
                        </listener-class>
                    </listener>
                    ...
                </web-app>
                ```

            * 或者，如果侦听器设置有问题，可以考虑使用Spring的RequestContextFilter

                ```xml
                <web-app>
                    ...
                    <filter>
                        <filter-name>requestContextFilter</filter-name>
                        <filter-class>org.springframework.web.filter.RequestContextFilter</filter-class>
                    </filter>
                    <filter-mapping>
                        <filter-name>requestContextFilter</filter-name>
                        <url-pattern>/*</url-pattern>
                    </filter-mapping>
                    ...
                </web-app>
                ```

            DispatcherServlet、RequestContextListener和RequestContextFilter都做完全相同的事情，即将HTTP请求对象绑定到服务该请求的线程。这使得请求和会话范围的bean在调用链的更下方可用。

        * Request

            考虑如下的bean定义：
            
            ```xml
            <bean id="loginAction" class="com.foo.LoginAction" scope="request"/>
            ```

            对于每个http请求，Spring容器会创建一个 LoginAction bean 的新实例。也就是说，loginAction bean 的作用域限于 HTTP 请求范围。 你可以在请求内随意修改这个bean实例的状态，因为其他 loginAction bean实例看不到这些变化，bean实例是与特定的请求相关的。 当请求处理完毕，对应的bean实例也就销毁（被回收）了。

        * Session

            考虑如下的bean定义:

            ```xml
            <bean id="userPreferences" class="com.foo.UserPreferences" scope="session"/>
            ```

            在每个HTTP Session的生命周期内，Spring容器会根据id为 userPreferences 的bean定义创建一个UserPreferences bean 的新实例。 也就是说，userPreferences bean 的作用域限于 HTTP Session范围。

        * Application

            考虑如下的bean定义:

            ```xml
            <bean id="appPreferences" class="com.something.AppPreferences" scope="application"/>
            ```

            Spring容器通过为整个web应用程序使用一次AppPreferences bean定义来创建AppPreferences bean的新实例。也就是说，appPreferences bean的作用域在ServletContext级别，并存储为一个常规的ServletContext属性。类似于单例，对于单个ServletContext是单独的，但对于ApplicationContext则不是。


        
    5. 自定义范围

        ![](spring/spring-core-ioc-container-scope.png)

        除了使用spring内置的scope，我们也可以自定义scope。要集成自定义scope，需要实现接口org.springframework.beans.factory.config.Scope，这个接口共有五个个方法

        方法|说明
        --|--
        Object get(String name, ObjectFactory<?> objectFactory)|从基础范围返回给定名称的对象
        Object remove(String name)|从基础范围移除给定名称的对象
        void registerDestructionCallback(String name, Runnable callback)|注册一个回调方法，当给定名称的对象被销毁时可以执行该方法
        Object resolveContextualObject(String key)|解析给定键的上下文对象
        String getConversationId()|返回当前基础范围的对话ID(如果有的话)。
        
6. 自定义Bean的性质

    Spring框架提供了许多接口，您可以使用它们定制bean的性质。将从以下三点讨论下

    * 生命周期回调
    * ApplicationContextAware和BeanNameAware
    * 其他Aware接口

    1. 生命周期回调

        要与容器对bean生命周期的管理进行交互，可以实现Spring的两个接口InitializingBean和DisposableBean，容器调用前者的afterPropertiesSet()方法及后者的destroy()方法。

        * 初始化回调

            InitializingBean接口只有一个方法
            
            ```java
            void afterPropertiesSet() throws Exception;
            ```

            不建议使用InitializingBean接口，因为它不必要地将代码耦合到Spring。经常会使用init-method属性来声明一个bean的初始化方法。

            ```xml
            <bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
            ```

            ```java
            public class ExampleBean {

                public void init() {
                    // do some initialization work
                }
            }
            ```

        * 销毁回调

            DisposableBean接口只有一个方法
            
            ```java
            void destroy() throws Exception;
            ```

            不建议使用DisposableBean接口，因为它不必要地将代码耦合到Spring。经常会使用destroy-method属性来声明一个bean的销毁方法。

            ```xml
            <bean id="exampleInitBean" class="examples.ExampleBean" destroy-method="cleanup"/>
            ```

            ```java
            public class ExampleBean {

                public void cleanup() {
                    // do some destruction work (like releasing pooled connections)
                }
            }
            ```

        * 默认的初始化和销毁回调

            每次都要在<bean/\>显式声明init-method和destroy-method比较麻烦，因此可以在<beans/\>标签上统一设置默认初始化方法init，因此每个bean只要实现了init，在初始化时就都可以被调用。

            ```xml
            <beans default-init-method="init">

                <bean id="blogService" class="com.something.DefaultBlogService">
                    <property name="blogDao" ref="blogDao" />
                </bean>

            </beans>
            ```

            ```java
            public class DefaultBlogService implements BlogService {

                private BlogDao blogDao;

                public void setBlogDao(BlogDao blogDao) {
                    this.blogDao = blogDao;
                }

                // this is (unsurprisingly) the initialization callback method
                public void init() {
                    if (this.blogDao == null) {
                        throw new IllegalStateException("The [blogDao] property must be set.");
                    }
                }
            }
            ```

        * 启动和关闭回调

            LifeCycle接口为任何有自己生命周期需求的对象定义了基本的方法(例如启动和停止一些后台进程)

            ```java
            public interface Lifecycle {

                void start();

                void stop();

                boolean isRunning();
            }
            ```

            任何实现了Lifecycle接口的对象，然后，当ApplicationContext本身接收启动和停止信号(例如，对于运行时的停止/重启场景)时，它将这些调用级联到该上下文中定义的所有生命周期实现。它通过委托给LifecycleProcessor来实现这一点

            ```java
            public interface LifecycleProcessor extends Lifecycle {

                void onRefresh();

                void onClose();
            }
            ```

        * 在非web应用程序中优雅地关闭Spring IoC容器

            ```java
            import org.springframework.context.ConfigurableApplicationContext;
            import org.springframework.context.support.ClassPathXmlApplicationContext;

            public final class Boot {

                public static void main(final String[] args) throws Exception {
                    ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");

                    // add a shutdown hook for the above context...
                    ctx.registerShutdownHook();

                    // app runs here...

                    // main method exits, hook is called prior to the app shutting down...
                }
            }
            ```

    2. ApplicationContextAware和BeanNameAware

        当ApplicationContext创建实现org.springframework.context的对象实例时，实例提供了对该ApplicationContext的引用。

        ```java
        public interface ApplicationContextAware {

            void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
        }
        ```

        一种用途是通过编程检索其他bean。有时这种能力是有用的。但是，一般来说，您应该避免使用它，因为它将代码耦合到Spring中，并且不遵循控制反转样式(将协作者作为属性提供给bean)。ApplicationContext的其他方法提供对文件资源的访问、发布应用程序事件和访问MessageSource。

        当应用程序上下文创建一个实现org.springframework.beans.factory.BeanNameAware接口的类时，提供了对其关联对象定义中定义名称的引用。

        ```java
        public interface BeanNameAware {

            void setBeanName(String name) throws BeansException;
        }
        ```

        回调函数在填充普通bean属性之后调用，但在初始化回调(如InitializingBean、afterPropertiesSet或自定义init-method)之前调用。

    3. 其他Aware接口

        除了applicationcontext ware和BeanNameAware(前面讨论过)之外，Spring还提供了广泛的可感知回调接口，让bean向容器表明它们需要某种基础设施依赖关系。通常，名称表示依赖项类型。

        名字|注入依赖
        --|--
        ApplicationContextAware|声明ApplicationContext
        ApplicationEventPublisherAware|封装的ApplicationContext的事件发布程序
        BeanClassLoaderAware|用于加载bean类的类加载器
        BeanFactoryAware|声明BeanFactory
        BeanNameAware|声明bean的名称
        BootstrapContextAware|资源适配器引导容器运行的上下文。通常只在支持jca的ApplicationContext实例中可用。	
        LoadTimeWeaverAware|定义了用于在加载时处理类定义的weaver。	
        MessageSourceAware|已配置的消息解析策略(支持参数化和国际化)。
        NotificationPublisherAware|Spring JMX通知发布程序。	
        ResourceLoaderAware|配置的加载程序，用于低层访问资源。
        ServletConfigAware|当前servlet配置容器运行。仅在可感知web的Spring应用程序上下文中有效。
        ServletContextAware|容器运行在当前ServletContext中。仅在可感知web的Spring应用程序上下文中有效。

        **再次注意，使用这些接口将代码绑定到Spring API，并且不遵循控制反转样式。因此，我们建议将它们用于需要对容器进行编程访问的基础设施bean。**

7. Bean定义继承

    子bean定义从父定义继承配置数据。子定义可以根据需要覆盖一些值或添加其他值。使用父bean和子bean定义可以节省大量输入。*实际上，这是模板的一种形式。*

    ```xml
    <bean id="inheritedTestBean" abstract="true"
        class="org.springframework.beans.TestBean">
        <property name="name" value="parent"/>
        <property name="age" value="1"/>
    </bean>

    <bean id="inheritsWithDifferentClass"
            class="org.springframework.beans.DerivedTestBean"
            parent="inheritedTestBean" init-method="initialize">  
        <property name="name" value="override"/>
        <!-- the age property value of 1 will be inherited from parent -->
    </bean>
    ```

8. 容器扩展点

    1. 通过使用BeanPostProcessor定制bean

        BeanPostProcessor接口定义了回调方法，您可以实现这些方法来提供您自己的(或覆盖容器的默认)实例化逻辑、依赖项解析逻辑等等。如果希望在Spring容器实例化、配置和初始化bean之后实现一些定制逻辑，可以插入一个或多个定制BeanPostProcessor实现。

        ```java
        package scripting;

        import org.springframework.beans.factory.config.BeanPostProcessor;

        public class InstantiationTracingBeanPostProcessor implements BeanPostProcessor {

            // simply return the instantiated bean as-is
            public Object postProcessBeforeInitialization(Object bean, String beanName) {
                return bean; // we could potentially return any object reference here...
            }

            public Object postProcessAfterInitialization(Object bean, String beanName) {
                System.out.println("Bean '" + beanName + "' created : " + bean.toString());
                return bean;
            }
        }
        ```

        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:lang="http://www.springframework.org/schema/lang"
            xsi:schemaLocation="http://www.springframework.org/schema/beans
                https://www.springframework.org/schema/beans/spring-beans.xsd
                http://www.springframework.org/schema/lang
                https://www.springframework.org/schema/lang/spring-lang.xsd">

            <lang:groovy id="messenger"
                    script-source="classpath:org/springframework/scripting/groovy/Messenger.groovy">
                <lang:property name="message" value="Fiona Apple Is Just So Dreamy."/>
            </lang:groovy>

            <!--
            when the above bean (messenger) is instantiated, this custom
            BeanPostProcessor implementation will output the fact to the system console
            -->
            <bean class="scripting.InstantiationTracingBeanPostProcessor"/>

        </beans>
        ```

        ```java
        import org.springframework.context.ApplicationContext;
        import org.springframework.context.support.ClassPathXmlApplicationContext;
        import org.springframework.scripting.Messenger;

        public final class Boot {

            public static void main(final String[] args) throws Exception {
                ApplicationContext ctx = new ClassPathXmlApplicationContext("scripting/beans.xml");
                Messenger messenger = (Messenger) ctx.getBean("messenger");
                System.out.println(messenger);
            }

        }
        ```

        执行输出结果

        ```
        Bean 'messenger' created : org.springframework.scripting.groovy.GroovyMessenger@272961
        org.springframework.scripting.groovy.GroovyMessenger@272961
        ```

    2. 使用BeanFactoryPostProcessor自定义配置元数据

        BeanFactoryPostProcessor这个接口的语义类似于BeanPostProcessor的语义，但有一个主要区别:BeanFactoryPostProcessor操作bean配置元数据。也就是说，Spring IoC容器允许BeanFactoryPostProcessor读取配置元数据，并可能在容器实例化除BeanFactoryPostProcessor实例之外的任何bean之前更改它。

        * PropertyPlaceholderConfigurer配置方式

            ```xml
            <bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
                <property name="locations" value="classpath:com/something/jdbc.properties"/>
            </bean>

            <bean id="dataSource" destroy-method="close"
                    class="org.apache.commons.dbcp.BasicDataSource">
                <property name="driverClassName" value="${jdbc.driverClassName}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </bean>
            ```

            jdbc.properties

            ```
            jdbc.driverClassName=org.hsqldb.jdbcDriver
            jdbc.url=jdbc:hsqldb:hsql://production:9002
            jdbc.username=sa
            jdbc.password=root
            ```

9. 基于注解的容器配置

    XML设置的另一种替代方法是基于注释的配置，它依赖于字节码元数据来连接组件，而不是角括号声明。开发人员不使用XML描述bean连接，而是通过使用相关类、方法或字段声明上的注释将配置移动到组件类本身。

    与往常一样，您可以将它们注册为单独的bean定义，但是也可以通过在基于xml的Spring配置中包含以下标记来隐式注册它们(请注意上下文名称空间的包含)。

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
            https://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            https://www.springframework.org/schema/context/spring-context.xsd">

        <context:annotation-config/>

    </beans>
    ```

    1. @Required

        @Required注释应用于bean属性setter方法。

        ```java
        public class SimpleMovieLister {

            private MovieFinder movieFinder;
            private int type;

            @Required
            public void setMovieFinder(MovieFinder movieFinder) {
                this.movieFinder = movieFinder;
            }

            public void setMovieFinder(int type) {
                this.type = type;
            }
        }
        ```

        在上面的例子中，movieFinder属性添加了@Required注解，而type没有。这往往是因为开发中我们更关心movieFinder必须要设置。如果bean定义如下

        ```xml
        <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:context="http://www.springframework.org/schema/context"
            xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context-2.5.xsd">

            <context:annotation-config />

            <bean id="simpleMovieLister" class="x.y.SimpleMovieLister">
                <property name="type" value="1" />
            </bean>        
        </beans>
        ```

        因为movieFinder的属性未设置，运行它，会抛出异常：org.springframework.beans.factory.BeanInitializationException

        **从Spring Framework 5.1开始，@Required注释就被正式弃用，而倾向于为所需的设置使用构造函数注入(或InitializingBean.afterPropertiesSet()和bean属性设置器方法的自定义实现)。**

    2. @Autowired

        @Autowired可以修饰方法，构造器，属性。@Autowired基本上是关于类型驱动的注入，带有可选的语义限定符。

        * 构造器方法

            ```java
            public class MovieRecommender {

                private final CustomerPreferenceDao customerPreferenceDao;

                @Autowired
                public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
                    this.customerPreferenceDao = customerPreferenceDao;
                }

                // ...
            }
            ```

        * 任何方法和多个参数

            ```java
            public class MovieRecommender {

                private MovieCatalog movieCatalog;

                private CustomerPreferenceDao customerPreferenceDao;

                @Autowired
                public void prepare(MovieCatalog movieCatalog,
                        CustomerPreferenceDao customerPreferenceDao) {
                    this.movieCatalog = movieCatalog;
                    this.customerPreferenceDao = customerPreferenceDao;
                }

            }
            ```

            还可以是Set，Collection，Map等。

            ```java
            public class MovieRecommender {

                private Set<MovieCatalog> movieCatalogs;

                @Autowired
                public void setMovieCatalogs(Set<MovieCatalog> movieCatalogs) {
                    this.movieCatalogs = movieCatalogs;
                }

                // ...
            }
            ```

        * 属性

            ```java
            public class MovieRecommender {
                @Autowired
                private MovieCatalog movieCatalog;
            }
            ```

        如果没有一个候选的bean可用，则自动装配会失败，但可以设置为非必需。

        ```java
        public class SimpleMovieLister {

            private MovieFinder movieFinder;

            @Autowired(required = false)
            public void setMovieFinder(MovieFinder movieFinder) {
                this.movieFinder = movieFinder;
            }

            // ...
        }
        ```

        另外两种非必填设置

        1. Java 8的java.util.Optional

            ```java
            public class SimpleMovieLister {

                @Autowired
                public void setMovieFinder(Optional<MovieFinder> movieFinder) {
                    ...
                }
            }
            ```

        2. @Nullable

            ```java
            public class SimpleMovieLister {

                @Autowired
                public void setMovieFinder(@Nullable MovieFinder movieFinder) {
                    ...
                }
            }
            ```


    3. @Primary

        当出现多个候选bean时，可通过@Primary来设置其中一个作为首选。

        ```java
        @Configuration
        public class MovieConfiguration {

            @Bean
            @Primary
            public MovieCatalog firstMovieCatalog() { ... }

            @Bean
            public MovieCatalog secondMovieCatalog() { ... }

            // ...
        }
        ```

        ```java
        public class MovieRecommender {

            @Autowired
            private MovieCatalog movieCatalog; //firstMovieCatalog会被注入

            // ...
        }
        ```

    4. @Qualifier

        当出现多个候选bean时，为了更灵活的选择bean，可以使用@Qualifier(限定符)。

        ```java
        public class MovieRecommender {

            @Autowired
            @Qualifier("main")
            private SimpleMovieCatalog mainMovieCatalog;

            @Autowired
            @Qualifier("action")
            private SimpleMovieCatalog actionMovieCatalog;
        }
        ```

        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:context="http://www.springframework.org/schema/context"
            xsi:schemaLocation="http://www.springframework.org/schema/beans
                https://www.springframework.org/schema/beans/spring-beans.xsd
                http://www.springframework.org/schema/context
                https://www.springframework.org/schema/context/spring-context.xsd">

            <context:annotation-config/>

            <bean class="example.SimpleMovieCatalog">
                <qualifier value="main"/>
            </bean>

            <bean class="example.SimpleMovieCatalog">
                <qualifier value="action"/>
            </bean>

            <bean id="movieRecommender" class="example.MovieRecommender"/>

        </beans>
        ```

        xml中定义了两个SimpleMovieCatalog，但它们拥有不同的qualifier值，因此在java中会根据对应的值进行装配对应的bean。

        另外上面用id作区分也可以。

        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:context="http://www.springframework.org/schema/context"
            xsi:schemaLocation="http://www.springframework.org/schema/beans
                https://www.springframework.org/schema/beans/spring-beans.xsd
                http://www.springframework.org/schema/context
                https://www.springframework.org/schema/context/spring-context.xsd">

            <context:annotation-config/>

            <bean class="example.SimpleMovieCatalog" id="main">
            </bean>

            <bean class="example.SimpleMovieCatalog" id="action">
            </bean>

            <bean id="movieRecommender" class="example.MovieRecommender"/>

        </beans>
        ```



    5. 使用泛型作为自动装配限定符

        ```java
        @Configuration
        public class MyConfiguration {

            @Bean
            public StringStore stringStore() {
                return new StringStore();
            }

            @Bean
            public IntegerStore integerStore() {
                return new IntegerStore();
            }
        }
        ```


        ```java
        @Autowired
        private Store<String> s1; // <String> qualifier, 注入stringStore bean

        @Autowired
        private Store<Integer> s2; // <Integer> qualifier, 注入integerStore bean
        ```

    6. 使用CustomAutowireConfigurer

        CustomAutowireConfigurer是一个BeanFactoryPostProcessor，它允许您注册自己的自定义限定符注释类型，即使它们没有使用Spring的@Qualifier注释进行注释。


        ```xml
        <bean id="customAutowireConfigurer"
            class="org.springframework.beans.factory.annotation.CustomAutowireConfigurer">
            <property name="customQualifierTypes">
                <set>
                    <value>example.CustomQualifier</value>
                </set>
            </property>
        </bean>
        ```

    7. @Resource

        @Resource接受name属性。默认情况下，Spring将该值解释为要注入的bean名称。换句话说，它遵循名称语义

        ```java
        public class SimpleMovieLister {

            private MovieFinder movieFinder;

            @Resource(name="myMovieFinder") 
            public void setMovieFinder(MovieFinder movieFinder) {
                this.movieFinder = movieFinder;
            }
        }
        ```

        如果没有显式指定名称，则默认名称派生自字段名称或setter方法。

    8. @PostConstruct和@PreDestroy

        在下面的例子中，缓存在初始化时被预填充，在销毁时被清除。

        ```java
        public class CachingMovieLister {

            @PostConstruct
            public void populateMovieCache() {
                // populates the movie cache upon initialization...
            }

            @PreDestroy
            public void clearMovieCache() {
                // clears the movie cache upon destruction...
            }
        }
        ```

10. ClassPath扫描和管理组件

    上文所提的bean定义仍然在xml中，本节描述通过扫描类路径隐式检测候选组件的选项。

    *从Spring 3.0开始，Spring JavaConfig项目提供的许多特性都是Spring核心框架的一部分。这允许您使用Java定义bean，而不是使用传统的XML文件。查看@Configuration、@Bean、@Import和@DependsOn注释，了解如何使用这些新特性。*

    1. @Component和更多构造型注解

        Spring提供的构造性注解(Stereotype Annotations)有：@Component，@Controller，@Service和@Repository。**@Controller， @Service和@Repository是@Component的特殊化。**

        * @Component：一般组件
        * @Controller：controller层组件
        * @Service：业务层组件
        * @Repository：任何满足存储库角色或原型(也称为数据访问对象或DAO)的类的标记

    2. 使用元注释和组合注释

        Spring提供的许多注释都可以在您自己的代码中用作元注释。元注释是可以应用于其他注释的注释。例如，前面提到的@Service注释是用@Component进行元注释的。

        ```java
        @Target(ElementType.TYPE)
        @Retention(RetentionPolicy.RUNTIME)
        @Documented
        @Component 
        public @interface Service {

            // ....
        }
        ```

    3. 自动检测类并注册Bean定义

        Spring可以自动检测原型类，并在ApplicationContext中注册相应的bean定义实例。

        ```java
        @Service
        public class SimpleMovieLister {

            private MovieFinder movieFinder;

            @Autowired
            public SimpleMovieLister(MovieFinder movieFinder) {
                this.movieFinder = movieFinder;
            }
        }
        ```

        ```java
        @Repository
        public class JpaMovieFinder implements MovieFinder {
            // implementation elided for clarity
        }
        ```

        要自动检测这些类并注册相应的bean，需要将@ComponentScan添加到@Configuration类中，其中basePackages属性是这两个类的公共父包。

        ```java
        @Configuration
        @ComponentScan(basePackages = "org.example")
        public class AppConfig  {
            ...
        }
        ```

        或者xml

        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:context="http://www.springframework.org/schema/context"
            xsi:schemaLocation="http://www.springframework.org/schema/beans
                https://www.springframework.org/schema/beans/spring-beans.xsd
                http://www.springframework.org/schema/context
                https://www.springframework.org/schema/context/spring-context.xsd">

            <context:component-scan base-package="org.example"/>

        </beans>
        ```

    4. 使用过滤器自定义扫描

        默认我们要为每个bean添加注解，比如@Component, @Repository, @Service, @Controller，这样bean就有对应的特性。在@ComponentScan注解扫描包时，可以通过includeFilters和excludeFilters来修改和扩展此行为。每个筛选器元素都需要类型和表达式属性。

        过滤器类型|	示例表达式|	描述
        --|--|--
        annotation(默认)|org.example.SomeAnnotation|要在目标组件的类型级别上显示的注释。
        assignable|org.example.SomeClass|目标组件可分配给(扩展或实现)的类(或接口)。	
        aspectj|org.example..*Service+|要由目标组件匹配的AspectJ类型表达式。	
        regex|org\.example\.Default.*|由目标组件类名匹配的正则表达式。
        custom|org.example.MyTypeFilter|一个自定义的org.springframework.core实现。.TypeFilter接口类型。

        下面的示例显示了忽略所有@Repository注释而使用“stub”存储库的配置:

        ```java
        @Configuration
        @ComponentScan(basePackages = "org.example",
                includeFilters = @Filter(type = FilterType.REGEX, pattern = ".*Stub.*Repository"),
                excludeFilters = @Filter(Repository.class))
        public class AppConfig {
            ...
        }
        ```

        或者xml

        ```xml
        <beans>
            <context:component-scan base-package="org.example">
                <context:include-filter type="regex"
                        expression=".*Stub.*Repository"/>
                <context:exclude-filter type="annotation"
                        expression="org.springframework.stereotype.Repository"/>
            </context:component-scan>
        </beans>
        ```

    5. 在组件中定义Bean元数据

        Spring组件还可以向容器提供bean定义元数据。您可以使用@Component注释类中用于定义bean元数据的@Bean注释来实现这一点。

        ```java
        @Component
        public class FactoryMethodComponent {

            private static int i;

            @Bean
            @Qualifier("public")
            public TestBean publicInstance() {
                return new TestBean("publicInstance");
            }

            //自定义限定符的使用和方法参数自动装配
            @Bean
            protected TestBean protectedInstance(
                    @Qualifier("public") TestBean spouse,
                    @Value("#{privateInstance.age}") String country) {
                TestBean tb = new TestBean("protectedInstance", 1);
                tb.setSpouse(spouse);
                tb.setCountry(country);
                return tb;
            }

            @Bean
            private TestBean privateInstance() {
                return new TestBean("privateInstance", i++);
            }

            @Bean
            @RequestScope
            public TestBean requestScopedInstance() {
                return new TestBean("requestScopedInstance", 3);
            }
        }
        ```

        @Bean注解告诉Spring这个方法将会返回一个对象，这个对象要注册为Spring应用上下文中的bean。通常方法体中包含了最终产生bean实例的逻辑。

    6. 命名自动检测组件

        当一个组件作为扫描过程的一部分被自动检测时，它的bean名称由扫描器所知道的BeanNameGenerator策略生成。

        ```java
        @Service("myMovieLister")
        public class SimpleMovieLister {
            // ...
        }
        ```

    7. 为自动检测组件提供Scope

        与一般的spring管理组件一样，自动检测组件的默认且最常见的作用域是单例的。但是，有时您需要一个可以由@Scope注释指定的不同范围。您可以在注释中提供作用域的名称。

        ```java
        @Scope("prototype")
        @Repository
        public class MovieFinderImpl implements MovieFinder {
            // ...
        }
        ```

    8. 使用注释提供限定符元数据

        本节中的示例演示了在解析autowire候选项时使用@Qualifier注释和自定义qualifier注释提供细粒度控制。
         
        ```java
        @Component
        @Qualifier("Action")
        public class ActionMovieCatalog implements MovieCatalog {
            // ...
        }
        ```

    9. 生成候选组件索引

        虽然类路径扫描非常快，但是可以通过在编译时创建一个静态候选列表来提高大型应用程序的启动性能。在此模式下，组件扫描的所有目标模块都必须使用该机制。

        要生成索引，请向每个包含组件的模块添加附加依赖项，这些组件是组件扫描指令的目标。

        ```xml
        <dependencies>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-context-indexer</artifactId>
                <version>5.1.6.RELEASE</version>
                <optional>true</optional>
            </dependency>
        </dependencies>
        ```

11. 使用JSR330标准注解

    1. 用@Inject和@Named的依赖注入

        可以使用@javax.inject.Inject而不是@Autowired。

        ```java
        import javax.inject.Inject;

        public class SimpleMovieLister {

            private MovieFinder movieFinder;

            @Inject
            public void setMovieFinder(MovieFinder movieFinder) {
                this.movieFinder = movieFinder;
            }

            public void listMovies() {
                this.movieFinder.findMovies(...);
                ...
            }
        }
        ```

        可以使用@javax.inject.Named而不是@Qualified

        ```java
        import javax.inject.Inject;
        import javax.inject.Named;

        public class SimpleMovieLister {

            private MovieFinder movieFinder;

            @Inject
            public void setMovieFinder(@Named("main") MovieFinder movieFinder) {
                this.movieFinder = movieFinder;
            }

            // ...
        }
        ```

    2. @Named和@ManagedBean:与@Component注释等价的标准

        ```java
        import javax.inject.Inject;
        import javax.inject.Named;

        @Named("movieListener")  // @ManagedBean("movieListener") could be used as well
        public class SimpleMovieLister {

            private MovieFinder movieFinder;

            @Inject
            public void setMovieFinder(MovieFinder movieFinder) {
                this.movieFinder = movieFinder;
            }

            // ...
        }
        ```

        当您使用@Named或@ManagedBean时，您可以像使用Spring注释一样使用组件扫描

        ```java
        @Configuration
        @ComponentScan(basePackages = "org.example")
        public class AppConfig  {
            ...
        }
        ```

    3. JSR-330标准注解的限制

        Spring组件和JSR-330对比

        Spring|javax.inject.*|javax.inject restrictions / comments
        --|--|--
        @Autowired|@Inject|@Inject 没有required属性，可以与Java 8的Optional一起使用。
        @Component|@Named / @ManagedBean|JSR-330不提供可组合模型，只提供了一种识别命名组件的方法。
        @Scope("singleton")|@Singleton|JSR-330默认范围类似于Spring的原型。但是，为了使它与Spring的一般缺省值保持一致，在Spring容器中声明的JSR-330 bean在缺省情况下是单例的。
        @Qualifier|@Qualifier / @Named|javax.inject.Qualifier只是用于构建自定义限定符的元注释，具体的字符串限定符(比如Spring的带有一个值的@Qualifier)可以通过javax.inject.Named关联起来
        @Value|-|没有等效的
        @Required|-|没有等效的
        @Lazy|-|没有等效的
        ObjectFactory|Provider|javax.inject.Provider是Spring的ObjectFactory的直接替代方法，只是具有更短的get()方法名。


12. 基于Java的容器配置

    本节通过以下主题来描述在Java代码中使用注释来配置Spring容器

    * 基本概念: @Bean和@Configuration
    * 通过使用AnnotationConfigApplicationContext实例化Spring容器
    * 使用@Bean注解
    * 使用@Configuration注解
    * 编写基于java的配置
    * Bean定义配置文件
    * PropertySource抽象
    * 使用@PropertySource
    * 语句中的占位符解析

    1. 基本概念: @Bean和@Configuration

        @Bean注释用于指示方法实例化、配置和初始化要由Spring IoC容器管理的新对象。

        @Configuration注释类表明其主要目的是作为bean定义的源。

        ```java
        @Configuration
        public class AppConfig {

            @Bean
            public MyService myService() {
                return new MyServiceImpl();
            }
        }
        ```

        等同于以下xml配置

        ```xml
        <beans>
            <bean id="myService" class="com.acme.services.MyServiceImpl"/>
        </beans>
        ```
    
    2. 通过使用AnnotationConfigApplicationContext实例化Spring容器

        * 简单构造方式

            ```java
            public static void main(String[] args) {
                ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
                MyService myService = ctx.getBean(MyService.class);
                myService.doStuff();
            }
            ```

        * 使用register(类<?>…)以编程方式构建容器。

            ```java
            public static void main(String[] args) {
                AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
                ctx.register(AppConfig.class, OtherConfig.class);
                ctx.register(AdditionalConfig.class);
                ctx.refresh();
                MyService myService = ctx.getBean(MyService.class);
                myService.doStuff();
            }
            ```

    3. 使用@Bean注解

        @Bean是一个方法级别的注解，等同与<bean/\>。

        * 声明bean
        
            ```java
            @Configuration
            public class AppConfig {

                @Bean
                public TransferServiceImpl transferService() {
                    return new TransferServiceImpl();
                }
            }
            ```

            ```xml
            <beans>
                <bean id="transferService" class="com.acme.TransferServiceImpl"/>
            </beans>
            ```

        * bean生命周期，名称，别名

            ```java
            public class BeanOne {
                
            }

            public class BeanTwo {

                public void init() {
                    // initialization logic
                }

                public void cleanup() {
                    // destruction logic
                }
            }

            @Configuration
            public class AppConfig {

                @Bean({"beanOne1", "beanOne2", "beanOne3"},)
                public BeanOne beanOne() {
                    return new BeanOne();
                }

                @Bean(name = "beanTwo",initMethod = "init", destroyMethod = "cleanup")
                public BeanTwo beanTwo() {
                    return new BeanTwo();
                }
            }
            ```

        * 与@Bean组合的注解，@Scope， @Description

            ```java
            @Configuration
            public class AppConfig {

                @Bean
                @Scope("prototype")
                @Description("Provides a basic example of a bean")
                public Thing thing() {
                    return new Thing();
                }
            }
            ```
    4. 使用@Configuration注解

        @Configuration是一个类级注释，指示对象是bean定义的源。

        * 查找方法注入

            ```java
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


            @Configuration
            public class AppConfig {

                @Bean
                @Scope("prototype")
                public AsyncCommand asyncCommand() {
                    AsyncCommand command = new AsyncCommand();
                    // inject dependencies here as required
                    return command;
                }

                @Bean
                public CommandManager commandManager() {
                    // return new anonymous implementation of CommandManager with createCommand()
                    // overridden to return a new prototype Command object
                    return new CommandManager() {
                        protected Command createCommand() {
                            return asyncCommand();
                        }
                    }
                }
            }
            ```

    5. 编写基于java的配置

        正如<import/\>元素在Spring XML文件中用于帮助模块化配置一样，@Import注释允许从另一个配置类加载@Bean定义

        ```java
        @Configuration
        public class ConfigA {

            @Bean
            public A a() {
                return new A();
            }
        }

        @Configuration
        @Import(ConfigA.class)
        public class ConfigB {

            @Bean
            public B b() {
                return new B();
            }
        }
        ```

        ```java
        public static void main(String[] args) {
            ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigB.class);

            // now both beans A and B will be available...
            A a = ctx.getBean(A.class);
            B b = ctx.getBean(B.class);
        }
        ```

13. 环境抽象

    环境接口是集成在容器中的抽象，它对应用程序环境的两个关键方面建模:profiles(概要文件)和properties(属性)。

    profile是一个命名的bean定义逻辑组，只有在给定概要文件处于活动状态时才向容器注册。

    properties在几乎所有的应用程序中都扮演着重要的角色，它可能来自各种各样的源:属性文件、JVM系统属性、系统环境变量、JNDI、servlet上下文参数、特殊属性对象、映射对象等等。

    1. Bean定义概要文件

        Bean定义概要文件在核心容器中提供了一种机制，允许在不同的环境中注册不同的Bean。

        * 在开发中处理内存中的数据源，而在QA或生产中从JNDI查找相同的数据源。
        * 只有在将应用程序部署到性能环境中时才注册监视基础设施。
        * 为客户A和客户B的部署注册bean的定制实现。

        考虑实际应用程序中需要数据源，测试环境中是要使用standaloneDataSource。

        ```java
        @Bean("dataSource")
        public DataSource standaloneDataSource() {
            return new EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.HSQL)
                .addScript("classpath:com/bank/config/sql/schema.sql")
                .addScript("classpath:com/bank/config/sql/test-data.sql")
                .build();
        }
        ```

        生产环境中是要使用jndiDataSource。

        ```java
        @Bean("dataSource")
        public DataSource jndiDataSource() throws Exception {
            Context ctx = new InitialContext();
            return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
        }
        ```

        * 使用@Profile

            为了适配两种环境选取不同的数据源，可以用@Profile标签

            ```java
            @Configuration
            public class AppConfig {

                @Bean("dataSource")
                @Profile("development") //@1
                public DataSource standaloneDataSource() {
                    return new EmbeddedDatabaseBuilder()
                        .setType(EmbeddedDatabaseType.HSQL)
                        .addScript("classpath:com/bank/config/sql/schema.sql")
                        .addScript("classpath:com/bank/config/sql/test-data.sql")
                        .build();
                }
                ```

                ```java
                @Bean("dataSource")
                @Profile("production") //@2
                public DataSource jndiDataSource() throws Exception {
                    Context ctx = new InitialContext();
                    return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
                }
            }
            ```

            @1 standaloneDataSource只在开发概要文件中有效。

            @2 jndiDataSource只在生产概要文件中有效。

        * xml定义概要文件

            ```xml
            <beans xmlns="http://www.springframework.org/schema/beans"
                xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                xmlns:jdbc="http://www.springframework.org/schema/jdbc"
                xmlns:jee="http://www.springframework.org/schema/jee"
                xsi:schemaLocation="...">

                <!-- other bean definitions -->

                <beans profile="development">
                    <jdbc:embedded-database id="dataSource">
                        <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
                        <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
                    </jdbc:embedded-database>
                </beans>

                <beans profile="production">
                    <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
                </beans>
            </beans>
            ```

        * 激活概要文件

            ```java
            AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
            ctx.getEnvironment().setActiveProfiles("development");
            ctx.register(SomeConfig.class, StandaloneDataConfig.class, JndiDataConfig.class);
            ctx.refresh();
            ```

        * 默认概要文件

            默认配置文件表示默认启用的配置文件。

            ```java
            @Configuration
            @Profile("default")
            public class DefaultDataConfig {

                @Bean
                public DataSource dataSource() {
                    return new EmbeddedDatabaseBuilder()
                        .setType(EmbeddedDatabaseType.HSQL)
                        .addScript("classpath:com/bank/config/sql/schema.sql")
                        .build();
                }
            }
            ```

    2. PropertySource抽象

        * 查找属性

            Spring的环境抽象在可配置的属性源层次结构上提供搜索操作。

            ```java
            ApplicationContext ctx = new GenericApplicationContext();
            Environment env = ctx.getEnvironment();
            boolean containsMyProperty = env.containsProperty("my-property");
            System.out.println("Does my environment contain the 'my-property' property? " + containsMyProperty);
            ```

            在前面的代码片段中，我们看到了询问Spring是否为当前环境定义了my-property属性的高级方法。为了回答这个问题，环境对象对一组PropertySource对象执行搜索。
            
            PropertySource是对任何键值对源的简单抽象，Spring的StandardEnvironment配置了两个PropertySource对象

            * 一个表示JVM系统属性集(System.getproperties())
            * 另一个表示系统环境变量集(System.getenv())。

        
        * 执行的搜索是分层的
        
            默认情况下，系统属性优先于环境变量。

            对于一个通用的StandardServletEnvironment，完整的层次结构如下，最高优先级的条目位于顶部:

            1. ServletConfig参数(如果适用——例如，对于DispatcherServlet上下文)
            2. ServletContext参数(web.xml context-param条目)
            3. JNDI环境变量(java:comp/env/ entries)
            4. JVM系统属性(-D命令行参数)
            5. JVM系统环境(操作系统环境变量)

        * 自定义属性
        
            整个机制是可配置的。想要集成到此搜索中的自定义属性源。为此，需要实现并实例化自己的PropertySource，并将其添加到当前环境的PropertySource集合中。

            ```java
            ConfigurableApplicationContext ctx = new GenericApplicationContext();
            MutablePropertySources sources = ctx.getEnvironment().getPropertySources();
            sources.addFirst(new MyPropertySource());
            ```
    
    3. 使用@PropertySource

        @PropertySource注释为向Spring的环境添加PropertySource提供了一种方便的声明性机制。

        ```java
        @Configuration
        @PropertySource("classpath:/com/myco/app.properties")
        public class AppConfig {

            @Autowired
            Environment env;

            @Bean
            public TestBean testBean() {
                TestBean testBean = new TestBean();
                testBean.setName(env.getProperty("testbean.name"));
                return testBean;
            }
        }
        ```

    4. 语句中的占位符解析

        ```xml
        <beans>
            <import resource="com/bank/service/${customer}-config.xml"/>
        </beans>
        ```

14. 注册一个LoadTimeWeaver

    Spring使用LoadTimeWeaver在类加载到Java虚拟机(JVM)时动态地转换它们。
    要启用加载时weaving，可以将@EnableLoadTimeWeaving添加到@Configuration类中

    ```java
    @Configuration
    @EnableLoadTimeWeaving
    public class AppConfig {

    }
    ```

    也可以用xml方式

    ```xml
    <beans>
        <context:load-time-weaver/>
    </beans>
    ```

    一旦为ApplicationContext配置好，该ApplicationContext中的任何bean都可以实现LoadTimeWeaverAware，从而接收对加载时weaver实例的引用。这在与Spring的JPA支持相结合时特别有用，因为加载时weaving可能是JPA类转换所必需的。

15. ApplicationContext附加功能

    为了以更面向框架的方式增强BeanFactory功能，上下文包还提供了以下功能:
    
    * 通过MessageSource接口访问i18n样式的消息。
    * 通过ResourceLoader接口访问资源，例如url和文件。
    * 事件发布，即通过使用ApplicationEventPublisher接口实现ApplicationListener接口的bean。
    * 加载多个(分层的)上下文，通过分层的beanfactory接口让每个上下文都集中于一个特定的层，例如应用程序的web层。

    1. 使用MessageSource国际化

        ApplicationContext接口扩展了一个名为MessageSource的接口，因此提供了国际化(“i18n”)功能。MessageSource接口定义如下

        方法|说明
        ---|---
        String getMessage(String code, Object[] args, String default, Locale loc)|用于从MessageSource检索消息的基本方法。如果没有找到指定区域设置的消息，则使用默认消息。使用标准库提供的MessageFormat功能，传入的任何参数都将成为替换值。
        String getMessage(String code, Object[] args, Locale loc)|本质上与前面的方法相同，但有一点不同:不能指定默认消息。如果找不到消息，则抛出NoSuchMessageException。
        String getMessage(MessageSourceResolvable resolvable, Locale locale)|前面方法中使用的所有属性都封装在一个名为MessageSourceResolvable的类中，您可以使用这个方法。

        ApplicationContext加载完后会在上下文自动查找MessageSource，查找顺序：

        1. 有没有名字为messageSource的bean，有则返回，没有则2
        2. ApplicationContext往父容器上查找，有则返回，没有则3
        3. 初始化一个空的DelegatingMessageSource并返回。

        Spring提供了两种MessageSource实现，ResourceBundleMessageSource和StaticMessageSource。它们都实现了分层的messagesource，以便执行嵌套消息传递。很少使用StaticMessageSource，但它提供了向源添加消息的编程方法。
        
        ![](spring/spring-core-ioc-container-MessageSource.png)

        ```xml
        <beans>
            <bean id="messageSource"
                    class="org.springframework.context.support.ResourceBundleMessageSource">
                <property name="basenames">
                    <list>
                        <value>format</value>
                        <value>exceptions</value>
                        <value>windows</value>
                    </list>
                </property>
            </bean>
        </beans>
        ```

        假设您的类路径中定义了三个名为format、exceptions和windows的资源包。

        ```
        # in format.properties
        message=Alligators rock!
        ```

        ```
        # in exceptions.properties
        argument.required=The {0} argument is required.
        ```

        ```java
        public static void main(String[] args) {
            MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
            String message = resources.getMessage("message", null, "Default", null);
            System.out.println(message);
        }
        ```

        输出结果
        
        ```java
        Alligators rock!
        ```

    2. 标准和自定义事件

        ApplicationContext中的事件处理是通过ApplicationEvent类和ApplicationListener接口提供的。
        
        如果将实现ApplicationListener接口的bean部署到上下文中，每当将ApplicationEvent发布到ApplicationContext时，就会通知该bean。本质上，这是标准的观察者设计模式。

        Spring提供的标准事件

        事件|解释
        --|--
        ContextRefreshedEvent|在初始化或刷新ApplicationContext时发布(例如，通过在ConfigurableApplicationContext接口上使用refresh()方法)。
        ContextStartedEvent|通过在ConfigurableApplicationContext接口上使用start()方法启动ApplicationContext时发布。这里，“started”意味着所有生命周期bean都接收一个显式的start信号。
        ContextStoppedEvent|通过在ConfigurableApplicationContext接口上使用stop()方法停止ApplicationContext时发布。这里，“stop”意味着所有生命周期bean都接收一个显式的stop信号。
        ContextClosedEvent|通过在ConfigurableApplicationContext接口上使用close()方法关闭ApplicationContext时发布。这里，“closed”意味着销毁所有单例bean。
        RequestHandledEvent|一个特定于web的事件，通知所有bean HTTP请求已得到服务。此事件在请求完成后发布。

        * 自定义事件

            ![](spring/spring-core-ioc-container-custom-event.png)


            1. 定义一个扩展Spring的ApplicationEvent的黑名单事件类

                ```java
                public class BlackListEvent extends ApplicationEvent {

                    private final String address;
                    private final String content;

                    public BlackListEvent(Object source, String address, String content) {
                        super(source);
                        this.address = address;
                        this.content = content;
                    }

                    // accessor and other methods...
                }
                ```
            
            2. 定义一个实现ApplicationEventPublisherAware的类，并且特定业务中(比如发邮件)注册自定义事件

                ```java
                public class EmailService implements ApplicationEventPublisherAware {

                    private List<String> blackList;
                    private ApplicationEventPublisher publisher;

                    public void setBlackList(List<String> blackList) {
                        this.blackList = blackList;
                    }

                    public void setApplicationEventPublisher(ApplicationEventPublisher publisher) {
                        this.publisher = publisher;
                    }

                    public void sendEmail(String address, String content) {
                        if (blackList.contains(address)) {
                            publisher.publishEvent(new BlackListEvent(this, address, content));
                            return;
                        }
                        // send email...
                    }
                }
                ```

            3. 定义一个实现ApplicationListener的类，接收定制的ApplicationEvent

                ```java
                public class BlackListNotifier implements ApplicationListener<BlackListEvent> {

                    private String notificationAddress;

                    public void setNotificationAddress(String notificationAddress) {
                        this.notificationAddress = notificationAddress;
                    }

                    public void onApplicationEvent(BlackListEvent event) {
                        // notify appropriate parties via notificationAddress...
                    }
                }
                ```

            EmailService和BlackListNotifier的xml定义

            ```xml
            <bean id="emailService" class="example.EmailService">
                <property name="blackList">
                    <list>
                        <value>known.spammer@example.org</value>
                        <value>known.hacker@example.org</value>
                        <value>john.doe@example.org</value>
                    </list>
                </property>
            </bean>

            <bean id="blackListNotifier" class="example.BlackListNotifier">
                <property name="notificationAddress" value="blacklist@example.org"/>
            </bean>
            ```

            总之，当调用emailService bean的sendEmail()方法时，如果有任何电子邮件消息应该被列入黑名单，则会发布BlackListEvent类型的自定义事件。blackListNotifier bean注册为ApplicationListener并接收BlackListEvent，此时它可以通知相关方。

        * 基于注解的事件监听器

            从Spring 4.2开始，您可以使用EventListener注释在托管bean的任何公共方法上注册事件侦听器。黑名单通知程序可以重写如下:

            ```java
            public class BlackListNotifier {

                private String notificationAddress;

                public void setNotificationAddress(String notificationAddress) {
                    this.notificationAddress = notificationAddress;
                }

                @EventListener
                public void processBlackListEvent(BlackListEvent event) {
                    // notify appropriate parties via notificationAddress...
                }
            }
            ```

        * 异步监听器

            如果希望特定的侦听器异步处理事件，可以重用常规的@Async支持。

            ```java
            @EventListener
            @Async
            public void processBlackListEvent(BlackListEvent event) {
                // BlackListEvent is processed in a separate thread
            }
            ```

            使用异步事件时要注意以下限制:
            * 如果事件侦听器抛出异常，则不会将其传播给调用者，请参阅AsyncUncaughtExceptionHandler了解更多细节。
            * 此类事件侦听器无法发送响应。如果您需要作为处理的结果发送另一个事件，请注入ApplicationEventPublisher手动发送事件。

        * 有序监听器

            如果需要在调用另一个侦听器之前调用一个侦听器，可以在方法声明中添加@Order注释

            ```java
            @EventListener
            @Order(42)
            public void processBlackListEvent(BlackListEvent event) {
                // notify appropriate parties via notificationAddress...
            }
            ```

    3. 方便地访问底层资源

        为了最佳地使用和理解应用程序上下文，您应该熟悉Spring的资源抽象。

        应用程序上下文是ResourceLoader，可用于加载资源对象。资源本质上是JDK java.net.URL类的一个功能更丰富的版本。实际上，资源的实现在适当的地方封装了java.net.URL的实例。资源可以以透明的方式从几乎任何位置获得底层资源，包括类路径、文件系统位置、任何可以用标准URL描述的位置，以及其他一些变体。
        
        这里先跳过，再开专门章节学习。

    4. 方便的Web应用程序的ApplicationContext实例化
        
        您可以通过使用ContextLoader(例如)声明式地创建ApplicationContext实例。当然，您也可以通过使用ApplicationContext实现之一以编程方式创建ApplicationContext实例。
        
        您可以使用ContextLoaderListener注册一个ApplicationContext。

        ```xml
        <context-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/daoContext.xml /WEB-INF/applicationContext.xml</param-value>
        </context-param>

        <listener>
            <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
        </listener>
        ```

    5. 将Spring ApplicationContext部署为Java EE RAR文件

        可以将Spring ApplicationContext部署为RAR文件，将上下文及其所需的所有bean类和库jar封装在Java EE RAR部署单元中。这相当于引导一个独立的ApplicationContext(仅托管在Java EE环境中)，使其能够访问Java EE服务器设施。
        
        RAR部署是部署无头WAR文件的一个更自然的替代方案——实际上，没有任何HTTP入口点的WAR文件只用于在JavaEE环境中引导Spring ApplicationContext。

16. BeanFactory

    BeanFactory API为Spring的IoC功能提供了基础。它的特定契约主要用于与Spring的其他部分和相关第三方框架集成，它的DefaultListableBeanFactory实现是高层GenericApplicationContext容器中的一个关键委托。

    1. BeanFactory还是ApplicationContext？

        BeanFactory和ApplicationContext区别

        特征|BeanFactory|ApplicationContext
        --|--|--
        Bean实例化/wiring|Yes|Yes
        集成的生命周期管理|No|Yes
        自动BeanPostProcessor注册|No|Yes
        自动BeanFactoryPostProcessor注册|No|Yes
        方便的消息源访问(用于内部化)|No|Yes
        内置的ApplicationEvent发布机制|No|Yes



17. 注解汇总

    
    除了spring注解，平时也会用到jdk自带注解，一个注解基本都用到三个基本注解

    * @Documented
    * @Target，ElementType枚举类型
        * TYPE
        * FIELD
        * METHOD
        * PARAMETER
        * CONSTRUCTOR
        * LOCAL_VARIABLE
        * ANNOTATION_TYPE
        * PACKAGE
        * TYPE_PARAMETER
        * TYPE_USE
    * @Retention，RetentionPolicy枚举类型
        * SOURCE
        * CLASS
        * RUNTIME
    
    下面列出两者常用的注解图，并就本章所涉及的注解做个统一简单的汇总。

    1. spring注解图

        ![](dia/Spring-framework-annotation.png)

        注解名称|@Documented|@Target|@Retention|说明
        --|--|--|--|--|--|--
        AliasFor|*|METHOD|RUNTIME|用于为注释属性声明别名
        Qualifier|*|FIELD, METHOD, PARAMETER, TYPE, ANNOTATION_TYPE|RUNTIME|当自动装配时，该注释可以作为候选bean的限定符用于字段或参数。
        Value|*|FIELD, METHOD, PARAMETER, ANNOTATION_TYPE|RUNTIME|该参数指示受影响参数的默认值表达式
        Required||METHOD|RUNTIME|将方法(通常是JavaBean setter方法)标记为“required”:即必须将setter方法配置为依赖注入值。
        Autowired|*|CONSTRUCTOR, METHOD, PARAMETER, FIELD, ANNOTATION_TYPE}|RUNTIME|将构造函数、字段、setter方法或配置方法标记为由Spring的依赖项注入工具生成的。
        ComponentScan|*|TYPE|RUNTIME|配置组件扫描指令，以便与@Configuration类一起使用。
        Configuration|*|TYPE|RUNTIME|指示类声明一个或多个@Bean方法，并可能由Spring容器处理，以便在运行时为这些bean生成bean定义和服务请求
        Bean|*|METHOD, ANNOTATION_TYPE|RUNTIME|指示一个方法生成要由Spring容器管理的bean。
        Primary|*|TYPE, METHOD|RUNTIME|指示当多个候选项有资格自动连接单值依赖项时，应优先考虑bean。如果在候选bean中只存在一个“primary”bean，那么它就是自动获取的值。
        Component|*|TYPE|RUNTIME|指示带注释的类是“组件”。当使用基于注释的配置和类路径扫描时，这些类被认为是自动检测的候选类。
        Controller|*|TYPE|RUNTIME|指示带注释的类是“控制器”(例如web控制器)。
        Service|*|TYPE|RUNTIME|表示一个带注释的类是一个“服务”，最初由Domain-DrivenDesign (Evans, 2003)定义为“一个操作作为一个独立于模型的接口提供，没有封装状态”。
        Repository|*|TYPE|RUNTIME|表示带注释的类是“存储库”，最初由域驱动设计(Evans, 2003)定义为“一种封装存储、检索和搜索行为的机制，模拟对象集合”。
        RestController|*|TYPE|RUNTIME|A convenience annotation that is itself annotated with @Controller and @ResponseBody.
        ResponseBody|*|TYPE|RUNTIME|表示方法返回值的注释应该绑定到web响应体。支持Servlet环境中带注释的处理程序方法

    2. jdk注解图

        ![](dia/jdk-annotation.png)

        注解名称|@Documented|@Target|@Retention|说明
        --|--|--|--|--|--|--
        Resource|*|TYPE, FIELD, METHOD|RUNTIME|资源注释标记应用程序需要的资源。
        PostConstruct|*|METHOD|RUNTIME|PostConstruct注释用于一个方法，该方法需要在依赖项注入完成后执行，以执行任何初始化。
        PreDestroy|*|METHOD|RUNTIME|PreDestroy注释用于方法上，作为回调通知，通知实例正在被容器删除。
        Resources|*|TYPE|RUNTIME|该类用于允许多个资源声明。

原文：https://docs.spring.io/spring/docs/5.1.6.RELEASE/spring-framework-reference/core.html#spring-core 