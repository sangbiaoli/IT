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
        2|FLOAT [(p)]|具有精度p的NUMBER数据类型的子类型。浮点值在内部表示为NUMBER。精度p可以从1到126位二进制数字。浮点值需要1到22个字节。换算关系：binary precision＝int(b*0.30103)，保留precision位精度。相当于转成科学记数法后，a=b*10^n，b最多保留precision位。
        8|LONG|可变长度的字符数据，最大可达2g，或2^31 -1字节。提供向后兼容性。尽可能不要创建这种类型的列，可以使用LOB列(CLOB, NCLOB, BLOB)
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

        从Oracle数据库12c开始，可以为VARCHAR2、NVARCHAR2和原始数据类型指定最大大小32767字节。通过设置初始化参数MAX_STRING_SIZE，可以控制数据库是否支持这个新的最大大小:

        * 如果MAX_STRING_SIZE = STANDARD，则适用Oracle数据库12c之前版本的大小限制:VARCHAR2和NVARCHAR2数据类型为4000字节，原始数据类型为2000字节。这是默认值。
        * 如果MAX_STRING_SIZE = EXTENDED，那么VARCHAR2、NVARCHAR2和原始数据类型的大小限制为32767字节。

        ```
        设置MAX_STRING_SIZE = EXTENDED可能会更新数据库对象，并可能使它们失效。有关此参数的含义以及如何设置和启用此新功能的完整信息，请参阅Oracle数据库参考资料。
        ```

    3. Rowid数据类型

        数据库中的每一行都有一个地址。下面的部分描述了Oracle数据库中的两种行地址形式。

        1. ROWID数据类型

            本机用于Oracle数据库的堆组织表中的行具有行地址，称之为rowid。

            您可以通过查询伪列rowid来检查rowid行地址。这个伪列的值是表示每一行地址的字符串。这些字符串的数据类型为ROWID。您还可以创建包含具有ROWID数据类型的实际列的表和集群。Oracle数据库不保证这些列的值是有效的rowid。

            Rowids包含以下信息:
            * 包含该行的数据文件的数据块。这个字符串的长度取决于您的操作系统。
            * 数据块中的行。
            * 包含该行的数据库文件。第一个数据文件是数字1。这个字符串的长度取决于您的操作系统。
            * 数据对象号，它是分配给每个数据库段的标识号。您可以从数据字典视图USER_OBJECTS、DBA_OBJECTS和ALL_OBJECTS中检索数据对象号。共享相同段的对象(例如，相同集群中的集群表)具有相同的对象号。
            
            行id存储为64个基本值，这些值可以包含字符A-Z、A-Z、0-9以及加号(+)和正斜杠(/)。无法直接使用rowid。您可以使用提供的包DBMS_ROWID来解释rowid内容。包函数提取并提供关于上面列出的四个rowid元素的信息。

        2. UROWID数据类型

            有些表的行具有非物理或永久地址，或者不是由Oracle数据库生成的地址。例如，索引组织的表的行地址存储在可以移动的索引叶中。外部表的行id(例如通过网关访问的DB2表)不是标准的Oracle行id。

            Oracle使用universal rowids (urowids)存储索引组织表和外表的地址。索引组织的表具有逻辑urowid，而外部表具有外部urowid。这两种类型的urowid都存储在ROWID伪列中(堆组织表的物理行也是如此)。

            Oracle基于表的主键创建逻辑行。只要主键不变，逻辑行就不会改变。索引组织表的ROWID伪列具有UROWID数据类型。您可以像访问堆组织表的ROWID伪列一样访问这个伪列(使用SELECT…ROWID声明)。如果希望存储索引组织表的行id，则可以为表定义一个类型为UROWID的列，并将ROWID伪列的值检索到该列中。

    4. ANSI，DB2及SQL/DS数据类型

        创建表和集群的SQL语句还可以使用ANSI数据类型和来自IBM产品SQL/DS和DB2的数据类型。

        Oracle识别与Oracle数据库数据类型名称不同的ANSI或IBM数据类型名称。

        它将数据类型转换为等效的Oracle数据类型，将Oracle数据类型记录为列数据类型的名称，并根据下表中显示的转换将列数据存储在Oracle数据类型中。

        1. ANSI数据类型转化为Oracle数据类型

            ANSI SQL数据类型|Oracle数据类型
            --|--
            CHARACTER(n)</br>CHAR(n)	|	CHAR(n)
            CHARACTER VARYING(n)</br>CHAR VARYING(n)	|	VARCHAR2(n)
            NATIONAL CHARACTER(n)</br>NATIONAL CHAR(n)</br>NCHAR(n)	|	NCHAR(n)
            NATIONAL CHARACTER VARYING(n)</br>NATIONAL CHAR VARYING(n)</br>NCHAR VARYING(n)	|	NVARCHAR2(n)
            NUMERIC[(p,s)]</br>	DECIMAL[(p,s)] (Note 1)	| NUMBER(p,s)
            INTEGER</br>INT</br>SMALLINT	|	NUMBER(p,0)
            FLOAT (Note 2)</br>DOUBLE PRECISION (Note 3)</br>REAL (Note 4)	|	FLOAT(126)</br>FLOAT(126)</br>FLOAT(63)

            注意点：
            1. NUMERIC和DECIMAL数据类型只能指定定点数字。对于这些数据类型，scale默认为0。
            2. FLOAT数据类型是具有二进制精度b的浮点数。该数据类型的默认精度为126二进制或38十进制。
            3. DOUBLE PRECISION数据类型是一个浮点数，具有二进制精度126。
            4. REAL数据类型是一个浮点数，二进制精度为63，或者十进制18。

        2. SQL/DS and DB2数据类型转化为Oracle数据类型

            SQL/DS and DB2数据类型|Oracle数据类型
            --|--
            CHARACTER(n)|CHAR(n)
            VARCHAR(n)|VARCHAR(n)
            LONG VARCHAR|LONG
            DECIMAL(p,s) (Note 1)|NUMBER(p,s)
            INTEGER</br>SMALLINT|NUMBER(p,0)
            FLOAT (Note 2)|NUMBER

            注意点：
            1. DECIMAL数据类型只能指定定点数字。对于这种数据类型，s默认为0。
            2. FLOAT数据类型是具有二进制精度b的浮点数。该数据类型的默认精度为126二进制或38十进制。

    5. 用户自定义类型
    6. Oracle提供的类型
    7. 任何类型
    8. XML类型
    9. 空间类型
    10. 媒体类型

2. 数据类型比较规则

    1. 数字值

        较大的值被认为大于较小的值。所有负数都小于零，所有正数都小于零。因此-1小于100;-100小于-1。
        
        浮点值NaN(不是数字)大于任何其他数字值，并且等于它本身。

    2. 日期值

        较晚的日期被认为比较早的日期更大。例如，相当于“29-MAR-2005”的日期小于“05-JAN-2006”的日期，而“05-JAN-2006 1:35pm”大于“05-JAN-2005 10:09am”的日期。

    3. 字符值

        字符值的比较基于两个度量:

        1. 二进制或语言排序

            在二进制比较中，默认为Oracle根据数据库字符集中字符的数字代码的串联值来比较字符串。也就是说，如果一个字符的数值大于另一个字符在字符集中的数值，则该字符大于另一个字符。Oracle认为空白比任何字符都小，在大多字符集中都是正确的

            常用的字符集
            * 7位ASCII(美国信息交换标准码)
            * EBCDIC码(扩展二进制编码的十进制交换码)
            * ISO 8859/1(国际标准化组织)
            * JEUC日本扩展UNIX


            如果数字代码的二进制序列与正在比较的字符的语言序列不匹配，则语言比较非常有用。如果NLS_SORT参数的设置不是二进制的，并且NLS_COMP参数被设置为language，则使用语言比较。在语言排序中，所有SQL排序和比较都基于NLS_SORT指定的语言规则。

        2. 空白填充或非填充比较语义

            使用空白填充的语义，如果两个值的长度不同，那么Oracle首先在较短的值的末尾添加空白，以便它们的长度相等。然后，Oracle将逐个比较字符直到出现不同字符。在第一个不同位置具有较大字符的值被认为是较大的。如果两个值没有不同的字符，则认为它们是相等的。这条规则意味着，如果两个值只在后面的空格数上不同，那么它们就是相等的。只有当比较中的两个值都是数据类型CHAR、NCHAR、文本文本或用户函数返回的值的表达式时，Oracle才使用空白填充的比较语义。

            使用非填充语义，Oracle按字符比较两个值，直到第一个不同的字符。在该位置具有较大字符的值被认为是较大的。如果两个长度不同的值在较短的值结束之前是相同的，则认为较长的值更大。如果两个长度相等的值没有不同的字符，则认为这些值是相等的。当比较中的一个或两个值具有VARCHAR2或NVARCHAR2数据类型时，Oracle使用非填充比较语义。

            使用不同的比较语义比较两个字符值的结果可能不同。下表显示了使用每种比较语义比较五对字符值的结果。通常，空白填充和非填充比较的结果是相同的。表中的最后一个比较说明了填充空白和非填充比较语义之间的差异。

            用空格填充|非填充
            --|--
            'ac' > 'ab'	| 'ac' > 'ab'
            'ab' > 'a  '| 'ab' > 'a   '
            'ab' > 'a'	| 'ab' > 'a'
            'ab' = 'ab'	| 'ab' = 'ab'
            'a ' = 'a'	| 'a ' > 'a'

    4. 对象值

        使用以下两个比较函数之一比较对象值:MAP和ORDER。这两个函数都比较对象类型实例，但是它们之间有很大的不同。必须将这些函数指定为将与其他对象类型进行比较的任何对象类型的一部分。

    5. varray和嵌套表(跳过)
    6. 数据类型优先级

        Oracle使用数据类型优先级来确定隐式数据类型转换，这将在下一节中讨论。Oracle数据类型的优先级如下:
        * Datetime和interval数据类型
        * BINARY_DOUBLE
        * BINARY_FLOAT
        * NUMBER
        * 字符数据类型
        * 所有其他内置数据类型

    7. 数据转换

        通常，表达式不能包含不同数据类型的值。例如，一个表达式不能将5乘以10然后加上“JAMES”。然而，Oracle支持从一种数据类型到另一种数据类型的隐式和显式值转换。

        隐式和显式数据转换

        Oracle建议您指定显式转换，而不是依赖于隐式或自动转换，原因如下:
        * 当使用显式数据类型转换函数时，SQL语句更容易理解。
        * 隐式数据类型转换可能对性能产生负面影响，特别是如果将列值的数据类型转换为常量的数据类型，而不是相反。
        * 隐式转换依赖于它发生的上下文，在每种情况下可能不会以相同的方式工作。例如，根据NLS_DATE_FORMAT参数的值，从datetime值隐式转换为VARCHAR2值可能会返回一个意外的年份。
        * 隐式转换的算法在不同的软件版本和Oracle产品之间可能会发生变化。显式转换的行为更易于预测。
        * 如果隐式数据类型转换发生在索引表达式中，那么Oracle数据库可能不会使用索引，因为索引是为预转换数据类型定义的。这可能会对性能产生负面影响。


        1. 隐式转换

            当这种转换有意义时，Oracle数据库会自动将值从一种数据类型转换为另一种数据类型。Oracle隐式转换的矩阵（省略）

            不能直接将LONG转换为INTERVAL，但是可以使用TO_CHAR(INTERVAL)将LONG转换为VARCHAR2，然后将得到的VARCHAR2值转换为INTERVAL。

            1. 隐式数据类型转换规则

                * 在INSERT 和UPDATE 操作期间，Oracle将值转换为受影响列的数据类型。
                * 在SELECT FROM操作期间，Oracle将列中的数据转换为目标变量的类型。
                * 在操作数值时，Oracle通常调整精度和比例，以允许最大容量。在这种情况下，这些操作产生的数值数据类型可能与底层表中找到的数值数据类型不同。
                * 当比较字符值和数值时，Oracle将字符数据转换为数值。
                * 字符值或数字值与浮点数值之间的转换可能不精确，因为字符类型和数字使用十进制精度表示数值，而浮点数使用二进制精度。
                * 当将CLOB值转换为字符数据类型(如VARCHAR2)或将BLOB转换为原始数据时，如果要转换的数据大于目标数据类型，那么数据库将返回一个错误。
                * 在从时间戳值转换为日期值期间，时间戳值的小数秒部分将被截断。这种行为与Oracle数据库的早期版本不同，在早期版本中，时间戳值的小数秒部分是四舍五入的。
                * 从BINARY_FLOAT到BINARY_DOUBLE的转换是精确的。
                * 如果BINARY_DOUBLE值使用了比BINARY_FLOAT所支持的更精确的位，那么从BINARY_DOUBLE到BINARY_FLOAT的转换就是不精确的。
                * 在比较字符值和日期值时，Oracle将字符数据转换为日期。
                * 当使用SQL函数或操作符时，如果参数不是它所接受的数据类型，Oracle会将该参数转换为所接受的数据类型。
                * 在进行赋值时，Oracle将等号(=)右侧的值转换为赋值对象左侧的数据类型。
                * 在连接操作期间，Oracle将非字符数据类型转换为CHAR或NCHAR。
                * 在对字符和非字符数据类型进行算术运算和比较时，Oracle会根据需要将任何字符数据类型转换为数字、日期或rowid。在CHAR/VARCHAR2和NCHAR/NVARCHAR2之间的算术运算中，Oracle转换为一个数字。
                * 大多数SQL字符函数都支持接受CLOB作为参数，Oracle在CLOB和字符类型之间执行隐式转换。因此，尚未为clob启用的函数可以通过隐式转换接受clob。在这种情况下，Oracle在调用函数之前将clob转换为CHAR或VARCHAR2。如果CLOB大于4000字节，那么Oracle只将前4000字节转换为CHAR。
                * 在将RAW 或LONG RAW转换为字符数据或从字符数据转换为字符数据时，二进制数据以十六进制形式表示，十六进制字符表示每四位RAW数据。有关更多信息，请参考“原始和长原始数据类型”。
                * 比较CHAR和VARCHAR2以及NCHAR和NVARCHAR2类型可能需要不同的字符集。在这种情况下，默认的转换方向是从数据库字符集到国家字符集。表2-11显示了不同字符类型之间隐式转换的方向（省略）。

            2. 隐式数据转换示例

                * 文本文字的例子

                    文本文字“10”具有数据类型CHAR。Oracle隐式地将它转换为数字数据类型，如果它出现在数字表达式中，如下面的语句所示:
                    
                    ```sql
                    SELECT salary + '10'
                    FROM employees;
                    ```

                * 字符和数值的例子

                    当条件比较字符值和数字值时，Oracle隐式地将字符值转换为数字值，而不是将数字值转换为字符值。在下面的语句中，Oracle隐式地将“200”转换为200:
                    
                    ```sql
                    SELECT last_name
                    FROM employees
                    WHERE employee_id = '200';
                    ```
                
                * 日期例子

                    在下面的语句中，Oracle使用默认日期格式“DD-MON-YY”隐式地将“24-JUN-06”转换为日期值:

                    ```sql
                    SELECT last_name
                    FROM employees 
                    WHERE hire_date = '24-JUN-06';
                    ```
        
        2. 显式数据转换

            可以使用SQL转换函数显式指定数据类型转换。表2-12（省略）显示了显式地将值从一种数据类型转换为另一种数据类型的SQL函数。
            
            在Oracle可以执行隐式数据类型转换的情况下，不能指定LONG和LONG RAW。例如，LONG和LONG RAW不能出现在带有函数或操作符的表达式中。有关LONG数据类型和LONG RAW数据类型的限制的信息，请参阅“长数据类型”。

    8. 数据转换中的安全考虑

3. 文字
4. 格式的模型
5. Null值
6. 数据库对象
7. 数据库对象名称和别名
8. SQL语句中模式对象和部分的语法

原文：https://docs.oracle.com/database/121/SQLRF/sql_elements001.htm#SQLRF50998