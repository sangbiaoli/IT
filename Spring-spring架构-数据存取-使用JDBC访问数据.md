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
    
    * core: org.springframework.jdbc.core 包含JdbcTemplate类及其各种回调接口，以及各种相关类。一个名为org.springframework.jdbc.core.simple的子包包含SimpleJdbcInsert和SimpleJdbcCall类。另一个名为org.springframework.jdbc.core.namedparam的子包包含NamedParameterJdbcTemplate类和相关的支持类。请参阅使用JDBC核心类控制基本JDBC处理和错误处理、JDBC批处理操作以及使用SimpleJdbc类简化JDBC操作。
    
    * datasource: org.springframework.jdbc.datasource 包含一个实用程序类，用于方便的数据源访问和各种简单的数据源实现，您可以使用这些实现在Java EE容器之外测试和运行未经修改的JDBC代码。一个名为org.springfamework.jdbc.datasource.embedded的子包支持使用Java数据库引擎(如HSQL、H2和Derby)创建嵌入式数据库。参见控制数据库连接和嵌入式数据库支持。
    
    * object: org.springframework.jdbc.object 包含表示RDBMS查询、更新和存储过程的类，这些类是线程安全的、可重用的对象。请参见将JDBC操作建模为Java对象。这种方法由JDO建模，尽管查询返回的对象自然与数据库断开连接。这种高层的JDBC抽象依赖于org.springframework.jdbc.core包的低层抽象。
    
    * support: org.springframework.jdbc.support 包提供了SQLException翻译功能和一些实用程序类。在JDBC处理期间抛出的异常被转换为org.springframework中定义的异常。dao包。这意味着使用Spring JDBC抽象层的代码不需要实现JDBC或特定于rdbms的错误处理。所有已翻译的异常都是未选中的，这使您可以选择捕获异常，以便在将其他异常传播给调用者的同时从中恢复。使用SQLExceptionTranslator看到。


3. 使用JDBC核心类来控制基本的JDBC处理和错误处理

    本节介绍如何使用JDBC核心类来控制基本的JDBC处理，包括错误处理。它包括下列主题:
    * 使用JdbcTemplate
    * 使用NamedParameterJdbcTemplate
    * 使用SQLExceptionTranslator
    * 运行报表
    * 运行查询
    * 更新数据库
    * 获取自动生成的键

    1. 使用JdbcTemplate

        JdbcTemplate是JDBC核心包中的中心类。它处理资源的创建和释放，这有助于避免常见的错误，比如忘记关闭连接。它执行核心JDBC工作流的基本任务(如语句创建和执行)，留下应用程序代码来提供SQL和提取结果。JdbcTemplate类:

        * 运行SQL查询
        * 更新语句和存储过程调用
        * 对ResultSet实例执行迭代并提取返回的参数值。
        * 捕获JDBC异常，并将它们转换为org.springframework.dao包中定义的通用的、信息更丰富的异常层次结构。

        当您为代码使用JdbcTemplate时，您只需要实现回调接口，给它们一个明确定义的契约。给定由JdbcTemplate类提供的连接，PreparedStatementCreator回调接口将创建一个准备好的语句，提供SQL和任何必要的参数。对于创建可调用语句的CallableStatementCreator接口也是如此。RowCallbackHandler接口从ResultSet的每一行中提取值。

        可以通过直接实例化数据源引用在DAO实现中使用JdbcTemplate，也可以在Spring IoC容器中配置它，并将其作为bean引用提供给DAOs。

        这个类发出的所有SQL都记录在DEBUG级别对应于模板实例的完全限定类名的类别下(通常是JdbcTemplate，但是如果使用JdbcTemplate类的自定义子类，情况可能有所不同)。

        下面几节提供了一些使用JdbcTemplate的示例。这些示例并不是JdbcTemplate公开的所有功能的详尽列表。请参阅相关的javadoc。

        1. Querying (SELECT)

            ```java
            int countOfActorsNamedJoe = this.jdbcTemplate.queryForObject("select count(*) from t_actor where first_name = ?", Integer.class, "Joe");
            ```

            ```java
            public List<Actor> findAllActors() {
                return this.jdbcTemplate.query( "select first_name, last_name from t_actor", new ActorMapper());
            }

            private static final class ActorMapper implements RowMapper<Actor> {

                public Actor mapRow(ResultSet rs, int rowNum) throws SQLException {
                    Actor actor = new Actor();
                    actor.setFirstName(rs.getString("first_name"));
                    actor.setLastName(rs.getString("last_name"));
                    return actor;
                }
            }
            ```

        2. 使用JdbcTemplate更新(INSERT, UPDATE, 和 DELETE)

            ```java
            this.jdbcTemplate.update("insert into t_actor (first_name, last_name) values (?, ?)","Leonor", "Watling");

            this.jdbcTemplate.update("update t_actor set last_name = ? where id = ?","Banjo", 5276L);

            this.jdbcTemplate.update("delete from actor where id = ?",Long.valueOf(actorId));
            ```
        

        3. JdbcTemplate的其他操作

            ```java
            this.jdbcTemplate.execute("create table mytable (id integer, name varchar(100))");

            this.jdbcTemplate.update("call SUPPORT.REFRESH_ACTORS_SUMMARY(?)",Long.valueOf(unionId));
            ```

        4. JdbcTemplate的最好实践

            一旦配置好，JdbcTemplate类的实例是线程安全的。这很重要，因为这意味着您可以配置一个JdbcTemplate的单个实例，然后安全地将这个共享引用注入多个dao(或存储库)。JdbcTemplate是有状态的，因为它维护对数据源的引用，但是这种状态不是会话状态。

            在使用JdbcTemplate类(以及相关的NamedParameterJdbcTemplate类)时，一个常见的实践是在Spring配置文件中配置一个数据源，然后将该共享数据源bean依赖地注入到DAO类中。JdbcTemplate是在数据源的setter中创建的。这导致dao类似于以下内容:

            ```java
            public class JdbcCorporateEventDao implements CorporateEventDao {

                private JdbcTemplate jdbcTemplate;

                public void setDataSource(DataSource dataSource) {
                    this.jdbcTemplate = new JdbcTemplate(dataSource);
                }

                // JDBC-backed implementations of the methods on the CorporateEventDao follow...
            }
            ```

            下面的示例显示了相应的XML配置:

            ```xml
            <?xml version="1.0" encoding="UTF-8"?>
            <beans xmlns="http://www.springframework.org/schema/beans"
                xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                xmlns:context="http://www.springframework.org/schema/context"
                xsi:schemaLocation="
                    http://www.springframework.org/schema/beans
                    https://www.springframework.org/schema/beans/spring-beans.xsd
                    http://www.springframework.org/schema/context
                    https://www.springframework.org/schema/context/spring-context.xsd">

                <bean id="corporateEventDao" class="com.example.JdbcCorporateEventDao">
                    <property name="dataSource" ref="dataSource"/>
                </bean>

                <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
                    <property name="driverClassName" value="${jdbc.driverClassName}"/>
                    <property name="url" value="${jdbc.url}"/>
                    <property name="username" value="${jdbc.username}"/>
                    <property name="password" value="${jdbc.password}"/>
                </bean>

                <context:property-placeholder location="jdbc.properties"/>

            </beans>
            ```

    2. 使用NamedParameterJdbcTemplate

        NamedParameterJdbcTemplate类通过使用命名参数来添加对JDBC语句编程的支持，而不是只使用经典占位符('?')参数来编程JDBC语句。NamedParameterJdbcTemplate类包装了一个JdbcTemplate，并委托给包装好的JdbcTemplate来完成它的大部分工作。
        
        本节只描述NamedParameterJdbcTemplate类中与JdbcTemplate本身不同的部分——即通过使用命名参数来编程JDBC语句。下面的例子展示了如何使用NamedParameterJdbcTemplate:

        ```java
        // some JDBC-backed DAO class...
        private NamedParameterJdbcTemplate namedParameterJdbcTemplate;

        public void setDataSource(DataSource dataSource) {
            this.namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
        }

        public int countOfActorsByFirstName(String firstName) {

            String sql = "select count(*) from T_ACTOR where first_name = :first_name";

            SqlParameterSource namedParameters = new MapSqlParameterSource("first_name", firstName);

            return this.namedParameterJdbcTemplate.queryForObject(sql, namedParameters, Integer.class);
        }
        ```

        注意，在分配给sql变量的值和插入到namedParameters变量(类型为MapSqlParameterSource)的对应值中使用了命名参数表示法。

        或者，可以使用基于映射的样式将命名参数及其对应的值传递给NamedParameterJdbcTemplate实例。NamedParameterJdbcOperations公开并由NamedParameterJdbcTemplate类实现的其余方法遵循类似的模式，这里不讨论。

        与NamedParameterJdbcTemplate(存在于同一个Java包中)相关的一个很好的特性是SqlParameterSource接口。您已经在前面的代码片段(MapSqlParameterSource类)中看到了这个接口的实现示例。SqlParameterSource是NamedParameterJdbcTemplate的已命名参数值的源。MapSqlParameterSource类是一个简单的实现，它是围绕java.util.Map的适配器，其中键是参数名，值是参数值。

        另一个SqlParameterSource实现是BeanPropertySqlParameterSource类。这个类封装了一个任意的JavaBean(即，一个遵循JavaBean约定的类的实例)，并使用封装的JavaBean的属性作为命名参数值的源。

        下面的例子展示了一个典型的JavaBean:

        ```java
        import lombok.Data

        @Data
        public class Actor {
            private Long id;
            private String firstName;
            private String lastName;
        }
        ```

        下面的例子使用一个NamedParameterJdbcTemplate返回前一个例子中显示的类成员的计数:

        ```java
        // some JDBC-backed DAO class...
        private NamedParameterJdbcTemplate namedParameterJdbcTemplate;

        public void setDataSource(DataSource dataSource) {
            this.namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
        }

        public int countOfActors(Actor exampleActor) {

            // notice how the named parameters match the properties of the above 'Actor' class
            String sql = "select count(*) from T_ACTOR where first_name = :firstName and last_name = :lastName";

            SqlParameterSource namedParameters = new BeanPropertySqlParameterSource(exampleActor);

            return this.namedParameterJdbcTemplate.queryForObject(sql, namedParameters, Integer.class);
        }
        ```

    3. 使用SQLExceptionTranslator
    4. 运行报表
    5. 运行查询
    6. 更新数据库
    7. 获取自动生成的键


4. 控制数据库连接
5. JDBC批处理操作
6. 使用SimpleJdbc类简化JDBC操作
7. 将JDBC操作建模为Java对象
8. 参数和数据值处理的常见问题
9. 嵌入式数据库的支持
10. 初始化数据源