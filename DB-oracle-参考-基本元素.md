## 基本元素

* 数据类型
* 数据类型比较规则
* 文字
* 格式的模型
* Null值
* 数据库对象
* 数据库对象名称和别名
* SQL语句中模式对象和部分的语法


1. 数据类型

    Oracle数据库操作的每个值都有一个数据类型。值的数据类型将一组固定的属性与该值关联起来。这些属性导致Oracle以不同的方式对待一种数据类型的值。例如，可以添加NUMBER数据类型的值，但不能添加RAW数据类型的值。

    Oracle数据库提供了一系列内置的数据类型以及用户定义的若干类别，均可用作数据类型，Oracle数据类型的语法出现在下面的图表中。

    数据类型可以是标量类型，也可以是非标量类型。
    * 标量类型包含原子值，大对象(LOB)是标量数据类型的一种特殊形式，表示二进制或字符数据的大标量值。
    * 非标量(有时称为“集合”)包含一组值。

    Oracle预编译器可以识别嵌入式SQL程序中的其他数据类型。这些数据类型称为外部数据类型，与主机变量相关联。不要将内置数据类型、用户定义的类型与外部数据类型混淆。

    **datatypes::=**

    ![](db/db-oracle-reference-base_element-datatypes.gif)


    Oracle内置的数据类型如下

    * **Oracle_built_in_datatypes::=**

        ![](db/db-oracle-reference-base_element-datatypes_oracle_built_in.gif)

        1. **character_datatypes::=**
        
            ![](db/db-oracle-reference-base_element-datatypes_character.gif)

        2. **number_datatypes::=**

            ![](db/db-oracle-reference-base_element-datatypes_number.gif)

        3. **long_and_raw_datatypes::=**

            ![](db/db-oracle-reference-base_element-datatypes_long_and_raw.gif)

        4. **datetime_datatypes::=**

            ![](db/db-oracle-reference-base_element-datatypes_datetime.gif)

        5. **large_object_datatypes::=**

            ![](db/db-oracle-reference-base_element-datatypes_large_object.gif)

        6. **rowid_datatypes::=**

            ![](db/db-oracle-reference-base_element-datatypes_rowid.gif)

    * **ANSI_supported_datatypes::=**

        ![](db/db-oracle-reference-base_element-ansi_supported_datatypes.gif)

    * **Oracle_supplied_types::=**

        ![](db/db-oracle-reference-base_element-oracle_supplied_types.gif)

    * **any_types::=**

        ![](db/db-oracle-reference-base_element-any_types.gif)

    * **XML_types::=**

        ![](db/db-oracle-reference-base_element-xml_types.gif)
        

    * **spatial_types::=**

        ![](db/db-oracle-reference-base_element-spatial_types.gif)

    * **media_types::=**

        ![](db/db-oracle-reference-base_element-media_types.gif)

    * **still_image_object_types::=**

        ![](db/db-oracle-reference-base_element-still_image_object_types.gif)

    1. Oracle内置的数据类型
    2. 扩展的数据类型
    3. Rowid数据类型
    4. ANSI，DB2及SQL/DS数据类型
    5. 用户自定义类型
    6. Oracle提供的类型
    7. 任何类型
    8. XML类型
    9. 空间类型
    10. 媒体类型

2. 数据类型比较规则
3. 文字
4. 格式的模型
5. Null值
6. 数据库对象
7. 数据库对象名称和别名
8. SQL语句中模式对象和部分的语法