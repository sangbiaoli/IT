## 领域模型

术语域模型来自于数据建模领域。它是最终描述您所工作的问题领域的模型。有时您还会听到持久性类这个术语。

最终，应用程序域模型是ORM中的中心字符。它们构成您希望映射的类。如果这些类遵循普通旧Java对象(POJO) / JavaBean编程模型，Hibernate的工作效果最好。然而，这些规则都不是硬性要求。实际上，Hibernate很少假设持久性对象的性质。您可以用其他方式表示域模型(例如使用java.util.Map实例的树)。

从历史上看，使用Hibernate的应用程序会为此使用其专有的XML映射文件格式。随着JPA的到来，现在大部分信息都是使用注释(和/或标准化XML格式)在ORM/JPA提供者之间以一种可移植的方式定义的。本章将尽可能关注JPA映射。对于JPA不支持的Hibernate映射特性，我们更喜欢Hibernate扩展注释。

1. 映射类型
    
    Hibernate同时理解应用程序数据的Java和JDBC表示。从数据库读取/写入数据的能力是Hibernate类型的功能。在本例中，类型是org.hibernate.type.Type接口的实现。这种Hibernate类型还描述了Java类型的各种行为方面，比如如何检查是否相等、如何克隆值等等。

    为了帮助理解类型分类，让我们看一下希望映射的简单表和域模型。

    ```sql
    create table Contact (
        id integer not null,
        first varchar(255),
        last varchar(255),
        middle varchar(255),
        notes varchar(255),
        starred boolean not null,
        website varchar(255),
        primary key (id)
    )
    ```

    ```java
    @Entity(name = "Contact")
    public static class Contact {

        @Id
        private Integer id;

        private Name name;

        private String notes;

        private URL website;

        private boolean starred;

        //Getters and setters are omitted for brevity
    }

    @Embeddable
    public class Name {

        private String first;

        private String middle;

        private String last;

        // getters and setters omitted
    }
    ```

    从最广泛的意义上说，Hibernate将类型分为两组:
    * 值类型
    * 实体类型

    1. 值类型

        值类型是一段不定义其自身生命周期的数据。实际上，它由定义其生命周期的实体拥有。
        
        从另一个角度看，实体的所有状态都完全由值类型组成。这些状态字段或JavaBean属性称为持久属性。Contact类的持久属性是值类型。
        
        值类型进一步分为三类:

        * 基本类型
            
            在映射Contact表时，除了name之外的所有属性都是基本类型。基本类型将在基本类型中详细讨论
        * 可嵌入的类型
            
            name属性是可嵌入类型的一个例子，将在可嵌入类型中详细讨论
        * 集合类型
            
            虽然前面的示例中没有提供集合类型的功能，但是集合类型也是值类型中的一个不同类别。集合类型将在集合中进一步讨论

    2. 实体类型

        实体由于其惟一标识符的性质独立于其他对象而值不独立存在。实体是使用唯一标识符与数据库表中的行关联的域模型类。由于需要惟一标识符，实体独立存在并定义自己的生命周期。Contact类本身就是实体的一个例子。

        实体中详细讨论了映射实体。

2. 命名策略

    对象模型到关系数据库的映射的一部分是将对象模型的名称映射到相应的数据库名称。Hibernate把这个过程看作两个阶段:

    * 第一个阶段是从域模型映射确定一个合适的逻辑名称。逻辑名称可以由用户显式地指定(例如使用@Column或@Table)，也可以由Hibernate通过ImplicitNamingStrategy约定隐式地确定。

    * 其次是将逻辑名称解析为物理名称，物理名称由PhysicalNamingStrategy约定定义。

    在核心，每个命名策略背后的思想都是最小化开发人员映射域模型时必须提供的重复信息的数量。

    1. ImplicitNamingStrategy

        当一个实体没有显式地命名它映射到的数据库表时，我们需要隐式地确定该表名。或者当某个特定属性没有显式地指定它映射到的数据库列时，我们需要隐式地确定该列名。下面是org.hibernate.boot.model.naming.ImplicitNamingStrategy角色的示例。当映射没有提供显式名称时，使用ImplicitNamingStrategy约定来确定逻辑名称。

        ![](orm/orm-hibernate-domain_model-implicit_naming_strategy_diagram.svg)

        Hibernate定义了多个ImplicitNamingStrategy的开箱即用实现。应用程序也可以免费使用插件自定义实现。

        有多种方法可以指定要使用的隐式命名策略。首先，应用程序可以使用hibernate.implicit_naming_strategy配置设置来指定实现，可以接受以下规则:
        
        * 为开箱即用的实现预定义“短名称”
            * default
            
                org.hibernate.boot.model.naming.ImplicitNamingStrategyJpaCompliantImpl-jpa别名

            * jpa

                org.hibernate.boot.model.naming.ImplicitNamingStrategyJpaCompliantImpl-JPA 2.0兼容的命名策略

            * legacy-hbm

                org.hibernate.boot.model.naming.ImplicitNamingStrategyLegacyHbmImpl-与原始Hibernate命名策略兼容

            * legacy-jpa

                org.hibernate.boot.model.naming.ImplicitNamingStrategyLegacyJpaImpl-与为JPA 1.0开发的遗留命名策略兼容，遗憾的是，在许多方面隐式命名规则都不清楚

            * component-path

                org.hibernate.boot.model.naming.ImplicitNamingStrategyComponentPathImpl-大多数遵循ImplicitNamingStrategyJpaCompliantImpl规则，除了它使用完整的复合路径，而不只是结束属性部分

        * 引用一个org.hibernate.boot.model.naming.ImplicitNamingStrategy的实现类
        * org.hibernate.boot.model.naming.ImplicitNamingStrategy的实现类的FQN(完全限定名)

    2. PhysicalNamingStrategy

        许多组织围绕数据库对象(表、列、外键等)的命名定义规则。PhysicalNamingStrategy的概念是帮助实现这样的命名规则，而不必通过显式的名称将它们硬编码到映射中。

        虽然ImplicitNamingStrategy的目的是确定一个名为accountNumber的属性映射到一个逻辑列名accountNumber当没有明确指定,PhysicalNamingStrategy的目的,例如,说应该略acct_num物理列名称。

        默认实现是简单地使用逻辑名称作为物理名称。然而，应用程序和集成可以定义这个PhysicalNamingStrategy约定的自定义实现。下面是一个名为Acme Corp的虚构公司的PhysicalNamingStrategy示例，该公司的命名标准是:
                
        * 喜欢用分数分隔的单词，而不是用驼峰框
        * 用标准缩写代替某些单词

        ```java
        /*
        * Hibernate, Relational Persistence for Idiomatic Java
        *
        * License: GNU Lesser General Public License (LGPL), version 2.1 or later.
        * See the lgpl.txt file in the root directory or <http://www.gnu.org/licenses/lgpl-2.1.html>.
        */
        package org.hibernate.userguide.naming;

        import java.util.LinkedList;
        import java.util.List;
        import java.util.Locale;
        import java.util.Map;
        import java.util.TreeMap;

        import org.hibernate.boot.model.naming.Identifier;
        import org.hibernate.boot.model.naming.PhysicalNamingStrategy;
        import org.hibernate.engine.jdbc.env.spi.JdbcEnvironment;

        import org.apache.commons.lang3.StringUtils;

        /**
        * An example PhysicalNamingStrategy that implements database object naming standards
        * for our fictitious company Acme Corp.
        * <p/>
        * In general Acme Corp prefers underscore-delimited words rather than camel casing.
        * <p/>
        * Additionally standards call for the replacement of certain words with abbreviations.
        *
        * @author Steve Ebersole
        */
        public class AcmeCorpPhysicalNamingStrategy implements PhysicalNamingStrategy {
            private static final Map<String,String> ABBREVIATIONS = buildAbbreviationMap();

            @Override
            public Identifier toPhysicalCatalogName(Identifier name, JdbcEnvironment jdbcEnvironment) {
                // Acme naming standards do not apply to catalog names
                return name;
            }

            @Override
            public Identifier toPhysicalSchemaName(Identifier name, JdbcEnvironment jdbcEnvironment) {
                // Acme naming standards do not apply to schema names
                return name;
            }

            @Override
            public Identifier toPhysicalTableName(Identifier name, JdbcEnvironment jdbcEnvironment) {
                final List<String> parts = splitAndReplace( name.getText() );
                return jdbcEnvironment.getIdentifierHelper().toIdentifier(
                        join( parts ),
                        name.isQuoted()
                );
            }

            @Override
            public Identifier toPhysicalSequenceName(Identifier name, JdbcEnvironment jdbcEnvironment) {
                final LinkedList<String> parts = splitAndReplace( name.getText() );
                // Acme Corp says all sequences should end with _seq
                if ( !"seq".equalsIgnoreCase( parts.getLast() ) ) {
                    parts.add( "seq" );
                }
                return jdbcEnvironment.getIdentifierHelper().toIdentifier(
                        join( parts ),
                        name.isQuoted()
                );
            }

            @Override
            public Identifier toPhysicalColumnName(Identifier name, JdbcEnvironment jdbcEnvironment) {
                final List<String> parts = splitAndReplace( name.getText() );
                return jdbcEnvironment.getIdentifierHelper().toIdentifier(
                        join( parts ),
                        name.isQuoted()
                );
            }

            private static Map<String, String> buildAbbreviationMap() {
                TreeMap<String,String> abbreviationMap = new TreeMap<> ( String.CASE_INSENSITIVE_ORDER );
                abbreviationMap.put( "account", "acct" );
                abbreviationMap.put( "number", "num" );
                return abbreviationMap;
            }

            private LinkedList<String> splitAndReplace(String name) {
                LinkedList<String> result = new LinkedList<>();
                for ( String part : StringUtils.splitByCharacterTypeCamelCase( name ) ) {
                    if ( part == null || part.trim().isEmpty() ) {
                        // skip null and space
                        continue;
                    }
                    part = applyAbbreviationReplacement( part );
                    result.add( part.toLowerCase( Locale.ROOT ) );
                }
                return result;
            }

            private String applyAbbreviationReplacement(String word) {
                if ( ABBREVIATIONS.containsKey( word ) ) {
                    return ABBREVIATIONS.get( word );
                }

                return word;
            }

            private String join(List<String> parts) {
                boolean firstPass = true;
                String separator = "";
                StringBuilder joined = new StringBuilder();
                for ( String part : parts ) {
                    joined.append( separator ).append( part );
                    if ( firstPass ) {
                        firstPass = false;
                        separator = "_";
                    }
                }
                return joined.toString();
            }
        }
        ```

        有多种方法可以指定要使用的PhysicalNamingStrategy。

        * 首先，应用程序可以使用hibernate指定实现。physical_naming_strategy配置设置，可接受:
            * 引用一个org.hibernate.boot.model.naming.PhysicalNamingStrategy实现类
            * org.hibernate.boot.model.naming.PhysicalNamingStrategy实现类的FQN(完全限定名)
        * 其次，应用程序和集成可以y用org.hibernate.boot.MetadataBuilder.applyPhysicalNamingStrategy方法发挥作用

3. 基本类型

    基本值类型通常将单个数据库列映射到单个非聚合Java类型。Hibernate提供了许多内置的基本类型，它们遵循JDBC规范推荐的自然映射。
    Hibernate在内部使用基本类型注册表来解析特定的org.hibernate.type.Type。

    1. Hibernate提供的基本类型

        1. 标准的基本类型（举例）

            Hibernate type (org.hibernate.type package)	|JDBC type	|Java type	|BasicTypeRegistry key(s)
            --|--|--|--
            StringType|VARCHAR|java.lang.String|string, java.lang.String
            MaterializedClob|CLOB|java.lang.String|materialized_clob

        2.  Java 8基本类型（举例）

            Hibernate type (org.hibernate.type package)	|JDBC type	|Java type	|BasicTypeRegistry key(s)
            --|--|--|--
            DurationType|BIGINT|java.time.Duration|Duration, java.time.Duration
            InstantType|TIMESTAMP|java.time.Instant|Instant, java.time.Instant

        3. Hibernate有限制的基本类型

            Hibernate type (org.hibernate.type package)	|JDBC type	|Java type	|BasicTypeRegistry key(s)
            --|--|--|--
            JTSGeometryType|depends on the dialect|com.vividsolutions.jts.geom.Geometry|jts_geometry, or the class name of Geometry or any of its subclasses
            GeolatteGeometryType|depends on the dialect|org.geolatte.geom.Geometry|geolatte_geometry, or the class name of Geometry or any of its subclasses

        这些映射由Hibernate中名为org.hibernate.type.BasicTypeRegistry的服务管理着，它本质上维护org.hibernate.type.BasicType的映射org.hibernate.type.Type的特例)，这就是前面表中“BasicTypeRegistry key(s)”列的目的。

    2. @Basic注释
    3. @ column注释
    4. BasicTypeRegistry
    5. 明确BasicTypes
    6. 自定义BasicTypes
    7. 映射枚举
    8. 映射lob
    9. 映射国有化字符数据
    10. 映射UUID值
    11. UUID作为二进制
    12. UUID (var)字符
    13. PostgreSQL-specific UUID
    14. UUID作为标识符
    15. 日期/时间值的映射
    16. AttributeConverters
    17. SQL引用标识符
    18. 生成的属性
    19. 列转换器:读写表达式
    20. @Formula
    21. @Where
    22. @WhereJoinTable
    23. @Filter
    24. @FilterJoinTable
    25. @Filter及@SqlFragmentAlias
    26. @Any映射
    27. @JoinFormula映射
    28. @JoinColumnOrFormula映射
    29. @Target映射
    30. @ parent映射
4. Embeddable types
    1. Component / Embedded
    2. Multiple embeddable types
    3. Overriding Embeddable types
    4. Embeddables and ImplicitNamingStrategy
    5. Collections of embeddable types
    6. Embeddable type as a Map key
    7. Embeddable type as identifier
5. Entity types
    1. POJO Models
    2. Prefer non-final classes
    3. Implement a no-argument constructor
    4. Declare getters and setters for persistent attributes
    5. Providing identifier attribute(s)
    6. Mapping the entity
    7. Implementing equals() and hashCode()
    8. Mapping the entity to a SQL query
    9. Define a custom entity proxy
    10. Dynamic entity proxies using the @Tuplizer annotation
    11. Define a custom entity persister
    12. Access strategies
6. Identifiers
    1. Simple identifiers
    2. Composite identifiers
    3. Composite identifiers with @EmbeddedId
    4. Composite identifiers with @IdClass
    5. Composite identifiers with associations
    6. Generated identifier values
    7. Interpreting AUTO
    8. Using sequences
    9. Using IDENTITY columns
    10. Using the table identifier generator
    11. Using UUID generation
    12. Optimizers
    13. Using @GenericGenerator
    14. Derived Identifiers
    15. @RowId
7. Associations
    1. @ManyToOne
    2. @OneToMany
    3. @OneToOne
    4. @ManyToMany
    5. @NotFound association mapping
8. Collections
    1. Collections as a value type
    2. Collections of value types
    3. Collections of entities
    4. Bags
    5. Ordered Lists
    6. Sets
    7. Sorted sets
    8. Maps
    9. Arrays
    10. Arrays as binary
    11. Collections as basic value type
    12. Custom collection types
9. Natural Ids
    1. Natural Id Mapping
    2. Natural Id API
    3. Natural Id - Mutability and Caching
    10. Dynamic Model
    1. Dynamic mapping models
    11. Inheritance
    1. MappedSuperclass
    2. Single table
    3. Joined table
    4. Table per class
    5. Implicit and explicit polymorphism
    12. Immutability
    1. Entity immutability
    2. Collection immutability

原文：https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html
