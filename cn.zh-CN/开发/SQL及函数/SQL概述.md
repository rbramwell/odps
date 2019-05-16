# SQL概述 {#concept_awk_jmb_5db .concept}

本文为您介绍MaxCompute SQL的关键字、类型转换说明、分区表、UNION ALL运算及使用限制。

## SQL概述 {#section_tdn_ppg_hfb .section}

MaxCompute SQL适用于海量数据（GB、TB、EB级别），离线批量计算的场合。MaxCompute作业提交后会有几十秒到数分钟不等的排队调度，所以适合处理跑批作业，一次作业批量处理海量数据，不适合直接对接需要每秒处理几千至数万笔事务的前台业务系统。

MaxCompute SQL采用的是类似于SQL的语法。它的语法是标准语法ANSI SQL92的一个子集，并有自己的扩展，但不能因此简单地把MaxCompute等价成一个数据库，它在很多方面并不具备数据库的特征，如事务、主键约束、索引等。目前在MaxCompute中允许的最大SQL长度是3MB。

## 关键字 {#section_nzx_m2l_vdb .section}

MaxCompute将SQL语句的关键字作为保留字。在对表、列或是分区命名时如若使用关键字，需给关键字加````符号进行转义，否则会报错。保留字不区分大小写。下面只给出常用的保留字列表，完整的保留字列表请参见[MaxCompute SQL保留字](intl.zh-CN/开发/SQL及函数/附录/保留字与关键字.md#)。

```
%    &    &&    (    )    *    +  
 -    .    /    ;    <    <=    <>  
 =    >    >=    ?    ADD    ALL    ALTER  
 AND  AS    ASC    BETWEEN    BIGINT    BOOLEAN    BY  
 CASE CAST  COLUMN    COMMENT    CREATE    DESC    DISTINCT  
 DISTRIBUTE    DOUBLE    DROP    ELSE    FALSE    FROM    FULL  
 GROUP    IF    IN    INSERT    INTO    IS    JOIN  
 LEFT    LIFECYCLE    LIKE    LIMIT    MAPJOIN    NOT    NULL  
 ON    OR    ORDER    OUTER    OVERWRITE    PARTITION    RENAME  
 REPLACE    RIGHT    RLIKE    SELECT    SORT    STRING    TABLE  
 THEN    TOUCH    TRUE    UNION    VIEW    WHEN    WHERE
```

## 类型转换说明 {#section_k1n_qfl_vdb .section}

MaxCompute SQL允许数据类型之间的转换，类型转换方式包括显式类型转换和隐式类型转换。更多详情请参见[类型转换](intl.zh-CN/开发/SQL及函数/类型转换.md#)。

-   显式类型转换：是指用cast将一种数据类型的值转换为另一种类型的值的行为。
-   隐式类型转换：是指在运行时，由MaxCompute依据上下文使用环境及类型转换规则自动进行的类型转换。隐式转换作用域包括各种运算符、内建函数等作用域。

## 分区表 {#section_m1n_qfl_vdb .section}

MaxCompute SQL支持分区表。指定分区表会对您带来诸多便利，例如提高SQL运行效率、减少计费等。关于分区的详情请参见[基本概念\>分区](intl.zh-CN/开发/基本概念/分区.md#)。

## UNION ALL {#section_n1n_qfl_vdb .section}

参与UNION ALL运算的所有列的数据类型、列个数、列名称必须完全一致，否则会报异常。

## 使用限制 {#section_zrv_jcm_sfb .section}

SQL限制项请参见[SQL限制项](intl.zh-CN/开发/SQL及函数/SQL限制项.md#)，不支持的DDL及DML语法请参见[与其他SQL语法的差异](intl.zh-CN/开发/SQL及函数/与其他SQL语法的差异.md#)。

**说明：** 与其他SQL的差异和不兼容带来的限制，可参见[与标准SQL的主要区别及解决方法](../../../../intl.zh-CN/实践经验/SQL/与标准SQL的主要区别及解决方法.md#)及[修改不兼容SQL实战](../../../../intl.zh-CN/实践经验/SQL/修改不兼容SQL实战.md#)解决。

其他限制：

-   [SCALAR SUBQUERY限制](intl.zh-CN/开发/SQL及函数/SELECT语句/子查询.md#section_f52_1w5_hfb)
-   Insert values限制：values必须是常量
-   MaxCompute最多允许256个表的union all/union
-   Mapjoin的小表不能超过512MB
-   由于国际标准化组织发布的中国时区信息调整，执行相关SQL时，日期显示某些时间段会存在时间差异：1900-1928年的日期时间差异5分52秒，1900年之前的日期时间差异9秒。

