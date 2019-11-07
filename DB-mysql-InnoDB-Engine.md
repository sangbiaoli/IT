#### Innodb

1. 参数
    * innodb_file_per_table
        * 启动后，每张表内的数据可以单独放到一个表空间。
        * 存放数据、索引和插入缓冲Bitmao页
        * 其他数据存放在原来的共享表空间
    * innodb_page_size
        * 可以设置为4K，8K，但是区的大小总是1M

2. 实践点

    1. 第一个非空唯一索引作为主键
    2. _rowid只能用于查看单个列为主键
    3. 启动参数innodb_file_per_table后，创建的表默认大小是96KB，因为每个段开始时，先用32个页大小的碎片页，使用完了才是64个连续页的申请。
    4. 行溢出问题
        * Mysql定义varchar类型可以存放65535，实际上只能存放65532，且是所有varchar字段总和不能超过65532。
        * 一般情况下，InnodDB存储在B-tree node的页中，但发生溢出时，则存放在Uncompressed BLOG页中。
        * 当插入的数据>=8098时，不会把行数据存放到BLOB页中。
    5. 约束
        * primary key
        * unique key
        * foreign key
        * default
        * not null
    6. 约束和索引的区别
        * 约束是一个逻辑概念，用来保证数据的完整性
        * 索引是一个数据结构，既有逻辑上的概念，在数据库中还代表着物理存储的方式。
    7. sql_mode
        * sql_mode为STRICT_TRANS_TABLES，则数据不满足约束不能插入和更新
        * sql_mode为NO_ENGINE_SUBSTITUTION时
            * 往not null字段插入null吗，会自动设置为'0'，
            * 往date字段插入'2019-02-30'，会自动设置'0000-00-00'
    8. 外键约束
        * MyISAM存储引擎不支持外键
        * InnoDB完整支持外键

        一般来说，称被引用的表为父表，引用的表为子表。

    9. 分区类型
        * RANGE分区：行数据基于属于一个给定连续区间的列值被放入分区
        * LIST分区：和RANGE分区类似，只是LIST分区面向的是离散的值
        * HASH分区：根据用户自定义的表达式的返回值来进行分区，返回值不能为负数
        * KEY分区：根据MySQL数据库提供的哈希函数来进行分区。