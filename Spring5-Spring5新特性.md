## Spring Framework 5.0 新特性

Spring Framework 5.0是在Spring Framework 4.0之后将近四年内一次重大的升级。 在这个时间框架内，主要的发展之一就是Spring Boot项目的演变。

Spring Framework 5.0
Spring Framework 5.0的最大特点之一是**响应式编程（Reactive Programming）**。 响应式编程核心功能和对响应式endpoints的支持可通过Spring Framework 5.0中获得。 重要变动如下列表所示：

* 常规升级
* 对JDK 9运行时兼容性
* 在Spring Framework代码中使用JDK 8特性
* 响应式编程支持
* 函数式Web框架
* Jigsaw的Java模块化
* 对Kotlin支持
* 舍弃的特性

1. 常规升级

    Spring Framework 5.0遵守JDK 8和Java EE 7规范。 基本上，这意味着以前的JDK和Java EE版本不再受支持了。

    Spring Framework 5.0的一些重要基于Java EE 7规范如下所示：

    * Servlet 3.1
    * JMS 2.0
    * JPA 2.1
    * JAX-RS 2.0
    * Bean Validation 1.1

    对于几个Java框架的最低支持版本有很多变化。下面的列表包含了一些重要框架的最低支持版本：

    * Hibernate 5
    * Jackson 2.6
    * EhCache 2.10
    * JUnit 5
    * Tiles 3

    下面列表显示了受支持的服务器对应的版本：

    * Tomcat 8.5+
    * Jetty 9.4+
    * WildFly 10+
    * Netty 4.1+ (for web reactive programming with Spring Web Flux)
    * Undertow 1.4+ (for web reactive programming with Spring Web Flux)

    使用早期版本的任何前述规范/框架的应用程序需要在使用Spring Framework 5.0之前至少升级到上边列出的版本。

2. 对JDK 9运行时兼容性

    JDK 9预计将于2017年年中发布。Spring Framework 5.0期望与JDK 9运行时保持兼容性。

    Tips
    关于Java 9的发布时间，定在9月末，关于更多关于Java 9 的知识，可以访问：Java 9揭秘

3. 在Spring Framework代码中使用JDK 8特性

    Spring Framework 4.x的基准版本是Java SE 6。这意味着它支持Java 6，7和8。必须支持Java SE 6和7对Spring Framework代码的约束。 框架代码不能使用Java 8中的任何新功能。所以，当世界其他地方升级到Java 8时，Spring Framework中的代码（至少主要部分）仅限于使用早期版本的Java。

    **使用Spring Framework 5.0，基准版本是Java 8**。Spring Framework代码现在已升级为使用Java 8中的新特性。会改进更可读和更有效的框架代码。 使用的一些Java 8特性如下：

    * 核心Spring接口中的Java 8 static 方法
    * 基于Java 8反射增强的内部代码改进
    * 在框架代码中使用函数式编程——lambdas表达式和stream流

4. 响应式编程支持

    响应式编程是Spring Framework 5.0最重要的功能之一。

    微服务通常基于事件通信的架构构建。 应用程序被设计为对事件（或消息）做出反应。

    响应式编程提供了一种可选的编程风格，专注于构建应对事件的应用程序。

    虽然Java 8没有内置的响应式性编程支持，但是有一些框架提供了对响应式编程的支持：

    * Reactive Streams：尝试定义与语言无关的响应性API。
    * Reactor：Spring Pivotal团队提供的响应式编程的Java实现。
    * Spring WebFlux：启用基于响应式编程的Web应用程序的开发。 提供类似于Spring MVC的编程模型。

5. 函数式Web框架

    除了响应式特性之外，Spring 5还提供了一个函数式Web框架。

    函数式Web框架提供了使用函数式编程风格来定义endpoints的功能。 这里显示一个简单的hello world示例：

    ```java
    RouterFunction<String> route =
        route(GET("/hello-world"),
        request -> Response.ok().body(fromObject("Hello World")));
    ```

    函数式Web框架也可用于定义更复杂的路由（routes），如以下示例所示：

    ```java
    RouterFunction<?> route = route(GET("/todos/{id}"),
        request -> {
            Mono<Todo> todo = Mono.justOrEmpty(request.pathVariable("id"))
            .map(Integer::valueOf)
            .then(repository::getTodo);
            return Response.ok().body(fromPublisher(todo, Todo.class));
        })
        .and(route(GET("/todos"),
        request -> {
            Flux<Todo> people = repository.allTodos();
            return Response.ok().body(fromPublisher(people, Todo.class));
        }))
        .and(route(POST("/todos"),
        request -> {
            Mono<Todo> todo = request.body(toMono(Todo.class));
            return Response.ok().build(repository.saveTodo(todo));
        }));
    ```

    需要注意的几件重要事项如下：

    RouterFunction评估匹配条件，以将请求路由到适当的处理程序函数。
    定义了三个endpoints，两个GET请求，一个POST请求，并将它们映射到不同的处理函数。

6. Jigsaw的Java模块化

    在Java 8之前，Java平台不是模块化的。因此存在一些重要的问题：

    * **Platform Bloa** Java模块化在过去的几十年中并没有引起人们的关注。然而，通过物联网(IOT-Internet of Things)和新的轻量级平台，如 Node.js迫切需要解决Java平台的膨胀问题。(最初版本的JDK的大小小于10 MB。最近的JDK版本需要超过200 MB。)
    * **JAR Hell** 当ClassLoader找到一个类时，它不会看到是否有其他可用类的定义。 它立即加载找到的第一个类。 如果应用程序的两个不同部分需要来自不同jar包的同一个类，那么它们就无法指定要加载类的jar包。

    Open System Gateway initiative(OSGi)是在1999年开始的一项计划，它将模块化引入到Java应用程序中。

    每个模块（module ）定义如下：

    * import 模块使用的其他包
    * export 模块导出的包

    每个模块都可以有自己的生命周期。 它可以自行安装，启动和停止。

    Jigsaw项目是Java Community Process（JCP）下的一个倡议，从Java 7开始，将模块化纳入Java。 它有两个主要目标：

    * 定义和实现JDK的模块化结构
    * 为Java平台上构建的应用程序定义模块系统

    Jigsaw将成为Java 9的一部分，Spring Framework 5.0将包含对Jigsaw模块的基本支持。

7. 对Kotlin支持

    Kotlin是一种静态类型的JVM语言，可以实现具有更好的表达性，简洁性和可读性的代码。 Spring框架5.0对Kotlin有很好的支持。

    考虑一个简单的Kotlin程序，说明一个data类，如下所示：

    ```js
    import java.util.*
        data class Todo(var description: String, var name: String, var  
        targetDate : Date)
        fun main(args: Array<String>) {
            var todo = Todo("Learn Spring Boot", "Jack", Date())
            println(todo)
                //Todo(description=Learn Spring Boot, name=Jack,
                //targetDate=Mon May 22 04:26:22 UTC 2017)
            var todo2 = todo.copy(name = "Jill")
            println(todo2)
                //Todo(description=Learn Spring Boot, name=Jill,
                //targetDate=Mon May 22 04:26:22 UTC 2017)
            var todo3 = todo.copy()
            println(todo3.equals(todo)) //true
        }
    ```

    在不到10行代码中，我们创建并测试了一个具有三个属性和以下方法的数据bean：

    * equals()
    * hashCode()
    * toString()
    * copy()

    Kotlin是强类型的。 但是不需要明确指定每个变量的类型：

    ```js
    val arrayList = arrayListOf("Item1", "Item2", "Item3")
        // Type is ArrayList
    ```

    命名参数允许你在调用方法时指定参数的名称，从而保持更可读的代码：

    ```js
    var todo = Todo(description = "Learn Spring Boot",
        name = "Jack", targetDate = Date())
    ```

    Kotlin通过提供默认变量（it）和诸如take，drop等方法使函数式编程更简单：

    ```js
    var first3TodosOfJack = students.filter { it.name == "Jack"
        }.take(3)
    ```

    还可以为Kotlin中的参数指定默认值：

    ```js
    import java.util.*
        data class Todo(var description: String, var name: String, var
        targetDate : Date = Date())
        fun main(args: Array<String>) {
            var todo = Todo(description = "Learn Spring Boot", name = "Jack")
        }
    ```

    由于它的所有特性使代码简洁而富有表现力，我们希望Kotlin能够成为一门语言。

8. 舍弃的特性

    Spring Framework 5是一个主要的Spring版本，基准大幅度增加。 随着Java，Java EE和其他几个框架的基准版本的增加，Spring Framework 5删除了对几个框架的支持：

    * Portlet
    * Velocity
    * JasperReports
    * XMLBeans
    * JDO
    * Guava
    如果你正在使用上面任何框架，建议计划迁移并使用Spring Framework 4.3——该框架一直支持到2019年。

原文：https://www.jianshu.com/p/cebc3cf0bec0