## 使用JDBC访问数据

Spring Framework JDBC抽象提供的值最好由下表中列出的操作序列来显示。该表显示了Spring负责哪些操作，以及哪些操作是您的职责。

Action|Spring|You
--|--|--
定义连接参数||X
打开连接|X|
指定SQL语句||X
声明参数并提供参数值||X
准备并执行语句|X|
设置循环来遍历结果(如果有的话)|X|
为每个迭代做工作||X
处理任何例外|X|
处理事务|X|
关闭连接、语句和resultset|X|

Spring框架负责处理所有底层细节，正是这些细节使得JDBC成为如此乏味的API。

1. 为JDBC数据库访问选择一种方法

    您可以在几种方法中进行选择，以形成JDBC数据库访问的基础。除了三种风格的JdbcTemplate之外，还有一种新的SimpleJdbcInsert和SimpleJdbcCall方法可以优化数据库元数据，RDBMS对象风格采用了一种更面向对象的方法，类似于JDO查询设计。一旦您开始使用其中一种方法，您仍然可以混合和匹配以包含来自不同方法的特性。所有方法都需要兼容JDBC 2.0的驱动程序，一些高级特性需要JDBC 3.0驱动程序。

    * JdbcTemplate是经典且最流行的Spring JDBC方法。这种“最低级别”方法和所有其他方法都在幕后使用JdbcTemplate。

    * NamedParameterJdbcTemplate封装了一个JdbcTemplate来提供命名参数，而不是传统的JDBC ?占位符。当您有一个SQL语句的多个参数时，这种方法提供了更好的文档和易用性。

    * SimpleJdbcInsert和SimpleJdbcCall优化数据库元数据，以限制必要配置的数量。这种方法简化了编码，因此只需要提供表或过程的名称，并提供匹配列名的参数映射。只有当数据库提供了足够的元数据时，这才有效。如果数据库不提供此元数据，则必须提供参数的显式配置。

    * RDBMS对象，包括MappingSqlQuery、SqlUpdate和StoredProcedure，要求您在初始化数据访问层期间创建可重用的、线程安全的对象。这种方法模仿JDO查询，在JDO查询中定义查询字符串、声明参数并编译查询。一旦您这样做了，就可以使用各种参数值多次调用execute方法。

2. 包的层次结构

    Spring框架的JDBC抽象框架由四个不同的包组成:
    
    * core: org.springframework.jdbc.core包包含JdbcTemplate类及其各种回调接口，以及各种相关类。一个名为org.springframework.jdbc.core.simple的子包包含SimpleJdbcInsert和SimpleJdbcCall类。另一个名为org.springframework.jdbc.core.namedparam的子包包含NamedParameterJdbcTemplate类和相关的支持类。请参阅使用JDBC核心类控制基本JDBC处理和错误处理、JDBC批处理操作以及使用SimpleJdbc类简化JDBC操作。
    
    * datasource: org.springframework.jdbc.datasource包包含一个实用程序类，用于方便的数据源访问和各种简单的数据源实现，您可以使用这些实现在Java EE容器之外测试和运行未经修改的JDBC代码。一个名为org.springfamework.jdbc.datasource的子包。嵌入式支持使用Java数据库引擎(如HSQL、H2和Derby)创建嵌入式数据库。参见控制数据库连接和嵌入式数据库支持。
    
    * object: org.springframework.jdbc.object包包含表示RDBMS查询、更新和存储过程的类，这些类是线程安全的、可重用的对象。请参见将JDBC操作建模为Java对象。这种方法由JDO建模，尽管查询返回的对象自然与数据库断开连接。这种高层的JDBC抽象依赖于org.springframework.jdbc中的低层抽象。核心包。
    
    * support: org.springframework.jdbc.support包包提供了SQLException翻译功能和一些实用程序类。在JDBC处理期间抛出的异常被转换为org.springframework中定义的异常。dao包。这意味着使用Spring JDBC抽象层的代码不需要实现JDBC或特定于rdbms的错误处理。所有已翻译的异常都是未选中的，这使您可以选择捕获异常，以便在将其他异常传播给调用者的同时从中恢复。使用SQLExceptionTranslator看到。


3. 使用JDBC核心类来控制基本的JDBC处理和错误处理
4. 控制数据库连接
5. JDBC批处理操作
6. 使用SimpleJdbc类简化JDBC操作
7. 将JDBC操作建模为Java对象
8. 参数和数据值处理的常见问题
9. 嵌入式数据库的支持
10. 初始化数据源