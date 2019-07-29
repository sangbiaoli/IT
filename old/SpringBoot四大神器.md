## Spring Boot 四大神器

    Spring Boot有四大神器，分别是auto-configuration、starters、cli、actuator。

1. auto-configuration

    自动配置是Spring Boot的最大亮点，完美的展示了CoC约定由于配置。Spring Boot能自动配置Spring各种子项目（Spring MVC, Spring Security, Spring Data, Spring Cloud, Spring Integration, Spring Batch等）以及第三方开源框架所需要定义的各种Bean。

    Spring Boot内部定义了各种各样的XxxxAutoConfiguration配置类，预先定义好了各种所需的Bean。只有在特定的情况下这些配置类才会被起效。

    1. 如何导入的自动配置类

        查看源码可以看看自动配置类是如何被引入的。

        * 应用入口
            ```java
            @SpringBootApplication  
            public class SpringBootDemoApplication {  

                public static void main(String[] args) {  
                    SpringApplication.run(SpringBootDemoApplication.class, args);  
                }  

            }  
            ```

        * 类注解 @SpringBootApplication = @EnableAutoConfiguration + @ComponentScan + @Configuration
            ```java
            @SpringBootConfiguration  
            @EnableAutoConfiguration  
            @ComponentScan(excludeFilters = {  
                    @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),  
                    @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })  
            public @interface SpringBootApplication {  
                // ...  
            }  

            @Configuration  
            public @interface SpringBootConfiguration {  
                // ...  
            }  
            ```

        * 开启自动配置注解 @EnableAutoConfiguration 
            ```java
            @AutoConfigurationPackage  
            @Import(EnableAutoConfigurationImportSelector.class)  
            public @interface EnableAutoConfiguration {  
                // ...  
            }  
            ```

        * 导入配置类 EnableAutoConfigurationImportSelector extends AutoConfigurationImportSelector 

            在AutoConfigurationImportSelector类中可以看到通过 SpringFactoriesLoader.loadFactoryNames() 

            spring-boot-autoconfigure-1.5.1.RELEASE.jar/META-INF/spring.factories

            ```
            引用
            # Auto Configure 
            org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
            org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
            org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
            ```
            应用启动后是如何加载这些类的呢？

            通过执行SpringApplication.run()方法，会把当前的SpringBootDemoApplication作为source传入而在SpringApplication类中还会读取spring.factories中的设置一并构建应用的Context。

        * 配置类也可自行导入
            ```java
            @Configuration  
            @Import({  
                    DispatcherServletAutoConfiguration.class,  
                    EmbeddedServletContainerAutoConfiguration.class,  
                    ErrorMvcAutoConfiguration.class,  
                    HttpEncodingAutoConfiguration.class,  
                    HttpMessageConvertersAutoConfiguration.class,  
                    JacksonAutoConfiguration.class,  
                    MultipartAutoConfiguration.class,  
                    ServerPropertiesAutoConfiguration.class,  
                    WebMvcAutoConfiguration.class  
            })  
            @ComponentScan  
            public class SpringBootDemoApplication {  
                public static void main(String[] args) {  
                    SpringApplication.run(SpringBootDemoApplication.class, args);  
                }  
            }
            ```
    2. 自动配置类内部构成

        举例查看 org.springframework.boot.autoconfigure.web.WebMvcAutoConfiguration

        ```java
        @Configuration  
        @ConditionalOnWebApplication(type = Type.SERVLET)  
        @ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurerAdapter.class })  
        @ConditionalOnMissingBean(WebMvcConfigurationSupport.class)  
        @AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)  
        @AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, ValidationAutoConfiguration.class })  
        public class WebMvcAutoConfiguration {  

            @Bean  
            @ConditionalOnMissingBean(HiddenHttpMethodFilter.class)  
            public OrderedHiddenHttpMethodFilter hiddenHttpMethodFilter() {  
                return new OrderedHiddenHttpMethodFilter();  
            }  

            // ...  

        }  
        ```

    3. 查看项目实际配置了什么

        运行时开启DEBUG模式后即可在控制台看到具体的自动配置结果。

        /src/main/resources/application.properties
        ```
        debug=true
        ```
    4. 关闭自动配置 

        * 全体无效化 

            * 使用@Configuration @ComponentScan 代替 @SpringBootApplication。
            * 参数设置 

                src/main/resources/application.properties
                ```
                spring.boot.enableautoconfiguration=false
                ```

        * 部分无效化 
            ```java
            @SpringBootApplication(exclude=HibernateJpaAutoConfiguration.class)  
            ```
            或 
            src/main/resources/application.properties 
            ```
            spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration
            ```
2. starters

    SpringBoot的starter主要用来简化依赖用的。
    * spring中已存在starter的依赖（不列太多，仅供参考）
        * spring-boot-starter-log4j2
            * org.apache.logging.log4j:log4j-slf4j-impl
            * org.apache.logging.log4j:log4j-api
            * org.apache.logging.log4j:log4j-core
            * org.slf4j:jcl-over-slf4j
            * org.slf4j:jul-to-slf4j
            
        * spring-boot-starter-logging
            * ch.qos.logback:logback-classic
            * org.slf4j:jcl-over-slf4j
            * org.slf4j:jul-to-slf4j
            * org.slf4j:log4j-over-slf4j

        来自\<\<Spring Boot in Action>>的附录B SpringBoot Starters的内容

    * 自己写一个starter（自定义starter）

        1. 选择已有的starters，在此基础上进行扩展.
        2. 创建自动配置文件并设定META-INF/spring.factories里的内容.
        3. 发布你的starter

        添加依赖管理
        ```xml
        <dependencyManagement>
            <dependencies>
                <dependency>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-dependencies</artifactId>
                    <version>${spring.boot.version}</version>
                    <type>pom</type>
                    <scope>import</scope>
                </dependency>
            </dependencies>
        </dependencyManagement>
        ```

        添加starter自己的依赖
        ```xml
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
            </dependency>
        </dependencies>
        ```

        新建configuration
        ```java
        @Configuration
        @ComponentScan( basePackages = {"com.patterncat.actuator"} )
        public class WebAutoConfiguration {
            /**
            * addViewController方法不支持placeholder的解析
            * 故在这里用变量解析出来
             */
            @Value("${actuator.web.base:}")    String actuatorBase;

            //@Bean
            //public ActuatorNavController actuatorNavController(){
            //     return new ActuatorNavController();
            //}
            @Bean
            public WebMvcConfigurerAdapter configStaticMapping() {
                return new WebMvcConfigurerAdapter() {
                    @Override
                    public void addViewControllers(ViewControllerRegistry registry) {         
                        //配置跳转
                        registry.addViewController(actuatorBase+"/nav").setViewName("forward:/static/nav.html");
                    }

                    @Override
                    public void addResourceHandlers(ResourceHandlerRegistry registry) {      registry.addResourceHandler("/static/**").addResourceLocations       ("classpath:/static/");
                    }
                };
            }
        }
        ```

        修改/META-INF/spring.factories
        ```properties
        # AutoConfigurations
        org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.patterncat.actuator.configuration.WebAutoConfiguration
        ```

        发布
        ```
        mvn clean install
        ```

        引用
        ```xml
        <dependency>
            <groupId>com.patterncat</groupId>
            <artifactId>spring-boot-starter-actuator-web</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        ```

        启动访问
        ```
        mvn spring-boot:run
        ```

        访问
        ```
        http://localhost:8080/nav
        ```
3. cli  

    这是Spring Boot的可选特性，借此你只需写代码就能完成完整的应用程序，无需传统项目构建。

    创建一个文件 app.groovy:
    ```java
    @RestController
    class ThisWillActuallyRun {
        @RequestMapping("/")
        String home() {
            "Hello World!"
        }
    }
    ```
    然后执行shell命令:
    ```
    $ spring run app.groovy
    ```
    在浏览器中打开 localhost:8080 预览结果：
    ```
    Hello World!
    ```
    Spring Boot CLI让只写代码即可实现应用程序成为可能。Spring Boot CLI 利用了起步依赖和自动配置，让你专注于代码本身。不仅如此，你是否注意代码里没有import ？CLI如何知道RequestMapping和RestController来自哪个包呢？说到这个问题，那些类最终又是怎么跑到Classpath 里的呢？

    说得简单一点，CLI能检测到你使用了哪些类，它知道要向Classpath 中添加哪些起步依赖才能让它运转起来。一旦那些依赖出现在Classpath 中，一系列自动配置就会接踵而来，确保启用DispatcherServlet和Spring MVC，这样控制器就能响应HTTP请求了。

    Spring Boot CLI 是Spring Boot 的非必要组成部分。虽然它为Spring 带来了惊人的力量，大大简化了开发，但也引入了一套不太常规的开发模型。要是这种开发模型与你的口味相去甚远，那也没关系，抛开CLI，你还是可以利用Spring Boot 提供的其他东西。

4. actuator

    actuator是spring boot提供的对应用系统的自省和监控的集成功能，可以对应用系统进行配置查看、相关功能统计等。

    * 使用actuator

        添加依赖
        ```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        ```
    * 主要暴露的功能

        HTTP方法	|路径	|描述	|鉴权
        --|--|--|--
        GET	|/autoconfig	|查看自动配置的使用情况	|true
        GET	|/configprops	|查看配置属性，包括默认配置	|true
        GET	|/beans	|查看bean及其关系列表	|true
        GET	|/dump	|打印线程栈	|true
        GET	|/env	|查看所有环境变量	|true
        GET	|/env/{name}	|查看具体变量值	|true
        GET	|/health	|查看应用健康指标	|false
        GET	|/info	|查看应用信息	|false
        GET	|/mappings	|查看所有url映射	|true
        GET	|/metrics	|查看应用基本指标	|true
        GET	|/metrics/{name}	|查看具体指标	|true
        POST	|/shutdown	|关闭应用	|true
        GET	|/trace	|查看基本追踪信息	|true

    * Actuator定制

        1. 修改端点ID

            每个Actuator端点都是有一个特定的ID用来决定端点的路径。/beans端点的默认ID就是 beans 。端点的路径是由ID决定的， 那么可以通过修改ID来改变端点的路径。 要做的就是设置一个属性，属性名是 endpoints.endpoint-id.id 。

            如把/beans改为/beansome：
            ```
            endpoints.beans.id=beansome
            ```
            这时要是想查看bean的信息时，路径就由原来的/beans变为/beansome;

        2. 启用和禁用端点

            默认情况下，所有端点（除了/shutdown）都是启用的。

            禁用端点所有的端点：
            ```
            endpoints.enabled=false
            ```
            禁用某个特定的端点：
            ```
            endpoints.endpoint-id.enabled=false
            ```
            注意奥！
            禁用后，再次访问URL时，会出现404错误。

        3. 添加自定义度量信息

            1. 简单的自定义度量信息：

                springBoot自动配置Actuator创建CounterService的实例，并将其注册为Spring的应用程序上下文中的Bean。

                CounterService这个接口里定义了三个方法分别用来增加、减少或重置特定名称的度量值。
                代码如下：
                ```java
                package org.springframework.boot.actuate.metrics;

                public interface CounterService {

                void increment(String metricName);

                void decrement(String metricName);

                void reset(String metricName);

                }
                ```

                Actuator的自动配置还会配置一个GaugeService类型的Bean。
                代码如下：
                ```java
                package org.springframework.boot.actuate.metrics;

                public interface GaugeService {

                    void submit(String metricName, double value);

                }
                ```

                自己写的一个实例：
                ```java
                @Controllerpublic class HelloController {
                    @Autowired
                    private CounterService counterService;  
                    @Autowired  
                    private GaugeService gaugeService;
                    @RequestMapping(value = "/login", method=RequestMethod.GET)
                    public String login() {
                        counterService.increment("logintimes:");
                        gaugeService.submit("loginLastTime:",System.currentTimeMillis());
                        return "login";
                    }
                }
                ```
                在运行/ metrics时，出现自定义的度量信息。

            2. 相对复杂点的自定义度量信息

                 主要是写一个类实现publicMetrics接口，提供自己想要的度量信息。该接口定义了一个metrics()方法，返回一个Metric对象的集合

                代码如下：
                ```java
                @Componentpublic
                class CustomMetrics implements PublicMetrics {
                    private ApplicationContext applicationContext;
                    @Autowide 
                    public CustomMetrics(ApplicationContext applicationContext) {
                    this.applicationContext = applicationContext;
                    } 
                    @Override  
                    public Collection<Metric<?>> metrics() {
                        List<Metric<?>> metrics = new ArrayList<Metric<?>>();
                        metrics.add(new Metric<Long>("spring.startup-date",applicationContext.getStartupDate()));
                        metrics.add(new Metric<Integer>("spring.bean.definitions",applicationContext.getBeanDefinitionCount()));
                        metrics.add(new Metric<Integer>("spring.beans",applicationContext.getBeanNamesForType(Object.class).length));
                        metrics.add(new Metric<Integer>("spring.controllers",applicationContext.getBeanNamesForAnnotation(Controller.class).length));
                        return metrics;  
                    }
                }
                ```
                运行结果中可以找到以上自定义的度量信息

                ```
                "spring.startup-date":1477211367363,
                "spring.bean.definitions":317,
                "spring.beans":331,
                "spring.controllers":3,
                ```

        4. 插入自定义健康指示器

            自定义健康指示器时，需要实现HealthIndicator，重写health()方法即可。
            调用withDetail()方法向健康记录里添加其他附加信息。有多个附加信息时，可多次调用withDetail()方法，每次设置一个健康记录的附加字段。

            示例（关于一个异常的健康指示器）如下：

            ```java
            @Component
            public class CustomHealth implements HealthIndicator{
                @Override
                public Health health() {
                    try {
                        RestTemplate restTemplate = new RestTemplate();
                        restTemplate.getForObject("http://localhost:8080/index",String.class);
                        return Health.up().build();
                    }catch (Exception e){
                    return Health.down().withDetail("down的原因：",e.getMessage()).build();
                    }
                }
            }
            ```
             运行“http://localhost:8080/health”
            之后就会出现一些列健康指示。有时候有的浏览器仅仅出现{"status":"DOWN"},为什么会是这样呢？官网给了我们答案
            ```
            Shows application health information (when the application is secure, a simple ‘status’ when accessed over an unauthenticated connection or full message details when authenticated)
            而且要求Sensitive Default 为flase
            ```

参考：

https://segmentfault.com/a/1190000004319673

https://segmentfault.com/a/1190000004318360

https://blog.csdn.net/zengys21/article/details/51066744

https://gtwlover.gitee.io/2017/12/02/Spring%20Boot/

https://www.jianshu.com/p/ddd4fc57a19c