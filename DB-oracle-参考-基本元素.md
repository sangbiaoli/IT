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

        当datetime值通过隐式转换或不指定格式模型的显式转换转换为文本时，格式模型由全球化会话参数之一定义。根据源数据类型，参数名为NLS_DATE_FORMAT、NLS_TIMESTAMP_FORMAT或NLS_TIMESTAMP_TZ_FORMAT。这些参数的值可以在客户机环境或ALTER SESSION语句中指定。

        当将没有显式格式模型的转换应用于连接到动态SQL语句文本的datetime值时，格式模型对会话参数的依赖可能会对数据库安全性产生负面影响。动态SQL语句是那些在将文本传递到数据库执行之前将文本从片段连接起来的语句。动态SQL通常与内置的PL/SQL包DBMS_SQL关联，或者与PL/SQL语句EXECUTE IMMEDIATE关联，但这些并不是动态构造的SQL文本作为参数传递的惟一地方。例如:

        ```sql
        EXECUTE IMMEDIATE
        'SELECT last_name FROM employees WHERE hire_date > ''' || start_date || '''';
        ```

        start_date具有DATE数据类型。

        在上面的例子中，start_date的值使用会话参数NLS_DATE_FORMAT中指定的格式模型转换为文本。结果被连接到SQL文本中。datetime格式模型可以简单地由双引号括起来的文本组成。因此，任何能够显式地为会话设置全球化参数的用户都可以决定通过上述转换生成什么文本。    
        * 如果SQL语句是由PL/SQL过程执行的，则该过程很容易通过会话参数进行SQL注入。   
        * 如果过程以definer的权限(比会话本身具有更高的权限)运行，用户可以获得对敏感数据的未经授权的访问。

        数值的隐式和显式转换也可能遇到类似的问题，因为转换结果可能取决于会话参数NLS_NUMERIC_CHARACTERS。此参数定义十进制和组分隔符。如果将十进制分隔符定义为引号或双引号，就会出现一些SQL注入的可能性。

3. 文字

    术语literal和constant值是同义词，指的是固定的数据值。例如，“JACK”、“BLUE ISLAND”和“101”都是字符文字;5001是一个数字文字。字符文本用单引号括起来，以便Oracle能够将它们与模式对象名称区分开来。
    本节包括以下主题:
    * 文本文字
    * 数字文字
    * 日期时间文字
    * 时间间隔文字

    许多SQL语句和函数要求您指定字符和数字文字值。还可以将文字指定为表达式和条件的一部分。
    * 使用“text”表示法指定字符表示法
    * 使用“N”表示法指定国家字符表示法
    * 使用整数或数字表示法指定数字表示法
    
    具体取决于文字的上下文。这些符号的语法形式出现在下面的小节中。

    要将datetime或interval数据类型指定为文字，必须考虑数据类型中包含的任何可选精度。“数据类型”的相关部分提供了将datetime和interval数据类型指定为文字的示例。

    1. 文本文字

        在此引用的其他部分的表达式、条件、SQL函数和SQL语句语法中出现字符串时，使用文本文字表示法指定值。
        
        此引用可互换使用术语**文本文本、字符文本和字符串**。文本、字符和字符串文本总是被单引号包围。如果语法使用术语char，则可以指定文本文本或解析为字符数据的另一个表达式——例如，hr.employee的last_name列。当char出现在语法中时，不使用单引号。

        文本文字或字符串的语法如下:

        string::=
        ![](db/db-oracle-reference-base_element-literals_string.gif)

        其中N或N使用国家字符集(NCHAR或NVARCHAR2数据)指定文字。默认情况下，使用这种表示法输入的文本在服务器使用时通过数据库字符集转换为国家字符集。为了避免在将文本文本转换为数据库字符集时可能丢失数据，将环境变量ORA_NCHAR_LITERAL_REPLACE设置为TRUE。这样做可以透明地替换内部的n'，并保留用于SQL处理的文本文本。

        * 在语法的顶部分支:
            1. c是用户字符集中的任何成员。文字中的单引号(')前必须有转义字符。要在文字中表示一个单引号，请输入两个单引号。
            2. ' '是两个单引号，分别以文本的开头和结尾。

        * 在语法的底部分支:
            1. Q或Q表示将使用另一种引用机制。这种机制允许为文本字符串使用多种分隔符。
            2. 最外层的' '是两个单引号，分别位于开始和结束quote_delimiter之前和之后。
            3. c是用户字符集的任何成员。您可以在由c字符组成的文本文本中包含引号(")。您还可以包括quote_delimiter，只要它不是紧跟在单引号后面。
            4. quote_delimiter是除空格、制表符和返回外的任何单字节或多字节字符。quote_delimiter可以是单引号。但是，如果quote_delimiter出现在文本文本本身中，请确保它不会立即后跟单引号。
            5. 如果开始的quote_delimiter是[、{、<或(，那么结束的quote_delimiter必须是相应的]、}、>或)之一。在所有其他情况下，开始和结束quote_delimiter必须是相同的字符。

        文本文本具有CHAR和VARCHAR2数据类型的属性:
            
            在表达式和条件中，Oracle通过使用空白填充的比较语义对文本文本进行比较，将其视为具有数据类型CHAR。

            如果初始化参数MAX_STRING_SIZE = STANDARD，文本文本的最大长度为4000字节。
            如果MAX_STRING_SIZE = EXTENDED，文本文本的最大长度为32767字节。

        下面是一些有效文本文本:

        ```
        'Hello'
        'ORACLE.dbs'
        'Jackie''s raincoat'
        '09-MAR-98'
        N'nchar literal'
        ```

        下面是一些使用替代引用机制的有效文本文本:

        ```
        q'!name LIKE '%DBMS_%%'!'
        q'<'So,' she said, 'It's finished.'>'
        q'{SELECT * FROM employees WHERE last_name = 'Smith';}'
        nq'ï Ÿ1234 ï'
        q'"name like '['"'
        ```
        
    2. 数字文字

        1. 整型文本

            在此引用的其他部分中描述的表达式、条件、SQL函数和SQL语句中出现整数时，必须使用整数符号指定整数。
            integer的语法如下:

            integer::=
            ![](db/db-oracle-reference-base_element-literals_integer.gif)

            digit是0,1,2,3,4,5,6,7,8,9其中之一

            一个整数最多可以存储38位精度。
            以下是一些有效整数:

            ```
            7
            +255
            ```

        2. 数字和浮点型文本

            在此引用的其他部分的表达式、条件、SQL函数和SQL语句中出现数字或n时，必须使用数字或浮点表示法指定值。
            数字的语法如下:

            number::=
            ![](db/db-oracle-reference-base_element-literals_number.gif)
            ![](db/db-oracle-reference-base_element-literals_float.gif)

            * +或-表示正或负的值。如果省略符号，则默认值为正值。
            * 数字是0、1、2、3、4、5、6、7、8或9中的一个。
            * e或E表示数字是用科学符号表示的。E后面的数字指定指数。指数可以从-130到125。
            * f或F表示该数字是一个32位二进制浮点数，类型为BINARY_FLOAT。
            * d或D表示该数字是一个64位二进制浮点数，类型为BINARY_DOUBLE。
            * 如果省略f或F和d或D，则该数字的类型为number。
            * 后缀f (f)和d (d)仅在浮点数文字中受支持，而不支持将转换为数字的字符串。例如，如果Oracle正在等待一个数字，并且它遇到了字符串“9”，那么它将把这个字符串转换为数字9。但是，如果Oracle遇到字符串“9f”，则转换失败并返回一个错误。


            一个数的类型数最多可存储38位精度。如果文字需要比NUMBER、BINARY_FLOAT或BINARY_DOUBLE提供的精度更高的精度，那么Oracle将截断该值。如果文字的范围超出NUMBER、BINARY_FLOAT或BINARY_DOUBLE所支持的范围，那么Oracle将引发一个错误。

            数字文字是SQL语法元素，对NLS设置不敏感。数字文字中的十进制分隔符总是句点(.)。但是，如果在需要数值的地方指定了文本文本，那么文本文本将以对nls敏感的方式隐式转换为数字。文本文本中包含的十进制分隔符必须是使用初始化参数NLS_NUMERIC_CHARACTERS建立的分隔符。Oracle建议在SQL脚本中使用数字文字，使它们独立于NLS环境工作。
            
            下面的示例演示了小数分隔符在数值和文本文本中的行为。这些例子假设您已经使用以下语句为当前会话建立了逗号(，)作为NLS十进制分隔符:
            
            ```sql
            ALTER SESSION SET NLS_NUMERIC_CHARACTERS=',.';
            ```

            前面的语句还将句点(.)设置为NLS组分隔符，但这与这些示例无关。
            
            本例在数字文字1.23中使用所需的十进制分隔符(.)，在文本文字“2,34”中使用已建立的NLS十进制分隔符(，)。文本文本被转换为数值2.34，输出使用逗号表示小数分隔符。

            ```sql
            SELECT 2 * 1.23, 3 * '2,34' FROM DUAL;

                2*1.23   3*'2,34'
            ---------- ----------
                2,46       7,02
            ```

            下一个例子显示，逗号不被视为数字文字的一部分。相反，逗号在一个由两个数字表达式组成的列表中被视为分隔符:2*1和23。

            ```sql
            SELECT 2 * 1,23 FROM DUAL;

                2*1         23
            ---------- ----------
                  2         23
            ```

            下一个示例显示，文本文本中的十进制分隔符必须匹配NLS十进制分隔符，以便隐式文本到数字的转换成功。由于十进制分隔符(.)与已建立的NLS十进制分隔符(，)不匹配，下面的语句失败:

            ```sql
            SELECT 3 * '2.34' FROM DUAL;
                        *
            ERROR at line 1:
            ORA-01722: invalid number 
            ```

            下面是一些有效的NUMBER文本
            
            ```
            25
            +6.34
            0.5
            25e-03
            -1
            ```

            以下是一些有效的浮点数文字:

            ```
            25f
            +6.34F
            0.5d
            -1D
            ```
    
    3. 日期时间文字

        Oracle数据库支持四种datetime数据类型:DATE, TIMESTAMP, TIMESTAMP WITH TIME ZONE, 和TIMESTAMP WITH LOCAL TIME ZONE.

        1. Date文字

            可以将DATE值指定为字符串文字，也可以使用TO_DATE函数将字符或数值转换为日期值。DATE文字是Oracle数据库接受TO_DATE表达式代替字符串文字的唯一情况。
            要将DATE值指定为文字值，必须使用公历。您可以指定一个ANSI文字，如下例所示:

            ```sql
            DATE '1998-12-25'
            ```

            ANSI日期文字不包含时间部分，必须以“YYYY-MM-DD”格式指定。或者，您可以指定Oracle日期值，如下面的示例所示:

            ```sql
            TO_DATE('98-DEC-25 17:30','YY-MON-DD HH24:MI')
            ```

            Oracle DATE值的默认日期格式由初始化参数NLS_DATE_FORMAT指定。这个示例日期格式包括一个表示月份的日期的两位数、月份名称的缩写、一年的最后两位数字和一个24小时时间指定。
            
            当在日期表达式中使用默认日期格式的字符值时，Oracle会自动将其转换为日期值。

            如果指定一个没有时间组件的日期值，则默认时间为午夜(分别为24小时和12小时的时钟时间，为00:00:00或12:00:00)。如果指定的日期值没有日期，则默认日期是当前月份的第一天。

            Oracle日期列总是同时包含日期和时间字段。因此，如果查询DATE列，则必须在查询中指定时间字段，或者确保将日期列中的时间字段设置为midnight。否则，Oracle可能不会返回您期望的查询结果。可以使用TRUNC date函数将时间字段设置为midnight，也可以在查询中包含大于或小于条件，而不是相等或不等条件。
            
            下面是一些例子，假设一个表my_table有一个数字列row_num和一个DATE列datecol:

            
            ```sql
            INSERT INTO my_table VALUES (1, SYSDATE);
            INSERT INTO my_table VALUES (2, TRUNC(SYSDATE));

            SELECT *
            FROM my_table;

            ROW_NUM DATECOL
            ---------- ---------
                    1 03-OCT-02
                    2 03-OCT-02

            SELECT *
            FROM my_table
            WHERE datecol > TO_DATE('02-OCT-02', 'DD-MON-YY');

            ROW_NUM DATECOL
            ---------- ---------
                    1 03-OCT-02
                    2 03-OCT-02

            SELECT *
            FROM my_table
            WHERE datecol = TO_DATE('03-OCT-02','DD-MON-YY');

            ROW_NUM DATECOL
            ---------- ---------
                    2 03-OCT-02
            ```

             如果您知道DATE列的时间字段设置为midnight，那么您可以查询DATE列，如下面的示例所示，或者使用DATE文字:

            ```sql
            SELECT *
            FROM my_table
            WHERE datecol = DATE '2002-10-03';


            ROW_NUM DATECOL
            ---------- ---------
                    2 03-OCT-02
            ```

            但是，如果DATE列包含midnight以外的值，则必须过滤查询中的时间字段以获得正确的结果。例如:

            ```sql
            SELECT *
            FROM my_table
            WHERE TRUNC(datecol) = DATE '2002-10-03';


            ROW_NUM DATECOL
            ---------- ---------
                    1 03-OCT-02
                    2 03-OCT-02
            ```

            Oracle将TRUNC函数应用于查询中的每一行，因此如果确保数据中时间字段的午夜值，性能会更好。要确保时间字段设置为午夜，请在插入和更新期间使用以下方法之一:

            * Use the TO_DATE function to mask out the time fields:
            
            ```sql
            INSERT INTO my_table
            VALUES (3, TO_DATE('3-OCT-2002','DD-MON-YYYY'));
            ```

            * Use the DATE literal:

            ```sql
            INSERT INTO my_table
            VALUES (4, '03-OCT-02');
            ```

            * Use the TRUNC function:

            ```sql
            INSERT INTO my_table
            VALUES (5, TRUNC(SYSDATE));
            ```

            date函数SYSDATE返回当前系统日期和时间。函数CURRENT_DATE返回当前会话日期。有关SYSDATE、TO_* datetime函数和默认日期格式的信息。

        2. TIMESTAMP文字

            TIMESTAMP数据类型存储年、月、日、小时、分钟、秒和小数秒值。当您将时间戳指定为文字时，fractional_seconds_precision值可以是任意数字，最多可以是9，如下所示:

            ```sql
            TIMESTAMP '1997-01-31 09:26:50.124'

            select * from my_table where datecol >= TIMESTAMP '1997-01-31 09:26:50.124'
            ```

        3. TIMESTAMP WITH TIME ZONE文字

            TIMESTAMP WITH TIME ZONE是TIMESTAMP的变体，它包含时区区域名称或时区偏移量。当您将TIMESTAMP WITH TIME ZONE指定为文字时，fractional_seconds_precision值可以是任意数字，最多可以是9。例如:

            ```sql
            TIMESTAMP '1997-01-31 09:26:56.66 +02:00'
            ```

            如果两个TIMESTAMP WITH TIME ZONE在UTC中表示相同的时刻，则认为它们是相同的，而不考虑存储在数据中的TIME ZONE偏移量。例如,

            ```sql
            TIMESTAMP '1999-04-15 8:00:00 -8:00'
            ```

            与下面的一样

            ```sql
            TIMESTAMP '1999-04-15 11:00:00 -5:00'
            ```

            太平洋标准时间的上午8点是东部标准时间的上午11点。

        4. TIMESTAMP WITH LOCAL TIME ZONE文字

            TIMESTAMP WITH LOCAL TIME ZONE数据类型与IMESTAMP WITH TIME ZONE不同，因为存储在数据库中的数据被规范化为数据库时区。时区偏移量不作为列数据的一部分存储。对于本地时区，没有用于TIMESTAMP WITH LOCAL TIME ZONE的文字。相反，您可以使用任何其他有效的datetime文字来表示这种数据类型的值。下表显示了一些可以用来将值插入带有TIMESTAMP WITH LOCAL TIME ZONE的格式，以及查询返回的相应值。

            插入语句的值|查询返回的值
            --|--
            '19-FEB-2004'|19-FEB-2004.00.00.000000 AM
            SYSTIMESTAMP|19-FEB-04 02.54.36.497659 PM
            TO_TIMESTAMP('19-FEB-2004', 'DD-MON-YYYY')|19-FEB-04 12.00.00.000000 AM
            SYSDATE|19-FEB-04 02.55.29.000000 PM
            TO_DATE('19-FEB-2004', 'DD-MON-YYYY')|19-FEB-04 12.00.00.000000 AM
            TIMESTAMP'2004-02-19 8:00:00 US/Pacific'|19-FEB-04 08.00.00.000000 AM

    4. 时间间隔文字


4. 格式的模型
5. Null值
6. 数据库对象
7. 数据库对象名称和别名
8. SQL语句中模式对象和部分的语法

原文：https://docs.oracle.com/database/121/SQLRF/sql_elements001.htm#SQLRF50998