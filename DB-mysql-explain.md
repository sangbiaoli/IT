## explain

1. explain输出列

    列|JSON名称|含义
    --|--|--
    id|select_id|选择标识符
    select_type|None|选择类型
    table|table_name|输出行的表
    partitions|partitions|匹配部分
    type|access_type|join类型
    possible_keys|possible_keys|要选择的可能索引
    key|key|实际选中索引
    key_len|key_length|选中索引的长度
    ref|ref|与索引进行比较的列
    rows|rows|要检查的行的估计值
    filtered|filtered|按表条件过滤的行的百分比
    Extra|None|附加信息

    1. id (JSON name: select_id)

        选择标识符。这是查询中SELECT的序号。如果行引用其他行的并集结果，则该值可以为空。在本例中，表列显示了一个值，如<unionM,N>，表示行引用id值为M和N的行的并集。

    2. select_type (JSON name: none)

        SELECT的类型，可以是下表中所示的任何类型。json格式的解释将SELECT类型公开为query_block的属性，除非它是SIMPLE或PRIMARY。JSON名称(如适用)也显示在表中。

        select_type值|JSON名称|含义
        --|--|--
        SIMPLE|None|简单SELECT(不使用UNION或子查询)
        PRIMARY|None|外层的SELECT
        UNION|None|UNION中的第二个或多个SELECT语句
        DEPENDENT UNION|dependent (true)|UNION中的第二个或多个SELECT语句，依赖于外部查询
        UNION RESULT|union_result|UNION的结果
        SUBQUERY|None|在子查询中首先选择
        DEPENDENT SUBQUERY|dependent (true)|在子查询中首先选择，依赖于外部查询
        DERIVED|None|派生表
        DEPENDENT DERIVED|dependent (true)|依赖于另一个表的派生表
        MATERIALIZED|materialized_from_subquery|物化子查询
        UNCACHEABLE SUBQUERY|cacheable (false)|无法缓存结果的子查询，必须为外部查询的每一行重新评估该子查询
        UNCACHEABLE UNION|cacheable (false)|属于非可缓存子查询的联合中的第二个或更晚的选择(参见非可缓存子查询)

    3. table (JSON name: table_name)

        输出行所指向的表的名称。这也可以是下列值之一:

        * <unionM,N>:id值为M和N的行的并集的行
        * <derivedN>:行引用id值为n的行的派生表结果。例如，派生表可能来自from子句中的子查询。
        * <subqueryN>: 该行引用id值为N的行的物化子查询的结果。

    4. partitions (JSON name: partitions)

        查询将匹配记录所在的分区。对于非分区表，该值为NULL。

    5. type (JSON name: access_type)

        join的类型

    6. possible_keys (JSON name: possible_keys)

        possible le_keys列指示MySQL可以从中选择哪些索引来查找该表中的行。注意，这一列完全独立于EXPLAIN输出中显示的表的顺序。这意味着在possible le_keys中的一些键在实际中生成表顺序可能无法使用。

    7. key (JSON name: key)

        key列表示MySQL实际决定使用的键(索引)。如果MySQL决定使用一个possible le_keys索引来查找行，那么该索引将作为键值列出。

    8. key_len (JSON name: key_length)

        key_len列表示MySQL决定使用的密钥的长度。key_len的值使您能够确定MySQL实际使用的multiple-part密钥有多少parts。如果键列说NULL, len_len列也说NULL。

    9. ref (JSON name: ref)

        ref列显示将哪些列或常量与键列中指定的索引进行比较，以从表中选择行。

        如果值是func，则使用的值是某个函数的结果。要查看哪个函数，请使用EXPLAIN后面的SHOW WARNINGS查看扩展的EXPLAIN输出。函数实际上可能是一个运算符，比如算术运算符。

    10. rows (JSON name: rows)

        rows列表示MySQL认为执行查询必须检查的行数。

        对于InnoDB表，这个数字只是一个估计值，可能并不总是准确的。

    11. filtered (JSON name: filtered)

        筛选后的列表示将由表条件筛选的表行的估计百分比。

    12. Extra (JSON name: none)

        这列包含关于MySQL如何解析查询的附加信息。

2. EXPLAIN Join类型

    EXPLAIN output的type列描述了表是如何连接的。在json格式的输出中，这些值作为access_type属性的值。下面的列表描述了连接类型，按照从最好的类型到最差的顺序排列:

    * system

        该表只有一行(= system表)。这是const连接类型的特殊情况。

    * const

        表最多有一个匹配行，在查询开始时读取。因为只有一行，所以这一行中的列的值可以被其他优化器视为常量。const表非常快，因为只读取一次。

        const用于将主键或惟一索引的所有部分与常数值进行比较。

        ```sql
        SELECT * FROM tbl_name WHERE primary_key=1;

        SELECT * FROM tbl_name
        WHERE primary_key_part1=1 AND primary_key_part2=2;
        ```

    * eq_ref

        从该表中读取一行对应前一个表中的每个行组合。除了system和const类型之外，这是最好的连接类型。当联接使用索引的所有部分并且索引是主键或惟一非空索引时，将使用此索引。

        eq_ref可以用于使用=操作符进行比较的索引列。

        ```sql
        SELECT * FROM ref_table,other_table
        WHERE ref_table.key_column=other_table.column;

        SELECT * FROM ref_table,other_table
        WHERE ref_table.key_column_part1=other_table.column
        AND ref_table.key_column_part2=1;
        ```

    * ref

        从这个表中读取具有匹配索引值的所有行对应对于前一个表中的每个行组合。如果连接只使用键的最左端前缀，或者键不是主键或惟一索引(换句话说，如果连接不能根据键值选择单个行)，则使用ref。如果使用的键只匹配几行，这是一个很好的连接类型。

        ref可以用于使用=或<=>操作符进行比较的索引列。

        ```sql
        SELECT * FROM ref_table WHERE key_column=expr;

        SELECT * FROM ref_table,other_table
        WHERE ref_table.key_column=other_table.column;

        SELECT * FROM ref_table,other_table
        WHERE ref_table.key_column_part1=other_table.column
        AND ref_table.key_column_part2=1;
        ```

    * fulltext

        连接是使用FULLTEXT执行的

    * ref_or_null

        



参考：https://dev.mysql.com/doc/refman/8.0/en/explain-output.html