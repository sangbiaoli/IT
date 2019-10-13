## 数据类型

1. 数据类型概观

    数据类型描述使用这些约定:

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

            

    2. 日期和时间类型概观
    3. 字符串类型概观

2. 数据类型
    1. 

参考：https://dev.mysql.com/doc/refman/8.0/en