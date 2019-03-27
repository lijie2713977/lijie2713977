---
title: 消息传送基础
date: 2016-05-21 23:43:49
tags: [oracle,数据库]
categories: [数据库,oracle]
---

#二进制串：RAW类型 #

文章摘自《Oracle Database 9i10g11g编程艺术 深入数据库体系结构（第2版）》

    Oracle除了支持文本，还支持二进制数据的存储。前面讨论了CHAR和VARCHAR2类型需要进行字符集转换，而二进制数据不会做这种字符集转换。因此，二进制数据类型不适合存储用户提供的文本，而适于存储加密信息，加密数据不是“文本”，而是原文本的一个二进制表示、包含二进制标记信息的字处理文档，等等。如果数据库不认为某些数据是“文本”(或任意其他基本数据类型，如，数值型、日期型等)，这些数据就应该采用一种二进制数据类型来存储，另外不应该应用字符集转换的数据也要使用二进制数据类型存储。

Oracle支持下面3种数据类型来存储二进制数据。
A.  RAW类型，这是这一节强调的重点，它很适合存储多达2000字节的RAW数据。
B.  BLOB类型，它支持更大的二进制数据，我们将在12.7节中再做介绍。
C.  LONG　RAW类型，这是为支持向后兼容性提供的，新应用不应考虑使用这个类型。

二进制RAW类型的语法很简单：
RAW(<size>)
例如，以下代码创建了一个每行能存储16字节二进制信息的表：
SQL> create table t(raw_data raw(16));
Table created.
SQL>
Ops$tkyte@ORA11GR2>create table t(raw_data raw(16));
Table created.
    从磁盘上的存储来看，RAW类型与VARCHAR2类型很相似。RAW类型是一个变长的二进制串，这说明前面创建的表T可以存储0-16字节的二进制数据。它不会像CHAR类型那样用空格填充。
    处理RAW数据时，你可能会发现它被隐式地转换为一个VARCHAR2类型，也就是说，诸如SQL*Plus之类的许多工具不会直接显示RAW数据，而是会将其转换为一种十六进制格式来显示，在以下例子中，我们使用SYS_GUID()在表中创建一些二进制数据，SYS_GUID()是一个内置函数，将返回一个全局唯一的16字节RAW串(GUID就代表全局唯一标识符，Globally Unique Identifier)：

SQL> insert into t values(sys_guid());
RAW_DATA
--------------------------------
370798BAA57BEAF0E05001A8653002BE
SQL>

    在此，你会马上注意到两点。首先，RAW数据看上去就像是一个字符串。SQL*Plus就是以字符串形式获取和打印RAW数据，但是RAW数据在磁盘上并不存储为字符串。SQL*Plus不能在屏幕上打印任意的二进制数据，因为这可能对显示有严重的副作用。要记住，二进制数据可能包含诸如回车或换行等控制字符，还可能是一个Ctrl+G字符，这会导致终端发出”嘟嘟“的叫声。
    其次，RAW数据看上去远远大于16字节，实际上，在这个例子中，你会看到32个字符。这是因为，每个二进制字节都显示为两个十六进制字符。所存储的RAW数据其实长度就是16字节，可以使用Oracle DUMP函数确认这一点。在此，我转储了这个二进制串的值，并使用了一个可选参数来指定显示各个字节值时应使用哪一种进制。这里使用了基数16，从而能将转储的结果与前面的串进行比较：

SQL> select dump(raw_data,16) from t;
DUMP(RAW_DATA,16)
----------------------------------------------
Typ=23 Len=16: 37,7,98,ba,a5,7b,ea,f0,e0,50,1,a8,65,30,2,be
SQL>

DUMP显示出，这个二进制串实际上长度为16字节（LEN=16）,另外还逐字节地显示了这个二进制数据。可以看到，这个转储显示与SQL*Plus将RAW数据获取为一个串时所执行的隐式转换是匹配的。
另一个方向上（插入）也会执行隐式转换：
SQL> insert into t values( 'abcdef' ); 
1 row created.

    这不会插入串abcdef，而会插入一个3字节的RAW数据，其字节分别是AB、CD、EF，如果用十进制表示则为字节171、205、239。如果试图使用一个包含非法16进制字符的串，就会收到一个错误消息：

SQL> insert into t values( 'abcdefgh' );
insert into t values( 'abcdefgh' )
                            *
ERROR at line 1:
ORA-01465: invalid hex number
SQL>

    RAW类型可以加索引，还能在谓词中使用，它与其他任何数据类型有同样的功能。不过，必须当心避免不希望的隐式转换，而且必须知道确实会发生隐式转换。
    在任何情况下我都喜欢使用显式转换，而且推荐这种做法，可以使用以下内置函数来执行这种操作。
A.  HEXTORAW：将十六进制字符串转换为RAW类型。
B.  RAWTOHEX：将RAW串转换为十六进制串。
SQL*Plus将RAW类型获取为一个串时，会隐式地调用RWATOHEX函数，而插入串时会隐式地调用HEXTORAW函数。应该避免隐式转换，而在编写代码时总是使用显式转换，这是一个很好的实践做法。
所以当前的例子应该写作：
SQL> select rawtohex(raw_data) from t
RAWTOHEX(RAW_DATA)
--------------------------------
370798BAA57BEAF0E05001A8653002BE
ABCDEF
SQL> insert into t values( hextoraw('abcdef') );
1 row created.

