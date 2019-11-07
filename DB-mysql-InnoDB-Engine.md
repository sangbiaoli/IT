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