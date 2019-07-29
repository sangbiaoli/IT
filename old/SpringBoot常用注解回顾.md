## SpringBoot常用注解回顾

* @SpringBootApplication:

    包含@Configuration、@EnableAutoConfiguration、@ComponentScan
    通常用在主类上。

* @Repository:

    用于标注数据访问组件，即DAO组件。

* @Service:

    用于标注业务层组件。

* @RestController:

    用于标注控制层组件(如struts中的action)，包含@Controller和@ResponseBody。

* @ResponseBody：

    表示该方法的返回结果直接写入HTTP response body中
    一般在异步获取数据时使用，在使用@RequestMapping后，返回值通常解析为跳转路径，加上@responsebody后返回结果不会被解析为跳转路径，而是直接写入HTTP response body中。比如异步获取json数据，加上@responsebody后，会直接返回json数据。

* @Component：

    泛指组件，当组件不好归类的时候，我们可以使用这个注解进行标注。
    ```java
    @Componentpublic 
    public class SpringContextUtil implements ApplicationContextAware {
        public static ApplicationContext applicationContext;
    }
    ```

* @ComponentScan：

    组件扫描。个人理解相当于<context:component-scan>，如果扫描到有
    @Component @Controller @Service等这些注解的类，则把这些类注册为bean。

    ```java
    @ComponentScan(basePackages = { "org.activiti.rest.diagram","com.qq", "com.baidu" }) // 扫描注解
    public class Application {
        public static void main(String[] args) {
            new  SpringApplicationBuilder(Application.class).web(true).run(args);
        }
    }
    ```
* @Configuration：

    指出该类是 Bean 配置的信息源，相当于XML中的<beans></beans>，一般加在主类上。

    ```java
    /**在线API 接口。**/
    @Configuration
    @EnableSwagger2
    /** http://localhost:8080/swagger-ui.html **/
    public class Swagger2Configuration {

    }
    ```
* @Bean:

    相当于XML中的<bean></bean>,放在方法的上面，而不是类，意思是产生一个bean,并交给spring管理。
    ```java
    @Configuration
    @EnableSwagger2
    /** http://localhost:8080/swagger-ui.html **/
    public class Swagger2Configuration {
        @Bean
        public Docket createRestApi() {
            return new Docket(DocumentationType.SWAGGER_2).apiInfo(apiInfo()).select().apis(RequestHandlerSelectors.any()).paths(PathSelectors.any()).build();
        }
    ```
* @EnableAutoConfiguration：

    让 Spring Boot 根据应用所声明的依赖来对 Spring 框架进行自动配置，一般加在主类上。
    ```java
    @EnableAutoConfiguration(exclude = {org.activiti.spring.boot.SecurityAutoConfiguration.class })
    @ComponentScan(basePackages = { "org.activiti.rest.diagram","com.qq", "com.baidu" }) // 扫描注解
    public class Application {
        public static void main(String[] args) {
            new SpringApplicationBuilder(Application.class).web(true).run(args);
        }
    }
    ```
* @AutoWired:

    byType方式。把配置好的Bean拿来用，完成属性、方法的组装，它可以对类成员变量、方法及构造函数进行标注，完成自动装配的工作。

    当加上（required=false）时，就算找不到bean也不报错。
    ```java
    public class RescourcesController {
        @Autowired
        ResourcesService resourcesService;
    }
    ```
* @Qualifier：

    当有多个同一类型的Bean时，可以用@Qualifier("name")来指定。与@Autowired配合使用

* @Resource(name="name",type="type")：

    没有括号内内容的话，默认byName。与@Autowired干类似的事。

* @RequestMapping：

    RequestMapping是一个用来处理请求地址映射的注解，可用于类或方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。
    该注解有六个属性：
    * params:指定request中必须包含某些参数值是，才让该方法处理。
    * headers:指定request中必须包含某些指定的header值，才能让该方法处理请求。
    * value:指定请求的实际地址，指定的地址可以是URI Template 模式
    * method:指定请求的method类型， GET、POST、PUT、DELETE等
    * consumes:指定处理请求的提交内容类型（Content-Type），如application/json,text/html;
    * produces:指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回

        * Accept：text/html,application/xml,application/json

            将按照如下顺序进行produces的匹配 ①text/html ②application/xml ③application/json

        * Accept：application/xml;q=0.5,application/json;q=0.9,text/html

            将按照如下顺序进行produces的匹配 ①text/html ②application/json ③application/xml

            q参数为媒体类型的质量因子，越大则优先权越高(从0到1)

        * Accept：*/*,text/*,text/html

            将按照如下顺序进行produces的匹配 ①text/html ②text/* ③*/*

    ```java
    @RequestMapping(value = "index.do", method = RequestMethod.GET)
    public String index(ModelMap map){
        return "/system/resources/index";
    }
    ```

* @RequestParams：

    用在方法上包含一组参数说明。

* @RequestParam：

    用在@ApiImplicitParams注解中，指定一个请求参数的各个方面

    * paramType：参数放在哪个地方

        参数类型 body、path、query、header、form中的一种

    * header-->请求参数的获取：@RequestHeader

        使用@RequestHeader接收数据

    * query-->请求参数的获取：@RequestParam

        普通查询参数 例如 ?query=q ,jquery ajax中data设置的值也可以，例如 {query:”q”},springMVC中不需要添加注解接收

    * path（用于restful接口）-->请求参数的获取：@PathVariable

        在url中配置{}的参数

    * body（不常用）

        使用@RequestBody接收数据 POST有效

    * form（不常用）

        笔者未使用，请查看官方API文档

    * name：参数名

    * dataType：参数类型

        数据类型，如果类型名称相同，请指定全路径，例如 dataType = “java.util.Date”，springfox会自动根据类型生成模型

    * required：参数是否必须传

        是否必须 boolean

    * value：参数的意思

        备注说明

    * defaultValue：参数的默认值

    * name 对应方法中接收参数名称

    ```java
    @ApiOperation(value = "用户列表-分页", notes = "", produces = MediaType.APPLICATION_JSON_VALUE)	@ApiImplicitParams({		
        @ApiImplicitParam(name = "pageNow", value = "当前页数", required = true, dataType = "java.lang.Integer"),
        @ApiImplicitParam(name = "qvo", value = "用户查询条件", required = false, dataType = "UserQvo")})
        @ResponseBody
        @RequestMapping(value = "list.do", method = RequestMethod.POST)	
        public Page list(@ModelAttribute UserQuery qvo, @ApiIgnore ModelMap map,@RequestParam(name = "pageNow", defaultValue = "0", required = false) Integer pageNow) throws Exception{
            qvo.setBlurred(true);
            Page page = new Page();
            page.setCur
    ```
* @ModelAttribute

    接收对象参数
    ```java
    @RequestMapping(value = "export.do", method = RequestMethod.GET)
    public void export(@ModelAttribute UserQuery qvo){

    }

* @Api：

    用在类上，说明该类的作用
    ```java
    @Api(value = "用户管理API", tags = "用户管理API")
    @Controller@RequestMapping(value = "/user/user_do/")
    public class UserController {
        @Autowired
        UserService us
    }
    ```
* @ApiOperation：

    用在方法上，说明方法的作用  value：api名称 notes：备注说明
    ```java
    @ApiOperation(value = "菜单管理首页", notes = "需要授权才可有访问的页面", produces = MediaType.TEXT_HTML_VALUE)@RequestMapping(value = "index.do", method = RequestMethod.GET)
    public String index(ModelMap map){
        return "/system/resources/index";
    }
    ```
* @ApiResponses：

    用于表示一组响应

* @ApiResponse：

    用在@ApiResponses中，一般用于表达一个错误的响应信息

    * code：数字，例如400
    * message：信息，例如"请求参数没填好"
    * response：抛出异常的类

* @ApiModel：

    描述一个Model的信息（这种一般用在post创建的时候，使用@RequestBody这样的场景，请求参数无法使用
    ```java
    @ApiModel(value = "用户实体类DO")
    public class RoleDO implements Serializable{
    }
    ```
* @ApiImplicitParam

    注解进行描述的时候
    ```java
    @ApiOperation(value = "获取单个资源", notes = "", produces = MediaType.APPLICATION_JSON_VALUE)
    @ApiImplicitParam(value = "授权资源ID", required = true, dataType = "java.lang.String")
    @ResponseBody
    @RequestMapping(value = "get.do", method = RequestMethod.GET)
    public ResponseEntity<Map<String,Object>> get(@RequestParam(name = "id", required = true) String id) throws Exception{
        Map<String, Object> result = new HashMap<String, Object>();
        SysResourcesVO res = resourcesService.get(id);
        result.put("menu", res);
        ResponseEntity<Map<String, Object>> bean = new ResponseEntity<Map<String,Object>>(result, HttpStatus.OK);return bean;
    }
    ```
* @ApiModelProperty

    对模型中属性添加说明，例如 上面的PageInfoBeen、BlogArticleBeen这两个类中使用，只能使用在类中。

    * value 参数名称
    * required 是否必须 boolean
    * hidden 是否隐藏 boolean 

    其他信息和上面同名属性作用相同，hidden属性对于集合不能隐藏，目前不知道原因
    ```java
    @ApiModelProperty(name = "id", value = "角色ID", dataType = "java.lang.String", hidden = false)
    private String id;
    ```

* @PathVariable:

    路径变量。参数与大括号里的名字一样要相同。
    ```java
    RequestMapping("user/get/mac/{macAddress}")
    public String getByMacAddress(@PathVariable String macAddress){　　
        //do something;
    }
    ```
* @Profiles

    Spring Profiles提供了一种隔离应用程序配置的方式，并让这些配置只能在特定的环境下生效。
    任何@Component或@Configuration都能被@Profile标记，从而限制加载它的时机。
    ```java
    @Configuration@Profile("prod")
    public class ProductionConfiguration {
        // ...
    }
    ```
* @ConfigurationProperties

    Spring Boot将尝试校验外部的配置，默认使用JSR-303（如果在classpath路径中）。
    你可以轻松的为你的@ConfigurationProperties类添加JSR-303 javax.validation约束注解：
    ```java
    @Component@ConfigurationProperties(prefix="connection")
    public class ConnectionSettings {
        @NotNull
        private InetAddress remoteAddress;
        // ... getters and setters
    }
    ```


全局异常处理

* @ControllerAdvice：

    包含@Component。可以被扫描到。

    统一处理异常。

* @ExceptionHandler（Exception.class）：

    用在方法上面表示遇到这个异常就执行以下方法。

    ```java
    @RestControllerAdvicepublic class MyExceptionHandler { 	
        private Logger logger = LoggerFactory.getLogger(getClass());

        /** 自定义异常处理 */
        @ExceptionHandler(MyException.class)
        public R handlerMyException(MyException e){
            R r = new R();
            r.put("code", e.getCode());
            r.put("msg", e.getMessage());
            return r;
        }
    }
    ```

参考：

https://blog.csdn.net/IT_lyd/article/details/78037105