## hibernate目录

1. Architecture
    1. Overview
2. Domain Model
    1. Mapping types
        1. Value types
        2. Entity types
    2. Naming strategies
        1. ImplicitNamingStrategy
        2. PhysicalNamingStrategy
    3. Basic Types
        1. Hibernate-provided BasicTypes
        2. The @Basic annotation
        3. The @Column annotation
        4. BasicTypeRegistry
        5. Explicit BasicTypes
        6. Custom BasicTypes
        7. Mapping enums
        8. Mapping LOBs
        9. Mapping Nationalized Character Data
        10. Mapping UUID Values
        11. UUID as binary
        12. UUID as (var)char
        13. PostgreSQL-specific UUID
        14. UUID as identifier
        15. Mapping Date/Time Values
        16. AttributeConverters
        17. SQL quoted identifiers
        18. Generated properties
        19. Column transformers: read and write expressions
        20. @Formula
        21. @Where
        22. @WhereJoinTable
        23. @Filter
        24. @FilterJoinTable
        25. @Filter with @SqlFragmentAlias
        26. @Any mapping
        27. @JoinFormula mapping
        28. @JoinColumnOrFormula mapping
        29. @Target mapping
        30. @Parent mapping
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
3. Bootstrap
    1. Native Bootstrapping
        1. Building the ServiceRegistry
        2. Event Listener registration
        3. Building the Metadata
        4. Building the SessionFactory
    2. JPA Bootstrapping
        1. JPA-compliant bootstrapping
        2. Externalizing XML mapping files
        3. Configuring the SessionFactory Metadata via the JPA bootstrap
4. Schema generation
    1. Importing script files
    2. Database objects
    3. Database-level checks
    4. Default value for a database column
    5. Columns unique constraint
    6. Columns index
5. Persistence Context
    1. Accessing Hibernate APIs from JPA
    2. Bytecode Enhancement
        1. Capabilities
        2. Performing enhancement
    3. Making entities persistent
    4. Deleting (removing) entities
    5. Obtain an entity reference without initializing its data
    6. Obtain an entity with its data initialized
    7. Obtain an entity by natural-id
    8. Modifying managed/persistent state
        1. Dynamic updates
    9. Refresh entity state
        1. Refresh gotchas
    10. Working with detached data
        1. Reattaching detached data
        2. Merging detached data
    11. Checking persistent state
    12. Evicting entities
    13. Cascading entity state transitions
        1. CascadeType.PERSIST
        2. CascadeType.MERGE
        3. CascadeType.REMOVE
        4. CascadeType.DETACH
        5. CascadeType.LOCK
        6. CascadeType.REFRESH
        7. CascadeType.REPLICATE
        8. @OnDelete cascade
    14. Exception handling
6. Flushing
    1. AUTO flush
        1. AUTO flush on commit
        2. AUTO flush on JPQL/HQL query
        3. AUTO flush on native SQL query
    2. COMMIT flush
    3. ALWAYS flush
    4. MANUAL flush
    5. Flush operation order
7. Database access
    1. ConnectionProvider
    2. Using DataSources
    3p0
    4. Using Proxool
        1. Using existing Proxool pools
        2. Configuring Proxool via XML
        3. Configuring Proxool via Properties
    5. Using HikariCP
    6. Using Vibur DBCP
    7. Using Agroal
    8. Using Hibernate’s built-in (and unsupported) pooling
    9. User-provided Connections
    10. ConnectionProvider support for transaction isolation setting
    11. Database Dialect
8. Transactions and concurrency control
    1. Physical Transactions
    2. JTA configuration
    3. Hibernate Transaction API
    4. Contextual sessions
    5. Transactional patterns (and anti-patterns)
        1. Session-per-operation anti-pattern
        2. Session-per-request pattern
        3. Conversations (application-level transactions)
        4. Session-per-application anti-pattern
9. JNDI
10. Locking
    1. Optimistic
    1. Mapping optimistic locking
    2. Pessimistic
    3. LockMode and LockModeType
    4. JPA locking query hints
    5. The buildLockRequest API
    6. Follow-on-locking
11. Fetching
    1. The basics
    2. Direct fetching vs. entity queries
    3. Applying fetch strategies
    4. No fetching
    5. Dynamic fetching via queries
    6. Dynamic fetching via JPA entity graph
        1. JPA entity subgraphs
    7. Dynamic fetching via Hibernate profiles
    8. Batch fetching
    9. The @Fetch annotation mapping
    10. FetchMode.SELECT
    11. FetchMode.SUBSELECT
    12. FetchMode.JOIN
    13. @LazyCollection
12. Batching
    1. JDBC batching
    2. Session batching
    1. Batch inserts
    2. Session scroll
    3. StatelessSession
    3. Hibernate Query Language for DML
    1. HQL/JPQL for UPDATE and DELETE
    2. HQL syntax for INSERT
    3. Bulk-id strategies
13. Caching
    1. Configuring second-level caching
        1. RegionFactory
        2. Caching configuration properties
        2. Configuring second-level cache mappings
        3. Entity inheritance and second-level cache mapping
    4. Entity cache
    5. Collection cache
    6. Query cache
        1. Query cache regions
    7. Managing the cached data
        1. Evicting cache entries
    8. Caching statistics
    9. JCache
        1. RegionFactory
        2. JCache CacheManager
        3. JCache missing cache strategy
    10. Ehcache
        1. RegionFactory
        2. Ehcache missing cache strategy
    11. Infinispan
14. Interceptors and events
    1. Interceptors
    2. Native Event system
    3. Mixing Events and Interceptors
    4. Hibernate declarative security
    5. JPA Callbacks
    6. Default entity listeners
        1. Exclude default entity listeners
15. HQL and JPQL
    1. Query API
    2. Example domain model
    3. JPA Query API
    4. Hibernate Query API
    1. Query scrolling
    5. Case Sensitivity
    6. Statement types
    7. Select statements
    8. Update statements
    9. Delete statements
    10. Insert statements
    11. The FROM clause
    12. Identification variables
    13. Root entity references
    14. Explicit joins
    15. Implicit joins (path expressions)
    16. Distinct
        1. Using DISTINCT with SQL projections
        2. Using DISTINCT with entity queries
    17. Collection member references
    18. Special case - qualified path expressions
    19. Polymorphism
    20. Expressions
    21. Identification variable
    22. Path expressions
    23. Literals
    24. Arithmetic
    25. Concatenation (operation)
    26. Aggregate functions
    27. Scalar functions
    28. JPQL standardized functions
    29. HQL functions
    30. Non-standardized functions
    31. Collection-related expressions
    32. Entity type
    33. CASE expressions
    34. Simple CASE expressions
    35. Searched CASE expressions
    36. NULLIF expressions
    37. COALESCE expressions
    38. The SELECT clause
    39. Predicates
    40. Relational comparisons
    41. Nullness predicate
    42. Like predicate
    43. Between predicate
    44. In predicate
    45. Exists predicate
    46. Empty collection predicate
    47. Member-of collection predicate
    48. NOT predicate operator
    49. AND predicate operator
    50. OR predicate operator
    51. The WHERE clause
    52. Group by
    53. Order by
    54. Read-only entities
16. Criteria
    1. Typed criteria queries
    2. Selecting an entity
    3. Selecting an expression
    4. Selecting multiple values
    5. Selecting a wrapper
    6. Tuple criteria queries
    7. FROM clause
    8. Roots
    9. Joins
    10. Fetches
    11. Path expressions
    12. Using parameters
    13. Using group by
17. Native SQL Queries
    1. Creating a native query using JPA
    2. Scalar queries
    3. Entity queries
    4. Handling associations and collections
    5. Returning multiple entities
    6. Alias and property references
    7. Returning DTOs (Data Transfer Objects)
    8. Handling inheritance
    9. Parameters
    10. Named SQL queries
        1. Named SQL queries selecting scalar values
        2. Named SQL queries selecting entities
    11. Resolving global catalog and schema in native SQL queries
    12. Using stored procedures for querying
    13. Using named queries to call stored procedures
    14. Custom SQL for CRUD (Create, Read, Update and Delete)
18. Spatial
    1. Overview
    2. Configuration
    1. Dependency
    2. Dialects
    3. Types
19. Multitenancy
    1. What is multitenancy?
    2. Multitenant data approaches
        1. Separate database
        2. Separate schema
    3. Partitioned (discriminator) data
    4. Multitenancy in Hibernate
        1. MultiTenantConnectionProvider
        2. CurrentTenantIdentifierResolver
        3. Caching
20. OSGi
    1. OSGi Specification and Environment
    2. hibernate-osgi
    3. features.xml
    4. QuickStarts/Demos
    5. Container-Managed JPA
    6. Enterprise OSGi JPA Container
    7. persistence.xml
    8. DataSource
    9. Bundle Package Imports
    10. Obtaining an EntityManger
    11. Unmanaged JPA
    12. persistence.xml
    13. Bundle Package Imports
    14. Obtaining an EntityMangerFactory
    15. Unmanaged Native
    16. Bundle Package Imports
    17. Obtaining a SessionFactory
    18. Optional Modules
    19. Extension Points
    20. Caveats
21. Envers
    1. Basics
    2. Configuration Properties
    3. Additional mapping annotations
    4. Choosing an audit strategy
    1. Configuring the ValidityAuditStrategy
    5. Revision Log
    6. Tracking entity names modified during revisions
    7. Tracking entity changes at the property level
    8. Queries
    9. Querying for entities of a class at a given revision
    10. Querying for entities using filtering criteria
    11. Querying for revisions, at which entities of a given class changed
    12. Querying for entity revisions that modified a given property
    13. Querying for revisions of entity including property names that were modified
    14. Querying for entity types modified in a given revision
    15. Querying for entities using entity relation joins
    16. Querying for revision information without loading entities
    17. Conditional auditing
    18. Understanding the Envers Schema
    2ddl tool
    20. Mapping exceptions
        1. What isn’t and will not be supported
        2. What isn’t and will be supported
    21. @OneToMany with @JoinColumn
    22. Advanced: Audit table partitioning
    23. Benefits of audit table partitioning
    24. Suitable columns for audit table partitioning
    25. Audit table partitioning example
    26. Determining a suitable partitioning column
    27. Determining a suitable partitioning scheme
    28. Envers links
22. Database Portability Considerations
    1. Portability Basics
    2. Dialect
    3. Dialect resolution
    4. Identifier generation
    5. Database functions
    6. Type mappings
        1. BLOB/CLOB mappings
        2. Boolean mappings
23. Configurations
    1. Strategy configurations
    2. General configuration
    3. JPA compliance
    4. Database connection properties
        1. Hibernate internal connection pool options
    0 properties
    6. Mapping Properties
    1. Table qualifying options
    2. Identifier options
    3. Quoting options
    4. Discriminator options
    5. Naming strategies
    6. Metadata scanning options
    7. JDBC-related options
    8. Bean Validation options
    9. Misc options
    7. Bytecode Enhancement Properties
    8. Query settings
    1. Multi-table bulk HQL operations
    9. Batching properties
    1. Fetching properties
    10. Statement logging and statistics
        1. SQL statement logging
        2. Statistics settings
    11. Cache Properties
    12. Infinispan properties
    13. Transactions properties
    14. Multi-tenancy settings
    15. Automatic schema generation
    16. Exception handling
    17. Session events
    18. JMX settings
    19. JACC settings
    20. ClassLoaders properties
    21. Bootstrap properties
    22. Miscellaneous properties
    23. Envers properties
    24. Spatial properties
    25. Internal properties
24. Mapping annotations
    1. JPA annotations
        1. @Access
        2. @AssociationOverride
        3. @AssociationOverrides
        4. @AttributeOverride
        5. @AttributeOverrides
        6. @Basic
        7. @Cacheable
        8. @CollectionTable
        9. @Column
        10. @ColumnResult
        11. @ConstructorResult
        12. @Convert
        13. @Converter
        14. @Converts
        15. @DiscriminatorColumn
        16. @DiscriminatorValue
        17. @ElementCollection
        18. @Embeddable
        19. @Embedded
        20. @EmbeddedId
        21. @Entity
        22. @EntityListeners
        23. @EntityResult
        24. @Enumerated
        25. @ExcludeDefaultListeners
        26. @ExcludeSuperclassListeners
        27. @FieldResult
        28. @ForeignKey
        29. @GeneratedValue
        30. @Id
        31. @IdClass
        32. @Index
        33. @Inheritance
        34. @JoinColumn
        35. @JoinColumns
        36. @JoinTable
        37. @Lob
        38. @ManyToMany
        39. @ManyToOne
        40. @MapKey
        41. @MapKeyClass
        42. @MapKeyColumn
        43. @MapKeyEnumerated
        44. @MapKeyJoinColumn
        45. @MapKeyJoinColumns
        46. @MapKeyTemporal
        47. @MappedSuperclass
        48. @MapsId
        49. @NamedAttributeNode
        50. @NamedEntityGraph
        51. @NamedEntityGraphs
        52. @NamedNativeQueries
        53. @NamedNativeQuery
        54. @NamedQueries
        55. @NamedQuery
        56. @NamedStoredProcedureQueries
        57. @NamedStoredProcedureQuery
        58. @NamedSubgraph
        59. @OneToMany
        60. @OneToOne
        61. @OrderBy
        62. @OrderColumn
        63. @PersistenceContext
        64. @PersistenceContexts
        65. @PersistenceProperty
        66. @PersistenceUnit
        67. @PersistenceUnits
        68. @PostLoad
        69. @PostPersist
        70. @PostRemove
        71. @PostUpdate
        72. @PrePersist
        73. @PreRemove
        74. @PreUpdate
        75. @PrimaryKeyJoinColumn
        76. @PrimaryKeyJoinColumns
        77. @QueryHint
        78. @SecondaryTable
        79. @SecondaryTables
        80. @SequenceGenerator
        81. @SqlResultSetMapping
        82. @SqlResultSetMappings
        83. @StoredProcedureParameter
        84. @Table
        85. @TableGenerator
        86. @Temporal
        87. @Transient
        88. @UniqueConstraint
        89. @Version
    2. Hibernate annotations
        1. @AccessType
        2. @Any
        3. @AnyMetaDef
        4. @AnyMetaDefs
        5. @AttributeAccessor
        6. @BatchSize
        7. @Cache
        8. @Cascade
        9. @Check
        10. @CollectionId
        11. @CollectionType
        12. @ColumnDefault
        13. @Columns
        14. @ColumnTransformer
        15. @ColumnTransformers
        16. @CreationTimestamp
        17. @DiscriminatorFormula
        18. @DiscriminatorOptions
        19. @DynamicInsert
        20. @DynamicUpdate
        21. @Entity
        22. @Fetch
        23. @FetchProfile
        24. @FetchProfile.FetchOverride
        25. @FetchProfiles
        26. @Filter
        27. @FilterDef
        28. @FilterDefs
        29. @FilterJoinTable
        30. @FilterJoinTables
        31. @Filters
        32. @ForeignKey
        33. @Formula
        34. @Generated
        35. @GeneratorType
        36. @GenericGenerator
        37. @GenericGenerators
        38. @Immutable
        39. @Index
        40. @IndexColumn
        41. @JoinColumnOrFormula
        42. @JoinColumnsOrFormulas
        43. @JoinFormula
        44. @LazyCollection
        45. @LazyGroup
        46. @LazyToOne
        47. @ListIndexBase
        48. @Loader
        49. @ManyToAny
        50. @MapKeyType
        51. @MetaValue
        52. @NamedNativeQueries
        53. @NamedNativeQuery
        54. @NamedQueries
        55. @NamedQuery
        56. @Nationalized
        57. @NaturalId
        58. @NaturalIdCache
        59. @NotFound
        60. @OnDelete
        61. @OptimisticLock
        62. @OptimisticLocking
        63. @OrderBy
        64. @ParamDef
        65. @Parameter
        66. @Parent
        67. @Persister
        68. @Polymorphism
        69. @Proxy
        70. @RowId
        71. @SelectBeforeUpdate
        72. @Sort
        73. @SortComparator
        74. @SortNatural
        75. @Source
        76. @SQLDelete
        77. @SQLDeleteAll
        78. @SqlFragmentAlias
        79. @SQLInsert
        80. @SQLUpdate
        81. @Subselect
        82. @Synchronize
        83. @Table
        84. @Tables
        85. @Target
        86. @Tuplizer
        87. @Tuplizers
        88. @Type
        89. @TypeDef
        90. @TypeDefs
        91. @UpdateTimestamp
        92. @ValueGenerationType
        93. @Where
        94. @WhereJoinTable
25. Performance Tuning and Best Practices
    1. Schema management
    2. Logging
    3. JDBC batching
    4. Mapping
        1. Identifiers
        2. Associations
    5. Inheritance
    6. Fetching
        1. Fetching associations
    7. Caching
26. Legacy Bootstrapping
27. Migration
28. Legacy Domain Model
29. Legacy Hibernate Criteria Queries
    1. Creating a Criteria instance
    2. JPA vs. Hibernate entity name
    3. Narrowing the result set
    4. Ordering the results
    5. Associations
    6. Dynamic association fetching
    7. Components
    8. Collections
    9. Example queries
    10. Projections, aggregation and grouping
    11. Detached queries and subqueries
    12. Queries by natural identifier
30. Legacy Hibernate Native Queries
    1. Legacy Named SQL queries
    2. Legacy return-property to explicitly specify column/alias names
    3. Legacy stored procedures for querying
    4. Legacy rules/limitations for using stored procedures
    5. Legacy custom SQL for create, update and delete
    6. Legacy custom SQL for loading
31. References



原文：https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html
