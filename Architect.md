## 成为顶尖架构师必须要面对的面试题
1. 数据结构与算法基础 

    * 说一下几种常见的排序算法和分别的复杂度。 

    * 用Java写一个冒泡排序算法 

    * 描述一下链式存储结构。 

    * 如何遍历一棵二叉树？ 

    * 倒排一个LinkedList。 

    * 用Java写一个递归遍历目录下面的所有文件。 

2. Java基础

    * 接口与抽象类的区别？ 
        1. 抽象类

            抽象类是用来捕捉子类的通用特性的 。它不能被实例化，只能被用作子类的超类。抽象类是被用来创建继承层级里子类的模板。以JDK中的GenericServlet为例：
            ```java
            public abstract class GenericServlet implements Servlet, ServletConfig, Serializable {
                // abstract method
                abstract void service(ServletRequest req, ServletResponse res);

                void init() {
                    // Its implementation
                }
                // other method related to Servlet
            }
            ```
            当HttpServlet类继承GenericServlet时，它提供了service方法的实现：
            ```java
            public class HttpServlet extends GenericServlet {
                void service(ServletRequest req, ServletResponse res) {
                    // implementation
                }

                protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
                    // Implementation
                }

                protected void doPost(HttpServletRequest req, HttpServletResponse resp) {
                    // Implementation
                }

                // some other methods related to HttpServlet
            }
            ```
        2. 接口

            接口是抽象方法的集合。如果一个类实现了某个接口，那么它就继承了这个接口的抽象方法。这就像契约模式，如果实现了这个接口，那么就必须确保使用这些方法。接口只是一种形式，接口自身不能做任何事情。以Externalizable接口为例：
            ```java
            public interface Externalizable extends Serializable {

                void writeExternal(ObjectOutput out) throws IOException;

                void readExternal(ObjectInput in) throws IOException, ClassNotFoundException;
            }
            ```
            当你实现这个接口时，你就需要实现上面的两个方法：
            ```java
            public class Employee implements Externalizable {

                int employeeId;
                String employeeName;

                @Override
                public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
                    employeeId = in.readInt();
                    employeeName = (String) in.readObject();

                }

                @Override
                public void writeExternal(ObjectOutput out) throws IOException {

                    out.writeInt(employeeId);
                    out.writeObject(employeeName);
                }
            }
            ```
        3. 抽象类和接口的对比

            参数|抽象类|接口
            --|--|--
            默认的方法实现|它可以有默认的方法实现|接口完全是抽象的。它根本不存在方法的实现
            实现|子类使用extends关键字来继承抽象类。如果子类不是抽象类的话，它需要提供抽象类中所有声明的方法的实现。|子类使用关键字implements来实现接口。它需要提供接口中所有声明的方法的实现
            构造器|抽象类可以有构造器|接口不能有构造器
            与正常Java类的区别|除了你不能实例化抽象类之外，它和普通Java类没有任何区别|接口是完全不同的类型
            访问修饰符|抽象方法可以有public、protected和default这些修饰符|接口方法默认修饰符是public。你不可以使用其它修饰符。
            main方法|抽象方法可以有main方法并且我们可以运行它|接口没有main方法，因此我们不能运行它。
            多继承|抽象方法可以继承一个类和实现多个接口|接口只可以继承一个或多个其它接口
            速度|它比接口速度要快|接口是稍微有点慢的，因为它需要时间去寻找在类中实现的方法。
            添加新方法|如果你往抽象类中添加新的方法，你可以给它提供默认的实现。因此你不需要改变你现在的代码。|如果你往接口中添加方法，那么你必须改变实现该接口的类。
        4. 什么时候使用抽象类和接口

            * 如果你拥有一些方法并且想让它们中的一些有默认实现，那么使用抽象类吧。
            * 如果你想实现多重继承，那么你必须使用接口。由于Java不支持多继承，子类不能够继承多个类，但可以实现多个接口。因此你就可以使用接口来解决它。
            * 如果基本功能在不断改变，那么就需要使用抽象类。如果不断改变基本功能并且使用接口，那么就需要改变所有实现了该接口的类。

        参考：http://www.importnew.com/12399.html

    * Java中的异常有哪几类？分别怎么使用？ 

        * 异常类有分为编译时异常和运行时异常
            1. 编译时异常:写代码的时候就会提醒你有异常

                * IOException
                * SQLException
                * CloneNotSupportedException
                * parseException

            2. 运行时异常:java.lang.RuntimeException,运行的时候会在控制台提示异常

                * NullPointerException: 空指针异常,一般出现于数组,空对象的变量和方法
                * ArrayIndexOutOfBoundsException: 数组越界异常
                * ArrayStoreException: 数据存储异常
                * NoClassDefFoundException: java运行时系统找不到所引用的类
                * ArithmeticException: 算数异常,一般在被除数是0中
                * ClassCastException: 类型转换异常
                * IllegalArgumentException: 非法参数异常
                * IllegalThreadStateException: 非法线程状态异常
                * NumberFormatException: 数据格式异常
                * OutOfMemoryException: 内存溢出异常
                * PatternSyntaxException: 正则异常

            3. 自定义异常:

                自定义一个类,继承某个异常类
                * 如果继承的是Exception那么就定义了一个编译时异常
                * 如果继承的是RuntimeException或者其子类,那么就定义了一个运行时异常

        * 怎么使用
            1. 一种是在方法中声名异常,谁调用就把异常抛给谁

            2. 一种是使用try{}..catch{}块处理异常
                * 如果多个异常处理的方式不同,可以用多个catch处理
                * 如果所有异常处理方式一样,可以捕获一个父类异常进行统一的处理
                * 如果多个异常分成了不同的组,那么同一组异常之间可以使用|隔开(jdk1.7开始)
                * jdk1.7还增加了增强tr(){}catch(){},通常用于自动关流

        * 异常知识扩展
            1. Throwable类是所有异常的超类,有两个子类,分为Error和Exception
                * Error:错误是无法处理的,只能更改代码,就像一个人得癌症一样
                * Exception:异常是可以处理的,就像是感冒一样,吃药就能好

            2. 在方法重写的时候
                * 子类抛出的编译时异常不能超过父类编译时异常范围
                * 子类不能抛出比父类更多的编译时异常(这里是指抛出异常的范围不能更大,但个数可以更多)
                编译时异常随你抛

        参考：https://blog.csdn.net/JetaimeHQ/article/details/83031899

    * 常用的集合类有哪些？比如List如何排序？ 

    * ArrayList和LinkedList内部的实现大致是怎样的？他们之间的区别和优缺点？ 

    * 内存溢出是怎么回事？请举一个例子？ 

    * ==和equals的区别？ 

    * hashCode方法的作用？ 

    * NIO是什么？适用于何种场景？ 

    * HashMap实现原理，如何保证HashMap的线程安全？ 

    * JVM内存结构，为什么需要GC？ 

    * NIO模型，select/epoll的区别，多路复用的原理 

    * Java中一个字符占多少个字节，扩展再问int, long, double占多少字节 

    * 创建一个类的实例都有哪些办法？ 

    * final/finally/finalize的区别？ 

    * Session/Cookie的区别？ 

    * String/StringBuffer/StringBuilder的区别，扩展再问他们的实现？ 

    * Servlet的生命周期？ 

    * 如何用Java分配一段连续的1G的内存空间？需要注意些什么？ 

    * Java有自己的内存回收机制，但为什么还存在内存泄露的问题呢？ 

    * 什么是java序列化，如何实现java序列化?(写一个实例)？ 

    * String s = new String("abc");创建了几个 String Object? 

3. JVM 

    * JVM堆的基本结构。 

    * JVM的垃圾算法有哪几种？CMS垃圾回收的基本流程？ 

    * JVM有哪些常用启动参数可以调整，描述几个？ 

    * 如何查看JVM的内存使用情况？ 

    * Java程序是否会内存溢出，内存泄露情况发生？举几个例子。 

    * 你常用的JVM配置和调优参数都有哪些？分别什么作用？ 

    * JVM的内存结构？ 

    * 常用的GC策略，什么时候会触发YGC，什么时候触发FGC？ 

4. 多线程/并发 

    * 如何创建线程？如何保证线程安全？ 

    * 如何实现一个线程安全的数据结构 

    * 如何避免死锁 

    * Volatile关键字的作用？ 

    * HashMap在多线程环境下使用需要注意什么？为什么？ 

    * Java程序中启动一个线程是用run还是start？ 

    * 什么是守护线程？有什么用？ 

    * 什么是死锁？如何避免 

    * 线程和进程的差别是什么？ 

    * Java里面的Threadlocal是怎样实现的？ 

    * ConcurrentHashMap的实现原理是？ 

    * sleep和wait区别 

    * notify和notifyAll区别 

    * volatile关键字的作 

    * ThreadLocal的作用与实现 

    * 两个线程如何串行执行 

    * 上下文切换是什么含义 

    * 可以运行时kill掉一个线程吗？ 

    * 什么是条件锁、读写锁、自旋锁、可重入锁？ 

    * 线程池ThreadPoolExecutor的实现原理？ 

5. Linux使用与问题分析排查 

    * 使用两种命令创建一个文件？ 

    * 硬链接和软链接的区别？ 

    * Linux常用命令有哪些？ 

    * 怎么看一个Java线程的资源耗用？ 

    * Load过高的可能性有哪些？ 

    * /etc/hosts文件什么做用？ 

    * 如何快速的将一个文本中所有“abc”替换为“xyz”？ 

    * 如何在log文件中搜索找出error的日志？ 

    * 发现磁盘空间不够，如何快速找出占用空间最大的文件？ 

    * Java服务端问题排查（OOM，CPU高，Load高，类冲突） 

    * Java常用问题排查工具及用法（top, iostat, vmstat, sar, tcpdump, jvisualvm, jmap, jconsole） 

    * Thread dump文件如何分析（Runnable，锁，代码栈，操作系统线程ID关联） 

    * 如何查看Java应用的线程信息？ 

6. 框架使用 

    * 描述一下Hibernate的三个状态？ 

    * Spring中Bean的生命周期。 

    * SpringMVC或Struts处理请求的流程。 

    * Spring AOP解决了什么问题？怎么实现的？ 

    * Spring事务的传播属性是怎么回事？它会影响什么？ 

    * Spring中BeanFactory和FactoryBean有什么区别？ 

    * Spring框架中IOC的原理是什么？ 

    * spring的依赖注入有哪几种方式 

    * struts工作流程 

    * 用Spring如何实现一个切面？ 

    * Spring 如何实现数据库事务？ 

    * Hibernate对一二级缓存的使用，Lazy-Load的理解； 

    * mybatis如何实现批量提交？ 

7. 数据库相关 

    * MySQL InnoDB、Mysaim的特点？ 

    * 乐观锁和悲观锁的区别？ 

    * 数据库隔离级别是什么？有什么作用？ 

    * MySQL主备同步的基本原理。 

    * select * from table t where size > 10 group by size order by size的sql语句执行顺序？ 

    * 如何优化数据库性能（索引、分库分表、批量操作、分页算法、升级硬盘SSD、业务优化、主从部署） 

    * SQL什么情况下不会使用索引（不包含，不等于，函数） 

    * 一般在什么字段上建索引（过滤数据最多的字段） 

    * 如何从一张表中查出name字段不包含“XYZ”的所有行？ 

    * MySQL，B+索引实现，行锁实现，SQL优化 

    * Redis，RDB和AOF，如何做高可用、集群 

    * 如何解决高并发减库存问题 

    * mysql存储引擎中索引的实现机制； 

    * 数据库事务的几种粒度； 

    * 行锁，表锁；乐观锁，悲观锁 

8. 网络协议和网络编程 

    * TCP建立连接的过程。 

    * TCP断开连接的过程。 

    * 浏览器发生302跳转背后的逻辑？ 

    * HTTP协议的交互流程。HTTP和HTTPS的差异，SSL的交互流程？ 

    * Rest和Http什么关系？大家都说Rest很轻量，你对Rest风格如何理解？ 

    * TCP的滑动窗口协议有什么用？讲讲原理。 

    * HTTP协议都有哪些方法？ 

    * 交换机和路由器的区别？ 

    * Socket交互的基本流程？ 

    * 协议（报文结构，断点续传，多线程下载，什么是长连接） 

    * tcp协议（建连过程，慢启动，滑动窗口，七层模型） 

    * webservice协议（wsdl/soap格式，与rest协议的区别） 

    * NIO的好处，Netty线程模型，什么是零拷贝 

9. Redis等缓存系统/中间件/NoSQL/一致性Hash等 

    * 列举一个常用的Redis客户端的并发模型。 

    * HBase如何实现模糊查询？ 

    * 列举一个常用的消息中间件，如果消息要保序如何实现？ 

    * 如何实现一个Hashtable？你的设计如何考虑Hash冲突？如何优化？ 

    * 分布式缓存，一致性hash 

    * LRU算法，slab分配，如何减少内存碎片 

    * 如何解决缓存单机热点问题 

    * 什么是布隆过滤器，其实现原理是？ False positive指的是？ 

    * memcache与redis的区别 

    * zookeeper有什么功能，选举算法如何进行 

    * map/reduce过程，如何用map/reduce实现两个数据源的联合统计 

10. 设计模式与重构 

    * 你能举例几个常见的设计模式 

    * 你在设计一个工厂的包的时候会遵循哪些原则？ 

    * 你能列举一个使用了Visitor/Decorator模式的开源项目/库吗？ 

    * 你在编码时最常用的设计模式有哪些？在什么场景下用？ 

    * 如何实现一个单例？ 

    * 代理模式（动态代理） 

    * 单例模式（懒汉模式，恶汉模式，并发初始化如何解决，volatile与lock的使用） 

    * JDK源码里面都有些什么让你印象深刻的设计模式使用，举例看看？ 