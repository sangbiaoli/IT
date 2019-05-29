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

        Code|Data Type|Description
        --|--|--
        1|VARCHAR2(size [BYTE \| CHAR])|具有最大长度大小字节或字符的可变长度字符串。</br>如果MAX_STRING_SIZE = EXTENDED，则为32767字节或字符</br>如果MAX_STRING_SIZE =STANDARD，则为4000字节或字符
        1|NVARCHAR2(size)|具有最大长度大小字符的可变长度Unicode字符串。对于AL16UTF16编码，字节数可以达到2倍大小，对于UTF8编码，字节数可以达到3倍大小。</br>如果MAX_STRING_SIZE = EXTENDED，则为32767字节或字符</br>如果MAX_STRING_SIZE =STANDARD，则为4000字节或字符
        2|NUMBER [ (p [, s]) ]|具有精度p和刻度s的数。精度p的范围从1到38，s的刻度从-84到127不等。精度和比例尺都是十进制的。数字值需要1到22个字节。</br>s为正数时，四舍五入保留s位小数；s为负数时，则从小数点左侧开始第s位进行四舍五入。通过s刻度计算后得到一个值，如果这个值的有效位超过了p位，则报错。</br>比如number(2,1) 有效位最大为2，小数点后最多保留1位，存11.1 得出错 有效位为3，大于2；</br>比如number(2,-3) 有效位最大为2+3=5，没有小数位，存99999 得出错，因为四舍五入后变为100000，有效位为6，大于5。
        2|FLOAT [(p)]|具有精度p的NUMBER数据类型的子类型。浮点值在内部表示为NUMBER。精度p可以从1到126位二进制数字。浮点值需要1到22个字节。
        8|LONG|可变长度的字符数据，最大可达2g，或2^31 -1字节。提供向后兼容性。
        12|DATE|有效日期从公元前4712年1月1日到公元9999年12月31日。默认格式由NLS_DATE_FORMAT参数显式地确定，或由NLS_TERRITORY参数隐式地确定。大小固定为7字节。此数据类型包含datetime字段YEAR、MONTH、DAY、HOUR、MINUTE和SECOND。它没有小数秒或时区。
        100|BINARY_FLOAT|32位浮点数。这种数据类型需要4个字节。
        101|BINARY_DOUBLE|64位浮点数。这种数据类型需要8个字节。
        180|TIMESTAMP [(fractional_seconds_precision)]|date的Year、month和day值，以及hour、minute和second值，其中fractional_seconds_precision是SECOND datetime字段的小数部分中的数字。fractional_seconds_precision的接受值为0到9。默认值是6。默认格式由NLS_TIMESTAMP_FORMAT参数显式地确定，或由NLS_TERRITORY参数隐式地确定。大小为7或11字节，这取决于精度。此数据类型包含datetime字段YEAR、MONTH、DAY、HOUR、MINUTE和SECOND。它包含小数秒，但没有时区。
        181|TIMESTAMP [(fractional_seconds_precision)] WITH TIME ZONE|跟TIMESTAMP一样</br>大小固定为13字节。这个数据类型包含datetime字段YEAR、MONTH、DAY、HOUR、MINUTE、SECOND、TIMEZONE_HOUR和TIMEZONE_MINUTE。它有小数秒和显式时区。
        231|TIMESTAMP [(fractional_seconds_precision)] WITH LOCAL TIME ZONE|跟TIMESTAMP WITH TIME ZONE一样</br>额外注意两点：当数据存储在数据库中时，它被规范化为数据库时区；检索数据时，用户将在会话时区中看到数据。
        182|INTERVAL YEAR [(year_precision)] TO MONTH|存储以年和月为单位的时间段，其中year_precision是YEAR datetime字段中的数字。接受的值是0到9。默认值是2。大小固定为5字节。
        183|INTERVAL DAY [(day_precision)] TO SECOND [(fractional_seconds_precision)]|以天、小时、分钟和秒为单位存储一段时间。</br>day_precision是DAY datetime字段中的最大数字。接受的值是0到9。默认值是2。</br>fractional_seconds_precision是SECOND字段的小数部分的位数。接受的值是0到9。默认值是6。大小固定为11字节。
        23|RAW(size)|长度大小为字节的原始二进制数据。必须为原始值指定大小。最大尺寸是:</br>如果MAX_STRING_SIZE = EXTENDED，则为32767字节</br>如果MAX_STRING_SIZE =STANDARD，则为2000字节
        24|LONG RAW|可变长度的原始二进制数据，最大可达2g。
        69|ROWID|以64为基数的字符串表示表中某一行的惟一地址。这种数据类型主要用于ROWID伪列返回的值。
        208|UROWID [(size)]|基64字符串表示索引组织表的一行的逻辑地址。可选的大小是UROWID类型的列的大小。最大大小和默认值是4000字节。
        96|CHAR [(size [BYTE | CHAR])]|长度大小为字节或字符的固定长度字符数据。最大大小为2000字节或字符。默认值和最小大小为1字节。BYTE 和CHAR具有与VARCHAR2相同的语义。
        96|NCHAR[(size)]|长度大小字符的固定长度字符数据。对于AL16UTF16编码，字节数可以达到2倍大小，对于UTF8编码，字节数可以达到3倍大小。最大大小由国家字符集定义确定，上限为2000字节。默认值和最小大小为1个字符。
        112|CLOB|包含单字节或多字节字符的字符大对象。支持固定宽度和可变宽度字符集，都使用数据库字符集。最大大小为(4 gb - 1) *(数据库块大小)。
        112|NCLOB|包含Unicode字符的字符大对象。支持固定宽度和可变宽度字符集，都使用数据库国家字符集。最大大小为(4 gb - 1) *(数据库块大小)。存储国家字符集数据。
        113|BLOB|二进制大对象。最大大小为(4 gb - 1) *(数据库块大小)。
        114|BFILE|包含存储在数据库外的大型二进制文件的定位器。启用对驻留在数据库服务器上的外部lob的字节流I/O访问。最大容量为4g。

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