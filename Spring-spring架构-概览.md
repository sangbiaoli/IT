##　spring架构概览

Spring使创建Java企业应用程序变得很容易。它提供了在企业环境中使用Java语言所需要的一切，支持Groovy和Kotlin作为JVM上的替代语言，并根据应用程序的需求灵活地创建多种体系结构。

Spring是开源的。它拥有一个大型且活跃的社区，该社区基于各种实际用例提供持续的反馈。这帮助Spring在很长一段时间内成功地进化。

1. spring的含义

    “spring”一词在不同的上下文中有不同的含义。它一开始用来引用Spring Framework项目本身。随着时间的推移，其他Spring项目已经构建在Spring框架之上。大多数情况下，当人们说“spring”，他们指的是整个spring家族的项目。

    spring架构分成众多模块。应用可以选择它们所需的模块。核心是核心容器的模块，包括**配置模型和依赖注入机制** 。除此之外，Spring框架还为不同的应用程序体系结构提供了基础支持，包括**消息传递、事务数据和持久性以及web** 。它还包括基于servlet的Spring MVC web框架，以及与之并行的Spring WebFlux反应性web框架。

2. spring和spring架构的历史

    Spring是在2003年作为对早期J2EE规范复杂性的响应而出现的。虽然有些人认为Java EE和Spring是竞争对手，但Spring实际上是Java EE的补充。Spring编程模型不包含Java EE平台规范;相反，它集成了从EE保护伞中精心选择的各个规范:

    * Servlet API ([JSR 340](https://jcp.org/en/jsr/detail?id=340))
    * WebSocket API ([JSR 356](https://jcp.org/en/jsr/detail?id=356))
    * Concurrency Utilities ([JSR 236](https://jcp.org/en/jsr/detail?id=236))
    * JSON Binding API ([JSR 367](https://jcp.org/en/jsr/detail?id=367))
    * Bean Validation ([JSR 303](https://jcp.org/en/jsr/detail?id=303))
    * JPA ([JSR 338](https://jcp.org/en/jsr/detail?id=338))
    * JMS ([JSR 914](https://jcp.org/en/jsr/detail?id=914))
    * JTA/JCA

    Spring框架还支持依赖项注入(JSR 330)和公共注释(JSR 250)规范，应用程序开发人员可以选择使用这些规范，而不是Spring框架提供的特定于Spring的机制。

    从Spring Framework 5.0开始，Spring至少需要Java EE 7级别(例如Servlet 3.1+、JPA 2.1+)，同时在运行时提供与Java EE 8级别(例如Servlet 4.0、JSON绑定API)较新的API的开箱即用集成。这使得Spring与Tomcat 8和Tomcat 9、WebSphere 9和JBoss EAP 7完全兼容。

3. 设计原则

    学习一个架构，不仅要知道它能做什么还要知道它所遵循的原则，下面是Spring架构的原则

    * 在每个级别上提供选择，Spring允许您尽可能推迟设计决策。
    * 容纳不同的观点，Spring支持灵活性，并且不关心应该如何做。
    * 保持强大的向后兼容性，Spring的发展已经被小心地管理，以在版本之间强制进行很少的破坏性更改。
    * 关心API设计，Spring团队投入了大量的思想和时间来开发直观的api，并支持多个版本和许多年。
    * 代码质量高标准，Spring框架非常强调有意义的、当前的和准确的javadoc。