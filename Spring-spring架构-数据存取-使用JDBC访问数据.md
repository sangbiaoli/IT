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

    org.springframework.jdbc.object包允许以更面向对象的方式访问数据库的类。例如，您可以执行查询并将结果作为包含业务对象的列表返回，其中关系列数据映射到业务对象的属性。您还可以运行存储过程和运行update、delete和insert语句。

    1. 理解SqlQuery

        SqlQuery是一个可重用的、线程安全的类，它封装了一个SQL查询。子类必须实现newRowMapper(..)方法，以提供一个RowMapper实例，该实例可以在查询执行期间创建的ResultSet上迭代获得的每一行中创建一个对象。SqlQuery类很少直接使用，因为MappingSqlQuery子类为将行映射到Java类提供了更方便的实现。扩展SqlQuery的其他实现还有MappingSqlQueryWithParameters和UpdatableSqlQuery。

    2. 理解MappingSqlQuery

        MappingSqlQuery是一个可重用查询，其中具体子类必须实现抽象mapRow(..)方法，以便将提供的ResultSet的每一行转换为指定类型的对象。下面的例子显示了一个自定义查询，它将数据从t_actor关系映射到Actor类的一个实例:

        ```java
        public class ActorMappingQuery extends MappingSqlQuery<Actor> {

            public ActorMappingQuery(DataSource ds) {
                super(ds, "select id, first_name, last_name from t_actor where id = ?");
                declareParameter(new SqlParameter("id", Types.INTEGER));
                compile();
            }

            @Override
            protected Actor mapRow(ResultSet rs, int rowNumber) throws SQLException {
                Actor actor = new Actor();
                actor.setId(rs.getLong("id"));
                actor.setFirstName(rs.getString("first_name"));
                actor.setLastName(rs.getString("last_name"));
                return actor;
            }

        }
        ```

        该类扩展了使用Actor类型参数化的MappingSqlQuery。此客户查询的构造函数将数据源作为唯一参数。在这个构造函数中，您可以使用数据源和应该执行的SQL调用超类上的构造函数来检索该查询的行。此SQL用于创建PreparedStatement，因此它可能包含占位符，以便在执行期间传递任何参数。必须使用传递SqlParameter的declareParameter方法声明每个参数。SqlParameter接受一个名称，以及java.sql.Types中定义的JDBC类型。定义完所有参数后，可以调用compile()方法，以便准备语句并在稍后运行。该类编译后是线程安全的，因此，只要在初始化DAO时创建了这些实例，就可以将它们作为实例变量保存并重用。下面的例子展示了如何定义这样一个类:

        ```java
        private ActorMappingQuery actorMappingQuery;

        @Autowired
        public void setDataSource(DataSource dataSource) {
            this.actorMappingQuery = new ActorMappingQuery(dataSource);
        }

        public Customer getCustomer(Long id) {
            return actorMappingQuery.findObject(id);
        }
        ```

        前面示例中的方法检索具有作为惟一参数传入的id的客户。因为我们只想返回一个对象，所以我们调用了findObject便利方法，并将id作为参数。如果我们有一个返回对象列表并接受其他参数的查询，我们将使用其中一个执行方法，该方法接受作为varargs传入的参数值数组。下面的例子展示了这样一个方法:

        ```java
        public List<Actor> searchForActors(int age, String namePattern) {
            List<Actor> actors = actorSearchMappingQuery.execute(age, namePattern);
            return actors;
        }
        ```

    3. 使用SqlUpdate

        SqlUpdate类封装了一个SQL更新。与查询一样，update对象是可重用的，并且与所有RdbmsOperation类一样，update可以有参数，并且是用SQL定义的。该类提供了许多update(..)方法，类似于查询对象的execute(..)方法。SQLUpdate类是具体的。它可以被子类化——例如，添加一个自定义更新方法。但是，您不必子类化SqlUpdate类，因为它可以通过设置SQL和声明参数轻松地进行参数化。下面的示例创建了一个名为execute的自定义更新方法:

        ```java
        import java.sql.Types;
        import javax.sql.DataSource;
        import org.springframework.jdbc.core.SqlParameter;
        import org.springframework.jdbc.object.SqlUpdate;

        public class UpdateCreditRating extends SqlUpdate {

            public UpdateCreditRating(DataSource ds) {
                setDataSource(ds);
                setSql("update customer set credit_rating = ? where id = ?");
                declareParameter(new SqlParameter("creditRating", Types.NUMERIC));
                declareParameter(new SqlParameter("id", Types.NUMERIC));
                compile();
            }

            /**
            * @param id for the Customer to be updated
            * @param rating the new value for credit rating
            * @return number of rows updated
            */
            public int execute(int id, int rating) {
                return update(rating, id);
            }
        }
        ```

    4. 使用StoredProcedure

        StoredProcedure类是用于RDBMS存储过程对象抽象的超类。该类是抽象的，它的各种execute(..)方法是protected的，防止通过提供更严格类型的子类以外的方法使用。

        继承的sql属性是RDBMS中存储过程的名称。

        要为StoredProcedure类定义参数，可以使用SqlParameter或其子类之一。必须在构造函数中指定参数名和SQL类型，如下面的代码片段所示:

        ```java
        new SqlParameter("in_id", Types.NUMERIC),
        new SqlOutParameter("out_first_name", Types.VARCHAR),
        ```

        SQL类型是使用java.sql.Types的常量。

        第一行(带有SqlParameter)声明了一个IN参数。您可以在参数中对存储过程调用和使用SqlQuery及其子类(在理解SqlQuery中涉及)的查询使用参数。

        第二行(带有SqlOutParameter)声明一个out参数，用于存储过程调用。InOut参数还有一个SqlInOutParameter(为过程提供in值并返回值的参数)。

        对于in参数，除了名称和SQL类型外，还可以为数值数据指定比例，或者为自定义数据库类型指定类型名称。对于out参数，您可以提供一个行映射器来处理REF游标返回的行映射。另一个选项是指定SqlReturnType，它允许您定义对返回值的自定义处理。

        下一个简单DAO示例使用StoredProcedure调用函数(sysdate())，该函数随任何Oracle数据库一起提供。要使用存储过程功能，您必须创建一个扩展StoredProcedure的类。在本例中，StoredProcedure类是一个内部类。但是，如果需要重用StoredProcedure，可以将其声明为顶级类。这个示例没有输入参数，但是通过使用SqlOutParameter类将输出参数声明为日期类型。execute()方法运行该过程并从结果映射中提取返回的日期。通过使用参数名作为键，结果映射为每个声明的输出参数都有一个条目(在本例中只有一个)。下面的清单显示了我们自定义的StoredProcedure类:

        ```java
        import java.sql.Types;
        import java.util.Date;
        import java.util.HashMap;
        import java.util.Map;
        import javax.sql.DataSource;
        import org.springframework.beans.factory.annotation.Autowired;
        import org.springframework.jdbc.core.SqlOutParameter;
        import org.springframework.jdbc.object.StoredProcedure;

        public class StoredProcedureDao {

            private GetSysdateProcedure getSysdate;

            @Autowired
            public void init(DataSource dataSource) {
                this.getSysdate = new GetSysdateProcedure(dataSource);
            }

            public Date getSysdate() {
                return getSysdate.execute();
            }

            private class GetSysdateProcedure extends StoredProcedure {

                private static final String SQL = "sysdate";

                public GetSysdateProcedure(DataSource dataSource) {
                    setDataSource(dataSource);
                    setFunction(true);
                    setSql(SQL);
                    declareParameter(new SqlOutParameter("date", Types.DATE));
                    compile();
                }

                public Date execute() {
                    // the 'sysdate' sproc has no input parameters, so an empty Map is supplied...
                    Map<String, Object> results = execute(new HashMap<String, Object>());
                    Date sysdate = (Date) results.get("date");
                    return sysdate;
                }
            }

        }
        ```

        下面的StoredProcedure示例有两个输出参数(在本例中是Oracle REF游标):

        ```java
        import java.util.HashMap;
        import java.util.Map;
        import javax.sql.DataSource;
        import oracle.jdbc.OracleTypes;
        import org.springframework.jdbc.core.SqlOutParameter;
        import org.springframework.jdbc.object.StoredProcedure;

        public class TitlesAndGenresStoredProcedure extends StoredProcedure {

            private static final String SPROC_NAME = "AllTitlesAndGenres";

            public TitlesAndGenresStoredProcedure(DataSource dataSource) {
                super(dataSource, SPROC_NAME);
                declareParameter(new SqlOutParameter("titles", OracleTypes.CURSOR, new TitleMapper()));
                declareParameter(new SqlOutParameter("genres", OracleTypes.CURSOR, new GenreMapper()));
                compile();
            }

            public Map<String, Object> execute() {
                // again, this sproc has no input parameters, so an empty Map is supplied
                return super.execute(new HashMap<String, Object>());
            }
        }
        ```

        注意，在TitlesAndGenresStoredProcedure构造函数中使用的declareParameter(..)方法的重载变体是如何传递RowMapper实现实例的。这是重用现有功能的一种非常方便和强大的方法。下面的两个示例提供了两个RowMapper实现的代码。

        TitleMapper类为提供的ResultSet中的每一行将一个ResultSet映射到一个Title域对象，如下所示:

        ```java
        import java.sql.ResultSet;
        import java.sql.SQLException;
        import com.foo.domain.Title;
        import org.springframework.jdbc.core.RowMapper;

        public final class TitleMapper implements RowMapper<Title> {

            public Title mapRow(ResultSet rs, int rowNum) throws SQLException {
                Title title = new Title();
                title.setId(rs.getLong("id"));
                title.setName(rs.getString("name"));
                return title;
            }
        }
        ```

        GenreMapper类为提供的ResultSet中的每一行将一个ResultSet映射到一个类型域对象，如下所示:

        ```java
        import java.sql.ResultSet;
        import java.sql.SQLException;
        import com.foo.domain.Genre;
        import org.springframework.jdbc.core.RowMapper;

        public final class GenreMapper implements RowMapper<Genre> {

            public Genre mapRow(ResultSet rs, int rowNum) throws SQLException {
                return new Genre(rs.getString("name"));
            }
        }
        ```

        要将参数传递给RDBMS中定义有一个或多个输入参数的存储过程，可以编写一个强类型的execute(..)方法，该方法将委托给超类中的untyped execute(Map)方法，如下例所示:

        ```java
        import java.sql.Types;
        import java.util.Date;
        import java.util.HashMap;
        import java.util.Map;
        import javax.sql.DataSource;
        import oracle.jdbc.OracleTypes;
        import org.springframework.jdbc.core.SqlOutParameter;
        import org.springframework.jdbc.core.SqlParameter;
        import org.springframework.jdbc.object.StoredProcedure;

        public class TitlesAfterDateStoredProcedure extends StoredProcedure {

            private static final String SPROC_NAME = "TitlesAfterDate";
            private static final String CUTOFF_DATE_PARAM = "cutoffDate";

            public TitlesAfterDateStoredProcedure(DataSource dataSource) {
                super(dataSource, SPROC_NAME);
                declareParameter(new SqlParameter(CUTOFF_DATE_PARAM, Types.DATE);
                declareParameter(new SqlOutParameter("titles", OracleTypes.CURSOR, new TitleMapper()));
                compile();
            }

            public Map<String, Object> execute(Date cutoffDate) {
                Map<String, Object> inputs = new HashMap<String, Object>();
                inputs.put(CUTOFF_DATE_PARAM, cutoffDate);
                return super.execute(inputs);
            }
        }
        ```

8. 参数和数据值处理的常见问题

    参数和数据值的常见问题存在于Spring Framework的JDBC支持提供的不同方法中。本节将介绍如何解决这些问题。

    1. 为参数提供SQL类型信息

        通常，Spring根据传入的参数类型确定参数的SQL类型。可以显式地提供设置参数值时使用的SQL类型。这对于正确设置NULL值有时是必要的。
        您可以通过以下几种方式提供SQL类型信息:

        * JdbcTemplate的许多更新和查询方法都采用int数组形式的附加参数。这个数组使用java.sql.Types中的常量值来指示相应参数的类。为每个参数提供一个条目。
        * 您可以使用SqlParameterValue类来包装需要此附加信息的参数值。为此，为每个值创建一个新实例，并在构造函数中传递SQL类型和参数值。还可以为数值提供可选的缩放参数。
        * 对于使用命名参数的方法，可以使用SqlParameterSource类、BeanPropertySqlParameterSource或MapSqlParameterSource。它们都有为任何命名参数值注册SQL类型的方法。

    2. 处理BLOB和CLOB对象

        您可以在数据库中存储图像、其他二进制数据和大块文本。这些大对象对于二进制数据称为blob(二进制大对象)，对于字符数据称为clob(字符大对象)。在Spring中，您可以通过直接使用JdbcTemplate来处理这些大型对象，也可以在使用RDBMS对象和SimpleJdbc类提供的高级抽象时处理这些对象。所有这些方法都使用LobHandler接口的实现来实际管理LOB(大对象)数据。LobHandler通过getLobCreator方法提供对LobCreator类的访问，该方法用于创建要插入的新LOB对象。

        LobCreator和LobHandler为LOB输入和输出提供了以下支持:

        * BLOB
            * byte[]: getBlobAsBytes和setBlobAsBytes
            * InputStream: getBlobAsBinaryStream和setBlobAsBinaryStream
        * CLOB
            * String: getClobAsString和setClobAsString
            * InputStream: getClobAsAsciiStream和setClobAsAsciiStream
            * Reader:getClobAsCharacterStream和setClobAsCharacterStream

        下一个示例显示如何创建和插入BLOB。稍后，我们将展示如何从数据库中读取它。

        本例使用JdbcTemplate和AbstractLobCreatingPreparedStatementCallback的实现。它实现了一个方法setValues。这个方法提供了一个LobCreator，我们使用它来设置SQL insert语句中LOB列的值。

        对于本例，我们假设有一个变量lobHandler，它已经被设置为DefaultLobHandler的一个实例。通常通过依赖项注入设置此值。

        下面的例子展示了如何创建和插入BLOB:

        ```java
        final File blobIn = new File("spring2004.jpg");
        final InputStream blobIs = new FileInputStream(blobIn);
        final File clobIn = new File("large.txt");
        final InputStream clobIs = new FileInputStream(clobIn);
        final InputStreamReader clobReader = new InputStreamReader(clobIs);

        jdbcTemplate.execute(
            "INSERT INTO lob_table (id, a_clob, a_blob) VALUES (?, ?, ?)",
            new AbstractLobCreatingPreparedStatementCallback(lobHandler) {  //1
                protected void setValues(PreparedStatement ps, LobCreator lobCreator) throws SQLException {
                    ps.setLong(1, 1L);
                    lobCreator.setClobAsCharacterStream(ps, 2, clobReader, (int)clobIn.length());  //2
                    lobCreator.setBlobAsBinaryStream(ps, 3, blobIs, (int)blobIn.length());  //3
                }
            }
        );

        blobIs.close();
        clobReader.close();
        ```

        1. 传入(在本例中)是普通DefaultLobHandler的lobHandler。
        2. 使用setClobAsCharacterStream方法传入CLOB的内容。
        3. 使用setBlobAsBinaryStream方法传入BLOB的内容。

        现在是时候从数据库中读取LOB数据了。同样，使用具有相同实例变量lobHandler和对DefaultLobHandler的引用的JdbcTemplate。下面的例子说明了如何做到这一点:

        ```java
        List<Map<String, Object>> l = jdbcTemplate.query("select id, a_clob, a_blob from lob_table",
        new RowMapper<Map<String, Object>>() {
            public Map<String, Object> mapRow(ResultSet rs, int i) throws SQLException {
                Map<String, Object> results = new HashMap<String, Object>();
                String clobText = lobHandler.getClobAsString(rs, "a_clob");  
                results.put("CLOB", clobText);
                byte[] blobBytes = lobHandler.getBlobAsBytes(rs, "a_blob");  
                results.put("BLOB", blobBytes);
                return results;
            }
        });
        ```

    3. 传入in子句的值列表（跳过）

    4. 处理存储过程调用的复杂类型（跳过）

9. 嵌入式数据库的支持

    org.springframework.jdbc.datasource.embedded提供对嵌入式Java数据库引擎的支持。本地提供对HSQL、H2和Derby的支持。还可以使用可扩展API插入新的嵌入式数据库类型和数据源实现。

    1. 为什么使用嵌入式数据库?

        嵌入式数据库在项目的开发阶段非常有用，因为它是轻量级的。其优点包括易于配置、快速启动时间、可测试性，以及在开发过程中快速发展SQL的能力。

    2. 使用Spring XML创建嵌入式数据库

        如果想在Spring ApplicationContext中将嵌入式数据库实例公开为bean，可以使用Spring -jdbc名称空间中的嵌入式数据库标记:

        ```xml
        <jdbc:embedded-database id="dataSource" generate-name="true">
            <jdbc:script location="classpath:schema.sql"/>
            <jdbc:script location="classpath:test-data.sql"/>
        </jdbc:embedded-database>
        ```

        前面的配置创建了一个嵌入式HSQL数据库，其中填充了来自类路径根中的sql资源schema.sql和test-data.sql数据。此外，作为最佳实践，嵌入式数据库被分配一个惟一生成的名称。嵌入式数据库作为javax.sql类型的bean提供给Spring容器。然后可以根据需要将数据源注入到数据访问对象中。

    3. 以编程方式创建嵌入式数据库

        EmbeddedDatabaseBuilder类为以编程方式构建嵌入式数据库提供了一个流畅的API。当您需要在独立环境或独立集成测试中创建嵌入式数据库时，可以使用此功能，如下例所示:

        ```java
        EmbeddedDatabase db = new EmbeddedDatabaseBuilder()
        .generateUniqueName(true)
        .setType(H2)
        .setScriptEncoding("UTF-8")
        .ignoreFailedDrops(true)
        .addScript("schema.sql")
        .addScripts("user_data.sql", "country_data.sql")
        .build();

        // perform actions against the db (EmbeddedDatabase extends javax.sql.DataSource)

        db.shutdown()
        ```

        有关所有受支持选项的详细信息，请参阅EmbeddedDatabaseBuilder的javadoc。
        您还可以使用EmbeddedDatabaseBuilder使用Java配置创建嵌入式数据库，如下面的示例所示:

        ```java
        @Configuration
        public class DataSourceConfig {

            @Bean
            public DataSource dataSource() {
                return new EmbeddedDatabaseBuilder()
                        .generateUniqueName(true)
                        .setType(H2)
                        .setScriptEncoding("UTF-8")
                        .ignoreFailedDrops(true)
                        .addScript("schema.sql")
                        .addScripts("user_data.sql", "country_data.sql")
                        .build();
            }
        }
        ```

    4. 选择嵌入式数据库类型

        本节介绍如何选择Spring支持的三个嵌入式数据库之一。它包括下列主题:
        * 使用HSQL
        * 使用H2
        * 使用Derby
        
        1. 使用HSQL

            Spring支持HSQL 1.8.0及以上版本。如果没有显式指定类型，HSQL是默认的嵌入式数据库。要显式指定HSQL，请将嵌入式数据库标记的type属性设置为HSQL。如果使用builder API，则使用EmbeddedDatabaseType.HSQL调用setType(EmbeddedDatabaseType)方法。

        2. 使用H2
        
            Spring支持H2数据库。要启用H2，请将嵌入数据库标记的type属性设置为H2。如果使用builder API，则使用EmbeddedDatabaseType.H2调用setType(EmbeddedDatabaseType)方法。

        3. 使用Derby
            
            Spring支持Apache Derby 10.5及以上版本。要启用Derby，请将嵌入数据库标记的type属性设置为Derby。如果使用builder API，请使用EmbeddedDatabaseType.DERBY调用setType(EmbeddedDatabaseType)方法。


    5. 使用嵌入式数据库测试数据访问逻辑

        嵌入式数据库提供了一种轻量级的方法来测试数据访问代码。下一个示例是使用嵌入式数据库的数据访问集成测试模板。当嵌入式数据库不需要跨测试类重用时，使用这样的模板可以一次性使用。然而,如果您希望创建一个共享的嵌入式数据库在一个测试套件,考虑使用Spring和TestContext框架和配置Spring ApplicationContext的嵌入式数据库作为一个bean创建所述嵌入式数据库通过使用Spring XML和创建嵌入式数据库编程。下面的清单显示了测试模板:

        ```java
        public class DataAccessIntegrationTestTemplate {

            private EmbeddedDatabase db;

            @Before
            public void setUp() {
                // creates an HSQL in-memory database populated from default scripts
                // classpath:schema.sql and classpath:data.sql
                db = new EmbeddedDatabaseBuilder()
                        .generateUniqueName(true)
                        .addDefaultScripts()
                        .build();
            }

            @Test
            public void testDataAccess() {
                JdbcTemplate template = new JdbcTemplate(db);
                template.query( /* ... */ );
            }

            @After
            public void tearDown() {
                db.shutdown();
            }

        }
        ```

    6. 为嵌入式数据库生成惟一的名称（跳过）
    7. 扩展嵌入式数据库支持（跳过）

10. 初始化数据源（跳过）

原文：https://docs.spring.io/spring/docs/current/spring-framework-reference/data-access.html