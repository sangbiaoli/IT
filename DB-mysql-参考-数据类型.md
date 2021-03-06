## 数据类型

1. 数字类型概观

    * BIT[(M)]

        一个比特值的类型。M表示每个值的比特数，从1到64。如果省略M，默认值是1。

    * TINYINT[(M)] [UNSIGNED] [ZEROFILL]

        一个很小的整数。有符号的范围是-128到127。无符号的范围是0到255。相当于一个字节(8bit)

    * BOOL, BOOLEAN

        这些类型是TINYINT(1)的同义词。0值为false。非零值为true。

        真值和假值只是分别为1和0的别名，比如
        * 0=false，结果为true
        * 1=true，结果为true
        * 2=true，结果为false

    * SMALLINT[(M)] [UNSIGNED] [ZEROFILL]

        一个小整数。符号范围是-32768到32767。无符号的范围是0到65535。
        相当于两个字节(16bit)

    * MEDIUMINT[(M)] [UNSIGNED] [ZEROFILL]

        一个中等大小的整数。符号范围是-8388608到8388607。无符号的范围是0到16777215。相当于三个字节(24bit)

    * INT[(M)] [UNSIGNED] [ZEROFILL]

        把普通的整数。符号范围是-2147483648到2147483647。无符号的范围是0到4294967295。相当于四个字节(32bit)

    * INTEGER[(M)] [UNSIGNED] [ZEROFILL]

        这种类型是INT的同义词。

    * BIGINT[(M)] [UNSIGNED] [ZEROFILL]

        一个大整数。签名范围是-9223372036854775808到9223372036854775807。无符号范围是0到18446744073709551615。相当于八个字节(64bit)

        SERIAL是BIGINT无符号NOT NULL AUTO_INCREMENT唯一的别名。

    * DECIMAL[(M[,D])] [UNSIGNED] [ZEROFILL]

        一个压缩的“精确”定点数字。M为总位数(精度)，D为小数点后的位数(刻度)。小数点和(负数)符号不在m中计数。如果D为0，则值没有小数点或小数部分。小数的最大位数(M)是65。支持的最大小数数(D)为30。如果省略D，默认值是0。如果省略M，默认值是10。

    * DEC[(M[,D])] [UNSIGNED] [ZEROFILL], NUMERIC[(M[,D])] [UNSIGNED] [ZEROFILL], FIXED[(M[,D])] [UNSIGNED] [ZEROFILL]

        这些类型是DECIMAL的同义词。固定的同义词可用于与其他数据库系统的兼容性。

    * FLOAT[(M,D)] [UNSIGNED] [ZEROFILL]

        一个小的(单精度)浮点数。允许值为-3.402823466E+38至-1.175494351E-38, 0，和1.175494351E-38至3.402823466E+38。这些是基于IEEE标准的理论限制。根据您的硬件或操作系统，实际范围可能会稍微小一些。

        M是总位数，D是小数点后的位数。如果省略M和D，则值存储在硬件允许的范围内。单精度浮点数精确到大约7位小数。

    * FLOAT(p) [UNSIGNED] [ZEROFILL]

        一个浮点数。p表示以位为单位的精度，但是MySQL只使用这个值来决定对结果数据类型使用FLOAT还是DOUBLE。

        如果p是从0到24，那么数据类型将变成没有M或D值的浮点数。
        如果p是从25到53，则数据类型变为DOUBLE，没有M或D值。

        结果列的范围与本节前面描述的单精度浮点数或双精度双数据类型相同。

    * DOUBLE[(M,D)] [UNSIGNED] [ZEROFILL]

        一个正常大小(双精度)的浮点数。允许值为-1.7976931348623157E+308 to -2.2250738585072014E-308, 0，和2.2250738585072014E-308 to 1.7976931348623157E+308。这些是基于IEEE标准的理论限制。根据您的硬件或操作系统，实际范围可能会稍微小一些。

        M是总位数，D是小数点后的位数。如果省略M和D，则值存储在硬件允许的范围内。双精度浮点数精确到大约15位小数。

    * DOUBLE PRECISION[(M,D)] [UNSIGNED] [ZEROFILL], REAL[(M,D)] [UNSIGNED] [ZEROFILL]

        这些类型是DOUBLE的同义词。

2. 日期和时间类型概观

    * DATE

        一个日期。支持的范围是“1000-01-01”到“9999-12-31”。MySQL以'YYYY-MM-DD'格式显示日期值，但允许使用字符串或数字对日期列赋值。

    * DATETIME[(fsp)]

        日期和时间的组合。支持的范围是“1000-01-01 00:00:00. 00.000000”到“9999-12-31 23:59:59.999999”。MySQL以'YYYY-MM-DD hh:mm:ss[.fraction]'格式显示日期时间值，但允许使用字符串或数字将值赋值给日期时间列。

        可以给出0到6范围内的可选fsp值，以指定小数秒精度。值为0表示没有小数部分。如果省略，默认精度为0。

    * TIMESTAMP[(fsp)]

        一个时间戳。范围是'1970-01-01 00:00:01.000000'UTC到'2038-01-19 03:14:07.999999'UTC。时间戳值存储为自epoch ('1970-01-01 00:00:00' UTC)以来的秒数。时间戳不能表示值'1970-01-01 00:00:00'，因为它等于从epoch开始的0秒，而值0保留用于表示'00:00 -00 00 00:00:00'，即'0'时间戳值。

    * TIME[(fsp)]

        一段时间。范围是'-838:59:59.000000'到'838:59:59.000000'。MySQL以'hh:mm:ss[.fraction]'格式显示时间值，但允许使用字符串或数字对时间列赋值。

    * YEAR[(4)]

        四位数的一年。MySQL以YYYY格式显示年份值，但允许使用字符串或数字对年份列赋值。值显示为1901到2155和0000。

3. 字符串类型概观

    * [NATIONAL] CHAR[(M)] [CHARACTER SET charset_name] [COLLATE collation_name]

        一个固定长度的字符串，存储时总是将空格右填充到指定长度。M表示以字符为单位的列长度。M的范围是0到255。如果省略M，长度是1。

    * [NATIONAL] VARCHAR(M) [CHARACTER SET charset_name] [COLLATE collation_name]

        一个变长字符串。M表示字符的最大列长度。M的范围是0到65535。VARCHAR的有效最大长度取决于最大行大小(所有列共享的65,535字节)和使用的字符集。

        例如，utf8字符每个字符最多需要三个字节，因此使用utf8字符集的VARCHAR列最多可以声明为21,844个字符。(21844=65535/3)

    * BINARY[(M)]

        二进制类型类似于CHAR类型，但是存储二进制字节字符串而不是非二进制字符串。可选长度M表示以字节为单位的列长度。如果省略，M默认为1。

    * VARBINARY(M)

        VARBINARY类型与VARCHAR类型类似，但是存储二进制字节字符串而不是非二进制字符串。M表示以字节为单位的最大列长度。

    * TINYBLOB

        一个最大长度为255(2^8−1)字节的BLOB列。每个TINYBLOB值都使用一个1字节长度的前缀存储，该前缀表示值中的字节数。

    * TINYTEXT [CHARACTER SET charset_name] [COLLATE collation_name]

        文本列，最大长度255(2^8−1)个字符。如果该值包含多字节字符，则有效最大长度将更小。每个TINYTEXT值都使用一个1字节长度的前缀来存储，该前缀表示值中的字节数。

    * BLOB[(M)]

        最大长度为65,535(2^16−1)字节的BLOB列。每个BLOB值都使用一个2字节长度的前缀存储，该前缀表示值中的字节数。

    * TEXT[(M)] [CHARACTER SET charset_name] [COLLATE collation_name]

        文本列，最大长度为65,535(2^16−1)个字符。如果该值包含多字节字符，则有效最大长度将更小。每个文本值都使用一个2字节长度的前缀来存储，该前缀表示值中的字节数

    * MEDIUMBLOB

        最大长度为16,777,215(2^24−1)字节的BLOB列。每个MEDIUMBLOB值都使用一个3字节长度的前缀存储，该前缀表示值中的字节数。

    * MEDIUMTEXT [CHARACTER SET charset_name] [COLLATE collation_name]

        最大长度为16,777,215(2^24−1)个字符的文本列。如果该值包含多字节字符，则有效最大长度将更小。每个MEDIUMTEXT值都使用一个3字节长度的前缀来存储，该前缀表示值中的字节数。

    * LONGBLOB

        最大长度为4,294,967,295或4GB(2^32−1)字节的BLOB列或者4GB (2^32 − 1) 字节.

    * LONGTEXT [CHARACTER SET charset_name] [COLLATE collation_name]

        最大长度为4,294,967,295或4GB(2^32−1)字符的文本列。

    * ENUM('value1','value2',...) [CHARACTER SET charset_name] [COLLATE collation_name]

        枚举。从值列表中选择的只能有一个值的字符串对象，'value1','value2',...

    * SET('value1','value2',...) [CHARACTER SET charset_name] [COLLATE collation_name]

        一个可以有零个或多个值的字符串对象，每个值必须从值列表中选择，'value1','value2',...

参考：https://dev.mysql.com/doc/refman/8.0/en