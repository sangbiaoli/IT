### 概述

当DispatcherServlet接受到客户端的请求后，SpringMVC 通过 HandlerMapping 找到请求的Controller。
HandlerMapping 在这里起到路由的作用，负责找到请求的Controller。

### Spring MVC 默认提供了4种 HandlerMapping的实现

1. org.springframework.web.servlet.handler.SimpleUrlHandlerMapping

    通过配置请求路径和Controller映射建立关系，找到相应的Controller
2. org.springframework.web.servlet.mvc.support.ControllerClassNameHandlerMapping

    通过 Controller 的类名找到请求的Controller。
3. org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping

    通过定义的 beanName 进行查找要请求的Controller
4. org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping

    通过注解 @RequestMapping("/userlist") 来查找对应的Controller。

### HandlerMapping 的4种配置

```xml
<bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
    <property name="mappings">
        <props>
            <prop key="/userlist.htm">userController</prop>
        </props>
    </property>
</bean>

<bean class="org.springframework.web.servlet.mvc.support.ControllerClassNameHandlerMapping"/>

<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>

<bean class="org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping"/>

<bean id="userController" name="/users" class="com.qunar.web.controller.UserController"></bean>
```

```java
UserController
@Controller
public class UserController extends AbstractController {

    @Override
    @RequestMapping("/userlist")
    protected ModelAndView handleRequestInternal(HttpServletRequest request, HttpServletResponse response)
            throws Exception {

        List<User> userList = new ArrayList<User>();

        userList.add(new User("zhangsan", 18));
        userList.add(new User("list", 16));

        return new ModelAndView("userList", "users", userList);
    }
}
```

### HandlerMapping 4种访问路径

1. SimpleUrlHandlerMapping

    访问方式: http://ip:port/project/userlist.htm，参考xml中第一个bean
2. ControllerClassNameHandlerMapping

    访问方式: http://ip:port/project/user，参考xml中第二个bean，这里通过该配置把user转为UserController

    注：类的首字母要小写
3. BeanNameUrlHandlerMapping

    访问方式: http://ip:port/project/users，参考xml中第五个bean

    注：bean name属性必须要以“/”开头。

4. DefaultAnnotationHandlerMapping

    访问方式: http://ip:port/project/userlist，参考xml中第四个bean，这是通过注解找到Controller的方式

    注：@RequestMapping("/userlist")定义的路径

### HandlerMapping 初始化原理

继续上一篇Spring mvc DispatchServlet 实现原理 初始化DispatchServlet的时候，执行了初始化HandlerMapping操作。

```java
protected void initStrategies(ApplicationContext context) {
    initMultipartResolver(context);
    initLocaleResolver(context);
    initThemeResolver(context);
    initHandlerMappings(context);
    initHandlerAdapters(context);
    initHandlerExceptionResolvers(context);
    initRequestToViewNameTranslator(context);
    initViewResolvers(context);
    initFlashMapManager(context);
}
```

#### initHandlerMapping() 方法

```java
private void initHandlerMappings(ApplicationContext context) {
    this.handlerMappings = null;

    if (this.detectAllHandlerMappings) {
        // Find all HandlerMappings in the ApplicationContext, including ancestor contexts.
        Map<String, HandlerMapping> matchingBeans =
                BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);
        if (!matchingBeans.isEmpty()) {
            this.handlerMappings = new ArrayList<HandlerMapping>(matchingBeans.values());
            // We keep HandlerMappings in sorted order.
            AnnotationAwareOrderComparator.sort(this.handlerMappings);
        }
    }
    else {
        try {
            HandlerMapping hm = context.getBean(HANDLER_MAPPING_BEAN_NAME, HandlerMapping.class);
            this.handlerMappings = Collections.singletonList(hm);
        }
        catch (NoSuchBeanDefinitionException ex) {
            // Ignore, we'll add a default HandlerMapping later.
        }
    }

    // Ensure we have at least one HandlerMapping, by registering
    // a default HandlerMapping if no other mappings are found.
    if (this.handlerMappings == null) {
        this.handlerMappings = getDefaultStrategies(context, HandlerMapping.class);
        if (logger.isDebugEnabled()) {
            logger.debug("No HandlerMappings found in servlet '" + getServletName() + "': using default");
        }
    }
}
```

1. 判断detectAllHandlerMappings是否为true，如果为true，则加载当前系统中所有实现了HandlerMapping接口的bean。
2. 如果为false，则加载bean名称为“handlermapping”的HandlerMapping实现类。
3. 如果还没有找到HandlerMapping，则加载SpvingMVC 配置文件中，默认配置的HandlerMapping。

#### detectAllHandlerMappings 设置

detectAllHandlerMappings 默认为true，如果只想加载自己指定的HandlerMapping，请使用下面的方式指定
```xml
<servlet>
    <servlet-name>spring</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring-servlet.xml</param-value>
    </init-param>
    <init-param>
        <param-name>detectAllHandlerMappings</param-name>
        <param-value>false</param-value>
    </init-param>
</servlet>
```

```xml
<bean id="handlerMapping" class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
    <property name="mappings">
        <props>
            <prop key="/userlist.htm">user</prop>
        </props>
    </property>
</bean>
```

如果这样指定，则Spring MVC 只会加载这个HandlerMapping，而不会加载配置的其它的HandlerMapping。

### SimpleUrlHandlerMapping

以SimpleUrlHandlerMapping 为例，简单分析下HandlerMapping

```java
public class SimpleUrlHandlerMapping extends AbstractUrlHandlerMapping {

    private final Map<String, Object> urlMap = new LinkedHashMap<String, Object>();


    public void setMappings(Properties mappings) {
        CollectionUtils.mergePropertiesIntoMap(mappings, this.urlMap);
    }
}
```

从SimpleUrlHandlerMapping 类结构中我们可以发现urlMap属性。这个urlMap中保存了xml中配置的映射关系，通过setMappings方法填充到urlMap中。

```xml
<bean id="handlerMapping" class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
    <property name="mappings">
        <props>
            <prop key="/userlist.htm">user</prop>
        </props>
    </property>
</bean>
```

这个urlMap就充当了SpringMVC的路由功能。

每个HandlerMapping都会有一个这样的Map。

#### DispatcherServlet.doDispatch()

当用户请求时，真正的请求会执行到DispatcherServlet的doDispatch()方法。

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HttpServletRequest processedRequest = request;
        HandlerExecutionChain mappedHandler = null;
        boolean multipartRequestParsed = false;

        WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

        try {
            ModelAndView mv = null;
            Exception dispatchException = null;

            try {
                processedRequest = checkMultipart(request);
                multipartRequestParsed = (processedRequest != request);

                // Determine handler for the current request.
                mappedHandler = getHandler(processedRequest);
                if (mappedHandler == null || mappedHandler.getHandler() == null) {
                    noHandlerFound(processedRequest, response);
                    return;
                    }
                ...
```

通过getHandler() 方法去查找HandlerMapping中查找对应的Controller，并封装成HandlerExecutionChain。
如果找不到，则执行noHandlerFound() 方法。

#### getHandler() 方法

```java
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    for (HandlerMapping hm : this.handlerMappings) {
        if (logger.isTraceEnabled()) {
            logger.trace(
                    "Testing handler map [" + hm + "] in DispatcherServlet with name '" + getServletName() + "'");
        }
        HandlerExecutionChain handler = hm.getHandler(request);
        if (handler != null) {
            return handler;
        }
    }
    return null;
}
```

迭代查找所有的HandlerMapping，如果找到则直接返回。

#### noHandlerFound() 方法

```java
protected void noHandlerFound(HttpServletRequest request, HttpServletResponse response) throws Exception {
    if (pageNotFoundLogger.isWarnEnabled()) {
        pageNotFoundLogger.warn("No mapping found for HTTP request with URI [" + getRequestUri(request) +
                "] in DispatcherServlet with name '" + getServletName() + "'");
    }
    if (this.throwExceptionIfNoHandlerFound) {
        throw new NoHandlerFoundException(request.getMethod(), getRequestUri(request),
                new ServletServerHttpRequest(request).getHeaders());
    }
    else {
        response.sendError(HttpServletResponse.SC_NOT_FOUND);
    }
}
```

如果找不到Controller 则后台抛出异常或响应给前台 404。


原文：https://my.oschina.net/u/2307589/blog/1834461