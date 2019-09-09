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

    如前所述，您可以选择要使用什么HTTP客户机。默认情况下，HttpInvokerProxy使用JDK的HTTP功能，但是您也可以通过设置httpInvokerRequestExecutor属性来使用Apache HttpComponents客户机。下面的例子说明了如何做到这一点:

    ```xml
    <property name="httpInvokerRequestExecutor">
        <bean class="org.springframework.remoting.httpinvoker.HttpComponentsHttpInvokerRequestExecutor"/>
    </property>
    ```

4. Web服务
5. 通过JMS公开服务
6. AMQP
7. 选择技术时的考虑因素
8. REST端点

参考：https://docs.spring.io/spring/docs/5.1.6.RELEASE/spring-framework-reference/integration.html#remoting