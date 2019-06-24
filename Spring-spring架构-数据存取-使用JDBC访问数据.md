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

        记住，NamedParameterJdbcTemplate类封装了一个经典的JdbcTemplate模板。如果需要访问包装好的JdbcTemplate实例来访问仅出现在JdbcTemplate类中的功能，可以使用getJdbcOperations()方法通过JdbcOperations接口访问包装好的JdbcTemplate。

    3. 使用SQLExceptionTranslator

        SQLExceptionTranslator是一个由类实现的接口，该类可以在SQLExceptions和Spring自己的org.springframework.dao.DataAccessException之间进行转换，它与数据访问策略无关。实现可以是通用的(例如，为JDBC使用SQLState代码)，也可以是专有的(例如，使用Oracle错误代码)，以获得更高的精度。

        SQLErrorCodeSQLExceptionTranslator是默认使用的SQLExceptionTranslator的实现。此实现使用特定的供应商代码。它比SQLState实现更精确。错误代码转换基于JavaBean类型类SQLErrorCodes中包含的代码。该类由SQLErrorCodesFactory创建并填充，SQLErrorCodesFactory(顾名思义)是根据名为sql-error-code .xml的配置文件的内容创建sqlerrorcode的工厂。该文件由供应商代码填充，并基于DatabaseProductName(取自DatabaseMetaData)。使用您正在使用的实际数据库的代码。
        
        SQLErrorCodeSQLExceptionTranslator按照以下顺序应用匹配规则:

        1. 由子类实现的任何自定义翻译。通常使用所提供的具体SQLErrorCodeSQLExceptionTranslator，因此不适用此规则。它只适用于实际提供了子类实现的情况。
        2. SQLErrorCodes类的customSqlExceptionTranslator属性提供的SQLExceptionTranslator接口的任何自定义实现。
        3. 将搜索CustomSQLErrorCodesTranslation类(为SQLErrorCodes类的customtranslation属性提供)的实例列表以寻找匹配项。
        4. 应用错误代码匹配。
        5. 使用后备翻译器。SQLExceptionSubclassTranslator是默认的回退转换器。如果这个翻译不可用，下一个后备翻译器是SQLStateSQLExceptionTranslator。

        您可以扩展SQLErrorCodeSQLExceptionTranslator，如下面的示例所示:

        ```java
        public class CustomSQLErrorCodesTranslator extends SQLErrorCodeSQLExceptionTranslator {

            protected DataAccessException customTranslate(String task, String sql, SQLException sqlex) {
                if (sqlex.getErrorCode() == -12345) {
                    return new DeadlockLoserDataAccessException(task, sqlex);
                }
                return null;
            }
        }
        ```

        在前面的示例中，将翻译特定的错误代码(-12345)，而其他错误将由缺省翻译器实现翻译。要使用这个自定义转换器，您必须通过setExceptionTranslator方法将其传递给JdbcTemplate，并且必须在需要这个转换器的所有数据访问处理中使用这个JdbcTemplate。下面的例子展示了如何使用这个自定义翻译器:

        ```java
        private JdbcTemplate jdbcTemplate;

        public void setDataSource(DataSource dataSource) {

            // create a JdbcTemplate and set data source
            this.jdbcTemplate = new JdbcTemplate();
            this.jdbcTemplate.setDataSource(dataSource);

            // create a custom translator and set the DataSource for the default translation lookup
            CustomSQLErrorCodesTranslator tr = new CustomSQLErrorCodesTranslator();
            tr.setDataSource(dataSource);
            this.jdbcTemplate.setExceptionTranslator(tr);

        }

        public void updateShippingCharge(long orderId, long pct) {
            // use the prepared JdbcTemplate for this update
            this.jdbcTemplate.update("update orders" +
                " set shipping_charge = shipping_charge * ? / 100" +
                " where id = ?", pct, orderId);
        }
        ```

        为了在sql-error-code .xml中查找错误代码，将向自定义转换器传递一个数据源。

    4. 运行Statements

        运行SQL语句只需要很少的代码。您需要一个数据源和一个JdbcTemplate，包括JdbcTemplate提供的便利方法。下面的例子展示了创建一个新表的最小但功能齐全的类需要包含什么:

        ```java
        import javax.sql.DataSource;
        import org.springframework.jdbc.core.JdbcTemplate;

        public class ExecuteAStatement {

            private JdbcTemplate jdbcTemplate;

            public void setDataSource(DataSource dataSource) {
                this.jdbcTemplate = new JdbcTemplate(dataSource);
            }

            public void doExecute() {
                this.jdbcTemplate.execute("create table mytable (id integer, name varchar(100))");
            }
        }
        ```

    5. 运行查询

        一些查询方法返回一个值。要从一行检索计数或特定值，请使用queryForObject(..)。后者将返回的JDBC类型转换为作为参数传入的Java类。如果类型转换无效，则抛出InvalidDataAccessApiUsageException。下面的示例包含两个查询方法，一个用于int，另一个用于查询字符串:

        ```java
        import javax.sql.DataSource;
        import org.springframework.jdbc.core.JdbcTemplate;

        public class RunAQuery {

            private JdbcTemplate jdbcTemplate;

            public void setDataSource(DataSource dataSource) {
                this.jdbcTemplate = new JdbcTemplate(dataSource);
            }

            public int getCount() {
                return this.jdbcTemplate.queryForObject("select count(*) from mytable", Integer.class);
            }

            public String getName() {
                return this.jdbcTemplate.queryForObject("select name from mytable", String.class);
            }
        }
        ```

        除了单个结果查询方法外，还有几个方法返回一个列表，其中查询返回的每一行都有一个条目。最通用的方法是queryForList(..)，它返回一个列表，其中每个元素都是一个映射，每个列包含一个条目，使用列名作为键。如果在前面的示例中添加一个方法来检索所有行的列表，它可能如下所示:

        ```java
        private JdbcTemplate jdbcTemplate;

        public void setDataSource(DataSource dataSource) {
            this.jdbcTemplate = new JdbcTemplate(dataSource);
        }

        public List<Map<String, Object>> getList() {
            return this.jdbcTemplate.queryForList("select * from mytable");
        }
        ```

        返回的列表如下:

        ```
        [{name=Bob, id=1}, {name=Mary, id=2}]
        ```

    6. 更新数据库

        下面的示例更新某个主键的列:

        ```java
        import javax.sql.DataSource;
        import org.springframework.jdbc.core.JdbcTemplate;

        public class ExecuteAnUpdate {

            private JdbcTemplate jdbcTemplate;

            public void setDataSource(DataSource dataSource) {
                this.jdbcTemplate = new JdbcTemplate(dataSource);
            }

            public void setName(int id, String name) {
                this.jdbcTemplate.update("update mytable set name = ? where id = ?", name, id);
            }
        }
        ```
    
        在前面的示例中，SQL语句为行参数提供占位符。您可以将参数值作为变量传递，或者作为对象数组传递。因此，应该在基元包装器类中显式地包装基元，或者使用自动装箱。

    7. 获取自动生成的键

        update()便利方法支持检索数据库生成的主键。这种支持是JDBC 3.0标准的一部分。详见规范第13.6章。该方法以PreparedStatementCreator作为第一个参数，这是指定所需insert语句的方式。另一个参数是KeyHolder，它包含更新成功返回时生成的密钥。没有标准的单一方法来创建适当的PreparedStatement(这解释了为什么方法签名是这样的)。下面的例子适用于Oracle，但可能不适用于其他平台:

        ```java
        final String INSERT_SQL = "insert into my_test (name) values(?)";
        final String name = "Rob";

        KeyHolder keyHolder = new GeneratedKeyHolder();
        jdbcTemplate.update(
            new PreparedStatementCreator() {
                public PreparedStatement createPreparedStatement(Connection connection) throws SQLException {
                    PreparedStatement ps = connection.prepareStatement(INSERT_SQL, new String[] {"id"});
                    ps.setString(1, name);
                    return ps;
                }
            },
            keyHolder);

        // keyHolder.getKey() now contains the generated key
        ```

4. 控制数据库连接

    本节将介绍:

    * 使用DataSource
    * 使用DataSourceUtils
    * 实现SmartDataSource
    * 扩展AbstractDataSource
    * 使用SingleConnectionDataSource
    * 使用DriverManagerDataSource
    * 使用TransactionAwareDataSourceProxy
    * 使用DataSourceTransactionManager

    1. 使用DataSource

        Spring通过数据源获得到数据库的连接。数据源是JDBC规范的一部分，是一个通用的连接工厂。它允许容器或框架对应用程序代码隐藏连接池和事务管理问题。作为开发人员，您不需要知道如何连接到数据库的详细信息。这是设置数据源的管理员的责任。您很可能在开发和测试代码时同时担任这两个角色，但是您不必知道如何配置生产数据源。

        当您使用Spring的JDBC层时，您可以从JNDI获得数据源，或者您可以使用第三方提供的连接池实现来配置您自己的数据源。流行的实现是Apache Jakarta Commons DBCP和C3P0。Spring发行版中的实现仅用于测试目的，不提供池。

        本节使用Spring的DriverManagerDataSource实现，稍后将介绍其他几个实现。

        要配置一个DriverManagerDataSource:
        1. 获取与DriverManagerDataSource的连接，就像您通常获取JDBC连接一样。
        2. 指定JDBC驱动程序的完全限定类名，以便驱动程序管理器可以加载驱动程序类。
        3. 提供一个在JDBC驱动程序之间变化的URL。(有关正确的值，请参阅驱动程序的文档。)
        4. 提供连接到数据库的用户名和密码。

        下面的例子展示了如何在Java中配置DriverManagerDataSource:

        ```java
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("org.hsqldb.jdbcDriver");
        dataSource.setUrl("jdbc:hsqldb:hsql://localhost:");
        dataSource.setUsername("sa");
        dataSource.setPassword("");
        ```

        下面的示例显示了相应的XML配置:

        ```xml
        <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
            <property name="driverClassName" value="${jdbc.driverClassName}"/>
            <property name="url" value="${jdbc.url}"/>
            <property name="username" value="${jdbc.username}"/>
            <property name="password" value="${jdbc.password}"/>
        </bean>

        <context:property-placeholder location="jdbc.properties"/>
        ```

        接下来的两个示例展示了DBCP和C3P0的基本连接和配置。要了解更多帮助控制池功能的选项，请参阅相关连接池实现的产品文档。
        
        下面的例子显示了DBCP配置:

        ```xml
        <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
            <property name="driverClassName" value="${jdbc.driverClassName}"/>
            <property name="url" value="${jdbc.url}"/>
            <property name="username" value="${jdbc.username}"/>
            <property name="password" value="${jdbc.password}"/>
        </bean>

        <context:property-placeholder location="jdbc.properties"/>
        ```

        下面的例子显示了C3P0配置:

        ```xml
        <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close">
            <property name="driverClass" value="${jdbc.driverClassName}"/>
            <property name="jdbcUrl" value="${jdbc.url}"/>
            <property name="user" value="${jdbc.username}"/>
            <property name="password" value="${jdbc.password}"/>
        </bean>

        <context:property-placeholder location="jdbc.properties"/>
        ```

    2. 使用DataSourceUtils

        DataSourceUtils类是一个方便且功能强大的助手类，它提供静态方法来从JNDI获取连接，并在必要时关闭连接。它支持与DataSourceTransactionManager等线程绑定连接。

    3. 实现SmartDataSource

        SmartDataSource接口应该由能够提供到关系数据库连接的类实现。它扩展了DataSource接口，让使用它的类查询在给定操作之后是否应该关闭连接。当您知道需要重用连接时，这种用法是有效的。

    4. 扩展AbstractDataSource

        AbstractDataSource是Spring数据源实现的抽象基类。它实现了所有数据源实现所共有的代码。如果编写自己的数据源实现，应该扩展AbstractDataSource类。

    5. 使用SingleConnectionDataSource

        SingleConnectionDataSource类是SmartDataSource接口的实现，它封装了一个在每次使用之后都不会关闭的连接。这是不支持多线程的。

        如果任何客户机代码调用都是基于池连接的假设关闭的(就像使用持久性工具时一样)，那么应该将suppressClose属性设置为true。此设置返回一个封装物理连接的关闭抑制代理。注意，您不能再将此转换为本机Oracle连接或类似对象。

        SingleConnectionDataSource主要是一个测试类。例如，它支持使用简单的JNDI环境在应用服务器外部轻松地测试代码。与DriverManagerDataSource相反，它始终重用相同的连接，避免过多地创建物理连接。

    6. 使用DriverManagerDataSource

        DriverManagerDataSource类是标准数据源接口的实现，它通过bean属性配置普通JDBC驱动程序，并每次返回一个新连接。

        此实现对于Java EE容器外部的测试和独立环境非常有用，可以作为Spring IoC容器中的数据源bean，也可以与简单的JNDI环境结合使用。close()调用close连接，因此任何数据源感知的持久性代码都应该可以工作。然而，使用javabean样式的连接池(例如commons-dbcp)非常简单，即使在测试环境中也是如此，因此在DriverManagerDataSource上使用这样的连接池，几乎总是首选。

    7. 使用TransactionAwareDataSourceProxy

        TransactionAwareDataSourceProxy是目标数据源的代理。代理将目标数据源包装起来，以增加对spring管理的事务的感知。在这方面，它类似于由Java EE服务器提供的事务性JNDI数据源。

    8. 使用DataSourceTransactionManager

        DataSourceTransactionManager类是针对单个JDBC数据源的平台transactionmanager实现。它将指定数据源的JDBC连接绑定到当前执行的线程，这可能允许每个数据源有一个线程连接。

        要通过DataSourceUtils.getConnection (DataSource)而不是Java EE的标准DataSource.getConnection检索JDBC连接，需要应用程序代码。它抛出未选中的org.springframework。dao异常而不是已检查的SQLExceptions。所有框架类(例如JdbcTemplate)都隐式地使用此策略。如果不与此事务管理器一起使用，则查找策略的行为与普通策略完全相同。因此，它可以在任何情况下使用。

        DataSourceTransactionManager类支持作为适当的JDBC语句查询超时应用的定制隔离级别和超时。要支持后者，应用程序代码必须使用JdbcTemplate或为每个创建的语句调用DataSourceUtils.applyTransactionTimeout(..)方法。
        
        在单资源的情况下，您可以使用这个实现代替JtaTransactionManager，因为它不需要容器来支持JTA。如果坚持使用所需的连接查找模式，在两者之间切换只是配置问题。JTA不支持自定义隔离级别。

5. JDBC批处理操作

    如果将多个调用批处理到同一个准备好的语句，大多数JDBC驱动程序都可以提供更好的性能。通过将更新分组为批，可以限制到数据库的往返次数。

    1. 使用JdbcTemplate进行基本的批处理操作

        您可以通过实现一个特殊接口的两个方法BatchPreparedStatementSetter来完成JdbcTemplate批处理，并将该实现作为batchUpdate方法调用中的第二个参数传递进来。您可以使用getBatchSize方法来提供当前批处理的大小。您可以使用setValues方法为准备好的语句的参数设置值。此方法被调用的次数是在getBatchSize调用中指定的次数。下面的示例根据列表中的条目更新actor表，整个列表用作批处理:

        ```java
        public class JdbcActorDao implements ActorDao {

            private JdbcTemplate jdbcTemplate;

            public void setDataSource(DataSource dataSource) {
                this.jdbcTemplate = new JdbcTemplate(dataSource);
            }

            public int[] batchUpdate(final List<Actor> actors) {
                return this.jdbcTemplate.batchUpdate(
                        "update t_actor set first_name = ?, last_name = ? where id = ?",
                        new BatchPreparedStatementSetter() {
                            public void setValues(PreparedStatement ps, int i) throws SQLException {
                                ps.setString(1, actors.get(i).getFirstName());
                                ps.setString(2, actors.get(i).getLastName());
                                ps.setLong(3, actors.get(i).getId().longValue());
                            }
                            public int getBatchSize() {
                                return actors.size();
                            }
                        });
            }

            // ... additional methods
        }
        ```

        如果您处理更新流或从文件中读取数据，您可能有一个首选的批大小，但是最后一批可能没有那么多条目。在本例中，您可以使用InterruptibleBatchPreparedStatementSetter接口，它允许您在输入源耗尽后中断批处理。isBatchExhausted方法允许您发出批处理结束的信号。

    2. 使用对象列表进行批处理操作

        JdbcTemplate和NamedParameterJdbcTemplate都提供了批处理更新的替代方法。您不需要实现特殊的批处理接口，而是将调用中的所有参数值作为列表提供。框架循环这些值并使用内部准备好的语句设置器。API会根据是否使用命名参数而变化。对于命名参数，您提供一个SqlParameterSource数组，每个批处理成员有一个条目。您可以使用SqlParameterSourceUtils.createBatch这个便利方法创建这个数组，传入一个bean样式的对象数组(带有与参数对应的getter方法)、String-keyed Map实例(包含作为值的相应参数)，或者两者兼用。
        
        下面的示例显示了使用指定参数的批处理更新:

        ```java
        public class JdbcActorDao implements ActorDao {

            private NamedParameterTemplate namedParameterJdbcTemplate;

            public void setDataSource(DataSource dataSource) {
                this.namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
            }

            public int[] batchUpdate(List<Actor> actors) {
                return this.namedParameterJdbcTemplate.batchUpdate(
                        "update t_actor set first_name = :firstName, last_name = :lastName where id = :id",
                        SqlParameterSourceUtils.createBatch(actors));
            }

            // ... additional methods
        }
        ```

        对于使用经典语句的SQL语句?占位符，传递一个包含对象数组和更新值的列表。这个对象数组必须为SQL语句中的每个占位符有一个条目，并且它们必须与SQL语句中定义的顺序相同。

        下面的示例与前面的示例相同，只是使用了经典JDBC ?占位符:

        ```java
        public class JdbcActorDao implements ActorDao {

            private JdbcTemplate jdbcTemplate;

            public void setDataSource(DataSource dataSource) {
                this.jdbcTemplate = new JdbcTemplate(dataSource);
            }

            public int[] batchUpdate(final List<Actor> actors) {
                List<Object[]> batch = new ArrayList<Object[]>();
                for (Actor actor : actors) {
                    Object[] values = new Object[] {
                            actor.getFirstName(), actor.getLastName(), actor.getId()};
                    batch.add(values);
                }
                return this.jdbcTemplate.batchUpdate(
                        "update t_actor set first_name = ?, last_name = ? where id = ?",
                        batch);
            }

            // ... additional methods
        }
        ```

        前面描述的所有批处理更新方法都返回一个int数组，其中包含每个批处理条目的受影响行数。这个计数由JDBC驱动程序报告。如果计数不可用，JDBC驱动程序将返回一个值-2。

    3. 具有多个批次的批处理操作

        前面的批更新示例处理的批非常大，您希望将它们分成几个较小的批。您可以通过对batchUpdate方法进行多次调用来使用前面提到的方法，但是现在有一个更方便的方法。除了SQL语句外，该方法还接受包含参数的对象集合、为每个批处理进行更新的次数，以及一个ParameterizedPreparedStatementSetter来设置所准备语句的参数的值。框架在提供的值上循环，并将更新调用分成指定大小的批次。
        
        下面的例子显示了一个批处理更新，它使用的批大小为100:

        ```java
        public class JdbcActorDao implements ActorDao {

            private JdbcTemplate jdbcTemplate;

            public void setDataSource(DataSource dataSource) {
                this.jdbcTemplate = new JdbcTemplate(dataSource);
            }

            public int[][] batchUpdate(final Collection<Actor> actors) {
                int[][] updateCounts = jdbcTemplate.batchUpdate(
                        "update t_actor set first_name = ?, last_name = ? where id = ?",
                        actors,
                        100,
                        new ParameterizedPreparedStatementSetter<Actor>() {
                            public void setValues(PreparedStatement ps, Actor argument) throws SQLException {
                                ps.setString(1, argument.getFirstName());
                                ps.setString(2, argument.getLastName());
                                ps.setLong(3, argument.getId().longValue());
                            }
                        });
                return updateCounts;
            }

            // ... additional methods
        }
        ```

       此调用的批处理更新方法返回一个int数组，其中包含每个批处理的数组条目，以及每个更新的受影响行数数组。顶层数组的长度表示执行的批数，而第二层数组的长度表示该批中的更新数。每个批中的更新数量应该是为所有批提供的批大小(除了最后一个可能更小)，这取决于所提供的更新对象的总数。每个update语句的更新计数由JDBC驱动程序报告。如果计数不可用，JDBC驱动程序将返回一个-2值。

6. 使用SimpleJdbc类简化JDBC操作

    SimpleJdbcInsert和SimpleJdbcCall类利用可以通过JDBC驱动程序检索的数据库元数据，提供了一个简化的配置。这意味着您需要预先配置的东西更少，尽管如果您希望在代码中提供所有细节，您可以覆盖或关闭元数据处理。

    1. 使用SimpleJdbcInsert插入数据

        我们首先看看SimpleJdbcInsert类，它具有最少的配置选项。您应该在数据访问层的初始化方法中实例化SimpleJdbcInsert。对于本例，初始化方法是setDataSource方法。您不需要子类化SimpleJdbcInsert类。相反，您可以创建一个新实例，并使用withTableName方法设置表名。该类的配置方法遵循返回SimpleJdbcInsert实例的流体样式，该实例允许您链接所有配置方法。下面的示例只使用了一种配置方法(稍后我们将展示多种方法的示例):

        ```java
        public class JdbcActorDao implements ActorDao {

            private JdbcTemplate jdbcTemplate;
            private SimpleJdbcInsert insertActor;

            public void setDataSource(DataSource dataSource) {
                this.jdbcTemplate = new JdbcTemplate(dataSource);
                this.insertActor = new SimpleJdbcInsert(dataSource).withTableName("t_actor");
            }

            public void add(Actor actor) {
                Map<String, Object> parameters = new HashMap<String, Object>(3);
                parameters.put("id", actor.getId());
                parameters.put("first_name", actor.getFirstName());
                parameters.put("last_name", actor.getLastName());
                insertActor.execute(parameters);
            }

            // ... additional methods
        }
        ```

    2. 使用SimpleJdbcInsert检索自动生成的密钥

        下一个示例使用与前一个示例相同的insert，但是它没有传递id，而是检索自动生成的密钥并将其设置在新的Actor对象上。当它创建SimpleJdbcInsert时，除了指定表名之外，还使用usingGeneratedKeyColumns方法指定生成的键列的名称。下面的清单显示了它是如何工作的:

        ```java
        public class JdbcActorDao implements ActorDao {

            private JdbcTemplate jdbcTemplate;
            private SimpleJdbcInsert insertActor;

            public void setDataSource(DataSource dataSource) {
                this.jdbcTemplate = new JdbcTemplate(dataSource);
                this.insertActor = new SimpleJdbcInsert(dataSource)
                        .withTableName("t_actor")
                        .usingGeneratedKeyColumns("id");
            }

            public void add(Actor actor) {
                Map<String, Object> parameters = new HashMap<String, Object>(2);
                parameters.put("first_name", actor.getFirstName());
                parameters.put("last_name", actor.getLastName());
                Number newId = insertActor.executeAndReturnKey(parameters);
                actor.setId(newId.longValue());
            }

            // ... additional methods
        }
        ```

        使用第二种方法运行插入时的主要区别是，不向映射添加id，而是调用executeAndReturnKey方法。这将返回java.lang.Number对象，可以使用该对象创建域类中使用的数值类型的实例。这里不能依赖所有数据库返回特定的Java类。java.lang.Number是您可以依赖的基类。如果您有多个自动生成的列，或者生成的值是非数值的，则可以使用从executeAndReturnKeyHolder方法返回的KeyHolder。

    3. 为SimpleJdbcInsert指定列

        通过使用usingColumns方法指定列名列表，可以限制insert的列，如下面的示例所示:

        ```java
        public class JdbcActorDao implements ActorDao {

            private JdbcTemplate jdbcTemplate;
            private SimpleJdbcInsert insertActor;

            public void setDataSource(DataSource dataSource) {
                this.jdbcTemplate = new JdbcTemplate(dataSource);
                this.insertActor = new SimpleJdbcInsert(dataSource)
                        .withTableName("t_actor")
                        .usingColumns("first_name", "last_name")
                        .usingGeneratedKeyColumns("id");
            }

            public void add(Actor actor) {
                Map<String, Object> parameters = new HashMap<String, Object>(2);
                parameters.put("first_name", actor.getFirstName());
                parameters.put("last_name", actor.getLastName());
                Number newId = insertActor.executeAndReturnKey(parameters);
                actor.setId(newId.longValue());
            }

            // ... additional methods
        }
        ```

        插入的执行与依赖元数据确定要使用哪些列一样。

    4. 使用SqlParameterSource提供参数值

        使用映射来提供参数值工作得很好，但是它不是最方便使用的类。Spring提供了SqlParameterSource接口的两个实现，您可以使用它们。第一个是BeanPropertySqlParameterSource，如果您有一个兼容javabean的类，其中包含您的值，那么它是一个非常方便的类。它使用相应的getter方法来提取参数值。下面的例子展示了如何使用BeanPropertySqlParameterSource:

        ```java
        public class JdbcActorDao implements ActorDao {

            private JdbcTemplate jdbcTemplate;
            private SimpleJdbcInsert insertActor;

            public void setDataSource(DataSource dataSource) {
                this.jdbcTemplate = new JdbcTemplate(dataSource);
                this.insertActor = new SimpleJdbcInsert(dataSource)
                        .withTableName("t_actor")
                        .usingGeneratedKeyColumns("id");
            }

            public void add(Actor actor) {
                SqlParameterSource parameters = new BeanPropertySqlParameterSource(actor);
                Number newId = insertActor.executeAndReturnKey(parameters);
                actor.setId(newId.longValue());
            }

            // ... additional methods
        }
        ```

        另一个选项是MapSqlParameterSource，它类似于映射，但提供了一个更方便的addValue方法，可以链接该方法。下面的例子展示了如何使用它:

        ```java
        public class JdbcActorDao implements ActorDao {

            private JdbcTemplate jdbcTemplate;
            private SimpleJdbcInsert insertActor;

            public void setDataSource(DataSource dataSource) {
                this.jdbcTemplate = new JdbcTemplate(dataSource);
                this.insertActor = new SimpleJdbcInsert(dataSource)
                        .withTableName("t_actor")
                        .usingGeneratedKeyColumns("id");
            }

            public void add(Actor actor) {
                SqlParameterSource parameters = new MapSqlParameterSource()
                        .addValue("first_name", actor.getFirstName())
                        .addValue("last_name", actor.getLastName());
                Number newId = insertActor.executeAndReturnKey(parameters);
                actor.setId(newId.longValue());
            }

            // ... additional methods
        }
        ```

        正如您所看到的，配置是相同的。只有执行中的代码需要更改才能使用这些替代输入类。
        
    5. 使用SimpleJdbcCall调用存储过程

        SimpleJdbcCall类使用数据库中的元数据来查找in和out参数的名称，这样您就不必显式地声明它们。您可以声明参数，如果您喜欢这样做，或者您有参数(如数组或结构)，而这些参数没有自动映射到Java类。第一个例子展示了一个简单的过程，它只从MySQL数据库返回VARCHAR和日期格式的标量值。示例过程读取指定的参与者条目，并以out参数的形式返回first_name、last_name和birth_date列。下面的清单显示了第一个例子:

        ```sql
        CREATE PROCEDURE read_actor (
            IN in_id INTEGER,
            OUT out_first_name VARCHAR(100),
            OUT out_last_name VARCHAR(100),
            OUT out_birth_date DATE)
        BEGIN
            SELECT first_name, last_name, birth_date
            INTO out_first_name, out_last_name, out_birth_date
            FROM t_actor where id = in_id;
        END;
        ```

        in_id参数包含您正在查找的参与者的id。out参数返回从表中读取的数据。
        
        您可以以类似于声明SimpleJdbcInsert的方式声明SimpleJdbcCall。您应该在数据访问层的初始化方法中实例化和配置该类。与StoredProcedure类相比，您不需要创建子类，也不需要声明可以在数据库元数据中查找的参数。下面的SimpleJdbcCall配置示例使用前面的存储过程(除了数据源之外，惟一的配置选项是存储过程的名称):

        ```java
        public class JdbcActorDao implements ActorDao {

            private JdbcTemplate jdbcTemplate;
            private SimpleJdbcCall procReadActor;

            public void setDataSource(DataSource dataSource) {
                this.jdbcTemplate = new JdbcTemplate(dataSource);
                this.procReadActor = new SimpleJdbcCall(dataSource)
                        .withProcedureName("read_actor");
            }

            public Actor readActor(Long id) {
                SqlParameterSource in = new MapSqlParameterSource()
                        .addValue("in_id", id);
                Map out = procReadActor.execute(in);
                Actor actor = new Actor();
                actor.setId(id);
                actor.setFirstName((String) out.get("out_first_name"));
                actor.setLastName((String) out.get("out_last_name"));
                actor.setBirthDate((Date) out.get("out_birth_date"));
                return actor;
            }

            // ... additional methods
        }
        ```

        为执行调用而编写的代码涉及创建包含IN参数的SqlParameterSource。必须将为输入值提供的名称与存储过程中声明的参数名称匹配。这种情况不需要匹配，因为您使用元数据来确定应该如何在存储过程中引用数据库对象。在源中为存储过程指定的不一定是存储在数据库中的方式。一些数据库将名称转换为所有大写字母，而其他数据库则使用小写字母或指定的大小写。
        
        execute方法接受IN参数并返回一个映射，其中包含按名称键控的任何out参数，如存储过程中指定的那样。在本例中，它们是out_first_name、out_last_name和out_birth_date。
        
        execute方法的最后一部分创建一个Actor实例，用于返回检索到的数据。同样，在存储过程中声明out参数时使用它们的名称也很重要。此外，存储在结果映射中的out参数名称的情况与数据库中的out参数名称的情况相匹配，这可能会因数据库而异。为了使代码更具可移植性，您应该执行不区分大小写的查找，或者指示Spring使用LinkedCaseInsensitiveMap。要实现后者，可以创建自己的JdbcTemplate，并将setResultsMapCaseInsensitive属性设置为true。然后，您可以将这个定制的JdbcTemplate实例传递到SimpleJdbcCall的构造函数中。下面的例子显示了这种配置:

        ```java
        public class JdbcActorDao implements ActorDao {

            private SimpleJdbcCall procReadActor;

            public void setDataSource(DataSource dataSource) {
                JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
                jdbcTemplate.setResultsMapCaseInsensitive(true);
                this.procReadActor = new SimpleJdbcCall(jdbcTemplate)
                        .withProcedureName("read_actor");
            }

            // ... additional methods
        }
        ```

        通过执行此操作，可以避免在用于返回的out参数名称的情况下发生冲突。

    6. 显式声明用于SimpleJdbcCall的参数

        在本章的前面，我们描述了如何从元数据推断参数，但是如果愿意，可以显式地声明它们。您可以通过使用declareParameters方法创建和配置SimpleJdbcCall来实现这一点，declareParameters方法接受可变数量的SqlParameter对象作为输入。有关如何定义SqlParameter的详细信息，请参见下一节。

        您可以选择显式地声明一个、一些或所有参数。在没有显式声明参数的情况下，仍然使用参数元数据。要绕过所有元数据查找潜在参数的处理，只使用声明的参数，可以调用方法withoutProcedureColumnMetaDataAccess作为声明的一部分。假设您为一个数据库函数声明了两个或多个不同的调用签名。在本例中，您调用useInParameterNames来指定要包含用于给定签名的In参数名列表。
        
        下面的例子显示了一个完整声明的过程调用，并使用了来自前一个例子的信息:

        ```java
        public class JdbcActorDao implements ActorDao {

            private SimpleJdbcCall procReadActor;

            public void setDataSource(DataSource dataSource) {
                JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
                jdbcTemplate.setResultsMapCaseInsensitive(true);
                this.procReadActor = new SimpleJdbcCall(jdbcTemplate)
                        .withProcedureName("read_actor")
                        .withoutProcedureColumnMetaDataAccess()
                        .useInParameterNames("in_id")
                        .declareParameters(
                                new SqlParameter("in_id", Types.NUMERIC),
                                new SqlOutParameter("out_first_name", Types.VARCHAR),
                                new SqlOutParameter("out_last_name", Types.VARCHAR),
                                new SqlOutParameter("out_birth_date", Types.DATE)
                        );
            }

            // ... additional methods
        }
        ```

        这两个示例的执行和最终结果是相同的。第二个示例显式地指定所有细节，而不是依赖于元数据。

    7. 如何定义SqlParameters

        要为SimpleJdbc类和RDBMS操作类定义一个参数(在将JDBC操作建模为Java对象中介绍过)，可以使用SqlParameter或它的一个子类。为此，通常要在构造函数中指定参数名和SQL类型。SQL类型由java.sql指定。类型的常量。在本章前面，我们看到了类似于下面的声明:

        ```java
        new SqlParameter("in_id", Types.NUMERIC),
        new SqlOutParameter("out_first_name", Types.VARCHAR),
        ```

        带有SqlParameter的第一行声明了一个IN参数。通过使用SqlQuery及其子类(在理解SqlQuery中有所涉及)，可以在存储过程调用和查询的参数中使用。
        
        第二行(带有SqlOutParameter)声明一个out参数，用于存储过程调用。InOut参数还有一个SqlInOutParameter(为过程提供IN值并返回值的参数)。

        对于IN参数，除了名称和SQL类型外，还可以为数值数据指定比例，或者为自定义数据库类型指定类型名称。对于out参数，您可以提供一个行映射器来处理REF游标返回的行映射。另一个选项是指定SqlReturnType，它提供了定义返回值的自定义处理的机会。

    8. 使用SimpleJdbcCall调用存储函数

        您可以以几乎与调用存储过程相同的方式调用存储函数，只不过提供的是函数名而不是过程名。您可以使用withFunctionName方法作为配置的一部分来指示您想要调用一个函数，并生成一个函数调用的相应字符串。专门的执行调用(executeFunction)用于执行函数，它将函数返回值作为指定类型的对象返回，这意味着您不必从结果映射中检索返回值。对于只有一个out参数的存储过程，也可以使用类似的便利方法(名为executeObject)。下面的例子(对于MySQL)基于一个名为get_actor_name的存储函数，该函数返回一个参与者的全名:

        ```sql
        CREATE FUNCTION get_actor_name (in_id INTEGER)
        RETURNS VARCHAR(200) READS SQL DATA
        BEGIN
            DECLARE out_name VARCHAR(200);
            SELECT concat(first_name, ' ', last_name)
                INTO out_name
                FROM t_actor where id = in_id;
            RETURN out_name;
        END;
        ```

        为了调用这个函数，我们再次在初始化方法中创建一个SimpleJdbcCall，如下例所示:

        ```java
        public class JdbcActorDao implements ActorDao {

            private JdbcTemplate jdbcTemplate;
            private SimpleJdbcCall funcGetActorName;

            public void setDataSource(DataSource dataSource) {
                this.jdbcTemplate = new JdbcTemplate(dataSource);
                JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
                jdbcTemplate.setResultsMapCaseInsensitive(true);
                this.funcGetActorName = new SimpleJdbcCall(jdbcTemplate)
                        .withFunctionName("get_actor_name");
            }

            public String getActorName(Long id) {
                SqlParameterSource in = new MapSqlParameterSource()
                        .addValue("in_id", id);
                String name = funcGetActorName.executeFunction(String.class, in);
                return name;
            }

            // ... additional methods
        }
        ```

        使用的executeFunction方法返回一个字符串，该字符串包含函数调用的返回值。

    9. 从SimpleJdbcCall返回ResultSet或REF游标

        调用返回结果集的存储过程或函数有点棘手。一些数据库在JDBC结果处理期间返回结果集，而另一些数据库则需要显式注册特定类型的out参数。这两种方法都需要额外的处理来遍历结果集并处理返回的行。使用SimpleJdbcCall，您可以使用returningResultSet方法，并声明一个行映射器实现，以用于特定的参数。如果在结果处理期间返回结果集，则没有定义名称，因此返回的结果必须与声明RowMapper实现的顺序匹配。指定的名称仍然用于将处理后的结果列表存储在从execute语句返回的结果映射中。

        下一个例子(对于MySQL)使用一个存储过程，它不接受参数，并返回t_actor表中的所有行:

        ```sql
        CREATE PROCEDURE read_all_actors()
        BEGIN
        SELECT a.id, a.first_name, a.last_name, a.birth_date FROM t_actor a;
        END;
        ```

        要调用此过程，可以声明行映射器。因为要映射到的类遵循JavaBean规则，所以可以使用BeanPropertyRowMapper，它是通过在newInstance方法中传递需要映射到的类创建的。下面的例子说明了如何做到这一点:

        ```java
        public class JdbcActorDao implements ActorDao {

            private SimpleJdbcCall procReadAllActors;

            public void setDataSource(DataSource dataSource) {
                JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
                jdbcTemplate.setResultsMapCaseInsensitive(true);
                this.procReadAllActors = new SimpleJdbcCall(jdbcTemplate)
                        .withProcedureName("read_all_actors")
                        .returningResultSet("actors",
                        BeanPropertyRowMapper.newInstance(Actor.class));
            }

            public List getActorsList() {
                Map m = procReadAllActors.execute(new HashMap<String, Object>(0));
                return (List) m.get("actors");
            }

            // ... additional methods
        }
        ```

        执行调用传递一个空映射，因为这个调用不接受任何参数。然后从结果映射检索参与者列表并返回给调用者。
        
7. 将JDBC操作建模为Java对象
8. 参数和数据值处理的常见问题
9. 嵌入式数据库的支持
10. 初始化数据源

原文：https://docs.spring.io/spring/docs/current/spring-framework-reference/data-access.html