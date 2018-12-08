## 成为顶尖架构师必须要面对的面试题

1. 数据结构与算法基础

    1. 说一下几种常见的排序算法和分别的复杂度

    2. 用Java写一个冒泡排序算法

    3. 描述一下链式存储结构

    4. 如何遍历一棵二叉树

    5. 倒排一个LinkedList

    6. 用Java写一个递归遍历目录下面的所有文件

2. Java基础

    1. 接口与抽象类的区别？

    2. Java中的异常有哪几类？分别怎么使用？

    3. 常用的集合类有哪些？比如List如何排序？

    4. ArrayList和LinkedList内部的实现大致是怎样的？他们之间的区别和优缺点？

    5. 内存溢出是怎么回事？请举一个例子？

    6. ==和equals的区别？

    7. hashCode方法的作用？

    8. NIO是什么？适用于何种场景？

    9. HashMap实现原理，如何保证HashMap的线程安全？

    10. JVM内存结构，为什么需要GC？

    11. NIO模型，select/epoll的区别，多路复用的原理

    12. Java中一个字符占多少个字节，扩展再问int, long, double占多少字节

    13. 创建一个类的实例都有哪些办法？

    14. final/finally/finalize的区别？

    15. Session/Cookie的区别？

    16. String/StringBuffer/StringBuilder的区别，扩展再问他们的实现？

    17. Servlet的生命周期？

    18. 如何用Java分配一段连续的1G的内存空间？需要注意些什么？

    19. Java有自己的内存回收机制，但为什么还存在内存泄露的问题呢？

    20. 什么是java序列化，如何实现java序列化?(写一个实例)？

    21. String s = new String("abc");创建了几个 String Object?

3. JVM

    1. JVM堆的基本结构。

    2. JVM的垃圾算法有哪几种？CMS垃圾回收的基本流程？

    3. JVM有哪些常用启动参数可以调整，描述几个？

    4. 如何查看JVM的内存使用情况？

    5. Java程序是否会内存溢出，内存泄露情况发生？举几个例子。

    6. 你常用的JVM配置和调优参数都有哪些？分别什么作用？

    7. JVM的内存结构？

    8. 常用的GC策略，什么时候会触发YGC，什么时候触发FGC？

4. 多线程/并发

    1. 如何创建线程？如何保证线程安全？

    2. 如何实现一个线程安全的数据结构

    3. 如何避免死锁

    4. Volatile关键字的作用？

    5. HashMap在多线程环境下使用需要注意什么？为什么？

    6. Java程序中启动一个线程是用run还是start？

    7. 什么是守护线程？有什么用？

    8. 什么是死锁？如何避免

    9. 线程和进程的差别是什么？

    10. Java里面的Threadlocal是怎样实现的？

    11. ConcurrentHashMap的实现原理是？

    12. sleep和wait区别

    13. notify和notifyAll区别

    14. volatile关键字的作

    15. ThreadLocal的作用与实现

    16. 两个线程如何串行执行

    17. 上下文切换是什么含义

    18. 可以运行时kill掉一个线程吗？

    19. 什么是条件锁、读写锁、自旋锁、可重入锁？

    20. 线程池ThreadPoolExecutor的实现原理？

5. Linux使用与问题分析排查

    1. 使用两种命令创建一个文件？

    2. 硬链接和软链接的区别？

    3. Linux常用命令有哪些？

    4. 怎么看一个Java线程的资源耗用？

    5. Load过高的可能性有哪些？

    6. /etc/hosts文件什么做用？

    7. 如何快速的将一个文本中所有“abc”替换为“xyz”？

    8. 如何在log文件中搜索找出error的日志？

    9. 发现磁盘空间不够，如何快速找出占用空间最大的文件？

    10. Java服务端问题排查（OOM，CPU高，Load高，类冲突）

    11. Java常用问题排查工具及用法（top, iostat, vmstat, sar, tcpdump, jvisualvm, jmap, jconsole）

    12. Thread dump文件如何分析（Runnable，锁，代码栈，操作系统线程ID关联）

    13. 如何查看Java应用的线程信息？

6. 框架使用

    1. 描述一下Hibernate的三个状态？

    2. Spring中Bean的生命周期。

    3. SpringMVC或Struts处理请求的流程。

    4. Spring AOP解决了什么问题？怎么实现的？

    5. Spring事务的传播属性是怎么回事？它会影响什么？

    6. Spring中BeanFactory和FactoryBean有什么区别？

    7. Spring框架中IOC的原理是什么？

    8. spring的依赖注入有哪几种方式

    9. struts工作流程

    10. 用Spring如何实现一个切面？

    11. Spring 如何实现数据库事务？

    12. Hibernate对一二级缓存的使用，Lazy-Load的理解； 

    13. mybatis如何实现批量提交？

7. 数据库相关

    1. MySQL InnoDB、Mysaim的特点？

    2. 乐观锁和悲观锁的区别？

    3. 数据库隔离级别是什么？有什么作用？

    4. MySQL主备同步的基本原理。

    5. select 1. from table t where size > 10 group by size order by size的sql语句执行顺序？

    6. 如何优化数据库性能（索引、分库分表、批量操作、分页算法、升级硬盘SSD、业务优化、主从部署）

    7. SQL什么情况下不会使用索引（不包含，不等于，函数）

    8. 一般在什么字段上建索引（过滤数据最多的字段）

    9. 如何从一张表中查出name字段不包含“XYZ”的所有行？

    10. MySQL，B+索引实现，行锁实现，SQL优化

    11. Redis，RDB和AOF，如何做高可用、集群

    12. 如何解决高并发减库存问题

    13. mysql存储引擎中索引的实现机制；

    14. 数据库事务的几种粒度；

    15. 行锁，表锁；乐观锁，悲观锁

8. 网络协议和网络编程

    1. TCP建立连接的过程。

    2. TCP断开连接的过程。

    3. 浏览器发生302跳转背后的逻辑？

    4. HTTP协议的交互流程。HTTP和HTTPS的差异，SSL的交互流程？

    5. Rest和Http什么关系？大家都说Rest很轻量，你对Rest风格如何理解？

    6. TCP的滑动窗口协议有什么用？讲讲原理。

    7. HTTP协议都有哪些方法？

    8. 交换机和路由器的区别？

    9. Socket交互的基本流程？

    10. 协议（报文结构，断点续传，多线程下载，什么是长连接）

    11. tcp协议（建连过程，慢启动，滑动窗口，七层模型）

    12. webservice协议（wsdl/soap格式，与rest协议的区别）

    13. NIO的好处，Netty线程模型，什么是零拷贝

9. Redis等缓存系统/中间件/NoSQL/一致性Hash等

    1. 列举一个常用的Redis客户端的并发模型。

    2. HBase如何实现模糊查询？

    3. 列举一个常用的消息中间件，如果消息要保序如何实现？

    4. 如何实现一个Hashtable？你的设计如何考虑Hash冲突？如何优化？

    5. 分布式缓存，一致性hash

    6. LRU算法，slab分配，如何减少内存碎片

    7. 如何解决缓存单机热点问题

    8. 什么是布隆过滤器，其实现原理是？False positive指的是？

    9. memcache与redis的区别

    10. zookeeper有什么功能，选举算法如何进行

    11. map/reduce过程，如何用map/reduce实现两个数据源的联合统计

10. 设计模式与重构

    1. 你能举例几个常见的设计模式

    2. 你在设计一个工厂的包的时候会遵循哪些原则？

    3. 你能列举一个使用了Visitor/Decorator模式的开源项目/库吗？

    4. 你在编码时最常用的设计模式有哪些？在什么场景下用？

    5. 如何实现一个单例？

    6. 代理模式（动态代理）

    7. 单例模式（懒汉模式，恶汉模式，并发初始化如何解决，volatile与lock的使用）

    8. JDK源码里面都有些什么让你印象深刻的设计模式使用，举例看看？