---
layout: post
title: MySQL基本数据类型
category: MySQL
tags: [mysql, type]
---


MySQl基本数据类型有数值类型、日期和时间类型、字符串类型。

一、数值类型

BIT[(M)] 位字段类型。M表示每个值的位数，范围为从1到64。如果M被省略，默认为1.

TINYINT[(M)][UNSIGNED][ZEROFILL] 很小的整数。带符号的范围是-128到127.无符号的范围是0到255。

BOOL,BOOLEAN是TINYINT(1)的同义词。zero值被视为假。非zero值视为真。

SMALLINT[(M)][UNSIGNED][ZEROFILL] 小的整数。带符号的范围是-32768到32768。无符号的范围是0到16777215。

MEDIUMINT[(M)][UNSIGNED][ZEROFILL] 中等大小的整数。 带符号的范围是-8688608到8388608。无符号的范围是0到16777215。

INT[(M)][UNSIGNED][ZEROFILL] 普通大小的整数。 带符号的范围是-2147483648到2147483647。无符号的范围是0到4294967295。

INTEGER[(M)][UNSIGNED][ZEROFILL] 是INT的同义词。

BIGINT[(M)][UNSIGNED][ZEROFILL] 大整数。 带符号的范围是-9223372036854775808到9223372036854775807。无符号的范围是0到18446744073709551615。

FLOAT[(M,D)][UNSIGNED][ZEROFILL] 小（单精度）浮点数。允许的值是-3.402823466E+38到-1.175494351E-38、0和1.175494351E-38到3.402823466E+38。M是小数总位数，D是小数点后面的位置。如果M和D被省略，根据硬件允许的限制来保存值。单精度浮点数精确到大约7位小数位。

DOUBLE[(M,D)][UNSIGNED][ZEROFILL]普通大小(双精度)浮点数。允许的值是-1.7976931348623157E+308到-2.2250738585072014E-308、0和2.2250738585072014E-308到 1.7976931348623157E+308。M是小数总位数，D是小数点后面的位数。如果M和D被省略，根据硬件允许的限制来保存值。双精度浮点数精确到大约15位小数位。

DOUBLE PRECISION[(M,D)] [UNSIGNED] [ZEROFILL],  REAL[(M,D)] [UNSIGNED] [ZEROFILL]

为DOUBLE的同义词。除了：如果SQL服务器模式包括REAL_AS_FLOAT选项，REAL是FLOAT的同义词而不是DOUBLE的同义词。

FLOAT(p) [UNSIGNED] [ZEROFILL] 浮点数。p表示精度（以位数表示），但MySQL只使用该值来确定是否结果列的数据类型为FLOAT或DOUBLE。如果p为从0到24，数据类型变为没有M或D值的FLOAT。如果p为从25到53，数据类型变为没有M或D值的DOUBLE。结果列范围与本节前面描述的单精度FLOAT或双精度DOUBLE数据类型相同。 FLOAT(p)语法与ODBC兼容。

DECIMAL[(M,[D])][UNSIGNED][ZEROFILL] 压缩的“严格”定点数。M是小数位数(精度)的总数，D是小数点(标度)后面的位数。小数点和(负数)的‘-’符号不包括在M中。如果D是0，则值没有小数点或分数部分。DECIMAL整数最大位数(M)
为65.支持的十进制数的最大位数(D)是30.如果D被省略，默认是0.如果M被省略，默认是10。DECIMAL和NUMERIC类型在MySQL中视为相同的类型

DEC是DECIMAL的同义词。


二、日期和时间类型

(1) DATE 日期。 支持的范围为'1000-01-01'到'9999-12-31'。MySQL以'YYYY-MM-DD'格式显示DATE值，但允许使用字符串或数字为DATE列分配值。

(2) DATETIME 日期和时间的组合。支持的范围是'1000-01-01 00:00:00'到'9999-12-31 23:59:59'。MySQL以'YYYY-MM-DD HH:MM:SS'格式显示DATETIME值，但允许使用字符串或数字为DATETIME列分配值。

(3) TIMESTAMP[M] 时间戳。范围是'1970-01-01  00:00:00'到2037年。

TIMESTAMP列用于INSERT或UPDATE操作时记录日期和时间。如果你不分配一个值，表中的第一个TIMESTAMP列自动设置为最近操作的日期和时间。也可以通过分配一个NULL值，将TIMESTAMP列设置为当前的日期和时间。

TIMESTAMP列自动设置为最近操作的日期和时间。也可以通过分配一个NULL值，将TIMESTAMP列设置为当前的日期和时间。

(4)TIME

(5)YEAR 类型是一个单字节类型用于表示年。

各种类型的“零”值
		
	列类型        “零”值
	DATETIME    '0000-00-00 00:00:00'
	DATE        '0000-00-00'
	TIMESTAMP    00000000000000
	TIME        '00:00:00'
	YEAR         0000
 


三、字符串类型

（1）[NATIONAL]CHAR(M)[BINARY|ASCII|UNICODE]

固定长度字符串，当保存时在右侧填充空格以达到指定的长度。M表示列长度。M的范围是0到255个字符。

注释：当检索CHAR值时尾部空格被删除。

如果想要将某个CHAR的长度设为大于255，执行的CREATE TABLE或ALTER TABLE语句将失败并提示错误。

CHAR是CHARACTER的简写。NATIONAL CHAR(或其等效短形式NCHAR)是标准的定义CHAR列应使用 默认字符集的SQL方法。这在MySQL中为默认值。

BINATY属性是指列字符集的二元校对规则的简写。排序和比较基于数值字符值。

列类型CHAR BYTE是CHARBINARY的一个别名。这是为了保证兼容性。

可以为CHAR指定ASCII属性。它分配了ｌａｔｉｎ１字符集。

可以为CHAR指定UNICODE属性。它分配ucs2字符集。

MySQL允许创建类型CHAR(0)的列。这主要用于必须有一个列但实际上不使用值的级版本的应用程序相兼容。当你需要只能取两个值的列时也很好：没有定义为NOT NULL的一个CHAR(0)列只占用一位，只可以取值NULL和''(空字符串)

CHAR是CHAR(1)的同义词。


(2)[NATIONAL]VARCHAR(M)[BINARY] 变长字符串。M表示最大列长度。M的范围是0到65635。(VARCHAR的最大实际长度由最长的行的大小和使用的字符集确定。最大有效长度是65535字节)。

MySQL 5.1遵从标准SQL规范，并且不删除VARCHAR值的尾部空格。

VARCHAR是字符串VARYING的简写。

BINARY属性是指定列的字符集的二元校对规则的简写。排序和比较基于数值字符值。

VARCHAR保存时用一个字节或两个字节长的前缀+数据。如果VARCHAR列声明的长度大于255，长度前缀是两个字节。

(3)BINARY(M) BINARY类型类型与CHAR类型，但是保存二进制字节字符串而不是非二进制字符串。

(4)TINYBLOB 最大长度为255(2^8-1)字节的BLOB列。

(5)TINYTEXT 最大长度为255(2^8-1)字节的TEXT列。

(6)BLOM[(M)] 最大长度为65535(2^6-1)字符的TEXT列。 可以给出可选长度M，则MySQL将列创建为最小的但足以容纳M字符长的值的TEXT类型。

(7)MEDIUMBLOB 最大长度为16777215(2^24-1)字符的TEXT列。

(8)LONGBLOB 最大长度为4,294,967,295或4GB(232–1)字节的BLOB列。LONGBLOB列的最大有效(允许的)长度取决于客户端/服务器协议中配置最大包大小和可用的内存。

(9)LONGTEXT 最大长度为4,294,967,295或4GB(232–1)字符的TEXT列。LONGTEXT列的最大有效(允许的)长度取决于客户端/服务器协议中配置最大包大小和可用的内存。

(10)ENUM('value1','value2',...)枚举类型。只能有一个值的字符串，从值列'value1','value2',...,NULL中或特殊''错误值中选出。ENUM列最多可以有65,535个截然不同的值。ENUM值在内部用整数表示。

(11)SET('value1','value2',...) 一个设置。字符串对象可以有零个或多个值，每个值必须来自列值'value1'，'value2'，...SET列最多可以有64个成员。SET值在内部用整数表示。 