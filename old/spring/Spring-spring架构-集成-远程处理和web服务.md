## 使用Spring的远程处理和Web服务

Spring支持远程技术如下：

* 远程方法调用(RMI):通过使用RmiProxyFactoryBean和RmiServiceExporter，Spring支持传统的RMI(java.rmi.Remote和java.rmi.RemoteException)和通过RMI调用程序进行透明远程操作(使用任何Java接口)

* Spring的HTTP程序调用：Spring提供了一种特殊的远程策略，允许通过HTTP进行Java序列化，支持任何Java接口(正如RMI调用程序所做的那样)。相应的支持类是HttpInvokerProxyFactoryBean和HttpInvokerServiceExporter

* Hessian:通过使用Spring的HessianProxyFactoryBean和HessianServiceExporter，您可以通过Caucho提供的基于http的轻量级二进制协议透明地公开您的服务。

* JAX-WS：Spring通过JAX-WS为web服务提供远程支持。

* JMS:通过JmsInvokerServiceExporter 和JmsInvokerProxyFactoryBean类支持使用JMS作为底层协议的远程处理。

* AMQP：Spring AMQP项目支持使用AMQP作为底层协议的远程处理。

在讨论Spring的远程处理功能时，我们使用了以下域模型和相应的服务:

```java
public class Account implements Serializable{

    private String name;

    public String getName(){
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

```java
public interface AccountService {

    public void insertAccount(Account account);

    public List<Account> getAccounts(String name);
}
```

```java
// the implementation doing nothing at the moment
public class AccountServiceImpl implements AccountService {

    public void insertAccount(Account acc) {
        // do something...
    }

    public List<Account> getAccounts(String name) {
        // do something...
    }
}
```

1. 通过使用RMI公开服务

    1. 使用RmiServiceExporter暴露服务

        ```xml
        <bean id="accountService" class="example.AccountServiceImpl">
            <!-- any additional properties, maybe a DAO? -->
        </bean>
        ```

        ```xml
        <bean class="org.springframework.remoting.rmi.RmiServiceExporter">
            <!-- does not necessarily have to be the same name as the bean to be exported -->
            <property name="serviceName" value="AccountService"/>
            <property name="service" ref="accountService"/>
            <property name="serviceInterface" value="example.AccountService"/>
            <!-- defaults to 1099 -->
            <property name="registryPort" value="1199"/>
        </bean>
        ```

    2. 在客户端连接服务

        ```java
        public class SimpleObject {

            private AccountService accountService;

            public void setAccountService(AccountService accountService) {
                this.accountService = accountService;
            }

            // additional methods using the accountService
        }
        ```

        ```xml
        <bean class="example.SimpleObject">
            <property name="accountService" ref="accountService"/>
        </bean>

        <bean id="accountService" class="org.springframework.remoting.rmi.RmiProxyFactoryBean">
            <property name="serviceUrl" value="rmi://HOST:1199/AccountService"/>
            <property name="serviceInterface" value="example.AccountService"/>
        </bean>
        ```

2. 使用Hessian通过HTTP远程调用服务

    1. 为Hessian接入DispatcherServlet

        ```xml
        <servlet>
            <servlet-name>remoting</servlet-name>
            <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
            <load-on-startup>1</load-on-startup>
        </servlet>

        <servlet-mapping>
            <servlet-name>remoting</servlet-name>
            <url-pattern>/remoting/*</url-pattern>
        </servlet-mapping>
        ```

    2. 通过HessianServiceExporter暴露你的bean

        ```xml
        <bean id="accountService" class="example.AccountServiceImpl">
            <!-- any additional properties, maybe a DAO? -->
        </bean>

        <bean name="/AccountService" class="org.springframework.remoting.caucho.HessianServiceExporter">
            <property name="service" ref="accountService"/>
            <property name="serviceInterface" value="example.AccountService"/>
        </bean>
        ```

    3. 连接到客户端

        ```xml
        <bean class="example.SimpleObject">
            <property name="accountService" ref="accountService"/>
        </bean>

        <bean id="accountService" class="org.springframework.remoting.caucho.HessianProxyFactoryBean">
            <property name="serviceUrl" value="http://remotehost:8080/remoting/AccountService"/>
            <property name="serviceInterface" value="example.AccountService"/>
        </bean>
        ```

    4. 对通过Hessian公开的服务应用HTTP基本身份验证

        ```xml
        <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping">
            <property name="interceptors" ref="authorizationInterceptor"/>
        </bean>

        <bean id="authorizationInterceptor"
                class="org.springframework.web.servlet.handler.UserRoleAuthorizationInterceptor">
            <property name="authorizedRoles" value="administrator,operator"/>
        </bean>
        ```

3. 使用HTTP调用程序公开服务

    与Hessian相反，Spring HTTP调用器是轻量级协议，它们使用自己的瘦序列化机制，并使用标准Java序列化机制通过HTTP公开服务。

    1. 暴露服务对象

        ```xml
        <bean name="/AccountService" class="org.springframework.remoting.httpinvoker.HttpInvokerServiceExporter">
            <property name="service" ref="accountService"/>
            <property name="serviceInterface" value="example.AccountService"/>
        </bean>
        ```

    2. 在客户端连接服务

        ```xml
        <bean id="httpInvokerProxy" class="org.springframework.remoting.httpinvoker.HttpInvokerProxyFactoryBean">
            <property name="serviceUrl" value="http://remotehost:8080/remoting/AccountService"/>
            <property name="serviceInterface" value="example.AccountService"/>
        </bean>
        ```

    如前所述，您可以选择要使用什么HTTP客户端。默认情况下，HttpInvokerProxy使用JDK的HTTP功能，但是您也可以通过设置httpInvokerRequestExecutor属性来使用Apache HttpComponents客户端。下面的例子说明了如何做到这一点:

    ```xml
    <property name="httpInvokerRequestExecutor">
        <bean class="org.springframework.remoting.httpinvoker.HttpComponentsHttpInvokerRequestExecutor"/>
    </property>
    ```

4. Web服务

    Spring提供了对标准Java web服务api的全面支持:
    * 使用JAX-WS公开web服务
    * 使用JAX-WS访问web服务

    1. 通过使用JAX-WS公开基于servlet的Web服务

        在业务层编写类，继承SpringBeanAutowiringSupport

        ```java
        /**
        * JAX-WS compliant AccountService implementation that simply delegates
        * to the AccountService implementation in the root web application context.
        *
        * This wrapper class is necessary because JAX-WS requires working with dedicated
        * endpoint classes. If an existing service needs to be exported, a wrapper that
        * extends SpringBeanAutowiringSupport for simple Spring bean autowiring (through
        * the @Autowired annotation) is the simplest JAX-WS compliant way.
        *
        * This is the class registered with the server-side JAX-WS implementation.
        * In the case of a Java EE server, this would simply be defined as a servlet
        * in web.xml, with the server detecting that this is a JAX-WS endpoint and reacting
        * accordingly. The servlet name usually needs to match the specified WS service name.
        *
        * The web service engine manages the lifecycle of instances of this class.
        * Spring bean references will just be wired in here.
        */
        import org.springframework.web.context.support.SpringBeanAutowiringSupport;

        @WebService(serviceName="AccountService")
        public class AccountServiceEndpoint extends SpringBeanAutowiringSupport {

            @Autowired
            private AccountService biz;

            @WebMethod
            public void insertAccount(Account acc) {
                biz.insertAccount(acc);
            }

            @WebMethod
            public Account[] getAccounts(String name) {
                return biz.getAccounts(name);
            }
        }
        ```

    2. 使用JAX-WS导出独立的Web服务

        ```xml
        <bean class="org.springframework.remoting.jaxws.SimpleJaxWsServiceExporter">
            <property name="baseAddress" value="http://localhost:8080/"/>
        </bean>

        <bean id="accountServiceEndpoint" class="example.AccountServiceEndpoint">
            ...
        </bean>
        ```

        AccountServiceEndpoint不一定要从SpringBeanAutowiringSupport派生出来，因为它有也是一个Spring管理的bean。

        ```java
        @WebService(serviceName="AccountService")
        public class AccountServiceEndpoint {

            @Autowired
            private AccountService biz;

            @WebMethod
            public void insertAccount(Account acc) {
                biz.insertAccount(acc);
            }

            @WebMethod
            public List<Account> getAccounts(String name) {
                return biz.getAccounts(name);
            }
        }
        ```

5. 通过JMS公开服务

    还可以使用JMS作为底层通信协议透明地公开服务。Spring框架中的JMS远程支持是非常的基本。它在同一个线程和同一个非事务性会话中发送和接收。

    服务器端和客户端均使用以下接口:

    ```java
    package com.foo;

    public interface CheckingAccountService {

        public void cancelAccount(Long accountId);
    }
    ```

    服务端代码实现

    ```java
    package com.foo;

    public class SimpleCheckingAccountService implements CheckingAccountService {

        public void cancelAccount(Long accountId) {
            System.out.println("Cancelling account [" + accountId + "]");
        }
    }
    ```

    下面的配置文件包含在客户端和服务器上共享的JMS构建基础的bean:

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
            https://www.springframework.org/schema/beans/spring-beans.xsd">

        <bean id="connectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
            <property name="brokerURL" value="tcp://ep-t43:61616"/>
        </bean>

        <bean id="queue" class="org.apache.activemq.command.ActiveMQQueue">
            <constructor-arg value="mmm"/>
        </bean>

    </beans>
    ```

    1. 服务端配置

        在服务端，需要通过使用JmsInvokerServiceExporter暴露服务对象，如下例所示:

        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.springframework.org/schema/beans
                https://www.springframework.org/schema/beans/spring-beans.xsd">

            <bean id="checkingAccountService"
                    class="org.springframework.jms.remoting.JmsInvokerServiceExporter">
                <property name="serviceInterface" value="com.foo.CheckingAccountService"/>
                <property name="service">
                    <bean class="com.foo.SimpleCheckingAccountService"/>
                </property>
            </bean>

            <bean class="org.springframework.jms.listener.SimpleMessageListenerContainer">
                <property name="connectionFactory" ref="connectionFactory"/>
                <property name="destination" ref="queue"/>
                <property name="concurrentConsumers" value="3"/>
                <property name="messageListener" ref="checkingAccountService"/>
            </bean>

        </beans>
        ```

        ```java
        package com.foo;

        import org.springframework.context.support.ClassPathXmlApplicationContext;

        public class Server {

            public static void main(String[] args) throws Exception {
                new ClassPathXmlApplicationContext("com/foo/server.xml", "com/foo/jms.xml");
            }
        }
        ```

    2. 客户端配置

        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.springframework.org/schema/beans
                https://www.springframework.org/schema/beans/spring-beans.xsd">

            <bean id="checkingAccountService"
                    class="org.springframework.jms.remoting.JmsInvokerProxyFactoryBean">
                <property name="serviceInterface" value="com.foo.CheckingAccountService"/>
                <property name="connectionFactory" ref="connectionFactory"/>
                <property name="queue" ref="queue"/>
            </bean>

        </beans>
        ```

        ```java
        package com.foo;

        import org.springframework.context.ApplicationContext;
        import org.springframework.context.support.ClassPathXmlApplicationContext;

        public class Client {

            public static void main(String[] args) throws Exception {
                ApplicationContext ctx = new ClassPathXmlApplicationContext("com/foo/client.xml", "com/foo/jms.xml");
                CheckingAccountService service = (CheckingAccountService) ctx.getBean("checkingAccountService");
                service.cancelAccount(new Long(10));
            }
        }
        ```

6. AMQP

    略过

7. 选择技术时的考虑因素

    这里介绍的每种技术都有其缺点。在选择技术时，您应该仔细考虑您的需求、您公开的服务和您通过网络发送的对象。

    * 在使用RMI时，您不能通过HTTP协议访问对象，除非您对RMI通信进行隧道。RMI是一个相当重量级的协议，因为它支持全对象序列化，当您使用需要通过网络进行序列化的复杂数据模型时，这一点非常重要。

    * **如果您需要基于HTTP的远程处理，但又依赖于Java序列化，那么Spring的HTTP调用程序是一个很好的选择**。它与RMI调用程序共享基本的基础设施，但使用HTTP作为传输。注意，HTTP调用器不仅限于java到java的远程处理，而且还包括客户端和服务器端的Spring。(后者也适用于Spring的非RMI接口的RMI调用程序。)

    * **当在异构环境中运行时，Hessian可能会提供重要的价值，因为它们显式地允许非java客户端。**然而，非java支持仍然是有限的。已知的问题包括Hibernate对象与延迟初始化集合的序列化。如果您有这样的数据模型，请考虑使用RMI或HTTP调用器而不是Hessian。

    * **JMS对于提供服务集群以及让JMS代理负责负载平衡、发现和自动故障转移非常有用**。默认情况下，Java序列化用于JMS远程处理，但是JMS提供程序可以使用不同的机制来进行连接格式化，比如XStream，以便让服务器可以在其他技术中实现。

    * **最后但并非最不重要的一点是，EJB比RMI有一个优势，因为它支持标准的基于角色的身份验证和授权以及远程事务传播**。也可以让RMI调用器或HTTP调用器支持安全上下文传播，尽管core Spring没有提供这一点

8. REST端点

    SpSpring框架为调用REST端点提供了两种选择:

    * RestTemplate:带有同步模板方法API的原生的Spring REST客户端。
    * WebClien:一个非阻塞、反应性的替代方案，支持同步和异步以及流场景。

    1. 使用RestTemplate

        方法|描述
        --|--
        getForObject|通过GET检索表示
        getForEntity|使用GET检索ResponseEntity(即状态、标题和主体)。
        headForHeaders|使用HEAD检索资源的所有标头。
        postForLocation|通过使用POST创建一个新资源，并从响应返回位置头。
        postForObject|通过使用POST创建一个新资源，并从响应返回表示。
        postForEntity|通过使用POST创建一个新资源，并从响应返回表示。
        PUT|使用PUT创建或更新资源。
        patchForObject|使用补丁更新资源并从响应返回表示。注意，JDK HttpURLConnection不支持补丁，但是Apache HttpComponents和其他组件支持。
        delete|使用DELETE删除指定URI上的资源。
        optionsForAllow|通过使用ALLOW检索资源的允许HTTP方法。
        exchange|上述方法的更通用(且不那么固执己见)版本，在需要时提供额外的灵活性。它接受RequestEntity(包括HTTP方法、URL、header和body作为输入)并返回ResponseEntity。
        execute|执行请求的最通用方法，通过回调接口完全控制请求准备和响应提取。

        * Initialization

            默认构造函数使用java.net.HttpURLConnection来执行请求。您可以使用ClientHttpRequestFactory的实现切换到另一个HTTP库。内置支持以下功能:

            * Apache HttpComponents
            * Netty
            * OkHttp

            ```java
            RestTemplate template = new RestTemplate(new HttpComponentsClientHttpRequestFactory());
            ```

        * URI

            下面的例子使用了一个字符串变量参数:

            ```java
            String result = restTemplate.getForObject(
            "https://example.com/hotels/{hotel}/bookings/{booking}", String.class, "42", "21");
            ```

            下面的例子使用Map<String, String>:

            ```java
            Map<String, String> vars = Collections.singletonMap("hotel", "42");

            String result = restTemplate.getForObject(
                    "https://example.com/hotels/{hotel}/rooms/{hotel}", String.class, vars);
            ```

        * Headers

            ```java
            String uriTemplate = "https://example.com/hotels/{hotel}";
            URI uri = UriComponentsBuilder.fromUriString(uriTemplate).build(42);

            RequestEntity<Void> requestEntity = RequestEntity.get(uri)
                    .header(("MyRequestHeader", "MyValue")
                    .build();

            ResponseEntity<String> response = template.exchange(requestEntity, String.class);

            String responseHeader = response.getHeaders().getFirst("MyResponseHeader");
            String body = response.getBody();
            ```

        * Body

            ```java
            URI location = template.postForLocation("https://example.com/people", person);
            ```

        * Message Conversion

            spring-web模块包含HttpMessageConverter契约，用于通过InputStream和OutputStream读写HTTP请求和响应的主体。

        * Jackson JSON Views

            ```java
            MappingJacksonValue value = new MappingJacksonValue(new User("eric", "7!jd#h23"));
            value.setSerializationView(User.WithoutPasswordView.class);

            RequestEntity<MappingJacksonValue> requestEntity =
                RequestEntity.post(new URI("https://example.com/user")).body(value);

            ResponseEntity<String> response = template.exchange(requestEntity, String.class);
            ```

        * Multipart

            要发送multipart数据，需要提供一个MultiValueMap<String， ?>，其值要么是表示部件内容的对象实例，要么是表示部件内容和头部的HttpEntity实例。

            ```java
             ​MultipartBodyBuilder builder = new MultipartBodyBuilder();
            ​builder.part("fieldPart", "fieldValue");
            ​builder.part("filePart", new FileSystemResource("...logo.png"));
            ​builder.part("jsonPart", new Person("Jason"));

            ​MultiValueMap<String, HttpEntity<?>> parts = builder.build();
            ```


参考：https://docs.spring.io/spring/docs/5.1.6.RELEASE/spring-framework-reference/integration.html#remoting