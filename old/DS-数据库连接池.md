## 数据库连接池的实现及原理

1. 背景介绍

    对于一个简单的数据库应用，由于对于数据库的访问不是很频繁。这时可以简单地在需要访问数据库时，就新创建一个连接，用完后就关闭它，这样做也不会带来什么明显的性能上的开销。但是对于一个复杂的数据库应用，情况就完全不同了。频繁的建立、关闭连接，会极大的减低系统的性能，因为对于连接的使用成了系统性能的瓶颈。

    连接复用。通过建立一个数据库连接池以及一套连接使用管理策略，使得一个数据库连接可以得到高效、安全的复用，避免了数据库连接频繁建立、关闭的开销。

    对于共享资源，有一个很著名的设计模式：资源池。该模式正是为了解决资源频繁分配、释放所造成的问题的。把该模式应用到数据库连接管理领域，就是建立一个数据库连接池，提供一套高效的连接分配、使用策略，最终目标是实现连接的高效、安全的复用。

    数据库连接池的基本原理是在内部对象池中维护一定数量的数据库连接，并对外暴露数据库连接获取和返回方法。如：

    外部使用者可通过getConnection 方法获取连接，使用完毕后再通过releaseConnection 方法将连接返回，注意此时连接并没有关闭，而是由连接池管理器回收，并为下一次使用做好准备。

    数据库连接池技术带来的优势：

    1. 资源重用

        由于数据库连接得到重用，避免了频繁创建、释放连接引起的大量性能开销。在减少系统消耗的基础上，另一方面也增进了系统运行环境的平稳性（减少内存碎片以及数据库临时进程/线程的数量）。

    2. 更快的系统响应速度

        数据库连接池在初始化过程中，往往已经创建了若干数据库连接置于池中备用。此时连接的初始化工作均已完成。对于业务请求处理而言，直接利用现有可用连接，避免了数据库连接初始化和释放过程的时间开销，从而缩减了系统整体响应时间。

    3. 新的资源分配手段

        对于多应用共享同一数据库的系统而言，可在应用层通过数据库连接的配置，实现数据库连接池技术，几年钱也许还是个新鲜话题，对于目前的业务系统而言，如果设计中还没有考虑到连接池的应用，那么…….快在设计文档中加上这部分的内容吧。某一应用最大可用数据库连接数的限制，避免某一应用独占所有数据库资源。

    4. 统一的连接管理，避免数据库连接泄漏

        在较为完备的数据库连接池实现中，可根据预先的连接占用超时设定，强制收回被占用连接。从而避免了常规数据库连接操作中可能出现的资源泄漏。一个最小化的数据库连接池实现。

2. 前言

    数据库应用，在许多软件系统中经常用到，是开发中大型系统不可缺少的辅助。但如果对数据库资源没有很好地管理(如：没有及时回收数据库的游标(ResultSet)、Statement、连接 (Connection)等资源)，往往会直接导致系统的稳定。这类不稳定因素，不单单由数据库或者系统本身一方引起，只有系统正式使用后，随着流量、用户的增加，才会逐步显露。

    在基于Java开发的系统中，JDBC是程序员和数据库打交道的主要途径，提供了完备的数据库操作方法接口。但考虑到规范的适用性，JDBC只提供了最直接的数据库操作规范，对数据库资源管理，如：对物理连接的管理及缓冲，期望第三方应用服务器(Application Server)的提供。

    本文，以JDBC规范为基础，介绍相关的数据库连接池机制，并就如果以简单的方式，实现有效地管理数据库资源介绍相关实现技术。

3. 连接池技术背景

    1. JDBC

        JDBC是一个规范，遵循JDBC接口规范，各个数据库厂家各自实现自己的驱动程序(Driver)，如下图所示:

        ![](ds/ds-connection_pool-jdbc.png)

        应用在获取数据库连接时，需要以URL的方式指定是那种类型的Driver，在获得特定的连接后，可按照固定的接口操作不同类型的数据库，如: 分别获取Statement、执行SQL获得ResultSet等，如下面的例子 :

        ```java
        DriverManager.registerDriver(new oracle.jdbc.driver.OracleDriver());
        Connection dbConn = DriverManager.getConnection("jdbc:oracle:thin:@127.0.0.1:1521:oracle","username","password");
        Statement st = dbConn.createStatement();
        ResultSet rs = st.executeQuery("select * from demo_table");

        rs.close();
        st.close();
        dbConn.close();
        ```

        在完成数据操作后，还一定要关闭所有涉及到的数据库资源。这虽然对应用程序的逻辑没有任何影响，但是关键的操作。上面是个简单的例子，如果搀和众多的if-else、exception，资源的管理也难免百密一疏。如同C中的内存泄漏问题，Java系统也同样会面临崩溃的恶运。所以数据库资源的管理依赖于应用系统本身，是不安全、不稳定的一种隐患。

    2. JDBC连接池

        在标准JDBC对应用的接口中，并没有提供资源的管理方法。所以，缺省的资源管理由应用自己负责。虽然在JDBC规范中，多次提及资源的关闭/回收及其他的合理运用。但最稳妥的方式，还是为应用提供有效的管理手段。所以，JDBC为第三方应用服务器（Application Server）提供了一个由数据库厂家实现的管理标准接口：连接缓冲(connection pooling)。引入了连接池( Connection Pool )的概念 ，也就是以缓冲池的机制管理数据库的资源。

        JDBC最常用的资源有三类:

        * Connection: 数据库连接。
        * Statement: 会话声明。
        * ResultSet: 结果集游标。

        分别存在以下的关系 ：

        ![](ds/ds-connection_pool-conn-relation.png)

        这是一种“爷—父—子”的关系，对Connection的管理，就是对数据库资源的管理。举个例子: 如果想确定某个数据库连接(Connection)是否超时，则需要确定其（所有的）子Statement是否超时，同样，需要确定所有相关的 ResultSet是否超时；在关闭Connection前，需要关闭所有相关的Statement和ResultSet。

        因此，连接池(Connection Pool)所起到的作用，不仅仅简单地管理Connection，还涉及到 Statement和ResultSet。

    3. 连接池(ConnectionPool)与资源管理

        ConnectionPool以缓冲池的机制，在一定数量上限范围内，控制管理Connection，Statement和ResultSet。任何数据库的资源是有限的，如果被耗尽，则无法获得更多的数据服务。在大多数情况下，资源的耗尽不是由于应用的正常负载过高，而是程序原因。

        在实际工作中，数据资源往往是瓶颈资源，不同的应用都会访问同一数据源。其中某个应用耗尽了数据库资源后，意味其他的应用也无法正常运行。因此，ConnectionPool需要有一定的限制配置。

        1. 连接池的大小 ：每个应用或系统可以拥有的最大资源。也就是确定连接池的大小(PoolSize)。
        2. 资源数：在连接池的大小(PoolSize)范围内，最大限度地使用资源，缩短数据库访问的使用周期。许多数据库中，连接（Connection）并不是资源的最小单元，控制Statement资源比Connection更重要。以oracle为例：总的Statement数量 =（并发物理连接数）×（每个连接可提供的Statement数量）。

4. 简单JDBC连接池的实现

    根据上面所说的原理机制，Snap-ConnectionPool（一种简单快速的连接池工具，可在www.snapbug.net下载）按照部分的JDBC规范，实现了连接池所具备的对数据库资源有效管理功能。

    1. 体系描述

        在JDBC规范中，应用通过驱动接口（Driver Interface）直接方法数据库的资源。为了有效、合理地管理资源，在应用与JDBC Driver之间，增加了连接池: Snap-ConnectionPool。并且通过面向对象的机制，使连接池的大部分操作是透明的。参见下图，Snap-ConnectionPool的体系：

        ![](ds/ds-connection_pool-snap.png)

        图中所示，通过实现JDBC的部分资源对象接口( Connection, Statement, ResultSet )，在 Snap-ConnectionPool内部分别产生三种逻辑资源对象: PooledConnection, PooledStatement和 PooledResultSet。它们也是连接池主要的管理操作对象，并且继承了JDBC中相应的从属关系。这样的体系有以下几个特点：

        1. 透明性，在不改变应用原有的使用JDBC驱动接口的前提下，提供资源管理的服务。应用系统，如同原有的 JDBC，使用连接池提供的逻辑对象资源。简化了应用程序的连接池改造。

        2. 资源封装。复杂的资源管理被封装在 Snap-ConnectionPool内部，不需要应用系统过多的干涉。管理操作的可靠性、安全性由连接池保证。应用的干涉（如：主动关闭资源），只起到优化系统性能的作用，遗漏操作不会带来负面影响。

        3. 资源合理应用，按照JDBC中资源的从属关系，Snap-ConnectionPool不仅对Connection进行缓冲处理，对Statement也有相应的机制处理。在2.3已描述，合理运用Connection和Statement之间的关系，可以更大限度地使用资源。所以，Snap- ConnectionPool封装了Connection资源，通过内部管理PooledConnection，为应用系统提供更多的Statement资源。

        4. 资源连锁管理，Snap-ConnectionPool包含的三种逻辑对象，继承了JDBC中相应对象之间的从属关系。在内部管理中，也依照从属关系进行连锁管理。例如：判断一个Connection是否超时，需要根据所包含的Statement是否活跃；判断Statement也要根据 ResultSet的活跃程度。

    2. 连接池集中管理ConnectionManager

        ConnectionPool是Snap-ConnectionPool的连接池对象。在Snap-ConnectionPool内部，可以指定多个不同的连接池(ConnectionPool)为应用服务。ConnectionManager管理所有的连接池，每个连接池以不同的名称区别。通过配置文件适应不同的数据库种类。如下图所示：

        ![](ds/ds-connection_pool-connection-manager.jpg)

        通过ConnectionManager，可以同时管理多个不同的连接池，提供通一的管理界面。在应用系统中通过 ConnectionManager和相关的配置文件，可以将凌乱散落在各自应用程序中的数据库配置信息（包括：数据库名、用户、密码等信息），集中在一个文件中。便于系统的维护工作。

    3. 连接池使用范例

        对3.1的标准JDBC的使用范例，改为使用连接池，结果如下：

        ```java

        ConnectionPool dbConn = ConnectionManager.getConnectionPool("testOracle");
        Statement st = dbConn.createStatement();
        ResultSet rs = st.executeQuery("select * from demo_table");

        rs.close();
        st.close();
        ```

        在例子中，Snap-ConnectionPool封装了应用对Connection的管理。只要改变JDBC获取Connection的方法，为获取连接池(ConnectionPool)(粗体部分)，其他的数据操作都可以不做修改。

        按照这样的方式，Snap-ConnectionPool可帮助应用有效地管理数据库资源。如果应用忽视了最后资源的释放: rs.close() 和 st.close()，连接池会通过超时(time-out)机制，自动回收。

5. 小结

    无论是Snap-ConnectionPool还是其他的数据库连接池，都应当具备一下基本功能：

    * 对源数据库资源的保护
    * 充分利用发挥数据库的有效资源
    * 简化应用的数据库接口，封闭资源管理。
    * 对应用遗留资源的自动回收和整理，提高资源的再次利用率。

　　在这个前提下，应用程序才能投入更多的精力于各自的业务逻辑中。数据库资源也不再成为系统的瓶颈。

原文：http://blog.sina.com.cn/s/blog_6f688450010148d2.html