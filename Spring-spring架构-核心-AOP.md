## AOP

面向方面编程(AOP)通过提供另一种考虑程序结构的方法来补充面向对象编程(OOP)。OOP中模块性的关键单元是类，而AOP中模块性的单元是方面。方面支持跨多个类型和对象的关注点的模块化(例如事务管理)。(在AOP文献中，这样的关注点通常被称为“横切”关注点。)

Spring的一个关键组件是AOP框架。虽然Spring IoC容器不依赖AOP(这意味着如果您不想使用AOP，就不需要使用AOP)，但是AOP补充了Spring IoC，提供了一个非常有用的中间件解决方案。

1. AOP概念

    核心AOP概念和术语

    * Aspect: 跨多个类的关注点的模块化。事务管理是企业Java应用程序中横切关注点的一个很好的例子。在Spring AOP中，方面是通过使用正则类(基于模式的方法)或使用@Aspect注释的正则类(@AspectJ样式)来实现的。

    * Join point: 程序执行过程中的一个点，如方法的执行或异常的处理。在Spring AOP中，连接点总是表示方法执行。

    * Advice: 方面在特定连接点上采取的操作。不同类型的建议包括“around”、“before”及“after”建议。(稍后将讨论通知类型。)许多AOP框架(包括Spring)将通知建模为拦截器，并围绕连接点维护一系列拦截器。

    * Pointcut: 匹配连接点的谓词。通知与切入点表达式相关联，并在切入点匹配的任何连接点上运行(例如，执行具有特定名称的方法)。连接点与切入点表达式匹配的概念是AOP的核心，Spring默认使用AspectJ切入点表达式语言。

    * Introduction: 代表类型声明其他方法或字段。Spring AOP允许您向任何被建议的对象引入新的接口(以及相应的实现)。例如，可以使用介绍使bean实现IsModified接口，以简化缓存。(在AspectJ社区中，介绍称为类型间声明。)

    * Target object: 被一个或多个方面建议的对象。也称为“被通知对象”。由于Spring AOP是通过使用运行时代理实现的，所以这个对象总是一个代理对象。

    * AOP proxy: AOP框架创建的一个对象，用于实现方面契约(建议方法执行等等)。在Spring框架中，AOP代理是JDK动态代理或CGLIB代理。

    * Weaving: 将方面与其他应用程序类型或对象链接，以创建建议的对象。这可以在编译时(例如，使用AspectJ编译器)、加载时或运行时完成。与其他纯Java AOP框架一样，Spring AOP在运行时执行编织。

    Spring AOP包含以下类型的advice

    * Before advice: 在连接点之前运行但不能阻止执行流继续到连接点的通知(除非抛出异常)。

    * After returning advice: 建议在连接点正常完成后运行(例如，如果方法返回时没有抛出异常)。

    * After throwing advice: 如果方法通过抛出异常退出，则要执行的通知。

    * After (finally) advice: 执行通知，无论连接点以何种方式退出(正常或异常返回)。

    * Around advice: 围绕连接点(如方法调用)的通知。这是最有力的建议。Around通知可以在方法调用前后执行自定义行为。它还负责选择是继续到连接点，还是通过返回它自己的返回值或抛出异常来简化建议的方法执行。

2. Spring AOP的功能和目标

    Spring AOP是用纯Java实现的。不需要特殊的编译过程。Spring AOP不需要控制类加载器层次结构，因此适合在servlet容器或应用程序服务器中使用。
    
    Spring AOP目前只支持方法执行连接点(建议在Spring bean上执行方法)。没有实现字段拦截，但是可以在不破坏核心Spring AOP api的情况下添加对字段拦截的支持。如果需要建议字段访问和更新连接点，请考虑AspectJ之类的语言。
    
    Spring AOP的AOP方法与大多数其他AOP框架不同。其目的不是提供最完整的AOP实现(尽管Spring AOP非常有能力)。相反，其目标是提供AOP实现和Spring IoC之间的紧密集成，以帮助解决企业应用程序中的常见问题。

3. AOP代理

    Spring AOP默认为AOP代理使用标准JDK动态代理。这允许代理任何接口(或一组接口)。

    Spring AOP还可以使用CGLIB代理。这对于代理类而不是接口是必要的。默认情况下，如果业务对象没有实现接口，则使用CGLIB。由于编程到接口而不是类是一种很好的实践，业务类通常实现一个或多个业务接口。

    JDK动态代理是代理模式的一种实现方式，其只能代理接口。

    1. 使用步骤
        1. 新建一个接口
        2. 为接口创建一个实现类
        3. 创建代理类实现java.lang.reflect.InvocationHandler接口
        4. 测试

    2. Demo代码

        新建接口

        ```java
        public interface Subject {

            void doSomething();
        }
        ```

        接口实现类

        ```java
        public class RealSubject implements Subject {
            @Override
            public void doSomething() {
                System.out.println("RealSubject do something");
            }
        }
        ```

        动态代理类

        ```java
        public class JDKDynamicProxy implements InvocationHandler {

            private Object target;

            public JDKDynamicProxy(Object target) {
                this.target = target;
            }

            /**
            * 获取被代理接口实例对象
            * @param <T>
            * @return
            */
            public <T> T getProxy() {
                return (T) Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), this);
            }

            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println("Do something before");
                Object result = method.invoke(target, args);
                System.out.println("Do something after");
                return result;
            }
        }
        ```

        测试类

        ```java
        public class Client {
            public static void main(String[] args) {
                // jdk动态代理测试
                Subject subject = new JDKDynamicProxy(new RealSubject()).getProxy();
                subject.doSomething();
            }
        }
        ```

        测试结果

        ```
        Do something before
        RealSubject do something
        Do something after
        ```

4. @AspectJ support

    @AspectJ指的是将方面声明为使用注解注释的常规Java类的样式。

    @AspectJ样式是AspectJ项目作为AspectJ 5发行版的一部分引入的。

    Spring使用AspectJ提供的用于切入点解析和匹配的库解释与AspectJ 5相同的注释。但是AOP运行时仍然是纯Spring AOP，并且不依赖于AspectJ编译器或编织器。

    1. 开启@AspectJ支持

        要在Spring配置中使用@AspectJ方面，您需要启用Spring支持来基于@AspectJ方面配置Spring AOP，以及基于这些方面是否建议自动代理bean。
        
        通过自动代理，我们的意思是，如果Spring确定一个bean被一个或多个方面通知，它将自动为该bean生成一个代理来拦截方法调用，并确保根据需要执行通知。

        可以通过XML或java风格的配置启用@AspectJ支持。在这两种情况下，还需要确保AspectJ的aspectjweaver.jar库位于应用程序的类路径上(版本1.8或更高)。这个库可以在AspectJ发行版的lib目录中使用，也可以从Maven中央存储库中使用。

        ```java
        @Configuration
        @EnableAspectJAutoProxy
        public class AppConfig {

        }
        ```

        或者xml方式

        ```xml
        <aop:aspectj-autoproxy/>
        ```

    2. 声明一个Aspect(切面)

        启用@AspectJ支持后，在应用程序上下文中使用@AspectJ方面(具有@Aspect注释)类定义的任何bean都会被Spring自动检测到，并用于配置Spring AOP。接下来的两个例子展示了一个不太有用的方面所需要的最小定义。

        ```xml
        <bean id="myAspect" class="org.xyz.NotVeryUsefulAspect">
            <!-- configure properties of the aspect here -->
        </bean>
        ```

        ```java
        package org.xyz;
        import org.aspectj.lang.annotation.Aspect;

        @Aspect
        public class NotVeryUsefulAspect {

        }
        ```

    3. 声明一个PointCut(切入点)

        切入点确定感兴趣的连接点，从而使我们能够控制何时执行通知。
        
        Spring AOP只支持Spring bean的方法执行连接点，所以您可以将切入点看作是匹配Spring bean上方法的执行。
        
        切入点声明由两部分组成:

        * 一个签名包含名称和任何参数，在AOP的@AspectJ注释风格中，切入点签名由一个正则方法定义提供
        * 一个切入点表达式，该表达式确定我们对哪个方法执行感兴趣。，切入点表达式由@Pointcut注释表示(作为切入点签名的方法必须有一个void返回类型)。

        ```java
        @Pointcut("execution(* transfer(..))")// the pointcut expression
        private void anyOldTransfer() {}// the pointcut signature
        ```

        1. 支持切入点指示器

            Spring AOP支持在切入点表达式中使用的以下AspectJ切入点设计器(PCD)

            * execution: 用于**匹配方法**执行的连接点。这是使用Spring AOP时要使用的主要切入点设计器。

            * within: 用于**匹配指定类型**内的方法执行。

            * this: 用于**匹配当前AOP代理对象类型的执行方法**；注意是AOP代理对象的类型匹配，这样就可能包括引入接口也类型匹配

            * target: 用于**匹配当前目标对象类型的执行方法**；注意是目标对象的类型匹配，这样就不包括引入接口也类型匹配

            * args: 用于**匹配当前执行的方法传入的参数为指定类型**的执行方法。

            * @target: 用于匹配当前目标对象类型的执行方法，其中目标对象持有指定的注解

            * @args: 用于匹配当前执行的方法传入的参数持有指定注解的执行

            * @within: 用于匹配所以持有指定注解类型内的方法

            * @annotation: 用于匹配当前执行方法持有指定注解的方法

            * bean：Spring AOP扩展的，AspectJ没有对应指示符，用于匹配特定名称的Bean对象的执行方法；

            * reference pointcut：表示引用其他命名切入点，只有@ApectJ风格支持，Schema风格不支持。

        2. 结合切入点表达式

            可以使用&&、||和!组合切入点表达式。您还可以通过名称引用切入点表达式。下面的例子展示了三个切入点表达式

            ```java
            @Pointcut("execution(public * *(..))")
            private void anyPublicOperation() {} 

            @Pointcut("within(com.xyz.someapp.trading..*)")
            private void inTrading() {} 

            @Pointcut("anyPublicOperation() && inTrading()")
            private void tradingOperation() {} 
            ```

            上面三个切入表达式解释如下：
            
            1. 如果方法执行连接点表示任何公共方法的执行，则anyPublicOperation匹配。
            2. 如果方法执行在trading模块中，则inTrading匹配。
            3. 如果方法执行表示trading模块中的任何公共方法，则tradingOperation匹配。

        3. 共享公共切入点定义

            在处理企业应用程序时，开发人员通常希望从几个方面引用应用程序的模块和特定的操作集。我们建议定义一个“系统体系结构”方面，它捕获用于此目的的公共切入点表达式。这样一个方面通常类似于下面的例子:

            ```java
            package com.xyz.someapp;

            import org.aspectj.lang.annotation.Aspect;
            import org.aspectj.lang.annotation.Pointcut;

            @Aspect
            public class SystemArchitecture {

                /**
                * A join point is in the web layer if the method is defined
                * in a type in the com.xyz.someapp.web package or any sub-package
                * under that.
                */
                @Pointcut("within(com.xyz.someapp.web..*)")
                public void inWebLayer() {}

                /**
                * A join point is in the service layer if the method is defined
                * in a type in the com.xyz.someapp.service package or any sub-package
                * under that.
                */
                @Pointcut("within(com.xyz.someapp.service..*)")
                public void inServiceLayer() {}

                /**
                * A join point is in the data access layer if the method is defined
                * in a type in the com.xyz.someapp.dao package or any sub-package
                * under that.
                */
                @Pointcut("within(com.xyz.someapp.dao..*)")
                public void inDataAccessLayer() {}

                /**
                * A business service is the execution of any method defined on a service
                * interface. This definition assumes that interfaces are placed in the
                * "service" package, and that implementation types are in sub-packages.
                *
                * If you group service interfaces by functional area (for example,
                * in packages com.xyz.someapp.abc.service and com.xyz.someapp.def.service) then
                * the pointcut expression "execution(* com.xyz.someapp..service.*.*(..))"
                * could be used instead.
                *
                * Alternatively, you can write the expression using the 'bean'
                * PCD, like so "bean(*Service)". (This assumes that you have
                * named your Spring service beans in a consistent fashion.)
                */
                @Pointcut("execution(* com.xyz.someapp..service.*.*(..))")
                public void businessService() {}

                /**
                * A data access operation is the execution of any method defined on a
                * dao interface. This definition assumes that interfaces are placed in the
                * "dao" package, and that implementation types are in sub-packages.
                */
                @Pointcut("execution(* com.xyz.someapp.dao.*.*(..))")
                public void dataAccessOperation() {}

            }
            ```

            您可以在任何需要切入点表达式的地方引用在这样一个方面中定义的切入点。例如，要使服务层具有事务性，可以编写以下代码:

            ```xml
            <aop:config>
                <aop:advisor
                    pointcut="com.xyz.someapp.SystemArchitecture.businessService()"
                    advice-ref="tx-advice"/>
            </aop:config>

            <tx:advice id="tx-advice">
                <tx:attributes>
                    <tx:method name="*" propagation="REQUIRED"/>
                </tx:attributes>
            </tx:advice>
            ```

        4. 例子

            Spring AOP用户可能最经常使用执行切入点指示器。执行表达式的格式如下:

            ```
            execution(modifiers-pattern? ret-type-pattern declaring-type-pattern?name-pattern(param-pattern)
            throws-pattern?)
            ```

            一些常用表达式的整理

            表达式|解释
            --|--
            execution(public * *(..))|任何公共方法
            execution(* set*(..))|任何以set开头的方法
            execution(* com.xyz.service.AccountService.*(..))|AccountService接口定义的任何方法
            execution(* com.xyz.service.*.*(..))|service包下的任何方法
            execution(* com.xyz.service..*.*(..))|service包及子包的任何方法
            within(com.xyz.service.*)|service包中的任何连接点(仅在Spring AOP中执行方法)
            within(com.xyz.service..*)|service包及子包中的任何连接点(仅在Spring AOP中执行方法)
            this(com.xyz.service.AccountService)|代理实现AccountService接口的任何连接点(仅在Spring AOP中执行方法)
            target(com.xyz.service.AccountService)|目标对象实现AccountService接口的任何连接点(仅在Spring AOP中执行方法)
            args(java.io.Serializable)|任何连接点(只在Spring AOP中执行方法)，它只接受一个参数，并且在运行时传递的参数是可序列化的
            @target(org.springframework.transaction.annotation.Transactional)|目标对象具有@Transactional注释的任何连接点(仅在Spring AOP中执行方法)
            @within(org.springframework.transaction.annotation.Transactional)|目标对象的声明类型具有@Transactional注释的任何连接点(仅在Spring AOP中执行方法)
            @annotation(org.springframework.transaction.annotation.Transactional)|任何连接点(只在Spring AOP中执行方法)，其中执行方法具有@Transactional注释
            @args(com.xyz.security.Classified)|任何接受单个参数的连接点(仅在Spring AOP中执行方法)，其中传递的参数的运行时类型有@classification注释
            bean(tradeService)|在名为tradeService的Spring bean上的任何连接点(仅在Spring AOP中执行方法)
            bean(\*Service)|在名称匹配通配符表达式*Service的Spring bean上的任何连接点(仅在Spring AOP中执行方法)

        5. 编写好的切入点

            AspectJ只能处理它被告知的内容。为了获得最佳匹配性能，您应该考虑他们试图实现什么，并在定义中尽可能缩小匹配的搜索空间。现有的设计器自然可以分为三组:kinded、scoping和context:
            
            * Kinded设计器选择特定类型的连接点:执行、获取、设置、调用和处理程序。
            * scoping设计器选择一组感兴趣的连接点(可能有很多种):内部和内部代码
            * context指示符根据上下文匹配(可选绑定):this、target和@annotation

    4. 声明Advice(通知)

        通知与切入点表达式相关联，并在切入点匹配的方法执行之前、之后或前后运行。切入点表达式可以是对指定切入点的简单引用，也可以是在适当位置声明的切入点表达式。

        * Before Advice

            ```java
            import org.aspectj.lang.annotation.Aspect;
            import org.aspectj.lang.annotation.Before;

            @Aspect
            public class BeforeExample {

                @Before("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
                public void doAccessCheck() {
                    // ...
                }

            }
            ```

            如果我们使用一个就地切入点表达式，我们可以将前面的例子重写为下面的例子:

            ```java
            import org.aspectj.lang.annotation.Aspect;
            import org.aspectj.lang.annotation.Before;

            @Aspect
            public class BeforeExample {

                @Before("execution(* com.xyz.myapp.dao.*.*(..))")
                public void doAccessCheck() {
                    // ...
                }

            }
            ``` 

        * After Returning Advice

            返回后，当匹配的方法执行正常返回时，将运行通知。

            ```java
            import org.aspectj.lang.annotation.Aspect;
            import org.aspectj.lang.annotation.AfterReturning;

            @Aspect
            public class AfterReturningExample {

                @AfterReturning("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
                public void doAccessCheck() {
                    // ...
                }

            }
            ```

            有时候，您需要在advice主体中访问返回的实际值。您可以使用@ afterreturn的形式绑定返回值来获得访问权限

            ```java
            import org.aspectj.lang.annotation.Aspect;
            import org.aspectj.lang.annotation.AfterReturning;

            @Aspect
            public class AfterReturningExample {

                @AfterReturning(
                    pointcut="com.xyz.myapp.SystemArchitecture.dataAccessOperation()",
                    returning="retVal")
                public void doAccessCheck(Object retVal) {
                    // ...
                }

            }
            ```

        * After Throwing Advice

            抛出通知后，当匹配的方法执行通过抛出异常退出时运行。

            ```java
            import org.aspectj.lang.annotation.Aspect;
            import org.aspectj.lang.annotation.AfterThrowing;

            @Aspect
            public class AfterThrowingExample {

                @AfterThrowing("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
                public void doRecoveryActions() {
                    // ...
                }

            }
            ```

            通常，您希望仅在抛出给定类型的异常时才运行通知，并且常常需要在通知主体中访问抛出的异常。您可以使用throw属性来限制匹配(如果需要的话—否则使用Throwable作为异常类型)，并将抛出的异常绑定到advice参数。

            ```java
            import org.aspectj.lang.annotation.Aspect;
            import org.aspectj.lang.annotation.AfterThrowing;

            @Aspect
            public class AfterThrowingExample {

                @AfterThrowing(
                    pointcut="com.xyz.myapp.SystemArchitecture.dataAccessOperation()",
                    throwing="ex")
                public void doRecoveryActions(DataAccessException ex) {
                    // ...
                }

            }
            ```

        * After (Finally) Advice

            当匹配的方法执行退出时，运行After (finally)通知。它是通过使用@After注释声明的。事后通知必须准备好处理正常和异常返回条件。它通常用于释放资源和类似的目的。

            ```java
            import org.aspectj.lang.annotation.Aspect;
            import org.aspectj.lang.annotation.After;

            @Aspect
            public class AfterFinallyExample {

                @After("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
                public void doReleaseLock() {
                    // ...
                }

            }
            ```

        * Around Advice

            最后一种建议是围绕建议。Around通知“绕过”匹配方法的执行。它有机会在方法执行之前和之后执行工作，并确定何时、如何执行，甚至是否真正执行方法。如果您需要以线程安全的方式(例如，启动和停止计时器)共享方法执行前后的状态，通常会使用Around建议。

            ```java
            import org.aspectj.lang.annotation.Aspect;
            import org.aspectj.lang.annotation.Around;
            import org.aspectj.lang.ProceedingJoinPoint;

            @Aspect
            public class AroundExample {

                @Around("com.xyz.myapp.SystemArchitecture.businessService()")
                public Object doBasicProfiling(ProceedingJoinPoint pjp) throws Throwable {
                    // start stopwatch
                    Object retVal = pjp.proceed();
                    // stop stopwatch
                    return retVal;
                }

            }
            ```
        
        * Advice Parameters

            Spring提供了全类型的通知，这意味着您可以在通知签名中声明所需的参数(正如我们前面看到的用于返回和抛出示例的参数)，而不是一直使用Object[]数组。

            * 访问当前连接点

                任何advice方法都可以将org.aspectj.lang.JoinPoint类型的参数声明为它的第一个参数。JoinPoint接口提供了许多有用的方法:

                * getArgs():返回方法参数。
                * getThis():返回代理对象。
                * getTarget():返回目标对象。
                * getSignature():返回被建议的方法的描述。
                * toString():打印建议的方法的有用描述。

            * 将参数传递给通知

                我们已经看到了如何绑定返回值或异常值(在返回和抛出建议之后使用)。要使参数值对通知主体可用，可以使用args的绑定形式。如果在args表达式中使用参数名代替类型名，则在调用通知时将传递相应参数的值作为参数值。

                ```java
                @Before("com.xyz.myapp.SystemArchitecture.dataAccessOperation() && args(account,..)")
                public void validateAccount(Account account) {
                    // ...
                }
                ```

            * 通知参数和泛型

                ```java
                public interface Sample<T> {
                    void sampleGenericMethod(T param);
                    void sampleGenericCollectionMethod(Collection<T> param);
                }
                ```

                您可以通过将advice参数键入要拦截方法的参数类型，将方法类型的拦截限制为某些参数类型

                ```java
                @Before("execution(* ..Sample+.sampleGenericMethod(*)) && args(param)")
                public void beforeSampleMethod(MyType param) {
                    // Advice implementation
                }
                ```

            * 确定参数的名字

                如果用户显式指定了参数名，则使用指定的参数名。通知和切入点注释都有一个可选的argNames属性，您可以使用该属性指定带注释的方法的参数名。这些参数名在运行时可用。

                ```java
                @Before(value="com.xyz.lib.Pointcuts.anyPublicMethod() && target(bean) && @annotation(auditable)",argNames="bean,auditable")
                public void audit(Object bean, Auditable auditable) {
                    AuditCode code = auditable.value();
                    // ... use code and bean
                }
                ```

        * Advice Ordering

            当多个通知都想在同一个连接点上运行时会发生什么?Spring AOP遵循与AspectJ相同的优先规则来确定通知执行的顺序。
            优先级最高的通知将首先“on The way in”运行(因此，给定两个before通知，优先级最高的通知将首先运行)。
            从连接点“退出”时，优先级最高的通知将最后运行(因此，给定两个after通知，优先级最高的通知将运行在第二位)。

    5. 介绍

    6. 实例化模型方面

    7. AOP样例

5. 基于Schema的AOP支持

    1. 声明一个Aspect(切面)

    2. 声明一个PointCut(切入点)

    3. 声明Advice(通知)

    4. 介绍

    5. 实例化模型方面

    6. Advisors

    7. AOP Schema样例

6. 选择要使用哪种AOP声明样式

    1. Spring AOP还是Full AspectJ?

    2. 对于Spring AOP用@AspectJ还是的XML?

7. 混合Aspect类型

8. 代理机制

    1. 理解AOP代理

9. @AspectJ代理的编程创建

10. 在Spring应用程序中使用AspectJ

    1. Spring中使用AspectJ来依赖注入域对象

    2. AspectJ的其他Spring方面

    3. 使用Spring IoC配置AspectJ方面

    4. 在Spring框架中使用AspectJ进行加载时编织

11. 更多资料

参考：https://docs.spring.io/spring/docs/5.1.6.RELEASE/spring-framework-reference/core.html#aop
http://www.cnblogs.com/zuidongfeng/p/8735241.html
https://blog.csdn.net/qq_23167527/article/details/78623639