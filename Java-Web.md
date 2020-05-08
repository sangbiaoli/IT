## web.xml详解

1. web.xml例子

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns="http://java.sun.com/xml/ns/javaee"
        xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
        version="3.0">
        <display-name>ciyun_person</display-name>
        
        <context-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:config/spring-context.xml</param-value>
        </context-param>
    
        <listener>
            <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
        </listener>
        
        <servlet>
            <servlet-name>SpringMVC</servlet-name>
            <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>		
        </servlet>
        <servlet-mapping>
            <servlet-name>SpringMVC</servlet-name>
            <url-pattern>/</url-pattern>
        </servlet-mapping>
        
        <filter>
            <filter-name>characterEncodingFilter</filter-name>
            <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
            <init-param>
                <param-name>encoding</param-name>
                <param-value>UTF-8</param-value>
            </init-param>
        </filter>

        <session-config>
            <session-timeout>30</session-timeout>
        </session-config>
        
        <error-page>
            <error-code>404</error-code>
            <location>/page/common/pagenotfound</location>
        </error-page>
        
        <welcome-file-list>
        <welcome-file>/</welcome-file>
        </welcome-file-list>
    </web-app>
    ```

2. web.xml加载顺序

    根据tomcat架构设计Context的实现类是StandardContext，全称org.apache.catalina.core.StandardContext。看到其实现Lifecycle接口，我们在StandardContext中找到startInternal方法

    ```java
    /**
    * Start this component and implement the requirements
    * of {@link org.apache.catalina.util.LifecycleBase#startInternal()}.
    *
    * @exception LifecycleException if this component detects a fatal error
    *  that prevents this component from being used
    */
    @Override
    protectedsynchronized void startInternal() throwsLifecycleException {
        //设置webappLoader 代码省略
        
        // Standard container startup 代码省略
    　　try{
    　　　　// Set up the context init params 
    　　　　mergeParameters();

    　　　　// Configure and call application event listeners
    　　　　if(ok) {
    　　　　　　if(!listenerStart()) {
    　　　　　　　　log.error("Error listenerStart");
    　　　　　　　　ok = false;
    　　　　　　}
    　　　　}
    
    　　　　// Configure and call application filters
    　　　　if(ok) {
    　　　　　　if(!filterStart()) {
    　　　　　　　　log.error("Error filterStart");
    　　　　　　　　ok = false;
    　　　　　　}
    　　　　}
    
    　　　　// Load and initialize all "load on startup" servlets
    　　　　if(ok) {
    　　　　　　loadOnStartup(findChildren());
    　　　　}
    
    　　　　// Start ContainerBackgroundProcessor thread
    　　　　super.threadStart();
    　　}finally{
    　　　　// Unbinding thread
    　　　　unbindThread(oldCCL);
    　　} 
    }
    ```

    那我们接着归纳和整理一下代码：

    1. 首先初始化context-param节点

    2. 接着配置和调用listeners 并开始监听

    3. 然后配置和调用filters filters开始起作用

    4. 最后加载和初始化配置在load on startup的servlets

    即元素加载顺序为：
    
    ```
    context-param --> listeners --> filters --> servlets
    ```

    ***注意：***

    1. 该加载顺序并不会受元素在web.xml文件中的位置的影响。
 
    2. 但对于某类配置节而言，与它们出现的顺序是有关的。

        以 filter 为例，web.xml 中当然可以定义多个 filter，与 filter 相关的一个配置节是 filter-mapping，这里一定要注意。

        * 对于拥有相同 filter-name 的 filter 和 filter-mapping 配置节而言，filter-mapping 必须出现在 filter 之后，否则当解析到 filter-mapping 时，它所对应的 filter-name 还未定义。
        * web 容器启动时初始化每个 filter 时，是按照 filter 配置节出现的顺序来初始化的
        * 当请求资源匹配多个 filter-mapping 时，filter 拦截资源是按照 filter-mapping 配置节出现的顺序来依次调用 doFilter() 方法的。

    接着让我们来回忆一下web项目的启动顺序

    1. web容器读取web.xml配置文件，并首先读取<context-param>和<listener>两个结点。

    2. 容器创建一个ServletContext（servlet上下文），该web项目的所有部分都将共享这个上下文。

    3. 容器将<context-param>转换为键值对，并交给servletContext。

    4. 容器按照load on startup中的启动顺序创建<listener>中的类实例，创建监听器。

    **关于load on startup**

    1. load-on-startup 元素在web应用启动的时候指定了servlet被加载的顺序，它的值必须是一个整数。

    2. 如果它的值是一个负整数或是这个元素不存在，那么容器会在该servlet被调用的时候，加载这个servlet 。

    3. 如果值是正整数或零，容器在配置的时候就加载并初始化这个servlet，容器必须保证值小的先被加载。如果值相等，容器可以自动选择先加载谁。

    4. 正数的值越小，启动该servlet的优先级越高。


原文：https://blog.csdn.net/u013215565/article/details/84977569

https://blog.csdn.net/qq_22075041/article/details/78692780